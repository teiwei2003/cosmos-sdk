# 環境

`context`は、ある関数から別の関数に渡されるように設計されたデータ構造であり、アプリケーションの現在の状態に関する情報を伝達します。 ブランチストレージ(状態全体の安全なブランチ)へのアクセスと、「gasMeter」、「ブロックの高さ」、「コンセンサスパラメータ」などの有用なオブジェクトと情報へのアクセスを提供します。 {まとめ}

## 前提条件の読書

- [CosmosSDKアプリケーションの構造](../basics/app-anatomy.md) {prereq}
- [トランザクションのライフサイクル](../basics/tx-lifecycle.md) {prereq}

## コンテキスト定義

Cosmos SDK `Context`は、Goのstdlib [` context`](https://golang.org/pkg/context)をベースとして含むカスタムデータ構造であり、その定義には多くのユニバース固有のSDKが含まれています。 `Context`は、モジュールがそれぞれの[store](./store.md#base-layer-kvstores in [` multistore`](./store.md#multistore))に簡単にアクセスできるようにするため、トランザクション処理の不可欠な部分です。 )そして、ブロックヘッダーやガスメーターなどのトランザクションコンテキストを取得します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/context.go#L16-L39

-**コンテキスト:**基本的なタイプはGo [Context](https://golang.org/pkg/context)です。さらに、[Go Context Package](#go-context-package)セクションで次のことを説明します。
-**マルチストア:**各アプリケーションの `BaseApp`には、`コンテキスト `の作成時に提供される[` CommitMultiStore`](./store.md#multistore)が含まれています。 `KVStore()`および `TransientStore()`メソッドを呼び出すと、モジュールは独自の `StoreKey`を使用して、それぞれの[` KVStore`](./store.md#base-layer-kvstores)を取得できます。
-** ABCIヘッダー:** [ヘッダー](https://tendermint.com/docs/spec/abci/abci.html#header)はABCIタイプです。現在のブロックの高さや提案者など、ブロックチェーンの状態に関する重要な情報が含まれています。
-**チェーンID:**ブロックが属するブロックチェーンの一意の識別番号。
-**Transaction Bytes:**コンテキストを使用して処理されているトランザクションの `[] byte`表現。各トランザクションは、Cosmos SDKのさまざまな部分とコンセンサスエンジン(Tendermintなど)によって[ライフサイクル](../basics/tx-lifecycle.md)全体で処理され、トランザクションの種類に関する知識がないものもあります。したがって、トランザクションは、ユニバーサル「[] byte」タイプにグループ化された[Amino](./encoding.md)などの特定の[encoding format](./encoding.md)を使用します。
-**ロガー:** Tendermintライブラリの `logger`。ログの詳細については、[こちら](https://tendermint.com/docs/tendermint-core/how-to-read-logs.html#how-to-read-logs)をご覧ください。モジュールはこのメソッドを呼び出して、独自のモジュール固有のロガーを作成します。
-** VoteInfo:** ABCIタイプリスト[`VoteInfo`](https://tendermint.com/docs/spec/abci/abci.html#voteinfo)には、バリデーターの名前と、それらがブロックに署名されています。
-**ガスメーター:**具体的には、[`gasMeter`](../basics/gas-fees.md#main-gas-meter)は、コンテキストと[` blockGasMeterを使用して現在処理されているトランザクションに使用されます`](../basics/gas-fees.md#block-gas-meter)は、それが属するブロック全体に使用されます。ユーザーは、トランザクションの実行に支払う金額を指定します。これらのガスメーターは、これまでにトランザクションまたはブロックで使用された[gas](../basics/gas-fees.md)の量を追跡します。ガスメーターがなくなると、実行が停止します。
-**CheckTxモード:**トランザクションを「CheckTx」モードと「DeliverTx」モードのどちらで処理するかを示すブール値。
-**最小ガス価格:**ノードがそのブロックにトランザクションを含めるために受け入れることをいとわない最低の[gas](../basics/gas-fees.md)価格。この価格は、各ノードによって個別に構成されたローカル値であるため、**状態遷移を引き起こすシーケンスで使用される関数では使用しないでください**。
-**コンセンサスパラメータ:** ABCIタイプ[コンセンサスパラメータ](https://tendermint.com/docs/spec/abci/apps.html#consensus-parameters)。これは、最大値など、ブロックチェーンの特定の制限を指定します。値ガスのブロック。
-**イベントマネージャー:**イベントマネージャーを使用すると、 `Context`にアクセスできるすべての呼び出し元が[` Events`](./events.md)を発行できます。モジュールはモジュール固有を定義できます
  `Events`は、さまざまな` Types`と `Attributes`を定義するか、` types/`にある共通の定義を使用します。顧客は、これらの「イベント」をサブスクライブまたはクエリできます。これらの「イベント」は、「DeliverTx」、「BeginBlock」、および「EndBlock」全体で収集され、インデックス作成のためにTendermintに返されます。例えば:
  
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
「With」関数を使用して、親から子コンテキストを作成します。 例えば: 

```go
childCtx = parentCtx.WithBlockHeader(header)
```

[Golang Context Package](https://golang.org/pkg/context) 文档指导开发者
显式传递上下文 `ctx` 作为进程的第一个参数。

##存储分支

`Context` 包含一个 `MultiStore`，它允许使用 `CacheMultiStore` 进行分支和缓存功能
(`CacheMultiStore` 中的查询被缓存以避免将来的往返)。
每个“KVStore”都分支在一个安全且隔离的临时存储中。进程可以自由地将更改写入
`CacheMultiStore`。如果状态转换序列没有问题地执行，存储分支可以
提交到序列末尾的底层存储，或者在出现某些情况时忽略它们
出错。 Context 的使用模式如下:

1. 一个进程从它的父进程接收一个上下文`ctx`，它提供了需要的信息
   执行过程。
2. `ctx.ms` 是一个**分支存储**，即[multistore](./store.md#multistore) 的一个分支，以便进程在执行时可以对状态进行更改，不改变原来的`ctx.ms`。这对于保护底层 multistore 很有用，以防在执行过程中的某个时刻需要恢复更改。
3. 进程在执行时可能会从 `ctx` 读取和写入。它可能会调用一个子进程并通过
   `ctx` 根据需要添加到它。
4. 当子进程返回时，它检查结果是成功还是失败。如果失败，什么都没有
   需要完成 - 分支 `ctx` 被简单地丢弃。如果成功，所做的更改
   `CacheMultiStore` 可以通过 `Write()` 提交给原始的 `ctx.ms`。

例如，这里是 [`runTx`](./baseapp.md#runtx-and-runmsgs) 函数的一个片段
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

这是过程:

1. 在对事务中的消息调用 `runMsgs` 之前，它使用 `app.cacheTxContext()`
    分支和缓存上下文和多存储。
2. `runMsgCtx` - 带有分支存储的上下文，在 `runMsgs` 中用于返回结果。
3.如果进程运行在[`checkTxMode`](./baseapp.md#checktx)，则不需要写
    更改 - 结果立即返回。
4.如果进程在[`deliverTxMode`](./baseapp.md#delivertx)下运行，结果显示
    成功运行所有消息后，分支多存储被写回原始。

## 下一个 {hide}

了解 [节点客户端](./node.md) {hide} 