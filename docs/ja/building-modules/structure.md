# 推荐的文件夹结构

本文档概述了 Cosmos SDK 模块的推荐结构。 这些想法旨在作为建议加以应用。 鼓励应用程序开发人员改进模块结构和开发设计并做出贡献。 {概要}

## 结构

一个典型的 Cosmos SDK 模块结构如下: 

```shell
proto
└── {project_name}
    └── {module_name}
        └── {proto_version}
            ├── {module_name}.proto
            ├── event.proto
            ├── genesis.proto
            ├── query.proto
            └── tx.proto
```

- `{module_name}.proto`:模块的通用消息类型定义。
- `event.proto`:与事件相关的模块消息类型定义。
- `genesis.proto`:与创世状态相关的模块消息类型定义。
- `query.proto`:模块的查询服务和相关的消息类型定义。
- `tx.proto`:模块的消息服务和相关消息类型定义。 

```shell
x/{module_name}
├── client
│   ├── cli
│   │   ├── query.go
│   │   └── tx.go
│   └── testutil
│       ├── cli_test.go
│       └── suite.go
├── exported
│   └── exported.go
├── keeper
│   ├── genesis.go
│   ├── grpc_query.go
│   ├── hooks.go
│   ├── invariants.go
│   ├── keeper.go
│   ├── keys.go
│   ├── msg_server.go
│   └── querier.go
├── module
│   └── module.go
├── simulation
│   ├── decoder.go
│   ├── genesis.go
│   ├── operations.go
│   └── params.go
├── spec
│   ├── 01_concepts.md
│   ├── 02_state.md
│   ├── 03_messages.md
│   └── 04_events.md
├── {module_name}.pb.go
├── abci.go
├── codec.go
├── errors.go
├── events.go
├── events.pb.go
├── expected_keepers.go
├── genesis.go
├── genesis.pb.go
├── keys.go
├── msgs.go
├── params.go
├── query.pb.go
└── tx.pb.go
```

- `client/`:模块的 CLI 客户端功能实现和模块的集成测试套件。
- `exported/`:模块的导出类型 - 通常是接口类型。如果一个模块依赖于另一个模块的 Keepers，则期望通过 `expected_keepers.go` 文件(见下文)接收作为接口合同的 Keeper，以避免直接依赖于实现 Keepers 的模块。但是，这些接口契约可以定义操作和/或返回特定于实现 Keepers 的模块的类型的方法，这就是“exported/”发挥作用的地方。 `exported/` 中定义的接口类型使用规范类型，允许模块通过 `expected_keepers.go` 文件接收作为接口合约的 Keeper。这种模式允许代码保持 DRY 并缓解导入周期混乱。
- `keeper/`:模块的`Keeper` 和`MsgServer` 实现。
- `module/`:模块的`AppModule` 和`AppModuleBasic` 实现。
- `simulation/`:模块的 [simulation](./simulator.html) 包定义了区块链模拟器应用程序 (`simapp`) 使用的函数。
- `spec/`:模块的规范文档概述了重要概念、状态存储结构以及消息和事件类型定义。
- 根目录包括消息、事件和创世状态的类型定义，包括协议缓冲区生成的类型定义。
    - `abci.go`:模块的`BeginBlocker` 和`EndBlocker` 实现(只有在需要定义`BeginBlocker` 和/或`EndBlocker` 时才需要此文件)。
    - `codec.go`:模块的接口类型注册方法。
    - `errors.go`:模块的哨兵错误。
    - `events.go`:模块的事件类型和构造函数。
    - `expected_keepers.go`:模块的 [expected keeper](./keeper.html#type-definition) 接口。
    - `genesis.go`:模块的创世状态方法和辅助函数。
    - `keys.go`:模块的存储键和相关的辅助函数。
    - `msgs.go`:模块的消息类型定义和相关方法。
    - `params.go`:模块的参数类型定义和相关方法。
    - `*.pb.go`:由 Protocol Buffers 生成的模块类型定义(在上面相应的 `*.proto` 文件中定义)。

## 下一个 {hide}

了解 [错误](./errors.md) {hide} 