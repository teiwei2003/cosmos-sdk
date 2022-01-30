# 升级模块

[就地存储迁移](../core/upgrade.html) 允许您的模块升级到包含重大更改的新版本。本文档概述了如何构建模块以利用此功能。 {概要}

## 先决条件阅读

- [就地存储迁移](../core/upgrade.md) {prereq}

## 共识版本

现有模块的成功升级需要每个`AppModule` 实现函数`ConsensusVersion() uint64`。

- 版本必须由模块开发人员硬编码。
- 初始版本**必须**设置为 1。

共识版本用作应用程序模块的状态破坏版本，并且必须在模块引入破坏性更改时递增。

## 注册迁移

要注册在模块升级期间发生的功能，您必须注册要发生的迁移。

迁移注册使用 `RegisterMigration` 方法在 `Configurator` 中进行。对配置器的“AppModule”引用位于“RegisterServices”方法中。

您可以注册一个或多个迁移。如果您注册了多个迁移脚本，请按升序列出迁移，并确保有足够多的迁移可生成所需的共识版本。例如，要迁移到模块的第 3 版，请为第 1 版和第 2 版注册单独的迁移，如下例所示: 

```golang
func (am AppModule) RegisterServices(cfg module.Configurator) {
   //--snip--
    cfg.RegisterMigration(types.ModuleName, 1, func(ctx sdk.Context) error {
       //Perform in-place store migrations from ConsensusVersion 1 to 2.
    })
     cfg.RegisterMigration(types.ModuleName, 2, func(ctx sdk.Context) error {
       //Perform in-place store migrations from ConsensusVersion 2 to 3.
    })
}
```

由于这些迁移是需要访问 Keeper 存储的函数，因此请使用名为“Migrator”的 Keeper 包装器，如下例所示:

+++ https://github.com/cosmos/cosmos-sdk/blob/6ac8898fec9bd7ea2c1e5c79e0ed0c3f827beb55/x/bank/keeper/migrations.go#L8-L21

## 编写迁移脚本

要定义在升级期间发生的功能，请编写迁移脚本并将这些功能放在 `migrations/` 目录中。 例如，要为 bank 模块编写迁移脚本，请将函数放在 `x/bank/migrations/` 中。 对这些函数使用推荐的命名约定。 例如，`v043bank` 是迁移包`x/bank/migrations/v043` 的脚本: 

```golang
//Migrating bank module from version 1 to 2
func (m Migrator) Migrate1to2(ctx sdk.Context) error {
	return v043bank.MigrateStore(ctx, m.keeper.storeKey)//v043bank is package `x/bank/migrations/v043`.
}
```

要查看在余额密钥迁移中实施的更改示例代码，请查看 [migrateBalanceKeys](https://github.com/cosmos/cosmos-sdk/blob/36f68eb9e041e20a5bb47e216ac5eb8b91f95471/x/bank/legacy/v043/store。 去#L41-L62)。 对于上下文，此代码引入了银行存储的迁移，该迁移更新了地址，以字节长度为前缀，如 [ADR-028](../architecture/adr-028-public-key-addresses.md) 中所述。 