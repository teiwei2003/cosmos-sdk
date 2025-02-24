# 客户

## 命令行界面

用户可以使用 CLI 查询“bank”模块并与之交互。

### 询问

`query` 命令允许用户查询 `bank` 状态。 

```
simd query bank --help
```

#### balances

`balances` 命令允许用户按地址查询账户余额。 
```
simd query bank balances [address] [flags]
```

Example:

```
simd query bank balances cosmos1..
```

Example Output:

```
balances:
- amount: "1000000000"
  denom: stake
pagination:
  next_key: null
  total: "0"
```

#### denom-metadata

`denom-metadata` 命令允许用户查询硬币面额的元数据。 用户可以使用 `--denom` 标志查询单个面额的元数据，或者没有它的所有面额。 

```
simd query bank denom-metadata [flags]
```

Example:

```
simd query bank denom-metadata --denom stake
```

Example Output:

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

`total` 命令允许用户查询硬币的总供应量。 用户可以使用 `--denom` 标志或没有它的所有硬币查询单个硬币的总供应量。 

```
simd query bank total [flags]
```

Example:

```
simd query bank total --denom stake
```

Example Output:

```
amount: "10000000000"
denom: stake
```

### Transactions

The `tx` commands allow users to interact with the `bank` module.

```
simd tx bank --help
```

#### send

The `send` command allows users to send funds from one account to another.

```
simd tx bank send [from_key_or_address] [to_address] [amount] [flags]
```

Example:

```
simd tx bank send cosmos1.. cosmos1.. 100stake
```

## gRPC

A user can query the `bank` module using gRPC endpoints.

### Balance

The `Balance` endpoint allows users to query account balance by address for a given denomination.

```
cosmos.bank.v1beta1.Query/Balance
```

Example:

```
grpcurl -plaintext \
    -d '{"address":"cosmos1..","denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/Balance
```

Example Output:

```
{
  "balance": {
    "denom": "stake",
    "amount": "1000000000"
  }
}
```

### AllBalances

The `AllBalances` endpoint allows users to query account balance by address for all denominations.

```
cosmos.bank.v1beta1.Query/AllBalances
```

Example:

```
grpcurl -plaintext \
    -d '{"address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

Example Output:

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

The `DenomMetadata` endpoint allows users to query metadata for a single coin denomination.

```
cosmos.bank.v1beta1.Query/DenomMetadata
```

Example:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomMetadata
```

Example Output:

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

The `DenomsMetadata` endpoint allows users to query metadata for all coin denominations.

```
cosmos.bank.v1beta1.Query/DenomsMetadata
```

Example:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomsMetadata
```

Example Output:

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

The `DenomOwners` endpoint allows users to query metadata for a single coin denomination.

```
cosmos.bank.v1beta1.Query/DenomOwners
```

Example:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/DenomOwners
```

Example Output:

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

The `TotalSupply` endpoint allows users to query the total supply of all coins.

```
cosmos.bank.v1beta1.Query/TotalSupply
```

Example:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/TotalSupply
```

Example Output:

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

The `SupplyOf` endpoint allows users to query the total supply of a single coin.

```
cosmos.bank.v1beta1.Query/SupplyOf
```

Example:

```
grpcurl -plaintext \
    -d '{"denom":"stake"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/SupplyOf
```

Example Output:

```
{
  "amount": {
    "denom": "stake",
    "amount": "10000000000"
  }
}
```

### Params

The `Params` endpoint allows users to query the parameters of the `bank` module.

```
cosmos.bank.v1beta1.Query/Params
```

Example:

```
grpcurl -plaintext \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/Params
```

Example Output:

```
{
  "params": {
    "defaultSendEnabled": true
  }
}
```
