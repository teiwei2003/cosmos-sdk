# 模块接口

本文档详细介绍了如何为模块构建 CLI 和 REST 接口。包括来自各种 Cosmos SDK 模块的示例。 {概要}

## 先决条件阅读

- [构建模块介绍](./intro.md) {prereq}

##命令行界面

应用程序的主要界面之一是[命令行界面](../core/cli.md)。此入口点添加来自应用程序模块的命令，使最终用户能够创建包裹在事务中的 [**messages**](./messages-and-queries.md#messages) 和 [**queries**](./messages-和-queries.md#queries)。 CLI 文件通常位于模块的`./client/cli` 文件夹中。

### 交易命令

 为了创建触发状态更改的消息，最终用户必须创建 [transactions](../core/transactions.md) 来包装和传递消息。事务命令创建包含一条或多条消息的事务。

 事务命令通常有自己的 `tx.go` 文件，该文件位于模块的 `./client/cli` 文件夹中。命令在 getter 函数中指定，函数名称应包含命令名称。

以下是来自 `x/bank` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/client/cli/tx.go#L28-L63

在该示例中，`NewSendTxCmd()` 为包装和传递 `MsgSend` 的交易创建并返回交易命令。 `MsgSend` 是用于将令牌从一个帐户发送到另一个帐户的消息。

通常，getter 函数执行以下操作:

- **构造命令:** 阅读 [Cobra 文档](https://godoc.org/github.com/spf13/cobra) 以获取有关如何创建命令的更多详细信息。
    - **Use:** 指定调用命令所需的用户输入的格式。在上面的例子中，`send` 是交易命令的名称，`[from_key_or_address]`、`[to_address]` 和 `[amount]` 是参数。
    - **Args:** 用户提供的参数数量。在这种情况下，正好有三个:“[from_key_or_address]”、“[to_address]”和“[amount]”。
    - **短和长:** 命令的描述。需要“简短”描述。 “长”描述可用于提供在用户添加“--help”标志时显示的附加信息。
    - **RunE:** 定义一个可以返回错误的函数。这是执行命令时调用的函数。该函数封装了创建新事务的所有逻辑。
        - 该函数通常从获取 `clientCtx` 开始，这可以通过 `client.GetClientTxContext(cmd)` 来完成。 `clientCtx` 包含与事务处理相关的信息，包括有关用户的信息。在这个例子中，`clientCtx` 用于通过调用 `clientCtx.GetFromAddress()` 来检索发件人的地址。
        - 如果适用，将解析命令的参数。在这个例子中，参数 `[to_address]` 和 `[amount]` 都被解析。
        - [message](./messages-and-queries.md) 是使用解析的参数和来自`clientCtx` 的信息创建的。直接调用消息类型的构造函数。在这种情况下，`types.NewMsgSend(fromAddr, toAddr, amount)`。在创建消息后调用 `msg.ValidateBasic()` 是一种很好的做法，它会对提供的参数运行健全性检查。
        - 根据用户的需求，交易要么离线生成，要么使用 `tx.GenerateOrBroadcastTxCLI(clientCtx, flags, msg)` 签名并广播到预先配置的节点。
- **添加事务标志:** 所有事务命令都必须添加一组事务[flags](#flags)。交易标志用于从用户那里收集附加信息)例如用户愿意支付的费用金额)。使用`AddTxFlagsToCmd(cmd)`将事务标志添加到构造的命令中。
- **返回指令:** 最后返回交易指令。 
每个模块都必须实现`NewTxCmd()`，它聚合了模块的所有事务命令。以下是来自 `x/bank` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/client/cli/tx.go#L13-L26

每个模块还必须为 `AppModuleBasic` 实现 `GetTxCmd()` 方法，该方法只返回 `NewTxCmd()`。这允许 root 命令轻松聚合每个模块的所有事务命令。下面是一个例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/module.go#L75-L78

### 查询命令

[查询](./messages-and-queries.md#queries) 允许用户收集有关应用程序或网络状态的信息；它们由应用程序路由并由定义它们的模块处理。查询命令通常在模块的 `./client/cli` 文件夹中有自己的 `query.go` 文件。与事务命令一样，它们在 getter 函数中指定。以下是来自 `x/auth` 模块的查询命令示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/client/cli/query.go#L75-L105

在示例中，`GetAccountCmd()` 创建并返回一个查询命令，该命令根据提供的帐户地址返回帐户状态。

通常，getter 函数执行以下操作:

- **构造命令:** 阅读 [Cobra 文档](https://godoc.org/github.com/spf13/cobra) 以获取有关如何创建命令的更多详细信息。
    - **Use:** 指定调用命令所需的用户输入的格式。在上面的例子中，`account` 是查询命令的名称，`[address]` 是参数。
    - **Args:** 用户提供的参数数量。在这种情况下，只有一个:`[address]`。
    - **短和长:** 命令的描述。需要“简短”描述。 “长”描述可用于提供在用户添加“--help”标志时显示的附加信息。
    - **RunE:** 定义一个可以返回错误的函数。这是执行命令时调用的函数。该函数封装了创建新查询的所有逻辑。
        - 该函数通常从获取 `clientCtx` 开始，这可以通过 `client.GetClientQueryContext(cmd)` 来完成。 `clientCtx` 包含与查询处理相关的信息。
        - 如果适用，将解析命令的参数。在这个例子中，解析了参数`[address]`。
        - 使用 `NewQueryClient(clientCtx)` 初始化一个新的 `queryClient`。然后使用 `queryClient` 调用适当的 [query](./messages-and-queries.md#grpc-queries)。
        - `clientCtx.PrintProto` 方法用于格式化 `proto.Message` 对象，以便可以将结果打印回给用户。
- **添加查询标志:** 所有查询命令都必须添加一组查询[flags](#flags)。使用`AddQueryFlagsToCmd(cmd)`将查询标志添加到构造的命令中。
- **返回命令:** 最后返回查询命令。

每个模块都必须实现`GetQueryCmd()`，它聚合了模块的所有查询命令。以下是来自 `x/auth` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/client/cli/query.go#L25-L42

每个模块还必须为`AppModuleBasic` 实现`GetQueryCmd()` 方法，该方法返回`GetQueryCmd()` 函数。这允许 root 命令轻松聚合每个模块的所有查询命令。下面是一个例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/module.go#L80-L83

### 标志

[Flags](../core/cli.md#flags) 允许用户自定义命令。 `--fees` 和 `--gas-prices` 是允许用户为其交易设置 [fees](../basics/gas-fees.md) 和 gas 价格的标志示例。

特定于模块的标志通常在模块的 `./client/cli` 文件夹中的 `flags.go` 文件中创建。创建标志时，开发人员设置值类型、标志名称、默认值以及有关标志的描述。开发人员还可以选择将标志标记为 _required_，以便在用户不包含标志值时抛出错误。

这是一个将 `--from` 标志添加到命令的示例: 

```go
cmd.Flags().String(FlagFrom, "", "Name or address of private key with which to sign")
```

在本例中，flag的值为`String`，flag的名称为`from`)`FlagFrom`常量的值)，flag的默认值为`""`，有 当用户将“--help”添加到命令时将显示的描述。

这是一个将 `--from` 标志标记为 _required_ 的示例: 

```go
cmd.MarkFlagRequired(FlagFrom)
```

有关创建标志的更多详细信息，请访问 [Cobra 文档](https://github.com/spf13/cobra)。

如[事务命令](#transaction-commands) 中所述，所有事务命令都必须添加一组标志。这是通过 Cosmos SDK 的 `./client/flags` 包中定义的 `AddTxFlagsToCmd` 方法完成的。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/client/flags/flags.go#L94-L120

由于 `AddTxFlagsToCmd(cmd *cobra.Command)` 包括事务命令所需的所有基本标志，模块开发人员可以选择不添加任何自己的标志)改为指定参数通常更合适)。

类似地，有一个 `AddQueryFlagsToCmd(cmd *cobra.Command)` 来向模块查询命令添加通用标志。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/client/flags/flags.go#L85-L92

## gRPC

[gRPC](https://grpc.io/) 是一个远程过程调用 (RPC) 框架。 RPC 是钱包和交易所等外部客户端与区块链交互的首选方式。

除了提供 ABCI 查询路径外，Cosmos SDK 还提供了 gRPC 代理服务器，将 gRPC 查询请求路由到 ABCI 查询请求。

为此，模块必须在“AppModuleBasic”上实现“RegisterGRPCGatewayRoutes(clientCtx client.Context, mux *runtime.ServeMux)”，以将客户端 gRPC 请求连接到模块内的正确处理程序。

下面是来自 `x/auth` 模块的一个例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/module.go#L68-L73

## gRPC 网关 REST

应用程序需要支持使用 HTTP 请求的网络服务)例如像 [Keplr](https://keplr.xyz) 这样的网络钱包)。 [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) 将 REST 调用转换为 gRPC 调用，这可能对不使用 gRPC 的客户端有用。

想要公开 REST 查询的模块应该将 `google.api.http` 注释添加到它们的 `rpc` 方法中，例如下面来自 `x/auth` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/auth/v1beta1/query.proto#L13-L29

gRPC 网关与应用程序和 Tendermint 一起在进程内启动。可以通过在 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 中设置 gRPC 配置 `enable` 来启用或禁用它。

Cosmos SDK 提供了用于生成 [Swagger](https://swagger.io/) 文档 (`protoc-gen-swagger`) 的命令。在 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 中设置 `swagger` 定义是否应该自动注册 swagger 文档。

## 下一个 {hide}

阅读推荐的 [模块结构](./structure.md) {hide} 