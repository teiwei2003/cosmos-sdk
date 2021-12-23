# 統合

IBCをアプリケーションに統合し、他のチェーンにパケットを送信する方法を学びます。 {まとめ}

このドキュメントでは、統合と構成の概要を説明します[IBC
モジュール](https://github.com/cosmos/ibc-go/tree/main/modules/core)をCosmosSDKアプリケーションに追加します
代替可能なトークン転送を他のチェーンに送信します。

## 統合IBCモジュール

IBCモジュールをCosmosSDKに基づくアプリケーションに統合するのは非常に簡単です。 一般的な変更は、次の手順として要約できます。

- 必要なモジュールを `module.BasicManager`に追加します
- タイプ `App`の新しいモジュール用に追加の` Keeper`フィールドを定義します
- モジュールの `StoreKeys`を追加し、それらの` Keepers`を初期化します
- `ibc`および `evidence`モジュールに対応するルーターとルートを設定します
- モジュールをモジュール `Manager`に追加します
- モジュールを `Begin/EndBlockers`と` InitGenesis`に追加します
- モジュール「SimulationManager」を更新してシミュレーションを有効にします

### モジュール `BasicManager`および` ModuleAccount`権限

最初のステップは、次のモジュールを `BasicManager`に追加することです。`x/capability`、` x/ibc`、
`x/evidence`と` x/ibc-transfer`。 その後、 `Minter`と` Burner`の権限を付与する必要があります
`ibc-transfer``ModuleAccount`は、リレートークンを作成および破棄するために使用されます。 

```go
//app.go
var (

  ModuleBasics = module.NewBasicManager(
   //...
    capability.AppModuleBasic{},
    ibc.AppModuleBasic{},
    evidence.AppModuleBasic{},
    transfer.AppModuleBasic{},//i.e ibc-transfer module
  )

 //module account permissions
  maccPerms = map[string][]string{
   //other module accounts permissions
   //...
    ibctransfertypes.ModuleName:    {authtypes.Minter, authtypes.Burner},
)
```

### アプリケーションエリア

次に、次のように `Keepers`を登録する必要があります。

```go
//app.go
type App struct {
 //baseapp, keys and subspaces definitions

 //other keepers
 //...
  IBCKeeper        *ibckeeper.Keeper//IBC Keeper must be a pointer in the app, so we can SetRouter on it correctly
  EvidenceKeeper   evidencekeeper.Keeper//required to set up the client misbehaviour route
  TransferKeeper   ibctransferkeeper.Keeper//for cross-chain fungible token transfers

 //make scoped keepers public for test purposes
  ScopedIBCKeeper      capabilitykeeper.ScopedKeeper
  ScopedTransferKeeper capabilitykeeper.ScopedKeeper

 ///...
 ///module and simulation manager definitions
}
```

### Configure the `Keepers`

初期化中、IBC `Keepers`(` x/ibc`の場合、および
`x/ibc-transfer`モジュール)、能力モジュールを介して特定の能力を付与する必要があります
各IBCのオブジェクト機能のアクセス許可を確認できるようにするための `ScopedKeepers`
チャネル。 

```go
func NewApp(...args) *App {
 //define codecs and baseapp

 //add capability keeper and ScopeToModule for ibc module
  app.CapabilityKeeper = capabilitykeeper.NewKeeper(appCodec, keys[capabilitytypes.StoreKey], memKeys[capabilitytypes.MemStoreKey])

 //grant capabilities for the ibc and ibc-transfer modules
  scopedIBCKeeper := app.CapabilityKeeper.ScopeToModule(ibchost.ModuleName)
  scopedTransferKeeper := app.CapabilityKeeper.ScopeToModule(ibctransfertypes.ModuleName)

 //... other modules keepers

 //Create IBC Keeper
  app.IBCKeeper = ibckeeper.NewKeeper(
  appCodec, keys[ibchost.StoreKey], app.StakingKeeper, scopedIBCKeeper,
  )

 //Create Transfer Keepers
  app.TransferKeeper = ibctransferkeeper.NewKeeper(
    appCodec, keys[ibctransfertypes.StoreKey],
    app.IBCKeeper.ChannelKeeper, &app.IBCKeeper.PortKeeper,
    app.AccountKeeper, app.BankKeeper, scopedTransferKeeper,
  )
  transferModule := transfer.NewAppModule(app.TransferKeeper)

 //Create evidence Keeper for to register the IBC light client misbehaviour evidence route
  evidenceKeeper := evidencekeeper.NewKeeper(
    appCodec, keys[evidencetypes.StoreKey], &app.StakingKeeper, app.SlashingKeeper,
  )

 //.. continues
}
```

### `ルーター`を登録する

IBCは、パケットをにルーティングできるように、どのモジュールがどのポートにバインドされているかを知る必要があります。
適切なモジュールを作成し、適切なコールバックを呼び出します。ポートのモジュール名へのマッピングは、次のように決定されます。
IBCのポート「Keeper」。ただし、関連するコールバックへのモジュール名のマッピングは完了しています
港で
[`ルーター`](https://github.com/cosmos/ibc-go/blob/main/modules/core/05-port/types/router.go)
IBCモジュール。

モジュールルーティングを追加すると、IBCハンドラーが
チャネルハンドシェイクまたはデータパケット。

必要な2番目の「ルーター」は証拠モジュールルーターです。このルーターは平凡なものを処理します
証拠の提出とビジネスロジックは、登録された各証拠処理プログラムにルーティングされます。このような状況下で
IBC、提出する必要がある[ライトクライアントの証拠
不正行為](https://github.com/cosmos/ics/tree/master/spec/ics-002-client-semantics#misbehaviour)
クライアントをフリーズし、それ以上のパケットの送受信を防ぐため。

現在、 `Router`は静的であるため、アプリケーションの初期化時に初期化して正しく設定する必要があります。
`Router`が設定されると、新しいルートを追加することはできません。 

```go
//app.go
func NewApp(...args) *App {
 //.. continuation from above

 //Create static IBC router, add ibc-tranfer module route, then set and seal it
  ibcRouter := port.NewRouter()
  ibcRouter.AddRoute(ibctransfertypes.ModuleName, transferModule)
 //Setting Router will finalize all routes by sealing router
 //No more routes can be added
  app.IBCKeeper.SetRouter(ibcRouter)

 //create static Evidence routers

  evidenceRouter := evidencetypes.NewRouter().
   //add IBC ClientMisbehaviour evidence handler
    AddRoute(ibcclient.RouterKey, ibcclient.HandlerClientMisbehaviour(app.IBCKeeper.ClientKeeper))

 //Setting Router will finalize all routes by sealing router
 //No more routes can be added
  evidenceKeeper.SetRouter(evidenceRouter)

 //set the evidence keeper from the section above
  app.EvidenceKeeper = *evidenceKeeper

 //.. continues
```

### Module Managers

IBCを使用するには、アプリケーションが[simulations](./../building-modules/simulator.md)をサポートしている場合に備えて、モジュール `Manager`と` SimulationManager`に新しいモジュールを追加する必要があります。 

```go
//app.go
func NewApp(...args) *App {
 //.. continuation from above

  app.mm = module.NewManager(
   //other modules
   //...
    capability.NewAppModule(appCodec, *app.CapabilityKeeper),
    evidence.NewAppModule(app.EvidenceKeeper),
    ibc.NewAppModule(app.IBCKeeper),
    transferModule,
  )

 //...

  app.sm = module.NewSimulationManager(
   //other modules
   //...
    capability.NewAppModule(appCodec, *app.CapabilityKeeper),
    evidence.NewAppModule(app.EvidenceKeeper),
    ibc.NewAppModule(app.IBCKeeper),
    transferModule,
  )

 //.. continues
```

### ABCI注文を適用する

IBCの新機能は、ステーキングモジュールに格納されている「HistoricalEntries」の概念です。
各エントリには、このチェーンの「Header」と「ValidatorSet」の履歴情報が含まれています。これらは保存されています。
「BeginBlock」呼び出し中の各高さ。 反映するために履歴情報が必要
この期間中のライトクライアント「ConsensusState」を検証するための過去の任意の高さでの履歴情報
接続ハンドシェイク。

IBCモジュールには
[`BeginBlock`](https://github.com/cosmos/ibc-go/blob/main/modules/core/02-client/abci.go)ロジックは
良い。 [localhostのみを使用できるため、これはオプションです。
クライアント](https://github.com/cosmos/ibc/tree/master/spec/client/ics-009-loopback-client)2つ接続します
同じチェーンからの異なるモジュール。

::: ヒント
アプリケーションが使用する場合
ローカルホスト(_aka_ループバック)クライアント。
::: 

```go
//app.go
func NewApp(...args) *App {
 //.. continuation from above

 //add evidence, staking and ibc modules to BeginBlockers
  app.mm.SetOrderBeginBlockers(
   //other modules ...
    evidencetypes.ModuleName, stakingtypes.ModuleName, ibchost.ModuleName,
  )

 //...

 //NOTE: Capability module must occur first so that it can initialize any capabilities
 //so that other modules that want to create or claim capabilities afterwards in InitChain
 //can do so safely.
  app.mm.SetOrderInitGenesis(
    capabilitytypes.ModuleName,
   //other modules ...
    ibchost.ModuleName, evidencetypes.ModuleName, ibctransfertypes.ModuleName,
  )

 //.. continues
```

::: 警告
**重要**：汎用モジュール**は `SetOrderInitGenesis`で最初に宣言する必要があります
:::

それでおしまい！ これで、IBCモジュールが配線され、代替可能なトークンを送信できるようになりました。
異なるチェーン。 変更の全体像を知りたい場合は、CosmosSDKを調べてください。
[`SimApp`](https://github.com/cosmos/ibc-go/blob/main/testing/simapp/app.go)。

## 次へ{非表示}

アプリケーション用の[カスタムIBCモジュール](./custom.md){hide}を作成する方法を学ぶ