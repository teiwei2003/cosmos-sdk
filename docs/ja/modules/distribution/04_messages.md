# 情報

## MsgSetWithdrawAddress

デフォルトでは、引き出しアドレスはプリンシパルのアドレスです。 引き出しアドレスを変更するには、プリンシパルは「MsgSetWithdrawAddress」メッセージを送信する必要があります。
パラメータ `WithdrawAddrEnabled`が` true`に設定されている場合にのみ、引き出しアドレスを変更できます。

引き出しアドレスをモジュールアカウントにすることはできません。 初期化中にこれらのアカウントをディストリビューション管理者の「blockedAddrs」アレイに追加することにより、これらのアカウントが抽出されないようにすることができます。

返事：

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

委任者はその報酬を引き出すことができます。
配布モジュールの内部では、このトランザクションは、委任者が同じ値の新しい委任を開始した場合と同じように、関連する報酬とともに以前の委任を同時に削除します。
報酬は、ディストリビューション `ModuleAccount`から引き出しアドレスにすぐに送信されます。
余り（切り捨てられた小数）は、コミュニティプールに送信されます。
委任の開始高さは現在のバリデーター期間に設定され、前の期間の参照カウントがデクリメントされます。
引き出された金額は、バリデーターの `ValidatorOutstandingRewards`変数から差し引かれます。

F1ディストリビューションでは、合計報酬はバリデーター期間ごとに計算され、委任者はバリデーターへの出資に比例してそれらの報酬の一部を受け取ります。
基本的なF1では、すべての委任者が期間の間に受けることができる報酬の合計は、次の方法で計算されます。
`R（X）`を、期間 `X`までに累積された報酬の合計を、その時点で賭けられたトークンで割ったものとします。委任者の割り当ては `R（X）* delegator_stake`です。
その場合、期間 `A`と` B`の間の賭けに対するすべての委任者の報酬は `（R（B）-R（A））*合計賭け金`です。
ただし、これらの計算された報酬はスラッシュを考慮していません。

スラッシュを考慮に入れるには、反復が必要です。
`F（X）`を、期間 `X`で発生したスラッシュイベントに対してバリデーターがスラッシュされる割合とします。
バリデーターが期間 `P1、...、PN`でスラッシュされた場合、` A <P1`、 `PN <B`の場合、配布モジュールは次のように個々の委任者の報酬` T（A、B） `を計算します。

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

過去の報酬は、すべてのスラッシュを再生し、各ステップで委任者の賭け金を減らすことによって遡及的に計算されます。
最終的に計算された賭け金は、丸め誤差による誤差のある、代表団の実際の賭け金と同等です。

応答：

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.4/proto/cosmos/distribution/v1beta1/tx.proto#L42-L50

## WithdrawValidatorCommission

バリデーターは、WithdrawValidatorCommissionメッセージを送信して、累積されたコミッションを引き出すことができます。
コミッションは `BeginBlock`の間にすべてのブロックで計算されるので、撤回するために反復は必要ありません。
引き出された金額は、バリデーターの `ValidatorOutstandingRewards`変数から差し引かれます。
整数の金額のみを送信できます。 累積されたアワードに小数がある場合、引き出しが送信される前に金額が切り捨てられ、残りは後で引き出しられるように残されます。。

## FundCommunityPool

このメッセージは、送信者からコミュニティプールにコインを直接送信します。

金額を送信者から配信モジュールアカウントに転送できない場合、トランザクションは失敗します。 

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

これらの操作は、さまざまなメッセージの中で行われます。

### 委任を初期化する

委任が変更されるたびに、報酬が取り消され、委任が再初期化されます。
委任を初期化すると、バリデーター期間が増分され、委任の開始期間が追跡されます。 

```go
// initialize starting info for a new delegation
func (k Keeper) initializeDelegation(ctx sdk.Context, val sdk.ValAddress, del sdk.AccAddress) {
    // period has already been incremented - we want to store the period ended by this delegation action
    previousPeriod := k.GetValidatorCurrentRewards(ctx, val).Period - 1

	// increment reference count for the period we're going to track
	k.incrementReferenceCount(ctx, val, previousPeriod)

	validator := k.stakingKeeper.Validator(ctx, val)
	delegation := k.stakingKeeper.Delegation(ctx, del, val)

	// calculate delegation stake in tokens
	// we don't store directly, so multiply delegation shares * (tokens per share)
	// note: necessary to truncate so we don't allow withdrawing more rewards than owed
	stake := validator.TokensFromSharesTruncated(delegation.GetShares())
	k.SetDelegatorStartingInfo(ctx, val, del, types.NewDelegatorStartingInfo(previousPeriod, stake, uint64(ctx.BlockHeight())))
}
```
