# ADR 031:Protobuf 消息服务

## 变更日志

- 2020-10-05:初稿
- 2021-04-21:删除 `ServiceMsg`s 以遵循 Protobuf `Any` 的规范，参见 [#9063](https://github.com/cosmos/cosmos-sdk/issues/9063)。

## 地位

公认

## 摘要

我们希望利用 protobuf `service` 定义来定义 `Msg`s，这将为我们提供重要的开发人员 UX
生成的代码方面的改进以及返回类型现在将得到明确定义的事实。

## 语境

目前，Cosmos SDK 中的 `Msg` 处理程序确实具有放置在响应的 `data` 字段中的返回值。
但是，除了在 golang 处理程序代码中，没有在任何地方指定这些返回值。

在早期对话中[有人提出](https://docs.google.com/document/d/1eEgYgvgZqLE45vETjhwIw4VOqK-5hwQtZtjVbiXnIGc/edit)
使用 protobuf 扩展字段捕获 `Msg` 返回类型 , ex:

```protobuf
package cosmos.gov;

message MsgSubmitProposal
	option (cosmos_proto.msg_return) = “uint64”;
	string delegator_address = 1;
	string validator_address = 2;
	repeated sdk.Coin amount = 3;
}
```

然而，这从未被采纳。

为`Msg`s 指定明确的返回值将改善客户端用户体验。例如，
在`x/gov` 中，`MsgSubmitProposal` 将提案ID 作为大端`uint64` 返回。
这在任何地方都没有真正记录下来，客户需要了解内部结构
Cosmos SDK 解析该值并将其返回给用户。

此外，在某些情况下，我们可能希望以编程方式使用这些返回值。
例如，https://github.com/cosmos/cosmos-sdk/issues/7093 提出了一种方法
使用 `Msg` 路由器进行模块间 Ocaps。一个明确定义的返回类型将
改进此方法的开发人员用户体验。

此外，`Msg` 类型的处理程序注册往往会增加一些
保持器顶部的样板，通常通过手动类型开关完成。
这不一定是坏事，但它确实增加了创建模块的开销。

## 决定

我们决定使用 protobuf `service` 定义来定义 `Msg`s 以及
他们生成的代码作为 `Msg` 处理程序的替代品。

下面我们定义这将如何查找来自 `x/gov` 模块的 `SubmitProposal` 消息。
我们从 `Msg` `service` 定义开始: 

```proto
package cosmos.gov;

service Msg {
  rpc SubmitProposal(MsgSubmitProposal) returns (MsgSubmitProposalResponse);
}

// Note that for backwards compatibility this uses MsgSubmitProposal as the request
// type instead of the more canonical MsgSubmitProposalRequest
message MsgSubmitProposal {
  google.protobuf.Any content = 1;
  string proposer = 2;
}

message MsgSubmitProposalResponse {
  uint64 proposal_id;
}
```

虽然这最常用于 gRPC，但像这样重载 protobuf 的“服务”定义并不违反
[protobuf 规范](https://developers.google.com/protocol-buffers/docs/proto3#services) 的意图是:
> 如果您不想使用 gRPC，也可以在您自己的 RPC 实现中使用协议缓冲区。
使用这种方法，我们将获得一个自动生成的 `MsgServer` 接口:

除了明确指定返回类型之外，这还有利于生成客户端和服务器代码。 在服务器上
一方面，这几乎就像一个自动生成的守门员方法，最终可能会被用于代替守门员
(见 [\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093)): 

```go
package gov

type MsgServer interface {
  SubmitProposal(context.Context, *MsgSubmitProposal) (*MsgSubmitProposalResponse, error)
}
```

在客户端，开发人员可以通过创建封装事务的 RPC 实现来利用这一点
逻辑。使用异步回调的 Protobuf 库，如 [protobuf.js](https://github.com/protobufjs/protobuf.js#using-services)
可以使用它来注册特定消息的回调，即使是包含多个 `Msg` 的事务。

每个 `Msg` 服务方法都应该有一个请求参数:它对应的 `Msg` 类型。例如，上面的`Msg`服务方法`/cosmos.gov.v1beta1.Msg/SubmitProposal`只有一个请求参数，即`Msg`类型`/cosmos.gov.v1beta1.MsgSubmitProposal`。重要的是读者必须清楚地理解 `Msg` 服务(Protobuf 服务)和 `Msg` 类型(Protobuf 消息)之间的命名差异，以及它们的完全限定名称的差异。

这个约定已经决定在更规范的 `Msg...Request` 名称上主要是为了向后兼容，但也为了更好的可读性在 `TxBody.messages`(见下面的 [编码部分](#encoding)):包含 `/ cosmos.gov.MsgSubmitProposal` 比那些包含 `/cosmos.gov.v1beta1.MsgSubmitProposalRequest` 的读起来更好。

这种约定的一个后果是每个 `Msg` 类型只能是一个 `Msg` 服务方法的请求参数。但是，我们认为这种限制是明确的一种很好的做法。

### 编码

使用 `Msg` 服务生成的交易编码与 [ADR-020](./adr-020-protobuf-transaction-encoding.md) 中定义的当前 Protobuf 交易编码没有区别。我们将 `Msg` 类型(正是 `Msg` 服务方法的请求参数)编码为 `Tx` 中的 `Any`，这涉及打包
二进制编码的 `Msg` 及其类型的 URL。

###解码

由于 `Msg` 类型被打包到 `Any` 中，所以通过将 `Any`s 解包到 `Msg` 类型来解码交易消息。更多信息请参考[ADR-020](./adr-020-protobuf-transaction-encoding.md#transactions)。

### 路由

我们建议在 BaseApp 中添加一个 `msg_service_router`。这个路由器是一个键/值映射，它将 `Msg` 类型'`type_url`s 映射到它们相应的 `Msg` 服务方法处理程序。由于 `Msg` 类型和 `Msg` 服务方法之间存在一对一的映射，所以 `msg_service_router` 每个 `Msg` 服务方法只有一个条目。

当 BaseApp 处理交易(在 CheckTx 或 DeliverTx 中)时，其“TxBody.messages”被解码为“Msg”。每个 `Msg` 的 `type_url` 都与 `msg_service_router` 中的一个条目匹配，并调用相应的 `Msg` 服务方法处理程序。

为了向后兼容，旧的处理程序还没有被删除。如果 BaseApp 接收到在 `msg_service_router` 中没有对应条目的旧版 `Msg`，它将通过其旧版 `Route()` 方法路由到旧版处理程序中。

### 模块配置

在[ADR 021](./adr-021-protobuf-query-encoding.md)中，我们引入了一个方法`RegisterQueryService`
到“AppModule”，它允许模块注册 gRPC 查询器。

为了注册 `Msg` 服务，我们尝试通过转换 `RegisterQueryService` 来尝试一种更具扩展性的方法
到更通用的`RegisterServices`方法: 

```go
type AppModule interface {
  RegisterServices(Configurator)
  ...
}

type Configurator interface {
  QueryServer() grpc.Server
  MsgServer() grpc.Server
}

// example module:
func (am AppModule) RegisterServices(cfg Configurator) {
	types.RegisterQueryServer(cfg.QueryServer(), keeper)
	types.RegisterMsgServer(cfg.MsgServer(), keeper)
}
```

`RegisterServices` 方法和 `Configurator` 接口旨在
进化以满足 [\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093) 中讨论的用例
和 [\#7122](https://github.com/cosmos/cosmos-sdk/issues/7421)。

当注册`Msg` 服务时，框架_应该_验证所有`Msg` 类型
实现 `sdk.Msg` 接口并在初始化期间抛出错误而不是
比在处理交易时更晚。

### `Msg` 服务实现

就像查询服务一样，`Msg` 服务方法可以检索`sdk.Context`
来自使用 `sdk.UnwrapSDKContext` 的 `context.Context` 参数方法
方法: 

```go
package gov

func (k Keeper) SubmitProposal(goCtx context.Context, params *types.MsgSubmitProposal) (*MsgSubmitProposalResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)
    ...
}
```

`sdk.Context` 应该有一个已经由 BaseApp 的 `msg_service_router` 附加的 `EventManager`。

这种方法不再需要单独的处理程序定义。

## 结果

这种设计改变了模块功能的公开和访问方式。它弃用了现有的 `Handler` 接口和 `AppModule.Route`，转而支持上述 [Protocol Buffer Services](https://developers.google.com/protocol-buffers/docs/proto3#services) 和服务路由。这极大地简化了代码。我们不再需要创建处理程序和守护程序。使用协议缓冲区自动生成的客户端清楚地分离了模块和模块用户之间的通信接口。控制逻辑(又名处理程序和守护程序)不再公开。模块接口可以看作是通过客户端 API 访问的黑匣子。值得注意的是，客户端接口也是由 Protocol Buffers 生成的。

这也允许我们改变我们执行功能测试的方式。我们将模拟客户端而不是模拟 AppModules 和路由器(服务器将保持隐藏)。更具体地说:我们永远不会在 `moduleB` 中模拟 `moduleA.MsgServer`，而是模拟 `moduleA.MsgClient`。可以将其视为使用外部服务(例如数据库或在线服务器...)。我们假设客户端和服务器之间的传输由生成的协议缓冲区正确处理。

最后，关闭客户端 API 的模块会打开 ADR-033 中讨论的理想 OCAP 模式。由于服务器实现和接口是隐藏的，没有人可以持有“管理员”/服务器，并且将被迫在客户端接口上进行中继，这将推动开发人员进行正确的封装和软件工程模式。

### 优点

- 清楚地传达返回类型
- 不再需要手动处理程序注册和返回类型编组，只需实现接口并注册即可
- 自动生成通信接口，开发人员现在可以只关注状态转换方法 - 这将改善 [\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093) 方法的 UX (1) 如果我们选择采用
- 生成的客户端代码可能对客户端和测试有用
- 大大减少和简化了代码

### 缺点

- 在 gRPC 上下文之外使用 `service` 定义可能会造成混淆(但不违反 proto3 规范)

## 参考

- [初始 Github 问题 \#7122](https://github.com/cosmos/cosmos-sdk/issues/7122)
- [proto 3 语言指南:定义服务](https://developers.google.com/protocol-buffers/docs/proto3#services)
- [初始预`Any``Msg`设计](https://docs.google.com/document/d/1eEgYgvgZqLE45vETjhwIw4VOqK-5hwQtZtjVbiXnIGc)
- [ADR 020](./adr-020-protobuf-transaction-encoding.md)
- [ADR 021](./adr-021-protobuf-query-encoding.md) 