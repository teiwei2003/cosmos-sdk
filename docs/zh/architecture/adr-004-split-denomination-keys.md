# ADR 004:拆分面额密钥

## 变更日志

- 2020-01-08:初始版本
- 2020-01-09:更改归属账户的处理
- 2020-01-14:评论反馈更新
- 2020-01-30:实施更新

### 词汇表

* 面额/面额密钥——唯一的令牌标识符。

##  上下文

使用无需许可的 IBC，任何人都可以将任意面额发送到任何其他帐户。目前，所有非零余额都与帐户一起存储在“sdk.Coins”结构中，这会产生潜在的拒绝服务问题，因为每次修改帐户时加载和存储过多面额都会变得昂贵.有关其他上下文，请参阅问题 [5467](https://github.com/cosmos/cosmos-sdk/issues/5467) 和 [4982](https://github.com/cosmos/cosmos-sdk/issues/4982) .

在面额计数限制后简单地拒绝入金是行不通的，因为它打开了一个悲痛的向量:有人可以通过 IBC 向用户发送大量无意义的硬币，然后阻止用户收到真实的面额(例如赌注奖励)。

## 决定

余额应按账户和面额存储在面额和账户唯一的密钥下，从而实现对特定面额的特定账户余额的 O(1) 读写访问。

### 账户接口(x/auth)

`GetCoins()` 和 `SetCoins()` 将从账户界面中删除，因为硬币余额将
现在存储在银行模块中并由银行模块管理。

归属账户界面将取代“SpendableCoins”以支持“LockedCoins”
不再需要帐户余额。此外，`TrackDelegation()` 现在将接受
以归属余额计价的所有代币的账户余额，而不是加载整个
账户余额。

归属账户将继续存储原始归属、免费委托和委托
归属硬币(这是安全的，因为它们不能包含任意面额)。

### 银行管理员 (x/bank)

以下 API 将被添加到 `x/bank` keeper:

-`GetAllBalances(ctx Context, addr AccAddress) Coins`
-`GetBalance(ctx Context, addr AccAddress, denom string) Coin`
- `SetBalance(ctx Context, addr AccAddress, coin Coin)`
- `LockedCoins(ctx Context, addr AccAddress) Coins`
- `SpendableCoins(ctx Context, addr AccAddress) Coins`

可能会添加额外的 API 以促进迭代和辅助功能
核心功能或持久性。

余额将首先按地址存储，然后按面额存储(反之亦然，
但假定检索单个帐户的所有余额的频率更高):

```golang
var BalancesPrefix = []byte("balances")

func (k Keeper) SetBalance(ctx Context, addr AccAddress, balance Coin) error {
  if !balance.IsValid() {
    return err
  }

  store := ctx.KVStore(k.storeKey)
  balancesStore := prefix.NewStore(store, BalancesPrefix)
  accountStore := prefix.NewStore(balancesStore, addr.Bytes())

  bz := Marshal(balance)
  accountStore.Set([]byte(balance.Denom), bz)

  return nil
}
```

这将导致余额由的字节表示索引
`余额/{地址}/{denom}`。

`DelegateCoins()` 和 `UndelegateCoins()` 将被更改为仅加载每个人
在(非)委托金额中找到的按面额划分的帐户余额。因此，
账户余额的任何变化将由面额进行。

`SubtractCoins()` 和 `AddCoins()` 将被更改以读取和写入余额
直接而不是调用`GetCoins()`/`SetCoins()`(不再存在)。

`trackDelegation()` 和 `trackUndelegation()` 将被更改为不再更新
账户余额。

外部 API 将需要扫描帐户下的所有余额以保持向后兼容性。它
建议这些 API 在以下情况下使用 `GetBalance` 和 `SetBalance` 而不是 `GetAllBalances`
可能不加载整个帐户余额。

### 供应模块

供应模块，为了实现总供应不变量，现在需要
扫描所有账户并使用 `x/bank` Keeper 调用 `GetAllBalances`，然后求和
余额并检查它们是否与预期的总供应量相匹配。

## 状态

公认。

## 结果

### 目的

- O(1) 读取和写入余额(关于面额数量)
其中一个帐户具有非零余额)。请注意，这与实际情况无关
I/O 成本，而不是所需的直接读取总数。

### 消极的

- 读取和写入所有余额时的读取/写入效率略低
交易中的单个帐户。

### 中性的

没有特别的。

## 参考

- Ref: https://github.com/cosmos/cosmos-sdk/issues/4982
- Ref: https://github.com/cosmos/cosmos-sdk/issues/5467
- Ref: https://github.com/cosmos/cosmos-sdk/issues/5492
