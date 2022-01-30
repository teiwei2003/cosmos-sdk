# 環境

`context`は、ある関数から別の関数に渡されるように設計されたデータ構造であり、アプリケーションの現在の状態に関する情報を伝達します。 ブランチストレージ(状態全体の安全なブランチ)へのアクセスと、[gasMeter]、[ブロックの高さ]、[コンセンサスパラメータ]などの有用なオブジェクトと情報へのアクセスを提供します。 {まとめ}

## 前提条件の読書

- [CosmosSDKアプリケーションの構造](../basics/app-anatomy.md) {prereq}
- [トランザクションのライフサイクル](../basics/tx-lifecycle.md) {prereq}

## コンテキスト定義

Cosmos SDK `Context`は、Goのstdlib [` context`](https://golang.org/pkg/context)をベースとして含むカスタムデータ構造であり、その定義には多くのユニバース固有のSDKが含まれています。 `Context`は、モジュールがそれぞれの[store](./store.md#base-layer-kvstores in [` multistore`](./store.md#multistore))に簡単にアクセスできるようにするため、トランザクション処理の不可欠な部分です。 )そして、ブロックヘッダーやガスメーターなどのトランザクションコンテキストを取得します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/context.go#L16-L39

-**コンテキスト:**基本的なタイプはGo [Context](https://golang.org/pkg/context)です。さらに、[Go Context Package](#go-context-package)セクションで次のことを説明します。
-**マルチストア:**各アプリケーションの `BaseApp`には、`コンテキスト `の作成時に提供される[` CommitMultiStore`](./store.md#multistore)が含まれています。 `KVStore()`および `TransientStore()`メソッドを呼び出すと、モジュールは独自の `StoreKey`を使用して、それぞれの[` KVStore`](./store.md#base-layer-kvstores)を取得できます。
-**ABCIヘッダー:** [ヘッダー](https://tendermint.com/docs/spec/abci/abci.html#header)はABCIタイプです。現在のブロックの高さや提案者など、ブロックチェーンの状態に関する重要な情報が含まれています。
-**チェーンID:**ブロックが属するブロックチェーンの一意の識別番号。
-**Transaction Bytes:**コンテキストを使用して処理されているトランザクションの `[] byte`表現。各トランザクションは、Cosmos SDKのさまざまな部分とコンセンサスエンジン(Tendermintなど)によって[ライフサイクル](../basics/tx-lifecycle.md)全体で処理され、トランザクションの種類に関する知識がないものもあります。したがって、トランザクションは、ユニバーサル[[] byte]タイプにグループ化された[Amino](./encoding.md)などの特定の[encoding format](./encoding.md)を使用します。
-**ロガー:** Tendermintライブラリの `logger`。ログの詳細については、[こちら](https://tendermint.com/docs/tendermint-core/how-to-read-logs.html#how-to-read-logs)をご覧ください。モジュールはこのメソッドを呼び出して、独自のモジュール固有のロガーを作成します。
-**VoteInfo:** ABCIタイプリスト[`VoteInfo`](https://tendermint.com/docs/spec/abci/abci.html#voteinfo)には、バリデーターの名前と、それらがブロックに署名されています。
-**ガスメーター:**具体的には、[`gasMeter`](../basics/gas-fees.md#main-gas-meter)は、コンテキストと[` blockGasMeterを使用して現在処理されているトランザクションに使用されます`](../basics/gas-fees.md#block-gas-meter)は、それが属するブロック全体に使用されます。ユーザーは、トランザクションの実行に支払う金額を指定します。これらのガスメーターは、これまでにトランザクションまたはブロックで使用された[gas](../basics/gas-fees.md)の量を追跡します。ガスメーターがなくなると、実行が停止します。
-**CheckTxモード:**トランザクションを[CheckTx]モードと[DeliverTx]モードのどちらで処理するかを示すブール値。
-**最小ガス価格:**ノードがそのブロックにトランザクションを含めるために受け入れることをいとわない最低の[gas](../basics/gas-fees.md)価格。この価格は、各ノードによって個別に構成されたローカル値であるため、**状態遷移を引き起こすシーケンスで使用される関数では使用しないでください**。
-**コンセンサスパラメータ:** ABCIタイプ[コンセンサスパラメータ](https://tendermint.com/docs/spec/abci/apps.html#consensus-parameters)。これは、最大値など、ブロックチェーンの特定の制限を指定します。値ガスのブロック。
-**イベントマネージャー:**イベントマネージャーを使用すると、 `Context`にアクセスできるすべての呼び出し元が[` Events`](./events.md)を発行できます。モジュールはモジュール固有を定義できます
  `Events`は、さまざまな` Types`と `Attributes`を定義するか、` types/`にある共通の定義を使用します。顧客は、これらの[イベント]をサブスクライブまたはクエリできます。これらの[イベント]は、[DeliverTx]、[BeginBlock]、および[EndBlock]全体で収集され、インデックス作成のためにTendermintに返されます。例えば:
  
```go
ctx.EventManager().EmitEvent(sdk.NewEvent(
    sdk.EventTypeMessage,
    sdk.NewAttribute(sdk.AttributeKeyModule, types.AttributeValueCategory)),
)
```

## コンテキストパッケージに移動

基本的な `Context`は、[Golang Context Package](https://golang.org/pkg/context)で定義されています。 `コンテキスト`
これは、APIとプロセスの間で要求されたデータを運ぶ不変のデータ構造です。 環境
また、並行性を有効にし、ゴルーチンで使用できるように設計されています。

コンテキストは**不変**であることが意図されており、編集しないでください。 代わりに、コンベンションは
[With]関数を使用して、親から子コンテキストを作成します。 例えば: 

```go
childCtx = parentCtx.WithBlockHeader(header)
```

[Golangコンテキストパッケージ](https://golang.org/pkg/context)ドキュメントガイド開発者
プロセスの最初のパラメータとしてコンテキスト `ctx`を明示的に渡します。

## ストレージブランチ

`Context`には` MultiStore`が含まれており、 `CacheMultiStore`を使用して関数を分岐およびキャッシュできます。
( `CacheMultiStore`のクエリは、将来のラウンドトリップを回避するためにキャッシュされます)。
各[KVStore]は、安全で隔離された一時的なストアに分岐します。プロセスは自由に変更を書き込むことができます
`CacheMultiStore`。状態遷移シーケンスが問題なく実行された場合、ストレージブランチは次のようになります。
シーケンスの最後に基になるストレージに送信するか、特定の状況が発生したときにそれらを無視します
何かがうまくいかなかった。 Contextの使用パターンは次のとおりです。

1. プロセスは、必要な情報を提供する親プロセスからコンテキスト `ctx`を受け取ります
   実装プロセス。
2. `ctx.ms`は**ブランチストア**、つまり[multistore](./store.md#multistore)のブランチであるため、プロセスは元の`を変更せずに実行中に状態を変更できます。 ctx.ms`。これは、実行中のある時点で変更を復元する必要がある場合に、基盤となるマルチストアを保護するのに役立ちます。
3. プロセスは、実行中に `ctx`から読み取りおよび書き込みを行う場合があります。子プロセスを呼び出してパスする場合があります
   必要に応じて `ctx`が追加されます。
4. 子プロセスが戻ると、結果が成功か失敗かをチェックします。それが失敗した場合、何もありません
   実行する必要があります-ブランチ `ctx`は単に破棄されます。成功した場合、行われた変更
   `CacheMultiStore`は、` Write() `を介して元の` ctx.ms`に送信できます。

たとえば、関数[`runTx`](./baseapp.md#runtx-and-runmsgs)のスニペットを次に示します。
[`baseapp`](./baseapp.md): 

```go
runMsgCtx, msCache := app.cacheTxContext(ctx, txBytes)
result = app.runMsgs(runMsgCtx, msgs, mode)
result.GasWanted = gasWanted

if mode != runTxModeDeliver {
  return result
}

if result.IsOK() {
  msCache.Write()
}
```

これがプロセスです:

1. トランザクション内のメッセージで `runMsgs`を呼び出す前に、` app.cacheTxContext() `を使用します
     ブランチとキャッシュのコンテキストと複数のストレージ。
2. `runMsgCtx`-ブランチストレージのコンテキスト。`runMsgs`で結果を返すために使用されます。
3. プロセスが[`checkTxMode`](./baseapp.md#checktx)で実行されている場合は、次のように記述する必要はありません。
     変更-結果はすぐに返されます。
4. プロセスが[`deliverTxMode`](./baseapp.md#delivertx)で実行されている場合、結果が表示されます
     すべてのメッセージが正常に実行された後、ブランチマルチストアは元のメッセージに書き戻されます。

## 次へ{hide}

[ノードクライアント](./node.md)を理解する{hide}