# 概念

## 授权和授予

`x/authz` 模块定义接口和消息授予执行操作的权限
代表一个账户到其他账户。该设计在 [ADR 030](../../../docs/architecture/adr-030-authz-module.md) 中定义。

*grant* 是受让人代表授予人执行消息的许可。
授权是一个接口，必须由具体的授权逻辑实现以验证和执行授权。授权是可扩展的，甚至可以在定义 Msg 方法的模块之外为任何 Msg 服务方法定义。有关更多详细信息，请参阅下一节中的“SendAuthorization”示例。

**注意:** authz 模块不同于 [auth(身份验证)](../modules/auth/) 模块，后者负责指定基础交易和账户类型。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/authz/authorizations.go#L11-L25

## 内置授权

Cosmos SDK `x/authz` 模块带有以下授权类型:

### 发送授权

`SendAuthorization` 为 `cosmos.bank.v1beta1.MsgSend` 消息实现了 `Authorization` 接口。它需要一个 `SpendLimit` 来指定受赠者可以花费的最大代币数量。 `SpendLimit` 会随着代币的花费而更新。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/bank/v1beta1/authz.proto#L10-L19

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/types/send_authorization.go#L25-L40

- `spend_limit` 跟踪授权中剩余的硬币数量。

### 通用授权

`GenericAuthorization` 实现了 `Authorization` 接口，该接口授予代表授予者帐户执行提供的 Msg 的不受限制的权限。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/authz.proto#L14-L19

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/authz/generic_authorization.go#L18-L31

- `msg` 存储消息类型的 URL。

## GAS费

为了防止 DoS 攻击，使用 `x/authz` 授予 `StakeAuthorization`s 会产生 gas。 `StakeAuthorization` 允许您授权另一个帐户委派、取消委派或重新委派给验证者。授权人可以定义他们允许或拒绝委托的验证器列表。 Cosmos SDK 迭代这些列表，并为两个列表中的每个验证器收取 10 gas。 