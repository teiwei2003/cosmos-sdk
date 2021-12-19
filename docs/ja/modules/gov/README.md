# `政府`

## 摘要

本文指定了 Cosmos-SDK 的 Governance 模块，它是第一个
[Cosmos 白皮书](https://cosmos.network/about/whitepaper) 中的描述
2016 年 6 月。

该模块使基于 Cosmos-SDK 的区块链能够支持链上治理
系统。在这个系统中，链上原生质押代币的持有者可以投票
在 1 代币 1 票基础上的提案。接下来是模块的功能列表
目前支持:

- **提案提交:** 用户可以通过存款提交提案。一旦
达到最低存款，提案进入投票期
- **投票:** 参与者可以对达到 MinDeposit 的提案进行投票
- **继承和惩罚:** 委托人继承验证人的投票，如果
他们不投票给自己。
- **申领押金:** 对提案进行了押金的用户可以收回他们的
如果提案被接受或提案从未进入投票期。

该模块将用于 Cosmos Hub，Cosmos 网络中的第一个 Hub。
[未来改进](05_future_improvements.md) 中描述了将来可能添加的功能。

## 内容

以下规范使用 *ATOM* 作为本机 staking 代币。模块
通过将 *ATOM* 替换为本机，可以适应任何权益证明区块链
链的 staking 代币。

1. **[概念](01_concepts.md)**
    - [提案提交](01_concepts.md#proposal-submission)
    - [投票](01_concepts.md#vote)
    - [软件升级](01_concepts.md#software-upgrade)
2. **[状态](02_state.md)**
    - [参数和基本类型](02_state.md#parameters-and-base-types)
    - [存款](02_state.md#deposit)
    - [ValidatorGovInfo](02_state.md#validatorgovinfo)
    - [提案](02_state.md#proposals)
    - [存储](02_state.md#stores)
    - [提案处理队列](02_state.md#proposal-processing-queue)
3. **[消息](03_messages.md)**
    - [提案提交](03_messages.md#proposal-submission)
    - [存款](03_messages.md#deposit)
    - [投票](03_messages.md#vote)
4. **[事件](04_events.md)**
    - [EndBlocker](04_events.md#endblocker)
    - [处理程序](04_events.md#handlers)
5. **[未来改进](05_future_improvements.md)**
6. **[参数](06_params.md)**
7. **[客户端](07_client.md)**
    - [CLI](07_client.md#cli)
    - [gRPC](07_client.md#grpc)
    - [REST](07_client.md#rest) 