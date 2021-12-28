# `x/slashing`

## 概要

このセクションでは、2016年6月に[Cosmosホワイトペーパー](https://cosmos.network/about/whitepaper)で最初に概説された機能を実装するCosmosSDKのスラッシュモジュールを指定します。

スラッシュモジュールを使用すると、Cosmos SDKベースのブロックチェーンは、プロトコルで認識されたアクターによる、価値のあるアクターによる帰属アクションのインセンティブを、ペナルティ（「スラッシュ」）によって無効にすることができます。

ペナルティには以下が含まれる場合がありますが、これに限定されません:

- 彼らの株のいくらかの量を燃やす
- 一定期間、将来のブロックに投票する能力を削除します。

このモジュールは、Cosmosエコシステムの最初のハブであるCosmosハブによって使用されます。

## コンテンツ

1. **[Concepts](01_concepts.md)**
   - [States](01_concepts.md#states)
   - [Tombstone Caps](01_concepts.md#tombstone-caps)
   - [ASCII timelines](01_concepts.md#ascii-timelines)
2. **[State](02_state.md)**
   - [Signing Info](02_state.md#signing-info)
3. **[Messages](03_messages.md)**
   - [Unjail](03_messages.md#unjail)
4. **[Begin-Block](04_begin_block.md)**
   - [Evidence handling](04_begin_block.md#evidence-handling)
   - [Uptime tracking](04_begin_block.md#uptime-tracking)
5. **[05_hooks.md](05_hooks.md)**
   - [Hooks](05_hooks.md#hooks)
6. **[Events](06_events.md)**
   - [BeginBlocker](06_events.md#beginblocker)
   - [Handlers](06_events.md#handlers)
7. **[Staking Tombstone](07_tombstone.md)**
   - [Abstract](07_tombstone.md#abstract)
8. **[Parameters](08_params.md)**
9. **[Client](09_client.md)**
    - [CLI](09_client.md#cli)
    - [gRPC](09_client.md#grpc)
    - [REST](09_client.md#rest)
