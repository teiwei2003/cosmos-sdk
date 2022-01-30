# ADR 038:KVStore 状态监听

## 变更日志

- 11/23/2020:初稿

## 地位

建议的

## 摘要

此 ADR 定义了一组更改，以启用侦听各个 KVStore 的状态更改并将这些数据公开给消费者。

## 语境

目前，KVStore数据可以通过[查询]远程访问(https://github.com/cosmos/cosmos-sdk/blob/master/docs/building-modules/messages-and-queries.md#queries)
通过 Tendermint 和 ABCI，或通过 gRPC 服务器进行。
除了这些请求/响应查询之外，有一种方法可以监听实时发生的状态变化。

## 决定

我们将修改 `MultiStore` 接口及其具体(`rootmulti` 和 `cachemulti`)实现，并引入一个新的 `listenkv.Store` 以允许监听底层 KVStore 的状态变化。
我们将介绍一个用于配置和运行流服务的插件系统，这些服务将这些状态变化及其周围的 ABCI 消息上下文写入不同的目的地。

###收听界面

在一个新文件 `store/types/listening.go` 中，我们将创建一个 `WriteListener` 接口，用于从 KVStore 流出状态更改。

```go
//WriteListener interface for streaming data out from a listenkv.Store
type WriteListener interface {
	//if value is nil then it was deleted
	//storeKey indicates the source KVStore, to facilitate using the same WriteListener across separate KVStores
	//delete bool indicates if it was a delete; true: delete, false: set
	OnWrite(storeKey StoreKey, key []byte, value []byte, delete bool) error
}
```

### 监听器类型

我们将在`store/types/listening.go`中创建`WriteListener`接口的具体实现，它写出protobuf
编码的 KV 对与底层的 `io.Writer`。

这将包括为 KV 对定义一个简单的 protobuf 类型。 除了键和值字段，这条消息
将包含原始 KVStore 的 StoreKey，以便我们可以从单独的 KVStore 写出到同一个流/文件
并确定每个 KV 对的来源。 

```protobuf
message StoreKVPair {
  optional string store_key = 1;//the store key for the KVStore this pair originates from
  required bool set = 2;//true indicates a set operation, false indicates a delete operation
  required bytes key = 3;
  required bytes value = 4;
}
```

```go
//StoreKVPairWriteListener is used to configure listening to a KVStore by writing out length-prefixed
//protobuf encoded StoreKVPairs to an underlying io.Writer
type StoreKVPairWriteListener struct {
	writer io.Writer
	marshaller codec.BinaryCodec
}

//NewStoreKVPairWriteListener wraps creates a StoreKVPairWriteListener with a provdied io.Writer and codec.BinaryCodec
func NewStoreKVPairWriteListener(w io.Writer, m codec.BinaryCodec) *StoreKVPairWriteListener {
	return &StoreKVPairWriteListener{
		writer: w,
		marshaller: m,
	}
}

//OnWrite satisfies the WriteListener interface by writing length-prefixed protobuf encoded StoreKVPairs
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

我们将创建一个新的 `Store` 类型 `listenkv.Store`，`MultiStore` 环绕一个 `KVStore` 以启用状态监听。
我们可以使用一组 `WriteListener` 配置 `Store`，将输出流式传输到特定目的地。 

```go
//Store implements the KVStore interface with listening enabled.
//Operations are traced on each core KVStore call and written to any of the
//underlying listeners with the proper key and operation permissions
type Store struct {
	parent    types.KVStore
	listeners []types.WriteListener
	parentStoreKey types.StoreKey
}

//NewStore returns a reference to a new traceKVStore given a parent
//KVStore implementation and a buffered writer.
func NewStore(parent types.KVStore, psk types.StoreKey, listeners []types.WriteListener) *Store {
	return &Store{parent: parent, listeners: listeners, parentStoreKey: psk}
}

//Set implements the KVStore interface. It traces a write operation and
//delegates the Set call to the parent KVStore.
func (s *Store) Set(key []byte, value []byte) {
	types.AssertValidKey(key)
	s.parent.Set(key, value)
	s.onWrite(false, key, value)
}

//Delete implements the KVStore interface. It traces a write operation and
//delegates the Delete call to the parent KVStore.
func (s *Store) Delete(key []byte) {
	s.parent.Delete(key)
	s.onWrite(true, key, nil)
}

//onWrite writes a KVStore operation to all the WriteListeners
func (s *Store) onWrite(delete bool, key, value []byte) {
	for _, l := range s.listeners {
		if err := l.OnWrite(s.parentStoreKey, key, value, delete); err != nil {
                   //log error
                }
	}
}
```

### MultiStore interface updates

我们将更新 `MultiStore` 接口，以允许我们围绕特定的 `KVStore` 包装一组侦听器。
此外，我们将更新 `CacheWrap` 和 `CacheWrapper` 接口以启用缓存层中的侦听。 

```go
type MultiStore interface {
	...

	//ListeningEnabled returns if listening is enabled for the KVStore belonging the provided StoreKey
	ListeningEnabled(key StoreKey) bool

	//AddListeners adds WriteListeners for the KVStore belonging to the provided StoreKey
	//It appends the listeners to a current set, if one already exists
	AddListeners(key StoreKey, listeners []WriteListener)
}
```

```go
type CacheWrap interface {
	...

	//CacheWrapWithListeners recursively wraps again with listening enabled
	CacheWrapWithListeners(storeKey types.StoreKey, listeners []WriteListener) CacheWrap
}

type CacheWrapper interface {
	...

	//CacheWrapWithListeners recursively wraps again with listening enabled
	CacheWrapWithListeners(storeKey types.StoreKey, listeners []WriteListener) CacheWrap
}
```

### MultiStore implementation updates

我们将修改所有 `Store` 和 `MultiStore` 实现以满足这些新接口，并调整 `rootmulti``GetKVStore` 方法
如果为该“Store”打开了侦听，则用“listenkv.Store”包装返回的“KVStore”。 

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

我们还将调整`cachemulti`构造方法和`rootmulti``CacheMultiStore`方法来转发监听器
并在缓存层中启用侦听。

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

我们将引入一个新的 `StreamingService` 接口，用于将 `WriteListener` 数据流暴露给外部消费者。
除了流状态改变为`StoreKVPair`s，接口满足一个`ABCIListener`接口，插入
进入 BaseApp 并中继 ABCI 请求和响应，以便服务可以将状态更改与 ABCI 分组
影响他们的请求以及他们影响的 ABCI 响应。 `ABCIListener` 接口还公开了一个
`ListenSuccess` 方法(可选)由`BaseApp` 用于等待消息的肯定确认
来自“StreamingService”的收据。 

```go
//ABCIListener interface used to hook into the ABCI message processing of the BaseApp
type ABCIListener interface {
	//ListenBeginBlock updates the streaming service with the latest BeginBlock messages
	ListenBeginBlock(ctx types.Context, req abci.RequestBeginBlock, res abci.ResponseBeginBlock) error
	//ListenEndBlock updates the steaming service with the latest EndBlock messages
	ListenEndBlock(ctx types.Context, req abci.RequestEndBlock, res abci.ResponseEndBlock) error
	//ListenDeliverTx updates the steaming service with the latest DeliverTx messages
	ListenDeliverTx(ctx types.Context, req abci.RequestDeliverTx, res abci.ResponseDeliverTx) error
	//ListenSuccess returns a chan that is used to acknowledge successful receipt of messages by the external service
	//after some configurable delay, `false` is sent to this channel from the service to signify failure of receipt
	ListenSuccess() <-chan bool
}

//StreamingService interface for registering WriteListeners with the BaseApp and updating the service with the ABCI messages using the hooks
type StreamingService interface {
	//Stream is the streaming service loop, awaits kv pairs and writes them to a destination stream or file
	Stream(wg *sync.WaitGroup) error
	//Listeners returns the streaming service's listeners for the BaseApp to register
	Listeners() map[types.StoreKey][]store.WriteListener
	//ABCIListener interface for hooking into the ABCI messages from inside the BaseApp
	ABCIListener
	//Closer interface
	io.Closer
}
```

#### BaseApp registration

我们将向`BaseApp` 添加一个新方法以启用`StreamingService`s 的注册:

```go
//SetStreamingService is used to set a streaming service into the BaseApp hooks and load the listeners into the multistore
func (app *BaseApp) SetStreamingService(s StreamingService) {
	//add the listeners for each StoreKey
	for key, lis := range s.Listeners() {
		app.cms.AddListeners(key, lis)
	}
	//register the StreamingService within the BaseApp
	//BaseApp will pass BeginBlock, DeliverTx, and EndBlock requests and responses to the streaming services to update their ABCI context
	app.abciListeners = append(app.abciListeners, s)
}
```

我们将向“BaseApp”添加一个新方法，用于配置接收肯定确认的全局等待限制
来自集成的“StreamingService”的消息接收。 

```go
func (app *BaseApp) SetGlobalWaitLimit(t time.Duration) {
	app.globalWaitLimit = t
}
```

我们还将修改“BeginBlock”、“EndBlock”和“DeliverTx”方法，以将 ABCI 请求和响应传递给任何已注册的流服务挂钩
使用`BaseApp`。 

```go
func (app *BaseApp) BeginBlock(req abci.RequestBeginBlock) (res abci.ResponseBeginBlock) {

	...

	//Call the streaming service hooks with the BeginBlock messages
	for _, listener := range app.abciListeners {
		listener.ListenBeginBlock(app.deliverState.ctx, req, res)
	}

	return res
}
```

```go
func (app *BaseApp) EndBlock(req abci.RequestEndBlock) (res abci.ResponseEndBlock) {

	...

	//Call the streaming service hooks with the EndBlock messages
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
		//If we throw an error, be sure to still call the streaming service's hook
		for _, listener := range app.abciListeners {
			listener.ListenDeliverTx(app.deliverState.ctx, req, res)
		}
		return res
	}

	res := abci.ResponseDeliverTx{
		GasWanted: int64(gInfo.GasWanted),//TODO: Should type accept unsigned ints?
		GasUsed:   int64(gInfo.GasUsed),  //TODO: Should type accept unsigned ints?
		Log:       result.Log,
		Data:      result.Data,
		Events:    sdk.MarkEventsToIndex(result.Events, app.indexEvents),
	}

	//Call the streaming service hooks with the DeliverTx messages
	for _, listener := range app.abciListeners {
		listener.ListenDeliverTx(app.deliverState.ctx, req, res)
	}

	return res
}
```

我们还将修改 `Commit` 方法以处理来自集成的 `StreamingService` 的 `success/failure` 信号，使用
`ABCIListener.ListenSuccess()` 方法。 每个“StreamingService”都有一个内部等待阈值，在此阈值之后它会发送
`ListenSuccess()` 通道为 `false`，并且 BaseApp 还强加了一个可配置的全局等待限制。
如果 `StreamingService` 以“即发即弃”模式运行，`ListenSuccess()` 应立即返回 `true`
尽管服务处于成功状态，但仍关闭频道。

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

	//each listener has an internal wait threshold after which it sends `false` to the ListenSuccess() channel
	//but the BaseApp also imposes a global wait limit
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
		//Halt the binary and allow Tendermint to receive the ResponseCommit
		//response with the commit ID hash. This will allow the node to successfully
		//restart and process blocks assuming the halt configuration has been
		//reset or moved to a more distant value.
		app.halt()
	}

	...

}
```

#### Plugin system

我们提出了一个插件架构来加载和运行 `StreamingService` 实现。 我们将介绍一个插件
加载/预加载系统，用于加载、初始化、注入、运行和停止 Cosmos-SDK 插件。 每个插件
必须实现以下接口: 

```go
//Plugin is the base interface for all kinds of cosmos-sdk plugins
//It will be included in interfaces of different Plugins
type Plugin interface {
	//Name should return unique name of the plugin
	Name() string

	//Version returns current version of the plugin
	Version() string

	//Init is called once when the Plugin is being loaded
	//The plugin is passed the AppOptions for configuration
	//A plugin will not necessarily have a functional Init
	Init(env serverTypes.AppOptions) error

	//Closer interface for shutting down the plugin process
	io.Closer
}
```

`Name` 方法返回插件的名称。
`Version` 方法返回插件的版本。
`Init` 方法使用提供的 `AppOptions` 初始化插件。
io.Closer 用于关闭插件服务。

出于此 ADR 的目的，我们引入了一种插件——状态流插件。
我们将定义一个 `StateStreamingPlugin` 接口，它扩展了上面的 `Plugin` 接口以支持状态流服务。 

```go
//StateStreamingPlugin interface for plugins that load a baseapp.StreamingService onto a baseapp.BaseApp
type StateStreamingPlugin interface {
	//Register configures and registers the plugin streaming service with the BaseApp
	Register(bApp *baseapp.BaseApp, marshaller codec.BinaryCodec, keys map[string]*types.KVStoreKey) error

	//Start starts the background streaming process of the plugin streaming service
	Start(wg *sync.WaitGroup)

	//Plugin is the base Plugin interface
	Plugin
}
```

`Register` 方法用于在 App 构建期间使用 BaseApp 的 `SetStreamingService` 方法向 App 的 BaseApp 注册插件的流服务。
在 App 构建过程中使用 `Start` 方法来启动已注册的插件流服务并与其保持同步。 

e.g. in `NewSimApp`:

```go
func NewSimApp(
	logger log.Logger, db dbm.DB, traceStore io.Writer, loadLatest bool, skipUpgradeHeights map[int64]bool,
	homePath string, invCheckPeriod uint, encodingConfig simappparams.EncodingConfig,
	appOpts servertypes.AppOptions, baseAppOptions ...func(*baseapp.BaseApp),
) *SimApp {

	...

	//this loads the preloaded and any plugins found in `plugins.dir`
	pluginLoader, err := loader.NewPluginLoader(appOpts, logger)
	if err != nil {
       //handle error
    }

	//initialize the loaded plugins
	if err := pluginLoader.Initialize(); err != nil {
		//hanlde error
    }

	keys := sdk.NewKVStoreKeys(
		authtypes.StoreKey, banktypes.StoreKey, stakingtypes.StoreKey,
		minttypes.StoreKey, distrtypes.StoreKey, slashingtypes.StoreKey,
		govtypes.StoreKey, paramstypes.StoreKey, ibchost.StoreKey, upgradetypes.StoreKey,
		evidencetypes.StoreKey, ibctransfertypes.StoreKey, capabilitytypes.StoreKey,
	)

	//register the plugin(s) with the BaseApp
	if err := pluginLoader.Inject(bApp, appCodec, keys); err != nil {
		//handle error
    }

	//start the plugin services, optionally use wg to synchronize shutdown using io.Closer
	wg := new(sync.WaitGroup)
	if err := pluginLoader.Start(wg); err != nil {
		//handler error
    }

	...

	return app
}
```


#### Configuration

插件系统将在应用程序的 app.toml 文件中进行配置。 

```toml
[plugins]
    on = false # turn the plugin system, as a whole, on or off
    disabled = ["list", "of", "plugin", "names", "to", "disable"]
    dir = "the directory to load non-preloaded plugins from; defaults to cosmos-sdk/plugin/plugins"
```

配置插件系统将有三个参数:`plugins.on`、`plugins.disabled` 和`plugins.dir`。
`plugins.on` 是一个布尔值，用于打开或关闭整个插件系统，`plugins.dir` 将系统定向到一个目录
从中加载插件，`plugins.disabled` 是我们要禁用的插件的名称列表(用于禁用预加载的插件)。

给定插件的配置最终是特定于插件的，但我们将在这里介绍一些标准:

插件 TOML 配置应为每种插件(例如`plugins.streaming`)拆分为单独的子表。
在这些子表中，此类特定插件的参数包含在另一个子表中(例如`plugins.streaming.file`)。
通常，但不是必需的，可以使用一组存储密钥配置流媒体服务插件
(例如`plugins.streaming.file.keys`)用于它侦听的存储和模式(例如`plugins.streaming.file.mode`)
这表示服务是否以即发即忘的能力(`faf`)运行，或者 BaseApp 应该要求正面
服务对消息接收的确认(`ack`)。 
e.g.

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

ADR-038 引入了用于从 KVStores 中流式传输状态更改的接口和类型，关联此
数据及其相关的 ABCI 请求和响应，并注册一个服务以使用这些数据并将其以最终格式流式传输到某个目的地。
不是在此 ADR 中规定最终数据格式，而是由特定的插件实现来定义和记录此格式。
我们采用这种方法是因为最终格式的灵活性对于支持广泛的流媒体服务插件是必要的。 例如，
将数据写入一组文件的流服务的数据格式与写入 Kafka 主题的数据格式不同。 

## Consequences

These changes will provide a means of subscribing to KVStore state changes in real time.

### Backwards Compatibility

- 此 ADR 更改了 `MultiStore`、`CacheWrap` 和 `CacheWrapper` 接口，支持这些接口的先前版本的实现将不支持新的接口 

### Positive

- 能够实时监听 KVStore 状态变化并将这些事件暴露给外部消费者 

### Negative

- 更改了`MultiStore`、`CacheWrap` 和`CacheWrapper` 接口 

### Neutral

- 为配置和运行 Cosmos 应用程序引入了额外但可选的复杂性
- 如果应用程序开发人员选择使用这些功能来公开数据，他们需要了解该数据公开的后果/风险，因为它与他们的应用程序的细节有关 
