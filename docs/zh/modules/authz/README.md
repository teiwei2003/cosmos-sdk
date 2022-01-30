# `authz`

## 内容

## 摘要

`x/authz` 是 Cosmos SDK 模块的实现，根据 [ADR 30](../../../docs/architecture/adr-030-authz-module.md)，它允许
从一个帐户(授予者)向另一个帐户(被授予者)授予任意权限。 必须使用“授权”接口的实现为特定的消息服务方法一一授予授权。

1. **[概念](01_concepts.md)**
     - [授权和授予](01_concepts.md#Authorization-and-Grant)
     - [内置授权](01_concepts.md#Built-in-Authorizations)
     - [GAS费](01_concepts.md#gas)
2. **[状态](02_state.md)**
3. **[消息](03_messages.md)**
     - [MsgGrant](03_messages.md#MsgGrant)
     - [MsgRevoke](03_messages.md#MsgRevoke)
     - [MsgExec](03_messages.md#MsgExec)
4. **[事件](04_events.md)**
5. **[客户端](05_client.md)**
     - [CLI](05_client.md#cli)
     - [gRPC](05_client.md#grpc)
     - [REST](05_client.md#rest) 