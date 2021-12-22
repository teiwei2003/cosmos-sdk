# 状态

##消息队列

消息在每个纪元结束时排队运行。 排队的消息有一个纪元号，对于每个纪元号，队列被迭代并执行每条消息。

### 消息队列

每个模块都有一个特定于该模块的唯一消息队列。

## 行动

一个模块将添加一个实现 `sdk.Msg` 接口的消息。 这些消息将在稍后执行(下一个纪元结束)。 

```go
type Msg interface {
  proto.Message

  // Return the message type.
  // Must be alphanumeric or empty.
  Route() string

  // Returns a human-readable string for the message, intended for utilization
  // within tags
  Type() string

  // ValidateBasic does a simple validation check that
  // doesn't require access to any other information.
  ValidateBasic() error

  // Get the canonical byte representation of the Msg.
  GetSignBytes() []byte

  // Signers returns the addrs of signers that must sign.
  // CONTRACT: All signatures must be present to be valid.
  // CONTRACT: Returns addrs in some deterministic order.
  GetSigners() []AccAddress
 }
```

## 缓冲消息导出/导入

目前，实现了 `x/epoching` 模块以导出所有没有纪元编号的缓冲消息。 导入状态时，缓冲的消息存储在当前纪元上，以在当前纪元结束时运行。

##创世交易

我们在执行创世交易后执行 epoch 以在节点启动之前立即查看更改。

## 在 epoch 上执行

- 尝试执行 epoch 的消息
- 如果成功，按原样进行更改
- 如果失败，请尝试在处理程序上执行恢复额外操作(例如 EpochDelegationPool 存款)
- 如果恢复失败，恐慌 