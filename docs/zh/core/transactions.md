# 交易

`Transactions` 是由最终用户创建的对象，用于触发应用程序中的状态更改。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## 交易

事务由保存在 [contexts](./context.md) 和 [`sdk.Msg`s](../building-modules/messages-and-queries.md) 中的元数据组成，这些元数据通过模块的 Protobuf [`Msg` 服务](../building-modules/msg-services.md)。

当用户想要与应用程序交互并进行状态更改(例如发送硬币)时，他们会创建交易。在将交易广播到网络之前，每个交易的 `sdk.Msg` 都必须使用与适当帐户关联的私钥进行签名。然后，交易必须包含在区块中，通过共识过程由网络进行验证和批准。要阅读有关交易生命周期的更多信息，请单击 [此处](../basics/tx-lifecycle.md)。

## 类型定义

交易对象是实现了“Tx”接口的 Cosmos SDK 类型

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/tx_msg.go#L49-L57

它包含以下方法:

- **GetMsgs:** 解包交易并返回包含的 `sdk.Msg` 列表 - 一个交易可能有一个或多个消息，由模块开发人员定义。
- **ValidateBasic:** 包括 ABCI 消息使用的轻量级 [_stateless_](../basics/tx-lifecycle.md#types-of-checks) 检查 [`CheckTx`](./baseapp.md#checktx)和 [`DeliverTx`](./baseapp.md#delivertx) 以确保交易不是无效的。例如，[`auth`](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth) 模块的 `StdTx` `ValidateBasic` 函数检查其交易是否由正确的数字签名签名者的数量，并且费用不超过用户的最高限额。请注意，此函数与用于 sdk.Msg 的 ValidateBasic 函数不同，后者仅对消息执行基本的有效性检查。例如，当 [`runTx`](./baseapp.md#runtx) 正在检查从 [`auth`](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth/spec) 模块，它首先在每条消息上运行 `ValidateBasic`，然后运行 ​​`auth` 模块 AnteHandler，它为交易本身调用 `ValidateBasic`。

作为开发人员，您应该很少直接操作“Tx”，因为“Tx”实际上是用于生成交易的中间类型。相反，开发人员应该更喜欢 `TxBuilder` 接口，您可以在 [下文](#transaction-generation) 中了解更多信息。

### 签署交易

交易中的每条消息都必须由其 `GetSigners` 指定的地址签名。 Cosmos SDK 目前允许以两种不同的方式签署交易。

#### `SIGN_MODE_DIRECT`(首选)

`Tx` 接口最常用的实现是 Protobuf `Tx` 消息，它在 `SIGN_MODE_DIRECT` 中使用:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L12-L25

由于 Protobuf 序列化不是确定性的，Cosmos SDK 使用额外的“TxRaw”类型来表示交易签名的固定字节。任何用户都可以为交易生成有效的“body”和“auth_info”，并使用 Protobuf 序列化这两条消息。 “TxRaw”然后固定用户的“body”和“auth_info”的精确二进制表示，分别称为“body_bytes”和“auth_info_bytes”。由交易的所有签名者签署的文档是`SignDoc`(使用[ADR-027](../architecture/adr-027-deterministic-protobuf-serialization.md)进行确定性序列化):

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L47-L64

一旦由所有签名者签名，`body_bytes`、`auth_info_bytes` 和`signatures` 将被收集到`TxRaw` 中，其序列化字节在网络上广播。

#### `SIGN_MODE_LEGACY_AMINO_JSON`

`Tx` 接口的遗留实现是来自 `x/auth` 的 `StdTx` 结构:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdtx.go#L120-L130

所有签名者签署的文件是`StdSignDoc`:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdsign.go#L20-L33

使用 Amino JSON 将其编码为字节。一旦所有签名都被收集到 `StdTx` 中，`StdTx` 就会使用 Amino JSON 进行序列化，并且这些字节将通过网络广播。

#### 其他符号模式

正在讨论其他符号模式，最著名的是“SIGN_MODE_TEXTUAL”。如果您想了解更多关于它们的信息，请参阅 [ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md)。

##交易流程

最终用户发送交易的过程是:

- 决定要放入交易的消息，
- 使用 Cosmos SDK 的 `TxBuilder` 生成交易，
- 使用可用接口之一广播交易。

下一段将按此顺序描述这些组件中的每一个。

### 消息

::: 小费
不要将模块 `sdk.Msg` 与 [ABCI Messages](https://tendermint.com/docs/spec/abci/abci.html#messages) 混淆，后者定义了 Tendermint 和应用程序层之间的交互。
:::

**消息**(或`sdk.Msg`s)是特定于模块的对象，在它们所属的模块范围内触发状态转换。模块开发者通过向 Protobuf [`Msg` 服务](../building-modules/msg-services.md) 添加方法来为他们的模块定义消息，并实现相应的 `MsgServer`。

每个 `sdk.Msg` 都与一个 Protobuf [`Msg` 服务](../building-modules/msg-services.md) RPC 相关，在每个模块的 `tx.proto` 文件中定义。 SKD 应用路由器会自动将每个 `sdk.Msg` 映射到相应的 RPC。 Protobuf 为每个模块`Msg`服务生成一个`MsgServer`接口，模块开发者需要实现这个接口。
这种设计让模块开发人员承担更多责任，允许应用程序开发人员重用通用功能，而无需重复实现状态转换逻辑。

要了解有关 Protobuf `Msg` 服务以及如何实现 `MsgServer` 的更多信息，请单击 [此处](../building-modules/msg-services.md)。

虽然消息包含状态转换逻辑的信息，但事务的其他元数据和相关信息存储在“TxBuilder”和“Context”中。

### 交易生成

`TxBuilder` 接口包含与交易生成密切相关的数据，最终用户可以自由设置以生成所需的交易:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/client/tx_config.go#L32-L45

- `Msg`s，交易中包含的 [messages](#messages) 数组。
- `GasLimit`，用户选择的选项，用于计算他们需要支付多少汽油。
- `备忘录`，随交易一起发送的注释或评论。
- `FeeAmount`，用户愿意支付的最大费用金额。
- `TimeoutHeight`，直到交易有效的区块高度。
- `Signatures`，来自交易所有签名者的签名数组。

由于目前有两种签署交易的签名模式，因此也有两种`TxBuilder`的实现:

- [wrapper](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/tx/builder.go#L19-L33) 用于为 `SIGN_MODE_DIRECT` 创建交易，
- [StdTxBuilder](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdtx_builder.go#L14-L20) 用于`SIGN_MODE_LEGACY_AMINO_JSON`。

然而，`TxBuilder` 的两个实现应该远离最终用户，因为他们应该更喜欢使用总体的 `TxConfig` 接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/client/tx_config.go#L21-L30

`TxConfig` 是用于管理事务的应用程序范围的配置。最重要的是，它保存了有关是否使用“SIGN_MODE_DIRECT”或“SIGN_MODE_LEGACY_AMINO_JSON”签署每个交易的信息。通过调用`txBuilder := txConfig.NewTxBuilder()`，将使用适当的符号模式创建一个新的`TxBuilder`。

一旦 `TxBuilder` 被上面公开的 setter 正确填充，`TxConfig` 也将负责正确编码字节(同样，使用 `SIGN_MODE_DIRECT` 或 `SIGN_MODE_LEGACY_AMINO_JSON`)。这是如何使用 `TxEncoder()` 方法生成和编码交易的伪代码片段:

```go
txBuilder := txConfig.NewTxBuilder()
txBuilder.SetMsgs(...)//and other setters on txBuilder

bz, err := txConfig.TxEncoder()(txBuilder.GetTx())
//bz are bytes to be broadcasted over the network
```

### 广播交易

一旦产生交易字节，目前有三种广播方式。

#### 命令行界面

应用程序开发人员通过创建 [命令行接口](../core/cli.md)、[gRPC 和/或 REST 接口](../core/grpc_rest.md) 来创建应用程序的入口点，通常在 应用程序的`./cmd` 文件夹。 这些接口允许用户通过命令行与应用程序交互。

对于[命令行界面](../building-modules/module-interfaces.md#cli)，模块开发人员创建子命令以作为子命令添加到应用程序顶级事务命令`TxCmd`。 CLI 命令实际上将事务处理的所有步骤捆绑到一个简单的命令中:创建消息、生成事务和广播。 有关具体示例，请参阅 [与节点交互](../run-node/interact-node.md) 部分。 使用 CLI 进行的示例事务如下所示: 

```bash
simd tx send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake
```

#### gRPC

[gRPC](https://grpc.io) 在 Cosmos SDK 0.40 中引入，作为 Cosmos SDK 的 RPC 层的主要组件。 gRPC 的主要用途是在模块的 [`Query` 服务](../building-modules) 的上下文中。但是，Cosmos SDK 还公开了一些其他与模块无关的 gRPC 服务，其中之一是“Tx”服务:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/service.proto

`Tx` 服务公开了一些实用功能，例如模拟交易或查询交易，以及一种广播交易的方法。

[此处](../run-node/txs.md#programmatically-with-go) 显示了广播和模拟交易的示例。

#### 休息

每个 gRPC 方法都有其对应的 REST 端点，使用 [gRPC-gateway](https://github.com/grpc-ecosystem/grpc-gateway) 生成。因此，除了使用 gRPC，您还可以使用 HTTP 在 `POST /cosmos/tx/v1beta1/txs` 端点上广播相同的事务。

可以看到一个例子 [here](../run-node/txs.md#using-rest)

#### Tendermint RPC

上面介绍的三种方法实际上是对 Tendermint RPC `/broadcast_tx_{async,sync,commit}` 端点的更高抽象，记录在 [此处](https://docs.tendermint.com/master/rpc/#/Tx)。这意味着您可以直接使用 Tendermint RPC 端点来广播交易，如果您愿意的话。

## 下一个 {hide}

了解 [context](./context.md) {hide} 