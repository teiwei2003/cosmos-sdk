# ADR 021:プロトコルバッファクエリコード

## 変更ログ

-2020年3月27日:最初のドラフト

## 状態

受け入れられました

## 環境

ADRは
[ADR 019](./adr-019-protobuf-state-encoding.md)および
[ARD 020](./adr-019-protobuf-transaction-encoding.md)、つまり、私たちの目標は設計することです
CosmosSDKクライアントのプロトコルバッファ移行パス。

このADRは[ARD020](./adr-020-protobuf-transaction-encoding.md)から継続します
クエリのエンコーディングを指定します。

## 決定

###カスタムクエリ定義

モジュールは、プロトコルバッファの「サービス」定義を介してカスタムクエリを定義します。
これらの「サービス」の定義は通常、
GRPCプロトコル。 ただし、プロトコルバッファの仕様には次のように記載されています。
これらは、使用される任意の要求/応答プロトコルでより一般的に使用できます。
プロトコルバッファエンコーディング。 したがって、 `service`定義を使用して指定できます
ABCIクエリをカスタマイズし、多くのGRPCインフラストラクチャを再利用することもできます。

カスタムクエリを使用するすべてのモジュールは、Queryという名前のサービスを正規に定義する必要があります。

```proto
/.x/bank/types/types.proto

service Query {
  rpc QueryBalance(QueryBalanceParams) returns (cosmos_sdk.v1.Coin) { }
  rpc QueryAllBalances(QueryAllBalancesParams) returns (QueryAllBalancesResponse) { }
}
```

####インターフェースタイプの処理

インターフェイスタイプを使用し、真にポリモーフィックである必要があるモジュールは、通常、強制されます
特定の実装アプリケーションレベルのセットを提供するまでの `oneof`
アプリケーションでサポートされているインターフェース。 ウェルカムアプリケーションは
アプリケーションレベルのクエリサービスのクエリと実装、モジュールの提案
`google.protobuf.Any`を介してこれらのインターフェースを公開するクエリメソッドを提供します。
トランザクションレベルで問題があります。つまり、 `Any`のオーバーヘッドが大きすぎます。
その使用を正当化するために高い。 しかし、クエリの場合、これは問題ではありません。
「Any」を使用して一般的なモジュールレベルのクエリを提供しても、アプリケーションは除外されません
また、アプリケーションレベルの `oneof`の使用を返すアプリケーションレベルのクエリも提供します。

`gov`モジュールの架空の例は次のとおりです。 

```proto
/.x/gov/types/types.proto

import "google/protobuf/any.proto";

service Query {
  rpc GetProposal(GetProposalParams) returns (AnyProposal) { }
}

message AnyProposal {
  ProposalBase base = 1;
  google.protobuf.Any content = 2;
}
```

### 自定义查询实现

为了实现查询服务，我们可以复用现有的[gogo protobuf](https://github.com/gogo/protobuf)
grpc 插件，它为名为 `Query` 的服务生成一个名为的接口
`QueryServer` 如下: 

```go
type QueryServer interface {
	QueryBalance(context.Context, *QueryBalanceParams) (*types.Coin, error)
	QueryAllBalances(context.Context, *QueryAllBalancesParams) (*QueryAllBalancesResponse, error)
}
```

モジュールのカスタムクエリは、このインターフェイスを実装することで実現されます。

この生成されたインターフェースの最初のパラメーターは、一般的な `context.Context`です。
クエリアメソッドは通常、読み取るために `sdk.Context`のインスタンスを必要とします
ストレージから。 `context.Context`には任意の値を追加できるため
`WithValue`および` Value`メソッドを使用して、CosmosSDKは関数を提供する必要があります
`sdk.UnwrapSDKContext`は、提供されたオブジェクトから` sdk.Context`を取得します
`コンテキスト。 コンテキスト `。

上記の銀行モジュールの `QueryBalance`の実装例は次のようになります。
のように見える: 

```go
type Querier struct {
	Keeper
}

func (q Querier) QueryBalance(ctx context.Context, params *types.QueryBalanceParams) (*sdk.Coin, error) {
	balance := q.GetBalance(sdk.UnwrapSDKContext(ctx), params.Address, params.Denom)
	return &balance, nil
}
```

### カスタムクエリの登録とルーティング

上記のクエリサーバーの実装は `AppModule`sに登録されます
簡単に実装できる新しいメソッド `RegisterQueryService(grpc.Server)`
次のように:  

```go
/.x/bank/module.go
func (am AppModule) RegisterQueryService(server grpc.Server) {
	types.RegisterQueryServer(server, keeper.Querier{am.keeper})
}
```

舞台裏では、新しいメソッド `RegisterService(sd * grpc.ServiceDesc、handler interface {})`
クエリをカスタムに追加するために、既存の `baseapp.QueryRouter`に追加されます
ルーティングテーブルを照会します(ルーティング方法については以下で説明します)。
このメソッドのシグネチャは、既存のメソッドと一致します
GRPCの `Server`タイプの` RegisterServer`メソッド。`handler`はカスタムです。
上記のクエリサーバーの実装。

GRPCのようなリクエストは、サービス名でルーティングされます(例: `cosmos_sdk.x.bank.v1.Query`)
そして、メソッド名(例: `QueryBalance`)を`/`と組み合わせて、完全なものを形成します
メソッド名(例: `/cosmos_sdk.x.bank.v1.Query/QueryBalance`)。これは翻訳されています
「custom/cosmos_sdk.x.bank.v1.Query/QueryBalance」などのABCIクエリを入力します。サービスハンドラ
`QueryRouter.RegisterService`に登録すると、この方法でルーティングされます。

メソッド名に加えて、GRPCリクエストはprotobufでエンコードされたペイロードも運びます。これは自然にマッピングされます
`RequestQuery.Data`に移動し、protobufでエンコードされた応答またはエラーを受け取ります。したがって
GRPCに似たrpcメソッドは、既存のものに非常に自然にマッピングされます
`sdk.Query`および` QueryRouter`インフラストラクチャ。

この基本仕様により、プロトコルバッファの「サービス」定義を再利用できます。
ABCIカスタムクエリ、手動デコード、および
queryメソッドでのエンコード。

### GRPCプロトコルのサポート

ABCIクエリパスを提供するだけでなく、GRPCも簡単に提供できます
ABCIクエリリクエストの場合、GRPCプロトコルのリクエストをプロキシサーバーにルーティングします
フードの下。このようにして、顧客は既存のホスト言語を使用できます
GRPCを使用して、CosmosSDKアプリケーションの直接クエリを実装します
これらの「サービス」が定義されています。このサーバーを機能させるには、 `QueryRouter`
BaseAppに公的に登録する必要があるサービスハンドラー
`QueryRouter.RegisterService`をプロキシサーバーの実装に追加します。ノードはできます
ABCIアプリケーションと同じプロセスで別のポートでプロキシサーバーを起動します
コマンドラインフラグ付き。

### RESTクエリとSwaggerの生成

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)はプロジェクトです
サービスで特別なアノテーションを使用して、REST呼び出しをGRPC呼び出しに変換します
方法。 RESTクエリを公開するモジュールは、 `google.api.http`を追加する必要があります
以下の例に示すように、それらの `rpc`メソッドについてコメントします。  

```proto
/.x/bank/types/types.proto

service Query {
  rpc QueryBalance(QueryBalanceParams) returns (cosmos_sdk.v1.Coin) {
    option (google.api.http) = {
      get: "/x/bank/v1/balance/{address}/{denom}"
    };
  }
  rpc QueryAllBalances(QueryAllBalancesParams) returns (QueryAllBalancesResponse) {
    option (google.api.http) = {
      get: "/x/bank/v1/balances/{address}"
    };
  }
}
```

grpc-gatewayは、上記のGRPCプロキシに対して直接機能します。
バックグラウンドでリクエストをABCIクエリに変換します。 grpc-gatewayも利用可能です
Swagger定義は自動的に生成されます。

現在のRESTクエリの実装では、各モジュールを実装する必要があります
ABCI Queryerメソッドに加えて、RESTクエリも手動で実行されます。 grpcゲートウェイを使用する
メソッド、個別のRESTクエリハンドラーを生成する必要はありません。
上記のクエリサーバーは、protobuf変換を処理するためのgrpc-gatewayとして使用されます
RESTとSwaggerの定義に。

Cosmos SDKは、アプリケーションがGRPCゲートウェイを開始するためのCLIコマンドを提供する必要があります
別のプロセスまたはABCIアプリケーションと同じプロセスであり、
grpc-gatewayプロキシの `.proto`ファイルと` swagger.json`を生成するためのコマンド
資料。

### クライアントの使用

gogo protobuf grpcプラグインは、サーバーに加えてクライアントインターフェイスを生成します
インターフェース。 上で定義した `Query`サービスの場合、` QueryClient`を取得します
次のようなインターフェース: 

```go
type QueryClient interface {
	QueryBalance(ctx context.Context, in *QueryBalanceParams, opts ...grpc.CallOption) (*types.Coin, error)
	QueryAllBalances(ctx context.Context, in *QueryAllBalancesParams, opts ...grpc.CallOption) (*QueryAllBalancesResponse, error)
}
```

小さなパッチをgogoprotobufに渡します([gogo/protobuf#675](https://github.com/gogo/protobuf/pull/675))
具体的なタイプの代わりにインターフェースを使用するようにgrpcコードジェネレーターを調整しました
生成されたクライアント構造の場合。 これにより、GRPCインフラストラクチャも再利用できます
ABCIクライアントクエリに使用されます。

1Context`は、 `ClientConn`を返す新しいメソッド` QueryConn`を受け取ります
ABCIクエリへの通話のルーティング

クライアント(CLIメソッドなど)は、次のようなクエリメソッドを呼び出すことができます。 
```go
clientCtx := client.NewContext()
queryClient := types.NewQueryClient(clientCtx.QueryConn())
params := &types.QueryBalanceParams{addr, denom}
result, err := queryClient.QueryBalance(gocontext.Background(), params)
```

### Testing

テストでは、キーパーと `sdk.Context`から直接クエリクライアントを作成できます。
「QueryServerTestHelper」を使用するための参照は次のとおりです。 

```go
queryHelper := baseapp.NewQueryServerTestHelper(ctx)
types.RegisterQueryServer(queryHelper, keeper.Querier{app.BankKeeper})
queryClient := types.NewQueryClient(queryHelper)
```

## 将来の改善

## 結果

### ポジティブ

*クエリャーの実装を大幅に簡素化します(手動でエンコード/デコードする必要はありません)
*クライアント生成を簡単に照会できます(既存のgrpcおよびswaggerツールを使用できます)
* RESTクエリを実装する必要はありません
*タイプセーフなクエリメソッド(grpcプラグインによって生成)
*先に進むと、クエリメソッドの中断が少なくなります。
bufによって提供される下位互換性の保証

### ネガティブ

*既存のABCI/RESTクエリを使用するすべてのクライアントをリファクタリングする必要があります
新しいGRPC/RESTクエリパスとprotobuf/proto-jsonエンコーディングの場合
データですが、これはprotobufの再構築では多かれ少なかれ避けられません

### ニュートラル

## 参照 