# ADR 007:专业化组

## 变更日志

- 2019 年 7 月 31 日:初稿

## 语境

这个想法最初是为了满足
创建分散的计算机应急响应小组 (dCERT)，其
成员将由治理社区选举，并将履行职责
在紧急情况下协调社区。这种思维
可以进一步抽象为“区块链专业化”的概念
团体”。

这些组的创建是专业化能力的开始
在更广泛的区块链社区中，可用于启用某些
委派职责的级别。可以是专业化的例子
对区块链社区有益的包括:代码审计、应急响应、
代码开发等。这种类型的社区组织为
个人利益相关者按问题类型委托投票，如果将来的话
治理提案包括一个问题类型字段。

## 决定

专业化组可以大致分为以下功能
(此处包含示例): 

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

专业化小组的成员资格准入可以在广泛的范围内进行
多种机制。 最明显的例子是通过在
整个社区，但是在某些系统中，社区可能希望允许
已经在专业化小组内部选举新成员的成员，
或者社区可能会为特定专业分配权限
组任命成员到其他 3rd 党组。 天空真的是极限
关于如何构建会员准入。 我们试图捕捉
其中一些可能出现在称为“选举人”的通用界面中。 为了
作为本 ADR 的一部分，我们建议一般
提供了选举抽象(`Electionator`)以及一个基本的
该抽象的实现允许连续选择
专业化小组的成员。 

``` golang
// The Electionator abstraction covers the concept space for
// a wide variety of election kinds.  
type Electionator interface {

    // is the election object accepting votes.
    Active() bool

    // functionality to execute for when a vote is cast in this election, here
    // the vote field is anticipated to be marshalled into a vote type used
    // by an election.
    //
    // NOTE There are no explicit ids here. Just votes which pertain specifically
    // to one electionator. Anyone can create and send a vote to the electionator item
    // which will presumably attempt to marshal those bytes into a particular struct
    // and apply the vote information in some arbitrary way. There can be multiple
    // Electionators within the Cosmos-Hub for multiple specialization groups, votes
    // would need to be routed to the Electionator upstream of here.
    Vote(addr sdk.AccAddress, vote []byte)

    // here lies all functionality to authenticate and execute changes for
    // when a member accepts being elected
    AcceptElection(sdk.AccAddress)

    // Register a revoker object
    RegisterRevoker(Revoker)

    // No more revokers may be registered after this function is called
    SealRevokers()

    // register hooks to call when an election actions occur
    RegisterHooks(ElectionatorHooks)

    // query for the current winner(s) of this election based on arbitrary
    // election ruleset
    QueryElected() []sdk.AccAddress

    // query metadata for an address in the election this
    // could include for example position that an address
    // is being elected for within a group
    //
    // this metadata may be directly related to
    // voting information and/or privileges enabled
    // to members within a group.
    QueryMetadata(sdk.AccAddress) []byte
}

// ElectionatorHooks, once registered with an Electionator,
// trigger execution of relevant interface functions when
// Electionator events occur.
type ElectionatorHooks interface {
    AfterVoteCast(addr sdk.AccAddress, vote []byte)
    AfterMemberAccepted(addr sdk.AccAddress)
    AfterMemberRevoked(addr sdk.AccAddress, cause []byte)
}

// Revoker defines the function required for a membership revocation rule-set
// used by a specialization group. This could be used to create self revoking,
// and evidence based revoking, etc. Revokers types may be created and
// reused for different election types.
//
// When revoking the "cause" bytes may be arbitrarily marshalled into evidence,
// memos, etc.
type Revoker interface {
    RevokeName() string      // identifier for this revoker type
    RevokeMember(addr sdk.AccAddress, cause []byte) error
}
```

现有代码之间可能存在某种程度的共性
`x/governance` 和选举所需的功能。 这种常见的
功能应该在实现过程中抽象出来。 同样对于每个
投票实现客户端 CLI/REST 功能应该被抽象化
可重复用于多次选举。

专业化组抽象首先扩展了`Electionator`
但也进一步定义了群体的特征。 

``` golang
type SpecializationGroup interface {
    Electionator
    GetName() string
    GetDescription() string

    // general soft contract the group is expected
    // to fulfill with the greater community
    GetContract() string

    // messages which can be executed by the members of the group
    Handler(ctx sdk.Context, msg sdk.Msg) sdk.Result

    // logic to be executed at endblock, this may for instance
    // include payment of a stipend to the group members
    // for participation in the security group.
    EndBlocker(ctx sdk.Context)
}
```

## Status

> Proposed

## Consequences

### Positive

- 增加区块链的专业化能力
- 改进 `x/gov/` 中的抽象，以便它们可以与专业化组一起使用 

### Negative

- 可用于增加社区内的中心化 

### Neutral

## References

- [dCERT ADR](./adr-008-dCERT-group.md)
