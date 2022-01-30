# ADR 031:Protobufメッセージサービス

## 変更ログ

-2020-10-05:最初のドラフト
-2021-04-21:Protobufの `Any`仕様に従うように` ServiceMsg`を削除します。[#9063](https://github.com/cosmos/cosmos-sdk/issues/9063)を参照してください。

## 状態

受け入れられました

## 概要

protobufの `service`定義を使用して` Msg`を定義し、重要な開発者UXを提供したいと考えています。
生成されたコードの改善と、戻り型が明確に定義されるようになりました。

## 環境

現在、CosmosSDKの `Msg`ハンドラーには、応答の` data`フィールドに配置された戻り値があります。
ただし、これらの戻り値は、golangハンドラーコード以外の場所では指定されていません。

初期の会話で[誰かが提案しました](https://docs.google.com/document/d/1eEgYgvgZqLE45vETjhwIw4VOqK-5hwQtZtjVbiXnIGc/edit)
protobuf拡張フィールドを使用して、 `Msg`リターンタイプをキャプチャします。例: 
```protobuf
package cosmos.gov;

message MsgSubmitProposal
	option (cosmos_proto.msg_return) = “uint64”;
	string delegator_address = 1;
	string validator_address = 2;
	repeated sdk.Coin amount = 3;
}
```

しかし、これは決して採用されませんでした。

`Msg`に明確な戻り値を指定すると、クライアントのユーザーエクスペリエンスが向上します。例えば、
`x .gov`では、` MsgSubmitProposal`はプロポーザルIDをビッグエンディアンの `uint64`として返します。
これは実際にはどこにも記録されていません。クライアントは内部構造を理解する必要があります
Cosmos SDKは値を解析し、ユーザーに返します。

さらに、場合によっては、これらの戻り値をプログラムで使用したいことがあります。
たとえば、https://github.com/cosmos/cosmos-sdk/issues/7093はメソッドを提案します
モジュール間Ocapsには `Msg`ルーターを使用します。明確に定義されたリターンタイプは
このメソッドの開発者ユーザーエクスペリエンスを向上させます。

さらに、タイプ `Msg`のハンドラーの登録が増える傾向があります
ホルダー上部のテンプレートは、通常、手動タイプのスイッチで完成します。
これは必ずしも悪いことではありませんが、モジュールの作成コストが増加します。

## 決定

protobufの `service`定義を使用して` Msg`と
それらが生成するコードは、 `Msg`ハンドラーの代わりとして機能します。

以下では、これが `x .gov`モジュールから` SubmitProposal`メッセージを見つける方法を定義します。
`Msg``service`の定義から始めます。 

```proto
package cosmos.gov;

service Msg {
  rpc SubmitProposal(MsgSubmitProposal) returns (MsgSubmitProposalResponse);
}

/.Note that for backwards compatibility this uses MsgSubmitProposal as the request
/.type instead of the more canonical MsgSubmitProposalRequest
message MsgSubmitProposal {
  google.protobuf.Any content = 1;
  string proposer = 2;
}

message MsgSubmitProposalResponse {
  uint64 proposal_id;
}
```

これはgRPCで最も一般的に使用されますが、このようにprotobufの[サービス]定義をオーバーロードしても違反しません。
[protobuf仕様](https://developers.google.com/protocol-buffers/docs/proto3#services)の目的は次のとおりです。
> gRPCを使用したくない場合は、独自のRPC実装でプロトコルバッファを使用することもできます。
このメソッドを使用すると、自動的に生成された `MsgServer`インターフェイスが得られます。

リターンタイプを明示的に指定することに加えて、これはクライアントおよびサーバーコードの生成も容易にします。 サーバー上
一方では、これはほとんど自動生成されたゴールキーパーメソッドのようなものであり、最終的にはゴールキーパーの代わりに使用される可能性があります
([\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093)を参照): 
```go
package gov

type MsgServer interface {
  SubmitProposal(context.Context, *MsgSubmitProposal) (*MsgSubmitProposalResponse, error)
}
```

クライアント側では、開発者はトランザクションをカプセル化するRPC実装を作成することでこれを利用できます
論理。 [protobuf.js](https://github.com/protobufjs/protobuf.js#using-services)などの非同期コールバックを使用するProtobufライブラリ
複数の `Msg`を含むトランザクションの場合でも、特定のメッセージのコールバックを登録するために使用できます。

各 `Msg`サービスメソッドには、対応する` Msg`タイプのリクエストパラメータが必要です。たとえば、上記の `Msg`サービスメソッド` .cosmos.gov.v1beta1.Msg .SubmitProposal`には、リクエストパラメータが1つだけあります。つまり、 `Msg`タイプ` .cosmos.gov.v1beta1.MsgSubmitProposal`です。読者は、 `Msg`サービス(Protobufサービス)と` Msg`タイプ(Protobufメッセージ)の名前の違い、および完全修飾名の違いを明確に理解することが重要です。

この規則は、主に下位互換性のためだけでなく、 `TxBody.messages`の読みやすさのために、より標準化された` Msg ... Request`名で決定されました([コーディングセクション](以下の#encoding)を参照)): `.cosmos.gov.MsgSubmitProposal`を含むものは、` .cosmos.gov.v1beta1.MsgSubmitProposalRequest`を含むものよりも読みやすくなります。

この合意の結果の1つは、各 `Msg`タイプが` Msg`サービスメソッドのリクエストパラメータにしかなれなくなることです。ただし、この制限は明確であり、良い習慣であると私たちは信じています。

### エンコーディング

`Msg`サービスを使用して生成されたトランザクションエンコーディングは、[ADR-020](..adr-020-protobuf-transaction-encoding.md)で定義されている現在のProtobufトランザクションエンコーディングと同じです。 `Msg`タイプ(` Msg`サービスメソッドのリクエストパラメータ)を `Tx`の` Any`としてエンコードします。これにはパッケージングが含まれます
バイナリコード化された `Msg`とそのタイプのURL。

### デコード

`Msg`タイプは` Any`にパックされているため、トランザクションメッセージは `Any`sを` Msg`タイプにアンパックすることでデコードされます。詳細については、[ADR-020](..adr-020-protobuf-transaction-encoding.md#transactions)を参照してください。

### ルーティング

BaseAppに `msg_service_router`を追加することをお勧めします。このルーターはキー/値マッピングであり、 `Msg`タイプの` type_url`を対応する `Msg`サービスメソッドハンドラーにマッピングします。 `Msg`タイプと` Msg`サービスメソッドの間には1対1のマッピングがあるため、 `msg_service_router`には` Msg`サービスメソッドごとに1つのエントリしかありません。

BaseAppが(CheckTxまたはDeliverTxで)トランザクションを処理するとき、その[TxBody.messages]は[Msg]としてデコードされます。各 `Msg`の` type_url`は `msg_service_router`のエントリと一致し、対応する` Msg`サービスメソッドハンドラが呼び出されます。

下位互換性のために、古いハンドラーは削除されていません。 BaseAppが `msg_service_router`に対応するエントリがない古いバージョンの` Msg`を受信した場合、古いバージョンの `Route()`メソッドを介して古いバージョンのハンドラーにルーティングされます。

### モジュール構成

[ADR 021](..adr-021-protobuf-query-encoding.md)で、メソッド `RegisterQueryService`を導入しました
[AppModule]に移動します。これにより、モジュールはgRPCクエリアを登録できます。

`Msg`サービスを登録するために、` RegisterQueryService`を変換することでより拡張可能なメソッドを試してみます
より一般的な `RegisterServices`メソッドへ: 

```go
type AppModule interface {
  RegisterServices(Configurator)
  ...
}

type Configurator interface {
  QueryServer() grpc.Server
  MsgServer() grpc.Server
}

/.example module:
func (am AppModule) RegisterServices(cfg Configurator) {
	types.RegisterQueryServer(cfg.QueryServer(), keeper)
	types.RegisterMsgServer(cfg.MsgServer(), keeper)
}
```

`RegisterServices`メソッドと` Configurator`インターフェースは次のように設計されています
[\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093)で説明されているユースケースに合わせて進化する
そして[\#7122](https://github.com/cosmos/cosmos-sdk/issues/7421)。

`Msg`サービスを登録するとき、フレームワークはすべての` Msg`タイプを検証する必要があります
`sdk.Msg`インターフェースを実装し、初期化中にエラーをスローする代わりに
トランザクションが処理された後。

### `Msg`サービスの実装

サービスをクエリするのと同じように、 `Msg`サービスメソッドは` sdk.Context`を取得できます
`sdk.UnwrapSDKContext`の` context.Context`パラメータメソッドの使用から
方法: 

```go
package gov

func (k Keeper) SubmitProposal(goCtx context.Context, params *types.MsgSubmitProposal) (*MsgSubmitProposalResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)
    ...
}
```

`sdk.Context` 应该有一个已经由 BaseApp 的 `msg_service_router` 附加的 `EventManager`。

このメソッドでは、個別のハンドラー定義は不要になりました。

## 結果

この設計により、モジュール機能の開示およびアクセス方法が変更されます。既存の `Handler`インターフェースと` AppModule.Route`を放棄し、代わりに上記の[Protocol Buffer Services](https://developers.google.com/protocol-buffers/docs/proto3#services)とサービスをサポートしますルーティング。これにより、コードが大幅に簡素化されます。ハンドラーとデーモンを作成する必要がなくなりました。プロトコルバッファを使用して自動生成されたクライアントは、モジュールとモジュールのユーザー間の通信インターフェイスを明確に分離します。制御ロジック(別名ハンドラーとデーモン)は公開されなくなりました。モジュールインターフェイスは、クライアントAPIを介してアクセスされるブラックボックスと見なすことができます。クライアントインターフェイスもプロトコルバッファによって生成されることに注意してください。

これにより、機能テストの実行方法を変更することもできます。 AppModuleとルーターをシミュレートする代わりに、クライアントをシミュレートします(サーバーは非表示のままになります)。具体的には、 `moduleB`で` moduleA.MsgServer`をシミュレートすることはなく、代わりに `moduleA.MsgClient`をシミュレートします。外部サービス(データベースやオンラインサーバーなど)を使用していると考えてください。クライアントとサーバー間の送信は、生成されたプロトコルバッファによって正しく処理されると想定しています。

最後に、クライアントAPIモジュールをオフにすると、ADR-033で説明されている理想的なOCAPモードがオンになります。サーバーの実装とインターフェースは隠されているため、誰も[管理者]/サーバーを保持できず、クライアントインターフェースで中継することを余儀なくされます。これにより、開発者にとって正しいパッケージングとソフトウェアエンジニアリングモデルが促進されます。

### アドバンテージ

-返品タイプを明確に伝える
-プログラム登録とリターンタイプマーシャリングを手動で処理する必要がなくなり、インターフェイスを実装して登録するだけです。
-通信インターフェースが自動的に生成され、開発者は状態遷移方法にのみ集中できるようになりました-これにより、[\#7093](https://github.com/cosmos/cosmos-sdk/issues/7093)のUXが向上します。 )方法(1)採用することを選択した場合
-生成されたクライアントコードは、クライアントとテストに役立つ場合があります
-コードを大幅に削減および簡素化

### 欠点

-gRPCコンテキストの外部で `service`定義を使用すると、混乱が生じる可能性があります(ただし、proto3仕様に違反しません)。

## 参照する

-[Initial Github Issue \#7122](https://github.com/cosmos/cosmos-sdk/issues/7122)
-[proto 3言語ガイド:サービスの定義](https://developers.google.com/protocol-buffers/docs/proto3#services)
-[`Any``Msg`の初期事前設計](https://docs.google.com/document/d/1eEgYgvgZqLE45vETjhwIw4VOqK-5hwQtZtjVbiXnIGc)
-[ADR 020](..adr-020-protobuf-transaction-encoding.md)
-[ADR 021](..adr-021-protobuf-query-encoding.md) 