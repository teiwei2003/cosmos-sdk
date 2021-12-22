# 状态

## 签名信息(活力)

每个区块都包含一组验证者对前一个区块的预提交，
称为 Tendermint 提供的“LastCommitInfo”。 `LastCommitInfo` 是有效的，所以
只要它包含来自总投票权的 +2/3 的预提交。

提议者被激励在 Tendermint 的“LastCommitInfo”中包含来自所有验证者的预提交
通过收取与投票之间的差异成比例的额外费用
`LastCommitInfo` 和 +2/3 中包含的权力(参见 [费用分配](x/distribution/spec/03_begin_block.md))。 

```
type LastCommitInfo struct {
	Round int32
	Votes []VoteInfo
}
```

验证器因某些原因未能包含在“LastCommitInfo”中而受到惩罚
通过被自动监禁、可能被削减和解除绑定来减少块的数量。

有关验证器活跃度活动的信息通过“ValidatorSigningInfo”进行跟踪。
它在存储中的索引如下:

- ValidatorSigningInfo: `0x01 | ConsAddrLen (1 字节) | ConsAddress -> ProtocolBuffer(ValSigningInfo)`
- MissedBlocksBitArray:`0x02 | ConsAddrLen (1 字节) |约束地址 | LittleEndianUint64(signArrayIndex) -> VarInt(didMiss)`(varint 是一种数字编码格式)

第一个映射允许我们轻松查找最近的签名信息
验证者基于验证者的共识地址。

第二个映射(`MissedBlocksBitArray`)作用
作为一个大小为“SignedBlocksWindow”的位数组，它告诉我们验证器是否错过了
位数组中给定索引的块。给出位数组中的索引
作为小端 uint64。
结果是一个 `varint` 取为 `0` 或 `1`，其中 `0` 表示
验证器没有错过(确实签名)相应的块，并且`1`表示
他们错过了街区(没有签名)。

请注意，“MissedBlocksBitArray”并未预先明确初始化。钥匙
当我们通过第一个“SignedBlocksWindow”块进行新的
绑定验证器。 `SignedBlocksWindow` 参数定义了大小
(块数)用于跟踪验证器活跃度的滑动窗口。

存储的用于跟踪验证者活跃度的信息如下:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/slashing/v1beta1/slashing.proto#L11-L33 