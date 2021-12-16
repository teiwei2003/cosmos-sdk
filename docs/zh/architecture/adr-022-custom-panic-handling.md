# ADR 022:自定义 BaseApp 恐慌处理

## 变更日志

- 2020 年 4 月 24 日:初稿
- 2021 年 9 月 14 日:被 ADR-045 取代

## 地位

由 ADR-045 取代

## 语境

BaseApp 的当前实现不允许开发人员在恐慌恢复期间编写自定义错误处理程序
[runTx()](https://github.com/cosmos/cosmos-sdk/blob/bad4ca75f58b182f600396ca350ad844c18fc80b/baseapp/baseapp.go#L539)
方法。我们认为这种方法可以更灵活，可以给 Cosmos SDK 用户更多的自定义选项，而无需
需要重写整个 BaseApp。 `sdk.ErrorOutOfGas` 错误处理还有一种特殊情况，就是这种情况
可能会以“标准”方式(中间件)与其他方式一起处理。

我们提出了中间件解决方案，可以帮助开发者实现以下案例:

* 添加外部日志记录(假设将报告发送到 [Sentry](https://sentry.io) 等外部服务)；
* 针对特定错误情况调用恐慌；

它还将使 `OutOfGas` case 和 `default` case 成为中间件之一。
`Default` case 将恢复对象包装为错误并将其记录下来([示例中间件实现](#Recovery-middleware))。

我们的项目有一个与区块链节点(智能合约虚拟机)一起运行的 sidecar 服务。它是
节点 <-> sidecar 连接对于 TX 处理保持稳定至关重要。所以当沟通中断时我们需要
使节点崩溃并在问题解决后重新启动它。该行为使节点的状态机执行
确定性的。由于 runTx 的 `defer()` 处理程序捕获了所有 keeper 恐慌，我们必须调整 BaseApp 代码
为了定制它。 
## Decision

### Design

#### Overview

我们建议使用一组可以自定义的中间件，而不是将自定义错误处理硬编码到 BaseApp 中
外部，并允许开发人员根据需要使用任意数量的自定义错误处理程序。 通过测试实现
可以在这里找到(https://github.com/cosmos/cosmos-sdk/pull/6053)。 

#### Implementation details

##### Recovery handler

添加了新的“RecoveryHandler”类型。 `recoveryObj` 输入参数是标准 Go 函数返回的对象
来自 `builtin` 包的 `recover()`。 

```go
type RecoveryHandler func(recoveryObj interface{}) error
```

处理程序应该输入断言(或其他方法)一个对象来定义是否应该处理对象。
如果输入对象不能被那个 `RecoveryHandler`(不是处理程序的目标类型)处理，则应该返回 `nil`。
如果处理了输入对象并且应该停止中间件链执行，则不应返回“nil”错误。

一个例子: 

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

这个例子中断了应用程序的执行，但它也可能丰富错误的上下文，如 `OutOfGas` 处理程序。

##### 恢复中间件

我们还添加了一个中间件类型(装饰器)。 该函数类型包装了 `RecoveryHandler` 并返回下一个中间件
执行链和处理程序的“错误”。 类型用于将实际的 `recovery()` 对象处理与中间件分开
链处理。 

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

这个例子中断了应用程序的执行，但它也可能丰富错误的上下文，如 `OutOfGas` 处理程序。

##### 恢复中间件

我们还添加了一个中间件类型(装饰器)。 该函数类型包装了 `RecoveryHandler` 并返回下一个中间件
执行链和处理程序的“错误”。 类型用于将实际的 `recovery()` 对象处理与中间件分开
链处理。 

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

`默认` 中间件示例: 

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

##### 恢复处理

中间件处理的基本链如下所示: 

```go
func processRecovery(recoveryObj interface{}, middleware recoveryMiddleware) error {
	if middleware == nil { return nil }

	next, err := middleware(recoveryObj)
	if err != nil { return err }
	if next == nil { return nil }

	return processRecovery(recoveryObj, next)
}
```

这样我们就可以创建一个从左到右执行的中间件链，最右边的中间件是
`default` 处理程序必须返回一个 `error`。

##### BaseApp 更改

`default` 中间件链必须存在于 `BaseApp` 对象中。 `Baseapp` 修改: 

```go
type BaseApp struct {
    // ...
    runTxRecoveryMiddleware recoveryMiddleware
}

func NewBaseApp(...) {
    // ...
    app.runTxRecoveryMiddleware = newDefaultRecoveryMiddleware()
}

func (app *BaseApp) runTx(...) {
    // ...
    defer func() {
        if r := recover(); r != nil {
            recoveryMW := newOutOfGasRecoveryMiddleware(gasWanted, ctx, app.runTxRecoveryMiddleware)
            err, result = processRecovery(r, recoveryMW), nil
        }

        gInfo = sdk.GasInfo{GasWanted: gasWanted, GasUsed: ctx.GasMeter().GasConsumed()}
    }()
    // ...
}
```

开发人员可以通过将 `AddRunTxRecoveryHandler` 作为 BaseApp 选项参数提供给 `NewBaseapp` 构造函数来添加他们的自定义 `RecoveryHandler`:

```go
func (app *BaseApp) AddRunTxRecoveryHandler(handlers ...RecoveryHandler) {
    for _, h := range handlers {
        app.runTxRecoveryMiddleware = newRecoveryMiddleware(h, app.runTxRecoveryMiddleware)
    }
}
```

此方法会将处理程序添加到现有链中。 

## Consequences

### Positive

- 基于 Cosmos SDK 的项目的开发人员可以将自定义恐慌处理程序添加到:
     * 为自定义恐慌源添加错误上下文(自定义保持器内部的恐慌)；
     * 发出`panic()`:将恢复对象传递给Tendermint 核心；
     * 其他必要的处理；
- 开发人员可以使用标准的 Cosmos SDK `BaseApp` 实现，而不是在他们的项目中重写它；
- 提议的解决方案不会破坏当前的“标准”`runTx()` 流程； 

### Negative

- 引入了对执行模型设计的更改。 

### Neutral

- `OutOfGas` 错误处理程序成为中间件之一；
- 默认的恐慌处理程序成为中间件之一； 

## References

- [PR-6053 with proposed solution](https://github.com/cosmos/cosmos-sdk/pull/6053)
- [Similar solution. ADR-010 Modular AnteHandler](https://github.com/cosmos/cosmos-sdk/blob/v0.38.3/docs/architecture/adr-010-modular-antehandler.md)
