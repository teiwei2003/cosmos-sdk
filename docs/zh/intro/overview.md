# 高级概述

## 什么是 Cosmos SDK？

[Cosmos SDK](https://github.com/cosmos/cosmos-sdk) 是一个开源框架，用于构建多资产公共权益证明(PoS)<df value="blockchain">区块链< /df>，如 Cosmos Hub，以及许可的权威证明 (PoA) 区块链。使用 Cosmos SDK 构建的区块链通常称为 **特定于应用程序的区块链**。

Cosmos SDK 的目标是让开发人员能够轻松地从头开始创建可以与其他区块链进行本地互操作的自定义区块链。我们将 Cosmos SDK 设想为类似 npm 的框架，以在 [Tendermint](https://github.com/tendermint/tendermint) 之上构建安全的区块链应用程序。基于 SDK 的区块链由可组合的 [模块](../building-modules/intro.md) 构建而成，其中大部分是开源的，可供任何开发人员使用。任何人都可以为 Cosmos SDK 创建一个模块，集成已经构建的模块就像将它们导入区块链应用程序一样简单。更重要的是，Cosmos SDK 是一个基于能力的系统，它允许开发人员更好地推理模块之间交互的安全性。要更深入地了解功能，请跳转到 [Object-Capability Model](../core/ocap.md)。

## 什么是特定于应用程序的区块链？

当今区块链世界的一个开发范式是像以太坊这样的虚拟机区块链，其中开发通常围绕在现有区块链之上构建分散式应用程序作为一组智能合约。虽然智能合约对于某些用例非常有用，例如一次性应用程序(例如 ICO)，但它们通常无法构建复杂的去中心化平台。更一般地说，智能合约在灵活性、主权和性能方面可能会受到限制。

特定于应用程序的区块链提供了与虚拟机区块链完全不同的开发范式。特定于应用程序的区块链是为操作单个应用程序而定制的区块链:开发人员可以完全自由地做出应用程序以最佳方式运行所需的设计决策。它们还可以提供更好的主权、安全和性能。

了解有关 [特定于应用程序的区块链](./why-app-specific.md) 的更多信息。

## 为什么是 Cosmos SDK？

Cosmos SDK 是当今构建自定义应用程序特定区块链的最先进框架。以下是您可能要考虑使用 Cosmos SDK 构建分散式应用程序的几个原因:

- Cosmos SDK 中可用的默认共识引擎是 [Tendermint Core](https://github.com/tendermint/tendermint)。 Tendermint 是现存最(也是唯一)成熟的 BFT 共识引擎。它在整个行业中被广泛使用，被认为是构建权益证明系统的黄金标准共识引擎。
- Cosmos SDK 是开源的，旨在使从可组合的 [模块](../../x/) 构建区块链变得容易。随着开源 Cosmos SDK 模块生态系统的发展，使用它构建复杂的去中心化平台将变得越来越容易。
- Cosmos SDK 的灵感来自基于功能的安全性，并从多年与区块链状态机的角力中汲取灵感。这使得 Cosmos SDK 成为构建区块链的非常安全的环境。
- 最重要的是，Cosmos SDK 已经被用于构建许多已经投入生产的特定于应用程序的区块链。其中，我们可以引用[Cosmos Hub](https://hub.cosmos.network)、[IRIS Hub](https://irisnet.org)、[Binance Chain](https://docs.binance.org) /)、[Terra](https://terra.money/) 或 [Kava](https://www.kava.io/)。 [更多](https://cosmos.network/ecosystem) 是基于 Cosmos SDK 构建的。

## Cosmos SDK 入门

- 详细了解 [Cosmos SDK 应用程序的架构](./sdk-app-architecture.md)
- 通过 [Cosmos SDK 教程](https://cosmos.network/docs/tutorial) 了解如何从头开始构建特定于应用程序的区块链

## 下一个 {hide}

了解 [特定于应用程序的区块链](./why-app-specific.md) {hide} 