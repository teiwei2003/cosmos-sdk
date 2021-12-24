# お客様

## コマンドラインインターフェイス

ユーザーはCLIを使用して、「bank」モジュールを照会および操作できます。

### query

`query`コマンドを使用すると、ユーザーは` bank`のステータスを照会できます。

```
simd query bank --help
```

#### balances

`balances`コマンドを使用すると、ユーザーは住所で口座残高を照会できます。
```
simd query bank balances [address] [flags]
```

例:

```
simd query bank balances cosmos1..
```

出力例:

```
balances:
- amount: "1000000000"
  denom: stake
pagination:
  next_key: null
  total: "0"
```

#### denom-metadata

`denom-metadata`コマンドを使用すると、ユーザーは硬貨の種類のメタデータを照会できます。 ユーザーは `--denom`フラグを使用して、単一の宗派のメタデータ、またはそれがないすべての宗派のメタデータを照会できます。

```
simd query bank denom-metadata [flags]
```

例:

```
simd query bank denom-metadata --denom stake
```

出力例:

```
metadata:
  base: stake
  denom_units:
  - aliases:
    - STAKE
    denom: stake
  description: native staking token of simulation app
  display: stake
  name: SimApp Token
  symbol: STK
```

#### total

`total`コマンドを使用すると、ユーザーはコインの総供給量を照会できます。 ユーザーは、 `--denom`フラグまたはそれなしのすべてのコインを使用して、単一のコインの総供給量を照会できます。

```
simd query bank total [flags]
```

例:

```
simd query bank total --denom stake
```

出力例:

```
amount: "10000000000"
denom: stake
```

### Transactions

`tx`コマンドを使用すると、ユーザーは` bank`モジュールを操作できます。

```
simd tx bank --help
```

#### send

`send`コマンドを使用すると、ユーザーはあるアカウントから別のアカウントに資金を送ることができます。

```
simd tx bank send [from_key_or_address] [to_address] [amount] [flags]
```

例:

```
simd tx bank send cosmos1.. cosmos1.. 100stake
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `bank`モジュールにクエリを実行できます。

### Balance

`Balance`エンドポイントを使用すると、ユーザーは特定の金種の住所ごとに口座残高を照会できます。

```
cosmos.bank.v1beta1.Query/Balance
```

例:

```
grpcurl -plaintext \
    -d '{"address":"cosmos1..","denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/Balance
```

出力例:

```
{
  "balance": {
    "denom": "stake",
    "amount": "1000000000"
  }
}
```

### AllBalances

`AllBalances`エンドポイントを使用すると、ユーザーはすべての金種の住所ごとに口座残高を照会できます。

```
cosmos.bank.v1beta1.Query/AllBalances
```

例:

```
grpcurl -plaintext \
    -d '{"address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

出力例:

```
{
  "balances": [
    {
      "denom": "stake",
      "amount": "1000000000"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### DenomMetadata

`DenomMetadata`エンドポイントを使用すると、ユーザーは単一の硬貨のメタデータを照会できます。

```
cosmos.bank.v1beta1.Query/DenomMetadata
```

例:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomMetadata
```

出力例:

```
{
  "metadata": {
    "description": "native staking token of simulation app",
    "denomUnits": [
      {
        "denom": "stake",
        "aliases": [
          "STAKE"
        ]
      }
    ],
    "base": "stake",
    "display": "stake",
    "name": "SimApp Token",
    "symbol": "STK"
  }
}
```

### DenomsMetadata

`DenomsMetadata`エンドポイントを使用すると、ユーザーはすべての硬貨のメタデータを照会できます。

```
cosmos.bank.v1beta1.Query/DenomsMetadata
```

例:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomsMetadata
```

出力例:

```
{
  "metadatas": [
    {
      "description": "native staking token of simulation app",
      "denomUnits": [
        {
          "denom": "stake",
          "aliases": [
            "STAKE"
          ]
        }
      ],
      "base": "stake",
      "display": "stake",
      "name": "SimApp Token",
      "symbol": "STK"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### DenomOwners

`DenomOwners`エンドポイントを使用すると、ユーザーは単一の硬貨のメタデータを照会できます。

```
cosmos.bank.v1beta1.Query/DenomOwners
```

例:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomOwners
```

出力例:

```
{
  "denomOwners": [
    {
      "address": "cosmos1..",
      "balance": {
        "denom": "stake",
        "amount": "5000000000"
      }
    },
    {
      "address": "cosmos1..",
      "balance": {
        "denom": "stake",
        "amount": "5000000000"
      }
    },
  ],
  "pagination": {
    "total": "2"
  }
}
```

### TotalSupply

`TotalSupply`エンドポイントを使用すると、ユーザーはすべてのコインの総供給量を照会できます。

```
cosmos.bank.v1beta1.Query/TotalSupply
```

例:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/TotalSupply
```

出力例:

```
{
  "supply": [
    {
      "denom": "stake",
      "amount": "10000000000"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

### SupplyOf

`Supply Of`エンドポイントを使用すると、ユーザーは1枚のコインの総供給量を照会できます。

```
cosmos.bank.v1beta1.Query/SupplyOf
```

例:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/SupplyOf
```

出力例:

```
{
  "amount": {
    "denom": "stake",
    "amount": "10000000000"
  }
}
```

### Params

`Params`エンドポイントを使用すると、ユーザーは` bank`モジュールのパラメーターを照会できます。

```
cosmos.bank.v1beta1.Query/Params
```

例:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/Params
```

出力例:

```
{
  "params": {
    "defaultSendEnabled": true
  }
}
```
