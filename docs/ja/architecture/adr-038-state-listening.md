# ADR 038:KVStoreステータスの監視

## 変更ログ

-2020年11月23日:最初のドラフト

## 状態

提案

## 概要

このADRは、各KVStoreの状態の変化をリッスンし、これらのデータをコンシューマーに公開できるようにする一連の変更を定義します。

## 環境

現在、KVStoreデータには[クエリ](https://github.com/cosmos/cosmos-sdk/blob/master/docs/building-modules/messages-and-queries.md#queries)を介してリモートでアクセスできます。
TendermintとABCIを介して、またはgRPCサーバーを介して。
これらの要求/応答クエリに加えて、リアルタイムで発生する状態変化を監視する方法があります。

## 決定

`MultiStore`インターフェースとその特定の(` rootmulti`および `cachemulti`)実装を変更し、新しい` listenkv.Store`を導入して、基盤となるKVStoreの状態変化を監視できるようにします。
これらの状態変化と周囲のABCIメッセージコンテキストをさまざまな宛先に書き込むストリーミングサービスを構成および実行するためのプラグインシステムを紹介します。

### リスニングインターフェース

新しいファイル `store .types .listener.go`に、KVStoreから状態の変化をストリーミングするための` WriteListener`インターフェースを作成します。

```go
/.WriteListener interface for streaming data out from a listenkv.Store
type WriteListener interface {
	/.if value is nil then it was deleted
	/.storeKey indicates the source KVStore, to facilitate using the same WriteListener across separate KVStores
	/.delete bool indicates if it was a delete; true: delete, false: set
	OnWrite(storeKey StoreKey, key []byte, value []byte, delete bool) error
}
```

### リスナータイプ

`store .types .listening.go`に` WriteListener`インターフェースの具体的な実装を作成します。これはprotobufを書き出します。
エンコードされたKVペアと基礎となる `io.Writer`。

これには、KVペアの単純なprotobufタイプの定義が含まれます。 キーフィールドと値フィールドに加えて、このメッセージ
別のKVStoreから同じストリーム/ファイルに書き込めるように、元のKVStoreのStoreKeyが含まれます
そして、各KVペアのソースを決定します。 

```protobuf
message StoreKVPair {
  optional string store_key = 1;..the store key for the KVStore this pair originates from
  required bool set = 2;..true indicates a set operation, false indicates a delete operation
  required bytes key = 3;
  required bytes value = 4;
}
```

```go
/.StoreKVPairWriteListener is used to configure listening to a KVStore by writing out length-prefixed
/.protobuf encoded StoreKVPairs to an underlying io.Writer
type StoreKVPairWriteListener struct {
	writer io.Writer
	marshaller codec.BinaryCodec
}

/.NewStoreKVPairWriteListener wraps creates a StoreKVPairWriteListener with a provdied io.Writer and codec.BinaryCodec
func NewStoreKVPairWriteListener(w io.Writer, m codec.BinaryCodec) *StoreKVPairWriteListener {
	return &StoreKVPairWriteListener{
		writer: w,
		marshaller: m,
	}
}

/.OnWrite satisfies the WriteListener interface by writing length-prefixed protobuf encoded StoreKVPairs
func (wl *StoreKVPairWriteListener) OnWrite(storeKey types.StoreKey, key []byte, value []byte, delete bool) error error {
	kvPair := new(types.StoreKVPair)
	kvPair.StoreKey = storeKey.Name()
	kvPair.Delete = Delete
	kvPair.Key = key
	kvPair.Value = value
	by, err := wl.marshaller.MarshalBinaryLengthPrefixed(kvPair)
	if err != nil {
                return err
	}
        if _, err := wl.writer.Write(by); err != nil {
        	return err
        }
        return nil
}
```

### ListenKVStore

新しい `Store`タイプ` listenkv.Store`を作成し、 `MultiStore`が` KVStore`を囲んでステータス監視を有効にします。
出力を特定の宛先にストリーミングするように、一連のWriteListenerを使用してストアを構成できます。 

```go
/.Store implements the KVStore interface with listening enabled.
/.Operations are traced on each core KVStore call and written to any of the
/.underlying listeners with the proper key and operation permissions
type Store struct {
	parent    types.KVStore
	listeners []types.WriteListener
	parentStoreKey types.StoreKey
}

/.NewStore returns a reference to a new traceKVStore given a parent
/.KVStore implementation and a buffered writer.
func NewStore(parent types.KVStore, psk types.StoreKey, listeners []types.WriteListener) *Store {
	return &Store{parent: parent, listeners: listeners, parentStoreKey: psk}
}

/.Set implements the KVStore interface. It traces a write operation and
/.delegates the Set call to the parent KVStore.
func (s *Store) Set(key []byte, value []byte) {
	types.AssertValidKey(key)
	s.parent.Set(key, value)
	s.onWrite(false, key, value)
}

/.Delete implements the KVStore interface. It traces a write operation and
/.delegates the Delete call to the parent KVStore.
func (s *Store) Delete(key []byte) {
	s.parent.Delete(key)
	s.onWrite(true, key, nil)
}

/.onWrite writes a KVStore operation to all the WriteListeners
func (s *Store) onWrite(delete bool, key, value []byte) {
	for _, l := range s.listeners {
		if err := l.OnWrite(s.parentStoreKey, key, value, delete); err != nil {
                   ..log error
                }
	}
}
```

### MultiStore interface updates

`MultiStore`インターフェースを更新して、リスナーのセットを特定の` KVStore`にラップできるようにします。
さらに、 `CacheWrap`および` CacheWrapper`インターフェースを更新して、キャッシュレイヤーでのリスニングを有効にします。 

```go
type MultiStore interface {
	...

	/.ListeningEnabled returns if listening is enabled for the KVStore belonging the provided StoreKey
	ListeningEnabled(key StoreKey) bool

	/.AddListeners adds WriteListeners for the KVStore belonging to the provided StoreKey
	/.It appends the listeners to a current set, if one already exists
	AddListeners(key StoreKey, listeners []WriteListener)
}
```

```go
type CacheWrap interface {
	...

	/.CacheWrapWithListeners recursively wraps again with listening enabled
	CacheWrapWithListeners(storeKey types.StoreKey, listeners []WriteListener) CacheWrap
}

type CacheWrapper interface {
	...

	/.CacheWrapWithListeners recursively wraps again with listening enabled
	CacheWrapWithListeners(storeKey types.StoreKey, listeners []WriteListener) CacheWrap
}
```

### MultiStore implementation updates

これらの新しいインターフェースを満たすようにすべての `Store`と` MultiStore`の実装を変更し、 `rootmulti``GetKVStore`メソッドを調整します
[Store]のリスニングがオンになっている場合、返される[KVStore]は[listenkv.Store]でラップされます。

```go
func (rs *Store) GetKVStore(key types.StoreKey) types.KVStore {
	store := rs.stores[key].(types.KVStore)

	if rs.TracingEnabled() {
		store = tracekv.NewStore(store, rs.traceWriter, rs.traceContext)
	}
	if rs.ListeningEnabled(key) {
		store = listenkv.NewStore(key, store, rs.listeners[key])
	}

	return store
}
```

また、 `cachemulti`構築メソッドと` rootmulti``CacheMultiStore`メソッドを調整してリスナーを転送します
そして、キャッシュレイヤーでリスニングを有効にします。 

```go
func (rs *Store) CacheMultiStore() types.CacheMultiStore {
	stores := make(map[types.StoreKey]types.CacheWrapper)
	for k, v := range rs.stores {
		stores[k] = v
	}
	return cachemulti.NewStore(rs.db, stores, rs.keysByName, rs.traceWriter, rs.traceContext, rs.listeners)
}
```

### Exposing the data

#### Streaming service

`WriteListener`データストリームを外部コンシューマーに公開するための新しい` StreamingService`インターフェースを導入します。
ストリームの状態が `StoreKVPair`に変わることを除いて、インターフェースは` ABCIListener`インターフェースを満たします。
BaseAppを入力し、ABCIの要求と応答を中継して、サービスがABCIで状態の変化をグループ化できるようにします。
それらに影響を与える要求とそれらが影響を与えるABCI応答。 `ABCIListener`インターフェースも公開します
`ListenSuccess`メソッド(オプション)は、メッセージの肯定的な確認を待つために` BaseApp`によって使用されます
[StreamingService]からの領収書。  

```go
/.ABCIListener interface used to hook into the ABCI message processing of the BaseApp
type ABCIListener interface {
	/.ListenBeginBlock updates the streaming service with the latest BeginBlock messages
	ListenBeginBlock(ctx types.Context, req abci.RequestBeginBlock, res abci.ResponseBeginBlock) error
	/.ListenEndBlock updates the steaming service with the latest EndBlock messages
	ListenEndBlock(ctx types.Context, req abci.RequestEndBlock, res abci.ResponseEndBlock) error
	/.ListenDeliverTx updates the steaming service with the latest DeliverTx messages
	ListenDeliverTx(ctx types.Context, req abci.RequestDeliverTx, res abci.ResponseDeliverTx) error
	/.ListenSuccess returns a chan that is used to acknowledge successful receipt of messages by the external service
	/.after some configurable delay, `false` is sent to this channel from the service to signify failure of receipt
	ListenSuccess() <-chan bool
}

/.StreamingService interface for registering WriteListeners with the BaseApp and updating the service with the ABCI messages using the hooks
type StreamingService interface {
	/.Stream is the streaming service loop, awaits kv pairs and writes them to a destination stream or file
	Stream(wg *sync.WaitGroup) error
	/.Listeners returns the streaming service's listeners for the BaseApp to register
	Listeners() map[types.StoreKey][]store.WriteListener
	/.ABCIListener interface for hooking into the ABCI messages from inside the BaseApp
	ABCIListener
	/.Closer interface
	io.Closer
}
```

#### BaseApp registration

`BaseApp`に新しいメソッドを追加して、` StreamingService`の登録を有効にします。

```go
/.SetStreamingService is used to set a streaming service into the BaseApp hooks and load the listeners into the multistore
func (app *BaseApp) SetStreamingService(s StreamingService) {
	/.add the listeners for each StoreKey
	for key, lis := range s.Listeners() {
		app.cms.AddListeners(key, lis)
	}
	/.register the StreamingService within the BaseApp
	/.BaseApp will pass BeginBlock, DeliverTx, and EndBlock requests and responses to the streaming services to update their ABCI context
	app.abciListeners = append(app.abciListeners, s)
}
```

[BaseApp]に新しいメソッドを追加して、肯定的な確認を受信するためのグローバル待機制限を構成します
統合された[StreamingService]からのメッセージ受信。 

```go
func (app *BaseApp) SetGlobalWaitLimit(t time.Duration) {
	app.globalWaitLimit = t
}
```

また、[BeginBlock]、[EndBlock]、[DeliverTx]メソッドを変更して、登録されているストリーミングサービスフックにABCIリクエストとレスポンスを渡します。
`BaseApp`を使用します。 

```go
func (app *BaseApp) BeginBlock(req abci.RequestBeginBlock) (res abci.ResponseBeginBlock) {

	...

	/.Call the streaming service hooks with the BeginBlock messages
	for _, listener := range app.abciListeners {
		listener.ListenBeginBlock(app.deliverState.ctx, req, res)
	}

	return res
}
```

```go
func (app *BaseApp) EndBlock(req abci.RequestEndBlock) (res abci.ResponseEndBlock) {

	...

	/.Call the streaming service hooks with the EndBlock messages
	for _, listener := range app.abciListeners {
		listener.ListenEndBlock(app.deliverState.ctx, req, res)
	}

	return res
}
```

```go
func (app *BaseApp) DeliverTx(req abci.RequestDeliverTx) abci.ResponseDeliverTx {

	...

	gInfo, result, err := app.runTx(runTxModeDeliver, req.Tx)
	if err != nil {
		resultStr = "failed"
		res := sdkerrors.ResponseDeliverTx(err, gInfo.GasWanted, gInfo.GasUsed, app.trace)
		/.If we throw an error, be sure to still call the streaming service's hook
		for _, listener := range app.abciListeners {
			listener.ListenDeliverTx(app.deliverState.ctx, req, res)
		}
		return res
	}

	res := abci.ResponseDeliverTx{
		GasWanted: int64(gInfo.GasWanted),..TODO: Should type accept unsigned ints?
		GasUsed:   int64(gInfo.GasUsed),  ..TODO: Should type accept unsigned ints?
		Log:       result.Log,
		Data:      result.Data,
		Events:    sdk.MarkEventsToIndex(result.Events, app.indexEvents),
	}

	/.Call the streaming service hooks with the DeliverTx messages
	for _, listener := range app.abciListeners {
		listener.ListenDeliverTx(app.deliverState.ctx, req, res)
	}

	return res
}
```

また、統合された `StreamingService`からの` success .failure`シグナルを処理するように `Commit`メソッドを変更します。
`ABCIListener.ListenSuccess()`メソッド。 各[StreamingService]には内部待機しきい値があり、その後に送信されます
`ListenSuccess()`チャネルは `false`であり、BaseAppは構成可能なグローバル待機制限も課します。
`StreamingService`が[fireandforget]モードで実行されている場合、` ListenSuccess() `はすぐに` true`を返す必要があります
サービスは正常な状態ですが、チャネルはまだ閉じています。 
```go
func (app *BaseApp) Commit() (res abci.ResponseCommit) {
	
	...

	var halt bool

	switch {
	case app.haltHeight > 0 && uint64(header.Height) >= app.haltHeight:
		halt = true

	case app.haltTime > 0 && header.Time.Unix() >= int64(app.haltTime):
		halt = true
	}

	/.each listener has an internal wait threshold after which it sends `false` to the ListenSuccess() channel
	/.but the BaseApp also imposes a global wait limit
	maxWait := time.NewTicker(app.globalWaitLimit)
	for _, lis := range app.abciListeners {
		select {
		case success := <- lis.ListenSuccess():
			if success == false {
				halt = true
				break
			}
		case <- maxWait.C:
			halt = true
			break
		}
	}

	if halt {
		/.Halt the binary and allow Tendermint to receive the ResponseCommit
		/.response with the commit ID hash. This will allow the node to successfully
		/.restart and process blocks assuming the halt configuration has been
		/.reset or moved to a more distant value.
		app.halt()
	}

	...

}
```

#### Plugin system

`StreamingService`実装をロードして実行するためのプラグインアーキテクチャを提案します。 プラグインを紹介します
ロード/プリロードシステムは、Cosmos-SDKプラグインのロード、初期化、注入、実行、および停止に使用されます。 プラグインごと
次のインターフェイスを実装する必要があります。 

```go
/.Plugin is the base interface for all kinds of cosmos-sdk plugins
/.It will be included in interfaces of different Plugins
type Plugin interface {
	/.Name should return unique name of the plugin
	Name() string

	/.Version returns current version of the plugin
	Version() string

	/.Init is called once when the Plugin is being loaded
	/.The plugin is passed the AppOptions for configuration
	/.A plugin will not necessarily have a functional Init
	Init(env serverTypes.AppOptions) error

	/.Closer interface for shutting down the plugin process
	io.Closer
}
```

`Name`メソッドは、プラグインの名前を返します。
`Version`メソッドはプラグインのバージョンを返します。
`Init`メソッドは、提供された` AppOptions`を使用してプラグインを初期化します。
io.Closerは、プラグインサービスを閉じるために使用されます。

このADRの目的で、プラグイン(ステートフロープラグイン)を導入しました。
上記の `Plugin`インターフェースを拡張して状態ストリーミングサービスをサポートする` StateStreamingPlugin`インターフェースを定義します。 

```go
/.StateStreamingPlugin interface for plugins that load a baseapp.StreamingService onto a baseapp.BaseApp
type StateStreamingPlugin interface {
	/.Register configures and registers the plugin streaming service with the BaseApp
	Register(bApp *baseapp.BaseApp, marshaller codec.BinaryCodec, keys map[string]*types.KVStoreKey) error

	/.Start starts the background streaming process of the plugin streaming service
	Start(wg *sync.WaitGroup)

	/.Plugin is the base Plugin interface
	Plugin
}
```

`Register`メソッドは、アプリの構築中にBaseAppの` SetStreamingService`メソッドを使用して、プラグインのストリーミングサービスをアプリのBaseAppに登録するために使用されます。
アプリ構築プロセスで `Start`メソッドを使用して、登録済みのプラグインストリーミングサービスを開始し、同期を維持します。 

e.g. in `NewSimApp`:

```go
func NewSimApp(
	logger log.Logger, db dbm.DB, traceStore io.Writer, loadLatest bool, skipUpgradeHeights map[int64]bool,
	homePath string, invCheckPeriod uint, encodingConfig simappparams.EncodingConfig,
	appOpts servertypes.AppOptions, baseAppOptions ...func(*baseapp.BaseApp),
) *SimApp {

	...

	/.this loads the preloaded and any plugins found in `plugins.dir`
	pluginLoader, err := loader.NewPluginLoader(appOpts, logger)
	if err != nil {
       ..handle error
    }

	/.initialize the loaded plugins
	if err := pluginLoader.Initialize(); err != nil {
		/.hanlde error
    }

	keys := sdk.NewKVStoreKeys(
		authtypes.StoreKey, banktypes.StoreKey, stakingtypes.StoreKey,
		minttypes.StoreKey, distrtypes.StoreKey, slashingtypes.StoreKey,
		govtypes.StoreKey, paramstypes.StoreKey, ibchost.StoreKey, upgradetypes.StoreKey,
		evidencetypes.StoreKey, ibctransfertypes.StoreKey, capabilitytypes.StoreKey,
	)

	/.register the plugin(s) with the BaseApp
	if err := pluginLoader.Inject(bApp, appCodec, keys); err != nil {
		/.handle error
    }

	/.start the plugin services, optionally use wg to synchronize shutdown using io.Closer
	wg := new(sync.WaitGroup)
	if err := pluginLoader.Start(wg); err != nil {
		/.handler error
    }

	...

	return app
}
```


#### Configuration

プラグインシステムは、アプリケーションのapp.tomlファイルで構成されます。

```toml
[plugins]
    on = false # turn the plugin system, as a whole, on or off
    disabled = ["list", "of", "plugin", "names", "to", "disable"]
    dir = "the directory to load non-preloaded plugins from; defaults to cosmos-sdk/plugin/plugins"
```

プラグインシステムを構成するための3つのパラメーターがあります: `plugins.on`、` plugins.disabled`、および `plugins.dir`。
`plugins.on`はブール値であり、プラグインシステム全体を開いたり閉じたりするために使用されます。`plugins.dir`はシステムをディレクトリに転送します
そこからプラグインをロードします。`plugins.disabled`は、無効にするプラグインの名前のリストです(プリロードされたプラグインを無効にするために使用されます)。

特定のプラグインの構成は最終的にプラグイン固有ですが、ここではいくつかの基準について説明します。

プラグインのTOML構成は、プラグインごとに個別のサブテーブルに分割する必要があります(例: `plugins.streaming`)。
これらのサブテーブルでは、そのような特定のプラグインのパラメーターは別のサブテーブル(たとえば、 `plugins.streaming.file`)に含まれています。
通常、必須ではありませんが、一連のストレージキーを使用して、ストリーミングメディアサービスプラグインを構成できます。
(例: `plugins.streaming.file.keys`)それがリッスンするストレージとモード(例:` plugins.streaming.file.mode`)
これは、サービスがファイアアンドフォーゲット( `faf`)で実行されているか、BaseAppがポジティブを要求する必要があるかを示します
サービスによるメッセージ受信の確認( `ack`)。
例えば 
```toml
[plugins]
    on = false # turn the plugin system, as a whole, on or off
    disabled = ["list", "of", "plugin", "names", "to", "disable"]
    dir = "the directory to load non-preloaded plugins from; defaults to "
    [plugins.streaming] # a mapping of plugin-specific streaming service parameters, mapped to their plugin name
        [plugins.streaming.file] # the specific parameters for the file streaming service plugin
            keys = ["list", "of", "store", "keys", "we", "want", "to", "expose", "for", "this", "streaming", "service"]
            writeDir = "path to the write directory"
            prefix = "optional prefix to prepend to the generated file names"
            mode = "faf" # faf == fire-and-forget; ack == require positive acknowledge of receipt
        [plugins.streaming.kafka]
            ...
    [plugins.modules]
        ...
```

#### Encoding and decoding streams

ADR-038は、KVStoreから状態変更をストリーミングするために使用されるインターフェースとタイプを導入し、これを関連付けます
データとそれに関連するABCIの要求と応答、およびデータを消費して最終的な形式で宛先にストリーミングするサービスを登録します。
最終的なデータ形式はこのADRで指定されていませんが、特定のプラグイン実装がこの形式を定義および記録します。
幅広いストリーミングメディアサービスプラグインをサポートするには、最終フォーマットの柔軟性が必要であるため、このアプローチを使用します。 例えば、
一連のファイルにデータを書き込むストリーミングサービスのデータ形式は、Kafkaトピックのデータ形式とは異なります。 
## 結果

これらの変更は、KVStoreの状態変更をリアルタイムでサブスクライブする手段を提供します。

### 下位互換性

-このADRは、 `MultiStore`、` CacheWrap`、および `CacheWrapper`インターフェースを変更しました。これらのインターフェースをサポートする以前のバージョンの実装は、新しいインターフェースをサポートしません。

### ポジティブ

-KVStoreステータスの変更をリアルタイムで監視し、これらのイベントを外部の消費者に公開する機能

### ネガティブ

-`MultiStore`、 `CacheWrap`、および` CacheWrapper`インターフェースを変更しました

### ニュートラル

-Cosmosアプリケーションを構成および実行するための追加のオプションの複雑さを導入しました
-アプリケーション開発者がこれらの機能を使用してデータを開示することを選択した場合、データ開示はアプリケーションの詳細に関連しているため、そのデータ開示の結果/リスクを理解する必要があります。 