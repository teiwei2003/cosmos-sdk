# 帐户

本文档介绍了 Cosmos SDK 的内置账号和公钥系统。 {概要}

### 先决条件阅读

- [Cosmos SDK 应用剖析](./app-anatomy.md) {prereq}

## 帐户定义

在 Cosmos SDK 中，一个 _account_ 指定一对 _public key_ `PubKey` 和 _private key_ `PrivKey`。可以派生出“公钥”以生成各种“地址”，用于识别应用程序中的用户(以及其他方)。 `Addresses` 还与 [`message`s](../building-modules/messages-and-queries.md#messages) 相关联，以识别 `message` 的发送者。 `PrivKey` 用于生成[数字签名](#signatures) 以证明与给定的“消息”的“PrivKey”关联的“地址”。

对于 HD 密钥派生，Cosmos SDK 使用称为 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 的标准。 BIP32 允许用户创建一个 HD 钱包(如 [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 中指定的)——一组源自初始秘密的账户种子。种子通常由 12 或 24 字的助记符创建。单个种子可以使用单向加密函数派生任意数量的“PrivKey”。然后，可以从 `PrivKey` 派生出一个 `PubKey`。自然地，助记符是最敏感的信息，因为如果保留助记符，私钥总是可以重新生成的。 
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

在 Cosmos SDK 中，密钥是通过一个名为 [`Keyring`](#keyring) 的对象来存储和管理的。

## 密钥、帐户、地址和签名

对用户进行身份验证的主要方式是使用 [数字签名](https://en.wikipedia.org/wiki/Digital_signature)。用户使用自己的私钥签署交易。签名验证是使用关联的公钥完成的。出于链上签名验证的目的，我们将公钥存储在“Account”对象中(以及正确交易验证所需的其他数据)。

在节点中，所有数据都使用 Protocol Buffers 序列化存储。

Cosmos SDK 支持以下用于创建数字签名的数字密钥方案:

- `secp256k1`，在 [Cosmos SDK 的 `crypto/keys/secp256k1` 包中实现](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keys/secp256k1/secp256k1。去)。
- `secp256r1`，在 [Cosmos SDK 的 `crypto/keys/secp256r1` 包中实现](https://github.com/cosmos/cosmos-sdk/blob/master/crypto/keys/secp256r1/pubkey.go) ,
- `tm-ed25519`，在 [Cosmos SDK `crypto/keys/ed25519` 包中实现](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keys/ed25519/ed25519.go)。该方案仅支持共识验证。 

|              | Address length in bytes | Public key length in bytes | Used for transaction authentication | Used for consensus (tendermint) |
|:------------:|:-----------------------:|:--------------------------:|:-----------------------------------:|:-------------------------------:|
| `secp256k1`  | 20                      |                         33 | yes                                 | no                              |
| `secp256r1`  | 32                      |                         33 | yes                                 | no                              |
| `tm-ed25519` | -- not used --          |                         32 | no                                  | yes                             |

## 地址

`Addresses` 和 `PubKey`s 都是识别应用程序中参与者的公共信息。 `Account` 用于存储认证信息。基本帐户实现由“BaseAccount”对象提供。

每个帐户都使用“地址”标识，地址是从公钥派生的字节序列。在 Cosmos SDK 中，我们定义了 3 种地址类型，用于指定使用帐户的上下文:

- `AccAddress` 标识用户(`message` 的发送者)。
- `ValAddress` 标识验证器操作符。
- `ConsAddress` 标识参与共识的验证器节点。验证器节点是使用 **`ed25519`** 曲线导出的。

这些类型实现了 `Address` 接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/address.go#L71-L90

地址构建算法在[ADR-28](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-028-public-key-addresses.md)中定义。
以下是从“pub”公钥获取帐户地址的标准方法: 

```go
sdk.AccAddress(pub.Address().Bytes())
```

值得注意的是，`Marshal()` 和 `Bytes()` 方法都返回地址的相同原始 `[]byte` 形式。 `Marshal()` 是 Protobuf 兼容性所必需的。

对于用户交互，地址使用 [Bech32](https://en.bitcoin.it/wiki/Bech32) 格式化并通过 `String` 方法实现。 Bech32 方法是与区块链交互时唯一支持使用的格式。 Bech32 人类可读部分(Bech32 前缀)用于表示地址类型。 例子: 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/types/address.go#L230-L244

|                    | Address Bech32 Prefix |
| ------------------ | --------------------- |
| Accounts           | cosmos                |
| Validator Operator | cosmosvaloper         |
| Consensus Nodes    | cosmosvalcons         |

### 公钥

Cosmos SDK 中的公钥由`cryptotypes.PubKey` 接口定义。由于公钥保存在存储中，`cryptotypes.PubKey` 扩展了 `proto.Message` 接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/types/types.go#L8-L17

压缩格式用于“secp256k1”和“secp256r1”序列化。

- 第一个字节是一个 0x02 字节，如果 `y` 坐标是与 `x` 坐标相关的两个字节中字典序最大的。
- 否则第一个字节是“0x03”。

此前缀后跟“x”坐标。

公钥不用于引用帐户(或用户)，通常在编写交易消息时不使用(除了少数例外:`MsgCreateValidator`、`Validator` 和 `Multisig` 消息)。
对于用户交互，`PubKey` 使用 Protobufs JSON ([ProtoMarshalJSON](https://github.com/cosmos/cosmos-sdk/blob/release/v0.42.x/codec/json.go#L12) 函数格式化)。例子:

+++ https://github.com/cosmos/cosmos-sdk/blob/7568b66/crypto/keyring/output.go#L23-L39

## 钥匙圈

`Keyring` 是一个存储和管理帐户的对象。在 Cosmos SDK 中，Keyring 实现遵循 Keyring 接口:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/crypto/keyring/keyring.go#L51-L89

`Keyring` 的默认实现来自第三方 [`99designs/keyring`](https://github.com/99designs/keyring) 库。

关于“Keyring”方法的一些注意事项:

- `Sign(uid string, payload []byte) ([]byte, sdkcrypto.PubKey, error)`严格处理`payload`字节的签名。您必须准备交易并将其编码为规范的 `[]byte` 形式。因为 protobuf 不是确定性的，所以在 [ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md) 中已经确定要签名的规范 `payload` 是 `SignDoc` 结构，确定性使用 [ADR-027](adr-027-deterministic-protobuf-serialization.md) 编码。请注意，Cosmos SDK 中默认没有实现签名验证，它推迟到 [`anteHandler`](../core/baseapp.md#antehandler)。
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/proto/cosmos/tx/v1beta1/tx.proto#L47-L64

- `NewAccount(uid, mnemonic, bip39Passwd, hdPath string, algo SignatureAlgo) (Info, error)` 基于 [`bip44 path`](https://github.com/bitcoin/bips/blob/) 创建一个新账户master/bip-0044.mediawiki)并将其保存在磁盘上。 `PrivKey` **永远不会未加密存储**，而是 [使用密码加密](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/crypto/armor.go ) 之前被坚持。在此方法的上下文中，密钥类型和序列号指的是 BIP44 派生路径中用于派生私有和助记符中的公钥。使用相同的助记符和派生路径，生成相同的`PrivKey`、`PubKey`和`Address`。密钥环支持以下密钥:

-`secp256k1`
-`ed25519`

- `ExportPrivKeyArmor(uid, encryptPassphrase string) (armor string, err error)` 使用给定的密码以 ASCII 装甲加密格式导出私钥。然后，您可以使用 `ImportPrivKey(uid, Armor, passphrase string)` 函数将私钥再次导入密钥环，或者使用 `UnarmorDecryptPrivKey(armorStr string, passphrase string)` 函数将其解密为原始私钥。

## 下一个 {hide}

了解 [gas 和费用](./gas-fees.md) {hide} 