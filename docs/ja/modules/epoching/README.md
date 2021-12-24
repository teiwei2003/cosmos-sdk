# `x/epoching`

## 概要

epochingモジュールを使用すると、モジュールは特定のブロックの高さで実行するためにメッセージをキューに入れることができます。 各モジュールには、エポックモジュールの独自のインスタンスがあります。これにより、各モジュールは、独自のメッセージキューとエポックの期間を持つことができます。

## 例

この例では、モジュールを使用してメッセージをキューに入れ、後で実行するモジュールのエポックキーパーを作成しています。

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
