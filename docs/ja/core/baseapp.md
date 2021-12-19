# 基础应用

本文档描述了“BaseApp”，它是实现 Cosmos SDK 应用程序核心功能的抽象。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}
- [Cosmos SDK 交易的生命周期](../basics/tx-lifecycle.md) {prereq}

## 介绍

`BaseApp` 是一个基本类型，它实现了 Cosmos SDK 应用程序的核心，即:

- [应用区块链接口](#abci)，用于状态机与底层共识引擎(例如 Tendermint)进行通信。
- [服务路由器](#service-routers)，将消息和查询路由到适当的模块。
- 不同的 [states](#states)，因为状态机可以根据收到的 ABCI 消息更新不同的易失性状态。

BaseApp 的目标是提供 Cosmos SDK 应用程序的基础层
开发人员可以轻松扩展以构建自己的自定义应用程序。通常，
开发人员将为他们的应用程序创建自定义类型，如下所示: 

```go
type App struct {
  // reference to a BaseApp
  *baseapp.BaseApp

  // list of application store keys

  // list of application keepers

  // module manager
}
```

使用“BaseApp”扩展应用程序可以让前者访问“BaseApp”的所有方法。
这允许开发人员使用他们想要的模块组合他们的自定义应用程序，而不是
必须关心实施 ABCI、服务路由器和状态的艰苦工作
管理逻辑。

## 类型定义

“BaseApp”类型为任何基于 Cosmos SDK 的应用程序保存了许多重要参数。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/baseapp.go#L46-L131

让我们来看看最重要的组件。

> **注意**:没有描述所有参数，仅描述最重要的参数。参考
> 完整列表的类型定义。

首先，在应用程序引导期间初始化的重要参数:

- [`CommitMultiStore`](./store.md#commitmultistore):这是应用程序的主存储，
  它保存在 [每个块的结尾](#commit) 提交的规范状态。这家存储
  **不** 缓存，这意味着它不用于更新应用程序的易失性(未提交)状态。
  `CommitMultiStore` 是一个多存储，意思是存储的存储。应用程序的每个模块
  在 multi-store 中使用一个或多个 `KVStores` 来持久化它们的状态子集。
- 数据库:`commitMultiStore` 使用 `db` 来处理数据持久性。
- [`Msg` 服务路由器](#msg-service-router):`msgServiceRouter` 有助于将 `sdk.Msg` 请求路由到适当的
  用于处理的模块 `Msg` 服务。这里的 `sdk.Msg` 指的是需要被处理的事务组件
  由服务处理以更新应用程序状态，而不是实现 ABCI 消息
  应用程序和底层共识引擎之间的接口。
- [gRPC 查询路由器](#grpc-query-router):`grpcQueryRouter` 有助于将 gRPC 查询路由到
  适当的模块进行处理。这些查询本身不是 ABCI 消息，但它们
  被中继到相关模块的 gRPC `Query` 服务。
- [`TxDecoder`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#TxDecoder):用于解码
  由底层 Tendermint 引擎中继的原始交易字节。
- [`ParamStore`](#paramstore):用于获取和设置应​​用程序共识参数的参数存储。
- [`AnteHandler`](#antehandler):这个handler用于处理签名验证，费用支付，
  和其他消息前执行检查何时收到交易。它在执行期间
  [`CheckTx/RecheckTx`](#checktx) 和 [`DeliverTx`](#delivertx)。
- [`InitChainer`](../basics/app-anatomy.md#initchainer),
  [`BeginBlocker` 和 `EndBlocker`](../basics/app-anatomy.md#beginblocker-and-endblocker):这些是
  当应用程序收到 `InitChain`、`BeginBlock` 和 `EndBlock` 时执行的函数
  来自底层 Tendermint 引擎的 ABCI 消息。

然后，用于定义 [易失性状态](#volatile-states)(即缓存状态)的参数:

- `checkState`:此状态在 [`CheckTx`](#checktx) 期间更新，并在 [`Commit`](#commit) 时重置。
- `deliverState`:此状态在 [`DeliverTx`](#delivertx) 期间更新，并在上设置为 `nil`
  [`Commit`](#commit) 并在 BeginBlock 上重新初始化。

最后，几个比较重要的参数:

- `voteInfos`:此参数携带缺少预提交的验证器列表，或者
  因为他们没有投票或因为提议者没有包括他们的投票。这个信息是
  由 [Context](#context) 携带，应用程序可以将其用于各种事情，例如
  惩罚缺席的验证者。
- `minGasPrices`:该参数定义了节点接受的最低汽油价格。这是一个
  **local** 参数，意味着每个全节点可以设置不同的 `minGasPrices`。它用于
  [`CheckTx`](#checktx)期间的`AnteHandler`，主要作为垃圾邮件防护机制。交易
  进入[mempool](https://tendermint.com/docs/tendermint-core/mempool.html#transaction-ordering)
  仅当交易的 gas 价格大于其中之一的最低 gas 价格时
  `minGasPrices`(例如，如果`minGasPrices == 1uatom,1photon`，则交易的`gas-price` 必须为
  大于“1uatom”或“1photon”)。
- `appVersion`:应用程序的版本。它设置在
  [应用程序的构造函数](../basics/app-anatomy.md#constructor-function)。

## 构造函数 

```go
func NewBaseApp(
  name string, logger log.Logger, db dbm.DB, txDecoder sdk.TxDecoder, options ...func(*BaseApp),
) *BaseApp {

  // ...
}
```

`BaseApp` 构造函数非常简单。唯一值得注意的是
可能提供额外的 [`options`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/options.go)
到`BaseApp`，它将按顺序执行它们。 `options` 通常是 `setter` 函数
对于重要参数，例如`SetPruning()` 设置修剪选项或`SetMinGasPrices()` 设置
节点的`min-gas-prices`。

当然，开发人员可以根据应用程序的需要添加额外的“选项”。

## 状态更新

`BaseApp` 维护两个主要的 volatile 状态和一个根或主状态。主要状态
是应用程序的规范状态和易失状态，`checkState` 和 `deliverState`，
用于处理在 [`Commit`](#commit) 期间所做的主状态之间的状态转换。

在内部，只有一个 `CommitMultiStore`，我们将其称为主状态或根状态。
从这个根状态，我们通过使用一种称为 _store 分支_(由`CacheWrap` 函数执行)的机制派生出两个易失性状态。
这些类型可以说明如下:

![类型](./baseapp_state_types.png)

### InitChain 状态更新

在 InitChain 期间，通过分支设置了两个 volatile 状态，即 checkState 和 deliverState
根`CommitMultiStore`。任何后续的读取和写入都发生在 `CommitMultiStore` 的分支版本上。
为了避免不必要的主状态往返，所有对分支存储的读取都被缓存。

![InitChain](./baseapp_state-initchain.png)

### CheckTx 状态更新

在 `CheckTx` 期间，`checkState`，它基于来自根的最后提交的状态
存储，用于任何读取和写入。这里我们只执行 `AnteHandler` 并验证一个服务路由器
交易中的每条消息都存在。注意，当我们执行 `AnteHandler` 时，我们分支
已经分支的`checkState`。这有副作用，如果 `AnteHandler` 失败，
状态转换不会反映在`checkState`中——即`checkState`仅在
成功。

![CheckTx](./baseapp_state-checktx.png)

### BeginBlock 状态更新

在`BeginBlock` 期间，`deliverState` 被设置为在随后的`DeliverTx` ABCI 消息中使用。这
`deliverState` 基于来自根存储的最后提交的状态并且是分支的。
请注意，在 [`Commit`](#commit) 上，`deliverState` 设置为 `nil`。

![BeginBlock](./baseapp_state-begin_block.png)

### DeliverTx 状态更新

“DeliverTx”的状态流与“CheckTx”几乎相同，除了状态转换发生在
执行交易中的 `deliverState` 和消息。与“CheckTx”类似，状态转换
发生在双分支状态 - `deliverState`。成功的消息执行结果
写入致力于`deliverState`。注意，如果消息执行失败，状态从
AnteHandler 被持久化。

![DeliverTx](./baseapp_state-deliver_tx.png)

### 提交状态更新

在 `Commit` 期间，发生在 `deliverState` 中的所有状态转换最终都会写入
根 `CommitMultiStore` 反过来提交到磁盘并产生一个新的应用程序
根哈希。这些状态转换现在被认为是最终的。最后，`checkState` 被设置为
新提交的状态和 `deliverState` 设置为 `nil` 以在 `BeginBlock` 上重置。

![提交](./baseapp_state-commit.png)

## 参数存储

在 `InitChain` 期间，`RequestInitChain` 提供包含参数的 `ConsensusParams`
除了证据参数之外，还与块执行相关，例如最大气体和大小。如果这些
参数非 nil，它们在 BaseApp 的 `ParamStore` 中设置。在幕后，`ParamStore`
实际上由一个 `x/params` 模块 `Subspace` 管理。这允许通过调整参数
链上治理。

## 服务路由器

当应用程序接收到消息和查询时，它们必须被路由到适当的模块以进行处理。路由是通过“BaseApp”完成的，它包含一个用于消息的“msgServiceRouter”和一个用于查询的“grpcQueryRouter”。

### `Msg` 服务路由器

[`sdk.Msg`s](#../building-modules/messages-and-queries.md#messages) 从交易中提取出来后需要路由，这些交易是通过 [` CheckTx`](#checktx) 和 [`DeliverTx`](#delivertx) ABCI 消息。为此，`BaseApp` 拥有一个 `msgServiceRouter`，它将完全限定的服务方法(`string`，在每个模块的 Protobuf `Msg` 服务中定义)映射到相应模块的 `MsgServer` 实现。

[`BaseApp` 中包含的默认 `msgServiceRouter`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/msg_service_router.go) 是无状态的。但是，某些应用程序可能希望使用更多有状态的路由机制，例如允许治理禁用某些路由或将它们指向新模块以进行升级。出于这个原因，`sdk.Context` 也被传递到每个 

应用程序的 `msgServiceRouter` 使用应用程序的 [module manager](../building-modules/module-manager.md#manager)(通过 `RegisterServices` 方法)使用所有路由进行初始化，它本身使用所有路由进行初始化应用程序的 [constructor](../basics/app-anatomy.md#app-constructor) 中的应用程序模块。

### gRPC 查询路由器

与`sdk.Msg`s类似，[`queries`](../building-modules/messages-and-queries.md#queries)需要路由到相应模块的[`Query`服务](../建筑模块/查询服务.md)。为此，`BaseApp` 拥有一个 `grpcQueryRouter`，它将模块的完全限定服务方法(`string`，在它们的 Protobuf `Query` gRPC 中定义)映射到它们的 `QueryServer` 实现。 `grpcQueryRouter` 在查询处理的初始阶段被调用，可以通过直接向 gRPC 端点发送 gRPC 查询，也可以通过 Tendermint RPC 端点上的 [`Query` ABCI 消息](#query)。

就像 `msgServiceRouter` 一样，`grpcQueryRouter` 使用应用程序的 [module manager](../building-modules/module-manager.md)(通过 `RegisterServices` 方法)使用所有查询路由进行初始化，它本身使用应用程序的 [constructor](../basics/app-anatomy.md#app-constructor) 中的所有应用程序模块进行初始化。

## 主要 ABCI 消息

[应用-区块链接口](https://tendermint.com/docs/spec/abci/) (ABCI) 是一个通用接口，将状态机与共识引擎连接起来，形成一个功能齐全的全节点。它可以用任何语言包装，并且需要由每个特定于应用程序的区块链来实现，这些区块链构建在与 ABCI 兼容的共识引擎(如 Tendermint)之上。

共识引擎处理两个主要任务:

- 网络逻辑，主要包括八卦区块部分、交易和共识投票。
- 共识逻辑，导致以区块形式确定的交易顺序。

**不是**共识引擎的作用来定义交易的状态或有效性。通常，交易由共识引擎以‘[]bytes’的形式处理，并通过 ABCI 中继到应用程序进行解码和处理。在网络和共识过程的关键时刻(例如，一个区块的开始，一个区块的提交，一个未确认的交易的接收，......)，共识引擎发出 ABCI 消息供状态机执行。

在 Cosmos SDK 之上构建的开发人员不需要自己实现 ABCI，因为 BaseApp 带有接口的内置实现。让我们来看看`BaseApp` 实现的主要 ABCI 消息:[`CheckTx`](#checktx) 和 [`DeliverTx`](#delivertx) 

### CheckTx

当新的未确认(即尚未包含在有效块中)时，底层共识引擎发送 `CheckTx`
交易由全节点接收。 `CheckTx`的作用是守护全节点的mempool
(其中存储未经确认的交易，直到它们被包含在一个块中)来自垃圾邮件交易。
未经确认的交易仅在通过“CheckTx”时才会转发给对等方。

`CheckTx()` 可以执行 _stateful_ 和 _stateless_ 检查，但开发人员应该努力
使它们轻便。在Cosmos SDK中，在[解码交易](./encoding.md)之后，实现了`CheckTx()`
做以下检查:

1. 从交易中提取`sdk.Msg`。
2. 通过对每个 `sdk.Msg` 调用 `ValidateBasic()` 来执行 _stateless_ 检查。这个完成了
   首先，因为 _stateless_ 检查比 _stateful_ 检查的计算成本更低。如果
   `ValidateBasic()` 失败，`CheckTx` 在运行 _stateful_ 检查之前返回，从而节省资源。
3. 对 [account](../basics/accounts.md) 执行非模块相关的 _stateful_ 检查。这一步主要是检查
   `sdk.Msg` 签名有效，提供足够的费用并且发送帐户
   有足够的资金支付上述费用。请注意，这里没有精确的 [`gas`](../basics/gas-fees.md) 计数，
   因为不处理 `sdk.Msg`。通常，[`AnteHandler`](../basics/gas-fees.md#antehandler) 会检查 `gas` 是否提供
   交易优于基于原始交易规模的最小参考气体量，
   为了避免垃圾邮件提供 0 gas 的交易。
4.确保每个`sdk.Msg`的完全限定服务方法匹配`msgServiceRouter`内的路由，但实际上**不**
   处理`sdk.Msg`s。 `sdk.Msg`s 只在规范状态需要更新时才需要处理，
   这发生在`DeliverTx`期间。

步骤 2. 和 3. 由 [`RunTx()`](#runtx-antehandler-and-runmsgs) 中的 [`AnteHandler`](../basics/gas-fees.md#antehandler) 执行
函数，`CheckTx()` 以 `runTxModeCheck` 模式调用。在`CheckTx()`的每一步中，
特殊的 [volatile state](#volatile-states) 称为 `checkState` 被更新。该状态用于保持
跟踪每个事务的 `CheckTx()` 调用触发的临时更改，无需修改
[主要规范状态](#main-state)。例如，当一个交易通过`CheckTx()`时，
交易费用从“checkState”中的发件人帐户中扣除。如果第二笔交易是
在处理第一个之前从同一个帐户收到，并且该帐户已经消耗了所有
在第一笔交易期间，`checkState` 中的资金，第二笔交易将失败 `CheckTx`() 和
被拒绝。在任何情况下，发送方的账户在交易之前不会实际支付费用
实际上包含在一个块中，因为 `checkState` 永远不会提交到主状态。这
每次一个块被[提交](#commit)时，`checkState` 被重置为主状态的最新状态。

`CheckTx` 向 [`abci.ResponseCheckTx`](https://tendermint.com/docs/spec/abci/abci.html#messages) 类型的底层共识引擎返回响应。
响应包含:

- `代码(uint32)`:响应代码。 `0` 如果成功。
- `Data ([]byte)`:结果字节，如果有的话。
- `Log (string):` 应用程序记录器的输出。可能是不确定的。
- `信息(字符串):` 附加信息。可能是不确定的。
- `GasWanted (int64)`:交易请求的燃料量。它由用户在生成交易时提供。
- `GasUsed (int64)`:交易消耗的气体量。在“CheckTx”期间，该值是通过将交易字节的标准成本乘以原始交易的大小来计算的。接下来是一个例子:
  +++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/auth/ante/basic.go#L104-L105
- `Events ([]cmn.KVPair)`:用于过滤和索引交易(例如按帐户)的键值标签。有关更多信息，请参阅 [`event`s](./events.md)。
- `代码空间(字符串)`:代码的命名空间。

#### 重新检查Tx

在`Commit` 之后，`CheckTx` 将在节点本地内存池中剩余的所有交易上再次运行
在过滤块中包含的那些之后。防止内存池重新检查所有交易
每次提交块时，都可以设置配置选项`mempool.recheck=false`。作为
Tendermint v0.32.1，一个额外的 `Type` 参数可用于 `CheckTx` 函数
指示传入的事务是新的(`CheckTxType_New`)还是重新检查(`CheckTxType_Recheck`)。
这允许在“CheckTxType_Recheck”期间可以跳过某些检查，例如签名验证。 

### DeliverTx

当底层共识引擎收到区块提议时，区块中的每笔交易都需要由应用程序处理。为此，底层共识引擎按顺序向每个交易的应用程序发送“DeliverTx”消息。

在处理给定块的第一笔交易之前，在 [`BeginBlock`](#beginblock) 期间初始化名为 `deliverState` 的 [volatile state](#volatile-states)。每次通过`DeliverTx`处理交易时都会更新此状态，并在块被[提交](#commit)时提交到[主状态](#main-state)，在设置为“nil”之后.

`DeliverTx` 执行 ** 与 `CheckTx`** 完全相同的步骤，在第 3 步有一点需要注意，并增加了第五步:

1. `AnteHandler` **不** 检查交易的 `gas-prices` 是否足够。那是因为检查的 `min-gas-prices` 值 `gas-prices` 是节点本地的，因此对于一个完整节点来说足够的可能不适用于另一个。这意味着提议者可能会免费包含交易，尽管他们没有这样做的动力，因为他们会从他们提议的区块的总费用中获得奖金。
2.对于交易中的每个`sdk.Msg`，路由到相应模块的Protobuf [`Msg`服务](../building-modules/msg-services.md)。执行额外的 _stateful_ 检查，并且在 `deliverState` 的 `context` 中保存的分支多存储由模块的 `keeper` 更新。如果 `Msg` 服务成功返回，则保存在 `context` 中的分支多存储被写入 `deliverState` `CacheMultiStore`。

在 (2) 中概述的附加第五步期间，对存储的每次读/写都会增加“GasConsumed”的值。您可以找到每个操作的默认成本:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/store/types/gas.go#L164-L175

在任何时候，如果 `GasConsumed > GasWanted`，函数返回 `Code != 0` 并且 `DeliverTx` 失败。

`DeliverTx` 向 [`abci.ResponseDeliverTx`](https://tendermint.com/docs/spec/abci/abci.html#delivertx) 类型的底层共识引擎返回响应。响应包含:

- `代码(uint32)`:响应代码。 `0` 如果成功。
- `Data ([]byte)`:结果字节，如果有的话。
- `Log (string):` 应用程序记录器的输出。可能是不确定的。
- `信息(字符串):` 附加信息。可能是不确定的。
- `GasWanted (int64)`:交易请求的燃料量。它由用户在生成交易时提供。
- `GasUsed (int64)`:交易消耗的气体量。在“DeliverTx”期间，该值的计算方法是将交易字节的标准成本乘以原始交易的大小，并在每次对存储进行读/写时添加 gas。
- `Events ([]cmn.KVPair)`:用于过滤和索引交易(例如按帐户)的键值标签。有关更多信息，请参阅 [`event`s](./events.md)。
- `代码空间(字符串)`:代码的命名空间。

## RunTx、AnteHandler 和 RunMsgs 

### RunTx

从 `CheckTx`/`DeliverTx` 调用 `RunTx` 来处理事务，以 `runTxModeCheck` 或 `runTxModeDeliver` 作为参数来区分两种执行模式。请注意，当 `RunTx` 接收到一个交易时，它已经被解码了。

`RunTx` 在被调用时所做的第一件事是通过以适当的模式(`runTxModeCheck` 或 `runTxModeDeliver`)调用 `getContextForTx()` 函数来检索 `context` 的 `CacheMultiStore`。这个 `CacheMultiStore` 是主存储的一个分支，具有缓存功能(用于查询请求)，在 `BeginBlock` 期间为 `DeliverTx` 实例化，并在前一个块的 `Commit` 期间为 `CheckTx` 实例化。之后，调用两个 `defer func()` 进行 [`gas`](../basics/gas-fees.md) 管理。它们在 `runTx` 返回时执行并确保 `gas` 实际消耗，并且会抛出错误(如果有)。

之后，`RunTx()` 在 `Tx` 中的每个 `sdk.Msg` 上调用 `ValidateBasic()`，运行初步的 _stateless_ 有效性检查。如果任何 `sdk.Msg` 未能通过 `ValidateBasic()`，`RunTx()` 将返回一个错误。

然后，运行应用程序的 [`anteHandler`](#antehandler)(如果存在)。在准备这一步时，`checkState`/`deliverState` 的`context` 和`context` 的`CacheMultiStore` 都使用`cacheTxContext()` 函数进行了分支。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/baseapp.go#L623-L630

这允许 `RunTx` 在 `anteHandler` 执行期间不提交对状态所做的更改，如果它最终失败。它还可以防止实现 `anteHandler` 的模块写入状态，这是 Cosmos SDK [object-capabilities](./ocap.md) 的重要组成部分。

最后，调用 [`RunMsgs()`](#runmsgs) 函数来处理 `Tx` 中的 `sdk.Msg`。在准备这一步时，就像`anteHandler`一样，`checkState`/`deliverState`的`context`和`context`的`CacheMultiStore`都使用`cacheTxContext()`函数进行分支。

### 前处理程序

`AnteHandler` 是一个特殊的处理程序，它实现了 `AnteHandler` 接口，用于在处理事务的内部消息之前对事务进行身份验证。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/handler.go#L6-L8

`AnteHandler` 理论上是可选的，但仍然是公共区块链网络的一个非常重要的组成部分。它有 3 个主要目的:

- 通过费用扣除和 [`sequence`](./transactions.md#transaction-generation) 检查，成为抵御垃圾邮件的主要防线和第二道防线(第一道防线是内存池)以防止交易重播。
- 执行初步的 _stateful_ 有效性检查，例如确保签名有效或发件人有足够的资金来支付费用。
- 通过收取交易费用，在激励利益相关者方面发挥作用。

`BaseApp` 持有一个 `anteHandler` 作为参数，该参数在 [应用程序的构造函数](../basics/app-anatomy.md#application-constructor) 中初始化。最广泛使用的`anteHandler`是[`auth`模块](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/ante/ante.go)。

单击 [此处](../basics/gas-fees.md#antehandler) 了解有关 `anteHandler` 的更多信息。

### RunMsgs

`RunMsgs` 是从 `RunTx` 调用的，以 `runTxModeCheck` 作为参数来检查交易中每条消息的路由是否存在，并使用 `runTxModeDeliver` 来实际处理 `sdk.Msg`。

首先，它通过检查代表 `sdk.Msg` 的 Protobuf `Any` 的 `type_url` 来检索 `sdk.Msg` 的完全限定类型名称。然后，使用应用程序的 [`msgServiceRouter`](#msg-service-router)，它会检查与该 `type_url` 相关的 `Msg` 服务方法是否存在。此时，如果`mode == runTxModeCheck`，则返回`RunMsgs`。否则，如果 `mode == runTxModeDeliver`，则在 `RunMsgs` 返回之前执行 [`Msg` 服务](../building-modules/msg-services.md) RPC。

## 其他 ABCI 消息

###初始化链

[`InitChain` ABCI 消息](https://tendermint.com/docs/app-dev/abci-spec.html#initchain) 是在链第一次启动时从底层 Tendermint 引擎发送的。它主要用于**初始化**参数和状态，如:

- [共识参数](https://tendermint.com/docs/spec/abci/apps.html#consensus-parameters) 通过`setConsensusParams`。
- [`checkState` 和`deliverState`](#volatile-states) 通过`setCheckState` 和`setDeliverState`。
- [block gas meter](../basics/gas-fees.md#block-gas-meter)，用无限的gas来处理创世交易。

最后，`BaseApp` 的`InitChain(req abci.RequestInitChain)` 方法调用应用程序的[`initChainer()`](../basics/app-anatomy.md#initchainer) 以初始化主状态从“创世文件”中获取应用程序的名称，如果已定义，则调用每个应用程序模块的 [`InitGenesis`](../building-modules/genesis.md#initgenesis) 函数。

### 开始块

[`BeginBlock` ABCI 消息](#https://tendermint.com/docs/app-dev/abci-spec.html#beginblock) 当收到正确提议者创建的区块提议时从底层 Tendermint 引擎发送, 在为块中的每个事务运行 [`DeliverTx`](#delivertx) 之前。它允许开发人员在每个块的开头执行逻辑。在 Cosmos SDK 中，`BeginBlock(req abci.RequestBeginBlock)` 方法执行以下操作:

- 使用通过`setDeliverState` 函数作为参数传递的`req abci.RequestBeginBlock` 使用最新的标头初始化[`deliverState`](#volatile-states)。
  +++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/baseapp/baseapp.go#L387-L397
  此功能还会重置 [主燃气表](../basics/gas-fees.md#main-gas-meter)。
- 使用 `maxGas` 限制初始化 [block gas meter](../basics/gas-fees.md#block-gas-meter)。块内消耗的“gas”不能超过“maxGas”。该参数在应用程序的共识参数中定义。
- 运行应用程序的[`beginBlocker()`](../basics/app-anatomy.md#beginblocker-and-endblock)，主要运行[`BeginBlocker()`](../building-modules/beginblock) -endblock.md#beginblock) 每个应用程序模块的方法。
- 设置应用程序的 [`VoteInfos`](https://tendermint.com/docs/app-dev/abci-spec.html#voteinfo)，即前一个区块的 _precommit_ 被包含在当前区块的提议者。此信息被携带到 [`Context`](./context.md) 中，以便它可以在 `DeliverTx` 和 `EndBlock` 期间使用。 

### EndBlock

[`EndBlock` ABCI 消息](#https://tendermint.com/docs/app-dev/abci-spec.html#endblock) 在 [`DeliverTx`](#delivertx) 之后从底层 Tendermint 引擎发送为为块中的每个事务运行。它允许开发人员在每个块的末尾执行逻辑。在Cosmos SDK中，批量`EndBlock(req abci.RequestEndBlock)`方法是运行应用程序的[`EndBlocker()`](../basics/app-anatomy.md#beginblocker-and-endblock)，主要是运行每个应用程序模块的 [`EndBlocker()`](../building-modules/beginblock-endblock.md#beginblock) 方法。

### 犯罪

[`Commit` ABCI 消息](https://tendermint.com/docs/app-dev/abci-spec.html#commit) 是在全节点从 2/3 收到 _precommits_ 后从底层 Tendermint 引擎发送的+ 验证者(按投票权加权)。在 BaseApp 端，实现了 Commit(res abci.ResponseCommit) 函数以提交在 BeginBlock、DeliverTx 和 EndBlock 期间发生的所有有效状态转换，并为下一个块重置状态。

为了提交状态转换，`Commit` 函数调用 `deliverState.ms` 上的 `Write()` 函数，其中 `deliverState.ms` 是主存储 `app.cms` 的分支多存储。然后，`Commit` 函数将`checkState` 设置为最新的头部(从`deliverState.ctx.BlockHeader` 获得)，将`deliverState` 设置为`nil`。

最后，`Commit` 将 `app.cms` 的承诺的哈希返回给底层的共识引擎。该散列用作下一个块的标头中的参考。

### 信息

[`Info` ABCI 消息](https://tendermint.com/docs/app-dev/abci-spec.html#info) 是来自底层共识引擎的简单查询，特别是用于将后者与应用程序同步在启动时发生的握手期间。调用时，来自 BaseApp 的 Info(res abci.ResponseInfo) 函数将返回应用程序的名称、版本和 app.cms 上次提交的哈希值。

### 询问

[`Query` ABCI 消息](https://tendermint.com/docs/app-dev/abci-spec.html#query) 用于服务从底层共识引擎接收的查询，包括通过 RPC 接收的查询，如 Tendermint RPC。它曾经是与应用程序构建接口的主要入口点，但随着 Cosmos SDK v0.40 中[gRPC 查询](../building-modules/query-services.md) 的引入，它的使用受到了更多限制。应用程序在实现 `Query` 方法时必须遵守一些规则，这些规则在 [here](https://tendermint.com/docs/app-dev/abci-spec.html#query) 中进行了概述。

每个 Tendermint `query` 都带有一个 `path`，它是一个 `string`，表示要查询的内容。如果 `path` 匹配一个 gRPC 完全限定的服务方法，那么 `BaseApp` 会将查询推迟到 `grpcQueryRouter` 并让它按照 [above](#grpc-query-router) 的解释进行处理。否则，`path` 表示 gRPC 路由器(尚未)处理的查询。 `BaseApp` 用 `/` 分隔符分割 `path` 字符串。按照惯例，拆分字符串的第一个元素 (`splitted[0]`) 包含 `query` 的类别(`app`、`p2p`、`​​store` 或 `custom`)。 `Query(req abci.RequestQuery)` 方法的 `BaseApp` 实现是一个简单的调度程序，为这 4 种主要查询类别提供服务:

- 与应用程序相关的查询，例如查询应用程序的版本，通过 `handleQueryApp` 方法提供。
- 对 multistore 的直接查询，由 `handlerQueryStore` 方法提供服务。这些直接查询不同于通过`app.queryRouter`的自定义查询，主要由第三方服务提供商使用，如区块浏览器。
- P2P 查询，通过`handleQueryP2P` 方法提供服务。这些查询返回“app.addrPeerFilter”或“app.ipPeerFilter”，其中包含分别按地址或 IP 过滤的对等点列表。这些列表首先通过`BaseApp` 的[constructor](#constructor) 中的`options` 初始化。
- 自定义查询，包括遗留查询(在引入 gRPC 查询之前)，通过 `handleQueryCustom` 方法提供。 `handleQueryCustom` 在使用从 `app.queryRouter` 获得的 `queryRoute` 将查询映射到相应模块的 [legacy `querier`](../building-modules/query-services.md#legacy-查询者)。

## 下一个 {hide}

详细了解 [transactions](./transactions.md) {hide} 