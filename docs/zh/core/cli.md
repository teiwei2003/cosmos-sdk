# 命令行界面

本文档描述了命令行界面 (CLI) 如何在高层次上工作，用于 [**应用程序**](../basics/app-anatomy.md)。可以在 [此处](../building-modules/module-interfaces.md#) 找到为 Cosmos SDK [**module**](../building-modules/intro.md) 实现 CLI 的单独文档cli)。 {概要}

## 命令行界面

### 示例命令

创建 CLI 没有固定的方法，但 Cosmos SDK 模块通常使用 [Cobra 库](https://github.com/spf13/cobra)。使用 Cobra 构建 CLI 需要定义命令、参数和标志。 [**Commands**](#commands) 了解用户希望采取的操作，例如用于创建交易的 `tx` 和用于查询应用程序的 `query`。每个命令还可以有嵌套的子命令，这是命名特定事务类型所必需的。用户还提供**参数**，例如将硬币发送到的帐号，以及 [**Flags**](#flags) 来修改命令的各个方面，例如汽油价格或广播到哪个节点。

下面是一个用户可能输入的命令示例，该命令与 simapp CLI `simd` 交互以发送一些令牌: 

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake --gas auto --gas-prices <gasPrices>
```

前四个字符串指定命令:

- 整个应用程序“simd”的根命令。
- 子命令`tx`，它包含让用户创建交易的所有命令。
- 子命令 `bank` 指示将命令路由到哪个模块([`x/bank`](../../x/bank/spec/README.md) 在这种情况下是模块)。
- 交易类型`send`。

接下来的两个字符串是参数:用户希望发送的“from_address”、收件人的“to_address”以及他们想要发送的“金额”。最后，命令的最后几个字符串是可选标志，用于指示用户愿意支付多少费用(使用用于执行交易的 gas 量和用户提供的 gas 价格计算)。

CLI 与 [node](../core/node.md) 交互以处理此命令。接口本身在 main.go 文件中定义。

### 构建 CLI

`main.go` 文件需要有一个 `main()` 函数来创建一个根命令，所有应用程序命令都将作为子命令添加到该根命令中。 root 命令另外处理:

- 通过读取配置文件(例如 Cosmos SDK 配置文件)来**设置配置**。
- **向其添加任何标志**，例如`--chain-id`。
- **通过调用应用程序的`MakeCodec()`函数(在`simapp`中称为`MakeTestEncodingConfig`)来实例化`codec`**。 [`codec`](../core/encoding.md) 用于对应用程序的数据结构进行编码和解码 - 存储只能持久化 `[]byte`，因此开发人员必须为其数据结构定义序列化格式或使用默认值 Protobuf。
- **为所有可能的用户交互添加子命令**，包括[交易命令](#transaction-commands)和[查询命令](#query-commands)。

`main()` 函数最终创建了一个执行器和 [execute](https://godoc.org/github.com/spf13/cobra#Command.Execute) 根命令。请参阅“simapp”应用程序中的“main()”函数示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/main.go#L12-L24

文档的其余部分将详细说明每个步骤需要实现的内容，并包括来自 `simapp` CLI 文件的较小部分代码。

## 向 CLI 添加命令

每个应用程序 CLI 首先构造一个根命令，然后通过使用 `rootCmd.AddCommand()` 聚合子命令(通常带有进一步嵌套的子命令)来添加功能。应用程序的大部分独特功能在于其事务和查询命令，分别称为“TxCmd”和“QueryCmd”。

### 根命令

root 命令(称为“rootCmd”)是用户首先在命令行中键入以指示他们希望与哪个应用程序交互的命令。用于调用命令的字符串(“Use”字段)通常是后缀为“-d”的应用程序名称，例如`simd` 或 `gaiad`。 root 命令通常包括以下命令以支持应用程序中的基本功能。

- 来自 Cosmos SDK rpc 客户端工具的 **Status** 命令，它打印有关连接的 [`Node`](../core/node.md) 状态的信息。节点的状态包括`NodeInfo`、`SyncInfo`和`ValidatorInfo`。
- **Keys** [commands](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/keys) 来自 Cosmos SDK 客户端工具，其中包括用于使用Cosmos SDK 加密工具中的关键功能，包括添加新密钥并将其保存到密钥环、列出存储在密钥环中的所有公钥以及删除密钥。例如，用户可以输入 `simd keys add <name>` 来添加新密钥并将加密副本保存到密钥环，使用标志 `--recover` 从种子短语或标志`- -multisig` 将多个密钥组合在一起以创建一个多重签名密钥。有关 `add` 键命令的完整详细信息，请参阅代码 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/keys/add.go)。有关使用 `--keyring-backend` 存储密钥凭证的更多详细信息，请查看 [keyring docs](../run-node/keyring.md)。
- 来自 Cosmos SDK 服务器包的 **Server** 命令。这些命令负责提供启动 ABCI Tendermint 应用程序所需的机制，并提供完全引导应用程序所需的 CLI 框架(基于 [cobra](github.com/spf13/cobra))。该包公开了两个核心函数:`StartCmd` 和 `ExportCmd`，它们分别创建用于启动应用程序和导出状态的命令。单击 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/server) 了解更多信息。
- [**交易**](#transaction-commands) 命令。
- [**查询**](#query-commands) 命令。

接下来是来自“simapp”应用程序的“rootCmd”函数示例。它实例化根命令，添加 [_persistent_ flag](#flags) 和在每次执行之前运行的 `PreRun` 函数，并添加所有必要的子命令。

+++ https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/simapp/simd/cmd/root.go#L37-L150

`rootCmd` 有一个名为 `initAppConfig()` 的函数，它对于设置应用程序的自定义配置很有用。
默认情况下，应用程序使用来自 Cosmos SDK 的 Tendermint 应用程序配置模板，可以通过 `initAppConfig()` 覆盖它。
这是覆盖默认`app.toml`模板的示例代码。

+++ https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/simapp/simd/cmd/root.go#L84-L117

`initAppConfig()` 还允许覆盖默认的 Cosmos SDK 的 [服务器配置](https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/server/config/config.go#L199)。一个例子是 `min-gas-prices` 配置，它定义了验证者在处理交易时愿意接受的最低 gas 价格。默认情况下，Cosmos SDK 将此参数设置为 `""`(空字符串)，这会强制所有验证器调整自己的 `app.toml` 并设置一个非空值，否则节点将在启动时停止。这可能不是验证器的最佳 UX，因此链开发人员可以在这个 `initAppConfig()` 函数中为验证器设置一个默认的 `app.toml` 值。

+++ https://github.com/cosmos/cosmos-sdk/blob/aa9b055ddb46aacd4737335a92d0b8a82d577341/simapp/simd/cmd/root.go#L101-L116

根级 `status` 和 `keys` 子命令在大多数应用程序中都很常见，并且不与应用程序状态交互。应用程序的大部分功能——用户实际上可以用它_做的事情——是由它的 `tx` 和 `query` 命令启用的。

### 交易命令

[Transactions](./transactions.md) 是包装 [`Msg`s](../building-modules/messages-and-queries.md#messages) 的对象，它们会触发状态更改。为了能够使用 CLI 界面创建交易，通常会在 `rootCmd` 中添加一个函数 `txCmd`:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L86-L92

这个 `txCmd` 函数为应用程序添加了最终用户可用的所有事务。这通常包括:

- **签名命令**来自 [`auth`](../../x/auth/spec/README.md) 模块，用于在交易中签署消息。要启用多重签名，请添加 `auth` 模块的 `MultiSign` 命令。由于每笔交易都需要某种签名才能有效，因此每个应用程序都需要签名命令。
- **来自 Cosmos SDK 客户端工具的广播命令**，用于广播交易。
- **所有[模块事务命令](../building-modules/module-interfaces.md#transaction-commands)** 应用程序依赖，通过使用[基本模块管理器](../building- modules/module-manager.md#basic-manager) `AddTxCommands()` 函数。

这是一个从“simapp”应用程序聚合这些子命令的“txCmd”示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L123-L149

### 查询命令

[**Queries**](../building-modules/messages-and-queries.md#queries) 是允许用户检索有关应用程序状态信息的对象。为了能够使用 CLI 界面创建交易，通常会在 `rootCmd` 中添加一个函数 `txCmd`:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L86-L92

这个 `queryCmd` 函数为应用程序添加了最终用户可用的所有查询。这通常包括:

- **QueryTx** 和/或其他交易查询命令]来自`auth`模块，允许用户通过输入其哈希值、标签列表或区块高度来搜索交易。这些查询允许用户查看交易是否已包含在区块中。
- 来自 `auth` 模块的 **Account 命令**，显示给定地址的帐户的状态(例如帐户余额)。
- **Validator command** 来自 Cosmos SDK rpc 客户端工具，显示给定高度的验证器集。
- 来自 Cosmos SDK rpc 客户端工具的 **Block 命令**，显示给定高度的块数据。
- **所有[模块查询命令](../building-modules/module-interfaces.md#query-commands)** 应用程序依赖，使用[基本模块管理器](../building- modules/module-manager.md#basic-manager) `AddQueryCommands()` 函数。 

这是一个从“simapp”应用程序聚合子命令的“queryCmd”示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L99-L121

## 标志

标志用于修改命令；开发人员可以使用 CLI 将它们包含在 `flags.go` 文件中。用户可以显式地将它们包含在命令中或通过在其 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 中预先配置它们。通常预先配置的标志包括用户希望与之交互的区块链的“--node”和“--chain-id”。

添加到命令的 _persistent_ 标志(与 _local_ 标志相对)超越其所有子命令:子命令将继承这些标志的配置值。此外，所有标志在添加到命令时都有默认值；有些关闭选项，但其他是用户需要覆盖以创建有效命令的空值。标志可以显式标记为 _required_ 以便在用户不提供值时自动抛出错误，但以不同方式处理意外丢失标志也是可以接受的。

标志直接添加到命令中(通常在定义模块命令的 [模块的 CLI 文件](../building-modules/module-interfaces.md#flags) 中)并且除了 `rootCmd` 持久标志之外没有任何标志必须添加在应用程序级别添加。通常为“--chain-id”(应用程序所属区块链的唯一标识符)添加一个 _persistent_ 标志到 root 命令。添加这个标志可以在`main()`函数中完成。添加此标志是有意义的，因为链 ID 不应在此应用程序 CLI 中跨命令更改。以下是来自“simapp”应用程序的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L118-L119

## 环境变量

每个标志都绑定到它各自命名的环境变量。然后环境变量的名称由两部分组成 - 大写的“basename”后跟标志的标志名称。 `-` 必须替换为 `_`。例如，基本名称为“GAIA”的应用程序的标志“--home”绑定到“GAIA_HOME”。它允许减少为常规操作键入的标志数量。例如，而不是: 

```sh
gaia --home=./--node=<node address> --chain-id="testchain-1" --keyring-backend=test tx ... --from=<key name>
```

这会更方便: 

```sh
# define env variables in .env, .envrc etc
GAIA_HOME=<path to home>
GAIA_NODE=<node address>
GAIA_CHAIN_ID="testchain-1"
GAIA_KEYRING_BACKEND="test"

# and later just use
gaia tx ... --from=<key name>
```

## 配置

应用程序的根命令使用 `PersistentPreRun()` cobra 命令属性来执行命令至关重要，因此所有子命令都可以访问服务器和客户端上下文。这些上下文最初被设置为它们的默认值，并且可能在它们各自的“PersistentPreRun()”函数中被修改，范围限定为命令。请注意，`client.Context` 通常预先填充有“默认”值，这些值可能对所有命令在必要时继承和覆盖很有用。

下面是一个来自 `simapp` 的 `PersistentPreRun()` 函数的例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L54-L60

`SetCmdClientContextHandler` 调用通过 `ReadPersistentCommandFlags` 读取持久标志，它创建一个 `client.Context` 并在根命令的 `Context` 上设置它。

`InterceptConfigsPreRunHandler` 调用创建了一个 viper 文字、默认的 `server.Context` 和一个记录器，并将其设置在根命令的 `Context` 上。 `server.Context` 将通过内部 `interceptConfigs` 调用修改并保存到磁盘，该调用根据提供的主路径读取或创建 Tendermint 配置。此外，`interceptConfigs` 还读取并加载应用程序配置 `app.toml`，并将其绑定到 `server.Context` viper 文字。这是至关重要的，因此应用程序不仅可以访问 CLI 标志，还可以访问此文件提供的应用程序配置值。

## 下一个 {hide}

了解 [事件](./events.md) {hide} 