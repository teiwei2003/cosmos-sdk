# クライアント

## CLI

ユーザーは、CLIを使用して`evidence`モジュールを照会および操作できます。

### Query

`query`コマンドを使用すると、ユーザーは`evidence`状態を照会できます。 

```bash
simd query evidence --help
```

### evidence

`evidence`コマンドを使用すると、ユーザーはすべての証拠または証拠をハッシュで一覧表示できます。 

Usage:

```bash
simd query evidence [flags]
```

ハッシュで証拠を照会するには

例:

```bash
simd query evidence "DF0C23E8634E480F84B9D5674A7CDC9816466DEC28A3358F73260F68D28D7660"
```

出力例:

```bash
evidence:
  consensus_address: cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h
  height: 11
  power: 100
  time: "2021-10-20T16:08:38.194017624Z"
```

すべての証拠を入手するには

例:

```bash
simd query evidence
```

出力例:

```bash
evidence:
  consensus_address: cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h
  height: 11
  power: 100
  time: "2021-10-20T16:08:38.194017624Z"
pagination:
  next_key: null
  total: "1"
```

 ## 休み

ユーザーは、RESTエンドポイントを使用して`evidence`モジュールを照会できます。

### 証拠

ハッシュで証拠を取得する

```bash
/cosmos/evidence/v1beta1/evidence/{evidence_hash}
```

例:

```bash
curl -X GET "http://localhost:1317/cosmos/evidence/v1beta1/evidence/DF0C23E8634E480F84B9D5674A7CDC9816466DEC28A3358F73260F68D28D7660"
```

出力例:

```bash
{
  "evidence": {
    "consensus_address": "cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h",
    "height": "11",
    "power": "100",
    "time": "2021-10-20T16:08:38.194017624Z"
  }
}
```

### すべての証拠

すべての証拠を入手する

```bash
/cosmos/evidence/v1beta1/evidence
```

例:

```bash
curl -X GET "http://localhost:1317/cosmos/evidence/v1beta1/evidence"
```

出力例:

```bash
{
  "evidence": [
    {
      "consensus_address": "cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h",
      "height": "11",
      "power": "100",
      "time": "2021-10-20T16:08:38.194017624Z"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して`evidence`モジュールをクエリできます。

### 証拠

ハッシュで証拠を取得する
```bash
cosmos.evidence.v1beta1.Query/Evidence
```

例:

```bash
grpcurl -plaintext -d'{"evidence_hash":"DF0C23E8634E480F84B9D5674A7CDC9816466DEC28A3358F73260F68D28D7660"}'localhost:9090 cosmos.evidence.v1beta1.Query/Evidence
```

出力例:

```bash
{
  "evidence": {
    "consensus_address": "cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h",
    "height": "11",
    "power": "100",
    "time": "2021-10-20T16:08:38.194017624Z"
  }
}
```

### すべての証拠

すべての証拠を入手する

```bash
cosmos.evidence.v1beta1.Query/AllEvidence
```

例:

```bash
grpcurl -plaintext localhost:9090 cosmos.evidence.v1beta1.Query/AllEvidence
```

出力例:

```bash
{
  "evidence": [
    {
      "consensus_address": "cosmosvalcons1ntk8eualewuprz0gamh8hnvcem2nrcdsgz563h",
      "height": "11",
      "power": "100",
      "time": "2021-10-20T16:08:38.194017624Z"
    }
  ],
  "pagination": {
    "total": "1"
  }
}
```
