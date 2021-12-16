# 消息

在本节中，我们描述了 staking 消息的处理和相应的状态更新。每个消息指定的所有创建/修改状态对象都在 [state](./02_state_transitions.md) 部分中定义。

## MsgCreateValidator

使用 `MsgCreateValidator` 消息创建验证器。
必须使用运营商的初始委托来创建验证器。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L16-L17

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L35-L51

如果出现以下情况，此消息预计会失败:

- 另一个具有此操作员地址的验证器已注册
- 另一个使用此公钥的验证器已经注册
- 初始自委托代币的面额未指定为绑定面额
- 佣金参数有问题，即:
    - `MaxRate` 是 > 1 或 < 0
    - 初始 `Rate` 为负数或 > `MaxRate`
    - 初始 `MaxChangeRate` 为负数或 > `MaxRate`
- 描述字段太大

此消息在适当的索引处创建并存储“验证器”对象。
此外，通过初始代币委托进行自我委托
令牌`委托`。验证器始终以未绑定状态开始，但可能已绑定
在第一个结束块中。

## MsgEditValidator

验证器的 `Description`、`CommissionRate` 可以使用
`MsgEditValidator` 消息。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L19-L20

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L56-L76

如果出现以下情况，此消息预计会失败:

- 初始`CommissionRate` 为负数或> `MaxRate`
- `CommissionRate` 已在过去 24 小时内更新
- `CommissionRate` 是 > `MaxChangeRate`
- 描述字段太大

此消息存储更新的“验证器”对象。

## MsgDelegate

在此消息中，委托人提供硬币，并作为回报收到
一定数量的验证者(新创建的)委托人股份
分配给`Delegation.Shares`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L22-L24

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L81-L90

如果出现以下情况，此消息预计会失败:

- 验证器不存在
- `Amount` `Coin` 的面额与 `params.BondDenom` 定义的面额不同
- 汇率无效，这意味着验证者没有代币(由于削减)但有流通股
- 委托金额低于允许的最低委托金额

如果提供的地址的现有“委托”对象还没有
存在然后它被创建作为此消息的一部分，否则现有的
`Delegation` 更新为包括新收到的共享。 

委托人以当前汇率接收新铸造的股票。
汇率是验证器中现有股份的数量除以
当前委托代币的数量。

验证器在`ValidatorByPower`索引中更新，委托是
在“验证器”索引中的验证器对象中进行跟踪。

可以委托给被监禁的验证者，唯一的区别是
不会被添加到力量指数中，直到它被释放。

![委托序列](../../../docs/uml/svg/delegation_sequence.svg)

## MsgUndelegate

`MsgUndelegate` 消息允许委托人取消委托他们的代币
验证器。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L30-L32

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L112-L121

此消息返回一个响应，其中包含取消委派的完成时间:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L123-L126

如果出现以下情况，此消息预计会失败:

- 代表团不存在
- 验证器不存在
- 代表团的份额少于“金额”的份额
- 现有的“UnbondingDelegation”具有由“params.MaxEntries”定义的最大条目
- `Amount` 的面额与 `params.BondDenom` 定义的面额不同

处理此消息时，会发生以下操作:

- 验证者的`DelegatorShares`和委托的`Shares`都被消息`SharesAmount`减少
- 计算股票的代币价值，删除验证器中持有的代币数量
- 使用那些已删除的令牌，如果验证器是:
    - `Bonded` - 将它们添加到 `UnbondingDelegation` 中的条目(如果它不存在，则创建 `UnbondingDelegation`)，完成时间从当前时间开始一个完整的解绑期。更新池份额以减少 BondedTokens 并通过份额的代币价值增加 NotBondedTokens。
    - `Unbonding` - 将它们添加到 `UnbondingDelegation` 中的条目(如果它不存在，则创建 `UnbondingDelegation`)与验证器(`UnbondingMinTime`)具有相同的完成时间。
    - `Unbonded` - 然后向硬币发送消息`DelegatorAddr`
- 如果委托中没有更多的`Shares`，那么委托对象将从存储中删除
    - 在这种情况下，如果委托是验证者的自我委托，那么也会监禁验证者。

![解绑序列](../../../docs/uml/svg/unbond_sequence.svg)

## MsgBeginRedelegate

重新委托命令允许委托人立即切换验证人。一次
解绑期已过，转授权自动完成
终结者。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L26-L28

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L95-L105

此消息返回一个响应，其中包含重新授权的完成时间:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/tx.proto#L107-L110

如果出现以下情况，此消息预计会失败:

- 代表团不存在
- 源或目标验证器不存在
- 代表团的份额少于“金额”的份额
- 源验证器有一个未成熟的接收重新委托(又名，重新委托可能是可传递的)
- 现有的“重新委托”具有由“params.MaxEntries”定义的最大条目
- `Amount` `Coin` 的面额与 `params.BondDenom` 定义的面额不同

处理此消息时，会发生以下操作:

- 源验证器的`DelegatorShares`和委托`Shares`都被消息`SharesAmount`减少
- 计算股票的代币价值，删除源验证器中持有的代币数量。
- 如果源验证器是:
    - `Bonded` - 将一个条目添加到 `Redelegation`(如果它不存在，则创建 `Redelegation`)，完成时间从当前时间开始一个完整的解除绑定期。更新池份额以减少 BondedTokens 并通过份额的代币价值增加 NotBondedTokens(然而，这可能在下一步中有效地逆转)。
    - `Unbonding` - 将一个条目添加到 `Redelegation`(如果它不存在，则创建 `Redelegation`)，其完成时间与验证器(`UnbondingMinTime`)相同。
    - `Unbonded` - 这一步不需要任何操作
- 将代币价值委托给目标验证器，可能会将代币移回绑定状态。
- 如果源委托中没有更多的`Shares`，那么源委托对象将从存储中删除
    - 在这种情况下，如果委托是验证者的自我委托，那么也会监禁验证者。

![开始重新委托序列](../../../docs/uml/svg/begin_redelegation_sequence.svg) 