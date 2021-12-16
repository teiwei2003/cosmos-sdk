# 事件

`Event`s 是包含有关应用程序执行信息的对象。它们主要被区块浏览器和钱包等服务提供商用来跟踪各种消息和索引交易的执行。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}
- [Tendermint 事件文档](https://docs.tendermint.com/master/spec/abci/abci.html#events) {prereq}

## 事件

事件在 Cosmos SDK 中作为 ABCI `Event` 类型的别名实现，并且
采用以下形式:`{eventType}.{attributeKey}={attributeValue}`。

+++ https://github.com/tendermint/tendermint/blob/v0.34.8/proto/tendermint/abci/types.proto#L304-L313

一个事件包含:

- 一种“类型”，用于在高级别对事件进行分类；例如，Cosmos SDK 使用 `"message"` 类型通过 `Msg` 过滤事件。
- `attributes` 列表是键值对，提供有关分类事件的更多信息。例如，对于 `"message"` 类型，我们可以使用 `message.action={some_action}`、`message.module={some_module}` 或 `message.sender={some_sender} 通过键值对过滤事件`.

::: 小费
要将属性值解析为字符串，请确保在每个属性值周围添加 `'`(单引号)。
:::

事件，`type` 和 `attributes` 在模块的 **per-module 基础** 定义
`/types/events.go` 文件，并从模块的 Protobuf [`Msg` 服务](../building-modules/msg-services.md) 触发
通过使用 [`EventManager`](#eventmanager)。此外，每个模块将其事件记录在
`spec/xx_events.md`。

事件在以下 ABCI 消息的响应中返回到底层共识引擎:

- [`BeginBlock`](./baseapp.md#beginblock)
- [`EndBlock`](./baseapp.md#endblock)
- [`CheckTx`](./baseapp.md#checktx)
- [`DeliverTx`](./baseapp.md#delivertx)

### 例子

以下示例展示了如何使用 Cosmos SDK 查询事件。 

| Event                                            | Description                                                                                                                                              |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tx.height=23`                                   | Query all transactions at height 23                                                                                                                      |
| `message.action='/cosmos.bank.v1beta1.Msg/Send'` | Query all transactions containing a x/bank `Send` [Service `Msg`](../building-modules/msg-services.md). Note the `'`s around the value.                  |
| `message.action='send'`                          | Query all transactions containing a x/bank `Send` [legacy `Msg`](../building-modules/msg-services.md#legacy-amino-msgs). Note the `'`s around the value. |
| `message.module='bank'`                          | Query all transactions containing messages from the x/bank module. Note the `'`s around the value.                                                       |
| `create_validator.validator='cosmosval1...'`     | x/staking-specific Event, see [x/staking SPEC](../../../cosmos-sdk/x/staking/spec/07_events.md).                                                         |

## EventManager

在 Cosmos SDK 应用程序中，事件由称为“EventManager”的抽象管理。
在内部，“EventManager”跟踪一个事件的整个执行流程的列表
交易或`BeginBlock`/`EndBlock`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/events.go#L17-L25

`EventManager` 带有一组有用的方法来管理事件。 方法
模块和应用程序开发人员使用最多的是“EmitEvent”，它跟踪
`EventManager` 中的一个事件。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/events.go#L33-L37

模块开发人员应通过每条消息中的 `EventManager#EmitEvent` 处理事件发射
`Handler` 和每个 `BeginBlock`/`EndBlock` 处理程序。 `EventManager` 是通过访问
[`Context`](./context.md)，其中事件发射通常遵循以下模式: 

```go
ctx.EventManager().EmitEvent(
    sdk.NewEvent(eventType, sdk.NewAttribute(attributeKey, attributeValue)),
)
```

模块的 `handler` 函数还应该为 `context` 设置一个新的 `EventManager`，以隔离每个 `message` 发出的事件:

```go
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        ctx = ctx.WithEventManager(sdk.NewEventManager())
        switch msg := msg.(type) {
```

有关更详细的信息，请参阅 [`Msg` 服务](../building-modules/msg-services.md) 概念文档
查看如何典型地实现事件并在模块中使用 `EventManager`。

## 订阅事件

您可以使用 Tendermint 的 [Websocket](https://docs.tendermint.com/master/tendermint-core/subscription.html#subscribing-to-events-via-websocket) 通过调用 `subscribe` RPC 方法订阅 Events : 

```json
{
  "jsonrpc": "2.0",
  "method": "subscribe",
  "id": "0",
  "params": {
    "query": "tm.event='eventCategory' AND eventType.eventAttribute='attributeValue'"
  }
}
```

您可以订阅的主要`eventCategory`是:

- `NewBlock`:包含在`BeginBlock` 和`EndBlock` 期间触发的事件。
- `Tx`:包含在 `DeliverTx`(即事务处理)期间触发的事件。
- `ValidatorSetUpdates`:包含块的验证器集更新。

这些事件是在提交块后从 `state` 包触发的。 你可以得到
事件类别的完整列表 [在 Tendermint Godoc 页面上](https://godoc.org/github.com/tendermint/tendermint/types#pkg-constants)。

`query` 的 `type` 和 `attribute` 值允许您过滤要查找的特定事件。 例如，一个 `transfer` 交易触发了一个类型为 `Transfer` 的 Event，并且将 `Recipient` 和 `Sender` 作为 `attributes`(在 `bank` 模块的 [`events.go` 文件中定义](https ://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/bank/types/events.go))。 订阅此事件的方式如下: 

```json
{
  "jsonrpc": "2.0",
  "method": "subscribe",
  "id": "0",
  "params": {
    "query": "tm.event='Tx' AND transfer.sender='senderAddress'"
  }
}
```

其中 `senderAddress` 是遵循 [`AccAddress`](../basics/accounts.md#addresses) 格式的地址。

## 类型事件(即将推出)

如前所述，事件是在每个模块的基础上定义的。 定义事件类型和事件属性是模块开发人员的责任。 除了在`spec/XX_events.md`文件中，遗憾的是这些事件类型和属性不容易被发现，所以Cosmos SDK建议使用Protobuf定义的[Typed Events](../architecture/adr-032-typed-events .md) 用于发出和查询事件。

Typed Events 提案尚未完全实施。 文档尚不可用。

## 下一个 {hide}

了解 Cosmos SDK [遥测](./telemetry.md) {hide} 