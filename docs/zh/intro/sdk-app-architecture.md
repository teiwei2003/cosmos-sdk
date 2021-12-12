<!--
order: 3
-->

# 区块链架构

## 状态机

区块链的核心是一个[复制的确定性状态机](https://en.wikipedia.org/wiki/State_machine_replication)。

状态机是一种计算机科学概念，其中一台机器可以有多个状态，但在任何给定时间只能有一个状态。 有一个“状态”，它描述系统的当前状态，还有“事务”，它触发状态转换。

给定一个状态 S 和一个事务 T，状态机将返回一个新的状态 S'。 

```
+--------+                 +--------+
|        |                 |        |
|   S    +---------------->+   S'   |
|        |    apply(T)     |        |
+--------+                 +--------+
```

在实践中，交易被捆绑在块中，以提高流程效率。 给定一个状态 S 和一个交易块 B，状态机将返回一个新的状态 S'。 

```
+--------+                              +--------+
|        |                              |        |
|   S    +----------------------------> |   S'   |
|        |   For each T in B: apply(T)  |        |
+--------+                              +--------+
```

在区块链上下文中，状态机是确定性的。 这意味着如果一个节点在给定状态下启动并重放相同的事务序列，它将始终以相同的最终状态结束。

Cosmos SDK 为开发人员提供了最大的灵活性来定义其应用程序的状态、事务类型和状态转换函数。 使用 Cosmos SDK 构建状态机的过程将在以下部分进行更深入的描述。 但首先，让我们看看如何使用 **Tendermint** 复制状态机。

## Tendermint

多亏了 Cosmos SDK，开发人员只需定义状态机，[*Tendermint*](https://tendermint.com/docs/introduction/what-is-tendermint.html) 将为他们处理网络上的复制 . 

```
                ^  +-------------------------------+  ^
                |  |                               |  |   Built with Cosmos SDK
                |  |  State-machine = Application  |  |
                |  |                               |  v
                |  +-------------------------------+
                |  |                               |  ^
Blockchain node |  |           Consensus           |  |
                |  |                               |  |
                |  +-------------------------------+  |   Tendermint Core
                |  |                               |  |
                |  |           Networking          |  |
                |  |                               |  |
                v  +-------------------------------+  v
```

[Tendermint](https://docs.tendermint.com/v0.34/introduction/what-is-tendermint.html) 是一个与应用程序无关的引擎，负责处理区块链。实际上，这意味着 Tendermint 负责传播和排序交易字节。 Tendermint Core 依靠同名的拜占庭容错 (BFT) 算法来就交易顺序达成共识。

Tendermint [共识算法](https://docs.tendermint.com/v0.34/introduction/what-is-tendermint.html#consensus-overview) 与一组称为 *Validators* 的特殊节点一起工作。验证者负责向区块链添加交易块。在任何给定的块上，都有一个验证器集 V。 V 中的一个验证器由算法选择作为下一个块的提议者。如果超过三分之二的 V 签署了 *[prevote](https://docs.tendermint.com/v0.34/spec/consensus/consensus.html#prevote-step-height-h-round -r)* 和一个 *[precommit](https://docs.tendermint.com/v0.34/spec/consensus/consensus.html#precommit-step-height-h-round-r)*，以及如果它包含的所有交易都是有效的。验证器集可以通过写入状态机的规则进行更改。

## ABCI

Tendermint 通过名为 [ABCI](https://docs.tendermint.com/v0.34/spec/abci/) 的接口将交易传递给应用程序，应用程序必须实现该接口。 

```
              +---------------------+
              |                     |
              |     Application     |
              |                     |
              +--------+---+--------+
                       ^   |
                       |   | ABCI
                       |   v
              +--------+---+--------+
              |                     |
              |                     |
              |     Tendermint      |
              |                     |
              |                     |
              +---------------------+
```

请注意，**Tendermint 仅处理交易字节**。它不知道这些字节的含义。 Tendermint 所做的就是确定性地对这些交易字节进行排序。 Tendermint 通过 ABCI 将字节传递给应用程序，并期望返回代码来通知它包含在交易中的消息是否已成功处理。

以下是 ABCI 最重要的信息:

- `CheckTx`:当 Tendermint Core 收到交易时，会将其传递给应用程序以检查是否满足一些基本要求。 `CheckTx` 用于保护全节点的内存池免受垃圾邮件交易的影响。一个名为 [`AnteHandler`](../basics/gas-fees.md#antehandler) 的特殊处理程序用于执行一系列验证步骤，例如检查是否有足够的费用和验证签名。如果检查有效，则将交易添加到 [mempool](https://docs.tendermint.com/v0.34/tendermint-core/mempool.html#mempool) 并中继到对等节点。请注意，“CheckTx”不会处理交易(即不会发生状态修改)，因为它们尚未包含在区块中。
- `DeliverTx`:当 Tendermint Core 收到一个[有效区块](https://docs.tendermint.com/v0.34/spec/blockchain/blockchain.html#validation) 时，区块中的每笔交易都会被传递给通过“DeliverTx”申请以进行处理。正是在这个阶段发生了状态转换。 `AnteHandler` 与实际的 [`Msg` 服务](../building-modules/msg-services.md) RPC 一起为事务中的每条消息再次执行。
- `BeginBlock`/`EndBlock`:这些消息在每个块的开始和结束时执行，无论该块是否包含交易。触发逻辑的自动执行很有用。但是请谨慎进行，因为计算成本高的循环可能会减慢您的区块链，如果循环是无限的，甚至会冻结它。

从 [Tendermint 文档](https://docs.tendermint.com/v0.34/spec/abci/abci.html#overview) 中找到有关 ABCI 方法的更详细视图。

任何基于 Tendermint 构建的应用程序都需要实现 ABCI 接口，以便与底层的本地 Tendermint 引擎进行通信。幸运的是，您不必实现 ABCI 接口。 Cosmos SDK 以 [baseapp](./sdk-design.md#baseapp) 的形式提供了它的样板实现。

## 下一个{hide}

阅读 [Cosmos SDK 的高级设计原则](./sdk-design.md) {hide} 