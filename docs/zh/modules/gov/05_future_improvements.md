# 未来的改进

当前文档只描述了最小可行产品
治理模块。未来的改进可能包括:

* **`BountyProposals`:** 如果接受，`BountyProposal` 创建一个开放的
  赏金。 `BountyProposal` 指定将提供多少原子
  完成。这些 Atom 将从“储备池”中取出。之后
  `BountyProposal` 被治理接受，任何人都可以提交
  `SoftwareUpgradeProposal` 和代码来领取赏金。请注意，一旦
  `BountyProposal`被接受，`reserve pool`中的相应资金
  被锁定，以便始终可以兑现付款。为了链接一个
  `SoftwareUpgradeProposal` 到一个开放的赏金，提交者
  `SoftwareUpgradeProposal` 将使用 `Proposal.LinkedProposal` 属性。
  如果链接到开放赏金的“SoftwareUpgradeProposal”被接受
  治理，保留的资金将自动转移到
  提交者。
* **复杂的委托:** 委托人可以选择其他代表，而不是
  他们的验证器。最终，代表链总会结束
  由验证者决定，但委托人可以继承他们选择的投票
  在他们继承验证人的投票之前。其他
  换句话说，如果他们的其他人，他们只会继承他们的验证者的投票
  指定代表没有投票。
* **更好的提案审核流程:** 将分为两部分
  `proposal.Deposit`，一个用于反垃圾邮件(与 MVP 相同)，另一个用于
  奖励第三方审计师。