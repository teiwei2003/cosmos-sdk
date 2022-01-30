# 编码

虽然 Cosmos SDK 中的编码过去主要由 `go-amino` 编解码器处理，但 Cosmos SDK 正在转向使用 `gogoprotobuf` 进行状态和客户端编码。 {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## 编码

Cosmos SDK 使用两个二进制线编码协议，[Amino](https://github.com/tendermint/go-amino/) 是一个对象编码规范和 [Protocol Buffers](https://developers.google.com/protocol-buffers)，Proto3 的一个子集，扩展为
接口支持。请参阅 [Proto3 规范](https://developers.google.com/protocol-buffers/docs/proto3)
有关 Proto3 的更多信息，Amino 在很大程度上兼容(但不兼容 Proto2)。

由于 Amino 具有显着的性能缺陷，基于反射，并且
没有任何有意义的跨语言/客户端支持，协议缓冲区，特别是
[gogoprotobuf](https://github.com/gogo/protobuf/)，用于代替 Amino。
请注意，这个在 Amino 上使用 Protocol Buffers 的过程仍然是一个持续的过程。

Cosmos SDK 中类型的二进制线编码可以分为两个主要的
类别、客户端编码和存储编码。客户端编码主要围绕
围绕交易处理和签名，而存储编码围绕
状态机转换中使用的类型以及最终存储在 Merkle 中的内容
树。

对于存储编码，protobuf 定义可以存在于任何类型，并且通常会
有一个基于氨基的“中介”类型。具体来说，基于protobuf的类型
定义用于序列化和持久化，而基于氨基的类型
用于状态机中的业务逻辑，它们可以来回转换。
请注意，基于氨基的类型可能会在未来逐渐淘汰，因此开发人员
应注意尽可能使用 protobuf 消息定义。

在`codec` 包中，存在两个核心接口，`Marshaler` 和`ProtoMarshaler`，
其中前者封装了当前的 Amino 接口，除了它对
实现后者而不是通用`interface{}` 类型的类型。

此外，存在两种`Marshaler` 实现。第一个是
`AminoCodec`，其中二进制和 JSON 序列化都通过 Amino 处理。这
第二个是 `ProtoCodec`，其中处理二进制和 JSON 序列化
通过 Protobuf。

这意味着模块可以使用 Amino 或 Protobuf 编码，但类型必须
实现`ProtoMarshaler`。如果模块希望避免实现此接口
对于他们的类型，他们可以直接使用 Amino 编解码器。 

### Amino

每个模块都使用 Amino 编解码器来序列化类型和接口。 这种编解码器通常
仅在该模块的域中注册了类型和接口(例如消息)，
但也有像 `x/gov` 这样的例外。 每个模块都暴露了一个 `RegisterLegacyAminoCodec` 函数
允许用户提供编解码器并注册所有类型。 一个应用程序
将为每个必要的模块调用此方法。

如果模块没有基于 protobuf 的类型定义(见下文)，Amino
用于将原始线字节编码和解码为具体类型或接口: 

```go
bz := keeper.cdc.MustMarshal(typeOrInterface)
keeper.cdc.MustUnmarshal(bz, &typeOrInterface)
```

请注意，上述功能存在以长度为前缀的变体，这是
通常用于数据需要流式传输或分组在一起时
(例如`ResponseDeliverTx.Data`) 

### Gogoproto

鼓励模块对其各自的类型使用 Protobuf 编码。在 Cosmos SDK 中，我们使用 Protobuf 规范的 [Gogoproto](https://github.com/gogo/protobuf) 特定实现，与官方 [Google protobuf 实现](https://github.com/protocolbuffers/protobuf)。

### protobuf 消息定义指南

除了[遵循官方协议缓冲区指南](https://developers.google.com/protocol-buffers/docs/proto3#simple)，我们建议在处理接口时在 .proto 文件中使用这些注释:

- 使用 `cosmos_proto.accepts_interface` 来注释接受接口的字段
- 将与 `protoName` 相同的完全限定名称传递给 `InterfaceRegistry.RegisterInterface`
- 使用 `cosmos_proto.implements_interface` 注释接口实现
- 将与 `protoName` 相同的完全限定名称传递给 `InterfaceRegistry.RegisterInterface`

### 交易编码

Protobuf 的另一个重要用途是编码和解码
[交易](./transactions.md)。事务由应用程序或
Cosmos SDK，但随后被传递到底层共识引擎以中继到
其他同龄人。由于底层共识引擎与应用程序无关，
共识引擎只接受原始字节形式的交易。

- `TxEncoder` 对象执行编码。
- `TxDecoder` 对象执行解码。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/types/tx_msg.go#L83-L87

这两个对象的标准实现可以在 [`auth` 模块](../../x/auth/spec/README.md) 中找到:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/x/auth/tx/decoder.go

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/x/auth/tx/encoder.go

有关交易如何编码的详细信息，请参阅 [ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md)。

### `Any` 的接口编码和使用

Protobuf DSL 是强类型的，这会使插入变量类型的字段变得困难。想象一下，我们想要创建一个 `Profile` protobuf 消息，作为 [an account](../basics/accounts.md) 的包装器:

```proto
message Profile {
 //account is the account associated to a profile.
  cosmos.auth.v1beta1.BaseAccount account = 1;
 //bio is a short description of the account.
  string bio = 4;
}
```

在这个 `Profile` 示例中，我们将 `account` 硬编码为 `BaseAccount`。但是，还有其他几种类型的[与归属相关的用户帐户](../../x/auth/spec/05_vesting.md)，例如“BaseVestingAccount”或“ContinuousVestingAccount”。所有这些帐户都不同，但它们都实现了`AccountI` 接口。您将如何创建一个允许所有这些类型帐户的“个人资料”，其中“帐户”字段接受“帐户I”接口？

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/types/account.go#L307-L330

在 [ADR-019](../architecture/adr-019-protobuf-state-encoding.md) 中，已决定使用 [`Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto)s 在 protobuf 中编码接口。 `Any` 包含一个任意序列化的字节消息，以及一个 URL，该 URL 充当全局唯一标识符并解析为该消息的类型。这种策略允许我们在 protobuf 消息中打包任意 Go 类型。我们的新“配置文件”看起来像:

```protobuf
message Profile {
 //account is the account associated to a profile.
  google.protobuf.Any account = 1 [
    (cosmos_proto.accepts_interface) = "AccountI";//Asserts that this field only accepts Go types implementing `AccountI`. It is purely informational for now.
  ];
 //bio is a short description of the account.
  string bio = 4;
}
```

要在配置文件中添加帐户，我们需要先将其“打包”到一个“Any”中，使用“codectypes.NewAnyWithValue”:

```go
var myAccount AccountI
myAccount = ...//Can be a BaseAccount, a ContinuousVestingAccount or any struct implementing `AccountI`

//Pack the account into an Any
accAny, err := codectypes.NewAnyWithValue(myAccount)
if err != nil {
  return nil, err
}

//Create a new Profile with the any.
profile := Profile {
  Account: accAny,
  Bio: "some bio",
}

//We can then marshal the profile as usual.
bz, err := cdc.Marshal(profile)
jsonBz, err := cdc.MarshalJSON(profile)
```

总而言之，要对接口进行编码，您必须 1/将接口打包到 `Any` 中，并且 2/封送 `Any`。 为方便起见，Cosmos SDK 提供了一个 `MarshalInterface` 方法来捆绑这两个步骤。 看看 [x/auth 模块中的一个真实例子](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/keeper/keeper.go#L218- L221)。

从 `Any` 内部检索具体 Go 类型的反向操作称为“解包”，是通过 `Any` 上的 `GetCachedValue()` 完成的。 

```go
profileBz := ...//The proto-encoded bytes of a Profile, e.g. retrieved through gRPC.
var myProfile Profile
//Unmarshal the bytes into the myProfile struct.
err := cdc.Unmarshal(profilebz, &myProfile)

//Let's see the types of the Account field.
fmt.Printf("%T\n", myProfile.Account)                 //Prints "Any"
fmt.Printf("%T\n", myProfile.Account.GetCachedValue())//Prints "BaseAccount", "ContinuousVestingAccount" or whatever was initially packed in the Any.

//Get the address of the accountt.
accAddr := myProfile.Account.GetCachedValue().(AccountI).GetAddress()
```

需要注意的是，要让 `GetCachedValue()` 工作，`Profile`(以及任何其他嵌入 `Profile` 的结构)必须实现 `UnpackInterfaces` 方法: 

```go
func (p *Profile) UnpackInterfaces(unpacker codectypes.AnyUnpacker) error {
  if p.Account != nil {
    var account AccountI
    return unpacker.UnpackAny(p.Account, &account)
  }

  return nil
}
```

`UnpackInterfaces` 在实现此方法的所有结构上被递归调用，以允许所有 `Any` 正确填充它们的 `GetCachedValue()`。 

有关接口编码的更多信息，尤其是关于`UnpackInterfaces` 以及如何使用`InterfaceRegistry` 解析`Any` 的`type_url`，请参阅[ADR-019](../architecture/adr-019- protobuf-state-encoding.md)。

#### Cosmos SDK 中的`Any` 编码

上面的“配置文件”示例是一个用于教育目的的虚构示例。在 Cosmos SDK 中，我们在几个地方使用了 `Any` 编码(非详尽列表):

- 用于编码不同类型公钥的`cryptotypes.PubKey`接口，
- `sdk.Msg` 接口，用于在交易中编码不同的 `Msg`，
- 用于在 x/auth 查询响应中编码不同类型帐户(类似于上面的示例)的 `AccountI` 接口，
- 用于在 x/evidence 模块中编码不同类型证据的 `Evidencei` 接口，
- 用于对不同类型的 x/authz 授权进行编码的 `AuthorizationI` 接口，
- [`Validator`](https://github.com/cosmos/cosmos-sdk/blob/v0.42.5/x/staking/types/staking.pb.go#L306-L337) 结构包含有关验证器。

以下示例显示了在 x/staking 中的 Validator 结构内将公钥编码为“Any”的真实示例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/staking/types/validator.go#L40-L61

## 常问问题

1.如何使用protobuf编码创建模块？

**定义模块类型**

可以定义 Protobuf 类型来编码:

- 状态
- [`Msg`s](../building-modules/messages-and-queries.md#messages)
- [查询服务](../building-modules/query-services.md)
- [创世](../building-modules/genesis.md)

**命名和约定**

我们鼓励开发者遵循行业指南:[Protocol Buffers style guide](https://developers.google.com/protocol-buffers/docs/style)
和 [Buf](https://buf.build/docs/style-guide)，请参阅 [ADR 023](../architecture/adr-023-protobuf-naming.md) 中的更多详细信息

2.如何将模块更新为protobuf编码？ 

如果模块不包含任何接口(例如`Account` 或`Content`)，那么它们
可以简单地迁移任何现有类型
通过其具体的 Amino 编解码器编码并持久化到 Protobuf(有关进一步的指导，请参见 1.)并接受“Marshaler”作为通过“ProtoCodec”实现的编解码器
无需任何进一步的定制。

然而，如果一个模块类型组成了一个接口，它必须将它包装在 `skd.Any`(来自 `/types` 包)类型中。为此，模块级 .proto 文件必须使用 [`google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto)各自的消息类型接口类型。

例如，在`x/evidence` 模块中定义了一个`Evidence` 接口，由`MsgSubmitEvidence` 使用。结构定义必须使用`sdk.Any`来包装证据文件。在proto文件中我们定义如下:

```protobuf
//proto/cosmos/evidence/v1beta1/tx.proto

message MsgSubmitEvidence {
  string              submitter = 1;
  google.protobuf.Any evidence  = 2 [(cosmos_proto.accepts_interface) = "Evidence"];
}
```

Cosmos SDK `codec.Codec` 接口提供支持方法 `MarshalInterface` 和 `UnmarshalInterface` 以轻松将状态编码为 `Any`。

模块应该使用`InterfaceRegistry`注册接口，它提供了一种注册接口的机制:`RegisterInterface(protoName string, iface interface{})`和实现:`RegisterImplementations(iface interface{}, impls ...proto.Message)`可以从 Any 中安全地解压，类似于使用 Amino 进行类型注册:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/codec/types/interface_registry.go#L25-L66

此外，应该在反序列化中引入一个 `UnpackInterfaces` 阶段，以便在需要它们之前解压接口。直接或通过其成员之一包含 protobuf `Any` 的 Protobuf 类型应该实现 `UnpackInterfacesMessage` 接口:

```go
type UnpackInterfacesMessage interface {
  UnpackInterfaces(InterfaceUnpacker) error
}
```

## 下一个 {hide}

了解 [gRPC、REST 和其他端点](./grpc_rest.md) {hide} 
