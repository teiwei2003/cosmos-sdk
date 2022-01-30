# `Msg`サービス

Protobuf `Msg`サービス処理[メッセージ](./messages-and-queries.md#messages)。 Protobuf `Msg`サービスは、それらが定義されているモジュールに固有であり、モジュールで定義されているメッセージのみを処理します。これらは、[`DeliverTx`](../core/baseapp.md#delivertx)中に` BaseApp`から呼び出されます。 {まとめ}

## 読むための前提条件

-[モジュールマネージャー](。/module-manager.md){前提条件}
-[メッセージとクエリ](./messages-and-queries.md){prereq}

## モジュール `Msg`サービスの実装

各モジュールは、Protobuf `Msg`サービスを定義する必要があります。このサービスは、要求の処理(` sdk.Msg`の実装)と応答の返送を担当します。

[ADR 031](../architecture/adr-031-msg-service.md)でさらに説明されているように、このメソッドの利点は、リターンタイプが明確に指定され、サーバーコードとクライアントコードが生成されることです。

Protobufは、 `Msg`サービスの定義に従って` MsgServer`インターフェースを生成します。モジュール開発者の責任は、各 `sdk.Msg`が受信されたときに発生するはずの状態遷移ロジックを実装することによってこのインターフェイスを実装することです。たとえば、次は `x/bank`用に生成された` MsgServer`インターフェースで、2つの `sdk.Msg`を公開します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/bank/types/tx.pb.go#L285-L291

可能であれば、既存のモジュールの[`Keeper`](keeper.md)は` MsgServer`を実装する必要があります。そうでない場合は、通常は `。/keeper/msg_server.go`に` Keeper`が埋め込まれた `msgServer`構造を作成できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/bank/keeper/msg_server.go#L14-L16

`msgServer`メソッドは` sdk.UnwrapSDKContext`を使用して、 `context.Context`パラメータメソッドから` sdk.Context`を取得できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/bank/keeper/msg_server.go#L27-L28

`sdk.Msg`の処理は通常、次の2つの手順に従います。

-最初に、[ステートフル]チェックを実行して、[メッセージ]が有効であることを確認します。この段階で、 `message`の` ValidateBasic() `メソッドが呼び出されました。これは、メッセージの*ステートレス*チェックが実行されたことを意味します(たとえば、パラメーター形式が正しいことを確認するため)。 `msgServer`メソッドで実行されるチェックは、よりコストがかかり、状態へのアクセスが必要になる可能性があります。たとえば、[転送]メッセージの[msgServer]メソッドは、送信アカウントに実際に転送を実行するのに十分な資金があるかどうかを確認する場合があります。状態にアクセスするには、 `msgServer`メソッドが[` keeper`'s](./keeper.md)のgetter関数を呼び出す必要があります。
-次に、チェックが成功すると、 `msgServer`メソッドは[` keeper`s](./keeper.md)setter関数を呼び出して、実際に状態遷移を実行します。

戻る前に、 `msgServer`メソッドは通常、` ctx`に保存されている `EventManager`を介して1つ以上の[events](../core/events.md)を送信します。

```go
ctx.EventManager()。EmitEvent(
sdk.NewEvent(
eventType、//例:メッセージの場合はsdk.EventTypeMessage、モジュールで定義されたカスタムイベントの場合はtypes.CustomEventType
sdk.NewAttribute(attributeKey、attributeValue)、
)、
    )。
```

これらのイベントは基盤となるコンセンサスエンジンに転送され、サービスプロバイダーはこれらのイベントを使用してアプリケーションの周囲にサービスを実装できます。イベントの詳細については、[こちら](../core/events.md)をクリックしてください。

呼び出された `msgServer`メソッドは、` proto.Message`応答と `error`を返します。次に、 `sdk.WrapServiceResult(ctx sdk.Context、res proto.Message、err error)`を使用して、これらの戻り値を `* sdk.Result`または` error`にラップします。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc2/baseapp/msg_service_router.go#L104-L104

このメソッドは、 `res`パラメーターをprotobufにマーシャリングし、` ctx.EventManager() `のイベントを` sdk.Result`にアタッチする役割を果たします。

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/proto/cosmos/base/abci/v1beta1/abci.proto#L81-L95

この図は、Protobuf `Msg`サービスの一般的な構造と、メッセージがモジュールを介してどのように伝播されるかを示しています。

！[トランザクションフロー](../uml/svg/transaction_flow.svg)

## Amino`LegacyMsg`s

### `ハンドラー`タイプ

CosmosSDKで定義されている `handler`タイプは非推奨になり、[` Msg` service](#implementation-of-a-module-msg-service)に置き換えられます。

以下は、 `handler`関数の典型的な構造です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc2/types/handler.go#L4-L4

それを分解しましょう:

-[`LegacyMsg`](./messages-and-queries.md#messages)は、処理されている実際のオブジェクトです。
-[`Context`](../core/context.md)には、` msg`の処理に必要なすべての情報と、ブランチの最新の状態が含まれています。 `msg`プロセスが成功すると、` ctx`に含まれる状態のブランチバージョンがメイン状態(ブランチ)に書き込まれます。
-[BaseApp]に返される[* Result]には、(とりわけ)[handler]と[events](../core/events.md)の実行に関する情報が含まれています。

モジュール[handler]は通常、モジュールフォルダの[./handler.go]ファイルに実装されています。[モジュールマネージャー](./module-manager.md)は、モジュールの `ハンドラー`をに追加するために使用されます
[アプリの `router`](../core/baseapp.md#message-routing)` Route() `メソッドを使用します。いつもの、
マネージャの `Route()`メソッドは、 `handler.go`で定義された` NewHandler() `メソッドを呼び出すルートを作成するだけです。

+++ https://github.com/cosmos/cosmos-sdk/blob/228728cce2af8d494c8b4e996d011492139b04ab/x/gov/module.go#L143-L146 

### 埋め込む

`NewHandler`関数は、通常はswitchステートメントを使用して、` LegacyMsg`を適切な処理関数に割り当てます。

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/x/bank/handler.go#L13-L29

まず、 `NewHandler`関数は、コンテキストに新しい` EventManager`を設定して、各 `msg`イベントを分離します。
次に、単純なスイッチが、 `LegacyMsg`タイプに基づいて適切な` handler`を呼び出します。

この点で、各モジュール `LegacyMsg`は` handler`の機能を実装する必要があります。 これには、LegacyMsgタイプの手動ハンドラー登録も含まれます。
`handler`の関数は、` * Result`と `error`を返す必要があります。

## テレメトリ

メッセージを処理するときに、 `msgServer`メソッドから新しい[テレメトリインジケーター](../core/telemetry.md)を作成できます。

これは、 `x/auth/vesting`モジュールの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/auth/vesting/msg_server.go#L73-L85

## 次へ{hide}

[クエリサービス](./query-services.md){hide}を理解する 