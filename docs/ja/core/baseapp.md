# 基本的なアプリケーション

このドキュメントでは、CosmosSDKアプリケーションのコア機能を実装する抽象化である「BaseApp」について説明します。 {まとめ}

## 読むための前提条件

-[Cosmos SDKアプリケーション分析](../basics/app-anatomy.md){前提条件}
-[Cosmos SDKトランザクションライフサイクル](../basics/tx-lifecycle.md){prereq}

## 導入する

`BaseApp`は基本的なタイプであり、CosmosSDKアプリケーションのコアを実装します。

-[アプリケーションブロックリンクポート](#abci)、ステートマシンと基盤となるコンセンサスエンジン(Tendermintなど)間の通信に使用されます。
-[サービスルーター](#service-routers)、メッセージとクエリを適切なモジュールにルーティングします。
-異なる[states](#states)。ステートマシンは、受信したABCIメッセージに応じて異なる揮発性状態を更新できるためです。

BaseAppの目標は、CosmosSDKアプリケーションのベースレイヤーを提供することです。
開発者は、独自のカスタムアプリケーションを構築するために簡単に拡張できます。 いつもの、
開発者は、次のようにアプリケーションのカスタムタイプを作成します。 

```go
type App struct {
 //reference to a BaseApp
  *baseapp.BaseApp

 //list of application store keys

 //list of application keepers

 //module manager
}
```

「BaseApp」拡張アプリケーションを使用すると、前者は「BaseApp」のすべてのメソッドにアクセスできます。
これにより、開発者は、カスタムアプリケーションを、代わりに必要なモジュールでアセンブルできます。
ABCI、サービスルーター、および状態を実装するためのハードワークについて心配する必要があります
管理ロジック。

## タイプ定義

「BaseApp」タイプは、CosmosSDKベースのアプリケーションの多くの重要なパラメーターを保存します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/baseapp.go#L46-L131

最も重要なコンポーネントを見てみましょう。

> **注**:すべてのパラメーターが説明されているわけではなく、最も重要なパラメーターのみが説明されています。参照する
>タイプ定義の完全なリスト。

まず、アプリケーションの起動中に初期化される重要なパラメータ:

-[`CommitMultiStore`](./store.md#commitmultistore):これはアプリケーションのメインストアです。
  [各ブロックの終わり](#commit)送信された仕様ステータスに保存されます。この店
  **しない**キャッシング。これは、アプリケーションの揮発性(コミットされていない)ステータスを更新するために使用されないことを意味します。
  `CommitMultiStore`はマルチストアであり、ストレージのストレージを意味します。アプリケーションの各モジュール
  マルチストアで1つ以上の `KVStores`を使用して、それらの状態のサブセットを永続化します。
-データベース: `commitMultiStore`は` db`を使用してデータの永続性を処理します。
-[`Msg`サービスルーター](#msg-service-router):` msgServiceRouter`は `sdk.Msg`リクエストを適切なものにルーティングするのに役立ちます
  処理用のモジュール `Msg`サービス。ここでの `sdk.Msg`は、処理する必要のあるトランザクションコンポーネントを指します
  ABCIメッセージを実装する代わりに、アプリケーションの状態を更新するためにサービスによって処理されます
  アプリケーションと基盤となるコンセンサスエンジン間のインターフェイス。
-[gRPCクエリルーター](#grpc-query-router): `grpcQueryRouter`は、gRPCクエリをにルーティングするのに役立ちます
  処理に適したモジュール。これらのクエリ自体はABCIメッセージではありませんが、
  関連するモジュールに中継されるgRPC`Query`サービス。
-[`TxDecoder`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#TxDecoder):デコード用
  基盤となるTendermintエンジンによって中継される生のトランザクションバイト。
-[`ParamStore`](#paramstore):アプリケーションのコンセンサスパラメータを取得および設定するために使用されるパラメータストア。
-[`AnteHandler`](#antehandler):このハンドラーは、署名の検証、料金の支払い、
  その他のメッセージを実行する前に、トランザクションがいつ受信されるかを確認してください。実行中
  [`CheckTx/RecheckTx`](#checktx)および[` DeliverTx`](#delivertx)。
-[`InitChainer`](../basics/app-anatomy.md#initchainer)、
  [`BeginBlocker`と` EndBlocker`](../basics/app-anatomy.md#beginblocker-and-endblocker):これらは
  アプリケーションが `InitChain`、` BeginBlock`、 `EndBlock`を受け取ったときに実行される関数
  基盤となるTendermintエンジンからのABCIメッセージ。

次に、[volatile state](#volatile-states)(つまり、キャッシュ状態)を定義するために使用されるパラメーター:

-`checkState`:この状態は、[`CheckTx`](#checktx)中に更新され、[` Commit`](#commit)中にリセットされます。
-`deliverState`:この状態は[`DeliverTx`](#delivertx)中に更新され、上記では` nil`に設定されます
  [`Commit`](#commit)そしてBeginBlockで再初期化します。

最後に、いくつかのより重要なパラメータ:

-`voteInfos`:このパラメーターは、事前コミットがないバリデーターのリスト、または
  彼らが投票しなかったため、または提案者が彼らの投票を含めなかったため。この情報は
  [Context](#context)によって運ばれ、アプリケーションはそれを次のようなさまざまな目的に使用できます。
  不在のバリデーターを罰します。
-`minGasPrices`:このパラメータは、ノードが受け入れる最小ガス価格を定義します。これは
  ** local **パラメータは、各フルノードが異なる `minGasPrices`を設定できることを意味します。に使用されます
  [`CheckTx`](#checktx)中の` AntiHandler`は、主にスパム保護メカニズムとして使用されます。トレード
  [mempool](https://tendermint.com/docs/tendermint-core/mempool.html#transaction-ordering)と入力します
  取引のガス価格が最低ガス価格の1つよりも高い場合のみ
  `minGasPrices`(たとえば、` minGasPrices == 1uatom、1photon`の場合、トランザクションのガス価格は次のようになります。
  「1uatom」または「1photon」より大きい)。
-`appVersion`:アプリケーションのバージョン。に設定されています
  [アプリケーションコンストラクター](../basics/app-anatomy.md#constructor-function)。

## 构造函数 

```go
func NewBaseApp(
  name string, logger log.Logger, db dbm.DB, txDecoder sdk.TxDecoder, options ...func(*BaseApp),
) *BaseApp {

 //...
}
```

`BaseApp`コンストラクターは非常に単純です。 注目に値するのは
追加の[`options`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/options.go)を提供する場合があります
`BaseApp`に対して、それらを順番に実行します。 `options`は通常` setter`関数です
プルーニングオプションを設定するための `SetPruning()`や設定するための `SetMinGasPrices()`などの重要なパラメータの場合
ノードの `min-gas-prices`。

もちろん、開発者はアプリケーションのニーズに応じて「オプション」を追加できます。

## 状況更新

`BaseApp`は、2つの主要な揮発性状態とルートまたはメイン状態を維持します。主な状況
アプリケーションの正規状態と揮発性状態、 `checkState`と` deliverState`、
[`Commit`](#commit)期間中に行われたメイン状態間の状態遷移を処理するために使用されます。

内部的には、メイン状態またはルート状態と呼ばれる `CommitMultiStore`が1つだけあります。
このルート状態から、_store branch_( `CacheWrap`関数によって実行される)と呼ばれるメカニズムを使用して、2つの揮発性状態を導き出します。
これらのタイプは次のように説明できます。

！[タイプ](./baseapp_state_types.png)

### InitChainステータスの更新

InitChainの実行中、2つの揮発性状態がブランチを介して設定されます。つまり、checkStateとdeliverStateです。
ルート `CommitMultiStore`。以降の読み取りと書き込みは、ブランチバージョンの `CommitMultiStore`で行われます。
メイン状態への不要なラウンドトリップを回避するために、ブランチストアへのすべての読み取りがキャッシュされます。

！[InitChain](./baseapp_state-initchain.png)

### CheckTxステータスの更新

`CheckTx`の間、ルートから最後にコミットされた状態に基づく` checkState`
ストレージ。読み取りと書き込みに使用されます。ここでは、 `AnteHandler`を実行し、サービスルーターを確認するだけです。
トランザクション内のすべてのメッセージが存在します。 `AnteHandler`を実行すると、分岐することに注意してください
分岐した `checkState`。これには副作用があります。`AnteHandler`が失敗した場合、
状態遷移は `checkState`には反映されません。つまり、` checkState`は
成功。

！[CheckTx](./baseapp_state-checktx.png)

### BeginBlockステータスの更新

`BeginBlock`の間、` deliverState`は後続の `DeliverTx`ABCIメッセージで使用されるように設定されます。この
`deliverState`は、ルートストアから最後にコミットされた状態に基づいており、分岐しています。
[`Commit`](#commit)では、` deliverState`が `nil`に設定されていることに注意してください。

！[BeginBlock](./baseapp_state-begin_block.png)

### DeliverTxステータスの更新

「DeliverTx」の状態フローは、状態遷移がで発生することを除いて、「CheckTx」の状態フローとほぼ同じです。
トランザクションで `deliverState`とメッセージを実行します。 「CheckTx」と同様に、状態遷移
デュアルブランチ状態-`deliverState`で発生します。成功したメッセージ実行結果
`deliverState`専用に書き込みます。メッセージの実行に失敗した場合、ステータスは次のように変化することに注意してください。
AnteHandlerは永続化されます。

！[DeliverTx](./baseapp_state-deliver_tx.png)

### ステータスの更新を送信する

`Commit`中に、` deliverState`で発生したすべての状態遷移が最終的に書き込まれます
ルート `CommitMultiStore`は次にディスクにコミットし、新しいアプリケーションを生成します
ルートハッシュ。これらの状態遷移は現在、最終的なものと見なされます。最後に、 `checkState`はに設定されます
新しく送信された状態と `deliverState`は` nil`に設定され、 `BeginBlock`でリセットされます。

！[送信](./baseapp_state-commit.png)

## パラメータストレージ

`InitChain`の間に、` RequestInitChain`はパラメータを含む `ConsensusParams`を提供します
エビデンスパラメータに加えて、最大ガスやサイズなどのブロック実行にも関連しています。 これらの場合
パラメータはnilではなく、BaseAppの `ParamStore`で設定されます。 舞台裏、 `ParamStore`
実際には、 `x/params`モジュール` Subspace`によって管理されます。 これにより、パラメータを調整できます
オンチェーンガバナンス。

## サービスルーター

アプリケーションがメッセージとクエリを受信すると、それらを適切なモジュールにルーティングして処理する必要があります。 ルーティングは、メッセージ用の「msgServiceRouter」とクエリ用の「grpcQueryRouter」を含む「BaseApp」を介して行われます。

### `Msg`サービスルーター

[`sdk.Msg`s](#../building-modules/messages-and-queries.md#messages)は、トランザクションから抽出された後にルーティングする必要があります。これらのトランザクションは、[` CheckTx`](# checktx)および[`DeliverTx`](#delivertx)ABCIメッセージ。このため、 `BaseApp`には` msgServiceRouter`があり、完全修飾サービスメソッド(各モジュールのProtobuf `Msg`サービスで定義されている` string`)を対応するモジュールの `MsgServer`実装にマップします。

[`BaseApp`に含まれるデフォルトの` msgServiceRouter`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/msg_service_router.go)はステートレスです。ただし、一部のアプリケーションでは、ガバナンスが特定のルートを無効にしたり、アップグレードのために新しいモジュールをポイントしたりできるようにするなど、よりステートフルなルーティングメカニズムを使用したい場合があります。このため、 `sdk.Context`もそれぞれに渡されます

アプリケーションの `msgServiceRouter`は、アプリケーションの[module manager](../building-modules/module-manager.md#manager)を使用して(` RegisterServices`メソッドを介して)初期化にすべてのルートを使用し、それ自体が初期化アプリケーションの[コンストラクター](../basics/app-anatomy.md#app-constructor)内のアプリケーションモジュール。

### gRPCクエリルーター

`sdk.Msg`と同様に、[` queries`](../building-modules/messages-and-queries.md#queries)は、対応するモジュールの[`Query`サービス](../構築モジュール/Query service.md)。 このため、 `BaseApp`には` grpcQueryRouter`があり、モジュールの完全修飾サービスメソッド(Protobuf `Query` gRPCで定義されている` string`)を `QueryServer`実装にマップします。 `grpcQueryRouter`は、クエリ処理の初期段階で、gRPCクエリをgRPCエンドポイントに直接送信するか、TendermintRPCエンドポイントの[` Query`ABCIメッセージ](#query)によって呼び出されます。

`msgServiceRouter`と同様に、` grpcQueryRouter`はアプリケーションの[module manager](../building-modules/module-manager.md)( `RegisterServices`メソッドを介して)を使用してすべてのクエリルートで初期化し、アプリケーションを使用しますそれ自体プログラムの[コンストラクター](../basics/app-anatomy.md#app-constructor)内のすべてのアプリケーションモジュールが初期化されます。

## メインABCIニュース

[アプリケーション-ブロックリンクポート](https://tendermint.com/docs/spec/abci/)(ABCI)は、ステートマシンをコンセンサスエンジンに接続して完全に機能するフルノードを形成する一般的なインターフェイスです。これは任意の言語でパッケージ化でき、ABCI互換のコンセンサスエンジン(Tendermintなど)上に構築された各アプリケーション固有のブロックチェーンによって実装する必要があります。

コンセンサスエンジンは、2つの主要なタスクを処理します。

-主にゴシップブロック、トランザクション、コンセンサス投票を含むネットワークロジック。
-コンセンサスロジック。ブロックの形式で決定されるトランザクションの順序につながります。

**ない**トランザクションのステータスまたは有効性を定義するためのコンセンサスエンジンの役割。通常、トランザクションは「[] bytes」の形式でコンセンサスエンジンによって処理され、デコードと処理のためにABCIを介してアプリケーションに中継されます。ネットワークとコンセンサスプロセスの重要な瞬間(たとえば、ブロックの開始、ブロックの送信、未確認のトランザクションの受信など)で、コンセンサスエンジンは、ステートマシンが実行するABCIメッセージを送信します。 。

BaseAppにはインターフェースの実装が組み込まれているため、Cosmos SDKに基づいて構築する開発者は、ABCI自体を実装する必要はありません。 `BaseApp`によって実装された主なABCIメッセージを見てみましょう:[` CheckTx`](#checktx)と[`DeliverTx`](#delivertx) 

### CheckTx

新しいものが確認されない場合(つまり、有効なブロックに含まれていない場合)、基盤となるコンセンサスエンジンは `CheckTx`を送信します
トランザクションはフルノードによって受信されます。 `CheckTx`の役割は、フルノードのmempoolを保護することです
(未確認のトランザクションがブロックに含まれるまで保存される場合)スパムトランザクションから。
未確認のトランザクションは、「CheckTx」を介してのみピアに転送されます。

`CheckTx()`は_stateful_および_stateless_チェックを実行できますが、開発者は一生懸命働く必要があります
それらを軽くします。 Cosmos SDKでは、[Decoding Transaction](./encoding.md)の後に、 `CheckTx()`が実装されています
次のチェックを行います。

1.トランザクションから `sdk.Msg`を抽出します。
2.各 `sdk.Msg`に対して` ValidateBasic() `を呼び出して、_stateless_チェックを実行します。これは行われます
   まず、_stateless_チェックは_stateful_チェックよりも計算コストが低いためです。もしも
   `ValidateBasic()`は失敗し、 `CheckTx`は_stateful_チェックを実行する前に戻るため、リソースを節約できます。
3.[account](../basics/accounts.md)でモジュールに関連しない_stateful_チェックを実行します。このステップは主にチェックすることです
   `sdk.Msg`署名は有効であり、十分な料金を提供し、アカウントを送信します
   上記の費用を賄うのに十分な資金があります。正確な[`gas`](../basics/gas-fees.md)カウントがないことに注意してください。
   `sdk.Msg`は処理されないためです。通常、[`AnteHandler`](../basics/gas-fees.md#antehandler)は` gas`が提供されているかどうかをチェックします
   トランザクションは、元のトランザクションサイズに基づく最小参照ガス量よりも優れています。
   スパムを回避するために、0のガス取引を提供します。
4.各 `sdk.Msg`の完全修飾サービスメソッドが` msgServiceRouter`のルートと一致することを確認しますが、実際には**一致しません**
   `sdk.Msg`を処理します。 `sdk.Msg`は、仕様の状態を更新する必要がある場合にのみ処理する必要があります。
   これは `DeliverTx`の間に起こりました。

手順2と3は、[`RunTx()`](#runtx-antehandler-and-runmsgs)の[`AnteHandler`](../basics/gas-fees.md#antehandler)によって実行されます。
関数 `CheckTx()`は `runTxModeCheck`モードで呼び出されます。 `CheckTx()`の各ステップで、
`checkState`と呼ばれる特別な[volatilestate](#volatile-states)が更新されます。この状態は維持するために使用されます
各トランザクションの `CheckTx()`呼び出しによってトリガーされた一時的な変更を変更せずに追跡します
[主な仕様状態](#main-state)。たとえば、トランザクションが `CheckTx()`を通過すると、
取引手数料は、「checkState」の送信者のアカウントから差し引かれます。 2番目のトランザクションが
最初のアカウントを処理する前に同じアカウントから受信し、そのアカウントはすべてを消費しました
最初のトランザクション、 `checkState`の資金、2番目のトランザクションは` CheckTx`()に失敗し、
拒否されます。いずれの場合も、送信者のアカウントは実際に取引前に料金を支払うことはありません
`checkState`がメイン状態に送信されることはないため、実際にはブロックに含まれています。この
ブロックが[コミット](#commit)されるたびに、 `checkState`はメイン状態の最新の状態にリセットされます。

`CheckTx`は、タイプ[` abci.ResponseCheckTx`](https://tendermint.com/docs/spec/abci/abci.html#messages)の基礎となるコンセンサスエンジンへの応答を返します。
応答には次のものが含まれます。

-`Code(uint32) `:応答コード。成功した場合は `0`。
-`Data([] byte) `:結果バイト(存在する場合)。
-`ログ(文字列): `アプリケーションロガーの出力。不確かかもしれません。
-`情報(文字列): `追加情報。不確かかもしれません。
-`GasWanted(int64) `:トランザクションによって要求されたガスの量。これは、トランザクションが生成されるときにユーザーによって提供されます。
-`GasUsed(int64) `:トランザクションによって消費されたガスの量。 「CheckTx」では、この値は、トランザクションバイトの標準コストに元のトランザクションのサイズを掛けて計算されます。次は例です:
  +++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/x/auth/ante/basic.go#L104-L105
-`Events([] cmn.KVPair) `:トランザクションのフィルタリングとインデックス作成に使用されるKey-Valueタグ(アカウントなど)。詳細については、[`event`s](./events.md)を参照してください。
-`コードスペース(文字列) `:コードの名前空間。

#### Txを再確認します

`Commit`の後、ノードのローカルメモリプールに残っているすべてのトランザクションで` CheckTx`が再度実行されます
フィルタブロックに含まれているものの後。 メモリプールがすべてのトランザクションを再チェックしないようにします
ブロックが送信されるたびに、構成オプション `mempool.recheck = false`を設定できます。 として
Tendermint v0.32.1、追加の `Type`パラメータを` CheckTx`関数で使用できます
着信トランザクションが新規( `CheckTxType_New`)か再チェック(` CheckTxType_Recheck`)かを示します。
これにより、「CheckTxType_Recheck」中に署名検証などの特定のチェックをスキップできます。 

### DeliverTx

基盤となるコンセンサスエンジンがブロックプロポーザルを受信すると、ブロック内のすべてのトランザクションをアプリケーションで処理する必要があります。この目的のために、基盤となるコンセンサスエンジンは、「DeliverTx」メッセージを各トランザクションアプリケーションに順番に送信します。

特定のブロックの最初のトランザクションを処理する前に、[`BeginBlock`](#beginblock)中に` deliverState`という名前の[volatilestate](#volatile-states)を初期化します。この状態は、トランザクションが `DeliverTx`で処理されるたびに更新され、ブロックが[commit](#commit)の場合、「nil」に設定した後、[main state](#main-state)に送信されます。

`DeliverTx`は**` CheckTx` **とまったく同じステップを実行します。ステップ3には注意すべき点が1つあり、5番目のステップが追加されています。

1. `AnteHandler` **いいえ**トランザクションの` gas-prices`が十分であるかどうかを確認します。これは、gas-pricesのチェックされたmin-gas-prices値がノードに対してローカルであるため、1つの完全なノードに十分な値が別のノードに適用されない場合があるためです。つまり、提案者は、提案するブロックの総コストからボーナスを受け取るため、インセンティブはありませんが、無料でトランザクションを含めることができます。
2.トランザクション内の `sdk.Msg`ごとに、対応するモジュールのProtobuf [` Msg`サービス](../building-modules/msg-services.md)にルーティングします。追加の_stateful_チェックを実行すると、 `deliverState`の` context`に格納されているブランチがモジュールの `keeper`によって更新されます。 `Msg`サービスが正常に戻ると、` context`に格納されているブランチマルチストアが `deliverState`と` CacheMultiStore`に書き込まれます。

(2)で概説した追加の5番目のステップでは、ストレージの読み取り/書き込みごとに「GasConsumed」の値が増加します。各操作のデフォルトのコストを見つけることができます:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/store/types/gas.go#L164-L175

いつでも、 `GasConsumed> GasWanted`の場合、関数は` Code！= 0`を返し、 `DeliverTx`は失敗します。

`DeliverTx`は、タイプ[` abci.ResponseDeliverTx`](https://tendermint.com/docs/spec/abci/abci.html#delivertx)の基礎となるコンセンサスエンジンに応答を返します。応答には次のものが含まれます。

-`Code(uint32) `:応答コード。成功した場合は `0`。
-`Data([] byte) `:結果バイト(存在する場合)。
-`ログ(文字列): `アプリケーションロガーの出力。不確かかもしれません。
-`情報(文字列): `追加情報。不確かかもしれません。
-`GasWanted(int64) `:トランザクションによって要求されたガスの量。これは、トランザクションが生成されるときにユーザーによって提供されます。
-`GasUsed(int64) `:トランザクションによって消費されたガスの量。 「DeliverTx」では、この値は、トランザクションバイトの標準コストに元のトランザクションのサイズを掛け、ストレージが読み取り/書き込みされるたびにガスを追加することによって計算されます。
-`Events([] cmn.KVPair) `:トランザクションのフィルタリングとインデックス作成に使用されるKey-Valueタグ(アカウントなど)。詳細については、[`event`s](./events.md)を参照してください。
-`コードスペース(文字列) `:コードの名前空間。

## RunTx、AnteHandler、RunMsgs 

### RunTx

`CheckTx`/` DeliverTx`から `RunTx`を呼び出してトランザクションを処理し、パラメーターとして` runTxModeCheck`または `runTxModeDeliver`を使用して2つの実行モードを区別します。 `RunTx`がトランザクションを受信したとき、それはすでにデコードされていることに注意してください。

`RunTx`が呼び出されたときに最初に行うことは、適切なモード(` runTxModeCheck`または `runTxModeDeliver`)で` getContextForTx() `関数を呼び出して、` context`の `CacheMultiStore`を取得することです。この `CacheMultiStore`は、キャッシュ機能(クエリ要求に使用)を備えたメインストアのブランチであり、` BeginBlock`の間に `DeliverTx`に対してインスタンス化され、前のブロックの` Commit`の間に `CheckTx`に対してインスタンス化されます。その後、[`gas`](../basics/gas-fees.md)管理のために2つの` defer func() `を呼び出します。これらは `runTx`が戻ったときに実行され、` gas`が実際に消費されていることを確認し、エラー(存在する場合)がスローされます。

その後、 `RunTx()`は `Tx`の各` sdk.Msg`で `ValidateBasic()`を呼び出して、予備的な_stateless_妥当性チェックを実行します。 `sdk.Msg`が` ValidateBasic() `を渡さなかった場合、` RunTx() `はエラーを返します。

次に、アプリケーションの[`anteHandler`](#antehandler)(存在する場合)を実行します。このステップの準備をするとき、 `checkState`/` deliverState`の `context`と` context`の `CacheMultiStore`は両方とも` cacheTxContext() `関数を使用して分岐しました。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/baseapp/baseapp.go#L623-L630

これにより、 `RunTx`は、最終的に失敗した場合に、` anteHandler`の実行中に状態への変更をコミットしないようになります。 また、Cosmos SDK [object-capabilities](./ocap.md)の重要な部分である `anteHandler`を実装するモジュールの書き込みステータスを防ぐこともできます。

最後に、[`RunMsgs()`](#runmsgs)関数を呼び出して、 `Tx`の` sdk.Msg`を処理します。 このステップの準備をするとき、 `anteHandler`と同様に、` checkState`/`deliverState`の` context`と `context`の` CacheMultiStore`は両方とも `cacheTxContext()`関数を使用して分岐します。

### 前処理プログラム

`AnteHandler`は、トランザクションの内部メッセージを処理する前にトランザクションを認証するために使用される` AnteHandler`インターフェースを実装する特別なハンドラーです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/handler.go#L6-L8

`AnteHandler`は理論的にはオプションですが、それでもパブリックブロックチェーンネットワークの非常に重要な部分です。これには3つの主な目的があります。

-料金を差し引いて[`sequence`](./transactions.md#transaction-generation)をチェックすることで、スパムに対する主要な防御線となり、2番目の防御線(最初の防御線はメモリプール)になります。トランザクションの再生を防ぎます。
-署名が有効であること、または送信者がコストをカバーするのに十分な資金を持っていることを確認するなど、予備的な_ステートフル_有効性チェックを実行します。
-取引手数料を請求することにより、利害関係者の動機付けに役割を果たす。

`BaseApp`は、パラメータとして` anteHandler`を保持します。これは、[アプリケーションコンストラクタ](../basics/app-anatomy.md#application-constructor)で初期化されます。最も広く使用されている `anteHandler`は[` auth`モジュール](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/ante/ante.go)です。

`anteHandler`の詳細については、[こちら](../basics/gas-fees.md#antehandler)をクリックしてください。

### RunMsgs

`RunMsgs`は` RunTx`から呼び出され、トランザクション内の各メッセージのルーティングが存在するかどうかをチェックするパラメータとして `runTxModeCheck`を使用し、` runTxModeDeliver`を使用して実際に `sdk.Msg`を処理します。

まず、 `sdk.Msg`を表すProtobuf`Any`の` type_url`をチェックすることにより、 `sdk.Msg`の完全修飾型名を取得します。 次に、アプリケーションの[`msgServiceRouter`](#msg-service-router)を使用して、` type_url`に関連する `Msg`サービスメソッドが存在するかどうかを確認します。 このとき、 `mode == runTxModeCheck`の場合は、` RunMsgs`に戻ります。 それ以外の場合、 `mode == runTxModeDeliver`の場合、` RunMsgs`が戻る前に[`Msg` service](../building-modules/msg-services.md)RPCを実行します。

## その他のABCIニュース

### 初期化チェーン

[`InitChain` ABCIメッセージ](https://tendermint.com/docs/app-dev/abci-spec.html#initchain)は、チェーンが最初に開始されたときに、基盤となるTendermintエンジンから送信されます。 これは主に、次のような**初期化**パラメータとステータスに使用されます。

-[コンセンサスパラメータ](https://tendermint.com/docs/spec/abci/apps.html#consensus-parameters)から `setConsensusParams`まで。
-[`checkState`および` deliverState`](#volatile-states)から `setCheckState`および` setDeliverState`まで。
-[ブロックガスメーター](../basics/gas-fees.md#block-gas-meter)、無制限のガスを使用して作成トランザクションを処理します。

最後に、 `BaseApp`の` InitChain(req abci.RequestInitChain) `メソッドは、アプリケーションの[` initChainer() `](../basics/app-anatomy.md#initchainer)を呼び出して、「Genesisファイル ""アプリケーションの名前を取得するには、アプリケーションが定義されている場合は、各アプリケーションモジュールの[`InitGenesis`](../building-modules/genesis.md#initgenesis)関数を呼び出します。

### スターティングブロック

[`BeginBlock` ABCIメッセージ](#https://tendermint.com/docs/app-dev/abci-spec.html#beginblock)正しい提案者によって作成されたブロック提案が受信されたときに、基盤となるTendermintエンジンから送信されます。ブロック内のトランザクションごとに[`DeliverTx`](#delivertx)を実行する前。これにより、開発者は各ブロックの先頭でロジックを実行できます。 Cosmos SDKでは、 `BeginBlock(req abci.RequestBeginBlock)`メソッドが次の操作を実行します。

-`setDeliverState`関数を介してパラメーターとして渡された `reqabci.RequestBeginBlock`を使用して、[` deliverState`](#volatile-states)を最新のヘッダーで初期化します。
  +++ https://github.com/cosmos/cosmos-sdk/blob/7d7821b9af132b0f6131640195326aa02b6751db/baseapp/baseapp.go#L387-L397
  この機能は、[メインガスメーター](../basics/gas-fees.md#main-gas-meter)もリセットします。
-`maxGas`を使用して、初期化を制限します[ブロックガスメーター](../basics/gas-fees.md#block-gas-meter)。ブロックで消費される「ガス」は「maxGas」を超えることはできません。このパラメーターは、アプリケーションのコンセンサスパラメーターで定義されます。
-[`beginBlocker()`](../basics/app-anatomy.md#beginblocker-and-endblock)は、主に[`BeginBlocker()`](../building-modules/beginblock)を実行します。 -endblock.md#beginblock)各アプリケーションモジュールのメソッド。
-アプリケーションの[`VoteInfos`](https://tendermint.com/docs/app-dev/abci-spec.html#voteinfo)を設定します。つまり、前のブロックの_precommit_が現在の提案に含まれますブロックする。この情報は[`Context`](./context.md)に取り込まれるため、` DeliverTx`および `EndBlock`で使用できます。 

### EndBlock

[`EndBlock` ABCIメッセージ](#https://tendermint.com/docs/app-dev/abci-spec.html#endblock)は、[` DeliverTx`](#delivertx )実行中の各トランザクション。 これにより、開発者は各ブロックの最後でロジックを実行できます。 Cosmos SDKでは、バッチの `EndBlock(req abci.RequestEndBlock)`メソッドは、主にアプリケーションの[`EndBlocker()`](../basics/app-anatomy.md#beginblocker-and-endblock)を実行します。アプリケーションモジュールごとに[`EndBlocker()`](../building-modules/beginblock-endblock.md#beginblock)メソッドを実行します。

### 犯罪

[`Commit` ABCIメッセージ](https://tendermint.com/docs/app-dev/abci-spec.html#commit)は、フルノードが2/3 + Verifierから_precommits_を受信した後、基盤となるTendermintエンジンから送信されます(投票力で重み付け)。 BaseApp側では、Commit(res abci.ResponseCommit)関数が実装され、BeginBlock、DeliverTx、およびEndBlock中に発生したすべての有効な状態遷移を送信し、次のブロックの状態をリセットします。

状態遷移をコミットするために、 `Commit`関数は` deliverState.ms`で `Write()`関数を呼び出します。ここで、 `deliverState.ms`はメインストア` app.cms`のブランチマルチストアです。 次に、 `Commit`関数は` checkState`を最新のヘッダー( `deliverState.ctx.BlockHeader`から取得)に設定し、` deliverState`を `nil`に設定します。

最後に、 `Commit`は` app.cms`の約束されたハッシュを基礎となるコンセンサスエンジンに返します。 このハッシュは、次のブロックのヘッダーで参照として使用されます。

### 情報

[`Info` ABCIニュース](https://tendermint.com/docs/app-dev/abci-spec.html#info)は、基盤となるコンセンサスエンジンからの単純なクエリであり、特に後者をアプリケーションと同期するためのものです。起動時に発生するハンドシェイク。 呼び出されると、BaseAppのInfo(res abci.ResponseInfo)関数は、アプリケーションの名前とバージョン、およびapp.cmsの最後の送信のハッシュ値を返します。

### 聞く

[`Query` ABCIメッセージ](https://tendermint.com/docs/app-dev/abci-spec.html#query)TendermintなどのRPCを介して受信したクエリを含む、基盤となるコンセンサスエンジンから受信したクエリを提供するために使用されますRPC。以前はアプリケーションとのインターフェースを構築するための主要なエントリポイントでしたが、Cosmos SDK v0.40に[gRPCクエリ](../building-modules/query-services.md)が導入されたことで、その使用がさらに影響を受けました。制限。アプリケーションは、[ここ](https://tendermint.com/docs/app-dev/abci-spec.html#query)で概説されている `Query`メソッドを実装するときに特定のルールに従う必要があります。

各Tendermintの `query`には、クエリするコンテンツを示す` string`である `path`があります。 `path`がgRPCの完全修飾サービスメソッドと一致する場合、` BaseApp`はクエリを `grpcQueryRouter`に延期し、[上記](#grpc-query-router)で説明されているように処理させます。それ以外の場合、 `path`はgRPCルーターによって処理されるクエリ(まだ)を表します。 `BaseApp`は、` path`文字列を `/`区切り文字で区切ります。慣例により、分割文字列の最初の要素( `splitted [0]`)には、 `query`のカテゴリ(` app`、 `p2p`、`​​ store`、または `custom`)が含まれます。 `Query(req abci.RequestQuery)`メソッドの `BaseApp`実装は、次の4つの主要なクエリカテゴリにサービスを提供する単純なスケジューラです。

-アプリケーションのバージョンのクエリなど、アプリケーションに関連するクエリは、 `handleQueryApp`メソッドを介して提供されます。
-マルチストアへの直接クエリは、 `handlerQueryStore`メソッドによって提供されます。これらの直接クエリは、 `app.queryRouter`を介したカスタムクエリとは異なり、主にブロックエクスプローラーなどのサードパーティのサービスプロバイダーによって使用されます。
-サービスを提供するための `handleQueryP2P`メソッドを介したP2Pクエリ。これらのクエリは「app.addrPeerFilter」または「app.ipPeerFilter」を返します。これには、それぞれアドレスまたはIPでフィルタリングされたピアのリストが含まれています。これらのリストは、最初に `BaseApp`の[constructor](#constructor)の` options`によって初期化されます。
-レガシークエリ(gRPCクエリの導入前)を含むカスタムクエリは、 `handleQueryCustom`メソッドを介して提供されます。 `handleQueryCustom`は、` app.queryRouter`から取得した `queryRoute`を使用して、クエリを対応するモジュールの[legacy` querier`](../building-modules/query-services.md#legacy-querier)にマッピングしています。

## 次へ{非表示}

[transactions](./transactions.md){hide}の詳細