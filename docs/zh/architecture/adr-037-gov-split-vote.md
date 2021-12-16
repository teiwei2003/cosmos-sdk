# ADR 037:治理分裂投票

## 变更日志

- 2020/10/28:初稿

## 地位

公认

## 摘要

该 ADR 定义了对治理模块的修改，允许权益人将他们的投票分成几个投票选项。例如，它可以用 70% 的投票权投赞成票，用 30% 的投票权投反对票。

## 语境

目前，一个地址只能用一个选项(是/否/弃权/否决权)进行投票，并在该选项背后使用他们的全部投票权。

但是，拥有该地址的实体通常可能不是一个人。例如，一家公司可能有不同的利益相关者想要以不同的方式投票，因此允许他们分配投票权是有意义的。另一个示例用例是交换。许多中心化交易所通常将其用户的一部分代币押在他们的托管中。目前，他们无法进行“传递投票”并赋予用户对其代币的投票权。然而，通过这个系统，交易所可以对用户的投票偏好进行投票，然后根据投票结果按比例在链上投票。

## 决定

我们将投票结构修改为 

```
type WeightedVoteOption struct {
  Option string
  Weight sdk.Dec
}

type Vote struct {
  ProposalID int64
  Voter      sdk.Address
  Options    []WeightedVoteOption
}
```

为了向后兼容，我们引入了 `MsgVoteWeighted`，同时保留了 `MsgVote`。 

```
type MsgVote struct {
  ProposalID int64
  Voter      sdk.Address
  Option     Option
}

type MsgVoteWeighted struct {
  ProposalID int64
  Voter      sdk.Address
  Options    []WeightedVoteOption
}
```

`MsgVoteWeighted` 结构的 `ValidateBasic` 将要求

1. 所有利率的总和等于 1.0
2.没有重复选项

治理计数函数将迭代投票中的所有选项，并将选民投票权的结果 * 该选项的比率添加到计数中。 

```
tally() {
    results := map[types.VoteOption]sdk.Dec

    for _, vote := range votes {
        for i, weightedOption := range vote.Options {
            results[weightedOption.Option] += getVotingPower(vote.voter) * weightedOption.Weight
        }
    }
}
```

用于创建多选项投票的 CLI 命令如下: 

```sh
simd tx gov vote 1 "yes=0.6,no=0.3,abstain=0.05,no_with_veto=0.05" --from mykey
```

To create a single-option vote a user can do either

```
simd tx gov vote 1 "yes=1" --from mykey
```

or

```sh
simd tx gov vote 1 yes --from mykey
```

to maintain backwards compatibility.

## Consequences

### Backwards Compatibility

- 以前的 VoteMsg 类型将保持不变，因此客户不必更新他们的程序，除非他们想要支持 WeightedVoteMsg 功能。
- 当从状态查询 Vote 结构时，它的结构会有所不同，因此想要显示所有选民及其各自投票的客户必须处理新格式以及单个选民可以拥有分裂选票的事实。
- 查询tally 函数的结果应该与客户端具有相同的API。 

### Positive

- 可以使代表多个利益相关者的地址的投票过程更加准确，通常是一些最大的地址。 

### Negative

- 比简单的投票更复杂，因此可能更难向用户解释。 但是，这在很大程度上得到了缓解，因为该功能是可选的。 

### Neutral

- 治理理货功能的变化相对较小。 
