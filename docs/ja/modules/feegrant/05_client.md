# クライアント

## CLI

ユーザーは、CLIを使用して`feegrant`モジュールを照会および操作できます。

### query

`query`コマンドを使用すると、ユーザーは`feegrant`状態を照会できます。

```
simd query feegrant --help
```

#### grant

`grant`コマンドを使用すると、ユーザーは特定の付与者と付与者のペアの付与を照会できます。

```
simd query feegrant grant [granter] [grantee] [flags]
```

例：

```
simd query feegrant grant cosmos1.. cosmos1..
```

出力例：

```
allowance:
  '@type': /cosmos.feegrant.v1beta1.BasicAllowance
  expiration: null
  spend_limit:
  - amount: "100"
    denom: stake
grantee: cosmos1..
granter: cosmos1..
```

#### grants

`grants`コマンドを使用すると、ユーザーは特定の担当者のすべての承認を照会できます。

```
simd query feegrant grants [grantee] [flags]
```

例：

```
simd query feegrant grants cosmos1..
```

出力例：

```
allowances:
- allowance:
    '@type': /cosmos.feegrant.v1beta1.BasicAllowance
    expiration: null
    spend_limit:
    - amount: "100"
      denom: stake
  grantee: cosmos1..
  granter: cosmos1..
pagination:
  next_key: null
  total: "0"
```

### Transactions

`tx`コマンドを使用すると、ユーザーは`feegrant`モジュールを操作できます。

```
simd tx feegrant --help
```

#### grant

`grant`コマンドを使用すると、ユーザーは別のアカウントに手数料を付与できます。 手数料手当には、有効期限、合計支出制限、および/または定期的な支出制限を含めることができます。 

```
simd tx feegrant grant [granter] [grantee] [flags]
```

Example (one-time spend limit):

```
simd tx feegrant grant cosmos1.. cosmos1.. --spend-limit 100stake
```

Example (periodic spend limit):

```
simd tx feegrant grant cosmos1.. cosmos1.. --period 3600 --period-limit 10stake
```

#### revoke

`revoke`コマンドを使用すると、ユーザーは付与された手数料を取り消すことができます。

```
simd tx feegrant revoke [granter] [grantee] [flags]
```

例：

```
simd tx feegrant revoke cosmos1.. cosmos1..
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して`feegrant`モジュールをクエリできます。

### Allowance

`Allowance`エンドポイントを使用すると、ユーザーは付与された料金の手当を照会できます。 

```
cosmos.feegrant.v1beta1.Query/Allowance
```

例：

```
grpcurl -plaintext \
    -d '{"grantee":"cosmos1..","granter":"cosmos1.."}' \
    localhost:9090 \
    cosmos.feegrant.v1beta1.Query/Allowance
```

出力例：

```
{
  "allowance": {
    "granter": "cosmos1..",
    "grantee": "cosmos1..",
    "allowance": {"@type":"/cosmos.feegrant.v1beta1.BasicAllowance","spendLimit":[{"denom":"stake","amount":"100"}]}
  }
}
```

### Allowances

`Allowances`エンドポイントを使用すると、ユーザーは、特定の被付与者に対して付与されたすべての手数料の許容値を照会できます。

```
cosmos.feegrant.v1beta1.Query/Allowances
```

例：

```
grpcurl -plaintext \
    -d '{"address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.feegrant.v1beta1.Query/Allowances
```

出力例：

```
{
  "allowances": [
    {
      "granter": "cosmos1..",
      "grantee": "cosmos1..",
      "allowance": {"@type":"/cosmos.feegrant.v1beta1.BasicAllowance","spendLimit":[{"denom":"stake","amount":"100"}]}
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```
