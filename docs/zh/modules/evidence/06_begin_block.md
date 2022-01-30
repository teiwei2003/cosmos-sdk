# 开始块

## 证据处理

Tendermint 块可以包括
[证据](https://github.com/tendermint/tendermint/blob/master/docs/spec/blockchain/blockchain.md#evidence) 表明验证者是否有恶意行为。 相关信息将作为 ABCI 证据在 abci.RequestBeginBlock 中转发给应用程序，以便验证器可以受到相应的惩罚。

### 模棱两可

目前，SDK 在 ABCI `BeginBlock` 中处理两种类型的证据:

-`DuplicateVoteEvidence`，
- `LightClientAttackEvidence`。

证据模块以相同的方式处理这两种证据类型。 首先，SDK 将 Tendermint 的具体证据类型转换为 SDK 的 `Evidence` 接口，使用 `Equivocation` 作为具体类型。 

```proto
//Equivocation implements the Evidence interface.
message Equivocation {
  int64                     height            = 1;
  google.protobuf.Timestamp time              = 2;
  int64                     power             = 3;
  string                    consensus_address = 4;
}
```

对于在 `block` 中提交的某些 `Equivocation` 有效，它必须满足:

`Evidence.Timestamp >= block.Timestamp - MaxEvidenceAge`

在哪里:

- `Evidence.Timestamp` 是块中高度为 `Evidence.Height` 的时间戳
- `block.Timestamp` 是当前区块的时间戳。

如果一个区块中包含有效的“Equivocation”证据，则验证者的权益为
由 `x/slashing` 模块定义的 `SlashFractionDoubleSign` 减少(斜线)
他们的利害关系是在违规发生时，而不是在发现证据时。
我们想要“关注赌注”，即导致违规的赌注
应该削减，即使它已经被重新授权或开始解除绑定。

此外，验证者被永久监禁和墓碑化，使其不可能
验证器永远重新进入验证器集。

`Equivocation` 证据处理如下: 

```go
func (k Keeper) HandleEquivocationEvidence(ctx sdk.Context, evidence *types.Equivocation) {
	logger := k.Logger(ctx)
	consAddr := evidence.GetConsensusAddress()

	if _, err := k.slashingKeeper.GetPubkey(ctx, consAddr.Bytes()); err != nil {
		//Ignore evidence that cannot be handled.
		//
		//NOTE: We used to panic with:
		//`panic(fmt.Sprintf("Validator consensus-address %v not found", consAddr))`,
		//but this couples the expectations of the app to both Tendermint and
		//the simulator.  Both are expected to provide the full range of
		//allowable but none of the disallowed evidence types.  Instead of
		//getting this coordination right, it is easier to relax the
		//constraints and ignore evidence that cannot be handled.
		return
	}

	//calculate the age of the evidence
	infractionHeight := evidence.GetHeight()
	infractionTime := evidence.GetTime()
	ageDuration := ctx.BlockHeader().Time.Sub(infractionTime)
	ageBlocks := ctx.BlockHeader().Height - infractionHeight

	//Reject evidence if the double-sign is too old. Evidence is considered stale
	//if the difference in time and number of blocks is greater than the allowed
	//parameters defined.
	cp := ctx.ConsensusParams()
	if cp != nil && cp.Evidence != nil {
		if ageDuration > cp.Evidence.MaxAgeDuration && ageBlocks > cp.Evidence.MaxAgeNumBlocks {
			logger.Info(
				"ignored equivocation; evidence too old",
				"validator", consAddr,
				"infraction_height", infractionHeight,
				"max_age_num_blocks", cp.Evidence.MaxAgeNumBlocks,
				"infraction_time", infractionTime,
				"max_age_duration", cp.Evidence.MaxAgeDuration,
			)
			return
		}
	}

	validator := k.stakingKeeper.ValidatorByConsAddr(ctx, consAddr)
	if validator == nil || validator.IsUnbonded() {
		//Defensive: Simulation doesn't take unbonding periods into account, and
		//Tendermint might break this assumption at some point.
		return
	}

	if ok := k.slashingKeeper.HasValidatorSigningInfo(ctx, consAddr); !ok {
		panic(fmt.Sprintf("expected signing info for validator %s but not found", consAddr))
	}

	//ignore if the validator is already tombstoned
	if k.slashingKeeper.IsTombstoned(ctx, consAddr) {
		logger.Info(
			"ignored equivocation; validator already tombstoned",
			"validator", consAddr,
			"infraction_height", infractionHeight,
			"infraction_time", infractionTime,
		)
		return
	}

	logger.Info(
		"confirmed equivocation",
		"validator", consAddr,
		"infraction_height", infractionHeight,
		"infraction_time", infractionTime,
	)

	//We need to retrieve the stake distribution which signed the block, so we
	//subtract ValidatorUpdateDelay from the evidence height.
	//Note, that this *can* result in a negative "distributionHeight", up to
	//-ValidatorUpdateDelay, i.e. at the end of the
	//pre-genesis block (none) = at the beginning of the genesis block.
	//That's fine since this is just used to filter unbonding delegations & redelegations.
	distributionHeight := infractionHeight - sdk.ValidatorUpdateDelay

	//Slash validator. The `power` is the int64 power of the validator as provided
	//to/by Tendermint. This value is validator.Tokens as sent to Tendermint via
	//ABCI, and now received as evidence. The fraction is passed in to separately
	//to slash unbonding and rebonding delegations.
	k.slashingKeeper.Slash(
		ctx,
		consAddr,
		k.slashingKeeper.SlashFractionDoubleSign(ctx),
		evidence.GetValidatorPower(), distributionHeight,
	)

	//Jail the validator if not already jailed. This will begin unbonding the
	//validator if not already unbonding (tombstoned).
	if !validator.IsJailed() {
		k.slashingKeeper.Jail(ctx, consAddr)
	}

	k.slashingKeeper.JailUntil(ctx, consAddr, types.DoubleSignJailEndTime)
	k.slashingKeeper.Tombstone(ctx, consAddr)
}
```

注意，slashing、jailing 和 tombstoning 调用是通过 `x/slashing` 模块委托的
发出信息事件并最终将调用委托给 `x/staking` 模块。 查看文档
关于 [x/staking spec](/.././cosmos-sdk/x/staking/spec/02_state_transitions.md) 中的削减和监禁。