# `能力`

## 概述

`x/capability` 是 Cosmos SDK 模块的实现，根据 [ADR 003](./../../../docs/architecture/adr-003-dynamic-capability-store.md)，
允许配置、跟踪和验证多所有者功能
在运行时。

keeper 维护两种状态:持久和短暂的内存。执着的
store 维护一个全局唯一的自动递增索引和一个映射
一组被定义为模块的能力所有者的能力索引和
能力名称元组。内存中的临时状态跟踪实际
能力，表示为本地内存中的地址，具有正向和反向索引。
前向索引将模块名称和能力元组映射到能力名称。这
模块和功能名称与功能本身之间的反向索引映射。

Keeper 允许创建“范围”的子管理器，这些子管理器与特定的
模块名称。必须在应用程序初始化和
传递给模块，然后模块可以使用它们来声明它们接收和
除了创建新功能外，还可以检索他们按名称拥有的功能
& 验证其他模块传递的功能。一个有范围的守护者无法逃脱它的范围，
所以一个模块不能干扰或检查其他模块拥有的功能。

Keeper 不提供其他模块中可以找到的其他核心功能
像查询器、REST 和 CLI 处理程序以及创世状态。

## 初始化

在应用程序初始化期间，Keeper 必须用一个持久化的实例化
存储密钥和内存存储密钥。 

```go
type App struct {
 //...

  capabilityKeeper *capability.Keeper
}

func NewApp(...) *App {
 //...

  app.capabilityKeeper = capability.NewKeeper(codec, persistentStoreKey, memStoreKey)
}
```

After the keeper is created, it can be used to create scoped sub-keepers which
are passed to other modules that can create, authenticate, and claim capabilities.
After all the necessary scoped keepers are created and the state is loaded, the
main capability keeper must be initialized and sealed to populate the in-memory
state and to prevent further scoped keepers from being created.

```go
func NewApp(...) *App {
 //...

 //Initialize and seal the capability keeper so all persistent capabilities
 //are loaded in-memory and prevent any further modules from creating scoped
 //sub-keepers.
  ctx := app.BaseApp.NewContext(true, tmproto.Header{})
  app.capabilityKeeper.InitializeAndSeal(ctx)

  return app
}
```

## Contents

1. **[Concepts](01_concepts.md)**
2. **[State](02_state.md)**
