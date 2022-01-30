# ADR 033:Protobufに基づくモジュール間の通信

## 変更ログ

-2020-10-05:最初のドラフト

## 状態

提案

## 概要

ADRは、protobuf`Query`と `Msg`を使用して許可されたモジュール間で通信するためのシステムを導入します
[ADR 021](..adr-021-protobuf-query-encoding.md)サービス定義と
[ADR 031](..adr-031-msg-service.md)は以下を提供します:

-安定したprotobufベースのモジュールインターフェイスは、将来的にキーパーパラダイムに取って代わる可能性があります
-モジュール間オブジェクト機能(OCAP)のより強力な保証
-モジュールアカウントとサブアカウントの承認

## 環境

現在のCosmosSDKドキュメントの[Object-CapabilityModel](...core .ocap.md)については、次のように述べられています。

>繁栄しているCosmosSDKモジュールエコシステムは、ブロックチェーンアプリケーションに簡単に組み合わせることができ、欠陥のあるモジュールや悪意のあるモジュールが含まれていると想定しています。

現在、繁栄しているCosmosSDKモジュールエコシステムはありません。これは部分的に次の理由によるものと想定しています。

1.モジュールを構築するための安定したv1.0CosmosSDKの欠如。モジュールインターフェイスは、から、時には劇的に変化しています
通常、ポイントリリースからポイントリリースへの正当な理由がありますが、これは安定した基盤を確立しません。
2.正しく実装されたオブジェクト機能またはオブジェクト指向のパッケージングシステムの欠如、リファクタリングにつながる
現在のインターフェースの制約は非常に貧弱であるため、モジュールホルダーインターフェースの数は避けられません。

### `x .bank`のケーススタディ

現在、 `x .bank`キーパーは、それを参照するすべてのモジュールにほぼ無制限にアクセスできます。例えば、
`SetBalance`メソッドを使用すると、呼び出し元は、適切な供給追跡をバイパスする場合でも、任意のアカウントの残高を任意に設定できます。

その後、いくつかのOCAPの外観を実現するために、モジュールレベルのキャスティングと住宅ローンを使用する試みがいくつかあったようです。
そして、許可を焼きます。これらの権限により、モジュールはモジュールの
自分のアカウント。これらの権限は、実際には、状態の[ModuleAccount]タイプの[[] string]配列に格納されます。

ただし、これらの権限は実際にはあまり効果がありません。これらは、[MintCoins]で参照できるモジュールを制御します。
`BurnCoins`メソッドと` DelegateCoins *** `メソッドがありますが、そのうちの1つには、アクセスを制御するための一意のオブジェクト機能トークンがありません-
単純な文字列。したがって、 `x .upgrade`モジュールは、を呼び出すことで、` x .stakeing`モジュールのトークンを単純に作成できます。
`MintCoins("ステーキング ")`。さらに、これらのキーパーメソッドにアクセスできるすべてのモジュールもアクセスできます
`SetBalance`は、OCAPによる他の試みを無効にし、基本的なオブジェクト指向のカプセル化さえも破壊します。

## 決定

[ADR-021](..adr-021-protobuf-query-encoding.md)と[ADR-031](..adr-031-msg-service.md)に基づいて、
セキュリティモジュールの承認とOCAPのためのモジュール間通信フレームワーク。
実装後、これは既存のパラダイムの代替として使用することもできます。
モジュール。ここで概説する方法は、Cosmos SDK v1.0の基礎を形成し、必要なものを提供することを目的としています。
安定性とパッケージングの保証により、繁栄するモジュールエコシステムが出現しました。

特に注目に値するのは、モジュールがそれを単独で採用することを決定できるように、この関数を_enable_決定することです。
既存のモジュールをこの新しいパラダイムに移行するという提案は、別の会話である必要があります。
このADRの修正として。

### 新しい[ガーディアン]パラダイム

[ADR 021](..adr-021-protobuf-query-encoding.md)では、protobufサービス定義を使用してクエリを定義するためのメカニズム
[ADR 31](..adr-031-msg-service.md)で導入され、protobufサービスを使用して `Msg`を定義するメカニズムが追加されました。
Protobufサービス定義は、サービスのクライアント側とサーバー側を表す2つのgolangインターフェースに加えて
いくつかのヘルプコード。以下は、銀行 `cosmos.bank.Msg .Send`のメッセージタイプの最小限の例です。 

```go
package bank

type MsgClient interface {
	Send(context.Context, *MsgSend, opts ...grpc.CallOption) (*MsgSendResponse, error)
}

type MsgServer interface {
	Send(context.Context, *MsgSend) (*MsgSendResponse, error)
}
```

[ADR 021](..adr-021-protobuf-query-encoding.md)および[ADR 31](..adr-031-msg-service.md)は、モジュールが生成された `QueryServer`を実装する方法を指定します
と `MsgServer`インターフェースは、それぞれ古いクエリャーと` Msg`ハンドラーの代わりに使用されます。

このADRでは、モジュールが生成された `QueryClient`を使用して、` Msg`をクエリして他のモジュールに送信する方法について説明しました。
`MsgClient`とインターフェイスし、既存の` Keeper`パラダイムを置き換えるこのメカニズムを提案します。明確にするために、
このADRでは、新しいprotobuf定義またはサービスを作成する必要はありません。代わりに、同じプロトタイプを使用します
クライアントがモジュール間の通信に使用したサービスインターフェイスに基づきます。

この `QueryClient`.` MsgClient`メソッドを使用すると、Keepersを外部モジュールに公開するよりも次の主な利点があります。

1. [buf](https://buf.build/docs/break-overview)を使用して、Protobufタイプの重大な変更を確認します。
protobufの設計により、前方互換性を確保しながら、強力な下位互換性が保証されます。
進化。
2.クライアントインターフェイスとサーバーインターフェイスを分離することで、2つの間に権限チェックコードを挿入できるようになります
これらの2つは、モジュールが指定された `Msg`を別のモジュールに送信して適切なものを提供することを許可されているかどうかを確認します
オブジェクト能力システム(以下を参照)。
3.モジュール間の通信に使用されるルーターは、トランザクションのロールバックを処理するための便利な場所を提供します。
操作のアトミック性を有効にします([現在の問題](https://github.com/cosmos/cosmos-sdk/issues/8030))。モジュール間呼び出しで障害が発生すると、モジュール全体で障害が発生します
トレード

このメカニズムには、次の追加の利点があります。

-コード生成を通じてボイラープレートを削減し、
-CosmWasmなどのVMまたはgRPCを使用する子プロセスを介して他の言語のモジュールを使用できるようにする

### モジュール間通信

protobufコンパイラによって生成された `Client`を使用するには、` grpc.ClientConn` [interface](https://github.com/regen-network/protobuf/blob/cosmos/grpc/types.go#L12)が必要です。
埋め込む。このために私たちは紹介します
`grpc.ClientConn`インターフェースを実装する新しいタイプ` ModuleKey`。 `ModuleKey`は[プライベート]と見なすことができます
[キー]はモジュールアカウントに対応し、特別な `Invoker()`関数を使用して認証を提供します。
これについては、以下で詳しく説明します。

ブロックチェーンユーザー(外部クライアント)は、アカウントの秘密鍵を使用して、署名者としてリストされている[メッセージ]を含むトランザクションに署名します(それぞれ
メッセージは `Msg.GetSigner`を使用して必要な署名者を指定します)。認証チェックは `AnteHandler`によって実行されます。

ここでは、モジュールを `Msg.GetSigners`で識別できるようにすることで、このプロセスを拡張します。モジュールが別のモジュールで `Msg`の実行をトリガーしたい場合、
その `ModuleKey`は送信者として機能し(以下で説明する` ClientConn`インターフェースを介して)、唯一の[署名者]として設定されます。注目に値する
この場合、暗号化署名は使用しません。
たとえば、モジュール `A`は、その` A.ModuleKey`を使用して、 `.cosmos.bank.Msg .Send`トランザクション用の` MsgSend`オブジェクトを作成できます。 `MsgSend`検証
これにより、 `from`アカウント(この場合は` A.ModuleKey`)が署名者になります。

以下は、モジュール `foo`が` x .bank`と相互作用すると仮定した例です。 

```go
package foo


type FooMsgServer {
 .....

  bankQuery bank.QueryClient
  bankMsg   bank.MsgClient
}

func NewFooMsgServer(moduleKey RootModuleKey, ...) FooMsgServer {
 .....

  return FooMsgServer {
   .....
    modouleKey: moduleKey,
    bankQuery: bank.NewQueryClient(moduleKey),
    bankMsg: bank.NewMsgClient(moduleKey),
  }
}

func (foo *FooMsgServer) Bar(ctx context.Context, req *MsgBarRequest) (*MsgBarResponse, error) {
  balance, err := foo.bankQuery.Balance(&bank.QueryBalanceRequest{Address: fooMsgServer.moduleKey.Address(), Denom: "foo"})

  ...

  res, err := foo.bankMsg.Send(ctx, &bank.MsgSendRequest{FromAddress: fooMsgServer.moduleKey.Address(), ...})

  ...
}
```

この設計は、たとえばキャストによって、よりきめ細かいライセンスのユースケースをカバーするように拡張できることも目的としています。
denomプレフィックスは、特定のモジュールに制限されています(例:
[#7459](https://github.com/cosmos/cosmos-sdk/pull/7459#discussion_r529545528))。

### `ModuleKey`sと` ModuleID`s

`ModuleKey`はモジュールアカウントの[秘密鍵]と見なすことができ、` ModuleID`は次のように見なすことができます
対応する[公開鍵]。 [ADR 028](..adr-028-public-key-addresses.md)から、モジュールはルートモジュールアカウントと任意の数のサブアカウントを持つことができます
または、さまざまなプール(例:誓約プール)または管理アカウント(例:グループ)で使用できます
アカウント)。 モジュールサブアカウントは派生キーに似ていると考えることもできます。ルートキーがあり、次にいくつかあります。
パスを導き出します。 `ModuleID`は、モジュール名とオプションの[派生]パスを含む単純な構造です。
そして、[ADR-028](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-028-public-key-addresses)の `AddressHash`メソッドに従ってアドレスを形成します。md): 

```go
type ModuleID struct {
  ModuleName string
  Path []byte
}

func (key ModuleID) Address() []byte {
  return AddressHash(key.ModuleName, key.Path)
}
```

この設計は、たとえばキャストによって、よりきめ細かいライセンスのユースケースをカバーするように拡張できることも目的としています。
denomプレフィックスは、特定のモジュールに制限されています(例:
[#7459](https://github.com/cosmos/cosmos-sdk/pull/7459#discussion_r529545528))。

### `ModuleKey`sと` ModuleID`s

`ModuleKey`はモジュールアカウントの[秘密鍵]と見なすことができ、` ModuleID`は次のように見なすことができます
対応する[公開鍵]。 [ADR 028](..adr-028-public-key-addresses.md)から、モジュールはルートモジュールアカウントと任意の数のサブアカウントを持つことができます
または、さまざまなプール(例:誓約プール)または管理アカウント(例:グループ)で使用できます
アカウント)。 モジュールサブアカウントは派生キーに似ていると考えることもできます。ルートキーがあり、次にいくつかあります。
パスを導き出します。 `ModuleID`は、モジュール名とオプションの[派生]パスを含む単純な構造です。
そして、[ADR-028](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-028-public-key-addresses)の `AddressHash`メソッドに従ってアドレスを形成します。md): 

```go
type Invoker func(callInfo CallInfo) func(ctx context.Context, request, response interface{}, opts ...interface{}) error

type CallInfo {
  Method string
  Caller ModuleID
}

type RootModuleKey struct {
  moduleName string
  invoker Invoker
}

func (rm RootModuleKey) Derive(path []byte) DerivedModuleKey {.* ... */}

type DerivedModuleKey struct {
  moduleName string
  path []byte
  invoker Invoker
}
```

モジュールは、 `RootModuleKey`で` Derive(path [] byte) `メソッドを使用して、` DerivedModuleKey`にアクセスできます。
このキーは、サブアカウントからの[メッセージ]を確認するために使用されます。 元: 

```go
package foo

func (fooMsgServer *MsgServer) Bar(ctx context.Context, req *MsgBar) (*MsgBarResponse, error) {
  derivedKey := fooMsgServer.moduleKey.Derive(req.SomePath)
  bankMsgClient := bank.NewMsgClient(derivedKey)
  res, err := bankMsgClient.Balance(ctx, &bank.MsgSend{FromAddress: derivedKey.Address(), ...})
  ...
}
```

このようにして、モジュールはルートアカウントと任意の数のサブアカウントにアクセスして送信するためのアクセス許可を取得できます
これらのアカウントからの認証された[メッセージ]。 `Invoker``callInfo.Caller`パラメータがバックグラウンドで使用されます
異なるモジュールアカウントを区別しますが、どちらの方法でも、 `Invoker`によって返される関数は` Msg`のみを許可します
ルートまたは派生モジュールアカウントから渡します。

`Invoker`自体は、渡された` CallInfo`に基づいて関数クロージャを返すことに注意してください。 これにより、クライアントは実装できるようになります
将来的には、ハッシュテーブルルックアップのオーバーヘッドを回避するために、各メソッドタイプの呼び出し関数がキャッシュされます。
これにより、このモジュール間通信方式のパフォーマンスオーバーヘッドが最小限に抑えられます。
権限を確認してください。

繰り返しになりますが、クロージャはアクセス許可呼び出しのみを許可します。 他のものにアクセスする方法はありません
名前になりすます。

以下は、[RootModuleKey]の[grpc.ClientConn.Invoke]実装の大まかなスケッチです。 

```go
func (key RootModuleKey) Invoke(ctx context.Context, method string, args, reply interface{}, opts ...grpc.CallOption) error {
  f := key.invoker(CallInfo {Method: method, Caller: ModuleID {ModuleName: key.moduleName}})
  return f(ctx, args, reply)
}
```

### `AppModule`の配線と要件

[ADR 031](..adr-031-msg-service.md)では、 `AppModule.RegisterService(Configurator)`メソッドが導入されました。 サポート
モジュール間の通信のために、 `Configurator`インターフェースを拡張して` ModuleKey`を渡し、モジュールを許可します
`RequireServer()`を使用して、他のモジュールへの依存関係を指定します。 

```go
type Configurator interface {
   MsgServer() grpc.Server
   QueryServer() grpc.Server

   ModuleKey() ModuleKey
   RequireServer(msgServer interface{})
}
```

`ModuleKey`は` RegisterService`メソッド自体でモジュールに渡されるため、 `RegisterServices`は単一のものとして
モジュールサービスのエントリポイントを構成します。 これは、ボイラープレートを大幅に削減するという副作用もあります。
`app.go`。 現在、 `ModuleKey`は` AppModuleBasic.Name() `に基づいて作成されますが、より柔軟なシステムは
将来。 `ModuleManager`は、バックグラウンドでモジュールアカウントの作成を処理します。

モジュールは相互に直接アクセスできなくなったため、モジュールの依存関係が満たされていない可能性があります。 確実に
起動時にモジュールの依存関係が解決された場合は、 `Configurator.RequireServer`メソッドを追加する必要があります。 `モジュールマネージャー`
これにより、アプリケーションが起動する前に、RequireServerで宣言されたすべての依存関係を解決できるようになります。 一例
モジュール `foo`は、次のように` x .bank`への依存関係を宣言できます。 

```go
package foo

func (am AppModule) RegisterServices(cfg Configurator) {
  cfg.RequireServer((*bank.QueryServer)(nil))
  cfg.RequireServer((*bank.MsgServer)(nil))
}
```

### 安全上のご注意

[ModuleKey]権限の確認に加えて、いくつかの追加のセキュリティ対策を講じる必要があります
基盤となるルーターインフラストラクチャ。

#### 再帰と再入可能

再帰的または再入可能なメソッド呼び出しは、潜在的なセキュリティの脅威を構成します。モジュールAの場合、これは問題である可能性があります
モジュールBとモジュールBを同じ呼び出しで呼び出します。モジュールAが再度呼び出されます。

ルータシステムがこの問題に対処するための基本的な方法は、モジュールを防ぐためにコールスタックを維持することです。
コールスタックで複数回参照されるため、再入力されません。 `map [string] interface {}`テーブル
このセキュリティチェックを実行するためにルータで使用できます。

#### お問い合わせ

Cosmos SDKのクエリは通常ライセンスがないため、あるモジュールが別のモジュールにクエリを実行できるようにすることは、構成されるべきではありません。
基本的な予防策を講じる場合、主要なセキュリティ上の脅威。ルーターシステムの基本的な注意事項
queryメソッドに渡された `sdk.Context`がストレージへの書き込みを許可されていないことを確認する必要があります。この
これは、[BaseApp]クエリに対して現在実行されているのと同じように、[CacheMultiStore]を使用して実行できるようになりました。

### 内部メソッド

多くの場合、モジュールが他のモジュールのメソッドを呼び出す必要があり、これらのモジュールはクライアントにまったく公開されません。このため
目標を達成するために、 `InternalServer`メソッドを` Configurator`に追加します。 

```go
type Configurator interface {
   MsgServer() grpc.Server
   QueryServer() grpc.Server
   InternalServer() grpc.Server
}
```

たとえば、Slash of x .stakeingはSlashof x .stakingを呼び出す必要がありますが、Slash of x .stakeingをエンドユーザーに公開したくありません。
顧客と。

内部protobufサービスは、指定されたモジュールの対応する[internal.proto]ファイルで定義されます。
プロトタイプパッケージ。

[InternalServer]に登録されているサービスは、他のモジュールから呼び出すことはできますが、外部クライアントから呼び出すことはできません。

内部アプローチの別の解決策には、[ここ](https://github.com/cosmos/cosmos-sdk/pull/7459#issuecomment-733807753)で説明されているフック/プラグインが含まれる場合があります。
フック/プラグインシステムのより詳細な評価は、このADRのフォローアップで、または個別に行われます。
ADR。

### 承認

デフォルトでは、モジュール間ルーターでは、GetSignersから返された最初の署名者がメッセージを送信する必要があります。この
モジュール間ルーターは、[ADR 030](https://github.com/cosmos/cosmos-sdk/blob/master/docs/architecture/adr-030-authz-module.md)などの承認されたミドルウェアも受け入れる必要があります。 。
ミドルウェアは、アカウントが他の方法で特定のモジュールアカウントに代わって操作を実行できるようにします。
承認ミドルウェアは、特定のモジュールに[管理者]権限を効果的に付与する必要性を考慮する必要があります
その他のモジュール。これは、別のADRまたはこのADRの更新で解決されます。

### 将来の仕事

その他の将来の改善には、次のものが含まれる可能性があります。

*カスタムコード生成:
    *インターフェイスを簡素化します(たとえば、コードを生成するには、 `context.Context`の代わりに` sdk.Context`を使用します)
    *モジュール間の呼び出しを最適化する-たとえば、最初の呼び出し後に解決をキャッシュする方法
* `StoreKey`と` ModuleKey`を1つのインターフェースに結合して、モジュールが個別のOCAPハンドルを持つようにします。
*モジュール間の通信をより効率的にするためのコード生成
*アプリケーションがルートモジュールアカウント名を上書きできるように、 `ModuleKey`の作成を` AppModuleBasic.Name() `から切り離します
*モジュール間フックとプラグイン

##代替プラン

### MsgServicesと `x .capability`

`x .capability`モジュールは、適切なオブジェクト機能の実装を提供します。
[\#5931](https://github.com/cosmos/cosmos-sdk/issues/5931)で説明されているように、CosmosSDKはモジュール間OCAPにも使用できます。

このADRで説明されている方法の主な利点は、CosmosSDKの他の部分と統合する方法にあります。
具体的には:

*次の目的でprotobuf:
    *インターフェイスのコード生成を使用して、開発ユーザーエクスペリエンスを向上させることができます
    * [buf](https://docs.buf.build/break-overview)を使用して、モジュールインターフェイスをバージョン管理し、損傷をチェックします
* ADR028のサブモジュールアカウントによると
*パラダイムと署名者を渡す一般的な `Msg`メソッドは` GetSigners`によって指定されます

さらに、これはキーパーの完全な代替品であり、モジュール間通信のすべてに適用できます。
#5931の `x .capability`メソッドは1つずつ適用する必要があります。

## 結果

### 下位互換性

このADRは、モジュール間の長期的な互換性が高いシナリオの方法を提供することを目的としています。
短期的には、これは過度に寛大なおよび/または
`Keeper`インターフェースを完全に置き換えます。
### ポジティブ

-安定したモジュール間インターフェースをより簡単に実装できるKeepersの代替
-適切なモジュール間OCAP
-一部の参加者がコメントしたように、モジュール開発者DevXを改善しました
     [アーキテクチャレビュー電話会議、12月3日](https://hackmd.io/E0wxxOvRQ5qVmTf6N_k84Q)
-大幅に簡素化された[app.go]の基盤を築く
-ルーターは、モジュールからモジュールへの呼び出しへのアトミックトランザクションを実施するように設定できます

### ネガティブ

-この方法のモジュールには、多くのリファクタリングが必要になります

### ニュートラル

## テストケース[オプション]

## 参照

-[ADR 021](..adr-021-protobuf-query-encoding.md)
-[ADR 031](..adr-031-msg-service.md)
-[ADR 028](..adr-028-public-key-addresses.md)
-[ADR 030ドラフト](https://github.com/cosmos/cosmos-sdk/pull/7105)
-[オブジェクト機能モデル](...docs .core .ocap.md) 