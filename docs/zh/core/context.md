#  上下文

`context` 是一种数据结构，旨在从一个函数传递到另一个函数，它携带有关应用程序当前状态的信息。它提供了对分支存储(整个状态的安全分支)以及有用的对象和信息的访问，如“gasMeter”、“块高度”、“共识参数”等。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}
- [交易的生命周期](../basics/tx-lifecycle.md) {prereq}

## 上下文定义

Cosmos SDK `Context` 是一个自定义数据结构，它包含 Go 的 stdlib [`context`](https://golang.org/pkg/context) 作为其基础，并且在其定义中具有许多特定于Cosmos SDK。 `Context` 是事务处理不可或缺的一部分，因为它允许模块轻松访问 [`multistore`](./store.md#multistore) 中各自的 [store](./store.md#base-layer-kvstores) ) 并检索交易上下文，例如区块头和燃气表。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/context.go#L16-L39

- **Context:** 基本类型是 Go [Context](https://golang.org/pkg/context)，在 [Go Context Package](#go-context-package) 部分进一步解释以下。
- **Multistore:** 每个应用程序的 `BaseApp` 都包含一个 [`CommitMultiStore`](./store.md#multistore)，它在创建 `Context` 时提供。调用 `KVStore()` 和 `TransientStore()` 方法允许模块使用它们唯一的 `StoreKey` 获取它们各自的 [`KVStore`](./store.md#base-layer-kvstores)。
- **ABCI Header:** [header](https://tendermint.com/docs/spec/abci/abci.html#header) 是一种 ABCI 类型。它携带有关区块链状态的重要信息，例如当前区块的区块高度和提议者。
- **Chain ID:** 区块所属区块链的唯一标识号。
- **交易字节数:** 正在使用上下文处理的交易的 `[]byte` 表示。每笔交易都由 Cosmos SDK 的各个部分和共识引擎(例如 Tendermint)在其整个 [生命周期](../basics/tx-lifecycle.md) 中处理，其中一些对交易类型没有任何了解。因此，事务使用某种[编码格式](./encoding.md)，例如[Amino](./encoding.md)，被编组为通用的“[]byte”类型。
- **Logger:** 来自 Tendermint 库的 `logger`。了解有关日志的更多信息 [此处](https://tendermint.com/docs/tendermint-core/how-to-read-logs.html#how-to-read-logs)。模块调用此方法来创建自己独特的特定于模块的记录器。
- **VoteInfo:** ABCI 类型列表 [`VoteInfo`](https://tendermint.com/docs/spec/abci/abci.html#voteinfo)，其中包括验证器的名称和布尔值表明他们是否已经签署了区块。
- **Gas Meters:** 具体来说，一个 [`gasMeter`](../basics/gas-fees.md#main-gas-meter) 用于当前正在使用上下文处理的交易和一个 [`blockGasMeter`](../basics/gas-fees.md#block-gas-meter) 用于它所属的整个块。用户指定他们希望为执行交易支付多少费用；这些燃气表跟踪到目前为止在交易或区块中使用了多少 [gas](../basics/gas-fees.md)。如果燃气表用完，则执行停止。
- **CheckTx 模式:** 一个布尔值，指示交易应在“CheckTx”还是“DeliverTx”模式下处理。
- **Min Gas Price:** 节点为了在其区块中包含交易而愿意接受的最低 [gas](../basics/gas-fees.md) 价格。此价格是由每个节点单独配置的本地值，因此**不应在导致状态转换的序列中使用的任何函数中使用**。
- **Consensus Params:** ABCI 类型 [Consensus Parameters](https://tendermint.com/docs/spec/abci/apps.html#consensus-parameters)，指定区块链的某些限制，例如最大值一个块的GAS费。
- **事件管理器:** 事件管理器允许任何有权访问 `Context` 的调用者发出 [`Events`](./events.md)。模块可以定义特定于模块的
  `Events` 通过定义各种 `Types` 和 `Attributes` 或使用在 `types/` 中找到的通用定义。客户可以订阅或查询这些“事件”。这些“事件”在整个“DeliverTx”、“BeginBlock”和“EndBlock”中收集，并返回给 Tendermint 进行索引。例如: 

```go
ctx.EventManager().EmitEvent(sdk.NewEvent(
    sdk.EventTypeMessage,
    sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory)),
)
```

## 去上下文包

[Golang Context Package](https://golang.org/pkg/context) 中定义了一个基本的`Context`。 一个`上下文`
是一种不可变的数据结构，它在 API 和进程之间承载请求范围的数据。 上下文
还旨在启用并发并在 goroutines 中使用。

上下文旨在**不可变**； 他们永远不应该被编辑。 相反，约定是
使用“With”函数从其父级创建子上下文。 例如: 

```go
childCtx = parentCtx.WithBlockHeader(header)
```

[Golang Context Package](https://golang.org/pkg/context) 文档指导开发者
显式传递上下文 `ctx` 作为进程的第一个参数。

## 存储分支

`Context` 包含一个 `MultiStore`，它允许使用 `CacheMultiStore` 进行分支和缓存功能
(`CacheMultiStore` 中的查询被缓存以避免将来的往返)。
每个“KVStore”都分支在一个安全且隔离的临时存储中。进程可以自由地将更改写入
`CacheMultiStore`。如果状态转换序列没有问题地执行，存储分支可以
提交到序列末尾的底层存储，或者在出现某些情况时忽略它们
出错。 Context 的使用模式如下:

1. 一个进程从它的父进程接收一个上下文`ctx`，它提供了需要的信息
   执行过程。
2. `ctx.ms` 是一个**分支存储**，即[multistore](./store.md#multistore) 的一个分支，以便进程在执行时可以对状态进行更改，不改变原来的`ctx.ms`。这对于保护底层 multistore 很有用，以防在执行过程中的某个时刻需要恢复更改。
3. 进程在执行时可能会从 `ctx` 读取和写入。它可能会调用一个子进程并通过
   `ctx` 根据需要添加到它。
4. 当子进程返回时，它检查结果是成功还是失败。如果失败，什么都没有
   需要完成 - 分支 `ctx` 被简单地丢弃。如果成功，所做的更改
   `CacheMultiStore` 可以通过 `Write()` 提交给原始的 `ctx.ms`。

例如，这里是 [`runTx`](./baseapp.md#runtx-and-runmsgs) 函数的一个片段
[`baseapp`](./baseapp.md): 

```go
runMsgCtx, msCache := app.cacheTxContext(ctx, txBytes)
result = app.runMsgs(runMsgCtx, msgs, mode)
result.GasWanted = gasWanted

if mode != runTxModeDeliver {
  return result
}

if result.IsOK() {
  msCache.Write()
}
```

这是过程:

1. 在对事务中的消息调用 `runMsgs` 之前，它使用 `app.cacheTxContext()`
    分支和缓存上下文和多存储。
2. `runMsgCtx` - 带有分支存储的上下文，在 `runMsgs` 中用于返回结果。
3.如果进程运行在[`checkTxMode`](./baseapp.md#checktx)，则不需要写
    更改 - 结果立即返回。
4.如果进程在[`deliverTxMode`](./baseapp.md#delivertx)下运行，结果显示
    成功运行所有消息后，分支多存储被写回原始。

## 下一个 {hide}

了解 [节点客户端](./node.md) {hide} 