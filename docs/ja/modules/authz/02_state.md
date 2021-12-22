# 状态

## 授予

授权通过组合授权者地址(授权者的地址字节)、被授权者地址(被授权者的地址字节)和授权类型(其类型 URL)来标识。 因此，我们只允许对 (granter, grantee, Authorization) 三元组进行一次授权。

- 授予:`0x01 | granter_address_len (1 字节) | granter_address_bytes | grantee_address_len (1 字节) | grantee_address_bytes | msgType_bytes-> ProtocolBuffer(AuthorizationGrant)`

grant 对象封装了一个 `Authorization` 类型和一个过期时间戳:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/authz/v1beta1/authz.proto#L21-L26 