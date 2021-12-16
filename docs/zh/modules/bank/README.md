# `x/银行`

## 摘要

本文档指定了 Cosmos SDK 的银行模块。

银行模块负责处理多资产硬币之间的转移
帐户和跟踪必须以不同方式工作的特殊情况的伪转移
具有特定类型的账户(特别是授权/取消授权以进行归属
帐户)。它公开了几个具有不同安全功能的接口
与必须改变用户余额的其他模块的交互。

此外，银行模块跟踪并提供总计查询支持。
应用程序中使用的所有资产的供应。

该模块将用于 Cosmos Hub。

## 供应

`supply` 功能:

- 被动跟踪链中硬币的总供应量，
- 为模块提供了一种模式来保存/与`Coins`交互，并且
- 引入不变检查来验证链的总供应量。

### 总供应量

网络的总“供应”等于所有硬币的总和
帐户。每次铸造“硬币”时都会更新总供应量(例如:作为一部分
通货膨胀机制)或烧毁(例如:由于削减或如果治理
提案被否决)。

## 模块帐户

供应功能引入了一种新类型的“auth.Account”，它可以被
分配代币的模块，在特殊情况下铸造或销毁代币。在一个基地
级别这些模块帐户能够发送/接收令牌
`auth.Account`s 和其他模块帐户。这种设计取代了以前的
替代设计，其中，为了持有代币，模块会烧掉传入的代币
来自发件人帐户的令牌，然后在内部跟踪这些令牌。之后，
为了发送令牌，该模块需要有效地铸造令牌
在目标帐户中。新设计消除了之间的重复逻辑
模块来执行此记帐。

`ModuleAccount` 接口定义如下: 

```go
type ModuleAccount interface {
  auth.Account               // same methods as the Account interface

  GetName() string           // name of the module; used to obtain the address
  GetPermissions() []string  // permissions of module account
  HasPermission(string) bool
}
```

> **警告！**
> 任何允许直接或间接发送资金的模块或消息处理程序必须明确保证这些资金不能发送到模块账户(除非被允许)。

供应`Keeper` 还为auth `Keeper` 引入了新的包装函数
以及与“ModuleAccount”相关的银行“Keeper”，以便能够
到:

- 通过提供`Name`来获取和设置`ModuleAccount`s。
- 在其他`ModuleAccount`s 或标准`Account`s 之间发送硬币
  (`BaseAccount` 或 `VestingAccount`)只传递 `Name`。
-`ModuleAccount` 的`Mint` 或`Burn` 硬币(受限于其权限)。

### 权限

每个 `ModuleAccount` 都有一组不同的权限，提供不同的权限
对象执行某些操作的能力。权限需要
在创建供应“Keeper”时注册，以便每次
`ModuleAccount` 调用允许的函数，`Keeper` 可以查找
该特定帐户的权限并执行或不执行该操作。

可用的权限是:

- `Minter`:允许模块铸造特定数量的硬币。
- `Burner`:允许模块燃烧特定数量的硬币。
- `Staking`:允许模块委托和取消委托特定数量的硬币。

## 内容

1. **[状态](01_state.md)**
2. **[Keepers](02_keepers.md)**
   - [常见类型](02_keepers.md#common-types)
   - [BaseKeeper](02_keepers.md#basekeeper)
   - [SendKeeper](02_keepers.md#sendkeeper)
   - [ViewKeeper](02_keepers.md#viewkeeper)
3. **[消息](03_messages.md)**
   - [MsgSend](03_messages.md#msgsend)
4. **[事件](04_events.md)**
   - [处理程序](04_events.md#handlers)
5. **[参数](05_params.md)**
6. **[客户端](06_client.md)**
    - [CLI](06_client.md#cli)
    - [gRPC](06_client.md#grpc) 