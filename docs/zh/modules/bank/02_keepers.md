# 守门员

bank 模块提供了这些导出的 keeper 接口，可以
传递给读取或更新帐户余额的其他模块。模块
应该使用提供它们的功能的最少许可的接口
要求。

最佳实践要求仔细审查“银行”模块代码，以确保
权限以您期望的方式受到限制。

## 屏蔽地址

`x/bank` 模块接受被视为阻止列表的地址映射
通过“MsgSend”等方式直接明确地接收资金，
`MsgMultiSend` 和直接 API 调用，如 `SendCoinsFromModuleToAccount`。

通常，这些地址是模块帐户。如果这些地址收到资金
在状态机的预期规则之外，不变量很可能是
损坏并可能导致网络停止。

通过为“x/bank”模块提供一组被屏蔽的地址，如果用户或客户试图直接或间接地将资金发送到被屏蔽的账户，例如，通过使用 [IBC](http://docs.cosmos.network/master/ibc/)。

## 常见类型

### 输入

多方转移的输入 

```protobuf
//Input models transaction input.
message Input {
  string   address                        = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}
```

### Output

An output of a multiparty transfer.

```protobuf
//Output models transaction outputs.
message Output {
  string   address                        = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}
```

## BaseKeeper

Base keeper 提供完全权限访问:能够任意修改任何帐户的余额并铸造或燃烧硬币。 

```go
//Keeper defines a module interface that facilitates the transfer of coins
//between accounts.
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

发送保管员提供对帐户余额的访问以及在之间转移硬币的能力
帐户。 发送管理员不会改变总供应量(铸造或燃烧硬币)。 

```go
//SendKeeper defines a module interface that facilitates the transfer of coins
//between accounts without the possibility of creating coins.
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

视图管理器提供对帐户余额的只读访问。 视图保持器没有平衡更改功能。 所有余额查找都是“O(1)”。 

```go
//ViewKeeper defines a module interface that facilitates read only access to
//account balances.
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
