# 前处理程序

`x/auth` 模块目前没有自己的事务处理程序，但确实公开了特殊的 `AnteHandler`，用于对事务执行基本的有效性检查，以便它可以被抛出内存池。
根据 [ADR 010](https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-alpha1/docs/架构/adr-010-modular-antehandler.md)。

请注意，在 `CheckTx` 和 `DeliverTx` 上都调用了 `AnteHandler`，因为 Tendermint 提议者目前能够将失败的 `CheckTx` 包含在他们提议的区块交易中。

## 装饰器

auth 模块提供了 `AnteDecorator`s，它们按以下顺序递归地链接到单个 `AnteHandler` 中:

- `SetUpContextDecorator`:在 `Context` 中设置 `GasMeter`，并用 defer 子句包装下一个 `AnteHandler`，以从 `AnteHandler` 链中的任何下游 `OutOfGas` 恐慌中恢复，并返回有关所提供气体的信息的错误和使用的气体。

- `RejectExtensionOptionsDecorator`:拒绝所有可以选择包含在 protobuf 事务中的扩展选项。

- `MempoolFeeDecorator`:在`CheckTx`期间检查`tx`费用是否高于本地内存池`minFee`参数。

- `ValidateBasicDecorator`:调用`tx.ValidateBasic` 并返回任何非零错误。

- `TxTimeoutHeightDecorator`:检查`tx` 高度超时。

- `ValidateMemoDecorator`:使用应用程序参数验证 `tx` 备忘录并返回任何非零错误。

- `ConsumeGasTxSizeDecorator`:根据应用程序参数消耗与`tx` 大小成比例的gas。

- `DeductFeeDecorator`:从 `tx` 的第一个签名者中扣除 `FeeAmount`。如果启用了 `x/feegrant` 模块并设置了费用授予者，它会从费用授予者帐户中扣除费用。

- `SetPubKeyDecorator`:从`tx` 的签名者设置公钥，该签名者还没有在状态机和当前上下文中保存其相应的公钥。

- `ValidateSigCountDecorator`:根据应用参数验证 `tx` 中的签名数量。

- `SigGasConsumeDecorator`:为每个签名消耗参数定义的气体量。这需要在上下文中为所有签名者设置公钥作为“SetPubKeyDecorator”的一部分。

- `SigVerificationDecorator`:验证所有签名是否有效。这需要在上下文中为所有签名者设置公钥作为“SetPubKeyDecorator”的一部分。

- `IncrementSequenceDecorator`:为每个签名者增加账户序列以防止重放攻击。 