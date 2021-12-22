# クエリサービス

Protobufクエリサービス処理[`queries`](./messages-and-queries.md#queries)。クエリサービスは、それらが定義されているモジュールに固有であり、モジュールで定義されている「クエリ」のみを処理します。これらは、 `BaseApp`の[` Query`メソッド](../core/baseapp.md#query)から呼び出されます。 {まとめ}

## 読むための前提条件

-[モジュールマネージャー](。/module-manager.md){前提条件}
-[メッセージとクエリ](./messages-and-queries.md){prereq}

## `Queryer`タイプ

Cosmos SDKで定義されている `querier`タイプは、[gRPC Services](#grpc-service)をサポートするために非推奨になります。これは、 `querier`関数の典型的な構造を指定します。

+++ https://github.com/cosmos/cosmos-sdk/blob/9a183ffbcc0163c8deb71c7fd5f8089a83e58f05/types/queryable.go#L9

それを分解しましょう:

-`path`はクエリタイプを含む `string`配列であり、` query`パラメータを含めることもできます。詳細については、[`queries`](./messages-and-queries.md#queries)を参照してください。
-`req`自体は主に、 `path`に入れるには大きすぎるパラメータを取得するために使用されます。これは、「req」の「Data」フィールドを使用して行われます。
-[`Context`](../core/context.md)には、` query`と最新の状態のブランチを処理するために必要なすべての情報が含まれています。これは主に[`keeper`](./keeper.md)が状態にアクセスするために使用します。
-`res`の結果は `BaseApp`に返され、アプリケーションの[` codec`](../core/encoding.md)がグループ化に使用されます。

## モジュールクエリサービスの実装

### gRPCサービス

Protobufの `Query`サービスを定義すると、すべてのサービスメソッドを備えた` QueryServer`インターフェースがモジュールごとに生成されます。 

```go
type QueryServer interface {
	QueryBalance(context.Context, *QueryBalanceParams) (*types.Coin, error)
	QueryAllBalances(context.Context, *QueryAllBalancesParams) (*QueryAllBalancesResponse, error)
}
```

これらのカスタムクエリメソッドは、モジュールのキーパーによって、通常は `。/keeper/grpc_query.go`に実装する必要があります。 これらのメソッドの最初のパラメータは一般的な `context.Context`であり、クエリメソッドは一般的に読み取るために` sdk.Context`のインスタンスを必要とします
ストレージから。 したがって、Cosmos SDKは、提供されたオブジェクトから `sdk.Context`を取得するための関数` sdk.UnwrapSDKContext`を提供します。
`コンテキスト。 コンテキスト `。

これは、銀行モジュールの実装例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/x/bank/keeper/grpc_query.go

###従来のクエリツール

モジュールによって残された「クエリア」は、通常、モジュールフォルダの「./keeper/querier.go」ファイルに実装されます。[Module Manager](./module-manager.md)は、モジュールの `querier`を[application's`queryRouter`](../core/baseapp.md#query-routing)に` NewQuerier()を介して追加するために使用されます。 `メソッド。 通常、マネージャの `NewQuerier()`メソッドは、 `keeper/querier.go`で定義されている` NewQuerier() `メソッドを次のように呼び出すだけです。  

```go
func NewQuerier(keeper Keeper) sdk.Querier {
	return func(ctx sdk.Context, path[]string, req abci.RequestQuery) ([]byte, error) {
		switch path[0] {
		case QueryType1:
			return queryType1(ctx, path[1:], req, keeper)

		case QueryType2:
			return queryType2(ctx, path[1:], req, keeper)

		default:
			return nil, sdkerrors.Wrapf(sdkerrors.ErrUnknownRequest, "unknown %s query endpoint: %s", types.ModuleName, path[0])
		}
	}
}
```

この単純なスイッチは、受信した `query`タイプに固有の` querier`関数を返します。[Query Lifecycle](../basics/query-lifecycle.md)のこの時点で、 `path`(` path[0] `)の最初の要素にはクエリのタイプが含まれています。 次の要素は空であるか、クエリの処理に必要なパラメータが含まれています。

`querier`関数自体は非常に単純です。 通常、[`keeper`](./keeper.md)を使用して、状態から1つ以上の値を取得します。 次に、[`codec`](../core/encoding.md)を使用して値をマーシャリングし、結果の`[] byte`を返します。

`querier`の詳細については、この[` querier`関数の実装例](https://github.com/cosmos/cosmos-sdk/blob/7f59723d889b69ca19966167f0b3a7fec7a39e53/x/gov/keeper/querier.go)を参照してください。バンクモジュールから。

## 次へ{hide}

[`BeginBlocker`と` EndBlocker`](./beginblock-endblock.md){hide}を理解する 