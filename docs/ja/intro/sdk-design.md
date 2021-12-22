# Cosmos SDK 的主要组成部分

Cosmos SDK 是一个框架，可促进在 Tendermint 之上开发安全状态机。 Cosmos SDK 的核心是 Golang 中 [ABCI](./sdk-app-architecture.md#abci) 的样板实现。它带有一个 [`multistore`](../core/store.md#multistore) 来持久化数据和一个 [`router`](../core/baseapp.md#routing) 来处理事务。

下面是一个简化的视图，说明在通过“DeliverTx”从 Tendermint 传输时，构建在 Cosmos SDK 之上的应用程序如何处理事务:

1. 解码从 Tendermint 共识引擎接收到的 `transactions`(记住 Tendermint 只处理 `[]bytes`)。
2. 从 `transactions` 中提取 `messages` 并进行基本的完整性检查。
3. 将每条消息路由到适当的模块，以便对其进行处理。
4. 提交状态更改。

## `基础应用`

`baseapp` 是 Cosmos SDK 应用程序的样板实现。它带有 ABCI 的实现，用于处理与底层共识引擎的连接。通常，Cosmos SDK 应用程序通过将“baseapp”嵌入到 [`app.go`](../basics/app-anatomy.md#core-application-file) 中来扩展它。请参阅 Cosmos SDK 应用程序教程中的示例:

+++ https://github.com/cosmos/sdk-tutorials/blob/c6754a1e313eb1ed973c5c91dcc606f2fd288811/app.go#L72-L92

`baseapp` 的目标是在存储和可扩展状态机之间提供一个安全的接口，同时尽可能少地定义状态机(忠于 ABCI)。

有关`baseapp`的更多信息，请单击[此处](../core/baseapp.md)。

## 多存储

Cosmos SDK 提供了一个 [`multistore`](../core/store.md#multistore) 用于持久化状态。 multistore 允许开发者声明任意数量的 [`KVStores`](../core/store.md#base-layer-kvstores)。这些 `KVStores` 只接受 `[]byte` 类型作为值，因此任何自定义结构都需要在存储之前使用 [a codec](../core/encoding.md) 进行编组。

多存储抽象用于将状态划分为不同的隔间，每个隔间由自己的模块管理。有关 multistore 的更多信息，请单击 [此处](../core/store.md#multistore)

## 模块

Cosmos SDK 的强大之处在于其模块化。 Cosmos SDK 应用程序是通过聚合一组可互操作的模块来构建的。每个模块定义状态的一个子集并包含自己的消息/事务处理器，而 Cosmos SDK 负责将每条消息路由到其各自的模块。

下面是一个简化的视图，当它在有效块中接收时，每个全节点的应用程序如何处理交易: 

```
                                      +
                                      |
                                      |  Transaction relayed from the full-node's
                                      |  Tendermint engine to the node's application
                                      |  via DeliverTx
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
                                                                  |  Message routed to
                                                                  |  the correct module
                                                                  |  to be processed
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
                                       | Return result to Tendermint
                                       | (0=Ok, 1=Err)
                                       v
```

每个模块都可以看作是一个小的状态机。开发者需要定义模块处理的状态的子集，以及修改状态的自定义消息类型(*注:*`messages`是`baseapp`从`transactions`中提取的)。一般来说，每个模块在 `multistore` 中声明自己的 `KVStore` 来持久化它定义的状态的子集。大多数开发人员在构建自己的模块时需要访问其他 3rd 方模块。由于 Cosmos SDK 是一个开放的框架，部分模块可能是恶意的，这意味着需要安全原则来推理模块间的交互。这些原则基于 [object-capabilities](../core/ocap.md)。实际上，这意味着不是让每个模块为其他模块保留访问控制列表，而是每个模块实现称为“keeper”的特殊对象，可以将其传递给其他模块以授予一组预定义的功能。

Cosmos SDK 模块定义在 Cosmos SDK 的 `x/` 文件夹中。一些核心模块包括:

- `x/auth`:用于管理账户和签名。
- `x/bank`:用于启用代币和代币转账。
- `x/staking` + `x/slashing`:用于构建权益证明区块链。

除了 `x/` 中已经存在的模块，任何人都可以在他们的应用程序中使用，Cosmos SDK 允许您构建自己的自定义模块。您可以查看 [教程中的示例](https://tutorials.cosmos.network/)。

## 下一个 {hide}

详细了解 [Cosmos SDK 应用程序剖析](../basics/app-anatomy.md) {hide} 