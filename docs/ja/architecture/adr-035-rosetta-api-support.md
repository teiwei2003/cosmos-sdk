# ADR 035:RosettaAPIのサポート

## 著者

-ジョナサン・ギメノ(@jgimeno)
-デビッドグリーソン(@senormonito)
-Alessio Treglia(@alessio)
-Frojdy Dymylja(@fdymylja)

## 変更ログ

-2021-05-12:外部ライブラリ[cosmos-rosetta-gateway](https://github.com/tendermint/cosmos-rosetta-gateway)がCosmosSDKに移動されました。

## 環境

[Rosetta API](https://www.rosetta-api.org/)は、Coinbaseによって開発されたオープンソース仕様およびツールセットです。
ブロックチェーンの相互作用を標準化します。

標準のAPIを使用してブロックチェーンアプリケーションを統合することにより、

*ユーザーが特定のブロックチェーンを簡単に操作できるようにする
*取引所が新しいブロックチェーンを迅速かつ簡単に統合できるようにする
*アプリケーション開発者が、ブロックエクスプローラー、ウォレット、dAppなどの次の場所でクロスブロックチェーンアプリケーションを構築できるようにします
  コストと作業負荷を大幅に削減します。

## 決定

明らかに、CosmosSDKにRosettaAPIサポートを追加すると、すべての開発者と
エコシステム内のCosmosSDKに基づくチェーン。実装方法が鍵となります。

提案された設計の推進原則は次のとおりです。

1. **スケーラビリティ:**ネットワークをセットアップするアプリケーション開発者は、可能な限りリスクがなく、痛みがない必要があります
   RosettaAPI互換サービスの構成を公開するために使用されます。
2. **長期サポート:**この提案は、サポートされているすべてのCosmosSDKバージョンシリーズのサポートを提供することを目的としています。
3. **コスト効率:** RosettaAPI仕様の後方移植が `master`からさまざまな安定したバージョンに変更されました
   Cosmos SDKのブランチは、コストを削減する必要があります。

これらの原則は、次の方法で実装します。

1.パッケージ `rosetta .lib`があります
   特に次のようなコアRosettaAPI関数を実装するために使用されます。
   一。設計と実装の詳細を分離するタイプとインターフェース( `Client`、` OfflineClient` ...)。
   `Server`関数は、CosmosSDKのバージョンに依存しません。
   C。 `Online .OnlineNetwork`、エクスポートしないでください。`Client`インターフェースを使用して、ロゼッタAPIの実装、ノードのクエリ、txのビルドなどを行ってください。
   d。ロゼッタエラーを拡張するために使用される `errors`パッケージ。
2. Cosmosリリースシリーズの違いにより、各シリーズには、[クライアント]インターフェイスの独自の実装があります。
3.アプリケーションでAPIサービスを開始するには、次の2つのオプションがあります。
   一。 API共有申請プロセス
   API固有のプロセス。

## 建築

### 外部購入

一部として、提案された外部ライブラリについて、サービスの実装、定義されたタイプおよびインターフェイスを含めて説明します。

#### サーバー

`Server`は、設定で指定されたポートで起動およびリッスンする単純な` struct`です。これは、アクティブにサポートされているすべてのバージョンのCosmosSDKで使用することを目的としています。

コンストラクターは次のとおりです。

`func NewServer(設定)(サーバー、エラー)`

新しいサーバーを構築するために使用される[設定]は次のとおりです。 

```go
/.Settings define the rosetta server settings
type Settings struct {
	/.Network contains the information regarding the network
	Network *types.NetworkIdentifier
	/.Client is the online API handler
	Client crgtypes.Client
	/.Listen is the address the handler will listen at
	Listen string
	/.Offline defines if the rosetta service should be exposed in offline mode
	Offline bool
	/.Retries is the number of readiness checks that will be attempted when instantiating the handler
	/.valid only for online API
	Retries int
	/.RetryWait is the time that will be waited between retries
	RetryWait time.Duration
}
```

#### タイプ

パッケージタイプはロゼッタタイプとカスタム定義タイプラッパーを混合し、クライアントは操作を実行するときにそれを解析して返す必要があります。

##### インターフェース

SDKの各バージョンは、接続(rpc、gRPCなど)、クエリ、トランザクションの構築に異なる形式を使用します。これを[クライアント]インターフェイスとは何かで抽象化します。
クライアントはロゼッタタイプを使用し、 `Online .OfflineNetwork`は正しく解析されたロゼッタ応答とエラーを返す責任があります。

Cosmos SDKの各バージョンシリーズには、独自の[クライアント]実装があります。
開発者は、必要に応じて独自のカスタム `Client`を実装できます。 

```go
/.Client defines the API the client implementation should provide.
type Client interface {
	/.Needed if the client needs to perform some action before connecting.
	Bootstrap() error
	/.Ready checks if the servicer constraints for queries are satisfied
	/.for example the node might still not be ready, it's useful in process
	/.when the rosetta instance might come up before the node itself
	/.the servicer must return nil if the node is ready
	Ready() error

	/.Data API

	/.Balances fetches the balance of the given address
	/.if height is not nil, then the balance will be displayed
	/.at the provided height, otherwise last block balance will be returned
	Balances(ctx context.Context, addr string, height *int64) ([]*types.Amount, error)
	/.BlockByHashAlt gets a block and its transaction at the provided height
	BlockByHash(ctx context.Context, hash string) (BlockResponse, error)
	/.BlockByHeightAlt gets a block given its height, if height is nil then last block is returned
	BlockByHeight(ctx context.Context, height *int64) (BlockResponse, error)
	/.BlockTransactionsByHash gets the block, parent block and transactions
	/.given the block hash.
	BlockTransactionsByHash(ctx context.Context, hash string) (BlockTransactionsResponse, error)
	/.BlockTransactionsByHash gets the block, parent block and transactions
	/.given the block hash.
	BlockTransactionsByHeight(ctx context.Context, height *int64) (BlockTransactionsResponse, error)
	/.GetTx gets a transaction given its hash
	GetTx(ctx context.Context, hash string) (*types.Transaction, error)
	/.GetUnconfirmedTx gets an unconfirmed Tx given its hash
	/.NOTE(fdymylja): NOT IMPLEMENTED YET!
	GetUnconfirmedTx(ctx context.Context, hash string) (*types.Transaction, error)
	/.Mempool returns the list of the current non confirmed transactions
	Mempool(ctx context.Context) ([]*types.TransactionIdentifier, error)
	/.Peers gets the peers currently connected to the node
	Peers(ctx context.Context) ([]*types.Peer, error)
	/.Status returns the node status, such as sync data, version etc
	Status(ctx context.Context) (*types.SyncStatus, error)

	/.Construction API

	/.PostTx posts txBytes to the node and returns the transaction identifier plus metadata related
	/.to the transaction itself.
	PostTx(txBytes []byte) (res *types.TransactionIdentifier, meta map[string]interface{}, err error)
	/.ConstructionMetadataFromOptions
	ConstructionMetadataFromOptions(ctx context.Context, options map[string]interface{}) (meta map[string]interface{}, err error)
	OfflineClient
}

/.OfflineClient defines the functionalities supported without having access to the node
type OfflineClient interface {
	NetworkInformationProvider
	/.SignedTx returns the signed transaction given the tx bytes (msgs) plus the signatures
	SignedTx(ctx context.Context, txBytes []byte, sigs []*types.Signature) (signedTxBytes []byte, err error)
	/.TxOperationsAndSignersAccountIdentifiers returns the operations related to a transaction and the account
	/.identifiers if the transaction is signed
	TxOperationsAndSignersAccountIdentifiers(signed bool, hexBytes []byte) (ops []*types.Operation, signers []*types.AccountIdentifier, err error)
	/.ConstructionPayload returns the construction payload given the request
	ConstructionPayload(ctx context.Context, req *types.ConstructionPayloadsRequest) (resp *types.ConstructionPayloadsResponse, err error)
	/.PreprocessOperationsToOptions returns the options given the preprocess operations
	PreprocessOperationsToOptions(ctx context.Context, req *types.ConstructionPreprocessRequest) (options map[string]interface{}, err error)
	/.AccountIdentifierFromPublicKey returns the account identifier given the public key
	AccountIdentifierFromPublicKey(pubKey *types.PublicKey) (*types.AccountIdentifier, error)
}
```

### 2. CosmosSDKの実装

バージョンベースのCosmosSDK実装は、 `Client`インターフェースを満たす役割を果たします。
Stargate、Launchpad、0.37では、rosetta.Msgの概念が導入されました。このメッセージは、Cosmos SDKのバージョンによってsdk.Msgタイプが異なるため、共有リポジトリにはありません。

rosetta.Msgインターフェースは次のとおりです。

```go
/.Msg represents a cosmos-sdk message that can be converted from and to a rosetta operation.
type Msg interface {
	sdk.Msg
	ToOperations(withStatus, hasError bool) []*types.Operation
	FromOperations(ops []*types.Operation) (sdk.Msg, error)
}
```

したがって、ロゼッタでサポートされている一連の操作を拡張したい開発者は、 `ToOperations`メソッドと` FromOperations`メソッドを使用してモジュールのsdk.Msgsを拡張するだけで済みます。

### 3.APIサービスの呼び出し

冒頭で述べたように、アプリケーション開発者は、RosettaAPIサービスを呼び出す2つの方法があります。

1.アプリケーションとAPIの共有プロセス
2.独立したAPIサービス

#### 共有プロセス(スターゲートのみ)

Rosetta APIサービスは、アプリケーションと同じ実行プロセスで実行できます。これはapp.toml設定で有効になります。gRPCが有効になっていない場合、ロゼッタインスタンスはオフラインモードでローテーションされます(txビルド機能のみ)。

#### 単一のAPIサービス

クライアントアプリケーションの開発者は、 `.server .rosetta`パッケージに含まれているrosettaコマンドを使用して、RosettaAPIサーバーを別のプロセスとして起動する新しいコマンドを作成することもできます。コマンドの構成は、CosmosSDKのバージョンによって異なります。例はStargateの[simd]にあり、他のバージョンシリーズは[contrib .rosetta .simapp]にあります。

## 状態

提案

## 結果

### ポジティブ

-CosmosSDKでのRosettaAPIサポートをすぐに使用できます。
-ブロックリンクインターフェイスの標準化

## 参照

-https://www.rosetta-api.org.