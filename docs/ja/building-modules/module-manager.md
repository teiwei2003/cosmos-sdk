# モジュールマネージャー

Cosmos SDKモジュールは、アプリケーションの[module manager](#module-manager)で管理できるように、[`AppModule`インターフェース](#application-module-interfaces)を実装する必要があります。モジュールマネージャーは、[`message`および` query`ルーティング](../core/baseapp.md#routing)で重要な役割を果たし、アプリケーション開発者が[`BeginBlocker`や` BeginBlocker`などのさまざまな関数の実行順序を設定できるようにします。 `EndBlocker`](../basics/app-anatomy.md#begingblocker-and-endblocker)。 {まとめ}

## 読むための前提条件

-[Cosmos SDKモジュールの紹介](。/intro.md){前提条件}

## アプリケーションモジュールインターフェイス

アプリケーションモジュールインターフェイスは、モジュールの組み合わせを容易にして機能的なCosmosSDKアプリケーションを形成するために存在します。 3つの主要なアプリケーションモジュールインターフェイスがあります。

-[`AppModuleBasic`](#appmodulebasic)は、独立したモジュール関数に使用されます。
-[`AppModule`](#appmodule)相互依存モジュール関数の場合)作成に関連する関数を除く)。
-[`AppModuleGenesis`](#appmodulegenesis)は、作成に関連する相互依存モジュール関数に使用されます。

`AppModuleBasic`インターフェースは、モジュールの独立したメソッド、つまり、アプリケーション内の他のモジュールに依存しないメソッドを定義するために存在します。これにより、基本的なアプリケーション構造をアプリケーション定義の早い段階で、通常は[メインアプリケーションファイル](../basics/app-anatomy.md#core-application-file)の `init()`関数で構築できます。

`AppModule`インターフェースは、相互に依存するモジュールメソッドを定義するために使用されます。多くのモジュールは、通常[`keeper`s](./keeper.md)を介して他のモジュールと対話する必要があります。これは、インターフェースが必要であることを意味します。ここで、モジュールは、参照する必要のある` keeper`sおよびその他のメソッドを一覧表示します。別のモジュールオブジェクト。 `AppModule`インターフェースを使用すると、モジュールマネージャーは` BeginBlock`や `EndBlock`などのモジュールメソッド間の実行順序を設定することもできます。これは、アプリケーションのコンテキストでモジュール間の実行順序が重要な場合に重要です。

最後に、ジェネシス関数「AppModuleGenesis」のインターフェースは、完全なモジュール関数「AppModule」から分離されているため、モジュールは
ジェネシスの場合のみ、多くのプレースホルダー関数を定義せずに `Module`モードを使用できます。 

### `AppModuleBasic`

`AppModuleBasic`インターフェースは、モジュールが実装する必要のある独立したメソッドを定義します。

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L49-L63

これらのメソッドを見てみましょう。

-`Name() `:モジュールの名前を` string`の形式で返します。
-`RegisterLegacyAminoCodec(* codec.LegacyAmino) `:モジュールの` amino`コーデックを登録します。これは `[] byte`との間で構造をマーシャリングおよびアンマーシャリングするために使用され、モジュールの` KVStore`に格納できるようにします。
-`RegisterInterfaces(codectypes.InterfaceRegistry) `:モジュールのインターフェースタイプとその特定の実装を` proto.Message`として登録します。
-`DefaultGenesis(codec.JSONCodec) `:モジュールのデフォルトの[` GenesisState`](./genesis.md#genesisstate)を、 `json.RawMessage`としてグループ化して返します。デフォルトの「GenesisState」はモジュール開発者が定義する必要があり、主にテストに使用されます。
-`ValidateGenesis(codec.JSONCodec、client.TxEncodingConfig、json.RawMessage) `:モジュール定義の検証に使用される` GenesisState`は、 `json.RawMessage`の形式で指定されます。通常、モジュール開発者が定義したカスタム[`ValidateGenesis`](./genesis.md#validategenesis)関数を実行する前に、` json`をアンマーシャリングします。
-`RegisterRESTRoutes(client.Context、* mux.Router) `:モジュールのRESTルートを登録します。これらのルートは、RESTリクエストをモジュールにマッピングして処理するために使用されます。詳細については、[gRPCおよびREST](../core/grpc_rest.md)を参照してください。
-`RegisterGRPCGatewayRoutes(client.Context、* runtime.ServeMux) `:モジュールのgRPCルートを登録します。
-`GetTxCmd() `:モジュールのルートを返します[` Tx`コマンド](./module-interfaces.md#tx)。エンドユーザーは、このrootコマンドのサブコマンドを使用して、モジュールで定義された[`message`s](./messages-and-queries.md#queries)を含む新しいトランザクションを生成します。
-`GetQueryCmd() `:モジュールのルートを返します[` query`コマンド](./module-interfaces.md#query)。エンドユーザーは、このrootコマンドのサブコマンドを使用して、モジュールによって定義された状態サブセットの新しいクエリを生成します。

アプリケーションのすべての `AppModuleBasic`は、[` BasicManager`](#basicmanager)によって管理されます。

### `AppModuleGenesis`

`AppModuleGenesis`インターフェースは、` AppModuleBasic`インターフェースに2つの追加メソッドを単純に埋め込んだものです。

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L152-L158

追加された2つの方法を見てみましょう。

-`InitGenesis(sdk.Context、codec.JSONCodec、json.RawMessage) `:モジュールによって管理される状態サブセットを初期化します。作成時(つまり、チェーンが最初に開始されたとき)に呼び出されます。
-`ExportGenesis(sdk.Context、codec.JSONCodec) `:新しいジェネシスファイルで使用するために、モジュールによって管理される最新の状態サブセットをエクスポートします。既存のチェーンの状態から新しいチェーンを開始する場合、各モジュールは `ExportGenesis`を呼び出します。

独自のマネージャーを持たず、[`AppModule`](#appmodule)とは別に存在します。作成機能を実装するモジュールにのみ使用されるため、` AppModule`のすべてのメソッドを実装せずに管理できます。モジュールは作成期間中に使用されるだけでなく、 `InitGenesis(sdk.Context、codec.JSONCodec、json.RawMessage)`と `ExportGenesis(sdk.Context、codec.JSONCodec)`は通常、実装する具体的なタイプのメソッドとして定義されます。 `AppModule`インターフェース。

### `AppModule`

`AppModule`インターフェースは、モジュールが実装する必要のある相互依存メソッドを定義します。

+++ https://github.com/cosmos/cosmos-sdk/blob/b4cce159bcc6a32ac78245c6866dd87c73f3720d/types/module/module.go#L160-L182

`AppModule`は[モジュールマネージャー](#manager)によって管理されます。このインターフェースには `AppModuleGenesis`インターフェースが組み込まれているため、マネージャーはモジュールのすべての独立したメソッドと作成に依存するメソッドにアクセスできます。つまり、 `AppModule`インターフェイスを実装する具象型は、` AppModuleGenesis`)のすべてのメソッドを実装して `AppModuleBasic`)に展開するか、パラメータとして具象型を含める必要があります。

`AppModule`のメソッドを見てみましょう。

-`RegisterInvariants(sdk.InvariantRegistry) `:登録されたモジュールの[` invariants`](./invariants.md)。不変条件が予測値から外れると、[`InvariantRegistry`](./invariants.md#registry)が適切なロジックをトリガーします)ほとんどの場合、チェーンは停止します)。
-`Route() `:[` BaseApp`](../core/baseapp.md#messages)によってルーティングされた[`message`s](./messages-and-queries.md#messages)のルートを返します)モジュール。 md#メッセージルーティング)。
-`QuerierRoute() `)は推奨されません):[` BaseApp`]から[`queries`](./messages-and-queries.md#queries)に使用されるモジュールのクエリルートの名前を返しますモジュールRouting)../core/baseapp.md#query-routing)。
-`LegacyQuerierHandler(* codec.LegacyAmino) `)は推奨されません):` query`を処理するために、クエリ `path`を指定して[` querier`](./query-services.md#legacy-queriers)を返します。
-`RegisterServices(Configurator) `:モジュールがサービスを登録できるようにします。
-`BeginBlock(sdk.Context、abci.RequestBeginBlock) `:このメソッドは、モジュール開発者に、各ブロックの開始時に自動的にトリガーされるロジックを実装するオプションを提供します。このモジュールの各ブロックの先頭でロジックをトリガーする必要がない場合、実装は空です。
-`EndBlock(sdk.Context、abci.RequestEndBlock) `:このメソッドは、モジュール開発者に、各ブロックの最後で自動的にトリガーされるロジックを実装するオプションを提供します。これは、モジュールがバリデーターセットへの変更を基礎となるコンセンサスエンジンに通知できる場所でもあります(たとえば、「ステーキング」モジュール)。このモジュールの各ブロックの最後でロジックをトリガーする必要がない場合、実装は空です。 

### アプリケーションモジュールインターフェイスを実装する

一般に、さまざまなアプリケーションモジュールインターフェイスは、モジュールのフォルダにある「module.go」という名前のファイル(たとえば、「。/x/module/module.go」)に実装されます。

ほとんどすべてのモジュールは、 `AppModuleBasic`および` AppModule`インターフェースを実装する必要があります。 モジュールが作成にのみ使用される場合は、 `AppModule`の代わりに` AppModuleGenesis`を実装します。 特定のタイプのインターフェースを実装するために、インターフェースのさまざまなメソッドを実装するために必要なパラメーターを追加できます。 たとえば、 `Route()`関数は `keeper/msg_server.go`で定義された` NewMsgServerImpl(k keeper) `関数を呼び出すことが多いため、モジュールの[` keeper`](./keeper.md)はパラメータ。 

```go
//example
type AppModule struct {
	AppModuleBasic
	keeper       Keeper
}
```

上記の例では、具体的なタイプ「AppModule」が「AppModuleGenesis」ではなく「AppModuleBasic」を参照していることがわかります。 これは、 `AppModuleGenesis`は、作成に関連する関数に焦点を当てたモジュールにのみ実装する必要があるためです。 ほとんどのモジュールでは、特定の `AppModule`タイプは` AppModuleBasic`を参照し、 `AppModuleGenesis`の2つの追加メソッドは` AppModule`タイプに直接実装されます。

パラメータが必要ない場合)(通常は `AppModuleBasic`が当てはまります)、次に示すように、空の具象型を宣言するだけです。

```go
type AppModuleBasic struct{}
```

## モジュールマネージャー

モジュールマネージャは、 `AppModuleBasic`と` AppModule`のコレクションを管理するために使用されます。

### `ベーシックマネージャー`

`BasicManager`は、アプリケーションのすべての` AppModuleBasic`を一覧表示する構造です。

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L65-L66

次のメソッドを実装します。

-`NewBasicManager(modules ... AppModuleBasic) `:コンストラクター。アプリケーションの「AppModuleBasic」リストを取得し、新しい「BasicManager」を構築します。この関数は通常、[`app.go`](../basics/app-anatomy.md#core-application-file)の` init() `関数で呼び出され、アプリケーションモジュールの独立した要素をすばやく初期化します)[ここをクリック])https://github.com/cosmos/gaia/blob/master/app/app.go#L59-L74)をクリックして、例を表示します)。
-`RegisterLegacyAminoCodec(cdc * codec.LegacyAmino) `:各アプリケーションの` AppModuleBasic`の[`codec.LegacyAmino`s](../core/encoding.md#amino)を登録します。この関数は通常、[アプリケーション構築](../basics/app-anatomy.md#constructor)の初期段階で呼び出されます。
-`RegisterInterfaces(registry codectypes.InterfaceRegistry) `:各アプリケーション` AppModuleBasic`のインターフェースタイプと実装を登録します。
-`DefaultGenesis(cdc codec.JSONCodec) `:各モジュールの[` DefaultGenesis(cdc codec.JSONCodec) `](./genesis.md#defaultgenesis)関数を呼び出して、アプリケーション情報でモジュールのデフォルトのジェネシスを指定します。これは、アプリケーションのデフォルトのジェネシスファイルを作成するために使用されます。
-`ValidateGenesis(cdc codec.JSONCodec、txEncCfg client.TxEncodingConfig、genesis map[string] json.RawMessage) `:[` ValidateGenesis(codec.JSONCodec、client.TxEncodingConfig、json.RawMessage) `を呼び出してジェネシス情報モジュールを検証する](./genesis.md#validategenesis)各モジュールの機能。
-`RegisterRESTRoutes(ctx client.Context、rtr * mux.Router) `:各モジュールの[` RegisterRESTRoutes`](./module-interfaces.md#register-routes)関数を呼び出して、モジュールのRESTルートを登録します。この関数は通常、[アプリケーションコマンドラインインターフェイス](../core/cli.md)の `main.go`関数から呼び出されます。
-`RegisterGRPCGatewayRoutes(clientCtx client.Context、rtr * runtime.ServeMux) `:モジュールのgRPCルートを登録します。
-`AddTxCommands(rootTxCmd * cobra.Command) `:モジュールのトランザクションコマンドをアプリケーションの[` rootTxCommand`](../core/cli.md#transaction-commands)に追加します。この関数は通常、[アプリケーションコマンドラインインターフェイス](../core/cli.md)の `main.go`関数から呼び出されます。
-`AddQueryCommands(rootQueryCmd * cobra.Command) `:モジュールのクエリコマンドをアプリケーションの[` rootQueryCommand`](../core/cli.md#query-commands)に追加します。この関数は通常、[アプリケーションコマンドラインインターフェイス](../core/cli.md)の `main.go`関数から呼び出されます。

### `マネージャー`

`Manager`は、アプリケーションのすべての` AppModule`を含む構造であり、これらのモジュールのいくつかの主要コンポーネントの実行順序を定義します。

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L223-L231

モジュールのコレクションに対して操作を実行する必要がある場合は常に、アプリケーション全体でモジュールマネージャーが使用されます。次のメソッドを実装します。

-`NewManager(modules ... AppModule) `:コンストラクタ。アプリケーションの「AppModule」リストを取得し、新しい「Manager」を構築します。これは通常、アプリケーションのメイン[コンストラクター](../basics/app-anatomy.md#constructor-function)から呼び出されます。
-`SetOrderInitGenesis(moduleNames ... string) `:アプリケーションを最初に起動したときの各モジュールの[` InitGenesis`](./genesis.md#initgenesis)関数の呼び出し順序を設定します。この関数は通常、アプリケーションのメイン[コンストラクター](../basics/app-anatomy.md#constructor-function)から呼び出されます。
-`SetOrderExportGenesis(moduleNames ... string) `:エクスポートの場合に各モジュールの[` ExportGenesis`](./genesis.md#exportgenesis)関数が呼び出される順序を設定します。この関数は通常、アプリケーションのメイン[コンストラクター](../basics/app-anatomy.md#constructor-function)から呼び出されます。
-`SetOrderBeginBlockers(moduleNames ... string) `:各モジュールの` BeginBlock() `関数が各ブロックの先頭で呼び出される順序を設定します。この関数は通常、アプリケーションのメイン[コンストラクター](../basics/app-anatomy.md#constructor-function)から呼び出されます。
-`SetOrderEndBlockers(moduleNames ... string) `:各モジュールの` EndBlock() `関数が各ブロックの最後で呼び出される順序を設定します。この関数は通常、アプリケーションのメイン[コンストラクター](../basics/app-anatomy.md#constructor-function)から呼び出されます。
-`RegisterInvariants(ir sdk.InvariantRegistry) `:各モジュールの[invariants](./invariants.md)を登録します。
-`RegisterRoutes(router sdk.Router、queryRouter sdk.QueryRouter、legacyQuerierCdc * codec.LegacyAmino) `:レガシー[` Msg`](./messages-and-queries.md#messages)と[`querier`](を登録します。/query-services.md#legacy-queriers)ルーティング。
-`RegisterServices(cfg Configurator) `:すべてのモジュールサービスを登録します。
-`InitGenesis(ctx sdk.Context、cdc codec.JSONCodec、genesisData map[string] json.RawMessage) `:各モジュールの[` InitGenesis`](./genesis.md#initgenesisをアプリケーションが最初に呼び出されたときに呼び出す)関数`OrderInitGenesis`で定義された順序で開始します。 `abci.ResponseInitChain`を基盤となるコンセンサスエンジンに返します。コンセンサスエンジンには、バリデーターの更新を含めることができます。
-`ExportGenesis(ctx sdk.Context、cdc codec.JSONCodec) `:各モジュールの[` ExportGenesis`](./genesis.md#exportgenesis)関数を `OrderExportGenesis`で定義された順序で呼び出します。 exportは、既存の状態からジェネシスファイルを作成します。これは、主にチェーンをハードフォークでアップグレードする必要がある場合に使用されます。
-`BeginBlock(ctx sdk.Context、req abci.RequestBeginBlock) `:各ブロックの先頭で、[` BaseApp`](../core/baseapp.md#beginblock)からこの関数を呼び出してから、各ブロックを呼び出します。モジュールの[`BeginBlock`](./beginblock-endblock.md)関数は、` OrderBeginBlockers`で定義された順序に従います。すべてのモジュールから送信された[events](../core/events.md)を集約するために、イベントマネージャーを使用して子[context](../core/context.md)を作成します。この関数は、上記のイベントを含む `abci.ResponseBeginBlock`を返します。
-`EndBlock(ctx sdk.Context、req abci.RequestEndBlock) `:各ブロックの最後で、[` BaseApp`](../core/baseapp.md#endblock)からこの関数を呼び出し、 `OrderEndBlockersに従います。各モジュールの[`EndBlock`](./beginblock-endblock.md)関数は、`で定義された順序で呼び出されます。すべてのモジュールから送信された[events](../core/events.md)を集約するために、イベントマネージャーを使用して子[context](../core/context.md)を作成します。この関数は、上記のイベントとバリデーターセットの更新(存在する場合)を含む「abci.ResponseEndBlock」を返します。

以下は、アプリケーションでの特定の統合の例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/2323f1ac0e9a69a0da6b43693061036134193464/simapp/app.go#L315-L362

## 次へ{hide}

[`message`sと` queries`](./messages-and-queries.md){hide}の詳細 