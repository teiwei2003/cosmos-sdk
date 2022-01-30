# ADR 3:動的容量ストレージ

## 変更ログ

-2019年12月12日:初期バージョン
-2020年4月2日:メモリストレージの改訂

## 環境

[IBC仕様](https://github.com/cosmos/ibs)の完全な実装では、実行時(つまり、トランザクションの実行中)にオブジェクト機能キーを作成および検証できる必要があります。
[ICS 5](https://github.com/cosmos/ibc/tree/master/spec/core/ics-005-port-allocation#technical-specification)で説明されているように。 IBC仕様では、新しく初期化されたものごとに
ポートとチャネルは、ポートまたはチャネルの将来の使用状況を確認するために使用されます。チャネルと潜在的なポートはトランザクションの実行中に初期化できるため、ステートマシンは作成できる必要があります
このときのオブジェクトファンクションキー。

現在、CosmosSDKにはこれを行う機能がありません。オブジェクトファンクションキーは、現在、アプリケーションの初期化時に `app.go`で作成される` StoreKey`構造のポインタ(メモリアドレス)です([例](https://github.com/cosmos/gaia/blob/dcbddd9f04b3086c0ad07ee65de16e7adedc7da4/app/app .go#L132))
そして、それを固定パラメーターとしてKeepers([example](https://github.com/cosmos/gaia/blob/dcbddd9f04b3086c0ad07ee65de16e7adedc7da4/app/app.go#L160))に渡します。キーパーは、トランザクションの実行中にファンクションキーを作成または保存することはできませんが、 `NewKVStoreKey`を呼び出して、メモリアドレスを取得することはできます。
返された構造については、各マシンのメモリアドレスが異なるため、Merklisedストレージに保存すると、コンセンサスエラーが発生します(これは意図的なものです。そうでない場合、キーは予測可能であり、使用できません。オブジェクト機能) 。

キーパーには、トランザクションの実行中に変更できるストレージキーのプライベートマッピングを保存する方法、アプリケーションの起動時または再起動時にこのマッピングで一意のメモリアドレス(ファンクションキー)を再生成するための適切なメカニズム、およびメカニズムが必要です。 txが失敗したときに作成する機能を復元します。
このADRは、そのようなインターフェースとメカニズムを提案します。

## 決定

Cosmos SDKには、新しい[CapabilityKeeper]抽象化が含まれます。
実行時に機能を追跡および検証します。 app.goでのアプリケーションの初期化中に、
`CapabilityKeeper`は、一意の関数リファレンスを介してモジュールに接続します
( `ScopeToModule`を呼び出すことにより、次のように定義されます)後で呼び出し元のモジュールを識別するため
移行。

ディスクから初期状態をロードする場合、 `CapabilityKeeper`の` Initialise`関数が作成します
以前に割り当てられたすべての機能識別子の新しい機能キー(実行中に割り当てられた)
過去のトランザクションと特定のパターンに割り当てられたもの)をメモリのみのストレージに保存し、
チェーンが実行されています。

`CapabilityKeeper`には、永続的な` KVStore`、 `MemoryStore`、およびメモリマップが含まれます。
永続性[KVStore]は、どの機能がどのモジュールによって所有されているかを追跡します。
`MemoryStore`は、モジュール名、機能タプルから機能名、および
モジュール名と関数名から関数インデックスへの逆マッピング。
機能のメモリ位置を変更せずに機能を `KVStore`にグループ化およびグループ解除することはできないため、
KVStoreの逆マッピングは、単にインデックスにマッピングされます。このインデックスは、一時ファイルのキーとして使用できます
Go-mapは、元のメモリ位置にある関数を取得します。

`CapabilityKeeper`は、次のタイプと関数を定義します。

`Capability`は` StoreKey`に似ていますが、代わりにグローバルに一意の `Index()`を持っています
名前。デバッグ用に `String()`メソッドが用意されています。

`Capability`は単なる構造であり、そのアドレスは実際の機能に使用されます。 

```golang
type Capability struct {
  index uint64
}
```

[CapabilityKeeper]には、永続ストレージキー、メモリストレージキー、および割り当てられたモジュール名のマッピングが含まれています。

```golang
type CapabilityKeeper struct {
  persistentKey StoreKey
  memKey        StoreKey
  capMap        map[uint64]*Capability
  moduleNames   map[string]interface{}
  sealed        bool
}
```

`CapabilityKeeper`は、*スコープ*サブマネージャーを作成する機能を提供します。
特定のモジュール名。 これらの `ScopedCapabilityKeeper`は、アプリケーションの初期化時に作成する必要があります
そしてモジュールに渡されると、モジュールはそれらを使用して、受信および取得する関数を宣言できます
新しい機能と認証機能を作成することに加えて、それらが名前で所有する機能
他のモジュールを介して。 

```golang
type ScopedCapabilityKeeper struct {
  persistentKey StoreKey
  memKey        StoreKey
  capMap        map[uint64]*Capability
  moduleName    string
}
```

`ScopeToModule`は、一意である必要がある特定の名前でスコープサブマネージャーを作成するために使用されます。
`InitialiseAndSeal`の前に呼び出す必要があります。 

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

`InitialiseAndSeal`は初期状態をロードし、すべてを作成する必要があります
新しく作成されたメモリで埋めるために必要な[ScopedCapabilityKeeper]
特定のモジュールによって以前に宣言されたキーの機能キーに従って、
新しい[ScopedCapabilityKeeper]を作成します。

```golang
func (ck CapabilityKeeper) InitialiseAndSeal(ctx Context) {
  if ck.sealed {
    panic("capability keeper is sealed")
  }

  persistentStore := ctx.KVStore(ck.persistentKey)
  map := ctx.KVStore(ck.memKey)
  
./initialise memory store for all names in persistent store
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

どのモジュールも[NewCapability]を呼び出して、新しい独自の偽造不可能なオブジェクト機能を作成できます
参照する。 新しく作成された機能は自動的に保持されます。モジュールを呼び出す必要はありません
`ClaimCapability`を呼び出します。 

```golang
func (sck ScopedCapabilityKeeper) NewCapability(ctx Context, name string) (Capability, error) {
./check name not taken in memory store
  if capStore.Get("rev/" + name) != nil {
    return nil, errors.New("name already taken")
  }

./fetch the current index
  index := persistentStore.Get("index")
  
./create a new capability
  capability := &CapabilityKey{index: index}
  
./set persistent store
  persistentStore.Set(index, Set.singleton(sck.moduleName + "/" + name))
  
./update the index
  index++
  persistentStore.Set("index", index)
  
./set forward mapping in memory store from capability to name
  memStore.Set(sck.moduleName + "/fwd/" + capability, name)
  
./set reverse mapping in memory store from name to index
  memStore.Set(sck.moduleName + "/rev/" + name, index)

./set the in-memory mapping from index to capability pointer
  capMap[index] = capability
  
./return the newly created capability
  return capability
}
```

どのモジュールでもAuthenticateCapabilityを呼び出して、機能を確認できます
実際には特定の名前に対応しています(名前は信頼できないユーザー入力である可能性があります)
モジュールを呼び出す前に、モジュールに関連付けられています。 

```golang
func (sck ScopedCapabilityKeeper) AuthenticateCapability(name string, capability Capability) bool {
./return whether forward mapping in memory store matches name
  return memStore.Get(sck.moduleName + "/fwd/" + capability) === name
}
```

`ClaimCapability`を使用すると、モジュールは別のモジュールから受け取った機能キーを宣言できます
その後、将来の `GetCapability`呼び出しは成功します。

受信機能モジュールが名前でアクセスする場合は、 `ClaimCapability`を呼び出す必要があります
将来。 機能は複数所有者であるため、複数のモジュールに単一の[機能]参照がある場合、
彼らは皆それを持っているでしょう。 

```golang
func (sck ScopedCapabilityKeeper) ClaimCapability(ctx Context, capability Capability, name string) error {
  persistentStore := ctx.KVStore(sck.persistentKey)

./set forward mapping in memory store from capability to name
  memStore.Set(sck.moduleName + "/fwd/" + capability, name)

./set reverse mapping in memory store from name to capability
  memStore.Set(sck.moduleName + "/rev/" + name, capability)

./update owner set in persistent store
  owners := persistentStore.Get(capability.Index())
  owners.add(sck.moduleName + "/" + name)
  persistentStore.Set(capability.Index(), owners)
}
```

`GetCapability`を使用すると、モジュールは以前に宣言された機能を名前で取得できます。
モジュールは、モジュールに属していない関数を取得することはできません。 

```golang
func (sck ScopedCapabilityKeeper) GetCapability(ctx Context, name string) (Capability, error) {
./fetch the index of capability using reverse mapping in memstore
  index := memStore.Get(sck.moduleName + "/rev/" + name)

./fetch capability from go-map using index
  capability := capMap[index]

./return the capability
  return capability
}
```

`ReleaseCapability`を使用すると、モジュールは以前に宣言された機能を解放できます。 そうでない場合
所有者がさらにいる場合、その能力はグローバルに削除されます。 

```golang
func (sck ScopedCapabilityKeeper) ReleaseCapability(ctx Context, capability Capability) err {
  persistentStore := ctx.KVStore(sck.persistentKey)

  name := capStore.Get(sck.moduleName + "/fwd/" + capability)
  if name == nil {
    return error("capability not owned by module")
  }

./delete forward mapping in memory store
  memoryStore.Delete(sck.moduleName + "/fwd/" + capability, name)

./delete reverse mapping in memory store
  memoryStore.Delete(sck.moduleName + "/rev/" + name, capability)

./update owner set in persistent store
  owners := persistentStore.Get(capability.Index())
  owners.remove(sck.moduleName + "/" + name)
  if owners.size() > 0 {
  ./there are still other owners, keep the capability around
    persistentStore.Set(capability.Index(), owners)
  } else {
  ./no more owners, delete the capability
    persistentStore.Delete(capability.Index())
    delete(capMap[capability.Index()])
  }
}
```

### 使用モード

#### 初期化

動的関数を使用するモジュールは、app.goでScopedCapabilityKeeperを提供する必要があります。

```golang
ck := NewCapabilityKeeper(persistentKey, memoryKey)
mod1Keeper := NewMod1Keeper(ck.ScopeToModule("mod1"), ....)
mod2Keeper := NewMod2Keeper(ck.ScopeToModule("mod2"), ....)

//other initialisation logic ...

//load initial state...

ck.InitialiseAndSeal(initialContext)
```

#### 機能の作成、転送、宣言、使用

[mod1]が能力を作成し、それを名前でリソース(IBCチャネルなど)に関連付けてから、それを[mod2]に渡し、後でそれを使用する状況を考えてみます。

モジュール1には次のコードが含まれます。 

```golang
capability := scopedCapabilityKeeper.NewCapability(ctx, "resourceABC")
mod2Keeper.SomeFunction(ctx, capability, args...)
```

`SomeFunction`は、モジュール2で実行され、関数を宣言できます。 

```golang
func (k Mod2Keeper) SomeFunction(ctx Context, capability Capability) {
  k.sck.ClaimCapability(ctx, capability, "resourceABC")
./other logic...
}
```

後で、モジュール2は名前で関数を取得し、それをモジュール1に渡すことができます。モジュール1は、リソースに基づいて関数を認証します。 

```golang
func (k Mod2Keeper) SomeOtherFunction(ctx Context, name string) {
  capability := k.sck.GetCapability(ctx, name)
  mod1.UseResource(ctx, capability, "resourceABC")
}
```

モジュール2にリソースの使用を許可する前に、モジュール1は、この機能キーがリソースを使用するために認証されているかどうかを確認します。

```golang
func (k Mod1Keeper) UseResource(ctx Context, capability Capability, resource string) {
  if !k.sck.AuthenticateCapability(name, capability) {
    return errors.New("unauthenticated")
  }
./do something with the resource
}
```

モジュール2がファンクションキーをモジュール3に渡すと、モジュール3はそれを宣言し、モジュール2のようにモジュール1を呼び出すことができます。
(この場合、モジュール1、モジュール2、およびモジュール3はすべてこの機能を使用できます)。

## ステータス

提案しました。

## 結果

### 目的

-動的機能のサポート。
-txが失敗したときに、メモリ内の永続的な[KVStore]および[MemoryStore]への書き込みを回復しながら、CapabilityKeeperがgo-mapから同じ機能ポインタを返すことを許可します。

### ネガティブ

-追加のゴールキーパーが必要です。
-既存の[StoreKey]システムとの重複があります(これはスーパーセット関数であるため、将来的にマージできます)。
-メモリストアはインデックスにマップする必要があり、実際の関数を取得するためにgoマップのキーとして使用する必要があるため、リバースマッピングでは追加レベルの間接参照が必要です。

### ニュートラル

(誰も知らない)

## 参照する

-[元のディスカッション](https://github.com/cosmos/cosmos-sdk/pull/5230#discussion_r343978513) 