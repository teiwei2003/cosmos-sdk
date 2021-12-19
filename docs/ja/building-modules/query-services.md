# 查询服务

Protobuf 查询服务处理 [`queries`](./messages-and-queries.md#queries)。查询服务特定于定义它们的模块，并且只处理在所述模块中定义的“查询”。它们是从`BaseApp` 的[`Query` 方法](../core/baseapp.md#query) 调用的。 {概要}

## 先决条件阅读

- [模块管理器](./module-manager.md) {prereq}
- [消息和查询](./messages-and-queries.md) {prereq}

## `查询器` 类型

Cosmos SDK 中定义的 `querier` 类型将被弃用，以支持 [gRPC Services](#grpc-service)。它指定了一个 `querier` 函数的典型结构:

+++ https://github.com/cosmos/cosmos-sdk/blob/9a183ffbcc0163c8deb71c7fd5f8089a83e58f05/types/queryable.go#L9

让我们分解一下:

- `path` 是一个包含查询类型的 `string` 数组，也可以包含 `query` 参数。有关更多信息，请参阅 [`queries`](./messages-and-queries.md#queries)。
- `req` 本身主要用于检索太大而无法放入 `path` 的参数。这是使用“req”的“Data”字段完成的。
- [`Context`](../core/context.md) 包含处理`query` 所需的所有必要信息，以及最新状态的一个分支。它主要由 [`keeper`](./keeper.md) 用来访问状态。
- 结果 `res` 返回给 `BaseApp`，使用应用程序的 [`codec`](../core/encoding.md) 进行编组。

##模块查询服务的实现

### gRPC 服务

在定义 Protobuf `Query` 服务时，会为每个模块生成一个带有所有服务方法的 `QueryServer` 接口: 

```go
type QueryServer interface {
	QueryBalance(context.Context, *QueryBalanceParams) (*types.Coin, error)
	QueryAllBalances(context.Context, *QueryAllBalancesParams) (*QueryAllBalancesResponse, error)
}
```

这些自定义查询方法应该由模块的 keeper 实现，通常在 `./keeper/grpc_query.go` 中。这些方法的第一个参数是一个通用的`context.Context`，而查询器方法一般需要一个`sdk.Context`的实例来读取
从存储。因此，Cosmos SDK 提供了一个函数 `sdk.UnwrapSDKContext` 来从提供的对象中检索 `sdk.Context`
`上下文。上下文`。

这是银行模块的示例实现:

+++ https://github.com/cosmos/cosmos-sdk/blob/d55c1a26657a0af937fa2273b38dcfa1bb3cff9f/x/bank/keeper/grpc_query.go

### 传统查询器

模块遗留的“查询器”通常在模块文件夹内的“./keeper/querier.go”文件中实现。 [模块管理器](./module-manager.md) 用于通过`NewQuerier 将模块的`querier`s 添加到[应用程序的`queryRouter`](../core/baseapp.md#query-routing) ()` 方法。通常，管理器的`NewQuerier()` 方法简单地调用`keeper/querier.go` 中定义的`NewQuerier()` 方法，如下所示: 

```go
func NewQuerier(keeper Keeper) sdk.Querier {
	return func(ctx sdk.Context, path []string, req abci.RequestQuery) ([]byte, error) {
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

这个简单的开关返回一个特定于接收到的 `query` 类型的 `querier` 函数。 在 [查询生命周期](../basics/query-lifecycle.md) 的这一点上，`path` (`path[0]`) 的第一个元素包含查询的类型。 以下元素为空或包含处理查询所需的参数。

`querier` 函数本身非常简单。 他们通常使用 [`keeper`](./keeper.md) 从状态中获取一个或多个值。 然后，他们使用 [`codec`](../core/encoding.md) 对值进行编组，并返回作为结果获得的 `[]byte`。

要更深入地了解`querier`，请参阅此[`querier` 函数的示例实现](https://github.com/cosmos/cosmos-sdk/blob/7f59723d889b69ca19966167f0b3a7fec7a39e53/x/gov/keeper/querier.go ) 来自银行模块。

## 下一个 {hide}

了解 [`BeginBlocker` 和 `EndBlocker`](./beginblock-endblock.md) {hide} 
