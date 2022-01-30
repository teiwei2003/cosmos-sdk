# トランザクションの生成、署名、およびブロードキャスト

このドキュメントでは、(署名されていない)トランザクションを生成し、(1つまたは複数のキーを使用して)署名し、ネットワークにブロードキャストする方法について説明します。{概要}

## CLIの使用

前のページで[ノードとの対話](./interact-node.md#using-the-cli)で見たように、トランザクションを送信する最も簡単な方法はCLIを使用することです。 たとえば、次のコマンドを実行します 

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake --chain-id my-test-chain --keyring-backend test
```

次の手順を実行します。

- 1つの`Msg`(`x/bank`の`MsgSend`)を使用してトランザクションを生成し、生成されたトランザクションをコンソールに出力します。
-`$ MY_VALIDATOR_ADDRESS`アカウントからトランザクションを送信するための確認をユーザーに依頼します。
- キーリングから`$ MY_VALIDATOR_ADDRESS`をフェッチします。 これが可能なのは、前の手順で[CLIのキーリングを設定](./keyring.md)したためです。
- 生成されたトランザクションにキーリングのアカウントで署名します。
- 署名されたトランザクションをネットワークにブロードキャストします。 これが可能なのは、CLIがノードのTendermintRPCエンドポイントに接続しているためです。

CLIは、必要なすべてのステップを使いやすいユーザーエクスペリエンスにバンドルします。 ただし、すべてのステップを個別に実行することも可能です。

### 取引の生成

トランザクションの生成は、任意の`tx`コマンドに`--generate-only`フラグを追加することで簡単に実行できます。例:

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake --chain-id my-test-chain --generate-only
```

これにより、署名されていないトランザクションがコンソールにJSONとして出力されます。 上記のコマンドに`> unsigned_tx.json`を追加することで、署名されていないトランザクションをファイルに保存することもできます(署名者間でより簡単に受け渡されます)。

### トランザクションへの署名

CLIを使用してトランザクションに署名するには、署名されていないトランザクションをファイルに保存する必要があります。 署名されていないトランザクションが、現在のディレクトリの`unsigned_tx.json`というファイルにあると仮定します(これを行う方法については、前の段落を参照してください)。 次に、次のコマンドを実行するだけです。

```bash
simd tx sign unsigned_tx.json --chain-id my-test-chain --keyring-backend test --from $MY_VALIDATOR_ADDRESS
```

このコマンドは、署名されていないトランザクションをデコードし、キーリングにすでに設定されている`$ MY_VALIDATOR_ADDRESS`のキーを使用して`SIGN_MODE_DIRECT`で署名します。 署名されたトランザクションはJSONとしてコンソールに出力されます。上記のように、`> signed_tx.json`を追加することでファイルに保存できます。

`txsign`コマンドで考慮すべきいくつかの便利なフラグ:

-`--sign-mode`:`SIGN_MODE_LEGACY_AMINO_JSON`を使用してトランザクションに署名するには、`amino-json`を使用できます。
-`--offline`:オフラインモードでサインインします。 これは、`tx sign`コマンドがノードに接続して、署名に必要な署名者のアカウント番号とシーケンスを取得しないことを意味します。 この場合、`--account-number`フラグと`--sequence`フラグを手動で指定する必要があります。 これは、オフライン署名、つまりインターネットにアクセスできない安全な環境での署名に役立ちます。

#### 複数の署名者による署名

::: 警告
少なくとも1人の署名者が`SIGN_MODE_DIRECT`を使用する、複数の署名者またはマルチシグアカウントでトランザクションに署名することはまだ不可能であることに注意してください。 詳細については、[このGithubの問題](https://github.com/cosmos/cosmos-sdk/issues/8141)をフォローしてください。
:::

複数の署名者による署名は、`txmultisign`コマンドを使用して実行されます。 このコマンドは、すべての署名者が`SIGN_MODE_LEGACY_AMINO_JSON`を使用することを前提としています。 フローは`tx sign`コマンドフローに似ていますが、署名されていないトランザクションファイルに署名する代わりに、各署名者が前の署名者によって署名されたファイルに署名します。`tx multisign`コマンドは、既存のトランザクションに署名を追加します。 署名者は、トランザクションで指定されたのと**同じ順序**でトランザクションに署名することが重要です。これは、`GetSigners()`メソッドを使用して取得できます。

たとえば、`unsigned_tx.json`から始めて、トランザクションに4人の署名者がいると仮定すると、次のように実行します。

```bash
# Let signer1 sign the unsigned tx.
simd tx multisign unsigned_tx.json signer_key_1 --chain-id my-test-chain --keyring-backend test > partial_tx_1.json
# Now signer1 will send the partial_tx_1.json to the signer2.
# Signer2 appends their signature:
simd tx multisign partial_tx_1.json signer_key_2 --chain-id my-test-chain --keyring-backend test > partial_tx_2.json
# Signer2 sends the partial_tx_2.json file to signer3, and signer3 can append his signature:
simd tx multisign partial_tx_2.json signer_key_3 --chain-id my-test-chain --keyring-backend test > partial_tx_3.json
```

### トランザクションのブロードキャスト

トランザクションのブロードキャストは、次のコマンドを使用して実行されます。

```bash
simd tx broadcast tx_signed.json
```

オプションで、`--broadcast-mode`フラグを渡して、ノードから受信する応答を指定できます。

-`block`:CLIはtxがブロックにコミットされるのを待ちます。
-`sync`:CLIはCheckTx実行応答のみを待機します。
-`async`:CLIはすぐに戻ります(トランザクションが失敗する可能性があります)。

### トランザクションのエンコード

gRPCまたはRESTエンドポイントを使用してトランザクションをブロードキャストするには、トランザクションを最初にエンコードする必要があります。 これは、CLIを使用して実行できます。

トランザクションのエンコードは、次のコマンドを使用して実行されます。

```bash
simd tx encode tx_signed.json
```

これにより、ファイルからトランザクションが読み取られ、Protobufを使用してシリアル化され、コンソールにトランザクションバイトがbase64として出力されます。

### トランザクションのデコード

CLIを使用して、トランザクションバイトをデコードすることもできます。

トランザクションのデコードは、次のコマンドを使用して実行されます。

```bash
simd tx decode [protobuf-byte-string]
```

これにより、トランザクションバイトがデコードされ、コンソールにトランザクションがJSONとして出力されます。 上記のコマンドに`> tx.json`を追加して、トランザクションをファイルに保存することもできます。

## プログラムでGoを使用

Cosmos SDKの`TxBuilder`インターフェースを使用して、Goを介してプログラムでトランザクションを操作することができます。

### 取引の生成

トランザクションを生成する前に、`TxBuilder`の新しいインスタンスを作成する必要があります。 Cosmos SDKはAminoトランザクションとProtobufトランザクションの両方をサポートしているため、最初のステップは、使用するエンコード方式を決定することです。`TxBuilder`はエンコーディングメカニズムを抽象化するため、AminoまたはProtobufのどちらを使用していても、以降のすべての手順は変更されません。 次のスニペットでは、Protobufを使用します。 

```go
import (
	"github.com/cosmos/cosmos-sdk/simapp"
)

func sendTx() error {
   //Choose your codec: Amino or Protobuf. Here, we use Protobuf, given by the
   //following function.
    encCfg := simapp.MakeTestEncodingConfig()

   //Create a new TxBuilder.
    txBuilder := encCfg.TxConfig.NewTxBuilder()

   //--snip--
}
```

トランザクションを送受信するいくつかのキーとアドレスを設定することもできます。 ここでは、チュートリアルの目的で、いくつかのダミーデータを使用してキーを作成します。 

```go
import (
	"github.com/cosmos/cosmos-sdk/testutil/testdata"
)

priv1, _, addr1 := testdata.KeyTestPubAddr()
priv2, _, addr2 := testdata.KeyTestPubAddr()
priv3, _, addr3 := testdata.KeyTestPubAddr()
```

Populating the`TxBuilder`can be done via its [methods](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/client/tx_config.go#L32-L45):

```go
import (
	banktypes "github.com/cosmos/cosmos-sdk/x/bank/types"
)

func sendTx() error {
   //--snip--

   //Define two x/bank MsgSend messages:
   //- from addr1 to addr3,
   //- from addr2 to addr3.
   //This means that the transactions needs two signers: addr1 and addr2.
    msg1 := banktypes.NewMsgSend(addr1, addr3, types.NewCoins(types.NewInt64Coin("atom", 12)))
    msg2 := banktypes.NewMsgSend(addr2, addr3, types.NewCoins(types.NewInt64Coin("atom", 34)))

    err := txBuilder.SetMsgs(msg1, msg2)
    if err != nil {
        return err
    }

    txBuilder.SetGasLimit(...)
    txBuilder.SetFeeAmount(...)
    txBuilder.SetMemo(...)
    txBuilder.SetTimeoutHeight(...)
}
```

この時点で、`TxBuilder`の基礎となるトランザクションに署名する準備ができています。

### 取引に署名する

デフォルトで`SIGN_MODE_DIRECT`を使用するProtobufを使用するようにencodingconfigを設定します。 [ADR-020](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/docs/architecture/adr-020-protobuf-transaction-encoding.md)に従って、各署名者は 他のすべての署名者の`SignerInfo`に署名します。 これは、2つのステップを順番に実行する必要があることを意味します。

- 署名者ごとに、署名者の`SignerInfo`を`TxBuilder`内に入力します。
- すべての`SignerInfo`が入力されたら、署名者ごとに、`SignDoc`(署名されるペイロード)に署名します。

現在の`TxBuilder`のAPIでは、両方のステップが同じメソッド`SetSignatures()`を使用して実行されます。 現在のAPIでは、最初に[SetSignatures()]のラウンドを_空の署名で_実行して、[SignerInfo]にデータを入力し、2回目の[SetSignatures()]を実行して実際に正しいペイロードに署名する必要があります。

```go
import (
    cryptotypes "github.com/cosmos/cosmos-sdk/crypto/types"
	"github.com/cosmos/cosmos-sdk/types/tx/signing"
	xauthsigning "github.com/cosmos/cosmos-sdk/x/auth/signing"
)

func sendTx() error {
   //--snip--

    privs := []cryptotypes.PrivKey{priv1, priv2}
    accNums:= []uint64{..., ...}//The accounts' account numbers
    accSeqs:= []uint64{..., ...}//The accounts' sequence numbers

   //First round: we gather all the signer infos. We use the "set empty
   //signature" hack to do that.
    var sigsV2 []signing.SignatureV2
    for i, priv := range privs {
        sigV2 := signing.SignatureV2{
            PubKey: priv.PubKey(),
            Data: &signing.SingleSignatureData{
                SignMode:  encCfg.TxConfig.SignModeHandler().DefaultMode(),
                Signature: nil,
            },
            Sequence: accSeqs[i],
        }

        sigsV2 = append(sigsV2, sigV2)
    }
    err := txBuilder.SetSignatures(sigsV2...)
    if err != nil {
        return err
    }

   //Second round: all signer infos are set, so each signer can sign.
    sigsV2 = []signing.SignatureV2{}
    for i, priv := range privs {
        signerData := xauthsigning.SignerData{
            ChainID:       chainID,
            AccountNumber: accNums[i],
            Sequence:      accSeqs[i],
        }
        sigV2, err := tx.SignWithPrivKey(
            encCfg.TxConfig.SignModeHandler().DefaultMode(), signerData,
            txBuilder, priv, encCfg.TxConfig, accSeqs[i])
        if err != nil {
            return nil, err
        }

        sigsV2 = append(sigsV2, sigV2)
    }
    err = txBuilder.SetSignatures(sigsV2...)
    if err != nil {
        return err
    }
}
```

これで、`TxBuilder`が正しく入力されます。 それを印刷するには、初期エンコーディング構成`encCfg`から`TxConfig`インターフェースを使用できます。

```go
func sendTx() error {
   //--snip--

   //Generated Protobuf-encoded bytes.
    txBytes, err := encCfg.TxConfig.TxEncoder()(txBuilder.GetTx())
    if err != nil {
        return err
    }

   //Generate a JSON string.
    txJSONBytes, err := encCfg.TxConfig.TxJSONEncoder()(txBuilder.GetTx())
    if err != nil {
        return err
    }
    txJSON := string(txJSONBytes)
}
```

### 取引の放送

トランザクションをブロードキャストするための好ましい方法はgRPCを使用することですが、REST(`gRPC-gateway`経由)またはTendermintRPCを使用することも可能です。 これらのメソッドの違いの概要は[ここ](../core/grpc_rest.md)に公開されています。 このチュートリアルでは、gRPCメソッドについてのみ説明します。

```go
import (
    "context"
    "fmt"

	"google.golang.org/grpc"

	"github.com/cosmos/cosmos-sdk/types/tx"
)

func sendTx(ctx context.Context) error {
   //--snip--

   //Create a connection to the gRPC server.
    grpcConn := grpc.Dial(
        "127.0.0.1:9090",//Or your gRPC server address.
        grpc.WithInsecure(),//The Cosmos SDK doesn't support any transport security mechanism.
    )
    defer grpcConn.Close()

   //Broadcast the tx via gRPC. We create a new client for the Protobuf Tx
   //service.
    txClient := tx.NewServiceClient(grpcConn)
   //We then call the BroadcastTx method on this client.
    grpcRes, err := txClient.BroadcastTx(
        ctx,
        &tx.BroadcastTxRequest{
            Mode:    tx.BroadcastMode_BROADCAST_MODE_SYNC,
            TxBytes: txBytes,//Proto-binary of the signed transaction, see previous step.
        },
    )
    if err != nil {
        return err
    }

    fmt.Println(grpcRes.TxResponse.Code)//Should be`0`if the tx is successful

    return nil
}
```

#### 取引のシミュレーション

トランザクションをブロードキャストする前に、トランザクションをドライランして、実際にコミットせずにトランザクションに関する情報を推定したい場合があります。 これはトランザクションのシミュレーションと呼ばれ、次のように実行できます。

```go
import (
	"context"
	"fmt"
	"testing"

	"github.com/cosmos/cosmos-sdk/client"
	"github.com/cosmos/cosmos-sdk/types/tx"
	authtx "github.com/cosmos/cosmos-sdk/x/auth/tx"
)

func simulateTx() error {
   //--snip--

   //Simulate the tx via gRPC. We create a new client for the Protobuf Tx
   //service.
    txClient := tx.NewServiceClient(grpcConn)
    txBytes :=/* Fill in with your signed transaction bytes. */

   //We then call the Simulate method on this client.
    grpcRes, err := txClient.Simulate(
        context.Background(),
        &tx.SimulateRequest{
            TxBytes: txBytes,
        },
    )
    if err != nil {
        return err
    }

    fmt.Println(grpcRes.GasInfo)//Prints estimated gas used.

    return nil
}
```

## gRPCの使用

gRPCを使用してトランザクションを生成または署名することはできず、ブロードキャストするだけです。 gRPCを使用してトランザクションをブロードキャストするには、CLIを使用するか、Goを使用してプログラムでトランザクションを生成、署名、およびエンコードする必要があります。

### 取引の放送

gRPCエンドポイントを使用してトランザクションをブロードキャストするには、次のように`BroadcastTx`リクエストを送信します。ここで、`txBytes`は署名されたトランザクションのprotobufでエンコードされたバイトです。

```bash
grpcurl -plaintext \
    -d '{"tx_bytes":"{{txBytes}}","mode":"BROADCAST_MODE_SYNC"}' \
    localhost:9090 \
    cosmos.tx.v1beta1.Service/BroadcastTx
```

## RESTの使用

RESTを使用してトランザクションを生成または署名することはできず、ブロードキャストするだけです。 RESTを使用してトランザクションをブロードキャストするには、CLIを使用するか、Goを使用してプログラムでトランザクションを生成、署名、およびエンコードする必要があります。

### 取引の放送

RESTエンドポイント(`gRPC-gateway`によって提供される)を使用してトランザクションをブロードキャストするには、次のようにPOSTリクエストを送信します。ここで、`txBytes`は署名されたトランザクションのprotobufエンコードされたバイトです。

```bash
curl -X POST \
    -H "Content-Type: application/json" \
    -d'{"tx_bytes":"{{txBytes}}","mode":"BROADCAST_MODE_SYNC"}' \
    localhost:1317/cosmos/tx/v1beta1/txs
```

## CosmOS(JavaScriptおよびTypeScript)の使用

CosmJSは、Webアプリケーションに埋め込むことができるJavaScriptでクライアントライブラリを構築することを目的としています。 詳細については、[https://cosmos.github.io/cosmjs](https://cosmos.github.io/cosmjs)を参照してください。 2021年1月の時点で、CosmJSのドキュメントはまだ作成中です。