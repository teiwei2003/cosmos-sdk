<!--
order: 6
-->

# BeginBlocker 和 EndBlocker

`BeginBlocker` 和 `EndBlocker` 是模块开发人员可以在他们的模块中实现的可选方法。当 [`BeginBlock`](../core/baseapp.md#beginblock) 和 [`EndBlock`](../core/baseapp.md #endblock) 从底层共识引擎接收 ABCI 消息。 {概要}

## 先决条件阅读

- [模块管理器](./module-manager.md) {prereq}

## BeginBlocker 和 EndBlocker

`BeginBlocker` 和 `EndBlocker` 是模块开发人员向其模块添加逻辑自动执行的一种方式。这是一个应该谨慎使用的强大工具，因为复杂的自动功能可能会减慢甚至停止链条。

需要时，`BeginBlocker` 和`EndBlocker` 作为[`AppModule` 接口](./module-manager.md#appmodule) 的一部分实现。 `module.go`中实现的接口的`BeginBlock`和`EndBlock`方法一般分别遵循`BeginBlocker`和`EndBlocker`方法，通常在`abci.go`中实现。

`abci.go` 中 `BeginBlocker` 和 `EndBlocker` 的实际实现与 [`Msg` 服务](./msg-services.md) 的实现非常相似:

- 他们通常使用 [`keeper`](./keeper.md) 和 [`ctx`](../core/context.md) 来检索有关最新状态的信息。
- 如果需要，他们使用 `keeper` 和 `ctx` 来触发状态转换。
- 如果需要，他们可以通过 `ctx` 的 `EventManager` 发出 [`events`](../core/events.md)。

`EndBlocker` 的一个特殊性是它可以以 [`[]abci.ValidatorUpdates`](https://tendermint.com/docs/app-dev/abci- spec.html#validatorupdate)。这是实现自定义验证器更改的首选方式。

开发人员可以通过模块的管理器“SetOrderBeginBlocker”/“SetOrderEndBlocker”方法定义每个应用程序模块的“BeginBlocker”/“EndBlocker”函数之间的执行顺序。有关模块管理器的更多信息，请单击 [此处](./module-manager.md#manager)。

从 `disr` 模块查看 `BeginBlocker` 的示例实现:

+++ https://github.com/cosmos/cosmos-sdk/blob/f33749263f4ecc796115ad6e789cb0f7cddf9148/x/distribution/abci.go#L14-L38

以及来自 `staking` 模块的 `EndBlocker` 的示例实现:

+++ https://github.com/cosmos/cosmos-sdk/blob/f33749263f4ecc796115ad6e789cb0f7cddf9148/x/staking/abci.go#L22-L27

## 下一个{hide}

了解 [`keeper`s](./keeper.md) {hide} 