# 就地存储迁移

::: 警告
在实时链上运行迁移之前，请阅读并理解所有就地存储迁移文档。
:::

使用自定义就地存储迁移逻辑顺利升级您的应用程序模块。 {概要}

Cosmos SDK 使用两种方法来执行升级。

- 使用 `export` CLI 命令将整个应用程序状态导出到 JSON 文件，进行更改，然后使用更改后的 JSON 文件作为创世文件启动一个新的二进制文件。参见[链升级指南 v0.42](https://docs.cosmos.network/v0.42/migrations/chain-upgrade-guide-040.html)。

- v0.44 及更高版本可以执行就地升级，以显着减少具有更大状态的链的升级时间。使用 [模块升级指南](../building-modules/upgrade.md) 设置您的应用程序模块以利用就地升级。

本文档提供了使用就地存储迁移升级方法的步骤。

## 跟踪模块版本

模块开发人员为每个模块分配了一个共识版本。共识版本作为模块的突破性变更版本。 Cosmos SDK 会跟踪 x/upgrade `VersionMap` 存储中的所有模块共识版本。在升级过程中，存储在 state 中的旧 `VersionMap` 和新的 `VersionMap` 之间的差异由 Cosmos SDK 计算。对于每个识别出的差异，将运行特定于模块的迁移，并递增每个升级模块的相应共识版本。

##创世状态

当启动一条新链时，每个模块的共识版本必须在应用程序的创世过程中保存到状态。要保存共识版本，请将以下行添加到“app.go”中的“InitChainer”方法中: 

```diff
func (app *MyApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
  ...
+ app.UpgradeKeeper.SetModuleVersionMap(ctx, app.mm.GetVersionMap())
  ...
}
```

Cosmos SDK 使用此信息来检测何时将具有较新版本的模块引入应用程序。

### 共识版本

共识版本由模块开发者在每个应用模块上定义，作为模块的突破性变更版本。共识版本通知 Cosmos SDK 哪些模块需要升级。例如，如果银行模块是版本 2，并且升级引入了银行模块 3，Cosmos SDK 会升级银行模块并运行“版本 2 到 3”迁移脚本。

### 版本映射

版本映射是模块名称到共识版本的映射。映射被持久化到 x/upgrade 的状态以在就地迁移期间使用。迁移完成后，更新后的版本映射将被持久化以保持状态。

## 升级处理程序

升级使用 `UpgradeHandler` 来促进迁移。应用开发者实现的 `UpgradeHandler` 函数必须符合以下函数签名。这些函数从 x/upgrade 的状态中检索 `VersionMap`，并在升级后返回新的 `VersionMap` 以存储在 x/upgrade 中。两个 `VersionMap` 之间的差异决定了哪些模块需要升级。 

```go
type UpgradeHandler func(ctx sdk.Context, plan Plan, fromVM VersionMap) (VersionMap, error)
```

在这些函数中，您必须执行任何升级逻辑以包含在提供的“计划”中。 所有升级处理程序函数都必须以以下代码行结束: 

```go
  return app.mm.RunMigrations(ctx, cfg, fromVM)
```

## 运行迁移

使用 `app.mm.RunMigrations(ctx, cfg, vm)` 在 `UpgradeHandler` 内部运行迁移。 `UpgradeHandler` 函数描述了升级过程中发生的功能。 `RunMigration` 函数循环遍历 `VersionMap` 参数，并为低于新二进制应用程序模块版本的所有版本运行迁移脚本。 迁移完成后，将返回一个新的“VersionMap”以将升级后的模块版本保存到状态。

```go
cfg := module.NewConfigurator(...)
app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap) (module.VersionMap, error) {

   //...
   //do upgrade logic
   //...

   //RunMigrations returns the VersionMap
   //with the updated module ConsensusVersions
    return app.mm.RunMigrations(ctx, vm)
})
```

要了解有关为模块配置迁移脚本的更多信息，请参阅 [模块升级指南](../building-modules/upgrade.md)。

## 在升级过程中添加新模块

您可以在升级期间向应用程序引入全新的模块。 新模块被识别是因为它们尚未在 `x/upgrade` 的 `VersionMap` 存储中注册。 在这种情况下，`RunMigrations` 从相应的模块调用 `InitGenesis` 函数来设置其初始状态。

### 为新模块添加 StoreUpgrades

所有准备运行就地存储迁移的连锁存储都需要为新模块手动添加存储升级，然后配置存储加载程序以应用这些升级。 这可确保在迁移开始之前将新模块的存储添加到多存储中。 

```go
upgradeInfo, err := app.UpgradeKeeper.ReadUpgradeInfoFromDisk()
if err != nil {
	panic(err)
}

if upgradeInfo.Name == "my-plan" && !app.UpgradeKeeper.IsSkipHeight(upgradeInfo.Height) {
	storeUpgrades := storetypes.StoreUpgrades{
		//add store upgrades for new modules
		//Example:
		//   Added: []string{"foo", "bar"},
		//...
	}

	//configure store loader that checks if version == upgradeHeight and applies store upgrades
	app.SetStoreLoader(upgradetypes.UpgradeStoreLoader(upgradeInfo.Height, &storeUpgrades))
}
```

## 覆盖创世函数

Cosmos SDK 提供了应用程序开发人员可以在其应用程序中导入的模块。 这些模块通常已经定义了一个 `InitGenesis` 函数。

您可以为导入的模块编写自己的 `InitGenesis` 函数。 为此，请在升级处理程序中手动触发您的自定义创世函数。

::: 警告
您必须在传递给 `UpgradeHandler` 函数的版本映射中手动设置共识版本。 如果没有这个，即使您在 `UpgradeHandler` 中触发了自定义函数，SDK 也会运行模块现有的 `InitGenesis` 代码。 
:::

```go
import foo "github.com/my/module/foo"

app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {

   //Register the consensus version in the version map
   //to avoid the SDK from triggering the default
   //InitGenesis function.
    vm["foo"] = foo.AppModule{}.ConsensusVersion()

   //Run custom InitGenesis for foo
    app.mm["foo"].InitGenesis(ctx, app.appCodec, myCustomGenesisState)

    return app.mm.RunMigrations(ctx, cfg, vm)
})
```

如果你没有自定义的创世函数并且想跳过模块的默认创世函数，你可以简单地在 `UpgradeHandler` 中使用版本映射注册模块，如示例所示: 

```go
import foo "github.com/my/module/foo"

app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {

   //Set foo's version to the latest ConsensusVersion in the VersionMap.
   //This will skip running InitGenesis on Foo
    vm["foo"] = foo.AppModule{}.ConsensusVersion()

    return app.mm.RunMigrations(ctx, cfg, vm)
})
```

## 将完整节点同步到升级后的区块链

您可以将完整节点同步到已使用 Cosmovisor 升级的现有区块链

为了成功同步，您必须从区块链在创世时开始的初始二进制文件开始。 如果所有软件升级计划都包含二进制指令，那么您可以运行带有自动下载选项的 Cosmovisor，以自动处理下载和切换到与每个顺序升级相关的二进制文件。 否则，您需要手动向 Cosmovisor 提供所有二进制文件。

要了解有关 Cosmovisor 的更多信息，请参阅 [Cosmovisor 快速入门](../run-node/cosmovisor.md)。 