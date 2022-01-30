# 交易生命周期

本文档描述了事务从创建到提交状态更改的生命周期。交易定义在 [不同的文档](../core/transactions.md) 中进行了描述。该交易将被称为“Tx”。 {概要}

### 先决条件阅读

- [Cosmos SDK 应用剖析](./app-anatomy.md) {prereq}

## 创作

### 交易创建

主要的应用程序界面之一是命令行界面。交易`Tx`可以通过用户从[命令行](../core/cli.md)输入以下格式的命令来创建，在`[command]`中提供交易类型，参数在`[args]`，以及 `[flags]` 中的 gas 价格等配置:

```bash
[appname] tx [command] [args] [flags]
```

该命令将自动**创建**交易，使用账户的私钥对其进行**签名**，并**广播**到指定的对等节点。

交易创建有几个必需和可选的标志。 `--from` 标志指定交易源自哪个 [account](./accounts.md)。例如，如果交易是发送硬币，则资金将从指定的“from”地址中提取。

#### 汽油和费用

此外，还有几个 [flags](../core/cli.md) 用户可以用来表明他们愿意在 [fees](./gas-fees.md) 中支付多少:

- `--gas`指的是`Tx`消耗了多少[gas](./gas-fees.md)，代表计算资源。 Gas 取决于交易，并且在执行之前不会精确计算，但可以通过提供 `auto` 作为 `--gas` 的值来估计。
- `--gas-adjustment`(可选)可用于扩展`gas`以避免低估。例如，用户可以将他们的 gas 调整指定为 1.5 以使用 1.5 倍的估计 gas。
- `--gas-prices` 指定用户愿意为每单位 gas 支付多少，可以是一种或多种面额的代币。例如，`--gas-prices=0.025uatom, 0.025upho` 表示用户愿意为每单位gas支付0.025uatom和0.025upho。
- `--fees` 指定用户愿意支付的总费用。
- `--timeout-height` 指定块超时高度，以防止 tx 提交超过特定高度。

支付费用的最终价值等于gas乘以gas价格。换句话说，`fees = ceil(gas * gasPrices)`。因此，由于可以使用 gas 价格计算费用，反之亦然，因此用户仅指定两者之一。

之后，验证者通过将给定或计算出的“gas-prices”与其本地“min-gas-prices”进行比较来决定是否将交易包含在他们的区块中。如果它的`gas-prices` 不够高，`Tx` 将被拒绝，因此激励用户支付更多。

#### CLI 示例

应用程序“app”的用户可以在他们的 CLI 中输入以下命令来生成一个交易，将 1000uatom 从“senderAddress”发送到“recipientAddress”。它指定了他们愿意支付多少 gas:自动估算按比例放大 1.5 倍，每单位 gas 的 gas 价格为 0.025uatom。 

```bash
appd tx send <recipientAddress> 1000uatom --from <senderAddress> --gas auto --gas-adjustment 1.5 --gas-prices 0.025uatom
```

#### 其他交易创建方法

命令行是与应用程序交互的一种简单方法，但也可以使用 [gRPC 或 REST 接口](../core/grpc_rest.md) 或应用程序开发人员定义的其他一些入口点来创建“Tx”。从用户的角度来看，交互取决于他们使用的网络界面或钱包(例如，使用 [Lunie.io](https://lunie.io/#/) 创建 `Tx` 并使用 Ledger Nano S 对其进行签名) .

## 添加到内存池

每个接收到“Tx”的全节点(运行 Tendermint)都会发送一个 [ABCI 消息](https://tendermint.com/docs/spec/abci/abci.html#messages)，
`CheckTx`，到应用层检查有效性，并收到一个 `abci.ResponseCheckTx`。如果“Tx”通过检查，则将其保存在节点中
[**Mempool**](https://tendermint.com/docs/tendermint-core/mempool.html#mempool)，每个节点独有的内存中交易池)等待包含在块中 - 诚实的节点将如果发现无效，则丢弃 `Tx`。在达成共识之前，节点会不断检查传入的交易并将其八卦给他们的对等方。

### 检查类型

全节点执行无状态，然后在“CheckTx”期间对“Tx”进行有状态检查，目的是
尽早识别并拒绝无效交易，以避免浪费计算。

**_Stateless_** 检查不需要节点访问状态——轻客户端或离线节点都可以
它们 - 因此计算成本较低。无状态检查包括确保地址
不为空，强制执行非负数，以及定义中指定的其他逻辑。

**_Stateful_** 根据提交的状态检查验证交易和消息。例子
包括检查相关值是否存在并且能够进行交易，地址
有足够的资金，并且发件人被授权或拥有正确的所有权进行交易。
在任何给定时刻，全节点通常有[多个版本](../core/baseapp.md#volatile-states)
用于不同目的的应用程序的内部状态。例如，节点将执行状态
在验证交易的过程中发生变化，但仍需要最后提交的副本
state 以回答查询 - 他们不应该使用带有未提交更改的 state 进行响应。

为了验证“Tx”，全节点调用“CheckTx”，其中包括 _stateless_ 和 _stateful_
检查。进一步的验证发生在 [`DeliverTx`](#delivertx) 阶段。 `CheckTx` 去
通过几个步骤，从解码“Tx”开始。

### 解码

当应用程序从底层共识引擎(例如 Tendermint)接收到 `Tx` 时，它仍然是 [encoded](../core/encoding.md) `[]byte` 形式，需要按顺序解组待处理。然后，调用 [`runTx`](../core/baseapp.md#runtx-and-runmsgs) 函数在 `runTxModeCheck` 模式下运行，这意味着该函数将运行所有检查但在执行消息和写入状态之前退出变化。

### ValidateBasic

[`sdk.Msg`s](../core/transactions.md#messages)从`Tx`中提取出来，运行模块开发者实现的`sdk.Msg`接口的方法`ValidateBasic`每一个人。 `ValidateBasic` 应该包括基本的 **stateless** 健全性检查。例如，如果消息是将硬币从一个地址发送到另一个地址，“ValidateBasic”可能会检查非空地址和非负硬币数量，但不需要了解状态，例如地址的帐户余额。

### 前处理程序

在 ValidateBasic 检查之后，运行 `AnteHandler`s。从技术上讲，它们是可选的，但在实践中，它们经常出现以执行与区块链交易相关的签名验证、gas 计算、费用扣除和其他核心操作。

缓存上下文的副本提供给 `AnteHandler`，它执行为事务类型指定的有限检查。使用副本允许 AnteHandler 在不修改最后提交的状态的情况下对 `Tx` 进行状态检查，并在执行失败时恢复到原始状态。

例如，[`auth`](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth/spec) 模块 `AnteHandler` 检查和递增序列号，检查签名和帐号，并从交易的第一个签名者那里扣除费用 - 所有状态更改都是使用 `checkState` 进行的。 

### Gas

[`Context`](../core/context.md) 保存了一个 `GasMeter`，它将跟踪在 `Tx` 的执行过程中使用了多少GAS费，被初始化。用户为“Tx”提供的GAS费量称为“GasWanted”。如果`GasConsumed`，即执行期间消耗的gas 量超过`GasWanted`，则执行将停止并且不会提交对状态缓存副本所做的更改。否则，`CheckTx` 设置 `GasUsed` 等于 `GasConsumed` 并在结果中返回它。在计算 gas 和费用值后，验证器节点检查用户指定的“gas-prices”是否大于他们本地定义的“min-gas-prices”。

### 丢弃或添加到内存池

如果在 `CheckTx` 期间的任何时候 `Tx` 失败，它将被丢弃并且事务生命周期结束
那里。否则，如果它成功通过`CheckTx`，默认协议是将其中继到peer
节点并将其添加到内存池，以便“Tx”成为包含在下一个块中的候选者。

**mempool** 用于跟踪所有全节点看到的交易。
全节点保留他们看到的最后一个 `mempool.cache_size` 交易的**mempool cache**，作为第一行
防御以防止重放攻击。理想情况下，`mempool.cache_size` 足够大以包含所有
整个内存池中的交易。如果 mempool 缓存太小而无法跟踪所有
交易，`CheckTx` 负责识别和拒绝重放的交易。

当前现有的预防措施包括费用和“序列”(nonce)计数器，以区分
从相同但有效的交易重放交易。如果攻击者试图向具有许多
“Tx”的副本，保留内存池缓存的全节点将拒绝相同的副本而不是运行
“CheckTx”对所有这些。即使副本增加了“序列”号，攻击者
因需要支付费用而感到沮丧。

验证器节点保留一个内存池以防止重放攻击，就像全节点一样，但也将其用作
准备区块包含的未确认交易池。请注意，即使`Tx`
在这个阶段通过了所有的检查，后面还是有可能被发现无效的，因为
`CheckTx` 不完全验证交易(即它实际上不执行消息)。

## 包含在块中

共识，验证节点就哪些交易达成一致的过程
接受，发生在**轮**。每一轮都从提议者创建一个区块开始
最近的交易并以**验证器**结束，具有投票权的特殊全节点
为了达成共识，同意接受该区块或改为使用“nil”区块。验证节点
执行共识算法，例如[Tendermint BFT](https://tendermint.com/docs/spec/consensus/consensus.html#terms)，
使用 ABCI 请求向应用程序确认交易，以达成本协议。

共识的第一步是**区块提案**。选择验证者中的一名提议者
通过共识算法创建和提议一个块 - 为了包含一个“Tx”，它
必须在这个提议者的内存池中。

## 状态变化

共识的下一步是执行交易以完全验证它们。所有全节点
收到来自正确提议者的区块提议的人通过调用 ABCI 函数执行交易
[`BeginBlock`](./app-anatomy.md#beginblocker-and-endblocker)，每笔交易的`DeliverTx`，
和 [`EndBlock`](./app-anatomy.md#beginblocker-and-endblocker)。虽然每个全节点运行一切
在本地，这个过程产生一个单一的、明确的结果，因为消息的状态转换是确定性的，而交易是
在区块提案中明确排序。 

```
		-----------------------
		|Receive Block Proposal|
		-----------------------
		          |
			  v
		-----------------------
		| BeginBlock	      |
		-----------------------
		          |
			  v
		-----------------------
		| DeliverTx(tx0)      |
		| DeliverTx(tx1)      |
		| DeliverTx(tx2)      |
		| DeliverTx(tx3)      |
		|	.	      |
		|	.	      |
		|	.	      |
		-----------------------
		          |
			  v
		-----------------------
		| EndBlock	      |
		-----------------------
		          |
			  v
		-----------------------
		| Consensus	      |
		-----------------------
		          |
			  v
		-----------------------
		| Commit	      |
		-----------------------
```

### DeliverTx

[`BaseApp`](../core/baseapp.md) 中定义的 `DeliverTx` ABCI 函数完成了大部分
状态转换:按提交的顺序为块中的每个事务运行
在达成共识期间。在幕后，`DeliverTx` 几乎与 `CheckTx` 相同，但调用
[`runTx`](../core/baseapp.md#runtx) 在交付模式而不是检查模式下运行。
全节点不使用它们的 `checkState`，而是使用 `deliverState`:

- **Decoding:** 因为 `DeliverTx` 是一个 ABCI 调用，`Tx` 是以编码的 `[]byte` 形式接收的。
  节点首先解组事务，使用应用程序中定义的 [`TxConfig`](./app-anatomy#register-codec)，然后在 `runTxModeDeliver` 中调用 `runTx`，它与 `CheckTx` 非常相似，但也执行并写入状态变化。

- **检查:**全节点再次调用`validateBasicMsgs`和`AnteHandler`。第二次检查
  发生的原因是他们在添加到 Mempool 阶段期间可能没有看到相同的交易\
  并且恶意提议者可能包含无效提议者。这里的一个区别是
  `AnteHandler` 不会将 `gas-prices` 与节点的 `min-gas-prices` 进行比较，因为该值是本地的
  到每个节点 - 节点之间的不同值会产生不确定的结果。

- **`MsgServiceRouter`:** 虽然 `CheckTx` 会退出，`DeliverTx` 继续运行
  [`runMsgs`](../core/baseapp.md#runtx-and-runmsgs) 在事务中完全执行每个 `Msg`。
  由于事务可能有来自不同模块的消息，BaseApp 需要知道哪个模块
  找到合适的处理程序。这是使用`BaseApp` 的`MsgServiceRouter` 实现的，以便它可以由模块的Protobuf [`Msg` 服务](../building-modules/msg-services.md) 处理。
  对于 `LegacyMsg` 路由，`Route` 函数通过 [module manager](../building-modules/module-manager.md) 调用以检索路由名称并找到遗留的 [`Handler`](../building-modules/msg-services.md#handler-type) 在模块中。

- **`Msg` 服务:** Protobuf `Msg` 服务，是 `AnteHandler` 的一个步骤，负责执行每个
  `Tx` 中的消息并导致状态转换在 `deliverTxState` 中持续存在。

- **Gas:** 在交付`Tx` 时，使用`GasMeter` 来跟踪有多少
  正在使用GAS费；如果执行完成，`GasUsed` 被设置并在
  `abci.ResponseDeliverTx`。如果执行因 `BlockGasMeter` 或 `GasMeter` 已用完或其他原因而停止
  错误，最后的延迟函数适当地错误或恐慌。

如果由于“Tx”无效或“GasMeter”耗尽而导致任何失败的状态更改，
事务处理终止并恢复任何状态更改。无效交易
区块提议导致验证者节点拒绝该区块并改为投票给“nil”区块。

### 犯罪

最后一步是让节点提交块和状态更改。验证节点
执行上一步执行状态转换以验证交易，
然后签署区块以确认它。不是验证器的完整节点不会
参与共识 - 即他们不能投票 - 但听取投票以了解是否或
他们不应该提交状态更改。

当他们收到足够多的验证者投票(2/3+ _precommits_ 由投票权加权)时，全节点提交一个新的区块以添加到区块链中，并且
在应用层完成状态转换。生成一个新的状态根作为
状态转换的默克尔证明。应用程序使用 [`Commit`](../core/baseapp.md#commit)
ABCI 方法继承自 [Baseapp](../core/baseapp.md);它通过以下方式同步所有状态转换
将 `deliverState` 写入应用程序的内部状态。一旦状态发生变化
已提交，`checkState` 从最近提交的状态和 `deliverState` 重新开始
重置为 `nil` 以保持一致并反映更改。

请注意，并非所有区块都具有相同数量的交易，并且有可能达成共识
导致一个 `nil` 块或一个根本没有的块。在公共区块链网络中，也有可能
验证者是**拜占庭**或恶意的，这可能会阻止“Tx”在
区块链。可能的恶意行为包括提议者决定通过以下方式审查“Tx”
将其从区块或投票反对区块的验证者中排除。

至此，一个“Tx”的交易生命周期结束:节点验证了其有效性，
通过执行其状态更改来交付它，并提交这些更改。 `Tx` 本身，
以 `[]byte` 形式存储在一个块中并附加到区块链中。

## 下一个 {hide}

了解 [accounts](./accounts.md) {hide} 