# イベント

`イベント`は、アプリケーションの実行に関する情報を含むオブジェクトです。これらは主に、ブロックエクスプローラーやウォレットなどのサービスプロバイダーが、さまざまなメッセージやインデックストランザクションの実行を追跡するために使用します。 {まとめ}

## 読むための前提条件

-[Cosmos SDKアプリケーション分析](../basics/app-anatomy.md){前提条件}
-[テンダーミントイベントドキュメント](https://docs.tendermint.com/master/spec/abci/abci.html#events){前提条件}

## イベント

イベントは、CosmosSDKのABCI`Event`タイプのエイリアスとして実装されます。
次の形式を取ります: `{eventType}。{attributeKey} = {attributeValue}`。

+++ https://github.com/tendermint/tendermint/blob/v0.34.8/proto/tendermint/abci/types.proto#L304-L313

イベントには次のものが含まれます。

-イベントを高レベルで分類するために使用される「タイプ」。たとえば、CosmosSDKは「メッセージ」タイプを使用してイベントを「メッセージ」でフィルタリングします。
-`attributes`リストは、分類されたイベントに関する詳細情報を提供するキーと値のペアです。たとえば、 `" message "`タイプの場合、 `message.action = {some_action}`、 `message.module = {some_module}`、または `message.sender = {some_sender}を使用して、Key-Valueでイベントをフィルタリングできます。ペア `。

::: ヒント
属性値を文字列に解析するには、各属性値の前後に必ず `'`(一重引用符)を追加してください。
:::

イベント、 `type`および` attributes`は、モジュールの**モジュールごとのベース**で定義されます
`/types/events.go`ファイルで、モジュールのProtobuf [` Msg`サービス](../building-modules/msg-services.md)からトリガーされます
[`EventManager`](#eventmanager)を使用する。さらに、各モジュールはそのイベントを次のように記録します
`spec/xx_events.md`。

イベントは、次のABCIメッセージに応答して、基になるコンセンサスエンジンに返されます。

-[`BeginBlock`](./baseapp.md#beginblock)
-[`EndBlock`](./baseapp.md#endblock)
-[`CheckTx`](./baseapp.md#checktx)
-[`DeliverTx`](./baseapp.md#delivertx)

### 例

次の例は、CosmosSDKを使用してイベントをクエリする方法を示しています。 

| Event                                            | Description                                                                                                                                              |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tx.height=23`                                   | Query all transactions at height 23                                                                                                                      |
| `message.action='/cosmos.bank.v1beta1.Msg/Send'` | Query all transactions containing a x/bank `Send` [Service `Msg`](../building-modules/msg-services.md). Note the `'`s around the value.                  |
| `message.action='send'`                          | Query all transactions containing a x/bank `Send` [legacy `Msg`](../building-modules/msg-services.md#legacy-amino-msgs). Note the `'`s around the value. |
| `message.module='bank'`                          | Query all transactions containing messages from the x/bank module. Note the `'`s around the value.                                                       |
| `create_validator.validator='cosmosval1...'`     | x/staking-specific Event, see [x/staking SPEC](../../../cosmos-sdk/x/staking/spec/07_events.md).                                                         |

## イベントマネージャ

Cosmos SDKアプリケーションでは、イベントは「EventManager」と呼ばれる抽象化によって管理されます。
内部的には、「EventManager」はイベントの実行フロー全体のリストを追跡します
トランザクションまたは `BeginBlock`/` EndBlock`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/events.go#L17-L25

`EventManager`には、イベントを管理するための便利なメソッドのセットが付属しています。 方法
モジュールおよびアプリケーション開発者が最もよく使用するのは、追跡する「EmitEvent」です。
`EventManager`のイベント。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/events.go#L33-L37

モジュール開発者は、各メッセージの `EventManager#EmitEvent`を介してイベントの発行を処理する必要があります
`Handler`と各` BeginBlock`/`EndBlock`ハンドラー。 `EventManager`はからアクセスされます
[`Context`](./context.md)、ここで、イベントの放出は通常、次のパターンに従います。 

```go
ctx.EventManager().EmitEvent(
    sdk.NewEvent(eventType, sdk.NewAttribute(attributeKey, attributeValue)),
)
```

モジュールの `handler`関数は、` context`に新しい `EventManager`を設定して、各` message`によって発行されたイベントを分離する必要があります。

```go
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        ctx = ctx.WithEventManager(sdk.NewEventManager())
        switch msg := msg.(type) {
```

詳細については、[`Msg`サービス](../building-modules/msg-services.md)コンセプトドキュメントを参照してください。
イベントが通常どのように実装されているかを確認し、モジュールでEventManagerを使用します。

## イベントを購読する

Tendermintの[Websocket](https://docs.tendermint.com/master/tendermint-core/subscription.html#subscribing-to-events-via-websocket)を使用して、 `subscribe`RPCメソッドを呼び出すことでイベントをサブスクライブできます。 : 

```json
{
  "jsonrpc": "2.0",
  "method": "subscribe",
  "id": "0",
  "params": {
    "query": "tm.event='eventCategory' AND eventType.eventAttribute='attributeValue'"
  }
}
```

購読できる主な `eventCategory`は次のとおりです。

-`NewBlock`: `BeginBlock`および` EndBlock`中にトリガーされたイベントが含まれます。
-`Tx`: `DeliverTx`(つまりトランザクション処理)中にトリガーされたイベントが含まれます。
-`ValidatorSetUpdates`:ブロックを含むバリデーター更新のセット。

これらのイベントは、ブロックが送信された後、 `state`パッケージからトリガーされます。 得られる
[Tendermint Godocページの]イベントカテゴリの完全なリスト(https://godoc.org/github.com/tendermint/tendermint/types#pkg-constants)。

`query`の` type`と `attribute`の値を使用すると、探している特定のイベントをフィルタリングできます。 たとえば、 `transfer`トランザクションは、タイプ` Transfer`のイベントをトリガーし、 `Recipient`と` Sender`を `attributes`として使用します(` bank`モジュールの[`events.go`ファイルで定義](https ://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/bank/types/events.go))。 このイベントを購読する方法は次のとおりです。 

```json
{
  "jsonrpc": "2.0",
  "method": "subscribe",
  "id": "0",
  "params": {
    "query": "tm.event='Tx' AND transfer.sender='senderAddress'"
  }
}
```

ここで、 `senderAddress`は、[` AccAddress`](../basics/accounts.md#addresses)の形式に従ったアドレスです。

## タイプイベント(近日公開)

前述のように、イベントは各モジュールに基づいて定義されます。 イベントタイプとイベント属性を定義するのはモジュール開発者の責任です。 `spec/XX_events.md`ファイルを除いて、残念ながらこれらのイベントタイプと属性を見つけるのは簡単ではないため、Cosmos SDKは[Typed Events](../archive/adr-032-typed-events defined byProtobuf]を使用することをお勧めします).md)は、イベントの発行とクエリに使用されます。

タイプされたイベントの提案はまだ完全には実装されていません。 ドキュメントはまだ利用できません。

## 次へ{非表示}

Cosmos SDKを理解する[テレメトリ](./telemetry.md){非表示}