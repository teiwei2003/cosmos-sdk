# `staking`

## 概要

このペーパーでは、2016年6月の[Cosmos Whitepaper(https://cosmos.network/about/whitepaper)]で最初に説明されたCosmos-SDKのステーキングモジュールについて説明します。

このモジュールにより、Cosmos-SDKベースのブロックチェーンが高度なプルーフオブステークシステムをサポートできるようになります。
このシステムでは、チェーンのネイティブステーキングトークンの所有者がバリデーターになり、トークンをバリデーターに委任して、最終的にシステムに有効なバリデーターセットを決定できます。

このモジュールは、Cosmosネットワークの最初のハブであるCosmosハブで使用されます。

## コンテンツ

1. **[State](01_state.md)**
    - [Pool](01_state.md#pool)
    - [LastTotalPower](01_state.md#lasttotalpower)
    - [Params](01_state.md#params)
    - [Validator](01_state.md#validator)
    - [Delegation](01_state.md#delegation)
    - [UnbondingDelegation](01_state.md#unbondingdelegation)
    - [Redelegation](01_state.md#redelegation)
    - [Queues](01_state.md#queues)
    - [HistoricalInfo](01_state.md#historicalinfo)
2. **[State Transitions](02_state_transitions.md)**
    - [Validators](02_state_transitions.md#validators)
    - [Delegations](02_state_transitions.md#delegations)
    - [Slashing](02_state_transitions.md#slashing)
3. **[Messages](03_messages.md)**
    - [MsgCreateValidator](03_messages.md#msgcreatevalidator)
    - [MsgEditValidator](03_messages.md#msgeditvalidator)
    - [MsgDelegate](03_messages.md#msgdelegate)
    - [MsgUndelegate](03_messages.md#msgundelegate)
    - [MsgBeginRedelegate](03_messages.md#msgbeginredelegate)
4. **[Begin-Block](04_begin_block.md)**
    - [Historical Info Tracking](04_begin_block.md#historical-info-tracking)
5. **[End-Block](05_end_block.md)**
    - [Validator Set Changes](05_end_block.md#validator-set-changes)
    - [Queues](05_end_block.md#queues-)
6. **[Hooks](06_hooks.md)**
7. **[Events](07_events.md)**
    - [EndBlocker](07_events.md#endblocker)
    - [Msg's](07_events.md#msg's)
8. **[Parameters](08_params.md)**
