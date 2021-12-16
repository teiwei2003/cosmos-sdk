#存储

存储是保存应用程序状态的数据结构。 {概要}

### 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## Cosmos SDK存储介绍

Cosmos SDK 附带了大量存储来持久化应用程序的状态。 默认情况下，Cosmos SDK 应用程序的主存储是一个“multistore”，即存储的存储。 开发人员可以根据他们的应用程序需求将任意数量的键值存储添加到多存储中。 multistore 的存在是为了支持 Cosmos SDK 的模块化，因为它允许每个模块声明和管理自己的状态子集。 多存储中的键值存储只能通过特定的能力`key`访问，该能力通常保存在声明存储的模块的[`keeper`](../building-modules/keeper.md)中。 

```
+-----------------------------------------------------+
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 1 - Manage by keeper of Module 1  |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 2 - Manage by keeper of Module 2  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 3 - Manage by keeper of Module 2  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 4 - Manage by keeper of Module 3  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|    +--------------------------------------------+   |
|    |                                            |   |
|    |  KVStore 5 - Manage by keeper of Module 4  |   |
|    |                                            |   |
|    +--------------------------------------------+   |
|                                                     |
|                    Main Multistore                  |
|                                                     |
+-----------------------------------------------------+

                   Application's State
```

###存储界面

在其核心，Cosmos SDK `store` 是一个对象，它包含一个 `CacheWrapper` 并具有一个 `GetStoreType()` 方法:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L15-L18

`GetStoreType` 是一个返回存储类型的简单方法，而一个 `CacheWrapper` 是一个简单的接口，它通过 `Write` 方法实现存储读取缓存和写入分支:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L240-L264

分支和缓存在 Cosmos SDK 中无处不在，并且需要在每种存储类型上实现。一个存储分支创建一个隔离的、临时的存储分支，可以在不影响主要底层存储的情况下传递和更新。这用于触发临时状态转换，如果发生错误，可以稍后恢复。在 [context](./context.md#Store-branching) 中阅读更多相关信息

### 提交存储

提交存储是一种能够提交对底层树或数据库所做更改的存储。 Cosmos SDK 通过使用 `Committer` 扩展基本存储接口来区分简单存储和提交存储:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L29-L33

`Committer` 是一个接口，它定义了将更改持久化到磁盘的方法:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L20-L27

`CommitID` 是状态树的确定性提交。它的哈希值返回到底层的共识引擎并存储在块头中。请注意，提交存储接口有多种用途，其中之一是确保不是每个对象都可以提交存储。作为 Cosmos SDK 的 [object-capabilities 模型](./ocap.md) 的一部分，只有 `baseapp` 应该具有提交存储的能力。例如，这就是为什么模块通常用来访问存储的 `ctx.KVStore()` 方法返回一个 `KVStore` 而不是 `CommitKVStore` 的原因。

Cosmos SDK 提供了多种存储类型，最常用的是 [`CommitMultiStore`](#multistore)、[`KVStore`](#kvstore) 和 [`GasKv` store](#gaskv-store)。 [其他类型的存储](#other-stores) 包括`Transient` 和`TraceKV` 存储。

## 多存储

### 多存储接口

每个 Cosmos SDK 应用程序都在其根部拥有一个多存储以保持其状态。 multistore 是一个遵循 `Multistore` 接口的 `KVStores` 存储:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L104-L133

如果启用了跟踪，那么分支 multistore 将首先将所有底层的 `KVStore` 包装在 [`TraceKv.Store`](#tracekv-store) 中。

### CommitMultiStore

Cosmos SDK 中使用的`Multistore` 的主要类型是`CommitMultiStore`，它是`Multistore` 接口的扩展:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L141-L184

至于具体实现，[`rootMulti.Store`] 是 `CommitMultiStore` 接口的首选实现。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/rootmulti/store.go#L43-L61

`rootMulti.Store` 是一个围绕 `db` 构建的基础层多存储，其上可以安装多个 `KVStores`，并且是 [`baseapp`](./baseapp.md) 中使用的默认多存储存储.

### CacheMultiStore 

每当 `rootMulti.Store` 需要分支时，一个 [`cachemulti.Store`](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/store/cachemulti/store.go) 是用过的。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/cachemulti/store.go#L17-L28

`cachemulti.Store` 在其构造函数中分支所有子存储(为每个子存储创建一个虚拟存储)并将它们保存在 `Store.stores` 中。此外缓存所有读取查询。 `Store.GetKVStore()` 从`Store.stores` 返回存储，并且`Store.Write()` 在所有子存储上递归调用`CacheWrap.Write()`。

## 基础层 KVStores

### `KVStore` 和 `CommitKVStore` 接口

`KVStore` 是一个简单的键值存储，用于存储和检索数据。 `CommitKVStore` 是一个也实现了 `Committer` 的 `KVStore`。默认情况下，安装在`baseapp` 的主`CommitMultiStore` 中的存储是`CommitKVStore`。 `KVStore` 接口主要用于限制模块访问提交者。

模块使用单个“KVStore”来管理全局状态的子集。 `KVStores` 可以被持有特定键的对象访问。这个`key` 应该只暴露给定义存储的模块的 [`keeper`](../building-modules/keeper.md)。

`CommitKVStore`s 由它们各自的 `key` 代理声明，并安装在应用程序的 [multistore](#multistore) 上的[主应用程序文件](../basics/app-anatomy.md#core-application-file )。在同一个文件中，`key` 也被传递给负责管理 store 的模块的 `keeper`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/store.go#L189-L219

除了传统的`Get` 和`Set` 方法，`KVStore` 必须提供一个`Iterator(start, end)` 方法，该方法返回一个`Iterator` 对象。它用于迭代一系列键，通常是共享公共前缀的键。以下是银行模块 keeper 的示例，用于迭代所有帐户余额:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/x/bank/keeper/view.go#L115-L134

### `IAVL`存储

`baseapp` 中使用的 `KVStore` 和 `CommitKVStore` 的默认实现是 `iavl.Store`。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/iavl/store.go#L37-L40

`iavl`存储基于 [IAVL 树](https://github.com/tendermint/iavl)，这是一种自平衡二叉树，可保证:

- `Get` 和 `Set` 操作的复杂度为 O(log n)，其中 n 是树中元素的数量。
- 迭代有效地返回范围内的排序元素。
- 每个树版本都是不可变的，即使在提交后也可以检索(取决于修剪设置)。

关于 IAVL 树的文档位于 [此处](https://github.com/cosmos/iavl/blob/v0.15.0-rc5/docs/overview.md)。

### `DbAdapter` 存储

`dbadapter.Store` 是 `dbm.DB` 的适配器，使其实现 `KVStore` 接口。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/dbadapter/store.go#L13-L16

`dbadapter.Store` 嵌入了 `dbm.DB`，这意味着大部分 `KVStore` 接口功能都已实现。其他功能(主要是杂项)是手动实现的。此存储主要用于 [Transient Stores](#transient-stores)

### `瞬态`存储

`Transient.Store` 是一个基础层 `KVStore`，它在块的末尾被自动丢弃。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/transient/store.go#L13-L16

`Transient.Store` 是一个带有 `dbm.NewMemDB()` 的 `dbadapter.Store`。所有`KVStore` 方法都被重用。当调用`Store.Commit()` 时，会分配一个新的`dbadapter.Store`，丢弃之前的引用并使其垃圾回收。

这种类型的存储对于保留仅与每个块相关的信息很有用。一个例子是存储参数更改(即，如果块中的参数发生更改，则将 bool 设置为“true”)。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/x/params/types/subspace.go#L20-L30

瞬态存储通常通过 [`context`](./context.md) 通过 `TransientStore()` 方法访问:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/types/context.go#L232-L235

## KVStore 包装器

### CacheKVStore

`cachekv.Store` 是一个包装器 `KVStore`，它在底层 `KVStore` 上提供缓冲写入/缓存读取功能。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/cachekv/store.go#L27-L34

每当需要分支 IAVL 存储以创建隔离存储时(通常当我们需要改变可能会在以后恢复的状态时)，就会使用这种类型。

#### `获取`

`Store.Get()` 首先检查 `Store.cache` 是否具有与键相关联的值。如果该值存在，则函数返回它。如果没有，该函数调用`Store.parent.Get()`，将结果缓存在`Store.cache`中，并返回它。 

#### `设置`

`Store.Set()` 将键值对设置为 `Store.cache`。 `cValue` 具有字段dirty bool，指示缓存值是否与底层值不同。当`Store.Set()`缓存一个新对时，`cValue.dirty`被设置为`true`，所以当`Store.Write()`被调用时，它可以被写入底层存储。

#### `迭代器`

`Store.Iterator()` 必须遍历缓存项和原始项。在`Store.iterator()` 中，为每个迭代器生成两个迭代器，并合并。 `memIterator` 本质上是 `KVPairs` 的一部分，用于缓存项。 `mergeIterator` 是两个迭代器的组合，遍历是在两个迭代器上有序进行的。

### `GasKv`存储

Cosmos SDK 应用程序使用 [`gas`](../basics/gas-fees.md) 来跟踪资源使用情况并防止垃圾邮件。 [`GasKv.Store`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/gaskv/store.go) 是一个 `KVStore` 包装器，可以自动消耗每个 Gas读取或写入存储的时间。它是在 Cosmos SDK 应用程序中跟踪存储使用情况的首选解决方案。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/gaskv/store.go#L13-L19

当父`KVStore`的方法被调用时，`GasKv.Store`会根据`Store.gasConfig`自动消耗适量的gas:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/gas.go#L153-L162

默认情况下，所有 `KVStores` 在检索时都包含在 `GasKv.Stores` 中。这是在 [`context`](./context.md) 的 `KVStore()` 方法中完成的:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/types/context.go#L227-L230

在这种情况下，使用默认气体配置:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/types/gas.go#L164-L175

### `TraceKv`存储

`tracekv.Store` 是一个包装器 `KVStore`，它在底层 `KVStore` 上提供操作跟踪功能。如果在父“MultiStore”上启用跟踪，Cosmos SDK 会自动在所有“KVStore”上应用它。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/tracekv/store.go#L20-L43

当每个 `KVStore` 方法被调用时，`tracekv.Store` 会自动将 `traceOperation` 记录到 `Store.writer`。 `traceOperation.Metadata` 填充为 `Store.context` 当它不为零时。 `TraceContext` 是一个 `map[string]interface{}`。

### `Prefix` 存储

`prefix.Store` 是一个包装器 `KVStore`，它在底层的 `KVStore` 上提供自动键前缀功能。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc6/store/prefix/store.go#L15-L21

当调用`Store.{Get, Set}()` 时，存储将调用转发给其父级，键以`Store.prefix` 为前缀。

当调用`Store.Iterator()` 时，它不会简单地给`Store.prefix` 加上前缀，因为它不能按预期工作。在这种情况下，即使某些元素不是以前缀开头，也会遍历它们。

### `ListenKv`存储

`listenkv.Store` 是一个包装器 `KVStore`，它提供了对底层 `KVStore` 的状态监听功能。
它由 Cosmos SDK 自动应用于任何在状态流配置期间指定了“StoreKey”的“KVStore”。
有关状态流配置的其他信息可以在 [store/streaming/README.md](../../store/streaming/README.md) 中找到。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.44.1/store/listenkv/store.go#L11-L18

当调用`KVStore.Set` 或`KVStore.Delete` 方法时，`listenkv.Store` 会自动将操作写入到`Store.listeners` 集合中。

## 新存储包 (`store/v2`)

SDK 正在过渡到使用此处列出的类型作为状态存储的默认接口。在撰写本文时，这些无法在应用程序中使用，并且与 `CommitMultiStore` 和相关类型不直接兼容。

### `BasicKVStore` 接口

仅提供基本 CRUD 功能(`Get`、`Set`、`Has` 和 `Delete` 方法)的接口，没有迭代或缓存。这用于部分公开较大存储的组件，例如`flat.Store`。

### 平铺

`flat.Store` 是新的默认持久化存储，它在内部解耦了状态存储和承诺方案的关注点。值直接存储在后备键值数据库(“存储”存储桶)中，而值的哈希映射到能够生成加密承诺的单独存储(“状态承诺”存储桶，使用 `smt.存储`)。

这可以选择性地构建为对每个存储桶使用不同的后端数据库。

<!-- TODO: 添加链接 +++ https://github.com/cosmos/cosmos-sdk/blob/v0.44.0/store/v2/flat/store.go -->

### SMT存储

一个 `BasicKVStore`，用于部分公开底层存储的功能(例如，允许访问 `flat.Store` 中的承诺存储)。

## 下一个 {hide}

了解 [encoding](./encoding.md) {hide} 
