# gRPC、REST、およびTendermintエンドポイント

このドキュメントでは、ノードによって公開されるすべてのエンドポイント(gRPC、REST、およびその他のエンドポイント)の概要を説明します。 {まとめ}

## すべてのエンドポイントの概要

各ノードは、ユーザーがノードと対話できるように次のエンドポイントを公開し、各エンドポイントは異なるポートでサービスを提供します。 エンドポイント独自のセクションには、各エンドポイントの構成方法に関する詳細情報が記載されています。

-gRPCサーバー(デフォルトポート: `9090`)、
-RESTサーバー(デフォルトポート: `1317`)、
-Tendermint RPCエンドポイント(デフォルトポート: `26657`)。

::: ヒント
このノードは、Tendermint P2Pエンドポイントや[Prometheusエンドポイント](https://docs.tendermint.com/master/nodes/metrics.html#metrics)など、UniverseSDKに接続されている他のエンドポイントも公開します。 これらのエンドポイントの詳細については、[Tendermintのドキュメント](https://docs.tendermint.com/master/tendermint-core/using-tendermint.html#configuration)を参照してください。
:::

## gRPCサーバー

::: 警告
`go-grpc v1.34.0`で導入されたパッチにより、gRPCが` gogoproto`ライブラリと互換性がなくなり、[gRPCクエリ](https://github.com/cosmos/cosmos-sdk/issues/8426)パニックが発生しました。 したがって、Cosmos SDKでは、go.modにgo-grpc <= v1.33.2をインストールする必要があります。

gRPCが正しく機能するようにするには、アプリケーションの `go.mod`に次の行を追加することを**強くお勧めします**。 

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

詳細については、[issue＃8392](https://github.com/cosmos/cosmos-sdk/issues/8392)を参照してください。
:::

Cosmos SDK v0.40は、メインの[encoding](./encoding)ライブラリとしてProtobufを導入しました。これにより、CosmosSDKにプラグインできるさまざまなProtobufベースのツールが提供されます。これらのツールの1つは[gRPC](https://grpc.io)です。これは、複数の言語で優れたクライアントサポートを備えた最新のオープンソースの高性能RPCフレームワークです。

各モジュールは、ステータスクエリを定義する[Protobuf `Query`サービス](../building-modules/messages-and-queries.md＃queries)を公開します。トランザクションをブロードキャストするための `Query`サービスとトランザクションサービスは、アプリケーションの次の機能を介してgRPCサーバーに接続します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-rc0/server/types/app.go#L39-L41

注:[Protobuf `Msg` service](../building-modules/messages-and-queries.md＃messages)エンドポイントをgRPC経由で公開することはできません。トランザクションは、gRPCを使用してブロードキャストする前に、CLIを使用して、またはプログラムで生成および署名する必要があります。詳細については、[トランザクションの生成、署名、およびブロードキャスト](../run-node/txs.html)を参照してください。

`grpc.Server`は特定のgRPCサーバーであり、すべてのgRPCクエリリクエストとブロードキャストトランザクションリクエストを生成して処理します。このサーバーは `〜/.simapp/config/app.toml`で設定できます。

-`grpc.enable = true | false`フィールドは、gRPCサーバーを有効にするかどうかを定義します。デフォルトは「true」です。
-`grpc.address = {string} `フィールドは、サーバーがバインドされるアドレスを定義します(ホストは、` 0.0.0.0`に維持される必要があるため、実際にはポートです)。デフォルトは `0.0.0.0:9090`です。

:::ヒント
`〜/.simapp`は、ノード構成とデータベースが保存されるディレクトリです。デフォルトでは、 `〜/。{app_name}`に設定されています。
:::

gRPCサーバーが起動すると、gRPCクライアントを使用してサーバーにリクエストを送信できます。いくつかの例は、[ノードとの対話](../run-node/interact-node.md＃using-grpc)チュートリアルに記載されています。

Cosmos SDKに含まれている利用可能なすべてのgRPCエンドポイントの概要は、[Protobuf Documentation](./proto-docs.md)です。

## RESTサーバー

Cosmos SDKは、gRPCゲートウェイを介したRESTルーティングをサポートしています。

すべてのルートは、 `〜/.simapp/config/app.toml`の次のフィールドで設定されます。

-`api.enable = true | false`フィールドは、RESTサーバーを有効にするかどうかを定義します。デフォルトは「false」です。
-`api.address = {string} `フィールドは、サーバーがバインドするアドレスを定義します(ホストは` 0.0.0.0`に維持する必要があるため、実際にはポート)。デフォルトは `tcp://0.0.0.0:1317`です。
-いくつかの追加のAPI構成オプションとコメントは `〜/.simapp/config/app.toml`で定義されています。このファイルを直接参照してください。

### gRPCゲートウェイRESTルーティング

さまざまな理由でgRPCを使用できない場合(たとえば、Webアプリケーションを構築していて、ブラウザーがHTTP2構築gRPCをサポートしていない場合)、CosmosSDKはgRPCゲートウェイを介したRESTルーティングを提供します。

[gRPC-gateway](https://grpc-ecosystem.github.io/grpc-gateway/)は、gRPCエンドポイントをRESTエンドポイントとして公開するためのツールです。 Protobuf `Query`サービスで定義されたgRPCエンドポイントごとに、CosmosSDKは同等のRESTを提供します。たとえば、残高のクエリは、 `/cosmos.bank.v1beta1.QueryAllBalances` gRPCエンドポイント、またはgRPCゲートウェイ` "/cosmos/bank/v1beta1/balances/{address}" `RESTエンドポイントを介して実行できます。同じ結果を返します。 Protobuf `Query`サービスで定義されたRPCメソッドごとに、対応するRESTエンドポイントがオプションとして定義されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.41.0/proto/cosmos/bank/v1beta1/query.proto#L19-L22

アプリケーション開発者の場合、gRPCゲートウェイのRESTルーティングはRESTサーバーに接続する必要があります。これは、ModuleManagerの `RegisterGRPCGatewayRoutes`関数を呼び出すことによって行われます。

### Swagger

[Swagger](https://swagger.io/)(またはOpenAPIv2)仕様ファイルは、APIサーバーの `/swagger`ルートで公開されます。 Swaggerは、サーバーが提供するAPIエンドポイントを説明するオープン仕様であり、説明、入力パラメーター、リターンタイプ、および各エンドポイントに関する詳細情報が含まれます。

`/swagger`エンドポイントの有効化は、`〜/.simapp/config/app.toml`で、デフォルトでtrueに設定されている `api.swagger`フィールドを介して構成できます。

アプリケーション開発者の場合、カスタムモジュールに基づいて独自のSwagger定義を生成することをお勧めします。 Cosmos SDKの[Swagger生成スクリプト](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/scripts/protoc-swagger-gen.sh)は良い出発点です。

## テンダーミントRPC

Cosmos SDKとは別に、TendermintはRPCサーバーも公開します。このRPCサーバーは、 `〜/.simapp/config/config.toml`の` rpc`テーブルでパラメーターを調整することで構成できます。デフォルトのリスニングアドレスは `tcp://0.0.0.0:26657`です。 [ここ](https://docs.tendermint.com/master/rpc/)は、すべてのTendermintRPCエンドポイントのOpenAPI仕様を提供します。

一部のTendermintRPCエンドポイントは、CosmosSDKに直接関連しています。

-`/abci_query`:このエンドポイントは、アプリケーションのステータスを照会します。 `path`パラメータとして、次の文字列を送信できます。
    -`/cosmos.bank.v1beta1.QueryAllBalances`などのProtobufの完全修飾サービスメソッド。 `data`フィールドには、Protobufエンコーディングのメソッドをバイトとして使用するリクエストパラメータが含まれている必要があります。
    -`/app/simulate`:これはトランザクションをシミュレートし、使用されたガスなどの情報を返します。
    -`/app/version`:これはアプリケーションのバージョンを返します。
    -`/store/{path} `:これはストアに直接クエリを実行します。
    -`/p2p/filter/addr/{port} `:これは、アドレスポートを介してノードのP2Pピアのフィルターされたリストを返します。
    -`/p2p/filter/id/{id} `:これは、IDに基づいてノードのP2Pピアのフィルターされたリストを返します。
-`/broadcast_tx_ {aync、async、commit} `:これらの3つのエンドポイントは、トランザクションを他のピアにブロードキャストします。 CLI、gRPC、およびRESTは、[トランザクションをブロードキャストする方法](./transactions.md＃broadcasting-the-transaction)を公開しますが、これら3つのTendermintRPCをバックグラウンドで使用します。

## 比較表

| Name           | Advantages                                                                                                                                                            | Disadvantages                                                                                                 |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| gRPC           | - can use code-generated stubs in various languages<br>- supports streaming and bidirectional communication (HTTP2)<br>- small wire binary sizes, faster transmission | - based on HTTP2, not available in browsers<br>- learning curve (mostly due to Protobuf)                      |
| REST           | - ubiquitous<br>- client libraries in all languages, faster implementation<br>                                                                                        | - only supports unary request-response communication (HTTP1.1)<br>- bigger over-the-wire message sizes (JSON) |
| Tendermint RPC | - easy to use                                                                                                                                                         | - bigger over-the-wire message sizes (JSON)                                                                   |

## 次へ{非表示}

[CLI](./cli.md){hide}を理解する
