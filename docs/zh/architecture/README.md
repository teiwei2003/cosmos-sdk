# 架构决策记录 (ADR)

这是在 Cosmos-SDK 中记录所有高级架构决策的位置。

架构决策 (**AD**) 是一种软件设计选择，用于解决在架构上具有重要意义的功能性或非功能性需求。
架构上重要的需求 (**ASR**) 是对软件系统的架构和质量具有可衡量影响的需求。
架构决策记录 (**ADR**) 捕获单个 AD，例如在撰写个人笔记或会议记录时经常做的；在项目中创建和维护的 ADR 集合构成了其决策日志。所有这些都在建筑知识管理 (AKM) 的主题之内。

您可以在此 [博客文章](https://product.reverb.com/documenting-architecture-decisions-the-reverb-way-a3563bb24bd0#.78xhdix6t) 中阅读有关 ADR 概念的更多信息。

## 基本原理

ADR 旨在成为提出新功能设计和新流程、收集社区对某个问题的意见以及记录设计决策的主要机制。
ADR 应提供：

- 相关目标和当前状态的背景
- 为实现目标而提出的变更
- 利弊总结
- 参考
- 变更日志

请注意 ADR 和规范之间的区别。 ADR 提供上下文、直觉、推理和
改变架构的理由，或某物的架构
新的。该规范对所有内容进行了更加压缩和简化的总结，因为
它今天站立。

如果发现缺少记录的决策，请召集讨论，在此处记录新决策，然后修改代码以匹配。

## 创建新的 ADR

阅读有关 [PROCESS](./PROCESS.md) 的信息。

#### 使用 RFC 2119 关键字

编写 ADR 时，请遵循编写 RFC 的相同最佳实践。在编写 RFC 时，关键字用于表示规范中的要求。这些词通常大写：“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”。它们将按照 [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) 中的描述进行解释。

## ADR 目录

### 接受

- [ADR 002：SDK 文档结构](./adr-002-docs-structure.md)
- [ADR 004：拆分面额密钥](./adr-004-split-denomination-keys.md)
- [ADR 006：秘密商店更换](./adr-006-secret-store-replacement.md)
- [ADR 009：证据模块](./adr-009-evidence-module.md)
- [ADR 010: 模块化 AnteHandler](./adr-010-modular-antehandler.md)
- [ADR 019：协议缓冲区状态编码](./adr-019-protobuf-state-encoding.md)
- [ADR 020：协议缓冲区事务编码](./adr-020-protobuf-transaction-encoding.md)
- [ADR 021: 协议缓冲区查询编码](./adr-021-protobuf-query-encoding.md)
- [ADR 023：协议缓冲区命名和版本控制](./adr-023-protobuf-naming.md)
- [ADR 029：费用补助模块](./adr-029-fee-grant-module.md)
- [ADR 030：消息授权模块](./adr-030-authz-module.md)
- [ADR 031：Protobuf 消息服务](./adr-031-msg-service.md)

### 建议的

- [ADR 003：动态能力存储](./adr-003-dynamic-capability-store.md)
- [ADR 011：推广创世账户](./adr-011-generalize-genesis-accounts.md)
- [ADR 012：状态访问器](./adr-012-state-accessors.md)
- [ADR 013：指标](./adr-013-metrics.md)
- [ADR 016：验证器共识密钥轮换](./adr-016-validator-consensus-key-rotation.md)
- [ADR 017：历史头模块](./adr-017-historical-header-module.md)
- [ADR 018：可延长投票期](./adr-018-extendable-voting-period.md)
- [ADR 022: 自定义 baseapp 恐慌处理](./adr-022-custom-panic-handling.md)
- [ADR 024：硬币元数据](./adr-024-coin-metadata.md)
- [ADR 027：确定性 Protobuf 序列化](./adr-027-deterministic-protobuf-serialization.md)
- [ADR 028：公钥地址](./adr-028-public-key-addresses.md)
- [ADR 032：类型化事件](./adr-032-typed-events.md)
- [ADR 033: 模块间 RPC](./adr-033-protobuf-inter-module-comm.md)
- [ADR 035：Rosetta API 支持](./adr-035-rosetta-api-support.md)
- [ADR 037：治理分裂投票](./adr-037-gov-split-vote.md)
- [ADR 038：状态监听](./adr-038-state-listening.md)
- [ADR 039: Epoched Staking](./adr-039-epoched-staking.md)
- [ADR 040：存储和 SMT 状态承诺](./adr-040-storage-and-smt-state-commitments.md)
- [ADR 046：模块参数](./adr-046-module-params.md)

### 草稿

- [ADR 044：Protobuf 定义更新指南](./adr-044-protobuf-updates-guidelines.md) 