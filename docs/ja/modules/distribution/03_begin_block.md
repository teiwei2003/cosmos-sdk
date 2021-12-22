# 开始块

在每个“BeginBlock”，前一个区块中收到的所有费用都转移到
分发 `ModuleAccount` 帐户。当委托人或验证人
撤回他们的奖励，他们被从`ModuleAccount` 中取出。开始时
块，对收取的费用的不同索赔更新如下:

- 前一个高度的区块提议者及其委托人获得 1% 到 5% 的费用奖励。
- 收取储备社区税。
- 剩余部分通过投票权按比例分配给所有绑定验证者

为了激励验证者等待并在区块中包含额外的预提交，区块提议者奖励是根据 Tendermint 预提交消息计算的。

## 分配方案

参数说明见[params](07_params.md)。

令 `fees` 为前一个区块收取的总费用，包括
对股权的通货膨胀奖励。所有费用都在特定模块中收取
阻止期间的帐户。在“BeginBlock”期间，它们被发送到
`“分发”` `ModuleAccount`。不会发生其他令牌发送。相反，
每个账户有权获得的奖励被存储，并且可以触发提款
通过消息 `FundCommunityPool`、`WithdrawValidatorCommission` 和
`WithdrawDelegatorReward`。

### 奖励给社区池

社区池获得`community_tax *费用`，加上之后的任何剩余灰尘
验证者获得的奖励总是四舍五入到最接近的
整数值。

### 奖励验证者

提议者获得‘费用 * baseproposerreward’的基础奖励和奖金
`fees * bonusproposerreward * P`，其中`P =(验证者的总权力与
包括预提交/总绑定验证器功率)`。预提交越多
提议者包括，较大的“P”是。 `P` 永远不能大于 `1.00`(因为
只有绑定的验证器可以提供有效的预提交)并且总是大于
`2/3`。

任何剩余的费用都分配给所有绑定的验证器，包括
提议者，与他们的共识权力成比例。

```
powFrac = validator power / total bonded validator power
proposerMul = baseproposerreward + bonusproposerreward * P
voteMul = 1 - communitytax - proposerMul
```

总的来说，提议者收到`fees * (voteMul * powFrac + ProposerMul)`。
所有其他验证器都会收到 `fees * voteMul * powFrac`。

### 对委托人的奖励

每个验证者的奖励都分配给其委托人。验证器也
有一个自我授权，被视为一个普通的授权
分布计算。

验证器设置佣金率。佣金率是灵活的，但每个
验证器设置最大速率和最大每日增量。这些最大值不能被超过，并保护委托人免受验证人佣金率突然增加的影响，以防止验证人获得所有奖励。

运营商有权获得的未兑现奖励存储在
`ValidatorAccumulatedCommission`，而委托人获得的奖励
存储在“ValidatorCurrentRewards”中。 [F1费用分配
scheme](01_concepts.md) 用于计算每个委托人的奖励，因为他们
撤回或更新他们的委托，因此不在“BeginBlock”中处理。

### 示例分布

对于这个示例分布，底层共识引擎在
与他们的权力相对于整个保税权力的比例。

所有验证者在将预提交包括在他们的提议中的表现是一样的
块。然后持有`(包括预提交)/(总绑定验证器权力)`
常数，因此验证者的摊销区块奖励为`(验证者权力/总绑定权力)*(1 - 社区税率)`
总奖励。因此，单个委托人的奖励是: 

```
(delegator proportion of the validator power / validator power) * (validator power / total bonded power)
  * (1 - community tax rate) * (1 - validator commision rate)
= (delegator proportion of the validator power / total bonded power) * (1 -
community tax rate) * (1 - validator commision rate)
```
