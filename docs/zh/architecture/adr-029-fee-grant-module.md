# ADR 029:费用补助模块

## 变更日志

- 2020/08/18:初稿
- 2021/05/05:删除了基于高度的过期支持和简化的命名。

## 地位

公认

## 语境

为了进行区块链交易，签名账户必须拥有足够的正确面额余额
以支付费用。有几类交易需要维护一个有足够费用的钱包是一种
采用的障碍。

例如，当设置了适当的权限时，有人可能会暂时将提案投票的能力委托给
存储在手机上的“刻录机”帐户，只有最低限度的安全性。

其他用例包括工人跟踪供应链中的物品或农民提交现场数据进行分析
或合规目的。

对于所有这些用例，通过消除对这些帐户的需求，用户体验将得到显着增强
保持适当的费用平衡。如果我们想实现某些东西的企业采用，则尤其如此
比如供应链跟踪。

虽然一种解决方案是提供一项服务以适当的费用自动填充这些帐户，但更好的用户体验
将通过允许这些帐户从具有适当支出限制的公共费用池帐户中提取来提供。
单个池将减少进行大量小型“填充”交易的流失，并且更有效地利用杠杆
设置池的组织的资源。

## 决定

作为一种解决方案，我们提出了一个模块 `x/feegrant`，它允许一个帐户，即“授予者”授予另一个帐户，即“被授予者”
在某些明确定义的限额内将授予人的账户余额用于费用的津贴。

费用限额由可扩展的`FeeAllowanceI` 接口定义: 

```go
type FeeAllowanceI {
 //Accept can use fee payment requested as well as timestamp of the current block
 //to determine whether or not to process this. This is checked in
 //Keeper.UseGrantedFees and the return values should match how it is handled there.
 //
 //If it returns an error, the fee payment is rejected, otherwise it is accepted.
 //The FeeAllowance implementation is expected to update it's internal state
 //and will be saved again after an acceptance.
 //
 //If remove is true (regardless of the error), the FeeAllowance will be deleted from storage
 //(eg. when it is used up). (See call to RevokeFeeAllowance in Keeper.UseGrantedFees)
  Accept(ctx sdk.Context, fee sdk.Coins, msgs []sdk.Msg) (remove bool, err error)

 //ValidateBasic should evaluate this FeeAllowance for internal consistency.
 //Don't allow negative amounts, or negative periods for example.
  ValidateBasic() error
}
```

Two basic fee allowance types, `BasicAllowance` and `PeriodicAllowance` are defined to support known use cases:

```proto
//BasicAllowance implements FeeAllowanceI with a one-time grant of tokens
//that optionally expires. The delegatee can use up to SpendLimit to cover fees.
message BasicAllowance {
 //spend_limit specifies the maximum amount of tokens that can be spent
 //by this allowance and will be updated as tokens are spent. If it is
 //empty, there is no spend limit and any amount of coins can be spent.
  repeated cosmos_sdk.v1.Coin spend_limit = 1;

 //expiration specifies an optional time when this allowance expires
  google.protobuf.Timestamp expiration = 2;
}

//PeriodicAllowance extends FeeAllowanceI to allow for both a maximum cap,
//as well as a limit per time period.
message PeriodicAllowance {
  BasicAllowance basic = 1;

 //period specifies the time duration in which period_spend_limit coins can
 //be spent before that allowance is reset
  google.protobuf.Duration period = 2;

 //period_spend_limit specifies the maximum number of coins that can be spent
 //in the period
  repeated cosmos_sdk.v1.Coin period_spend_limit = 3;

 //period_can_spend is the number of coins left to be spent before the period_reset time
  repeated cosmos_sdk.v1.Coin period_can_spend = 4;

 //period_reset is the time at which this period resets and a new one begins,
 //it is calculated from the start time of the first transaction after the
 //last period ended
  google.protobuf.Timestamp period_reset = 5;
}

```

Allowances can be granted and revoked using `MsgGrantAllowance` and `MsgRevokeAllowance`:

```proto
//MsgGrantAllowance adds permission for Grantee to spend up to Allowance
//of fees from the account of Granter.
message MsgGrantAllowance {
     string granter = 1;
     string grantee = 2;
     google.protobuf.Any allowance = 3;
 }

//MsgRevokeAllowance removes any existing FeeAllowance from Granter to Grantee.
 message MsgRevokeAllowance {
     string granter = 1;
     string grantee = 2;
 }
```

In order to use allowances in transactions, we add a new field `granter` to the transaction `Fee` type:

```proto
package cosmos.tx.v1beta1;

message Fee {
  repeated cosmos.base.v1beta1.Coin amount = 1;
  uint64 gas_limit = 2;
  string payer = 3;
  string granter = 4;
}
```

`granter` 必须要么留空，要么必须对应于已授予
费用支付者的费用津贴(第一个签名者或“payer”字段的值)。

将创建一个名为“DeductGrantedFeeDecorator”的新“AnteDecorator”以处理与“fee_payer”的交易
根据费用津贴设置并正确扣除费用。 

## Consequences

### Positive

- 改进了用户体验，适用于仅仅为了费用而维护账户余额很麻烦的用例

### Negative

### Neutral

- 改进了用户体验，适用于仅仅为了费用而维护账户余额很麻烦的用例

## References

- 描述初始工作的博客文章:https://medium.com/regen-network/hacking-the-cosmos-cosmwasm-and-key-management-a08b9f561d1b
- 初始公开规范:https://gist.github.com/aaronc/b60628017352df5983791cad30babe56
- 来自 B-harvest 的原始子密钥提案影响了这个设计:https://github.com/cosmos/cosmos-sdk/issues/4480 