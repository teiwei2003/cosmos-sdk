# 消息

## 消息发送

将硬币从一个地址发送到另一个地址。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/bank/v1beta1/tx.proto#L19-L28

在以下情况下，消息将失败:

- 硬币没有启用发送
- `to` 地址受到限制

## MsgMultiSend

从和向一系列不同的地址发送硬币。 如果任何接收地址与现有帐户不对应，则会创建一个新帐户。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/bank/v1beta1/tx.proto#L33-L39

在以下情况下，消息将失败:

- 任何硬币都没有启用发送
- 任何“to”地址都受到限制
- 任何硬币都被锁定
- 输入和输出没有正确对应 