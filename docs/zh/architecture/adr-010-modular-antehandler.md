# ADR 010:模块化 AnteHandler

## 变更日志

- 2019 年 8 月 31 日:初稿
- 2021 年 9 月 14 日:被 ADR-045 取代

## 地位

由 ADR-045 取代

## 语境

当前的 AnteHandler 设计允许用户使用 `x/auth` 中提供的默认 AnteHandler 或从头开始构建自己的 AnteHandler。理想情况下，AnteHandler 功能被拆分为多个模块化函数，这些函数可以与自定义 ante-functions 一起链接在一起，以便用户在想要实现自定义行为时不必重写常见的 antehandler 逻辑。

例如，假设用户想要实现一些自定义签名验证逻辑。在当前的代码库中，用户必须从头开始编写他们自己的 Antehandler，主要是重新实现大部分相同的代码，然后在 baseapp 中设置他们自己的自定义、整体式 antehandler。相反，我们希望允许用户在必要时指定自定义行为，并以尽可能模块化和灵活的方式将它们与默认的 ante-handler 功能相结合。

## 提案

### 每模块 AnteHandler

一种方法是使用 [ModuleManager](https://godoc.org/github.com/cosmos/cosmos-sdk/types/module) 并让每个模块实现自己的 antehandler，如果它需要自定义 antehandler 逻辑。然后 ModuleManager 可以按照与 BeginBlockers 和 EndBlockers 的顺序相同的方式在 AnteHandler 顺序中传递。 ModuleManager 返回一个 AnteHandler 函数，该函数将接收一个 tx 并以指定的顺序运行每个模块的 `AnteHandle`。模块管理器的 AnteHandler 被设置为 baseapp 的 AnteHandler。

优点:

1. 实现简单
2.利用现有的ModuleManager架构

缺点:

1. 改进了粒度，但仍然无法获得比每个模块更精细的粒度。例如如果 auth 的 `AnteHandle` 函数负责验证备忘录和签名，则用户不能在保留 auth 的其余 `AnteHandle` 功能的同时交换签名检查功能。
2. 模块AnteHandler 一个接一个运行。一个 AnteHandler 无法包装或“装饰”另一个。

### 装饰模式

[weave 项目](https://github.com/iov-one/weave) 通过使用装饰器模式实现了 AnteHandler 模块化。界面设计如下: 

```go
// Decorator wraps a Handler to provide common functionality
// like authentication, or fee-handling, to many Handlers
type Decorator interface {
	Check(ctx Context, store KVStore, tx Tx, next Checker) (*CheckResult, error)
	Deliver(ctx Context, store KVStore, tx Tx, next Deliverer) (*DeliverResult, error)
}
```

每个装饰器的工作方式类似于模块化的 Cosmos SDK 前处理函数，但它可以接收一个 `next` 参数，该参数可能是另一个装饰器或一个 Handler(不接收下一个参数)。 这些装饰器可以链接在一起，一个装饰器作为链中前一个装饰器的“next”参数传入。 该链以路由器结束，该路由器可以接受 tx 并路由到适当的 msg 处理程序。

这种方法的一个主要好处是一个装饰器可以将其内部逻辑包装在下一个检查器/交付器周围。 编织装饰器可以执行以下操作:

```go
// Example Decorator's Deliver function
func (example Decorator) Deliver(ctx Context, store KVStore, tx Tx, next Deliverer) {
    // Do some pre-processing logic

    res, err := next.Deliver(ctx, store, tx)

    // Do some post-processing logic given the result and error
}
```

优点:

1. Weave 装饰器可以包裹链中的下一个装饰器/处理程序。在某些情况下，预处理和后处理的能力可能很有用。
2. 提供上述解决方案中无法实现的嵌套模块化结构，同时还允许像上述解决方案一样采用线性的一个接一个的结构。

缺点:

1. 乍一看很难理解在给定 `ctx`、`store` 和 `tx` 的装饰器运行后会发生的状态更新。一个装饰器可以在其函数体内调用任意数量的嵌套装饰器，每个装饰器都可能在调用链上的下一个装饰器之前进行一些预处理和后处理。因此，要了解装饰器在做什么，还必须了解链条上的其他装饰器也在做什么。这可能会变得非常难以理解。一种线性的、一个接一个的方法虽然不那么强大，但可能更容易推理。

### 链式微函数

Weave 方法的好处是装饰器可以非常简洁，当链接在一起时可以实现最大的可定制性。然而，嵌套结构会变得非常复杂，因此难以推理。

另一种方法是将 AnteHandler 功能拆分为范围紧密的“微功能”，同时保留来自 ModuleManager 方法的一个接一个排序。

然后我们可以有一种方法来链接这些微函数，以便它们一个接一个地运行。模块可以定义多个 ante 微功能，然后还提供一个默认的每模块 AnteHandler，为这些微功能实现默认的建议顺序。

用户只需使用 ModuleManager 即可轻松订购 AnteHandler。 ModuleManager 将接收一个 AnteHandler 列表并返回一个单独的 AnteHandler，它按照提供的列表的顺序运行每个 AnteHandler。如果用户对每个模块的默认顺序感到满意，这就像提供每个模块的 antehandler 列表一样简单(与 BeginBlocker 和 EndBlocker 完全相同)。

但是，如果用户希望更改顺序或以任何方式添加、修改或删除前微功能；他们总是可以定义自己的前微函数并将它们显式添加到传递给模块管理器的列表中。

#### 默认工作流

这是用户选择不制作任何自定义微功能的 AnteHandler 的示例。

##### Cosmos SDK 代码 

```go
// Chains together a list of AnteHandler micro-functions that get run one after the other.
// Returned AnteHandler will abort on first error.
func Chainer(order []AnteHandler) AnteHandler {
    return func(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
        for _, ante := range order {
            ctx, err := ante(ctx, tx, simulate)
            if err != nil {
                return ctx, err
            }
        }
        return ctx, err
    }
}
```

```go
// AnteHandler micro-function to verify signatures
func VerifySignatures(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
    // verify signatures
    // Returns InvalidSignature Result and abort=true if sigs invalid
    // Return OK result and abort=false if sigs are valid
}

// AnteHandler micro-function to validate memo
func ValidateMemo(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
    // validate memo
}

// Auth defines its own default ante-handler by chaining its micro-functions in a recommended order
AuthModuleAnteHandler := Chainer([]AnteHandler{VerifySignatures, ValidateMemo})
```

```go
// Distribution micro-function to deduct fees from tx
func DeductFees(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
    // Deduct fees from tx
    // Abort if insufficient funds in account to pay for fees
}

// Distribution micro-function to check if fees > mempool parameter
func CheckMempoolFees(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
    // If CheckTx: Abort if the fees are less than the mempool's minFee parameter
}

// Distribution defines its own default ante-handler by chaining its micro-functions in a recommended order
DistrModuleAnteHandler := Chainer([]AnteHandler{CheckMempoolFees, DeductFees})
```

```go
type ModuleManager struct {
    // other fields
    AnteHandlerOrder []AnteHandler
}

func (mm ModuleManager) GetAnteHandler() AnteHandler {
    retun Chainer(mm.AnteHandlerOrder)
}
```

##### User Code

```go
// Note: Since user is not making any custom modifications, we can just SetAnteHandlerOrder with the default AnteHandlers provided by each module in our preferred order
moduleManager.SetAnteHandlerOrder([]AnteHandler(AuthModuleAnteHandler, DistrModuleAnteHandler))

app.SetAnteHandler(mm.GetAnteHandler())
```

#### 自定义工作流

这是针对想要实现自定义处理程序逻辑的用户的示例工作流。 在这个例子中，用户想要实现自定义签名验证并更改 antehandler 的顺序，以便在签名验证之前运行验证备忘录。

##### 用户代码 

```go
// User can implement their own custom signature verification antehandler micro-function
func CustomSigVerify(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
    // do some custom signature verification logic
}
```

```go
// Micro-functions allow users to change order of when they get executed, and swap out default ante-functionality with their own custom logic.
// Note that users can still chain the default distribution module handler, and auth micro-function along with their custom ante function
moduleManager.SetAnteHandlerOrder([]AnteHandler(ValidateMemo, CustomSigVerify, DistrModuleAnteHandler))
```

优点:

1. 允许 ante 功能尽可能模块化。
2.对于不需要自定义前置功能的用户，前置处理程序的工作方式与ModuleManager中BeginBlock和EndBlock的工作方式几乎没有区别。
3.还是很容易理解的

缺点:

1. 不能像 Weave 那样用装饰器包装 antehandlers。

### 简单的装饰器

这种方法从 Weave 的装饰器设计中汲取灵感，同时尝试最大限度地减少对 Cosmos SDK 的重大更改并最大限度地简化。与 Weave 装饰器一样，这种方法允许一个 `AnteDecorator` 包装下一个 AnteHandler 以对结果进行预处理和后处理。这很有用，因为装饰器可以在 AnteHandler 返回后进行延迟/清理以及预先执行一些设置。与 Weave 装饰器不同，这些 `AnteDecorator` 函数只能包装 AnteHandler 而不是整个处理程序执行路径。这是故意的，因为我们希望来自不同模块的装饰器对“tx”执行身份验证/验证。但是，我们不希望装饰器能够包装和修改 `MsgHandler` 的结果。

此外，这种方法不会破坏任何核心 Cosmos SDK API。由于我们保留了 AnteHandler 的概念，并且仍然在 baseapp 中设置了单个 AnteHandler，因此装饰器只是一种可供需要更多自定义的用户使用的附加方法。模块的 API(即`x/auth`)可能会因这种方法而中断，但核心 API 保持不变。

允许装饰器接口可以链接在一起以创建 Cosmos SDK AnteHandler。

这允许用户在自己实现 AnteHandler 和在基本应用程序中设置它之间进行选择，或者使用装饰器模式按照他们希望的顺序将他们的自定义装饰器与 Cosmos SDK 提供的装饰器链接起来。 

```go
// An AnteDecorator wraps an AnteHandler, and can do pre- and post-processing on the next AnteHandler
type AnteDecorator interface {
    AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error)
}
```

```go
// ChainAnteDecorators will recursively link all of the AnteDecorators in the chain and return a final AnteHandler function
// This is done to preserve the ability to set a single AnteHandler function in the baseapp.
func ChainAnteDecorators(chain ...AnteDecorator) AnteHandler {
    if len(chain) == 1 {
        return func(ctx Context, tx Tx, simulate bool) {
            chain[0].AnteHandle(ctx, tx, simulate, nil)
        }
    }
    return func(ctx Context, tx Tx, simulate bool) {
        chain[0].AnteHandle(ctx, tx, simulate, ChainAnteDecorators(chain[1:]))
    }
}
```

#### 示例代码

定义 AnteDecorator 函数

```go
// Setup GasMeter, catch OutOfGasPanic and handle appropriately
type SetUpContextDecorator struct{}

func (sud SetUpContextDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
    ctx.GasMeter = NewGasMeter(tx.Gas)

    defer func() {
        // recover from OutOfGas panic and handle appropriately
    }

    return next(ctx, tx, simulate)
}

// Signature Verification decorator. Verify Signatures and move on
type SigVerifyDecorator struct{}

func (svd SigVerifyDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
    // verify sigs. Return error if invalid

    // call next antehandler if sigs ok
    return next(ctx, tx, simulate)
}

// User-defined Decorator. Can choose to pre- and post-process on AnteHandler
type UserDefinedDecorator struct{
    // custom fields
}

func (udd UserDefinedDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
    // pre-processing logic

    ctx, err = next(ctx, tx, simulate)

    // post-processing logic
}
```

链接 AnteDecorators 以创建最终的 AnteHandler。 在 baseapp 中设置这个 AnteHandler。

```go
// Create final antehandler by chaining the decorators together
antehandler := ChainAnteDecorators(NewSetUpContextDecorator(), NewSigVerifyDecorator(), NewUserDefinedDecorator())

// Set chained Antehandler in the baseapp
bapp.SetAnteHandler(antehandler)
```

优点:

1. 允许一个装饰器对下一个 AnteHandler 进行预处理和后处理，类似于 Weave 设计。
2. 不需要破坏baseapp API。 如果用户愿意，他们仍然可以设置单个 AnteHandler。

缺点:

1. 装饰器模式可能有一个难以理解的深层嵌套结构，这可以通过在 `ChainAnteDecorators` 函数中明确列出装饰器顺序来缓解。
2. 不使用 ModuleManager 设计。 由于这已经用于 BeginBlocker/EndBlocker，因此该提议似乎与该设计模式不一致。

## 结果

由于每种方法都写了优缺点，因此本节省略

## 参考 

- [#4572](https://github.com/cosmos/cosmos-sdk/issues/4572):  Modular AnteHandler Issue
- [#4582](https://github.com/cosmos/cosmos-sdk/pull/4583): Initial Implementation of Per-Module AnteHandler Approach
- [Weave Decorator Code](https://github.com/iov-one/weave/blob/master/handler.go#L35)
- [Weave Design Videos](https://vimeo.com/showcase/6189877)
