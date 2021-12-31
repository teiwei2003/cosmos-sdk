# クライアント

## CLI

ユーザーは、CLIを使用して `mint`モジュールを照会および操作できます。

### Query

`query`コマンドを使用すると、ユーザーは`mint`状態を照会できます。

```
simd query mint --help
```

#### annual-provisions

`annual-provisions`コマンドを使用すると、ユーザーは現在の造幣局の年間引当金の値を照会できます。

```
simd query mint annual-provisions [flags]
```

例:

```
simd query mint annual-provisions
```

出力例:

```
22268504368893.612100895088410693
```

#### inflation

`inflation`コマンドを使用すると、ユーザーは現在のミンティングインフレ値を照会できます。

```
simd query mint inflation [flags]
```

例:

```
simd query mint inflation
```

出力例:

```
0.199200302563256955
```

#### params

`params`コマンドを使用すると、ユーザーは現在のミンティングパラメータを照会できます。

```
simd query mint params [flags]
```

例:

```
blocks_per_year: "4360000"
goal_bonded: "0.670000000000000000"
inflation_max: "0.200000000000000000"
inflation_min: "0.070000000000000000"
inflation_rate_change: "0.130000000000000000"
mint_denom: stake
```

## gRPC

ユーザーは、gRPCエンドポイントを使用して `mint`モジュールにクエリを実行できます。

### AnnualProvisions

`AnnualProvisions`エンドポイントを使用すると、ユーザーは現在の造幣局の年間引当金の値を照会できます。

```
/cosmos.mint.v1beta1.Query/AnnualProvisions
```

例:

```
grpcurl -plaintext localhost:9090 cosmos.mint.v1beta1.Query/AnnualProvisions
```

出力例:

```
{
  "annualProvisions": "1432452520532626265712995618"
}
```

### Inflation

`Inflation`エンドポイントにより、ユーザーは現在のミンティングインフレ値を照会できます

```
/cosmos.mint.v1beta1.Query/Inflation
```

例:

```
grpcurl -plaintext localhost:9090 cosmos.mint.v1beta1.Query/Inflation
```

出力例:

```
{
  "inflation": "130197115720711261"
}
```

### Params

`Params`エンドポイントにより、ユーザーは現在のミンティングパラメータをクエリできます

```
/cosmos.mint.v1beta1.Query/Params
```

例:

```
grpcurl -plaintext localhost:9090 cosmos.mint.v1beta1.Query/Params
```

出力例:

```
{
  "params": {
    "mintDenom": "stake",
    "inflationRateChange": "130000000000000000",
    "inflationMax": "200000000000000000",
    "inflationMin": "70000000000000000",
    "goalBonded": "670000000000000000",
    "blocksPerYear": "6311520"
  }
}
```

## REST

ユーザーは、RESTエンドポイントを使用して `mint`モジュールにクエリを実行できます。

### annual-provisions

```
/cosmos/mint/v1beta1/annual_provisions
```

例:

```
curl "localhost:1317/cosmos/mint/v1beta1/annual_provisions"
```

出力例:

```
{
  "annualProvisions": "1432452520532626265712995618"
}
```

### inflation

```
/cosmos/mint/v1beta1/inflation
```

例:

```
curl "localhost:1317/cosmos/mint/v1beta1/inflation"
```

出力例:

```
{
  "inflation": "130197115720711261"
}
```

### params

```
/cosmos/mint/v1beta1/params
```

例:

```
curl "localhost:1317/cosmos/mint/v1beta1/params"
```

出力例:

```
{
  "params": {
    "mintDenom": "stake",
    "inflationRateChange": "130000000000000000",
    "inflationMax": "200000000000000000",
    "inflationMin": "70000000000000000",
    "goalBonded": "670000000000000000",
    "blocksPerYear": "6311520"
  }
}
```
