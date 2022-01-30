# クライアント

## CLI

ユーザーは、CLIを使用して `upgrade`モジュールを照会および操作できます。

### Query

`query`コマンドを使用すると、ユーザーは` upgrade`状態を照会できます。 

```bash
simd query upgrade --help
```

#### applied

`applied`コマンドを使用すると、ユーザーは、完了したアップグレードが適用された高さをブロックヘッダーに問い合わせることができます。 

```bash
simd query upgrade applied [upgrade-name] [flags]
```

upgrade-nameが以前にチェーンで実行されていた場合、これはそれが適用されたブロックのヘッダーを返します。
これは、クライアントが特定の範囲のブロックでどのバイナリが有効であったかを判断するのに役立ち、過去の移行を理解するためのより多くのコンテキストを提供します。 

例:

```bash
simd query upgrade applied "test-upgrade"
```

出力例:

```bash
"block_id": {
    "hash": "A769136351786B9034A5F196DC53F7E50FCEB53B48FA0786E1BFC45A0BB646B5",
    "parts": {
      "total": 1,
      "hash": "B13CBD23011C7480E6F11BE4594EE316548648E6A666B3575409F8F16EC6939E"
    }
  },
  "block_size": "7213",
  "header": {
    "version": {
      "block": "11"
    },
    "chain_id": "testnet-2",
    "height": "455200",
    "time": "2021-04-10T04:37:57.085493838Z",
    "last_block_id": {
      "hash": "0E8AD9309C2DC411DF98217AF59E044A0E1CCEAE7C0338417A70338DF50F4783",
      "parts": {
        "total": 1,
        "hash": "8FE572A48CD10BC2CBB02653CA04CA247A0F6830FF19DC972F64D339A355E77D"
      }
    },
    "last_commit_hash": "DE890239416A19E6164C2076B837CC1D7F7822FC214F305616725F11D2533140",
    "data_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
    "validators_hash": "A31047ADE54AE9072EE2A12FF260A8990BA4C39F903EAF5636B50D58DBA72582",
    "next_validators_hash": "A31047ADE54AE9072EE2A12FF260A8990BA4C39F903EAF5636B50D58DBA72582",
    "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
    "app_hash": "28ECC486AFC332BA6CC976706DBDE87E7D32441375E3F10FD084CD4BAF0DA021",
    "last_results_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
    "evidence_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
    "proposer_address": "2ABC4854B1A1C5AA8403C4EA853A81ACA901CC76"
  },
  "num_txs": "0"
}
```

#### module versions

`module_versions`コマンドは、モジュール名とそれぞれのコンセンサスバージョンのリストを取得します。

特定のモジュール名でコマンドを実行すると、のみが返されます
そのモジュールの情報。

```bash
simd query upgrade module_versions [optional module_name] [flags]
```

例:

```bash
simd query upgrade module_versions
```

出力例:

```bash
module_versions:
- name: auth
  version: "2"
- name: authz
  version: "1"
- name: bank
  version: "2"
- name: capability
  version: "1"
- name: crisis
  version: "1"
- name: distribution
  version: "2"
- name: evidence
  version: "1"
- name: feegrant
  version: "1"
- name: genutil
  version: "1"
- name: gov
  version: "2"
- name: ibc
  version: "2"
- name: mint
  version: "1"
- name: params
  version: "1"
- name: slashing
  version: "2"
- name: staking
  version: "2"
- name: transfer
  version: "1"
- name: upgrade
  version: "1"
- name: vesting
  version: "1"
```

例:

```bash
regen query upgrade module_versions ibc
```

出力例:

```bash
module_versions:
- name: ibc
  version: "2"
```

#### plan

`plan`コマンドは、現在スケジュールされているアップグレードプラン(存在する場合)を取得します。

```bash
regen query upgrade plan [flags]
```

例:

```bash
simd query upgrade plan
```

出力例:

```bash
height: "130"
info: ""
name: test-upgrade
time: "0001-01-01T00:00:00Z"
upgraded_client_state: null
```

## REST

ユーザーは、RESTエンドポイントを使用して `upgrade`モジュールを照会できます。

### Applied Plan

`AppliedPlan`は、以前に適用されたアップグレードプランをその名前で照会します。

```bash
/cosmos/upgrade/v1beta1/applied_plan/{name}
```

例:

```bash
curl -X GET "http://localhost:1317/cosmos/upgrade/v1beta1/applied_plan/v2.0-upgrade" -H "accept: application/json"
```

出力例:

```bash
{
  "height": "30"
}
```

### Current Plan

`CurrentPlan`は、現在のアップグレードプランを照会します。

```bash
/cosmos/upgrade/v1beta1/current_plan
```

例:

```bash
curl -X GET "http://localhost:1317/cosmos/upgrade/v1beta1/current_plan" -H "accept: application/json"
```

出力例:

```bash
{
  "plan": "v2.1-upgrade"
}
```

### Module versions

`ModuleVersions`は、状態からモジュールバージョンのリストを照会します。

```bash
/cosmos/upgrade/v1beta1/module_versions
```

例:

```bash
curl -X GET "http://localhost:1317/cosmos/upgrade/v1beta1/module_versions" -H "accept: application/json"
```

出力例:

```bash
{
  "module_versions": [
    {
      "name": "auth",
      "version": "2"
    },
    {
      "name": "authz",
      "version": "1"
    },
    {
      "name": "bank",
      "version": "2"
    },
    {
      "name": "capability",
      "version": "1"
    },
    {
      "name": "crisis",
      "version": "1"
    },
    {
      "name": "distribution",
      "version": "2"
    },
    {
      "name": "evidence",
      "version": "1"
    },
    {
      "name": "feegrant",
      "version": "1"
    },
    {
      "name": "genutil",
      "version": "1"
    },
    {
      "name": "gov",
      "version": "2"
    },
    {
      "name": "ibc",
      "version": "2"
    },
    {
      "name": "mint",
      "version": "1"
    },
    {
      "name": "params",
      "version": "1"
    },
    {
      "name": "slashing",
      "version": "2"
    },
    {
      "name": "staking",
      "version": "2"
    },
    {
      "name": "transfer",
      "version": "1"
    },
    {
      "name": "upgrade",
      "version": "1"
    },
    {
      "name": "vesting",
      "version": "1"
    }
  ]
}
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `upgrade`モジュールをクエリできます。

### Applied Plan

`AppliedPlan`は、以前に適用されたアップグレードプランをその名前で照会します。

```bash
cosmos.upgrade.v1beta1.Query/AppliedPlan
```

例:

```bash
grpcurl -plaintext \
    -d '{"name":"v2.0-upgrade"}' \
    localhost:9090 \
    cosmos.upgrade.v1beta1.Query/AppliedPlan
```

出力例:

```bash
{
  "height": "30"
}
```

### Current Plan

`CurrentPlan`は、現在のアップグレードプランを照会します。

```bash
cosmos.upgrade.v1beta1.Query/CurrentPlan
```

例:

```bash
grpcurl -plaintext localhost:9090 cosmos.slashing.v1beta1.Query/CurrentPlan
```

出力例:

```bash
{
  "plan": "v2.1-upgrade"
}
```

### Module versions

`ModuleVersions`は、状態からモジュールバージョンのリストを照会します。

```bash
cosmos.upgrade.v1beta1.Query/ModuleVersions
```

例:

```bash
grpcurl -plaintext localhost:9090 cosmos.slashing.v1beta1.Query/ModuleVersions
```

出力例:

```bash
{
  "module_versions": [
    {
      "name": "auth",
      "version": "2"
    },
    {
      "name": "authz",
      "version": "1"
    },
    {
      "name": "bank",
      "version": "2"
    },
    {
      "name": "capability",
      "version": "1"
    },
    {
      "name": "crisis",
      "version": "1"
    },
    {
      "name": "distribution",
      "version": "2"
    },
    {
      "name": "evidence",
      "version": "1"
    },
    {
      "name": "feegrant",
      "version": "1"
    },
    {
      "name": "genutil",
      "version": "1"
    },
    {
      "name": "gov",
      "version": "2"
    },
    {
      "name": "ibc",
      "version": "2"
    },
    {
      "name": "mint",
      "version": "1"
    },
    {
      "name": "params",
      "version": "1"
    },
    {
      "name": "slashing",
      "version": "2"
    },
    {
      "name": "staking",
      "version": "2"
    },
    {
      "name": "transfer",
      "version": "1"
    },
    {
      "name": "upgrade",
      "version": "1"
    },
    {
      "name": "vesting",
      "version": "1"
    }
  ]
}
```
