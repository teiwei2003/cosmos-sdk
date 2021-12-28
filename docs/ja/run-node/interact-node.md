# ノードとの相互作用

ノードと対話するには、CLIを使用する、gRPCを使用する、RESTエンドポイントを使用するなどの複数の方法があります。{概要}

## 前提条件の読み

- [gRPC、REST、Tendermintエンドポイント](../core/grpc_rest.md){prereq}
- [ノードの実行](./run-node.md){前提条件}

## CLIの使用

チェーンが実行されたので、作成した最初のアカウントから2番目のアカウントにトークンを送信してみます。 新しいターミナルウィンドウで、次のクエリコマンドを実行することから始めます。

```bash
simd query bank balances $MY_VALIDATOR_ADDRESS --chain-id my-test-chain
```

作成したアカウントの現在の残高が表示されます。これは、付与した「ステーク」の元の残高から「gentx」を介して委任した金額を差し引いたものに等しくなります。 次に、2番目のアカウントを作成します。

```bash
simd keys add recipient --keyring-backend test

# Put the generated address in a variable for later use.
RECIPIENT=$(simd keys show recipient -a --keyring-backend test)
```

上記のコマンドは、チェーンにまだ登録されていないローカルキーペアを作成します。 アカウントは、別のアカウントからトークンを初めて受け取ったときに作成されます。 次に、次のコマンドを実行して、トークンを`recipient`アカウントに送信します。

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000000stake --chain-id my-test-chain --keyring-backend test

# Check that the recipient account did receive the tokens.
simd query bank balances $RECIPIENT --chain-id my-test-chain
```

最後に、`recipient`アカウントに送信されたステークトークンの一部をバリデーターに委任します。

```bash
simd tx staking delegate $(simd keys show my_validator --bech val -a --keyring-backend test) 500stake --from recipient --chain-id my-test-chain --keyring-backend test

# Query the total delegations to`validator`.
simd query staking delegations-to $(simd keys show my_validator --bech val -a --keyring-backend test) --chain-id my-test-chain
```

2つの委任が表示されます。最初の委任は`gentx`から作成され、2番目の委任は`recipient`アカウントから実行したばかりです。

## gRPCの使用

Protobufエコシステムは、`* .proto`ファイルからさまざまな言語へのコード生成など、さまざまなユースケース向けのツールを開発しました。これらのツールを使用すると、クライアントを簡単に構築できます。多くの場合、クライアント接続(つまり、トランスポート)は、非常に簡単に接続および交換できます。 最も人気のあるトランスポートの1つである[gRPC](../core/grpc_rest.md)を見てみましょう。

コード生成ライブラリは独自の技術スタックに大きく依存しているため、次の3つの選択肢のみを紹介します。

- 一般的なデバッグとテスト用の`grpcurl`、
- プログラムでGoを介して、
- JavaScript/TypeScript開発者向けのCosmJS。

### grpcurl

[grpcurl](https://github.com/fullstorydev/grpcurl)は`curl`に似ていますが、gRPC用です。 Goライブラリとしても利用できますが、デバッグとテストの目的でCLIコマンドとしてのみ使用します。 前のリンクの指示に従ってインストールしてください。

ローカルノードが実行されている(ローカルネットまたはライブネットワークに接続されている)と仮定すると、次のコマンドを実行して、使用可能なProtobufサービスを一覧表示できます(`localhost：9000`を別のgRPCサーバーエンドポイントに置き換えることができます) ノード。[`app.toml`](../run-node/run-node.md＃configuring-the-node-using-apptoml))内の`grpc.address`フィールドで構成されます。 

```bash
grpcurl -plaintext localhost:9090 list
```

`cosmos.bank.v1beta1.Query`のようなgRPCサービスのリストが表示されます。 これはリフレクションと呼ばれ、使用可能なすべてのエンドポイントの説明を返すProtobufエンドポイントです。 これらはそれぞれ異なるProtobufサービスを表し、各サービスはクエリを実行できる複数のRPCメソッドを公開します。

サービスの説明を取得するには、次のコマンドを実行できます。 

```bash
grpcurl \
    localhost:9090 \
    describe cosmos.bank.v1beta1.Query                  # Service we want to inspect
```

RPC呼び出しを実行して、ノードに情報を照会することもできます。

```bash
grpcurl \
    -plaintext
    -d '{"address":"$MY_VALIDATOR"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

利用可能なすべてのgRPCクエリエンドポイントのリストは[近日公開](https://github.com/cosmos/cosmos-sdk/issues/7786)です。

#### grpcurlを使用して履歴状態を照会する

[gRPCメタデータ](https://github.com/grpc/grpc-go/blob/master/Documentation/grpc-metadata.md)をクエリに渡すことで履歴データをクエリすることもできます：`x-cosmos -block-height`メタデータには、クエリするブロックが含まれている必要があります。 上記のようにgrpcurlを使用すると、コマンドは次のようになります。

```bash
grpcurl \
    -plaintext \
    -H "x-cosmos-block-height: 279256" \
    -d '{"address":"$MY_VALIDATOR"}' \
    localhost:9090 \
    cosmos.bank.v1beta1.Query/AllBalances
```

そのブロックの状態がノードによってまだプルーニングされていないと仮定すると、このクエリは空でない応答を返す必要があります。

### Go経由でプログラム的に

次のスニペットは、Goプログラム内でgRPCを使用して状態をクエリする方法を示しています。 アイデアは、gRPC接続を作成し、Protobufで生成されたクライアントコードを使用してgRPCサーバーにクエリを実行することです。 

```go
import (
    "context"
    "fmt"

	"google.golang.org/grpc"

    sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/types/tx"
)

func queryState() error{
    myAddress, err := sdk.AccAddressFromBech32("cosmos1...")
    if err != nil{
        return err
    }

   //Create a connection to the gRPC server.
    grpcConn := grpc.Dial(
        "127.0.0.1:9090",//your gRPC server address.
        grpc.WithInsecure(),//The Cosmos SDK doesn't support any transport security mechanism.
    )
    defer grpcConn.Close()

   //This creates a gRPC client to query the x/bank service.
    bankClient := banktypes.NewQueryClient(grpcConn)
    bankRes, err := bankClient.Balance(
        context.Background(),
        &banktypes.QueryBalanceRequest{Address: myAddress, Denom: "atom"},
    )
    if err != nil{
        return err
    }

    fmt.Println(bankRes.GetBalance())//Prints the account balance

    return nil
}
```

クエリクライアント(ここでは`x/bank`を使用しています)を他のProtobufサービスから生成されたものに置き換えることができます。 利用可能なすべてのgRPCクエリエンドポイントのリストは[近日公開](https://github.com/cosmos/cosmos-sdk/issues/7786)です。

#### Goを使用して履歴状態を照会します

履歴ブロックのクエリは、gRPCリクエストにブロックの高さのメタデータを追加することで実行されます。 

```go
import (
    "context"
    "fmt"

    "google.golang.org/grpc"
    "google.golang.org/grpc/metadata"

    grpctypes "github.com/cosmos/cosmos-sdk/types/grpc"
	"github.com/cosmos/cosmos-sdk/types/tx"
)

func queryState() error{
   //--snip--

    var header metadata.MD
    bankRes, err = bankClient.Balance(
        metadata.AppendToOutgoingContext(context.Background(), grpctypes.GRPCBlockHeightHeader, "12"),//Add metadata to request
        &banktypes.QueryBalanceRequest{Address: myAddress, Denom: denom},
        grpc.Header(&header),//Retrieve header from response
    )
    if err != nil{
        return err
    }
    blockHeight = header.Get(grpctypes.GRPCBlockHeightHeader)

    fmt.Println(blockHeight)//Prints the block height (12)

    return nil
}
```

### CosmJS

CosmJSのドキュメントは、[https://cosmos.github.io/cosmjs](https://cosmos.github.io/cosmjs)にあります。 2021年1月の時点で、CosmJSのドキュメントはまだ作成中です。

## RESTエンドポイントの使用

[gRPCガイド](../core/grpc_rest.md)で説明されているように、Cosmos SDKのすべてのgRPCサービスは、gRPCゲートウェイを介してより便利なRESTベースのクエリで利用できるようになっています。 URLパスの形式は、Protobufサービスメソッドの完全修飾名に基づいていますが、最終的なURLがより慣用的に見えるように、小さなカスタマイズが含まれている場合があります。 たとえば、`cosmos.bank.v1beta1.Query/AllBalances`メソッドのRESTエンドポイントは`GET/cosmos/bank/v1beta1/balances/{address}`です。 リクエスト引数はクエリパラメータとして渡されます。

具体的な例として、残高要求を行うための`curl`コマンドは次のとおりです。

```bash
curl \
    -X GET \
    -H "Content-Type: application/json" \
    http://localhost:1317/cosmos/bank/v1beta1/balances/$MY_VALIDATOR
```

必ず`localhost：1317`を、`api.address`フィールドで設定されたノードのRESTエンドポイントに置き換えてください。

使用可能なすべてのRESTエンドポイントのリストは、Swagger仕様ファイルとして入手でき、`localhost：1317/swagger`で表示できます。 [`app.toml`](../run-node/run-node.md＃configuring-the-node-using-apptoml)ファイルで`api.swagger`フィールドがtrueに設定されていることを確認してください。

### RESTを使用して履歴状態を照会する

履歴状態のクエリは、HTTPヘッダー`x-cosmos-block-height`を使用して行われます。 たとえば、curlコマンドは次のようになります。 

```bash
curl \
    -X GET \
    -H "Content-Type: application/json" \
    -H "x-cosmos-block-height: 279256"
    http://localhost:1317/cosmos/bank/v1beta1/balances/$MY_VALIDATOR
```

そのブロックの状態がノードによってまだプルーニングされていないと仮定すると、このクエリは空でない応答を返す必要があります。

### クロスオリジンリソースシェアリング(CORS)

[CORSポリシー](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)は、セキュリティを支援するためにデフォルトで有効になっていません。 パブリック環境でrest-serverを使用する場合は、リバースプロキシを提供することをお勧めします。これは、[nginx](https://www.nginx.com/)を使用して実行できます。 テストと開発の目的で、[`app.toml`](../run-node/run-node.md＃configuring-the-node-using-apptoml)内に`enabled-unsafe-cors`フィールドがあります。

## 次へ{非表示}

gRPCとRESTを使用してトランザクションを送信するには、トランザクションを生成し、署名し、最後にブロードキャストするという追加の手順が必要です。 [トランザクションの生成と署名](./txs.md)についてお読みください。{非表示} 