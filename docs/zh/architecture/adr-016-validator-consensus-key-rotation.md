# ADR 016:验证者共识密钥轮换

## 变更日志

- 2019 年 10 月 23 日:初稿
- 2019 年 11 月 28 日:增加密钥轮换费

## 语境

为了更安全的验证者密钥管理策略(例如 https://github.com/tendermint/tendermint/issues/1136)，验证者共识密钥轮换功能已经被讨论和请求了很长时间。因此，我们建议在 Cosmos SDK 上使用最简单的验证器共识密钥轮换实现形式之一。

我们不需要对 Tendermint 中的共识逻辑进行任何更新，因为 Tendermint 没有任何共识密钥和验证者操作者密钥的映射信息，这意味着从 Tendermint 的角度来看，验证者的共识密钥轮换只是替换了另一个的共识密钥。

另外，应该注意的是，这个 ADR 只包括最简单的共识密钥轮换形式，没有考虑多个共识密钥的概念。这种多重共识密钥的概念仍将是 Tendermint 和 Cosmos SDK 的长期目标。

## 决定

### 共识密钥轮换的伪过程

- 创建新的随机共识密钥。
- 使用“MsgRotateConsPubKey”创建和广播交易，该交易声明新的共识密钥现在与验证器操作符耦合，并带有来自验证器操作符密钥的签名。
- 旧的共识密钥在链上密钥映射状态更新后立即无法参与共识。
- 开始使用新的共识密钥进行验证。
- 当`MsgRotateConsPubKey`提交到区块链时，使用HSM和KMS的验证者应该更新HSM中的共识密钥以在高度`h`之后使用新的轮换密钥。

### 注意事项

- 共识密钥映射信息管理策略
    - 在 kvstore 中存储每个键映射更改的历史记录。
    - 状态机可以在最近的解除绑定期间搜索与给定验证器操作符配对的任意高度的相应共识密钥。
    - 状态机不需要任何超过解除绑定期的历史映射信息。
- 与 LCD 和 IBC 相关的密钥轮换成本
    - LCD和IBC在频繁的电源变化时会产生流量/计算负担
    - 在当前的 Tendermint 设计中，共识密钥轮换被视为从 LCD 或 IBC 的角度来看权力的变化
    - 因此，为了尽量减少不必要的频繁密钥轮换行为，我们限制了最近解除绑定期间的最大轮换次数，并且还应用了成倍增加的轮换费用
- 限制
    - 验证者在任何解除绑定期间都不能旋转其共识密钥超过 `MaxConsPubKeyRotations` 时间，以防止垃圾邮件。
    - 参数可以由治理决定并存储在创世文件中。
- 密钥轮换费
    - 验证者应该支付`KeyRotationFee`来轮换共识密钥，计算如下
    - `KeyRotationFee` = (max(`VotingPowerPercentage` *100, 1)* `InitialKeyRotationFee`) * 2^(最近解绑期间`ConsPubKeyRotationHistory`中的旋转次数)
- 证据模块
    - 证据模块可以从slashing keeper中搜索任何高度的相应共识密钥，以便它可以决定对于给定的高度应该使用哪个共识密钥。
- abci.ValidatorUpdate
    -tendermint 已经能够通过 ABCI 通信(`ValidatorUpdate`)更改共识密钥。
    - 验证者共识密钥更新可以通过将功率更改为零来创建新的 + 删除旧的来完成。
    - 因此，我们希望我们甚至根本不需要更改tendermint 代码库来实现此功能。
- `staking` 模块中的新创世参数
    - `MaxConsPubKeyRotations`:在最近的解除绑定期间验证者可以执行的最大轮换次数。建议使用默认值 10(第 11 次密钥轮换将被拒绝)
    - `InitialKeyRotationFee`:在最近解绑期间没有发生密钥轮换时的初始密钥轮换费用。建议使用默认值 1atom(最近解绑期间第一次密钥轮换的费用为 1atom) 

### Workflow

1. 验证者生成一个新的共识密钥对。
2. 验证器使用他们的操作符密钥和新的 ConsPubKey 生成并签署一个 `MsgRotateConsPubKey` tx 

    ```go
    type MsgRotateConsPubKey struct {
        ValidatorAddress  sdk.ValAddress
        NewPubKey         crypto.PubKey
    }
    ```

3.`handleMsgRotateConsPubKey`获取`MsgRotateConsPubKey`，调用`RotateConsPubKey`并带有emits事件
4.`RotateConsPubKey`
     - 检查 `NewPubKey` 是否在 `ValidatorsByConsAddr` 上没有重复
     - 通过迭代 `ConsPubKeyRotationHistory` 检查验证器是否不超过参数 `MaxConsPubKeyRotations`
     - 检查签名账户是否有足够的余额来支付`KeyRotationFee`
     - 向社区基金支付`KeyRotationFee`
     - 覆盖 `validator.ConsPubKey` 中的 `NewPubKey`
     - 删除旧的`ValidatorByConsAddr`
     -`NewPubKey` 的`SetValidatorByConsAddr`
     - 添加 `ConsPubKeyRotationHistory` 用于跟踪旋转 

    ```go
    type ConsPubKeyRotationHistory struct {
        OperatorAddress         sdk.ValAddress
        OldConsPubKey           crypto.PubKey
        NewConsPubKey           crypto.PubKey
        RotatedHeight           int64
    }
    ```

5.`ApplyAndReturnValidatorSetUpdates`检查是否有`ConsPubKeyRotationHistory`和`ConsPubKeyRotationHistory.RotatedHeight == ctx.BlockHeight()`，如果有，生成2个`ValidatorUpdate`，一个用于删除验证器，一个用于创建新验证器 

    ```go
    abci.ValidatorUpdate{
        PubKey: tmtypes.TM2PB.PubKey(OldConsPubKey),
        Power:  0,
    }

    abci.ValidatorUpdate{
        PubKey: tmtypes.TM2PB.PubKey(NewConsPubKey),
        Power:  v.ConsensusPower(),
    }
    ```

6.在`AllocateTokens`的`previousVotes`迭代逻辑中，`previousVote`使用`OldConsPubKey`匹配`ConsPubKeyRotationHistory`，并替换validator进行令牌分配
7. 将 `ValidatorSigningInfo` 和 `ValidatorMissedBlockBitArray` 从 `OldConsPubKey` 迁移到 `NewConsPubKey`

- 注意:以上所有功能都应在`staking` 模块中实现。 

## Status

Proposed

## Consequences

### Positive

- 验证者可以立即或定期轮换他们的共识密钥以获得更好的安全策略
- 提高了针对远程攻击的安全性(https://nearprotocol.com/blog/long-range-attacks-and-a-new-fork-choice-rule)，因为验证器丢弃了旧的共识密钥 

### Negative

- Slash 模块需要更多的计算，因为它需要为每个高度查找相应的验证器共识密钥
- 频繁的密钥轮换会降低轻客户端二分的效率 
### Neutral

## References

- on tendermint repo : https://github.com/tendermint/tendermint/issues/1136
- on cosmos-sdk repo : https://github.com/cosmos/cosmos-sdk/issues/5231
- about multiple consensus keys : https://github.com/tendermint/tendermint/issues/1758#issuecomment-545291698
