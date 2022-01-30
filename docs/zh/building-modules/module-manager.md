# 模块管理器

Cosmos SDK 模块需要实现[`AppModule` 接口](#application-module-interfaces)，以便由应用程序的[模块管理器](#module-manager) 进行管理。模块管理器在 [`message` 和 `query` 路由](../core/baseapp.md#routing) 中扮演着重要的角色，并允许应用程序开发者设置各种函数的执行顺序，例如 [`BeginBlocker ` 和 `EndBlocker`](../basics/app-anatomy.md#begingblocker-and-endblocker)。 {概要}

## 先决条件阅读

- [Cosmos SDK 模块介绍](./intro.md) {prereq}

## 应用模块接口

应用程序模块接口的存在是为了便于将模块组合在一起以形成功能性 Cosmos SDK 应用程序。有3个主要的应用模块接口:

- [`AppModuleBasic`](#appmodulebasic) 用于独立模块功能。
- [`AppModule`](#appmodule) 用于相互依赖的模块功能)与创世相关的功能除外)。
- [`AppModuleGenesis`](#appmodulegenesis) 用于相互依赖的与创世相关的模块功能。

`AppModuleBasic` 接口的存在是为了定义模块的独立方法，即那些不依赖于应用程序中其他模块的方法。这允许在应用程序定义的早期构建基本应用程序结构，通常在[主应用程序文件](../basics/app-anatomy.md#core-application-file)的`init()`函数中.

`AppModule` 接口用于定义相互依赖的模块方法。许多模块需要与其他模块交互，通常通过 [`keeper`s](./keeper.md)，这意味着需要一个接口，其中模块列出它们的 `keeper`s 和其他需要引用的方法另一个模块的对象。 `AppModule` 接口还使模块管理器能够设置模块方法之间的执行顺序，如 `BeginBlock` 和 `EndBlock`，这在模块之间的执行顺序在应用程序上下文中很重要的情况下很重要。

最后，创世功能“AppModuleGenesis”的接口与完整的模块功能“AppModule”分离，以便模块
仅用于 genesis 可以利用 `Module` 模式，而无需定义许多占位符函数。

### `AppModuleBasic`

`AppModuleBasic` 接口定义了模块需要实现的独立方法。

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L49-L63

让我们来看看这些方法:

- `Name()`:以 `string` 形式返回模块的名称。
- `RegisterLegacyAminoCodec(*codec.LegacyAmino)`:为模块注册 `amino` 编解码器，用于向/从 `[]byte` 编组和解组结构，以便将它们保存在模块的 `KVStore` 中。
- `RegisterInterfaces(codectypes.InterfaceRegistry)`:将模块的接口类型及其具体实现注册为`proto.Message`。
- `DefaultGenesis(codec.JSONCodec)`:返回模块的默认 [`GenesisState`](./genesis.md#genesisstate)，编组为 `json.RawMessage`。默认的“GenesisState”需要由模块开发者定义，主要用于测试。
- `ValidateGenesis(codec.JSONCodec, client.TxEncodingConfig, json.RawMessage)`:用于验证模块定义的`GenesisState`，以`json.RawMessage`形式给出。它通常会在运行模块开发人员定义的自定义 [`ValidateGenesis`](./genesis.md#validategenesis) 函数之前解组 `json`。
- `RegisterRESTRoutes(client.Context, *mux.Router)`:为模块注册 REST 路由。这些路由将用于将 REST 请求映射到模块以处理它们。有关更多信息，请参阅 [gRPC 和 REST](../core/grpc_rest.md)。
- `RegisterGRPCGatewayRoutes(client.Context, *runtime.ServeMux)`:为模块注册 gRPC 路由。
- `GetTxCmd()`:返回模块的根 [`Tx` 命令](./module-interfaces.md#tx)。最终用户使用此根命令的子命令来生成包含模块中定义的 [`message`s](./messages-and-queries.md#queries) 的新事务。
- `GetQueryCmd()`:返回模块的根 [`query` 命令](./module-interfaces.md#query)。最终用户使用此根命令的子命令来生成对模块定义的状态子集的新查询。

应用程序的所有`AppModuleBasic` 都由[`BasicManager`](#basicmanager) 管理。

### `AppModuleGenesis`

`AppModuleGenesis` 接口是带有两个附加方法的 `AppModuleBasic` 接口的简单嵌入。 

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L152-L158

让我们来看看添加的两个方法:

- `InitGenesis(sdk.Context, codec.JSONCodec, json.RawMessage)`:初始化模块管理的状态子集。它在创世时)即链第一次启动时)被调用。
- `ExportGenesis(sdk.Context, codec.JSONCodec)`:导出模块管理的最新状态子集，用于新的创世文件。当从现有链的状态启动新链时，每个模块都会调用 `ExportGenesis`。

它没有自己的管理器，并且与 [`AppModule`](#appmodule) 分开存在，仅用于实现创世功能的模块，这样它们就可以被管理而无需实现所有 `AppModule` 的方法.如果模块不仅在创世期间使用，`InitGenesis(sdk.Context, codec.JSONCodec, json.RawMessage)` 和 `ExportGenesis(sdk.Context, codec.JSONCodec)` 通常会被定义为实现的具体类型的方法`AppModule` 接口。

### `AppModule`

`AppModule` 接口定义了模块需要实现的相互依赖的方法。

+++ https://github.com/cosmos/cosmos-sdk/blob/b4cce159bcc6a32ac78245c6866dd87c73f3720d/types/module/module.go#L160-L182

`AppModule`s 由 [模块管理器](#manager) 管理。该接口嵌入了`AppModuleGenesis`接口，以便管理器可以访问模块的所有独立和创世相互依赖的方法。这意味着实现 `AppModule` 接口的具体类型必须要么实现 `AppModuleGenesis`)以及扩展为 `AppModuleBasic`)的所有方法，或者包含一个作为参数的具体类型。

让我们来看看`AppModule`的方法:

- `RegisterInvariants(sdk.InvariantRegistry)`:注册模块的[`invariants`](./invariants.md)。如果一个不变量偏离了它的预测值，[`InvariantRegistry`](./invariants.md#registry) 会触发适当的逻辑)大多数情况下链将被停止)。
- `Route()`:返回 [`message`s](./messages-and-queries.md#messages) 的路由，由 [`BaseApp`](../core/baseapp.md#messages) 路由到模块。 md#消息路由)。
- `QuerierRoute()`)不推荐使用):返回模块的查询路由的名称，用于 [`queries`](./messages-and-queries.md#queries) 是由 [`BaseApp`] 到模块的路由)../core/baseapp.md#query-routing)。
- `LegacyQuerierHandler(*codec.LegacyAmino)`)不推荐使用):返回一个 [`querier`](./query-services.md#legacy-queriers) 给定查询 `path`，以便处理 `query`。
- `RegisterServices(Configurator)`:允许模块注册服务。
- `BeginBlock(sdk.Context, abci.RequestBeginBlock)`:此方法为模块开发人员提供了实现在每个块开始时自动触发的逻辑的选项。如果不需要在此模块的每个块的开头触发逻辑，则实现为空。
- `EndBlock(sdk.Context, abci.RequestEndBlock)`:此方法为模块开发人员提供了实现在每个块结束时自动触发的逻辑的选项。这也是模块可以将验证器集更改通知底层共识引擎的地方)例如，`staking` 模块)。如果不需要在此模块的每个块的末尾触发逻辑，则实现为空。

### 实现应用程序模块接口

通常，各种应用程序模块接口在名为“module.go”的文件中实现，该文件位于模块的文件夹中)例如，“./x/module/module.go”)。

几乎每个模块都需要实现`AppModuleBasic` 和`AppModule` 接口。如果模块仅用于创世，它将实现`AppModuleGenesis`而不是`AppModule`。实现接口的具体类型可以添加实现接口的各种方法所需的参数。例如，`Route()`函数经常调用`keeper/msg_server.go`中定义的`NewMsgServerImpl(k keeper)`函数，因此需要将模块的[`keeper`](./keeper.md)作为一个参数。 

```go
//example
type AppModule struct {
	AppModuleBasic
	keeper       Keeper
}
```

在上面的示例中，您可以看到“AppModule”具体类型引用了“AppModuleBasic”，而不是“AppModuleGenesis”。 那是因为`AppModuleGenesis` 只需要在专注于与创世相关功能的模块中实现。 在大多数模块中，具体的`AppModule` 类型将引用一个`AppModuleBasic`，并直接在`AppModule` 类型中实现`AppModuleGenesis` 的两个添加方法。

如果不需要参数)`AppModuleBasic` 通常就是这种情况)，只需声明一个空的具体类型，如下所示: 

```go
type AppModuleBasic struct{}
```

## 模块管理器

模块管理器用于管理`AppModuleBasic` 和`AppModule` 的集合。

### `基本管理器`

`BasicManager` 是一个列出应用程序所有 `AppModuleBasic` 的结构:

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L65-L66

它实现了以下方法:

- `NewBasicManager(modules ...AppModuleBasic)`:构造函数。它获取应用程序的“AppModuleBasic”列表并构建一个新的“BasicManager”。该函数一般在[`app.go`](../basics/app-anatomy.md#core-application-file)的`init()`函数中调用，用于快速初始化应用程序模块的独立元素)单击 [此处])https://github.com/cosmos/gaia/blob/master/app/app.go#L59-L74)查看示例)。
- `RegisterLegacyAminoCodec(cdc *codec.LegacyAmino)`:注册每个应用程序的`AppModuleBasic`的[`codec.LegacyAmino`s](../core/encoding.md#amino)。这个函数通常在[应用程序的构造](../basics/app-anatomy.md#constructor)的早期被调用。
- `RegisterInterfaces(registry codectypes.InterfaceRegistry)`:注册每个应用程序`AppModuleBasic`的接口类型和实现。
- `DefaultGenesis(cdc codec.JSONCodec)`:通过调用每个模块的[`DefaultGenesis(cdc codec.JSONCodec)`](./genesis.md#defaultgenesis)函数，为应用程序中的模块提供默认的创世信息。它用于为应用程序构建一个默认的创世文件。
- `ValidateGenesis(cdc codec.JSONCodec, txEncCfg client.TxEncodingConfig, genesis map[string]json.RawMessage)`:通过调用[`ValidateGenesis(codec.JSONCodec, client.TxEncodingConfig, json.RawMessage)`验证创世信息模块](./genesis.md#validategenesis) 每个模块的函数。
- `RegisterRESTRoutes(ctx client.Context, rtr *mux.Router)`:通过调用每个模块的 [`RegisterRESTRoutes`](./module-interfaces.md#register-routes) 函数，为模块注册 REST 路由。该函数通常从[应用程序的命令行界面](../core/cli.md)的`main.go`函数中调用。
-`RegisterGRPCGatewayRoutes(clientCtx client.Context, rtr *runtime.ServeMux)`:为模块注册gRPC路由。
- `AddTxCommands(rootTxCmd *cobra.Command)`:将模块的事务命令添加到应用程序的 [`rootTxCommand`](../core/cli.md#transaction-commands)。该函数通常从[应用程序的命令行界面](../core/cli.md)的`main.go`函数中调用。
- `AddQueryCommands(rootQueryCmd *cobra.Command)`:将模块的查询命令添加到应用程序的 [`rootQueryCommand`](../core/cli.md#query-commands)。该函数通常从[应用程序的命令行界面](../core/cli.md)的`main.go`函数中调用。

### `Manager`

`Manager` 是一个包含应用程序所有 `AppModule` 的结构，并定义了这些模块的几个关键组件之间的执行顺序:

+++ https://github.com/cosmos/cosmos-sdk/blob/325be6ff215db457c6fc7668109640cd7fdac461/types/module/module.go#L223-L231

每当需要对模块集合执行操作时，都会在整个应用程序中使用模块管理器。它实现了以下方法: 

- `NewManager(modules ...AppModule)`:构造函数。它获取应用程序的“AppModule”列表并构建一个新的“Manager”。它通常从应用程序的主要[构造函数](../basics/app-anatomy.md#constructor-function) 调用。
- `SetOrderInitGenesis(moduleNames ...string)`:设置应用程序第一次启动时每个模块的 [`InitGenesis`](./genesis.md#initgenesis) 函数的调用顺序。这个函数一般是从应用程序的主[构造函数](../basics/app-anatomy.md#constructor-function)调用的。
- `SetOrderExportGenesis(moduleNames ...string)`:设置在导出的情况下每个模块的 [`ExportGenesis`](./genesis.md#exportgenesis) 函数将被调用的顺序。这个函数一般是从应用程序的主[构造函数](../basics/app-anatomy.md#constructor-function)调用的。
- `SetOrderBeginBlockers(moduleNames ...string)`:设置在每个块的开头调用每个模块的`BeginBlock()`函数的顺序。这个函数一般是从应用程序的主[构造函数](../basics/app-anatomy.md#constructor-function)调用的。
- `SetOrderEndBlockers(moduleNames ...string)`:设置在每个块的末尾调用每个模块的`EndBlock()`函数的顺序。这个函数一般是从应用程序的主[构造函数](../basics/app-anatomy.md#constructor-function)调用的。
- `RegisterInvariants(ir sdk.InvariantRegistry)`:注册每个模块的[invariants](./invariants.md)。
- `RegisterRoutes(router sdk.Router, queryRouter sdk.QueryRouter, legacyQuerierCdc *codec.LegacyAmino)`:注册遗留 [`Msg`](./messages-and-queries.md#messages) 和 [`querier`](./query-services.md#legacy-queriers) 路由。
- `RegisterServices(cfg Configurator)`:注册所有模块服务。
- `InitGenesis(ctx sdk.Context, cdc codec.JSONCodec, genesisData map[string]json.RawMessage)`:在应用程序第一次调用时调用各个模块的[`InitGenesis`](./genesis.md#initgenesis)函数开始，按照 `OrderInitGenesis` 中定义的顺序。向底层共识引擎返回一个 `abci.ResponseInitChain`，其中可以包含验证器更新。
- `ExportGenesis(ctx sdk.Context, cdc codec.JSONCodec)`:按照`OrderExportGenesis`中定义的顺序调用每个模块的[`ExportGenesis`](./genesis.md#exportgenesis)函数。 export 从之前存在的状态构造一个创世文件，主要用于需要对链进行硬分叉升级的时候。
- `BeginBlock(ctx sdk.Context, req abci.RequestBeginBlock)`:在每个块的开头，从 [`BaseApp`](../core/baseapp.md#beginblock) 调用这个函数，然后，调用每个模块的 [`BeginBlock`](./beginblock-endblock.md) 函数，按照 `OrderBeginBlockers` 中定义的顺序。它创建一个带有事件管理器的子 [context](../core/context.md) 来聚合从所有模块发出的 [events](../core/events.md)。该函数返回一个包含上述事件的 `abci.ResponseBeginBlock`。
- `EndBlock(ctx sdk.Context, req abci.RequestEndBlock)`:在每个块的末尾，从 [`BaseApp`](../core/baseapp.md#endblock) 调用这个函数，然后，按照 `OrderEndBlockers` 中定义的顺序调用每个模块的 [`EndBlock`](./beginblock-endblock.md) 函数。它创建一个带有事件管理器的子 [context](../core/context.md) 来聚合从所有模块发出的 [events](../core/events.md)。该函数返回一个“abci.ResponseEndBlock”，其中包含上述事件以及验证器集更新)如果有)。

以下是应用程序中具体集成的示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/2323f1ac0e9a69a0da6b43693061036134193464/simapp/app.go#L315-L362

## 下一个 {hide}

详细了解 [`message`s 和 `queries`](./messages-and-queries.md) {hide} 