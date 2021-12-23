# テレメトリー

カスタムメトリックとテレメトリを使用して、アプリケーションとモジュールに関する関連する洞察を収集します。 {まとめ}

Cosmos SDKを使用すると、オペレーターと開発者は理解できます
それらのアプリケーションは、「テレメトリ」パッケージを使用して行われます。 CosmosSDKは現在サポートしています
テレメトリ受信機としてメモリとPrometheusを有効にします。 これにより、クエリとクロールが可能になります
単一のパブリックAPIエンドポイントからのメトリック-`/metrics？format = {text | prometheus} `、デフォルト値は
`テキスト`。

構成によってテレメトリが有効になっている場合、単一のグローバルメトリックコレクターは次の方法で登録されます
[go-metrics](https://github.com/armon/go-metrics)ライブラリ。 これにより、起動と収集が可能になります
インジケーターは単純なAPIを介して呼び出されます。

例： 

```go
func EndBlocker(ctx sdk.Context, k keeper.Keeper) {
  defer telemetry.ModuleMeasureSince(types.ModuleName, time.Now(), telemetry.MetricKeyEndBlocker)

 //...
}
```

開発者は、インジケーターAPIのラッパーを提供する `telemetry`パッケージを直接使用できます。
有用なタグの追加を含む、またはそれらはgo-metricsライブラリを直接使用する必要があります。 多くの
可能な限り多くのコンテキストと十分なディメンションをメジャーに追加して、「テレメトリ」パッケージを作成します
提案。 使用するパッケージや方法に関係なく、CosmosSDKは次のインジケーターをサポートします
タイプ：

*  メーター
*  要約する
*  カウンター

## ラベル

モジュールの特定のコンポーネントは、それらの名前をタグとして自動的に追加します(たとえば、「BeginBlock」)。
オペレーターは、アプリケーションに一連のグローバルタグを提供することもできます。これは、すべてのユーザーに適用されます。
`telemetry`パッケージを使用して送信されたメトリック(chain-idなど)。 グローバルタグはリストとして提供されます
[名前、値]タプル。

例： 

```toml
global-labels = [
  ["chain_id", "chain-OfXo4V"],
]
```

## ベース

カーディナリティ、特にラベルとキーカーディナリティが重要です。 ベースは、一意の値の数です
何か。 したがって、当然、粒度と加えられる圧力の間にはトレードオフがあります。
インデックス作成、クロール、クエリパフォーマンスのためのテレメトリレシーバー。

開発者は、十分なディメンションと粒度でメトリックをサポートすることに注意を払う必要があります
便利ですが、受信機の制限を超えてベースを増やすことはありません。 一般的な経験則は、
10を超える基数。

十分な粒度と十分なカーディナリティを持つ次の例を検討してください。

*  インターセプター時間の開始/終了
*  使用されるTXガス
* ブロックガスを使用
* 鋳造されたコインの数
* 作成されたアカウントの数

次の例では、カーディナリティが多すぎるため、役に立たない場合があります。

* 金額のある口座間の送金
* 一意のアドレスからの投票/デポジット額

## サポートされているインジケーター 

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

## 次へ{非表示}

[object-capability](./ocap.md)モデルについて学ぶ{hide}
