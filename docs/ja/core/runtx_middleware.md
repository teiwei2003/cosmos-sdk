# RunTxリカバリミドルウェア

`BaseApp.runTx()`関数は、トランザクションの実行中に発生する可能性のあるGolangパニックを処理します。たとえば、キーパーは無効な状態に直面してパニックになります。
パニックのタイプに応じて、エラーログメッセージを出力するデフォルトのハンドラーなど、さまざまなハンドラーが使用されます。
リカバリミドルウェアは、CosmosSDKアプリケーション開発者向けのカスタム緊急リカバリを追加するために使用されます。

より多くのコンテキストは、対応する[ADR-022](../architecture/adr-022-custom-panic-handling.md)にあります。

実装は[recovery.go](../../baseapp/recovery.go)ファイルにあります。

## インターフェース 

```go
type RecoveryHandler func(recoveryObj interface{}) error
```

`recoveryObj`は、` buildin`Golangパッケージの `recover()`関数の戻り値です。

**合同:**

-`recoveryObj`が処理されず、次のリカバリミドルウェアに渡す必要がある場合、RecoveryHandlerは `nil`を返します。
-`recoveryObj`が処理されると、RecoveryHandlerはゼロ以外の `error`を返します。

## カスタムRecoveryHandler登録

`BaseApp.AddRunTxRecoveryHandler(handlers ...RecoveryHandler)`

BaseAppメソッドは、リカバリミドルウェアをデフォルトのリカバリチェーンに追加します。

## 例

特定のエラーが発生したときに[コンセンサス障害]チェーン状態を発行するとします。

パニック状態のモジュール管理者がいます: 

```go
func (k FooKeeper) Do(obj interface{}) {
    if obj == nil {
       //that shouldn't happen, we need to crash the app
        err := sdkErrors.Wrap(fooTypes.InternalError, "obj is nil")
        panic(err)
    }
}
```

デフォルトでは、パニックは回復し、エラーメッセージがログに出力されます。 この動作をオーバーライドするには、カスタムRecoveryHandlerを登録する必要があります。

```go
//Cosmos SDK application constructor
customHandler := func(recoveryObj interface{}) error {
    err, ok := recoveryObj.(error)
    if !ok {
        return nil
    }

    if fooTypes.InternalError.Is(err) {
        panic(fmt.Errorf("FooKeeper did panic with error: %w", err))
    }

    return nil
}

baseApp := baseapp.NewBaseApp(...)
baseApp.AddRunTxRecoveryHandler(customHandler)
```

## 次へ{hide}

[IBC](./../ibc/README.md)プロトコルを理解する{hide}
