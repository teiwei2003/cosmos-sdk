# ADR 041:オンサイトストレージ移行

## 変更ログ

-17.02.2021:最初のドラフト

## 状態

受け入れられました

## 概要

このADRは、チェーンソフトウェアのアップグレード中にインプレース状態のストレージ移行を実行するメカニズムを導入します。

## 環境

チェーンのアップグレードによってモジュール内の状態に破壊的な変更が導入された場合、現在のプロセスには、状態全体をJSONファイルにエクスポートし( `simd export`コマンドを使用)、JSONで移行スクリプト(` simdmigrate`コマンド)を実行することが含まれますファイルを作成し、ストレージをクリアして( `simd unsafe-reset-all`コマンド)、移行したJSONファイルを新しい作成として使用して(カスタムの初期ブロック高さを使用することを選択できます)、新しいチェーンを開始します。 [Cosmos Hub 3-> 4移行ガイド](https://github.com/cosmos/gaia/blob/v4.0.3/docs/migration/cosmoshub-3.md#Upgradeプログラム)。

このプロセスは、さまざまな理由で面倒です。

-手続きには時間がかかります。 `export`コマンドの実行には数時間かかる場合があり、移行されたJSONを使用して新しいチェーンで` InitChain`を実行する場合はさらに時間がかかる場合があります。
-エクスポートされたJSONファイルは重く(〜100MB-1GB)、表示、編集、転送が難しい場合があります。これにより、これらの問題を解決するための追加の作業が発生します(例:[ストリーミングジェネシス](https://github.com).cosmos .cosmos-sdk .issues .6936))。

## 決定

上記のJSONエクスポート-プロセス-インポートプロセスを使用せずに、KVストレージのその場での変更に基づく移行プロセスを提案します。

### モジュール `ConsensusVersion`

`AppModule`インターフェースに新しいメソッドを導入しました。 

```go
type AppModule interface {
   ..--snip--
    ConsensusVersion() uint64
}
```

このメソッドは、モジュールの状態破棄バージョンとしてuint64を返します。 モジュールによって導入されたコンセンサスを破る変更ごとにインクリメントする必要があります。 デフォルト値の潜在的なエラーを回避するには、モジュールの初期バージョンを1に設定する必要があります。 Cosmos SDKでは、バージョン1はv0.41シリーズのモジュールに対応しています。

### モジュール固有の移行機能

モジュールによって導入されたコンセンサスを破る変更ごとに、新しく追加された `RegisterMigration`メソッドを使用して、ConsensusVersion`N`からバージョン` N + 1`への移行スクリプトを `Configurator`に登録する必要があります。 すべてのモジュールは、移行機能を登録する必要がある[AppModule]の[RegisterServices]メソッドでコンフィギュレーターへの参照を受け取ります。 移行機能は昇順で登録する必要があります。  

```go
func (am AppModule) RegisterServices(cfg module.Configurator) {
   ..--snip--
    cfg.RegisterMigration(types.ModuleName, 1, func(ctx sdk.Context) error {
       ..Perform in-place store migrations from ConsensusVersion 1 to 2.
    })
     cfg.RegisterMigration(types.ModuleName, 2, func(ctx sdk.Context) error {
       ..Perform in-place store migrations from ConsensusVersion 2 to 3.
    })
   ..etc.
}
```

たとえば、モジュールの新しいConsensusVersionが `N`の場合、` N-1`移行機能をコンフィギュレータに登録する必要があります。

Cosmos SDKでは、キーパーがインプレースストレージ移行の実行に使用される `sdk.StoreKey`を保持しているため、移行機能は各モジュールのキーパーによって処理されます。 キーパーに過負荷をかけないように、各モジュールは `Migrator`ラッパーを使用して移行機能を処理します。 

```go
/.Migrator is a struct for handling in-place store migrations.
type Migrator struct {
  BaseKeeper
}
```

移行関数は、各モジュールの `migrations.`フォルダーに配置され、移行者のメソッドによって呼び出される必要があります。 メソッド名の形式は `Migrate {M} to {N}`にすることをお勧めします。 

```go
/.Migrate1to2 migrates from version 1 to 2.
func (m Migrator) Migrate1to2(ctx sdk.Context) error {
	return v043bank.MigrateStore(ctx, m.keeper.storeKey)..v043bank is package `x/bank/migrations/v043`.
}
```

各モジュールの移行機能は、モジュールのストレージの進化に固有であり、このADRでは説明されていません。 ADR-028の長さのプレフィックスアドレスの導入後のx .bankストアキーの移行の例は、この[store.goコード](https://github.com/cosmos/cosmos-sdk/blob/36f68eb9e041e20a5bb47e216ac5eb8b91f95471.x .bank .legacy .v043 .store.go#L41-L62)。

### `x .upgrade`でモジュールバージョンを追跡する

`x .upgrade`のストレージに新しいプレフィックスストレージを導入しました。 ストレージは各モジュールの現在のバージョンを追跡し、モジュール名からモジュールConsensusVersionへの `map [string] uint64`としてモデル化でき、移行の実行時に使用されます(詳細については次のセクションを参照してください)。 使用されるキープレフィックスは `0x1`であり、キー/値の形式は次のとおりです。 

```
0x2 | {bytes(module_name)} => BigEndian(module_consensus_version)
```

ストレージの初期状態は、 `app.go`の` InitChainer`メソッドによって設定されます。

UpgradeHandler署名を更新して、 `VersionMap`を取得し、アップグレードされた` VersionMap`とエラーを返す必要があります。

```diff
- type UpgradeHandler func(ctx sdk.Context, plan Plan)
+ type UpgradeHandler func(ctx sdk.Context, plan Plan, versionMap VersionMap) (VersionMap, error)
```

アップグレードを適用するには、 `x .upgrade`ストアから` VersionMap`を照会し、それをハンドラーに渡します。 ハンドラーは実際の移行関数を実行し(次のセクションを参照)、成功すると、状態に格納される更新された[VersionMap]を返します。

```diff
func (k UpgradeKeeper) ApplyUpgrade(ctx sdk.Context, plan types.Plan) {
   ..--snip--
-   handler(ctx, plan)
+   updatedVM, err := handler(ctx, plan, k.GetModuleVersionMap(ctx))..k.GetModuleVersionMap() fetches the VersionMap stored in state.
+   if err != nil {
+       return err
+   }
+
+  ..Set the updated consensus versions to state
+   k.SetModuleVersionMap(ctx, updatedVM)
}
```

gRPCクエリエンドポイントも追加され、 `x .upgrade`状態で保存されている` VersionMap`をクエリします。これにより、アプリケーション開発者は、アップグレードハンドラーを実行する前に `VersionMap`を再確認できます。

### 移行の実行

すべての移行ハンドラーがコンフィギュレーターに登録されると(起動時に発生します)、 `module.Manager`で` RunMigrations`メソッドを呼び出すことで移行を実行できます。この関数は、すべてのモジュールと各モジュールを繰り返し処理します。

-モジュールの古いConsensusVersionを `VersionMap`パラメーターから取得します(これを` M`と呼びます)。
-`AppModule`の `ConsensusVersion()`メソッドからモジュールの新しいConsensusVersion( `N`と呼びます)を取得します。
-`N> M`の場合、モジュール `M-> M + 1-> M + 2 ...`の登録されたすべての移行を `N`まで実行します。
    -モジュールにConsensusVersionがない特殊なケースがあります。これは、モジュールがアップグレードプロセス中に新たに追加されたことを意味するためです。この場合、移行機能は実行されず、モジュールの現在のConsensusVersionが `x .upgrade`のストレージに保存されます。

必要な移行が欠落している場合(たとえば、 `Configurator`に登録されていない場合)、` RunMigrations`関数はエラーを出します。

実際には、 `RunMigrations`メソッドは` UpgradeHandler`内から呼び出す必要があります。 

```go
app.UpgradeKeeper.SetUpgradeHandler("my-plan", func(ctx sdk.Context, plan upgradetypes.Plan, vm module.VersionMap)  (module.VersionMap, error) {
    return app.mm.RunMigrations(ctx, vm)
})
```

チェーンがブロックnでアップグレードされたとすると、プログラムは次のように実行されます。

-ブロック[N]を開始すると、古いバイナリファイルは[BeginBlock]で停止します。そのストレージには、古いバイナリモジュールのコンセンサスバージョンが保存されます。
-新しいバイナリファイルはブロック[N]から始まります。 UpgradeHandlerは新しいバイナリファイルに設定されているため、新しいバイナリファイルの[BeginBlock]で実行されます。 `x .upgrade`の` ApplyUpgrade`では、 `VersionMap`が(古いバイナリ)ストレージから取得され、` RunMigrations`関数に渡されて、モジュール自体の `BeginBlockの前にすべてのモジュールストレージ`が移行されます。

## 結果

### 下位互換性

このADRは、 `AppModule`に新しいメソッド` ConsensusVersion() `を導入し、すべてのモジュールがこのメソッドを実装する必要があります。また、UpgradeHandler関数のシグネチャも変更します。したがって、下位互換性はありません。

モジュールはConsensusVersionsと衝突するときに移行関数を登録する必要がありますが、アップグレードハンドラーを使用してこれらのスクリプトを実行することはオプションです。アプリケーションは、アップグレードハンドラーで `RunMigrations`を呼び出さず、代わりに古いJSON移行パスを引き続き使用することを決定する可能性があります。

### ポジティブ

-JSONファイルを操作せずにチェーンのアップグレードを実行します。
-ベースラインは確立されていませんが、インプレースストレージ移行はJSON移行よりも時間がかからない場合があります。このステートメントをサポートする主な理由は、古いバイナリファイルの `simdexport`コマンドと新しいバイナリファイルの` InitChain`関数がスキップされるためです。

### ネガティブ

-モジュール開発者は、コンセンサスを破るモジュールの変更を正しく追跡する必要があります。対応する `ConsensusVersion()`バンプなしでコンセンサスを破る変更がモジュールに導入された場合、 `RunMigrations`関数は移行を検出せず、チェーンのアップグレードが失敗する可能性があります。ドキュメントはこれを明確に反映している必要があります。

### ニュートラル

-Cosmos SDKは、既存の `simdexport`および` simdmigrate`コマンドによるJSON移行を引き続きサポートします。
-現在のADRでは、ストレージの作成、名前の変更、削除は許可されていません。変更できるのは、既存のストレージキーと値のみです。 Cosmos SDKは、これらの操作に `StoreLoader`を提供しています。
## さらなる議論

## 参照

-予備的なディスカッション:https://github.com/cosmos/cosmos-sdk/discussions/8429
-`ConsensusVersion`と `RunMigrations`の実装:https://github.com/cosmos/cosmos-sdk/pull/8485
-`x .upgrade`デザインの問題について話し合います:https://github.com/cosmos/cosmos-sdk/issues/8514 