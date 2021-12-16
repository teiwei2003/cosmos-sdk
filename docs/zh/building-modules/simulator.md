# 模块模拟

## 先决条件

* [Cosmos 区块链模拟器](./../using-the-sdk/simulation.md)

## 概要

本文档详细介绍了如何定义每个模块的模拟功能
与应用程序“SimulationManager”集成。

* [仿真包](#simulation-package)
    * [存储解码器](#store-decoders)
    * [随机起源](#randomized-genesis)
    * [随机参数](#randomized-parameters)
    * [随机加权操作](#random-weighted-operations)
    * [随机提案内容](#random-proposal-contents)
* [注册模块模拟功能](#registering-simulation-functions)
* [应用模拟器管理器](#app-simulator-manager)
* [模拟测试](#simulation-tests)

## 模拟包

每个实现 Cosmos SDK 模拟器的模块都需要有一个 `x/<module>/simulation`
包含模糊测试所需主要功能的包:存储
解码器、随机创世状态和参数、加权操作和提议
内容。

### 存储解码器

`AppImportExport` 需要注册存储解码器。这允许
来自要解码的存储的键值对(_i.e_ unmarshalled)
到它们对应的类型。特别是，它将键匹配到具体类型
然后将 `KVPair` 中的值解组为提供的类型。

您可以使用分发模块中的示例 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/distribution/simulation/decoder.go) 来实现您的存储解码器。

### 随机起源

模拟器测试不同的场景和创世参数值
为了充分测试特定模块的边缘情况。每个模块的 `simulator` 包必须公开一个 `RandomizedGenState` 函数以从给定的种子生成初始随机 `GenesisState`。

一旦模块创世参数是随机生成的(或使用密钥和
`params` 文件中定义的值)，它们被编组为 JSON 格式并添加
到应用程序 genesis JSON 以在模拟中使用它。

您可以查看有关如何创建随机创世的示例 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/staking/simulation/genesis.go)。

###随机参数变化

模拟器能够随机测试参数变化。每个模块的模拟器包必须包含一个“RandomizedParams”函数，它将在整个模拟生命周期中模拟模块的参数变化。

您可以在 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/staking/simulation/params.go) 中查看完全测试参数更改所需的示例

### 随机加权操作

操作是 Cosmos SDK 模拟的关键部分之一。他们是交易
(`Msg`) 用随机字段值模拟。操作的发送者
也是随机分配的。

使用完整的 [事务周期](../core/transactions.md) 模拟模拟操作
公开“BaseApp”的“ABCI”应用程序。

下面显示的是如何设置权重:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/staking/simulation/operations.go#L18-L68

如您所见，在这种情况下，权重是预定义的。存在使用不同权重覆盖此行为的选项。一种选择是使用 `*rand.Rand` 为操作定义随机权重，或者您可以注入自己的预定义权重。

以下是如何覆盖上述包“simappparams”。

+++ https://github.com/cosmos/gaia/blob/master/sims.mk#L9-L22

对于最后一个测试，一个名为 runsim <!-- # TODO: add link to runsim readme when its created --> 被使用，这用于并行化 go 测试实例，向 Github 提供信息和松弛集成以向您的团队了解模拟的运行方式。

### 随机提案内容

Cosmos SDK 模拟器也支持随机治理提案。每个
模块必须定义他们公开和注册的治理提案`内容`
它们用于参数。

##注册模拟函数

现在所有必需的功能都已定义，我们需要将它们集成到`module.go` 中的模块模式中:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.0/x/distribution/module.go

## 应用模拟器管理器

以下步骤是在应用程序级别设置“SimulatorManager”。这
下一步需要模拟测试文件。 

```go
type CustomApp struct {
  ...
  sm *module.SimulationManager
}
```

然后在应用程序的实例化时，我们创建 `SimulationManager`
实例与我们创建`ModuleManager` 的方式相同，但这次我们只通过
实现来自`AppModuleSimulation`的模拟功能的模块
界面如上。 

```go
func NewCustomApp(...) {
  // create the simulation manager and define the order of the modules for deterministic simulations
  app.sm = module.NewSimulationManager(
    auth.NewAppModule(app.accountKeeper),
    bank.NewAppModule(app.bankKeeper, app.accountKeeper),
    supply.NewAppModule(app.supplyKeeper, app.accountKeeper),
    ov.NewAppModule(app.govKeeper, app.accountKeeper, app.supplyKeeper),
    mint.NewAppModule(app.mintKeeper),
    distr.NewAppModule(app.distrKeeper, app.accountKeeper, app.supplyKeeper, app.stakingKeeper),
    staking.NewAppModule(app.stakingKeeper, app.accountKeeper, app.supplyKeeper),
    slashing.NewAppModule(app.slashingKeeper, app.accountKeeper, app.stakingKeeper),
  )

  // register the store decoders for simulation tests
  app.sm.RegisterStoreDecoders()
  ...
}
```
