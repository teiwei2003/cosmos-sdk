# メッセージ

## 提案の提出

提案は、`MsgSubmitProposal`トランザクションを介して任意のアカウントで送信できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L24-L39

`MsgSubmitProposal`メッセージの`Content`には、ガバナンスモジュールに適切なルーターが設定されている必要があります。

**状態の変更:**

- 新しい `proposalID`を生成します
- 新しい `プロポーザル`を作成します
- `Proposal`の属性を初期化します
- `InitialDeposit`によって送信者の残高を減らします
- `MinDeposit`に達した場合：
     - `ProposalProcessingQueue`で `proposalID`をプッシュします
- `InitialDeposit`を `Proposer`からガバナンス` ModuleAccount`に転送します

`MsgSubmitProposal`トランザクションは、次の擬似コードに従って処理できます。

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

## 保証金

プロポーザルが送信されると、`Proposal.TotalDeposit <ActiveParam.MinDeposit`の場合、Atomホルダーは` MsgDeposit`トランザクションを送信して、プロポーザルのデポジットを増やすことができます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L61-L72

**状態の変更：**

- `預金 `によって送信者の残高を減らします
- `proposal.Deposits`に送信者の `deposit`を追加します
- 送信者の `deposit`によって` proposal.TotalDeposit`を増やします
- `MinDeposit`に達した場合：
     - `ProposalProcessingQueueEnd`で `proposalID`をプッシュします
- `Deposit`を `proposer`からガバナンス` ModuleAccount`に転送します

`MsgDeposit`トランザクションが有効になるには、いくつかのチェックを行う必要があります。
これらのチェックは、次の擬似コードで概説されています。 

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

`ActiveParam.MinDeposit`に達すると、投票期間が始まります。 そこから、結合されたAtom保有者は、「MsgVote」トランザクションを送信して、提案に投票することができます。。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/gov/v1beta1/tx.proto#L46-L56

**状態の変更：**

- 送信者の「投票」を記録する

_注：このメッセージのガスコストは、EndBlockerでの投票の将来の集計を考慮に入れる必要があります_

次は、`MsgVote`トランザクションの方法の擬似コードの概要です。
処理：

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
