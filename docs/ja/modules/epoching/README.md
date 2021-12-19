# `x/epoching`

## 摘要

epoching 模块允许模块将消息排队以在某个块高度执行。 每个模块都有自己的纪元模块实例，这允许每个模块有自己的消息队列和纪元的持续时间。

## 例子

在这个例子中，我们正在为一个模块创建一个 epochkeeper，该模块将使用该模块将消息排队以在稍后的时间点执行。 

```go
type Keeper struct {
  storeKey           sdk.StoreKey
  cdc                codec.BinaryMarshaler
  epochKeeper        epochkeeper.Keeper
}

// NewKeeper creates a new staking Keeper instance
func NewKeeper(cdc codec.BinaryMarshaler, key sdk.StoreKey) Keeper {
 return Keeper{
  storeKey:           key,
  cdc:                cdc,
  epochKeeper:        epochkeeper.NewKeeper(cdc, key),
 }
}
```

### Contents

1. **[State](01_state.md)**
