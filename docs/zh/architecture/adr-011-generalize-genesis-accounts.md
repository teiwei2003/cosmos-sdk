# ADR 011:推广创世纪账户

## 变更日志

- 2019-08-30:初稿

## 语境

目前，Cosmos SDK 允许自定义帐户类型； `auth` keeper 存储满足其 `Account` 接口的任何类型。然而，`auth` 不处理从创世文件导出或加载帐户，这是由 `genaccounts` 完成的，它只处理 4 种具体帐户类型(`BaseAccount`、`ContinuousVestingAccount`、`DelayedVestingAccount` 和`ModuleAccount)中的一种`)。

希望使用自定义帐户(例如自定义归属帐户)的项目需要分叉和修改“genaccounts”。

## 决定

总而言之，我们将(取消)使用amino 直接编组所有帐户(接口类型)，而不是转换为`genaccounts` 的`GenesisAccount` 类型。由于这样做会删除大部分 `genaccounts` 的代码，我们将把 `genaccounts` 合并到 `auth` 中。编组的帐户将存储在“auth”的创世状态中。

详细变化:

### 1) (Un)Marshal 帐户直接使用氨基

`auth` 模块的 `GenesisState` 获得了一个新的字段 `Accounts`。请注意，由于第 3 节中概述的原因，这些不是 `exported.Account` 类型。

```go
//GenesisState - all auth state that must be provided at genesis
type GenesisState struct {
    Params   Params           `json:"params" yaml:"params"`
    Accounts []GenesisAccount `json:"accounts" yaml:"accounts"`
}
```

现在`auth` 的`InitGenesis` 和`ExportGenesis` (un)marshal 帐户以及定义的参数。 

```go
//InitGenesis - Init store state from genesis data
func InitGenesis(ctx sdk.Context, ak AccountKeeper, data GenesisState) {
    ak.SetParams(ctx, data.Params)
   //load the accounts
    for _, a := range data.Accounts {
        acc := ak.NewAccount(ctx, a)//set account number
        ak.SetAccount(ctx, acc)
    }
}

//ExportGenesis returns a GenesisState for a given context and keeper
func ExportGenesis(ctx sdk.Context, ak AccountKeeper) GenesisState {
    params := ak.GetParams(ctx)

    var genAccounts []exported.GenesisAccount
    ak.IterateAccounts(ctx, func(account exported.Account) bool {
        genAccount := account.(exported.GenesisAccount)
        genAccounts = append(genAccounts, genAccount)
        return false
    })

    return NewGenesisState(params, genAccounts)
}
```

### 2) 在 `auth` 编解码器上注册自定义帐户类型

`auth` 编解码器必须注册所有自定义帐户类型以对其进行编组。 我们将遵循“gov”中建立的提案模式。

自定义帐户定义示例:

```go
import authtypes "github.com/cosmos/cosmos-sdk/x/auth/types"

//Register the module account type with the auth module codec so it can decode module accounts stored in a genesis file
func init() {
    authtypes.RegisterAccountTypeCodec(ModuleAccount{}, "cosmos-sdk/ModuleAccount")
}

type ModuleAccount struct {
    ...
```

`auth` 编解码器定义: 

```go
var ModuleCdc *codec.LegacyAmino

func init() {
    ModuleCdc = codec.NewLegacyAmino()
   //register module msg's and Account interface
    ...
   //leave the codec unsealed
}

//RegisterAccountTypeCodec registers an external account type defined in another module for the internal ModuleCdc.
func RegisterAccountTypeCodec(o interface{}, name string) {
    ModuleCdc.RegisterConcrete(o, name, nil)
}
```

### 3) 自定义账户类型的创世验证

模块实现了一个 `ValidateGenesis` 方法。 由于“auth”不知道帐户的实现，因此帐户需要自我验证。

我们将把账户解组到一个包含一个 Validate 方法的 GenesisAccount 接口中。 

```go
type GenesisAccount interface {
    exported.Account
    Validate() error
}
```

然后`auth``ValidateGenesis`函数变成:

```go
//ValidateGenesis performs basic validation of auth genesis data returning an
//error for any failed validation criteria.
func ValidateGenesis(data GenesisState) error {
   //Validate params
    ...

   //Validate accounts
    addrMap := make(map[string]bool, len(data.Accounts))
    for _, acc := range data.Accounts {

       //check for duplicated accounts
        addrStr := acc.GetAddress().String()
        if _, ok := addrMap[addrStr]; ok {
            return fmt.Errorf("duplicate account found in genesis state; address: %s", addrStr)
        }
        addrMap[addrStr] = true

       //check account specific validation
        if err := acc.Validate(); err != nil {
            return fmt.Errorf("invalid account found in genesis state; address: %s, error: %s", addrStr, err.Error())
        }

    }
    return nil
}
```

### 4) 将 add-genesis-account cli 移动到 `auth`

`genaccounts` 模块包含一个 cli 命令，用于将基本帐户或归属帐户添加到创世文件中。

这将被移动到`auth`。 我们将留给项目编写自己的命令来添加自定义帐户。 可以创建一个类似于 `gov` 的可扩展 cli 处理程序，但对于这个次要用例来说，复杂性不值得。

### 5) 更新模块和归属账户

在新方案下，模块和归属账户类型需要一些小的更新:

- 在 `auth` 的编解码器上键入注册(如上所示)
- 每个“Account”具体类型的“Validate”方法 

## Status

Proposed

## Consequences

### Positive

- 无需分叉`genaccounts`即可使用自定义帐户
- 减少代码行数 

### Negative

### Neutral

- `genaccounts` 模块不再存在
- 创世文件中的帐户存储在“auth”中的“accounts”下，而不是“genaccounts”模块中。
-`add-genesis-account` cli 命令现在在 `auth` 中 

## References
