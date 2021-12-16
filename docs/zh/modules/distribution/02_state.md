# 状态

## 费用池

所有用于分发的全局跟踪参数都存储在
`费用池`。 收集奖励并添加到奖励池中
从这里分发给验证者/委托者。

请注意，奖励池包含十进制硬币(`DecCoins`)以允许
从通货膨胀等操作中获得一小部分硬币。
当硬币从池中分配时，它们会被截断回
`sdk.Coins` 非十进制。

- 费用池:`0x00 -> ProtocolBuffer(FeePool)` 

```go
// coins with decimal
type DecCoins []DecCoin

type DecCoin struct {
    Amount sdk.Dec
    Denom  string
}
```

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/distribution/v1beta1/distribution.proto#L94-L101

## 验证器分发

相关验证器的验证器分布信息每次都会更新:

1. 更新了验证者的委托数量，
2. 验证者成功提出区块并获得奖励，
3. 任何委托人退出验证人，或
4. 验证者撤回其佣金。 

- ValidatorDistInfo: `0x02 | ValOperatorAddrLen (1 byte) | ValOperatorAddr -> ProtocolBuffer(validatorDistribution)`

```go
type ValidatorDistInfo struct {
    OperatorAddress     sdk.AccAddress
    SelfBondRewards     sdk.DecCoins
    ValidatorCommission types.ValidatorAccumulatedCommission
}
```

## 委托分发

每个委托分布只需要记录它最后的高度
撤回费用。 因为代表团每次都必须撤回费用
属性改变(又名绑定令牌等)它的属性将保持不变
并且委托人的_accumulation_因子可以被动计算
只有最后一次提款的高度及其当前属性。

- 委托分布信息:`0x02 | DelegatorAddrLen (1 字节) | 委托人地址 | ValOperatorAddrLen (1 字节) | ValOperatorAddr -> ProtocolBuffer(delegatorDist)` 

```go
type DelegationDistInfo struct {
    WithdrawalHeight int64    // last time this delegation withdrew rewards
}
```
