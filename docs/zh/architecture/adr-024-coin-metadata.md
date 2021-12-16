# ADR 024:硬币元数据

## 变更日志

- 05/19/2020:初稿

## 地位

建议的

## 语境

Cosmos SDK 中的资产通过“Coins”类型表示，该类型由“amount”和“denom”组成，
其中“金额”可以是任意大或小值。此外，Cosmos SDK 使用
基于账户的模型有两种类型的主要账户——基本账户和模块账户。
所有帐户类型都有一组由“硬币”组成的余额。 `x/bank` 模块保持
跟踪所有账户的所有余额，并跟踪一个账户中的余额总供应量
应用。

关于余额`amount`，Cosmos SDK 假设一个静态和固定的面额单位，
不管面额本身。换句话说，客户端和应用程序构建在基于 Cosmos-SDK 的
链可以选择定义和使用任意的面额单位来提供更丰富的用户体验，但是，通过
当 tx 或操作到达 Cosmos SDK 状态机时，`amount` 被视为单个
单元。例如，对于 Cosmos Hub (Gaia)，客户端假设 1 ATOM = 10^6 uatom，因此所有 txs 和
Cosmos SDK 中的操作以 10^6 为单位工作。

这显然提供了一个糟糕且有限的用户体验，尤其是随着网络互操作性的增加和
因此，资产类型的总量增加了。我们建议额外保留`x/bank`
跟踪每个“denom”的元数据，以帮助客户、钱包提供商和探索者改进他们的
UX 并删除对面额单位进行任何假设的要求。

## 决定

`x/bank` 模块将更新为通过 `denom` 存储和索引元数据，特别是“base”或
最小单位——Cosmos SDK 状态机使用的单位。

元数据还可以包括非零长度的面额列表。每个条目包含的名称
面额`denom`，基数的指数和别名列表。一个条目是
解释为“1 denom = 10^exponent base_denom”(例如，“1 ETH = 10^18 wei”和“1 uatom = 10^0 uatom”)。

有两种面额对客户来说非常重要:“基数”，它是最小的
可能的单位和“显示”，这是人类交流中通常提到的单位
和交流。这些字段中的值链接到面额列表中的条目。

`denom_units` 中的列表和 `display` 条目可以通过管理进行更改。

因此，我们可以定义类型如下: 

```protobuf
message DenomUnit {
  string denom    = 1;
  uint32 exponent = 2;  
  repeated string aliases = 3;
}

message Metadata {
  string description = 1;
  repeated DenomUnit denom_units = 2;
  string base = 3;
  string display = 4;
}
```

例如，ATOM 的元数据可以定义如下: 

```json
{
  "description": "The native staking token of the Cosmos Hub.",
  "denom_units": [
    {
      "denom": "uatom",
      "exponent": 0,
      "aliases": [
        "microatom"
      ],
    },
    {
      "denom": "matom",
      "exponent": 3,
      "aliases": [
        "milliatom"
      ]
    },
    {
      "denom": "atom",
      "exponent": 6,
    }
  ],
  "base": "uatom",
  "display": "atom",
}
```

鉴于上述元数据，客户端可以推断出以下内容:

- 4.3atom = 4.3 * (10^6) = 4,300,000uatom
- 字符串“atom”可用作标记列表中的显示名称。
- 天平 4300000 可显示为 4,300,000uatom 或 4,300matom 或 4.3atom。
   如果客户端的作者没有制作，`display` 面额 4.3atom 是一个很好的默认值
   选择不同表示的明确决定。

客户端应该能够通过 CLI 和 REST 接口按名称查询元数据。 在
此外，我们将向这些接口添加处理程序，以将任何单元转换为另一个给定单元，
因为这个基础框架已经存在于 Cosmos SDK 中。

最后，我们需要确保元数据存在于 `x/bank` 模块的 `GenesisState` 中，该模块也是
由基础`denom`索引。 

```go
type GenesisState struct {
  SendEnabled   bool        `json:"send_enabled" yaml:"send_enabled"`
  Balances      []Balance   `json:"balances" yaml:"balances"`
  Supply        sdk.Coins   `json:"supply" yaml:"supply"`
  DenomMetadata []Metadata  `json:"denom_metadata" yaml:"denom_metadata"`
}
```

## 未来的工作

为了让客户避免必须将资产转换为基本面额 - 手动或
通过端点，我们可以考虑支持给定单位输入的自动转换。 
## Consequences

### Positive

- 为客户、钱包提供商和区块浏览器提供额外的数据
   资产面额以改善用户体验并消除任何假设的需要
   面额单位。 

### Negative

- `x/bank` 模块中需要的少量额外存储空间。 数量
   额外存储的数量应该最小，因为总资产的数量不应该
   大。 

### Neutral

## References
