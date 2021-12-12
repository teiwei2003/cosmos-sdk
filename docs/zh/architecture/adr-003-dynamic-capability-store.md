# ADR 3：动态能力存储

## 变更日志

- 2019 年 12 月 12 日：初始版本
- 2020 年 4 月 2 日：内存存储修订

## 语境

[IBC 规范](https://github.com/cosmos/ibs) 的完整实现需要能够在运行时（即在事务执行期间）创建和验证对象能力密钥，
如 [ICS 5](https://github.com/cosmos/ibc/tree/master/spec/core/ics-005-port-allocation#technical-specification) 中所述。在 IBC 规范中，为每个新初始化的
端口和通道，用于验证端口或通道的未来使用情况。由于通道和潜在的端口可以在事务执行期间初始化，状态机必须能够创建
此时的对象功能键。

目前，Cosmos SDK 没有能力做到这一点。对象功能键当前是在应用程序初始化时在 `app.go` 中创建的 `StoreKey` 结构的指针（内存地址）（[示例]（https://github.com/cosmos/gaia/blob/dcbddd9f04b3086c0ad07ee65de16e7adedc7da4/app/app .go#L132))
并作为固定参数传递给 Keepers（[示例]（https://github.com/cosmos/gaia/blob/dcbddd9f04b3086c0ad07ee65de16e7adedc7da4/app/app.go#L160））。 Keepers 无法在交易执行期间创建或存储功能密钥——尽管他们可以调用 `NewKVStoreKey` 并获取内存地址
对于返回的结构，将其存储在 Merklised 存储中会导致共识错误，因为每台机器上的内存地址将不同（这是故意的 - 如果不是这种情况，密钥将是可预测的，并且不能用作对象能力）。

Keepers 需要一种方法来保存可以在事务执行期间更改的存储密钥的私有映射，以及在应用程序启动或重新启动时重新生成此映射中唯一内存地址（功能密钥）的合适机制，以及一种机制在 tx 失败时恢复能力创建。
本 ADR 提出了这样的接口和机制。

## 决定

Cosmos SDK 将包含一个新的“CapabilityKeeper”抽象，它负责供应、
在运行时跟踪和验证功能。在 app.go 中的应用程序初始化期间，
`CapabilityKeeper` 将通过唯一的函数引用连接到模块
（通过调用`ScopeToModule`，定义如下）以便稍后识别调用模块
调用。

当从磁盘加载初始状态时，`CapabilityKeeper` 的 `Initialise` 函数将创建
所有先前分配的能力标识符的新能力密钥（在执行过程中分配）
过去的事务并分配给特定模式），并将它们保存在仅内存存储中，而
链正在运行。

`CapabilityKeeper` 将包括一个持久化的 `KVStore`、一个 `MemoryStore` 和一个内存映射。
持久性“KVStore”跟踪哪些功能由哪些模块拥有。
`MemoryStore` 存储从模块名称、能力元组到能力名称和
从模块名称、功能名称映射到功能索引的反向映射。
由于我们不能在不改变能力的内存位置的情况下将能力编组到`KVStore`中并解组，
KVStore 中的反向映射将简单地映射到索引。然后可以将此索引用作临时文件中的键
go-map 在原始内存位置检索功能。

`CapabilityKeeper` 将定义以下类型和功能：

`Capability` 类似于 `StoreKey`，但有一个全局唯一的 `Index()` 而不是
一个名字。提供了一个 `String()` 方法用于调试。

`Capability` 只是一个结构体，它的地址被用于实际的能力。

```golang
type Capability struct {
  index uint64
}
```

“CapabilityKeeper”包含一个持久存储键、内存存储键和已分配模块名称的映射。

```golang
type CapabilityKeeper struct {
  persistentKey StoreKey
  memKey        StoreKey
  capMap        map[uint64]*Capability
  moduleNames   map[string]interface{}
  sealed        bool
}
```

`CapabilityKeeper` 提供了创建 *scoped* 子管理器的能力，这些子管理器绑定到一个
特定的模块名称。 这些`ScopedCapabilityKeeper`s 必须在应用程序初始化时创建
并传递给模块，然后模块可以使用它们来声明它们接收和检索的功能
除了创建新功能和身份验证功能之外，他们按名称拥有的功能
通过其他模块。

```golang
type ScopedCapabilityKeeper struct {
  persistentKey StoreKey
  memKey        StoreKey
  capMap        map[uint64]*Capability
  moduleName    string
}
```

`ScopeToModule` 用于创建具有特定名称的作用域子管理器，该名称必须是唯一的。
它必须在 `InitialiseAndSeal` 之前调用。

```golang
func (ck CapabilityKeeper) ScopeToModule(moduleName string) ScopedCapabilityKeeper {
	if k.sealed {
		panic("cannot scope to module via a sealed capability keeper")
	}

	if _, ok := k.scopedModules[moduleName]; ok {
		panic(fmt.Sprintf("cannot create multiple scoped keepers for the same module name: %s", moduleName))
	}

	k.scopedModules[moduleName] = struct{}{}

	return ScopedKeeper{
		cdc:      k.cdc,
		storeKey: k.storeKey,
		memKey:   k.memKey,
		capMap:   k.capMap,
		module:   moduleName,
	}
}
```

`InitialiseAndSeal` 必须在加载初始状态并创建所有
必要的“ScopedCapabilityKeeper”，以便用新创建的内存填充
根据特定模块先前声明的密钥的能力密钥，并防止
创建任何新的“ScopedCapabilityKeeper”。

```golang
func (ck CapabilityKeeper) InitialiseAndSeal(ctx Context) {
  if ck.sealed {
    panic("capability keeper is sealed")
  }

  persistentStore := ctx.KVStore(ck.persistentKey)
  map := ctx.KVStore(ck.memKey)
  
  // initialise memory store for all names in persistent store
  for index, value := range persistentStore.Iter() {
    capability = &CapabilityKey{index: index}

    for moduleAndCapability := range value {
      moduleName, capabilityName := moduleAndCapability.Split("/")
      memStore.Set(moduleName + "/fwd/" + capability, capabilityName)
      memStore.Set(moduleName + "/rev/" + capabilityName, index)

      ck.capMap[index] = capability
    }
  }

  ck.sealed = true
}
```

任何模块都可以调用“NewCapability”来创建一个新的独特的、不可伪造的对象能力
参考。 新创建的能力会自动持久化； 调用模块不需要
调用`ClaimCapability`。

```golang
func (sck ScopedCapabilityKeeper) NewCapability(ctx Context, name string) (Capability, error) {
  // check name not taken in memory store
  if capStore.Get("rev/" + name) != nil {
    return nil, errors.New("name already taken")
  }

  // fetch the current index
  index := persistentStore.Get("index")
  
  // create a new capability
  capability := &CapabilityKey{index: index}
  
  // set persistent store
  persistentStore.Set(index, Set.singleton(sck.moduleName + "/" + name))
  
  // update the index
  index++
  persistentStore.Set("index", index)
  
  // set forward mapping in memory store from capability to name
  memStore.Set(sck.moduleName + "/fwd/" + capability, name)
  
  // set reverse mapping in memory store from name to index
  memStore.Set(sck.moduleName + "/rev/" + name, index)

  // set the in-memory mapping from index to capability pointer
  capMap[index] = capability
  
  // return the newly created capability
  return capability
}
```

任何模块都可以调用 AuthenticateCapability 来检查能力
实际上确实对应于特定名称（该名称可以是不受信任的用户输入）
调用模块之前与之关联的。

```golang
func (sck ScopedCapabilityKeeper) AuthenticateCapability(name string, capability Capability) bool {
  // return whether forward mapping in memory store matches name
  return memStore.Get(sck.moduleName + "/fwd/" + capability) === name
}
```

`ClaimCapability` 允许一个模块声明它从另一个模块收到的能力密钥
这样未来的`GetCapability` 调用就会成功。

如果接收能力的模块希望通过名称访问它，则必须调用`ClaimCapability`
将来。 能力是多所有者的，所以如果多个模块有一个单一的“能力”引用，
他们都会拥有它。

```golang
func (sck ScopedCapabilityKeeper) ClaimCapability(ctx Context, capability Capability, name string) error {
  persistentStore := ctx.KVStore(sck.persistentKey)

  // set forward mapping in memory store from capability to name
  memStore.Set(sck.moduleName + "/fwd/" + capability, name)

  // set reverse mapping in memory store from name to capability
  memStore.Set(sck.moduleName + "/rev/" + name, capability)

  // update owner set in persistent store
  owners := persistentStore.Get(capability.Index())
  owners.add(sck.moduleName + "/" + name)
  persistentStore.Set(capability.Index(), owners)
}
```

`GetCapability` 允许模块获取它之前通过名称声明的能力。
不允许模块检索不属于它的功能。

```golang
func (sck ScopedCapabilityKeeper) GetCapability(ctx Context, name string) (Capability, error) {
  // fetch the index of capability using reverse mapping in memstore
  index := memStore.Get(sck.moduleName + "/rev/" + name)

  // fetch capability from go-map using index
  capability := capMap[index]

  // return the capability
  return capability
}
```

`ReleaseCapability` 允许模块释放它之前声明的能力。 如果不
存在更多所有者，该能力将被全局删除。

```golang
func (sck ScopedCapabilityKeeper) ReleaseCapability(ctx Context, capability Capability) err {
  persistentStore := ctx.KVStore(sck.persistentKey)

  name := capStore.Get(sck.moduleName + "/fwd/" + capability)
  if name == nil {
    return error("capability not owned by module")
  }

  // delete forward mapping in memory store
  memoryStore.Delete(sck.moduleName + "/fwd/" + capability, name)

  // delete reverse mapping in memory store
  memoryStore.Delete(sck.moduleName + "/rev/" + name, capability)

  // update owner set in persistent store
  owners := persistentStore.Get(capability.Index())
  owners.remove(sck.moduleName + "/" + name)
  if owners.size() > 0 {
    // there are still other owners, keep the capability around
    persistentStore.Set(capability.Index(), owners)
  } else {
    // no more owners, delete the capability
    persistentStore.Delete(capability.Index())
    delete(capMap[capability.Index()])
  }
}
```

### 使用模式

#### 初始化

任何使用动态功能的模块都必须在 `app.go` 中提供一个 `ScopedCapabilityKeeper`：

```golang
ck := NewCapabilityKeeper(persistentKey, memoryKey)
mod1Keeper := NewMod1Keeper(ck.ScopeToModule("mod1"), ....)
mod2Keeper := NewMod2Keeper(ck.ScopeToModule("mod2"), ....)

// other initialisation logic ...

// load initial state...

ck.InitialiseAndSeal(initialContext)
```

#### 创建、传递、声明和使用功能

考虑这样一种情况，“mod1”想要创建一个能力，通过名称将其与资源（例如 IBC 通道）相关联，然后将其传递给“mod2”，后者将在稍后使用它：

模块 1 将具有以下代码：

```golang
capability := scopedCapabilityKeeper.NewCapability(ctx, "resourceABC")
mod2Keeper.SomeFunction(ctx, capability, args...)
```

`SomeFunction`，在模块 2 中运行，然后可以声明该功能：

```golang
func (k Mod2Keeper) SomeFunction(ctx Context, capability Capability) {
  k.sck.ClaimCapability(ctx, capability, "resourceABC")
  // other logic...
}
```

稍后，模块 2 可以通过名称检索该功能并将其传递给模块 1，模块 1 将根据资源对其进行身份验证：

```golang
func (k Mod2Keeper) SomeOtherFunction(ctx Context, name string) {
  capability := k.sck.GetCapability(ctx, name)
  mod1.UseResource(ctx, capability, "resourceABC")
}
```

在允许模块 2 使用资源之前，模块 1 将检查此功能密钥是否经过身份验证以使用资源：

```golang
func (k Mod1Keeper) UseResource(ctx Context, capability Capability, resource string) {
  if !k.sck.AuthenticateCapability(name, capability) {
    return errors.New("unauthenticated")
  }
  // do something with the resource
}
```

如果模块 2 将功能密钥传递给模块 3，则模块 3 可以声明它并像模块 2 一样调用模块 1
（在这种情况下，模块 1、模块 2 和模块 3 都可以使用此功能）。

## 地位

建议的。

## 结果

### 积极的

- 动态能力支持。
- 允许 CapabilityKeeper 从 go-map 返回相同的能力指针，同时在 tx 失败时恢复对持久性“KVStore”和内存中“MemoryStore”的任何写入。

### 消极的

- 需要额外的守门员。
- 与现有的“StoreKey”系统有些重叠（将来它们可以合并，因为这是一个超集功能）。
- 在反向映射中需要额外的间接级别，因为 MemoryStore 必须映射到索引，然后必须将其用作 go map 中的键以检索实际功能

### 中性的

（无人知晓）

## 参考

- [原创讨论](https://github.com/cosmos/cosmos-sdk/pull/5230#discussion_r343978513)