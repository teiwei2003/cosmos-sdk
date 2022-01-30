# ADR 027:确定性 Protobuf 序列化

## 变更日志

- 2020-08-07:初稿
- 2020-09-01:进一步明确规则

## 地位

建议的

## 摘要

完全确定性结构序列化，适用于多种语言和客户端，
签名消息时需要。我们需要确保每当我们序列化
一个数据结构，无论支持哪种语言，原始字节
将保持不变。
[Protobuf](https://developers.google.com/protocol-buffers/docs/proto3)
序列化不是双射的(即实际上存在无限数量的
给定 protobuf 文档的有效二进制表示)<sup>1</sup>。

本文档描述了一个确定性的序列化方案
protobuf 文档的一个子集，涵盖了这个用例，但可以在
其他情况也是如此。

###  上下文

对于 Cosmos SDK 中的签名验证，签名者和验证者需要达成一致
与中定义的`SignDoc` 相同的序列化
[ADR-020](./adr-020-protobuf-transaction-encoding.md) 不传输
连载。

目前，对于块签名，我们正在使用一种解决方法:我们创建一个新的 [TxRaw](https://github.com/cosmos/cosmos-sdk/blob/9e85e81e0e8140067dd893421290c191529c148c/proto/cosmos/tx/v1proto#L30x )
实例(定义在 [adr-020-protobuf-transaction-encoding](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-020-protobuf-transaction-encoding.md #交易))
通过转换所有 [Tx](https://github.com/cosmos/cosmos-sdk/blob/9e85e81e0e8140067dd893421290c191529c148c/proto/cosmos/tx/v1beta1/tx.proto#L13)
字段到客户端的字节。这增加了一个额外的手册
发送和签署交易时的步骤。

### 决定

以下编码方案将被其他 ADR 使用，
特别是对于`SignDoc` 序列化。 

## Specification

### Scope

这个 ADR 定义了一个 protobuf3 序列化器。输出是一个有效的protobuf
序列化，这样每个 protobuf 解析器都可以解析它。

由于定义一个的复杂性，版本 1 中不支持映射
确定性序列化。这在未来可能会改变。实现必须
拒绝包含地图的文档作为无效输入。

### 背景 - Protobuf3 编码

protobuf3 中的大多数数字类型都编码为
[varints](https://developers.google.com/protocol-buffers/docs/encoding#varints)。
varints 最多 10 个字节，由于每个 varint 字节有 7 位数据，
varints 是 `uint70`(70 位无符号整数)的表示。什么时候
编码，数值从它们的基类型转换为`uint70`，并且当
解码时，解析的 `uint70` 被转换为适当的数字类型。

符合 protobuf3 的 varint 的最大有效值为
`FF FF FF FF FF FF FF FF FF 7F`(即`2**70 -1`)。如果字段类型是
`{,u,s}int64`，70 位的最高 6 位在解码过程中被丢弃，
引入 6 位延展性。如果字段类型为 `{,u,s}int32`，则
70 的最高 38 位在解码过程中被丢弃，引入 38 位
延展性。

在非确定性的其他来源中，此 ADR 消除了以下可能性
编码延展性。

### 序列化规则

序列化基于
[protobuf3 编码](https://developers.google.com/protocol-buffers/docs/encoding)
增加了以下内容:

1.字段只能按升序序列化一次
2. 不得添加额外字段或任何额外数据
3. [默认值](https://developers.google.com/protocol-buffers/docs/proto3#default)
   必须省略
4. 标量数值类型的`repeated`字段必须使用
   [打包编码](https://developers.google.com/protocol-buffers/docs/encoding#packed)
5. Varint 编码不能超过需要:
    * 没有尾随零字节(在小端，即大中没有前导零
      字节序)。根据上面的规则 3，必须省略默认值“0”，因此
      此规则不适用于此类情况。
    * varint 的最大值必须是`FF FF FF FF FF FF FF FF FF 01`。
      换句话说，解码时，70 位无符号的最高 6 位
      整数必须为“0”。 (10 字节 varints 是 10 组 7 位，即
      70 位，其中只有最低的 70-6=64 有用。)
    * varint 编码中 32 位值的最大值必须是`FF FF FF FF 0F`
      除了一个例外(如下)。换句话说，解码时，最高的 38
      70 位无符号整数的位必须为“0”。
        * 上述的一个例外是 _negative_ `int32`，它必须是
          使用完整的 10 个字节编码符号扩展<sup>2</sup>。
    * varint 编码中布尔值的最大值必须为“01”(即
      它必须是“0”或“1”)。根据上面的规则 3，'0' 的默认值必须
      被省略，因此如果包含布尔值，它的值必须为“1”。

虽然规则 1. 和 2. 应该非常直截了当并描述了
作者知道的所有 protobuf 编码器的默认行为，第三条规则
更有趣。在 protobuf3 反序列化之后，您无法区分
在未设置字段和设置为默认值的字段之间<sup>3</sup>。在
然而，序列化级别，可以将字段设置为空
重视或完全省略它们。这是一个显着的区别，例如JSON
其中属性可以为空(`""`、`0`)、`null` 或未定义，导致 3
不同的文件。 

省略设置为默认值的字段是有效的，因为解析器必须分配
序列化中缺少的字段的默认值<sup>4</sup>。对于标量
类型，规范要求省略默认值<sup>5</sup>。对于“重复”
字段，而不是序列化它们是表达空列表的唯一方法。枚举必须
有一个数值为 0 的第一个元素，这是默认的<sup>6</sup>。和
消息字段默认为 unset<sup>7</sup>。

省略默认值允许一定程度的向前兼容性:
protobuf 模式的较新版本产生与用户相同的序列化
只要不使用新添加的字段(即设置为它们的
默认值)。

### 执行

共有三种主要的实施策略，从低到高排序
大多数定制开发:

- **默认使用遵循上述规则的 protobuf 序列化器。** 例如
  [gogoproto](https://pkg.go.dev/github.com/gogo/protobuf/gogoproto) 已知
  在大多数情况下都符合，但在某些注释(例如
  使用`nullable = false`。也可能是配置一个选项
  相应地现有序列化程序。
- **在编码之前对默认值进行标准化。** 如果您的序列化程序遵循
  规则 1. 和 2. 并允许您显式取消序列化字段，
  您可以将默认值标准化为未设置。这可以在使用时完成
  [protobuf.js](https://www.npmjs.com/package/protobufjs): 

  ```js
  const bytes = SignDoc.encode({
    bodyBytes: body.length > 0 ? body : null,//normalize empty bytes to unset
    authInfoBytes: authInfo.length > 0 ? authInfo : null,//normalize empty bytes to unset
    chainId: chainId || null,//normalize "" to unset
    accountNumber: accountNumber || null,//normalize 0 to unset
    accountSequence: accountSequence || null,//normalize 0 to unset
  }).finish();
  ```

- **针对您需要的类型使用手写序列化程序。** 如果以上都不是
   方法对您有用，您可以自己编写序列化程序。 对于 SignDoc 这个
   在 Go 中看起来像这样，构建在现有的 protobuf 实用程序上:

  ```go
  if !signDoc.body_bytes.empty() {
      buf.WriteUVarInt64(0xA)//wire type and field number for body_bytes
      buf.WriteUVarInt64(signDoc.body_bytes.length())
      buf.WriteBytes(signDoc.body_bytes)
  }

  if !signDoc.auth_info.empty() {
      buf.WriteUVarInt64(0x12)//wire type and field number for auth_info
      buf.WriteUVarInt64(signDoc.auth_info.length())
      buf.WriteBytes(signDoc.auth_info)
  }

  if !signDoc.chain_id.empty() {
      buf.WriteUVarInt64(0x1a)//wire type and field number for chain_id
      buf.WriteUVarInt64(signDoc.chain_id.length())
      buf.WriteBytes(signDoc.chain_id)
  }

  if signDoc.account_number != 0 {
      buf.WriteUVarInt64(0x20)//wire type and field number for account_number
      buf.WriteUVarInt(signDoc.account_number)
  }

  if signDoc.account_sequence != 0 {
      buf.WriteUVarInt64(0x28)//wire type and field number for account_sequence
      buf.WriteUVarInt(signDoc.account_sequence)
  }
  ```

### Test vectors

鉴于 protobuf 定义 `Article.proto`

```protobuf
package blog;
syntax = "proto3";

enum Type {
  UNSPECIFIED = 0;
  IMAGES = 1;
  NEWS = 2;
};

enum Review {
  UNSPECIFIED = 0;
  ACCEPTED = 1;
  REJECTED = 2;
};

message Article {
  string title = 1;
  string description = 2;
  uint64 created = 3;
  uint64 updated = 4;
  bool public = 5;
  bool promoted = 6;
  Type type = 7;
  Review review = 8;
  repeated string comments = 9;
  repeated string backlinks = 10;
};
```

serializing the values

```yaml
title: "The world needs change 🌳"
description: ""
created: 1596806111080
updated: 0
public: true
promoted: false
type: Type.NEWS
review: Review.UNSPECIFIED
comments: ["Nice one", "Thank you"]
backlinks: []
```

must result in the serialization

```
0a1b54686520776f726c64206e65656473206368616e676520f09f8cb318e8bebec8bc2e280138024a084e696365206f6e654a095468616e6b20796f75
```

When inspecting the serialized document, you see that every second field is
omitted:

```
$ echo 0a1b54686520776f726c64206e65656473206368616e676520f09f8cb318e8bebec8bc2e280138024a084e696365206f6e654a095468616e6b20796f75 | xxd -r -p | protoc --decode_raw
1: "The world needs change \360\237\214\263"
3: 1596806111080
5: 1
7: 2
9: "Nice one"
9: "Thank you"
```

## Consequences

拥有这样的编码可以让我们获得确定性的序列化
对于我们在 Cosmos SDK 签名上下文中需要的所有 protobuf 文档。 

### Positive

- 可以独立于参考进行验证的明确定义的规则
   执行
- 足够简单以降低实现交易签名的障碍
- 它允许我们在 SignDoc 中继续使用 0 和其他空值，避免
   需要解决 0 个序列。 这并不意味着从
   https://github.com/cosmos/cosmos-sdk/pull/6949 不应该被合并，但不是
   太重要了。 

### Negative

- 在实现交易签名时，必须符合上述编码规则
   理解和执行。
- 规则编号 3 的需要增加了一些实现的复杂性。
- 某些数据结构可能需要自定义代码进行序列化。 因此
   代码不是很便携 - 每个都需要额外的工作
   客户端实现序列化以正确处理自定义数据结构。

### Neutral

### Usage in Cosmos SDK

由于上述原因(“否定”部分)，我们更愿意保留变通方法
用于共享数据结构。 示例:前面提到的 `TxRaw` 使用原始字节
作为解决方法。 这允许他们使用任何有效的 Protobuf 库，而无需
需要实现符合此标准的自定义序列化程序(以及相关的错误风险)。 

## References

- <sup>1</sup> _当一个消息被序列化时，没有保证顺序
  其已知或未知字段应如何写入。序列化顺序是
  实现细节和任何特定实现的细节可能
  未来的改变。因此，协议缓冲区解析器必须能够解析
  任何顺序的字段。_来自
  https://developers.google.com/protocol-buffers/docs/encoding#order
- <sup>2</sup> https://developers.google.com/protocol-buffers/docs/encoding#signed_integers
- <sup>3</sup> _注意对于标量消息字段，一旦消息被解析
  没有办法判断一个字段是否显式设置为默认值
  值(例如布尔值是否设置为 false)或根本不设置:
  在定义消息类型时，您应该牢记这一点。例如，
  没有一个布尔值在设置为 false 时开启某些行为，如果你
  不希望这种行为也发生在默认情况下。_来自
  https://developers.google.com/protocol-buffers/docs/proto3#default
- <sup>4</sup> _当一个消息被解析时，如果编码的消息没有
  包含一个特定的奇异元素，解析中的对应字段
  对象设置为该字段的默认值。_来自
  https://developers.google.com/protocol-buffers/docs/proto3#default
- <sup>5</sup> _还请注意，如果标量消息字段设置为其默认值，
  该值不会在线上序列化。_来自
  https://developers.google.com/protocol-buffers/docs/proto3#default
- <sup>6</sup> _对于枚举，默认值为第一个定义的枚举值，
  必须是 0._ 来自
  https://developers.google.com/protocol-buffers/docs/proto3#default
- <sup>7</sup> _对于消息字段，该字段未设置。它的确切值是
  语言相关。_来自
  https://developers.google.com/protocol-buffers/docs/proto3#default
- 编码规则和部分推理取自
  [canonical-proto3 Aaron Craelius](https://github.com/regen-network/canonical-proto3) 