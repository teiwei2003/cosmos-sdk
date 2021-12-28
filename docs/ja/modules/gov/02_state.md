# 状態

## 提案

`Proposal`オブジェクトは、投票を集計し、一般的に提案の状態を追跡するために使用されます。
これらには、ガバナンスモジュールが解決を試み、提案が通過した場合に実行しようとする任意の `sdk.Msg`の配列が含まれています。 `プロポーザル`は一意のIDで識別され、一連のタイムスタンプが含まれます： `submit_time`、` deposit_end_time`、 `voting_start_time`、` voting_end_time`プロポーザルのライフサイクルを追跡します。

+++ https://github.com/cosmos/cosmos-sdk/blob/4a129832eb16f37a89e97652a669f0cdc9196ca9/proto/cosmos/gov/v1beta2/gov.proto#L42-L52

提案は通常、その目的を説明するための一連のメッセージ以上のものを必要としますが、より大きな正当化が必要であり、関心のある参加者が提案について話し合い、議論するための手段を可能にします。 ほとんどの場合、オンチェーンガバナンスプロセスをサポートするオフチェーンシステムを使用することをお勧めします。 これに対応するために、プロポーザルには、プロポーザルにコンテキストを追加するために使用できるバイトの配列である特別な「メタデータ」フィールドが含まれています。 `metadata`フィールドはネットワークのカスタム使用を許可しますが、フィールドには[IPFS]（https://docs.ipfs.io/concepts/content-などのシステムを使用するURLまたは何らかの形式のCIDが含まれていることが期待されます。 アドレッシング/）。
ネットワーク間の相互運用性のケースをサポートするために、SDKは `metadata`が次の` JSON`テンプレートを表すことを推奨しています。

```json
{
  "title": "...",
  "description": "...",
  "forum": "...", // a link to the discussion platform (i.e. Discord)
  "other": "..." // any extra data that doesn't correspond to the other fields
}
```

これにより、クライアントが複数のネットワークをサポートするのがはるかに簡単になります。

### ガバナンスを使用するモジュールの作成

チェーン、またはさまざまなパラメーターの変更など、ガバナンスを使用して実行する必要がある個々のモジュールには、多くの側面があります。 これは非常に簡単です。 まず、メッセージタイプと `MsgServer`の実装を書き出します。 コンストラクターに入力されるキーパーに `authority`フィールドを追加します。
ガバナンスモジュールアカウント： `govKeeper.GetGovernanceAccount().GetAddress()`。
次に、 `msg_server.go`のメソッドについて、署名者が`authority`と一致するというメッセージのチェックを実行します。 これにより、ユーザーはそのメッセージを実行できなくなります。

## パラメータと基本タイプ

`Parameters`は、投票が実行されるルールを定義します。できるのは任意の時点で1つのアクティブなパラメータセットである。ガバナンスが変更したい場合、値を変更するか、パラメータフィールドを追加/削除するためのパラメータセット、新しいパラメータセットを作成し、前のパラメータセットを非アクティブにする必要があります。

### DepositParams

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L127-L145

### VotingParams

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L147-L156

### TallyParams

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L158-L183

パラメータはグローバルな`GlobalParams`KVStoreに保存されます。

さらに、いくつかの基本的なタイプを紹介します。 

```go
type Vote byte

const (
    VoteYes         = 0x1
    VoteNo          = 0x2
    VoteNoWithVeto  = 0x3
    VoteAbstain     = 0x4
)

type ProposalType  string

const (
    ProposalTypePlainText       = "Text"
    ProposalTypeSoftwareUpgrade = "SoftwareUpgrade"
)

type ProposalStatus byte


const (
	StatusNil           ProposalStatus = 0x00
    StatusDepositPeriod ProposalStatus = 0x01  // Proposal is submitted. Participants can deposit on it but not vote
    StatusVotingPeriod  ProposalStatus = 0x02  // MinDeposit is reached, participants can vote
    StatusPassed        ProposalStatus = 0x03  // Proposal passed and successfully executed
    StatusRejected      ProposalStatus = 0x04  // Proposal has been rejected
    StatusFailed        ProposalStatus = 0x05  // Proposal passed but failed execution
)
```

## 保証金

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L43-L53

## ValidatorGovInfo

このタイプは、集計時に一時マップで使用されます

```go
  type ValidatorGovInfo struct {
    Minus     sdk.Dec
    Vote      Vote
  }
```

##　ストア

_Storesは、マルチストア内のKVStoresです。 ストアを見つけるための鍵は、list_`の最初のパラメーターです。1つのKVStore `Governance`を使用して、2つのマッピングを保存します。

- `proposalID | 'proposal'`から `Proposal`へのマッピング。
- `proposalID | 'addresses' | address`から `Vote`へのマッピング。 このマッピングにより、 `proposalID：addresses`で範囲クエリを実行することにより、提案に投票したすべてのアドレスとその投票をクエリできます。擬似コードの目的で、ストアでの読み取りまたは書き込みに使用する2つの関数を次に示します。

- ` load（StoreKey、Key） `：マルチストアのキー` StoreKey`にあるストアのキー `Key`に保存されているアイテムを取得します
- `store（StoreKey、Key、value） `：マルチストアのキー` StoreKey`にあるストアのキー `Key`に値` Value`を書き込みます

## 提案処理キュー

**ストア：**

- `ProposalProcessingQueue`： `MinDeposit`に到達したプロポーザルのすべての` ProposalIDs`を含むキュー `queue [proposalID]`。 各 `EndBlock`の間に、投票期間の終わりに達したすべての提案が処理されます。
完成した提案を処理するために、アプリケーションは投票を集計し、各バリデーターの投票を計算し、バリデーターセット内のすべてのバリデーターが投票したかどうかを確認します。 提案が受理された場合、保証金は返金されます。 最後に、プロポーザルコンテンツ `Handler`が実行されます。

そして、 `ProposalProcessingQueue`の擬似コード：

```go
  in EndBlock do

    for finishedProposalID in GetAllFinishedProposalIDs(block.Time)
      proposal = load(Governance, <proposalID|'proposal'>) // proposal is a const key

      validators = Keeper.getAllValidators()
      tmpValMap := map(sdk.AccAddress)ValidatorGovInfo

      // Initiate mapping at 0. This is the amount of shares of the validator's vote that will be overridden by their delegator's votes
      for each validator in validators
        tmpValMap(validator.OperatorAddr).Minus = 0

      // Tally
      voterIterator = rangeQuery(Governance, <proposalID|'addresses'>) //return all the addresses that voted on the proposal
      for each (voterAddress, vote) in voterIterator
        delegations = stakingKeeper.getDelegations(voterAddress) // get all delegations for current voter

        for each delegation in delegations
          // make sure delegation.Shares does NOT include shares being unbonded
          tmpValMap(delegation.ValidatorAddr).Minus += delegation.Shares
          proposal.updateTally(vote, delegation.Shares)

        _, isVal = stakingKeeper.getValidator(voterAddress)
        if (isVal)
          tmpValMap(voterAddress).Vote = vote

      tallyingParam = load(GlobalParams, 'TallyingParam')

      // Update tally if validator voted they voted
      for each validator in validators
        if tmpValMap(validator).HasVoted
          proposal.updateTally(tmpValMap(validator).Vote, (validator.TotalShares - tmpValMap(validator).Minus))



      // Check if proposal is accepted or rejected
      totalNonAbstain := proposal.YesVotes + proposal.NoVotes + proposal.NoWithVetoVotes
      if (proposal.Votes.YesVotes/totalNonAbstain > tallyingParam.Threshold AND proposal.Votes.NoWithVetoVotes/totalNonAbstain  < tallyingParam.Veto)
        //  proposal was accepted at the end of the voting period
        //  refund deposits (non-voters already punished)
        for each (amount, depositor) in proposal.Deposits
          depositor.AtomBalance += amount

        stateWriter, err := proposal.Handler()
        if err != nil
            // proposal passed but failed during state execution
            proposal.CurrentStatus = ProposalStatusFailed
         else
            // proposal pass and state is persisted
            proposal.CurrentStatus = ProposalStatusAccepted
            stateWriter.save()
      else
        // proposal was rejected
        proposal.CurrentStatus = ProposalStatusRejected

      store(Governance, <proposalID|'proposal'>, proposal)
```
