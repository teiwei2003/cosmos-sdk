# 链升级指南到 v0.44

本文档提供了从 v0.42 到 v0.44 的链式升级指南以及使用“simapp”的升级过程示例。 {概要}

::: 提示
在升级到 v0.44 之前，您必须升级到 Stargate v0.42。如果您还没有这样做，请参阅[链升级指南到 v0.42](/v0.42/migrations/chain-upgrade-guide-040.html)。请注意，v0.43 发布后不久就停产了，所有链应直接从 v0.42 升级到 v0.44。
:::

## 先决条件阅读

- [升级模块](../building-modules/upgrade.html) {prereq}
- [就地存储迁移](../core/upgrade.html) {prereq}
- [Cosmovisor](../run-node/cosmovisor.html) {prereq}

Cosmos SDK v0.44 引入了一种处理链升级的新方法，不再需要将状态导出到 JSON，进行必要的更改，然后使用修改后的 JSON 作为新的创世文件创建新链。

Cosmos SDK 的 IBC 模块已移至 v0.42 及更高版本的[自己的存储库](https://github.com/cosmos/ibc-go)。如果您正在使用 IBC，请确保还通过 [IBC 迁移文档](https://github.com/cosmos/ibc-go/blob/main/docs/migrations/ibc-migration-043.md)。

升级二进制文件将读取现有数据库并执行就地存储迁移，而不是启动新链。这种处理链升级的新方法可以与 [Cosmovisor](../run-node/cosmovisor.html) 一起使用，使升级过程无缝。

## 就地存储迁移

我们建议使用 [In-Place Store Migrations](../core/upgrade.html) 将您的链从 v0.42 升级到 v0.44。第一步是确保您的所有模块都遵循 [模块升级指南](../building-modules/upgrade.html)。第二步是向`app.go`添加一个[升级处理程序](../core/upgrade.html#running-migrations)。

在本文档中，我们将首次提供一个示例，说明链升级模块版本的升级处理程序的样子。需要注意的是，每个模块的初始共识版本必须设置为“1”而不是“0”，否则升级处理程序将重新初始化每个模块。

除了迁移现有模块之外，升级处理程序还为新模块执行存储升级。在下面的示例中，我们将为 v0.44 中提供的两个新模块添加存储迁移:`x/authz` 和 `x/feegrant`。

## 使用 Cosmovisor

我们建议验证器使用 [Cosmovisor](../run-node/cosmovisor.html)，这是一个用于运行应用程序二进制文件的进程管理器。出于安全原因，我们建议验证器构建自己的升级二进制文件，而不是启用自动下载选项。如果必要的安全保证到位(即，可下载升级二进制文件的升级建议中提供的 URL 包括正确的校验和)，验证者仍然可以选择使用自动下载选项。

::: 提示
如果验证者想要启用自动下载选项，并且他们当前正在使用 Cosmos SDK `v0.42` 运行应用程序，他们将需要使用 Cosmovisor [`v0.1`](https://github.com/cosmos/cosmos-sdk/releases/tag/cosmovisor%2Fv0.1.0)。如果启用了自动下载选项，更高版本的 Cosmovisor 不支持 Cosmos SDK `v0.42` 或更早版本。 
:::

验证者可以使用自动重启选项来防止升级过程中出现不必要的停机。一旦链在建议的升级高度停止，自动重启选项将使用升级二进制文件自动重启链。使用自动重启选项，验证者可以提前准备升级二进制文件，然后在升级时放松。

## 迁移 app.toml

随着对 `v0.44` 的更新，新的服务器配置选项已添加到 `app.toml`。更新包括 Rosetta 和 gRPC Web 的新配置部分以及状态同步的新配置选项。查看最新版本的默认 [`app.toml`](https://github.com/cosmos/cosmos-sdk/blob/release/v0.44.x/server/config/toml.go) 文件`v0.44` 了解更多信息。

## 示例:Simapp 升级

以下示例将使用“simapp”作为我们的区块链应用程序来完成升级过程。我们将把 `simapp` 从 v0.42 升级到 v0.44。我们将自己构建升级二进制文件并启用自动重启选项。

::: 提示
在下面的例子中，我们从 `v0.42` 开始一个新链。此版本的二进制文件将是创世二进制文件。对于第一次在现有链上使用 Cosmovisor 的验证者，要么将链当前版本的二进制文件用作创世二进制文件(即起始二进制文件)，要么验证器应更新“当前”符号链接以指向升级目录。有关详细信息，请参阅 [Cosmovisor](../run-node/cosmovisor.html)。
:::

### 初始设置

从 `cosmos-sdk` 存储库中，查看最新的 `v0.42.x` 版本: 

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

现在我们的节点已经初始化，我们准备开始一个新的 `simapp` 链，让我们设置 `cosmovisor` 和 genesis 二进制文件。

### Cosmovisor 设置

安装 `cosmovisor` 二进制文件: 

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0
```

::: 提示
如果你使用的是 go `v1.15` 或更早版本，你需要把 `cosmos-sdk` 目录改出，运行 `go get` 而不是 `go install`，然后再改回 `cosmos-sdk ` 存储库。
:::

设置所需的环境变量: 

```
export DAEMON_NAME=simd
export DAEMON_HOME=$HOME/.simapp
```

Set the optional environment variable to trigger an automatic restart:

```
export DAEMON_RESTART_AFTER_UPGRADE=true
```

为 genesis 二进制文件创建文件夹并复制 `v0.42.x` 二进制文件:

```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/genesis/bin
```

现在已经安装了 `cosmovisor` 并添加了 genesis 二进制文件，让我们将升级处理程序添加到 `simapp/app.go` 并准备升级二进制文件。

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

将以下内容添加到 `NewSimApp` 之后的 `simapp/app.go`(要了解有关升级处理程序的更多信息，请参阅 [In-Place Store Migrations](../core/upgrade.html)): 

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

为升级二进制文件创建文件夹并复制 `v0.44.x` 二进制文件: 

```
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/v0.44/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/upgrades/v0.44/bin
```

现在我们已经添加了升级处理程序并准备了升级二进制文件，我们准备启动 `cosmovisor` 并模拟升级建议过程。
### Upgrade Proposal

Start the node using `cosmovisor`:

```
cosmovisor start
```

打开一个新的终端窗口并提交升级建议以及存款和投票(这些命令必须在 20 秒内运行): 

```
./build/simd tx gov submit-proposal software-upgrade v0.44 --title upgrade --description upgrade --upgrade-height 20 --from validator --yes
./build/simd tx gov deposit 1 10000000stake --from validator --yes
./build/simd tx gov vote 1 yes --from validator --yes
```

确认链在高度 20 自动升级。 
