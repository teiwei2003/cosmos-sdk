# ADR 033:基于 Protobuf 的模块间通信

## 变更日志

- 2020-10-05:初稿

## 地位

建议的

## 摘要

该 ADR 引入了一个系统，用于利用 protobuf `Query` 和 `Msg` 进行许可的模块间通信
[ADR 021](./adr-021-protobuf-query-encoding.md) 中定义的服务定义和
[ADR 031](./adr-031-msg-service.md) 提供:

- 稳定的基于 protobuf 的模块接口可能会在以后取代 keeper 范式
- 更强的模块间对象能力(OCAPs)保证
- 模块账号和子账号授权

## 语境

在当前的 Cosmos SDK 文档中关于 [Object-Capability Model](../core/ocap.md)，它指出:

> 我们假设一个繁荣的 Cosmos SDK 模块生态系统很容易组合成区块链应用程序，将包含有缺陷或恶意的模块。

目前没有一个蓬勃发展的 Cosmos SDK 模块生态系统。我们假设这部分是由于:

1. 缺乏稳定的 v1.0 Cosmos SDK 来构建模块。模块接口正在发生变化，有时是戏剧性的，从
点发布到点发布，通常有充分的理由，但这并不能建立稳定的基础。
2. 缺乏正确实现的对象能力甚至面向对象的封装系统，导致重构
模块保持器接口的数量不可避免，因为当前接口的约束很差。

### `x/bank` 案例研究

目前，`x/bank` keeper 几乎可以不受限制地访问任何引用它的模块。例如，
`SetBalance` 方法允许调用者将任何帐户的余额设置为任何内容，甚至绕过适当的供应跟踪。

后来似乎有一些尝试使用模块级铸造、抵押来实现一些 OCAP 的外观
和刻录权限。这些权限允许模块参考模块的
自己的账户。这些权限实际上存储在状态中的“ModuleAccount”类型上的“[]string”数组中。

但是，这些权限实际上并没有多大作用。它们控制可以在“MintCoins”中引用哪些模块，
`BurnCoins` 和 `DelegateCoins***` 方法，但其中一个没有唯一的对象能力令牌来控制访问——
只是一个简单的字符串。因此，`x/upgrade` 模块可以通过调用简单地为 `x/staking` 模块铸造令牌
`MintCoins(“staking”)`。此外，所有可以访问这些 keeper 方法的模块也可以访问
`SetBalance` 否定了 OCAP 的任何其他尝试，甚至破坏了基本的面向对象封装。

## 决定

基于[ADR-021](./adr-021-protobuf-query-encoding.md)和[ADR-031](./adr-031-msg-service.md)，我们引入了
用于安全模块授权和 OCAP 的模块间通信框架。
实施后，这也可以作为现有范式的替代方案，即在
模块。此处概述的方法旨在构成 Cosmos SDK v1.0 的基础，提供必要的
稳定性和封装保证，使蓬勃发展的模块生态系统得以出现。

特别值得注意的是 - 决定_启用_此功能，以便模块自行决定采用。
将现有模块迁移到这种新范式的建议必须是一个单独的对话，可能
作为对本 ADR 的修正处理。

### 新的“守护者”范式

在[ADR 021](./adr-021-protobuf-query-encoding.md)中，一种使用protobuf服务定义来定义查询器的机制
引入并在 [ADR 31](./adr-031-msg-service.md) 中，添加了使用 protobuf 服务定义 `Msg` 的机制。
Protobuf 服务定义生成两个 golang 接口，分别代表服务的客户端和服务器端加上
一些帮助代码。以下是银行 `cosmos.bank.Msg/Send` 消息类型的最小示例: 

```go
package bank

type MsgClient interface {
	Send(context.Context, *MsgSend, opts ...grpc.CallOption) (*MsgSendResponse, error)
}

type MsgServer interface {
	Send(context.Context, *MsgSend) (*MsgSendResponse, error)
}
```

[ADR 021](./adr-021-protobuf-query-encoding.md) 和 [ADR 31](./adr-031-msg-service.md) 指定模块如何实现生成的`QueryServer`
和 `MsgServer` 接口分别作为旧查询器和 `Msg` 处理程序的替代品。

在这个 ADR 中，我们解释了模块如何使用生成的 `QueryClient` 进行查询和发送 `Msg` 到其他模块
和 `MsgClient` 接口，并提出这种机制来替代现有的 `Keeper` 范式。要清楚，
此 ADR 不需要创建新的 protobuf 定义或服务。相反，它利用相同的原型
基于客户端已经用于模块间通信的服务接口。

使用这种 `QueryClient`/`MsgClient` 方法比将 Keepers 暴露给外部模块有以下主要好处:

1. 使用 [buf](https://buf.build/docs/break-overview) 检查 Protobuf 类型的破坏性更改，因为
protobuf 的设计方式将为我们提供强大的向后兼容性保证，同时允许向前
进化。
2. 客户端和服务器接口之间的分离将允许我们在两者之间插入权限检查代码
这两个检查一个模块是否被授权将指定的 `Msg` 发送到另一个模块提供适当的
对象能力系统(见下文)。
3. 用于模块间通信的路由器为我们提供了一个方便的地方来处理事务的回滚，
启用操作原子性([目前有问题](https://github.com/cosmos/cosmos-sdk/issues/8030))。模块到模块调用中的任何故障都将导致整个模块的故障
交易

这种机制具有以下额外好处:

- 通过代码生成减少样板，以及
- 允许通过像 CosmWasm 这样的 VM 或使用 gRPC 的子进程使用其他语言的模块

### 模块间通信

要使用 protobuf 编译器生成的 `Client`，我们需要一个 `grpc.ClientConn` [接口](https://github.com/regen-network/protobuf/blob/cosmos/grpc/types.go#L12)
执行。为此我们介绍
一个新类型，`ModuleKey`，它实现了`grpc.ClientConn` 接口。 `ModuleKey` 可以被认为是“私有的
密钥”对应于模块帐户，其中通过使用特殊的`Invoker()` 函数提供身份验证，
下面更详细地描述。

区块链用户(外部客户)使用他们帐户的私钥来签署包含“消息”的交易，其中他们被列为签名者(每个
消息使用`Msg.GetSigner`指定所需的签名者)。身份验证检查由 `AnteHandler` 执行。

在这里，我们通过允许在 `Msg.GetSigners` 中识别模块来扩展这个过程。当一个模块想要触发另一个模块中的 `Msg` 执行时，
它的 `ModuleKey` 作为发送者(通过我们在下面描述的 `ClientConn` 接口)并被设置为唯一的“签名者”。值得注意的是
在这种情况下我们不使用任何加密签名。
例如，模块`A`可以使用它的`A.ModuleKey`为`/cosmos.bank.Msg/Send`交易创建`MsgSend`对象。 `MsgSend` 验证
将确保 `from` 帐户(在本例中为 `A.ModuleKey`)是签名者。

下面是一个假设模块 `foo` 与 `x/bank` 交互的示例: 

```go
package foo


type FooMsgServer {
  // ...

  bankQuery bank.QueryClient
  bankMsg   bank.MsgClient
}

func NewFooMsgServer(moduleKey RootModuleKey, ...) FooMsgServer {
  // ...

  return FooMsgServer {
    // ...
    modouleKey: moduleKey,
    bankQuery: bank.NewQueryClient(moduleKey),
    bankMsg: bank.NewMsgClient(moduleKey),
  }
}

func (foo *FooMsgServer) Bar(ctx context.Context, req *MsgBarRequest) (*MsgBarResponse, error) {
  balance, err := foo.bankQuery.Balance(&bank.QueryBalanceRequest{Address: fooMsgServer.moduleKey.Address(), Denom: "foo"})

  ...

  res, err := foo.bankMsg.Send(ctx, &bank.MsgSendRequest{FromAddress: fooMsgServer.moduleKey.Address(), ...})

  ...
}
```

此设计还旨在可扩展以涵盖更细粒度的许可的用例，例如通过以下方式进行铸造
denom 前缀仅限于某些模块(如在
[#7459](https://github.com/cosmos/cosmos-sdk/pull/7459#discussion_r529545528))。

### `ModuleKey`s 和 `ModuleID`s

`ModuleKey` 可以被认为是模块帐户的“私钥”，而 `ModuleID` 可以被认为是
对应的“公钥”。从 [ADR 028](./adr-028-public-key-addresses.md)，模块可以有一个根模块账户和任意数量的子账户
或可用于不同池(例如质押池)或管理帐户(例如组
帐户)。我们也可以认为模块子账户类似于派生密钥——有一个根密钥，然后是一些
派生路径。 `ModuleID` 是一个简单的结构体，它包含模块名称和可选的“派生”路径，
并根据 [ADR-028](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-028-public-key-addresses) 中的`AddressHash` 方法形成其地址.md): 

```go
type ModuleID struct {
  ModuleName string
  Path []byte
}

func (key ModuleID) Address() []byte {
  return AddressHash(key.ModuleName, key.Path)
}
```

除了能够生成“ModuleID”和地址之外，“ModuleKey”还包含一个名为
`Invoker` 是安全模块间访问的关键。 `Invoker` 创建了一个 `InvokeFn` 闭包，在
`grpc.ClientConn` 接口和引擎盖下能够将消息路由到适当的 `Msg` 和 `Query` 处理程序
对“消息”执行适当的安全检查。 这允许比 keeper 的更安全的模块间访问
私有成员变量可以通过反射进行操作。 Golang 不支持对函数的反射
真正的恶意模块需要绕过捕获的变量和内存的直接操作
`ModuleKey` 安全性。

两个 `ModuleKey` 类型是 `RootModuleKey` 和 `DerivedModuleKey`: 

```go
type Invoker func(callInfo CallInfo) func(ctx context.Context, request, response interface{}, opts ...interface{}) error

type CallInfo {
  Method string
  Caller ModuleID
}

type RootModuleKey struct {
  moduleName string
  invoker Invoker
}

func (rm RootModuleKey) Derive(path []byte) DerivedModuleKey { /* ... */}

type DerivedModuleKey struct {
  moduleName string
  path []byte
  invoker Invoker
}
```

一个模块可以访问一个 `DerivedModuleKey`，使用 `RootModuleKey` 上的 `Derive(path []byte)` 方法，然后
将使用此密钥来验证来自子帐户的“消息”。 Ex:

```go
package foo

func (fooMsgServer *MsgServer) Bar(ctx context.Context, req *MsgBar) (*MsgBarResponse, error) {
  derivedKey := fooMsgServer.moduleKey.Derive(req.SomePath)
  bankMsgClient := bank.NewMsgClient(derivedKey)
  res, err := bankMsgClient.Balance(ctx, &bank.MsgSend{FromAddress: derivedKey.Address(), ...})
  ...
}
```

通过这种方式，模块可以获得对 root 帐户和任意数量的子帐户的许可访问权并发送
来自这些帐户的经过身份验证的“消息”。 `Invoker` `callInfo.Caller` 参数在后台使用
区分不同的模块帐户，但无论哪种方式，`Invoker` 返回的函数都只允许`Msg`s
从根或派生模块帐户中通过。

请注意，`Invoker` 本身根据传入的 `CallInfo` 返回一个函数闭包。这将允许客户端实现
将来缓存每个方法类型的调用函数，避免哈希表查找的开销。
这会将这种模块间通信方法的性能开销减少到最低限度
检查权限。

重申一遍，闭包只允许访问授权调用。无论如何都无法访问其他任何东西
名字冒充。

下面是“RootModuleKey”的“grpc.ClientConn.Invoke”实现的粗略草图: 

```go
func (key RootModuleKey) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...grpc.CallOption) error {
  f := key.invoker(CallInfo {Method: method, Caller: ModuleID {ModuleName: key.moduleName}})
  return f(ctx, args, reply)
}
```

### `AppModule` 接线和要求

在 [ADR 031](./adr-031-msg-service.md) 中，引入了 `AppModule.RegisterService(Configurator)` 方法。 支持
模块间通信，我们扩展了 `Configurator` 接口以传入 `ModuleKey` 并允许模块
使用 `RequireServer()` 指定它们对其他模块的依赖: 

```go
type Configurator interface {
   MsgServer() grpc.Server
   QueryServer() grpc.Server

   ModuleKey() ModuleKey
   RequireServer(msgServer interface{})
}
```

`ModuleKey` 被传递给 `RegisterService` 方法本身中的模块，以便 `RegisterServices` 作为单个
配置模块服务的入口点。 这也旨在具有大大减少样板文件的副作用
`app.go`。 目前，`ModuleKey`s 将基于 `AppModuleBasic.Name()` 创建，但更灵活的系统可能是
将来推出。 `ModuleManager` 将在后台处理模块帐户的创建。

由于模块不再能够直接相互访问，因此模块可能具有未满足的依赖关系。 确保;确定
如果模块依赖在启动时解决，则应添加 `Configurator.RequireServer` 方法。 `模块管理器`
将确保在应用程序启动之前可以解决所有使用 RequireServer 声明的依赖项。 一个例子
模块 `foo` 可以像这样声明它对 `x/bank` 的依赖: 

```go
package foo

func (am AppModule) RegisterServices(cfg Configurator) {
  cfg.RequireServer((*bank.QueryServer)(nil))
  cfg.RequireServer((*bank.MsgServer)(nil))
}
```

### 安全注意事项

除了检查“ModuleKey”权限外，还需要采取一些额外的安全预防措施
底层路由器基础设施。

#### 递归和重入

递归或可重入方法调用构成潜在的安全威胁。如果模块 A，这可能是一个问题
在同一个调用中调用模块 B 和模块 B 再次调用模块 A。

路由器系统处理此问题的一种基本方法是维护一个调用堆栈，以防止模块
在调用堆栈中被多次引用，因此不会重新进入。一个 `map[string]interface{}` 表
在路由器中可用于执行此安全检查。

#### 查询

Cosmos SDK 中的查询通常是未经许可的，因此允许一个模块查询另一个模块不应构成
在采取基本预防措施的情况下，任何重大的安全威胁。路由器系统的基本注意事项
需要确保传递给查询方法的 `sdk.Context` 不允许写入存储。这
现在可以使用“CacheMultiStore”来完成，就像目前为“BaseApp”查询所做的那样。

### 内部方法

在许多情况下，我们可能希望模块调用其他模块上的方法，而这些模块根本没有暴露给客户端。为了这
为了达到目的，我们将 `InternalServer` 方法添加到 `Configurator` 中: 

```go
type Configurator interface {
   MsgServer() grpc.Server
   QueryServer() grpc.Server
   InternalServer() grpc.Server
}
```

比如x/staking的Slash必须调用x/staking的Slash，但是我们不想把x/staking的Slash暴露给终端用户
和客户。

内部 protobuf 服务将在给定模块的相应“internal.proto”文件中定义
原型包。

针对“InternalServer”注册的服务可以从其他模块调用，但不能被外部客户端调用。

内部方法的另一种解决方案可能涉及 [here](https://github.com/cosmos/cosmos-sdk/pull/7459#issuecomment-733807753) 中讨论的钩子/插件。
钩子/插件系统的更详细评估将在此 ADR 的后续跟进中或作为单独的
ADR。

### 授权

默认情况下，模块间路由器要求消息由 GetSigners 返回的第一个签名者发送。这
模块间路由器还应接受授权中间件，例如 [ADR 030](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-030-authz-module.md )。
该中间件将允许帐户以其他方式代表特定模块帐户执行操作。
授权中间件应考虑有效授予某些模块“管理员”权限的需要
其他模块。这将在单独的 ADR 或此 ADR 的更新中解决。

### 未来的工作

未来的其他改进可能包括:

* 自定义代码生成:
    * 简化接口(例如，使用 `sdk.Context` 而不是 `context.Context` 生成代码)
    * 优化模块间调用 - 例如在第一次调用后缓存解析的方法
* 将`StoreKey`s 和`ModuleKey`s 组合到一个接口中，以便模块有一个单独的 OCAPs 句柄
* 代码生成，使模块间通信更高效
* 将 `ModuleKey` 创建与 `AppModuleBasic.Name()` 解耦，以便应用程序可以覆盖根模块帐户名称
* 模块间挂钩和插件

## 备择方案

### MsgServices 与`x/capability`

`x/capability` 模块确实提供了一个合适的对象能力实现，可以被
Cosmos SDK，甚至可以用于模块间 OCAP，如 [\#5931](https://github.com/cosmos/cosmos-sdk/issues/5931) 中所述。

本 ADR 中描述的方法的优势主要在于它如何与 Cosmos SDK 的其他部分集成，
具体来说:

* protobuf 以便:
    * 可以利用接口的代码生成来获得更好的开发用户体验
    * 使用 [buf](https://docs.buf.build/break-overview) 对模块接口进行版本控制和检查是否损坏
* 根据 ADR 028 的子模块帐户
* 一般的 `Msg` 传递范式和签名者的方式由 `GetSigners` 指定

此外，这是对 Keepers 的完全替代，可以应用于 _all_ 模块间通信，而
#5931 中的`x/capability` 方法需要逐个应用。 

## Consequences

### Backwards Compatibility

此 ADR 旨在为模块之间具有更大长期兼容性的场景提供途径。
在短期内，这可能会导致某些过于宽松和/或
完全替换 `Keeper` 接口。 
### Positive

- 可以更轻松地实现稳定的模块间接口的 Keepers 的替代方案
- 适当的模块间 OCAP
- 改进的模块开发者 DevX，正如一些参与者所评论的
     [架构审查电话会议，12 月 3 日](https://hackmd.io/E0wxxOvRQ5qVmTf6N_k84Q)
- 为可以大大简化的“app.go”奠定基础
- 可以设置路由器来强制执行模块到模块调用的原子事务 

### Negative

- 采用这种方式的模块将需要大量重构 

### Neutral

## Test Cases [optional]

## References

- [ADR 021](./adr-021-protobuf-query-encoding.md)
- [ADR 031](./adr-031-msg-service.md)
- [ADR 028](./adr-028-public-key-addresses.md)
- [ADR 030 draft](https://github.com/cosmos/cosmos-sdk/pull/7105)
- [Object-Capability Model](../docs/core/ocap.md)
