# ADR 041:就地存储迁移

## 变更日志

- 17.02.2021:初稿

## 地位

公认

## 摘要

此 ADR 引入了一种机制，可在链软件升级期间执行就地状态存储迁移。

## 语境

当链升级在模块内引入状态破坏性更改时，当前过程包括将整个状态导出到 JSON 文件(通过 `simd export` 命令)，在 JSON 文件上运行迁移脚本(`simd migrate` 命令)，清除存储(`simd unsafe-reset-all` 命令)，并使用迁移的 JSON 文件作为新的创世(可以选择使用自定义初始块高度)开始一个新链。 [在 Cosmos Hub 3->4 迁移指南中](https://github.com/cosmos/gaia/blob/v4.0.3/docs/migration/cosmoshub-3.md#升级程序)。

由于多种原因，此过程很麻烦:

- 程序需要时间。运行 `export` 命令可能需要几个小时，使用迁移的 JSON 在新链上运行 `InitChain` 可能需要一些额外的时间。
- 导出的 JSON 文件可能很重(~100MB-1GB)，难以查看、编辑和传输，这反过来又引入了额外的工作来解决这些问题(例如 [streaming genesis](https://github.com) /cosmos/cosmos-sdk/issues/6936))。

## 决定

我们提出了一种基于就地修改 KV 存储而不涉及​​上述 JSON 导出-过程-导入流程的迁移过程。

### 模块`ConsensusVersion`

我们在`AppModule` 接口上引入了一个新方法: 

```go
type AppModule interface {
    // --snip--
    ConsensusVersion() uint64
}
```

此方法返回一个 `uint64` 作为模块的状态破坏版本。 它必须在模块引入的每个破坏共识的更改上递增。 为了避免默认值的潜在错误，模块的初始版本必须设置为 1。在 Cosmos SDK 中，版本 1 对应于 v0.41 系列中的模块。

### 特定于模块的迁移函数

对于模块引入的每个破坏共识的更改，必须使用其新添加的 `RegisterMigration` 方法在 `Configurator` 中注册从 ConsensusVersion `N` 到版本 `N+1` 的迁移脚本。 所有模块都在“AppModule”上的“RegisterServices”方法中收到对配置器的引用，这是应该注册迁移功能的地方。 迁移函数应按升序注册。 

```go
func (am AppModule) RegisterServices(cfg module.Configurator) {
    // --snip--
    cfg.RegisterMigration(types.ModuleName, 1, func(ctx sdk.Context) error {
        // Perform in-place store migrations from ConsensusVersion 1 to 2.
    })
     cfg.RegisterMigration(types.ModuleName, 2, func(ctx sdk.Context) error {
        // Perform in-place store migrations from ConsensusVersion 2 to 3.
    })
    // etc.
}
```

例如，如果模块的新 ConsensusVersion 是 `N`，则必须在配置器中注册 `N-1` 迁移函数。

在 Cosmos SDK 中，迁移功能由每个模块的 keeper 处理，因为 keeper 持有用于执行就地存储迁移的 `sdk.StoreKey`。 为了不让 keeper 过载，每个模块都使用一个 `Migrator` 包装器来处理迁移功能: 

```go
// Migrator is a struct for handling in-place store migrations.
type Migrator struct {
  BaseKeeper
}
```

迁移函数应该位于每个模块的 `migrations/` 文件夹中，并由迁移器的方法调用。 我们建议方法名称的格式为`Migrate{M}to{N}`。 

```go
// Migrate1to2 migrates from version 1 to 2.
func (m Migrator) Migrate1to2(ctx sdk.Context) error {
	return v043bank.MigrateStore(ctx, m.keeper.storeKey) // v043bank is package `x/bank/migrations/v043`.
}
```

每个模块的迁移功能都特定于模块的存储演变，在本 ADR 中没有描述。 引入ADR-028长度前缀地址后x/bank store key迁移的例子可以在这个[store.go代码](https://github.com/cosmos/cosmos-sdk/blob/36f68eb9e041e20a5bb47e216ac5eb8b91f95471/ x/bank/legacy/v043/store.go#L41-L62)。

### 跟踪 `x/upgrade` 中的模块版本

我们在 `x/upgrade` 的存储中引入了一个新的前缀存储。 该存储将跟踪每个模块的当前版本，它可以被建模为模块名称到模块 ConsensusVersion 的 `map[string]uint64`，并将在运行迁移时使用(详情请参阅下一节)。 使用的键前缀为`0x1`，键/值格式为:

```
0x2 | {bytes(module_name)} => BigEndian(module_consensus_version)
```

存储的初始状态是通过 `app.go` 的 `InitChainer` 方法设置的。

UpgradeHandler 签名需要更新以获取 `VersionMap`，并返回升级后的 `VersionMap` 和错误: 

```diff
- type UpgradeHandler func(ctx sdk.Context, plan Plan)
+ type UpgradeHandler func(ctx sdk.Context, plan Plan, versionMap VersionMap) (VersionMap, error)
```

为了应用升级，我们从 `x/upgrade` 存储中查询 `VersionMap` 并将其传递给处理程序。 处理程序运行实际的迁移功能(见下一节)，如果成功，则返回更新的“VersionMap”以存储在状态中。 

```diff
func (k UpgradeKeeper) ApplyUpgrade(ctx sdk.Context, plan types.Plan) {
    // --snip--
-   handler(ctx, plan)
+   updatedVM, err := handler(ctx, plan, k.GetModuleVersionMap(ctx)) // k.GetModuleVersionMap() fetches the VersionMap stored in state.
+   if err != nil {
+       return err
+   }
+
+   // Set the updated consensus versions to state
+   k.SetModuleVersionMap(ctx, updatedVM)
}
```

还将添加一个 gRPC 查询端点，用于查询存储在 `x/upgrade` 状态中的 `VersionMap`，以便应用程序开发人员可以在升级处理程序运行之前仔细检查 `VersionMap`。 

### Running Migrations

一旦在配置器中注册了所有迁移处理程序(在启动时发生)，就可以通过调用 `module.Manager` 上的 `RunMigrations` 方法来运行迁移。此函数将遍历所有模块，并为每个模块:

- 从它的 `VersionMap` 参数中获取模块的旧 ConsensusVersion(我们称之为 `M`)。
- 从`AppModule` 上的`ConsensusVersion()` 方法获取模块的新ConsensusVersion(称之为`N`)。
- 如果`N>M`，则依次运行模块的所有注册迁移`M -> M+1 -> M+2...` 直到`N`。
    - 有一种特殊情况，模块没有 ConsensusVersion，因为这意味着模块在升级过程中被新添加。在这种情况下，不运行迁移功能，模块当前的 ConsensusVersion 保存到 `x/upgrade` 的存储中。

如果缺少所需的迁移(例如，如果尚未在 `Configurator` 中注册)，则 `RunMigrations` 函数将出错。

在实践中，应该从 `UpgradeHandler` 内部调用 `RunMigrations` 方法。 

```go
app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {
    return app.mm.RunMigrations(ctx, vm)
})
```

假设链在块 n 升级，程序应如下运行:

- 当启动块“N”时，旧的二进制文件将在“BeginBlock”中停止。 在它的存储中，存储了旧二进制模块的共识版本。
- 新的二进制文件将从块“N”开始。 UpgradeHandler 设置在新的二进制文件中，因此将在新二进制文件的“BeginBlock”处运行。 在`x/upgrade` 的`ApplyUpgrade` 中，`VersionMap` 将从(旧二进制的)存储中检索，并传递到`RunMigrations` 函数，在模块自己的`BeginBlock 之前就地迁移所有模块存储 `s. 

## Consequences

### Backwards Compatibility

该 ADR 在 `AppModule` 上引入了一个新方法 `ConsensusVersion()`，所有模块都需要实现该方法。 它还改变了 UpgradeHandler 函数签名。 因此，它不是向后兼容的。

虽然模块在碰撞 ConsensusVersions 时必须注册它们的迁移函数，但使用升级处理程序运行这些脚本是可选的。 应用程序很可能决定不在其升级处理程序中调用 `RunMigrations`，而是继续使用旧的 JSON 迁移路径。 

### Positive

- 在不操作 JSON 文件的情况下执行链升级。
- 虽然尚未制定基准，但就地存储迁移可能比 JSON 迁移花费的时间更少。 支持这种说法的主要原因是旧二进制文件上的 `simd export` 命令和新二进制文件上的 `InitChain` 函数都将被跳过。

### Negative

- 模块开发人员必须正确跟踪其模块中破坏共识的更改。 如果在模块中引入了破坏共识的更改而没有相应的 `ConsensusVersion()` 凸点，那么 `RunMigrations` 函数将不会检测到迁移，并且链升级可能不成功。 文档应该清楚地反映这一点。 

### Neutral

- Cosmos SDK 将继续通过现有的 `simd export` 和 `simd migrate` 命令支持 JSON 迁移。
- 当前的 ADR 不允许创建、重命名或删除存储，只能修改现有的存储键和值。 Cosmos SDK 已经为这些操作提供了`StoreLoader`。 
## Further Discussions

## References

- 初步讨论:https://github.com/cosmos/cosmos-sdk/discussions/8429
- `ConsensusVersion` 和 `RunMigrations` 的实现:https://github.com/cosmos/cosmos-sdk/pull/8485
- 讨论 `x/upgrade` 设计的问题:https://github.com/cosmos/cosmos-sdk/issues/8514 
