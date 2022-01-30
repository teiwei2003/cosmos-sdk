# 罗塞塔

`rosetta` 包实现了 Coinbase 的 [Rosetta API](https://www.rosetta-api.org)。 本文档提供有关如何使用 Rosetta API 集成的说明。 有关动机和设计选择的信息，请参阅 [ADR 035](../architecture/adr-035-rosetta-api-support.md)。

## 添加 Rosetta 命令

Rosetta API 服务器是一个独立的服务器，连接到使用 Cosmos SDK 开发的链的节点。

要启用 Rosetta API 支持，需要将 `RosettaCommand` 添加到应用程序的根命令文件(例如 `appd/cmd/root.go`)。

导入 `server` 包: 

```go
    "github.com/cosmos/cosmos-sdk/server"
```

Find the following line:

```go
initRootCmd(rootCmd, encodingConfig)
```

After that line, add the following:

```go
rootCmd.AddCommand(
  server.RosettaCommand(encodingConfig.InterfaceRegistry, encodingConfig.Marshaler)
)
```

`RosettaCommand` 函数构建 `rosetta` 根命令，并在 Cosmos SDK 的 `server` 包中定义。

由于我们已更新 Cosmos SDK 以与 Rosetta API 配合使用，因此您只需更新应用程序的根命令文件即可。

一个实现示例可以在 `simapp` 包中找到。

## 使用 Rosetta 命令

要在应用程序 CLI 中运行 Rosetta，请使用以下命令: 

```
appd rosetta --help
```

To test and run Rosetta API endpoints for applications that are running and exposed, use the following command:

```
appd rosetta
     --blockchain "your application name (ex: gaia)"
     --network "your chain identifier (ex: testnet-1)"
     --tendermint "tendermint endpoint (ex: localhost:26657)"
     --grpc "gRPC endpoint (ex: localhost:9090)"
     --addr "rosetta binding address (ex: :8080)"
```

## 扩展

您可以通过两种方式使用自定义设置自定义和扩展实现。

### 消息扩展

为了使 rosetta 可以理解 `sdk.Msg`，唯一需要的是将方法添加到满足 `rosetta.Msg` 接口的消息中。 有关如何执行此操作的示例可以在诸如“MsgDelegate”之类的抵押类型或诸如“MsgSend”之类的银行类型中找到。

### 客户端界面覆盖

如果需要更多自定义，可以嵌入 Client 类型并覆盖需要自定义的方法。 

Example:

```go
package custom_client
import (

"context"
"github.com/coinbase/rosetta-sdk-go/types"
"github.com/cosmos/cosmos-sdk/server/rosetta/lib"
)

//CustomClient embeds the standard cosmos client
//which means that it implements the cosmos-rosetta-gateway Client
//interface while at the same time allowing to customize certain methods
type CustomClient struct {
    *rosetta.Client
}

func (c *CustomClient) ConstructionPayload(_ context.Context, request *types.ConstructionPayloadsRequest) (resp *types.ConstructionPayloadsResponse, err error) {
   //provide custom signature bytes
    panic("implement me")
}
```

注意:当使用自定义客户端时，该命令不能使用，因为所需的构造函数**可能**不同，因此需要创建一个新的构造函数。 我们打算在未来提供一种无需编写额外代码即可初始化自定义客户端的方法。

### 错误扩展

由于 rosetta 需要向网络选项提供“返回”错误。 为了声明一个新的 rosetta 错误，我们在 cosmos-rosetta-gateway 中使用了 `errors` 包。 

Example:

```go
package custom_errors
import crgerrs "github.com/cosmos/cosmos-sdk/server/rosetta/lib/errors"

var customErrRetriable = true
var CustomError = crgerrs.RegisterError(100, "custom message", customErrRetriable, "description")
```

注意:必须在调用 cosmos-rosetta-gateway 的 `Server`.`Start` 方法之前注册错误。 否则注册将被忽略。 具有相同代码的错误也将被忽略。