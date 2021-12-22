# 结束块

每个 abci 结束块调用，更新队列和验证器集的操作
更改被指定执行。

## 验证器集更改

在此过程中通过状态转换更新质押验证器集
在每个块的末尾运行。作为此过程的一部分，任何更新
验证器也会返回到 Tendermint 以包含在 Tendermint 中
验证器集，负责验证 Tendermint 消息
共识层。操作如下:

- 新的验证器集被视为最高的 `params.MaxValidators` 数量
  从“ValidatorsByPower”索引中检索的验证器
- 将先前的验证器集与新的验证器集进行比较:
    - 缺失的验证者开始解除绑定，他们的“代币”从
    `BondedPool` 到 `NotBondedPool` `ModuleAccount`
    - 新的验证者立即绑定，他们的“代币”从
    `NotBondedPool` 到 `BondedPool` `ModuleAccount`

在所有情况下，任何离开或进入绑定验证器集的验证器或
更改余额并留在绑定验证器集合中会导致更新
消息报告他们新的共识权力，该权力被传递回 Tendermint。

`LastTotalPower` 和 `LastValidatorsPower` 保存总功率的状态
和来自最后一个区块末尾的验证者权力，用于检查
`ValidatorsByPower` 中发生的变化和新的总权力，其中
在“EndBlock”期间计算。

## 队列

在 staking 中，某些状态转换不是即时的而是发生的
在一段时间内(通常是解绑期)。当这些
转换已经成熟，必须进行某些操作才能完成
状态操作。这是通过使用队列来实现的
在每个块的末尾检查/处理。

### 解除绑定验证器

当验证者被踢出绑定验证者集时(通过
被监禁，或没有足够的绑定代币)它开始解除绑定
进程及其所有代表团开始解除绑定(同时仍在
委托给这个验证器)。在这一点上，验证者被认为是一个
“非绑定验证器”，它将成熟成为“非绑定验证器”
解绑期过后。

验证者队列的每个区块都将被检查是否有成熟的解绑验证者
(即完成时间<=当前时间和完成高度<=当前
块高度)。在这一点上，任何成熟的验证器没有任何
剩余的代表团从状态中删除。对于所有其他成熟的解绑
仍然有剩余委托的验证器，`validator.Status` 是
从 `types.Unbonding` 切换到
`types.Unbonded`。

### 解除绑定委托

完成所有成熟的`UnbondingDelegations.Entries` 内的解除绑定
`UnbondingDelegations` 队列具有以下过程:

- 将余额币转移到委托人的钱包地址
- 从`UnbondingDelegation.Entries`中删除成熟的条目
- 如果没有，则从存储中删除 `UnbondingDelegation` 对象
  剩余条目。

### 重新授权

完成所有成熟的“Redelegation.Entries”的解绑
`Redelegations` 队列具有以下过程:

- 从`Redelegation.Entries`中删除成熟的条目
- 如果没有，则从存储中删除 `Redelegation` 对象
  剩余条目。 