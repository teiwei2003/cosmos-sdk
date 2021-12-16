# ADR 046:模块参数

## 变更日志

- 2021 年 9 月 22 日:初稿

## 地位

建议的

## 摘要

此 ADR 描述了 Cosmos SDK 模块如何使用、交互、
并存储它们各自的参数。

## 语境

目前，在 Cosmos SDK 中，需要使用参数的模块使用
`x/params` 模块。 `x/params` 通过让模块定义参数来工作，
通常通过一个简单的“Params”结构，并将该结构注册到
`x/params` 模块通过属于各自的唯一 `Subspace`
注册模块。然后，注册模块可以唯一地访问其各自的
`子空间`。通过这个`Subspace`，模块可以获取和设置它的`Params`
结构体。

此外，Cosmos SDK 的`x/gov` 模块直接支持更改
通过“ParamChangeProposal”治理提案类型在链上设置参数，其中
利益相关者可以对建议的参数更改进行投票。

使用 `x/params` 模块来管理单个
模块参数。也就是说，管理参数本质上是“免费”的
开发人员只需要定义`Params`结构、`Subspace`和
各种辅助功能，例如`ParamSetPairs`，在 `Params` 类型上。然而，
有一些明显的缺点。这些缺点包括参数
通过非常慢的 JSON 在状态中序列化。此外，参数
通过 `ParamChangeProposal` 治理提案进行的更改无法读取
或写信给国家。换句话说，目前不可能有任何
尝试更改参数期间应用程序中的状态转换。

## 决定

我们将建立在 `x/gov` 和 `x/authz` 的对齐基础上
[#9810](https://github.com/cosmos/cosmos-sdk/pull/9810)。即模块开发者
将创建一个或多个必须序列化的唯一参数数据结构
陈述。 Param 数据结构必须实现`sdk.Msg` 接口与各自的
Protobuf Msg 服务方法，它将验证和更新所有参数
必要的改变。 `x/gov` 模块通过在
[#9810](https://github.com/cosmos/cosmos-sdk/pull/9810)，会调度Param
消息，将由 Protobuf Msg 服务处理。

请注意，由开发人员决定如何构建他们的参数和
相应的 `sdk.Msg` 消息。考虑当前定义的参数
`x/auth` 使用 `x/params` 模块进行参数管理: 

```protobuf
message Params {
  uint64 max_memo_characters       = 1;
  uint64 tx_sig_limit              = 2;
  uint64 tx_size_cost_per_byte     = 3;
  uint64 sig_verify_cost_ed25519   = 4;
  uint64 sig_verify_cost_secp256k1 = 5;
}
```

开发人员可以选择为每个字段创建唯一的数据结构
`Params` 或者他们可以创建一个单一的 `Params` 结构，如上所述
“x/auth”的情况。

在前者中，`x/params` 方法，需要为每个单独的创建一个 `sdk.Msg`
字段以及处理程序。 如果有很多，这可能会成为负担
参数字段。 在后一种情况下，只有一个数据结构和
因此只有一个消息处理程序，但是，消息处理程序可能必须是
更复杂，因为它可能需要了解正在使用的参数
更改与未更改的参数。

参数更改建议是使用 `x/gov` 模块提出的。 执行是通过
`x/authz` 授权给根 `x/gov` 模块的帐户。

继续使用`x/auth`，我们演示一个更完整的例子: 
```go
type Params struct {
	MaxMemoCharacters      uint64
	TxSigLimit             uint64
	TxSizeCostPerByte      uint64
	SigVerifyCostED25519   uint64
	SigVerifyCostSecp256k1 uint64
}

type MsgUpdateParams struct {
	MaxMemoCharacters      uint64
	TxSigLimit             uint64
	TxSizeCostPerByte      uint64
	SigVerifyCostED25519   uint64
	SigVerifyCostSecp256k1 uint64
}

type MsgUpdateParamsResponse struct {}

func (ms msgServer) UpdateParams(goCtx context.Context, msg *types.MsgUpdateParams) (*types.MsgUpdateParamsResponse, error) {
  ctx := sdk.UnwrapSDKContext(goCtx)

  // verification logic...

  // persist params
  params := ParamsFromMsg(msg)
  ms.SaveParams(ctx, params)

  return &types.MsgUpdateParamsResponse{}, nil
}

func ParamsFromMsg(msg *types.MsgUpdateParams) Params {
  // ...
}
```

还应提供 gRPC `Service` 查询，例如: 

```protobuf
service Query {
  // ...
  
  rpc Params(QueryParamsRequest) returns (QueryParamsResponse) {
    option (google.api.http).get = "/cosmos/<module>/v1beta1/params";
  }
}

message QueryParamsResponse {
  Params params = 1 [(gogoproto.nullable) = false];
}
```

## Consequences

作为实施模块参数方法的结果，我们获得了能力
使模块参数更改有状态且可扩展以适应几乎所有
应用程序的用例。我们将能够发出事件(并触发注册的钩子
使用 [even hooks](https://github.com/cosmos/cosmos-sdk/discussions/9656)) 中提出的工作的那些事件，
调用其他消息服务方法或执行迁移。
此外，在阅读方面会有显着的性能提升
以及从状态和向状态写入参数，特别是如果一组特定的参数
在一致的基础上阅读。

但是，这种方法将需要开发人员实现更多类型和
如果存在许多参数，消息服务方法可能会变得很麻烦。此外，
需要开发者实现模块参数的持久化逻辑。
然而，这应该是微不足道的。

### 向后兼容性

处理模块参数的新方法自然不会倒退
与现有的`x/params` 模块兼容。但是，`x/params` 将
保留在 Cosmos SDK 中，并且将被标记为已弃用，不再添加
除了潜在的错误修复之外，还添加了功能。注意，`x/params`
模块可能会在未来的版本中完全删除。 
### Positive

- 更有效地序列化模块参数
- 模块能够对参数更改做出反应并执行其他操作。
- 可以发出特殊事件，允许触发钩子。 
### Negative

- 模块参数对于模块开发者来说变得稍微有些负担:
     - 模块现在负责持久化和检索参数状态
     - 现在需要模块具有唯一的消息处理程序来处理参数
       每个唯一参数数据结构的变化。 

### Neutral

- Requires [#9810](https://github.com/cosmos/cosmos-sdk/pull/9810) to be reviewed
  and merged.

<!-- ## Further Discussions

While an ADR is in the DRAFT or PROPOSED stage, this section should contain a summary of issues to be solved in future iterations (usually referencing comments from a pull-request discussion).
Later, this section can optionally list ideas or improvements the author or reviewers found during the analysis of this ADR. -->

## References

- https://github.com/cosmos/cosmos-sdk/pull/9810
- https://github.com/cosmos/cosmos-sdk/issues/9438
- https://github.com/cosmos/cosmos-sdk/discussions/9913
