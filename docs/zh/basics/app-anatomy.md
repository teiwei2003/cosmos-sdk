<!--
order: 1
-->

# Cosmos SDK 应用剖析

本文档描述了 Cosmos SDK 应用程序的核心部分。 在整个文档中，将使用名为“app”的占位符应用程序。 {概要}

## 节点客户端

Daemon 或 [Full-Node Client](../core/node.md) 是基于 Cosmos SDK 的区块链的核心进程。 网络中的参与者运行这个过程来初始化他们的状态机，与其他全节点连接并在新块进入时更新他们的状态机。 

```
                ^  +-------------------------------+  ^
                |  |                               |  |
                |  |  State-machine = Application  |  |
                |  |                               |  |   Built with Cosmos SDK
                |  |            ^      +           |  |
                |  +----------- | ABCI | ----------+  v
                |  |            +      v           |  ^
                |  |                               |  |
Blockchain Node |  |           Consensus           |  |
                |  |                               |  |
                |  +-------------------------------+  |   Tendermint Core
                |  |                               |  |
                |  |           Networking          |  |
                |  |                               |  |
                v  +-------------------------------+  v
```

区块链全节点将自己呈现为二进制文件，通常后缀“-d”表示“守护进程”(例如，“appd”表示“app”或“gaiad”表示“gaia”)。这个二进制文件是通过运行一个简单的 [`main.go`](../core/node.md#main-function) 函数来构建的，该函数放置在 `./cmd/appd/` 中。此操作通常通过 [Makefile](#dependencies-and-makefile) 发生。

构建主二进制文件后，可以通过运行 [`start` 命令](../core/node.md#start-command) 来启动节点。这个命令函数主要做三件事:

1.创建一个在[`app.go`](#core-application-file)中定义的状态机实例。
2. 使用从存储在`~/.app/data` 文件夹中的`db` 中提取的最新已知状态初始化状态机。此时，状态机处于“appBlockHeight”高度。
3. 创建并启动一个新的 Tendermint 实例。除其他事项外，该节点将与其对等方执行握手。它将从它们那里获取最新的 `blockHeight`，如果它大于本地 `appBlockHeight`，则重放块以同步到这个高度。如果`appBlockHeight`为`0`，则节点从创世开始，Tendermint通过ABCI向`app`发送`InitChain`消息，触发[`InitChainer`](#initchainer)。

## 核心应用程序文件

通常，状态机的核心定义在名为“app.go”的文件中。它主要包含**应用程序的类型定义**和**创建和初始化应用程序**的函数。

### 应用的类型定义

`app.go` 中定义的第一件事是应用程序的 `type`。它一般由以下几部分组成:

- **对[`baseapp`](../core/baseapp.md)的引用。**`app.go`中定义的自定义应用程序是`baseapp`的扩展。当 Tendermint 将交易中继到应用程序时，`app` 使用 `baseapp` 的方法将它们路由到适当的模块。 `baseapp` 实现了应用程序的大部分核心逻辑，包括所有 [ABCI 方法](https://tendermint.com/docs/spec/abci/abci.html#overview) 和 [路由逻辑](../core/baseapp.md#routing)。
- **存储键列表**。包含整个状态的 [store](../core/store.md) 在Cosmos SDK。每个模块使用 multistore 中的一个或多个 store 来持久化它们的状态部分。可以使用在“app”类型中声明的特定键来访问这些商店。这些键和 `keepers` 是 Cosmos SDK [对象功能模型](../core/ocap.md) 的核心。
- **模块的`keeper`s 列表。** 每个模块都定义了一个名为[`keeper`](../building-modules/keeper.md) 的抽象，它处理这个模块存储的读写.一个模块的`keeper`的方法可以从其他模块(如果授权)调用，这就是为什么它们在应用程序的类型中声明并作为接口导出到其他模块，以便后者只能访问授权的功能。
- **对 [`appCodec`](../core/encoding.md) 的引用。** 应用程序的 `appCodec` 用于序列化和反序列化数据结构以存储它们，因为存储只能持久化` []字节`。默认编解码器是 [Protocol Buffers](../core/encoding.md)。
- **对 [`legacyAmino`](../core/encoding.md) 编解码器的引用。** Cosmos SDK 的某些部分尚未迁移到使用上面的 `appCodec`，并且仍然硬编码使用氨基。其他部分明确使用 Amino 以实现向后兼容性。由于这些原因，该应用程序仍然保留对旧版 Amino 编解码器的引用。请注意，Amino 编解码器将在即将发布的版本中从 SDK 中删除。
- **对 [模块管理器](../building-modules/module-manager.md#manager)** 和 [基本模块管理器](../building-modules/module-manager.md# 的引用基本管理器)。模块管理器是一个包含应用程序模块列表的对象。它促进了与这些模块相关的操作，例如注册它们的 [`Msg` 服务](../core/baseapp.md#msg-services) 和 [gRPC `Query` 服务](../core/baseapp.md#grpc -query-services)，或为各种功能设置模块之间的执行顺序，如 [`InitChainer`](#initchainer)、[`BeginBlocker` 和 `EndBlocker`](#beginblocker-and-endblocker)。

请参阅“simapp”中的应用程序类型定义示例，Cosmos SDK 自己的应用程序用于演示和测试目的:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L145-L187

### 构造函数

该函数构造了一个新的应用程序，该应用程序的类型在上一节中定义。它必须满足 `AppCreator` 签名才能在应用程序守护程序命令的 [`start` 命令](../core/node.md#start-command) 中使用。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/types/app.go#L48-L50

以下是此功能执行的主要操作:

- 实例化一个新的 [`codec`](../core/encoding.md) 并使用 [basic manager](../building-modules/module-manager.md) 初始化每个应用程序模块的 `codec` #basicmanager)
- 使用对“baseapp”实例、编解码器和所有适当存储键的引用来实例化新应用程序。
- 使用每个应用程序模块的`NewKeeper` 函数实例化应用程序`type` 中定义的所有[`keeper`s](#keeper)。请注意，`keepers` 必须以正确的顺序实例化，因为一个模块的 `NewKeeper` 可能需要引用另一个模块的 `keeper`。
- 使用每个应用程序模块的 [`AppModule`](#application-module-interface) 对象实例化应用程序的 [模块管理器](../building-modules/module-manager.md#manager)。
- 使用模块管理器，初始化应用程序的 [`Msg` 服务](../core/baseapp.md#msg-services), [gRPC `Query` 服务](../core/baseapp.md#grpc-query -services)、[旧版`Msg` 路由](../core/baseapp.md#routing) 和[旧版查询路由](../core/baseapp.md#query-routing)。当 Tendermint 通过 ABCI 将交易中继到应用程序时，它会使用此处定义的路由路由到相应模块的 [`Msg` 服务](#msg-services)。同样，当应用程序接收到 gRPC 查询请求时，它会使用此处定义的 gRPC 路由路由到相应模块的 [`gRPC 查询服务`](#grpc-query-services)。 Cosmos SDK 仍然支持旧版 `Msg` 和旧版 Tendermint 查询，它们分别使用旧版 `Msg` 路由和旧版查询路由进行路由。
- 使用模块管理器，注册[应用程序模块的不变量](../building-modules/invariants.md)。不变量是在每个块结束时评估的变量(例如令牌的总供应量)。检查不变量的过程是通过一个称为 [`InvariantsRegistry`](../building-modules/invariants.md#invariant-registry) 的特殊模块完成的。不变量的值应等于模块中定义的预测值。如果该值与预测值不同，则将触发不变注册表中定义的特殊逻辑(通常会停止链)。这有助于确保没有严重错误被忽视并产生难以修复的持久影响。
- 使用模块管理器，设置每个[应用程序模块](#application-module-interface) 的`InitGenesis`、`BeginBlocker` 和`EndBlocker` 函数之间的执行顺序。请注意，并非所有模块都实现了这些功能。
- 设置应用程序的其余参数:
    - [`InitChainer`](#initchainer):用于在应用程序第一次启动时对其进行初始化。
    - [`BeginBlocker`, `EndBlocker`](#beginblocker-and-endlbocker):在每个块的开头和结尾调用)。
    - [`anteHandler`](../core/baseapp.md#antehandler):用于处理费用和签名验证。
- 安装商店。
- 退回申请。

请注意，此函数仅创建应用程序的一个实例，而实际状态要么在节点重新启动时从`~/.app/data` 文件夹中转移过来，要么在节点启动时从创世文件中生成。第一次。

请参阅“simapp”中的应用程序构造函数示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L198-L441

### InitChainer

`InitChainer` 是一个函数，它从创世文件(即创世账户的代币余额)初始化应用程序的状态。当应用程序从 Tendermint 引擎接收到 `InitChain` 消息时调用它，这发生在节点在 `appBlockHeight == 0` 处启动时(即在创世时)。应用程序必须通过 [`SetInitChainer`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp#BaseApp.SetInitChainer) 在其 [构造函数](#constructor-function) 中设置 `InitChainer` ) 方法。

一般来说，`InitChainer` 主要由应用程序各个模块的 [`InitGenesis`](../building-modules/genesis.md#initgenesis) 函数组成。这是通过调用模块管理器的 `InitGenesis` 函数来完成的，模块管理器又会调用它包含的每个模块的 `InitGenesis` 函数。请注意，必须使用 [模块管理器的](../building-modules/module-manager.md) `SetOrderInitGenesis` 方法在模块管理器中设置必须调用模块的 `InitGenesis` 函数的顺序。这是在[应用程序的构造函数](#application-constructor) 中完成的，并且必须在“SetInitChainer”之前调用“SetOrderInitGenesis”。

请参阅“simapp”中的“InitChainer”示例: 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L464-L471

### BeginBlocker and EndBlocker

Cosmos SDK 为开发人员提供了将代码自动执行作为其应用程序的一部分的可能性。这是通过两个名为“BeginBlocker”和“EndBlocker”的函数实现的。当应用程序分别从 Tendermint 引擎接收到 `BeginBlock` 和 `EndBlock` 消息时调用它们，这发生在每个块的开头和结尾。应用程序必须通过 [`SetBeginBlocker`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp) 在其 [构造函数](#constructor-function) 中设置 `BeginBlocker` 和 `EndBlocker` #BaseApp.SetBeginBlocker) 和 [`SetEndBlocker`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp#BaseApp.SetEndBlocker) 方法。

一般来说，`BeginBlocker` 和`EndBlocker` 函数主要由每个应用程序模块的[`BeginBlock` 和`EndBlock`](../building-modules/beginblock-endblock.md) 函数组成。这是通过调用模块管理器的`BeginBlock` 和`EndBlock` 函数来完成的，模块管理器又会调用它包含的每个模块的`BeginBlock` 和`EndBlock` 函数。请注意，必须分别使用“SetOrderBeginBlock”和“SetOrderEndBlock”方法在模块管理器中设置必须调用模块“BeginBlock”和“EndBlock”函数的顺序。这是通过[应用程序的构造函数](#application-constructor)中的[模块管理器](../building-modules/module-manager.md)完成的，并且必须调用`SetOrderBeginBlock`和`SetOrderEndBlock`方法在`SetBeginBlocker` 和`SetEndBlocker` 函数之前。

作为旁注，重要的是要记住特定于应用程序的区块链是确定性的。开发人员必须注意不要在“BeginBlocker”或“EndBlocker”中引入不确定性，并且还必须注意不要使它们的计算成本过高，因为 [gas](./gas-fees.md) 不会限制成本`BeginBlocker` 和 `EndBlocker` 执行。

请参阅“simapp”中的“BeginBlocker”和“EndBlocker”函数示例 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L454-L462

### Register Codec

`EncodingConfig` 结构是 `app.go` 文件的最后一个重要部分。 此结构的目标是定义将在整个应用程序中使用的编解码器。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/params/encoding.go#L9-L16

以下是对四个字段中每个字段含义的说明:

- `InterfaceRegistry`:Protobuf 编解码器使用 `InterfaceRegistry` 来处理使用 [`google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto)。 `Any` 可以被认为是一个包含 `type_url`(实现接口的具体类型的名称)和一个 `value`(它的编码字节)的结构。 `InterfaceRegistry` 提供了一种注册接口和实现的机制，可以从 `Any` 安全地解包。应用程序的每个模块都实现了`RegisterInterfaces` 方法，该方法可用于注册模块自己的接口和实现。
    - 您可以在 [ADR-19](../architecture/adr-019-protobuf-state-encoding.md#usage-of-any-to-encode-interfaces) 中阅读有关 Any 的更多信息。
    - 为了更详细地了解，Cosmos SDK 使用了名为 [`gogoprotobuf`](https://github.com/gogo/protobuf) 的 Protobuf 规范的实现。默认情况下，[`Any` 的gogo protobuf 实现](https://godoc.org/github.com/gogo/protobuf/types) 使用[全局类型注册](https://github.com/gogo/protobuf/blob/master/proto/properties.go#L540) 将封装在 `Any` 中的值解码为具体的 Go 类型。这引入了一个漏洞，依赖树中的任何恶意模块都可以在全局 protobuf 注册表中注册一个类型，并导致它被在 `type_url` 字段中引用它的事务加载和解组。更多信息请参考[ADR-019](../architecture/adr-019-protobuf-state-encoding.md)。
- `Marshaler`:整个 Cosmos SDK 中使用的默认编解码器。它由用于编码和解码状态的 `BinaryCodec` 和用于向用户输出数据的 `JSONCodec` 组成(例如在 [CLI](#cli) 中)。默认情况下，SDK 使用 Protobuf 作为`Marshaler`。
- `TxConfig`:`TxConfig` 定义了一个接口，客户端可以利用它来生成应用程序定义的具体事务类型。目前，SDK 处理两种交易类型:“SIGN_MODE_DIRECT”(使用 Protobuf 二进制作为在线编码)和“SIGN_MODE_LEGACY_AMINO_JSON”(取决于 Amino)。阅读有关交易的更多信息 [此处](../core/transactions.md)。
- `Amino`:Cosmos SDK 的一些遗留部分仍然使用 Amino 来实现向后兼容。每个模块都公开了一个 `RegisterLegacyAmino` 方法来在 Amino 中注册模块的特定类型。应用程序开发人员不应再使用此 `Amino` 编解码器，并将在未来版本中删除。

Cosmos SDK 公开了一个 `MakeTestEncodingConfig` 函数，用于为应用程序构造函数(`NewApp`)创建一个 `EncodingConfig`。它使用 Protobuf 作为默认的“Marshaler”。
注意:此功能已标记为已弃用，只能用于创建应用程序或在测试中使用。我们正在努力在星际之门发布后重构编解码器管理。

请参阅“simapp”中的“MakeTestEncodingConfig”示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/590358652cc1cbc13872ea1659187e073ea38e75/simapp/encoding.go#L8-L19

## Modules

[模块](../building-modules/intro.md) 是 Cosmos SDK 应用程序的核心和灵魂。它们可以被视为嵌套在状态机中的状态机。当交易通过 ABCI 从底层 Tendermint 引擎中继到应用程序时，它由 [`baseapp`](../core/baseapp.md) 路由到适当的模块以进行处理。这种范式使开发人员能够轻松构建复杂的状态机，因为他们需要的大多数模块通常已经存在。对于开发人员而言，构建 Cosmos SDK 应用程序所涉及的大部分工作都围绕构建其应用程序所需的尚不存在的自定义模块，并将它们与已经存在的模块集成到一个连贯的应用程序中。在应用程序目录中，标准做法是将模块存储在 `x/` 文件夹中(不要与 Cosmos SDK 的 `x/` 文件夹混淆，其中包含已构建的模块)。

### 应用模块接口

模块必须实现 Cosmos SDK 中定义的 [interfaces](../building-modules/module-manager.md#application-module-interfaces)，[`AppModuleBasic`](../building-modules/module-manager.md #appmodulebasic) 和 [`AppModule`](../building-modules/module-manager.md#appmodule)。前者实现模块的基本非依赖元素，例如`codec`，而后者处理模块的大部分方法(包括需要引用其他模块的`keeper`的方法)。 `AppModule` 和 `AppModuleBasic` 类型都在名为 `./module.go` 的文件中定义。

`AppModule` 在模块上公开了一组有用的方法，这些方法有助于将模块组合成一个连贯的应用程序。这些方法从管理应用程序模块集合的`模块管理器`(../building-modules/module-manager.md#manager) 调用。

### `Msg` 服务

每个模块定义了两个 [Protobuf 服务](https://developers.google.com/protocol-buffers/docs/proto#services):一个用于处理消息的 `Msg` 服务，以及一个用于处理查询的 gRPC `Query` 服务。如果我们将模块视为状态机，那么“Msg”服务就是一组状态转换 RPC 方法。
每个 Protobuf `Msg` 服务方法都与一个 Protobuf 请求类型 1:1 相关，它必须实现 `sdk.Msg` 接口。
请注意，`sdk.Msg`s 捆绑在 [transactions](../core/transactions.md) 中，每个交易包含一条或多条消息。

当全节点收到一个有效的交易块时，Tendermint 通过 [`DeliverTx`](https://tendermint.com/docs/app-dev/abci-spec.html#delivertx) 将每个交易块中继到应用程序.然后，应用程序处理事务:

1. 收到交易后，应用程序首先将其从 `[]bytes` 中解组。
2. 然后，在提取交易中包含的 `Msg`(s) 之前，它会验证一些关于交易的事情，比如 [费用支付和签名](#gas-fees.md#antehandler)。
3. `sdk.Msg`s 使用 Protobuf [`Any`s](#register-codec) 进行编码。通过分析每个 `Any` 的 `type_url`，baseapp 的 `msgServiceRouter` 将 `sdk.Msg` 路由到相应模块的 `Msg` 服务。
4. 如果消息处理成功，则更新状态。

有关更多详细信息，请查看事务 [lifecycle](./tx-lifecycle.md)。

模块开发人员在构建自己的模块时会创建自定义的 `Msg` 服务。一般的做法是在 `tx.proto` 文件中定义 `Msg` Protobuf 服务。例如，`x/bank` 模块定义了一个服务，它具有两种传输令牌的方法:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/bank/v1beta1/tx.proto#L10-L17

服务方法使用 `keeper` 来更新模块状态。

每个模块还应该实现 `RegisterServices` 方法作为 [`AppModule` 接口](#application-module-interface) 的一部分。这个方法应该调用生成的 Protobuf 代码提供的 `RegisterMsgServer` 函数。

### gRPC `Query` 服务

gRPC `Query` 服务是在 v0.40 Stargate 版本中引入的。它们允许用户使用 [gRPC](https://grpc.io) 查询状态。它们默认启用，可以在 [`app.toml`](../run-node/run-node.md#configuring-the- 内的 `grpc.enable` 和 `grpc.address` 字段下配置节点使用应用程序)。

gRPC `Query` 服务在模块的 Protobuf 定义文件中定义，特别是在 `query.proto` 中。 `query.proto` 定义文件公开了一个 `Query` [Protobuf 服务](https://developers.google.com/protocol-buffers/docs/proto#services)。每个 gRPC 查询端点对应一个服务方法，以 rpc 关键字开头，位于 Query 服务中。

Protobuf 为每个模块生成一个 `QueryServer` 接口，包含所有服务方法。一个模块的 [`keeper`](#keeper) 则需要通过提供每个服务方法的具体实现来实现这个 `QueryServer` 接口。这个具体的实现是对应的 gRPC 查询端点的处理程序。

最后，每个模块还应该实现 `RegisterServices` 方法作为 [`AppModule` 接口](#application-module-interface) 的一部分。这个方法应该调用生成的 Protobuf 代码提供的 `RegisterQueryServer` 函数。

### 守门员

[`Keepers`](../building-modules/keeper.md) 是其模块商店的看门人。要在模块的存储中读取或写入，必须通过其 `keeper` 方法之一。这是由 Cosmos SDK 的 [object-capabilities](../core/ocap.md) 模型确保的。只有持有存储密钥的对象才能访问它，并且只有模块的`keeper` 应该持有模块存储的密钥。

`Keepers` 通常定义在一个名为 `keeper.go` 的文件中。它包含 `keeper` 的类型定义和方法。

`keeper` 类型定义通常包括:

- **Key(s)** 到 multistore 中模块的 store(s)。
- 参考**其他模块的`keepers`**。仅当 `keeper` 需要访问其他模块的存储(读取或写入它们)时才需要。
- 对应用程序的 **codec** 的引用。 `keeper` 需要它在存储它们之前编组结构，或者在检索它们时解组它们，因为存储只接受 `[]bytes` 作为值。

与类型定义一起，`keeper.go` 文件的下一个重要组件是 `keeper` 的构造函数，`NewKeeper`。这个函数实例化一个上面定义的类型的新的`keeper`，带有一个`codec`，存储`keys`和潜在的对其他模块`keeper`s的引用作为参数。 `NewKeeper` 函数是从 [应用程序的构造函数](#constructor-function) 调用的。文件的其余部分定义了 `keeper` 的方法，主要是 getter 和 setter。

### 命令行、gRPC 服务和 REST 接口

每个模块都定义了命令行命令、gRPC 服务和 REST 路由，这些路由通过 [应用程序的接口](#application-interfaces) 暴露给最终用户。这使最终用户能够创建模块中定义的类型的消息，或查询模块管理的状态子集。

#### 命令行界面

通常，[与模块相关的命令](../building-modules/module-interfaces.md#cli) 定义在模块文件夹中名为`client/cli` 的文件夹中。 CLI 将命令分为两类，事务和查询，分别在 `client/cli/tx.go` 和 `client/cli/query.go` 中定义。这两个命令都建立在 [Cobra 库](https://github.com/spf13/cobra) 之上:

- 交易命令让用户生成新交易，以便它们可以包含在一个块中并最终更新状态。应该为模块中定义的每个 [消息类型](#message-types) 创建一个命令。该命令使用最终用户提供的参数调用消息的构造函数，并将其包装到事务中。 Cosmos SDK 处理签名和其他交易元数据的添加。
- 查询让用户查询模块定义的状态的子集。查询命令将查询转发到 [application's query router](../core/baseapp.md#query-routing)，后者将它们路由到适当的 [querier](#querier) 提供的 `queryRoute` 参数。 

#### gRPC

[gRPC](https://grpc.io) 是一个现代开源的高性能 RPC 框架，支持多种语言。这是外部客户端(如钱包、浏览器和其他后端服务)与节点交互的推荐方式。

每个模块都可以公开 gRPC 端点，称为 [服务方法](https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition) 并在 [模块的 Protobuf `query.proto 中定义`文件](#grpc-query-services)。服务方法由其名称、输入参数和输出响应定义。然后模块需要:

- 在`AppModuleBasic` 上定义一个`RegisterGRPCGatewayRoutes` 方法，将客户端gRPC 请求连接到模块内的正确处理程序。
- 对于每个服务方法，定义一个相应的处理程序。处理程序实现服务 gRPC 请求所需的核心逻辑，位于 `keeper/grpc_query.go` 文件中。

#### gRPC 网关 REST 端点

一些外部客户端可能不希望使用 gRPC。在这种情况下，Cosmos SDK 提供了一个 gRPC 网关服务，它将每个 gRPC 服务公开为一个对应的 REST 端点。请参阅 [grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) 文档以了解更多信息。

REST 端点与 gRPC 服务一起在 Protobuf 文件中定义，使用 Protobuf 注释。想要公开 REST 查询的模块应该在它们的 `rpc` 方法中添加 `google.api.http` 注释。默认情况下，SDK 中定义的所有 REST 端点都有一个以 `/cosmos/` 前缀开头的 URL。

Cosmos SDK 还提供了一个开发端点来为这些 REST 端点生成 [Swagger](https://swagger.io/) 定义文件。可以在 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 配置文件中的 `api.swagger` 键下启用此端点。

## 应用程序接口

[接口](#command-line-grpc-services-and-rest-interfaces) 让终端用户与全节点客户端交互。这意味着从全节点查询数据或创建和发送由全节点中继并最终包含在一个块中的新交易。

主界面是[命令行界面](../core/cli.md)。 Cosmos SDK 应用程序的 CLI 是通过聚合应用程序使用的每个模块中定义的 [CLI 命令](#cli) 构建的。应用程序的 CLI 与守护进程(例如 `appd`)相同，并在名为 `appd/main.go` 的文件中定义。该文件包含:

- **一个`main()`函数**，用于构建`appd`接口客户端。此函数准备每个命令，并在构建它们之前将它们添加到 `rootCmd`。在`appd` 的根目录下，该函数添加了诸如`status`、`keys` 和`config` 等通用命令、查询命令、tx 命令和`rest-server`。
- **查询命令**是通过调用`queryCmd`函数添加的。此函数返回一个 Cobra 命令，其中包含在每个应用程序模块中定义的查询命令(从 `main()` 函数作为 `sdk.ModuleClients` 数组传递)，以及一些其他较低级别的查询命令，例如块或验证器查询。使用 CLI 的命令 `appd query [query]` 调用查询命令。
- 通过调用`txCmd` 函数添加**交易命令**。与 `queryCmd` 类似，该函数返回一个 Cobra 命令，其中包含在每个应用程序模块中定义的 tx 命令，以及较低级别的 tx 命令，如交易签名或广播。使用 CLI 的命令 `appd tx [tx]` 调用 Tx 命令。

请参阅 [nameservice 教程](https://github.com/cosmos/sdk-tutorials/tree/master/nameservice) 中的应用程序主命令行文件示例

+++ https://github.com/cosmos/sdk-tutorials/blob/86a27321cf89cc637581762e953d0c07f8c78ece/nameservice/cmd/nscli/main.go

## 依赖项和 Makefile

::: 警告
`go-grpc v1.34.0` 中引入的补丁使 gRPC 与 `gogoproto` 库不兼容，导致一些 [gRPC 查询](https://github.com/cosmos/cosmos-sdk/issues/8426) 恐慌。因此，Cosmos SDK 要求在你的 `go.mod` 中安装 `go-grpc <=v1.33.2`。

为确保 gRPC 正常工作，**强烈建议**在您的应用程序的 `go.mod` 中添加以下行: 

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

请参阅 [issue #8392](https://github.com/cosmos/cosmos-sdk/issues/8392) 了解更多信息。
:::

这部分是可选的，因为开发人员可以自由选择他们的依赖管理器和项目构建方法。也就是说，当前最常用的版本控制框架是 [`go.mod`](https://github.com/golang/go/wiki/Modules)。它确保在整个应用程序中使用的每个库都以正确的版本导入。请参阅 [nameservice 教程](https://github.com/cosmos/sdk-tutorials/tree/master/nameservice) 中的示例:

+++ https://github.com/cosmos/sdk-tutorials/blob/c6754a1e313eb1ed973c5c91dcc606f2fd288811/go.mod#L1-L18

为了构建应用程序，通常使用 [Makefile](https://en.wikipedia.org/wiki/Makefile)。 Makefile 主要确保在构建应用程序的两个入口点 [`appd`](#node-client) 和 [`appd`](#application-interface) 之前运行 `go.mod`。请参阅 [nameservice 教程](https://tutorials.cosmos.network/nameservice/tutorial/00-intro.html) 中的 Makefile 示例

+++ https://github.com/cosmos/sdk-tutorials/blob/86a27321cf89cc637581762e953d0c07f8c78ece/nameservice/Makefile

## 下一个{hide}

详细了解 [交易的生命周期](./tx-lifecycle.md) {hide} 