# RunTx 恢复中间件

`BaseApp.runTx()` 函数处理在事务执行过程中可能发生的 Golang 恐慌，例如，keeper 面临无效状态并出现恐慌。
根据恐慌类型使用不同的处理程序，例如默认的一个打印错误日志消息。
恢复中间件用于为 Cosmos SDK 应用程序开发人员添加自定义紧急恢复。

更多上下文可以在相应的 [ADR-022](../architecture/adr-022-custom-panic-handling.md) 中找到。

可以在 [recovery.go](../../baseapp/recovery.go) 文件中找到实现。

## 界面 

```go
type RecoveryHandler func(recoveryObj interface{}) error
```

`recoveryObj` 是 `buildin` Golang 包中 `recover()` 函数的返回值。

**合同:**

- 如果 `recoveryObj` 没有被处理并且应该传递给下一个恢复中间件，RecoveryHandler 返回 `nil`；
- 如果处理了 `recoveryObj`，RecoveryHandler 返回一个非零的 `error`；

##自定义RecoveryHandler注册

`BaseApp.AddRunTxRecoveryHandler(handlers ...RecoveryHandler)`

BaseApp 方法将恢复中间件添加到默认恢复链中。

## 例子

假设我们想要在发生某些特定错误时发出“共识失败”链状态。

我们有一个恐慌的模块管理员: 

```go
func (k FooKeeper) Do(obj interface{}) {
    if obj == nil {
        // that shouldn't happen, we need to crash the app
        err := sdkErrors.Wrap(fooTypes.InternalError, "obj is nil")
        panic(err)
    }
}
```

默认情况下，恐慌会被恢复，错误信息将被打印到日志中。 要覆盖该行为，我们应该注册一个自定义 RecoveryHandler: 

```go
// Cosmos SDK application constructor
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

## 下一个 {hide}

了解 [IBC](./../ibc/README.md) 协议 {hide} 
