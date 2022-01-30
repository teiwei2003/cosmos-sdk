# ADR 040:存储和 SMT 状态承诺

## 变更日志

- 2020-01-15:草案

## 地位

草案未实施

## 摘要

稀疏默克尔树 ([SMT](https://osf.io/8mcnh/)) 是具有各种存储和性能优化的默克尔树的一个版本。此 ADR 定义了状态承诺与数据存储的分离以及 Cosmos SDK 从 IAVL 到 SMT 的转换。

##  上下文

目前，Cosmos SDK 将 IAVL 用于状态 [承诺](https://cryptography.fandom.com/wiki/Commitment_scheme) 和数据存储。

IAVL 已有效地成为 Cosmos 生态系统中的一个孤立项目，并且已被证明是一种低效的状态承诺数据结构。
在当前的设计中，IAVL 用于数据存储和状态承诺的默克尔树。 IAVL 旨在成为独立的默克尔化键/值数据库，但它使用 KV 数据库引擎来存储所有树节点。因此，每个节点都存储在 KV DB 中的单独记录中。这会导致许多低效率和问题:

+ 每个对象查询都需要从根开始遍历树。对同一对象的后续查询缓存在 Cosmos SDK 级别。
+ 每条边遍历都需要一个 DB 查询。
+ 创建快照 [昂贵](https://github.com/cosmos/cosmos-sdk/issues/7215#issuecomment-684804950)。导出不到 100 MB 的状态大约需要 30 秒(截至 2020 年 3 月)。
+ IAVL 中的更新可能会触发树重组和可能的 O(log(n)) 哈希重新计算，这可能会成为 CPU 瓶颈。
+ 节点结构非常昂贵——它包含一个标准的树节点元素(键、值、左右元素)和额外的元数据，如高度、版本(Cosmos SDK 不需要)。对整个节点进行哈希处理，并将该哈希用作底层数据库中的键，[ref](https://github.com/cosmos/iavl/blob/master/docs/node/node.md
)。

此外，IAVL 项目缺乏支持和维护者，我们已经看到了更好和完善的替代方案。我们没有优化 IAVL，而是在研究其他解决方案，以实现存储和状态承诺。

## 决定

我们建议将共识所需的状态承诺 (**SC**) 和状态机所需的状态存储 (**SS**) 的关注点分开。最后我们用 [Celestia's SMT](https://github.com/lazyledger/smt) 替换 IAVL。 Celestia SMT 基于 Diem(称为 jellyfish)设计 [*] - 它使用计算优化的 SMT，通过用单个节点替换仅具有默认值的子树(Ethereum2 使用相同的方法)并实现紧凑证明。

此处介绍的存储模型不处理数据结构或序列化。它是一个键值数据库，其中键和值都是二进制文件。存储用户负责数据序列化。

### 将状态承诺与存储分离

存储和承诺的分离(通过 SMT)将允许根据不同组件的使用和访问模式对其进行优化。

`SC` (SMT) 用于提交数据和计算 Merkle 证明。 `SS` 用于直接访问数据。为了避免冲突，`SS` 和 `SC` 都将使用单独的存储命名空间(它们可以在下面使用相同的数据库)。 `SS` 将直接存储每条记录(将 `(key, value)` 映射为 `key → value`)。

SMT 是一种默克尔树结构:我们不直接存储密钥。对于每个`(key, value)`对，`hash(key)`用作叶子路径(我们散列一个键以在树中均匀分布叶子)和`hash(value)`作为叶子内容。 [下面](#smt-for-state-commitment) 更深入地指定了树结构。

对于数据访问，我们建议使用 2 个额外的 KV 存储桶(作为键值对的命名空间实现，有时称为 [列族](https://github.com/facebook/rocksdb/wiki/Terminology)):

1. B1:`key → value`:状态机使用的主体对象存储，在Cosmos SDK`KVStore`接口后面:提供key直接访问，允许前缀迭代(KV DB后端必须支持)。
2. B2:`hash(key) → key`:从SMT路径中获取密钥的反向索引。在内部，SMT 会将 `(key, value)` 存储为 `prefix ||哈希(键) ||哈希(值)`。所以，我们可以通过组合`hash(key) → B2 → B1`来获得一个对象值。
3. 如果需要，我们可以使用更多的桶来优化应用程序的使用。

我们建议为“SS”和“SC”使用 KV 数据库。存储接口将允许对“SS”和“SC”以及两个单独的数据库使用相同的物理数据库后端。后一种选项允许将 `SS` 和 `SC` 分离到不同的硬件单元中，为更复杂的设置场景提供支持并提高整体性能:可以使用不同的后端(例如 RocksDB 和 Badger)以及独立调整底层数据库配置。 

### Requirements

状态存储要求:

+ 范围查询
+ 快速(键、值)访问
+ 创建快照
+ 历史版本
+ 修剪(垃圾收集)

国家承诺要求:

+ 快速更新
+ 树路径应该很短
+ 使用 ICS-23 标准查询历史承诺证明
+ 修剪(垃圾收集)

### 国家承诺的 SMT

稀疏 Merkle 树基于难以处理的大小的完整 Merkle 树的想法。这里的假设是，由于树的大小是难以处理的，因此相对于树的大小，将只有少数具有有效数据块的叶节点，从而呈现稀疏树。

完整规范可以在 [Celestia](https://github.com/celestiaorg/celestia-specs/blob/ec98170398dfc6394423ee79b00b71038879e211/src/specs/data_structures.md#sparse-merkle-tree) 找到。总之:

* SMT 由二叉 Merkle 树组成，以与 [证书透明度 (RFC-6962)](https://tools.ietf.org/html/rfc6962) 中所述相同的方式构建，但用作散列函数 SHA -2-256，如 [FIPS 180-4](https://doi.org/10.6028/NIST.FIPS.180-4) 中所定义。
* 叶节点和内部节点的散列方式不同:叶节点前面加上一字节的“0x00”，而内部节点前面加上“0x01”。
* 给空叶节点赋予默认值。
* 虽然上述规则足以预先计算作为空子树根的中间节点的值，但进一步简化是将此默认值扩展到作为空子树根的所有节点。 32 字节的零用作默认值。此规则优先于上述规则。
* 作为包含一个非空叶子的子树根的内部节点被该叶子的叶子节点替换。

### 用于存储同步和状态版本控制的快照

下面，通过简单的 _snapshot_，我们指的是数据库快照机制，而不是 _ABCI 快照同步_。后者将被称为 _snapshot sync_(将直接使用数据库快照，如下所述)。

数据库快照是特定时间或事务的数据库状态视图。它不是数据库的完整副本(它会太大)。通常快照机制基于 _copy on write_，它允许在某个阶段有效地传递 DB 状态。
一些数据库引擎支持快照。因此，我们建议将该功能重用于状态同步和版本控制(如下所述)。我们将支持的数据库引擎限制为有效实现快照的引擎。在最后一节中，我们将讨论评估的 DB。

Stargate 的核心功能之一是`/snapshot` 包中提供的_snapshot sync_。它提供了一种无需信任地同步区块链而无需重复所有交易的方法。此功能在 Cosmos SDK 中实现，需要存储支持。目前 IAVL 是唯一受支持的后端。它的工作原理是将某个版本的“SS”的快照与标头链一起流式传输到客户端。

将在每个“EndBlocker”中创建一个新的数据库快照，并由块高度标识。 `root` 存储跟踪可用快照以提供特定版本的 `SS`。 `root` 存储实现了下面描述的 `RootStore` 接口。本质上，`RootStore` 封装了一个`Committer` 接口。 `Committer` 有一个 `Commit`、`SetPruning`、`GetPruning` 函数，用于创建和删除快照。 `rootStore.Commit` 函数创建一个新快照并在每次调用时增加版本，并检查是否需要删除旧版本。我们需要更新 SMT 接口以实现 `Committer` 接口。
注意:每个块必须只调用一次 `Commit`。否则，我们可能会面临版本号和区块高度不同步的风险。
注意:对于 Cosmos SDK 存储，我们可以考虑将该接口拆分为 `Committer` 和 `PruningCommitter` - 只有 multiroot 应该实现 `PruningCommitter`(缓存和前缀存储不需要修剪)。 

`abci.RequestQuery` 和状态同步快照的历史版本数是节点配置的一部分，而不是链配置(区块链共识隐含的配置)。配置应该允许指定过去块的数量和过去块的数量以某个数字为模(例如:过去的 100 个块和过去 2000 个块的每 100 个块的一个快照)。存档节点可以保留所有过去的版本。

修剪旧快照是由数据库有效完成的。每当我们更新 `SC` 中的记录时，SMT 不会更新节点——而是在更新路径上创建新节点，而不删除旧节点。由于我们正在对每个块进行快照，因此我们需要更改该机制以立即从数据库中删除孤立节点。这是一项安全操作 - 快照将跟踪记录并在访问过去版本时使其可用。

为了管理活动快照，我们将使用 DB _max number of snapshots_ 选项(如果可用)，或者我们将在“EndBlocker”中删除 DB 快照。通过识别具有块高度的快照并调用存储函数来删除过去的版本，可以有效地完成后一个选项。

#### 访问旧的状态版本

功能要求之一是访问旧状态。这是通过“abci.RequestQuery”结构完成的。版本由块高度指定(因此我们在块高度“H”处通过键“K”查询对象)。 `abci.RequestQuery` 支持的旧版本数量是可配置的。访问旧状态是通过使用可用快照完成的。
`abci.RequestQuery` 不需要 `SC` 的旧状态，除非设置了 `prove=true` 参数。仅当 `SC` 和 `SS` 都有所请求版本的快照时，SMT 默克尔证明必须包含在 `abci.ResponseQuery` 中。

此外，Cosmos SDK 可以提供一种直接访问历史状态的方法。但是，状态机不应该这样做——因为快照的数量是可配置的，它会导致非确定性执行。

我们积极[验证](https://github.com/cosmos/cosmos-sdk/discussions/8297) 一种版本控制和快照机制，用于查询我们评估的数据库的旧状态。

### 状态证明

对于存储在状态存储 (SS) 中的任何对象，我们在“SC”中都有相应的对象。由键“K”标识的对象“V”的证明是“SC”的一个分支，其中路径对应于键“hash(K)”，叶子是“hash(K, V)”。

### 回滚

如果事务失败，我们需要能够处理事务和回滚状态更新。这可以通过以下方式完成:在事务处理期间，我们将所有状态更改请求(写入)保存在“CacheWrapper”抽象中(就像今天所做的那样)。一旦我们完成块处理，在“Endblocker”中，我们提交一个根存储 - 那时，所有更改都会写入 SMT 和“SS”，并创建快照。

### 提交一个对象而不保存它

我们确定了用例，其中模块需要保存对象承诺而不存储对象本身。有时客户端会收到复杂的对象，如果不知道存储布局，他们就无法证明该对象的正确性。对于这些用例，提交对象而不直接存储它会更容易。

### 重构 MultiStore

Stargate `/store` 实现 (store/v1) 在 SDK存储构建中添加了一个额外的层 - `MultiStore` 结构。 multistore 的存在是为了支持 Cosmos SDK 的模块化——每个模块都使用自己的 IAVL 实例，但在当前的实现中，所有实例共享同一个数据库。然而，后者表明该实现没有提供真正的模块化。相反，它会导致与竞争条件和原子数据库提交相关的问题(请参阅:[\#6370](https://github.com/cosmos/cosmos-sdk/issues/6370) 和 [讨论](https://github.com)。 com/cosmos/cosmos-sdk/discussions/8297#discussioncomment-757043))。

我们建议减少 SDK 中的多存储概念，并在“RootStore”对象中使用“SC”和“SS”的单个实例。为了避免混淆，我们应该将 `MultiStore` 接口重命名为 `RootStore`。 `RootStore` 将具有以下界面；为简洁起见，省略了配置跟踪和侦听器的方法。 

```go
//Used where read-only access to versions is needed.
type BasicRootStore interface {
    Store
    GetKVStore(StoreKey) KVStore
    CacheRootStore() CacheRootStore
}

//Used as the main app state, replacing CommitMultiStore.
type CommitRootStore interface {
    BasicRootStore
    Committer
    Snapshotter

    GetVersion(uint64) (BasicRootStore, error)
    SetInitialVersion(uint64) error

    ...//Trace and Listen methods
}

//Replaces CacheMultiStore for branched state.
type CacheRootStore interface {
    BasicRootStore
    Write()

    ...//Trace and Listen methods
}

//Example of constructor parameters for the concrete type.
type RootStoreConfig struct {
    Upgrades        *StoreUpgrades
    InitialVersion  uint64

    ReservePrefix(StoreKey, StoreType)
}
```

<!-- TODO: 查看是否可以进一步减少或简化这些类型 -->
<!-- TODO: RootStorePersistentCache 类型 -->

与“MultiStore”相反，“RootStore”不允许动态挂载子存储或为单个子存储提供任意的后备数据库。

注意:模块将能够使用特殊的承诺和他们自己的数据库。例如:一个将 ZK 证明用于状态的模块可以在 `RootStore`(通常作为单个记录)中存储和提交这个证明，并私下或使用 `SC` 低级接口管理专门的存储。

#### 兼容性支持

为了让用户更轻松地过渡到这个新界面，我们可以创建一个 shim，它包装了一个 `CommitMultiStore` 但提供了一个 `CommitRootStore` 接口，并公开函数以安全地创建和访问底层的 `CommitMultiStore`。

新的 `RootStore` 和支持类型可以在 `store/v2` 包中实现，以避免破坏现有代码。

####默克尔证明和IBC

目前，IBC (v1.0) Merkle 证明路径由两个元素(`["<store-key>", "<record-key>"]`)组成，每个键对应一个单独的证明。这些都根据个人 [ICS-23 规范](https://github.com/cosmos/ibc-go/blob/f7051429e1cf833a6f65d51e6c3df1609290a549/modules/core/23-commitment/types/merkle.go#L17) 进行验证将每一步的结果哈希作为下一步的提交值，直到得到一个根提交哈希。
`"<record-key>"` 证明的根散列与 `"<store-key>"` 进行散列，以针对应用程序散列进行验证。

这与“RootStore”不兼容，后者将所有记录存储在单个 Merkle 树结构中，并且不会为存储键和记录键生成单独的证明。理想情况下，可以省略证明的存储密钥组件，并更新为使用“无操作”规范，因此只使用记录密钥。但是，由于 IBC 验证码对 `"ibc"` 前缀进行硬编码，并将其作为证明路径的一个单独元素应用于 SDK 证明，所以如果不进行重大更改，这是不可能的。打破这种行为将严重影响已经广泛采用 IBC 模块的 Cosmos 生态系统。跨链请求更新 IBC 模块是一项耗时的工作，而且不容易实现。

作为一种解决方法，“RootStore”必须使用两个单独的 SMT(它们可以使用相同的底层数据库):一个用于 IBC 状态，另一个用于其他所有内容。引用这些 SMT 的简单 Merkle 映射将充当 Merkle 树以创建最终的 App 哈希。 Merkle 映射不存储在 DB 中——它是在运行时构建的。 IBC 子存储密钥必须是“ibc”。

变通方法仍然可以保证原子同步:[提议的数据库后端](#evaluated-kv-databases) 支持原子事务和高效回滚，这将在提交阶段使用。

在 IBC 模块完全升级以支持单元素承诺证明之前，可以使用所提出的解决方法。

### 优化:压缩模块键前缀

我们考虑通过创建从模块键到整数的映射来压缩前缀键，并使用 varint 编码序列化整数。 Varint 编码确保不同的值没有共同的字节前缀。对于 Merkle 证明，我们不能使用前缀压缩 - 所以它应该只适用于 `SS` 键。此外，前缀压缩应该只应用于模块命名空间。更确切地说:

+ 每个模块都有自己的命名空间；
+ 当访问模块命名空间时，我们创建一个带有嵌入前缀的 KVStore；
+ 该前缀仅在访问和管理“SS”时才会被压缩。

我们需要确保代码不会改变。我们可以在一个特殊的键下修复一个静态变量(由应用程序提供)或 SS 状态中的映射。

TODO:需要决定密钥压缩。 
## 优化:SS 密钥压缩

某些对象可能会使用 key 保存，其中包含 Protobuf 消息类型。 这样的键很长。 如果我们可以在 varint 中映射 Protobuf 消息类型，我们可以节省大量空间。

TODO:完成此操作或移至另一个 ADR。 

## Consequences

### Backwards Compatibility

此 ADR 不会引入任何 Cosmos SDK 级别的 API 更改。

我们更改了状态机的存储布局，需要进行存储硬分叉和网络升级来合并这些更改。 SMT 提供了默克尔证明功能，但它与 ICS23 不兼容。 需要更新 ICS23 兼容性的证明。 

### Positive

+ 将状态与状态承诺解耦为进一步优化和更好的存储模式引入了更好的工程机会。
+ 性能改进。
+ 加入基于 SMT 的阵营，该阵营比 IAVL 具有更广泛且经过验证的采用。 决定采用 SMT 的示例项目:Ethereum2、Diem(Libra)、Trillan、Tezos、Celestia。
+ Multistore 移除修复了当前 MultiStore 设计的一个长期问题。
+ 简化默克尔证明——除 IBC 外，所有模块都只有一次默克尔证明。 

### Negative

+ 存储迁移
+ LL SMT 不支持修剪 - 我们需要添加和测试该功能。
+ `SS` 键将有一个键前缀的开销。 这不会影响 `SC`，因为 `SC` 中的所有键都具有相同的大小(它们被散列)。 

### Neutral

+ 弃用 IAVL，这是 Cosmos 白皮书的核心提案之一。 

## Alternative designs

大多数替代设计在[状态承诺和存储报告](https://paper.dropbox.com/published/State-commitments-and-storage-review--BDvA1MLwRtOx55KRihJ5xxLbBw-KeEB7eOd11pNrZvVtqUgL3h)中进行了评估。

以太坊研究发表 [Verkle Trie](https://dankradfeist.de/ethereum/2021/06/18/verkle-trie-for-eth1.html) - 将多项式承诺与默克尔树相结合以减少树的想法高度。这个概念有很好的潜力，但我们认为现在实施还为时过早。一旦其他研究实施了所有必要的库，当前基于 SMT 的设计就可以轻松更新到 Verkle Trie。本 ADR 中描述的设计的主要优点是将状态承诺与数据存储分离并设计了更强大的接口。

## 进一步讨论

### 评估的 KV 数据库

我们验证了用于评估快照支持的现有数据库 KV 数据库。以下数据库提供了高效的快照机制:Badger、RocksDB、[Pebble](https://github.com/cockroachdb/pebble)。不提供此类支持或未做好生产准备的数据库:boltdb、leveldb、goleveldb、membdb、lmdb。

### 关系型数据库

使用 RDBMS 而不是简单的 KV 存储状态。使用 RDBMS 将需要 Cosmos SDK API 重大更改(`KVStore` 接口)，并将提供更好的数据提取和索引解决方案。不是将对象保存为单个字节块，我们可以将其保存为状态存储层中的表中的记录，并作为上面概述的 SMT 中的“hash(key, protobuf(object))”。为了验证在 RDBMS 中注册的对象与提交给 SMT 的对象相同，需要从 RDBMS 加载它，使用 protobuf 编组，散列并进行 SMT 搜索。

### 连锁存储

我们正在讨论模块可以使用支持数据库的用例，该数据库不会自动提交。模块将负责拥有一个健全的存储模型，并且可以选择使用 __Committing to an object without save it_ 部分中讨论的功能。

## 参考

+ [IAVL 下一步是什么？](https://github.com/cosmos/cosmos-sdk/issues/7100)
+ [IAVL 概述](https://docs.google.com/document/d/16Z_hW2rSAmoyMENO-RlAhQjAG3mSNKsQueMnKpmcBv0/edit#heading=h.yd2th7x3o1iv) v0.15
+ [状态承诺和存储报告](https://paper.dropbox.com/published/State-commitments-and-storage-review--BDvA1MLwRtOx55KRihJ5xxLbBw-KeEB7eOd11pNrZvVtqUgL3h)
+ [Celestia (LazyLedger) SMT](https://github.com/lazyledger/smt)
+ Facebook Diem (Libra) SMT [设计](https://developers.diem.com/papers/jellyfish-merkle-tree/2021-01-14.pdf)
+ [Trillian 撤销透明度](https://github.com/google/trillian/blob/master/docs/papers/RevocationTransparency.pdf)、[Trillian 可验证数据结构](https://github.com/google/trillian/blob/master/docs/papers/VerifiableDataStructures.pdf)。
+ 设计和实现 [讨论](https://github.com/cosmos/cosmos-sdk/discussions/8297)。
+ [如何升级IBC链及其客户端](https://github.com/cosmos/ibc-go/blob/main/docs/ibc/upgrades/quick-guide.md)
+ [ADR-40 对 IBC 的影响](https://github.com/cosmos/ibc-go/discussions/256) 