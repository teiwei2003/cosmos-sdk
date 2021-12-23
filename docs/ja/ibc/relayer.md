# リピータ

## 前提条件の読書

- [IBCの概要](./overview.md) {prereq}
- [イベント](https://github.com/cosmos/cosmos-sdk/blob/master/docs/core/events.md) {prereq}

## イベント

基盤となるアプリケーションによって処理されたトランザクションごとにイベントを発行して、実行を示します
一部のお客様は、ロジックを理解したいと思うかもしれません。 これは、IBCパケットを中継するときに非常に役立ちます。
IBCを使用するメッセージは、実行で定義された対応するTAOロジックのイベントを発行します。
[IBCイベント仕様]（https://github.com/cosmos/ibc-go/blob/main/modules/core/spec/06_events.md）。

Cosmos SDKでは、すべてのメッセージに「メッセージ」タイプのイベントがあると見なすことができます。
属性キー `action`、および送信されたメッセージのタイプを示す属性値
（ `channel_open_init`は` MsgChannelOpenInit`の属性値になります）。 リピーターがクエリを実行する場合
トランザクションイベントの場合、このイベントタイプとプロパティキーのペアを使用して、メッセージイベントを分割できます。

属性キー「モジュール」を持つイベントタイプ「メッセージ」は、単一のイベントに対して複数回発行される場合があります
アプリケーションのコールバックが原因で生成されたメッセージ。 実行されたTAOロジックは、次の結果になると想定できます。
属性値が「ibc_ <submodulename>」のモジュールイベントが送信されます（02-clientは「ibc_client」を送信します）。

### テンダーミントを購読する

[TendermintのWebsocket]（https://docs.tendermint.com/master/rpc/）を介してTendermintRPCメソッド `Subscribe`を呼び出すと、次のメソッドを使用してイベントが返されます。
それらのテンダーミントの内部表現。 彼らのようにイベントのリストを受け取る代わりに
送信後、Tendermintはタイプ `map [string] [] string`を返します。
`<event_type>。<attribute_key>`から `attribute_value`へ。 これは、イベントの抽出につながります
注文は簡単ではありませんが、それでも可能です。

中継者は `message.action`キーを使用して、トランザクション内のメッセージの数を抽出する必要があります
そして、送信されたIBCトランザクションのタイプ。 文字列配列内のIBCトランザクションごと
`message.action`、必要な情報は他のイベントフィールドから抽出する必要があります。 もしも
`send_packet`は` message.action`値のインデックス2に表示され、リピーターは使用する必要があります
キー「send_packet.packet_sequence」のインデックス2の値。 このプロセスはすべての人に繰り返す必要があります
データパケットを中継するために必要な情報。

## 実装例

- [Golang Relayer](https://github.com/iqlusioninc/relayer)
- [Hermes](https://github.com/informalsystems/ibc-rs/tree/master/relayer)
- [Typescript Relayer](https://github.com/confio/ts-relayer) 