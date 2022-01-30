# 状态

## 水池

池用于跟踪债券面额的保税和非保税代币供应。

## LastTotalPower

LastTotalPower 跟踪在前一个结束块期间记录的绑定令牌总量。
以“Last”为前缀的存储条目必须保持不变，直到 EndBlock。

- LastTotalPower: `0x12 -> ProtocolBuffer(sdk.Int)`

## 参数

params 是一个模块范围的配置结构，用于存储系统参数
并定义了 staking 模块的整体功能。

- 参数:`Paramsspace("staking") -> legacy_amino(params)`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.1/proto/cosmos/staking/v1beta1/staking.proto#L230-L241

## 验证器

验证器可以具有以下三种状态之一

- `Unbonded`:验证器不在活动集中。他们不能签署区块，也不能赚取
  奖励。他们可以接待代表团。
- `Bonded`":一旦验证器收到足够的绑定代币，他们就会自动加入
  [`EndBlock`](./05_end_block.md#validator-set-changes) 期间的活动集，并且它们的状态更新为“已绑定”。
  他们正在签署区块并获得奖励。他们可以接待更多的代表团。
  他们可能会因不当行为而受到惩罚。解除其委托的该验证者的委托人
  必须等待 UnbondingTime 的持续时间，一个特定于链的参数，在此期间
  如果犯下这些罪行，它们仍然可以因源验证器的罪行而受到惩罚
  在代币被绑定的时间段内。
- `Unbonding`:当验证者离开活动集时，无论是出于选择还是由于削减、监禁或
  墓碑，开始解除他们所有代表团的束缚。然后所有代表团必须等待 UnbondingTime
  在他们的代币从“BondedPool”转移到他们的账户之前。

验证器对象应该主要由
`OperatorAddr`，验证者操作者的 SDK 验证者地址。二
每个验证器对象维护额外的索引，以实现
削减和验证器集更新所需的查找。第三个特殊索引
(`LastValidatorPower`) 也保持不变，但保持不变
在每个区块中，与反映验证者的前两个​​索引不同
块内的记录。 

- 验证器:`0x21 | OperatorAddrLen (1 字节) | OperatorAddr -> ProtocolBuffer(validator)`
- ValidatorsByConsAddr: `0x22 | ConsAddrLen (1 字节) | ConsAddr -> OperatorAddr`
- ValidatorsByPower:`0x23 | BigEndian(ConsensusPower) | OperatorAddrLen (1 字节) | OperatorAddr -> OperatorAddr`
- LastValidatorsPower:`0x11 | OperatorAddrLen (1 字节) | OperatorAddr -> ProtocolBuffer(ConsensusPower)`

`Validators` 是主要索引 - 它确保每个操作符只能有一个
关联的验证器，该验证器的公钥可以在
未来。委托人可以引用验证人的不可变操作符，无需
关注不断变化的公钥。

`ValidatorByConsAddr` 是一个额外的索引，可以进行斜线查找。
Tendermint 上报证据时，会提供验证人地址，所以这个
需要 map 才能找到操作符。请注意，`ConsAddr` 对应于
可以从验证器的“ConsPubKey”派生的地址。

`ValidatorsByPower` 是一个额外的索引，它提供了一个排序列表
潜在的验证器来快速确定当前的活动集。这里
默认情况下，ConsensusPower 是 validator.Tokens/10^6。请注意，所有验证器
`Jailed` 为 true 的地方不存储在这个索引中。

`LastValidatorsPower` 是一个特殊的索引，它提供了历史列表
最后一个区块的绑定验证器。该索引在块期间保持不变，但
在 [`EndBlock`](./05_end_block.md) 中发生的验证器集更新过程中更新。

每个验证器的状态都存储在一个“验证器”结构中:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L65-L99

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L24-L63

## 代表团

委托通过结合`DelegatorAddr`(委托人的地址)来识别
使用 `ValidatorAddr` 委托人在存储中被索引如下:

- 委托:`0x31 | DelegatorAddrLen (1 字节) |委托人地址 | ValidatorAddrLen(1 字节)| ValidatorAddr -> ProtocolBuffer(delegation)`

利益相关者可以将硬币委托给验证者；在这种情况下他们的
资金保存在“委托”数据结构中。它归一个人所有
委托人，并与一个验证人的份额相关联。发件人
交易是债券的所有者。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L159-L170

### 委托人股份

当一个人将代币委托给验证者时，他们会根据
动态汇率，根据委托给代币的代币总数计算如下
验证器和迄今为止发行的股票数量:

`每个令牌的份额 = validator.TotalShares()/validator.Tokens()`

仅收到的股份数量存储在委托条目中。当委托人然后
Undelegates，他们收到的代币数量是根据他们当前的股份数量计算的
持有和反向汇率:

`每股代币数 = validator.Tokens()/validatorShares()`

这些“股份”只是一种会计机制。它们不是可替代的资产。的原因
这种机制是为了简化围绕削减的核算。而不是反复削减
每个委托条目的代币，而不是验证者的总绑定代币可以被削减，
有效地降低了每份已发行的委托人股份的价值。 

## 解除绑定委托

“委托”中的股份可以不绑定，但它们必须存在一段时间
一个“UnbondingDelegation”，如果拜占庭行为是可以减少的
检测到。

`UnbondingDelegation` 在存储中被索引为:

- 解除绑定委托:`0x32 | DelegatorAddrLen (1 字节) |委托人地址 | ValidatorAddrLen(1 字节)| ValidatorAddr -> ProtocolBuffer(unbondingDelegation)`
- UnbondingDelegationsFromValidator:`0x33 | ValidatorAddrLen(1 字节)|验证器地址 | DelegatorAddrLen (1 字节) | DelegatorAddr -> nil`

此处的第一张地图用于查询，以查找所有未绑定的委托
给定的委托人，而第二张地图用于斜线，以查找所有
与给定验证器关联的解除绑定委托需要
削减。

每次启动解除绑定时都会创建一个 UnbondingDelegation 对象。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L172-L198

## 重新授权

价值“委托”的保税代币可以立即从
源验证器到不同的验证器(目标验证器)。然而当
发生这种情况时，他们必须在“重新委托”对象中进行跟踪，从而他们的
如果他们的代币导致了拜占庭故障，则可以削减股票
由源验证器提交。

`Redelegation` 在存储中被索引为:

- 重新委托:`0x34 | DelegatorAddrLen (1 字节) |委托人地址 | ValidatorAddrLen(1 字节)| ValidatorSrcAddr | ValidatorDstAddr -> ProtocolBuffer(redelegation)`
- RedelelegationsBySrc:`0x35 | ValidatorSrcAddrLen(1 字节)| ValidatorSrcAddr | ValidatorDstAddrLen(1 字节)| ValidatorDstAddr | DelegatorAddrLen (1 字节) | DelegatorAddr -> nil`
- RedelelegationsByDst:`0x36 | ValidatorDstAddrLen(1 字节)| ValidatorDstAddr | ValidatorSrcAddrLen(1 字节)| ValidatorSrcAddr | DelegatorAddrLen (1 字节) | DelegatorAddr -> nil`

此处的第一张地图用于查询，以查找给定的所有重新授权
委托人。第二个地图用于基于`ValidatorSrcAddr`的斜线，
而第三张地图用于基于`ValidatorDstAddr`的削减。

每次重新委派发生时都会创建一个重新委派对象。阻止
在以下情况下可能不会发生“重新授权跳跃”重新授权:

-(重新)委托人已经有另一个不成熟的重新委托正在进行中
  带有验证器的目的地(我们称之为“验证器 X”)
- 并且，(重新)委托人正在尝试创建_新_重新委托
  这个新的重新授权的源验证器是“Validator X”。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L200-L228

## 队列

所有队列对象都按时间戳排序。任何队列中使用的时间是
首先四舍五入到最接近的纳秒然后排序。可排序的时间格式
used 是对 RFC3339Nano 的轻微修改并使用格式字符串
`"2006-01-02T15:04:05.000000000"`。特别是这种格式:

- 右填充全零
- 删除时区信息(使用 UTC)

在所有情况下，存储的时间戳代表队列的成熟时间
元素。 

### UnbondingDelegationQueue

为了跟踪解绑授权的进展，解绑授权
保留代表团队列。

- 解除绑定委托:`0x41 |格式(时间)-> []DVPair`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L123-L133

### 重新授权队列

为了跟踪重新授权的进度，重新授权队列是
保留。

- 重新委托队列:`0x42 |格式(时间)-> []DVVTriplet`

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L140-L152

### 验证器队列

为了跟踪解除绑定验证器的进度，验证器
队列被保留。

- ValidatorQueueTime:`0x43 |格式(时间)-> []sdk.ValAddress`

作为每个键的存储对象是一个验证器操作员地址数组，来自
可以访问验证器对象。通常预计只有
单个验证器记录将与给定的时间戳相关联，但这是可能的
多个验证器存在于同一位置的队列中。

## 历史信息

历史信息对象在每个块中被存储和修剪，以便 staking 保持者持续存在
由 staking 模块参数定义的 `n` 最近的历史信息:`HistoricalEntries`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/staking/v1beta1/staking.proto#L15-L22

在每个 BeginBlock 中，Staking 管理员将保留当前的 ​​Header 和提交的验证器
`HistoricalInfo` 对象中的当前块。验证者按其地址排序，以确保
它们处于确定性顺序中。
最旧的历史条目将被修剪以确保只存在参数定义的数量
历史条目。 