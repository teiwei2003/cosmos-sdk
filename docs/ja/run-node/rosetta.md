# Rosetta

`rosetta`パッケージは、Coinbaseの[Rosetta API](https://www.rosetta-api.org)を実装します。 このドキュメントでは、RosettaAPI統合の使用方法について説明します。 動機と設計の選択については、[ADR 035](../architecture/adr-035-rosetta-api-support.md)を参照してください。

## Rosettaコマンドを追加

Rosetta APIサーバーは、CosmosSDKで開発されたチェーンのノードに接続するスタンドアロンサーバーです。

Rosetta APIサポートを有効にするには、アプリケーションのルートコマンドファイル(例:`appd/cmd/root.go`)に`RosettaCommand`を追加する必要があります。

`server`パッケージをインポートします。 

```go
    "github.com/cosmos/cosmos-sdk/server"
```

次の行を見つけます。

```go
initRootCmd(rootCmd, encodingConfig)
```

その行の後に、以下を追加します。

```go
rootCmd.AddCommand(
  server.RosettaCommand(encodingConfig.InterfaceRegistry, encodingConfig.Marshaler)
)
```

`RosettaCommand`関数は`rosetta`ルートコマンドを構築し、CosmosSDKの`server`パッケージで定義されます。

RosettaAPIで動作するようにCosmosSDKを更新したため、アプリケーションのルートコマンドファイルを更新するだけで済みます。

実装例は`simapp`パッケージにあります。

## Rosettaコマンドを使用する

アプリケーションCLIでRosettaを実行するには、次のコマンドを使用します。

```
appd rosetta --help
```

実行および公開されているアプリケーションのRosettaAPIエンドポイントをテストおよび実行するには、次のコマンドを使用します。

```
appd rosetta
     --blockchain "your application name (ex: gaia)"
     --network "your chain identifier (ex: testnet-1)"
     --tendermint "tendermint endpoint (ex: localhost:26657)"
     --grpc "gRPC endpoint (ex: localhost:9090)"
     --addr "rosetta binding address (ex: :8080)"
```

## 拡張機能

カスタム設定を使用して実装をカスタマイズおよび拡張する方法は2つあります。

### メッセージ拡張

ロゼッタが`sdk.Msg`を理解できるようにするために必要なのは、`rosetta.Msg`インターフェースを満たすメソッドをメッセージに追加することだけです。 その方法の例は、`MsgDelegate`などのステーキングタイプまたは`MsgSend`などのバンクタイプにあります。

### クライアントインターフェイスのオーバーライド

さらにカスタマイズが必要な場合は、クライアントタイプを埋め込み、カスタマイズが必要なメソッドをオーバーライドすることができます。 

例:

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

注:カスタマイズされたクライアントを使用する場合、必要なコンストラクターは**異なる**可能性があるため、コマンドは使用できません。そのため、新しいコンストラクターを作成する必要があります。 将来的には、余分なコードを記述せずに、カスタマイズされたクライアントを初期化する方法を提供する予定です。

### エラー拡張

ロゼッタはネットワークオプションに[返された]エラーを提供する必要があるため。 新しいロゼッタエラーを宣言するために、cosmos-rosetta-gatewayの`errors`パッケージを使用します。 

例:

```go
package custom_errors
import crgerrs "github.com/cosmos/cosmos-sdk/server/rosetta/lib/errors"

var customErrRetriable = true
var CustomError = crgerrs.RegisterError(100, "custom message", customErrRetriable, "description")
```

注:cosmos-rosetta-gatewayの`Server`.`Start`メソッドを呼び出す前に、エラーを登録する必要があります。 それ以外の場合、登録は無視されます。 同じコードのエラーも無視されます。