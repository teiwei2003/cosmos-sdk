# 参数

bank 模块包含以下参数: 

| Key                | Type          | Example                            |
| ------------------ | ------------- | ---------------------------------- |
| SendEnabled        | []SendEnabled | [{denom: "stake", enabled: true }] |
| DefaultSendEnabled | bool          | true                               |

## 发送启用

发送启用参数是一组 SendEnabled 条目映射硬币
面额到其 send_enabled 状态。 此列表中的条目需要
优先于 `DefaultSendEnabled` 设置。

## 默认发送启用

默认发送启用值控制所有发送传输能力
硬币面额，除非特别包含在 `SendEnabled` 数组中
参数。 