# お客様

# 認証

## コマンドラインインターフェイス

ユーザーはCLIを使用して、[auth]モジュールを照会および操作できます。

### query

`query`コマンドを使用すると、ユーザーは` auth`のステータスを照会できます。

```bash
simd query auth --help
```

#### account

`account`コマンドを使用すると、ユーザーは自分のアドレスでアカウントを照会できます。 

```bash
simd query auth account [address] [flags]
```

Example:

```bash
simd query auth account cosmos1...
```

Example Output:

```bash
'@type':/cosmos.auth.v1beta1.BaseAccount
account_number: "0"
address: cosmos1zwg6tpl8aw4rawv8sgag9086lpw5hv33u5ctr2
pub_key:
  '@type':/cosmos.crypto.secp256k1.PubKey
  key: ApDrE38zZdd7wLmFS9YmqO684y5DG6fjZ4rVeihF/AQD
sequence: "1"
```

#### accounts

`accounts`コマンドを使用すると、ユーザーは使用可能なすべてのアカウントを照会できます。

```bash
simd query auth accounts [flags]
```

Example:

```bash
simd query auth accounts
```

Example Output:

```bash
accounts:
- '@type':/cosmos.auth.v1beta1.BaseAccount
  account_number: "0"
  address: cosmos1zwg6tpl8aw4rawv8sgag9086lpw5hv33u5ctr2
  pub_key:
    '@type':/cosmos.crypto.secp256k1.PubKey
    key: ApDrE38zZdd7wLmFS9YmqO684y5DG6fjZ4rVeihF/AQD
  sequence: "1"
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "8"
    address: cosmos1yl6hdjhmkf37639730gffanpzndzdpmhwlkfhr
    pub_key: null
    sequence: "0"
  name: transfer
  permissions:
  - minter
  - burner
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "4"
    address: cosmos1fl48vsnmsdzcv85q5d2q4z5ajdha8yu34mf0eh
    pub_key: null
    sequence: "0"
  name: bonded_tokens_pool
  permissions:
  - burner
  - staking
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "5"
    address: cosmos1tygms3xhhs3yv487phx3dw4a95jn7t7lpm470r
    pub_key: null
    sequence: "0"
  name: not_bonded_tokens_pool
  permissions:
  - burner
  - staking
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "6"
    address: cosmos10d07y265gmmuvt4z0w9aw880jnsr700j6zn9kn
    pub_key: null
    sequence: "0"
  name: gov
  permissions:
  - burner
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "3"
    address: cosmos1jv65s3grqf6v6jl3dp4t6c9t9rk99cd88lyufl
    pub_key: null
    sequence: "0"
  name: distribution
  permissions: []
- '@type':/cosmos.auth.v1beta1.BaseAccount
  account_number: "1"
  address: cosmos147k3r7v2tvwqhcmaxcfql7j8rmkrlsemxshd3j
  pub_key: null
  sequence: "0"
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "7"
    address: cosmos1m3h30wlvsf8llruxtpukdvsy0km2kum8g38c8q
    pub_key: null
    sequence: "0"
  name: mint
  permissions:
  - minter
- '@type':/cosmos.auth.v1beta1.ModuleAccount
  base_account:
    account_number: "2"
    address: cosmos17xpfvakm2amg962yls6f84z3kell8c5lserqta
    pub_key: null
    sequence: "0"
  name: fee_collector
  permissions: []
pagination:
  next_key: null
  total: "0"
```

#### params

`params`コマンドを使用すると、ユーザーは現在の認証パラメータを照会できます。

```bash
simd query auth params [flags]
```

例:

```bash
simd query auth params
```

出力例:

```bash
max_memo_characters: "256"
sig_verify_cost_ed25519: "590"
sig_verify_cost_secp256k1: "1000"
tx_sig_limit: "7"
tx_size_cost_per_byte: "10"
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `auth`モジュールをクエリできます。

### account

`account`エンドポイントを使用すると、ユーザーはそのアドレスでアカウントを照会できます。

```bash
cosmos.auth.v1beta1.Query/Account
```

例:

```bash
grpcurl -plaintext \
    -d '{"address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.auth.v1beta1.Query/Account
```

出力例:

```bash
{
  "account":{
    "@type":"/cosmos.auth.v1beta1.BaseAccount",
    "address":"cosmos1zwg6tpl8aw4rawv8sgag9086lpw5hv33u5ctr2",
    "pubKey":{
      "@type":"/cosmos.crypto.secp256k1.PubKey",
      "key":"ApDrE38zZdd7wLmFS9YmqO684y5DG6fjZ4rVeihF/AQD"
    },
    "sequence":"1"
  }
}
```

### accounts

The `accounts` endpoint allow users to query all the available accounts.

```bash
cosmos.auth.v1beta1.Query/Accounts
```

例:

```bash
grpcurl -plaintext \
    localhost:9090 \
    cosmos.auth.v1beta1.Query/Accounts
```

出力例:

```bash
{
   "accounts":[
      {
         "@type":"/cosmos.auth.v1beta1.BaseAccount",
         "address":"cosmos1zwg6tpl8aw4rawv8sgag9086lpw5hv33u5ctr2",
         "pubKey":{
            "@type":"/cosmos.crypto.secp256k1.PubKey",
            "key":"ApDrE38zZdd7wLmFS9YmqO684y5DG6fjZ4rVeihF/AQD"
         },
         "sequence":"1"
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos1yl6hdjhmkf37639730gffanpzndzdpmhwlkfhr",
            "accountNumber":"8"
         },
         "name":"transfer",
         "permissions":[
            "minter",
            "burner"
         ]
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos1fl48vsnmsdzcv85q5d2q4z5ajdha8yu34mf0eh",
            "accountNumber":"4"
         },
         "name":"bonded_tokens_pool",
         "permissions":[
            "burner",
            "staking"
         ]
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos1tygms3xhhs3yv487phx3dw4a95jn7t7lpm470r",
            "accountNumber":"5"
         },
         "name":"not_bonded_tokens_pool",
         "permissions":[
            "burner",
            "staking"
         ]
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos10d07y265gmmuvt4z0w9aw880jnsr700j6zn9kn",
            "accountNumber":"6"
         },
         "name":"gov",
         "permissions":[
            "burner"
         ]
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos1jv65s3grqf6v6jl3dp4t6c9t9rk99cd88lyufl",
            "accountNumber":"3"
         },
         "name":"distribution"
      },
      {
         "@type":"/cosmos.auth.v1beta1.BaseAccount",
         "accountNumber":"1",
         "address":"cosmos147k3r7v2tvwqhcmaxcfql7j8rmkrlsemxshd3j"
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos1m3h30wlvsf8llruxtpukdvsy0km2kum8g38c8q",
            "accountNumber":"7"
         },
         "name":"mint",
         "permissions":[
            "minter"
         ]
      },
      {
         "@type":"/cosmos.auth.v1beta1.ModuleAccount",
         "baseAccount":{
            "address":"cosmos17xpfvakm2amg962yls6f84z3kell8c5lserqta",
            "accountNumber":"2"
         },
         "name":"fee_collector"
      }
   ],
   "pagination":{
      "total":"9"
   }
}
```

### params

`params`エンドポイントを使用すると、ユーザーは現在の認証パラメータを照会できます。

```bash
cosmos.auth.v1beta1.Query/Params
```

例:

```bash
grpcurl -plaintext \
    localhost:9090 \
    cosmos.auth.v1beta1.Query/Params
```

出力例:

```bash
{
  "params": {
    "maxMemoCharacters": "256",
    "txSigLimit": "7",
    "txSizeCostPerByte": "10",
    "sigVerifyCostEd25519": "590",
    "sigVerifyCostSecp256k1": "1000"
  }
}
```

## 休み

ユーザーは、RESTエンドポイントを使用して `auth`モジュールにクエリを実行できます。

### account

`account`エンドポイントを使用すると、ユーザーはそのアドレスでアカウントを照会できます。

```bash
/cosmos/auth/v1beta1/account?address={address}
```

### accounts

`accounts`エンドポイントを使用すると、ユーザーは使用可能なすべてのアカウントを照会できます。

```bash
/cosmos/auth/v1beta1/accounts
```

### params

`params`エンドポイントを使用すると、ユーザーは現在の認証パラメータを照会できます。

```bash
/cosmos/auth/v1beta1/params
```

# Vesting

## CLI

ユーザーは、CLIを使用して `vesting`モジュールを照会および操作できます。

### Transactions

`tx`コマンドを使用すると、ユーザーは` vesting`モジュールを操作できます。

```bash
simd tx vesting --help
```

#### create-periodic-vesting-account

`create-periodic-vesting-account`コマンドは、トークンの割り当てで資金を供給される新しい権利確定アカウントを作成します。ここで、コインのシーケンスと期間の長さは秒単位です。 期間は連続しており、期間の期間は前の期間の終わりにのみ開始されます。 最初の期間の期間は、アカウントの作成時に始まります。

```bash
simd tx vesting create-periodic-vesting-account [to_address] [periods_json_file] [flags]
```

Example:

```bash
simd tx vesting create-periodic-vesting-account cosmos1.. periods.json
```

#### create-vesting-account

`create-vesting-account`コマンドは、トークンの割り当てで資金提供される新しい権利確定アカウントを作成します。 アカウントは、[-delayed]フラグによって決定される遅延または継続的な権利確定アカウントのいずれかです。 作成されたすべての権利確定アカウントには、コミットされたブロックの時間によって設定された開始時間があります。 end_timeは、UNIXエポックタイムスタンプとして指定する必要があります。

```bash
simd tx vesting create-vesting-account [to_address] [amount] [end_time] [flags]
```

Example:

```bash
simd tx vesting create-vesting-account cosmos1.. 100stake 2592000
```
