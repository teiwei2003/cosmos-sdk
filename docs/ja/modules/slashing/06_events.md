# イベント

スラッシュモジュールは、次のイベントを発行します。 

## MsgServer

### MsgUnjail

| Type    | Attribute Key | Attribute Value |
| ------- | ------------- | --------------- |
| message | module        | slashing        |
| message | sender        | {validatorAddress} |

## Keeper

## BeginBlocker: HandleValidatorSignature

| Type  | Attribute Key | Attribute Value             |
| ----- | ------------- | --------------------------- |
| slash | address       | {validatorConsensusAddress} |
| slash | power         | {validatorPower}            |
| slash | reason        | {slashReason}               |
| slash | jailed [0]    | {validatorConsensusAddress} |
| slash | burned coins  | {sdk.Int}                   |

- [0] Only included if the validator is jailed.

| Type     | Attribute Key | Attribute Value             |
| -------- | ------------- | --------------------------- |
| liveness | address       | {validatorConsensusAddress} |
| liveness | missed_blocks | {missedBlocksCounter}       |
| liveness | height        | {blockHeight}               |

### Slash

+ `HandleValidatorSignature`の` "slash"`イベントと同じですが、`jailed`属性はありません

### Jail

| Type  | Attribute Key | Attribute Value    |
| ----- | ------------- | ------------------ |
| slash | jailed        | {validatorAddress} |
