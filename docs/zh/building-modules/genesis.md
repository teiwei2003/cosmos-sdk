<!--
order: 9
-->

# 模块创世纪

模块通常处理状态的一个子集，因此，它们需要定义创世文件的相关子集以及初始化、验证和导出它的方法。 {概要}

## 先决条件阅读

- [模块管理器](./module-manager.md) {prereq}
- [Keepers](./keeper.md) {prereq}

## 类型定义

从给定模块定义的创世状态的子集通常在 `genesis.proto` 文件中定义([更多信息](../core/encoding.md#gogoproto)关于如何定义 protobuf 消息)。定义模块的创世状态子集的结构通常称为“GenesisState”，它包含所有需要在创世过程中初始化的模块相关值。

查看来自 `auth` 模块的 `GenesisState` protobuf 消息定义示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/a9547b54ffac9729fe1393651126ddfc0d236cff/proto/cosmos/auth/v1beta1/genesis.proto

接下来，我们将介绍模块开发人员需要实现的主要与创世相关的方法，以便他们的模块在 Cosmos SDK 应用程序中使用。

### `DefaultGenesis`

`DefaultGenesis()` 方法是一个简单的方法，它使用每个参数的默认值调用 `GenesisState` 的构造函数。查看来自 `auth` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/module.go#L48-L52

### `ValidateGenesis`

调用 `ValidateGenesis(genesisState GenesisState)` 方法来验证提供的 `genesisState` 是否正确。它应该对“GenesisState”中列出的每个参数执行有效性检查。查看来自 `auth` 模块的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/types/genesis.go#L57-L70

## 其他创世方法

除了与 `GenesisState` 直接相关的方法，模块开发者需要实现另外两个方法作为 [`AppModuleGenesis` 接口](./module-manager.md#appmodulegenesis) 的一部分(仅当模块需要初始化一个创世中状态的子集)。这些方法是 [`InitGenesis`](#initgenesis) 和 [`ExportGenesis`](#exportgenesis)。

###`InitGenesis`

`InitGenesis` 方法在应用程序第一次启动时在 [`InitChain`](../core/baseapp.md#initchain) 期间执行。给定一个 `GenesisState`，它通过对 `GenesisState` 中的每个参数使用模块的 [`keeper`](./keeper.md) setter 函数来初始化模块管理的状态子集。

应用的[模块管理器](./module-manager.md#manager)负责依次调用应用的各个模块的`InitGenesis`方法。此顺序由应用程序开发人员通过管理器的`SetOrderGenesisMethod` 设置，该方法在[应用程序的构造函数](../basics/app-anatomy.md#constructor-function) 中调用。

请参阅“auth”模块中的“InitGenesis”示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/genesis.go#L13-L28

###`ExportGenesis`

每当进行状态导出时，都会执行“ExportGenesis”方法。它采用模块管理的状态子集的最新已知版本，并从中创建一个新的“GenesisState”。这主要用于需要通过硬分叉升级链时。

请参阅“auth”模块中的“ExportGenesis”示例。

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/genesis.go#L31-L42

## 下一个{hide}

了解 [模块接口](module-interfaces.md) {hide} 