# 概念

## 证据

提交给“x/evidence”模块的任何具体类型的证据都必须满足
“证据”合同概述如下。 并非所有具体类型的证据都能满足
本合同以同样的方式和某些数据可能与某些完全无关
证据类型。 一个额外的“ValidatorEvidence”，它扩展了“Evidence”，
还创建了一个合同，用于针对恶意验证者的证据。 

```go
// Evidence defines the contract which concrete evidence types of misbehavior
// must implement.
type Evidence interface {
	proto.Message

	Route() string
	Type() string
	String() string
	Hash() tmbytes.HexBytes
	ValidateBasic() error

	// Height at which the infraction occurred
	GetHeight() int64
}

// ValidatorEvidence extends Evidence interface to define contract
// for evidence against malicious validators
type ValidatorEvidence interface {
	Evidence

	// The consensus address of the malicious validator at time of infraction
	GetConsensusAddress() sdk.ConsAddress

	// The total power of the malicious validator at time of infraction
	GetValidatorPower() int64

	// The total validator set power at time of infraction
	GetTotalPower() int64
}
```

## 注册和处理

`x/evidence` 模块必须首先知道它所期望的所有类型的证据
处理。 这是通过在 `Evidence` 中注册 `Route` 方法来完成的
与所谓的“路由器”(定义如下)签订合同。 `Router` 接受
`Evidence` 并尝试为 `Evidence` 找到相应的 `Handler`
通过`Route`方法。 

```go
type Router interface {
  AddRoute(r string, h Handler) Router
  HasRoute(r string) bool
  GetRoute(path string) Handler
  Seal()
  Sealed() bool
}
```

`Handler`(定义如下)负责执行整个
处理“证据”的业务逻辑。 这通常包括验证
证据，通过“ValidateBasic”进行无状态检查和通过任何方式进行的有状态检查
提供给 `Handler` 的 Keepers。 此外，`Handler` 还可以执行
诸如削减和监禁验证者之类的功能。 处理所有“证据”
由`Handler` 应该被持久化。 

```go
// Handler defines an agnostic Evidence handler. The handler is responsible
// for executing all corresponding business logic necessary for verifying the
// evidence as valid. In addition, the Handler may execute any necessary
// slashing and potential jailing.
type Handler func(sdk.Context, Evidence) error
```
