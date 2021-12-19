# 事件

Feegrant 模块发出以下事件: 
# Msg Server

### MsgGrantAllowance

| Type     | Attribute Key | Attribute Value    |
| -------- | ------------- | ------------------ |
| message  | action        | set_feegrant       |
| message  | granter       | {granterAddress}   |
| message  | grantee       | {granteeAddress}   |

### MsgRevokeAllowance

| Type     | Attribute Key | Attribute Value    |
| -------- | ------------- | ------------------ |
| message  | action        | revoke_feegrant    |
| message  | granter       | {granterAddress}   |
| message  | grantee       | {granteeAddress}   |

### Exec fee allowance

| Type     | Attribute Key | Attribute Value    |
| -------- | ------------- | ------------------ |
| message  | action        | use_feegrant       |
| message  | granter       | {granterAddress}   |
| message  | grantee       | {granteeAddress}   |
