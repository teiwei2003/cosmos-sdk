#ADR 022:カスタムBaseAppパニック処理

##変更ログ

-2020年4月24日:最初のドラフト
-2021年9月14日:ADR-045に置き換えられました

## 状態

ADR-045に置き換えられました

## 環境

BaseAppの現在の実装では、開発者はパニックリカバリ中にカスタムエラーハンドラーを作成できません。
[runTx()](https://github.com/cosmos/cosmos-sdk/blob/bad4ca75f58b182f600396ca350ad844c18fc80b/baseapp/baseapp.go#L539)
方法。この方法はより柔軟で、CosmosSDKユーザーにカスタマイズオプションを提供できると考えています。
BaseApp全体を書き直す必要があります。 `sdk.ErrorOutOfGas`エラー処理の特殊なケースがあります。これはケースです。
他の方法と一緒に[標準]の方法(ミドルウェア)で処理される場合があります。

開発者が次のケースを実装するのに役立つミドルウェアソリューションを提案しました。

*外部ログレコードを追加します(レポートが[Sentry](https://sentry.io)などの外部サービスに送信されると想定)。
*特定のエラー状態に対してパニックを引き起こします。

また、 `OutOfGas`ケースと` default`ケースをミドルウェアの1つにします。
`Default`の場合は、リカバリオブジェクトをエラーとしてラップし、それを記録します([Sample Middleware Implementation](#Recovery-middleware))。

私たちのプロジェクトには、ブロックチェーンノード(スマートコントラクト仮想マシン)で実行されるサイドカーサービスがあります。それは
ノード<->サイドカー接続は、TX処理の安定性に不可欠です。したがって、通信が中断された場合、
ノードをクラッシュさせ、問題が解決した後で再起動します。この動作により、ノードのステートマシンが実行されます
決定論的。 runTxの `defer()`ハンドラーがすべてのキーパーパニックをキャッチしたため、BaseAppコードを調整する必要があります
それをカスタマイズするために。
## 決断

### 設計

#### 概要

カスタムエラー処理をBaseAppにハードコーディングする代わりに、カスタマイズ可能なミドルウェアのセットを使用することをお勧めします
外部であり、開発者は必要な数のカスタムエラーハンドラーを使用できます。テストによって達成
ここ(https://github.com/cosmos/cosmos-sdk/pull/6053)にあります。

####実装の詳細

#####リカバリハンドラ

新しい[RecoveryHandler]タイプが追加されました。 `recoveryObj`入力パラメータは、標準のGo関数によって返されるオブジェクトです。
`builtin`パッケージから` recover() `。 

```go
type RecoveryHandler func(recoveryObj interface{}) error
```

ハンドラーは、オブジェクトにアサーション(または他のメソッド)を入力して、オブジェクトを処理する必要があるかどうかを定義する必要があります。
入力オブジェクトがその `RecoveryHandler`(ハンドラーのターゲットタイプではない)で処理できない場合は、` nil`を返す必要があります。
入力オブジェクトが処理され、ミドルウェアチェーンの実行を停止する必要がある場合、[nil]エラーは返されません。

一例: 

```go
func exampleErrHandler(recoveryObj interface{}) error {
    err, ok := recoveryObj.(error)
    if !ok { return nil }

    if someSpecificError.Is(err) {
        panic(customPanicMsg)
    } else {
        return nil
    }
}
```

この例では、アプリケーションの実行が中断されますが、 `OutOfGas`ハンドラーなどのエラーコンテキストが強化される場合もあります。

##### リカバリミドルウェア

ミドルウェアタイプ(デコレータ)も追加しました。 この関数型は `RecoveryHandler`をラップし、次のミドルウェアを返します
実行チェーンとハンドラーの[エラー]。 タイプは、 `recovery()`オブジェクトの実際の処理をミドルウェアから分離するために使用されます
チェーン処理。 
```go
type recoveryMiddleware func(recoveryObj interface{}) (recoveryMiddleware, error)

func newRecoveryMiddleware(handler RecoveryHandler, next recoveryMiddleware) recoveryMiddleware {
    return func(recoveryObj interface{}) (recoveryMiddleware, error) {
        if err := handler(recoveryObj); err != nil {
            return nil, err
        }
        return next, nil
    }
}
```

この例では、アプリケーションの実行が中断されますが、 `OutOfGas`ハンドラーなどのエラーコンテキストが強化される場合もあります。

##### リカバリミドルウェア

ミドルウェアタイプ(デコレータ)も追加しました。 この関数型は `RecoveryHandler`をラップし、次のミドルウェアを返します
実行チェーンとハンドラーの[エラー]。 タイプは、 `recovery()`オブジェクトの実際の処理をミドルウェアから分離するために使用されます
チェーン処理。 

```go
func newOutOfGasRecoveryMiddleware(gasWanted uint64, ctx sdk.Context, next recoveryMiddleware) recoveryMiddleware {
    handler := func(recoveryObj interface{}) error {
        err, ok := recoveryObj.(sdk.ErrorOutOfGas)
        if !ok { return nil }

        return sdkerrors.Wrap(
            sdkerrors.ErrOutOfGas, fmt.Sprintf(
                "out of gas in location: %v; gasWanted: %d, gasUsed: %d", err.Descriptor, gasWanted, ctx.GasMeter().GasConsumed(),
            ),
        )
    }

    return newRecoveryMiddleware(handler, next)
}
```

`default`ミドルウェアの例: 

```go
func newDefaultRecoveryMiddleware() recoveryMiddleware {
    handler := func(recoveryObj interface{}) error {
        return sdkerrors.Wrap(
            sdkerrors.ErrPanic, fmt.Sprintf("recovered: %v\nstack:\n%v", recoveryObj, string(debug.Stack())),
        )
    }

    return newRecoveryMiddleware(handler, nil)
}
```

##### 回復処理

ミドルウェア処理の基本的なチェーンは次のとおりです。 

```go
func processRecovery(recoveryObj interface{}, middleware recoveryMiddleware) error {
	if middleware == nil { return nil }

	next, err := middleware(recoveryObj)
	if err != nil { return err }
	if next == nil { return nil }

	return processRecovery(recoveryObj, next)
}
```

このようにして、左から右に実行されるミドルウェアチェーンを作成できます。右端のミドルウェアは
`default`ハンドラーは` error`を返す必要があります。

##### BaseAppの変更

`default`ミドルウェアチェーンは` BaseApp`オブジェクトに存在する必要があります。 `Baseapp`の変更: 

```go
type BaseApp struct {
   .....
    runTxRecoveryMiddleware recoveryMiddleware
}

func NewBaseApp(...) {
   .....
    app.runTxRecoveryMiddleware = newDefaultRecoveryMiddleware()
}

func (app *BaseApp) runTx(...) {
   .....
    defer func() {
        if r := recover(); r != nil {
            recoveryMW := newOutOfGasRecoveryMiddleware(gasWanted, ctx, app.runTxRecoveryMiddleware)
            err, result = processRecovery(r, recoveryMW), nil
        }

        gInfo = sdk.GasInfo{GasWanted: gasWanted, GasUsed: ctx.GasMeter().GasConsumed()}
    }()
   .....
}
```

開発者は、BaseAppオプションパラメーターとして `AddRunTxRecoveryHandler`を` NewBaseapp`コンストラクターに提供することにより、カスタムの `RecoveryHandler`を追加できます。 

```go
func (app *BaseApp) AddRunTxRecoveryHandler(handlers ...RecoveryHandler) {
    for _, h := range handlers {
        app.runTxRecoveryMiddleware = newRecoveryMiddleware(h, app.runTxRecoveryMiddleware)
    }
}
```

このメソッドは、ハンドラーを既存のチェーンに追加します。 

## 結果

### ポジティブ

-Cosmos SDKベースのプロジェクトの開発者は、カスタムパニックハンドラーを次の場所に追加できます。
      *カスタムパニックソース(カスタムキーパー内のパニック)にエラーコンテキストを追加します。
      * `panic()`を発行します:リカバリオブジェクトをTendermintコアに渡します。
      *その他の必要な処理。
-開発者は、プロジェクトで書き直す代わりに、標準のCosmos SDK`BaseApp`実装を使用できます。
-提案されたソリューションは、現在の[標準]の `runTx()`プロセスを中断しません。

### ネガティブ

-実行モデルの設計に変更が導入されました。

### ニュートラル

-`OutOfGas`エラーハンドラーはミドルウェアの1つになります。
-デフォルトのパニックハンドラーはミドルウェアの1つになります。

## 参照

-[提案されたソリューションを使用したPR-6053](https://github.com/cosmos/cosmos-sdk/pull/6053)
-[同様のソリューション。ADR-010モジュラーAnteHandler](https://github.com/cosmos/cosmos-sdk/blob/v0.38.3/docs/architecture/adr-010-modular-antehandler.md) 