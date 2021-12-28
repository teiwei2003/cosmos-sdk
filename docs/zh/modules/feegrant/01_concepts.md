# 概念

## 授予

`Grant` 存储在 KVStore 中以记录具有完整上下文的授权。每个赠款都将包含“授予者”、“受赠者”以及授予的“津贴”类型。 `granter` 是一个账户地址，它允许 `grantee`(受益人账户地址)支付部分或全部 `grantee` 的交易费用。 `allowance` 定义了授予 `grantee` 什么样的费用津贴(`BasicAllowance` 或 `PeriodicAllowance`，见下文)。 `allowance` 接受一个实现 `FeeAllowanceI` 的接口，编码为 `Any` 类型。 “受让人”和“受让人”只能有一项现有的费用补助，不允许自行补助。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L75-L81

`FeeAllowanceI` 看起来像:

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/x/feegrant/fees.go#L9-L32

## 费用津贴类型

目前有两种类型的费用津贴:

-`基本津贴`
- `PeriodicAllowance`

## 基本津贴

`BasicAllowance` 是允许 `grantee` 使用来自 `granter` 帐户的费用。如果“spend_limit”或“expiration”中的任何一个达到其限制，则该授权将从状态中删除。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L13-L26

- `spend_limit` 是允许从 `granter` 帐户使用的硬币限制。如果它为空，则假定没有支出限制，`grantee` 可以在到期前使​​用来自 `granter` 帐户地址的任意数量的可用代币。

- `expiration` 指定该配额到期的可选时间。如果该值留空，则授权没有到期。

- 当使用“spend_limit”和“expiration”的空值创建授权时，它仍然是有效的授权。它不会限制 `grantee` 使用来自 `granter` 的任意数量的令牌，并且不会有任何过期时间。限制“受赠者”的唯一方法是撤销授予。

## 定期津贴

`PeriodicAllowance` 是上述期间的重复费用津贴，我们可以提及补助金何时到期以及何时可以重置。我们还可以定义在提到的时间段内可以使用的最大硬币数量。

+++ https://github.com/cosmos/cosmos-sdk/blob/691032b8be0f7539ec99f8882caecefc51f33d1f/proto/cosmos/feegrant/v1beta1/feegrant.proto#L28-L73

- `basic` 是 `BasicAllowance` 的实例，对于定期费用津贴是可选的。如果为空，则赠款将没有“expiration”和“spend_limit”。

- `period` 是具体的时间段，每个时间段过去后，`period_spend_limit` 将被重置。

- `period_spend_limit` 指定在该期间内可以花费的最大硬币数量。

- `period_can_spend` 是在 period_reset 时间之前剩余的硬币数量。

- `period_reset` 会记录下一个周期重置应该发生的时间。

## FeeGranter 标志

`feegrant` 模块为 CLI 引入了一个 `FeeGranter` 标志，以便与费用授予者执行交易。设置此标志后，`clientCtx` 将为通过 CLI 生成的交易附加授予者帐户地址。

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/client/cmd.go#L224-L235

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/client/tx/tx.go#L120

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/x/auth/tx/builder.go#L268-L277

+++ https://github.com/cosmos/cosmos-sdk/blob/d97e7907f176777ed8a464006d360bb3e1a223e4/proto/cosmos/tx/v1beta1/tx.proto#L160-L181

示例命令:

```Go
./simd tx gov submit-proposal --title="Test Proposal" --description="My Awesome Proposal" --type="Text" --from validator-key --fee-granter=cosmos1xh44hxt7spr67hqaa7nyx5gnutrz5fraw6grxn --chain-id =testnet --fees="10stake"
```

## 授予的费用减免

费用从 `x/auth` 前处理程序的赠款中扣除。要了解有关 ante 处理程序如何工作的更多信息，请阅读 [Auth Module AnteHandlers Guide](../../auth/spec/03_antehandlers.md)。

## 气体

为了防止 DoS 攻击，使用过滤的 `x/feegrant` 会产生 gas。 SDK 必须确保“被授予者”的交易都符合“授予者”设置的过滤器。 SDK 通过迭代过滤器中允许的消息并为每个过滤的消息收取 10 gas 来做到这一点。然后，SDK 将迭代“受赠者”发送的消息，以确保消息符合过滤器，同时每条消息收取 10 gas。如果 SDK 发现不符合过滤器的消息，将停止迭代并使事务失败。

**警告**:根据授予的津贴收取天然气费用。在使用您的配额发送交易之前，请确保您的消息符合过滤器(如果有)。