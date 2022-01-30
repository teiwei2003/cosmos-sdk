# インプレースストレージ移行

::: 警告
ライブチェーンで移行を実行する前に、すべてのインプレースストレージ移行ドキュメントを読んで理解してください。
:::

カスタムのインプレースストレージ移行ロジックを使用して、アプリケーションモジュールをスムーズにアップグレードします。 {まとめ}

Cosmos SDKは、2つの方法を使用してアップグレードを実行します。

- `export` CLIコマンドを使用してアプリケーションの状態全体をJSONファイルにエクスポートし、変更を加えてから、変更したJSONファイルをジェネシスファイルとして使用して、新しいバイナリファイルを開始します。 [チェーンアップグレードガイドv0.42](https://docs.cosmos.network/v0.42/migrations/chain-upgrade-guide-040.html)を参照してください。

- v0.44以降のバージョンでは、インプレースアップグレードを実行して、状態が大きいチェーンのアップグレード時間を大幅に短縮できます。 [モジュールアップグレードガイド](../building-modules/upgrade.md)を使用して、インプレースアップグレードを利用するようにアプリケーションモジュールを設定します。

このドキュメントでは、インプレースストレージ移行のアップグレード方法を使用する手順について説明します。

## トラッキングモジュールバージョン

モジュール開発者は、各モジュールにコンセンサスバージョンを割り当てます。 コンセンサスバージョンは、モジュールの画期的な変更バージョンとして機能します。 Cosmos SDKは、x/upgradeの `VersionMap`ストア内のすべてのモジュールのコンセンサスバージョンを追跡します。 アップグレードプロセス中に、状態に保存されている古い `VersionMap`と新しい` VersionMap`の差がCosmosSDKによって計算されます。 識別された違いごとに、モジュール固有の移行が実行され、アップグレードされた各モジュールの対応するコンセンサスバージョンがインクリメントされます。

## 作成状態

新しいチェーンを開始するときは、各モジュールのコンセンサスバージョンをアプリケーションの作成中の状態に保存する必要があります。 コンセンサスバージョンを保存するには、[app.go]の[InitChainer]メソッドに次の行を追加します。

```diff
func (app *MyApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
  ...
+ app.UpgradeKeeper.SetModuleVersionMap(ctx, app.mm.GetVersionMap())
  ...
}
```

Cosmos SDKはこの情報を使用して、新しいバージョンのモジュールがアプリケーションに導入されたことを検出します。

### コンセンサスバージョン

コンセンサスバージョンは、モジュールの画期的な変更バージョンとして、各アプリケーションモジュールのモジュール開発者によって定義されます。 コンセンサスバージョンは、どのモジュールをアップグレードする必要があるかをCosmosSDKに通知します。 たとえば、バンクモジュールがバージョン2で、アップグレードによってバンクモジュール3が導入された場合、Cosmos SDKはバンクモジュールをアップグレードし、[バージョン2から3]の移行スクリプトを実行します。

### バージョンマッピング

バージョンマッピングは、モジュール名からコンセンサスバージョンへのマッピングです。 マッピングは、インプレース移行中に使用するためにx/upgradeの状態に保持されます。 移行が完了すると、更新されたバージョンマップが保持され、状態が維持されます。

## アップグレードハンドラ

アップグレードでは、 `UpgradeHandler`を使用して移行を容易にします。 アプリケーション開発者が実装する `UpgradeHandler`関数は、次の関数シグネチャに準拠している必要があります。 これらの関数は、x/upgradeの状態から `VersionMap`を取得し、アップグレード後に新しい` VersionMap`を返してx/upgradeに保存します。 2つの `VersionMap`の違いにより、アップグレードする必要のあるモジュールが決まります。

```go
type UpgradeHandler func(ctx sdk.Context, plan Plan, fromVM VersionMap) (VersionMap, error)
```

これらの機能では、提供される[計画]に含まれるアップグレードロジックを実行する必要があります。 すべてのアップグレードハンドラー関数は、次のコード行で終了する必要があります。

```go
  return app.mm.RunMigrations(ctx, cfg, fromVM)
```

## 移行を実行する

`app.mm.RunMigrations(ctx、cfg、vm)`を使用して、 `UpgradeHandler`内で移行を実行します。 `UpgradeHandler`関数は、アップグレードプロセス中に発生する関数を記述します。 `RunMigration`関数は` VersionMap`パラメータをループし、新しいバイナリアプリケーションモジュールバージョンより前のすべてのバージョンに対して移行スクリプトを実行します。 移行が完了すると、新しい[VersionMap]が返され、アップグレードされたモジュールのバージョンが状態に保存されます。

```go
cfg := module.NewConfigurator(...)
app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap) (module.VersionMap, error) {

  //...
  //do upgrade logic
  //...

  //RunMigrations returns the VersionMap
  //with the updated module ConsensusVersions
    return app.mm.RunMigrations(ctx, vm)
})
```

モジュールの移行スクリプトの構成の詳細については、[モジュールアップグレードガイド](../building-modules/upgrade.md)を参照してください。

## アップグレードプロセス中に新しいモジュールを追加する

アップグレード中に、アプリケーションにまったく新しいモジュールを導入できます。 新しいモジュールは、 `x/upgrade`の` VersionMap`ストアに登録されていないために認識されます。 この場合、 `RunMigrations`は対応するモジュールから` InitGenesis`関数を呼び出して初期状態を設定します。

### 新しいモジュールのStoreUpgradesを追加します

インプレースストレージ移行を実行する準備ができているすべてのチェーンストアは、ストレージアップグレードを新しいモジュールに手動で追加してから、これらのアップグレードを適用するようにストレージローダーを構成する必要があります。 これにより、移行が開始される前に、新しいモジュールのストレージがマルチストレージに追加されます。

```go
upgradeInfo, err := app.UpgradeKeeper.ReadUpgradeInfoFromDisk()
if err != nil {
	panic(err)
}

if upgradeInfo.Name == "my-plan" && !app.UpgradeKeeper.IsSkipHeight(upgradeInfo.Height) {
	storeUpgrades := storetypes.StoreUpgrades{
		//add store upgrades for new modules
		//Example:
		//  Added: []string{"foo", "bar"},
		//...
	}

	//configure store loader that checks if version == upgradeHeight and applies store upgrades
	app.SetStoreLoader(upgradetypes.UpgradeStoreLoader(upgradeInfo.Height, &storeUpgrades))
}
```

## ジェネシス関数をオーバーライドする

Cosmos SDKは、アプリケーション開発者がアプリケーションにインポートできるモジュールを提供します。 これらのモジュールは通常、 `InitGenesis`関数を定義しています。

インポートされたモジュールに対して独自の `InitGenesis`関数を作成できます。 これを行うには、アップグレードハンドラーでカスタムジェネシス関数を手動でトリガーしてください。

::: 警告
`UpgradeHandler`関数に渡されるバージョンマップでコンセンサスバージョンを手動で設定する必要があります。 これがないと、UpgradeHandlerでカスタム関数をトリガーしても、SDKはモジュールの既存のInitGenesisコードを実行します。
:::

```go
import foo "github.com/my/module/foo"

app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {

  //Register the consensus version in the version map
  //to avoid the SDK from triggering the default
  //InitGenesis function.
    vm["foo"] = foo.AppModule{}.ConsensusVersion()

  //Run custom InitGenesis for foo
    app.mm["foo"].InitGenesis(ctx, app.appCodec, myCustomGenesisState)

    return app.mm.RunMigrations(ctx, cfg, vm)
})
```

カスタム作成関数がなく、モジュールのデフォルトの作成関数をスキップしたい場合は、例に示すように、[UpgradeHandler]でバージョンマッピングを使用してモジュールを登録するだけです。

```go
import foo "github.com/my/module/foo"

app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {

  //Set foo's version to the latest ConsensusVersion in the VersionMap.
  //This will skip running InitGenesis on Foo
    vm["foo"] = foo.AppModule{}.ConsensusVersion()

    return app.mm.RunMigrations(ctx, cfg, vm)
})
```

## フルノードをアップグレードされたブロックチェーンに同期します

Cosmovisorでアップグレードされた既存のブロックチェーンにノード全体を同期できます

正常に同期するには、作成時にブロックチェーンが開始した最初のバイナリファイルから開始する必要があります。 すべてのソフトウェアアップグレードプランにバイナリ命令が含まれている場合は、自動ダウンロードオプションを使用してCosmovisorを実行し、ダウンロードを自動的に処理して、各順次アップグレードに関連付けられたバイナリファイルに切り替えることができます。 それ以外の場合は、すべてのバイナリをCosmovisorに手動で提供する必要があります。

Cosmovisorの詳細については、[Cosmovisorクイックスタート](../run-node/cosmovisor.md)を参照してください。