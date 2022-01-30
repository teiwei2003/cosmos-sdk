# 帰属

- [帰属](#vesting)
    - [はじめにと要件](#intro-and-requirements)
    - [注](#note)
    - [アトリビューションアカウントタイプ](#vesting-account-types)
    - [権利確定アカウントの仕様](#vesting-account-specification)
        - [権利確定と権利確定額の決定](#determining-vesting--vested-amounts)
            - [継続的に権利が確定するアカウント](#continuously-vesting-accounts)
        - [定期的な権利確定アカウント](#periodic-vesting-accounts)
            - [遅延/個別の権利確定アカウント](#delayeddiscrete-vesting-accounts)
        - [転送/送信](#transferringsending)
            - [キーパー/ハンドラー](#keepershandlers)
        - [委任](#delegating)
            - [キーパー/ハンドラー](#keepershandlers-1)
        - [委任のキャンセル](#委任のキャンセル)
            - [キーパー/ハンドラー](#keepershandlers-2)
    - [キーパーとハンドラー](#keepers--ハンドラー)
    - [ジェネシス初期化](#genesis-初期化)
    - [例](#examples)
        - [シンプル](#simple)
        - [スラッシュ](#slashing)
        - [定期的な権利確定](#定期的な権利確定)
    - [用語集](#glossary)

## はじめにと要件

仕様は、によって使用される帰属アカウントの実現を定義します
宇宙の中心。このアトリビューションアカウントの要件は、
作成期間中に、開始残高[X]と帰属の終了を使用して初期化します
時間 `ET`。権利確定勘定は、権利確定開始時刻 `ST`で初期化できます。
そして、いくつかの権利確定期間 `P`。アトリビューションの開始時刻が含まれている場合は、
権利確定期間は開始時間まで開始されません。権利確定期間の場合
含まれる、権利確定は指定された期間数で発生します。

すべての既得アカウントについて、既得アカウントの所有者は委任できます
バリデーターからの委任をキャンセルしますが、コインを別のコインに転送することはできません
これらのコインが確定するまで説明します。仕様は4つを許可します
さまざまな種類の帰属:

- 権利確定の遅延。[ET]に達すると、すべてのコインが権利確定します。
- コインが[ST]に帰属し始め、線形に帰属する継続的な帰属
[ET]に到達するまでの時間について
- 定期的な権利確定、コインは[ST]に属し始め、定期的に権利が確定します
期間数と各期間の運動量に基づきます。
期間の数、各期間の長さ、および各期間の量は次のとおりです。
構成可能。定期的な権利確定勘定は継続的なものとは異なります
トークンの帰属アカウントは、バッチでリリースできます。にとって
たとえば、通常の権利確定勘定を権利確定の取り決めに使用できます
コインは、四半期ごと、毎年、またはその他の機能を通じてリリースされます
時間の経過に伴うトークン。
- 所有権は永続的にロックされ、コインは永続的にロックされます。このアカウントのコインは
ロックされている場合でも、委任とガバナンスの投票に使用できます。

## ノート

既得のアカウントは、いくつかの既得のトークンと属性のないトークンで初期化できます。
権利が確定していないトークンはすぐに譲渡できます。帰属の遅延と
作成後、通常のメッセージでContinuousVestingアカウントを作成できます。
他のタイプのアトリビューションアカウントは、作成時に作成するか、次のように使用する必要があります。
手動ネットワークアップグレードの一部。現在の仕様では、
_無条件_属性の場合(つまり、[ET]に到達することは不可能であり、
帰属できないコインがあります)。

## アトリビューションアカウントタイプ 

```go
//VestingAccount defines an interface that any vesting account type must
//implement.
type VestingAccount interface {
  Account

  GetVestedCoins(Time)  Coins
  GetVestingCoins(Time) Coins

 //TrackDelegation performs internal vesting accounting necessary when
 //delegating from a vesting account. It accepts the current block time, the
 //delegation amount and balance of all coins whose denomination exists in
 //the account's original vesting balance.
  TrackDelegation(Time, Coins, Coins)

 //TrackUndelegation performs internal vesting accounting necessary when a
 //vesting account performs an undelegation.
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
//Stores all vesting periods passed as part of a PeriodicVestingAccount
type Periods []Period

```

### PeriodicVestingAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L64-L73

一時的な型のチェックとアサーションおよびサポートを容易にするため
アカウントの残高を使用する柔軟性、既存の `x/bank``ViewKeeper`インターフェース
以下を含むように更新します。

```go
type ViewKeeper interface {
 //...

 //Calculates the total locked account balance.
  LockedCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins

 //Calculates the total spendable balance that can be sent to other accounts.
  SpendableCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
}
```

### PermanentLockedAccount

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/vesting/v1beta1/vesting.proto#L78-L83

## Vesting Account Specification

アトリビューションアカウントを指定すると、処理操作で次のように定義されます。

- `OV`:元の帰属コインの数。定数値です。
- `V`: `OV`トークンの数はまだ_権利確定_です。それはからです
`OV`、` StartTime`、 `EndTime`。この値は、に基づいてではなく、オンデマンドで計算されます
基礎のすべての部分。
- `V'`:_既得_(ロック解除)の `OV`コインの数。この値は
ブロックごとではなく、オンデマンドで計算します。
- `DV`:委託された_vesting_コインの数。可変値です。それは
アトリビューションアカウントに直接保存して変更します。
- `DF`:委託された_既得_(ロック解除)コインの数。変数です
価値。アトリビューションアカウントに直接保存および変更されます。
- `BC`: `OV`コインの数から転送されたコインを差し引いた数
(否定的または委託することができます)。バランスが取れていると見なされます
埋め込まれた基本アカウント。アトリビューションアカウントに直接保存および変更されます。

### 権利確定と権利確定額の決定

これらの値は、に基づいてではなく、オンデマンドで計算されることに注意してください。
ブロックごとに必須(たとえば、 `BeginBlocker`または` EndBlocker`)。

#### アカウントを継続的に権利確定

所与のブロック時間[Ｔ]に起因するトークンの数を決定するために、
以下をせよ:

1. Compute `X := T - StartTime`
2. Compute `Y := EndTime - StartTime`
3. Compute `V' := OV * (X/Y)`
4. Compute `V := OV - V'`

Thus, the total amount of _vested_ coins is `V'` and the remaining amount, `V`,
is _vesting_.

```go
func (cva ContinuousVestingAccount) GetVestedCoins(t Time) Coins {
    if t <= cva.StartTime {
       //We must handle the case where the start time for a vesting account has
       //been set into the future or when the start of the chain is not exactly
       //known.
        return ZeroCoins
    } else if t >= cva.EndTime {
        return cva.OriginalVesting
    }

    x := t - cva.StartTime
    y := cva.EndTime - cva.StartTime

    return cva.OriginalVesting * (x/y)
}

func (cva ContinuousVestingAccount) GetVestingCoins(t Time) Coins {
    return cva.OriginalVesting - cva.GetVestedCoins(t)
}
```

### 定期的な権利確定アカウント

通常の権利確定勘定は、各期間にリリースされたコインを計算する必要があります
指定されたブロック時間[T]の期間。 複数の期間が経過している可能性があることに注意してください
`GetVestedCoins`を呼び出すときは、各サイクルを次のように繰り返す必要があります。
その期間の終わりは[T]の後です。 

1. Set `CT := StartTime`
2. Set `V' := 0`

各期間Pについて:

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
  ct := pva.StartTime//The start of the vesting schedule
  vested := 0
  periods = pva.GetPeriods()
  for _, period  := range periods {
    if t - ct < period.Length {
      break
    }
    vested += period.Amount
    ct += period.Length//increment ct to the start of the next vesting period
  }
  return vested
}

func (pva PeriodicVestingAccount) GetVestingCoins(t Time) Coins {
    return pva.OriginalVesting - cva.GetVestedCoins(t)
}
```

#### Delayed/Discrete Vesting Accounts

遅延した権利確定勘定は、完全なものしかないため、推論が容易です。
その金額は特定の時間に権利が確定し、その後すべてのコインが権利が確定します(ロックが解除されます)。
これには、アカウントが最初に持っている可能性のあるロック解除されたコインは含まれません。

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

いつでも、帰属アカウントを転送できます: `min((BC + DV)-V、BC)`。

つまり、アトリビューションアカウントは、ベースアカウントの最小値を転送できます
残高と基本口座残高に現在の注文数を加えたもの
既得トークンは、これまでの既得トークンの数よりも少なくなっています。

ただし、アカウントの残高は `x/bank`モジュールを介して追跡されるため、
アカウントの残高全体が読み込まれないようにしたいので、OKに変更できます
ロックされたバランスは `max(V-DV、0)`として定義され、推測されます
使うことができるバランス。

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

   //non-vesting accounts do not have any locked coins
    return NewCoins()
}
```

#### Keepers/Handlers

対応する `x/bank`管理者はコインの送信を適切に処理する必要があります
アカウントが帰属アカウントであるかどうかによる。

```go
func (k Keeper) SendCoins(ctx Context, from Account, to Account, amount Coins) {
    bc := k.GetBalances(ctx, from)
    v := k.LockedCoins(ctx, from)

    spendable := bc - v
    newCoins := spendable - amount
    assert(newCoins >= 0)

    from.SetBalance(newCoins)
    to.AddBalance(amount)

   //save balances...
}
```

### Delegating

[D]通貨を委任しようとするアトリビューションアカウントの場合、次の操作を実行します:

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

**注** `TrackDelegation`は` DelegatedVesting`と `DelegatedFree`のみを変更しました
フィールド、アップストリームの呼び出し元は、 `amount`を減算して` Coins`フィールドを変更する必要があります。

#### Keepers/Handlers

```go
func DelegateCoins(t Time, from Account, amount Coins) {
    if isVesting(from) {
        from.TrackDelegation(t, amount)
    } else {
        from.SetBalance(sc - amount)
    }

   //save account...
}
```

### Undelegating

[D]トークンの委任をキャンセルしようとするアトリビューションアカウントの場合、次の操作を実行します。
注: `DV <D`および`(DV + DF)<D`は、丸めの癖が原因である可能性があります
委任ロジックを委任/キャンセルします。

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

**注** `TrackUnDelegation`は` DelegatedVesting`と `DelegatedFree`のみを変更しました
フィールド、アップストリームの呼び出し元は、 `amount`を追加して` Coins`フィールドを変更する必要があります。

**注**:承認が減らされると、継続的な権利確定アカウントは終了します
すべてのトークンが確定した後でも、[DV]が過剰になります。 それの訳は
承認された無料コインのキャンセルが優先されます。

**注**:委託のキャンセル(保証金の払い戻し)の金額は、委託を超える場合があります
承認の取り消しにより、社債の返済方法が打ち切られるため、既得(社債)額、
もしも
コミットされていないトークンは完全ではありません。

#### Keepers/Handlers

```go
func UndelegateCoins(to Account, amount Coins) {
    if isVesting(to) {
        if to.DelegatedFree + to.DelegatedVesting >= amount {
            to.TrackUndelegation(amount)
           //save account ...
        }
    } else {
        AddBalance(to, amount)
       //save account...
    }
}
```

## Keepers & Handlers

`VestingAccount`の実装は` x/auth`にあります。 ただし、ゴールキーパーは
帰属するモジュールを使用する可能性があることを望んでいます(例: `x/staking`でのステーキング)
コインの場合、明示的なメソッドを `x/bank`キーパーで呼び出す必要があります(たとえば、` DelegateCoins`)
[SendCoins]および[SubtractCoins]とは異なります。

さらに、既得のアカウントは、任意の通貨を使用できる必要があります。
他のユーザーから受け取ります。 したがって、バンキングモジュールの `MsgSend`ハンドラーは
アトリビューションアカウントがその金額を超えて送信しようとすると、エラーが発生します
ロック解除されたコインの数。

完全な実装の詳細については、上記の仕様を参照してください。
## ジェネシスの初期化

属性付きアカウントと属性なしアカウントを初期化するには、[GenesisAccount]構造
新しいフィールド[Vesting]、[StartTime]、[EndTime]を含めます。 アカウントは
[BaseAccount]タイプまたは属性のないタイプには[Vesting = false]があります。 この
ジェネシス初期化ロジック(例: `initFromGenesisState`)を解決する必要があります
そして、これらのフィールドに従って、正しいアカウントがそれに応じて返されます。

```go
type GenesisAccount struct {
   //...

   //vesting account fields
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
           //return a continuous vesting account
        } else if ga.EndTime != 0 {
           //return a delayed vesting account
        } else {
           //invalid genesis vesting account provided
            panic()
        }
    }

    return bacc
}
```

## Examples

### Simple

10の帰属コインを持つ継続的な帰属アカウントが与えられます。

```
OV = 10
DF = 0
DV = 0
BC = 10
V = 10
V' = 0
```

1. すぐに1枚のコインを受け取ります

    ```
    BC = 11
    ```

2. 時間が経つと、2枚のコインがベストになります

    ```
    V = 8
    V' = 2
    ```

3. 4枚のコインをバリデーターAに委任します

    ```
    DV = 4
    BC = 7
    ```

4. 3枚のコインを送ります

    ```
    BC = 4
    ```

5. より多くの時間が経過し、さらに2枚のコインがベストになります

    ```
    V = 6
    V' = 4
    ```

6. 2枚のコインを送ります。 この時点で、アカウントはさらに送信することができなくなります
コインがベストになるか、追加のコインを受け取ります。 ただし、委任することはできます。

    ```
    BC = 2
    ```

### 斬る

簡単な例と同じ初期開始条件。

1. 時間が経つと、5枚のコインがベストになります

    ```
    V = 5
    V' = 5
    ```

2. 5枚のコインをバリデーターAに委任します

    ```
    DV = 5
    BC = 5
    ```

3. 5枚のコインをバリデーターBに委任します

    ```
    DF = 5
    BC = 0
    ```

4. バリデーターAが50％削減され、Aへの委任は2.5コインの価値になります
5. バリデーターAからの委任を解除します(2.5コイン)

    ```
    DF = 5 - 2.5 = 2.5
    BC = 0 + 2.5 = 2.5
    ```

6. バリデーターB(5コイン)から委任を解除します。 この時点でのアカウントは
より多くのコインを受け取らない限り、またはより多くのコインが確定するまで、2.5コインを送ってください。
ただし、委任することはできます。

    ```
    DV = 5 - 2.5 = 2.5
    DF = 2.5 - 2.5 = 0
    BC = 2.5 + 5 = 7.5
    ```

    過剰な量の `DV`があることに注意してください。

### 定期的な権利確定

権利確定アカウントが作成され、1年間で100個のトークンがリリースされます。
各四半期に権利が確定するトークンの1/4。 権利確定スケジュールは次のようになります。

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

1. すぐに1枚のコインを受け取ります

    ```
    BC = 101
    ```

2. 権利確定期間1が経過し、25コインが権利確定します

    ```
    V = 75
    V' = 25
    ```

3. 権利確定期間2の間に、5枚のコインが転送され、5枚のコインが委任されます

    ```
    DV = 5
    BC = 91
    ```

4. 権利確定期間2パス、25コインが権利確定

    ```
    V = 50
    V' = 50
    ```

## 用語集

- OriginalVesting:最初にあるコインの量(金種ごと)
権利確定口座の一部。これらのコインは創世​​記に設定されています。
- StartTime:権利確定アカウントが権利確定を開始するBFT時間。
-EndTime:権利確定アカウントが完全に権利が確定するBFT時間。
- DelegatedFree:追跡されたコインの量(金種ごと)
委任時に完全に権利が確定した権利確定アカウントから委任された。
- DelegatedVesting:追跡されたコインの量(金種ごと)
委任時に権利が確定していた権利確定アカウントから委任されました。
- ContinuousVestingAccount:コインを権利確定する権利確定アカウントの実装
時間の経過とともに直線的に。
- DelayedVestingAccount:完全に権利が確定するだけの権利確定アカウントの実装
与えられた時間にすべてのコイン。
- PeriodicVestingAccount:コインを権利確定する権利確定アカウントの実装
カスタムの権利確定スケジュールに従って。
- PermanentLockedAccount:コインを解放することはなく、無期限にロックします。
このアカウントのコインは、ロックされている場合でも、委任やガバナンスの投票に使用できます。