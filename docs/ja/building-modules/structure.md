# 推奨されるフォルダ構造

このドキュメントでは、CosmosSDKモジュールの推奨される構造の概​​要を説明します。 これらのアイデアは、提案として適用することを目的としています。 アプリケーション開発者に、モジュール構造と開発設計を改善し、貢献するように促します。 {まとめ}

## 構造

典型的なCosmosSDKモジュール構造は次のとおりです。 

```shell
proto
└── {project_name}
    └── {module_name}
        └── {proto_version}
            ├── {module_name}.proto
            ├── event.proto
            ├── genesis.proto
            ├── query.proto
            └── tx.proto
```

-` {module_name} .proto`:モジュールの一般的なメッセージタイプの定義。
-`event.proto`:イベントに関連するモジュールメッセージタイプの定義。
-`genesis.proto`:作成状態に関連するモジュールメッセージタイプの定義。
-`query.proto`:モジュールのクエリサービスと関連するメッセージタイプの定義。
-`tx.proto`:モジュールのメッセージサービスと関連するメッセージタイプの定義。 

```shell
x/{module_name}
├── client
│   ├── cli
│   │   ├── query.go
│   │   └── tx.go
│   └── testutil
│       ├── cli_test.go
│       └── suite.go
├── exported
│   └── exported.go
├── keeper
│   ├── genesis.go
│   ├── grpc_query.go
│   ├── hooks.go
│   ├── invariants.go
│   ├── keeper.go
│   ├── keys.go
│   ├── msg_server.go
│   └── querier.go
├── module
│   └── module.go
├── simulation
│   ├── decoder.go
│   ├── genesis.go
│   ├── operations.go
│   └── params.go
├── spec
│   ├── 01_concepts.md
│   ├── 02_state.md
│   ├── 03_messages.md
│   └── 04_events.md
├── {module_name}.pb.go
├── abci.go
├── codec.go
├── errors.go
├── events.go
├── events.pb.go
├── expected_keepers.go
├── genesis.go
├── genesis.pb.go
├── keys.go
├── msgs.go
├── params.go
├── query.pb.go
└── tx.pb.go
```

-`client/`:モジュールのCLIクライアント関数の実装とモジュールの統合テストスイート。
-`exported/`:モジュールのエクスポートされたタイプ-通常はインターフェースタイプ。モジュールが別のモジュールのキーパーに依存している場合、キーパーを実装するモジュールに直接依存することを避けるために、 `expected_keepers.go`ファイル(以下を参照)を介してインターフェースコントラクトとしてキーパーを受け取ることが期待されます。ただし、これらのインターフェイスコントラクトは、Keepersを実装する特定のタイプのモジュールを返す操作やメソッドを定義できます。ここで、[exported/]が機能します。 `exported/`で定義されたインターフェースタイプは標準タイプを使用し、モジュールが `expected_keepers.go`ファイルを介してインターフェースコントラクトとしてKeeperを受信できるようにします。このモードでは、コードをDRYのままにして、インポートサイクルの混乱を緩和できます。
-`keeper/`:モジュールの` Keeper`と `MsgServer`の実装。
-`module/`:モジュールの` AppModule`と `AppModuleBasic`の実装。
-`simulation/`:モジュールの[simulation](./simulator.html)パッケージは、ブロックチェーンシミュレーターアプリケーション(` simapp`)によって使用される関数を定義します。
-`spec/`:モジュールの仕様書には、重要な概念、状態ストレージ構造、メッセージおよびイベントタイプの定義の概要が記載されています。
-ルートディレクトリには、メッセージ、イベント、および作成ステータスのタイプ定義(プロトコルバッファ生成のタイプ定義を含む)が含まれます。
    -`abci.go`:モジュールの `BeginBlocker`および` EndBlocker`の実装(このファイルは、 `BeginBlocker`および/または` EndBlocker`を定義する必要がある場合にのみ必要です)。
    -`codec.go`:モジュールのインターフェースタイプ登録方法。
    -`errors.go`:モジュールのセンチネルエラー。
    -`events.go`:モジュールのイベントタイプとコンストラクター。
    -`expected_keepers.go`:モジュールの[expected keeper](./keeper.html#type-definition)インターフェース。
    -`genesis.go`:モジュールのジェネシス状態メソッドと補助関数。
    -`keys.go`:モジュールストレージキーと関連する補助機能。
    -`msgs.go`:モジュールのメッセージタイプ定義と関連メソッド。
    -`params.go`:モジュールパラメータタイプの定義と関連メソッド。
    -` * .pb.go`:プロトコルバッファによって生成されたモジュールタイプの定義(上記の対応する `* .proto`ファイルで定義されています)。

## 次へ{hide}

[errors](./errors.md){hide}を理解する 