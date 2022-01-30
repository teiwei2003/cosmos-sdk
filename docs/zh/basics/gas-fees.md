# 汽油和费用

本文档描述了在 Cosmos SDK 应用程序中处理 gas 和费用的默认策略。 {概要}

### 先决条件阅读

- [Cosmos SDK 应用剖析](./app-anatomy.md) {prereq}

## `Gas` 和 `Fees` 介绍

在 Cosmos SDK 中，`gas` 是一个特殊的单元，用于跟踪执行过程中资源的消耗情况。 `gas` 通常在对存储进行读写时消耗，但如果需要进行昂贵的计算，它也可以消耗。它有两个主要目的/

- 确保块不会消耗太多资源并且将被最终确定。这是通过 [block gas meter](#block-gas-meter) 在 Cosmos SDK 中默认实现的。
- 防止来自最终用户的垃圾邮件和滥用。为此，在 [`message`](../building-modules/messages-and-queries.md#messages) 执行期间消耗的 `gas` 通常是定价的，从而产生 `fee`(`fees = gas * gas -价格`)。 “费用”通常必须由“消息”的发送者支付。请注意，Cosmos SDK 默认不强制执行`gas` 定价，因为可能有其他方法来防止垃圾邮件(例如带宽方案)。尽管如此，大多数应用程序将实施“费用”机制来防止垃圾邮件。这是通过 [`AnteHandler`](#antehandler) 完成的。

## 煤气表

在 Cosmos SDK 中，`gas` 是 `uint64` 的简单别名，由一个名为 _gas meter_ 的对象管理。燃气表实现了`GasMeter`接口

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/store/types/gas.go#L34-L43

在哪里/

- `GasConsumed()` 返回燃气表实例消耗的燃气量。
- `GasConsumedToLimit()` 返回燃气表实例消耗的燃气量，或者达到限制。
- `Limit()` 返回燃气表实例的限制。 `0` 如果燃气表是无限的。
- `ConsumeGas(amount Gas,descriptor string)`消耗提供的`gas`量。如果 `gas` 溢出，它会因 `descriptor` 消息而恐慌。如果燃气表不是无限的，如果消耗的`gas`超过限制，它就会恐慌。
- 如果燃气表实例消耗的燃气量严格高于限制，则`IsPastLimit()` 返回`true`，否则返回`false`。
- 如果燃气表实例消耗的燃气量高于或等于限制，则`IsOutOfGas()` 返回`true`，否则返回`false`。

gas 表一般保存在 [`ctx`](../core/context.md) 中，消耗 gas 的方式如下/

```go
ctx.GasMeter().ConsumeGas(amount, "description")
```

默认情况下，Cosmos SDK 使用两种不同的燃气表，[主燃气表](#main-gas-metter[) 和[块燃气表](#block-gas-meter)。

### 主燃气表

`ctx.GasMeter()` 是应用程序的主要燃气表。主燃气表通过`setDeliverState`在`BeginBlock`中初始化，然后在导致状态转换的执行序列期间跟踪燃气消耗，即最初由[`BeginBlock`](../core/baseapp.md#触发的那些beginblock)、[`DeliverTx`](../core/baseapp.md#delivertx) 和 [`EndBlock`](../core/baseapp.md#endblock)。在每个 `DeliverTx` 开始时，[`AnteHandler`](#antehandler) 中的主燃气表**必须设置为 0**，以便它可以跟踪每笔交易的燃气消耗。

Gas 消耗可以手动完成，通常由模块开发人员在 [`BeginBlocker`, `EndBlocker`](../building-modules/beginblock-endblock.md) 或 [`Msg` 服务](../building- modules/msg-services.md)，但大多数情况下，只要对存储进行读取或写入，它就会自动完成。这种自动gas消耗逻辑在一个名为[`GasKv`](../core/store.md#gaskv-store)的特殊存储中实现。

### 块煤气表

`ctx.BlockGasMeter()` 是燃气表，用于跟踪每个区块的燃气消耗并确保它不会超过某个限制。每次调用 [`BeginBlock`](../core/baseapp.md#beginblock) 时都会创建一个 `BlockGasMeter` 的新实例。 `BlockGasMeter` 是有限的，每个区块的 gas 限制在应用程序的共识参数中定义。默认情况下，Cosmos SDK 应用程序使用 Tendermint 提供的默认共识参数/

+++ https://github.com/tendermint/tendermint/blob/v0.34.0-rc6/types/params.go#L34-L41

当一个新的 [transaction](../core/transactions.md) 正在通过 `DeliverTx` 处理时，会检查 `BlockGasMeter` 的当前值是否超过限制。如果是，`DeliverTx` 立即返回。即使是区块中的第一笔交易，这种情况也可能发生，因为“BeginBlock”本身可以消耗 gas。如果不是，则交易正常处理。在`DeliverTx`结束时，`ctx.BlockGasMeter()`跟踪的gas增加了处理交易消耗的数量/

```go
ctx.BlockGasMeter().ConsumeGas(
	ctx.GasMeter().GasConsumedToLimit(),
	"block gas meter",
)
```

## AnteHandler

在“CheckTx”和“DeliverTx”期间为每个事务运行“AnteHandler”，在事务中每个“sdk.Msg”的 Protobuf“Msg”服务方法之前运行。 `AnteHandler`s 具有以下签名/

```go
//AnteHandler authenticates transactions, before their internal messages are handled.
//If newCtx.IsZero(), ctx is used instead.
type AnteHandler func(ctx Context, tx Tx, simulate bool) (newCtx Context, result Result, abort bool)
```

`anteHandler` 不是在核心 Cosmos SDK 中实现的，而是在一个模块中实现的。这使开发人员可以选择适合其应用程序需要的 `AnteHandler` 版本。也就是说，今天的大多数应用程序都使用 [`auth` 模块](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth) 中定义的默认实现。以下是`anteHandler` 在普通 Cosmos SDK 应用程序中的作用/

- 验证交易类型是否正确。事务类型在实现`anteHandler`的模块中定义，它们遵循事务接口/
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/tx_msg.go#L49-L57
  这使开发人员可以使用各种类型来处理他们的应用程序事务。在默认的 `auth` 模块中，默认的交易类型是 `Tx`/
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L12-L25
- 验证交易中包含的每个 [`message`](../building-modules/messages-and-queries.md#messages) 的签名。每个“消息”应由一个或多个发送者签名，并且这些签名必须在“anteHandler”中进行验证。
- 在`CheckTx`期间，验证交易提供的gas价格是否高于当地的`min-gas-prices`(提醒一下，gas-price可以从以下等式中扣除/`fees = gas * gas-价格`)。 `min-gas-prices` 是每个全节点的本地参数，在 `CheckTx` 期间用于丢弃不提供最低费用的交易。这确保了内存池不会被垃圾交易发送垃圾邮件。
- 验证交易的发送者是否有足够的资金来支付“费用”。当最终用户生成交易时，他们必须指明以下 3 个参数中的 2 个(第三个是隐式参数)/`fees`、`gas` 和 `gas-prices`。这表明他们愿意为节点支付多少来执行他们的交易。提供的 `gas` 值存储在名为 `GasWanted` 的参数中以备后用。
- 将 `newCtx.GasMeter` 设置为 0，限制为 `GasWanted`。 **这一步非常重要**，因为它不仅确保交易不会消耗无限的gas，而且在每个`DeliverTx`之间重置`ctx.GasMeter`(`ctx`设置为`newCtx`在运行`anteHandler` 之后，并且每次调用`DeliverTx` 时都会运行`anteHandler`)。

如上所述，`anteHandler` 返回交易在执行期间可以消耗的最大 `gas` 限制，称为 `GasWanted`。最终消耗的实际数量以`GasUsed`计价，因此我们必须有`GasUsed =< GasWanted`。当 [`DeliverTx`](../core/baseapp.md#delivertx) 返回时，`GasWanted` 和 `GasUsed` 都被中继到底层共识引擎。

## 下一个 {hide}

了解 [baseapp](../core/baseapp.md) {hide} 