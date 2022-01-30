# v0.44へのチェーンアップグレードガイド

このドキュメントは、v0.42からv0.44へのチェーンアップグレードガイドと、[simapp]を使用したアップグレードプロセスの例を提供します。 {まとめ}

::: ヒント
v0.44にアップグレードする前に、Stargatev0.42にアップグレードする必要があります。 まだ行っていない場合は、[v0.42へのチェーンアップグレードガイド](/v0.42/migrations/chain-upgrade-guide-040.html)を参照してください。 v0.43はリリース直後に廃止され、すべてのチェーンをv0.42からv0.44に直接アップグレードする必要があることに注意してください。
:::

## 前提条件の読書

- [アップグレードモジュール](../building-modules/upgrade.html) {prereq}
- [インプレースストレージ移行](../core/upgrade.html) {prereq}
- [Cosmovisor](../run-node/cosmovisor.html) {prereq}

Cosmos SDK v0.44では、チェーンのアップグレードを処理するための新しい方法が導入されています。状態をJSONにエクスポートし、必要な変更を加えてから、変更したJSONを新しいジェネシスファイルとして使用して新しいチェーンを作成する必要がなくなりました。

Cosmos SDKのIBCモジュールは、v0.42以降の[独自のリポジトリ](https://github.com/cosmos/ibc-go)に移動されました。 IBCを使用している場合は、[IBC移行ドキュメント](https://github.com/cosmos/ibc-go/blob/main/docs/migrations/ibc-migration-043.md)も必ず渡してください。

アップグレードバイナリは、新しいチェーンを開始する代わりに、既存のデータベースを読み取り、インプレースストレージ移行を実行します。 チェーンのアップグレードを処理するこの新しい方法は、[Cosmovisor](../run-node/cosmovisor.html)で使用して、アップグレードプロセスをシームレスにすることができます。

## インプレースストレージ移行

[In-Place Store Migrations](../core/upgrade.html)を使用して、チェーンをv0.42からv0.44にアップグレードすることをお勧めします。 最初のステップは、すべてのモジュールが[モジュールアップグレードガイド](../building-modules/upgrade.html)に従っていることを確認することです。 2番目のステップは、[アップグレードハンドラー](../core/upgrade.html#running-migrations)を `app.go`に追加することです。

このドキュメントでは、チェーンアップグレードモジュールバージョンのアップグレードハンドラーがどのように見えるかを示すために、初めて例を提供します。 各モジュールの初期コンセンサスバージョンは、[0]ではなく[1]に設定する必要があることに注意してください。設定しないと、アップグレードハンドラが各モジュールを再初期化します。

アップグレードハンドラーは、既存のモジュールの移行に加えて、新しいモジュールのストレージアップグレードも実行します。 以下の例では、v0.44で提供される2つの新しいモジュール `x/authz`と` x/feegrant`にストレージの移行を追加します。

## Cosmovisorを使用する

バリデーターは、アプリケーションのバイナリファイルを実行するためのプロセスマネージャーである[Cosmovisor](../run-node/cosmovisor.html)を使用することをお勧めします。 セキュリティ上の理由から、自動ダウンロードオプションを有効にするのではなく、バリデーターが独自のアップグレードバイナリを構築することをお勧めします。 必要なセキュリティ保証が設定されている場合(つまり、ダウンロード可能なアップグレードバイナリのアップグレード推奨で提供されるURLに正しいチェックサムが含まれている場合)、検証者は自動ダウンロードオプションの使用を選択できます。

::: ヒント
バリデーターが自動ダウンロードオプションを有効にし、現在Cosmos SDK `v0.42`を使用してアプリケーションを実行している場合は、Cosmovisor [` v0.1`](https://github.com)を使用する必要があります。/cosmos/cosmos-sdk/releases/tag/cosmovisor%2Fv0.1.0)。 自動ダウンロードオプションが有効になっている場合、Cosmovisorの新しいバージョンはCosmos SDK`v0.42`以前のバージョンをサポートしません。 
:::

検証者は、自動再起動オプションを使用して、アップグレードプロセス中の不要なダウンタイムを防ぐことができます。 チェーンが推奨されるアップグレードの高さで停止すると、自動再起動オプションは、アップグレードバイナリファイルを使用してチェーンを自動的に再起動します。 自動再起動オプションを使用すると、ベリファイアはアップグレードバイナリファイルを事前に準備し、アップグレード中にリラックスできます。

## app.tomlを移行します

`v0.44`のアップデートにより、新しいサーバー構成オプションが` app.toml`に追加されました。 このアップデートには、RosettaおよびgRPC Webの新しい構成セクションと、状態同期の新しい構成オプションが含まれています。 デフォルトの[`app.toml`](https://github.com/cosmos/cosmos-sdk/blob/release/v0.44.x/server/config/toml.go)ファイル` v0の最新バージョンを表示する.44`詳細。

## 例:Simappのアップグレード

以下示例将使用“simapp”作为我们的区块链应用程序来完成升级过程。我们将把 `simapp` 从 v0.42 升级到 v0.44。我们将自己构建升级二进制文件并启用自动重启选项。

::: ヒント
以下の例では、 `v0.42`から新しいチェーンを開始します。 このバージョンのバイナリファイルは、ジェネシスバイナリファイルになります。 既存のチェーンでCosmovisorを初めて使用する検証者は、チェーンの現在のバージョンのバイナリファイルをジェネシスバイナリファイル(つまり、開始バイナリファイル)として使用するか、検証者が[現在の]シンボリックリンクを更新する必要があります。アップグレードコンテンツをポイントします。 詳しくは、[Cosmovisor](../run-node/cosmovisor.html)をご覧ください。
:::

### デフォルト設定

`cosmos-sdk`リポジトリから、最新の` v0.42.x`バージョンを確認します。

```
git checkout release/v0.42.x
```

Build the `simd` binary for the latest `v0.42.x` release (the genesis binary):

```
make build
```

Reset `~/.simapp` (never do this in a production environment):

```
./build/simd unsafe-reset-all
```

Configure the `simd` binary for testing:

```
./build/simd config chain-id test
./build/simd config keyring-backend test
./build/simd config broadcast-mode block
```

Initialize the node and overwrite any previous genesis file (never do this in a production environment):

<!-- TODO: init does not read chain-id from config -->

```
./build/simd init test --chain-id test --overwrite
```

Set the minimum gas price to `0stake` in `~/.simapp/config/app.toml`:

```
minimum-gas-prices = "0stake"
```

For the purpose of this demonstration, change `voting_period` in `genesis.json` to a reduced time of 20 seconds (`20s`):

```
cat <<< $(jq '.app_state.gov.voting_params.voting_period = "20s"' $HOME/.simapp/config/genesis.json) > $HOME/.simapp/config/genesis.json
```

Create a new key for the validator, then add a genesis account and transaction:

<!-- TODO: add-genesis-account does not read keyring-backend from config -->
<!-- TODO: gentx does not read chain-id from config -->

```
./build/simd keys add validator
./build/simd add-genesis-account validator 5000000000stake --keyring-backend test
./build/simd gentx validator 1000000stake --chain-id test
./build/simd collect-gentxs
```

ノードが初期化されたので、新しい `simapp`チェーンを開始する準備ができました。次に、` cosmovisor`とジェネシスバイナリを設定しましょう。

### コスモバイザー設定

安装 `cosmovisor` 二进制文件: 

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0
```

::: ヒント
go `v1.15`以前を使用している場合は、` cosmos-sdk`ディレクトリを変更し、 `goinstall`の代わりに` go get`を実行してから、 `cosmos-sdk`ストレージライブラリに戻す必要があります。
:::

必要な環境変数を設定します。

```
export DAEMON_NAME=simd
export DAEMON_HOME=$HOME/.simapp
```

Set the optional environment variable to trigger an automatic restart:

```
export DAEMON_RESTART_AFTER_UPGRADE=true
```

ジェネシスバイナリファイル用のフォルダを作成し、 `v0.42.x`バイナリファイルをコピーします。

```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/genesis/bin
```

`cosmovisor`がインストールされ、ジェネシスバイナリファイルが追加されたので、アップグレードハンドラーを` simapp/app.go`に追加して、バイナリファイルをアップグレードする準備をしましょう。

### Chain Upgrade

<!-- TODO: update example to use v0.44.x -->

Check out `release/v0.44.x`:

```
git checkout release/v0.44.x
```

Add the following to `simapp/app.go` inside `NewSimApp` and after `app.UpgradeKeeper`:

```go
	app.registerUpgradeHandlers()
```

`NewSimApp`の後に` simapp/app.go`に以下を追加します(アップグレードハンドラーの詳細については、[In-Place Store Migrations](../core/upgrade.html)を参照してください)。

```go
func (app *SimApp) registerUpgradeHandlers() {
	app.UpgradeKeeper.SetUpgradeHandler("v0.44", func(ctx sdk.Context, plan upgradetypes.Plan, _ module.VersionMap) (module.VersionMap, error) {
		//1st-time running in-store migrations, using 1 as fromVersion to
		//avoid running InitGenesis.
		fromVM := map[string]uint64{
			"auth":         1,
			"bank":         1,
			"capability":   1,
			"crisis":       1,
			"distribution": 1,
			"evidence":     1,
			"gov":          1,
			"mint":         1,
			"params":       1,
			"slashing":     1,
			"staking":      1,
			"upgrade":      1,
			"vesting":      1,
			"ibc":          1,
			"genutil":      1,
			"transfer":     1,
		}

		return app.mm.RunMigrations(ctx, app.configurator, fromVM)
	})

	upgradeInfo, err := app.UpgradeKeeper.ReadUpgradeInfoFromDisk()
	if err != nil {
		panic(err)
	}

	if upgradeInfo.Name == "v0.44" && !app.UpgradeKeeper.IsSkipHeight(upgradeInfo.Height) {
		storeUpgrades := storetypes.StoreUpgrades{
			Added: []string{"authz", "feegrant"},
		}

		//configure store loader that checks if version == upgradeHeight and applies store upgrades
		app.SetStoreLoader(upgradetypes.UpgradeStoreLoader(upgradeInfo.Height, &storeUpgrades))
	}
}
```

Add `storetypes` to imports:

```go
	storetypes "github.com/cosmos/cosmos-sdk/store/types"
```

Build the `simd` binary for `v0.44.x` (the upgrade binary):

```
make build
```

アップグレードバイナリファイル用のフォルダを作成し、 `v0.44.x`バイナリファイルをコピーします。 

```
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/v0.44/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/upgrades/v0.44/bin
```

アップグレードハンドラーを追加し、アップグレードバイナリを準備したので、 `cosmovisor`を起動して、アップグレード提案プロセスをシミュレートする準備が整いました。
### Upgrade Proposal

Start the node using `cosmovisor`:

```
cosmovisor start
```

新しいターミナルウィンドウを開き、アップグレードの提案とデポジットと投票を送信します(これらのコマンドは20秒以内に実行する必要があります)。

```
./build/simd tx gov submit-proposal software-upgrade v0.44 --title upgrade --description upgrade --upgrade-height 20 --from validator --yes
./build/simd tx gov deposit 1 10000000stake --from validator --yes
./build/simd tx gov vote 1 yes --from validator --yes
```

チェーンが高さ20で自動的にアップグレードされることを確認します。 
