# `认证`

## 摘要

本文档指定了 Cosmos SDK 的 auth 模块。

auth 模块负责指定基础交易和账户类型
对于应用程序，因为 SDK 本身与这些细节无关。它包含
ante 处理程序，其中所有基本交易有效性检查(签名、随机数、辅助字段)
执行，并暴露帐户管理器，允许其他模块读取、写入和修改帐户。

该模块用于 Cosmos Hub。

## 内容

1. **[概念](01_concepts.md)**
   - [Gas & Fees](01_concepts.md#gas-&-fees)
2. **[状态](02_state.md)**
   - [账户](02_state.md#accounts)
3. **[AnteHandlers](03_antehandlers.md)**
   - [处理程序](03_antehandlers.md#handlers)
4. **[Keepers](04_keepers.md)**
   - [账户管理员](04_keepers.md#account-keeper)
5. **[归属](05_vesting.md)**
   - [介绍和要求](05_vesting.md#intro-and-requirements)
   - [归属账户类型](05_vesting.md#vesting-account-types)
   - [归属账户规范](05_vesting.md#vesting-account-specification)
   - [Keepers & Handlers](05_vesting.md#keepers-&-handlers)
   - [创世初始化](05_vesting.md#genesis-initialization)
   - [例子](05_vesting.md#examples)
   - [词汇](05_vesting.md#glossary)
6. **[参数](06_params.md)**
7. **[客户端](07_client.md)**
   - **[认证](07_client.md#auth)**
      - [CLI](07_client.md#cli)
      - [gRPC](07_client.md#grpc)
      - [REST](07_client.md#rest)
   - **[归属](07_client.md#vesting)**
      - [CLI](07_client.md#vesting#cli) 