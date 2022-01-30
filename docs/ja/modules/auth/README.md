# `認証`

## 概要

このドキュメントでは、CosmosSDKの認証モジュールを指定します。

authモジュールは、基本的なトランザクションとアカウントタイプを指定する役割を果たします
アプリケーションの場合、SDK自体はこれらの詳細とは何の関係もないためです。 を含む
すべての基本的なトランザクションの有効性チェック(署名、乱数、補助フィールド)が行われるアンティハンドラー
アカウントマネージャーを実行して公開し、他のモジュールがアカウントの読み取り、書き込み、変更を行えるようにします。

このモジュールは、Cosmosハブで使用されます。

## コンテンツ

1。**[コンセプト](01_concepts.md)**
   - [ガスと料金](01_concepts.md#gas-＆-料金)
2。**[State](02_state.md)**
   - [アカウント](02_state.md#accounts)
3。**[AnteHandlers](03_antehandlers.md)**
   - [ハンドラー](03_antehandlers.md#handlers)
4。**[キーパー](04_keepers.md)**
   - [アカウント管理者](04_keepers.md#account-keeper)
5。**[帰属](05_vesting.md)**
   - [はじめにと要件](05_vesting.md#intro-and-requirements)
   - [アトリビューションアカウントタイプ](05_vesting.md#vesting-account-types)
   - [権利確定アカウントの仕様](05_vesting.md#vesting-account-specification)
   - [キーパーとハンドラー](05_vesting.md#keepers-＆-ハンドラー)
   - [ジェネシス初期化](05_vesting.md#genesis-初期化)
   - [例](05_vesting.md#examples)
   - [語彙](05_vesting.md#glossary)
6。**[パラメータ](06_params.md)**
7。**[クライアント](07_client.md)**
   - **[認証](07_client.md#auth)**
      - [CLI](07_client.md#cli)
      - [gRPC](07_client.md#grpc)
      - [REST](07_client.md#rest)
   - **[帰属](07_client.md#vesting)**
      - [CLI](07_client.md#vesting#cli)