# ADR 012:状态访问器

## 变更日志

- 2019 年 9 月 4 日:初稿

## 语境

Cosmos SDK 模块目前使用 `KVStore` 接口和 `Codec` 来访问它们各自的状态。尽管
这为模块开发人员提供了很大的自由度，很难模块化，而且 UX
平庸。

首先，每次模块尝试访问状态时，它必须编组值并设置或获取
值并最终解组。通常这是通过声明 `Keeper.GetXXX` 和 `Keeper.SetXXX` 函数来完成的，
它们是重复的且难以维护。

其次，这使得更难与对象能力定理保持一致:访问
状态被定义为一个“StoreKey”，它提供了对整个 Merkle 树的完全访问权限，因此模块不能
将特定键值对(或一组键值对)的访问权限安全地发送到另一个模块。

最后，因为 getter/setter 函数被定义为模块的 `Keeper` 的方法，审阅者
在查看访问状态任何部分的函数时，必须考虑整个 Merkle 树空间。
没有静态方法可以知道函数正在访问状态的哪一部分(哪些不是)。 

## Decision

我们将定义一个名为“Value”的类型:

```go
type Value struct {
  m   Mapping
  key []byte
}
```

`Value` 用作状态中键值对的引用，其中 `Value.m` 定义键值
它将访问的空间和 `Value.key` 定义引用的确切键。

我们将定义一个名为 `Mapping` 的类型: 

```go
type Mapping struct {
  storeKey sdk.StoreKey
  cdc      *codec.LegacyAmino
  prefix   []byte
}
```

`Mapping` 用作状态中键值空间的引用，其中 `Mapping.storeKey` 定义
IAVL(子)树和`Mapping.prefix` 定义了可选的子空间前缀。

我们将为 `Value` 类型定义以下核心方法: 

```go
// Get and unmarshal stored data, noop if not exists, panic if cannot unmarshal
func (Value) Get(ctx Context, ptr interface{}) {}

// Get and unmarshal stored data, return error if not exists or cannot unmarshal
func (Value) GetSafe(ctx Context, ptr interface{}) {}

// Get stored data as raw byte slice
func (Value) GetRaw(ctx Context) []byte {}

// Marshal and set a raw value
func (Value) Set(ctx Context, o interface{}) {}

// Check if a raw value exists
func (Value) Exists(ctx Context) bool {}

// Delete a raw value value
func (Value) Delete(ctx Context) {}
```

We will define the following core methods for the `Mapping` type:

```go
// Constructs key-value pair reference corresponding to the key argument in the Mapping space
func (Mapping) Value(key []byte) Value {}

// Get and unmarshal stored data, noop if not exists, panic if cannot unmarshal
func (Mapping) Get(ctx Context, key []byte, ptr interface{}) {}

// Get and unmarshal stored data, return error if not exists or cannot unmarshal
func (Mapping) GetSafe(ctx Context, key []byte, ptr interface{})

// Get stored data as raw byte slice
func (Mapping) GetRaw(ctx Context, key []byte) []byte {}

// Marshal and set a raw value
func (Mapping) Set(ctx Context, key []byte, o interface{}) {}

// Check if a raw value exists
func (Mapping) Has(ctx Context, key []byte) bool {}

// Delete a raw value value
func (Mapping) Delete(ctx Context, key []byte) {}
```

传递参数 `ctx`、`key` 和 `args...` 的 `Mapping` 类型的每个方法都将代理
对带有参数 ctx 和 args... 的 Mapping.Value(key) 的调用。

此外，我们将定义并提供一组从 `Value` 类型派生的通用类型:

```go
type Boolean struct { Value }
type Enum struct { Value }
type Integer struct { Value; enc IntEncoding }
type String struct { Value }
// ...
```

在编码方案可以不同的地方，核心方法中的 `o` 参数是类型化的，而 `ptr` 参数
在核心方法中被显式返回类型替换。

最后，我们将定义从 Mapping 类型派生的一系列类型:

```go
type Indexer struct {
  m   Mapping
  enc IntEncoding
}
```

键入核心方法中的“key”参数的位置。

访问器类型的一些属性是:

- 状态访问仅在调用以 `Context` 作为参数的函数时发生
- 访问器类型结构仅授予访问该结构所引用的状态的权限，没有其他权限
- 编组/解组在核心方法中隐式发生 

## Status

Proposed

## Consequences

### Positive

- 序列化将自动完成
- 更短的代码大小，更少的样板，更好的用户体验
- 对状态的引用可以安全地转移
- 明确的访问范围 

### Negative

- 序列化将自动完成
- 更短的代码大小，更少的样板，更好的用户体验
- 对状态的引用可以安全地转移
- 明确的访问范围 

### Neutral

## References

- [#4554](https://github.com/cosmos/cosmos-sdk/issues/4554)
