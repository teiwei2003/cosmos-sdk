# アップグレードモジュール

[インプレースストレージ移行](../core/upgrade.html)を使用すると、モジュールを新しいバージョンにアップグレードして、大幅な変更を加えることができます。このドキュメントでは、この機能を利用するためのモジュールを構築する方法の概要を説明します。 {まとめ}

## 読むための前提条件

-[インプレースストレージ移行](../core/upgrade.md){前提条件}

## コンセンサスバージョン

既存のモジュールを正常にアップグレードするには、各 `AppModule`が関数` ConsensusVersion()uint64`を実装する必要があります。

-バージョンは、モジュール開発者がハードコーディングする必要があります。
-初期バージョンは** 1に設定する必要があります**。

コンセンサスバージョンは、アプリケーションモジュールの状態破壊バージョンとして使用され、モジュールが破壊的な変更を導入するときにインクリメントする必要があります。

## 移行にサインアップ

モジュールのアップグレード中に発生する機能を登録するには、発生する移行を登録する必要があります。

移行登録は、 `RegisterMigration`メソッドを使用して` Configurator`で実行されます。 configuratorへの[AppModule]参照は[RegisterServices]メソッドにあります。

1つ以上の移行を登録できます。複数の移行スクリプトを登録している場合は、移行を昇順でリストし、必要なコンセンサスバージョンを生成するのに十分な移行があることを確認してください。たとえば、モジュールのバージョン3に移行するには、次の例に示すように、バージョン1とバージョン2に別々の移行を登録します。 

```golang
func (am AppModule) RegisterServices(cfg module.Configurator) {
  //--snip--
    cfg.RegisterMigration(types.ModuleName, 1, func(ctx sdk.Context) error {
      //Perform in-place store migrations from ConsensusVersion 1 to 2.
    })
     cfg.RegisterMigration(types.ModuleName, 2, func(ctx sdk.Context) error {
      //Perform in-place store migrations from ConsensusVersion 2 to 3.
    })
}
```

これらの移行はKeeperストレージへのアクセスを必要とする関数であるため、次の例に示すように、[Migrator]と呼ばれるKeeperラッパーを使用します。

+++ https://github.com/cosmos/cosmos-sdk/blob/6ac8898fec9bd7ea2c1e5c79e0ed0c3f827beb55/x/bank/keeper/migrations.go#L8-L21

##移行スクリプトを作成する

アップグレード中に発生する関数を定義するには、移行スクリプトを作成し、これらの関数を `migrations/`ディレクトリに配置します。 たとえば、bankモジュールの移行スクリプトを作成するには、関数を `x/bank/migrations/`に配置します。 これらの関数には、推奨される命名規則を使用してください。 たとえば、 `v043bank`は移行パッケージ` x/bank/migrations/v043`のスクリプトです。

```golang
//Migrating bank module from version 1 to 2
func (m Migrator) Migrate1to2(ctx sdk.Context) error {
	return v043bank.MigrateStore(ctx, m.keeper.storeKey)//v043bank is package `x/bank/migrations/v043`.
}
```

バランスキーの移行で実装された変更のサンプルコードを確認するには、[migrateBalanceKeys](https://github.com/cosmos/cosmos-sdk/blob/36f68eb9e041e20a5bb47e216ac5eb8b91f95471/x/bank/legacy/v043/store)を確認してください。 #L41-L62)。 コンテキストとして、このコードは、[ADR-028](../architecture/adr-028-public-key-addresses.md)のように、前にバイト長が付いたアドレスを更新する銀行ストレージの移行を導入します。 