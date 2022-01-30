# ADR 045:ミドルウェアとしてのBaseApp `{Check、Deliver} Tx`

## 変更ログ

-20.08.2021:最初のドラフト。
-07.12.2021: `tx.Handler`インターフェースを更新します([\#10693](https://github.com/cosmos/cosmos-sdk/pull/10693))。

## 状態

受け入れられました

## 概要

このADRは、現在のBaseApp`runTx`およびアンティハンドラーの設計をミドルウェアベースの設計に置き換えます。

## 環境

BaseAppのABCI` {Check、Deliver} Tx() `と独自の` Simulate() `メソッドの実装は、基礎となる` runTx`メソッドを呼び出します。このメソッドは、最初にアンティハンドラーを実行し、次に `Msg`を実行します。ただし、[トランザクションのヒント](https://github.com/cosmos/cosmos-sdk/issues/9406)および[未使用のガスの払い戻し](https://github.com/cosmos/cosmos-sdk/issues.2150 )ユースケースでは、 `Msg`の実行後にカスタムロジックを実行する必要があります。現在、これを達成する方法はありません。

簡単な解決策は、ポスト `Msg`フックをBaseAppに追加することです。ただし、Cosmos SDKチームは、アプリケーションの配線を容易にする全体像も検討しています([#9181](https://github.com/cosmos/cosmos-sdk/discussions/9182))。これには、BaseAppの軽量化とモジュール性。

## 決定

BaseappのABCI` {Check、Deliver} Tx`実装と独自の `Simulate`メソッドを変換して、ミドルウェアベースの設計を使用することにしました。

```go
type Handler interface {
    CheckTx(ctx context.Context, req Request, checkReq RequestCheckTx) (Response, ResponseCheckTx, error)
    DeliverTx(ctx context.Context, req Request) (Response, error)
    SimulateTx(ctx context.Context, req Request (Response, error)
}

type Middleware func(Handler) Handler
```

ここでは、次のパラメーターと戻り値のタイプを定義しました。 

```go
type Request struct {
	Tx      sdk.Tx
	TxBytes []byte
}

type Response struct {
	GasWanted uint64
	GasUsed   uint64
	/.MsgResponses is an array containing each Msg service handler's response
	/.type, packed in an Any. This will get proto-serialized into the `Data` field
	/.in the ABCI Check/DeliverTx responses.
	MsgResponses []*codectypes.Any
	Log          string
	Events       []abci.Event
}

type RequestCheckTx struct {
	Type abci.CheckTxType
}

type ResponseCheckTx struct {
	Priority int64
}
```

CheckTxはメモリプールの優先順位付けに関連する個別のロジックを処理するため、そのシグネチャはDeliverTxおよびSimulateTxとは異なることに注意してください。

BaseAppは `tx.Handler`への参照を保持します: 

```go
type BaseApp  struct {
   ..other fields
    txHandler tx.Handler
}
```

BaseappのABCI` {Check、Deliver} Tx() `および` Simulate() `メソッドは、関連するパラメーターを使用して` app.txHandler。{Check、Deliver、Simulate} Tx() `を呼び出すだけです。 たとえば、 `DeliverTx`の場合: 

```go
func (app *BaseApp) DeliverTx(req abci.RequestDeliverTx) abci.ResponseDeliverTx {
    var abciRes abci.ResponseDeliverTx
	ctx := app.getContextForTx(runTxModeDeliver, req.Tx)
	res, err := app.txHandler.DeliverTx(ctx, tx.Request{TxBytes: req.Tx})
	if err != nil {
		abciRes = sdkerrors.ResponseDeliverTx(err, uint64(res.GasUsed), uint64(res.GasWanted), app.trace)
		return abciRes
	}

	abciRes, err = convertTxResponseToDeliverTx(res)
	if err != nil {
		return sdkerrors.ResponseDeliverTx(err, uint64(res.GasUsed), uint64(res.GasWanted), app.trace)
	}

	return abciRes
}

/.convertTxResponseToDeliverTx converts a tx.Response into a abci.ResponseDeliverTx.
func convertTxResponseToDeliverTx(txRes tx.Response) (abci.ResponseDeliverTx, error) {
	data, err := makeABCIData(txRes)
	if err != nil {
		return abci.ResponseDeliverTx{}, nil
	}

	return abci.ResponseDeliverTx{
		Data:   data,
		Log:    txRes.Log,
		Events: txRes.Events,
	}, nil
}

/.makeABCIData generates the Data field to be sent to ABCI Check/DeliverTx.
func makeABCIData(txRes tx.Response) ([]byte, error) {
	return proto.Marshal(&sdk.TxMsgData{MsgResponses: txRes.MsgResponses})
}
```

[BaseApp.CheckTx]と[BaseApp.Simulate]の実装は似ています。

`baseapp.txHandler`の3つのメソッドの実装は明らかに単一の関数ですが、モジュール化のために、ミドルウェアが単なる関数であるミドルウェアの組み合わせ設計を提案します。ミドルウェアは` tx.Handler`を受け入れ、別の `txを返します。 .Handler``は前のものにラップされています。

### ミドルウェアを実装する

実際、ミドルウェアはGo関数によって作成されます。この関数は、ミドルウェアに必要ないくつかのパラメーターをパラメーターとして受け取り、 `tx.Middleware`を返します。

たとえば、任意の `MyMiddleware`を作成するには、次のように実装できます。 

```go
/.myTxHandler is the tx.Handler of this middleware. Note that it holds a
/.reference to the next tx.Handler in the stack.
type myTxHandler struct {
   ..next is the next tx.Handler in the middleware stack.
    next tx.Handler
   ..some other fields that are relevant to the middleware can be added here
}

/.NewMyMiddleware returns a middleware that does this and that.
func NewMyMiddleware(arg1, arg2) tx.Middleware {
    return func (txh tx.Handler) tx.Handler {
        return myTxHandler{
            next: txh,
           ..optionally, set arg1, arg2... if they are needed in the middleware
        }
    }
}

/.Assert myTxHandler is a tx.Handler.
var _ tx.Handler = myTxHandler{}

func (h myTxHandler) CheckTx(ctx context.Context, req Request, checkReq RequestcheckTx) (Response, ResponseCheckTx, error) {
   ..CheckTx specific pre-processing logic

   ..run the next middleware
    res, checkRes, err := txh.next.CheckTx(ctx, req, checkReq)

   ..CheckTx specific post-processing logic

    return res, checkRes, err
}

func (h myTxHandler) DeliverTx(ctx context.Context, req Request) (Response, error) {
   ..DeliverTx specific pre-processing logic

   ..run the next middleware
    res, err := txh.next.DeliverTx(ctx, tx, req)

   ..DeliverTx specific post-processing logic

    return res, err
}

func (h myTxHandler) SimulateTx(ctx context.Context, req Request) (Response, error) {
   ..SimulateTx specific pre-processing logic

   ..run the next middleware
    res, err := txh.next.SimulateTx(ctx, tx, req)

   ..SimulateTx specific post-processing logic

    return res, err
}
```

### 複合ミドルウェア

BaseAppは `tx.Handler`への参照のみを保持しますが、この` tx.Handler`自体はミドルウェアスタックを使用して定義されます。 Cosmos SDKは、メッセージを実行する `RunMsgsTxHandler`と呼ばれる基本的な(つまり最も内側の)` tx.Handler`を公開します。

次に、アプリケーション開発者は、基本的な `tx.Handler`の上に複数のミドルウェアを組み合わせることができます。 前のセクションで説明したように、各ミドルウェアは、次のミドルウェアの周りで前処理ロジックと後処理ロジックを実行できます。 概念的には、例として、ミドルウェア `A`、` B`、および `C`とベース` tx.Handler` `H`を考えると、スタックは次のようになります。 

```
A.pre
    B.pre
        C.pre
            H # The base tx.handler, for example `RunMsgsTxHandler`
        C.post
    B.post
A.post
```

ミドルウェアをアセンブルするための `ComposeMiddlewares`関数を定義します。 基本的な処理プログラムを最初のパラメータとし、[外部から内部へ]の順にミドルウェアを使用します。 上記のスタックの場合、最終的な `tx.Handler`は次のとおりです。 

```go
txHandler := middleware.ComposeMiddlewares(H, A, B, C)
```

ミドルウェアは、[SetTxHandler]セッターを介してBaseAppに設定されます。 

```go
/.simapp/app.go

txHandler := middleware.ComposeMiddlewares(...)
app.SetTxHandler(txHandler)
```

アプリケーション開発者は、独自のミドルウェアを定義するか、CosmosSDKの `middleware.NewDefaultTxHandler()`から事前定義されたミドルウェアを使用できます。 

### ミドルウェアはCosmosSDKによって維持されています

アプリケーション開発者は選択したミドルウェアを定義して組み合わせることができますが、Cosmos SDKは、エコシステムの最も一般的なユースケースを満たすミドルウェアのセットを提供します。 これらのミドルウェアは次のとおりです。

| Middleware              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RunMsgsTxHandler        | This is the base `tx.Handler`. It replaces the old baseapp's `runMsgs`, and executes a transaction's `Msg`s.                                                                                                                                                                                                                                                                                                                                                                             |
| TxDecoderMiddleware     | This middleware takes in transaction raw bytes, and decodes them into a `sdk.Tx`. It replaces the `baseapp.txDecoder` field, so that BaseApp stays as thin as possible. Since most middlewares read the contents of the `sdk.Tx`, the TxDecoderMiddleware should be run first in the middelware stack.                                                                                                                                                                                   |
| {Antehandlers}          | Each antehandler is converted to its own middleware. These middlewares perform signature verification, fee deductions and other validations on the incoming transaction.                                                                                                                                                                                                                                                                                                                 |
| IndexEventsTxMiddleware | This is a simple middleware that chooses which events to index in Tendermint. Replaces `baseapp.indexEvents` (which unfortunately still exists in baseapp too, because it's used to index Begin/EndBlock events)                                                                                                                                                                                                                                                                         |
| RecoveryTxMiddleware    | This index recovers from panics. It replaces baseapp.runTx's panic recovery described in [ADR-022](./adr-022-custom-panic-handling.md).                                                                                                                                                                                                                                                                                                                                                  |
| GasTxMiddleware         | This replaces the [`Setup`](https://github.com/cosmos/cosmos-sdk/blob/v0.43.0/x/auth/ante/setup.go) Antehandler. It sets a GasMeter on sdk.Context. Note that before, GasMeter was set on sdk.Context inside the antehandlers, and there was some mess around the fact that antehandlers had their own panic recovery system so that the GasMeter could be read by baseapp's recovery system. Now, this mess is all removed: one middleware sets GasMeter, another one handles recovery. |

### アンティハンドラーとミドルウェアの類似点と相違点

ミドルウェアベースの設計は、[ADR-010](..adr-010-modular-antehandler.md)で説明されている既存のハンドラー設計に基づいています。 ADR-010の最終決定は[シンプルデコレータ]方式を採用することですが、ミドルウェアの設計は実際には他の[デコレータパターン](..adr-010-modular-antehandler.md#decorator-pattern)の提案と一致しています。 In [weave](https://github.com/iov-one/weave)も使用します。

#### アンティハンドラーとの類似点

-小さなモジュラーパーツをリンク/結合するように設計されています。
-` {Check、Deliver} Tx`と `Simulate`のコードの再利用を許可します。
-`app.go`で設定し、アプリ開発者が簡単にカスタマイズできます。
-順序は重要です。

#### アンティハンドラーとの違い

-アンチハンドラは `Msg`が実行される前に実行され、ミドルウェアは前後に実行できます。
-ミドルウェアメソッドは `{Check、Deliver、Simulate} Tx`に別のメソッドを使用し、アンティハンドラーは` Simulate bool`フラグを渡し、 `sdkCtx.Is {Check、Recheck} Tx()`フラグを使用してトランザクションモードです。
-ミドルウェアの設計により、各ミドルウェアは次のミドルウェアへの参照を保持でき、ハンドラーは `AnteHandle`メソッドで` next`パラメーターを渡します。
-ミドルウェアの設計ではGoの標準の `context.Context`を使用しますが、アンティハンドラーでは` sdk.Context`を使用します。

## 結果

### 下位互換性

このリファクタリングは一部のロジックをBaseAppからミドルウェアに移動するため、アプリケーション開発者向けのAPIに重大な変更が加えられます。特に、アプリケーション開発者は `app.go`にアンティハンドラチェーンを作成する必要はありませんが、ミドルウェアスタックを作成する必要があります。

```diff
- anteHandler, err := ante.NewAnteHandler(
-    ante.HandlerOptions{
-        AccountKeeper:   app.AccountKeeper,
-        BankKeeper:      app.BankKeeper,
-        SignModeHandler: encodingConfig.TxConfig.SignModeHandler(),
-        FeegrantKeeper:  app.FeeGrantKeeper,
-        SigGasConsumer:  ante.DefaultSigVerificationGasConsumer,
-    },
-)
+txHandler, err := authmiddleware.NewDefaultTxHandler(authmiddleware.TxHandlerOptions{
+    Debug:             app.Trace(),
+    IndexEvents:       indexEvents,
+    LegacyRouter:      app.legacyRouter,
+    MsgServiceRouter:  app.msgSvcRouter,
+    LegacyAnteHandler: anteHandler,
+    TxDecoder:         encodingConfig.TxConfig.TxDecoder,
+})
if err != nil {
    panic(err)
}
- app.SetAnteHandler(anteHandler)
+ app.SetTxHandler(txHandler)
```

CHANGELOGは、他の小さなAPIの重大な変更も提供します。いつものように、Cosmos SDKは、アプリケーション開発者向けのバージョン移行ドキュメントを提供します。

このADRは、ステートマシン、クライアント、またはCLIに破壊的な変更を導入しません。

### ポジティブ

-[メッセージ]の実行後にカスタムロジックを実行できるようにします。これにより、[ヒント](https://github.com/cosmos/cosmos-sdk/issues/9406)と[ガスの払い戻し](https://github.com/cosmos/cosmos-sdk/issues/2150)のユースケースが作成されます、他の場合もあります。
-BaseAppをより軽量にし、複雑なロジックを小さなモジュラーコンポーネントに渡します。
-異なるリターンタイプの `{Check、Deliver、Simulate} Tx`の個別のパス。これにより、読みやすさが向上し( `if sdkCtx.IsRecheckTx()&&！simulate {...}`を別のメソッドに置き換えます)、柔軟性が向上します(たとえば、 `ResponseCheckTx`で` priority`を返します)。
### ネガティブ

-一見すると、 `sdk.Context`と` tx`を指定してミドルウェアを実行した後に発生する状態の更新を理解するのは困難です。ミドルウェアは、その関数本体でネストされたミドルウェアをいくつでも呼び出すことができ、各ミドルウェアは、チェーン内の次のミドルウェアを呼び出す前に、前処理と後処理を実行する場合があります。したがって、ミドルウェアが何をしているのかを理解するには、チェーン内の他のミドルウェアも何をしているのかを理解する必要があります。ミドルウェアの順序は非常に重要です。これは理解するのが非常に難しくなる可能性があります。
-アプリケーション開発者向けのAPIへの画期的な変更。

### ニュートラル

中立的な結果はありません。

## さらなる議論

-[#9934](https://github.com/cosmos/cosmos-sdk/discussions/9934)BaseAppの他のABCIメソッドをミドルウェアに分解します。
-`sdk.Tx`インターフェースを `tx.Handler`メソッドシグネチャの具象protobufTxタイプに置き換えます。

## テストケース

新しいミドルウェアAPIを使用するように既存のbaseappテストとantehandlersテストを更新しますが、リグレッションの発生を避けるために同じテストケースとロジックを維持します。既存のCLIテストも変更されません。

新しいミドルウェアについては、単体テストを導入しました。ミドルウェアは小さいので、単体テストは非常に適しています。

## 参照する

-予備的なディスカッション:https://github.com/cosmos/cosmos-sdk/issues/9585
-実装:[#9920 BaseAppリファクタリング](https://github.com/cosmos/cosmos-sdk/pull/9920)および[#10028アンティハンドラーの移行](https://github.com/cosmos/cosmos-sdk.プル.10028) 