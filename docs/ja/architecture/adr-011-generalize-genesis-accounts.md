# ADR 011:ジェネシスアカウントを宣伝する

## 変更ログ

-2019-08-30:最初のドラフト

## 環境

現在、Cosmos SDKはカスタムアカウントタイプを許可しています。`auth`キーパーは、その `Account`インターフェイスを満たすすべてのタイプを保存します。ただし、 `auth`はジェネシスファイルからのアカウントのエクスポートまたはロードを処理しません。これは、4つの特定のアカウントタイプ(` BaseAccount`、 `ContinuousVestingAccount`、` DelayedVestingAccount`、および `ModuleAccount)のみを処理する` genaccounts`によって行われます。 `)。

カスタムアカウント(カスタムアトリビューションアカウントなど)を使用するプロジェクトは、[genaccounts]をフォークして変更する必要があります。

## 決定

全体として、[genaccounts]の[GenesisAccount]タイプに変換する代わりに、アミノを使用してすべてのアカウント(インターフェースタイプ)を直接グループ化します(キャンセル)。そうすることで `genaccounts`のコードのほとんどが削除されるので、` genaccounts`を `auth`にマージします。グループ化されたアカウントは、[auth]の作成状態で保存されます。

詳細な変更:

### 1)(Un)マーシャルアカウントはアミノを直接使用します

`auth`モジュールの` GenesisState`は、新しいフィールド `Accounts`を取得しました。セクション3で概説されている理由により、これらはタイプ `exported.Account`ではないことに注意してください。 

```go
//GenesisState - all auth state that must be provided at genesis
type GenesisState struct {
    Params   Params           `json:"params" yaml:"params"`
    Accounts []GenesisAccount `json:"accounts" yaml:"accounts"`
}
```

これで、 `auth`と定義されたパラメータの` InitGenesis`と `ExportGenesis`(非)マーシャルアカウントが作成されました。  

```go
//InitGenesis - Init store state from genesis data
func InitGenesis(ctx sdk.Context, ak AccountKeeper, data GenesisState) {
    ak.SetParams(ctx, data.Params)
  ./load the accounts
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

### 2) `auth`コーデックにカスタムアカウントタイプを登録します

`auth`コーデックは、それらをグループ化するためにすべてのカスタムアカウントタイプを登録する必要があります。 [政府]で確立された提案モデルに従います。

カスタムアカウント定義の例: 

```go
import authtypes "github.com/cosmos/cosmos-sdk/x/auth/types"

//Register the module account type with the auth module codec so it can decode module accounts stored in a genesis file
func init() {
    authtypes.RegisterAccountTypeCodec(ModuleAccount{}, "cosmos-sdk/ModuleAccount")
}

type ModuleAccount struct {
    ...
```

`auth`コーデックの定義: 

```go
var ModuleCdc *codec.LegacyAmino

func init() {
    ModuleCdc = codec.NewLegacyAmino()
  ./register module msg's and Account interface
    ...
  ./leave the codec unsealed
}

//RegisterAccountTypeCodec registers an external account type defined in another module for the internal ModuleCdc.
func RegisterAccountTypeCodec(o interface{}, name string) {
    ModuleCdc.RegisterConcrete(o, name, nil)
}
```

### 3)カスタムアカウントタイプの作成検証

モジュールは `ValidateGenesis`メソッドを実装します。 [auth]はアカウントの実現を知らないため、アカウントを自己確認する必要があります。

アカウントを、Validateメソッドを含むGenesisAccountインターフェースにグループ解除します。 

```go
type GenesisAccount interface {
    exported.Account
    Validate() error
}
```

次に、 `auth``ValidateGenesis`関数は次のようになります。 

```go
//ValidateGenesis performs basic validation of auth genesis data returning an
//error for any failed validation criteria.
func ValidateGenesis(data GenesisState) error {
  ./Validate params
    ...

  ./Validate accounts
    addrMap := make(map[string]bool, len(data.Accounts))
    for _, acc := range data.Accounts {

      ./check for duplicated accounts
        addrStr := acc.GetAddress().String()
        if _, ok := addrMap[addrStr]; ok {
            return fmt.Errorf("duplicate account found in genesis state; address: %s", addrStr)
        }
        addrMap[addrStr] = true

      ./check account specific validation
        if err := acc.Validate(); err != nil {
            return fmt.Errorf("invalid account found in genesis state; address: %s, error: %s", addrStr, err.Error())
        }

    }
    return nil
}
```

### 4)add-genesis-accountcliを `auth`に移動します

`genaccounts`モジュールには、ジェネシスファイルに基本アカウントまたは帰属アカウントを追加するためのcliコマンドが含まれています。

これは `auth`に移動されます。 カスタムアカウントを追加するための独自のコマンドを作成するのはプロジェクトに任せます。 `gov`に似た拡張可能なCLIハンドラーを作成することは可能ですが、この2番目のユースケースでは、複雑さはそれだけの価値はありません。

### 5)モジュールとアトリビューションアカウントを更新する

新しいスキームでは、モジュールとアトリビューションアカウントタイプにいくつかのマイナーアップデートが必要です。

-`auth`のコーデックでのタイプ登録(上記のように)
-[アカウント]の特定のタイプごとに[検証]メソッド

## 状態

提案

## 結果

### ポジティブ

-カスタムアカウントを使用するために `genaccounts`をフォークする必要はありません
-コードの行数を減らす

### ネガティブ

### ニュートラル

-`genaccounts`モジュールはもう存在しません
-ジェネシスファイルのアカウントは、[genaccounts]モジュールではなく、[auth]の[accounts]の下に保存されます。
-`add-genesis-account`cliコマンドが `auth`になりました

## 参照 