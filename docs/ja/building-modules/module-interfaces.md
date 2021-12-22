# モジュールインターフェース

このドキュメントでは、モジュールのCLIおよびRESTインターフェースを構築する方法について詳しく説明します。さまざまなCosmosSDKモジュールの例が含まれています。 {まとめ}

## 読むための前提条件

-[モジュールの構築の概要](。/intro.md){前提条件}

## コマンドラインインターフェイス

アプリケーションの主要なインターフェイスの1つは、[コマンドラインインターフェイス](../core/cli.md)です。このエントリポイントは、アプリケーションモジュールからコマンドを追加して、エンドユーザーが[** messages **](./messages-and-queries.md#messages)および[** queries **](トランザクションにラップされた./messages)を作成できるようにします。 -and-queries.md#queries)。 CLIファイルは通常、モジュールの `。/client/cli`フォルダーにあります。

### 取引注文

 状態変化をトリガーするメッセージを作成するには、エンドユーザーは[transactions](../core/transactions.md)を作成して、メッセージをラップして配信する必要があります。 transactionコマンドは、1つ以上のメッセージを含むトランザクションを作成します。

 トランザクションコマンドには通常、モジュールの `。/client/cli`フォルダーにある独自の` tx.go`ファイルがあります。コマンドはgetter関数で指定され、関数名にはコマンド名が含まれている必要があります。

以下は、 `x/bank`モジュールの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/client/cli/tx.go#L28-L63

この例では、 `NewSendTxCmd()`は、 `MsgSend`のトランザクションをパッケージ化して渡すためのトランザクションコマンドを作成して返します。 `MsgSend`は、あるアカウントから別のアカウントにトークンを送信するために使用されるメッセージです。

一般に、getter関数は次のことを行います。

-**構築コマンド:**コマンドの作成方法の詳細については、[Cobraドキュメント](https://godoc.org/github.com/spf13/cobra)をお読みください。
    -**使用:**コマンドを呼び出すために必要なユーザー入力の形式を指定します。上記の例では、 `send`はトランザクションコマンドの名前であり、`[from_key_or_address] `、`[to_address] `、および`[amount] `はパラメータです。
    -**引数:**ユーザーが指定したパラメーターの数。この場合、「[from_key_or_address]」、「[to_address]」、「[amount]」の3つだけがあります。
    -** Short and Long:**コマンドの説明。 「短い」説明が必要です。 「長い」説明は、ユーザーが「--help」フラグを追加したときに表示される追加情報を提供するために使用できます。
    -** RunE:**エラーを返す可能性のある関数を定義します。これは、コマンドの実行時に呼び出される関数です。この関数は、新しいトランザクションを作成するためのすべてのロジックをカプセル化します。
        -この関数は通常、 `client.GetClientTxContext(cmd)`で実行できる `clientCtx`の取得から始まります。 `clientCtx`には、ユーザーに関する情報など、トランザクション処理に関連する情報が含まれています。この例では、 `clientCtx`を使用して、` clientCtx.GetFromAddress() `を呼び出して送信者のアドレスを取得します。
        -該当する場合、コマンドのパラメーターが解析されます。この例では、パラメータ `[to_address]`と `[amount]`の両方が解決されます。
        -[message](./messages-and-queries.md)は、解析されたパラメーターと `clientCtx`からの情報を使用して作成されます。メッセージタイプのコンストラクタを直接呼び出します。この場合、 `types.NewMsgSend(fromAddr、toAddr、amount)`です。メッセージの作成後に `msg.ValidateBasic()`を呼び出すことをお勧めします。これにより、提供されたパラメーターに対して健全性チェックが実行されます。
        -ユーザーのニーズに応じて、トランザクションはオフラインで生成されるか、 `tx.GenerateOrBroadcastTxCLI(clientCtx、flags、msg)`を使用して署名され、事前構成されたノードにブロードキャストされます。
-**トランザクションフラグの追加:**すべてのトランザクションコマンドは、一連のトランザクション[フラグ](#flags)を追加する必要があります。トランザクションフラグは、ユーザーから追加情報(たとえば、ユーザーが支払う意思のある料金の金額)を収集するために使用されます。 `AddTxFlagsToCmd(cmd)`を使用して、作成されたコマンドにトランザクションフラグを追加します。
-** Return命令:**最後にトランザクション命令に戻ります。
各モジュールは、モジュールのすべてのトランザクションコマンドを集約する `NewTxCmd()`を実装する必要があります。以下は、 `x/bank`モジュールの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/client/cli/tx.go#L13-L26

各モジュールは、 `AppModuleBasic`の` GetTxCmd() `メソッドも実装する必要があります。このメソッドは、` NewTxCmd() `のみを返します。これにより、rootコマンドは各モジュールのすべてのトランザクションコマンドを簡単に集約できます。以下に例を示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/bank/module.go#L75-L78 

### クエリコマンド

[クエリ](./messages-and-queries.md#queries)を使用すると、ユーザーはアプリケーションまたはネットワークステータスに関する情報を収集できます。これらの情報は、アプリケーションによってルーティングされ、それらを定義するモジュールによって処理されます。 queryコマンドは通常、モジュールの `。/client/cli`フォルダーに独自の` query.go`ファイルを持っています。トランザクションコマンドと同様に、それらはgetter関数で指定されます。以下は、 `x/auth`モジュールからのクエリコマンドの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/client/cli/query.go#L75-L105

この例では、 `GetAccountCmd()`は、指定されたアカウントアドレスに基づいてアカウントステータスを返すクエリコマンドを作成して返します。

一般に、getter関数は次のことを行います。

-**構築コマンド:**コマンドの作成方法の詳細については、[Cobraドキュメント](https://godoc.org/github.com/spf13/cobra)をお読みください。
    -**使用:**コマンドを呼び出すために必要なユーザー入力の形式を指定します。上記の例では、 `account`はクエリコマンドの名前であり、`[address] `はパラメータです。
    -**引数:**ユーザーが指定したパラメーターの数。この場合、 `[address]`は1つだけです。
    -** Short and Long:**コマンドの説明。 「短い」説明が必要です。 「長い」説明は、ユーザーが「--help」フラグを追加したときに表示される追加情報を提供するために使用できます。
    -** RunE:**エラーを返す可能性のある関数を定義します。これは、コマンドの実行時に呼び出される関数です。この関数は、新しいクエリを作成するためのすべてのロジックをカプセル化します。
        -この関数は通常、 `client.GetClientQueryContext(cmd)`で実行できる `clientCtx`の取得から始まります。 `clientCtx`には、クエリ処理に関連する情報が含まれています。
        -該当する場合、コマンドのパラメーターが解析されます。この例では、パラメータ `[address]`が解析されます。
        -`NewQueryClient(clientCtx) `を使用して、新しい` queryClient`を初期化します。次に、 `queryClient`を使用して適切な[query](./messages-and-queries.md#grpc-queries)を呼び出します。
        -`clientCtx.PrintProto`メソッドを使用して `proto.Message`オブジェクトをフォーマットし、結果をユーザーに出力できるようにします。
-**クエリフラグの追加:**すべてのクエリコマンドは、一連のクエリ[フラグ](#flags)を追加する必要があります。 `AddQueryFlagsToCmd(cmd)`を使用して、作成されたコマンドにクエリフラグを追加します。
-** Returnコマンド:**最後に、クエリコマンドが返されます。

すべてのモジュールは、モジュールのすべてのクエリコマンドを集約する `GetQueryCmd()`を実装する必要があります。 `x/auth`モジュールの例を次に示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/client/cli/query.go#L25-L42

各モジュールは、 `GetQueryCmd()`関数を返す `AppModuleBasic`の` GetQueryCmd() `メソッドも実装する必要があります。これにより、rootコマンドは各モジュールのすべてのクエリコマンドを簡単に集約できます。以下に例を示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/module.go#L80-L83

### ロゴ

[フラグ](../core/cli.md#flags)を使用すると、ユーザーはコマンドをカスタマイズできます。 `--fees`と` --gas-prices`は、ユーザーがトランザクションの[fees](../basics/gas-fees.md)とガス価格を設定できるフラグの例です。

モジュール固有のフラグは通常、モジュールの `。/client/cli`フォルダーの` flags.go`ファイルに作成されます。ロゴを作成するとき、開発者は値のタイプ、ロゴ名、デフォルト値、およびロゴの説明を設定します。開発者は、ユーザーがフラグ値を含めない場合にエラーをスローするために、フラグを_required_としてマークすることを選択することもできます。

これは、コマンドに `--from`フラグを追加する例です。

```go
cmd.Flags().String(FlagFrom, "", "Name or address of private key with which to sign")
```

この例では、flagの値は `String`、フラグの名前は` from`)、 `FlagFrom`定数の値)、flagのデフォルト値は` "" `です。ユーザーが「-」を追加することもあります。コマンド中に表示される説明。

これは、 `--from`フラグを_required_としてマークする例です。

```go
cmd.MarkFlagRequired(FlagFrom)
```

ロゴの作成の詳細については、[Cobra Documentation](https://github.com/spf13/cobra)にアクセスしてください。

[トランザクションコマンド](#transaction-commands)で説明されているように、すべてのトランザクションコマンドはフラグのセットを追加する必要があります。これは、CosmosSDKの `。/client/flags`パッケージで定義されている` AddTxFlagsToCmd`メソッドを介して行われます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/client/flags/flags.go#L94-L120

`AddTxFlagsToCmd(cmd * cobra.Command)`には、トランザクションコマンドに必要なすべての基本フラグが含まれているため、モジュール開発者は独自のフラグを追加しないことを選択できます。通常は、代わりにパラメーターを指定する方が適切です)。

同様に、モジュールクエリコマンドに共通のフラグを追加するための `AddQueryFlagsToCmd(cmd * cobra.Command)`があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/client/flags/flags.go#L85-L92

## gRPC

[gRPC](https://grpc.io/)は、リモートプロシージャコール(RPC)フレームワークです。 RPCは、ウォレットや取引所などの外部クライアントがブロックチェーンと対話するための推奨される方法です。

ABCIクエリパスを提供することに加えて、Cosmos SDKは、gRPCクエリリクエストをABCIクエリリクエストにルーティングするためのgRPCプロキシサーバーも提供します。

このため、モジュールは「AppModuleBasic」に「RegisterGRPCGatewayRoutes(clientCtx client.Context、mux * runtime.ServeMux)」を実装して、クライアントgRPCリクエストをモジュール内の正しいハンドラーに接続する必要があります。

`x/auth`モジュールの例を次に示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/x/auth/module.go#L68-L73

## gRPCゲートウェイREST

アプリケーションは、[Keplr](https://keplr.xyz)などのWebウォレットなどのHTTPリクエストを使用するWebサービスをサポートする必要があります。[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)は、REST呼び出しをgRPC呼び出しに変換します。これは、gRPCを使用しないクライアントに役立つ場合があります。

RESTクエリを公開するモジュールは、 `google.api.http`アノテーションを` rpc`メソッドに追加する必要があります。たとえば、 `x/auth`モジュールの次の例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-beta1/proto/cosmos/auth/v1beta1/query.proto#L13-L29

gRPCゲートウェイは、アプリケーションおよびTendermintと一緒にインプロセスで開始されます。[`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml)でgRPC構成` enable`を設定することで有効または無効にできます。

Cosmos SDKは、[Swagger](https://swagger.io/)ドキュメント( `protoc-gen-swagger`)を生成するためのコマンドを提供します。[`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml)で` swagger`を設定して、Swaggerドキュメントを自動的に登録するかどうかを定義します。

## 次へ{hide}

推奨される[モジュール構造](./structure.md){hide}を読む 