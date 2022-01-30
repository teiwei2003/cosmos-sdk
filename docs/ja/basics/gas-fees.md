# ガソリンと費用

このドキュメントでは、CosmosSDKアプリケーションでガスと料金を処理するためのデフォルトの戦略について説明します。 {まとめ}

### 読むための前提条件

-[Cosmos SDKアプリケーション分析](./app-anatomy.md){前提条件}

## `ガス`と `料金`の紹介

Cosmos SDKでは、 `gas`は実行中のリソース消費を追跡するために使用される特別なユニットです。 `gas`は通常、ストレージの読み取りと書き込み時に消費されますが、高価な計算が必要な場合にも消費される可能性があります。それには2つの主な目的があります/

-ブロックが多くのリソースを消費せず、ファイナライズされることを確認します。これは、デフォルトで[block gas Meter](#block-gas-meter)を介してCosmosSDKに実装されています。
-エンドユーザーからのスパムや悪用を防ぎます。このため、[`message`](../building-modules/messages-and-queries.md#messages)の実行中に消費される` gas`は通常価格が設定され、結果として `fee`(` Fees = gas *ガス価格 `)。 [料金]は通常、[メッセージ]の送信者が支払う必要があります。 Cosmos SDKは、スパムを防ぐ他の方法(帯域幅プランなど)がある可能性があるため、デフォルトでは[ガス]料金を適用しないことに注意してください。それにもかかわらず、ほとんどのアプリケーションはスパムを防ぐために[料金]メカニズムを実装します。これは[`AnteHandler`](#antehandler)で行われます。

## ガスメーター

Cosmos SDKでは、 `gas`は` uint64`の単純なエイリアスであり、_gasmeter_という名前のオブジェクトによって管理されます。ガスメーターは `GasMeter`インターフェースを実装しています

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/store/types/gas.go#L34-L43

どこ/

-`GasConsumed() `は、ガスメーターインスタンスによって消費されたガスの量を返します。
-`GasConsumedToLimit() `は、ガスメーターインスタンスによって消費されたガスの量を返すか、制限に達しました。
-`Limit() `は、ガスメーターインスタンスの制限を返します。 `0`ガスメーターが無制限の場合。
-`ConsumeGas(amount Gas、descriptor string) `は、提供された` gas`の量を消費します。 `gas`がオーバーフローすると、` descriptor`メッセージでパニックになります。ガスメーターが無制限でない場合、[ガス]の消費量が制限を超えるとパニックになります。
-ガスメーターインスタンスのガス消費量が制限よりも厳密に高い場合、 `IsPastLimit()`は `true`を返し、そうでない場合は` false`を返します。
-ガスメーターインスタンスによって消費されるガスの量が制限以上の場合、 `IsOutOfGas()`は `true`を返し、そうでない場合は` false`を返します。

ガステーブルは通常[`ctx`](../core/context.md)に格納されており、ガスの消費方法は次のとおりです。

```go
ctx.GasMeter().ConsumeGas(amount, "description")
```

デフォルトでは、Cosmos SDKは2つの異なるガスメーター、[メインガスメーター](#main-gas-metter[)と[ブロックガスメーター](#block-gas-meter)を使用します。

### メインガスメーター

`ctx.GasMeter()`は、アプリケーションのメインガスメーターです。メインガスメーターは `beginBlock`で` setDeliverState`によって初期化され、状態遷移につながる実行シーケンス中のガス消費量、つまり[`BeginBlock`](../によって最初にトリガーされるbeginblocks)を追跡します。 core/baseapp.md#)、[`DeliverTx`](../core/baseapp.md#delivertx)および[` EndBlock`](../core/baseapp.md#endblock)。各 `DeliverTx`の開始時に、[` AntiHandler`](#antehandler)のメインガスメーター**を0 **に設定して、各トランザクションのガス消費量を追跡できるようにする必要があります。

ガス消費は手動で行うことができます。通常、モジュール開発者は[`BeginBlocker`、` EndBlocker`](../building-modules/beginblock-endblock.md)または[`Msg` service](../building-modules/msg-services.md)ですが、ほとんどの場合、ストレージの読み取りまたは書き込みが行われている限り、自動的に実行されます。この自動ガス消費ロジックは、[`GasKv`](../core/store.md#gaskv-store)という名前の特別なストアに実装されています。

### ブロックガスメーター

`ctx.BlockGasMeter()`はガスメーターであり、各ブロックのガス消費量を追跡し、特定の制限を超えないようにするために使用されます。[`BeginBlock`](../core/baseapp.md#beginblock)が呼び出されるたびに、` BlockGasMeter`の新しいインスタンスが作成されます。 `BlockGasMeter`は制限されており、各ブロックのガス制限はアプリケーションのコンセンサスパラメータで定義されています。デフォルトでは、Cosmos SDKアプリケーションは、Tendermint/によって提供されるデフォルトのコンセンサスパラメーターを使用します。

+++ https://github.com/tendermint/tendermint/blob/v0.34.0-rc6/types/params.go#L34-L41

新しい[transaction](../core/transaction.md)が `DeliverTx`によって処理されているとき、` BlockGasMeter`の現在の値が制限を超えているかどうかをチェックします。そうである場合、 `DeliverTx`はすぐに戻ります。 [BeginBlock]自体がガスを消費する可能性があるため、これはブロック内の最初のトランザクションでも発生する可能性があります。そうでない場合、トランザクションは正常に処理されます。 `DeliverTx`の終わりに、` ctx.BlockGasMeter() `によって追跡されるガスは、消費されるトランザクションの数を増やします/

```go
ctx.BlockGasMeter().ConsumeGas(
	ctx.GasMeter().GasConsumedToLimit(),
	"block gas meter",
)
```

## AnteHandler

[CheckTx]および[DeliverTx]中にトランザクションごとに[AnteHandler]を実行し、トランザクション内の各[sdk.Msg]Protobuf[Msg]サービスメソッドの前に実行します。 `AnteHandler`には次の署名があります/

```go
//AnteHandler authenticates transactions, before their internal messages are handled.
//If newCtx.IsZero(), ctx is used instead.
type AnteHandler func(ctx Context, tx Tx, simulate bool) (newCtx Context, result Result, abort bool)
```

`anteHandler`はコアCosmosSDKではなく、モジュールに実装されています。これにより、開発者はアプリケーションのニーズに合ったバージョンの `AnteHandler`を選択できます。つまり、今日のほとんどのアプリケーションは、[`auth`モジュール](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth)で定義されているデフォルトの実装を使用しています。以下は、通常のCosmosSDKアプリケーションでの `anteHandler`の役割です/

-トランザクションタイプが正しいことを確認します。トランザクションタイプは、 `anteHandler`を実装するモジュールで定義され、トランザクションインターフェイスに従います/
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/tx_msg.go#L49-L57
  これにより、開発者はさまざまなタイプを使用してアプリケーショントランザクションを処理できます。デフォルトの `auth`モジュールでは、デフォルトのトランザクションタイプは` Tx`/です。
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L12-L25
-トランザクションに含まれる各[`message`](../building-modules/messages-and-queries.md#messages)の署名を確認します。各[メッセージ]は1人以上の送信者によって署名される必要があり、これらの署名は[anteHandler]で検証される必要があります。
-`CheckTx`中に、トランザクションによって提供されるガス価格がローカルの `min-gas-prices`よりも高いかどうかを確認します(ガス価格は次の式から差し引くことができることに注意してください/`料金=ガス*ガス価格 ` )。 `min-gas-prices`は、各フルノードのローカルパラメータであり、` CheckTx`中に最低料金を提供しないトランザクションを破棄するために使用されます。これにより、メモリプールがスパムトランザクションによって送信されるスパムにならないことが保証されます。
-トランザクションの送信者が[手数料]を支払うのに十分な資金を持っていることを確認します。エンドユーザーがトランザクションを生成するときは、次の3つのパラメーターのうち2つを指定する必要があります(3つ目は暗黙的なパラメーターです)/`fees`、` gas`、および `gas-prices`。これは、ノードがトランザクションを実行するために支払う意思がある金額を示しています。提供された `gas`値は、後で使用するために` GasWanted`という名前のパラメーターに保存されます。
-`newCtx.GasMeter`を0に設定し、 `GasWanted`に制限します。 **この手順は非常に重要です**。これは、トランザクションが無制限のガスを消費しないようにするだけでなく、各 `DeliverTx`間で` ctx.GasMeter`をリセットするためです( `ctx`は`の実行中に `newCtx`に設定されますanteHandler `その後、` DeliverTx`が呼び出されるたびに、 `anteHandler`が実行されます)。

前述のように、 `anteHandler`は、トランザクションが実行中に消費できる最大の` gas`制限を返します。これは `GasWanted`と呼ばれます。最終的に消費される実際の量は `GasUsed`で表されるため、` GasUsed = <GasWanted`である必要があります。[`DeliverTx`](../core/baseapp.md#delivertx)が戻ると、` GasWanted`と `GasUsed`の両方が基盤となるコンセンサスエンジンに中継されます。

## 次へ{hide}

[baseapp](../core/baseapp.md){hide}を理解する 