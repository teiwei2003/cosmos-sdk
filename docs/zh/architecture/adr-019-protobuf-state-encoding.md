# ADR 019:协议缓冲区状态编码

## 变更日志

- 2020 年 2 月 15 日:初稿
- 2020 年 2 月 24 日:更新处理带有接口字段的消息
- 2020 年 4 月 27 日:将用于接口的 `oneof` 的用法转换为 `Any`
- 2020 年 5 月 15 日:描述“cosmos_proto”扩展和氨基兼容性
- 2020 年 12 月 4 日:将“MarshalAny”和“UnmarshalAny”移动并重命名到“codec.Codec”界面中。
- 2021 年 2 月 24 日:删除在 [#6843](https://github.com/cosmos/cosmos-sdk/pull/6843) 中已放弃的“HybridCodec”的提及。

## 地位

公认

## 语境

目前，Cosmos SDK 使用 [go-amino](https://github.com/tendermint/go-amino/) 进行二进制
和 JSON 对象编码通过线路在逻辑对象和持久性对象之间带来奇偶校验。

来自氨基文档:

> Amino 是一种对象编码规范。它是 Proto3 的子集，带有接口扩展
> 支持。有关更多信息，请参阅 [Proto3 规范](https://developers.google.com/protocol-buffers/docs/proto3)
> 关于 Proto3 的信息，Amino 在很大程度上兼容(但不兼容 Proto2)。
>
> Amino 编码协议的目标是将奇偶校验引入逻辑对象和持久对象。

Amino 还旨在实现以下目标(不是完整列表):

- 二进制字节必须可以用模式解码。
- 架构必须是可升级的。
- 编码器和解码器逻辑必须相当简单。

但是，我们认为 Amino 并没有完全实现这些目标，也没有完全满足
Cosmos SDK 中真正灵活的跨语言和多客户端兼容编码协议的需求。
也就是说，在支持跨对象序列化方面，Amino 已被证明是一个很大的痛点。
客户端用各种语言编写，同时几乎没有提供真正的倒退方式
兼容性和可升级性。此外，通过分析和各种基准测试，Amino 已经
在 Cosmos SDK <sup>1</sup> 中被证明是一个非常大的性能瓶颈。这是
主要体现在模拟性能和应用事务吞吐量上。

因此，我们需要采用符合以下状态序列化标准的编码协议:

- 语言不可知论
- 平台不可知
- 丰富的客户支持和蓬勃发展的生态系统
- 高性能
- 最小编码消息大小
- 基于代码生成而不是基于反射
- 支持向后和向前兼容

请注意，从 Amino 迁移应该被视为一种双管齐下的方法，状态和客户端编码。
此 ADR 侧重于 Cosmos SDK 状态机中的状态序列化。相应的 ADR 将是
用于解决客户端编码。

## 决定

我们将采用 [Protocol Buffers](https://developers.google.com/protocol-buffers) 进行序列化
在 Cosmos SDK 中持久化结构化数据，同时为
希望继续使用 Amino 的应用程序。我们将通过更新模块来提供这种机制
接受一个编解码器接口，`Marshaler`，而不是一个具体的 Amino 编解码器。此外，Cosmos SDK
将提供 `Marshaler` 接口的两个具体实现:`AminoCodec` 和 `ProtoCodec`。

- `AminoCodec`:使用 Amino 进行二进制和 JSON 编码。
- `ProtoCodec`:使用 Protobuf 进行二进制和 JSON 编码。

模块将使用在应用程序中实例化的任何编解码器。默认情况下，Cosmos SDK 的`simapp`
在`MakeTestEncodingConfig`中实例化一个`ProtoCodec`作为`Marshaler`的具体实现
功能。如果他们愿意，这可以很容易地被应用程序开发人员覆盖。 

最终目标将是用 Protobuf 编码替换 Amino JSON 编码，从而有
模块接受和/或扩展`ProtoCodec`。 在那之前，Amino JSON 仍然提供给遗留用例。
Cosmos SDK 中的少数地方仍然使用 Amino JSON 硬编码，例如 Legacy API REST 端点
和 `x/params` 存储。 它们计划逐步转换为 Protobuf。

### 模块编解码器

不需要能够处理和序列化接口的模块，Protobuf 的路径
迁移非常简单。 这些模块是为了简单地迁移任何现有的类型
通过他们的具体 Amino 编解码器对 Protobuf 进行编码和持久化，并让他们的管理员接受
`Marshaler` 将是一个 `ProtoCodec`。 此迁移很简单，因为事情将按原样运行。

请注意，任何需要对基本类型(如“bool”或“int64”)进行编码的业务逻辑都应使用
[gogoprotobuf](https://github.com/gogo/protobuf) 值类型。

例子: 

```go
  ts, err := gogotypes.TimestampProto(completionTime)
  if err != nil {
    // ...
  }

  bz := cdc.MustMarshal(ts)
```

但是，模块的用途和设计可能会有很大差异，因此我们必须支持模块的能力
能够编码和使用接口(例如`Account` 或`Content`)。 对于这些模块，他们
必须定义自己的编解码器接口来扩展`Marshaler`。 这些特定的接口是独一无二的
到模块，并将包含知道如何序列化所需接口的方法契约。

例子: 

```go
// x/auth/types/codec.go

type Codec interface {
  codec.Codec

  MarshalAccount(acc exported.Account) ([]byte, error)
  UnmarshalAccount(bz []byte) (exported.Account, error)

  MarshalAccountJSON(acc exported.Account) ([]byte, error)
  UnmarshalAccountJSON(bz []byte) (exported.Account, error)
}
```

### 使用 `Any` 来编码接口

通常，模块级 .proto 文件应该定义对接口进行编码的消息
使用 [`google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto)。
[扩展讨论](https://github.com/cosmos/cosmos-sdk/issues/6030)后，
这被选为应用程序级`oneof`s的首选替代方案
就像我们最初的 protobuf 设计一样。支持“Any”的论据可以是
总结如下:

* `Any` 提供了一个更简单、更一致的客户端用户体验来处理
比应用级`oneof`s 需要更多协调的接口
仔细跨应用程序。创建通用事务
使用`oneof`s 签名库可能很麻烦，并且可能需要关键逻辑
为每个链重新实现
* `Any` 比 `oneof` 更能抵抗人为错误
* `Any` 通常更容易为模块和应用程序实现

使用“Any”的主要反驳围绕其额外空间
以及可能的性能开销。可以使用处理空间开销
未来持久层的压缩和性能影响
可能很小。因此，不使用 `Any` 似乎是一种过早的优化，
将用户体验作为更高层次的关注。

请注意，鉴于 Cosmos SDK 决定采用所描述的“编解码器”接口
上面，应用程序仍然可以选择使用`oneof`来编码状态和交易
但这不是推荐的方法。如果应用确实选择使用`oneof`s
而不是“任何”，他们可能会失去与客户端应用程序的兼容性
支持多链。因此开发者应该仔细考虑是否
他们更关心什么可能是过早的优化或最终用户
和客户端开发人员 UX。

### `Any` 的安全使用

默认情况下，[gogo protobuf 实现`Any`](https://godoc.org/github.com/gogo/protobuf/types)
使用[全局类型注册]( https://github.com/gogo/protobuf/blob/master/proto/properties.go#L540)
将“Any”中打包的值解码为具体
去类型。这引入了一个漏洞，其中任何恶意模块
在依赖树中可以使用全局 protobuf 注册表注册一个类型
并使其被引用的事务加载和解组
它在 `type_url` 字段中。

为了防止这种情况，我们引入了一种用于解码“Any”的类型注册机制
通过`InterfaceRegistry`接口将值转换为具体类型
与 Amino 的类型注册有一些相似之处:

```go
type InterfaceRegistry interface {
    // RegisterInterface associates protoName as the public name for the
    // interface passed in as iface
    // Ex:
    //   registry.RegisterInterface("cosmos_sdk.Msg", (*sdk.Msg)(nil))
    RegisterInterface(protoName string, iface interface{})

    // RegisterImplementations registers impls as a concrete implementations of
    // the interface iface
    // Ex:
    //  registry.RegisterImplementations((*sdk.Msg)(nil), &MsgSend{}, &MsgMultiSend{})
    RegisterImplementations(iface interface{}, impls ...proto.Message)

}
```

除了作为白名单，InterfaceRegistry还可以作为
传达满足客户端接口的具体类型列表。

在 .proto 文件中:

* 接受接口的字段应该用 `cosmos_proto.accepts_interface` 注释
使用与 `protoName` 相同的全限定名称传递给 `InterfaceRegistry.RegisterInterface`
* 接口实现应该用 `cosmos_proto.implements_interface` 注释
使用与 `protoName` 相同的全限定名称传递给 `InterfaceRegistry.RegisterInterface`

未来，`protoName`、`cosmos_proto.accepts_interface`、`cosmos_proto.implements_interface`
可以通过代码生成、反射和/或静态 linting 使用。

实现`InterfaceRegistry`的相同结构也将实现一个
接口`InterfaceUnpacker` 用于解包`Any`s: 

```go
type InterfaceUnpacker interface {
    // UnpackAny unpacks the value in any to the interface pointer passed in as
    // iface. Note that the type in any must have been registered with
    // RegisterImplementations as a concrete type for that interface
    // Ex:
    //    var msg sdk.Msg
    //    err := ctx.UnpackAny(any, &msg)
    //    ...
    UnpackAny(any *Any, iface interface{}) error
}
```

请注意，`InterfaceRegistry` 的用法不会偏离标准的 protobuf
`Any` 的用法，它只是为
golang用法。

`InterfaceRegistry` 将成为 `ProtoCodec` 的成员
如上所述。 为了让模块注册接口类型，应用模块
可以选择实现以下接口: 

```go
type InterfaceModule interface {
    RegisterInterfaceTypes(InterfaceRegistry)
}
```

模块管理器将包括一个方法来调用`RegisterInterfaceTypes`
每个实现它以填充`InterfaceRegistry`的模块。

### 使用 `Any` 来编码状态

Cosmos SDK 将提供支持方法 `MarshalInterface` 和 `UnmarshalInterface` 以隐藏将接口类型包装到 `Any` 中的复杂性并允许轻松序列化。 

```go
import "github.com/cosmos/cosmos-sdk/codec"

// note: eviexported.Evidence is an interface type
func MarshalEvidence(cdc codec.BinaryCodec, e eviexported.Evidence) ([]byte, error) {
	return cdc.MarshalInterface(e)
}

func UnmarshalEvidence(cdc codec.BinaryCodec, bz []byte) (eviexported.Evidence, error) {
	var evi eviexported.Evidence
	err := cdc.UnmarshalInterface(&evi, bz)
    return err, nil
}
```

### 在 `sdk.Msg`s 中使用 `Any`

类似的概念适用于包含接口字段的消息。
例如，我们可以定义 `MsgSubmitEvidence` 如下，其中 `Evidence` 是
一个界面: 

```protobuf
// x/evidence/types/types.proto

message MsgSubmitEvidence {
  bytes submitter = 1
    [
      (gogoproto.casttype) = "github.com/cosmos/cosmos-sdk/types.AccAddress"
    ];
  google.protobuf.Any evidence = 2;
}
```

请注意，为了从 `Any` 中解压证据，我们确实需要引用
`接口注册`。 为了参考方法中的证据，例如
`ValidateBasic` 不应该知道 `InterfaceRegistry`，我们
将 `UnpackInterfaces` 阶段引入反序列化，解包
接口在需要之前。

### 解包接口

实现解包的反序列化的`UnpackInterfaces`阶段
在需要之前用 `Any` 包裹的接口，我们创建一个接口
`sdk.Msg`s 和其他类型可以实现: 

```go
type UnpackInterfacesMessage interface {
  UnpackInterfaces(InterfaceUnpacker) error
}
```

我们还在 `Any` 上引入了一个私有的 `cachedValue interface{}` 字段
使用公共 getter `GetCachedValue() interface{}` 构造自身。

`UnpackInterfaces` 方法将在消息反序列化期间调用
在 `Unmarshal` 和封装在 `Any` 中的任何接口值都将被解码
并存储在 `cachedValue` 中供以后参考。

然后解压后的接口值可以安全地在任何代码中使用
不知道`InterfaceRegistry`
并且消息可以引入一个简单的 getter 来将缓存的值转换为
正确的接口类型。

这有一个额外的好处，即解组 `Any` 值只发生一次
在初始反序列化期间而不是每次读取值时。还，
当 `Any` 值第一次被打包时(例如在调用
`NewMsgSubmitEvidence`)，缓存原始接口值，以便
再次阅读它不需要解组。

`MsgSubmitEvidence` 可以实现 `UnpackInterfaces`，加上一个便利的 getter
`GetEvidence` 如下: 

```go
func (msg MsgSubmitEvidence) UnpackInterfaces(ctx sdk.InterfaceRegistry) error {
  var evi eviexported.Evidence
  return ctx.UnpackAny(msg.Evidence, *evi)
}

func (msg MsgSubmitEvidence) GetEvidence() eviexported.Evidence {
  return msg.Evidence.GetCachedValue().(eviexported.Evidence)
}
```

### 氨基相容性

如果使用的话，我们自定义的 `Any` 实现可以透明地与 Amino 一起使用
使用正确的编解码器实例。这意味着接口封装在
`Any` 将像常规的 Amino 接口一样进行氨基封送(假设它们
已在 Amino 正确注册)。

为了使此功能正常工作:

- **所有遗留代码必须使用 `*codec.LegacyAmino` 而不是 `*amino.Codec`，后者是
  现在是一个可以正确处理 `Any`** 的包装器
- **所有新代码都应该使用`Marshaler`，它与氨基和
  原型缓冲区**
- 此外，在 v0.39 之前，`codec.LegacyAmino` 将重命名为 `codec.LegacyAmino`。

### 为什么没有选择 X

有关与替代协议的更完整比较，请参阅 [此处](https://codeburst.io/json-vs-protocol-buffers-vs-flatbuffers-a4247f8bda6f)。

### Cap'n Proto

虽然 [Cap'n Proto](https://capnproto.org/) 似乎是 Protobuf 的有利替代品
由于它对接口/泛型的原生支持和内置的规范化，它确实缺乏
与 Protobuf 相比，富客户端生态系统还不够成熟。

### FlatBuffers

[FlatBuffers](https://google.github.io/flatbuffers/) 也是一个潜在可行的替代方案，具有
主要区别在于 FlatBuffers 不需要解析/解包步骤到辅助
在您可以访问数据之前，通常与每个对象的内存分配相结合。

然而，这需要大量的研究和充分了解迁移的范围
和前进的道路——这还不是很清楚。此外，FlatBuffers 不是为
不受信任的输入。

## 未来的改进和路线图

未来我们可能会考虑在持久化的正上方增加一个压缩层
不改变 tx 或默克尔树哈希的层，但减少了存储
`Any` 的开销。此外，我们可能会采用 protobuf 命名约定
使类型 URL 更加简洁，同时保持描述性。

围绕 `Any` 的使用的额外代码生成支持是
未来也可以探索，为 Go 开发人员提供更多的用户体验
无缝的。 

## Consequences

### Positive

- 显着的性能提升。
- 支持向后和向前类型兼容性。
- 更好地支持跨语言客户端。

### Negative

- 理解和实现 Protobuf 消息所需的学习曲线。
- 由于使用了`Any`，消息大小略大，尽管这可以被抵消
   将来通过压缩层 

### Neutral

## References

1. https://github.com/cosmos/cosmos-sdk/issues/4977
2. https://github.com/cosmos/cosmos-sdk/issues/5444
