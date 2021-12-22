# ADR 023:协议缓冲区命名和版本约定

## 变更日志

- 2020 年 4 月 27 日:初稿
- 2020 年 8 月 5 日:更新指南

## 地位

公认

## 语境

Protocol Buffers 提供了一个基本的[风格指南](https://developers.google.com/protocol-buffers/docs/style)
[Buf](https://buf.build/docs/style-guide) 以此为基础。到
在可能的情况下，我们希望遵循行业公认的准则和智慧
protobuf 的有效使用，偏离那些只有在有明确
我们用例的基本原理。

### 采用`Any`

采用“google.protobuf.Any”作为推荐的编码方法
接口类型(而不是`oneof`)使包命名成为核心部分
编码为完全限定的消息名称现在出现在编码中
消息。

### 当前目录组织

到目前为止，我们主要遵循 [Buf's](https://buf.build) [DEFAULT](https://buf.build/docs/lint-checkers#default)
建议，禁用 [`PACKAGE_DIRECTORY_MATCH`](https://buf.build/docs/lint-checkers#file_layout) 的小偏差
虽然方便开发代码，但带有警告
从 Buf 说:

> 如果您不这样做，您将在各种语言的许多 Protobuf 插件中度过一段非常糟糕的时光

### 采用 gRPC 查询

在[ADR 021](adr-021-protobuf-query-encoding.md)中，Protobuf采用gRPC
本机查询。完整的 gRPC 服务路径因此成为 ABCI 查询的关键部分
小路。将来，可能会允许在持久脚本中进行 gRPC 查询
通过 CosmWasm 等技术，这些查询路由将存储在
脚本二进制文件。

## 决定

此 ADR 的目标是提供周到的命名约定:

* 鼓励用户直接与之交互时的良好用户体验
.proto 文件和完全限定的 protobuf 名称
* 平衡简洁性与过度优化的可能性(使
名称太短和神秘)或优化不足(只接受臃肿的名称
有很多冗余信息)

这些指南旨在作为 Cosmos SDK 和
第三方模块。

作为起点，我们应该采用所有 [DEFAULT](https://buf.build/docs/lint-checkers#default)
[Buf's](https://buf.build) 中的跳棋，包括 [`PACKAGE_DIRECTORY_MATCH`](https://buf.build/docs/lint-checkers#file_layout)，
除了: 

* [PACKAGE_VERSION_SUFFIX](https://buf.build/docs/lint-checkers#package_version_suffix)
* [SERVICE_SUFFIX](https://buf.build/docs/lint-checkers#service_suffix)

将在下面描述的进一步指南。

### 原则

#### 简洁和描述性的名称

名称应具有足够的描述性以传达其含义并区分
他们来自其他名字。

鉴于我们在其中使用了完全限定的名称
`google.protobuf.Any` 以及在 gRPC 查询路由中，我们的目标应该是
保持名称简洁，不要过分。一般的经验法则应该
如果一个较短的名字会传达更多或同样的东西，选择较短的
名称。

例如，`cosmos.bank.MsgSend`(19 字节)传达的信息大致相同
如`cosmos_sdk.x.bank.v1.MsgSend`(28 字节)但更简洁。

这种简洁使名称更易于使用且占用更少
交易和线上的空间。

我们还应该通过命名来抵制过度优化的诱惑
带有缩写的神秘短。例如，我们不应该尝试
将 `cosmos.bank.MsgSend` 减少到 `csm.bk.MSnd` 只是为了节省几个字节。

目标是使名称 **_concise 但不神秘_**。

#### 名称是为客户服务的

应为用户的利益选择包和类型名称，而不是
必然是因为与 go 代码库相关的遗留问题。

#### 长寿计划

为了长期支持，我们应该计划我们做的名字
选择长期使用，所以现在是制造的机会
未来的最佳选择。

### 版本控制

#### 稳定包版本指南

一般来说，模式演化是更新 protobuf 模式的方式。这意味着新的领域，
消息和 RPC 方法被_添加_到现有模式和旧字段、消息和 RPC 方法
尽可能长时间地保持。

在区块链场景中，破坏事物通常是不可接受的。例如，不可变的智能合约
可能取决于主机链上的某些数据模式。如果宿主链破坏了这些模式，智能
合同可能无法挽回地破裂。即使事情可以修复(例如在客户端软件中)，
这通常需要付出高昂的代价。

我们不应该破坏事物，而应该尽一切努力发展模式而不是仅仅破坏它们。
[Buf](https://buf.build) 重大变更检测应该用于所有稳定(非 alpha 或 beta)包
以防止此类破损。

考虑到这一点，应该或多或少地考虑包的不同稳定版本(即`v1` 或`v2`)
不同的包，这应该是升级 protobuf 模式的最后手段。创造的场景
`v2` 可能有意义的是:

* 我们想创建一个与现有模块功能相似的新模块，添加 `v2` 是最自然的
方法来做到这一点。在这种情况下，实际上只有两个具有不同 API 的不同但相似的模块。
* 我们想为现有模块添加一个新的改进 API，但将其添加到现有包中太麻烦了，
所以把它放在 `v2` 中对用户来说更干净。在这种情况下，应注意不要弃用对
`v1` 如果它在不可变的智能合约中被积极使用。

#### 不稳定(alpha 和 beta)包版本指南

建议遵循以下准则将包标记为 alpha 或 beta:

* 将某些内容标记为 `alpha` 或 `beta` 应该是最后的手段，只需将某些内容放入
稳定包(即`v1`或`v2`)应该是首选
* 一个包*应该*被标记为`alpha` *当且仅当*有要删除的活跃讨论
或在不久的将来显着改变包装
* 一个包*应该*被标记为`beta` *当且仅当*有一个积极的讨论
在不久的将来显着重构/重新设计功能，但不会删除它
* 模块*can 和should* 具有稳定(即`v1` 或`v2`)和不稳定(`alpha` 或`beta`)包的类型。

*`alpha` 和 `beta` 不应用于避免维护兼容性的责任。*
每当代码被发布到野外时，尤其是在区块链上，更改事物的成本很高。在一些
例如，对于不可变的智能合约，重大更改可能无法修复。

将某些内容标记为“alpha”或“beta”时，维护人员应该提出以下问题: 

* 要求其他人更改他们的代码的成本与我们保持更改它的可选性的好处是什么？
* 将其移至 `v1` 的计划是什么，这将如何影响用户？

`alpha` 或 `beta` 真的应该用于传达“计划中的更改”。

作为一个案例研究，gRPC 反射位于`grpc.reflection.v1alpha` 包中。从那以后就没有改变
2017，它现在​​用于其他广泛使用的软件，如 gRPCurl。有些人可能在生产服务中使用它
所以如果他们真的去把包改成`grpc.reflection.v1`，一些软件就会崩溃并且
他们可能不想那样做……所以现在`v1alpha` 包或多或少是事实上的`v1`。我们不要那样做。

以下是使用非稳定包的指南:

* [Buf推荐的版本后缀](https://buf.build/docs/lint-checkers#package_version_suffix)
(例如`v1alpha1`)_应该_用于非稳定包
* 不稳定的包通常应该被排除在破坏性变更检测之外
* 不可变的智能合约模块(即 CosmWasm)_应该_阻止智能合约/持久
与 `alpha`/`beta` 包交互的脚本

#### 省略v1后缀

而不是使用 [Buf 推荐的版本后缀](https://buf.build/docs/lint-checkers#package_version_suffix)，
对于实际上没有第二个版本的包，我们可以省略 `v1`。这
允许为常见用例提供更简洁的名称，例如 `cosmos.bank.Send`。
有第二个或第三个版本的包可以用`.v2`表示
或`.v3`。

### 包命名

#### 采用简短的、唯一的顶级包名称

顶级包应采用已知不冲突的短名称
Cosmos 生态系统中常用的其他名称。在不久的将来，一个
应创建注册表以保留和索引使用的顶级包名称
在 Cosmos 生态系统中。因为 Cosmos SDK 旨在提供
Cosmos 项目的顶级类型，顶级包名 `cosmos`
建议在 Cosmos SDK 中使用，而不是更长的 `cosmos_sdk`。
[ICS](https://github.com/cosmos/ics) 规范可以考虑一个
基于标准编号的简短顶级包，如`ics23`。

#### 限制子包深度

应谨慎增加分包深度。一般是单
模块或库需要子包。即使 `x` 或 `modules`
在源代码中用于表示模块，这对于 .proto 来说通常是不必要的
作为模块的文件是子包的主要用途。只有那些
已知不常使用的应具有较深的子封装深度。

对于Cosmos SDK，建议我们简单的写成`cosmos.bank`，
`cosmos.gov` 等，而不是 `cosmos.x.bank`。在实践中，大多数非模块
类型可以直接放在 `cosmos` 包中，或者我们可以引入一个
如果需要，`cosmos.base` 包。注意这个命名_不会_改变
go 包名称，即 `cosmos.bank` protobuf 包仍然存在
`x/银行`。

### 消息命名

消息类型名称应尽可能简洁而不失清晰度。 `sdk.Msg`
交易中使用的类型将保留 `Msg` 前缀，因为它提供
有用的上下文。

### 服务和 RPC 命名

[ADR 021](adr-021-protobuf-query-encoding.md) 指定模块应该
实现 gRPC 查询服务。我们应该考虑简洁的原则
用于查询服务和 RPC 名称，因为它们可以从持久化脚本中调用
CosmWasm 等模块。此外，用户可以使用这些来自工具的查询路径，例如
[gRPCurl](https://github.com/fullstorydev/grpcurl)。例如，我们可以缩短
`/cosmos_sdk.x.bank.v1.QueryService/QueryBalance` 到
`/cosmos.bank.Query/Balance` 不会丢失很多有用的信息。

RPC 请求和响应类型_应该_遵循`ServiceNameMethodNameRequest`/
`ServiceNameMethodNameResponse` 命名约定。即对于名为“Balance”的 RPC 方法
在 `Query` 服务上，请求和响应类型将是 `QueryBalanceRequest`
和`QueryBalanceResponse`。这将比 `BalanceRequest` 更不言自明
和“平衡响应”。

#### 只使用`Query` 作为查询服务

而不是[Buf的默认服务后缀推荐](https://github.com/cosmos/cosmos-sdk/pull/6033)，
我们应该简单地使用较短的 `Query` 来提供查询服务。

对于其他类型的 gRPC 服务，我们应该考虑坚持使用 Buf 的
默认推荐。 

#### 从查询服务RPC名称中省略`Get`和`Query`

`Query` 服务名称中应该省略 `Get` 和 `Query`，因为它们是
在完全限定的名称中是多余的。 例如，`/cosmos.bank.Query/QueryBalance`
只是说“查询”两次，没有任何新信息。

## 未来的改进

应创建顶级包名称的注册表以协调命名
跨生态，防止冲突，也帮助开发者发现
有用的模式。 一个简单的起点是一个 git 存储库
基于社区的治理。 
## Consequences

### Positive

*名称将更简洁，更易于阅读和输入
* 所有使用 `Any` 的交易都将缩短(`_sdk.x` 和 `.v1` 将被删除)
* `.proto` 文件导入将更加标准(没有 `"third_party/proto"` 在
路径)
* 客户端的代码生成将更容易，因为 .proto 文件将是
在单个 `proto/` 目录中，可以复制而不是分散
整个 Cosmos SDK 

### Negative

### Neutral

* `.proto` 文件需要重新组织和重构
* 某些模块可能需要标记为 alpha 或 beta 

## References
