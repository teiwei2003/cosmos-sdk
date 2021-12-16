# `x/斜线`

## 摘要

本节指定Cosmos SDK的slashing模块，实现功能
2016 年 6 月在 [Cosmos 白皮书](https://cosmos.network/about/whitepaper) 中首次概述。

slashing 模块使基于 Cosmos SDK 的区块链能够抑制任何可归因的行为
由协议认可的行为者通过惩罚他们(“削减”)而将价值置于危险之中。

处罚可能包括但不限于:

- 烧掉他们的一些股份
- 取消他们在一段时间内对未来区块进行投票的能力。

该模块将由 Cosmos Hub(Cosmos 生态系统中的第一个枢纽)使用。

## 内容

1. **[概念](01_concepts.md)**
   - [状态](01_concepts.md#states)
   - [墓碑帽](01_concepts.md#tombstone-caps)
   - [ASCII 时间线](01_concepts.md#ascii-timelines)
2. **[状态](02_state.md)**
   - [签名信息](02_state.md#signing-info)
3. **[消息](03_messages.md)**
   - [出狱](03_messages.md#unjail)
4. **[开始区块](04_begin_block.md)**
   - [证据处理](04_begin_block.md#evidence-handling)
   - [正常运行时间跟踪](04_begin_block.md#uptime-tracking)
5. **[05_hooks.md](05_hooks.md)**
   - [钩子](05_hooks.md#hooks)
6. **[事件](06_events.md)**
   - [BeginBlocker](06_events.md#beginblocker)
   - [处理程序](06_events.md#handlers)
7. **[Staking Tombstone](07_tombstone.md)**
   - [摘要](07_tombstone.md#abstract)
8. **[参数](08_params.md)**
9. **[客户端](09_client.md)**
    - [CLI](09_client.md#cli)
    - [gRPC](09_client.md#grpc)
    - [REST](09_client.md#rest) 