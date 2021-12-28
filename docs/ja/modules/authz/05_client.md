# お客様

## コマンドラインインターフェイス

ユーザーはCLIを使用して、「authz」モジュールを照会および操作できます。

### query

`query` 命令允许用户查询 `authz` 状态。 

```bash
simd query authz --help
```

#### grants

`grants`コマンドを使用すると、ユーザーは承認者の承認を受けた人の承認を照会できます。 メッセージタイプのURLが設定されている場合、そのメッセージタイプの認証のみが選択されます。

```bash
simd query authz grants [granter-addr] [grantee-addr] [msg-type-url]? [flags]
```

例:

```bash
simd query authz grants cosmos1.. cosmos1.. /cosmos.bank.v1beta1.MsgSend
```

出力例:

```bash
grants:
- authorization:
    '@type': /cosmos.bank.v1beta1.SendAuthorization
    spend_limit:
    - amount: "100"
      denom: stake
  expiration: "2022-01-01T00:00:00Z"
pagination: null
```

### Transactions

`tx`コマンドを使用すると、ユーザーは` authz`モジュールを操作できます。

```bash
simd tx authz --help
```

#### exec

`exec`コマンドを使用すると、被付与者は、付与者に代わってトランザクションを実行できます。

```bash
  simd tx authz exec [tx-json-file] --from [grantee] [flags]
```

例:

```bash
simd tx authz exec tx.json --from=cosmos1..
```

#### grant

`grant`コマンドを使用すると、付与者は被付与者に許可を与えることができます。

```bash
simd tx authz grant <grantee> <authorization_type="send"|"generic"|"delegate"|"unbond"|"redelegate"> --from <granter> [flags]
```

例:

```bash
simd tx authz grant cosmos1.. send --spend-limit=100stake --from=cosmos1..
```

#### revoke

`revoke`コマンドを使用すると、付与者は被付与者からの承認を取り消すことができます。

```bash
simd tx authz revoke [grantee] [msg-type-url] --from=[granter] [flags]
```

例:

```bash
simd tx authz revoke cosmos1.. /cosmos.bank.v1beta1.MsgSend --from=cosmos1..
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `authz`モジュールをクエリできます。

### Grants

`Grants`エンドポイントを使用すると、ユーザーは、助成者と助成金のペアの助成金を照会できます。 メッセージタイプのURLが設定されている場合、そのメッセージタイプに対してのみ許可が選択されます。

```bash
cosmos.authz.v1beta1.Query/Grants
```

例:

```bash
grpcurl -plaintext \
    -d '{"granter":"cosmos1..","grantee":"cosmos1..","msg_type_url":"/cosmos.bank.v1beta1.MsgSend"}' \
    localhost:9090 \
    cosmos.authz.v1beta1.Query/Grants
```

出力例:

```bash
{
  "grants": [
    {
      "authorization": {
        "@type": "/cosmos.bank.v1beta1.SendAuthorization",
        "spendLimit": [
          {
            "denom":"stake",
            "amount":"100"
          }
        ]
      },
      "expiration": "2022-01-01T00:00:00Z"
    }
  ]
}
```

## 休み

ユーザーは、RESTエンドポイントを使用して `authz`モジュールを照会できます。

```bash
/cosmos/authz/v1beta1/grants
```

例:

```bash
curl "localhost:1317/cosmos/authz/v1beta1/grants?granter=cosmos1..&grantee=cosmos1..&msg_type_url=/cosmos.bank.v1beta1.MsgSend"
```

出力例:

```bash
{
  "grants": [
    {
      "authorization": {
        "@type": "/cosmos.bank.v1beta1.SendAuthorization",
        "spend_limit": [
          {
            "denom": "stake",
            "amount": "100"
          }
        ]
      },
      "expiration": "2022-01-01T00:00:00Z"
    }
  ],
  "pagination": null
}
```
