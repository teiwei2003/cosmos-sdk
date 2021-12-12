<!--
order: 1
-->

# Cosmos SDK 模块介绍

模块定义了 Cosmos SDK 应用程序的大部分逻辑。开发人员使用 Cosmos SDK 将模块组合在一起，以构建他们自定义的特定于应用程序的区块链。本文档概述了 SDK 模块背后的基本概念以及如何进行模块管理。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}
- [Cosmos SDK 交易的生命周期](../basics/tx-lifecycle.md) {prereq}

## 模块在 Cosmos SDK 应用程序中的作用

Cosmos SDK 可以被认为是区块链开发的 Ruby-on-Rails。它带有一个核心，可提供每个区块链应用程序所需的基本功能，例如 [ABCI 的样板实现](../core/baseapp.md) 与底层共识引擎进行通信，[`multistore`](. ./core/store.md#multistore) 来持久化状态，一个 [server](../core/node.md) 来形成一个全节点和一个 [interfaces](./module-interfaces.md) 来处理查询.

在此核心之上，Cosmos SDK 使开发人员能够构建实现其应用程序业务逻辑的模块。换句话说，SDK 模块实现了应用程序的大部分逻辑，而核心负责布线并使模块能够组合在一起。最终目标是构建一个强大的开源 Cosmos SDK 模块生态系统，使构建复杂的区块链应用程序变得越来越容易。

Cosmos SDK 模块可以看作是状态机中的小状态机。他们通常使用 [main multistore](../core/store.md) 中的一个或多个 `KVStore` 来定义状态的子集，以及 [消息类型](./messages-and-查询.md#messages)。这些消息由 Cosmos SDK 核心的主要组件之一 [`BaseApp`](../core/baseapp.md) 路由到模块 Protobuf [`Msg` 服务](./msg-services.md)这定义了它们。 

```
                                      +
                                      |
                                      |  Transaction relayed from the full-node's consensus engine
                                      |  to the node's application via DeliverTx
                                      |
                                      |
                                      |
                +---------------------v--------------------------+
                |                 APPLICATION                    |
                |                                                |
                |     Using baseapp's methods: Decode the Tx,    |
                |     extract and route the message(s)           |
                |                                                |
                +---------------------+--------------------------+
                                      |
                                      |
                                      |
                                      +---------------------------+
                                                                  |
                                                                  |
                                                                  |
                                                                  |  Message routed to the correct
                                                                  |  module to be processed
                                                                  |
                                                                  |
+----------------+  +---------------+  +----------------+  +------v----------+
|                |  |               |  |                |  |                 |
|  AUTH MODULE   |  |  BANK MODULE  |  | STAKING MODULE |  |   GOV MODULE    |
|                |  |               |  |                |  |                 |
|                |  |               |  |                |  | Handles message,|
|                |  |               |  |                |  | Updates state   |
|                |  |               |  |                |  |                 |
+----------------+  +---------------+  +----------------+  +------+----------+
                                                                  |
                                                                  |
                                                                  |
                                                                  |
                                       +--------------------------+
                                       |
                                       | Return result to the underlying consensus engine (e.g. Tendermint)
                                       | (0=Ok, 1=Err)
                                       v
```

由于这种架构，构建 Cosmos SDK 应用程序通常围绕编写模块来实现应用程序的专用逻辑，并将它们与现有模块组合以完成应用程序。开发人员通常会处理实现其特定用例所需逻辑但尚不存在的模块，并将使用现有模块实现更通用的功能，如抵押、账户或令牌管理。

## 作为开发人员如何处理构建模块

虽然编写模块没有明确的指导方针，但开发人员在构建模块时应牢记以下一些重要的设计原则:

- **可组合性**:Cosmos SDK 应用程序几乎总是由多个模块组成。这意味着开发人员不仅需要仔细考虑他们的模块与 Cosmos SDK 核心的集成，还需要仔细考虑与其他模块的集成。前者是通过遵循 [here](#main-components-of-sdk-modules) 概述的标准设计模式来实现的，而后者是通过 [`keeper`]( ./keeper.md)。
- **专业化**:**可组合性**特性的一个直接后果是模块应该是**专业化的**。开发人员应仔细确定其模块的范围，而不是将多个功能批量合并到同一个模块中。这种关注点分离使模块能够在其他项目中重用，并提高应用程序的可升级性。 **专业化**在 Cosmos SDK 的 [对象-能力模型](../core/ocap.md) 中也扮演着重要的角色。
- **功能**:大多数模块需要读取和/或写入其他模块的存储。但是，在开源环境中，某些模块可能是恶意的。这就是为什么模块开发人员不仅需要仔细考虑他们的模块如何与其他模块交互，还要考虑如何访问模块的存储。 Cosmos SDK 采用面向功能的方法来实现模块间安全。这意味着模块定义的每个 store 都由一个 `key` 访问，该 `key` 由模块的 [`keeper`](./keeper.md) 持有。这个 `keeper` 定义了如何访问商店以及在什么条件下。对模块存储的访问是通过传递对模块的`keeper` 的引用来完成的。

## Cosmos SDK 模块的主要组件

按照惯例，模块定义在`./x/` 子文件夹中(例如，`bank` 模块将定义在`./x/bank` 文件夹中)。它们通常共享相同的核心组件:

- [`keeper`](./keeper.md)，用于访问模块的存储并更新状态。
- 一个[`Msg`服务](./messages-and-queries.md#messages)，当消息被[`BaseApp`](../core/baseapp.md#message)路由到模块时，用于处理消息-routing)并触发状态转换。
- [查询服务](./query-services.md)，当用户查询被[`BaseApp`](../core/baseapp.md#query-routing)路由到模块时，用于处理用户查询。
- 接口，供最终用户查询模块定义的状态子集并创建模块中定义的自定义类型的“消息”。

除了这些组件之外，模块还实现了`AppModule` 接口，以便由[`module manager`](./module-manager.md) 进行管理。

请参阅[结构文档](./structure.md) 了解推荐的模块目录结构。

## 下一个{hide}

阅读更多关于 [`AppModule` 接口和 `module manager`](./module-manager.md) {hide} 