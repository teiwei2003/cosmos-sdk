# ADR 028:公钥地址

## 变更日志

- 2020/08/18:初始版本
- 2021/01/15:分析和算法更新

## 地位

建议的

## 摘要

此 ADR 定义了所有可寻址 Cosmos SDK 帐户的地址格式。这包括:新的公钥算法、多重签名公钥和模块帐户。

## 语境

问题 [\#3685](https://github.com/cosmos/cosmos-sdk/issues/3685) 确定了公钥
地址空间目前正在重叠。我们确认它显着降低了 Cosmos SDK 的安全性。

### 问题

攻击者可以控制地址生成功能的输入。这会导致生日攻击，从而显着降低安全空间。
为了克服这个问题，我们需要将不同类型账户类型的输入分开:
一种账户类型的安全中断不应影响其他账户类型的安全。

### 初步建议

一项最初的提议是延长地址长度和
为不同类型的地址添加前缀。

@ethanfrey 解释了最初在 https://github.com/iov-one/weave 中使用的替代方法:

> 我在构建 weave 时花了很多时间思考这个问题……另一个宇宙 Sdk。
> 基本上我将一个条件定义为一种类型和格式，作为人类可读的字符串，并附加一些二进制数据。这个条件被散列成一个地址(同样是 20 个字节)。使用此前缀使得无法找到具有不同条件的给定地址的原像(例如 ed25519 与 secp256k1)。
> 这里有详细解释 https://weave.readthedocs.io/en/latest/design/permissions.html
> 而代码就在这里，主要看我们处理条件的顶部。 https://github.com/iov-one/weave/blob/master/conditions.go

并解释了这种方法应该如何具有足够的抗碰撞性:

> 是的，AFAIK，当原像唯一且不可延展时，20 字节应该是抗碰撞的。 2 ^ 160 的空间预计在 2 ^ 80 元素附近可能会发生一些碰撞(生日悖论)。如果你想找到数据库中某个现有元素的冲突，它仍然是 2^160。 2^80 仅当所有这些元素都写入状态时。
> 你提出的好例子是例如。公钥字节是编解码器支持的两种算法的有效公钥。这意味着如果其中任何一个被破坏，即使它们被更安全的变体保护，你也会破坏帐户。这只是当原像中不存在区分类型信息时(在散列到地址之前)的问题。
> 如果 20 字节空间是一个实际的安全问题，我想听到一个争论，因为我很乐意在编织中增加我的地址大小。我只是认为 cosmos、以太坊和比特币都使用 20 个字节，应该足够了。而上面的论点让我觉得它是安全的。但我没有做更深入的分析。

这导致了第一个提案(我们证明它不够好):
我们将一个密钥类型与一个公钥连接起来，对它进行散列并取该散列的前 20 个字节，总结为 `sha256(keyTypePrefix || keybytes)[:20]`。

### 回顾和讨论

在 [\#5694](https://github.com/cosmos/cosmos-sdk/issues/5694) 中，我们讨论了各种解决方案。
我们一致认为 20 字节不是未来证明，并且扩展地址长度是允许不同类型、各种签名类型等的地址的唯一方法。
这使最初的提案失去资格。

在这个问题中，我们讨论了各种修改:

+ 散列函数的选择。
+ 将前缀移出哈希函数:`keyTypePrefix + sha256(keybytes)[:20]` [post-hash-prefix-proposal]。
+ 使用双重散列:`sha256(keyTypePrefix + sha256(keybytes)[:20])`。
+ 将 keybytes 哈希切片从 20 字节增加到 32 或 40 字节。我们得出的结论是，由良好的散列函数产生的 32 字节是未来安全的。 

### Requirements

+ 支持当前使用的工具 - 我们不想破坏生态系统，或增加较长的适应期。参考:https://github.com/cosmos/cosmos-sdk/issues/8041
+ 尽量保持地址长度小——地址在状态中被广泛使用，作为键和对象值的一部分。

### 范围

该 ADR 仅定义了一个生成地址字节的过程。对于最终用户与地址的交互(通过 API 或 CLI 等)，我们仍然使用 bech32 将这些地址格式化为字符串。该 ADR 不会改变这一点。
使用 Bech32 进行字符串编码为我们提供了对校验和错误代码和用户拼写错误处理的支持。

## 决定

我们定义了以下账户类型，我们为其定义了地址函数:

1. 简单账户:由一个普通的公钥表示(即:secp256k1，sr25519)
2. naive multisig:由其他可寻址对象组成的帐户(即:naive multisig)
3. 使用本机地址键组合帐户(即:bls，组模块帐户)
4. 模块账户:基本上是任何不能签署交易且由模块内部管理的账户

### 遗留公钥地址不会改变

目前(2021 年 1 月)，唯一官方支持的 Cosmos SDK 用户帐户是“secp256k1”基本帐户和传统的氨基多重签名。
它们用于现有的 Cosmos SDK 区域。它们使用以下地址格式:

- secp256k1:`ripemd160(sha256(pk_bytes))[:20]`
- 传统的氨基多重签名:`sha256(aminoCdc.Marshal(pk))[:20]`

我们不想更改现有地址。所以这两种密钥类型的地址将保持不变。

当前的多重签名公钥使用氨基序列化来生成地址。我们将保留
那些公钥及其地址格式，并称它们为“传统氨基”多重签名公钥
在 protobuf 中。我们还将创建不带氨基地址的多重签名公钥，如下所述。 

### 哈希函数选择

与 Cosmos SDK 的其他部分一样，我们将使用 `sha256`。

### 基本地址

我们首先定义用于生成地址的基本哈希算法。 值得注意的是，它用于由单个密钥对表示的帐户。 对于每个公钥模式，我们必须有一个关联的 `typ` 字符串，我们将在下面的部分中讨论。 `hash` 是上一节中定义的加密哈希函数。 

```go
const A_LEN = 32

func Hash(typ string, key []byte) []byte {
    return hash(hash(typ) + key)[:A_LEN]
}
```

`+` 是字节连接，不使用任何分隔符。

该算法是与专业密码学家协商的结果。
动机:该算法保持地址相对较小(`typ` 的长度不影响最终地址的长度)
并且它比 [post-hash-prefix-proposal](使用公钥哈希的前 20 个字节，显着减少地址空间)更安全。
此外，密码学家促使选择在散列中添加“typ”以防止切换表攻击。

我们使用 `address.Hash` 函数为由单个键表示的所有帐户生成地址:

* 简单的公钥:`address.Hash(keyType, pubkey)`

+ 聚合键(例如:BLS):`address.Hash(keyType,aggregatedPubKey)`
+ 模块:`address.Hash("module", moduleName)`

### 组合地址

对于简单的组合账户(如新的朴素多重签名)，我们概括了“address.Hash”。地址是通过为子账户递归创建地址、对地址进行排序并将它们组合成单个地址来构建的。它确保键的顺序不会影响结果地址。 

```go
//We don't need a PubKey interface - we need anything which is addressable.
type Addressable interface {
    Address() []byte
}

func Composed(typ string, subaccounts []Addressable) []byte {
    addresses = map(subaccounts, \a -> LengthPrefix(a.Address()))
    addresses = sort(addresses)
    return address.Hash(typ, addresses[0] + ... + addresses[n])
}
```

`typ` 参数应该是一个模式描述符，包含所有具有确定性序列化的重要属性(例如:utf8 字符串)。
`LengthPrefix` 是一个在地址前添加 1 个字节的函数。 该字节的值是前置之前的地址位的长度。 地址的长度最多为 255 位。
我们使用 `LengthPrefix` 来消除冲突 - 它确保对于 2 个地址列表:`as = {a1, a2, ..., an}` 和 `bs = {b1, b2, ..., bm} ` 这样每个 `bi` 和 `ai` 最多 255 长，`concatenate(map(as, \a -> LengthPrefix(a))) = map(bs, \b -> LengthPrefix(b))` iff `as = bs`。

实现提示:帐户实现应该缓存地址。

#### 多重签名地址

对于新的多重签名公钥，我们定义了不基于任何编码方案(amino 或 protobuf)的 `typ` 参数。 这避免了编码方案中的不确定性问题。

例子: 

```proto
package cosmos.crypto.multisig;

message PubKey {
  uint32 threshold = 1;
  repeated google.protobuf.Any pubkeys = 2;
}
```

```go
func (multisig PubKey) Address() {
 //first gather all nested pub keys
  var keys []address.Addressable //cryptotypes.PubKey implements Addressable
  for _, _key := range multisig.Pubkeys {
    keys = append(keys, key.GetCachedValue().(cryptotypes.PubKey))
  }

 //form the type from the message name (cosmos.crypto.multisig.PubKey) and the threshold joined together
  prefix := fmt.Sprintf("%s/%d", proto.MessageName(multisig), multisig.Threshold)

 //use the Composed function defined above
  return address.Composed(prefix, keys)
}
```

#### 模块帐户地址

注意:本节尚未完成，正在积极讨论中。

在基本地址部分，我们将模块帐户地址定义为: 

```go
address.Hash("module", moduleName)
```

我们使用“模块”作为所有模块派生地址的模式类型。 模块账户可以有子账户。 派生过程有一个定义的顺序:模块名称、子模块键、子子模块键。
模块帐户地址在 Cosmos SDK 中大量使用，因此优化派生过程是有意义的:我们使用空字节 (`'\x00'`) 作为分隔符，而不是使用 `LengthPrefix` 作为模块名称。 这是有效的，因为空字节不是有效模块名称的一部分。 

```go
func Module(moduleName string, key []byte) []byte{
	return Hash("module", []byte(moduleName) + 0 + key)
}
```

**Example**  A lending BTC pool address would be:

```
btcPool := address.Module("lending", btc.Addrress()})
```

如果我们想根据多个键为一个模块帐户创建一个地址，我们可以将它们连接起来:

```
btcAtomAMM := address.Module("amm", btc.Addrress() + atom.Address()})
```

#### 派生地址

我们必须能够通过密码从另一个地址中导出一个地址。 推导过程必须保证哈希属性，因此我们使用已经定义的`Hash`函数: 

```go
func Derive(address []byte, derivationKey []byte) []byte {
    return Hash(addres, derivationKey)
}
```

注意:`Module` 是更通用的 _derived_ 地址的一个特例，我们为 _from address_ 设置了 `"module"` 字符串。

**示例** 对于 cosmwasm 智能合约地址，我们可以使用以下结构: 

```
smartContractAddr := Derived(Module("cosmwasm", smartContractsNamespace), []{smartContractKey})
```

### 架构类型

`Hash` 函数中使用的 `typ` 参数对于每个账户类型应该是唯一的。
由于所有 Cosmos SDK 账户类型都在 state 中进行了序列化，我们建议使用 protobuf 消息名称字符串。

示例:所有公钥类型都有一个唯一的 protobuf 消息类型，类似于: 

```proto
package cosmos.crypto.sr25519;

message PubKey {
  bytes key = 1;
}
```

所有 protobuf 消息都具有唯一的完全限定名称，在此示例中为“cosmos.crypto.sr25519.PubKey”。
这些名称以标准化方式直接从 .proto 文件派生并使用
在其他地方，例如`Any`s 中的类型 URL。 我们可以使用以下方法轻松获取名称
`proto.MessageName(msg)`。 

## Consequences

### Backwards Compatibility

此 ADR 与 Cosmos SDK 存储库中提交和直接支持的内容兼容。 

### Positive

- 一种为新公钥、复杂账户和模块生成地址的简单算法
- 该算法概括了_原生组合键_
- 提高地址的安全性和抗冲突性
- 该方法可扩展用于未来的用例 - 可以使用其他地址类型，只要它们与此处指定的地址长度(20 或 32 字节)不冲突。
- 支持新的帐户类型。 

### Negative

- 地址不传达密钥类型，前缀方法可以做到这一点
- 地址要长 60%，并且会消耗更多的存储空间
- 需要重构 KVStore 存储密钥来处理可变长度地址 

### Neutral

- protobuf message names are used as key type prefixes

## Further Discussions

一些帐户可以有一个固定的名称或可以以其他方式构建(例如:模块)。 我们正在讨论一个可以由机构使用的具有预定义名称(例如:`me.regen`)的帐户的想法。
无需赘述，这些类型的地址与此处描述的基于哈希的地址兼容，只要它们的长度不同即可。
更具体地说，任何特殊帐户地址的长度不得等于 20 或 32 个字节。 

## Appendix: Consulting session

2020 年 12 月末，我们与 [Alan Szepieniec](https://scholar.google.be/citations?user=4LyZn8oAAAAJ&hl=en) 进行了一次会议，以咨询上述方法。

Alan 一般意见:

+ 我们不需要 2 原像电阻
+ 我们需要 32 字节的地址空间来抵抗碰撞
+ 当攻击者可以通过地址控制对象的输入时，我们就有了生日攻击的问题
+ 用于散列的智能合约存在问题
+ sha2 挖掘可用于破解地址原像

哈希算法

+ 任何破坏 blake3 的攻击都会破坏 blake2
+ Alan 对 blake 哈希算法的当前安全分析非常有信心。入围了决赛，作者在证券分析方面很有名。

算法:

+ Alan 建议对前缀进行哈希处理:`address(pub_key) = hash(hash(key_type) + pub_key)[:32]`，主要优点:
    + 我们可以自由使用任意长前缀名称
    + 我们仍然不会冒碰撞的风险
    + 切换表
+ 关于惩罚的讨论 -> 关于添加前缀后哈希
+ Aaron 询问了后哈希前缀(`address(pub_key) = key_type + hash(pub_key)`)和差异。 Alan 指出，这种方法具有更长的地址空间并且更强大。

复杂/组合键的算法:

+ 使用相同算法合并类似地址的树很好

模块地址:模块地址应该有不同的大小来区分吗？

+ 我们需要为模块地址设置一个前映像前缀以将它们保留在 32 字节空间中:`hash(hash('module') + module_key)`
+ Aaron 观察:我们已经需要处理可变长度(为了不破坏 secp256k1 密钥)。

ZKP算术哈希函数讨论

+ 波塞冬/救援
+ 问题:风险要大得多，因为我们对算术结构的密码分析技术和历史知之甚少。这仍然是一个活跃研究的新领域和领域。

后量子签名大小

+ Alan 建议:Falcon:速度/体型比 - 非常好。
+ Aaron - 我们应该考虑一下吗？
  Alan:根据早期推断，这东西将能够在 2050 年破解 EC 密码学。但这有很大的不确定性。但是递归/链接/模拟会发生神奇的事情，这可以加快进度。

其他想法

+ 假设我们为 2 个不同的用例使用相同的密钥和两种不同的地址算法。使用它仍然安全吗？ Alan:如果我们想隐藏公钥(这不是我们的用例)，那么它的安全性会降低，但有修复方法。 

### References

+ [Notes](https://hackmd.io/_NGWI4xZSbKzj1BkCqyZMw)
