# 概念

## 証拠

`x/evidence`モジュールに提出される具体的な種類の証拠は、次の条件を満たす必要があります。
以下に概説する[証拠]契約。 すべての具体的な種類の証拠が満たされるわけではありません
この契約は同じように、一部のデータは特定のデータとはまったく無関係である可能性があります
証拠の種類。`Evidence`を拡張する追加`ValidatorEvidence`、
悪意のあるバリデーターに対する証拠の契約を定義するためにも作成されました。

```go
//Evidence defines the contract which concrete evidence types of misbehavior
//must implement.
type Evidence interface {
	proto.Message

	Route() string
	Type() string
	String() string
	Hash() tmbytes.HexBytes
	ValidateBasic() error

	//Height at which the infraction occurred
	GetHeight() int64
}

//ValidatorEvidence extends Evidence interface to define contract
//for evidence against malicious validators
type ValidatorEvidence interface {
	Evidence

	//The consensus address of the malicious validator at time of infraction
	GetConsensusAddress() sdk.ConsAddress

	//The total power of the malicious validator at time of infraction
	GetValidatorPower() int64

	//The total validator set power at time of infraction
	GetTotalPower() int64
}
```

## 登録と処理

`x/evidence`モジュールは、最初に、期待するすべてのタイプの証拠を知る必要があります
対処する。 これは、`Route`メソッドを`Evidence`に登録することによって行われます。
いわゆる[ルーター](以下に定義)と契約を結びます。`ルーター`は受け入れる
`Evidence`そして`Evidence`に対応する`Handler`を見つけようとします
`Route`メソッドを介して。

```go
type Router interface {
  AddRoute(r string, h Handler) Router
  HasRoute(r string) bool
  GetRoute(path string) Handler
  Seal()
  Sealed() bool
}
```

`Handler`(以下に定義)は、
`証拠`を処理するためのビジネスロジック。 これには通常、検証が含まれます
証拠、`ValidateBasic`によるステートレスチェックと任意のステートフルチェックの両方
`ハンドラー`に提供されるキーパー。 さらに、`ハンドラー`も実行する可能性があります
バリデーターのスラッシュやジェイルなどの機能。 処理されたすべての`証拠`
`ハンドラー`によって永続化する必要があります。 

```go
//Handler defines an agnostic Evidence handler. The handler is responsible
//for executing all corresponding business logic necessary for verifying the
//evidence as valid. In addition, the Handler may execute any necessary
//slashing and potential jailing.
type Handler func(sdk.Context, Evidence) error
```
