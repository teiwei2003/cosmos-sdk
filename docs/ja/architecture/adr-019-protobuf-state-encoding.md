# ADR 019:プロトコルバッファステータスコード

## 変更ログ

-2020年2月15日:最初のドラフト
-2020年2月24日:インターフェースフィールドを使用したメッセージの処理を更新
-2020年4月27日:インターフェースの `oneof`の使用法を` Any`に変換します
-2020年5月15日:[cosmos_proto]拡張機能とアミノ互換性について説明する
-2020年12月4日:[MarshalAny]と[UnmarshalAny]を[codec.Codec]インターフェースに移動して名前を変更します。
-2021年2月24日:[#6843](https://github.com/cosmos/cosmos-sdk/pull/6843)で放棄された[HybridCodec]の言及を削除します。

## 状態

受け入れられました

## 環境

現在、Cosmos SDKはバイナリに[go-amino](https://github.com/tendermint/go-amino/)を使用しています
また、JSONオブジェクトエンコーディングは、ワイヤーを介して論理オブジェクトと永続オブジェクトの間に同等性をもたらします。

アミノのドキュメントから:

> Aminoはオブジェクトエンコーディング仕様です。これは、インターフェイス拡張機能を備えたProto3のサブセットです。
>サポート。詳細については、[Proto3仕様](https://developers.google.com/protocol-buffers/docs/proto3)を参照してください。
> Proto3に関する情報に関しては、Aminoはほぼ互換性があります(Proto2は互換性がありません)。
>>
> Aminoエンコーディングプロトコルの目標は、論理オブジェクトと永続オブジェクトにパリティを導入することです。

Aminoは、次の目標の達成も目​​指しています(完全なリストではありません)。

-バイナリバイトはパターンデコード可能である必要があります。
-アーキテクチャはアップグレード可能である必要があります。
-エンコーダーとデコーダーのロジックはかなり単純でなければなりません。

しかし、アミノはこれらの目標を完全には達成しておらず、完全に満足しているわけでもありません。
CosmosSDKの非常に柔軟なクロスランゲージおよびマルチクライアント互換のエンコーディングプロトコル要件。
言い換えれば、Aminoは、クロスオブジェクトのシリアル化をサポートする上で大きな問題点であることが証明されています。
クライアントはさまざまな言語で書かれており、実際のフォールバックはほとんどありません。
互換性とアップグレード性。さらに、分析とさまざまなベンチマークテストを通じて、Aminoは
これは、Cosmos SDK <sup> 1 <.sup>の非常に大きなパフォーマンスのボトルネックであることが判明しました。これは
主にシミュレーションパフォーマンスとアプリケーショントランザクションスループットに反映されます。

したがって、次の州のシリアル化標準を満たすエンコーディングプロトコルを採用する必要があります。

-言語不可知論
-プラットフォームにとらわれない
-豊富なカスタマーサポートと繁栄するエコシステム
-ハイパフォーマンス
-エンコードされたメッセージの最小サイズ
-リフレクションではなくコード生成に基づく
-下位互換性と上位互換性をサポートします

Aminoからの移行は、状態とクライアントのコーディングという2つのアプローチと見なす必要があることに注意してください。
このADRは、CosmosSDKステートマシンでの状態のシリアル化に焦点を当てています。対応するADRは
クライアントコーディングを解決するために使用されます。

## 決定

シリアル化には[ProtocolBuffers](https://developers.google.com/protocol-buffers)を使用します
Cosmos SDKで構造化データを永続化すると同時に、
Aminoのアプリケーションを引き続き使用したいと考えています。モジュールを更新することにより、このメカニズムを提供します
特定のAminoコーデックではなく、コーデックインターフェイス `Marshaler`を受け入れます。さらに、Cosmos SDK
`Marshaler`インターフェースの2つの具体的な実装が提供されます:` AminoCodec`と `ProtoCodec`。

-`AminoCodec`:バイナリおよびJSONエンコーディングにAminoを使用します。
-`ProtoCodec`:バイナリおよびJSONエンコーディングにProtobufを使用します。

モジュールは、アプリケーションでインスタンス化されたコーデックを使用します。デフォルトでは、CosmosSDKの `simapp`
`Marshaler`の具体的な実装として、` MakeTestEncodingConfig`で `ProtoCodec`をインスタンス化します
特徴。必要に応じて、これはアプリケーション開発者が簡単に上書きできます。

最終的な目標は、AminoJSONエンコーディングをProtobufエンコーディングに置き換えることです。
モジュールは `ProtoCodec`を受け入れたり拡張したりします。それまでは、AminoJSONはレガシーユースケースで引き続き利用できます。
Cosmos SDKのいくつかの場所は、Legacy APIRESTエンドポイントなどのAminoJSONを使用してハードコーディングされています
そして `x .params`ストレージ。彼らは徐々にProtobufに変換することを計画しています。

###モジュールコーデック

インターフェイスモジュール、Protobufパスを処理およびシリアル化できる必要はありません
移行は非常に簡単です。これらのモジュールは、既存のタイプを単純に移行するためのものです
Protobufを特定のAminoコーデックでエンコードして永続化し、管理者に受け入れさせます
`Marshaler`は` ProtoCodec`になります。物事はそのまま実行されるため、この移行は簡単です。

基本型([bool]や[int64]など)をエンコードする必要があるビジネスロジックを使用する必要があることに注意してください
[gogoprotobuf](https://github.com/gogo/protobuf)値のタイプ。

例: 

```go
  ts, err := gogotypes.TimestampProto(completionTime)
  if err != nil {
   .....
  }

  bz := cdc.MustMarshal(ts)
```

ただし、モジュールの目的と設計は大きく異なる可能性があるため、モジュールの機能をサポートする必要があります
インターフェイス( `Account`や` Content`など)をコーディングして使用する機能。 これらのモジュールの場合、
`Marshaler`を拡張するには、独自のコーデックインターフェイスを定義する必要があります。 これらの特定のインターフェースは一意です
モジュールに、必要なインターフェースをシリアル化する方法を知っているメソッドコントラクトが含まれます。

例: 

```go
/.x/auth/types/codec.go

type Codec interface {
  codec.Codec

  MarshalAccount(acc exported.Account) ([]byte, error)
  UnmarshalAccount(bz []byte) (exported.Account, error)

  MarshalAccountJSON(acc exported.Account) ([]byte, error)
  UnmarshalAccountJSON(bz []byte) (exported.Account, error)
}
```

### `Any`を使用してインターフェースをエンコードします

通常、モジュールレベルの.protoファイルは、インターフェイスをエンコードするメッセージを定義する必要があります
[`google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto)を使用します。
[拡張ディスカッション]後(https://github.com/cosmos/cosmos-sdk/issues/6030)、
これは、アプリケーションレベルの[oneof]の推奨される代替手段として選択されました。
オリジナルのprotobufデザインと同じように。 [Any]の引数は次のようになります
次のように要約します。

* `Any`は、よりシンプルで一貫性のあるクライアントユーザーエクスペリエンスを提供します
アプリケーションレベルの `oneof`よりも多くの調整されたインターフェースが必要
アプリケーションを慎重にクロスします。一般的なトランザクションを作成する
`oneof`署名ライブラリの使用は面倒であり、重要なロジックが必要になる場合があります
各チェーンの再実装
* `Any`は` oneof`よりもヒューマンエラーに対して耐性があります
* `Any`は通常、モジュールとアプリケーションに実装する方が簡単です

[Any]を使用することへの主な反対意見は、その余分なスペースを中心に展開します
そして、可能なパフォーマンスのオーバーヘッド。処理スペースのオーバーヘッド
将来の永続層の圧縮とパフォーマンスへの影響
小さいかもしれません。したがって、 `Any`を使用しないことは時期尚早の最適化のようです。
ユーザーエクスペリエンスをより高いレベルの懸念事項と見なします。

Cosmos SDKは、説明されている[コーデック]インターフェイスを採用することを決定したため、注意してください
上記では、アプリケーションは引き続き `oneof`を使用してステータスとトランザクションをエンコードすることを選択できます
ただし、これは推奨される方法ではありません。アプリケーションが `oneof`の使用を選択した場合
[any]の代わりに、クライアントアプリケーションとの互換性が失われる可能性があります
複数のチェーンをサポートします。したがって、開発者は慎重に検討する必要があります
彼らは、時期尚早の最適化やエンドユーザーの可能性についてより懸念しています
そしてクライアント開発者のUX。

### `Any`の安全な使用

デフォルトでは、[gogoprotobufは `Any`を実装します](https://godoc.org/github.com/gogo/protobuf/types)
[グローバルタイプ登録](https://github.com/gogo/protobuf/blob/master/proto/properties.go#L540)を使用します
[Any]にパックされた値を具象にデコードします
タイプに移動します。これにより、悪意のあるモジュールが存在する脆弱性が発生します
グローバルprotobufレジストリを使用して、依存関係ツリーにタイプを登録できます
そして、参照されるトランザクションをロードしてマーシャリング解除します
これは `type_url`フィールドにあります。

これを防ぐために、[Any]をデコードするための型登録メカニズムを導入しました
`InterfaceRegistry`インターフェースを介して値を具象型に変換します
Aminoの型登録にはいくつかの類似点があります。 

```go
type InterfaceRegistry interface {
   ..RegisterInterface associates protoName as the public name for the
   ..interface passed in as iface
   ..Ex:
   ..  registry.RegisterInterface("cosmos_sdk.Msg", (*sdk.Msg)(nil))
    RegisterInterface(protoName string, iface interface{})

   ..RegisterImplementations registers impls as a concrete implementations of
   ..the interface iface
   ..Ex:
   .. registry.RegisterImplementations((*sdk.Msg)(nil), &MsgSend{}, &MsgMultiSend{})
    RegisterImplementations(iface interface{}, impls ...proto.Message)

}
```

ホワイトリストであることに加えて、InterfaceRegistryは次のように使用することもできます
クライアントインターフェイスを満たす特定のタイプのリストを伝達します。

.protoファイル内:

*インターフェースを受け入れるフィールドには、 `cosmos_proto.accepts_interface`の注釈を付ける必要があります
`protoName`と同じ完全修飾名を使用して、` InterfaceRegistry.RegisterInterface`に渡します
*インターフェースの実装には `cosmos_proto.implements_interface`の注釈を付ける必要があります
`protoName`と同じ完全修飾名を使用して、` InterfaceRegistry.RegisterInterface`に渡します

将来的には、 `protoName`、` cosmos_proto.accepts_interface`、 `cosmos_proto.implements_interface`
これは、コード生成、リフレクション、および/または静的リンティングを通じて使用できます。

`InterfaceRegistry`を実装する同じ構造は、
インターフェイス `InterfaceUnpacker`は、` Any`を解凍するために使用されます。 

```go
type InterfaceUnpacker interface {
   ..UnpackAny unpacks the value in any to the interface pointer passed in as
   ..iface. Note that the type in any must have been registered with
   ..RegisterImplementations as a concrete type for that interface
   ..Ex:
   ..   var msg sdk.Msg
   ..   err := ctx.UnpackAny(any, &msg)
   ..   ...
    UnpackAny(any *Any, iface interface{}) error
}
```

`InterfaceRegistry`の使用法は標準のprotobufから逸脱しないことに注意してください
`Any`の使用法、それはただのためです
golangの使用法。

`InterfaceRegistry`は` ProtoCodec`のメンバーになります
上記のように。 モジュールがインターフェースタイプを登録するために、アプリケーションモジュール
次のインターフェースを実装することを選択できます。 

```go
type InterfaceModule interface {
    RegisterInterfaceTypes(InterfaceRegistry)
}
```

モジュールマネージャーには、 `RegisterInterfaceTypes`を呼び出すメソッドが含まれます
それを実装するすべてのモジュールは、 `InterfaceRegistry`にデータを入力します。

### `Any`を使用して状態をエンコードします

Cosmos SDKは、サポートメソッド `MarshalInterface`と` UnmarshalInterface`を提供して、インターフェイスタイプを `Any`にラップする複雑さを隠し、簡単にシリアル化できるようにします。 

```go
import "github.com/cosmos/cosmos-sdk/codec"

/.note: eviexported.Evidence is an interface type
func MarshalEvidence(cdc codec.BinaryCodec, e eviexported.Evidence) ([]byte, error) {
	return cdc.MarshalInterface(e)
}

func UnmarshalEvidence(cdc codec.BinaryCodec, bz []byte) (eviexported.Evidence, error) {
	var evi eviexported.Evidence
	err := cdc.UnmarshalInterface(&evi, bz)
    return err, nil
}
```

### `sdk.Msg`sで` Any`を使用する

同様の概念が、インターフェースフィールドを含むメッセージにも当てはまります。
たとえば、 `MsgSubmitEvidence`を次のように定義できます。ここで` Evidence`は
インターフェース: 

```protobuf
/.x/evidence/types/types.proto

message MsgSubmitEvidence {
  bytes submitter = 1
    [
      (gogoproto.casttype) = "github.com/cosmos/cosmos-sdk/types.AccAddress"
    ];
  google.protobuf.Any evidence = 2;
}
```

`Any`から証拠を抽出するには、引用する必要があることに注意してください
`インターフェース登録`。 メソッド内の証拠を参照するために、例えば
`ValidateBasic`は` InterfaceRegistry`を知らないはずです。
逆シリアル化して解凍するための `UnpackInterfaces`ステージを導入します
インターフェースは必要になる前にあります。

### インターフェースを解凍します

解凍と逆シリアル化を実装するための `UnpackInterfaces`ステージ
`Any`でラップされたインターフェースが必要になる前に、インターフェースを作成します
`sdk.Msg`sおよびその他のタイプを実装できます。 

```go
type UnpackInterfacesMessage interface {
  UnpackInterfaces(InterfaceUnpacker) error
}
```

また、 `Any`にプライベート` cachedValue interface {} `フィールドを導入しました
パブリックゲッター `GetCachedValue()interface {}`を使用してそれ自体を構築します。

`UnpackInterfaces`メソッドは、メッセージの逆シリアル化中に呼び出されます
`Unmarshal`にあり、` Any`にカプセル化されているインターフェイス値はすべてデコードされます
そして、将来の参照のためにそれを `cachedValue`に保存します。

その後、解凍されたインターフェイス値を任意のコードで安全に使用できます
`InterfaceRegistry`がわからない
そして、メッセージは、キャッシュされた値をに変換するための簡単なゲッターを紹介することができます
正しいインターフェイスタイプ。

これには、 `Any`値のアンマーシャリングが1回だけ発生するという追加の利点があります
値が読み取られるたびではなく、最初の逆シリアル化中。 また、
Any値が初めてパックされたとき(例:
`NewMsgSubmitEvidence`)を使用して、元のインターフェイス値をキャッシュし、
もう一度読むには、アンマーシャリングは必要ありません。

`MsgSubmitEvidence`は` UnpackInterfaces`と便利なゲッターを実装できます
`GetEvidence`は次のとおりです。  

```go
func (msg MsgSubmitEvidence) UnpackInterfaces(ctx sdk.InterfaceRegistry) error {
  var evi eviexported.Evidence
  return ctx.UnpackAny(msg.Evidence, *evi)
}

func (msg MsgSubmitEvidence) GetEvidence() eviexported.Evidence {
  return msg.Evidence.GetCachedValue().(eviexported.Evidence)
}
```

### アミノの互換性

使用する場合、カスタムの `Any`実装をAminoで透過的に使用できます
正しいコーデックインスタンスを使用してください。これは、インターフェースがにカプセル化されていることを意味します
`Any`は、通常のAminoインターフェースのようにアミノマーシャリングされます(
Aminoに正しく登録されています)。

この関数を機能させるには:

-**すべてのレガシーコードは、 `* amino.Codec`の代わりに` * codec.LegacyAmino`を使用する必要があります。
  `Any` **を正しく処理できるラッパーになりました
-**すべての新しいコードは、アミノと組み合わせた `Marshaler`を使用する必要があります
  プロトタイプバッファー**
-さらに、v0.39より前では、 `codec.LegacyAmino`は` codec.LegacyAmino`に名前が変更されます。

### Xを選んでみませんか

代替プロトコルとのより完全な比較については、[ここ](https://codeburst.io/json-vs-protocol-buffers-vs-flatbuffers-a4247f8bda6f)を参照してください。

### Cap'n Proto

[Cap'n Proto](https://capnproto.org/)はProtobufの好ましい代替手段のようですが
インターフェイス/ジェネリックスのネイティブサポートと組み込みの正規化のために、それは本当に欠けています
Protobufと比較すると、リッチクライアントエコシステムは十分に成熟していません。

### FlatBuffers

[FlatBuffers](https://google.github.io/flatbuffers/)も、
主な違いは、FlatBuffersは補助的な解析/解凍手順を必要としないことです
データにアクセスする前に、通常、各オブジェクトのメモリ割り当てと組み合わされます。

ただし、これには多くの調査と移行の範囲の完全な理解が必要です。
そして今後の方向性-これはまだあまり明確ではありません。さらに、FlatBuffersは
信頼できない入力。

##将来の改善とロードマップ

将来的には、永続性のすぐ上に圧縮レイヤーを追加することを検討する可能性があります
txまたはMerkelツリーハッシュのレベルを変更しませんが、ストレージを削減します
`Any`のオーバーヘッド。さらに、protobuf命名規則を採用する場合があります
わかりやすくしながら、タイプURLをより簡潔にします。

`Any`の使用を取り巻く追加のコード生成サポートは
Go開発者により多くのユーザーエクスペリエンスを提供するために、将来的に検討することもできます
シームレス。

## 結果

### ポジティブ

-大幅なパフォーマンスの向上。
-後方および前方タイプの互換性をサポートします。
-クロスランゲージクライアントのサポートが向上しました。

### ネガティブ

-Protobufメッセージを理解して実装するために必要な学習曲線。
-`Any`を使用しているため、メッセージサイズはわずかに大きくなりますが、これは相殺できます
   圧縮層

### ニュートラル

## 参照

1. https://github.com/cosmos/cosmos-sdk/issues/4977
2. https://github.com/cosmos/cosmos-sdk/issues/5444 