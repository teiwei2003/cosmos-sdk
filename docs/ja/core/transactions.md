# トレード

`Transactions` これは、アプリケーションの状態変更をトリガーするためにエンドユーザーによって作成されたオブジェクトです。 {まとめ}

## 前提条件の読書

- [Cosmos SDK アプリケーションプロファイリング](../basics/app-anatomy.md) {prereq}

## トレード

トランザクションは、[contexts](./context.md)と[`sdk.Msg`s](../building-modules/messages-and-queries.md)に保存されているメタデータで構成され、モジュールProtobufを介して渡されます。 [`Msg`サービス](../building-modules/msg-services.md)。

ユーザーがアプリケーションを操作して状態を変更したい場合(コインの送信など)、トランザクションを作成します。 トランザクションをネットワークにブロードキャストする前に、各トランザクションの `sdk.Msg`は、適切なアカウントに関連付けられた秘密鍵で署名する必要があります。 次に、トランザクションをブロックに含め、コンセンサスプロセスを通じてネットワークによって検証および承認される必要があります。 トランザクションのライフサイクルの詳細については、[こちら](../basics/tx-lifecycle.md)をクリックしてください。

## タイプ定義

トランザクションオブジェクトは、「Tx」インターフェイスを実装するCosmosSDKタイプです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/tx_msg.go#L49-L57

次のメソッドが含まれています。

- **GetMsgs:** トランザクションを解凍し、含まれている `sdk.Msg`リストを返します。トランザクションには、モジュール開発者によって定義された1つ以上のメッセージが含まれる場合があります。
- **ValidateBasic:** ABCIメッセージ[`CheckTx`](./baseapp.md＃checktx)および[` DeliverTx`]( ./baseapp.md#delivertx)を使用して、トランザクションが無効にならないようにします。 たとえば、[`auth`](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth)モジュールの` StdTx` `ValidateBasic`関数は、トランザクションが署名されているかどうかを確認します。正しいデジタル署名者ユーザーの数、およびコストはユーザーの上限を超えません。 この関数は、メッセージに対して基本的な有効性チェックのみを実行するsdk.Msgで使用されるValidateBasic関数とは異なることに注意してください。 たとえば、[`runTx`](./baseapp.md＃runtx)が[` auth`](https://github.com/cosmos/cosmos-sdk/tree/master/x/auth/spec )モジュール。最初に各メッセージに対して `ValidateBasic`を実行し、次にトランザクション自体に対して` ValidateBasic`を呼び出す `auth`モジュールAnteHandlerを実行します。

「Tx」は実際にはトランザクションの生成に使用される中間タイプであるため、開発者は「Tx」を直接操作することはめったにありません。 代わりに、開発者は `TxBuilder`インターフェースを好むべきです。詳細については、[以下](＃transaction-generation)を参照してください。

### トランザクションに署名します

トランザクション内の各メッセージは、その `GetSigners`で指定されたアドレスで署名する必要があります。 Cosmos SDKでは現在、2つの異なる方法でトランザクションに署名できます。

#### `SIGN_MODE_DIRECT`(優先)

`Tx`インターフェースの最も一般的に使用される実装は、` SIGN_MODE_DIRECT`で使用されるProtobuf`Tx`メッセージです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L12-L25

Protobufのシリアル化は決定論的ではないため、Cosmos SDKは追加の「TxRaw」タイプを使用して、トランザクション署名の固定バイトを表します。 すべてのユーザーは、トランザクションの有効な「body」と「auth_info」を生成し、Protobufを使用してこれら2つのメッセージをシリアル化できます。 次に、「TxRaw」は、ユーザーの「body」と「auth_info」の正確なバイナリ表現を修正します。それぞれ「body_bytes」と「auth_info_bytes」と呼ばれます。 トランザクションのすべての署名者によって署名されたドキュメントは `SignDoc`です(決定論的シリアル化には[ADR-027](../architecture/adr-027-deterministic-protobuf-serialization.md)を使用)：

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/tx.proto#L47-L64

すべての署名者によって署名されると、 `body_bytes`、` auth_info_bytes`、および `signatures`が` TxRaw`に収集され、それらのシリアル化されたバイトがネットワーク上でブロードキャストされます。

#### `SIGN_MODE_LEGACY_AMINO_JSON`

`Tx`インターフェースのレガシー実装は、` x/auth`の `StdTx`構造です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdtx.go#L120-L130

すべての署名者が署名した文書は `StdSignDoc`です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdsign.go#L20-L33

AminoJSONを使用してバイトにエンコードします。 すべての署名が `StdTx`に収集されると、` StdTx`はAminoJSONを使用してシリアル化され、これらのバイトはネットワークを介してブロードキャストされます。

#### その他のシンボルモード

他のシンボルモードが議論されていますが、最も有名なのは「SIGN_MODE_TEXTUAL」です。 それらについて詳しく知りたい場合は、[ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md)を参照してください。

## トランザクションプロセス

エンドユーザーがトランザクションを送信するプロセスは次のとおりです。

- トランザクションに入れるニュースを決定し、
- Cosmos SDKの `TxBuilder`を使用して、トランザクションを生成します。
- 利用可能なインターフェースの1つを使用して、トランザクションをブロードキャストします。

次の段落では、これらの各コンポーネントについてこの順序で説明します。

### 情報

::: ヒント
モジュール `sdk.Msg`を[ABCIメッセージ](https://tendermint.com/docs/spec/abci/abci.html#messages)と混同しないでください。これは、Tendermintとアプリケーション層の間の相互作用を定義します。
:::

**メッセージ**(または `sdk.Msg`s)は、それらが属するモジュールのスコープ内で状態遷移をトリガーするモジュール固有のオブジェクトです。モジュール開発者は、Protobuf [`Msg`サービス](../building-modules/msg-services.md)にメソッドを追加して、モジュールのメッセージを定義し、対応する` MsgServer`を実装します。

各 `sdk.Msg`は、各モジュールの` tx.proto`ファイルで定義されているProtobuf [`Msg` service](../building-modules/msg-services.md)RPCに関連付けられています。 SKDアプリケーションルーターは、各 `sdk.Msg`を対応するRPCに自動的にマップします。 Protobufは、モジュールの `Msg`サービスごとに` MsgServer`インターフェースを生成し、モジュール開発者はこのインターフェースを実装する必要があります。
この設計により、モジュール開発者はより多くの責任を引き受けることができ、アプリケーション開発者は状態遷移ロジックを繰り返し実装することなく、共通の機能を再利用できます。

Protobufの `Msg`サービスと` MsgServer`の実装方法の詳細については、[こちら](../building-modules/msg-services.md)をクリックしてください。

メッセージには状態遷移ロジックに関する情報が含まれていますが、トランザクションの他のメタデータおよび関連情報は「TxBuilder」および「Context」に格納されています。

### トランザクションの生成

`TxBuilder`インターフェースには、トランザクション生成に密接に関連するデータが含まれており、エンドユーザーはそれを自由に設定して必要なトランザクションを生成できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/client/tx_config.go#L32-L45

-`Msg`s、トランザクションに含まれる[messages](＃messages)配列。
-ユーザーが選択したオプションである `GasLimit`は、ユーザーが支払う必要のあるガスの量を計算するために使用されます。
-`メモ `、トランザクションで送信されるメモまたはコメント。
-`FeeAmount`、ユーザーが支払う意思のある最大金額。
-`TimeoutHeight`、トランザクションが有効になるまでのブロックの高さ。
-`Signatures`、トランザクションのすべての署名者からの署名の配列。

現在、トランザクションに署名するための2つの署名モードがあるため、 `TxBuilder`の2つの実装もあります。

-[wrapper](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/tx/builder.go#L19-L33)は、 `SIGN_MODE_DIRECT`のトランザクションを作成するために使用されます、
-`SIGN_MODE_LEGACY_AMINO_JSON`の[StdTxBuilder](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/x/auth/legacy/legacytx/stdtx_builder.go#L14-L20)。

ただし、 `TxBuilder`の2つの実装は、エンドユーザーから遠く離れている必要があります。これは、エンドユーザーが全体的な` TxConfig`インターフェイスを使用することを好むためです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/client/tx_config.go#L21-L30

`TxConfig`は、トランザクションの管理に使用されるアプリケーション全体の構成です。最も重要なことは、「SIGN_MODE_DIRECT」と「SIGN_MODE_LEGACY_AMINO_JSON」のどちらを使用して各トランザクションに署名するかに関する情報を保存することです。 `txBuilder：= txConfig.NewTxBuilder()`を呼び出すことにより、新しい `TxBuilder`が適切なシンボルモードで作成されます。

`TxBuilder`が上記のセッターによって正しく入力されると、` TxConfig`はバイトを正しくエンコードする役割も果たします(ここでも、 `SIGN_MODE_DIRECT`または` SIGN_MODE_LEGACY_AMINO_JSON`を使用します)。これは、 `TxEncoder()`メソッドを使用してトランザクションを生成およびエンコードする方法の擬似コードスニペットです。

```go
txBuilder := txConfig.NewTxBuilder()
txBuilder.SetMsgs(...)//and other setters on txBuilder

bz, err := txConfig.TxEncoder()(txBuilder.GetTx())
//bz are bytes to be broadcasted over the network
```

### ブロードキャストトランザクション

トランザクションバイトが生成されると、現在3つのブロードキャスト方法があります。

#### コマンドラインインターフェイス

アプリケーション開発者は、[コマンドラインインターフェイス](../core/cli.md)、[gRPCおよび/またはRESTインターフェイス](../core/grpc_rest.md)を作成することにより、アプリケーションのエントリポイントを作成します。アプリケーションの。/cmd`フォルダ。 これらのインターフェースにより、ユーザーはコマンドラインを介してアプリケーションと対話できます。

[コマンドラインインターフェイス](../building-modules/module-interfaces.md＃cli)の場合、モジュール開発者はサブコマンドを作成して、アプリケーションの最上位のトランザクションコマンド `TxCmd`にサブコマンドとして追加します。 CLIコマンドは、実際には、トランザクション処理のすべてのステップを1つの単純なコマンドにバンドルします。つまり、メッセージの作成、トランザクションの生成、およびブロードキャストです。 具体的な例については、[ノードとの対話](../run-node/interact-node.md)セクションを参照してください。 CLIを使用したトランザクションの例は次のとおりです。

```bash
simd tx send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake
```

#### gRPC

[gRPC](https://grpc.io)は、Cosmos SDK0.40でCosmosSDKのRPCレイヤーのメインコンポーネントとして導入されました。 gRPCの主な目的は、モジュールの[`Query`サービス](../building-modules)のコンテキストにあります。 ただし、Cosmos SDKは、モジュールに関連しない他のいくつかのgRPCサービスも公開します。その1つが「Tx」サービスです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/tx/v1beta1/service.proto

`Tx`サービスは、シミュレートされたトランザクションやクエリトランザクション、トランザクションをブロードキャストする方法など、いくつかの便利な機能を公開します。

[ここ](../run-node/txs.md＃programmatically-with-go)は、ブロードキャストおよびシミュレートされた取引の例を示しています。

#### 休み

各gRPCメソッドには対応するRESTエンドポイントがあり、[gRPC-gateway](https://github.com/grpc-ecosystem/grpc-gateway)を使用して生成されます。 したがって、gRPCの使用に加えて、HTTPを使用して `POST/cosmos/tx/v1beta1/txs`エンドポイントで同じトランザクションをブロードキャストすることもできます。

例を見ることができます[here](../run-node/txs.md＃using-rest)

#### Tendermint RPC

上記で紹介した3つのメソッドは、実際にはTendermint RPC `/broadcast_tx_ {async、sync、commit}`エンドポイントをより高度に抽象化したものであり、[ここ](https://docs.tendermint.com/master/rpc/＃)に記載されています。/Tx)。 これは、必要に応じて、TendermintRPCエンドポイントを直接使用してトランザクションをブロードキャストできることを意味します。

## 次へ{非表示}

[context](./context.md){hide}を理解する