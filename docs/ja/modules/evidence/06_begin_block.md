# BeginBlock

## 証拠の処理

テンダーミントブロックには次のものが含まれます
[Evidence](https://github.com/tendermint/tendermint/blob/master/docs/spec/blockchain/blockchain.md#evidence)は、バリデーターが悪意のある動作を行ったかどうかを示します。 関連情報は、`abci.RequestBeginBlock`のABCIEvidenceとしてアプリケーションに転送されるため、バリデーターはそれに応じて罰せられます。

### 多義語の誤謬

現在、SDKはABCI`BeginBlock`内で2種類の証拠を処理します。

-`DuplicateVoteEvidence`，
-`LightClientAttackEvidence`。

エビデンスモジュールは、これら2つのエビデンスタイプを同じ方法で処理します。 まず、SDKは、具体的なタイプとして`Equivocation`を使用して、Tendermintの具体的な証拠タイプをSDKの`Evidence`インターフェイスに変換します。

```proto
//Equivocation implements the Evidence interface.
message Equivocation {
  int64                     height            = 1;
  google.protobuf.Timestamp time              = 2;
  int64                     power             = 3;
  string                    consensus_address = 4;
}
```

`block`で送信された一部の`Equivocation`が有効であるためには、次の条件を満たす必要があります。

`Evidence.Timestamp >= block.Timestamp --MaxEvidenceAge`

どこ:

- `Evidence.Timestamp`は、高さ`Evidence.Height`のブロック内のタイムスタンプです。
- `block.Timestamp`は現在のブロックタイムスタンプです。

有効な[Equivocation]証拠がブロックに含まれている場合、バリデーターの利害関係は
`x/slashing`モジュールで定義されているように`SlashFractionDoubleSign`によって削減(スラッシュ)されます
証拠が発見されたときではなく、違反が発生したときの彼らの利害関係について。
[賭け金をたどる]、つまり違反の原因となった賭け金を追跡したい
その後、再委任されたり、結合が解除されたりした場合でも、スラッシュする必要があります。

さらに、バリデーターは永久に投獄され、墓石に入れられて、それを不可能にします
バリデーターセットを再入力するためのバリデーター。

`Equivocation`の証拠は次のように処理されます。

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

	//Slash validator. The`power`is the int64 power of the validator as provided
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

スラッシュ、ジェイル、およびトゥームストーニングの呼び出しは、`x/slashing`モジュールを介して委任されることに注意してください
これは有益なイベントを発行し、最後に呼び出しを`x/staking`モジュールに委任します。 ドキュメントを参照してください
[x/staking spec](/.././cosmos-sdk/x/staking/spec/02_state_transitions.md)でのスラッシュとジェイルについて。