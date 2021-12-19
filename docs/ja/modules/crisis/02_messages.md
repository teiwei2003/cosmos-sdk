# 消息

在本节中，我们将描述危机消息的处理以及
状态的相应更新。

## MsgVerifyInvariant

可以使用“MsgVerifyInvariant”消息检查区块链不变量。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc7/proto/cosmos/crisis/v1beta1/tx.proto#L14-L22

如果出现以下情况，此消息预计会失败:

- 发件人没有足够的硬币来支付固定费用
- 不变路由未注册

此消息检查提供的不变量，如果不变量被破坏
恐慌，停止区块链。 如果不变量被破坏，则恒定费用为
从未扣除，因为交易从未提交到区块(相当于
被退还)。 但是，如果不变量不破，恒定费用将
不予退还。 