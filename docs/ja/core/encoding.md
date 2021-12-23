# エンコーディング

Cosmos SDKのエンコーディングは、主に `go-amino`コーデックによって処理されていましたが、Cosmos SDKは、状態およびクライアント側のエンコーディングに` gogoprotobuf`を使用するようになりました。{まとめ}

## 読むための前提条件

-[Cosmos SDKアプリケーション分析](../basics/app-anatomy.md){前提条件}

## エンコーディング

Cosmos SDKは、2つのバイナリワイヤエンコーディングプロトコルを使用します。[Amino](https://github.com/tendermint/go-amino/)はオブジェクトエンコーディング仕様であり、[Protocol Buffers](https://developers.google.com/プロトコル) -buffers)、Proto3のサブセット、に拡張
インターフェイスのサポート。 [Proto3仕様](https://developers.google.com/protocol-buffers/docs/proto3)を参照してください
Proto3の詳細については、Aminoはほぼ互換性があります(ただし、Proto2は互換性がありません)。

Aminoには重大なパフォーマンス上の欠陥があるため、リフレクションに基づいており、
意味のあるクロスランゲージ/クライアントサポート、プロトコルバッファ、特にありません
[gogoprotobuf](https://github.com/gogo/protobuf/)、Aminoの代わりに使用されます。
AminoでProtocolBuffersを使用するこのプロセスは、まだ進行中のプロセスであることに注意してください。

Cosmos SDKのバイナリラインコードのタイプは、主に2つのタイプに分けることができます。
カテゴリ、クライアントコード、ストレージコード。クライアントコーディングは主に中心に展開します
トランザクション処理と署名を中心に展開し、ストレージコーディングは中心に展開します
ステートマシン遷移で使用されるタイプとMerkleに格納される最終コンテンツ
木。

ストレージエンコーディングの場合、protobuf定義は任意のタイプで存在でき、通常は
アミノ基に基づく「中間体」のタイプがあります。具体的には、protobufタイプに基づく
シリアル化と永続性、およびアミノベースのタイプ用に定義されています
ステートマシンのビジネスロジックで使用され、前後に変換できます。
アミノベースのタイプは将来段階的に廃止される可能性があるため、開発者は注意してください
可能な限りprotobufメッセージ定義を使用するように注意する必要があります。

`codec`パッケージには、` Marshaler`と `ProtoMarshaler`の2つのコアインターフェイスがあります。
前者は、現在のAminoインターフェイスをカプセル化します。
ジェネリック `interface {}`型の代わりに後者を実装する型。

さらに、 `Marshaler`には2つの実装があります。最初は
`AminoCodec`。バイナリとJSONの両方のシリアル化がAminoによって処理されます。この
2つ目は `ProtoCodec`で、バイナリとJSONのシリアル化を処理します
Protobuf経由。

これは、モジュールがAminoまたはProtobufエンコーディングを使用できることを意味しますが、タイプは
`ProtoMarshaler`を実装します。モジュールがこのインターフェースの実装を避けたい場合
それらのタイプについては、Aminoコーデックを直接使用できます。

### アミノ

各モジュールは、Aminoコーデックを使用してタイプとインターフェイスをシリアル化します。このコーデックは通常
モジュールのドメインに登録されているのは、タイプとインターフェース(メッセージなど)のみです。
ただし、 `x/gov`のような例外があります。各モジュールは `RegisterLegacyAminoCodec`関数を公開します
ユーザーがコーデックを提供し、すべてのタイプを登録できるようにします。アプリケーション
このメソッドは、必要なすべてのモジュールに対して呼び出されます。

モジュールにprotobufに基づく型定義がない場合(以下を参照)、Amino
生のラインバイトを特定のタイプまたはインターフェイスにエンコードおよびデコードするために使用されます。

```go
bz := keeper.cdc.MustMarshal(typeOrInterface)
keeper.cdc.MustUnmarshal(bz, &typeOrInterface)
```

接頭辞として長さを持つ上記の関数のバリエーションがあることに注意してください。
通常、データをストリーミングまたはグループ化する必要がある場合に使用されます
(例: `ResponseDeliverTx.Data`)

### Gogoproto

モジュールに、それぞれのタイプにProtobufエンコーディングを使用するように促します。 Cosmos SDKでは、Protobuf仕様の[Gogoproto](https://github.com/gogo/protobuf)固有の実装と、公式の[Google protobuf実装](https://github.com/protocolbuffers/protobuf)。

### protobufメッセージ定義ガイド

[公式のプロトコルバッファガイドラインに従う](https://developers.google.com/protocol-buffers/docs/proto3#simple)に加えて、インターフェイスを扱う場合は、.protoファイルで次のコメントを使用することをお勧めします。

-`cosmos_proto.accepts_interface`を使用して、インターフェースを受け入れるフィールドに注釈を付けます
-`protoName`と同じ完全修飾名を `InterfaceRegistry.RegisterInterface`に渡します
-`cosmos_proto.implements_interface`を使用して、インターフェースの実装に注釈を付けます
-`protoName`と同じ完全修飾名を `InterfaceRegistry.RegisterInterface`に渡します

### トランザクションコード

Protobufのもう1つの重要な用途は、エンコードとデコードです。
[トランザクション](./transactions.md)。アプリケーションによるトランザクションまたは
Cosmos SDKですが、その後、基盤となるコンセンサスエンジンに渡され、
他のピア。基盤となるコンセンサスエンジンはアプリケーションとは何の関係もないため、
コンセンサスエンジンは、rawバイトの形式のトランザクションのみを受け入れます。

-`TxEncoder`オブジェクトはエンコードを実行します。
-`TxDecoder`オブジェクトはデコードを実行します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/types/tx_msg.go#L83-L87

これら2つのオブジェクトの標準実装は、[`auth`モジュール](../../x/auth/spec/README.md)にあります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/x/auth/tx/decoder.go

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/x/auth/tx/encoder.go

トランザクションのエンコード方法の詳細については、[ADR-020](../architecture/adr-020-protobuf-transaction-encoding.md)を参照してください。

### インターフェースのコーディングと `Any`の使用

Protobuf DSLは強く型付けされているため、可変型のフィールドを挿入するのは困難です。 [アカウント](../basics/accounts.md)のラッパーとして `Profile`protobufメッセージを作成したいとします。

```proto
message Profile {
 //account is the account associated to a profile.
  cosmos.auth.v1beta1.BaseAccount account = 1;
 //bio is a short description of the account.
  string bio = 4;
}
```

この `Profile`の例では、`account`を `BaseAccount`としてハードコーディングしました。ただし、「BaseVestingAccount」や「ContinuousVestingAccount」など、他にもいくつかの種類の[アトリビューション関連のユーザーアカウント](../../x/auth/spec/05_vesting.md)があります。これらのアカウントはすべて異なりますが、すべて `AccountI`インターフェースを実装しています。これらすべてのタイプのアカウントを許可する「個人プロファイル」をどのように作成しますか。「アカウント」フィールドは「アカウントI」インターフェースを受け入れます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/types/account.go#L307-L330

[ADR-019](../architecture/adr-019-protobuf-state-encoding.md)では、[`Any`](https://github.com/protocolbuffers/protobuf/blob)を使用することが決定されました。/master/src/google/protobuf/any.proto)sインターフェースをprotobufでエンコードします。`Any`には、任意にシリアル化されたバイトメッセージと、グローバル一意識別子として機能し、メッセージのタイプに解決されるURLが含まれます。この戦略により、protobufメッセージに任意のGoタイプをパックできます。新しい「プロファイル」は次のようになります。

```protobuf
message Profile {
 //account is the account associated to a profile.
  google.protobuf.Any account = 1 [
    (cosmos_proto.accepts_interface) = "AccountI";//Asserts that this field only accepts Go types implementing `AccountI`. It is purely informational for now.
  ];
 //bio is a short description of the account.
  string bio = 4;
}
```

構成ファイルにアカウントを追加するには、まず「codectypes.NewAnyWithValue」を使用して、アカウントを「Any」に「パッケージ化」する必要があります。

```go
var myAccount AccountI
myAccount = ...//Can be a BaseAccount, a ContinuousVestingAccount or any struct implementing `AccountI`

//Pack the account into an Any
accAny, err := codectypes.NewAnyWithValue(myAccount)
if err != nil {
  return nil, err
}

//Create a new Profile with the any.
profile := Profile {
  Account: accAny,
  Bio: "some bio",
}

//We can then marshal the profile as usual.
bz, err := cdc.Marshal(profile)
jsonBz, err := cdc.MarshalJSON(profile)
```

全体として、インターフェースをエンコードするには、1/インターフェースを `Any`にパックし、2/` Any`をマーシャリングする必要があります。 便宜上、Cosmos SDKには、これら2つのステップをバンドルするための `MarshalInterface`メソッドが用意されています。 [x/authモジュールの実際の例](https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/auth/keeper/keeper.go#L218-L221)をご覧ください。 。

`Any`の内部から特定のGoタイプを取得する逆の操作は「アンパック」と呼ばれ、` Any`の `GetCachedValue()`によって実行されます。 

```go
profileBz := ...//The proto-encoded bytes of a Profile, e.g. retrieved through gRPC.
var myProfile Profile
//Unmarshal the bytes into the myProfile struct.
err := cdc.Unmarshal(profilebz, &myProfile)

//Let's see the types of the Account field.
fmt.Printf("%T\n", myProfile.Account)                 //Prints "Any"
fmt.Printf("%T\n", myProfile.Account.GetCachedValue())//Prints "BaseAccount", "ContinuousVestingAccount" or whatever was initially packed in the Any.

//Get the address of the accountt.
accAddr := myProfile.Account.GetCachedValue().(AccountI).GetAddress()
```

`GetCachedValue()`が機能するには、 `Profile`(および` Profile`を埋め込むその他の構造)が `UnpackInterfaces`メソッドを実装する必要があることに注意してください。 

```go
func (p *Profile) UnpackInterfaces(unpacker codectypes.AnyUnpacker) error {
  if p.Account != nil {
    var account AccountI
    return unpacker.UnpackAny(p.Account, &account)
  }

  return nil
}
```

`UnpackInterfaces`は、このメソッドを実装するすべての構造体で再帰的に呼び出され、すべての` Any`が `GetCachedValue()`を正しく埋めることができるようにします。

インターフェイスエンコーディングの詳細、特に `UnpackInterfaces`と、` InterfaceRegistry`を使用して `Any`の` type_url`を解析する方法については、[ADR-019](../architecture/adr-019-protobuf-state)を参照してください。 --encoding.md)。

#### CosmosSDKでの `Any`エンコーディング

上記の「プロファイル」の例は、教育目的の架空の例です。 Cosmos SDKでは、いくつかの場所で `Any`エンコーディングを使用しています(網羅的ではないリスト)。

-さまざまなタイプの公開鍵をエンコードするための `cryptotypes.PubKey`インターフェース、
-トランザクションでさまざまな `Msg`をエンコードするために使用される` sdk.Msg`インターフェース
-x/authクエリ応答でさまざまなタイプのアカウント(上記の例と同様)をエンコードするために使用される `AccountI`インターフェイス、
-x/evidenceモジュールでさまざまなタイプの証拠をエンコードするための `Evidencei`インターフェース
-さまざまなタイプのx/authz認証をエンコードするための `AuthorizationI`インターフェース、
-[`Validator`](https://github.com/cosmos/cosmos-sdk/blob/v0.42.5/x/staking/types/staking.pb.go#L306-L337)構造には関連するバリデーターが含まれています。

次の例は、x/stakeingのValidator構造で公開鍵を「Any」としてエンコードする実際の例を示しています。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.42.1/x/staking/types/validator.go#L40-L61

## よくある質問

1. protobufコーディングを使用してモジュールを作成するにはどうすればよいですか？

**モジュールタイプを定義します**

エンコードするProtobufタイプを定義できます。

- 州
-[`Msg`s](../building-modules/messages-and-queries.md#messages)
-[クエリサービス](../building-modules/query-services.md)
-[创世](../building-modules/genesis.md)

**命名と慣習**

開発者は業界のガイドラインに従うことをお勧めします:[Protocol Buffersスタイルガイド](https://developers.google.com/protocol-buffers/docs/style)
また、[Buf](https://buf.build/docs/style-guide)の詳細については、[ADR 023](../architecture/adr-023-protobuf-naming.md)を参照してください。

2. モジュールをprotobufエンコーディングに更新するにはどうすればよいですか？

モジュールにインターフェース( `Account`や` Content`など)が含まれていない場合、それらは
既存のタイプを簡単に移行できます
特定のAminoコーデックを介してProtobufにエンコードして永続化し(詳細なガイダンスについては、1を参照)、「ProtoCodec」を介して実装されたコーデックとして「Marshaler」を受け入れます。
これ以上のカスタマイズは必要ありません。

ただし、モジュールタイプがインターフェイスを構成する場合は、タイプ `skd.Any`(`/types`パッケージから)でラップする必要があります。このため、モジュールレベルの.protoファイルはそれぞれのメッセージタイプ[`google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any .proto)インターフェイスタイプ。

たとえば、 `Evidence`インターフェースは` MsgSubmitEvidence`によって使用される `x/evidence`モジュールで定義されます。構造体定義では、 `sdk.Any`を使用して証拠ファイルをパッケージ化する必要があります。 protoファイルで次のように定義します。

```protobuf
//proto/cosmos/evidence/v1beta1/tx.proto

message MsgSubmitEvidence {
  string              submitter = 1;
  google.protobuf.Any evidence  = 2 [(cosmos_proto.accepts_interface) = "Evidence"];
}
```

Cosmos SDKの `codec.Codec`インターフェイスは、状態を` Any`として簡単にエンコードするためのサポートメソッド `MarshalInterface`と` UnmarshalInterface`を提供します。

モジュールは `InterfaceRegistry`を使用してインターフェースを登録する必要があります。これにより、インターフェースを登録するためのメカニズムが提供されます:` RegisterInterface(protoName string、iface interface {}) `および実装:` RegisterImplementations(iface interface {}、impls ... proto .Message) `タイプ登録にAminoを使用するのと同様に、Anyから安全に抽出できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc4/codec/types/interface_registry.go#L25-L66

さらに、インターフェイスが必要になる前にアンパックするために、逆シリアル化に「UnpackInterfaces」フェーズを導入する必要があります。 protobuf `Any`を直接またはそのメンバーの1つを介して含むProtobufタイプは、` UnpackInterfacesMessage`インターフェースを実装する必要があります。

```go
type UnpackInterfacesMessage interface {
  UnpackInterfaces(InterfaceUnpacker) error
}
```

## 次へ{非表示}

[gRPC、REST、その他のエンドポイント](./grpc_rest.md){hide}を理解する