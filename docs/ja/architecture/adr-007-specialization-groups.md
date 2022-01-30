# ADR 007:スペシャライゼーショングループ

## 変更ログ

-2019年7月31日:最初のドラフト

## 環境

アイデアはもともと満足することでした
分散型コンピュータ緊急対応チーム(dCERT)を作成します。
メンバーはガバナンスコミュニティによって選出され、職務を遂行します
緊急事態でコミュニティを調整します。 このような考え方
[ブロックチェーンスペシャライゼーション]の概念としてさらに抽象化できます
グループ"。

これらのグループの作成は、専門化機能の始まりです
より広範なブロックチェーンコミュニティでは、特定の機能を有効にするために使用できます
委任された責任のレベル。 専門化の例になることができます
ブロックチェーンコミュニティのメリットには、コード監査、緊急対応、
コード開発など このタイプのコミュニティ組織は
将来の場合、個々の利害関係者は質問の種類ごとに投票を委託します
ガバナンス提案には、質問タイプのフィールドが含まれています。

## 決定

専門グループは大きく分けて以下の機能に分けられます。
(ここに含まれる例):

- Membership Admittance
- Membership Acceptance
- Membership Revocation
    - (probably) Without Penalty
        - member steps down (self-Revocation)
        - replaced by new member from governance
    - (probably) With Penalty
        - due to breach of soft-agreement (determined through governance)
        - due to breach of hard-agreement (determined by code)
- Execution of Duties
    - Special transactions which only execute for members of a specialization
     group (for example, dCERT members voting to turn off transaction routes in
     an emergency scenario)
- Compensation
    - Group compensation (further distribution decided by the specialization group)
    - Individual compensation for all constituents of a group from the
     greater community

専門グループへの会員アクセスは、幅広い範囲で実施できます。
複数のメカニズム。 最も明白な例は
コミュニティ全体ですが、一部のシステムでは、コミュニティは許可したい場合があります
専門家グループ内で新会員を選出した会員、
または、コミュニティが特定の職業に権限を割り当てる場合があります
このグループは、他のサードパーティグループにメンバーを任命します。 空は本当に限界です
メンバーシップアクセスを構成する方法について。 キャプチャしようとします
それらのいくつかは、[選挙人]と呼ばれる共通のインターフェースに表示される場合があります。 にとって
このADRの一部として、一般的なものをお勧めします
選挙の抽象化( `Electionator`)と基本的な
この抽象化の実装により、継続的な選択が可能になります
プロチームのメンバー。   

``` golang
//The Electionator abstraction covers the concept space for
//a wide variety of election kinds.  
type Electionator interface {

  ./is the election object accepting votes.
    Active() bool

  ./functionality to execute for when a vote is cast in this election, here
  ./the vote field is anticipated to be marshalled into a vote type used
  ./by an election.
  ./
  ./NOTE There are no explicit ids here. Just votes which pertain specifically
  ./to one electionator. Anyone can create and send a vote to the electionator item
  ./which will presumably attempt to marshal those bytes into a particular struct
  ./and apply the vote information in some arbitrary way. There can be multiple
  ./Electionators within the Cosmos-Hub for multiple specialization groups, votes
  ./would need to be routed to the Electionator upstream of here.
    Vote(addr sdk.AccAddress, vote []byte)

  ./here lies all functionality to authenticate and execute changes for
  ./when a member accepts being elected
    AcceptElection(sdk.AccAddress)

  ./Register a revoker object
    RegisterRevoker(Revoker)

  ./No more revokers may be registered after this function is called
    SealRevokers()

  ./register hooks to call when an election actions occur
    RegisterHooks(ElectionatorHooks)

  ./query for the current winner(s) of this election based on arbitrary
  ./election ruleset
    QueryElected() []sdk.AccAddress

  ./query metadata for an address in the election this
  ./could include for example position that an address
  ./is being elected for within a group
  ./
  ./this metadata may be directly related to
  ./voting information and/or privileges enabled
  ./to members within a group.
    QueryMetadata(sdk.AccAddress) []byte
}

//ElectionatorHooks, once registered with an Electionator,
//trigger execution of relevant interface functions when
//Electionator events occur.
type ElectionatorHooks interface {
    AfterVoteCast(addr sdk.AccAddress, vote []byte)
    AfterMemberAccepted(addr sdk.AccAddress)
    AfterMemberRevoked(addr sdk.AccAddress, cause []byte)
}

//Revoker defines the function required for a membership revocation rule-set
//used by a specialization group. This could be used to create self revoking,
//and evidence based revoking, etc. Revokers types may be created and
//reused for different election types.
//
//When revoking the "cause" bytes may be arbitrarily marshalled into evidence,
//memos, etc.
type Revoker interface {
    RevokeName() string    ./identifier for this revoker type
    RevokeMember(addr sdk.AccAddress, cause []byte) error
}
```

既存のコード間にいくつかの共通点があるかもしれません
`x/governance`と選挙に必要な機能。 この一般的な
関数は、実現プロセスで抽象化する必要があります。 それぞれ同じ
クライアントCLI/REST関数を実装するための投票を抽象化する必要があります
複数の選挙に再利用できます。

専門グループの抽象化は、最初に `Electionator`を拡張します
しかし、それはまた、グループの特徴をさらに定義します。  

``` golang
type SpecializationGroup interface {
    Electionator
    GetName() string
    GetDescription() string

  ./general soft contract the group is expected
  ./to fulfill with the greater community
    GetContract() string

  ./messages which can be executed by the members of the group
    Handler(ctx sdk.Context, msg sdk.Msg) sdk.Result

  ./logic to be executed at endblock, this may for instance
  ./include payment of a stipend to the group members
  ./for participation in the security group.
    EndBlocker(ctx sdk.Context)
}
```

## 状態

>提案

## 結果

### ポジティブ

-ブロックチェーンの専門化能力を向上させる
-`x/gov/`の抽象化を改善して、特殊化グループで使用できるようにします

### ネガティブ

-コミュニティ内の集中化を強化するために使用できます

### ニュートラル

## 参照

-[dCERT ADR](./adr-008-dCERT-group.md) 