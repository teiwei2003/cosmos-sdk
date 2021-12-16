# ADR 036:任意消息签名规范

## 变更日志

- 28/10/2020 - 初稿

##作者

- Antoine Herzog (@antoineherzog)
- Zaki Manian (@zmanian)
- 亚历山大·贝佐布丘克 (alexanderbez) [1]
- Frojdi Dymylja (@fdymylja)

## 地位

草稿

## 摘要

目前，在 Cosmos SDK 中，没有像以太坊那样签署任意消息的约定。我们在此规范中为 Cosmos SDK 生态系统提出了一种对链下任意消息进行签名和验证的方法。

该规范旨在涵盖每个用例，这意味着 cosmos-sdk 应用程序开发人员决定如何序列化并向用户表示“数据”。

## 语境

事实证明，能够在链外签署消息已被证明是几乎所有区块链的基本方面。在链外签署消息的概念有许多额外的好处，例如节省计算成本和减少交易吞吐量和开销。在 Cosmos 的背景下，签署此类数据的一些主要应用包括但不限于提供加密安全和可验证的方式来证明验证者身份，并可能将其与其他框架或组织相关联。此外，能够使用 Ledger 或类似的 HSM 设备对 Cosmos 消息进行签名。

更多上下文和用例可以在参考链接中找到。

## 决定

目标是能够签署任意消息，甚至使用 Ledger 或类似的 HSM 设备。

因此，签名消息应该大致类似于 Cosmos SDK 消息，但**不能**是有效的链上交易。 `chain-id`、`account_number` 和 `sequence` 都可以分配无效值。

Cosmos SDK 0.40 还引入了“auth_info”的概念，可以指定SIGN_MODES。

规范应该包括一个支持 SIGN_MODE_DIRECT 和 SIGN_MODE_LEGACY_AMINO 的 `auth_info`。

创建 `offchain` proto 定义，我们使用 `offchain` 包扩展 auth 模块以提供验证和签署离线消息的功能。

链下交易遵循以下规则:

- 备忘录必须是空的
- nonce，序号必须等于0
- 链 ID 必须等于“”
- 费用gas必须等于0
- 费用金额必须是一个空数组

除了上面强调的规范差异之外，链下交易的验证遵循与链上交易相同的规则。

添加到 `offchain` 包的第一条消息是 `MsgSignData`。

`MsgSignData` 允许开发人员仅对有效的链外字节进行签名。其中“Signer”是签名者的账户地址。 `Data` 是可以表示 `text`、`files`、`object`s 的任意字节。应用程序开发人员决定“数据”应该如何反序列化、序列化以及它可以在其上下文中表示的对象。

应用程序开发人员决定如何处理“数据”，我们所处理的意思是序列化和反序列化过程以及“数据”应该代表的对象。

原型定义: 

```proto
// MsgSignData defines an arbitrary, general-purpose, off-chain message
message MsgSignData {
    // Signer is the sdk.AccAddress of the message signer
    bytes Signer = 1 [(gogoproto.jsontag) = "signer", (gogoproto.casttype) = "github.com/cosmos/cosmos-sdk/types.AccAddress"];
    // Data represents the raw bytes of the content that is signed (text, json, etc)
    bytes Data = 2 [(gogoproto.jsontag) = "data"];
}
```

签名 MsgSignData json 示例: 

```json
{
  "type": "cosmos-sdk/StdTx",
  "value": {
    "msg": [
      {
        "type": "sign/MsgSignData",
        "value": {
          "signer": "cosmos1hftz5ugqmpg9243xeegsqqav62f8hnywsjr4xr",
          "data": "cmFuZG9t"
        }
      }
    ],
    "fee": {
      "amount": [],
      "gas": "0"
    },
    "signatures": [
      {
        "pub_key": {
          "type": "tendermint/PubKeySecp256k1",
          "value": "AqnDSiRoFmTPfq97xxEb2VkQ/Hm28cPsqsZm9jEVsYK9"
        },
        "signature": "8y8i34qJakkjse9pOD2De+dnlc4KvFgh0wQpes4eydN66D9kv7cmCEouRrkka9tlW9cAkIL52ErB+6ye7X5aEg=="
      }
    ],
    "memo": ""
  }
}
```

## Consequences

有一个关于如何形成不打算广播到实时链的消息的规范。 

### Backwards Compatibility

由于这是一个新的消息规范定义，因此保持向后兼容性。 

### Positive

- 一种通用格式，可被多个应用程序用于签署和验证链外消息。
- 规范是原始的，这意味着它可以涵盖每个用例，而不会限制可能适合其中的内容。
- 它为其他链下消息规范提供了空间，这些规范旨在针对更具体和常见的用例，例如基于链下的 authN/authZ 层 [2]。 
### Negative

- 当前的提案需要账户地址和公钥之间的固定关系。
- 不适用于多重签名帐户。 

## 进一步讨论

- 关于 `MsgSignData` 中的安全性，使用 `MsgSignData` 的开发人员负责使放置在 `Data` 中的内容在需要时不可重播。
- 链下包将通过针对特定用例的额外消息进一步扩展，例如但不限于应用程序中的身份验证、支付渠道、一般的 L2 解决方案。 

## References

1. https://github.com/cosmos/ics/pull/33
2. https://github.com/cosmos/cosmos-sdk/pull/7727#discussion_r515668204
3. https://github.com/cosmos/cosmos-sdk/pull/7727#issuecomment-722478477
4. https://github.com/cosmos/cosmos-sdk/pull/7727#issuecomment-721062923
