# 状态转换

本文档描述了与以下相关的状态转换操作:

1. [验证器](./02_state_transitions.md#validators)
2. [委托](./02_state_transitions.md#delegations)
3. [斜线](./02_state_transitions.md#slashing)

## 验证器

验证器中的状态转换在每个 [`EndBlock`](./05_end_block.md#validator-set-changes) 上执行
为了检查活动的“ValidatorSet”中的变化。

验证器可以是“Unbonded”、“Unbonding”或“Bonded”。 `未绑定`
和“Unbonding”统称为“Not Bonded”。验证者可以移动
直接在所有状态之间，除了从“Bonded”到“Unbonded”。

### 未绑定到绑定

当验证者在 ValidatorPowerIndex 中的排名超过
“LastValidator”的那个。

- 将 `validator.Status` 设置为 `Bonded`
- 将 `validator.Tokens` 从 `NotBondedTokens` 发送到 `BondedPool` `ModuleAccount`
- 从 `ValidatorByPowerIndex` 中删除现有记录
- 向 `ValidatorByPowerIndex` 添加一条新的更新记录
- 更新此验证器的 `Validator` 对象
- 如果存在，删除此验证器的任何“ValidatorQueue”记录

### 绑定到解绑

当验证者开始解除绑定过程时，会发生以下操作:

- 将 `validator.Tokens` 从 `BondedPool` 发送到 `NotBondedTokens` `ModuleAccount`
- 将 `validator.Status` 设置为 `Unbonding`
- 从 `ValidatorByPowerIndex` 中删除现有记录
- 向 `ValidatorByPowerIndex` 添加一条新的更新记录
- 更新此验证器的 `Validator` 对象
- 为这个验证器在 `ValidatorQueue` 中插入一条新记录

### 解除绑定到解除绑定

当“ValidatorQueue”对象出现时，验证器从解除绑定状态变为解除绑定状态
从保税到非保税

- 更新此验证器的 `Validator` 对象
- 将 `validator.Status` 设置为 `Unbonded`

### 入狱/出狱

当验证者被监禁时，它实际上会从 Tendermint 集合中删除。
这个过程也可以逆转。发生以下操作:

- 设置 `Validator.Jailed` 并更新对象
- 如果被监禁，从`ValidatorByPowerIndex` 中删除记录
- 如果未监禁，则将记录添加到 `ValidatorByPowerIndex`

被监禁的验证器不存在于以下任何存储中:

- 电力存储(从共识权力到地址) 

## Delegations

### Delegate

当委托发生时，验证器和委托对象都会受到影响

- 根据委托的代币和验证者的汇率确定委托人的份额
- 从发送帐户中删除令牌
- 添加共享委托对象或将它们添加到创建的验证器对象
- 添加新的委托人份额并更新`Validator`对象
- 将`delegation.Amount`从委托人的账户转移到`BondedPool`或`NotBondedPool``ModuleAccount`，具体取决于`validator.Status`是否为`Bonded`
- 从 `ValidatorByPowerIndex` 中删除现有记录
- 向 `ValidatorByPowerIndex` 添加一条新的更新记录

### 开始解绑

作为 Undelegate 和 Complete Unbonding 状态转换的一部分 Unbond
可以调用委托。

- 从委托人中减去未绑定股份
- 如果验证器是“Unbonding”或“Bonded”，则将代币添加到“UnbondingDelegation”条目中
- 如果验证人是“Unbonded”，则将代币直接发送给提款人
  帐户
- 如果没有更多共享，则更新委托或删除委托
- 如果委托是验证者的操作者并且没有更多的股份存在，则触发监狱验证者
- 更新验证器，删除委托人股份和相关代币
- 如果验证者状态为“Bonded”，则转移未绑定的“Coins”价值
  从`BondedPool` 到`NotBondedPool``ModuleAccount` 的共享
- 如果验证器未绑定并且没有更多的委托份额，则删除验证器。

### 完全解绑

对于未立即完成的取消委托，请执行以下操作
当解除绑定委托队列元素成熟时发生:

- 从 `UnbondingDelegation` 对象中删除条目
- 将代币从 `NotBondedPool` `ModuleAccount` 转移到委托人 `Account`

### 开始重新授权

重新委派会影响委派、源和目标验证器。

- 执行来自源验证器的“unbond”委托以检索未绑定股票的代币价值
- 使用未绑定的代币，将它们“委托”给目标验证器
- 如果`sourceValidator.Status` 是`Bonded`，而`destinationValidator` 不是，
  将新委托的代币从 `BondedPool` 转移到 `NotBondedPool` `ModuleAccount`
- 否则，如果 `sourceValidator.Status` 不是 `Bonded`，并且 `destinationValidator`
  是`Bonded`，将新委托的代币从`NotBondedPool`转移到`BondedPool``ModuleAccount`
- 在相关“重新授权”的新条目中记录代币数量

从重新授权开始到完成，被授权人处于“伪解绑”状态，并且仍然可以
因在重新授权开始之前发生的违规行为而受到惩罚。

### 完成重新授权

当重新授权完成时，会发生以下情况:

- 从 `Redelegation` 对象中删除条目

## 削减

### 斜线验证器

当验证器被削减时，会发生以下情况:

- 总`slashAmount` 计算为`slashFactor`(链参数)\* `TokensFromConsensusPower`，
  违规时绑定到验证器的令牌总数。
- 每次解除绑定授权和伪解除绑定重新授权，使得违规发生在解除绑定之前或
  从验证器开始的重新授权被初始余额的“slashFactor”百分比削减。
- 从重新授权和解除授权中削减的每一笔金额都从
  总斜线金额。
- 然后从 `BondedPool` 中的验证者代币中削减 `remaingSlashAmount` 或
  `NonBondedPool` 取决于验证器的状态。这减少了代币的总供应量。

如果由于任何需要提交证据的违规行为(例如双重签名)而出现斜线，则斜线
发生在包含证据的区块，而不是发生违规的区块。
否则，验证者不会被追溯削减，只有当他们被抓住时。

### 斜线解除绑定委托

当验证者被削减时，那些开始解除绑定的验证者的解除绑定委托也会被削减
违规后。来自验证者的每个解除绑定委托中的每个条目
被“slashFactor”削减。削减的金额是根据“InitialBalance”计算的
并设置上限以防止由此产生的负余额。完成(或成熟)的解除绑定不会被削减。

### 斜线重新授权

当验证器被削减时，所有在验证器之后开始的验证器的重新授权也会被削减
违规。重新授权被“slashFactor”削减。
在违规之前开始的重新授权不会被削减。
削减的金额是根据委托的“InitialBalance”计算的，上限为
防止由此产生的负余额。
成熟的再授权(已完成伪解除绑定)不会被削减。

## 如何计算份额

在任何给定的时间点，每个验证者都有一定数量的代币“T”，并有一定数量的已发行股份“S”。
每个委托人‘i’持有一定数量的股份‘S_i’。
代币的数量是委托给验证者的所有代币的总和，加上奖励，减去斜线。

委托人有权获得与其股份比例成比例的一部分基础代币。
因此，委托人`i` 有权获得验证人代币的`T * S_i / S`。

当委托人将新代币委托给验证人时，他们会收到与他们的贡献成正比的份额。
因此，当委托人`j` 委托`T_j` 代币时，他们会收到`S_j = S * T_j / T` 份额。
现在代币总数为‘T + T_j’，股份总数为‘S + S_j’。
`j 的份额比例与其贡献的总代币的比例相同:`(S + S_j) / S = (T + T_j) / T`。

一个特殊情况是初始委托，当`T = 0` 和`S = 0` 时，所以`T_j / T` 是未定义的。
对于初始委托，委托“T_j”代币的委托人“j”获得“S_j = T_j”份额。
所以一个没有收到任何奖励也没有被削减的验证者将拥有 `T = S`。 