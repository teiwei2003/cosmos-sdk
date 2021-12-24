## 状態

## 料金プール

配布に使用されるすべてのグローバル追跡パラメータは、に保存されます
「コストプール」。 報酬を収集し、報酬プールに追加します
ここから検証者/委任者に配布します。

報酬プールには、許可するための小数コイン( `DecCoins`)が含まれていることに注意してください
インフレなどの操作からコインのごく一部を取得します。
コインがプールから配布されると、切り捨てられます
`sdk.Coins`は10進数ではありません。

- 料金プール： `0x00-> ProtocolBuffer(FeePool)`

```go
// coins with decimal
type DecCoins []DecCoin

type DecCoin struct {
    Amount sdk.Dec
    Denom  string
}
```

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/proto/cosmos/distribution/v1beta1/distribution.proto#L94-L101

## バリデーターの配布

関連するバリデーターのバリデーター配布情報は、毎回更新されます。

1. バリデーターのデリゲートの数を更新しました。
2. バリデーターはブロックを正常に提案し、報酬を受け取ります。
3. 委任者がバリデーターから撤退する、または
4. 検証者はその手数料を撤回します。

- ValidatorDistInfo： `0x02 | ValOperatorAddrLen(1バイト)| ValOperatorAddr-> ProtocolBuffer(validatorDistribution)`

```go
type ValidatorDistInfo struct {
    OperatorAddress     sdk.AccAddress
    SelfBondRewards     sdk.DecCoins
    ValidatorCommission types.ValidatorAccumulatedCommission
}
```

## デリゲート配布

各コミッション配分は、最終的な高さを記録するだけで済みます
手数料を引き出します。 代表団は毎回料金を撤回しなければならないので
プロパティの変更(別名バインディングトークンなど)のプロパティは変更されません
そして、クライアントの_蓄積_係数は受動的に計算することができます
最後の引き出しの高さとその現在の属性のみ。

- 委任された配布情報： `0x02 | DelegatorAddrLen(1バイト)|委任アドレス| ValOperatorAddrLen(1バイト)| ValOperatorAddr-> ProtocolBuffer(delegatorDist)`

```go
type DelegationDistInfo struct {
    WithdrawalHeight int64    // last time this delegation withdrew rewards
}
```
