# ADR 012:ステータスアクセサー

## 変更ログ

-2019年9月4日:最初のドラフト

## 環境

Cosmos SDKモジュールは現在、 `KVStore`インターフェースと` Codec`を使用してそれぞれの状態にアクセスしています。でも
これは、モジュール開発者に多くの自由を提供し、モジュール化が困難であり、UX
平凡。

まず、モジュールが状態にアクセスしようとするたびに、値をマーシャリングし、設定または取得する必要があります
値を付け、最後にグループ化を解除します。通常、これは `Keeper.GetXXX`関数と` Keeper.SetXXX`関数を宣言することによって行われます。
それらは反復的であり、維持するのが困難です。

次に、これにより、オブジェクト機能の定理であるアクセスとの整合がより困難になります。
状態は「StoreKey」として定義され、Merkleツリー全体へのフルアクセスを提供するため、モジュールはできません
特定のキーと値のペア(またはキーと値のペアのセット)のアクセス権を別のモジュールに安全に送信します。

最後に、getter/setter関数は、モジュールの `Keeper`のメソッドとして定義されているため、レビュー担当者は
状態の任意の部分にアクセスする関数を見るときは、マークルツリー空間全体を考慮する必要があります。
関数がアクセスしている状態の部分(アクセスしていない部分)を静的に知る方法はありません。

## 決断

「値」と呼ばれるタイプを定義します。 

```go
type Value struct {
  m   Mapping
  key []byte
}
```

`Value` 用作状态中键值对的引用，其中 `Value.m` 定义键值
它将访问的空间和 `Value.key` 定义引用的确切键。

我们将定义一个名为 `Mapping` 的类型: 

```go
type Mapping struct {
  storeKey sdk.StoreKey
  cdc      *codec.LegacyAmino
  prefix   []byte
}
```

`Mapping` 用作状态中键值空间的引用，其中 `Mapping.storeKey` 定义
IAVL(子)树和`Mapping.prefix` 定义了可选的子空间前缀。

我们将为 `Value` 类型定义以下核心方法: 

```go
//Get and unmarshal stored data, noop if not exists, panic if cannot unmarshal
func (Value) Get(ctx Context, ptr interface{}) {}

//Get and unmarshal stored data, return error if not exists or cannot unmarshal
func (Value) GetSafe(ctx Context, ptr interface{}) {}

//Get stored data as raw byte slice
func (Value) GetRaw(ctx Context) []byte {}

//Marshal and set a raw value
func (Value) Set(ctx Context, o interface{}) {}

//Check if a raw value exists
func (Value) Exists(ctx Context) bool {}

//Delete a raw value value
func (Value) Delete(ctx Context) {}
```

We will define the following core methods for the `Mapping` type:

```go
//Constructs key-value pair reference corresponding to the key argument in the Mapping space
func (Mapping) Value(key []byte) Value {}

//Get and unmarshal stored data, noop if not exists, panic if cannot unmarshal
func (Mapping) Get(ctx Context, key []byte, ptr interface{}) {}

//Get and unmarshal stored data, return error if not exists or cannot unmarshal
func (Mapping) GetSafe(ctx Context, key []byte, ptr interface{})

//Get stored data as raw byte slice
func (Mapping) GetRaw(ctx Context, key []byte) []byte {}

//Marshal and set a raw value
func (Mapping) Set(ctx Context, key []byte, o interface{}) {}

//Check if a raw value exists
func (Mapping) Has(ctx Context, key []byte) bool {}

//Delete a raw value value
func (Mapping) Delete(ctx Context, key []byte) {}
```

パラメータ `ctx`、` key`、および `args ...`を渡す `Mapping`タイプの各メソッドはプロキシします
パラメータctxおよびargsを使用したMapping.Value(key)の呼び出し...

さらに、 `Value`型から派生したジェネリック型のセットを定義して提供します。 

```go
type Boolean struct { Value }
type Enum struct { Value }
type Integer struct { Value; enc IntEncoding }
type String struct { Value }
//...
```

エンコード方式が異なる場合は、コアメソッドの `o`パラメーターを入力し、` ptr`パラメーターを入力します。
コアメソッドの明示的な戻り型に置き換えられました。

最後に、マッピングタイプから派生した一連のタイプを定義します。 

```go
type Indexer struct {
  m   Mapping
  enc IntEncoding
}
```

コアメソッドの「key」パラメータの場所を入力します。

アクセサタイプのいくつかのプロパティは次のとおりです。

-状態アクセスは、パラメータとして `Context`を使用して関数を呼び出す場合にのみ発生します
-アクセサータイプの構造体は、構造体によって参照される状態へのアクセスのみを許可し、他のアクセス許可は許可しません
-マーシャリング/アンマーシャリングはコアメソッドで暗黙的に発生します

## 状態

提案

## 結果

### ポジティブ

-シリアル化は自動的に行われます
-コードサイズが短く、定型文が少なく、ユーザーエクスペリエンスが向上しています
-状態への参照を安全に転送できます
-アクセス範囲を明確にする

### ネガティブ

-シリアル化は自動的に行われます
-コードサイズが短く、定型文が少なく、ユーザーエクスペリエンスが向上しています
-状態への参照を安全に転送できます
-アクセス範囲を明確にする

### ニュートラル

## 参照

-[#4554](https://github.com/cosmos/cosmos-sdk/issues/4554) 