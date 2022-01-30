# gRPC、REST 和 Tendermint 端点

本文档概述了节点公开的所有端点:gRPC、REST 以及其他一些端点。 {概要}

## 所有端点的概述

每个节点公开以下端点供用户与节点交互，每个端点在不同的端口上提供服务。端点自己的部分提供了有关如何配置每个端点的详细信息。

- gRPC 服务器(默认端口:`9090`)，
- REST 服务器(默认端口:`1317`)，
- Tendermint RPC 端点(默认端口:`26657`)。

::: 小费
该节点还暴露了一些其他端点，例如 Tendermint P2P 端点，或 [Prometheus 端点](https://docs.tendermint.com/master/nodes/metrics.html#metrics)，这些端点与Cosmos SDK。有关这些端点的更多信息，请参阅 [Tendermint 文档](https://docs.tendermint.com/master/tendermint-core/using-tendermint.html#configuration)。
:::

## gRPC 服务器

::: 警告
`go-grpc v1.34.0` 中引入的补丁使 gRPC 与 `gogoproto` 库不兼容，导致一些 [gRPC 查询](https://github.com/cosmos/cosmos-sdk/issues/8426) 恐慌。因此，Cosmos SDK 要求在你的 `go.mod` 中安装 `go-grpc <=v1.33.2`。

为确保 gRPC 正常工作，**强烈建议**在您的应用程序的 `go.mod` 中添加以下行: 

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

请参阅 [issue #8392](https://github.com/cosmos/cosmos-sdk/issues/8392) 了解更多信息。
:::

Cosmos SDK v0.40 引入了 Protobuf 作为主要的 [encoding](./encoding) 库，这带来了广泛的基于 Protobuf 的工具，可以插入 Cosmos SDK。其中一个工具是 [gRPC](https://grpc.io)，这是一种现代开源高性能 RPC 框架，具有多种语言的良好客户端支持。

每个模块公开一个定义状态查询的 [Protobuf `Query` 服务](../building-modules/messages-and-queries.md#queries)。 `Query` 服务和用于广播交易的交易服务通过应用程序中的以下函数连接到 gRPC 服务器:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.43.0-rc0/server/types/app.go#L39-L41

注意:不可能通过 gRPC 公开任何 [Protobuf `Msg` 服务](../building-modules/messages-and-queries.md#messages) 端点。交易必须使用 CLI 或以编程方式生成和签名，然后才能使用 gRPC 进行广播。有关详细信息，请参阅 [生成、签名和广播事务](../run-node/txs.html)。

`grpc.Server` 是一个具体的 gRPC 服务器，它产生并服务所有 gRPC 查询请求和广播事务请求。这个服务器可以在`~/.simapp/config/app.toml` 中配置:

- `grpc.enable = true|false` 字段定义是否应启用 gRPC 服务器。默认为“真”。
- `grpc.address = {string}` 字段定义了服务器应该绑定到的地址(实际上是端口，因为主机应该保持在 `0.0.0.0`)。默认为`0.0.0.0:9090`。

:::小费
`~/.simapp` 是存储节点配置和数据库的目录。默认情况下，它设置为`~/.{app_name}`。
:::

一旦 gRPC 服务器启动，您就可以使用 gRPC 客户端向它发送请求。我们的 [与节点交互](../run-node/interact-node.md#using-grpc) 教程中给出了一些示例。

Cosmos SDK 附带的所有可用 gRPC 端点的概述是 [Protobuf 文档](./proto-docs.md)。

## REST 服务器

Cosmos SDK 通过 gRPC 网关支持 REST 路由。

所有路由都在`~/.simapp/config/app.toml`中的以下字段下配置:

- `api.enable = true|false` 字段定义是否应该启用 REST 服务器。默认为“假”。
- `api.address = {string}` 字段定义了服务器应该绑定到的地址(实际上是端口，因为主机应该保持在 `0.0.0.0`)。默认为`tcp://0.0.0.0:1317`。
- 在`~/.simapp/config/app.toml`中定义了一些额外的API配置选项，以及注释，请直接参考该文件。

### gRPC 网关 REST 路由

如果由于各种原因您无法使用 gRPC(例如，您正在构建一个 Web 应用程序，并且浏览器不支持构建 gRPC 的 HTTP2)，那么 Cosmos SDK 将通过 gRPC 网关提供 REST 路由。

[gRPC-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) 是一种将 gRPC 端点公开为 REST 端点的工具。对于 Protobuf `Query` 服务中定义的每个 gRPC 端点，Cosmos SDK 提供了一个 REST 等价物。例如，查询余额可以通过 `/cosmos.bank.v1beta1.QueryAllBalances` gRPC 端点完成，或者通过 gRPC 网关 `"/cosmos/bank/v1beta1/balances/{address}"` REST 端点完成:两者都将返回相同的结果。对于 Protobuf `Query` 服务中定义的每个 RPC 方法，相应的 REST 端点被定义为一个选项:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.41.0/proto/cosmos/bank/v1beta1/query.proto#L19-L22

对于应用程序开发人员，gRPC 网关 REST 路由需要连接到 REST 服务器，这是通过调用 ModuleManager 上的 `RegisterGRPCGatewayRoutes` 函数来完成的。 

### Swagger

[Swagger](https://swagger.io/)(或 OpenAPIv2)规范文件在 API 服务器上的 `/swagger` 路由下公开。 Swagger 是一个开放规范，描述了服务器所服务的 API 端点，包括描述、输入参数、返回类型以及关于每个端点的更多信息。

启用 `/swagger` 端点可以通过 `api.swagger` 字段在 `~/.simapp/config/app.toml` 中进行配置，默认情况下该字段设置为 true。

对于应用程序开发人员，您可能希望基于自定义模块生成自己的 Swagger 定义。 Cosmos SDK 的 [Swagger 生成脚本](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/scripts/protoc-swagger-gen.sh) 是一个很好的起点。

## Tendermint RPC

独立于 Cosmos SDK，Tendermint 还公开了一个 RPC 服务器。这个RPC服务器可以通过`~/.simapp/config/config.toml`中`rpc`表下的参数调优来配置，默认监听地址是`tcp://0.0.0.0:26657`。 [此处](https://docs.tendermint.com/master/rpc/) 提供了所有 Tendermint RPC 端点的 OpenAPI 规范。

一些 Tendermint RPC 端点与 Cosmos SDK 直接相关:

- `/abci_query`:此端点将查询应用程序的状态。作为 `path` 参数，您可以发送以下字符串:
    - 任何 Protobuf 完全限定的服务方法，例如`/cosmos.bank.v1beta1.QueryAllBalances`。 `data` 字段应该包含使用 Protobuf 编码为字节的方法的请求参数。
    - `/app/simulate`:这将模拟一个交易，并返回一些信息，例如使用的gas。
    - `/app/version`:这将返回应用程序的版本。
    - `/store/{path}`:这将直接查询存储。
    - `/p2p/filter/addr/{port}`:这将通过地址端口返回节点的 P2P 对等点的过滤列表。
    - `/p2p/filter/id/{id}`:这将根据 ID 返回节点的 P2P 对等点的过滤列表。
- `/broadcast_tx_{aync,async,commit}`:这 3 个端点将向其他对等点广播交易。 CLI、gRPC 和 REST 公开了[一种广播交易的方法](./transactions.md#broadcasting-the-transaction)，但它们都在幕后使用了这 3 个 Tendermint RPC。

## 比较表 

| Name           | Advantages                                                                                                                                                            | Disadvantages                                                                                                 |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| gRPC           | - can use code-generated stubs in various languages<br>- supports streaming and bidirectional communication (HTTP2)<br>- small wire binary sizes, faster transmission | - based on HTTP2, not available in browsers<br>- learning curve (mostly due to Protobuf)                      |
| REST           | - ubiquitous<br>- client libraries in all languages, faster implementation<br>                                                                                        | - only supports unary request-response communication (HTTP1.1)<br>- bigger over-the-wire message sizes (JSON) |
| Tendermint RPC | - easy to use                                                                                                                                                         | - bigger over-the-wire message sizes (JSON)                                                                   |

## 下一个 {hide}

了解 [CLI](./cli.md) {hide} 
