# 生成、签署和广播交易

本文档描述了如何生成(未签名)交易、对其进行签名(使用一个或多个密钥)并将其广播到网络。 {概要}

## 使用 CLI

发送交易的最简单方法是使用 CLI，正如我们在上一页中看到的 [与节点交互](./interact-node.md#using-the-cli)。 例如，运行以下命令 

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake --chain-id my-test-chain --keyring-backend test
```

将运行以下步骤:

- 用一个 `Msg`(`x/bank` 的 `MsgSend`)生成一个交易，并将生成的交易打印到控制台。
- 要求用户确认从`$MY_VALIDATOR_ADDRESS` 帐户发送交易。
- 从钥匙圈中获取`$MY_VALIDATOR_ADDRESS`。 这是可能的，因为我们在上一步中[设置了 CLI 的密钥环](./keyring.md)。
- 使用密钥环的帐户签署生成的交易。
- 将签名的交易广播到网络。 这是可能的，因为 CLI 连接到节点的 Tendermint RPC 端点。

CLI 将所有必要的步骤捆绑到易于使用的用户体验中。 但是，也可以单独运行所有步骤。

### 生成交易

生成交易可以简单地通过在任何 `tx` 命令上附加 `--generate-only` 标志来完成，例如: 

```bash
simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000stake --chain-id my-test-chain --generate-only
```

这将在控制台中将未签名的交易输出为 JSON。 我们还可以通过将 `> unsigned_tx.json` 附加到上述命令，将未签名的交易保存到一个文件中(以便在签名者之间更容易地传递)。

### 签署交易

使用 CLI 签署交易需要将未签名的交易保存在文件中。 让我们假设未签名的交易位于当前目录中名为“unsigned_tx.json”的文件中(请参阅上一段了解如何执行此操作)。 然后，只需运行以下命令: 

```bash
simd tx sign unsigned_tx.json --chain-id my-test-chain --keyring-backend test --from $MY_VALIDATOR_ADDRESS
```

此命令将解码未签名的交易，并使用“SIGN_MODE_DIRECT”和“$MY_VALIDATOR_ADDRESS”的密钥对其进行签名，我们已经在密钥环中设置了该密钥。签名的交易将作为 JSON 输出到控制台，如上所述，我们可以通过附加 `> signed_tx.json` 将其保存到文件中。

在 `tx sign` 命令中需要考虑的一些有用标志:

- `--sign-mode`:你可以使用 `amino-json` 使用 `SIGN_MODE_LEGACY_AMINO_JSON` 来签署交易，
- `--offline`:在离线模式下登录。这意味着 `tx sign` 命令不会连接到节点来检索签名者的帐号和序列，这两者都是签名所必需的。在这种情况下，您必须手动提供 `--account-number` 和 `--sequence` 标志。这对于离线签名很有用，即在无法访问互联网的安全环境中签名。

#### 与多个签名者签名

::: 警告
请注意，使用多个签名者或多重签名帐户签署交易(其中至少有一个签​​名者使用“SIGN_MODE_DIRECT”)尚不可能。您可以关注 [this Github issue](https://github.com/cosmos/cosmos-sdk/issues/8141) 了解更多信息。
:::

使用“tx multisign”命令完成与多个签名者的签名。此命令假定所有签名者都使用“SIGN_MODE_LEGACY_AMINO_JSON”。该流程类似于“tx sign”命令流程，但不是签署未签名的交易文件，而是每个签署者签署由先前签署者签署的文件。 `tx multisign` 命令会将签名附加到现有交易中。签名者以与交易给出的相同顺序**签署交易非常重要，该顺序可以使用`GetSigners()`方法进行检索。

例如，从 `unsigned_tx.json` 开始，假设交易有 4 个签名者，我们将运行: 

```bash
# Let signer1 sign the unsigned tx.
simd tx multisign unsigned_tx.json signer_key_1 --chain-id my-test-chain --keyring-backend test > partial_tx_1.json
# Now signer1 will send the partial_tx_1.json to the signer2.
# Signer2 appends their signature:
simd tx multisign partial_tx_1.json signer_key_2 --chain-id my-test-chain --keyring-backend test > partial_tx_2.json
# Signer2 sends the partial_tx_2.json file to signer3, and signer3 can append his signature:
simd tx multisign partial_tx_2.json signer_key_3 --chain-id my-test-chain --keyring-backend test > partial_tx_3.json
```

### Broadcasting a Transaction

使用以下命令广播事务: 

```bash
simd tx broadcast tx_signed.json
```

您可以选择传递 `--broadcast-mode` 标志来指定从节点接收哪个响应:

- `block`:CLI 等待 tx 在一个块中提交。
- `sync`:CLI 仅等待 CheckTx 执行响应。
- `async`:CLI 立即返回(交易可能会失败)。

### 对交易进行编码

为了使用 gRPC 或 REST 端点广播交易，需要先对交易进行编码。 这可以使用 CLI 来完成。

使用以下命令对交易进行编码: 

```bash
simd tx encode tx_signed.json
```

这将从文件中读取交易，使用 Protobuf 对其进行序列化，并在控制台中以 base64 格式输出交易字节。

### 解码交易

CLI 还可用于解码事务字节。

使用以下命令对交易进行解码: 

```bash
simd tx decode [protobuf-byte-string]
```

这将解码交易字节并在控制台中将交易输出为 JSON。 您还可以通过将 `> tx.json` 附加到上述命令来将交易保存到文件中。

## 用 Go 编程

可以使用 Cosmos SDK 的“TxBuilder”接口通过 Go 以编程方式操作事务。

### 生成交易

在生成交易之前，需要创建一个“TxBuilder”的新实例。 由于 Cosmos SDK 支持 Amino 和 Protobuf 事务，第一步是决定使用哪种编码方案。 所有后续步骤都保持不变，无论您使用的是 Amino 还是 Protobuf，因为“TxBuilder”抽象了编码机制。 在以下代码段中，我们将使用 Protobuf。 

```go
import (
	"github.com/cosmos/cosmos-sdk/simapp"
)

func sendTx() error {
    // Choose your codec: Amino or Protobuf. Here, we use Protobuf, given by the
    // following function.
    encCfg := simapp.MakeTestEncodingConfig()

    // Create a new TxBuilder.
    txBuilder := encCfg.TxConfig.NewTxBuilder()

    // --snip--
}
```

我们还可以设置一些将发送和接收交易的密钥和地址。 在这里，出于本教程的目的，我们将使用一些虚拟数据来创建密钥。 

```go
import (
	"github.com/cosmos/cosmos-sdk/testutil/testdata"
)

priv1, _, addr1 := testdata.KeyTestPubAddr()
priv2, _, addr2 := testdata.KeyTestPubAddr()
priv3, _, addr3 := testdata.KeyTestPubAddr()
```

Populating the `TxBuilder` can be done via its [methods](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/client/tx_config.go#L32-L45):

```go
import (
	banktypes "github.com/cosmos/cosmos-sdk/x/bank/types"
)

func sendTx() error {
    // --snip--

    // Define two x/bank MsgSend messages:
    // - from addr1 to addr3,
    // - from addr2 to addr3.
    // This means that the transactions needs two signers: addr1 and addr2.
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

此时，`TxBuilder` 的底层交易已准备好进行签名。

### 签署交易

我们将编码配置设置为使用 Protobuf，默认使用 `SIGN_MODE_DIRECT`。 根据 [ADR-020](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/docs/architecture/adr-020-protobuf-transaction-encoding.md)，每个签名者需要 签署所有其他签名者的`SignerInfo`s。 这意味着我们需要依次执行两个步骤:

- 对于每个签名者，在 `TxBuilder` 中填充签名者的 `SignerInfo`，
- 一旦所有`SignerInfo`s 被填充，对于每个签名者，签署`SignDoc`(要签名的有效负载)。

在当前的 `TxBuilder` 的 API 中，这两个步骤都是使用相同的方法完成的:`SetSignatures()`。 当前的 API 要求我们首先执行一轮 `SetSignatures()` _带有空签名_，只填充 `SignerInfo`s，第二轮 `SetSignatures()` 以实际签署正确的有效负载。

```go
import (
    cryptotypes "github.com/cosmos/cosmos-sdk/crypto/types"
	"github.com/cosmos/cosmos-sdk/types/tx/signing"
	xauthsigning "github.com/cosmos/cosmos-sdk/x/auth/signing"
)

func sendTx() error {
    // --snip--

    privs := []cryptotypes.PrivKey{priv1, priv2}
    accNums:= []uint64{..., ...} // The accounts' account numbers
    accSeqs:= []uint64{..., ...} // The accounts' sequence numbers

    // First round: we gather all the signer infos. We use the "set empty
    // signature" hack to do that.
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

    // Second round: all signer infos are set, so each signer can sign.
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

“TxBuilder”现在已正确填充。 要打印它，您可以使用初始编码配置 `encCfg` 中的 `TxConfig` 接口: 

```go
func sendTx() error {
    // --snip--

    // Generated Protobuf-encoded bytes.
    txBytes, err := encCfg.TxConfig.TxEncoder()(txBuilder.GetTx())
    if err != nil {
        return err
    }

    // Generate a JSON string.
    txJSONBytes, err := encCfg.TxConfig.TxJSONEncoder()(txBuilder.GetTx())
    if err != nil {
        return err
    }
    txJSON := string(txJSONBytes)
}
```

### Broadcasting a Transaction

广播交易的首选方式是使用 gRPC，但也可以使用 REST(通过 `gRPC-gateway`)或 Tendermint RPC。 [此处](../core/grpc_rest.md) 公开了这些方法之间差异的概述。 在本教程中，我们将仅描述 gRPC 方法。

```go
import (
    "context"
    "fmt"

	"google.golang.org/grpc"

	"github.com/cosmos/cosmos-sdk/types/tx"
)

func sendTx(ctx context.Context) error {
    // --snip--

    // Create a connection to the gRPC server.
    grpcConn := grpc.Dial(
        "127.0.0.1:9090", // Or your gRPC server address.
        grpc.WithInsecure(), // The Cosmos SDK doesn't support any transport security mechanism.
    )
    defer grpcConn.Close()

    // Broadcast the tx via gRPC. We create a new client for the Protobuf Tx
    // service.
    txClient := tx.NewServiceClient(grpcConn)
    // We then call the BroadcastTx method on this client.
    grpcRes, err := txClient.BroadcastTx(
        ctx,
        &tx.BroadcastTxRequest{
            Mode:    tx.BroadcastMode_BROADCAST_MODE_SYNC,
            TxBytes: txBytes, // Proto-binary of the signed transaction, see previous step.
        },
    )
    if err != nil {
        return err
    }

    fmt.Println(grpcRes.TxResponse.Code) // Should be `0` if the tx is successful

    return nil
}
```

#### Simulating a Transaction

在广播一个事务之前，我们有时可能想要试运行该事务，以在不实际提交它的情况下估计有关该事务的一些信息。 这称为模拟交易，可以按如下方式完成:

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
    // --snip--

    // Simulate the tx via gRPC. We create a new client for the Protobuf Tx
    // service.
    txClient := tx.NewServiceClient(grpcConn)
    txBytes := /* Fill in with your signed transaction bytes. */

    // We then call the Simulate method on this client.
    grpcRes, err := txClient.Simulate(
        context.Background(),
        &tx.SimulateRequest{
            TxBytes: txBytes,
        },
    )
    if err != nil {
        return err
    }

    fmt.Println(grpcRes.GasInfo) // Prints estimated gas used.

    return nil
}
```

## Using gRPC

不可能使用 gRPC 生成或签署交易，只能广播一个。 为了使用 gRPC 广播交易，您需要使用 CLI 或使用 Go 以编程方式生成、签署和编码交易。

### 广播交易

可以通过发送如下的“BroadcastTx”请求来使用 gRPC 端点广播交易，其中“txBytes”是签名交易的 protobuf 编码字节:

```bash
grpcurl -plaintext \
    -d '{"tx_bytes":"{{txBytes}}","mode":"BROADCAST_MODE_SYNC"}' \
    localhost:9090 \
    cosmos.tx.v1beta1.Service/BroadcastTx
```

## 使用 REST

不可能使用 REST 生成或签署交易，只能广播一个。 为了使用 REST 广播交易，您需要使用 CLI 或使用 Go 以编程方式生成、签署和编码交易。

### 广播交易

使用 REST 端点(由 `gRPC-gateway` 提供服务)广播交易可以通过如下发送 POST 请求来完成，其中 `txBytes` 是签名交易的 protobuf 编码字节: 

```bash
curl -X POST \
    -H "Content-Type: application/json" \
    -d'{"tx_bytes":"{{txBytes}}","mode":"BROADCAST_MODE_SYNC"}' \
    localhost:1317/cosmos/tx/v1beta1/txs
```

## 使用 CosmJS(JavaScript 和 TypeScript)

CosmJS 旨在用 JavaScript 构建可以嵌入 Web 应用程序的客户端库。 请参阅 [https://cosmos.github.io/cosmjs](https://cosmos.github.io/cosmjs) 了解更多信息。 截至 2021 年 1 月，CosmJS 文档仍在进行中。 
