# ライフサイクルを照会する

このドキュメントでは、Cosmos SDKアプリケーションでのクエリのライフサイクルについて、ユーザーインターフェイスからアプリケーションストレージまで、およびその逆について説明します。 {まとめ}

## 読むための前提条件

-[トランザクションライフサイクル](./tx-lifecycle.md){前提条件}

## クエリの作成

[** query **](../building-modules/messages-and-queries.md#queries)は、アプリケーションのエンドユーザーがインターフェースを介して送信し、ノード全体で処理される情報リクエストです。ユーザーは、アプリケーションのストレージまたはモジュールから、ネットワーク、アプリケーション自体、およびアプリケーションのステータスに関する情報を直接照会できます。クエリは[transactions](../core/transactions.md)(ライフサイクル[ここ](./tx-lifecycle.md)を参照)とは異なることに注意してください。特に、コンセンサスを処理する必要はありません(コンセンサスがある場合)。状態遷移をトリガーしないでください);完全なノードで完全に処理できます。

クエリのライフサイクルを説明するために、「MyQuery」が「simapp」という名前のアプリケーションで特定の委任アドレスによって作成された委任のリストを要求しているとします。予想どおり、[`staking`](../../x/staking/spec/README.md)モジュールがこのクエリを処理します。ただし、最初に、ユーザーはいくつかの方法で「MyQuery」を作成できます。

### コマンドラインインターフェイス

アプリケーションのメインインターフェイスは、コマンドラインインターフェイスです。ユーザーはフルノードに接続し、マシンから直接CLIを実行します-CLIはフルノードと直接対話します。端末から「MyQuery」を作成するには、ユーザーは次のコマンドを入力します。 

```bash
simd query staking delegations <delegatorAddress>
```

このクエリコマンドは、[`staking`](../../x/staking/spec/README.md)モジュールの開発者によって定義され、CLIの作成時にアプリケーション開発者によってサブコマンドリストに追加されます。

一般的な形式は次のとおりです。 

```bash
simd query[moduleName][command] <arguments> --flag <flagArg>
```

「--node」(CLIが接続されている完全なノード)などの値を提供するには、ユーザーは[`app.toml`](../run-node/run-node.md#configuring- -node -using-apptoml)構成ファイルを使用して、それらを設定したり、フラグとして提供したりします。

CLIは、アプリケーション開発者によって階層構造で定義された特定のコマンドのセットを理解します。[rootコマンド](../core/cli.md#root-command)( `simd`)から、コマンドタイプ(` Myquery `)、コマンド(`ステーキング `)とコマンド自体(` delegations`)を含むモジュール。 したがって、CLIはどのモジュールがコマンドを処理するかを正確に認識し、そこに直接呼び出しを渡します。

### gRPC

::: 暖かい
`go-grpc v1.34.0`で導入されたパッチにより、gRPCが` gogoproto`ライブラリと互換性がなくなり、[gRPCクエリ](https://github.com/cosmos/cosmos-sdk/issues/8426)パニックが発生しました。 したがって、Cosmos SDKでは、go.modにgo-grpc <= v1.33.2をインストールする必要があります。

gRPCが正しく機能するようにするには、アプリケーションの `go.mod`に次の行を追加することを**強くお勧めします**。 

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

詳細については、[issue#8392](https://github.com/cosmos/cosmos-sdk/issues/8392)を参照してください。
:::

Cosmos SDK v0.40で導入されたユーザーがクエリできるもう1つのインターフェイスは、[gRPC](https://grpc.io)から[gRPCサーバー](../core/grpc_rest.md#grpc-server)です。 エンドポイントは、Protobuf独自の言語に依存しないインターフェイス定義言語(IDL)で記述された、 `.proto`ファイルの[Protocol Buffers](https://developers.google.com/protocol-buffers)サービスメソッドとして定義されます。 Protobufエコシステムは、「*。proto」ファイルからさまざまな言語へのコード生成ツールを開発しました。 これらのツールを使用すると、gRPCクライアントを簡単に構築できます。

ツールの1つは[grpcurl](https://github.com/fullstorydev/grpcurl)であり、このクライアントを使用した `MyQuery`へのgRPCリクエストは次のとおりです。 

```bash
grpcurl \
    -plaintext                                           # We want results in plain test
    -import-path ./proto \                               # Import these .proto files
    -proto ./proto/cosmos/staking/v1beta1/query.proto \  # Look into this .proto file for the Query protobuf service
    -d '{"address":"$MY_DELEGATOR"}' \                   # Query arguments
    localhost:9090 \                                     # gRPC server endpoint
    cosmos.staking.v1beta1.Query/Delegations             # Fully-qualified service method name
```

::: REST

ユーザーがクエリを実行できるもう1つのインターフェースは、HTTP経由で[RESTサーバー](../core/grpc_rest.md#rest-server)をリクエストすることです。 RESTサーバーは、[gRPC-gateway](https://github.com/grpc-ecosystem/grpc-gateway)を使用して、Protobufサービスから完全に自動的に生成されます。 

`MyQuery`のHTTPリクエストの例は次のとおりです。 

```bash
wget http://localhost:1317/cosmos/staking/v1beta1/delegators/{delegatorAddr}/delegations
```

## CLIがクエリを処理する方法

上記の例は、外部ユーザーがノードのステータスを照会することによってノードと対話する方法を示しています。クエリの正確なライフサイクルについて詳しく知るために、CLIがクエリを準備する方法とノードがクエリを処理する方法を詳しく見ていきましょう。ユーザーの観点からは、相互作用は少し異なりますが、モジュール開発者によって定義された同じコマンドの実装であるため、基本的な機能はほとんど同じです。この処理ステップは、CLI、gRPC、またはRESTサーバーで行われ、多くの「client.Context」が含まれます。

::: 環境

CLIコマンドを実行するときに最初に作成されるのは `client.Context`です。 `client.Context`は、ユーザー側でリクエストを処理するために必要なすべてのデータを格納するオブジェクトです。特に、 `client.Context`は以下を格納します。

-**コーデック**:アプリケーションがTendermint RPCリクエストを発行する前にパラメーターとクエリをマーシャリングし、返された応答をJSONオブジェクトにアンマーシャリングするために使用する[encoder/decode](../core/encoding.md)デフォルトのコーデックCLIで使用されるのはProtobufです。
-**アカウントデコーダー**:[`auth`](../..//x/auth/spec/README.md)モジュールのアカウントデコーダー。`[] byte`をアカウントに変換します。
-** RPCクライアント**:リクエストの中継先となるTendermintRPCクライアントまたはノード。
-** Keyring **:[Key Manager](../basics/accounts.md#keyring)は、トランザクションに署名し、キーを使用して他の操作を処理するために使用されます。
-**出力ライター**:[ライター](https://golang.org/pkg/io/#Writer)を使用して応答を出力します。
-**構成**:ユーザーがこのコマンド用に構成したフラグ( `--height`を含む)は、クエリするブロックチェーンの高さを指定し、` --indent`は、JSON応答にインデントを追加することを意味します。

`client.Context`には、RPCクライアントを取得してABCI呼び出しを行い、クエリをフルノードに中継する` Query() `などのさまざまな関数も含まれています。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/context.go#L20-L50

`client.Context`の主な機能は、エンドユーザーとのやり取り中に使用されるデータを保存し、これらのデータとやり取りするためのメソッドを提供することです。これは、クエリがフルノードによって処理される前後に使用されます。具体的には、「MyQuery」を処理するときに、「client.Context」を使用してクエリパラメータをエンコードし、完全なノードを取得し、出力を書き込みます。フルノードはアプリケーションとは関係がなく、特定のタイプを理解しないため、フルノードに中継される前に、クエリを「[] byte」の形式でエンコードする必要があります。フルノード(RPCクライアント)自体は、ユーザーCLIが接続されているノードを認識している `client.Context`を使用して取得されます。クエリは、処理のためにこのフルノードに中継されます。最後に、 `client.Context`には、応答が返されたときに出力を書き込むために使用される` Writer`が含まれています。これらの手順については、後のセクションで詳しく説明します。

### パラメータとルートの作成

ライフサイクルのこの時点で、ユーザーはクエリに含めたいすべてのデータを含むCLIコマンドを作成します。 `client.Context`は、残りの` MyQuery`を支援するために存在します。次のステップは、コマンドまたはリクエストを解析し、パラメーターを抽出して、すべてをエンコードすることです。これらの手順はすべて、対話するインターフェースのユーザー側で行われます。

#### コーディング

この例(クエリアドレスの委任)では、 `MyQuery`には[address](./accounts.md#addresses)` delegatorAddress`が唯一のパラメータとして含まれています。ただし、リクエストには「[] byte」しか含めることができません。これは、アプリケーションタイプ(Tendermint Coreなど)に関する固有の知識を持たないフルノードのコンセンサスエンジンに中継されるためです。したがって、 `client.Context`の` codec`は、アドレスをマーシャリングするために使用されます。

CLIコマンドのコードは次のとおりです。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/staking/client/cli/query.go#L324-L327

#### gRPCクエリクライアントの作成

Cosmos SDKは、Protobufサービスから生成されたコードをクエリに使用します。 `stake`モジュールの` MyQuery`サービスは `queryClient`を生成し、CLIはそれをクエリに使用します。これは関連するコードです:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/staking/client/cli/query.go#L318-L342

下部にある `client.Context`には、事前設定されたノードを取得してクエリを転送するための` Query() `関数があります。この関数は、完全修飾されたサービスメソッド名をパスとしてクエリします(この例では、`/cosmos.staking.v1beta1.Query/Delegations`)、パラメータをパラメータとして使用します。最初に、このクエリを中継するためにユーザーによって構成されたRPCクライアント([** node **](../core/node.md)と呼ばれる)を取得し、「ABCIQueryOptions」(ABCI呼び出し用にフォーマットされた)パラメーターを作成します)) 。次に、このノードを使用してABCI呼び出し `ABCIQueryWithOptions()`を作成します。

コードは次のようになります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/query.go#L65-L91

## RPC

「ABCIQueryWithOptions()」を呼び出すと、[full node](../core/encoding.md)は「MyQuery」を受信し、リクエストを処理します。 RPCはフルノードコンセンサスエンジン(Tendermint Coreなど)用ですが、クエリはコンセンサスの一部ではなく、ネットワークの他の部分にブロードキャストされないことに注意してください。これは、ネットワークの同意を必要としないためです。ニーズ。

ABCIクライアントとTendermintRPCの詳細については、Tendermintのドキュメント[こちら](https://tendermint.com/rpc)を参照してください。

## アプリケーションクエリ処理

クエリは、基盤となるコンセンサスエンジンから中継された後、ノード全体で受信されると、特定のタイプのアプリケーションを理解し、状態のコピーを持つ環境で処理されます。[`baseapp`](../core/baseapp.md)は、ABCI[` Query() `](../core/baseapp.md#query)関数を実装し、gRPCクエリを処理します。クエリルートが解析され、既存のサービスメソッドの完全修飾サービスメソッド名と一致し(ほとんどの場合、モジュールの1つにあります)、 `baseapp`がリクエストを関連するモジュールに中継します。

gRPCルーティングに加えて、 `baseapp`は、` app`、 `store`、` p2p`、および `custom`の4つの異なるタイプのクエリも処理します。最初の3つのタイプ( `app`、` store`、 `p2p`)は純粋にアプリケーションレベルであるため、` baseapp`またはストアによって直接処理されますが、 `custom`クエリタイプではクエリをルーティングするために` baseapp`が必要ですモジュール[legacyquerier](../building-modules/query-services.md#legacy-queriers)に移動します。これらのクエリの詳細については、[このガイド](../core/grpc_rest.md#tendermint-rpc)を参照してください。

`MyQuery`には` staking`モジュールからのProtobuf完全修飾サービスメソッド名があるため( `/cosmos.staking.v1beta1.Query/Delegations`を思い出してください)、` baseapp`は最初にパスを解析してから独自の内部 `GRPCQueryRouter`を使用します対応するgRPCハンドラーを取得し、クエリをモジュールにルーティングします。 gRPCハンドラーは、このクエリを識別し、アプリケーションのストレージから適切な値を取得し、応答を返す役割を果たします。クエリサービスの詳細については、[こちら](../building-modules/query-services.md)をご覧ください。

クエリアから結果を受け取ると、 `baseapp`はユーザーに応答を返すプロセスを開始します。

:: 返事

`Query()`はABCI関数であるため、 `baseapp`は応答を[` abci.ResponseQuery`](https://tendermint.com/docs/spec/abci/abci.html#messages)タイプとして返​​します。 `client.Context`` Query()`ルーチンは応答とを受け取ります。

### CLI応答

アプリケーション[`codec`](../core/encoding.md)は、応答をJSONにアンマーシャリングするために使用され、` client.Context`は出力をコマンドラインに出力し、出力タイプ(テキスト、JSONまたはYAML)。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/context.go#L248-L283

これはパッケージです！クエリ結果はCLIからコンソールに出力されます。

## 次へ{hide}

[accounts](./accounts.md)の詳細をご覧ください。 {隠れる} 