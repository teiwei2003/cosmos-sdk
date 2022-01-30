# ADR 010:モジュラーAnteHandler

## 変更ログ

-2019年8月31日:最初のドラフト
-2021年9月14日:ADR-045に置き換えられました

## 状態

ADR-045に置き換えられました

## 環境

現在のAnteHandlerの設計では、ユーザーは `x/auth`で提供されるデフォルトのAnteHandlerを使用するか、独自のAnteHandlerを最初から作成できます。理想的には、AnteHandler関数は複数のモジュラー関数に分割され、カスタムante-functionとリンクできるため、ユーザーはカスタム動作を実装するときに共通のantehandlerロジックを書き直す必要がありません。

たとえば、ユーザーがカスタム署名検証ロジックを実装したいとします。現在のコードベースでは、ユーザーは、主に同じコードのほとんどを再実装し、baseappで独自のカスタムモノリシックアンティハンドラーを設定することにより、独自のアンティハンドラーを最初から作成する必要があります。代わりに、ユーザーが必要に応じてカスタム動作を指定できるようにし、可能な限りモジュール化された柔軟な方法で、それらをデフォルトのアンティハンドラー機能と組み合わせたいと考えています。

## 提案

### モジュールごとのAnteHandler

1つの方法は、[ModuleManager](https://godoc.org/github.com/cosmos/cosmos-sdk/types/module)を使用し、カスタムのアンティハンドラロジックが必要な場合は、各モジュールに独自のアンティハンドラを実装させることです。次に、ModuleManagerは、BeginBlockersおよびEndBlockersのシーケンスと同じ方法でAnteHandlerシーケンスで渡すことができます。 ModuleManagerはAnteHandler関数を返します。この関数は、txを受け取り、指定された順序で各モジュールの `AnteHandle`を実行します。モジュールマネージャーのAnteHandlerは、baseappのAnteHandlerに設定されます。

アドバンテージ:

1.実装が簡単
2.既存のModuleManagerアーキテクチャを活用する

欠点:

1.粒度を改善しましたが、それでも各モジュールよりも細かい粒度を取得することはできません。たとえば、authの `AnteHandle`関数がメモと署名の検証を担当している場合、ユーザーはauthの残り​​の` AnteHandle`関数を保持したまま署名チェック機能を交換することはできません。
2.モジュールAnteHandlerが次々に実行されます。 1つのAnteHandlerは、別のAnteHandlerをラップまたは[装飾]することはできません。

### デコレーションモード

[Weave Project](https://github.com/iov-one/weave)AnteHandlerのモジュール性は、デコレータパターンを使用して実現されています。インターフェイスのデザインは次のとおりです。 

```go
//Decorator wraps a Handler to provide common functionality
//like authentication, or fee-handling, to many Handlers
type Decorator interface {
	Check(ctx Context, store KVStore, tx Tx, next Checker) (*CheckResult, error)
	Deliver(ctx Context, store KVStore, tx Tx, next Deliverer) (*DeliverResult, error)
}
```

各デコレータは、モジュラーCosmos SDK前処理関数のように機能しますが、[next]パラメータを受け取ることができます。これは、別のデコレータまたはハンドラ(次のパラメータを受け取らない)の場合があります。 これらのデコレータは一緒にチェーンすることができ、1つのデコレータがチェーン内の前のデコレータの[次の]パラメータとして渡されます。 チェーンはルーターで終わります。ルーターはtxを受け入れ、適切なmsgハンドラーにルーティングできます。

このアプローチの主な利点の1つは、デコレータがその内部ロジックを次のチェッカー/デリバラにラップできることです。 ブレードデコレータは次のことを実行できます。 

```go
//Example Decorator's Deliver function
func (example Decorator) Deliver(ctx Context, store KVStore, tx Tx, next Deliverer) {
  ./Do some pre-processing logic

    res, err := next.Deliver(ctx, store, tx)

  ./Do some post-processing logic given the result and error
}
```

アドバンテージ:

1. Weaveデコレータは、チェーン内の次のデコレータ/ハンドラをラップできます。場合によっては、前処理機能と後処理機能が役立つことがあります。
2.上記のソリューションでは実現できないネストされたモジュラー構造を提供すると同時に、上記のソリューションのような線形の1つずつの構造を可能にします。

欠点:

1.一見すると、特定の `ctx`、` store`、および `tx`のデコレータが実行された後に発生するステータスの更新を理解するのは困難です。デコレータは、関数本体でネストされたデコレータをいくつでも呼び出すことができ、各デコレータは、チェーン内の次のデコレータを呼び出す前に、前処理と後処理を実行できます。したがって、デコレータが何をしているのかを理解するには、チェーン内の他のデコレータも何をしているのかを理解する必要があります。これは理解するのが非常に難しくなる可能性があります。線形の1つずつの方法はそれほど強力ではありませんが、推論する方が簡単な場合があります。

### 連鎖マイクロ機能

Weaveメソッドの利点は、デコレータが非常に簡潔になり、チェーン化されたときに最大限のカスタマイズを実現できることです。ただし、ネスト構造は非常に複雑になる可能性があるため、推論が困難になります。

もう1つのアプローチは、ModuleManagerメソッドからの1つずつの順序を維持しながら、AnteHandler関数を厳密にスコープされた[マイクロ関数]に分割することです。

次に、これらのマイクロ関数をリンクして、次々に実行する方法を用意できます。モジュールは複数のアンティマイクロ関数を定義でき、次にこれらのマイクロ関数のデフォルトの推奨順序を実装するためのデフォルトのモジュールごとのAnteHandlerも提供します。

ユーザーは、ModuleManagerを使用するだけでAnteHandlerを簡単に注文できます。 ModuleManagerは、AnteHandlerのリストを受け取り、提供されたリストの順序で各AnteHandlerを実行する単一のAnteHandlerを返します。ユーザーが各モジュールのデフォルトの順序に満足している場合は、各モジュールのアンティハンドラーのリストを提供するだけです(BeginBlockerおよびEndBlockerとまったく同じです)。

ただし、ユーザーが何らかの方法でフロントマイクロファンクションを追加、変更、または削除したい場合は、いつでも独自のフロントマイクロファンクションを定義して、モジュールマネージャーに渡されるリストに明示的に追加できます。

#### デフォルトのワークフロー

これは、ユーザーがカスタムマイクロ関数を作成しないことを選択したAnteHandlerの例です。

##### CosmosSDKコード  

```go
//Chains together a list of AnteHandler micro-functions that get run one after the other.
//Returned AnteHandler will abort on first error.
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
//AnteHandler micro-function to verify signatures
func VerifySignatures(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
  ./verify signatures
  ./Returns InvalidSignature Result and abort=true if sigs invalid
  ./Return OK result and abort=false if sigs are valid
}

//AnteHandler micro-function to validate memo
func ValidateMemo(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
  ./validate memo
}

//Auth defines its own default ante-handler by chaining its micro-functions in a recommended order
AuthModuleAnteHandler := Chainer([]AnteHandler{VerifySignatures, ValidateMemo})
```

```go
//Distribution micro-function to deduct fees from tx
func DeductFees(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
  ./Deduct fees from tx
  ./Abort if insufficient funds in account to pay for fees
}

//Distribution micro-function to check if fees > mempool parameter
func CheckMempoolFees(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
  ./If CheckTx: Abort if the fees are less than the mempool's minFee parameter
}

//Distribution defines its own default ante-handler by chaining its micro-functions in a recommended order
DistrModuleAnteHandler := Chainer([]AnteHandler{CheckMempoolFees, DeductFees})
```

```go
type ModuleManager struct {
  ./other fields
    AnteHandlerOrder []AnteHandler
}

func (mm ModuleManager) GetAnteHandler() AnteHandler {
    retun Chainer(mm.AnteHandlerOrder)
}
```

##### User Code

```go
//Note: Since user is not making any custom modifications, we can just SetAnteHandlerOrder with the default AnteHandlers provided by each module in our preferred order
moduleManager.SetAnteHandlerOrder([]AnteHandler(AuthModuleAnteHandler, DistrModuleAnteHandler))

app.SetAnteHandler(mm.GetAnteHandler())
```

#### カスタムワークフロー

これは、カスタムハンドラロジックを実装するユーザー向けのサンプルワークフローです。 この例では、ユーザーはカスタム署名検証を実装し、署名検証の前に検証メモを実行するようにアンティハンドラーの順序を変更したいと考えています。

##### ユーザーコード  

```go
//User can implement their own custom signature verification antehandler micro-function
func CustomSigVerify(ctx Context, tx Tx, simulate bool) (newCtx Context, err error) {
  ./do some custom signature verification logic
}
```

```go
//Micro-functions allow users to change order of when they get executed, and swap out default ante-functionality with their own custom logic.
//Note that users can still chain the default distribution module handler, and auth micro-function along with their custom ante function
moduleManager.SetAnteHandlerOrder([]AnteHandler(ValidateMemo, CustomSigVerify, DistrModuleAnteHandler))
```

アドバンテージ:

1.アンティ機能を可能な限りモジュール化できるようにします。
2.前処理プログラムをカスタマイズする必要がないユーザーの場合、前処理プログラムの動作モードは、ModuleManagerのBeginBlockおよびEndBlockの動作モードとほぼ同じです。
3.それでも理解しやすい

欠点:

1.アンティハンドラをWeaveのようなデコレータでラップすることはできません。

### シンプルなデコレータ

このアプローチは、Cosmos SDKの主要な変更を最小限に抑え、簡素化を最小限に抑えながら、Weaveのデコレータ設計からインスピレーションを得ています。 Weaveデコレータと同様に、このメソッドを使用すると、AnteDecoratorで次のAnteHandlerをラップして、結果を前処理および後処理できます。これは、AnteHandlerが戻った後、デコレータが事前に遅延/クリーンアップしていくつかの設定を実行できるため便利です。 Weaveデコレータとは異なり、これらの `AnteDecorator`関数は、ハンドラ実行パス全体ではなく、AnteHandlerのみをラップできます。さまざまなモジュールのデコレータに[tx]の認証/検証を実行させたいため、これは意図的なものです。ただし、デコレータが `MsgHandler`の結果をラップして変更できるようにする必要はありません。

さらに、このメソッドはコアCosmos SDKAPIを壊しません。 AnteHandlerの概念を維持し、baseappで単一のAnteHandlerを設定しているため、デコレータは、さらにカスタマイズが必要なユーザーのための単なる追加のメソッドです。モジュールのAPI(つまり、 `x/auth`)はこのメソッドによって中断される可能性がありますが、コアAPIは変更されません。

デコレータインターフェイスをチェーン接続して、Cosmos SDKAnteHandlerを作成できるようにします。

これにより、ユーザーはAnteHandlerを独自に実装するか、基本アプリケーションで設定するかを選択できます。または、デコレータパターンを使用して、カスタムデコレータをCosmosSDKが提供するデコレータに必要な順序でリンクできます。  

```go
//An AnteDecorator wraps an AnteHandler, and can do pre- and post-processing on the next AnteHandler
type AnteDecorator interface {
    AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error)
}
```

```go
//ChainAnteDecorators will recursively link all of the AnteDecorators in the chain and return a final AnteHandler function
//This is done to preserve the ability to set a single AnteHandler function in the baseapp.
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

#### サンプルコード

AnteDecorator関数を定義します 

```go
//Setup GasMeter, catch OutOfGasPanic and handle appropriately
type SetUpContextDecorator struct{}

func (sud SetUpContextDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
    ctx.GasMeter = NewGasMeter(tx.Gas)

    defer func() {
      ./recover from OutOfGas panic and handle appropriately
    }

    return next(ctx, tx, simulate)
}

//Signature Verification decorator. Verify Signatures and move on
type SigVerifyDecorator struct{}

func (svd SigVerifyDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
  ./verify sigs. Return error if invalid

  ./call next antehandler if sigs ok
    return next(ctx, tx, simulate)
}

//User-defined Decorator. Can choose to pre- and post-process on AnteHandler
type UserDefinedDecorator struct{
  ./custom fields
}

func (udd UserDefinedDecorator) AnteHandle(ctx Context, tx Tx, simulate bool, next AnteHandler) (newCtx Context, err error) {
  ./pre-processing logic

    ctx, err = next(ctx, tx, simulate)

  ./post-processing logic
}
```

AnteDecoratorsをリンクして、最終的なAnteHandlerを作成します。 このAnteHandlerをbaseappで設定します。 

```go
//Create final antehandler by chaining the decorators together
antehandler := ChainAnteDecorators(NewSetUpContextDecorator(), NewSigVerifyDecorator(), NewUserDefinedDecorator())

//Set chained Antehandler in the baseapp
bapp.SetAnteHandler(antehandler)
```

アドバンテージ:

1. Weaveデザインと同様に、デコレータが次のAnteHandlerを前処理および後処理できるようにします。
2. baseappAPIを壊す必要はありません。 ユーザーが必要に応じて、単一のAnteHandlerを設定することもできます。

欠点:

1.デコレータパターンは、理解しにくい深いネスト構造を持っている可能性があります。これは、 `ChainAnteDecorators`関数にデコレータの順序を明示的にリストすることで軽減できます。
2.ModuleManagerデザインを使用しないでください。 これはBeginBlocker/EndBlockerに使用されているため、提案はデザインパターンと矛盾しているようです。

## 結果

それぞれの方法の長所と短所が書かれているので、このセクションは省略されています

## 参照する 

-[#4572](https://github.com/cosmos/cosmos-sdk/issues/4572):モジュラーAnteHandlerの問題
-[#4582](https://github.com/cosmos/cosmos-sdk/pull/4583):モジュールごとのAnteHandlerアプローチの初期実装
-[ウィーブデコレータコード](https://github.com/iov-one/weave/blob/master/handler.go#L35)
-[織りデザインビデオ](https://vimeo.com/showcase/6189877)