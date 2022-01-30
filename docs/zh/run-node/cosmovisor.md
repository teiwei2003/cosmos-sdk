# cosmovisor

`cosmovisor` 是 Cosmos SDK 应用程序二进制文件的小型进程管理器，用于监控传入链升级提议的治理模块。如果看到一个提案获得批准，`cosmovisor` 可以自动下载新的二进制文件，停止当前的二进制文件，从旧的二进制文件切换到新的二进制文件，最后用新的二进制文件重新启动节点。

#### 设计

Cosmovisor 旨在用作“Cosmos SDK”应用程序的包装器:

* 它将参数传递给关联的应用程序(由`DAEMON_NAME` 环境变量配置)。
  运行 `cosmovisor run arg1 arg2 ....` 将运行 `app arg1 arg2 ...`；
* 如果需要，它将通过重新启动和升级来管理应用程序；
* 它是使用环境变量配置的，而不是位置参数。

*注意:如果应用程序的新版本未设置为运行就地存储迁移，则需要在使用新二进制文件重新启动 `cosmovisor` 之前手动运行迁移。因此，我们建议应用程序采用就地存储迁移。*

*注意:如果验证者想要启用自动下载选项([我们不推荐](#auto-download))，并且他们当前正在使用 Cosmos SDK `v0.42` 运行应用程序，他们将需要使用 Cosmovisor [`v0.1`](https://github.com/cosmos/cosmos-sdk/releases/tag/cosmovisor%2Fv0.1.0)。如果启用了自动下载选项，更高版本的 Cosmovisor 不支持 Cosmos SDK `v0.44.3` 或更早版本。*

## 贡献

Cosmovisor 是 Cosmos SDK monorepo 的一部分，但它是一个单独的模块，有自己的发布时间表。

发布分支具有以下格式`release/cosmovisor/vA.B.x`，其中 A 和 B 是一个数字(例如`release/cosmovisor/v0.1.x`)。使用以下格式标记版本:`cosmovisor/vA.B.C`。

## 设置

### 安装

要安装最新版本的 `cosmovisor`，请运行以下命令: 

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest
```

要安装以前的版本，您可以指定版本。 重要提示:使用 Cosmos-SDK v0.44.3 或更早版本(例如 v0.44.2)并希望使用自动下载功能的链必须使用 Cosmovisor v0.1.0 

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0
```

使用 Cosmovisor v1.0.0 时可以确认 cosmovisor 的版本，但使用 `v0.1.0` 则无法这样做。 

您还可以通过拉取 cosmos-sdk 存储库并切换到正确的版本并按如下方式构建来从源代码安装: 

```
git clone git@github.com:cosmos/cosmos-sdk
cd cosmos-sdk
git checkout cosmovisor/vx.x.x
cd cosmovisor
make
```

这将在您的当前目录中构建 cosmovisor。 之后，您可能希望将其放入机器的 PATH 中，如下所示: 

```
cp cosmovisor ~/go/bin/cosmovisor
```

*注意:如果您使用的是 go `v1.15` 或更早版本，则需要使用 `go get`，并且您可能希望在项目目录外运行该命令。*

### 命令行参数和环境变量

传递给 `cosmovisor` 的第一个参数是 `cosmovisor` 要采取的操作。选项是:

* `help`、`--help` 或 `-h` - 输出 `cosmovisor` 帮助信息并检查你的 `cosmovisor` 配置。
* `run` - 使用提供的其余参数运行配置的二进制文件。
* `version` 或 `--version` - 输出 `cosmovisor` 版本并使用 `version` 参数运行二进制文件。

传递给“cosmovisor run”的所有参数都将传递给应用程序二进制文件(作为子进程)。 `cosmovisor` 将返回子进程的 `/dev/stdout` 和 `/dev/stderr` 作为它自己的。出于这个原因，`cosmovisor run` 不能接受除应用程序二进制文件可用的命令行参数之外的任何命令行参数。

*注意:不推荐使用没有动作参数之一的`cosmovisor`。为了向后兼容，如果第一个参数不是动作参数，则假定为 `run`。但是，此回退可能会在未来版本中删除，因此建议您始终提供 `run`。

`cosmovisor` 从环境变量中读取其配置:

* `DAEMON_HOME` 是保存 `cosmovisor/` 目录的位置，该目录包含创世二进制文件、升级二进制文件以及与每个二进制文件相关的任何其他辅助文件(例如 `$HOME/.gaiad`、`$HOME/. regend`、`$HOME/.simd` 等)。
* `DAEMON_NAME` 是二进制文件本身的名称(例如`gaiad`、`regend`、`simd` 等)。
* `DAEMON_ALLOW_DOWNLOAD_BINARIES`(*可选*)，如果设置为`true`，将启用新二进制文件的自动下载(出于安全原因，这适用于完整节点而不是验证器)。默认情况下，`cosmovisor` 不会自动下载新的二进制文件。
* `DAEMON_RESTART_AFTER_UPGRADE`(*可选*，默认值 = `true`)，如果为 `true`，则在成功升级后使用相同的命令行参数和标志(但使用新的二进制文件)重新启动子进程。否则 (`false`)，`cosmovisor` 在升级后停止运行，需要系统管理员手动重启。注意restart 是升级后才会出现的，不会在出现错误后自动重启子进程。
* `DAEMON_POLL_INTERVAL` 是轮询升级计划文件的间隔长度。该值可以是数字(以毫秒为单位)或持续时间(例如“1s”)。默认值:300 毫秒。
* `UNSAFE_SKIP_BACKUP`(默认为`false`)，如果设置为`true`，直接升级而不执行备份。否则(`false`，默认)在尝试升级之前备份数据。默认值 false 很有用，建议在发生故障和备份需要回滚时使用。我们建议使用默认备份选项`UNSAFE_SKIP_BACKUP=false`。
* `DAEMON_PREUPGRADE_MAX_RETRIES`(默认为 `0`)。退出状态为“31”后，应用程序中调用“pre-upgrade”的最大次数。达到最大重试次数后，cosmovisor 升级失败。

### 文件夹布局

`$DAEMON_HOME/cosmovisor` 应该完全属于 `cosmovisor` 及其控制的子进程。文件夹内容组织如下: 

```
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── $DAEMON_NAME
└── upgrades
    └── <name>
        ├── bin
        │   └── $DAEMON_NAME
        └── upgrade-info.json
```

`cosmovisor/` 目录为应用程序的每个版本(即 `genesis` 或 `upgrades/<name>`)包含一个子目录。在每个子目录中是应用程序二进制文件(即`bin/$DAEMON_NAME`)和与每个二进制文件关联的任何其他辅助文件。 `current` 是指向当前活动目录(即 `genesis` 或 `upgrades/<name>`)的符号链接。 `upgrades/<name>` 中的 `name` 变量是升级模块计划中指定的升级的 URI 编码名称。

请注意`$DAEMON_HOME/cosmovisor` 只存储*应用程序二进制文件*。 `cosmovisor` 二进制文件本身可以存储在任何典型的位置(例如 `/usr/local/bin`)。应用程序将继续将其数据存储在默认数据目录(例如`$HOME/.gaiad`)或使用`--home` 标志指定的数据目录中。 `$DAEMON_HOME` 独立于数据目录，可以设置为任何位置。如果您将 `$DAEMON_HOME` 设置为与数据目录相同的目录，您将得到如下配置: 

```
.gaiad
├── config
├── data
└── cosmovisor
```

## 用法

系统管理员负责:

- 安装 `cosmovisor` 二进制文件
- 配置主机的初始化系统(例如`systemd`、`launchd`等)
- 适当设置环境变量
- 手动安装 `genesis` 文件夹
- 手动安装 `upgrades/<name>` 文件夹

`cosmovisor` 会在第一次启动时将 `current` 链接设置为指向 `genesis`(即不存在 `current` 链接时)，然后在正确的时间点处理切换二进制文件，以便系统管理员可以提前几天准备并在升级时放松。

为了支持可下载的二进制文件，需要打包每个升级二进制文件的 tarball 并通过规范 URL 提供。此外，包含 genesis 二进制文件和所有可用升级二进制文件的 tarball 可以打包并提供，以便可以轻松下载从一开始就同步全节点所需的所有必要二进制文件。

`DAEMON` 特定的代码和操作(例如，tendermint 配置、应用程序数据库、同步块等)都按预期工作。应用程序二进制文件的指令(例如命令行标志和环境变量)也按预期工作。

### 检测升级

`cosmovisor` 正在轮询 `$DAEMON_HOME/data/upgrade-info.json` 文件以获取新的升级说明。当检测到升级并且区块链达到升级高度时，该文件由“BeginBlocker”中的 x/upgrade 模块创建。
应用以下启发式方法来检测升级:

+ 启动时，`cosmovisor` 不太了解当前正在运行的升级，除了`current/bin/` 二进制文件。它尝试读取“current/update-info.json”文件以获取有关当前升级名称的信息。
+ 如果`cosmovisor/current/upgrade-info.json` 和`data/upgrade-info.json` 都不存在，那么`cosmovisor` 将等待`data/upgrade-info.json` 文件触发升级。
+ 如果`cosmovisor/current/upgrade-info.json` 不存在但`data/upgrade-info.json` 存在，那么`cosmovisor` 假设`data/upgrade-info.json` 中的任何内容都是有效的升级请求。在这种情况下，`cosmovisor` 会立即尝试根据 `data/upgrade-info.json` 中的 `name` 属性进行升级。
+ 否则，`cosmovisor` 会等待 `upgrade-info.json` 中的更改。一旦文件中记录了新的升级名称，`cosmovisor` 将触发升级机制。

当升级机制被触发时，`cosmovisor` 将:

1. 如果启用了`DAEMON_ALLOW_DOWNLOAD_BINARIES`，首先将新的二进制文件自动下载到`cosmovisor/<name>/bin`(其中`<name>` 是`upgrade-info.json:name` 属性)；
2.更新`current`符号链接指向新目录，并将`data/upgrade-info.json`保存到`cosmovisor/current/upgrade-info.json`。

### 自动下载

通常，`cosmovisor` 要求系统管理员在升级之前将所有相关的二进制文件放在磁盘上。然而，对于不需要这种控制并想要自动设置的人(也许他们正在同步一个非验证全节点并且想要做很少的维护)，还有另一种选择。

**注意:我们不建议使用自动下载**，因为它不会提前验证二进制文件是否可用。如果下载二进制文件有任何问题，cosmovisor 将停止并且不会重新启动应用程序(这可能导致链停止)。 

如果 `DAEMON_ALLOW_DOWNLOAD_BINARIES` 设置为 `true`，并且在触发升级时找不到本地二进制文件，`cosmovisor` 将尝试根据 `data` 中的 `info` 属性中的说明下载并安装二进制文件本身/upgrade-info.json` 文件。 这些文件由 x/upgrade 模块构建，并包含来自升级“计划”对象的数据。 `Plan` 有一个信息字段，它应该具有以下两种有效格式之一来指定下载:

1. 将 os/architecture -> 二进制 URI 映射存储在升级计划信息字段中作为“二进制”键下的 JSON。 例如: 

```json
{
  "binaries": {
    "linux/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f"
  }
}
```

您可以一次包含多个二进制文件，以确保多个环境将收到正确的二进制文件: 

```json
{
  "binaries": {
    "linux/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f",
    "linux/arm64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f",
    "darwin/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f"
    }
}
```

将此作为提案提交时，请确保没有空格。 使用“gaiad”的示例命令可能如下所示: 

```
> gaiad tx gov submit-proposal software-upgrade Vega \
--title Vega \
--deposit 100uatom \
--upgrade-height 7368420 \
--upgrade-info '{"binaries":{"linux/amd64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-linux-amd64","linux/arm64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-linux-arm64","darwin/amd64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-darwin-amd64"}}' \
--description "upgrade to Vega" \
--gas 400000 \
--from user \
--chain-id test \
--home test/val2 \
--node tcp://localhost:36657 \
--yes
```

2. 以上述格式存储包含所有信息的文件的链接(例如，如果您想指定大量二进制文件、更改日志信息等，而无需填充区块链)。 例如: 

```
https://example.com/testnet-1001-info.json?checksum=sha256:deaaa99fda9407c4dbe1d04bd49bab0cc3c1dd76fa392cd55a9425be074af01e
```

当 `cosmovisor` 被触发下载新的二进制文件时，`cosmovisor` 会解析 `"binaries"` 字段，使用 [go-getter](https://github.com/hashicorp/go-getter) 下载新的二进制文件 ，并将新的二进制文件解压到 `upgrades/<name>` 文件夹中，这样它就可以像手动安装一样运行。

请注意，为了让此机制提供强大的安全保证，所有 URL 都应包含 SHA 256/512 校验和。 这确保不会运行虚假的二进制文件，即使有人入侵服务器或劫持 DNS。 `go-getter` 将始终确保下载的文件与提供的校验和匹配。 `go-getter` 还将处理将档案解压到目录中(在这种情况下，下载链接应该指向 `bin` 目录中所有数据的 `zip` 文件)。

要在 linux 上正确创建 sha256 校验和，您可以使用 `sha256sum` 实用程序。 例如: 

```
sha256sum ./testdata/repo/zip_directory/autod.zip
```

结果将类似于以下内容:`29139e1381b8177aec909fab9a75d11381cab5adf7d3af0c05ff1c9c117743a7`。

如果您希望使用更长的散列，您也可以使用 `sha512sum`，或者如果您希望使用损坏的散列，您也可以使用 `md5sum`。 无论您选择哪种方式，请确保在 URL 的校验和参数中正确设置哈希算法。

## 示例:SimApp 升级

以下说明使用 Cosmos SDK 源代码附带的模拟应用程序 (`simapp`) 演示了 `cosmovisor`。 以下命令将从 `cosmos-sdk` 存储库中运行。

首先，查看最新的 `v0.42` 版本: 

```
git checkout v0.42.7
```

Compile the `simd` binary:

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

Create a new key for the validator, then add a genesis account and transaction:

<!-- TODO: add-genesis-account does not read keyring-backend from config -->
<!-- TODO: gentx does not read chain-id from config -->

```
./build/simd keys add validator
./build/simd add-genesis-account validator 1000000000stake --keyring-backend test
./build/simd gentx validator 1000000stake --chain-id test
./build/simd collect-gentxs
```

Set the required environment variables:

```
export DAEMON_NAME=simd
export DAEMON_HOME=$HOME/.simapp
```

Set the optional environment variable to trigger an automatic restart:

```
export DAEMON_RESTART_AFTER_UPGRADE=true
```

Create the folder for the genesis binary and copy the `simd` binary:

```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/genesis/bin
```

For the sake of this demonstration, amend `voting_period` in `genesis.json` to a reduced time of 20 seconds (`20s`):

```
cat <<< $(jq '.app_state.gov.voting_params.voting_period = "20s"' $HOME/.simapp/config/genesis.json) > $HOME/.simapp/config/genesis.json
```

Next, we will hardcode a modification in `simapp` to simulate a code change. In `simapp/app.go`, find the line containing the `UpgradeKeeper` initialization. It should look like the following:

```go
app.UpgradeKeeper = upgradekeeper.NewKeeper(skipUpgradeHeights, keys[upgradetypes.StoreKey], appCodec, homePath)
```

After that line, add the following:

```go
app.UpgradeKeeper.SetUpgradeHandler("test1", func(ctx sdk.Context, plan upgradetypes.Plan) {
	//Add some coins to a random account
	addr, err := sdk.AccAddressFromBech32("cosmos18cgkqduwuh253twzmhedesw3l7v3fm37sppt58")
	if err != nil {
		panic(err)
	}
	err = app.BankKeeper.AddCoins(ctx, addr, sdk.Coins{sdk.Coin{Denom: "stake", Amount: sdk.NewInt(345600000)}})
	if err != nil {
		panic(err)
	}
})
```

Now recompile the `simd` binary with the added upgrade handler:

```
make build
```

Create the folder for the upgrade binary and copy the `simd` binary:

```
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/test1/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/upgrades/test1/bin
```

Start `cosmosvisor`:

```
cosmovisor run start
```

Open a new terminal window and submit an upgrade proposal along with a deposit and a vote (these commands must be run within 20 seconds of each other):

```
./build/simd tx gov submit-proposal software-upgrade test1 --title upgrade --description upgrade --upgrade-height 20 --from validator --yes
./build/simd tx gov deposit 1 10000000stake --from validator --yes
./build/simd tx gov vote 1 yes --from validator --yes
```

The upgrade will occur automatically at height 20.
