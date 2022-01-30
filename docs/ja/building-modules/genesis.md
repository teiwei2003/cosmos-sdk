# モジュールジェネシス

モジュールは通常、状態の​​サブセットを処理するため、ジェネシスファイルの関連するサブセットと、それを初期化、検証、およびエクスポートするためのメソッドを定義する必要があります。 {まとめ}

## 読むための前提条件

-[モジュールマネージャー](。/module-manager.md){前提条件}
-[キーパー](。/keeper.md){前提条件}

## タイプ定義

特定のモジュールから定義されたジェネシス状態のサブセットは、通常、 `genesis.proto`ファイルで定義されます(protobufメッセージの定義方法に関する[詳細情報](../core/encoding.md#gogoproto))。モジュールのジェネシス状態のサブセットを定義する構造は、通常[GenesisState]と呼ばれ、作成プロセス中に初期化する必要があるすべてのモジュール関連の値が含まれています。

`auth`モジュールからの` GenesisState`protobufメッセージの定義の例を表示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/a9547b54ffac9729fe1393651126ddfc0d236cff/proto/cosmos/auth/v1beta1/genesis.proto

次に、モジュールをCosmos SDKアプリケーションで使用できるようにするために、モジュール開発者が実装する必要のある主な作成関連のメソッドを紹介します。

### `DefaultGenesis`

`DefaultGenesis()`メソッドは、各パラメーターのデフォルト値を使用して `GenesisState`コンストラクターを呼び出す単純なメソッドです。 `auth`モジュールの例を確認してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/module.go#L48-L52

### `ValidateGenesis`

`ValidateGenesis(genesisState GenesisState)`メソッドを呼び出して、提供された `genesisState`が正しいことを確認します。 [GenesisState]にリストされている各パラメーターの妥当性チェックを実行する必要があります。 `auth`モジュールの例を確認してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/types/genesis.go#L57-L70

## その他の作成方法

`GenesisState`に直接関連するメソッドに加えて、モジュール開発者は[` AppModuleGenesis`インターフェース](./module-manager.md#appmodulegenesis)の一部として他の2つのメソッドを実装する必要があります(モジュールがジェネシスを初期化する必要がある場合のみ)状態のサブセット)。これらのメソッドは、[`InitGenesis`](#initgenesis)および[` ExportGenesis`](#exportgenesis)です。

### `InitGenesis`

`InitGenesis`メソッドは、アプリケーションが最初に起動されたときの[` InitChain`](../core/baseapp.md#initchain)中に実行されます。 `GenesisState`を指定すると、` GenesisState`の各パラメーターに対してモジュールの[`keeper`](./keeper.md)セッター関数を使用して、モジュールによって管理される状態のサブセットを初期化します。

アプリケーションの[modulemanager](./module-manager.md#manager)は、アプリケーションの各モジュールの `InitGenesis`メソッドを順番に呼び出す役割を果たします。この順序は、[アプリケーションのコンストラクター](../basics/app-anatomy.md#constructor-function)で呼び出されるマネージャーの `SetOrderGenesisMethod`を介してアプリケーション開発者によって設定されます。

[auth]モジュールの[InitGenesis]の例を参照してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/genesis.go#L13-L28

### `ExportGenesis`

状態がエクスポートされるたびに、[ExportGenesis]メソッドが実行されます。モジュールによって管理される状態サブセットの最新の既知のバージョンを取得し、そこから新しい[GenesisState]を作成します。これは主に、チェーンをハードフォークでアップグレードする必要がある場合に使用されます。

[auth]モジュールの[ExportGenesis]の例を参照してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/64b6bb5270e1a3b688c2d98a8f481ae04bb713ca/x/auth/genesis.go#L31-L42

## 次へ{hide}

[module interface](module-interfaces.md){hide}を理解する 