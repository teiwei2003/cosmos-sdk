# ADR 020:协议缓冲区事务编码

## 变更日志

- 2020 年 3 月 6 日:初稿
- 2020 年 3 月 12 日:API 更新
- 2020 年 4 月 13 日:添加了有关“oneof”接口处理的详细信息
- 2020 年 4 月 30 日:切换到“任何”
- 2020 年 5 月 14 日:描述公钥编码
- 2020 年 6 月 8 日:将“TxBody”和“AuthInfo”作为字节存储在“SignDoc”中；文档“TxRaw”作为广播和存储类型。
- 2020 年 8 月 7 日:使用 ADR 027 序列化 `SignDoc`。
- 2020 年 8 月 19 日:将序列字段从 `SignDoc` 移动到 `SignerInfo`，如 [#6966](https://github.com/cosmos/cosmos-sdk/issues/6966) 中所述。
- 2020 年 9 月 25 日:删除“PublicKey”类型，取而代之的是“secp256k1.PubKey”、“ed25519.PubKey”和“multisig.LegacyAminoPubKey”。
- 2020 年 10 月 15 日:向“AccountRetriever”接口添加“GetAccount”和“GetAccountWithHeight”方法。
- 2021 年 2 月 24 日:Cosmos SDK 不再使用 Tendermint 的 `PubKey` 接口，而是使用它自己的 `cryptotypes.PubKey`。更新以反映这一点。
- 2021 年 5 月 3 日:将 `clientCtx.JSONMarshaler` 重命名为 `clientCtx.JSONCodec`。
- 2021 年 6 月 10 日:添加`clientCtx.Codec: codec.Codec`。

## 地位

公认

##  上下文

该 ADR 是在
[ADR 019](./adr-019-protobuf-state-encoding.md)，即我们的目标是设计
Cosmos SDK 客户端的协议缓冲区迁移路径。

具体来说，客户端迁移路径主要包括 tx 生成和
签名、消息构建和路由，以及 CLI 和 REST 处理程序和
业务逻辑(即查询器)。

考虑到这一点，我们将通过两个主要领域来处理迁移路径，txs 和
查询。但是，此 ADR 仅关注交易。查询应该是
在未来的 ADR 中解决，但它应该建立在这些建议的基础上。

基于详细讨论 ([\#6030](https://github.com/cosmos/cosmos-sdk/issues/6030)
和 [\#6078](https://github.com/cosmos/cosmos-sdk/issues/6078))，原始
交易的设计从`oneof`/JSON-signing 发生了很大变化
下面描述的方法的方法。

## 决定

### 交易

由于接口值在 state 中使用 `google.protobuf.Any` 进行编码(参见 [ADR 019](adr-019-protobuf-state-encoding.md))，
`sdk.Msg`s 在交易中使用 `Any` 进行编码。

使用 `Any` 来编码接口值的主要目标之一是有一个
由应用程序重用的核心类型集，以便
客户端可以安全地与尽可能多的链兼容。

提供灵活的跨链交易是本规范的目标之一
可以在不破坏客户端的情况下为各种用例提供​​服务的格式
兼容性。

为了方便签名，交易被分成`TxBody`，
下面的`SignDoc` 和`signatures` 将重新使用它:


```proto
//types/types.proto
package cosmos_sdk.v1;

message Tx {
    TxBody body = 1;
    AuthInfo auth_info = 2;
   //A list of signatures that matches the length and order of AuthInfo's signer_infos to
   //allow connecting signature meta information like public key and signing mode by position.
    repeated bytes signatures = 3;
}

//A variant of Tx that pins the signer's exact binary represenation of body and
//auth_info. This is used for signing, broadcasting and verification. The binary
//`serialize(tx: TxRaw)` is stored in Tendermint and the hash `sha256(serialize(tx: TxRaw))`
//becomes the "txhash", commonly used as the transaction ID.
message TxRaw {
   //A protobuf serialization of a TxBody that matches the representation in SignDoc.
    bytes body = 1;
   //A protobuf serialization of an AuthInfo that matches the representation in SignDoc.
    bytes auth_info = 2;
   //A list of signatures that matches the length and order of AuthInfo's signer_infos to
   //allow connecting signature meta information like public key and signing mode by position.
    repeated bytes signatures = 3;
}

message TxBody {
   //A list of messages to be executed. The required signers of those messages define
   //the number and order of elements in AuthInfo's signer_infos and Tx's signatures.
   //Each required signer address is added to the list only the first time it occurs.
   //
   //By convention, the first required signer (usually from the first message) is referred
   //to as the primary signer and pays the fee for the whole transaction.
    repeated google.protobuf.Any messages = 1;
    string memo = 2;
    int64 timeout_height = 3;
    repeated google.protobuf.Any extension_options = 1023;
}

message AuthInfo {
   //This list defines the signing modes for the required signers. The number
   //and order of elements must match the required signers from TxBody's messages.
   //The first element is the primary signer and the one which pays the fee.
    repeated SignerInfo signer_infos = 1;
   //The fee can be calculated based on the cost of evaluating the body and doing signature verification of the signers. This can be estimated via simulation.
    Fee fee = 2;
}

message SignerInfo {
   //The public key is optional for accounts that already exist in state. If unset, the
   //verifier can use the required signer address for this position and lookup the public key.
    google.protobuf.Any public_key = 1;
   //ModeInfo describes the signing mode of the signer and is a nested
   //structure to support nested multisig pubkey's
    ModeInfo mode_info = 2;
   //sequence is the sequence of the account, which describes the
   //number of committed transactions signed by a given address. It is used to prevent
   //replay attacks.
    uint64 sequence = 3;
}

message ModeInfo {
    oneof sum {
        Single single = 1;
        Multi multi = 2;
    }

   //Single is the mode info for a single signer. It is structured as a message
   //to allow for additional fields such as locale for SIGN_MODE_TEXTUAL in the future
    message Single {
        SignMode mode = 1;
    }

   //Multi is the mode info for a multisig public key
    message Multi {
       //bitarray specifies which keys within the multisig are signing
        CompactBitArray bitarray = 1;
       //mode_infos is the corresponding modes of the signers of the multisig
       //which could include nested multisig public keys
        repeated ModeInfo mode_infos = 2;
    }
}

enum SignMode {
    SIGN_MODE_UNSPECIFIED = 0;

    SIGN_MODE_DIRECT = 1;

    SIGN_MODE_TEXTUAL = 2;

    SIGN_MODE_LEGACY_AMINO_JSON = 127;
}
```

正如下面将要讨论的，为了包含尽可能多的“Tx”
在 `SignDoc` 中，`SignerInfo` 与签名分开，因此只有
原始签名本身存在于已签署的内容之外。

因为我们的目标是灵活、可扩展的跨链交易
格式，新的事务处理选项应尽快添加到`TxBody`
这些用例被发现了，即使它们还不能实现。

因为这里有协调开销，`TxBody` 包括一个
`extension_options` 字段，可用于任何事务处理
尚未涵盖的选项。尽管如此，应用程序开发人员应该
尝试上游对“Tx”的重要改进。

### 签名

以下所有签名模式旨在提供以下保证:

- **无延展性**:一旦交易，`TxBody` 和`AuthInfo` 不能改变
  已签署
- **可预测的GAS费**:如果我正在签署一项我支付费用的交易，
  最后的gas完全取决于我签署的内容

这些保证为消息签名者提供了最大的信心
中间人对“Tx”的操纵不会导致任何有意义的变化。

#### `SIGN_MODE_DIRECT`

“直接”签名行为是将原始“TxBody”字节签名为广播
电线。这具有以下优点:

- 需要超出标准协议的最少附加客户端功能
  缓冲区实现
- 为交易延展性留下有效的零漏洞(即没有
  签名和编码格式之间的细微差别可能
  可能被攻击者利用)

签名使用下面的`SignDoc` 结构化，它重用了
`TxBody` 和 `AuthInfo`，只添加签名所需的字段: 

```proto
//types/types.proto
message SignDoc {
   //A protobuf serialization of a TxBody that matches the representation in TxRaw.
    bytes body = 1;
   //A protobuf serialization of an AuthInfo that matches the representation in TxRaw.
    bytes auth_info = 2;
    string chain_id = 3;
    uint64 account_number = 4;
}
```

为了以默认模式登录，客户端执行以下步骤:

1. 使用任何有效的 protobuf 实现序列化 `TxBody` 和 `AuthInfo`。
2. 创建一个 `SignDoc` 并使用 [ADR 027](./adr-027-deterministic-protobuf-serialization.md) 对其进行序列化。
3. 对编码的`SignDoc` 字节进行签名。
4. 构建一个 `TxRaw` 并将其序列化以进行广播。

签名验证基于比较原始的“TxBody”和“AuthInfo”
`TxRaw` 中编码的字节不基于任何 ["规范化"](https://github.com/regen-network/canonical-proto3)
除了防止
某些形式的可升级性(将在本文档后面讨论)。

签名验证者做:

1. 反序列化一个 `TxRaw` 并拉出 `body` 和 `auth_info`。
2. 根据消息创建所需签名者地址的列表。
3. 对于每个需要的签名者:
   - 从状态中提取帐号和序列。
   - 从 state 或 `AuthInfo` 的 `signer_infos` 获取公钥。
   - 创建一个 `SignDoc` 并使用 [ADR 027](./adr-027-deterministic-protobuf-serialization.md) 对其进行序列化。
   - 针对序列化的“SignDoc”验证相同列表位置的签名。

#### `SIGN_MODE_LEGACY_AMINO`

为了支持旧钱包和交易所，Amino JSON 将暂时
支持交易签名。一旦钱包和交易所有了
有机会升级到基于 protobuf 的签名，此选项将被禁用。在
同时，可以预见禁用当前的 Amino 签名将导致
太多的破损是不可行的。请注意，这主要是要求
Cosmos Hub 和其他链可能会选择立即禁用 Amino 签名。

旧客户将能够使用当前的 Amino 签署交易
JSON 格式并使用 REST `/tx/encode` 将其编码为 protobuf
广播前的端点。

#### `SIGN_MODE_TEXTUAL`

正如 [\#6078](https://github.com/cosmos/cosmos-sdk/issues/6078) 中广泛讨论的那样，
需要人类可读的签名编码，尤其是硬件
像 [Ledger](https://www.ledger.com) 这样显示的钱包
交易内容在签署前告知用户。 JSON 是一个尝试，但是
达不到理想。

`SIGN_MODE_TEXTUAL` 旨在作为人类可读的占位符
将替换氨基 JSON 的编码。这种新的编码应该更
比 JSON 更注重可读性，可能基于格式化字符串，例如
[消息格式](http://userguide.icu-project.org/formatparse/messages)。

为了确保新的人类可读格式不会受到
交易延展性问题，`SIGN_MODE_TEXTUAL`
要求将 _human-readable 字节与原始 `SignDoc`_ 连接起来
生成符号字节。

可能支持多种人类可读的格式(甚至可能是本地化的消息)
在实现时通过`SIGN_MODE_TEXTUAL`。

### 未知字段过滤

protobuf 消息中的未知字段通常应该被事务拒绝
处理器，因为:

- 重要数据可能存在于未知字段中，如果忽略，将
  导致客户出现意外行为
- 他们提出了一个延展性漏洞，攻击者可以在其中膨胀 tx 大小
  通过将随机未解释的数据添加到未签名的内容(即主“Tx”，
  不是`TxBody`)

还有一些场景我们可以选择安全地忽略未知字段
(https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-624400188)到
提供与较新客户端的优雅向前兼容性。

我们建议设置第 11 位的字段编号(对于大多数用例，这是
1024-2047 的范围)被认为是可以安全的非关键领域
如果未知，则忽略。

为了解决这个问题，我们需要一个未知字段过滤器:

- 始终拒绝未签名内容中的未知字段(即顶级`Tx` 和
  `AuthInfo` 的未签名部分(如果基于签名模式存在)
- 拒绝所有消息(包括嵌套的`Any`s)中的未知字段，除了
  设置了第 11 位的字段

这可能需要一个自定义的 protobuf 解析器传递，它接受消息字节
和`FileDescriptor`s ​​并返回一个布尔结果。

### 公钥编码

Cosmos SDK 中的公钥实现了 `cryptotypes.PubKey` 接口。
我们建议使用 `Any` 进行 protobuf 编码，就像我们使用其他接口一样(例如，在 `BaseAccount.PubKey` 和 `SignerInfo.PublicKey` 中)。
实现了以下公钥:secp256k1、secp256r1、ed25519 和 legacy-multisignature。

Ex: 

```proto
message PubKey {
    bytes key = 1;
}
```

`multisig.LegacyAminoPubKey` 有一个 `Any` 的成员数组来支持任何
protobuf 公钥类型。

应用程序应该只尝试处理他们注册的一组公钥
已经测试。 提供的签名验证 ante 处理程序装饰器将
强制执行此操作。

### CLI 和 REST

目前，REST 和 CLI 处理程序通过 Amino 对类型和 txs 进行编码和解码
使用具体的 Amino 编解码器的 JSON 编码。 由于某些类型处理
在客户端可以是接口，类似于我们在 [ADR 019](./adr-019-protobuf-state-encoding.md) 中描述的方式，
客户端逻辑现在需要采用一个编解码器接口，该接口不仅知道如何
处理所有类型，但也知道如何生成交易、签名、
和消息。 

```go
type AccountRetriever interface {
  GetAccount(clientCtx Context, addr sdk.AccAddress) (client.Account, error)
  GetAccountWithHeight(clientCtx Context, addr sdk.AccAddress) (client.Account, int64, error)
  EnsureExists(clientCtx client.Context, addr sdk.AccAddress) error
  GetAccountNumberSequence(clientCtx client.Context, addr sdk.AccAddress) (uint64, uint64, error)
}

type Generator interface {
  NewTx() TxBuilder
  NewFee() ClientFee
  NewSignature() ClientSignature
  MarshalTx(tx types.Tx) ([]byte, error)
}

type TxBuilder interface {
  GetTx() sdk.Tx

  SetMsgs(...sdk.Msg) error
  GetSignatures() []sdk.Signature
  SetSignatures(...sdk.Signature)
  GetFee() sdk.Fee
  SetFee(sdk.Fee)
  GetMemo() string
  SetMemo(string)
}
```

然后我们更新 `Context` 以获得新字段:`Codec`、`TxGenerator`、
和`AccountRetriever`，我们更新`AppModuleBasic.GetTxCmd`
一个“上下文”，它应该预先填充所有这些字段。

然后每个客户端方法应该使用“Init”方法之一来重新初始化
预填充的“上下文”。 `tx.GenerateOrBroadcastTx` 可用于
生成或广播交易。 例如: 

```go
import "github.com/spf13/cobra"
import "github.com/cosmos/cosmos-sdk/client"
import "github.com/cosmos/cosmos-sdk/client/tx"

func NewCmdDoSomething(clientCtx client.Context) *cobra.Command {
	return &cobra.Command{
		RunE: func(cmd *cobra.Command, args []string) error {
			clientCtx := ctx.InitWithInput(cmd.InOrStdin())
			msg := NewSomeMsg{...}
			tx.GenerateOrBroadcastTx(clientCtx, msg)
		},
	}
}
```

## Future Improvements

### `SIGN_MODE_TEXTUAL` specification

“SIGN_MODE_TEXTUAL”的具体规范和实现旨在
作为近期未来的改进，以便分类帐应用程序和其他钱包
可以优雅地从 Amino JSON 过渡。

### `SIGN_MODE_DIRECT_AUX`

(\*在 https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-628026933 中记录为选项(3))

我们可以添加一个模式`SIGN_MODE_DIRECT_AUX`
支持多个签名的场景
正在被收集到单个事务中，但消息编辑器没有
还知道哪些签名将包含在最终交易中。例如，
我可能有一个 3/5 的多重签名钱包，并想向所有 5 个人发送一个“TxBody”
签名者看谁先签名。一有3个签名我就去
提前并建立完整的交易。

使用`SIGN_MODE_DIRECT`，每个签名者需要
签署完整的“AuthInfo”，其中包括所有签名者的完整列表和
他们的签名模式，使上述场景变得非常困难。

`SIGN_MODE_DIRECT_AUX` 将允许“辅助”签名者创建他们的签名
只使用 `TxBody` 和他们自己的 `PublicKey`。这允许完整的列表
AuthInfo 中的签名者将被延迟，直到收集到签名。

“辅助”签名人是除主要签名人之外的任何签名人
费用。对于主要签名者，实际上需要完整的 AuthInfo 来计算 gas 和费用
因为这取决于有多少签名者以及哪些密钥类型和签名
他们正在使用的模式。但是，辅助签名者无需担心
费用或汽油，因此可以只签署“TxBody”。

要在“SIGN_MODE_DIRECT_AUX”中生成签名，将遵循以下步骤:

1. 编码`SignDocAux`(与字段必须序列化的要求相同
   为了): 

```proto
//types/types.proto
message SignDocAux {
    bytes body_bytes = 1;
   //PublicKey is included in SignDocAux :
   //1. as a special case for multisig public keys. For multisig public keys,
   //the signer should use the top-level multisig public key they are signing
   //against, not their own public key. This is to prevent against a form
   //of malleability where a signature could be taken out of context of the
   //multisig key that was intended to be signed for
   //2. to guard against scenario where configuration information is encoded
   //in public keys (it has been proposed) such that two keys can generate
   //the same signature but have different security properties
   //
   //By including it here, the composer of AuthInfo cannot reference the
   //a public key variant the signer did not intend to use
    PublicKey public_key = 2;
    string chain_id = 3;
    uint64 account_number = 4;
}
```

2.对编码后的`SignDocAux`字节进行签名
3. 将他们的签名和 `SignerInfo` 发送给主要签名者，然后他们将
    签署并广播最终交易(使用“SIGN_MODE_DIRECT”和“AuthInfo”
    添加)一旦收集到足够的签名 
### `SIGN_MODE_DIRECT_RELAXED`

(_Documented as option (1)(a) in https://github.com/cosmos/cosmos-sdk/issues/6078#issuecomment-628026933_)

这是“SIGN_MODE_DIRECT”的变体，其中多个签名者不需要
提前协调公钥和签名模式。 这将涉及一个替代
`SignDoc` 类似于上面的 `SignDocAux`，收费。 这可以在未来添加
如果客户端开发者发现提前收集公钥和模式的负担
太累赘了。 

## Consequences

### Positive

- 显着的性能提升。
- 支持向后和向前类型兼容性。
- 更好地支持跨语言客户端。
- 多种签名模式允许更大的协议演变 

### Negative

- `google.protobuf.Any` 类型的 URL 会增加交易规模
   可能可以忽略不计，或者压缩可以减轻它。 

### Neutral

## References
