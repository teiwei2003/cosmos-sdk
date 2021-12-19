# `质押`

## 摘要

本文指定了 Cosmos-SDK 的 Staking 模块。
[Cosmos 白皮书](https://cosmos.network/about/whitepaper) 中描述
2016 年 6 月。

该模块使基于 Cosmos-SDK 的区块链能够支持高级
权益证明系统。在这个系统中，原生质押代币的持有者
链可以成为验证者，并且可以将代币委托给验证者，
最终确定系统的有效验证器集。

该模块将用于 Cosmos Hub，Cosmos 中的第一个 Hub
网络。

## 内容

1. **[状态](01_state.md)**
    - [池](01_state.md#pool)
    - [LastTotalPower](01_state.md#lasttotalpower)
    - [参数](01_state.md#params)
    - [验证器](01_state.md#validator)
    - [委托](01_state.md#delegation)
    - [UnbondingDelegation](01_state.md#unbondingdelegation)
    - [重新委托](01_state.md#redelegation)
    - [队列](01_state.md#queues)
    - [历史信息](01_state.md#historicalinfo)
2. **[状态转换](02_state_transitions.md)**
    - [验证器](02_state_transitions.md#validators)
    - [委托](02_state_transitions.md#delegations)
    - [斜线](02_state_transitions.md#slashing)
3. **[消息](03_messages.md)**
    - [MsgCreateValidator](03_messages.md#msgcreatevalidator)
    - [MsgEditValidator](03_messages.md#msgeditvalidator)
    - [MsgDelegate](03_messages.md#msgdelegate)
    - [MsgUndelegate](03_messages.md#msgundelegate)
    - [MsgBeginRedelegate](03_messages.md#msgbeginredelegate)
4. **[开始区块](04_begin_block.md)**
    - [历史信息追踪](04_begin_block.md#historical-info-tracking)
5. **[结束块](05_end_block.md)**
    - [验证器设置更改](05_end_block.md#validator-set-changes)
    - [队列](05_end_block.md#queues-)
6. **[钩子](06_hooks.md)**
7. **[事件](07_events.md)**
    - [EndBlocker](07_events.md#endblocker)
    - [Msg's](07_events.md#msg's)
8. **[参数](08_params.md)** 