# 消息

## MsgSetWithdrawAddress

默认情况下，提现地址为委托人地址。 要更改其提款地址，委托人必须发送“MsgSetWithdrawAddress”消息。
只有当参数 `WithdrawAddrEnabled` 设置为 `true` 时，才可以更改提款地址。

提现地址不能是任何模块账户。 通过在初始化时将这些帐户添加到分发管理员的“blockedAddrs”数组中，可以阻止这些帐户被提取地址。

回复: 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.4/proto/cosmos/distribution/v1beta1/tx.proto#L29-L37

```go
func (k Keeper) SetWithdrawAddr(ctx sdk.Context, delegatorAddr sdk.AccAddress, withdrawAddr sdk.AccAddress) error
	if k.blockedAddrs[withdrawAddr.String()] {
		fail with "`{withdrawAddr}` is not allowed to receive external funds"
	}

	if !k.GetWithdrawAddrEnabled(ctx) {
		fail with `ErrSetWithdrawAddrDisabled`
	}

	k.SetDelegatorWithdrawAddr(ctx, delegatorAddr, withdrawAddr)
```

## MsgWithdrawDelegatorReward

委托人可以撤回其奖励。
在分配模块内部，该交易同时删除了先前的委托和相关的奖励，就像委托人只是简单地开始了一个相同价值的新委托。
奖励会立即从分配“ModuleAccount”发送到提款地址。
任何余数(截断的小数)都被发送到社区池。
委托的起始高度设置为当前validator周期，上一周期的引用计数递减。
提取的金额从验证器的“ValidatorOutstandingRewards”变量中扣除。

在 F1 分配中，总奖励是按每个验证者周期计算的，委托人根据他们在验证者中的股份比例获得这些奖励的一部分。
在基本的 F1 中，所有委托人在不同时期之间有权获得的总奖励按以下方式计算。
令‘R(X)’是直到‘X’期的总累积奖励除以当时质押的代币。委托人分配是`R(X) * delegator_stake`。
那么所有委托人在“A”和“B”期间质押的奖励为“(R(B) - R(A)) * 总质押”。
然而，这些计算的奖励没有考虑到削减。

考虑斜线需要迭代。
令`F(X)` 是验证者因在时间段`X` 发生的削减事件而削减的分数。
如果验证人在‘P1, ..., PN’期间被削减，其中‘A < P1’，‘PN < B’，分配模块计算个体委托人的奖励，‘T(A, B)’，如下所示: 

```
stake := initial stake
rewards := 0
previous := A
for P in P1, ..., PN`:
    rewards = (R(P) - previous) * stake
    stake = stake * F(P)
    previous = P
rewards = rewards + (R(B) - R(PN)) * stake
```

历史奖励是通过回放所有斜线然后在每一步减少委托人的赌注来追溯计算的。
最终计算的赌注相当于代表团中实际赌注的硬币，由于舍入误差而存在误差。

回复:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.4/proto/cosmos/distribution/v1beta1/tx.proto#L42-L50

## WithdrawValidatorCommission

验证者可以发送 WithdrawValidatorCommission 消息来提取他们累积的佣金。
在“BeginBlock”期间，每个区块都会计算佣金，因此无需迭代即可退出。
提取的金额从验证器的“ValidatorOutstandingRewards”变量中扣除。
只能发送整数金额。如果累积奖励有小数，则在发送提款前将金额截断，剩余部分留待稍后提款。

## FundCommunityPool

该消息直接从发送者向社区池发送硬币。

如果金额无法从发送方转移到分发模块帐户，则交易失败。 

```go
func (k Keeper) FundCommunityPool(ctx sdk.Context, amount sdk.Coins, sender sdk.AccAddress) error {
    if err := k.bankKeeper.SendCoinsFromAccountToModule(ctx, sender, types.ModuleName, amount); err != nil {
        return err
    }

	feePool := k.GetFeePool(ctx)
	feePool.CommunityPool = feePool.CommunityPool.Add(sdk.NewDecCoinsFromCoins(amount...)...)
	k.SetFeePool(ctx, feePool)

	return nil
}
```

## Common distribution operations

这些操作发生在许多不同的消息中。

### 初始化委托

每次委托变更时，奖励被撤回，委托重新初始化。
初始化委托会增加验证器周期并跟踪委托的开始周期。 

```go
//initialize starting info for a new delegation
func (k Keeper) initializeDelegation(ctx sdk.Context, val sdk.ValAddress, del sdk.AccAddress) {
   //period has already been incremented - we want to store the period ended by this delegation action
    previousPeriod := k.GetValidatorCurrentRewards(ctx, val).Period - 1

	//increment reference count for the period we're going to track
	k.incrementReferenceCount(ctx, val, previousPeriod)

	validator := k.stakingKeeper.Validator(ctx, val)
	delegation := k.stakingKeeper.Delegation(ctx, del, val)

	//calculate delegation stake in tokens
	//we don't store directly, so multiply delegation shares * (tokens per share)
	//note: necessary to truncate so we don't allow withdrawing more rewards than owed
	stake := validator.TokensFromSharesTruncated(delegation.GetShares())
	k.SetDelegatorStartingInfo(ctx, val, del, types.NewDelegatorStartingInfo(previousPeriod, stake, uint64(ctx.BlockHeight())))
}
```
