# アカウント

このドキュメントでは、CosmosSDKの組み込みアカウントと公開鍵システムを紹介します。 {まとめ}

### 読むための前提条件

-[Cosmos SDKアプリケーション分析](./app-anatomy.md){前提条件}

## アカウント定義

Cosmos SDKでは、_account_は_public key_`PubKey`と_privatekey_`PrivKey`のペアを指定します。 [公開鍵]は、アプリケーション内のユーザー(およびその他の関係者)を識別するために使用されるさまざまな[アドレス]を生成するために導出できます。 `Addresses`は、` message`の送信者を識別するために[`message`s](../building-modules/messages-and-queries.md#messages)にも関連付けられています。 `PrivKey`は、[デジタル署名](#signatures)を生成して、指定された[メッセージ]の[PrivKey]に関連付けられた[アドレス]を証明するために使用されます。

HDキーの導出には、Cosmos SDKは[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)と呼ばれる標準を使用します。 BIP32を使用すると、ユーザーはHDウォレットを作成できます([BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)で指定)-最初のシークレットから派生したアカウントシードのセット。シードは通常、12語または24語のニーモニックによって作成されます。単一のシードは、一方向の暗号化機能を使用して、任意の数の[PrivKey]を導出できます。次に、 `PubKey`を` PrivKey`から派生させることができます。当然のことながら、ニーモニックは最も機密性の高い情報です。ニーモニックを保持しておけば、秘密鍵をいつでも再生成できるからです。 
```
     Account 0                         Account 1                         Account 2

+------------------+              +------------------+               +------------------+
|                  |              |                  |               |                  |
|    Address 0     |              |    Address 1     |               |    Address 2     |
|        ^         |              |        ^         |               |        ^         |
|        |         |              |        |         |               |        |         |
|        |         |              |        |         |               |        |         |
|        |         |              |        |         |               |        |         |
|        +         |              |        +         |               |        +         |
|  Public key 0    |              |  Public key 1    |               |  Public key 2    |
|        ^         |              |        ^         |               |        ^         |
|        |         |              |        |         |               |        |         |
|        |         |              |        |         |               |        |         |
|        |         |              |        |         |               |        |         |
|        +         |              |        +         |               |        +         |
|  Private key 0   |              |  Private key 1   |               |  Private key 2   |
|        ^         |              |        ^         |               |        ^         |
+------------------+              +------------------+               +------------------+
         |                                 |                                  |
         |                                 |                                  |
         |                                 |                                  |
         +--------------------------------------------------------------------+
                                           |
                                           |
                                 +---------+---------+
                                 |                   |
                                 |  Master PrivKey   |
                                 |                   |
                                 +-------------------+
                                           |
                                           |
                                 +---------+---------+
                                 |                   |
                                 |  Mnemonic (Seed)  |
                                 |                   |
                                 +-------------------+
```

Cosmos SDKでは、キーは[`Keyring`](#keyring)という名前のオブジェクトによって保存および管理されます。 

## キー、アカウント、アドレス、署名

ユーザーを認証する主な方法は、[デジタル署名](https://en.wikipedia.org/wiki/Digital_signature)を使用することです。ユーザーは自分の秘密鍵を使用してトランザクションに署名します。署名の検証は、関連付けられた公開鍵を使用して行われます。チェーンでの署名検証の目的で、公開鍵を[Account]オブジェクト(および正しいトランザクション検証に必要なその他のデータ)に格納します。

ノードでは、すべてのデータがシリアル化され、プロトコルバッファを使用して保存されます。

Cosmos SDKは、デジタル署名を作成するための次のデジタルキースキームをサポートしています。

-`secp256k1`、[Cosmos SDK` crypto/keys/secp256k1`パッケージ](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keys/secp256k1/secp256k1。gowith )。
-`secp256r1`、[Cosmos SDK` crypto/keys/secp256r1`パッケージ](https://github.com/cosmos/cosmos-sdk/blob/master/crypto/keys/secp256r1/pubkey.go)に実装されています。
-`tm-ed25519`、[Cosmos SDK` crypto/keys/ed25519`パッケージ](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keys/ed25519/ed25519に実装されています。行く)。このソリューションは、コンセンサス検証のみをサポートします。 

|              | Address length in bytes | Public key length in bytes | Used for transaction authentication | Used for consensus (tendermint) |
|:------------:|:-----------------------:|:--------------------------:|:-----------------------------------:|:-------------------------------:|
| `secp256k1`  | 20                      |                         33 | yes                                 | no                              |
| `secp256r1`  | 32                      |                         33 | yes                                 | no                              |
| `tm-ed25519` | -- not used --          |                         32 | no                                  | yes                             |

## 住所

`Addresses`と` PubKey`はどちらも、アプリケーションの参加者を識別する公開情報です。 `アカウント`は認証情報を保存するために使用されます。 基本的なアカウントの実装は、[BaseAccount]オブジェクトによって提供されます。

各アカウントは、公開鍵から派生した一連のバイトである[アドレス]によって識別されます。 Cosmos SDKでは、アカウントを使用するコンテキストを指定するために3種類のアドレスを定義します。

-`AccAddress`は、ユーザー( `message`の送信者)を識別します。
-`ValAddress`はバリデーター演算子を識別します。
-`ConsAddress`は、コンセンサスに参加しているバリデーターノードを識別します。 バリデーターノードは、** `ed25519` **曲線を使用して導出されます。

これらのタイプは、アドレスインターフェイスを実装します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/address.go#L71-L90

アドレス構築アルゴリズムは[ADR-28](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-028-public-key-addresses.md)で定義されています。
以下は、[pub]公開鍵からアカウントアドレスを取得する標準的な方法です。 

```go
sdk.AccAddress(pub.Address().Bytes())
```

`Marshal()`メソッドと `Bytes()`メソッドはどちらも、同じ元の `[] byte`形式のアドレスを返すことに注意してください。 `Marshal()`はProtobufの互換性のために必要です。

ユーザーとの対話のために、アドレスは[Bech32](https://en.bitcoin.it/wiki/Bech32)を使用してフォーマットされ、 `String`メソッドによって実装されます。 Bech32メソッドは、ブロックチェーンと対話するときにサポートされる唯一の形式です。 Bech32の人間が読める部分(Bech32プレフィックス)は、アドレスタイプを示すために使用されます。 例: 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/address.go#L230-L244

|                    | Address Bech32 Prefix |
| ------------------ | --------------------- |
| Accounts           | cosmos                |
| Validator Operator | cosmosvaloper         |
| Consensus Nodes    | cosmosvalcons         |

### 公開鍵

Cosmos SDKの公開鍵は、 `cryptotypes.PubKey`インターフェースによって定義されます。公開鍵はストレージに保存されるため、 `cryptotypes.PubKey`は` proto.Message`インターフェースを拡張します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/types/types.go#L8-L17

圧縮形式は、[secp256k1]および[secp256r1]のシリアル化に使用されます。

-`y`座標が `x`座標に関連する2バイトの辞書式順序で最大の場合、最初のバイトは0x02バイトです。
-それ以外の場合、最初のバイトは[0x03]です。

この接頭辞の後に[x]座標が続きます。

公開鍵はアカウント(またはユーザー)の参照には使用されず、通常、トランザクションメッセージの作成には使用されません(いくつかの例外: `MsgCreateValidator`、` Validator`、および `Multisig`メッセージを除く)。
ユーザーとの対話には、 `PubKey`はProtobufsJSONを使用します([ProtoMarshalJSON](https://github.com/cosmos/cosmos-sdk/blob/release/v0.42.x/codec/json.go#L12)関数形式) 。例:

+++ https://github.com/cosmos/cosmos-sdk/blob/7568b66/crypto/keyring/output.go#L23-L39

## キーホルダー

`Keyring`は、アカウントを保存および管理するためのオブジェクトです。 Cosmos SDKでは、Keyringの実装はKeyringインターフェースに従います。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keyring/keyring.go#L51-L89

`Keyring`のデフォルトの実装は、サードパーティの[` 99designs/keyring`](https://github.com/99designs/keyring)ライブラリから取得されます。

[キーリング]方式に関する注意事項:

-`Sign(uid string、payload[] byte)([] byte、sdkcrypto.PubKey、error) `は、` payload`バイトの署名を厳密に処理します。トランザクションを準備し、それを正規の `[] byte`形式にエンコードする必要があります。 protobufは決定論的ではないため、[ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md)で、署名される仕様 `payload`は` SignDoc`構造であると判断されています。 deterministic[ADR-027](adr-027-deterministic-protobuf-serialization.md)エンコーディングを使用します。署名の検証はCosmosSDKにデフォルトで実装されておらず、[`anteHandler`](../core/baseapp.md#antehandler)に延期されていることに注意してください。
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/proto/cosmos/tx/v1beta1/tx.proto#L47-L64

-`NewAccount(uid、mnemonic、bip39Passwd、hdPath string、algo SignatureAlgo)(Info、error) `[` bip44path`](https://github.com/bitcoin/bips/blob/に基づいて新しいアカウントマスターを作成します)/bip-0044.mediawiki)そしてそれをディスクに保存します。 `PrivKey` **暗号化されていない状態で保存することはありません**が、[パスワードで暗号化](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/crypto/armor.go)以前に主張されていました。この方法のコンテキストでは、キータイプとシリアル番号は、BIP44派生パスで秘密キーとニーモニックを派生させるために使用される公開キーを参照します。同じニーモニックおよび派生パスを使用して、同じ `PrivKey`、` PubKey`、および `Address`を生成します。キーリングは次のキーをサポートします。

-`secp256k1`
-`ed25519`

-`ExportPrivKeyArmor(uid、encryptPassphrase string)(armor string、err error) `指定されたパスワードを使用して、ASCII装甲暗号化形式で秘密鍵をエクスポートします。次に、 `ImportPrivKey(uid、Armor、passphrase string)`関数を使用して秘密鍵をキーリングに再度インポートするか、 `UnarmorDecryptPrivKey(armorStr string、passphrase string)`関数を使用して元の秘密鍵に復号化します。 。

## 次へ{hide}

[ガスと料金](./gas-fees.md)を理解する{hide} 