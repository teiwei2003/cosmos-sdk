# 遥测

使用自定义指标和遥测收集有关您的应用程序和模块的相关见解。 {概要}

Cosmos SDK 使运营商和开发人员能够深入了解
他们的应用程序通过使用“遥测”包。 Cosmos SDK 目前支持
启用内存和普罗米修斯作为遥测接收器。 这允许查询和抓取的能力
来自单个公开 API 端点的指标 -- `/metrics?format={text|prometheus}`，默认值为
`文本`。

如果通过配置启用遥测，则通过以下方式注册单个全局指标收集器
[go-metrics](https://github.com/armon/go-metrics) 库。 这允许发射和收集
指标通过简单的 API 调用。

例子: 

```go
func EndBlocker(ctx sdk.Context, k keeper.Keeper) {
  defer telemetry.ModuleMeasureSince(types.ModuleName, time.Now(), telemetry.MetricKeyEndBlocker)

 //...
}
```

开发人员可以直接使用 `telemetry` 包，它提供了指标 API 的包装器
包括添加有用的标签，或者他们必须直接使用 `go-metrics` 库。 最好
为度量添加尽可能多的上下文和足够的维度，因此“遥测”包
建议。 无论使用何种包或方法，Cosmos SDK 都支持以下指标
类型:

* 仪表
* 总结
* 计数器

## 标签

模块的某些组件将自动将其名称添加为标签(例如“BeginBlock”)。
运营商还可以为应用程序提供一组全局标签，这些标签将应用于所有
使用`telemetry` 包(例如chain-id)发出的指标。 全局标签以列表形式提供
[名称，值] 元组。

例子: 

```toml
global-labels = [
  ["chain_id", "chain-OfXo4V"],
]
```

## 基数

基数是关键，特别是标签和键基数。 基数是有多少个唯一值
有的东西。 所以自然要在粒度和施加的压力之间进行权衡
在索引、抓取和查询性能方面的遥测接收器上。

开发人员应该注意支持具有足够维度和粒度的指标
有用，但不会增加超出接收器限制的基数。 一般的经验法则是不要
超过 10 的基数。

考虑以下具有足够粒度和足够基数的示例:

* 开始/结束拦截器时间
* 使用的 TX GAS费
* 使用块GAS费
* 铸造的代币数量
* 创建的帐户数量

以下示例暴露了太多的基数，甚至可能没有用处:

* 有金额的账户间转账
*来自唯一地址的投票/存款金额

## 支持的指标 

| Metric                          | Description                                                                               | Unit            | Type    |
|:--------------------------------|:------------------------------------------------------------------------------------------|:----------------|:--------|
| `tx_count`                      | Total number of txs processed via `DeliverTx`                                             | tx              | counter |
| `tx_successful`                 | Total number of successful txs processed via `DeliverTx`                                  | tx              | counter |
| `tx_failed`                     | Total number of failed txs processed via `DeliverTx`                                      | tx              | counter |
| `tx_gas_used`                   | The total amount of gas used by a tx                                                      | gas             | gauge   |
| `tx_gas_wanted`                 | The total amount of gas requested by a tx                                                 | gas             | gauge   |
| `tx_msg_send`                   | The total amount of tokens sent in a `MsgSend` (per denom)                                | token           | gauge   |
| `tx_msg_withdraw_reward`        | The total amount of tokens withdrawn in a `MsgWithdrawDelegatorReward` (per denom)        | token           | gauge   |
| `tx_msg_withdraw_commission`    | The total amount of tokens withdrawn in a `MsgWithdrawValidatorCommission` (per denom)    | token           | gauge   |
| `tx_msg_delegate`               | The total amount of tokens delegated in a `MsgDelegate`                                   | token           | gauge   |
| `tx_msg_begin_unbonding`        | The total amount of tokens undelegated in a `MsgUndelegate`                               | token           | gauge   |
| `tx_msg_begin_begin_redelegate` | The total amount of tokens redelegated in a `MsgBeginRedelegate`                          | token           | gauge   |
| `tx_msg_ibc_transfer`           | The total amount of tokens transferred via IBC in a `MsgTransfer` (source or sink chain)  | token           | gauge   |
| `ibc_transfer_packet_receive`   | The total amount of tokens received in a `FungibleTokenPacketData` (source or sink chain) | token           | gauge   |
| `new_account`                   | Total number of new accounts created                                                      | account         | counter |
| `gov_proposal`                  | Total number of governance proposals                                                      | proposal        | counter |
| `gov_vote`                      | Total number of governance votes for a proposal                                           | vote            | counter |
| `gov_deposit`                   | Total number of governance deposits for a proposal                                        | deposit         | counter |
| `staking_delegate`              | Total number of delegations                                                               | delegation      | counter |
| `staking_undelegate`            | Total number of undelegations                                                             | undelegation    | counter |
| `staking_redelegate`            | Total number of redelegations                                                             | redelegation    | counter |
| `ibc_transfer_send`             | Total number of IBC transfers sent from a chain (source or sink)                          | transfer        | counter |
| `ibc_transfer_receive`          | Total number of IBC transfers received to a chain (source or sink)                        | transfer        | counter |
| `ibc_client_create`             | Total number of clients created                                                           | create          | counter |
| `ibc_client_update`             | Total number of client updates                                                            | update          | counter |
| `ibc_client_upgrade`            | Total number of client upgrades                                                           | upgrade         | counter |
| `ibc_client_misbehaviour`       | Total number of client misbehaviours                                                      | misbehaviour    | counter |
| `ibc_connection_open-init`      | Total number of connection `OpenInit` handshakes                                          | handshake       | counter |
| `ibc_connection_open-try`       | Total number of connection `OpenTry` handshakes                                           | handshake       | counter |
| `ibc_connection_open-ack`       | Total number of connection `OpenAck` handshakes                                           | handshake       | counter |
| `ibc_connection_open-confirm`   | Total number of connection `OpenConfirm` handshakes                                       | handshake       | counter |
| `ibc_channel_open-init`         | Total number of channel `OpenInit` handshakes                                             | handshake       | counter |
| `ibc_channel_open-try`          | Total number of channel `OpenTry` handshakes                                              | handshake       | counter |
| `ibc_channel_open-ack`          | Total number of channel `OpenAck` handshakes                                              | handshake       | counter |
| `ibc_channel_open-confirm`      | Total number of channel `OpenConfirm` handshakes                                          | handshake       | counter |
| `ibc_channel_close-init`        | Total number of channel `CloseInit` handshakes                                            | handshake       | counter |
| `ibc_channel_close-confirm`     | Total number of channel `CloseConfirm` handshakes                                         | handshake       | counter |
| `tx_msg_ibc_recv_packet`        | Total number of IBC packets received                                                      | packet          | counter |
| `tx_msg_ibc_acknowledge_packet` | Total number of IBC packets acknowledged                                                  | acknowledgement | counter |
| `ibc_timeout_packet`            | Total number of IBC timeout packets                                                       | timeout         | counter |
| `abci_check_tx`                 | Duration of ABCI `CheckTx`                                                                | ms              | summary |
| `abci_deliver_tx`               | Duration of ABCI `DeliverTx`                                                              | ms              | summary |
| `abci_commit`                   | Duration of ABCI `Commit`                                                                 | ms              | summary |
| `abci_query`                    | Duration of ABCI `Query`                                                                  | ms              | summary |
| `abci_begin_block`              | Duration of ABCI `BeginBlock`                                                             | ms              | summary |
| `abci_end_block`                | Duration of ABCI `EndBlock`                                                               | ms              | summary |
| `begin_blocker`                 | Duration of `BeginBlock` for a given module                                               | ms              | summary |
| `end_blocker`                   | Duration of `EndBlock` for a given module                                                 | ms              | summary |
| `store_iavl_get`                | Duration of an IAVL `Store#Get` call                                                      | ms              | summary |
| `store_iavl_set`                | Duration of an IAVL `Store#Set` call                                                      | ms              | summary |
| `store_iavl_has`                | Duration of an IAVL `Store#Has` call                                                      | ms              | summary |
| `store_iavl_delete`             | Duration of an IAVL `Store#Delete` call                                                   | ms              | summary |
| `store_iavl_commit`             | Duration of an IAVL `Store#Commit` call                                                   | ms              | summary |
| `store_iavl_query`              | Duration of an IAVL `Store#Query` call                                                    | ms              | summary |

## 下一个 {hide}

Learn about the [object-capability](./ocap.md) model {hide}
