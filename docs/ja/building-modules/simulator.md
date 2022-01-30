# モジュールシミュレーション

## 前提条件

*[Cosmos Blockchain Simulator](./../using-the-sdk/Simulation.md)

## 概要

このドキュメントでは、各モジュールのシミュレーション機能を定義する方法について詳しく説明します
アプリケーション[SimulationManager]との統合。

*[シミュレーションパッケージ](#simulation-package)
     *[ストレージデコーダー](#store-decoders)
     *[ランダムオリジン](#randomized-genesis)
     *[ランダム化されたパラメーター](#randomized-parameters)
     *[ランダム加重操作](#random-weighted-operations)
     *[ランダムプロポーザルコンテンツ](#random-proposal-contents)
※【モジュールシミュレーション機能の登録】(#registering-simulation-functions)
*[アプリシミュレーターマネージャー](#app-simulator-manager)
*[シミュレーションテスト](#simulation-tests)

## シミュレーションパッケージ

Cosmos SDKシミュレーターを実装するすべてのモジュールには、 `x/<module>/simulation`が必要です。
ファジングに必要な主な機能を含むパッケージ:ストレージ
デコーダー、ランダムな作成状態とパラメーター、重み付けされた操作と提案
コンテンツ。

### ストレージデコーダー 

`AppImportExport`はストレージデコーダーを登録する必要があります。これにより、
デコードされるストレージからのキーと値のペア(_i.e_ unmarshalled)
対応するタイプに。特に、キーを特定のタイプに一致させます
次に、 `KVPair`の値を指定されたタイプにグループ化解除します。

配布モジュール[ここ](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/distribution/simulation/decoder.go)の例を使用して、ストレージデコーダーを実装できます。

### ランダムオリジン

シミュレーターは、さまざまなシナリオと作成パラメーター値をテストします
特定のモジュールのエッジケースを完全にテストするため。各モジュールの `simulator`パッケージは、指定されたシードから初期のランダムな` GenesisState`を生成するために、 `RandomizedGenState`関数を公開する必要があります。

モジュール作成パラメータがランダムに生成されたら(またはキーを使用して
`params`ファイルで定義された値)、それらはJSON形式にグループ化されて追加されます
アプリケーションジェネシスJSONに移動して、シミュレーションで使用します。

ランダムジェネシスを作成する方法の例を[ここ](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/staking/simulation/genesis.go)で表示できます。

### パラメータのランダムな変更

シミュレーターは、パラメーターの変更をランダムにテストできます。各モジュールのシミュレーターパッケージには、シミュレーションライフサイクル全体でモジュールのパラメーター変更をシミュレートする[RandomizedParams]関数が含まれている必要があります。

パラメータの変更を完全にテストするために必要な例は、[こちら](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/staking/simulation/params.go)で確認できます。

### ランダム重み付け操作

操作は、CosmosSDKシミュレーションの重要な部分の1つです。彼らはお得な情報です
( `Msg`)ランダムなフィールド値でシミュレートします。アクションの送信者
また、ランダムに割り当てられます。

完全な[トランザクションサイクル](../core/transaction.md)を使用して、シミュレーション操作をシミュレートします
[BaseApp]の[ABCI]アプリケーションを公開します。

以下に、重みを設定する方法を示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/staking/simulation/operations.go#L18-L68

ご覧のとおり、この場合、重みは事前定義されています。さまざまな重みでこの動作をオーバーライドするオプションがあります。 1つのオプションは、 `* rand.Rand`を使用して操作のランダムな重みを定義するか、独自の事前定義された重みを注入することです。

上記のパッケージ[simappparams]を上書きする方法は次のとおりです。

+++ https://github.com/cosmos/gaia/blob/master/sims.mk#L9-L22

最後のテストでは、runsim <！-#TODO:作成時にrunsim readmeへのリンクを追加します->を使用します。これは、goテストインスタンスを並列化し、Githubに情報を提供し、統合を緩めてチームに提供します。シミュレーションがどのように機能するかを理解しています。

### ランダムな提案コンテンツ

Cosmos SDKシミュレーターは、ランダムなガバナンス提案もサポートします。各
モジュールは、公開および登録されたガバナンス提案の[コンテンツ]を定義する必要があります
それらはパラメータに使用されます。

##シミュレーション機能の登録

必要なすべての関数が定義されたので、それらを `module.go`のモジュールパターンに統合する必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/distribution/module.go

## アプリケーションシミュレータマネージャー

次の手順は、アプリケーションレベルで[SimulatorManager]を設定するためのものです。この
次のステップは、テストファイルをシミュレートすることです。  

```go
type CustomApp struct {
  ...
  sm *module.SimulationManager
}
```

次に、アプリケーションがインスタンス化されたら、 `SimulationManager`を作成します
インスタンスは `ModuleManager`を作成した方法と同じですが、今回は渡すだけです
`AppModuleSimulation`のシミュレーション関数を実装するモジュール
インターフェースは上記の通りです。  

```go
func NewCustomApp(...) {
//create the simulation manager and define the order of the modules for deterministic simulations
  app.sm = module.NewSimulationManager(
    auth.NewAppModule(app.accountKeeper),
    bank.NewAppModule(app.bankKeeper, app.accountKeeper),
    supply.NewAppModule(app.supplyKeeper, app.accountKeeper),
    ov.NewAppModule(app.govKeeper, app.accountKeeper, app.supplyKeeper),
    mint.NewAppModule(app.mintKeeper),
    distr.NewAppModule(app.distrKeeper, app.accountKeeper, app.supplyKeeper, app.stakingKeeper),
    staking.NewAppModule(app.stakingKeeper, app.accountKeeper, app.supplyKeeper),
    slashing.NewAppModule(app.slashingKeeper, app.accountKeeper, app.stakingKeeper),
  )

//register the store decoders for simulation tests
  app.sm.RegisterStoreDecoders()
  ...
}
```
