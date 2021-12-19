# 状态

## 提案

`Proposal` 对象用于统计投票并通常跟踪提案的状态。
它们包含治理模块将尝试的任意 `sdk.Msg` 数组
如果提案通过，则解决然后执行。 “提案”由一个标识
唯一 id 并包含一系列时间戳:`submit_time`、`deposit_end_time`、
`voting_start_time`, `voting_end_time` 跟踪提案的生命周期

+++ https://github.com/cosmos/cosmos-sdk/blob/4a129832eb16f37a89e97652a669f0cdc9196ca9/proto/cosmos/gov/v1beta2/gov.proto#L42-L52

一个提案通常需要的不仅仅是一组消息来解释它的
目的但需要一些更大的理由并为感兴趣的参与者提供一种手段
讨论和辩论该提案。在大多数情况下，鼓励有一个链下
支持链上治理过程的系统。为了适应这种情况，一个
提案包含一个特殊的“元数据”字段，一个字节数组，可用于
为提案添加上下文。 `metadata` 字段允许自定义使用网络，但是，
预计该字段包含 URL 或使用系统的某种形式的 CID，例如
[IPFS](https://docs.ipfs.io/concepts/content-addressing/)。为了支持的情况
跨网络的互操作性，SDK 建议“元数据”代表
以下`JSON`模板: 

```json
{
  "title": "...",
  "description": "...",
  "forum": "...", // a link to the discussion platform (i.e. Discord)
  "other": "..." // any extra data that doesn't correspond to the other fields
}
```

这使得客户端更容易支持多个网络。

### 编写一个使用治理的模块

链或您可能想要的单个模块的许多方面
使用治理来执行例如更改各种参数。这很简单
去做。首先，写出你的消息类型和 `MsgServer` 实现。添加一个
`authority` 字段给 keeper ，它将在构造函数中填充
治理模块帐户:`govKeeper.GetGovernanceAccount().GetAddress()`。那么对于
`msg_server.go` 中的方法，对签名者发送的消息执行检查
匹配“权威”。这将阻止任何用户执行该消息。

## 参数和基本类型

`Parameters` 定义了运行投票的规则。只能有
是任何给定时间的一个活动参数集。如果治理想要改变一个
参数集，用于修改值或添加/删除参数字段，一个新的
必须创建参数集并且前一个被渲染为非活动状态。

### 存款参数

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L127-L145

### 投票参数

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L147-L156

### TallyParams

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L158-L183

参数存储在全局`GlobalParams` KVStore 中。

此外，我们还介绍了一些基本类型: 

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

## Deposit

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/gov.proto#L43-L53

## ValidatorGovInfo

This type is used in a temp map when tallying

```go
  type ValidatorGovInfo struct {
    Minus     sdk.Dec
    Vote      Vote
  }
```

## 存储

_Stores 是 multi-store 中的 KVStores。找存储的关键是第一
列表中的参数_`

我们将使用一个 KVStore `Governance` 来存储两个映射:

- 从 `proposalID|'proposal'` 到 `Proposal` 的映射。
- 从 `proposalID|'addresses'|address` 到 `Vote` 的映射。这种映射允许
  我们查询所有对该提案进行投票的地址以及他们的投票
  对 `proposalID:addresses` 进行范围查询。

出于伪代码的目的，以下是我们将用于在存储中读取或写入的两个函数:

- `load(StoreKey, Key)`:检索存储在多存储中键`StoreKey`的存储中存储在键`Key`中的项目
- `store(StoreKey, Key, value)`:在多存储中的键`StoreKey`中找到的存储中的键`Key`中写入值`Value`

## 提案处理队列

**存储:**

- `ProposalProcessingQueue`:一个队列 `queue[proposalID]` 包含所有
  达到“MinDeposit”的提案的“ProposalIDs”。在每个“EndBlock”期间，
  所有已到投票期结束的提案都将被处理。
  为了处理完成的提案，应用程序会统计投票数，计算
  每个验证者的投票，并检查验证者集中的每个验证者是否有
  投票。如果提案被接受，押金将被退还。最后，提案
  内容 `Handler` 被执行。

以及“ProposalProcessingQueue”的伪代码: 

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
