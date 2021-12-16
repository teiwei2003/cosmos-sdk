# 查询生命周期

本文档描述了 Cosmos SDK 应用程序中查询的生命周期，从用户界面到应用程序存储再返回。 {概要}

## 先决条件阅读

- [交易生命周期](./tx-lifecycle.md) {prereq}

## 查询创建

[**query**](../building-modules/messages-and-queries.md#queries) 是应用程序的最终用户通过接口发出并由全节点处理的信息请求。用户可以直接从应用程序的存储或模块中查询有关网络、应用程序本身和应用程序状态的信息。请注意，查询不同于 [transactions](../core/transactions.md)(查看生命周期 [here](./tx-lifecycle.md))，特别是它们不需要处理共识(如它们不会触发状态转换)；它们可以由一个全节点完全处理。

为了解释查询生命周期，假设“MyQuery”正在请求由名为“simapp”的应用程序中的某个委托地址做出的委托列表。正如预期的那样，[`staking`](../../x/staking/spec/README.md) 模块处理这个查询。但首先，用户可以通过几种方式创建“MyQuery”。

### 命令行界面

应用程序的主界面是命令行界面。用户连接到全节点并直接从他们的机器运行 CLI - CLI 直接与全节点交互。要从他们的终端创建“MyQuery”，用户输入以下命令: 

```bash
simd query staking delegations <delegatorAddress>
```

此查询命令由 [`staking`](../../x/staking/spec/README.md) 模块开发人员定义，并在创建 CLI 时由应用程序开发人员添加到子命令列表中。

注意一般格式如下: 

```bash
simd query [moduleName] [command] <arguments> --flag <flagArg>
```

要提供诸如“--node”(CLI 连接到的完整节点)之类的值，用户可以使用 [`app.toml`](../run-node/run-node.md#configuring-the -node-using-apptoml) 配置文件来设置它们或将它们作为标志提供。

CLI 理解一组特定的命令，由应用程序开发人员在分层结构中定义:从 [root command](../core/cli.md#root-command) (`simd`)，命令类型 ( `Myquery`)，包含命令(`staking`)和命令本身(`delegations`)的模块。因此，CLI 确切地知道哪个模块处理此命令并直接在那里传递调用。

### gRPC

::: 警告
`go-grpc v1.34.0` 中引入的补丁使 gRPC 与 `gogoproto` 库不兼容，导致一些 [gRPC 查询](https://github.com/cosmos/cosmos-sdk/issues/8426) 恐慌。因此，Cosmos SDK 要求在你的 `go.mod` 中安装 `go-grpc <=v1.33.2`。

为确保 gRPC 正常工作，**强烈建议**在您的应用程序的 `go.mod` 中添加以下行: 

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

请参阅 [issue #8392](https://github.com/cosmos/cosmos-sdk/issues/8392) 了解更多信息。
:::

在 Cosmos SDK v0.40 中引入的另一个用户可以进行查询的接口是 [gRPC](https://grpc.io) 对 [gRPC 服务器](../core/grpc_rest.md#grpc-server )。 端点被定义为 `.proto` 文件中的 [Protocol Buffers](https://developers.google.com/protocol-buffers) 服务方法，用 Protobuf 自己的语言无关接口定义语言 (IDL) 编写。 Protobuf 生态系统开发了从“*.proto”文件到各种语言的代码生成工具。 这些工具允许轻松构建 gRPC 客户端。

其中一个工具是 [grpcurl](https://github.com/fullstorydev/grpcurl)，使用该客户端对 `MyQuery` 的 gRPC 请求如下所示:

```bash
grpcurl \
    -plaintext                                           # We want results in plain test
    -import-path ./proto \                               # Import these .proto files
    -proto ./proto/cosmos/staking/v1beta1/query.proto \  # Look into this .proto file for the Query protobuf service
    -d '{"address":"$MY_DELEGATOR"}' \                   # Query arguments
    localhost:9090 \                                     # gRPC server endpoint
    cosmos.staking.v1beta1.Query/Delegations             # Fully-qualified service method name
```

::: REST

用户可以通过其进行查询的另一个接口是通过 HTTP 请求到 [REST 服务器](../core/grpc_rest.md#rest-server)。 REST 服务器完全从 Protobuf 服务自动生成，使用 [gRPC-gateway](https://github.com/grpc-ecosystem/grpc-gateway)。

`MyQuery` 的 HTTP 请求示例如下所示:

```bash
wget http://localhost:1317/cosmos/staking/v1beta1/delegators/{delegatorAddr}/delegations
```

## CLI 如何处理查询

上面的例子展示了外部用户如何通过查询节点的状态来与节点交互。要详细了解查询的确切生命周期，让我们深入研究 CLI 如何准备查询，以及节点如何处理它。从用户的角度来看，交互有点不同，但底层功能几乎相同，因为它们是由模块开发人员定义的相同命令的实现。这一步处理发生在 CLI、gRPC 或 REST 服务器中，并且大量涉及“client.Context”。

::: 语境

在执行 CLI 命令时创建的第一件事是 `client.Context`。 `client.Context` 是一个对象，它存储在用户端处理请求所需的所有数据。特别是，`client.Context` 存储以下内容:

- **Codec**:应用程序使用的[encoder/decoder](../core/encoding.md)，用于在发出Tendermint RPC请求之前编组参数和查询，并将返回的响应解组为JSON对象. CLI 使用的默认编解码器是 Protobuf。
- **Account Decoder**:来自 [`auth`](../..//x/auth/spec/README.md) 模块的帐户解码器，将 `[]byte` 转换为帐户。
- **RPC 客户端**:请求将中继到的 Tendermint RPC 客户端或节点。
- **密钥环**:[密钥管理器](../basics/accounts.md#keyring) 用于签署交易和处理其他使用密钥的操作。
- **Output Writer**:一个 [Writer](https://golang.org/pkg/io/#Writer) 用于输出响应。
- **Configurations**:用户为此命令配置的标志，包括`--height`，指定要查询的区块链高度和`--indent`，表示在JSON响应中添加缩进。

`client.Context` 还包含各种函数，例如 `Query()`，它检索 RPC 客户端并进行 ABCI 调用以将查询中继到全节点。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/context.go#L20-L50

`client.Context` 的主要作用是存储在与最终用户交互期间使用的数据并提供与这些数据交互的方法 - 它在全节点处理查询之前和之后使用。具体来说，在处理“MyQuery”时，“client.Context”用于对查询参数进行编码、检索全节点并写入输出。在被中继到全节点之前，查询需要被编码为“[]byte”形式，因为全节点与应用程序无关并且不了解特定类型。全节点(RPC 客户端)本身是使用 `client.Context` 检索的，它知道用户 CLI 连接到哪个节点。查询被中继到这个全节点进行处理。最后，`client.Context` 包含一个 `Writer`，用于在返回响应时写入输出。这些步骤将在后面的部分中进一步描述。

### 参数和路由创建

在生命周期的这一点上，用户创建了一个 CLI 命令，其中包含他们希望包含在查询中的所有数据。 `client.Context` 的存在是为了协助 `MyQuery` 的其余部分。现在，下一步是解析命令或请求，提取参数，并对所有内容进行编码。这些步骤都发生在他们与之交互的界面中的用户端。

#### 编码

在我们的例子中(查询地址的委托)，`MyQuery` 包含一个 [address](./accounts.md#addresses) `delegatorAddress` 作为它的唯一参数。但是，请求只能包含“[]byte”，因为它将被中继到一个对应用程序类型没有固有知识的全节点的共识引擎(例如 Tendermint Core)。因此，`client.Context` 的`codec` 用于编组地址。

CLI 命令的代码如下所示:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/staking/client/cli/query.go#L324-L327

#### gRPC 查询客户端创建 

Cosmos SDK 利用从 Protobuf 服务生成的代码进行查询。 `staking` 模块的 `MyQuery` 服务生成一个 `queryClient`，CLI 将使用它进行查询。这是相关的代码:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/staking/client/cli/query.go#L318-L342

在底层，`client.Context` 有一个 `Query()` 函数，用于检索预先配置的节点并将查询转发给它；该函数将查询完全限定的服务方法名称作为路径(在我们的例子中:`/cosmos.staking.v1beta1.Query/Delegations`)，并将参数作为参数。它首先检索用户配置的 RPC 客户端(称为 [**node**](../core/node.md))以将此查询中继到，并创建“ABCIQueryOptions”(为 ABCI 调用设置格式的参数) )。然后使用该节点进行 ABCI 调用，`ABCIQueryWithOptions()`。

下面是代码的样子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/query.go#L65-L91

## RPC

通过调用“ABCIQueryWithOptions()”，[全节点](../core/encoding.md) 接收到“MyQuery”，然后它将处理请求。请注意，虽然 RPC 是针对全节点的共识引擎(例如 Tendermint Core)进行的，但查询不是共识的一部分，不会广播到网络的其余部分，因为它们不需要网络需要的任何东西同意。

在 Tendermint 文档 [此处](https://tendermint.com/rpc) 中阅读有关 ABCI 客户端和 Tendermint RPC 的更多信息。

## 应用程序查询处理

当查询在从底层共识引擎中继后被全节点接收时，它现在正在一个理解应用程序特定类型并具有状态副本的环境中处理。 [`baseapp`](../core/baseapp.md) 实现了 ABCI [`Query()`](../core/baseapp.md#query) 函数并处理 gRPC 查询。查询路由被解析，它匹配现有服务方法的完全限定服务方法名称(最有可能在其中一个模块中)，然后`baseapp` 将请求中继到相关模块。

除了 gRPC 路由之外，`baseapp` 还处理四种不同类型的查询:`app`、`store`、`p2p` 和 `custom`。前三种类型(`app`、`store`、`p2p`)纯粹是应用级的，因此直接由`baseapp` 或stores 处理，但是`custom` 查询类型需要`baseapp` 将查询路由到模块的 [legacy 查询器](../building-modules/query-services.md#legacy-queriers)。要了解有关这些查询的更多信息，请参阅 [本指南](../core/grpc_rest.md#tendermint-rpc)。

由于 `MyQuery` 具有来自 `staking` 模块的 Protobuf 完全限定服务方法名称(回想一下 `/cosmos.staking.v1beta1.Query/Delegations`)，`baseapp` 首先解析路径，然后使用其自己的内部 `GRPCQueryRouter ` 检索相应的 gRPC 处理程序，并将查询路由到模块。 gRPC 处理程序负责识别此查询，从应用程序的存储中检索适当的值，并返回响应。阅读有关查询服务的更多信息 [此处](../building-modules/query-services.md)。

一旦从查询器接收到结果，`baseapp` 就会开始向用户返回响应的过程。

:: 回复

由于 `Query()` 是一个 ABCI 函数，`baseapp` 将响应作为 [`abci.ResponseQuery`](https://tendermint.com/docs/spec/abci/abci.html#messages) 类型返回。 `client.Context` `Query()` 例程接收响应并且。

### CLI 响应

应用程序 [`codec`](../core/encoding.md) 用于将响应解组为 JSON，`client.Context` 将输出打印到命令行，应用任何配置，例如输出类型(文本、JSON 或 YAML)。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/context.go#L248-L283

这是一个包装！查询结果由 CLI 输出到控制台。

## 下一个 {hide}

阅读有关 [accounts](./accounts.md) 的更多信息。 {hide}} 