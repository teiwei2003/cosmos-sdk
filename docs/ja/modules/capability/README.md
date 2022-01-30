# `capability`

## 概述

`x/capability`は、[ADR 003](./../../../docs/architecture/adr-003-dynamic-capability-store.md)によるCosmosSDKモジュールの実装です。
これにより、複数所有者の機能のプロビジョニング、追跡、および認証が可能になります
実行時。

Keeperは、永続メモリと短命メモリの2つの状態を維持します。持続的
ストアは、グローバルに一意の自動インクリメントインデックスとマッピングを維持します
モジュールの機能所有者として定義された機能インデックスのセットと
機能名タプル。メモリ内の一時的な状態は、実際の状態を追跡します
順方向および逆方向のインデックスを使用して、ローカルメモリ内のアドレスとして表される機能。
フォワードインデックスは、モジュール名と機能タプルを機能名にマップします。この
モジュール名と関数名、および関数自体の間の逆インデックスマッピング。

Keeperを使用すると、特定のサブマネージャーに関連付けられた[スコープ]サブマネージャーを作成できます。
モジュール名。アプリケーションで初期化する必要があり、
モジュールに渡されると、モジュールはそれらを使用して、受信し、
新しい機能を作成するだけでなく、名前で機能を取得することもできます
＆他のモジュールから渡された機能を確認します。スコープを持つ保護者は、そのスコープから逃れることはできません。
したがって、1つのモジュールが他のモジュールが所有する機能を妨害したりチェックしたりすることはできません。

Keeperは、他のモジュールにある他のコア機能を提供しません
クエリャー、RESTおよびCLIハンドラー、および作成ステータスと同様です。

## 初期化

アプリケーションの初期化中に、Keeperを永続的にインスタンス化する必要があります
ストレージキーとメモリストレージキー。

```go
type App struct {
 //...

  capabilityKeeper *capability.Keeper
}

func NewApp(...) *App {
 //...

  app.capabilityKeeper = capability.NewKeeper(codec, persistentStoreKey, memStoreKey)
}
```

Keeperが作成された後、それを使用してスコープ付きサブKeeperを作成できます。
機能を作成、認証、および要求できる他のモジュールに渡されます。
必要なスコープKeeperがすべて作成され、状態が読み込まれた後、
インメモリにデータを入力するには、メイン機能Keeperを初期化して封印する必要があります
状態を設定し、スコープがさらに設定されたKeeperが作成されないようにします。

```go
func NewApp(...) *App {
 //...

 //Initialize and seal the capability keeper so all persistent capabilities
 //are loaded in-memory and prevent any further modules from creating scoped
 //sub-keepers.
  ctx := app.BaseApp.NewContext(true, tmproto.Header{})
  app.capabilityKeeper.InitializeAndSeal(ctx)

  return app
}
```

## Contents

1. **[Concepts](01_concepts.md)**
2. **[State](02_state.md)**
