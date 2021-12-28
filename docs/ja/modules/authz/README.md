# `authz`

## コンテンツ

## 概要

`x/authz`はCosmosSDKモジュールの実装であり、[ADR 30](../../../docs/architecture/adr-030-authz-module.md)によると、
あるアカウント(the grant)から別のアカウント(the grant)に任意のアクセス許可を付与します。 `Authorization`インターフェースの実装は、特定のメッセージサービスメソッドの承認を1つずつ付与するために使用する必要があります。

1。**[コンセプト](01_concepts.md)**
   - [承認と付与](01_concepts.md#Authorization-and-Grant)
   - [組み込みの承認](01_concepts.md#Built-in-Authorizations)
   - [ガス](01_concepts.md#gas)
2。**[State](02_state.md)**
3。**[メッセージ](03_messages.md)**
   - [MsgGrant](03_messages.md#MsgGrant)
   - [MsgRevoke](03_messages.md#MsgRevoke)
   - [MsgExec](03_messages.md#MsgExec)
4。**[イベント](04_events.md)**
5。**[クライアント](05_client.md)**
   - [CLI](05_client.md#cli)
   - [gRPC](05_client.md#grpc)
   - [REST](05_client.md#rest)