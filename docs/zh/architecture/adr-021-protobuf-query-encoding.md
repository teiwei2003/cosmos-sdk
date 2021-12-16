# ADR 021:协议缓冲区查询编码

## 变更日志

- 2020 年 3 月 27 日:初稿

## 地位

公认

## 语境

该 ADR 是在
[ADR 019](./adr-019-protobuf-state-encoding.md) 和
[ARD 020](./adr-019-protobuf-transaction-encoding.md)，即我们的目标是设计
Cosmos SDK 客户端的协议缓冲区迁移路径。

此 ADR 从 [ARD 020](./adr-020-protobuf-transaction-encoding.md) 继续
指定查询的编码。

## 决定

### 自定义查询定义

模块通过协议缓冲区“服务”定义定义自定义查询。
这些“服务”定义通常与
GRPC 协议。但是，协议缓冲区规范表明
它们可以被任何使用的请求/响应协议更一般地使用
协议缓冲区编码。因此，我们可以使用`service`定义来指定
自定义 ABCI 查询，甚至重用大量 GRPC 基础设施。

每个带有自定义查询的模块都应该定义一个规范命名为 `Query` 的服务: 

```proto
// x/bank/types/types.proto

service Query {
  rpc QueryBalance(QueryBalanceParams) returns (cosmos_sdk.v1.Coin) { }
  rpc QueryAllBalances(QueryAllBalancesParams) returns (QueryAllBalancesResponse) { }
}
```

#### 接口类型的处理

使用接口类型并需要真正多态的模块通常会强制
`oneof` 直到提供一组具体实现的应用程序级别
该应用程序支持的界面。 虽然欢迎应用程序为
查询并实现应用级查询服务，建议模块
提供通过`google.protobuf.Any`公开这些接口的查询方法。
在事务级别上存在一个问题，即`Any` 的开销太大
高以证明其使用是合理的。 但是对于查询，这不是问题，并且
提供使用“Any”的通用模块级查询并不排除应用程序
还提供返回使用应用程序级`oneof`s的应用程序级查询。

`gov` 模块的假设示例如下所示: 

```proto
// x/gov/types/types.proto

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

我们模块的自定义查询是通过实现这个接口来实现的。

这个生成的接口中的第一个参数是一个通用的`context.Context`，
而查询器方法通常需要一个 `sdk.Context` 实例来读取
从存储。 由于可以将任意值附加到 `context.Context`
使用 `WithValue` 和 `Value` 方法，Cosmos SDK 应该提供一个函数
`sdk.UnwrapSDKContext` 从提供的对象中检索 `sdk.Context`
`上下文。上下文`。

上面银行模块的`QueryBalance`的示例实现将
看起来像:

```go
type Querier struct {
	Keeper
}

func (q Querier) QueryBalance(ctx context.Context, params *types.QueryBalanceParams) (*sdk.Coin, error) {
	balance := q.GetBalance(sdk.UnwrapSDKContext(ctx), params.Address, params.Denom)
	return &balance, nil
}
```

### 自定义查询注册和路由

上面的查询服务器实现将使用`AppModule`s 注册
一种可以简单实现的新方法`RegisterQueryService(grpc.Server)`
如下: 

```go
// x/bank/module.go
func (am AppModule) RegisterQueryService(server grpc.Server) {
	types.RegisterQueryServer(server, keeper.Querier{am.keeper})
}
```

在幕后，一个新方法`RegisterService(sd *grpc.ServiceDesc, handler interface{})`
将添加到现有的`baseapp.QueryRouter` 以将查询添加到自定义
查询路由表(路由方法如下所述)。
此方法的签名与现有的匹配
GRPC `Server` 类型上的 `RegisterServer` 方法，其中 `handler` 是自定义的
上面描述的查询服务器实现。

类似 GRPC 的请求由服务名称路由(例如`cosmos_sdk.x.bank.v1.Query`)
和方法名称(例如`QueryBalance`)与`/`s 组合形成一个完整的
方法名称(例如`/cosmos_sdk.x.bank.v1.Query/QueryBalance`)。这被翻译
进入 ABCI 查询，如“custom/cosmos_sdk.x.bank.v1.Query/QueryBalance”。服务处理程序
注册到 `QueryRouter.RegisterService` 将被路由这种方式。

除了方法名称之外，GRPC 请求还携带一个 protobuf 编码的有效载荷，它自然映射
到 `RequestQuery.Data`，并接收 protobuf 编码的响应或错误。因此
类似 GRPC 的 rpc 方法有一个非常自然的映射到现有的
`sdk.Query` 和 `QueryRouter` 基础设施。

这个基本规范允许我们重用协议缓冲区“服务”定义
对于 ABCI 自定义查询，大大减少了手动解码和
查询方法中的编码。

### GRPC 协议支持

除了提供 ABCI 查询路径外，我们还可以轻松提供 GRPC
将 GRPC 协议中的请求路由到 ABCI 查询请求的代理服务器
在引擎盖下。通过这种方式，客户可以使用他们的宿主语言现有的
使用 GRPC 实现对 Cosmos SDK 应用程序进行直接查询
这些“服务”定义。为了让这个服务器工作，`QueryRouter`
在 BaseApp 上需要公开注册的服务处理程序
`QueryRouter.RegisterService` 到代理服务器实现。节点可以
在与 ABCI 应用程序相同的进程中在单独的端口上启动代理服务器
带有命令行标志。

### REST 查询和 Swagger 生成

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) 是一个项目
使用服务上的特殊注释将 REST 调用转换为 GRPC 调用
方法。想要公开 REST 查询的模块应该添加 `google.api.http`
对他们的 `rpc` 方法进行注释，如下面的示例所示。 

```proto
// x/bank/types/types.proto

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

grpc-gateway 将直接针对上述 GRPC 代理工作，这将
在后台将请求转换为 ABCI 查询。 grpc-gateway 也可以
自动生成 Swagger 定义。

在目前的REST查询实现中，每个模块都需要实现
除了 ABCI 查询器方法之外，还手动进行 REST 查询。使用 grpc 网关
方法，将不需要生成单独的 REST 查询处理程序，只需
上面描述的查询服务器作为 grpc-gateway 处理 protobuf 的翻译
到 REST 以及 Swagger 定义。

Cosmos SDK 应为应用程序提供 CLI 命令以启动 GRPC 网关
一个单独的过程或与 ABCI 应用程序相同的过程，以及提供一个
用于生成 grpc-gateway 代理 `.proto` 文件和 `swagger.json` 的命令
文件。

### 客户端使用

gogo protobuf grpc 插件除了服务器之外还生成客户端接口
接口。对于上面定义的 `Query` 服务，我们会得到一个 `QueryClient`
界面如: 

```go
type QueryClient interface {
	QueryBalance(ctx context.Context, in *QueryBalanceParams, opts ...grpc.CallOption) (*types.Coin, error)
	QueryAllBalances(ctx context.Context, in *QueryAllBalancesParams, opts ...grpc.CallOption) (*QueryAllBalancesResponse, error)
}
```

通过一个小补丁到 gogo protobuf ([gogo/protobuf#675](https://github.com/gogo/protobuf/pull/675))
我们已经调整了 grpc 代码生成器以使用接口而不是具体类型
对于生成的客户端结构。 这使我们还可以重用 GRPC 基础设施
用于 ABCI 客户端查询。

1Context`将接收一个新方法`QueryConn`，该方法返回一个`ClientConn`
将呼叫路由到 ABCI 查询

客户端(例如 CLI 方法)将能够调用这样的查询方法: 

```go
clientCtx := client.NewContext()
queryClient := types.NewQueryClient(clientCtx.QueryConn())
params := &types.QueryBalanceParams{addr, denom}
result, err := queryClient.QueryBalance(gocontext.Background(), params)
```

### Testing

测试将能够直接从 keeper 和 `sdk.Context` 创建查询客户端
使用“QueryServerTestHelper”的引用如下: 

```go
queryHelper := baseapp.NewQueryServerTestHelper(ctx)
types.RegisterQueryServer(queryHelper, keeper.Querier{app.BankKeeper})
queryClient := types.NewQueryClient(queryHelper)
```

## Future Improvements

## Consequences

### Positive

* 大大简化了查询器的实现(无需手动编码/解码)
* 轻松查询客户端生成(可以使用现有的 grpc 和 swagger 工具)
* 不需要 REST 查询实现
* 类型安全的查询方法(通过 grpc 插件生成)
* 往前走，查询方法的破坏会更少，因为
buf 提供的向后兼容性保证 

### Negative

* 所有使用现有 ABCI/REST 查询的客户端都需要重构
对于新的 GRPC/REST 查询路径以及 protobuf/proto-json 编码
数据，但这在 protobuf 重构中或多或少是不可避免的 

### Neutral

## References
