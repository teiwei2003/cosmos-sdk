# スターティングブロック

## 活力追跡

各ブロックの開始時に、各バリデーターの `ValidatorSigningInfo`を更新し、スライディングウィンドウ上でそれらが活性しきい値を下回ったかどうかを確認します。
このスライディングウィンドウは、 `SignedBlocksWindow`と
このウィンドウのインデックスは、バリデーターの `ValidatorSigningInfo`にある` IndexOffset`によって決定されます。 処理されたブロックごとに、バリデーターが署名したかどうかに関係なく、 `IndexOffset`がインクリメントされます。 インデックスが決定されると、それに応じて `MissedBlocksBitArray`と` MissedBlocksCounter`が更新されます。

最後に、バリデーターが活性しきい値を下回ったかどうかを判断するために、欠落したブロックの最大数 `maxMissed`をフェッチします。これは` SignedBlocksWindow-(MinSignedPerWindow * SignedBlocksWindow) `であり、活性を判断できる最小の高さです。 `minHeight`。 現在のブロックが `minHeight`より大きく、バリデーターの` MissedBlocksCounter`がより大きい場合
`maxMissed`は、` SlashFractionDowntime`によってスラッシュされ、 `DowntimeJailDuration`のために投獄され、次の値がリセットされます。
`MissedBlocksBitArray`、` MissedBlocksCounter`、および `IndexOffset`。

**注**:活気のあるスラッシュは、墓を踏みにじることにはなりません。 

```go
height := block.Height

for vote in block.LastCommitInfo.Votes {
  signInfo := GetValidatorSigningInfo(vote.Validator.Address)

 //This is a relative index, so we counts blocks the validator SHOULD have
 //signed. We use the 0-value default signing info if not present, except for
 //start height.
  index := signInfo.IndexOffset % SignedBlocksWindow()
  signInfo.IndexOffset++

 //Update MissedBlocksBitArray and MissedBlocksCounter. The MissedBlocksCounter
 //just tracks the sum of MissedBlocksBitArray. That way we avoid needing to
 //read/write the whole array each time.
  missedPrevious := GetValidatorMissedBlockBitArray(vote.Validator.Address, index)
  missed := !signed

  switch {
  case !missedPrevious && missed:
   //array index has changed from not missed to missed, increment counter
    SetValidatorMissedBlockBitArray(vote.Validator.Address, index, true)
    signInfo.MissedBlocksCounter++

  case missedPrevious && !missed:
   //array index has changed from missed to not missed, decrement counter
    SetValidatorMissedBlockBitArray(vote.Validator.Address, index, false)
    signInfo.MissedBlocksCounter--

  default:
   //array index at this index has not changed; no need to update counter
  }

  if missed {
   //emit events...
  }

  minHeight := signInfo.StartHeight + SignedBlocksWindow()
  maxMissed := SignedBlocksWindow() - MinSignedPerWindow()

 //If we are past the minimum height and the validator has missed too many
 //jail and slash them.
  if height > minHeight && signInfo.MissedBlocksCounter > maxMissed {
    validator := ValidatorByConsAddr(vote.Validator.Address)

   //emit events...

   //We need to retrieve the stake distribution which signed the block, so we
   //subtract ValidatorUpdateDelay from the block height, and subtract an
   //additional 1 since this is the LastCommit.
   //
   //Note, that this CAN result in a negative "distributionHeight" up to
   //-ValidatorUpdateDelay-1, i.e. at the end of the pre-genesis block (none) = at the beginning of the genesis block.
   //That's fine since this is just used to filter unbonding delegations & redelegations.
    distributionHeight := height - sdk.ValidatorUpdateDelay - 1

    Slash(vote.Validator.Address, distributionHeight, vote.Validator.Power, SlashFractionDowntime())
    Jail(vote.Validator.Address)

    signInfo.JailedUntil = block.Time.Add(DowntimeJailDuration())

   //We need to reset the counter & array so that the validator won't be
   //immediately slashed for downtime upon rebonding.
    signInfo.MissedBlocksCounter = 0
    signInfo.IndexOffset = 0
    ClearValidatorMissedBlockBitArray(vote.Validator.Address)
  }

  SetValidatorSigningInfo(vote.Validator.Address, signInfo)
}
```
