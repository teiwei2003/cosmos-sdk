# 消息

在本节中，我们将描述 authz 模块的消息处理。

## 消息授权

使用“MsgGrant”消息创建授权许可。
如果已经有对 `(granter, grantee, Authorization)` 三元组的授权，那么新的授权会覆盖之前的授权。要更新或扩展现有授权，应创建具有相同“(授权者、被授权者、授权)”三元组的新授权。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L32-L37

如果出现以下情况，消息处理应该会失败:

- 授予者和受赠者的地址相同。
- 提供的`Expiration` 时间小于当前的 unix 时间戳。
- 如果没有实现`Grant.Authorization`。
- `Authorization.MsgTypeURL()` 未在路由器中定义(应用路由器中没有定义的处理程序来处理该消息类型)。

## 消息撤销

可以使用 `MsgRevoke` 消息删除授权。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L60-L64

如果出现以下情况，消息处理应该会失败:

- 授予者和受赠者的地址相同。
- 假设 `MsgTypeUrl` 为空。

注意:如果授权已过期，`MsgExec` 消息将删除授权。

## 消息执行

当受赠人想要代表受赠人执行交易时，他们必须发送“MsgExec”。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/tx.proto#L47-L53

如果出现以下情况，消息处理应该会失败:

- 前提是未实现“授权”。
- 受让人无权运行交易。
- 如果授予的授权已过期。 