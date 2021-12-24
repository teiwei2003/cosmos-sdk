# Keepers

bankモジュールは、これらのエクスポートされたkeeperインターフェイスを提供します。
アカウントの残高を読み取ったり更新したりする他のモジュールに渡します。 モジュール
それらの機能を提供する最も許可されていないインターフェースを使用する必要があります
必須。

ベストプラクティスでは、「バンク」モジュールコードを注意深く確認して、
権限は、期待する方法で制限されます。

## ブロックリストアドレス

`x / bank`モジュールは、ブロックされたリストと見なされるアドレスマッピングを受け入れます
「MsgSend」などの方法で直接かつ明示的に資金を受け取ります。
`MsgMultiSend`および` SendCoinsFromModuleToAccount`などの直接API呼び出し。

通常、これらのアドレスはモジュールアカウントです。 これらの住所が資金を受け取った場合
ステートマシンの予想されるルールの外では、不変条件は
損傷し、ネットワークが停止する可能性があります。

ユーザーまたは顧客がブロックされたアカウントに直接または間接的に資金を送金しようとした場合、たとえば[IBC]（http：//docs.cosmos）を使用して、「x/bank」モジュールにブロックされたアドレスのセットを提供する。 network/master/ibc/）。

## 一般的なタイプ

### 入力

マルチパーティ転送入力

```protobuf
// Input models transaction input.
message Input {
  string   address                        = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}
```

### Output

マルチパーティ転送の出力。

```protobuf
// Output models transaction outputs.
message Output {
  string   address                        = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}
```

## BaseKeeper

Base keeper はフルアクセスを提供します：あなたは任意のアカウントの残高を変更し、自由にコインをミントまたは燃やすことができます。

```go
// Keeper defines a module interface that facilitates the transfer of coins
// between accounts.
type Keeper interface {
    SendKeeper

    InitGenesis(sdk.Context, *types.GenesisState)
    ExportGenesis(sdk.Context) *types.GenesisState

    GetSupply(ctx sdk.Context, denom string) sdk.Coin
    GetPaginatedTotalSupply(ctx sdk.Context, pagination *query.PageRequest) (sdk.Coins, *query.PageResponse, error)
    IterateTotalSupply(ctx sdk.Context, cb func(sdk.Coin) bool)
    GetDenomMetaData(ctx sdk.Context, denom string) (types.Metadata, bool)
    SetDenomMetaData(ctx sdk.Context, denomMetaData types.Metadata)
    IterateAllDenomMetaData(ctx sdk.Context, cb func(types.Metadata) bool)

    SendCoinsFromModuleToAccount(ctx sdk.Context, senderModule string, recipientAddr sdk.AccAddress, amt sdk.Coins) error
    SendCoinsFromModuleToModule(ctx sdk.Context, senderModule, recipientModule string, amt sdk.Coins) error
    SendCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
    DelegateCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
    UndelegateCoinsFromModuleToAccount(ctx sdk.Context, senderModule string, recipientAddr sdk.AccAddress, amt sdk.Coins) error
    MintCoins(ctx sdk.Context, moduleName string, amt sdk.Coins) error
    BurnCoins(ctx sdk.Context, moduleName string, amt sdk.Coins) error

    DelegateCoins(ctx sdk.Context, delegatorAddr, moduleAccAddr sdk.AccAddress, amt sdk.Coins) error
    UndelegateCoins(ctx sdk.Context, moduleAccAddr, delegatorAddr sdk.AccAddress, amt sdk.Coins) error

    types.QueryServer
}
```

## SendKeeper

SendKeeperは、口座残高へのアクセスと、間でコインを転送する機能を提供します
アカウント。 送信マネージャーは、総供給量（コインの鋳造または燃焼）を変更しません。

```go
// SendKeeper defines a module interface that facilitates the transfer of coins
// between accounts without the possibility of creating coins.
type SendKeeper interface {
    ViewKeeper

    InputOutputCoins(ctx sdk.Context, inputs []types.Input, outputs []types.Output) error
    SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error

    GetParams(ctx sdk.Context) types.Params
    SetParams(ctx sdk.Context, params types.Params)

    IsSendEnabledCoin(ctx sdk.Context, coin sdk.Coin) bool
    IsSendEnabledCoins(ctx sdk.Context, coins ...sdk.Coin) error

    BlockedAddr(addr sdk.AccAddress) bool
}
```

## ViewKeeper

ViewKeeperは、アカウント残高への読み取り専用アクセスを提供します。 ビューホルダーにはバランス変更機能はありません。 すべてのバランスルックアップは「O（1）」です。

```go
// ViewKeeper defines a module interface that facilitates read only access to
// account balances.
type ViewKeeper interface {
    ValidateBalance(ctx sdk.Context, addr sdk.AccAddress) error
    HasBalance(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coin) bool

    GetAllBalances(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
    GetAccountsBalances(ctx sdk.Context) []types.Balance
    GetBalance(ctx sdk.Context, addr sdk.AccAddress, denom string) sdk.Coin
    LockedCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
    SpendableCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins

    IterateAccountBalances(ctx sdk.Context, addr sdk.AccAddress, cb func(coin sdk.Coin) (stop bool))
    IterateAllBalances(ctx sdk.Context, cb func(address sdk.AccAddress, coin sdk.Coin) (stop bool))
}
```
