# イベント

crisisモジュールは、次のイベントを発行します。 

## Handlers

### MsgVerifyInvariance

| Type      | Attribute Key | Attribute Value  |
|-----------|---------------|------------------|
| invariant | route         | {invariantRoute} |
| message   | module        | crisis           |
| message   | action        | verify_invariant |
| message   | sender        | {senderAddress}  |
