# メッセージ

## MsgSubmitEvidence

証拠は、「MsgSubmitEvidence」メッセージを介して送信されます。

```protobuf
// MsgSubmitEvidence represents a message that supports submitting arbitrary
// Evidence of misbehavior such as equivocation or counterfactual signing.
message MsgSubmitEvidence {
  string              submitter = 1;
  google.protobuf.Any evidence  = 2;
}
```

`MsgSubmitEvidence`メッセージの`Evidence`には対応するものが必要であることに注意してください
処理するために `x / evidence`モジュールの`Router`に登録された `Handler`
正しくルーティングされます。

`Evidence`が対応する`Handler`に登録されている場合、それは処理されます
次のように：

```go
func SubmitEvidence(ctx Context, evidence Evidence) error {
  if _, ok := GetEvidence(ctx, evidence.Hash()); ok {
    return sdkerrors.Wrap(types.ErrEvidenceExists, evidence.Hash().String())
  }
  if !router.HasRoute(evidence.Route()) {
    return sdkerrors.Wrap(types.ErrNoEvidenceHandlerExists, evidence.Route())
  }

  handler := router.GetRoute(evidence.Route())
  if err := handler(ctx, evidence); err != nil {
    return sdkerrors.Wrap(types.ErrInvalidEvidence, err.Error())
  }

  ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			types.EventTypeSubmitEvidence,
			sdk.NewAttribute(types.AttributeKeyEvidenceHash, evidence.Hash().String()),
		),
	)

  SetEvidence(ctx, evidence)
  return nil
}
```

まず、まったく同じの有効な提出された`Evidence`がすでに存在していてはなりません
タイプ。 次に、`Evidence`は`Handler`にルーティングされ、実行されます。 ついに、
`Evidence`の処理にエラーがない場合、イベントが発行され、状態に保持されます。 
