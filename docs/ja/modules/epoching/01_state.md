# 状態

## メッセージキュー

メッセージは、各エポックの終わりに実行されるようにキューに入れられます。 キューに入れられたメッセージにはエポック番号があり、エポック番号ごとに、キューが繰り返されて各メッセージが実行されます。

### メッセージキュー

各モジュールには、そのモジュールに固有の固有のメッセージキューがあります。

## アクション

モジュールは、`sdk.Msg`インターフェースを実装するメッセージを追加します。 これらのメッセージは後で実行されます(次のエポックの終わり)。

```go
type Msg interface {
  proto.Message

 //Return the message type.
 //Must be alphanumeric or empty.
  Route() string

 //Returns a human-readable string for the message, intended for utilization
 //within tags
  Type() string

 //ValidateBasic does a simple validation check that
 //doesn't require access to any other information.
  ValidateBasic() error

 //Get the canonical byte representation of the Msg.
  GetSignBytes() []byte

 //Signers returns the addrs of signers that must sign.
 //CONTRACT: All signatures must be present to be valid.
 //CONTRACT: Returns addrs in some deterministic order.
  GetSigners() []AccAddress
 }
```

## バッファメッセージのエクスポート/インポート

現在、`x/epiching`モジュールは、エポック番号なしですべてのバッファリングされたメッセージをエクスポートするために実装されています。 状態をインポートすると、バッファリングされたメッセージは現在のエポックに保存され、現在のエポックの最後に実行されます。

## 作成トランザクション

ジェネシストランザクションを実行した後にエポックを実行して、ノードが起動する直前の変更を確認します。

## エポックで実行

- エポックメッセージを実行してみてください
- 成功した場合は、そのまま変更します
- 失敗した場合は、ハンドラーで追加のリカバリ操作を実行してみてください(例:EpochDelegationPoolデポジット)
- 回復に失敗した場合、パニック