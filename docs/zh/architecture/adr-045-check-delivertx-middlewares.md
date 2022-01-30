# ADR 045:BaseApp `{Check,Deliver}Tx` 作为中间件

## 变更日志

- 20.08.2021:初稿。
- 07.12.2021: 更新 `tx.Handler` 接口 ([\#10693](https://github.com/cosmos/cosmos-sdk/pull/10693))。

## 地位

公认

## 摘要

此 ADR 用基于中间件的设计替换了当前的 BaseApp `runTx` 和 antehandlers 设计。

## 语境

BaseApp 的 ABCI `{Check,Deliver}Tx()` 的实现和它自己的 `Simulate()` 方法调用了底层的 `runTx` 方法，它首先运行 antehandlers，然后执行 `Msg`s。但是，[交易提示](https://github.com/cosmos/cosmos-sdk/issues/9406)和[退还未使用的gas](https://github.com/cosmos/cosmos-sdk/issues/2150 ) 用例要求在执行 `Msg` 后运行自定义逻辑。目前没有办法实现这一点。

一个简单的解决方案是向 BaseApp 添加 post-`Msg` 钩子。然而，Cosmos SDK 团队同时考虑使应用程序布线更简单的大局([#9181](https://github.com/cosmos/cosmos-sdk/discussions/9182))，其中包括使 BaseApp 更轻量级和模块化。

## 决定

我们决定将 Baseapp 的 ABCI `{Check,Deliver}Tx` 实现及其自己的 `Simulate` 方法转换为使用基于中间件的设计。

以下两个接口是中间件设计的基础，并在 `types/tx` 中定义: 

```go
type Handler interface {
    CheckTx(ctx context.Context, req Request, checkReq RequestCheckTx) (Response, ResponseCheckTx, error)
    DeliverTx(ctx context.Context, req Request) (Response, error)
    SimulateTx(ctx context.Context, req Request (Response, error)
}

type Middleware func(Handler) Handler
```

我们在这里定义了以下参数和返回类型: 

```go
type Request struct {
	Tx      sdk.Tx
	TxBytes []byte
}

type Response struct {
	GasWanted uint64
	GasUsed   uint64
	//MsgResponses is an array containing each Msg service handler's response
	//type, packed in an Any. This will get proto-serialized into the `Data` field
	//in the ABCI Check/DeliverTx responses.
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

请注意，由于 CheckTx 处理与内存池优先化相关的单独逻辑，因此其签名与 DeliverTx 和 SimulateTx 不同。 

BaseApp 持有对 `tx.Handler` 的引用: 

```go
type BaseApp  struct {
   //other fields
    txHandler tx.Handler
}
```

Baseapp 的 ABCI `{Check,Deliver}Tx()` 和 `Simulate()` 方法简单地调用带有相关参数的 `app.txHandler.{Check,Deliver,Simulate}Tx()`。 例如，对于`DeliverTx`: 

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

//convertTxResponseToDeliverTx converts a tx.Response into a abci.ResponseDeliverTx.
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

//makeABCIData generates the Data field to be sent to ABCI Check/DeliverTx.
func makeABCIData(txRes tx.Response) ([]byte, error) {
	return proto.Marshal(&sdk.TxMsgData{MsgResponses: txRes.MsgResponses})
}
```

“BaseApp.CheckTx”和“BaseApp.Simulate”的实现类似。

`baseapp.txHandler` 的三个方法的实现显然可以是单体函数，但是为了模块化我们提出了中间件组合设计，其中中间件只是一个函数，它接受一个 `tx.Handler`，并返回另一个 `tx.Handler` ` 包裹在前一个。

### 实现中间件

实际上，中间件是由 Go 函数创建的，该函数将中间件所需的一些参数作为参数，并返回一个 `tx.Middleware`。

例如，为了创建一个任意的`MyMiddleware`，我们可以实现: 

```go
//myTxHandler is the tx.Handler of this middleware. Note that it holds a
//reference to the next tx.Handler in the stack.
type myTxHandler struct {
   //next is the next tx.Handler in the middleware stack.
    next tx.Handler
   //some other fields that are relevant to the middleware can be added here
}

//NewMyMiddleware returns a middleware that does this and that.
func NewMyMiddleware(arg1, arg2) tx.Middleware {
    return func (txh tx.Handler) tx.Handler {
        return myTxHandler{
            next: txh,
           //optionally, set arg1, arg2... if they are needed in the middleware
        }
    }
}

//Assert myTxHandler is a tx.Handler.
var _ tx.Handler = myTxHandler{}

func (h myTxHandler) CheckTx(ctx context.Context, req Request, checkReq RequestcheckTx) (Response, ResponseCheckTx, error) {
   //CheckTx specific pre-processing logic

   //run the next middleware
    res, checkRes, err := txh.next.CheckTx(ctx, req, checkReq)

   //CheckTx specific post-processing logic

    return res, checkRes, err
}

func (h myTxHandler) DeliverTx(ctx context.Context, req Request) (Response, error) {
   //DeliverTx specific pre-processing logic

   //run the next middleware
    res, err := txh.next.DeliverTx(ctx, tx, req)

   //DeliverTx specific post-processing logic

    return res, err
}

func (h myTxHandler) SimulateTx(ctx context.Context, req Request) (Response, error) {
   //SimulateTx specific pre-processing logic

   //run the next middleware
    res, err := txh.next.SimulateTx(ctx, tx, req)

   //SimulateTx specific post-processing logic

    return res, err
}
```

### 组合中间件

虽然 BaseApp 只是持有对 `tx.Handler` 的引用，但这个 `tx.Handler` 本身是使用中间件堆栈定义的。 Cosmos SDK 公开了一个基础的(即最里面的)`tx.Handler`，称为 `RunMsgsTxHandler`，它执行消息。

然后，应用程序开发人员可以在基础 `tx.Handler` 之上组合多个中间件。 每个中间件都可以围绕其下一个中间件运行预处理和后处理逻辑，如上一节所述。 从概念上讲，作为一个例子，给定中间件 `A`、`B` 和 `C` 以及基础 `tx.Handler` `H`，堆栈看起来像: 

```
A.pre
    B.pre
        C.pre
            H # The base tx.handler, for example `RunMsgsTxHandler`
        C.post
    B.post
A.post
```

我们定义了一个 `ComposeMiddlewares` 函数来组合中间件。 它将基本处理程序作为第一个参数，并按照“从外到内”的顺序使用中间件。 对于上面的堆栈，最终的 `tx.Handler` 是: 

```go
txHandler := middleware.ComposeMiddlewares(H, A, B, C)
```

中间件通过其“SetTxHandler”设置器在 BaseApp 中设置: 

```go
//simapp/app.go

txHandler := middleware.ComposeMiddlewares(...)
app.SetTxHandler(txHandler)
```

应用程序开发人员可以定义自己的中间件，或使用 Cosmos SDK 的来自 `middleware.NewDefaultTxHandler()` 的预定义中间件。

### 中间件由 Cosmos SDK 维护

虽然应用程序开发人员可以定义和组合他们选择的中间件，但 Cosmos SDK 提供了一组满足生态系统最常见用例的中间件。 这些中间件是:

| Middleware              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RunMsgsTxHandler        | This is the base `tx.Handler`. It replaces the old baseapp's `runMsgs`, and executes a transaction's `Msg`s.                                                                                                                                                                                                                                                                                                                                                                             |
| TxDecoderMiddleware     | This middleware takes in transaction raw bytes, and decodes them into a `sdk.Tx`. It replaces the `baseapp.txDecoder` field, so that BaseApp stays as thin as possible. Since most middlewares read the contents of the `sdk.Tx`, the TxDecoderMiddleware should be run first in the middelware stack.                                                                                                                                                                                   |
| {Antehandlers}          | Each antehandler is converted to its own middleware. These middlewares perform signature verification, fee deductions and other validations on the incoming transaction.                                                                                                                                                                                                                                                                                                                 |
| IndexEventsTxMiddleware | This is a simple middleware that chooses which events to index in Tendermint. Replaces `baseapp.indexEvents` (which unfortunately still exists in baseapp too, because it's used to index Begin/EndBlock events)                                                                                                                                                                                                                                                                         |
| RecoveryTxMiddleware    | This index recovers from panics. It replaces baseapp.runTx's panic recovery described in [ADR-022](./adr-022-custom-panic-handling.md).                                                                                                                                                                                                                                                                                                                                                  |
| GasTxMiddleware         | This replaces the [`Setup`](https://github.com/cosmos/cosmos-sdk/blob/v0.43.0/x/auth/ante/setup.go) Antehandler. It sets a GasMeter on sdk.Context. Note that before, GasMeter was set on sdk.Context inside the antehandlers, and there was some mess around the fact that antehandlers had their own panic recovery system so that the GasMeter could be read by baseapp's recovery system. Now, this mess is all removed: one middleware sets GasMeter, another one handles recovery. |

### Antehandlers 和 Middlewares 之间的异同

基于中间件的设计建立在 [ADR-010](./adr-010-modular-antehandler.md) 中描述的现有处理程序设计之上。尽管 ADR-010 的最终决定是采用“简单装饰器”方法，但中间件设计实际上与其他 [装饰器模式](./adr-010-modular-antehandler.md#decorator-pattern ) 提案，也用于 [weave](https://github.com/iov-one/weave)。

####与Antehandlers的相似之处

- 设计为链接/组合小型模块化部件。
- 允许对`{Check,Deliver}Tx` 和`Simulate` 进行代码重用。
- 在`app.go` 中设置，并由应用程序开发人员轻松定制。
- 顺序很重要。

####与Antehandlers的差异

- Antehandlers 在 `Msg` 执行之前运行，而中间件可以在之前和之后运行。
- 中间件方法对 `{Check,Deliver,Simulate}Tx` 使用单独的方法，而 antehandlers 传递一个 `simulate bool` 标志并使用 `sdkCtx.Is{Check,Recheck}Tx()` 标志来确定在哪个我们是交易模式。
- 中间件设计让每个中间件都持有对下一个中间件的引用，而处理程序在 `AnteHandle` 方法中传递一个 `next` 参数。
- 中间件设计使用 Go 的标准 `context.Context`，而 antehandlers 使用 `sdk.Context`。 

## Consequences

### Backwards Compatibility

由于此重构将一些逻辑从 BaseApp 移到中间件中，因此它为应用程序开发人员引入了 API 破坏性更改。 最值得注意的是，应用程序开发人员不需要在 `app.go` 中创建一个 antehandler 链，而是需要创建一个中间件堆栈: 

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

CHANGELOG 中还将提供其他更小的 API 中断更改。 像往常一样，Cosmos SDK 将为应用程序开发人员提供版本迁移文档。

此 ADR 不会引入任何状态机、客户端或 CLI 破坏性更改。 

### Positive

- 允许自定义逻辑在“消息”执行之后运行。 这使 [tips](https://github.com/cosmos/cosmos-sdk/issues/9406) 和 [gas 退款](https://github.com/cosmos/cosmos-sdk/issues/2150) 使用 案件，可能还有其他案件。
- 使 BaseApp 更加轻量级，将复杂的逻辑交给小型模块化组件。
- 不同返回类型的`{Check,Deliver,Simulate}Tx` 的单独路径。 这允许提高可读性(用单独的方法替换 `if sdkCtx.IsRecheckTx() && !simulate {...}`)和更大的灵活性(例如在 `ResponseCheckTx` 中返回 `priority`)。
### Negative

- 乍一看很难理解在给定 `sdk.Context` 和 `tx` 的情况下中间件运行后会发生的状态更新。 中间件可以在其函数体内调用任意数量的嵌套中间件，每个中间件都可能在调用链上的下一个中间件之前进行一些预处理和后处理。 因此，要了解中间件在做什么，还必须了解链上其他中间件也在做什么，中间件的顺序很重要。 这可能会变得非常难以理解。
- 应用程序开发人员的 API 突破性更改。 

### Neutral

No neutral consequences.

## 进一步讨论

- [#9934](https://github.com/cosmos/cosmos-sdk/discussions/9934) 将 BaseApp 的其他 ABCI 方法分解为中间件。
- 用`tx.Handler` 方法签名中的具体protobuf Tx 类型替换`sdk.Tx` 接口。

## 测试用例

我们更新现有的 baseapp 和 antehandlers 测试以使用新的中间件 API，但保持相同的测试用例和逻辑，以避免引入回归。 现有的 CLI 测试也将保持不变。

对于新的中间件，我们引入了单元测试。 由于中间件很小，因此单元测试非常适合。

## 参考

- 初步讨论:https://github.com/cosmos/cosmos-sdk/issues/9585
- 实现:[#9920 BaseApp 重构](https://github.com/cosmos/cosmos-sdk/pull/9920) 和 [#10028 Antehandlers 迁移](https://github.com/cosmos/cosmos-sdk/拉/10028) 