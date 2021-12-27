## 概要

このドキュメントでは、フィーグラントモジュールを指定します。 完全なADRについては、[Fee Grant ADR-029](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/docs/architecture/adr-029-fee-grant-module.mdを参照してください )

このモジュールを使用すると、アカウントは手数料の許容範囲を付与し、アカウントからの手数料を使用できます。 被付与者は、十分な料金を維持する必要なしに、任意のトランザクションを実行できます。

## 目次

1. **[Concepts](01_concepts.md)**
    - [Grant](01_concepts.md#grant)
    - [Fee Allowance types](01_concepts.md#fee-allowance-types)
    - [BasicAllowance](01_concepts.md#basicallowance)
    - [PeriodicAllowance](01_concepts.md#periodicallowance)
    - [FeeAccount flag](01_concepts.md#feeaccount-flag)
    - [Granted Fee Deductions](01_concepts.md#granted-fee-deductions)
    - [Gas](01_concepts.md#gas)
2. **[State](02_state.md)**
    - [FeeAllowance](02_state.md#feeallowance)
3. **[Messages](03_messages.md)**
    - [Msg/GrantAllowance](03_messages.md#msggrantallowance)
    - [Msg/RevokeAllowance](03_messages.md#msgrevokeallowance)
4. **[Events](04_events.md)**
    - [MsgGrantAllowance](04_events.md#msggrantallowance)
    - [MsgRevokeAllowance](04_events.md#msgrevokeallowance)
    - [Exec fee allowance](04_events.md#exec-fee-allowance)
5. **[Client](05_client.md)**
    - [CLI](05_client.md#cli)
    - [gRPC](05_client.md#grpc)
