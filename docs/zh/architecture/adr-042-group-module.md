# ADR 042:组模块

## 变更日志

- 2020/04/09:初稿

## 地位

草稿

## 摘要

该 ADR 定义了“x/group”模块，该模块允许创建和管理链上多重签名帐户，并基于可配置的决策策略启用对消息执行的投票。

## 语境

Cosmos SDK 遗留的氨基多重签名机制有一定的局限性:

- 密钥轮换是不可能的，尽管这可以通过 [account rekeying](adr-034-account-rekeying.md) 解决。
- 无法更改阈值。
- UX 对于非技术用户来说很麻烦 ([#5661](https://github.com/cosmos/cosmos-sdk/issues/5661))。
- 它需要 `legacy_amino` 符号模式 ([#8141](https://github.com/cosmos/cosmos-sdk/issues/8141))。

虽然组模块并不意味着要完全替代当前的多重签名帐户，但它提​​供了一种解决上述限制的方法，具有更灵活的密钥管理系统，可以在其中添加、更新或删除密钥，以及可配置的阈值。
它旨在与其他访问控制模块一起使用，例如 [`x/feegrant`](./adr-029-fee-grant-module.md) ans [`x/authz`](adr-030-authz-module .md) 以简化个人和组织的密钥管理。

group 模块的概念证明可以在 https://github.com/regen-network/regen-ledger/tree/master/proto/regen/group/v1alpha1 和 https://github.com/regen-网络/regen-ledger/tree/master/x/group。

## 决定

我们建议将 `x/group` 模块与其支持的 [ORM/Table Store 包](https://github.com/regen-network/regen-ledger/tree/master/orm) ([#7098](https ://github.com/cosmos/cosmos-sdk/issues/7098)) 进入 Cosmos SDK 并在此处继续开发。 ORM 包将有一个专用的 ADR。

### 团体

组是具有相关权重的帐户组合。它不是
一个帐户并且没有余额。它本身没有任何
某种投票权重或决策权重。
组成员可以使用不同的决策策略通过组帐户创建提案并对其进行投票。

它有一个“admin”账户，可以管理群组成员，更新群组
元数据并设置新的管理员。 

```proto
message GroupInfo {

   //group_id is the unique ID of this group.
    uint64 group_id = 1;

   //admin is the account address of the group's admin.
    string admin = 2;

   //metadata is any arbitrary metadata to attached to the group.
    bytes metadata = 3;

   //version is used to track changes to a group's membership structure that
   //would break existing proposals. Whenever a member weight has changed,
   //or any member is added or removed, the version is incremented and will
   //invalidate all proposals from older versions.
    uint64 version = 4;

   //total_weight is the sum of the group members' weights.
    string total_weight = 5;
}
```

```proto
message GroupMember {

   //group_id is the unique ID of the group.
    uint64 group_id = 1;

   //member is the member data.
    Member member = 2;
}

//Member represents a group member with an account address,
//non-zero weight and metadata.
message Member {

   //address is the member's account address.
    string address = 1;

   //weight is the member's voting weight that should be greater than 0.
    string weight = 2;

   //metadata is any arbitrary metadata to attached to the member.
    bytes metadata = 3;
}
```

### 组帐户

组帐户是与组和决策策略相关联的帐户。
组帐户确实有余额。

组帐户是从组中抽象出来的，因为单个组可能有
不同类型动作的多种决策策略。 管理组
与决策策略分开的成员资格导致最少的开销
并在不同的政策中保持成员资格一致。 那个图案
建议为给定组设置一个主组帐户，
然后创建具有不同决策策略的单独组帐户
并将所需的权限从主帐户委派给
那些使用 [`x/authz` 模块](adr-030-authz-module.md) 的“子账户”。 

```proto
message GroupAccountInfo {

   //address is the group account address.
    string address = 1;

   //group_id is the ID of the Group the GroupAccount belongs to.
    uint64 group_id = 2;

   //admin is the account address of the group admin.
    string admin = 3;

   //metadata is any arbitrary metadata of this group account.
    bytes metadata = 4;

   //version is used to track changes to a group's GroupAccountInfo structure that
   //invalidates active proposal from old versions.
    uint64 version = 5;

   //decision_policy specifies the group account's decision policy.
    google.protobuf.Any decision_policy = 6 [(cosmos_proto.accepts_interface) = "DecisionPolicy"];
}
```

与组管理员类似，组帐户管理员可以更新其元数据、决策策略或设置新的组帐户管理员。

群组帐户也可以是管理员或群组成员。
例如，组管理员可以是另一个可以“选举”成员的组帐户，也可以是选举自己的同一个组。

### 决策政策

决策策略是组成员可以投票的机制
提案。

所有决策策略都应该有一个最小和最大投票窗口。
最小投票窗口是必须按顺序通过的最短持续时间
一个可能通过的提案，它可能被设置为 0。最大投票数
窗口是提案可以被投票和执行的最长时间，如果
它在关闭之前获得了足够的支持。
这两个值都必须小于全链最大投票窗口参数。

我们定义了所有决策策略都必须实现的 `DecisionPolicy` 接口: 

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

#### 阈值决策策略

阈值决策策略基于计数定义了最低支持票数 (_yes_)
选民权重，以通过提案。 为了
这个决定政策，弃权和否决被视为不支持(_no_)。 

```proto
message ThresholdDecisionPolicy {

   //threshold is the minimum weighted sum of support votes for a proposal to succeed.
    string threshold = 1;

   //voting_period is the duration from submission of a proposal to the end of voting period
   //Within this period, votes and exec messages can be submitted.
    google.protobuf.Duration voting_period = 2 [(gogoproto.nullable) = false];
}
```

### Proposal

群组的任何成员都可以提交提案以供群组帐户决定。
提案由一组 `sdk.Msg` 组成，如果提案
通过以及与提案相关的任何元数据。 这些 `sdk.Msg` 被验证为 `Msg/CreateProposal` 请求验证的一部分。 他们还应该将他们的签名者设置为组帐户。

在内部，提案还跟踪:

- 它当前的“状态”:已提交、关闭或中止
- 它的“结果”:未完成、接受或拒绝
- 它的“VoteState”以“Tally”的形式，根据新投票和执行提案时计算。 

```proto
//Tally represents the sum of weighted votes.
message Tally {
    option (gogoproto.goproto_getters) = false;

   //yes_count is the weighted sum of yes votes.
    string yes_count = 1;

   //no_count is the weighted sum of no votes.
    string no_count = 2;

   //abstain_count is the weighted sum of abstainers.
    string abstain_count = 3;

   //veto_count is the weighted sum of vetoes.
    string veto_count = 4;
}
```

### 投票

小组成员可以对提案进行投票。投票时有四种选择 - 是、否、弃权和否决。不是
所有的决策政策都会支持他们。投票可以包含一些可选的元数据。
在当前的实现中，投票窗口在提案后立即开始
被提交。

如果需要，投票会在内部更新提案“VoteState”以及“Status”和“Result”。

### 执行提案

在当前设计中，链不会自动执行提案，
而是用户必须提交一个 `Msg/Exec` 事务来尝试执行
基于当前投票和决策政策的提案。未来的升级可能
自动执行此操作并让组帐户(或费用授予者)支付。

#### 更改组成员身份

在目前的实现中，提交提案后更新群组或群组账号会使其失效。如果有人调用`Msg/Exec`，它只会失败，最终会被垃圾收集。

### 当前实现的注意事项

本节概述了组模块概念验证中使用的当前实现，但这可能会发生变化和迭代。

#### ORM

[ORM 包](https://github.com/cosmos/cosmos-sdk/discussions/9156) 定义了组模块中使用的表、序列和二级索引。

组作为“groupTable”的一部分存储在状态中，“group_id”是一个自动递增的整数。组成员存储在“groupMemberTable”中。

组帐户存储在“groupAccountTable”中。组帐户地址是基于自动递增整数生成的，该整数用于将组模块 `RootModuleKey` 派生为 `DerivedModuleKey`，如 [ADR-033](adr-033-protobuf-inter-module-comm .md#modulekeys-and-moduleids)。群组帐户通过`x/auth` 添加为新的`ModuleAccount`。

提案使用“Proposal”类型存储为“proposalTable”的一部分。 `proposal_id` 是一个自动递增的整数。

投票存储在“voteTable”中。主键基于投票的`proposal_id`和`voter`账户地址。

#### ADR-033 路​​由提案消息

[ADR-033](adr-033-protobuf-inter-module-comm.md) 引入的模块间通信可用于使用与提案的组帐户对应的“DerivedModuleKey”路由提案的消息。 

## Consequences

### Positive

- 改进了多签名帐户的 UX，允许密钥轮换和自定义决策策略。 

### Negative

### Neutral

- 它使用 ADR 033，因此需要在 Cosmos SDK 中实现，但这并不意味着对现有 Cosmos SDK 模块进行任何大规模重构。
- group 模块的当前实现使用 ORM 包。

## 进一步讨论

- `/group` 和 `x/gov` 作为支持提案和投票的收敛:https://github.com/cosmos/cosmos-sdk/discussions/9066
- `x/group` 未来可能的改进:
     - 在提交时执行提案 (https://github.com/regen-network/regen-ledger/issues/288)
     - 撤回提案(https://github.com/regen-network/cosmos-modules/issues/41)
     - 使“Tally”更加灵活并支持非二元选择

## 参考

- 初始规格:
     - https://gist.github.com/aaronc/b60628017352df5983791cad30babe56#group-module
     - [#5236](https://github.com/cosmos/cosmos-sdk/pull/5236)
- 在 Cosmos SDK 中添加 `x/group` 的建议:[#7633](https://github.com/cosmos/cosmos-sdk/issues/7633) 