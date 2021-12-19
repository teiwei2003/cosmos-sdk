# 客户

##命令行界面

用户可以使用 CLI 查询“feegrant”模块并与之交互。

### 询问

`query` 命令允许用户查询 `feegrant` 状态。 

```
simd query feegrant --help
```

#### grant

The `grant` command allows users to query a grant for a given granter-grantee pair.

```
simd query feegrant grant [granter] [grantee] [flags]
```

Example:

```
simd query feegrant grant cosmos1.. cosmos1..
```

Example Output:

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

`grants` 命令允许用户查询给定受让人的所有授权。 

```
simd query feegrant grants [grantee] [flags]
```

Example:

```
simd query feegrant grants cosmos1..
```

Example Output:

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

`tx` 命令允许用户与 `feegrant` 模块交互。

```
simd tx feegrant --help
```

#### grant

`grant` 命令允许用户向另一个帐户授予费用津贴。 费用津贴可以具有到期日期、总支出限额和/或定期支出限额。 

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

`revoke` 命令允许用户撤销授予的费用津贴。

```
simd tx feegrant revoke [granter] [grantee] [flags]
```

Example:

```
simd tx feegrant revoke cosmos1.. cosmos1..
```

## gRPC

用户可以使用 gRPC 端点查询 `feegrant` 模块。

### Allowance

`Allowance` 端点允许用户查询授予的费用津贴。 

```
cosmos.feegrant.v1beta1.Query/Allowance
```

Example:

```
grpcurl -plaintext \
    -d '{"grantee":"cosmos1..","granter":"cosmos1.."}' \
    localhost:9090 \
    cosmos.feegrant.v1beta1.Query/Allowance
```

Example Output:

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

The `Allowances` endpoint allows users to query all granted fee allowances for a given grantee.

```
cosmos.feegrant.v1beta1.Query/Allowances
```

Example:

```
grpcurl -plaintext \
    -d '{"address":"cosmos1.."}' \
    localhost:9090 \
    cosmos.feegrant.v1beta1.Query/Allowances
```

Example Output:

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
