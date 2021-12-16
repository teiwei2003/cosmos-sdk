# `Msg` 服务

Protobuf `Msg` 服务处理 [messages](./messages-and-queries.md#messages)。 Protobuf `Msg` 服务特定于定义它们的模块，并且只处理在所述模块中定义的消息。它们在 [`DeliverTx`](../core/baseapp.md#delivertx) 期间从 `BaseApp` 调用。 {概要}

## 先决条件阅读

- [模块管理器](./module-manager.md) {prereq}
- [消息和查询](./messages-and-queries.md) {prereq}

## 模块`Msg`服务的实现

每个模块都应该定义一个 Protobuf `Msg` 服务，它将负责处理请求(实现 `sdk.Msg`)并返回响应。

正如 [ADR 031](../architecture/adr-031-msg-service.md) 中进一步描述的那样，这种方法的优点是明确指定返回类型并生成服务器和客户端代码。

Protobuf 根据 `Msg` 服务的定义生成一个 `MsgServer` 接口。模块开发人员的职责是通过实现在接收到每个 `sdk.Msg` 时应该发生的状态转换逻辑来实现这个接口。例如，这里是为 `x/bank` 生成的 `MsgServer` 接口，它暴露了两个 `sdk.Msg`:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/bank/types/tx.pb.go#L285-L291

如果可能，现有模块的 [`Keeper`](keeper.md) 应该实现 `MsgServer`，否则可以创建嵌入 `Keeper` 的 `msgServer` 结构，通常在 `./keeper/msg_server.go` 中:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/bank/keeper/msg_server.go#L14-L16

`msgServer` 方法可以使用 `sdk.UnwrapSDKContext` 从 `context.Context` 参数方法中检索 `sdk.Context`:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/bank/keeper/msg_server.go#L27-L28

`sdk.Msg` 处理通常遵循以下 2 个步骤:

- 首先，他们执行 *stateful* 检查以确保 `message` 有效。在这个阶段，已经调用了 `message` 的 `ValidateBasic()` 方法，这意味着已经执行了对消息的 *stateless* 检查(例如确保参数格式正确)。在 `msgServer` 方法中执行的检查可能更昂贵并且需要访问状态。例如，“transfer”消息的“msgServer”方法可能会检查发送帐户是否有足够的资金来实际执行转移。要访问状态，`msgServer` 方法需要调用 [`keeper`'s](./keeper.md) 的 getter 函数。
- 然后，如果检查成功，`msgServer` 方法调用 [`keeper`'s](./keeper.md) setter 函数来实际执行状态转换。

在返回之前，`msgServer` 方法通常通过 `ctx` 中保存的 `EventManager` 发出一个或多个 [events](../core/events.md): 

```go
ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			eventType, //e.g. sdk.EventTypeMessage for a message, types.CustomEventType for a custom event defined in the module
			sdk.NewAttribute(attributeKey, attributeValue),
		),
    )
```

这些事件被转发回底层的共识引擎，服务提供商可以使用这些事件来围绕应用程序实现服务。单击 [此处](../core/events.md) 了解有关事件的更多信息。

调用的 `msgServer` 方法返回一个 `proto.Message` 响应和一个 `error`。然后使用 `sdk.WrapServiceResult(ctx sdk.Context, res proto.Message, err error)` 将这些返回值包装到 `*sdk.Result` 或 `error` 中:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc2/baseapp/msg_service_router.go#L104-L104

此方法负责将 `res` 参数编组到 protobuf，并将 `ctx.EventManager()` 上的任何事件附加到 `sdk.Result`。

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/proto/cosmos/base/abci/v1beta1/abci.proto#L81-L95

此图显示了 Protobuf `Msg` 服务的典型结构，以及消息如何通过模块传播。

![交易流程](../uml/svg/transaction_flow.svg)

## 氨基`LegacyMsg`s

### `处理程序` 类型

Cosmos SDK 中定义的 `handler` 类型将被弃用，取而代之的是 [`Msg` 服务](#implementation-of-a-module-msg-service)。

以下是`handler`函数的典型结构:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc2/types/handler.go#L4-L4

让我们分解一下:

- [`LegacyMsg`](./messages-and-queries.md#messages) 是正在处理的实际对象。
- [`Context`](../core/context.md) 包含处理 `msg` 所需的所有必要信息，以及最新状态的分支。如果 `msg` 处理成功，`ctx` 中包含的状态的分支版本将被写入主状态(分支)。
- 返回给“BaseApp”的“*Result”包含(除其他外)关于“handler”和[events](../core/events.md)执行的信息。

模块“handler”通常在模块文件夹内的“./handler.go”文件中实现。 [模块管理器](./module-manager.md) 用于将模块的`handler`s 添加到
[应用程序的`router`](../core/baseapp.md#message-routing) 通过`Route()` 方法。通常，
manager 的 `Route()` 方法简单地构造了一个调用 `handler.go` 中定义的 `NewHandler()` 方法的路由。

+++ https://github.com/cosmos/cosmos-sdk/blob/228728cce2af8d494c8b4e996d011492139b04ab/x/gov/module.go#L143-L146

### 执行

`NewHandler` 函数将 `LegacyMsg` 分派给适当的处理函数，通常使用 switch 语句:

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/x/bank/handler.go#L13-L29

首先，`NewHandler` 函数为上下文设置一个新的 `EventManager`，以隔离每个 `msg` 的事件。
然后，一个简单的开关根据 `LegacyMsg` 类型调用适当的 `handler`。

对此，每个模块`LegacyMsg`都需要实现`handler`的功能。这也将涉及对 LegacyMsg 类型的手动处理程序注册。
`handler` 的函数应该返回一个 `*Result` 和一个 `error`。

## 遥测

处理消息时，可以从 `msgServer` 方法创建新的 [遥测指标](../core/telemetry.md)。

这是来自 `x/auth/vesting` 模块的一个例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/auth/vesting/msg_server.go#L73-L85

## 下一个 {hide}

了解 [查询服务](./query-services.md) {hide} 