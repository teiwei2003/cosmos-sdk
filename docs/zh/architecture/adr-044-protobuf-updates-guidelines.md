# ADR 044:Protobuf 定义更新指南

## 变更日志

- 28.06.2021:初稿
- 02.12.2021:为新字段添加`Since:` 注释

## 地位

草稿

## 摘要

此 ADR 提供更新 Protobuf 定义时的指南和推荐做法。这些指南针对模块开发人员。

##  上下文

Cosmos SDK 维护了一套[Protobuf 定义](https://github.com/cosmos/cosmos-sdk/tree/master/proto/cosmos)。正确设计 Protobuf 定义以避免在同一版本中发生任何破坏性更改非常重要。原因是不要破坏工具(包括索引器和资源管理器)、钱包和其他第三方集成。

在更改这些 Protobuf 定义时，Cosmos SDK 当前仅遵循 [Buf's](https://docs.buf.build/) 建议。但是我们注意到，在某些情况下，Buf 的建议可能仍会导致 SDK 发生重大变化。例如:

- 向`Msg`s 添加字段。添加字段不是 Protobuf 规范破坏的操作。但是，当向`Msg`s 添加新字段时，未知字段拒绝将在将新`Msg` 发送到旧节点时抛出错误。
- 将字段标记为“保留”。 Protobuf 建议使用 `reserved` 关键字来删除字段，而无需修改包版本。但是，这样做会破坏客户端的向后兼容性，因为 Protobuf 不会为“保留”字段生成任何内容。有关此问题的更多详细信息，请参阅 [#9446](https://github.com/cosmos/cosmos-sdk/issues/9446)。

此外，模块开发人员经常面临有关 Protobuf 定义的其他问题，例如“我可以重命名字段吗？”或“我可以弃用一个字段吗？”本 ADR 旨在通过提供有关 Protobuf 定义允许更新的明确指南来回答所有这些问题。

## 决定

我们决定保留 [Buf's](https://docs.buf.build/) 的建议，但以下情况除外:

- `UNARY_RPC`:Cosmos SDK 目前不支持流式 RPC。
- `COMMENT_FIELD`:Cosmos SDK 允许没有注释的字段。
- `SERVICE_SUFFIX`:我们使用`Query` 和`Msg` 服务命名约定，不使用`-Service` 后缀。
- `PACKAGE_VERSION_SUFFIX`:一些包，例如`cosmos.crypto.ed25519`，不使用版本后缀。
- `RPC_REQUEST_STANDARD_NAME`:对`Msg` 服务的请求没有`-Request` 后缀以保持向后兼容性。

在 Buf 的建议之上，我们添加了以下特定于 Cosmos SDK 的指南。

### 在没有碰撞版本的情况下更新 Protobuf 定义

#### 1. `Msg`s 不能有新字段

在处理 `Msg` 时，Cosmos SDK 的前处理程序是严格的，不允许在 `Msg` 中出现未知字段。这是通过 [`codec/unknownproto` 包](https://github.com/cosmos/cosmos-sdk/blob/master/codec/unknownproto) 中的未知字段拒绝来检查的。

现在想象一个 v0.43 节点接受一个 `MsgExample` 交易，并且在 v0.44 中，链开发者决定向 `MsgExample` 添加一个字段。仅操作 Protobuf 定义的客户端开发人员会看到 `MsgExample` 有一个新字段，并将填充它。但是，将新的“MsgExample”发送到旧的 v0.43 节点会导致 v0.43 节点因为未知字段而拒绝“MsgExample”。必须保证可以跨多个节点版本使用相同的 Protobuf 版本的期望。

出于这个原因，模块开发人员不得向现有的 `Msg` 中添加新字段。

值得一提的是，这不限制向 `Msg` 添加字段，还限制向 `Msg` 内的所有嵌套结构和 `Any` 添加字段。 

#### 2. 与`Msg` 无关的Protobuf 定义可能有新的字段，但必须添加一个`Since:` 注释

另一方面，模块开发人员可以向与“查询”服务或存储在存储中的对象相关的 Protobuf 定义中添加新字段。 此建议遵循 Protobuf 规范，但为了清楚起见添加到本文档中。

SDK 要求新字段的 Protobuf 注释包含一行，格式如下: 

```protobuf
//Since: cosmos-sdk <version>{, <version>...}
```

其中每个`version`表示该字段可用的次要(“0.45”)或补丁(“0.44.5”)版本。 这将极大地帮助客户端库，他们可以选择使用反射或自定义代码生成来根据目标节点版本显示/隐藏这些字段。

例如，以下评论是有效的: 

```protobuf
//Since: cosmos-sdk 0.44

//Since: cosmos-sdk 0.42.11, 0.44.5
```

and the following ones are NOT valid:

```protobuf
//Since cosmos-sdk v0.44

//since: cosmos-sdk 0.44

//Since: cosmos-sdk 0.42.11 0.44.5

//Since: Cosmos SDK 0.42.11, 0.44.5
```

#### 3. 字段可以被标记为`deprecated`，并且节点可以实现协议破坏性的更改来处理这些字段

Protobuf 支持 [`deprecated` 字段选项](https://developers.google.com/protocol-buffers/docs/proto#options)，并且该选项可以用于任何字段，包括 `Msg` 字段。如果节点处理带有非空弃用字段的 Protobuf 消息，则节点可能会在处理它时改变其行为，即使以破坏协议的方式。如果可能，节点必须在不破坏共识的情况下处理向后兼容性(除非我们增加原型版本)。

例如，Cosmos SDK v0.42 到 v0.43 更新包含两个 Protobuf 破坏性更改，如下所列。 SDK 团队并没有将软件包版本从 `v1beta1` 提升到 `v1`，而是决定遵循这一准则，通过还原破坏性更改、将这些更改标记为已弃用，并在处理带有弃用字段的消息时修改节点实现。进一步来说:

- Cosmos SDK 最近取消了对[基于时间的软件升级](https://github.com/cosmos/cosmos-sdk/pull/8849) 的支持。因此，“时间”字段已在“cosmos.upgrade.v1beta1.Plan”中标记为已弃用。此外，节点将拒绝任何包含“时间”字段非空的升级计划的提议。
- Cosmos SDK 现在支持 [治理分裂投票](./adr-037-gov-split-vote.md)。在查询投票时，返回的 `cosmos.gov.v1beta1.Vote` 消息的 `option` 字段(用于 1 票选项)被弃用，取而代之的是它的 `options` 字段(允许多个投票选项)。只要有可能，SDK 仍会填充已弃用的 `option` 字段，即当且仅当 `len(options) == 1` 和 `options[0].Weight == 1.0`。

#### 4. 字段不得重命名

虽然官方 Protobuf 建议不禁止重命名字段，因为它不会破坏 Protobuf 二进制表示，但 SDK 明确禁止重命名 Protobuf 结构中的字段。这种选择的主要原因是避免为客户端引入破坏性更改，这些更改通常依赖于生成类型的硬编码字段。此外，重命名字段将导致 Protobuf 定义的客户端破坏 JSON 表示，用于 REST 端点和 CLI。

### 递增 Protobuf 包版本

TODO，需要架构审查。一些主题:

- 碰撞版本频率
- 碰撞版本时，Cosmos SDK 是否应该支持两个版本？
  - 即 v1beta1 -> v1，我们是否应该在 Cosmos SDK 中有两个文件夹，以及两个版本的处理程序？
- 提及 ADR-023 Protobuf 命名

## 结果

> 本节描述应用决策后的结果上下文。所有的后果都应该在这里列出，而不仅仅是“积极的”后果。一个特定的决定可能会产生积极的、消极的和中性的后果，但所有这些都会影响未来的团队和项目。

### 向后兼容性

> 引入向后不兼容的所有 ADR 必须包括描述这些不兼容及其严重性的部分。 ADR 必须解释作者打算如何处理这些不兼容性。没有足够的向后兼容性论文的 ADR 提交可能会被彻底拒绝。 

### Positive

- 减少工具开发人员的痛苦
- 生态系统中的更多兼容性 
- ...

### Negative

{negative consequences}

### Neutral

- more rigor in Protobuf review

## Further Discussions

这个ADR还处于DRAFT阶段，一旦我们决定如何正确去做，就会填写“增量Protobuf包版本”。

## 测试用例 [可选]

对于影响共识更改的 ADR，实施的测试用例是强制性的。 如果适用，其他 ADR 可以选择包含指向测试用例的链接。

## 参考

- [#9445](https://github.com/cosmos/cosmos-sdk/issues/9445) 发布 proto 定义 v1
- [#9446](https://github.com/cosmos/cosmos-sdk/issues/9446) 解决 v1beta1 proto 的重大变化 