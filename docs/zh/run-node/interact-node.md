# 与节点交互

有多种与节点交互的方式:使用 CLI、使用 gRPC 或使用 REST 端点。 {概要}

## 先决条件阅读

- [gRPC、REST 和 Tendermint 端点](../core/grpc_rest.md) {prereq}
- [运行节点](./run-node.md) {prereq}

## 使用 CLI

现在您的链正在运行，是时候尝试将代币从您创建的第一个帐户发送到第二个帐户了。 在新的终端窗口中，首先运行以下查询命令:

```bash
simd query bank balances $MY_VALIDATOR_ADDRESS --chain-id my-test-chain
```

您应该看到您创建的帐户的当前余额，等于您授予它的原始“stake”余额减去您通过“gentx”委托的金额。 现在，创建第二个帐户: 

```bash
simd keys add recipient --keyring-backend test

# Put the generated address in a variable for later use.
RECIPIENT=$(simd keys show recipient -a --keyring-backend test)
```

上面的命令创建了一个尚未在链上注册的本地密钥对。 第一次从另一个账户接收代币时会创建一个账户。 现在，运行以下命令将令牌发送到“收件人”帐户: 

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000000stake --chain-id my-test-chain --keyring-backend test

# Check that the recipient account did receive the tokens.
simd query bank balances $RECIPIENT --chain-id my-test-chain
```

Finally, delegate some of the stake tokens sent to the `recipient` account to the validator:

```bash
simd tx staking delegate $(simd keys show my_validator --bech val -a --keyring-backend test) 500stake --from recipient --chain-id my-test-chain --keyring-backend test

# Query the total delegations to `validator`.
simd query staking delegations-to $(simd keys show my_validator --bech val -a --keyring-backend test) --chain-id my-test-chain
```

You should see two delegations, the first one made from the `gentx`, and the second one you just performed from the `recipient` account.

## 使用 gRPC

Protobuf 生态系统为不同的用例开发了工具，包括从 `*.proto` 文件到各种语言的代码生成。 这些工具允许轻松构建客户端。 通常，客户端连接(即传输)可以很容易地插入和替换。 让我们探索一种最流行的传输方式:[gRPC](../core/grpc_rest.md)。

由于代码生成库很大程度上取决于您自己的技术堆栈，我们将仅提供三种替代方案: 

- `grpcurl` 用于通用调试和测试，
- 通过 Go 编程，
- 适用于 JavaScript/TypeScript 开发人员的 CosmJS。

### grpcurl

[grpcurl](https://github.com/fullstorydev/grpcurl) 类似于 `curl` 但对于 gRPC。 它也可用作 Go 库，但我们将仅将其用作 CLI 命令以进行调试和测试。 按照上一个链接中的说明进行安装。

假设您有一个本地节点正在运行(本地网络或连接的实时网络)，您应该能够运行以下命令来列出可用的 Protobuf 服务(您可以将 `localhost:9000` 替换为另一个的 gRPC 服务器端点 节点，在 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml)) 中的 `grpc.address` 字段下配置: 

```bash
grpcurl -plaintext localhost:9090 list
```

您应该会看到一个 gRPC 服务列表，例如 `cosmos.bank.v1beta1.Query`。 这称为反射，它是一个 Protobuf 端点，返回所有可用端点的描述。 每一个都代表一个不同的 Protobuf 服务，每个服务都公开了多个可以查询的 RPC 方法。

为了获得服务的描述，您可以运行以下命令: 

```bash
grpcurl \
    localhost:9090 \
    describe cosmos.bank.v1beta1.Query                  # Service we want to inspect
```

It's also possible to execute an RPC call to query the node for information:

```bash
grpcurl \
    -plaintext
    -d '{"address":"$MY_VALIDATOR"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

所有可用 gRPC 查询端点的列表[即将推出](https://github.com/cosmos/cosmos-sdk/issues/7786)。

#### 使用 grpcurl 查询历史状态

您还可以通过将一些 [gRPC 元数据](https://github.com/grpc/grpc-go/blob/master/Documentation/grpc-metadata.md) 传递给查询来查询历史数据:`x-cosmos -block-height` 元数据应包含要查询的块。 如上使用 grpcurl，命令如下所示: 

```bash
grpcurl \
    -plaintext \
    -H "x-cosmos-block-height: 279256" \
    -d '{"address":"$MY_VALIDATOR"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

假设该块的状态尚未被节点修剪，此查询应返回非空响应。

### 通过 Go 编程

以下代码段展示了如何在 Go 程序中使用 gRPC 查询状态。 这个想法是创建一个gRPC连接，并使用Protobuf生成的客户端代码来查询gRPC服务器。 
```go
import (
    "context"
    "fmt"

	"google.golang.org/grpc"

    sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/tx"
)

func queryState() error {
    myAddress, err := sdk.AccAddressFromBech32("cosmos1...")
    if err != nil {
        return err
    }

    // Create a connection to the gRPC server.
    grpcConn := grpc.Dial(
        "127.0.0.1:9090", // your gRPC server address.
        grpc.WithInsecure(), // The Cosmos SDK doesn't support any transport security mechanism.
    )
    defer grpcConn.Close()

    // This creates a gRPC client to query the x/bank service.
    bankClient := banktypes.NewQueryClient(grpcConn)
    bankRes, err := bankClient.Balance(
        context.Background(),
        &banktypes.QueryBalanceRequest{Address: myAddress, Denom: "atom"},
    )
    if err != nil {
        return err
    }

    fmt.Println(bankRes.GetBalance()) // Prints the account balance

    return nil
}
```

您可以使用从任何其他 Protobuf 服务生成的客户端替换查询客户端(这里我们使用的是“x/bank”)。 所有可用 gRPC 查询端点的列表[即将推出](https://github.com/cosmos/cosmos-sdk/issues/7786)。

#### 使用 Go 查询历史状态

历史区块的查询是通过在 gRPC 请求中添加区块高度元数据来完成的。 

```go
import (
    "context"
    "fmt"

    "google.golang.org/grpc"
    "google.golang.org/grpc/metadata"

    grpctypes "github.com/cosmos/cosmos-sdk/types/grpc"
	"github.com/cosmos/cosmos-sdk/types/tx"
)

func queryState() error {
    // --snip--

    var header metadata.MD
    bankRes, err = bankClient.Balance(
        metadata.AppendToOutgoingContext(context.Background(), grpctypes.GRPCBlockHeightHeader, "12"), // Add metadata to request
        &banktypes.QueryBalanceRequest{Address: myAddress, Denom: denom},
        grpc.Header(&header), // Retrieve header from response
    )
    if err != nil {
        return err
    }
    blockHeight = header.Get(grpctypes.GRPCBlockHeightHeader)

    fmt.Println(blockHeight) // Prints the block height (12)

    return nil
}
```

### CosmJS

CosmJS 文档可以在 [https://cosmos.github.io/cosmjs](https://cosmos.github.io/cosmjs) 找到。 截至 2021 年 1 月，CosmJS 文档仍在进行中。

## 使用 REST 端点

如 [gRPC 指南](../core/grpc_rest.md) 中所述，Cosmos SDK 上的所有 gRPC 服务都可用于通过 gRPC 网关进行更方便的基于 REST 的查询。 URL 路径的格式基于 Protobuf 服务方法的完全限定名称，但可能包含小的自定义，以便最终 URL 看起来更地道。 例如，`cosmos.bank.v1beta1.Query/AllBalances` 方法的 REST 端点是 `GET /cosmos/bank/v1beta1/balances/{address}`。 请求参数作为查询参数传递。

作为一个具体的例子，发出余额请求的 `curl` 命令是: 

```bash
curl \
    -X GET \
    -H "Content-Type: application/json" \
    http://localhost:1317/cosmos/bank/v1beta1/balances/$MY_VALIDATOR
```

确保将“localhost:1317”替换为您节点的 REST 端点，在“api.address”字段下配置。

所有可用 REST 端点的列表可作为 Swagger 规范文件提供，可以在 `localhost:1317/swagger` 中查看。 确保在您的 [`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 文件中将 `api.swagger` 字段设置为 true。

### 使用 REST 查询历史状态

查询历史状态是使用 HTTP 标头 `x-cosmos-block-height` 完成的。 例如，curl 命令如下所示: 

```bash
curl \
    -X GET \
    -H "Content-Type: application/json" \
    -H "x-cosmos-block-height: 279256"
    http://localhost:1317/cosmos/bank/v1beta1/balances/$MY_VALIDATOR
```

假设该块的状态尚未被节点修剪，此查询应返回非空响应。

### 跨域资源共享(CORS)

[CORS 政策](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 默认未启用以帮助提高安全性。 如果您想在公共环境中使用 rest-server，我们建议您提供反向代理，这可以通过 [nginx](https://www.nginx.com/) 来完成。 出于测试和开发目的，[`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml) 中有一个“enabled-unsafe-cors”字段。

## 下一个 {hide}

使用 gRPC 和 REST 发送交易需要一些额外的步骤:生成交易、签名，最后广播它。 阅读[生成和签署交易](./txs.md)。 {hide} 