# 归属

- [归属](#vesting)
    - [介绍和要求](#intro-and-requirements)
    - [注意](#note)
    - [归属账户类型](#vesting-account-types)
    - [归属账户规范](#vesting-account-specification)
        - [确定归属和归属金额](#determining-vesting--vested-amounts)
            - [持续归属账户](#continuously-vesting-accounts)
        - [定期归属账户](#periodic-vesting-accounts)
            - [延迟/离散归属账户](#delayeddiscrete-vesting-accounts)
        - [传输/发送](#transferringsending)
            - [守护者/处理者](#keepershandlers)
        - [委托](#delegating)
            - [守护者/处理者](#keepershandlers-1)
        - [取消委派](#取消委派)
            - [守护者/处理者](#keepershandlers-2)
    - [Keepers & Handlers](#keepers--handlers)
    - [创世初始化](#genesis-initialization)
    - [例子](#examples)
        - [简单](#simple)
        - [Slashing](#slashing)
        - [定期归属](#periodic-vesting)
    - [词汇表](#glossary)

## 介绍和要求

该规范定义了由以下人员使用的归属账户实现
宇宙中心。这个归属账户的要求是它应该是
在创世期间用起始余额“X”和归属结束初始化
时间`ET`。归属账户可以用归属开始时间`ST`初始化
和一些归属期`P`。如果包括归属开始时间，则
归属期直到开始时间才开始。如果归属期
包括在内，归属发生在指定的时期数。

对于所有归属账户，归属账户的所有者可以委托
并从验证者那里取消委托，但是他们不能将硬币转移到另一个
帐户，直到这些硬币归属。该规范允许四个
不同类型的归属:

- 延迟归属，一旦达到“ET”，所有硬币都归属。
- 连续归属，其中硬币开始归属于“ST”并线性归属于
关于到达“ET”的时间
- 定期归属，硬币开始归属于“ST”并定期归属
根据期数和每期的行权金额。
期数、每期长度和每期金额为
可配置。定期归属账户有别于连续的
该代币的归属账户可以分批释放。为了
例如，定期归属账户可用于归属安排
硬币每季度、每年或通过任何其他功能发布
随着时间的推移令牌。
- 永久锁定归属，硬币永久锁定。这个账户里的币可以
即使在锁定时，仍可用于委托和治理投票。

## 笔记

归属账户可以用一些归属和非归属代币初始化。
非归属代币将可立即转让。延迟归属和
可以在创世后使用普通消息创建ContinuousVesting 帐户。
其他类型的归属账户必须在创世时创建，或作为
手动网络升级的一部分。目前的规范只允许
对于_无条件_归属(即，不可能达到“ET”和
有硬币无法归属)。

## 归属账户类型 

```go
// VestingAccount defines an interface that any vesting account type must
// implement.
type VestingAccount interface {
  Account

  GetVestedCoins(Time)  Coins
  GetVestingCoins(Time) Coins

  // TrackDelegation performs internal vesting accounting necessary when
  // delegating from a vesting account. It accepts the current block time, the
  // delegation amount and balance of all coins whose denomination exists in
  // the account's original vesting balance.
  TrackDelegation(Time, Coins, Coins)

  // TrackUndelegation performs internal vesting accounting necessary when a
  // vesting account performs an undelegation.
  TrackUndelegation(Coins)

  GetStartTime() int64
  GetEndTime()   int64
}
```

### BaseVestingAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L10-L33

### ContinuousVestingAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L35-L43

### DelayedVestingAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L45-L53

### Period

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L56-L62

```go
// Stores all vesting periods passed as part of a PeriodicVestingAccount
type Periods []Period

```

### PeriodicVestingAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L64-L73

为了促进较少的临时类型检查和断言并支持
账户余额使用的灵活性，现有的`x/bank``ViewKeeper`接口
更新为包含以下内容: 

```go
type ViewKeeper interface {
  // ...

  // Calculates the total locked account balance.
  LockedCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins

  // Calculates the total spendable balance that can be sent to other accounts.
  SpendableCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
}
```

### PermanentLockedAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L78-L83

## Vesting Account Specification

给定一个归属账户，我们在处理操作中定义以下内容:

- `OV`:原始归属币数量。 它是一个常数值。
- `V`:仍然 _vesting_ 的 `OV` 代币数量。 它是由
`OV`、`StartTime` 和 `EndTime`。 该值是按需计算的，而不是根据
每块基础。
- `V'`:_vested_(解锁)的 `OV` 硬币数量。 这个值是
按需计算，而不是按块计算。
- `DV`:委托_vesting_硬币的数量。 它是一个可变值。 它是
直接在归属账户中存储和修改。
- `DF`:委托的 _vested_(解锁)硬币的数量。 它是一个变量
价值。 它直接在归属账户中存储和修改。
- `BC`:`OV` 硬币的数量减去任何转移的硬币
(可以是否定的或委托的)。 它被认为是平衡的
嵌入式基本帐户。 它直接在归属账户中存储和修改。 
### Determining Vesting & Vested Amounts

需要注意的是，这些值是按需计算的，而不是根据
强制每个块基础(例如`BeginBlocker` 或`EndBlocker`)。

#### Continuously Vesting Accounts

为了确定在给定的区块时间“T”内归属的代币数量，
执行以下操作: 

1. Compute `X := T - StartTime`
2. Compute `Y := EndTime - StartTime`
3. Compute `V' := OV * (X / Y)`
4. Compute `V := OV - V'`

Thus, the total amount of _vested_ coins is `V'` and the remaining amount, `V`,
is _vesting_.

```go
func (cva ContinuousVestingAccount) GetVestedCoins(t Time) Coins {
    if t <= cva.StartTime {
        // We must handle the case where the start time for a vesting account has
        // been set into the future or when the start of the chain is not exactly
        // known.
        return ZeroCoins
    } else if t >= cva.EndTime {
        return cva.OriginalVesting
    }

    x := t - cva.StartTime
    y := cva.EndTime - cva.StartTime

    return cva.OriginalVesting * (x / y)
}

func (cva ContinuousVestingAccount) GetVestingCoins(t Time) Coins {
    return cva.OriginalVesting - cva.GetVestedCoins(t)
}
```

### Periodic Vesting Accounts

定期归属账户需要计算每个时期释放的硬币
给定区块时间‘T’的时间段。 请注意，多个时期可能已经过去
当调用`GetVestedCoins`时，我们必须迭代每个周期，直到
那个时期的结束是在“T”之后。 

1. Set `CT := StartTime`
2. Set `V' := 0`

For each Period P:

  1. Compute `X := T - CT`
  2. IF `X >= P.Length`
      1. Compute `V' += P.Amount`
      2. Compute `CT += P.Length`
      3. ELSE break
  3. Compute `V := OV - V'`

```go
func (pva PeriodicVestingAccount) GetVestedCoins(t Time) Coins {
  if t < pva.StartTime {
    return ZeroCoins
  }
  ct := pva.StartTime // The start of the vesting schedule
  vested := 0
  periods = pva.GetPeriods()
  for _, period  := range periods {
    if t - ct < period.Length {
      break
    }
    vested += period.Amount
    ct += period.Length // increment ct to the start of the next vesting period
  }
  return vested
}

func (pva PeriodicVestingAccount) GetVestingCoins(t Time) Coins {
    return pva.OriginalVesting - cva.GetVestedCoins(t)
}
```

#### Delayed/Discrete Vesting Accounts

延迟归属账户更容易推理，因为它们只有完整的
金额归属到某个时间，然后所有硬币都归属(解锁)。
这不包括帐户最初可能拥有的任何解锁硬币。 

```go
func (dva DelayedVestingAccount) GetVestedCoins(t Time) Coins {
    if t >= dva.EndTime {
        return dva.OriginalVesting
    }

    return ZeroCoins
}

func (dva DelayedVestingAccount) GetVestingCoins(t Time) Coins {
    return dva.OriginalVesting - dva.GetVestedCoins(t)
}
```

### Transferring/Sending

在任何给定时间，归属账户可以转移:`min((BC + DV) - V, BC)`。

换句话说，一个归属账户可以转移基础账户的最小值
余额和基本账户余额加上当前委托的数量
归属代币少于迄今为止归属代币的数量。

然而，鉴于账户余额是通过 `x/bank` 模块跟踪的，并且
我们想避免加载整个帐户余额，我们可以改为确定
锁定余额，可以定义为`max(V - DV, 0)`，并推断出
可花费的余额。 

```go
func (va VestingAccount) LockedCoins(t Time) Coins {
   return max(va.GetVestingCoins(t) - va.DelegatedVesting, 0)
}
```

The `x/bank` `ViewKeeper` can then provide APIs to determine locked and spendable
coins for any account:

```go
func (k Keeper) LockedCoins(ctx Context, addr AccAddress) Coins {
    acc := k.GetAccount(ctx, addr)
    if acc != nil {
        if acc.IsVesting() {
            return acc.LockedCoins(ctx.BlockTime())
        }
    }

    // non-vesting accounts do not have any locked coins
    return NewCoins()
}
```

#### Keepers/Handlers

相应的`x/bank` 管理员应该适当地处理发送硬币
根据账户是否为归属账户。

```go
func (k Keeper) SendCoins(ctx Context, from Account, to Account, amount Coins) {
    bc := k.GetBalances(ctx, from)
    v := k.LockedCoins(ctx, from)

    spendable := bc - v
    newCoins := spendable - amount
    assert(newCoins >= 0)

    from.SetBalance(newCoins)
    to.AddBalance(amount)

    // save balances...
}
```

### Delegating

对于尝试委托“D”币的归属账户，执行以下操作:

1. Verify `BC >= D > 0`
2. Compute `X := min(max(V - DV, 0), D)` (portion of `D` that is vesting)
3. Compute `Y := D - X` (portion of `D` that is free)
4. Set `DV += X`
5. Set `DF += Y`

```go
func (va VestingAccount) TrackDelegation(t Time, balance Coins, amount Coins) {
    assert(balance <= amount)
    x := min(max(va.GetVestingCoins(t) - va.DelegatedVesting, 0), amount)
    y := amount - x

    va.DelegatedVesting += x
    va.DelegatedFree += y
}
```

**注意** `TrackDelegation` 只修改了 `DelegatedVesting` 和 `DelegatedFree`
字段，因此上游调用者必须通过减去 `amount` 来修改 `Coins` 字段。

#### Keepers/Handlers

```go
func DelegateCoins(t Time, from Account, amount Coins) {
    if isVesting(from) {
        from.TrackDelegation(t, amount)
    } else {
        from.SetBalance(sc - amount)
    }

    // save account...
}
```

### Undelegating

对于试图取消“D”代币委托的归属账户，执行以下操作:
注意:`DV < D` 和 `(DV + DF) < D` 可能是由于四舍五入的怪癖
委托/取消委托逻辑。

1. Verify `D > 0`
2. Compute `X := min(DF, D)` (portion of `D` that should become free, prioritizing free coins)
3. Compute `Y := min(DV, D - X)` (portion of `D` that should remain vesting)
4. Set `DF -= X`
5. Set `DV -= Y`

```go
func (cva ContinuousVestingAccount) TrackUndelegation(amount Coins) {
    x := min(cva.DelegatedFree, amount)
    y := amount - x

    cva.DelegatedFree -= x
    cva.DelegatedVesting -= y
}
```

**注意** `TrackUnDelegation` 只修改了 `DelegatedVesting` 和 `DelegatedFree`
字段，因此上游调用者必须通过添加 `amount` 来修改 `Coins` 字段。

**注**:如果授权被削减，则持续归属账户结束
即使在所有代币都已归属之后，也有多余的“DV”金额。 这是因为
优先考虑取消授权的免费硬币。

**注**:取消委托(退押金)金额可能会超过委托
由于取消授权截断了债券退款的方式，归属(债券)金额，
如果
未委托代币是非完整的。 
#### Keepers/Handlers

```go
func UndelegateCoins(to Account, amount Coins) {
    if isVesting(to) {
        if to.DelegatedFree + to.DelegatedVesting >= amount {
            to.TrackUndelegation(amount)
            // save account ...
        }
    } else {
        AddBalance(to, amount)
        // save account...
    }
}
```

## Keepers & Handlers

`VestingAccount` 实现驻留在 `x/auth` 中。 然而，任何门将
希望潜在地利用任何归属的模块(例如在`x/staking`中进行质押)
硬币，必须在`x/bank` keeper 上调用显式方法(例如`DelegateCoins`)
与“SendCoins”和“SubtractCoins”相反。

另外，归属账户也应该可以花掉任何币吧
从其他用户接收。 因此，银行模块的 `MsgSend` 处理程序应该
如果归属账户试图发送超过其数量的金额，则会出现错误
解锁的硬币数量。

有关完整的实现细节，请参阅上述规范。 
## Genesis Initialization

要初始化归属和非归属账户，“GenesisAccount”结构
包括新字段:“Vesting”、“StartTime”和“EndTime”。 帐户应该是
“BaseAccount”类型或任何非归属类型具有“Vesting = false”。 这
创世初始化逻辑(例如`initFromGenesisState`)必须解析
并根据这些字段相应地返回正确的帐户。 

```go
type GenesisAccount struct {
    // ...

    // vesting account fields
    OriginalVesting  sdk.Coins `json:"original_vesting"`
    DelegatedFree    sdk.Coins `json:"delegated_free"`
    DelegatedVesting sdk.Coins `json:"delegated_vesting"`
    StartTime        int64     `json:"start_time"`
    EndTime          int64     `json:"end_time"`
}

func ToAccount(gacc GenesisAccount) Account {
    bacc := NewBaseAccount(gacc)

    if gacc.OriginalVesting > 0 {
        if ga.StartTime != 0 && ga.EndTime != 0 {
            // return a continuous vesting account
        } else if ga.EndTime != 0 {
            // return a delayed vesting account
        } else {
            // invalid genesis vesting account provided
            panic()
        }
    }

    return bacc
}
```

## Examples

### Simple

给定一个具有 10 个归属币的连续归属账户。 

```
OV = 10
DF = 0
DV = 0
BC = 10
V = 10
V' = 0
```

1. Immediately receives 1 coin

    ```
    BC = 11
    ```

2. Time passes, 2 coins vest

    ```
    V = 8
    V' = 2
    ```

3. Delegates 4 coins to validator A

    ```
    DV = 4
    BC = 7
    ```

4. Sends 3 coins

    ```
    BC = 4
    ```

5. More time passes, 2 more coins vest

    ```
    V = 6
    V' = 4
    ```

6. Sends 2 coins. At this point the account cannot send anymore until further
coins vest or it receives additional coins. It can still however, delegate.

    ```
    BC = 2
    ```

### Slashing

Same initial starting conditions as the simple example.

1. Time passes, 5 coins vest

    ```
    V = 5
    V' = 5
    ```

2. Delegate 5 coins to validator A

    ```
    DV = 5
    BC = 5
    ```

3. Delegate 5 coins to validator B

    ```
    DF = 5
    BC = 0
    ```

4. Validator A gets slashed by 50%, making the delegation to A now worth 2.5 coins
5. Undelegate from validator A (2.5 coins)

    ```
    DF = 5 - 2.5 = 2.5
    BC = 0 + 2.5 = 2.5
    ```

6. Undelegate from validator B (5 coins). The account at this point can only
send 2.5 coins unless it receives more coins or until more coins vest.
It can still however, delegate.

    ```
    DV = 5 - 2.5 = 2.5
    DF = 2.5 - 2.5 = 0
    BC = 2.5 + 5 = 7.5
    ```

    Notice how we have an excess amount of `DV`.

### Periodic Vesting

A vesting account is created where 100 tokens will be released over 1 year, with
1/4 of tokens vesting each quarter. The vesting schedule would be as follows:

```yaml
Periods:
- amount: 25stake, length: 7884000
- amount: 25stake, length: 7884000
- amount: 25stake, length: 7884000
- amount: 25stake, length: 7884000
```

```
OV = 100
DF = 0
DV = 0
BC = 100
V = 100
V' = 0
```

1. Immediately receives 1 coin

    ```
    BC = 101
    ```

2. Vesting period 1 passes, 25 coins vest

    ```
    V = 75
    V' = 25
    ```

3. During vesting period 2, 5 coins are transfered and 5 coins are delegated

    ```
    DV = 5
    BC = 91
    ```

4. Vesting period 2 passes, 25 coins vest

    ```
    V = 50
    V' = 50
    ```

## Glossary

- OriginalVesting: The amount of coins (per denomination) that are initially
part of a vesting account. These coins are set at genesis.
- StartTime: The BFT time at which a vesting account starts to vest.
- EndTime: The BFT time at which a vesting account is fully vested.
- DelegatedFree: The tracked amount of coins (per denomination) that are
delegated from a vesting account that have been fully vested at time of delegation.
- DelegatedVesting: The tracked amount of coins (per denomination) that are
delegated from a vesting account that were vesting at time of delegation.
- ContinuousVestingAccount: A vesting account implementation that vests coins
linearly over time.
- DelayedVestingAccount: A vesting account implementation that only fully vests
all coins at a given time.
- PeriodicVestingAccount: A vesting account implementation that vests coins
according to a custom vesting schedule.
- PermanentLockedAccount: It does not ever release coins, locking them indefinitely.
Coins in this account can still be used for delegating and for governance votes even while locked.
