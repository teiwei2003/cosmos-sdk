# ADR 030:授权模块

## 变更日志

- 2019-11-06:初稿
- 2020-10-12:更新草案
- 2020-11-13:接受
- 2020-05-06:proto API 更新，使用 `sdk.Msg` 而不是 `sdk.ServiceMsg`(后一个概念已从 Cosmos SDK 中删除)

## 地位

公认

## 摘要

此 ADR 定义了 `x/authz` 模块，该模块允许帐户授予执行操作的权限
代表该帐户到其他帐户。

##  上下文

激发该模块的具体用例包括:

- 希望将提案投票权委托给除自己拥有的账户之外的其他账户
委托权益
- “子密钥”功能，最初在 [\#4480](https://github.com/cosmos/cosmos-sdk/issues/4480) 中提出
是一个术语，用于描述此模块提供的功能以及
[ADR 029](./adr-029-fee-grant-module.md) 和 [group 模块](https://github.com/regen-network/cosmos-modules/tree/) 中的 `fee_grant` 模块主/孵化器/组)。

“子密钥”功能粗略地指的是一个帐户授予其某些功能子集的能力
其他帐户可能不太强大，但更容易使用安全措施。例如，一个主账户代表
组织可以授予将组织的少量资金用于个人员工帐户的能力。
或者拥有多重签名钱包的个人(或团体)可以授予任何成员对提案进行投票的能力
键。

目前
实施基于 [Gaian 团队在 Hackatom Berlin 2019](https://github.com/cosmos-gaians/cosmos-sdk/tree/hackatom/x/delegation) 所做的工作。

## 决定

我们将创建一个名为 `authz` 的模块，它为
从一个帐户(_granter_)向另一个帐户(_grantee_)授予任意权限。授权
必须使用实现为特定的 `Msg` 服务方法一一授予
“授权”界面。

### 类型

授权决定了授予哪些特权。它们是可扩展的
并且可以为任何 `Msg` 服务方法定义，甚至在模块之外
定义了 `Msg` 方法。 `Authorization`s 使用它们的 TypeURL 引用 `Msg`s。 

#### Authorization

```go
type Authorization interface {
	proto.Message

	//MsgTypeURL returns the fully-qualified Msg TypeURL (as described in ADR 020),
	//which will process and accept or reject a request.
	MsgTypeURL() string

	//Accept determines whether this grant permits the provided sdk.Msg to be performed, and if
	//so provides an upgraded authorization instance.
	Accept(ctx sdk.Context, msg sdk.Msg) (AcceptResponse, error)

	//ValidateBasic does a simple validation check that
	//doesn't require access to any other information.
	ValidateBasic() error
}

//AcceptResponse instruments the controller of an authz message if the request is accepted
//and if it should be updated or deleted.
type AcceptResponse struct {
	//If Accept=true, the controller can accept and authorization and handle the update.
	Accept bool
	//If Delete=true, the controller must delete the authorization object and release
	//storage resources.
	Delete bool
	//Controller, who is calling Authorization.Accept must check if `Updated != nil`. If yes,
	//it must use the updated version and handle the update on the storage level.
	Updated Authorization
}
```

For example a `SendAuthorization` like this is defined for `MsgSend` that takes
a `SpendLimit` and updates it down to zero:

```go
type SendAuthorization struct {
	//SpendLimit specifies the maximum amount of tokens that can be spent
	//by this authorization and will be updated as tokens are spent. If it is
	//empty, there is no spend limit and any amount of coins can be spent.
	SpendLimit sdk.Coins
}

func (a SendAuthorization) MsgTypeURL() string {
	return sdk.MsgTypeURL(&MsgSend{})
}

func (a SendAuthorization) Accept(ctx sdk.Context, msg sdk.Msg) (authz.AcceptResponse, error) {
	mSend, ok := msg.(*MsgSend)
	if !ok {
		return authz.AcceptResponse{}, sdkerrors.ErrInvalidType.Wrap("type mismatch")
	}
	limitLeft, isNegative := a.SpendLimit.SafeSub(mSend.Amount)
	if isNegative {
		return authz.AcceptResponse{}, sdkerrors.ErrInsufficientFunds.Wrapf("requested amount is more than spend limit")
	}
	if limitLeft.IsZero() {
		return authz.AcceptResponse{Accept: true, Delete: true}, nil
	}

	return authz.AcceptResponse{Accept: true, Delete: false, Updated: &SendAuthorization{SpendLimit: limitLeft}}, nil
}
```

A different type of capability for `MsgSend` could be implemented
using the `Authorization` interface with no need to change the underlying
`bank` module.

### `Msg` Service

```proto
service Msg {
 //Grant grants the provided authorization to the grantee on the granter's
 //account with the provided expiration time.
  rpc Grant(MsgGrant) returns (MsgGrantResponse);

 //Exec attempts to execute the provided messages using
 //authorizations granted to the grantee. Each message should have only
 //one signer corresponding to the granter of the authorization.
  rpc Exec(MsgExec) returns (MsgExecResponse);

 //Revoke revokes any authorization corresponding to the provided method name on the
 //granter's account that has been granted to the grantee.
  rpc Revoke(MsgRevoke) returns (MsgRevokeResponse);
}

//Grant gives permissions to execute
//the provided method with expiration time.
message Grant {
  google.protobuf.Any       authorization = 1 [(cosmos_proto.accepts_interface) = "Authorization"];
  google.protobuf.Timestamp expiration    = 2 [(gogoproto.stdtime) = true, (gogoproto.nullable) = false];
}

message MsgGrant {
  string granter = 1;
  string grantee = 2;

  Grant grant = 3 [(gogoproto.nullable) = false];
}

message MsgExecResponse {
  cosmos.base.abci.v1beta1.Result result = 1;
}

message MsgExec {
  string   grantee                  = 1;
 //Authorization Msg requests to execute. Each msg must implement Authorization interface
  repeated google.protobuf.Any msgs = 2 [(cosmos_proto.accepts_interface) = "sdk.Msg"];;
}
```

### Router Middleware

`authz` `Keeper` 将公开一个 `DispatchActions` 方法，该方法允许其他模块发送 `Msg`s
到基于“授权”授权的路由器: 

```go
type Keeper interface {
	//DispatchActions routes the provided msgs to their respective handlers if the grantee was granted an authorization
	//to send those messages by the first (and only) signer of each msg.
    DispatchActions(ctx sdk.Context, grantee sdk.AccAddress, msgs []sdk.Msg) sdk.Result`
}
```

### CLI

#### `tx exec` Method

当 CLI 用户想要使用 `MsgExec` 代表另一个帐户运行事务时，他们
可以使用`exec`方法。 例如`gaiacli tx gov vote 1 yes --from <grantee> --generate-only | gaiacli tx authz exec --send-as <granter> --from <grantee>`
会发送这样的交易: 

```go
MsgExec {
  Grantee: mykey,
  Msgs: []sdk.Msg{
    MsgVote {
      ProposalID: 1,
      Voter: cosmos3thsdgh983egh823
      Option: Yes
    }
  }
}
```

#### `tx grant <grantee> <authorization> --from <granter>`

这个 CLI 命令将发送一个 `MsgGrant` 事务。 `authorization` 应该被编码为
CLI 上的 JSON。 

#### `tx revoke <grantee> <method-name> --from <granter>`

这个 CLI 命令将发送一个 `MsgRevoke` 事务。

### Built-in Authorizations

#### `SendAuthorization`

```proto
//SendAuthorization allows the grantee to spend up to spend_limit coins from
//the granter's account.
message SendAuthorization {
  repeated cosmos.base.v1beta1.Coin spend_limit = 1;
}
```

#### `GenericAuthorization`

```proto
//GenericAuthorization gives the grantee unrestricted permissions to execute
//the provided method on behalf of the granter's account.
message GenericAuthorization {
  option (cosmos_proto.implements_interface) = "Authorization";

 //Msg, identified by it's type URL, to grant unrestricted permissions to execute
  string msg = 1;
}
```

## Consequences

### Positive

- 用户将能够代表他们的帐户向其他人授权任意操作
用户，改进了许多用例的密钥管理
- 该解决方案比以前考虑的方法更通用，并且
`Authorization` 接口方法可以通过以下方式扩展以涵盖其他用例
SDK用户 

### Negative

### Neutral

## References

- 初始 Hackatom 实施:https://github.com/cosmos-gaians/cosmos-sdk/tree/hackatom/x/delegation
- 后 Hackatom 规范:https://gist.github.com/aaronc/b60628017352df5983791cad30babe56#delegation-module
- B-Harvest 子密钥规范:https://github.com/cosmos/cosmos-sdk/issues/4480 