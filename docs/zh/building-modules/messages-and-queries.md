# 消息和查询

`Msg`s 和 `Queries` 是模块处理的两个主要对象。模块中定义的大多数核心组件，如 `Msg` 服务、`keeper`s 和 `Query` 服务，都用于处理 `message`s 和 `queries`。 {概要}

## 先决条件阅读

- [Cosmos SDK 模块介绍](./intro.md) {prereq}

## 消息

`Msg`s 是其最终目标是触发状态转换的对象。它们包含在 [transactions](../core/transactions.md) 中，其中可能包含其中一个或多个。

当交易从底层共识引擎中继到 Cosmos SDK 应用程序时，它首先被 [`BaseApp`](../core/baseapp.md) 解码。然后，事务中包含的每条消息都被提取并通过`BaseApp` 的`MsgServiceRouter` 路由到适当的模块，以便模块的[`Msg` 服务](./msg-services.md) 处理它。有关事务生命周期的更详细说明，请单击 [此处](../basics/tx-lifecycle.md)。

### `Msg` 服务

从 v0.40 开始，定义 Protobuf `Msg` 服务是处理消息的推荐方式。应该为每个模块创建 Protobuf `Msg` 服务，通常在 `tx.proto` 中(请参阅有关 [约定和命名](../core/encoding.md#faq) 的更多信息)。它必须为模块中的每条消息定义一个 RPC 服务方法。

查看来自 `x/bank` 模块的 `Msg` 服务定义示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/proto/cosmos/bank/v1beta1/tx.proto#L10-L17

每个 `Msg` 服务方法必须只有一个参数，该参数必须实现 `sdk.Msg` 接口和一个 Protobuf 响应。命名约定是调用 RPC 参数 `Msg<service-rpc-name>` 和 RPC 响应 `Msg<service-rpc-name>Response`。例如: 

```
  rpc Send(MsgSend) returns (MsgSendResponse);
```

`sdk.Msg` 接口是 [下文](#legacy-amino-msgs) 描述的 Amino `LegacyMsg` 接口的简化版本，只有 `ValidateBasic()` 和 `GetSigners()` 方法。为了向后兼容 [Amino `LegacyMsg`s](#legacy-amino-msgs)，现有的 `LegacyMsg` 类型应该用作 `service` RPC 定义的请求参数。较新的 `sdk.Msg` 类型只支持 `service` 定义，应该使用规范的 `Msg...` 名称。

Cosmos SDK 使用 Protobuf 定义生成客户端和服务器代码:

* `MsgServer` 接口定义了 `Msg` 服务的服务器 API，其实现在 [`Msg` 服务](./msg-services.md) 文档中进行了描述。
* 为所有 RPC 请求和响应类型生成结构。

还生成了一个 `RegisterMsgServer` 方法，它应该用于在来自 [`AppModule` 接口](./module-manager.md#appmodule) 的 `RegisterServices` 方法中注册模块的 `MsgServer` 实现。

为了让客户端(CLI 和 grpc-gateway)注册这些 URL，Cosmos SDK 提供了函数 `RegisterMsgServiceDesc(registry codectypes.InterfaceRegistry, sd *grpc.ServiceDesc)`，应该在模块的 [`RegisterInterfaces`]( module-manager.md#appmodulebasic) 方法，使用原型生成的 `&_Msg_serviceDesc` 作为 `*grpc.ServiceDesc` 参数。

### Legacy Amino `LegacyMsg`s

不推荐使用以下定义消息的方式，首选使用 [`Msg` 服务](#msg-services)。

Amino `LegacyMsg`s 可以定义为 protobuf 消息。消息定义通常包括处理消息所需的参数列表，当最终用户想要创建包含所述消息的新事务时，他们将提供这些参数。

`LegacyMsg` 通常伴随着一个标准的构造函数，它是从 [module's interface](./module-interfaces.md) 之一调用的。 `message`s 还需要实现 `sdk.Msg` 接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/4a1b2fba43b1052ca162b3a1e0b6db6db9c26656/types/tx_msg.go#L10-L33

它扩展了 `proto.Message` 并包含以下方法:

- `Route() string`:此消息的路由名称。通常，模块中的所有“消息”都具有相同的路由，通常是模块的名称。
- `Type() string`:消息的类型，主要用于 [events](../core/events.md)。这应该返回一个特定于消息的“字符串”，通常是消息本身的面额。
- `ValidateBasic() error`:这个方法在处理`message`(在[`CheckTx`](../core/baseapp.md#checktx)和[`DeliverTx中)的早期就被`BaseApp`调用`](../core/baseapp.md#delivertx))，以便丢弃明显无效的消息。 `ValidateBasic` 应该只包括 *stateless* 检查，即不需要访问状态的检查。这通常包括检查消息的参数格式是否正确且有效(即“金额”对于传输而言严格为正数)。
- `GetSignBytes() []byte`:返回消息的规范字节表示。用于生成签名。
- `GetSigners() []AccAddress`:返回签名者列表。 Cosmos SDK 将确保交易中包含的每条“消息”都由此方法返回的列表中列出的所有签名者签名。

查看来自 `gov` 模块的 `message` 的示例实现:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/gov/types/msgs.go#L77-L125

## 查询

“查询”是应用程序的最终用户通过接口发出并由全节点处理的信息请求。全节点通过其共识引擎接收“查询”，并通过 ABCI 中继到应用程序。然后通过`BaseApp` 的`queryrouter` 将其路由到适当的模块，以便模块的查询服务(./query-services.md) 可以对其进行处理。要更深入地了解“查询”的生命周期，请单击 [此处](../basics/query-lifecycle.md)。

### gRPC 查询 

从 v0.40 开始，定义查询的首选方法是使用 [Protobuf 服务](https://developers.google.com/protocol-buffers/docs/proto#services)。应该在 `query.proto` 中为每个模块创建一个 `Query` 服务。该服务列出了以 `rpc` 开头的端点。

以下是此类“查询”服务定义的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/proto/cosmos/auth/v1beta1/query.proto#L12-L23

作为`proto.Message`s，生成的`Response`类型默认实现[`fmt.Stringer`](https://golang.org/pkg/fmt/#Stringer)的`String()`方法。

还生成了一个 `RegisterQueryServer` 方法，应该用于在 [`AppModule` 接口](./module-manager.md#appmodule) 的 `RegisterServices` 方法中注册模块的查询服务器。

### 遗留查询

在 Cosmos SDK 中引入 Protobuf 和 gRPC 之前，通常没有模块开发人员定义的特定“查询”对象，与“消息”相反。相反，Cosmos SDK 采用了更简单的方法，使用简单的“路径”来定义每个“查询”。 `path` 包含 `query` 类型和处理它所需的所有参数。对于大多数模块查询，`path` 应该如下所示:

``
queryCategory/queryRoute/queryType/arg1/arg2/...
``

在哪里:

- `queryCategory` 是`query` 的类别，通常用于模块查询的`custom`。它用于区分 `BaseApp` 的 [`Query` 方法](../core/baseapp.md#query) 中不同类型的查询。
- `baseApp` 的 [`queryRouter`](../core/baseapp.md#query-routing) 使用 `queryRoute` 将 `query` 映射到其模块。通常，`queryRoute` 应该是模块的名称。
- 模块的 [`querier`](./query-services.md#legacy-queriers) 使用 `queryType` 将 `query` 映射到模块内适当的 `querier function`。
- `args` 是处理 `query` 所需的实际参数。它们由最终用户填写。请注意，对于更大的查询，您可能更喜欢在请求 `req` 的 `Data` 字段而不是 `path` 中传递参数。

每个`query`的`path`必须由模块开发者在模块的[命令行界面文件](./module-interfaces.md#query-commands)中定义。总的来说，模块开发者需要3个主要组件实现以使由其模块定义的状态子集可查询:

- [`querier`](./query-services.md#legacy-queriers)，一旦它被[路由到模块](../core/baseapp.md#query-routing) 处理`query` )。
- [查询命令](./module-interfaces.md#query-commands) 在模块的 CLI 文件中，其中指定了每个 `query` 的 `path`。
- `query` 返回类型。通常在文件 `types/querier.go` 中定义，它们指定每个模块的 `queries` 的结果类型。这些自定义类型必须实现 [`fmt.Stringer`](https://golang.org/pkg/fmt/#Stringer) 的 `String()` 方法。

### 存储查询

存储查询直接查询存储键。他们使用 `clientCtx.QueryABCI(req abci.RequestQuery)` 返回完整的 `abci.ResponseQuery` 并包含 Merkle 证明。

请参阅以下示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/080fcf1df25ccdf97f3029b6b6f83caaf5a235e4/x/ibc/core/client/query.go#L36-L46

+++ https://github.com/cosmos/cosmos-sdk/blob/080fcf1df25ccdf97f3029b6b6f83caaf5a235e4/baseapp/abci.go#L722-L749

## 下一个 {hide}

了解 [`Msg` 服务](./msg-services.md) {hide} 