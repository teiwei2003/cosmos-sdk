# CLI

ユーザーは、CLIを使用して`slashing`モジュールを照会および操作できます。

### Query

`query` 命令允许用户查询 `slashing` 状态。 

```bash
simd query slashing --help
```

#### params

`params`コマンドを使用すると、ユーザーはスラッシュモジュールのジェネシスパラメータを照会できます。

```bash
simd query slashing params [flags]
```

例:

```bash
simd query slashing params
```

出力例:

```bash
downtime_jail_duration: 600s
min_signed_per_window: "0.500000000000000000"
signed_blocks_window: "100"
slash_fraction_double_sign: "0.050000000000000000"
slash_fraction_downtime: "0.010000000000000000"
```

#### signing-info

`signing-info`コマンドを使用すると、ユーザーはコンセンサス公開鍵を使用してバリデーターの署名情報を照会できます。

```bash
simd query slashing signing-infos [flags]
```

例:

```bash
simd query slashing signing-info '{"@type":"/cosmos.crypto.ed25519.PubKey","key":"Auxs3865HpB/EfssYOzfqNhEJjzys6jD5B6tPgC8="}'

```

出力例:

```bash
address: cosmosvalcons1nrqsld3aw6lh6t082frdqc84uwxn0t958c
index_offset: "2068"
jailed_until: "1970-01-01T00:00:00Z"
missed_blocks_counter: "0"
start_height: "0"
tombstoned: false
```

#### signing-infos

`signing-infos`コマンドを使用すると、ユーザーはすべてのバリデーターの署名情報を照会できます。

```bash
simd query slashing signing-infos [flags]
```

例:

```bash
simd query slashing signing-infos
```

出力例:

```bash
info:
- address: cosmosvalcons1nrqsld3aw6lh6t082frdqc84uwxn0t958c
  index_offset: "2075"
  jailed_until: "1970-01-01T00:00:00Z"
  missed_blocks_counter: "0"
  start_height: "0"
  tombstoned: false
pagination:
  next_key: null
  total: "0"
```

### Transactions

`tx`コマンドを使用すると、ユーザーは` slashing`モジュールを操作できます。

```bash
simd tx slashing --help
```

#### unjail

`unjail`コマンドを使用すると、ユーザーは以前にダウンタイムのために投獄されていたバリデーターを投獄解除できます。

```bash
  simd tx slashing unjail --from mykey [flags]
```

例:

```bash
simd tx slashing unjail --from mykey
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `slashing`モジュールをクエリできます。

### Params

`Params`エンドポイントを使用すると、ユーザーはスラッシュモジュールのパラメーターを照会できます。

```bash
cosmos.slashing.v1beta1.Query/Params
```

例:

```bash
grpcurl -plaintext localhost:9090 cosmos.slashing.v1beta1.Query/Params
```

出力例:

```bash
{
  "params": {
    "signedBlocksWindow": "100",
    "minSignedPerWindow": "NTAwMDAwMDAwMDAwMDAwMDAw",
    "downtimeJailDuration": "600s",
    "slashFractionDoubleSign": "NTAwMDAwMDAwMDAwMDAwMDA=",
    "slashFractionDowntime": "MTAwMDAwMDAwMDAwMDAwMDA="
  }
}
```

### SigningInfo

SigningInfoは、指定された短所アドレスの署名情報を照会します。

```bash
cosmos.slashing.v1beta1.Query/SigningInfo
```

例:

```bash
grpcurl -plaintext -d '{"cons_address":"cosmosvalcons1nrqsld3aw6lh6t082frdqc84uwxn0t958c"}' localhost:9090 cosmos.slashing.v1beta1.Query/SigningInfo
```

出力例:

```bash
{
  "valSigningInfo": {
    "address": "cosmosvalcons1nrqsld3aw6lh6t082frdqc84uwxn0t958c",
    "indexOffset": "3493",
    "jailedUntil": "1970-01-01T00:00:00Z"
  }
}
```

### SigningInfos

SigningInfosは、すべてのバリデーターの署名情報を照会します。

```bash
cosmos.slashing.v1beta1.Query/SigningInfos
```

例:

```bash
grpcurl -plaintext localhost:9090 cosmos.slashing.v1beta1.Query/SigningInfos
```

出力例:

```bash
{
  "info": [
    {
      "address": "cosmosvalcons1nrqslkwd3pz096lh6t082frdqc84uwxn0t958c",
      "indexOffset": "2467",
      "jailedUntil": "1970-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

## REST

ユーザーは、RESTエンドポイントを使用して `slashing`モジュールにクエリを実行できます。

### Params

```bash
/cosmos/slashing/v1beta1/params
```

例:

```bash
curl "localhost:1317/cosmos/slashing/v1beta1/params"
```

出力例:

```bash
{
  "params": {
    "signed_blocks_window": "100",
    "min_signed_per_window": "0.500000000000000000",
    "downtime_jail_duration": "600s",
    "slash_fraction_double_sign": "0.050000000000000000",
    "slash_fraction_downtime": "0.010000000000000000"
}
```

### signing_info

```bash
/cosmos/slashing/v1beta1/signing_infos/%s
```

例:

```bash
curl "localhost:1317/cosmos/slashing/v1beta1/signing_infos/cosmosvalcons1nrqslkwd3pz096lh6t082frdqc84uwxn0t958c"
```

出力例:

```bash
{
  "val_signing_info": {
    "address": "cosmosvalcons1nrqslkwd3pz096lh6t082frdqc84uwxn0t958c",
    "start_height": "0",
    "index_offset": "4184",
    "jailed_until": "1970-01-01T00:00:00Z",
    "tombstoned": false,
    "missed_blocks_counter": "0"
  }
}
```

### signing_infos

```bash
/cosmos/slashing/v1beta1/signing_infos
```

例:

```bash
curl "localhost:1317/cosmos/slashing/v1beta1/signing_infos
```

出力例:

```bash
{
  "info": [
    {
      "address": "cosmosvalcons1nrqslkwd3pz096lh6t082frdqc84uwxn0t958c",
      "start_height": "0",
      "index_offset": "4169",
      "jailed_until": "1970-01-01T00:00:00Z",
      "tombstoned": false,
      "missed_blocks_counter": "0"
    }
  ],
  "pagination": {
    "next_key": null,
    "total": "1"
  }
}
```
