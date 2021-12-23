# ストレージ

ストレージは、アプリケーションの状態を保存するデータ構造です。 {まとめ}

### 前提条件の読書

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## CosmosSDKストレージの紹介

Cosmos SDKには、アプリケーションの状態を維持するための多くのストレージが付属しています。 デフォルトでは、Cosmos SDKアプリケーションのメインストレージは「マルチストア」、つまりストレージストレージです。 開発者は、アプリケーションの要件に応じて、任意の数のKey-Valueストアをマルチストアに追加できます。 マルチストアは、Cosmos SDKのモジュール性をサポートするために存在します。これは、各モジュールが独自の状態のサブセットを宣言および管理できるようにするためです。 マルチストアのKey-Valueストアには、特定の機能 `key`を介してのみアクセスできます。これは通常、モジュールの[` keeper`](../building-modules/keeper.md)に格納されています。ストレージが宣言されています。 

```
+-----------------------------------------------------+
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 1 - Manage by keeper of Module 1  |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 2 - Manage by keeper of Module 2  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 3 - Manage by keeper of Module 2  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 4 - Manage by keeper of Module 3  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 5 - Manage by keeper of Module 4  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|                    Main Multistore                  |
|                                                     |
+-----------------------------------------------------+

                   Application's State
```

### ストレージインターフェイス

コアとなるCosmosSDKストアは、CacheWrapperを含み、GetStoreType()メソッドを持つオブジェクトです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L15-L18

`GetStoreType`はストレージタイプを返す単純なメソッドであり、` CacheWrapper`は単純なインターフェイスであり、 `Write`メソッドを介してストレージの読み取りキャッシュと書き込みブランチを実装します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L240-L264

分岐とキャッシュはCosmosSDKに遍在しており、各ストレージタイプに実装する必要があります。 ストレージブランチは、基盤となるメインストレージに影響を与えることなく転送および更新できる、分離された一時ストレージブランチを作成します。 これは、一時的な状態遷移をトリガーするために使用されます。エラーが発生した場合は、後で復元できます。 詳細については、[context](./context.md＃Store-branching)をご覧ください。

### ストレージを送信する

コミットストレージは、基になるツリーまたはデータベースに加えられた変更をコミットできるストレージです。 Cosmos SDKは、 `Committer`を使用して基本的なストレージインターフェイスを拡張することにより、シンプルなストレージと送信ストレージを区別します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L29-L33

`Committer`は、ディスクへの変更を永続化するメソッドを定義するインターフェイスです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L20-L27

`CommitID`は、状態ツリーの決定論的コミットです。そのハッシュ値は、基になるコンセンサスエンジンに返され、ブロックヘッダーに格納されます。送信ストレージインターフェイスには多くの用途があることに注意してください。その1つは、すべてのオブジェクトをストレージに送信できるわけではないことを確認することです。 Cosmos SDKの[object-capabilitiesモデル](./ocap.md)の一部として、 `baseapp`のみがストレージを送信できる必要があります。たとえば、モジュールがストレージへのアクセスに通常使用する `ctx.KVStore()`メソッドが、 `CommitKVStore`ではなく` KVStore`を返すのはこのためです。

Cosmos SDKはさまざまなストレージタイプを提供しますが、最も一般的に使用されるのは[`CommitMultiStore`](＃multistore)、[` KVStore`](＃kvstore)、[`GasKv`ストア](＃gaskv-store)です。 [Other-stores](＃other-stores)には、 `Transient`ストアと` TraceKV`ストアが含まれます。

## 複数のストレージ

### マルチストレージインターフェース

すべてのCosmosSDKアプリケーションには、その状態を維持するためにルートにマルチストアがあります。 multistoreは、 `Multistore`インターフェースに続く` KVStores`ストアです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L104-L133

トレースが有効になっている場合、ブランチマルチストアは最初に[`TraceKv.Store`](＃tracekv-store)内の基礎となるすべての` KVStore`をラップします。

### CommitMultiStore

CosmosSDKで使用される `Multistore`の主なタイプは、` Multistore`インターフェースの拡張である `CommitMultiStore`です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L141-L184

具体的な実装としては、[`rootMulti.Store`]が` CommitMultiStore`インターフェースの推奨実装です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/rootmulti/store.go#L43-L61

`rootMulti.Store`は` db`を中心に構築されたベースレイヤーマルチストレージであり、その上に複数の `KVStores`をインストールでき、[` baseapp`](./baseapp.md)で使用されるデフォルトのマルチストレージです。 。

### CacheMultiStore 

`rootMulti.Store`が分岐する必要があるときはいつでも、[` cachemulti.Store`](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/store/cachemulti/store.go)が使用されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/cachemulti/store.go#L17-L28

`cachemulti.Store`は、コンストラクター内のすべてのサブストアを分岐し(サブストアごとに仮想ストアを作成します)、それらを` Store.stores`に保存します。 さらに、すべての読み取りクエリがキャッシュされます。 `Store.GetKVStore()`は `Store.stores`からストアを返し、` Store.Write() `はすべての子ストアで` CacheWrap.Write() `を再帰的に呼び出します。

## 基本レイヤーKVStores

### `KVStore`および` CommitKVStore`インターフェース

`KVStore`は、データを保存および取得するための単純なKey-Valueストアです。 `CommitKVStore`は` Committer`も実装する `KVStore`です。デフォルトでは、 `baseapp`のメインの` CommitMultiStore`にインストールされているストアは `CommitKVStore`です。 `KVStore`インターフェースは、主にサブミッターへのモジュールアクセスを制限するために使用されます。

モジュールは、単一の「KVStore」を使用して、グローバル状態のサブセットを管理します。 `KVStores`には、特定のキーを保持しているオブジェクトからアクセスできます。この `key`は、ストレージを定義するモジュールの[` keeper`](../building-modules/keeper.md)にのみ公開する必要があります。

`CommitKVStore`は、それぞれの` key`エージェントによって宣言され、アプリケーションの[multistore](＃multistore)ファイルの[メインアプリケーションファイル](../basics/app-anatomy.md＃core-application)にインストールされます。 )。同じファイルで、`key`はストアの管理を担当するモジュールの`keeper`にも渡されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L189-L219

従来の `Get`メソッドと` Set`メソッドに加えて、 `KVStore`は` Iterator`オブジェクトを返す `Iterator(start、end)`メソッドを提供する必要があります。これは、一連のキー(通常は共通のプレフィックスを共有するキー)を反復処理するために使用されます。以下は、すべての口座残高を反復処理するために使用される銀行モジュールキーパーの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/x/bank/keeper/view.go#L115-L134

### `IAVL`ストレージ

`baseapp`で使用される` KVStore`と `CommitKVStore`のデフォルトの実装は` iavl.Store`です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/iavl/store.go#L37-L40

`iavl`ストレージは[IAVLツリー](https://github.com/tendermint/iavl)に基づいています。これは、以下を保証する自己平衡二分木です。

-`Get`および `Set`操作の複雑さはO(log n)です。ここで、nはツリー内の要素の数です。
-反復は、範囲内の並べ替えられた要素を効果的に返します。
-各ツリーバージョンは不変であり、送信後も取得できます(プルーニング設定によって異なります)。

IAVLツリーのドキュメントは[ここ](https://github.com/cosmos/iavl/blob/v0.15.0-rc5/docs/overview.md)にあります。

### `DbAdapter`ストレージ

`dbadapter.Store`は` dbm.DB`のアダプタであり、これにより `KVStore`インターフェースを実装できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/dbadapter/store.go#L13-L16

`dbadapter.Store`には` dbm.DB`が埋め込まれています。これは、 `KVStore`のほとんどのインターフェース機能が実装されていることを意味します。 その他の機能(主にその他)は手動で実装されます。 このストアは主に[一時的なストア](＃transient-stores)に使用されます

### `一時的な`ストレージ

`Transient.Store`はベースレイヤーの` KVStore`であり、ブロックの最後で自動的に破棄されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/transient/store.go#L13-L16

`Transient.Store`は` dbm.NewMemDB() `を持つ` dbadapter.Store`です。 すべての `KVStore`メソッドが再利用されます。 `Store.Commit()`を呼び出すと、新しい `dbadapter.Store`が割り当てられ、以前の参照が破棄され、ガベージコレクションが行われます。

このタイプのストレージは、各ブロックにのみ関連する情報を保持するのに役立ちます。 例として、パラメーターの変更を保存します(つまり、ブロック内のパラメーターが変更された場合は、boolを「true」に設定します)。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/x/params/types/subspace.go#L20-L30

一時ストレージは通常、[`context`](./context.md)を介して` TransientStore() `メソッドを介してアクセスされます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/types/context.go#L232-L235

## KVStoreラッパー

### CacheKVStore

`cachekv.Store`はラッパー` KVStore`であり、基礎となる `KVStore`にバッファ書き込み/キャッシュ読み取り関数を提供します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/cachekv/store.go#L27-L34

このタイプは、IAVLストレージを分岐して分離ストレージを作成する必要がある場合(通常、将来復元される可能性のある状態を変更する必要がある場合)に使用されます。

#### `入手`

`Store.Get()`は、最初に `Store.cache`にキーに関連付けられた値があるかどうかをチェックします。 値が存在する場合、関数はその値を返します。 そうでない場合、関数は `Store.parent.Get()`を呼び出し、結果を `Store.cache`にキャッシュして、それを返します。

#### `設定`

`Store.Set()`は、キーと値のペアを `Store.cache`に設定します。 `cValue`には、キャッシュされた値が基になる値と異なるかどうかを示すダーティブール値のフィールドがあります。 `Store.Set()`が新しいペアをキャッシュするとき、 `cValue.dirty`は` true`に設定されるので、 `Store.Write()`が呼び出されると、基礎となるストレージに書き込むことができます。

#### `イテレータ`

`Store.Iterator()`は、キャッシュアイテムと元のアイテムをトラバースする必要があります。 `Store.iterator()`では、イテレータごとに2つのイテレータが生成され、マージされます。 `memIterator`は本質的に` KVPairs`の一部であり、アイテムをキャッシュするために使用されます。 `mergeIterator`は2つのイテレータの組み合わせであり、トラバーサルは2つのイテレータに対して整然と実行されます。

### `GasKv`ストレージ

Cosmos SDKアプリケーションは、[`gas`](../basics/gas-fees.md)を使用して、リソースの使用状況を追跡し、スパムを防止します。 [`GasKv.Store`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/gaskv/store.go)は、毎回自動的に消費できる` KVStore`ラッパーです。各ガスがストレージに対して読み取りまたは書き込みを行います。これは、CosmosSDKアプリケーションでストレージの使用状況を追跡するための推奨ソリューションです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/gaskv/store.go#L13-L19

親の `KVStore`メソッドが呼び出されると、` GasKv.Store`は `Store.gasConfig`に従って適切な量のガスを自動的に消費します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/gas.go#L153-L162

デフォルトでは、すべての `KVStores`は取得時に` GasKv.Stores`に含まれます。これは、[`context`](./context.md)の` KVStore() `メソッドで行われます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/types/context.go#L227-L230

この場合、デフォルトのガス構成を使用します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/gas.go#L164-L175

### `TraceKv`ストレージ

`tracekv.Store`はラッパー` KVStore`であり、基礎となる `KVStore`で操作追跡機能を提供します。 親の「MultiStore」でトラッキングが有効になっている場合、CosmosSDKはそれをすべての「KVStore」に自動的に適用します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/tracekv/store.go#L20-L43

各 `KVStore`メソッドが呼び出されると、` tracekv.Store`は自動的に `traceOperation`を` Store.writer`に記録します。 `traceOperation.Metadata`は、ゼロでない場合は` Store.context`で埋められます。 `TraceContext`は` map [string] interface {} `です。

### `プレフィックス`ストレージ

`prefix.Store`はラッパー` KVStore`であり、基礎となる `KVStore`に自動キープレフィックス機能を提供します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/prefix/store.go#L15-L21

`Store。{Get、Set}()`を呼び出すと、ストアはその呼び出しを親に転送し、キーのプレフィックスは `Store.prefix`になります。

`Store.Iterator()`を呼び出す場合、期待どおりに機能しないため、単に `Store.prefix`のプレフィックスを付けることはありません。 この場合、一部の要素がプレフィックスで始まらない場合でも、それらはトラバースされます。

### `ListenKv`ストレージ

`listenkv.Store`はラッパー` KVStore`であり、基盤となる `KVStore`のステータス監視機能を提供します。
これは、Cosmos SDKによって、状態フローの構成中に「StoreKey」が指定されているすべての「KVStore」に自動的に適用されます。
状態ストリームの構成に関する追加情報は、[store/streaming/README.md](../../store/streaming/README.md)にあります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.44.1/store/listenkv/store.go#L11-L18

`KVStore.Set`または` KVStore.Delete`メソッドを呼び出すと、 `listenkv.Store`は自動的に操作を` Store.listeners`コレクションに書き込みます。

## 新しいストレージパッケージ( `store/v2`)

SDKは、状態ストレージのデフォルトのインターフェイスとしてここにリストされているタイプを使用するように移行しています。 執筆時点では、これらはアプリケーションで使用できず、CommitMultiStoreおよび関連するタイプと直接互換性がありません。。

### `BasicKVStore`インターフェース

基本的なCRUD関数( `Get`、` Set`、 `Has`、` Delete`メソッド)のインターフェースのみを提供し、反復やキャッシュは行いません。 これは、 `flat.Store`などのより大きなストレージコンポーネントを部分的に公開するために使用されます。

### タイル張り

`flat.Store`は、新しいデフォルトの永続ストレージであり、状態ストレージとコミットメントスキームの懸念を内部的に切り離します。 値はバッキングキー値データベース(「ストレージ」バケット)に直接格納され、値のハッシュは暗号化プロミスを生成できる別のストレージにマップされます(「状態プロミス」バケット、 `smt.storage`を使用) 。

これは、オプションで、バケットごとに異なるバックエンドデータベースを使用するように構築できます。

<!-- TODO: リンクを追加する +++ https://github.com/cosmos/cosmos-sdk/blob/v0.44.0/store/v2/flat/store.go -->

### SMTストレージ

基盤となるストレージの機能を部分的に公開するために使用される `BasicKVStore`(たとえば、` flat.Store`の約束されたストアへのアクセスを許可する)。

## 次へ{非表示}

[encoding](./encoding.md){hide}を理解する
