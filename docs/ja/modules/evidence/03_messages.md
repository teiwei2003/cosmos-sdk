# 消息

## MsgSubmitEvidence

证据是通过“MsgSubmitEvidence”消息提交的: 

```protobuf
// MsgSubmitEvidence represents a message that supports submitting arbitrary
// Evidence of misbehavior such as equivocation or counterfactual signing.
message MsgSubmitEvidence {
  string              submitter = 1;
  google.protobuf.Any evidence  = 2;
}
```

注意，一个 `MsgSubmitEvidence` 消息的 `Evidence` 必须有一个对应的
`Handler` 注册到 `x/evidence` 模块的 `Router` 以便处理
并正确路由。

鉴于 `Evidence` 注册到相应的 `Handler`，它被处理
如下: 

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

首先，必须不存在完全相同的有效提交“证据”
类型。 其次，`Evidence` 被路由到`Handler` 并执行。 最后，
如果在处理“证据”时没有错误，则会发出一个事件并将其持久化以保持状态。 
