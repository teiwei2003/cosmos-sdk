# お客様

## コマンドラインインターフェイス

ユーザーはCLIを使用して、「配布」モジュールを照会および操作できます。

### query

`query`コマンドを使用すると、ユーザーは` distribution`のステータスを照会できます。

```
simd query distribution --help
```

#### commission

`commission`コマンドを使用すると、ユーザーはアドレスごとにバリデーターのコミッション報酬を照会できます。

```
simd query distribution commission [address] [flags]
```

例:

```
simd query distribution commission cosmosvaloper1..
```

出力例:

```
commission:
- amount: "1000000.000000000000000000"
  denom: stake
```

#### community-pool

`community-pool`コマンドを使用すると、ユーザーはコミュニティプール内のすべてのコイン残高を照会できます。

```
simd query distribution community-pool [flags]
```

例:

```
simd query distribution community-pool
```

出力例:

```
pool:
- amount: "1000000.000000000000000000"
  denom: stake
```

#### params

`params`コマンドを使用すると、ユーザーは` distribution`モジュールのパラメーターを照会できます。

```
simd query distribution params [flags]
```

例:

```
simd query distribution params
```

出力例:

```
base_proposer_reward: "0.010000000000000000"
bonus_proposer_reward: "0.040000000000000000"
community_tax: "0.020000000000000000"
withdraw_addr_enabled: true
```

#### rewards

`rewards`コマンドを使用すると、ユーザーは委任者の報酬を照会できます。 ユーザーは、オプションでバリデーターアドレスを含めて、特定のバリデーターから獲得した報酬を照会できます。

```
simd query distribution rewards [delegator-addr] [validator-addr] [flags]
```

例:

```
simd query distribution rewards cosmos1..
```

出力例:

```
rewards:
- reward:
  - amount: "1000000.000000000000000000"
    denom: stake
  validator_address: cosmosvaloper1..
total:
- amount: "1000000.000000000000000000"
  denom: stake
```

#### slashes

`slashes`コマンドを使用すると、ユーザーは特定のブロック範囲のすべてのスラッシュを照会できます。

```
simd query distribution slashes [validator] [start-height] [end-height] [flags]
```

例:

```
simd query distribution slashes cosmosvaloper1.. 1 1000
```

出力例:

```
pagination:
  next_key: null
  total: "0"
slashes:
- validator_period: 20,
  fraction: "0.009999999999999999"
```

#### validator-outstanding-rewards

`validator-outstanding-rewards`コマンドを使用すると、ユーザーは、バリデーターとそのすべての委任について、すべての未処理の（取り消されていない）報酬を照会できます。

```
simd query distribution validator-outstanding-rewards [validator] [flags]
```

例:

```
simd query distribution validator-outstanding-rewards cosmosvaloper1..
```

出力例:

```
rewards:
- amount: "1000000.000000000000000000"
  denom: stake
```

### Transactions

`tx`コマンドを使用すると、ユーザーは` distribution`モジュールを操作できます。

```
simd tx distribution --help
```

#### fund-community-pool

`fund-community-pool`コマンドを使用すると、ユーザーはコミュニティプールに資金を送ることができます。

```
simd tx distribution fund-community-pool [amount] [flags]
```

例:

```
simd tx distribution fund-community-pool 100stake --from cosmos1..
```

#### set-withdraw-addr

`set-withdraw-addr`コマンドを使用すると、ユーザーは委任者アドレスに関連付けられた報酬の撤回アドレスを設定できます。

```
simd tx distribution set-withdraw-addr [withdraw-addr] [flags]
```

例:

```
simd tx distribution set-withdraw-addr cosmos1.. --from cosmos1..
```

#### withdraw-all-rewards

`withdraw-all-rewards`コマンドを使用すると、ユーザーは委任者のすべての報酬を引き出すことができます。

```
simd tx distribution withdraw-all-rewards [flags]
```

例:

```
simd tx distribution withdraw-all-rewards --from cosmos1..
```

#### withdraw-rewards

`withdraw-rewards`コマンドを使用すると、ユーザーは特定の委任アドレスからすべての報酬を引き出すことができます。
また、指定された委任アドレスがバリデーターオペレーターであり、ユーザーが `--commision`フラグを証明した場合は、オプションでバリデーターコミッションを撤回します。

```
simd tx distribution withdraw-rewards [validator-addr] [flags]
```

例:

```
simd tx distribution withdraw-rewards cosmosvaloper1.. --from cosmos1.. --commision
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `distribution`モジュールにクエリを実行できます。

### Params

`Params`エンドポイントを使用すると、ユーザーは` distribution`モジュールのパラメーターを照会できます。

例:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/Params
```

出力例:

```
{
  "params": {
    "communityTax": "20000000000000000",
    "baseProposerReward": "10000000000000000",
    "bonusProposerReward": "40000000000000000",
    "withdrawAddrEnabled": true
  }
}
```

### ValidatorOutstandingRewards

`ValidatorOutstandingRewards`エンドポイントを使用すると、ユーザーはバリデーターアドレスの報酬を照会できます。

例:

```
grpcurl -plaintext \
    -d '{"validator_address":"cosmosvalop1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/ValidatorOutstandingRewards
```

出力例:

```
{
  "rewards": {
    "rewards": [
      {
        "denom": "stake",
        "amount": "1000000000000000"
      }
    ]
  }
}
```

### ValidatorCommission

`commission`コマンドを使用すると、ユーザーはアドレスで検証者のコミッション報酬を照会できます。 

例:

```
grpcurl -plaintext \
    -d '{"validator_address":"cosmosvalop1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/ValidatorCommission
```

出力例:

```
{
  "commission": {
    "commission": [
      {
        "denom": "stake",
        "amount": "1000000000000000"
      }
    ]
  }
}
```

### ValidatorSlashes

`ValidatorSlashes`エンドポイントを使用すると、ユーザーはバリデーターのスラッシュイベントを照会できます。 

例:

```
grpcurl -plaintext \
    -d '{"validator_address":"cosmosvalop1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/ValidatorSlashes
```

出力例:

```
{
  "slashes": [
    {
      "validator_period": "20",
      "fraction": "0.009999999999999999"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### DelegationRewards

`DelegationRewards`エンドポイントを使用すると、ユーザーは委任によって生成された報酬の合計を照会できます。 

例:

```
grpcurl -plaintext \
    -d '{"delegator_address":"cosmos1..","validator_address":"cosmosvalop1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/DelegationRewards
```

出力例:

```
{
  "rewards": [
    {
      "denom": "stake",
      "amount": "1000000000000000"
    }
  ]
}
```

### DelegationTotalRewards

`DelegationTotalRewards`エンドポイントを使用すると、ユーザーは各バリデーターによって蓄積された報酬の合計を照会できます。。 

例:

```
grpcurl -plaintext \
    -d '{"delegator_address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/DelegationTotalRewards
```

出力例:

```
{
  "rewards": [
    {
      "validatorAddress": "cosmosvaloper1..",
      "reward": [
        {
          "denom": "stake",
          "amount": "1000000000000000"
        }
      ]
    }
  ],
  "total": [
    {
      "denom": "stake",
      "amount": "1000000000000000"
    }
  ]
}
```

### DelegatorValidators

`DelegatorValidators`エンドポイントを使用すると、ユーザーは特定のプリンシパルのすべてのバリデーターにクエリを実行できます。

例:

```
grpcurl -plaintext \
    -d '{"delegator_address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/DelegatorValidators
```

出力例:

```
{
  "validators": [
    "cosmosvaloper1.."
  ]
}
```

### DelegatorWithdrawAddress

`DelegatorWithdrawAddress`エンドポイントを使用すると、ユーザーはプリンシパルの引き出しアドレスを照会できます。 

例:

```
grpcurl -plaintext \
    -d '{"delegator_address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/DelegatorWithdrawAddress
```

出力例:

```
{
  "withdrawAddress": "cosmos1.."
}
```

### CommunityPool

`CommunityPool`エンドポイントを使用すると、ユーザーはコミュニティプールコインを照会できます。

例:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.distribution.v1beta1.Query/CommunityPool
```

出力例:

```
{
  "pool": [
    {
      "denom": "stake",
      "amount": "1000000000000000000"
    }
  ]
}
```
