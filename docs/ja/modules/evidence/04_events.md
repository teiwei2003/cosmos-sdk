# イベント

`x/evidence`モジュールは次のイベントを発行します。

## Handlers

### MsgSubmitEvidence

| Type            | Attribute Key | Attribute Value |
| --------------- | ------------- | --------------- |
| submit_evidence | evidence_hash | {evidenceHash}  |
| message         | module        | evidence        |
| message         | sender        | {senderAddress} |
| message         | action        | submit_evidence |
