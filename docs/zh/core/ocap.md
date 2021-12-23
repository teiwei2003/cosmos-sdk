# Object-Capability Model

## Intro

在考虑安全性时，最好从特定的威胁模型开始。我们的威胁模型如下:

> 我们假设一个繁荣的 Cosmos SDK 模块生态系统很容易组合成区块链应用程序，将包含有缺陷或恶意的模块。

Cosmos SDK 旨在通过成为
对象能力系统的基础。

> 对象能力系统的结构特性有利于
> 代码设计的模块化并确保在
> 代码实现。
>
> 这些结构特性有助于分析一些
> 对象能力程序或操作的安全属性
> 系统。其中一些——特别是信息流属性
> — 可以在对象引用和
> 连通性，独立于代码的任何知识或分析
> 决定对象的行为。
>
> 因此，可以建立这些安全属性
> 并在存在包含未知的新对象的情况下进行维护
> 和可能的恶意代码。
>
> 这些结构特性源于两个管理规则
> 访问现有对象:
>
> 1. 只有当对象 A 持有一个对象 A 才能向 B 发送消息
> 参考 B。
> 2.对象A只能获得对C的引用
> 如果对象 A 收到一条包含对 C 的引用的消息。作为
> 这两条规则的结果，一个对象可以获得一个引用
> 只能通过预先存在的引用链到另一个对象。
> 简而言之，“只有连通性才会产生连通性。”

有关对象功能的介绍，请参阅此 [维基百科文章](https://en.wikipedia.org/wiki/Object-capability_model)。

## 实践中的 Ocaps

这个想法是只揭示完成工作所必需的东西。

例如，以下代码片段违反了对象功能
原则:

```go
type AppAccount struct {...}
account := &AppAccount{
    Address: pub.Address(),
    Coins: sdk.Coins{sdk.NewInt64Coin("ATM", 100)},
}
sumValue := externalModule.ComputeSumValue(account)
```

`ComputeSumValue` 方法隐含了一个纯函数，但隐含的
接受指针值的能力是修改该值的能力
价值。首选方法签名应改为副本。 

```go
sumValue := externalModule.ComputeSumValue(*account)
```

在 Cosmos SDK 中，可以看到这个原理的应用
盖亚应用程序。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.41.4/simapp/app.go#L249-L273

下图显示了 Keeper 之间的当前依赖关系。

![Keeper 依赖项](../uml/svg/keeper_dependencies.svg)

## 下一个 {hide}

了解 [`runTx` 中间件](./runtx_middleware.md) {hide} 