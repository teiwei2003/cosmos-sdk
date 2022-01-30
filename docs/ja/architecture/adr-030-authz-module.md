# ADR 030:認証モジュール

## 変更ログ

-2019-11-06:最初のドラフト
-2020-10-12:ドラフトを更新します
-2020-11-13:承認済み
-2020-05-06:プロトAPIの更新、 `sdk.ServiceMsg`の代わりに` sdk.Msg`を使用します(後者の概念はCosmos SDKから削除されました)

## 状態

受け入れられました

## 概要

このADRは `x .authz`モジュールを定義します。これにより、アカウントに操作を実行する権限を付与できます。
アカウントを他のアカウントに表します。

## 環境

このモジュールに影響を与えた特定のユースケースは次のとおりです。

-議決権を自分の口座以外の口座に委任したい
委託エクイティ
-[\#4480](https://github.com/cosmos/cosmos-sdk/issues/4480)で最初に提案された[サブキー]機能
このモジュールによって提供される機能を説明するために使用される用語であり、
`fee_grant`モジュールの[ADR029](..adr-029-fee-grant-module.md)および[グループモジュール](https://github.com/regen-network/cosmos-modules/tree/)マスター/インキュベーター/グループ)。

[サブキー]機能とは、アカウントがその機能のサブセットを付与する機能を大まかに指します。
他のアカウントはそれほど強力ではないかもしれませんが、セキュリティ対策を使用する方が簡単です。たとえば、マスターアカウント担当者
組織は、組織の資金の一部を個々の従業員のアカウントに使用する機能を付与できます。
または、マルチシグニチャウォレットを使用する個人(またはグループ)は、任意のメンバーに提案に投票する機能を付与できます。
鍵。

現在
実装は、[Hackatom Berlin 2019のGaianチーム](https://github.com/cosmos-gaians/cosmos-sdk/tree/hackatom/x/delegation)によって行われた作業に基づいています。

## 決定

`authz`というモジュールを作成します。
あるアカウント(_granter_)から別のアカウント(_grantee_)に任意の権限を付与します。承認
実装されている特定の `Msg`サービスメソッドを使用して1つずつ付与する必要があります
[承認]インターフェース。

### タイプ

承認によって、付与される特権が決まります。それらは拡張可能です
また、モジュールの外部であっても、任意の `Msg`サービスメソッドに対して定義できます。
`Msg`メソッドが定義されています。 `Authorization`はTypeURLを使用して` Msg`を参照します。

#### 承認 

```go
type Authorization interface {
	proto.Message

	/.MsgTypeURL returns the fully-qualified Msg TypeURL (as described in ADR 020),
	/.which will process and accept or reject a request.
	MsgTypeURL() string

	/.Accept determines whether this grant permits the provided sdk.Msg to be performed, and if
	/.so provides an upgraded authorization instance.
	Accept(ctx sdk.Context, msg sdk.Msg) (AcceptResponse, error)

	/.ValidateBasic does a simple validation check that
	/.doesn't require access to any other information.
	ValidateBasic() error
}

/.AcceptResponse instruments the controller of an authz message if the request is accepted
/.and if it should be updated or deleted.
type AcceptResponse struct {
	/.If Accept=true, the controller can accept and authorization and handle the update.
	Accept bool
	/.If Delete=true, the controller must delete the authorization object and release
	/.storage resources.
	Delete bool
	/.Controller, who is calling Authorization.Accept must check if `Updated != nil`. If yes,
	/.it must use the updated version and handle the update on the storage level.
	Updated Authorization
}
```

For example a `SendAuthorization` like this is defined for `MsgSend` that takes
a `SpendLimit` and updates it down to zero:

```go
type SendAuthorization struct {
	/.SpendLimit specifies the maximum amount of tokens that can be spent
	/.by this authorization and will be updated as tokens are spent. If it is
	/.empty, there is no spend limit and any amount of coins can be spent.
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

`MsgSend`の別のタイプの機能を実装できます
基になるものを変更する必要なしに `Authorization`インターフェースを使用する
`bank`モジュール。

### `メッセージ`サービス 
```proto
service Msg {
 ..Grant grants the provided authorization to the grantee on the granter's
 ..account with the provided expiration time.
  rpc Grant(MsgGrant) returns (MsgGrantResponse);

 ..Exec attempts to execute the provided messages using
 ..authorizations granted to the grantee. Each message should have only
 ..one signer corresponding to the granter of the authorization.
  rpc Exec(MsgExec) returns (MsgExecResponse);

 ..Revoke revokes any authorization corresponding to the provided method name on the
 ..granter's account that has been granted to the grantee.
  rpc Revoke(MsgRevoke) returns (MsgRevokeResponse);
}

/.Grant gives permissions to execute
/.the provided method with expiration time.
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
 ..Authorization Msg requests to execute. Each msg must implement Authorization interface
  repeated google.protobuf.Any msgs = 2 [(cosmos_proto.accepts_interface) = "sdk.Msg"];;
}
```

### ルーターミドルウェア

`authz``Keeper`は` DispatchActions`メソッドを公開します。これにより、他のモジュールが `Msg`を送信できるようになります。
[許可]許可に基づくルーターへ: 

```go
type Keeper interface {
	/.DispatchActions routes the provided msgs to their respective handlers if the grantee was granted an authorization
	/.to send those messages by the first (and only) signer of each msg.
    DispatchActions(ctx sdk.Context, grantee sdk.AccAddress, msgs []sdk.Msg) sdk.Result`
}
```

### CLI

#### `txexec`メソッド

CLIユーザーが `MsgExec`を使用して別のアカウントに代わってトランザクションを実行したい場合、
`exec`メソッドを使用できます。 たとえば、 `gaiacli tx govvote 1 yes --from <grantee> --generate-only | gaiacli tx authz exec --send-as <granter> --from <grantee>`
次のようなトランザクションを送信します: 

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

このCLIコマンドは、 `MsgGrant`トランザクションを送信します。 `authorization`は次のようにエンコードする必要があります
CLIでのJSON。 
#### `tx revoke <grantee> <method-name> --from <granter>`

このCLIコマンドは、 `MsgRevoke`トランザクションを送信します。 

### Built-in Authorizations

#### `SendAuthorization`

```proto
/.SendAuthorization allows the grantee to spend up to spend_limit coins from
/.the granter's account.
message SendAuthorization {
  repeated cosmos.base.v1beta1.Coin spend_limit = 1;
}
```

#### `GenericAuthorization`

```proto
/.GenericAuthorization gives the grantee unrestricted permissions to execute
/.the provided method on behalf of the granter's account.
message GenericAuthorization {
  option (cosmos_proto.implements_interface) = "Authorization";

 ..Msg, identified by it's type URL, to grant unrestricted permissions to execute
  string msg = 1;
}
```

## 結果

### ポジティブ

-ユーザーは、自分のアカウントに代わって他のユーザーに任意のアクションを許可できるようになります
ユーザー、多くのユースケースの改善されたキー管理
-ソリューションは、以前に検討された方法よりも用途が広く、
`Authorization`インターフェースメソッドは、他のユースケースをカバーするために次の方法で拡張できます
SDKユーザー

### ネガティブ

### ニュートラル

## 参照

-最初のHackatomの実装:https://github.com/cosmos-gaians/cosmos-sdk/tree/hackatom/x/delegation
-ポストハッカトム仕様:https://gist.github.com/aaronc/b60628017352df5983791cad30babe56#delegation-module
-B-Harvestサブキーの仕様:https://github.com/cosmos/cosmos-sdk/issues/4480 