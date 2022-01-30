# ニュースとクエリ

`Msg`sと` Queries`は、モジュールによって処理される2つの主要なオブジェクトです。 [Msg]サービス、[keeper]、[Query]サービスなど、モジュールで定義されているコアコンポーネントのほとんどは、[メッセージ]と[クエリ]の処理に使用されます。 {まとめ}

## 読むための前提条件

-[Cosmos SDKモジュールの紹介](。/intro.md){前提条件}

## 情報

`Msg`sは、状態遷移をトリガーすることを最終的な目標とするオブジェクトです。それらは[transactions](../core/transactions.md)に含まれており、1つ以上含まれている可能性があります。

トランザクションが基盤となるコンセンサスエンジンからCosmosSDKアプリケーションに中継されると、最初に[`BaseApp`](../core/baseapp.md)によってデコードされます。次に、トランザクションに含まれる各メッセージが抽出され、モジュールの[`Msg`サービス](./msg-services.md)が処理できるように、` BaseApp`の `MsgServiceRouter`を介して適切なモジュールにルーティングされます。トランザクションライフサイクルの詳細については、[こちら](../basics/tx-lifecycle.md)をクリックしてください。

### `Msg`サービス

v0.40以降では、Protobuf`Msg`サービスを定義することがメッセージを処理するための推奨される方法です。 Protobufの `Msg`サービスは、モジュールごとに、通常は` tx.proto`で作成する必要があります(詳細については、[Convention and Naming](../core/encoding.md#faq)を参照してください)。モジュール内のメッセージごとにRPCサービスメソッドを定義する必要があります。

`x/bank`モジュールからの` Msg`サービス定義の例を表示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/proto/cosmos/bank/v1beta1/tx.proto#L10-L17

各 `Msg`サービスメソッドにはパラメータが1つだけ必要であり、これは` sdk.Msg`インターフェイスとProtobuf応答を実装する必要があります。命名規則では、RPCパラメーター `Msg <service-rpc-name>`とRPC応答 `Msg <service-rpc-name> Response`を呼び出します。例えば: 

```
  rpc Send(MsgSend) returns (MsgSendResponse);
```

`sdk.Msg`インターフェースは、[以下](#legacy-amino-msgs)で説明されているAminoの` LegacyMsg`インターフェースの簡略版であり、 `ValidateBasic()`メソッドと `GetSigners()`メソッドのみが含まれています。[Amino `LegacyMsg`s](#legacy-amino-msgs)との下位互換性を保つには、既存の` LegacyMsg`タイプを `service`RPCで定義されたリクエストパラメーターとして使用する必要があります。新しい `sdk.Msg`タイプは` service`定義のみをサポートし、正規の `Msg ...`名を使用する必要があります。

Cosmos SDKは、Protobuf定義を使用して、クライアントおよびサーバーコードを生成します。

* `MsgServer`インターフェースは、` Msg`サービスのサーバーAPIを定義します。これは、実際には[`Msg`サービス](./msg-services.md)ドキュメントで説明されています。
*すべてのRPC要求および応答タイプの構造を生成します。

`RegisterMsgServer`メソッドも生成されます。これは、モジュールの` MsgServer`実装を[`AppModule`インターフェイス](./module-manager.md#appmodule)の` RegisterServices`メソッドに登録するために使用する必要があります。

クライアント(CLIおよびgrpc-gateway)がこれらのURLを登録できるようにするために、CosmosSDKは関数 `RegisterMsgServiceDesc(registry codectypes.InterfaceRegistry、sd * grpc.ServiceDesc)`を提供します。これは、モジュールの[`RegisterInterfaces`]にある必要があります。 (module-manager。md#appmodulebasic)メソッド。プロトタイプによって生成された `＆_Msg_serviceDesc`を` * grpc.ServiceDesc`パラメーターとして使用します。

### レガシーアミノ `LegacyMsg`s

次のメッセージの定義方法はお勧めできません。[`Msg`service](#msg-services)をお勧めします。

Aminoの `LegacyMsg`は、protobufメッセージとして定義できます。メッセージ定義には通常、メッセージの処理に必要なパラメーターのリストが含まれており、エンドユーザーがメッセージを含む新しいトランザクションを作成する場合は、これらのパラメーターを提供します。

`LegacyMsg`には通常、[モジュールのインターフェース](./module-interfaces.md)の1つから呼び出される標準コンストラクターが付属しています。 `message`sは` sdk.Msg`インターフェースも実装する必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/4a1b2fba43b1052ca162b3a1e0b6db6db9c26656/types/tx_msg.go#L10-L33

これは `proto.Message`を拡張し、次のメソッドを含みます。

-`Route()string`:このメッセージのルート名。一般に、モジュール内のすべての[メッセージ]は同じルート、通常はモジュールの名前を持っています。
-`Type()string`:主に[events](../core/events.md)で使用されるメッセージのタイプ。これにより、メッセージ固有の[文字列](通常はメッセージ自体の額面)が返されます。
-`ValidateBasic()エラー `:このメソッドは、` message`([`CheckTx`](../core/baseapp.md#checktx)および[` DeliverTx)](../core/baseapp.md#delivertx))明らかに無効なメッセージを破棄するため。 `ValidateBasic`には、*ステートレス*チェックのみを含める必要があります。つまり、アクセス状態チェックは必要ありません。これには通常、メッセージのパラメータ形式が正しく有効であるかどうかのチェックが含まれます(つまり、[量]は送信に対して厳密に正です)。
-`GetSignBytes()[] byte`:メッセージの正規のバイト表現を返します。署名を生成するために使用されます。
-`GetSigners()[] AccAddress`:署名者のリストを返します。 Cosmos SDKは、トランザクションに含まれるすべての[メッセージ]が、このメソッドによって返されるリストにリストされているすべての署名者によって署名されていることを確認します。

`gov`モジュールからの` message`の実装例を確認してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc1/x/gov/types/msgs.go#L77-L125

## お問い合わせ

[照会]は、アプリケーションのエンドユーザーがインターフェースを介して送信し、ノード全体で処理される情報要求です。フルノードは、コンセンサスエンジンを介して[クエリ]を受信し、ABCIを介してアプリケーションに中継します。次に、モジュールのクエリサービス(./query-services.md)が処理できるように、 `BaseApp`の` queryrouter`を介して適切なモジュールにルーティングされます。 [クエリ]ライフサイクルの詳細については、[こちら](../basics/query-lifecycle.md)をクリックしてください。

### gRPCクエリ

v0.40以降、クエリを定義するための推奨される方法は、[Protobuf Service](https://developers.google.com/protocol-buffers/docs/proto#services)を使用することです。 `query.proto`のモジュールごとに` Query`サービスを作成する必要があります。このサービスは、 `rpc`で始まるエンドポイントを一覧表示します。

以下は、そのような[クエリ]サービス定義の例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/proto/cosmos/auth/v1beta1/query.proto#L12-L23

`proto.Message`sとして、生成された` Response`タイプは、デフォルトで[`fmt.Stringer`](https://golang.org/pkg/fmt/#Stringer)の` String() `メソッドを実装します。

`RegisterQueryServer`メソッドも生成されます。これは、モジュールのクエリサーバーを[` AppModule`インターフェイス](./module-manager.md#appmodule)の `RegisterServices`メソッドに登録するために使用する必要があります。

### レガシークエリ

Cosmos SDKにProtobufとgRPCが導入される前は、通常、[メッセージ]ではなく、モジュール開発者によって定義された特定の[クエリ]オブジェクトはありませんでした。代わりに、Cosmos SDKは、単純な[パス]を使用して各[クエリ]を定義する、より単純なアプローチを採用しています。 `path`には、` query`タイプとそれを処理するために必要なすべてのパラメータが含まれています。ほとんどのモジュールクエリでは、 `path`は次のようになります。

```
queryCategory/queryRoute/queryType/arg1/arg2/...
```

どこ:

-`queryCategory`は `query`のカテゴリであり、通常はモジュールクエリの` custom`に使用されます。これは、 `BaseApp`の[` Query`メソッド](../core/baseapp.md#query)でさまざまなタイプのクエリを区別するために使用されます。
-`baseApp`の[`queryRouter`](../core/baseapp.md#query-routing)は、` queryRoute`を使用して `query`をそのモジュールにマップします。通常、 `queryRoute`はモジュールの名前である必要があります。
-モジュールの[`querier`](./query-services.md#legacy-queriers)は、` queryType`を使用して `query`をモジュール内の適切な` querier関数 `にマップします。
-`args`は、 `query`を処理するために必要な実際のパラメータです。それらはエンドユーザーによって記入されます。大規模なクエリの場合は、リクエストの[パス]ではなく[リクエスト]の[データ]フィールドにパラメータを渡すことをお勧めします。

各 `query`の` path`は、モジュールの[コマンドラインインターフェイスファイル](./module-interfaces.md#query-commands)でモジュール開発者が定義する必要があります。一般に、モジュール開発者は、モジュールによって定義された状態サブセットをクエリ可能にするために、3つの主要なコンポーネントの実装が必要です。

-[`querier`](./query-services.md#legacy-queriers)、[route to module](../core/baseapp.md#query-routing))によって処理されると。
-[クエリコマンド](。/module-interfaces.md#query-commands)モジュールのCLIファイルでは、各 `query`の` path`が指定されています。
-`query`リターンタイプ。通常、ファイル `types/querier.go`で定義され、各モジュールの` queries`の結果タイプを指定します。これらのカスタム型は、[`fmt.Stringer`](https://golang.org/pkg/fmt/#Stringer)の` String() `メソッドを実装する必要があります。

### ストアクエリ

ストレージクエリは、ストレージキーを直接クエリします。 `clientCtx.QueryABCI(req abci.RequestQuery)`を使用して、完全な `abci.ResponseQuery`を返し、Merkle証明を含めます。

次の例を参照してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/080fcf1df25ccdf97f3029b6b6f83caaf5a235e4/x/ibc/core/client/query.go#L36-L46

+++ https://github.com/cosmos/cosmos-sdk/blob/080fcf1df25ccdf97f3029b6b6f83caaf5a235e4/baseapp/abci.go#L722-L749

## 次へ{hide}

[`Msg`サービス](./msg-services.md){hide}を理解する 