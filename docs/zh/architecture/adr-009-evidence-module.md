# ADR 009:证据模块

## 变更日志

- 2019 年 7 月 31 日:初稿
- 2019 年 10 月 24 日:初步实施

## 地位

公认

## 语境

为了支持构建高度安全、健壮和可互操作的区块链
应用程序，Cosmos SDK 公开一种机制至关重要，其中任意
可以提交、评估和验证证据，从而达成一些共识
对验证者的任何不当行为的惩罚，例如模棱两可(双重投票)，
未绑定时签名，签名不正确的状态转换(将来)等。
此外，这种机制对于任何
[IBC](https://github.com/cosmos/ics/blob/master/ibc/2_IBC_ARCHITECTURE.md) 或
跨链验证协议实现以支持能力
将任何不当行为从抵押链传递回主链
链，以便可以削减模棱两可的验证器。

## 决定

我们将在 Cosmos SDK 中实现一个支持以下功能的证据模块
功能:

- 为开发人员提供定义所需的抽象和接口
  自定义证据消息、消息处理程序以及削减和惩罚的方法
  相应地为不当行为。
- 支持将证据消息路由到任何模块中的处理程序的能力
  确定提交的不当行为的有效性。
- 通过治理支持修改任何惩罚的能力
  证据类型。
- 查询器实现以支持查询参数、证据类型、参数和
  所有提交的有效不当行为。

### 类型

首先，我们定义了`Evidence` 接口类型。 `x/evidence` 模块可以实现
它自己的类型可以被许多链使用(例如`CounterFactualEvidence`)。
此外，其他模块可能会以类似的方式实现自己的“证据”类型
治理可扩展的方式。重要的是要注意任何具体的
实现 `Evidence` 接口的类型可能包括任意字段，例如
违规时间。我们希望 `Evidence` 类型尽可能保持灵活。

当向`x/evidence` 模块提交证据时，具体类型必须提供
验证者的共识地址，应该由`x/slashing` 知道
模块(假设违规有效)，违规的高度
发生并且验证者的权力与违规发生的高度相同。

```go
type Evidence interface {
  Route() string
  Type() string
  String() string
  Hash() HexBytes
  ValidateBasic() error

  // The consensus address of the malicious validator at time of infraction
  GetConsensusAddress() ConsAddress

  // Height at which the infraction occurred
  GetHeight() int64

  // The total power of the malicious validator at time of infraction
  GetValidatorPower() int64

  // The total validator set power at time of infraction
  GetTotalPower() int64
}
```

### 路由和处理

每个“证据”类型必须映射到特定的唯一路线并注册
`x/evidence` 模块。 它通过`Router` 实现来实现这一点。 

```go
type Router interface {
  AddRoute(r string, h Handler) Router
  HasRoute(r string) bool
  GetRoute(path string) Handler
  Seal()
}
```

通过 `x/evidence` 模块成功路由后，`Evidence` 类型
通过`Handler`传递。 这个`Handler`负责执行所有
验证证据有效所需的相应业务逻辑。 在
此外，`Handler` 可以执行任何必要的削减和潜在的监禁。
由于斜线分数通常由某种形式的静态函数产生，
允许 `Handler` 这样做提供了最大的灵活性。 一个例子可以
是 `k * evidence.GetValidatorPower()` 其中 `k` 是一个受控的链上参数
通过治理。 `Evidence` 类型应该提供所有的外部信息
为了让“处理程序”进行必要的状态转换，这是必要的。
如果没有返回错误，则“证据”被认为是有效的。 

```go
type Handler func(Context, Evidence) error
```

### Submission

`Evidence` 通过内部的 `MsgSubmitEvidence` 消息类型提交
由 `x/evidence` 模块的 `SubmitEvidence` 处理。

```go
type MsgSubmitEvidence struct {
  Evidence
}

func handleMsgSubmitEvidence(ctx Context, keeper Keeper, msg MsgSubmitEvidence) Result {
  if err := keeper.SubmitEvidence(ctx, msg.Evidence); err != nil {
    return err.Result()
  }

  // emit events...

  return Result{
    // ...
  }
}
```

`x/evidence` 模块的 keeper 负责将 `Evidence` 与
模块的路由器并调用相应的`Handler`，其中可能包括
削减和监禁验证器。 成功后，提交的证据将被保留。

```go
func (k Keeper) SubmitEvidence(ctx Context, evidence Evidence) error {
  handler := keeper.router.GetRoute(evidence.Route())
  if err := handler(ctx, evidence); err != nil {
    return ErrInvalidEvidence(keeper.codespace, err)
  }

  keeper.setEvidence(ctx, evidence)
  return nil
}
```

###创世纪

最后，我们需要表示 `x/evidence` 模块的创世状态。 这
模块只需要所有提交的有效违规和任何必要参数的列表
模块需要处理提交的证据。 `x/证据`
模块将自然地定义和路由它最需要的本地证据类型
可能需要削减惩罚常数。 

```go
type GenesisState struct {
  Params       Params
  Infractions  []Evidence
}
```

## Consequences

### Positive

- 允许状态机处理链上提交的不当行为并进行惩罚
   基于商定的削减参数的验证器。
- 允许任何模块定义和处理证据类型。 这进一步允许
   削减和监禁由更复杂的机制定义。
- 不完全依赖 Tendermint 提交证据。 
### Negative

- 通过实时链上的治理来引入新的证据类型并不容易
   由于无法引入新证据类型的相应处理程序 

### Neutral

- 我们应该无限期地坚持违规吗？ 还是我们应该更依赖于事件？ 

## References

- [ICS](https://github.com/cosmos/ics)
- [IBC Architecture](https://github.com/cosmos/ics/blob/master/ibc/1_IBC_ARCHITECTURE.md)
- [Tendermint Fork Accountability](https://github.com/tendermint/spec/blob/7b3138e69490f410768d9b1ffc7a17abc23ea397/spec/consensus/fork-accountability.md)
