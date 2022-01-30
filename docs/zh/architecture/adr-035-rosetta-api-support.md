# ADR 035:Rosetta API 支持

## 作者

- 乔纳森·吉梅诺 (@jgimeno)
- 大卫格里森(@senormonito)
- Alessio Treglia (@alessio)
- Frojdy Dymylja (@fdymylja)

## 变更日志

- 2021-05-12:外部库 [cosmos-rosetta-gateway](https://github.com/tendermint/cosmos-rosetta-gateway) 已移入 Cosmos SDK。

## 语境

[Rosetta API](https://www.rosetta-api.org/) 是 Coinbase 开发的开源规范和工具集，用于
标准化区块链交互。

通过使用标准 API 来集成区块链应用程序，它将

* 让用户更容易与给定的区块链交互
* 允许交易所快速轻松地集成新的区块链
* 使应用程序开发人员能够在以下位置构建跨区块链应用程序，例如区块浏览器、钱包和 dApps
  大大降低成本和工作量。

## 决定

很明显，将 Rosetta API 支持添加到 Cosmos SDK 将为所有开发人员和
生态系统中基于 Cosmos SDK 的链。如何实施是关键。

拟议设计的驱动原则是:

1. **可扩展性:** 应用程序开发人员设置网络必须尽可能无风险和无痛
   用于公开 Rosetta API 兼容服务的配置。
2. **长期支持:** 本提案旨在为所有受支持的 Cosmos SDK 版本系列提供支持。
3. **成本效率:** 将 Rosetta API 规范的更改从 `master` 向后移植到各种稳定版
   Cosmos SDK 的分支是需要降低的成本。

我们将通过以下方式实现这些原则:

1. 会有一个包`rosetta/lib`
   用于实现核心 Rosetta API 功能，特别是:
   一个。类型和接口(`Client`、`OfflineClient`...)，这将设计与实现细节分开。
   湾`Server` 功能独立于 Cosmos SDK 版本。
   C。 `Online/OfflineNetwork`，不导出，使用`Client`接口实现rosetta API，查询节点，构建tx等。
   d.用于扩展 rosetta 错误的 `errors` 包。
2.由于Cosmos发布系列的差异，每个系列都会有自己的`Client`接口的具体实现。
3. 在应用程序中启动 API 服务有两种选择:
   一个。 API共享应用进程
   湾API 特定的过程。

## 建筑学

### 外部回购

作为部分将描述建议的外部库，包括服务实现，以及定义的类型和接口。

#### 服务器 

`Server` 是一个简单的 `struct`，它启动并侦听设置中指定的端口。 这旨在用于所有积极支持的 Cosmos SDK 版本。

构造函数如下:

`func NewServer(设置设置)(服务器，错误)`

用于构建新服务器的“设置”如下: 

```go
//Settings define the rosetta server settings
type Settings struct {
	//Network contains the information regarding the network
	Network *types.NetworkIdentifier
	//Client is the online API handler
	Client crgtypes.Client
	//Listen is the address the handler will listen at
	Listen string
	//Offline defines if the rosetta service should be exposed in offline mode
	Offline bool
	//Retries is the number of readiness checks that will be attempted when instantiating the handler
	//valid only for online API
	Retries int
	//RetryWait is the time that will be waited between retries
	RetryWait time.Duration
}
```

#### 类型

包类型混合使用 rosetta 类型和自定义定义的类型包装器，客户端必须在执行操作时解析和返回。

##### 接口

每个 SDK 版本使用不同的格式来连接(rpc、gRPC 等)、查询和构建事务，我们在什么是`Client` 接口中抽象了这一点。
客户端使用 rosetta 类型，而 `Online/OfflineNetwork` 负责返回正确解析的 rosetta 响应和错误。

每个 Cosmos SDK 版本系列都有自己的“客户端”实现。
开发者可以根据需要实现自己的自定义`Client`。 

```go
//Client defines the API the client implementation should provide.
type Client interface {
	//Needed if the client needs to perform some action before connecting.
	Bootstrap() error
	//Ready checks if the servicer constraints for queries are satisfied
	//for example the node might still not be ready, it's useful in process
	//when the rosetta instance might come up before the node itself
	//the servicer must return nil if the node is ready
	Ready() error

	//Data API

	//Balances fetches the balance of the given address
	//if height is not nil, then the balance will be displayed
	//at the provided height, otherwise last block balance will be returned
	Balances(ctx context.Context, addr string, height *int64) ([]*types.Amount, error)
	//BlockByHashAlt gets a block and its transaction at the provided height
	BlockByHash(ctx context.Context, hash string) (BlockResponse, error)
	//BlockByHeightAlt gets a block given its height, if height is nil then last block is returned
	BlockByHeight(ctx context.Context, height *int64) (BlockResponse, error)
	//BlockTransactionsByHash gets the block, parent block and transactions
	//given the block hash.
	BlockTransactionsByHash(ctx context.Context, hash string) (BlockTransactionsResponse, error)
	//BlockTransactionsByHash gets the block, parent block and transactions
	//given the block hash.
	BlockTransactionsByHeight(ctx context.Context, height *int64) (BlockTransactionsResponse, error)
	//GetTx gets a transaction given its hash
	GetTx(ctx context.Context, hash string) (*types.Transaction, error)
	//GetUnconfirmedTx gets an unconfirmed Tx given its hash
	//NOTE(fdymylja): NOT IMPLEMENTED YET!
	GetUnconfirmedTx(ctx context.Context, hash string) (*types.Transaction, error)
	//Mempool returns the list of the current non confirmed transactions
	Mempool(ctx context.Context) ([]*types.TransactionIdentifier, error)
	//Peers gets the peers currently connected to the node
	Peers(ctx context.Context) ([]*types.Peer, error)
	//Status returns the node status, such as sync data, version etc
	Status(ctx context.Context) (*types.SyncStatus, error)

	//Construction API

	//PostTx posts txBytes to the node and returns the transaction identifier plus metadata related
	//to the transaction itself.
	PostTx(txBytes []byte) (res *types.TransactionIdentifier, meta map[string]interface{}, err error)
	//ConstructionMetadataFromOptions
	ConstructionMetadataFromOptions(ctx context.Context, options map[string]interface{}) (meta map[string]interface{}, err error)
	OfflineClient
}

//OfflineClient defines the functionalities supported without having access to the node
type OfflineClient interface {
	NetworkInformationProvider
	//SignedTx returns the signed transaction given the tx bytes (msgs) plus the signatures
	SignedTx(ctx context.Context, txBytes []byte, sigs []*types.Signature) (signedTxBytes []byte, err error)
	//TxOperationsAndSignersAccountIdentifiers returns the operations related to a transaction and the account
	//identifiers if the transaction is signed
	TxOperationsAndSignersAccountIdentifiers(signed bool, hexBytes []byte) (ops []*types.Operation, signers []*types.AccountIdentifier, err error)
	//ConstructionPayload returns the construction payload given the request
	ConstructionPayload(ctx context.Context, req *types.ConstructionPayloadsRequest) (resp *types.ConstructionPayloadsResponse, err error)
	//PreprocessOperationsToOptions returns the options given the preprocess operations
	PreprocessOperationsToOptions(ctx context.Context, req *types.ConstructionPreprocessRequest) (options map[string]interface{}, err error)
	//AccountIdentifierFromPublicKey returns the account identifier given the public key
	AccountIdentifierFromPublicKey(pubKey *types.PublicKey) (*types.AccountIdentifier, error)
}
```

### 2. Cosmos SDK 实现

基于版本的 Cosmos SDK 实现负责满足 `Client` 接口。
在 Stargate、Launchpad 和 0.37 中，我们引入了 rosetta.Msg 的概念，此消息不在共享存储库中，因为 sdk.Msg 类型因 Cosmos SDK 版本而异。

rosetta.Msg 界面如下: 

```go
//Msg represents a cosmos-sdk message that can be converted from and to a rosetta operation.
type Msg interface {
	sdk.Msg
	ToOperations(withStatus, hasError bool) []*types.Operation
	FromOperations(ops []*types.Operation) (sdk.Msg, error)
}
```

因此，想要扩展 rosetta 支持的操作集的开发人员只需要使用 `ToOperations` 和 `FromOperations` 方法扩展他们模块的 sdk.Msgs。

### 3. API 服务调用

如开头所述，应用程序开发人员将有两种调用 Rosetta API 服务的方法:

1. 应用程序和API共享进程
2. 独立API服务

#### 共享进程(仅限星际之门)

Rosetta API 服务可以在与应用程序相同的执行过程中运行。这将通过 app.toml 设置启用，如果未启用 gRPC，rosetta 实例将在离线模式下旋转(仅限 tx 构建功能)。

#### 单独的API服务

客户端应用程序开发人员也可以使用包含在`/server/rosetta` 包中的 rosetta 命令编写一个新命令来启动 Rosetta API 服务器作为一个单独的进程。命令的构建取决于 Cosmos SDK 版本。示例可以在“simd”中找到，用于星际之门，在“contrib/rosetta/simapp”中可以找到其他版本系列。 

## Status

Proposed

## Consequences

### Positive

- Cosmos SDK 中的开箱即用 Rosetta API 支持。
- 区块链接口标准化 

## References

- https://www.rosetta-api.org/
