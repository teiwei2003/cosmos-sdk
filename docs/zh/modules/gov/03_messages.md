# 消息

## 提案提交

任何帐户都可以通过“MsgSubmitProposal”提交提案
交易。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L24-L39

`MsgSubmitProposal` 消息的 `Content` 必须有一个合适的路由器
在治理模块中设置。

**状态修改:**

- 生成新的`proposalID`
- 创建新的“提案”
- 初始化`Proposal`的属性
- 通过`InitialDeposit`减少发件人的余额
- 如果达到`MinDeposit`:
     - 在`ProposalProcessingQueue`中推送`proposalID`
- 将 `InitialDeposit` 从 `Proposer` 转移到治理 `ModuleAccount`

一个 `MsgSubmitProposal` 事务可以按照以下方式处理
伪代码。 

```go
// PSEUDOCODE //
// Check if MsgSubmitProposal is valid. If it is, create proposal //

upon receiving txGovSubmitProposal from sender do

  if !correctlyFormatted(txGovSubmitProposal)
    // check if proposal is correctly formatted and the messages have routes to other modules. Includes fee payment.
    throw

  initialDeposit = txGovSubmitProposal.InitialDeposit
  if (initialDeposit.Atoms <= 0) OR (sender.AtomBalance < initialDeposit.Atoms)
    // InitialDeposit is negative or null OR sender has insufficient funds
    throw

  if (txGovSubmitProposal.Type != ProposalTypePlainText) OR (txGovSubmitProposal.Type != ProposalTypeSoftwareUpgrade)

  sender.AtomBalance -= initialDeposit.Atoms

  depositParam = load(GlobalParams, 'DepositParam')

  proposalID = generate new proposalID
  proposal = NewProposal()

  proposal.Title = txGovSubmitProposal.Title
  proposal.Description = txGovSubmitProposal.Description
  proposal.Type = txGovSubmitProposal.Type
  proposal.TotalDeposit = initialDeposit
  proposal.SubmitTime = <CurrentTime>
  proposal.DepositEndTime = <CurrentTime>.Add(depositParam.MaxDepositPeriod)
  proposal.Deposits.append({initialDeposit, sender})
  proposal.Submitter = sender
  proposal.YesVotes = 0
  proposal.NoVotes = 0
  proposal.NoWithVetoVotes = 0
  proposal.AbstainVotes = 0
  proposal.CurrentStatus = ProposalStatusOpen

  store(Proposals, <proposalID|'proposal'>, proposal) // Store proposal in Proposals mapping
  return proposalID
```

## 订金

提交提案后，如果
`Proposal.TotalDeposit < ActiveParam.MinDeposit`，Atom 持有者可以发送
`MsgDeposit` 交易以增加提案的存款。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L61-L72

**状态修改:**

- 通过`存款`减少发件人的余额
- 在`proposal.Deposits`中添加发件人的`deposit`
- 通过发送者的`deposit`增加`proposal.TotalDeposit`
- 如果达到`MinDeposit`:
     - 在`ProposalProcessingQueueEnd`中推送`proposalID`
- 将`Deposit`从`proposer`转移到治理`ModuleAccount`

`MsgDeposit` 交易必须经过多次检查才能有效。
以下伪代码概述了这些检查。 

```go
// PSEUDOCODE //
// Check if MsgDeposit is valid. If it is, increase deposit and check if MinDeposit is reached

upon receiving txGovDeposit from sender do
  // check if proposal is correctly formatted. Includes fee payment.

  if !correctlyFormatted(txGovDeposit)
    throw

  proposal = load(Proposals, <txGovDeposit.ProposalID|'proposal'>) // proposal is a const key, proposalID is variable

  if (proposal == nil)
    // There is no proposal for this proposalID
    throw

  if (txGovDeposit.Deposit.Atoms <= 0) OR (sender.AtomBalance < txGovDeposit.Deposit.Atoms) OR (proposal.CurrentStatus != ProposalStatusOpen)

    // deposit is negative or null
    // OR sender has insufficient funds
    // OR proposal is not open for deposit anymore

    throw

  depositParam = load(GlobalParams, 'DepositParam')

  if (CurrentBlock >= proposal.SubmitBlock + depositParam.MaxDepositPeriod)
    proposal.CurrentStatus = ProposalStatusClosed

  else
    // sender can deposit
    sender.AtomBalance -= txGovDeposit.Deposit.Atoms

    proposal.Deposits.append({txGovVote.Deposit, sender})
    proposal.TotalDeposit.Plus(txGovDeposit.Deposit)

    if (proposal.TotalDeposit >= depositParam.MinDeposit)
      // MinDeposit is reached, vote opens

      proposal.VotingStartBlock = CurrentBlock
      proposal.CurrentStatus = ProposalStatusActive
      ProposalProcessingQueue.push(txGovDeposit.ProposalID)

  store(Proposals, <txGovVote.ProposalID|'proposal'>, proposal)
```

## Vote

一旦达到`ActiveParam.MinDeposit`，投票期开始。 从那里，
绑定的 Atom 持有者能够发送“MsgVote”交易来投下他们的
对该提案进行投票。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L46-L56

**状态修改:**

- 记录发件人的“投票”

_注意:此消息的 Gas 成本必须考虑到 EndBlocker 中未来的投票统计_

接下来是“MsgVote”交易方式的伪代码概述
处理: 

```go
  // PSEUDOCODE //
  // Check if MsgVote is valid. If it is, count vote//

  upon receiving txGovVote from sender do
    // check if proposal is correctly formatted. Includes fee payment.

    if !correctlyFormatted(txGovDeposit)
      throw

    proposal = load(Proposals, <txGovDeposit.ProposalID|'proposal'>)

    if (proposal == nil)
      // There is no proposal for this proposalID
      throw


    if  (proposal.CurrentStatus == ProposalStatusActive)


        // Sender can vote if
        // Proposal is active
        // Sender has some bonds

        store(Governance, <txGovVote.ProposalID|'addresses'|sender>, txGovVote.Vote)   // Voters can vote multiple times. Re-voting overrides previous vote. This is ok because tallying is done once at the end.
```
