# ADR 042:グループモジュール

## 変更ログ

-2020/04/09:最初のドラフト

## 状態

下書き

## 概要

ADRは、「x .group」モジュールを定義します。これにより、チェーン上のマルチシグニチャアカウントの作成と管理が可能になり、構成可能な決定ポリシーに基づいてメッセージ実行に投票できるようになります。

## 環境

Cosmos SDKによって残されたアミノマルチシグニチャメカニズムには、特定の制限があります。

-キーのローテーションはできませんが、[アカウントのキーの再生成](adr-034-account-rekeying.md)で解決できます。
-しきい値は変更できません。
-UXは、技術者以外のユーザーにとっては厄介です([#5661](https://github.com/cosmos/cosmos-sdk/issues/5661))。
-`legacy_amino`シンボルパターン([#8141](https://github.com/cosmos/cosmos-sdk/issues/8141))が必要です。

グループモジュールは、現在のマルチ署名アカウントを完全に置き換えることを意図したものではありませんが、上記の制限を解決する方法を提供し、キーを追加、更新、または削除できる、より柔軟なキー管理システムと、構成可能なしきい値を備えています。
[`x .feegrant`](..adr-029-fee-grant-module.md)ans [` x .authz`](adr-030-authz)などの他のアクセス制御モジュールで使用することを目的としています。 -module .md)個人および組織のキー管理を簡素化します。

グループモジュールの概念実証は、https://github.com/regen-network/regen-ledger/tree/master/proto/regen/group/v1alpha1およびhttps://github.com/regen-にあります。 network .regen-ledger .tree .master .x .group。

## 決定

`x .group`モジュールをサポートされている[ORM .Table Storeパッケージ](https://github.com/regen-network/regen-ledger/tree/master/orm)([#7098](https ://github.com/cosmos/cosmos-sdk/issues/7098))Cosmos SDKに入り、ここで開発を続行します。 ORMパッケージには専用のADRがあります。

### グループ

グループは、関連する重みを持つアカウントの組み合わせです。そうではない
アカウントと残高なし。何もありません
特定の投票の重みまたは決定の重み。
グループメンバーは、さまざまな意思決定戦略を使用して提案を作成し、グループアカウントを通じて投票することができます。

グループメンバーを管理し、グループを更新できる「admin」アカウントがあります
メタデータを作成し、新しい管理者を設定します。  

```proto
message GroupInfo {

   ..group_id is the unique ID of this group.
    uint64 group_id = 1;

   ..admin is the account address of the group's admin.
    string admin = 2;

   ..metadata is any arbitrary metadata to attached to the group.
    bytes metadata = 3;

   ..version is used to track changes to a group's membership structure that
   ..would break existing proposals. Whenever a member weight has changed,
   ..or any member is added or removed, the version is incremented and will
   ..invalidate all proposals from older versions.
    uint64 version = 4;

   ..total_weight is the sum of the group members' weights.
    string total_weight = 5;
}
```

```proto
message GroupMember {

   ..group_id is the unique ID of the group.
    uint64 group_id = 1;

   ..member is the member data.
    Member member = 2;
}

/.Member represents a group member with an account address,
/.non-zero weight and metadata.
message Member {

   ..address is the member's account address.
    string address = 1;

   ..weight is the member's voting weight that should be greater than 0.
    string weight = 2;

   ..metadata is any arbitrary metadata to attached to the member.
    bytes metadata = 3;
}
```

### グループアカウント

グループアカウントは、グループおよび意思決定戦略に関連付けられたアカウントです。
グループアカウントには残高があります。

単一のグループが持っている可能性があるため、グループアカウントはグループから抽象化されます
さまざまなタイプのアクションに対する複数の意思決定戦略。 経営陣
意思決定戦略とは別のメンバーシップは、最小限のオーバーヘッドにつながります
また、さまざまなポリシーで一貫したメンバーシップを維持します。 そのパターン
特定のグループのプライマリグループアカウントを設定することをお勧めします。
次に、異なる意思決定戦略で個別のグループアカウントを作成します
そして、必要な権限をマスターアカウントからに委任します
[`x .authz`モジュール](adr-030-authz-module.md)を使用する「サブアカウント」。 

```proto
message GroupAccountInfo {

   ..address is the group account address.
    string address = 1;

   ..group_id is the ID of the Group the GroupAccount belongs to.
    uint64 group_id = 2;

   ..admin is the account address of the group admin.
    string admin = 3;

   ..metadata is any arbitrary metadata of this group account.
    bytes metadata = 4;

   ..version is used to track changes to a group's GroupAccountInfo structure that
   ..invalidates active proposal from old versions.
    uint64 version = 5;

   ..decision_policy specifies the group account's decision policy.
    google.protobuf.Any decision_policy = 6 [(cosmos_proto.accepts_interface) = "DecisionPolicy"];
}
```

グループ管理者と同様に、グループアカウント管理者は、メタデータ、意思決定ポリシーを更新したり、新しいグループアカウント管理者を設定したりできます。

グループアカウントは、管理者またはグループメンバーになることもできます。
たとえば、グループ管理者は、メンバーを「選出」できる別のグループアカウントにすることも、自分自身を選出する同じグループにすることもできます。

### 決定ポリシー

意思決定戦略は、グループメンバーが投票できるメカニズムです。
提案。

すべての意思決定戦略には、最小および最大の投票ウィンドウが必要です。
最小投票ウィンドウは、順番に通過する必要がある最短の期間です
提案の可能性があるため、0に設定できます。 最大投票数
ウィンドウは、プロポーザルに投票して実行できる最大時間です。
閉店前に十分なサポートを受けました。
これらの値は両方とも、チェーン全体の最大投票ウィンドウパラメータよりも小さくする必要があります。

すべての決定ポリシーが実装する必要がある `DecisionPolicy`インターフェースを定義します。  

```go
type DecisionPolicy interface {
	codec.ProtoMarshaler

	ValidateBasic() error
	GetTimeout() types.Duration
	Allow(tally Tally, totalPower string, votingDuration time.Duration) (DecisionPolicyResult, error)
	Validate(g GroupInfo) error
}

type DecisionPolicyResult struct {
	Allow bool
	Final bool
}
```

#### しきい値決定戦略

しきい値決定戦略は、カウントに基づいてサポート投票の最小数を定義します(_yes_)
有権者は提案に合格するように加重されます。 にとって
この決定ポリシー、棄権、拒否権はサポートされていないと見なされます(_no_)。 

```proto
message ThresholdDecisionPolicy {

   ..threshold is the minimum weighted sum of support votes for a proposal to succeed.
    string threshold = 1;

   ..voting_period is the duration from submission of a proposal to the end of voting period
   ..Within this period, votes and exec messages can be submitted.
    google.protobuf.Duration voting_period = 2 [(gogoproto.nullable) = false];
}
```

### 提案

グループのメンバーは誰でも、グループアカウントが決定するための提案を提出できます。
提案の場合、提案は一連の `sdk.Msg`で構成されます
パスと提案に関連するメタデータ。 これらの `sdk.Msg`は、` Msg .CreateProposal`リクエスト検証の一部として検証されます。 また、署名者をグループアカウントとして設定する必要があります。

内部的には、提案は以下も追跡します。

-現在の「ステータス」:送信済み、クローズ済み、または中止済み
-その「結果」:不完全、承認、または拒否
-その「VoteState」は「Tally」の形式であり、新しい投票と提案の実行に基づいて計算されます。 

```proto
/.Tally represents the sum of weighted votes.
message Tally {
    option (gogoproto.goproto_getters) = false;

   ..yes_count is the weighted sum of yes votes.
    string yes_count = 1;

   ..no_count is the weighted sum of no votes.
    string no_count = 2;

   ..abstain_count is the weighted sum of abstainers.
    string abstain_count = 3;

   ..veto_count is the weighted sum of vetoes.
    string veto_count = 4;
}
```

### 投票

グループメンバーは提案に投票できます。投票するときは、「はい」、「いいえ」、「棄権」、「拒否」の4つの選択肢があります。いいえ
すべての意思決定ポリシーがそれらをサポートします。投票には、オプションのメタデータを含めることができます。
現在の実装では、投票ウィンドウは提案の直後に開始されます
提出されました。

必要に応じて、投票により提案「VoteState」と「Status」および「Result」が内部で更新されます。

### 提案を実行する

現在の設計では、チェーンは提案を自動的に実行しません。
代わりに、ユーザーは実行を試みるために `Msg .Exec`トランザクションを送信する必要があります
現在の投票および意思決定方針に基づく提案。将来のアップグレードの可能性
これを自動的に行い、グループアカウント(または手数料の付与者)に支払いを依頼します。

#### グループメンバーシップを変更する

現在の実装では、提案を送信した後にグループまたはグループアカウントを更新すると、それが無効になります。誰かが `Msg .Exec`を呼び出すと、失敗するだけで、最終的にはガベージコレクションされます。

### 現在の実装に関する注意

このセクションでは、グループモジュールの概念実証で使用されている現在の実装の概要を説明しますが、これは変更および反復される可能性があります。

#### ORM

[ORMパッケージ](https://github.com/cosmos/cosmos-sdk/discussions/9156)は、グループモジュールで使用されるテーブル、シーケンス、およびセカンダリインデックスを定義します。

グループは「groupTable」の一部として状態に格納され、「group_id」は自動インクリメント整数です。グループメンバーは「groupMemberTable」に保存されます。

グループアカウントは「groupAccountTable」に保存されます。グループアカウントアドレスは、自動インクリメント整数に基づいて生成されます。この整数は、グループモジュール `RootModuleKey`を[ADR-033](adr-033-protobuf-inter-module-commなどの` DerivedModuleKey`に派生させるために使用されます。 md#modulekeys-および-moduleids)。グループアカウントは、 `x .auth`を介して新しい` ModuleAccount`として追加されます。

プロポーザルは、「proposalTable」の一部として保存される「Proposal」タイプを使用します。 `proposal_id`は自動インクリメント整数です。

投票は「voteTable」に保存されます。主キーは、投票された `proposal_id`と` voter`アカウントアドレスに基づいています。

#### ADR-033プロポーザルニュースによるルート

[ADR-033](adr-033-protobuf-inter-module-comm.md)導入されたモジュール間通信は、プロポーザルのグループアカウントに対応する「DerivedModuleKey」を使用してプロポーザルのメッセージをルーティングするために使用できます。

## 結果

### ポジティブ

-マルチ署名アカウントのUXを改善し、キーローテーションとカスタム意思決定戦略を可能にしました。

### ネガティブ

### ニュートラル

-ADR 033を使用しているため、Cosmos SDKに実装する必要がありますが、これは既存のCosmosSDKモジュールの大規模なリファクタリングを意味するものではありません。
-グループモジュールの現在の実装では、ORMパッケージを使用しています。

## さらなる議論

-` .group`と `x .gov`は、提案と投票の収束をサポートするために使用されます:https://github.com/cosmos/cosmos-sdk/discussions/9066
-`x .group`の将来の改善の可能性:
     -提出時に提案を実行します(https://github.com/regen-network/regen-ledger/issues/288)
     -提案を撤回します(https://github.com/regen-network/cosmos-modules/issues/41)
     -「タリー」をより柔軟にし、非バイナリオプションをサポートします

## 参照する

-初期仕様:
     -https://gist.github.com/aaronc/b60628017352df5983791cad30babe56#group-module
     -[#5236](https://github.com/cosmos/cosmos-sdk/pull/5236)
-CosmosSDKに `x .group`を追加する提案:[#7633](https://github.com/cosmos/cosmos-sdk/issues/7633) 