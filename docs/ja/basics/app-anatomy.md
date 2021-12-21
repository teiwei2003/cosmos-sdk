# CosmosSDKアプリケーション分析

このドキュメントでは、CosmosSDKアプリケーションのコア部分について説明します。 このドキュメント全体を通して、「app」という名前のプレースホルダーアプリケーションが使用されます。 {まとめ}

## ノードクライアント

デーモンまたは[フルノードクライアント](../core/node.md)は、CosmosSDKに基づくブロックチェーンのコアプロセスです。 ネットワークの参加者は、このプロセスを実行してステートマシンを初期化し、他のフルノードに接続し、新しいブロックが入るとステートマシンを更新します。

```
                ^  +-------------------------------+  ^
                |  |                               |  |
                |  |  State-machine = Application  |  |
                |  |                               |  |   Built with Cosmos SDK
                |  |            ^      +           |  |
                |  +----------- | ABCI | ----------+  v
                |  |            +      v           |  ^
                |  |                               |  |
Blockchain Node |  |           Consensus           |  |
                |  |                               |  |
                |  +-------------------------------+  |   Tendermint Core
                |  |                               |  |
                |  |           Networking          |  |
                |  |                               |  |
                v  +-------------------------------+  v
```

ブロックチェーンのフルノードは、通常、「デーモン」の接尾辞「-d」が付いたバイナリファイルとして表示されます(たとえば、「app」の場合は「appd」、「gaia」の場合は「gaiad」)。このバイナリファイルは、 `。/cmd/appd/`に配置されている単純な[`main.go`](../core/node.md#main-function)関数を実行して作成されます。この操作は通常、[Makefile](#dependencies-and-makefile)を介して行われます。

メインのバイナリファイルを作成した後、[`start`コマンド](../core/node.md#start-command)を実行してノードを起動できます。このコマンド関数は主に次の3つのことを行います。

1.[`app.go`](#core-application-file)で定義されたステートマシンインスタンスを作成します。
2. `〜/.app/data`フォルダに保存されている` db`から抽出された最新の既知の状態でステートマシンを初期化します。この時点で、ステートマシンは「appBlockHeight」の高さにあります。
3.新しいTendermintインスタンスを作成して開始します。特に、ノードはピアとのハンドシェイクを実行します。それらから最新の `blockHeight`を取得し、ローカルの` appBlockHeight`よりも大きい場合は、ブロックを再生してこの高さに同期します。 `appBlockHeight`が` 0`の場合、ノードは作成から開始し、TendermintはABCIを介して `App`に` InitChain`メッセージを送信し、[`InitChainer`](#initchainer)をトリガーします。 
##コアアプリケーションファイル

通常、ステートマシンのコアは、「app.go」という名前のファイルで定義されます。これには主に、**アプリケーションタイプの定義**と**アプリケーションを作成および初期化するための関数**が含まれています。

###アプリケーションタイプの定義

`app.go`で最初に定義されるのは、アプリケーションの` type`です。通常、次の部分で構成されます。

-**[`baseapp`](../core/baseapp.md)への参照。 ** `app.go`で定義されているカスタムアプリケーションは` baseapp`の拡張です。 Tendermintがトランザクションをアプリケーションに中継するとき、 `app`は` baseapp`のメソッドを使用してそれらを適切なモジュールにルーティングします。 `baseapp`は、すべての[ABCIメソッド](https://tendermint.com/docs/spec/abci/abci.html#overview)と[ルーティングロジック](../core)を含む、アプリケーションのコアロジックのほとんどを実装します。 )/baseapp.md#routing)。
-**ストレージキーリスト**。状態全体を含む[store](../core/store.md)は、CosmosSDKにあります。各モジュールは、マルチストア内の1つ以上のストアを使用して、状態部分を永続化します。これらのストアには、「アプリ」タイプで宣言された特定のキーを使用してアクセスできます。これらのキーと `キーパー`は、Cosmos SDK[オブジェクト関数モデル](../core/ocap.md)のコアです。
-**モジュールの `キーパー`リスト。 **各モジュールは、このモジュールに格納されている読み取りと書き込みを処理する[`keeper`](../building-modules/keeper.md)という名前の抽象化を定義します。モジュールの` keeper`メソッドは、他のモジュールからのものにすることができます(許可されている場合)が呼び出されます。これが、アプリケーションタイプで宣言され、他のモジュールが許可された機能にのみアクセスできるように、インターフェイスとして他のモジュールにエクスポートされる理由です。
-**[`appCodec`](../core/encoding.md)への参照。 **アプリケーションの `appCodec`は、データ構造をシリアル化および逆シリアル化して保存するために使用されます。これは、ストレージが永続化できるのは`[] bytes`のみであるためです。デフォルトのコーデックは[ProtocolBuffers](../core/encoding.md)です。
-**[`legacyAmino`](../core/encoding.md)コーデックへの参照。 ** Cosmos SDKの一部は、上記の `appCodec`を使用するようにまだ移行されておらず、アミノを使用するようにハードコードされています。他の部分は、下位互換性のために明示的にAminoを使用しています。これらの理由により、アプリケーションは引き続き古いAminoコーデックへの参照を保持します。今後のリリースでは、AminoコーデックがSDKから削除されることに注意してください。
-**[Module Manager](../building-modules/module-manager.md#manager)**および[Basic Module Manager](../building-modules/module-manager.md#Basic Manager)への参照。モジュールマネージャは、アプリケーションモジュールのリストを含むオブジェクトです。[`Msg`サービス](../core/baseapp.md#msg-services)や[gRPC`クエリ`サービス](../core/baseapp.md#)の登録など、これらのモジュールに関連する操作が容易になります。 grpc -query-services)、または[`InitChainer`](#initchainer)、[` BeginBlocker`および `EndBlocker`](#beginblocker-and-endblocker)などのさまざまな関数のモジュール間の実行順序を設定します。

「simapp」のアプリケーションタイプ定義の例を参照してください。CosmosSDK独自のアプリケーションは、デモンストレーションとテストの目的で使用されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L145-L187

### コンストラクタ

この関数は、前のセクションで型が定義された新しいアプリケーションを作成します。アプリケーションデーモンコマンドの[`start`コマンド](../core/node.md#start-command)で使用する前に、` AppCreator`シグニチャを満たす必要があります。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/types/app.go#L48-L50

この関数によって実行される主な操作は次のとおりです。

-新しい[`codec`](../core/encoding.md)をインスタンス化し、[basic manager](../building-modules/module-manager.md)を使用して各アプリケーションモジュールの`コーデックを初期化します `# basicmanager)
-「baseapp」インスタンス、コーデック、およびすべての適切なストレージキーへの参照を使用して、新しいアプリケーションをインスタンス化します。
-各アプリケーションモジュールの `NewKeeper`関数を使用して、アプリケーションの` type`で定義されているすべての[`keeper`s](#keeper)をインスタンス化します。あるモジュールの `NewKeeper`が別のモジュールの` keeper`を参照する必要がある場合があるため、 `keepers`は正しい順序でインスタンス化する必要があることに注意してください。
-各アプリケーションモジュールの[`AppModule`](#application-module-interface)オブジェクトを使用して、アプリケーションの[Module Manager](../building-modules/module-manager.md#manager)をインスタンス化します。
-モジュールマネージャーを使用して、アプリケーションの[`Msg`サービス](../core/baseapp.md#msg-services)、[gRPC`クエリ`サービス](../core/baseapp.md#grpc-queryを初期化します-services)、[古いバージョンの `Msg`ルーティング](../core/baseapp.md#routing)および[古いバージョンのクエリルーティング](../core/baseapp.md#query-routing)。 TendermintがABCIを介してトランザクションをアプリケーションに中継する場合、ここで定義されたルートを使用して、対応するモジュールの[`Msg`サービス](#msg-services)にルーティングします。同様に、アプリケーションがgRPCクエリリクエストを受信すると、ここで定義されたgRPCルートを使用して、対応するモジュールの[`gRPCクエリサービス`](#grpc-query-services)にルーティングします。 Cosmos SDKは、古いバージョンの `Msg`と古いバージョンのTendermintクエリを引き続きサポートします。これらは、古いバージョンの` Msg`ルートと古いバージョンのクエリルートをそれぞれルーティングに使用します。
-モジュールマネージャーを使用して、[アプリケーションモジュールの不変条件](../building-modules/invariants.md)を登録します。不変条件は、各ブロックの最後で評価される変数です(たとえば、トークンの総供給量)。不変条件をチェックするプロセスは、[`InvariantsRegistry`](../building-modules/invariants.md#invariant-registry)と呼ばれる特別なモジュールを介して行われます。不変条件の値は、モジュールで定義された予測値と等しくなければなりません。値が予測値と異なる場合、不変レジストリで定義された特別なロジックがトリガーされます(通常はチェーンが停止します)。これにより、重大なエラーが無視されたり、修復が困難な永続的な影響が発生したりすることがなくなります。
-モジュールマネージャを使用して、各[アプリケーションモジュール](#application-module-interface)の `InitGenesis`、` BeginBlocker`、および `EndBlocker`関数の実行順序を設定します。すべてのモジュールがこれらの機能を実装しているわけではないことに注意してください。
-アプリケーションの残りのパラメータを設定します。
    -[`InitChainer`](#initchainer):アプリケーションが最初に起動されたときにアプリケーションを初期化するために使用されます。
    -[`BeginBlocker`、` EndBlocker`](#beginblocker-and-endlbocker):各ブロックの最初と最後で呼び出されます)。
    -[`anteHandler`](../core/baseapp.md#antehandler):手数料と署名の検証に使用されます。
-ストレージをインストールします。
-アプリケーションを返します。

この関数はアプリケーションのインスタンスのみを作成し、実際の状態はノードの再起動時に `〜/.app/data`フォルダーから転送されるか、ノードの起動時にジェネシスファイルから生成されることに注意してください。初めて。

「simapp」のアプリケーションコンストラクターの例を参照してください。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L198-L441

### InitChainer

`InitChainer`は、ジェネシスファイル(つまり、ジェネシスアカウントのトークンバランス)からアプリケーションの状態を初期化する関数です。これは、アプリケーションがTendermintエンジンから `InitChain`メッセージを受信したときに呼び出されます。これは、ノードが` appBlockHeight == 0`で開始されたとき(つまり、作成時)に発生します。アプリケーションは、[`SetInitChainer`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp#BaseApp.SetInitChainer)を渡して、その[コンストラクター](#constructor-function)`に `InitChainerを設定する必要があります。 ) 方法。

一般的に、 `InitChainer`は、主にアプリケーションの各モジュールの[` InitGenesis`](../building-modules/genesis.md#initgenesis)関数で構成されています。これは、モジュールマネージャーの `InitGenesis`関数を呼び出すことによって行われます。この関数は、モジュールマネージャーに含まれる各モジュールの` InitGenesis`関数を呼び出します。モジュールマネージャーでモジュールの `InitGenesis`関数を呼び出す順序を設定するには、[Module Manager's](../building-modules/module-manager.md)` SetOrderInitGenesis`メソッドを使用する必要があることに注意してください。これは[application-constructor](#application-constructor)で行われ、「SetOrderInitGenesis」は「SetInitChainer」の前に呼び出す必要があります。

「simapp」の「InitChainer」の例を参照してください。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L464-L471

### BeginBlocker and EndBlocker

Cosmos SDKは、開発者にアプリケーションの一部としてコード実行を自動化する可能性を提供します。これは、「BeginBlocker」と「EndBlocker」という名前の2つの関数によって実現されます。これらは、アプリケーションがTendermintエンジンからそれぞれ `BeginBlock`メッセージと` EndBlock`メッセージを受信したときに呼び出されます。これは、各ブロックの最初と最後に発生します。アプリケーションは、[`SetBeginBlocker`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp)`#を介して[コンストラクター](#constructor-function)に `BeginBlocker`と` EndBlockerを設定する必要があります。 BaseApp.SetBeginBlocker)および[`SetEndBlocker`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp#BaseApp.SetEndBlocker)メソッド。

一般的に、 `BeginBlocker`関数と` EndBlocker`関数は、主に各アプリケーションモジュールの[`BeginBlock`と` EndBlock`](../building-modules/beginblock-endblock.md)関数で構成されています。これは、モジュールマネージャの `BeginBlock`および` EndBlock`関数を呼び出すことによって行われます。次に、モジュールマネージャは、それに含まれる各モジュールの `BeginBlock`および` EndBlock`関数を呼び出します。モジュールマネージャでモジュール「BeginBlock」および「EndBlock」関数を呼び出す順序を設定するには、それぞれ「SetOrderBeginBlock」メソッドと「SetOrderEndBlock」メソッドを使用する必要があることに注意してください。これは、[Application Constructor](#application-constructor)の[Module Manager](../building-modules/module-manager.md)を介して行われ、 `SetOrderBeginBlock`と` SetOrderEndBlock`を呼び出す必要があります。メソッドは `の前にあります。 SetBeginBlocker`および `SetEndBlocker`関数。

補足として、アプリケーション固有のブロックチェーンは決定論的であることを覚えておくことが重要です。開発者は、「BeginBlocker」または「EndBlocker」に不確実性を導入しないように注意する必要があります。また、[gas](./gas-fees.md)はコストを制限しないため、計算コストを高くしすぎないように注意する必要があります。 BeginBlocker`と `EndBlocker`が実行されます。

「simapp」の「BeginBlocker」および「EndBlocker」関数の例を参照してください。  

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/app.go#L454-L462

### Register Codec

`EncodingConfig` 结构是 `app.go` 文件的最后一个重要部分。 此结构的目标是定义将在整个应用程序中使用的编解码器。 

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/params/encoding.go#L9-L16

以下は、4つのフィールドのそれぞれの意味の説明です。

-`InterfaceRegistry`:Protobufコーデックは `InterfaceRegistry`を使用して[` google.protobuf.Any`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any .proto)。 `Any`は、` type_url`(インターフェースを実装する具象型の名前)と `value`(そのエンコードバイト)を含む構造体と考えることができます。 `InterfaceRegistry`は、インターフェースと実装を登録するためのメカニズムを提供します。これは、` Any`から安全に解凍できます。アプリケーションの各モジュールは `RegisterInterfaces`メソッドを実装します。このメソッドは、モジュール自体のインターフェースと実装を登録するために使用できます。
    -Anyの詳細については、[ADR-19](../architecture/adr-019-protobuf-state-encoding.md#usage-of-any-to-encode-interfaces)を参照してください。
    -より詳細な理解のために、Cosmos SDKは[`gogoprotobuf`](https://github.com/gogo/protobuf)という名前のProtobuf仕様の実装を使用します。デフォルトでは、[`Any`のgogoprotobuf実装](https://godoc.org/github.com/gogo/protobuf/types)は[Global Type Registration](https://github.com/gogo/protobuf/blob/master/proto/properties.go#L540) `Any`にカプセル化された値を特定のGoタイプにデコードします。これにより、依存関係ツリー内の悪意のあるモジュールがグローバルprotobufレジストリにタイプを登録し、 `type_url`フィールドでそれを参照するトランザクションによってロードおよびグループ解除される可能性があるという脆弱性が導入されます。詳細については、[ADR-019](../architecture/adr-019-protobuf-state-encoding.md)を参照してください。
-`Marshaler`:CosmosSDK全体で使用されるデフォルトのコーデック。これは、ステータスをエンコードおよびデコードするための `BinaryCodec`と、ユーザーにデータを出力するための` JSONCodec`で構成されます(たとえば、[CLI](#cli))。デフォルトでは、SDKはProtobufを `Marshaler`として使用します。
-`TxConfig`: `TxConfig`は、クライアントがアプリケーションによって定義された特定のトランザクションタイプを生成するために使用できるインターフェースを定義します。現在、SDKは「SIGN_MODE_DIRECT」(オンラインコードとしてProtobufバイナリを使用)と「SIGN_MODE_LEGACY_AMINO_JSON」(Aminoに依存)の2つのトランザクションタイプを処理します。トランザクションの詳細については、[こちら](../core/transaction.md)をご覧ください。
-`Amino`:Cosmos SDKの一部のレガシー部分は、下位互換性のために引き続きAminoを使用しています。各モジュールは `RegisterLegacyAmino`メソッドを公開して、特定のタイプのモジュールをAminoに登録します。アプリケーション開発者は、この `Amino`コーデックを使用しないようにする必要があり、将来のバージョンで削除される予定です。

Cosmos SDKは、アプリケーションコンストラクター( `NewApp`)の` EncodingConfig`を作成するために使用される `MakeTestEncodingConfig`関数を公開します。デフォルトの「Marshaler」としてProtobufを使用します。
注:この機能は非推奨としてマークされており、アプリケーションの作成またはテストでの使用にのみ使用できます。 Stargateのリリース後、コーデック管理のリファクタリングに取り組んでいます。

「simapp」の「MakeTestEncodingConfig」の例を参照してください。 

+++ https://github.com/cosmos/cosmos-sdk/blob/590358652cc1cbc13872ea1659187e073ea38e75/simapp/encoding.go#L8-L19

## Modules

[モジュール](../building-modules/intro.md)は、CosmosSDKアプリケーションの核心です。それらは、ステートマシンにネストされたステートマシンと見なすことができます。トランザクションが基盤となるTendermintエンジンからABCIを介してアプリケーションに中継されると、[`baseapp`](../core/baseapp.md)によって適切なモジュールにルーティングされて処理されます。このパラダイムにより、開発者は通常、必要なモジュールのほとんどがすでに存在するため、複雑なステートマシンを簡単に構築できます。開発者にとって、Cosmos SDKアプリケーションの構築に関連する作業のほとんどは、アプリケーション用にまだ存在していないカスタムモジュールを構築し、それらを既存のモジュールと統合してコヒーレントアプリケーションにすることを中心に展開されます。アプリケーションディレクトリでは、標準的な方法はモジュールを `x/`フォルダに保存することです(ビルドされたモジュールを含むCosmosSDKの `x/`フォルダと混同しないでください)。

### アプリケーションモジュールインターフェース

モジュールは、Cosmos SDK[`AppModuleBasic`](../building-modules/module-managerで定義されている[interfaces](../building-modules/module-manager.md#application-module-interfaces)を実装する必要があります.md #appmodulebasic)および[`AppModule`](../building-modules/module-manager.md#appmodule)。前者は `codec`などのモジュールの基本的な非依存要素を実装し、後者はモジュールのほとんどのメソッド(他のモジュールを参照する必要がある` keeper`メソッドを含む)を処理します。 `AppModule`と` AppModuleBasic`の両方のタイプは、 `。/module.go`という名前のファイルで定義されています。

`AppModule`は、モジュール上の一連の便利なメソッドを公開します。これらのメソッドは、モジュールをコヒーレントアプリケーションにアセンブルするのに役立ちます。これらのメソッドは、アプリケーションモジュールのコレクションを管理する `Module Manager`(../building-modules/module-manager.md#manager)から呼び出されます。

### `Msg`サービス

各モジュールは、2つの[Protobufサービス](https://developers.google.com/protocol-buffers/docs/proto#services)を定義します。1つはメッセージを処理するための `Msg`サービスで、もう1つはクエリを処理するためのGRPC`Query`サービスです。モジュールをステートマシンと見なす場合、「Msg」サービスは状態遷移RPCメソッドのセットです。
各Protobuf`Msg`サービスメソッドはProtobufリクエストタイプ1:1に関連付けられており、 `sdk.Msg`インターフェイスを実装する必要があります。
`sdk.Msg`は[transactions](../core/transactions.md)にバンドルされており、各トランザクションには1つ以上のメッセージが含まれていることに注意してください。

フルノードが有効なトランザクションブロックを受信すると、Tendermintは各トランザクションブロックを[`DeliverTx`](https://tendermint.com/docs/app-dev/abci-spec.html#delivertx)アプリケーションに中継します。次に、アプリケーショントランザクションを処理します。

1.トランザクションを受信した後、アプリケーションは最初にトランザクションを `[] bytes`からグループ解除します。
2.次に、トランザクションに含まれる `Msg`を抽出する前に、[料金の支払いと署名](#gas-fees.md#antehandler)など、トランザクションに関するいくつかのことを確認します。
3. `sdk.Msg`sは、Protobuf[` Any`s](#register-codec)を使用してエンコードします。各 `Any`の` type_url`を分析することにより、baseappの `msgServiceRouter`は` sdk.Msg`を対応するモジュールの `Msg`サービスにルーティングします。
4.メッセージが正常に処理されると、ステータスが更新されます。

詳細については、トランザクション[ライフサイクル](./tx-lifecycle.md)を参照してください。

モジュール開発者は、独自のモジュールを構築するときにカスタムの `Msg`サービスを作成します。一般的なアプローチは、 `tx.proto`ファイルで` Msg`Protobufサービスを定義することです。たとえば、 `x/bank`モジュールは、トークンを転送するための2つのメソッドを持つサービスを定義します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/proto/cosmos/bank/v1beta1/tx.proto#L10-L17

サービスメソッドは `keeper`を使用してモジュールステータスを更新します。

各モジュールは、[`AppModule`インターフェース](#application-module-interface)の一部として` RegisterServices`メソッドも実装する必要があります。このメソッドは、生成されたProtobufコードによって提供される `RegisterMsgServer`関数を呼び出す必要があります。

### gRPC`Query`サービス

gRPCの `Query`サービスはv0.40Stargateバージョンで導入されました。ユーザーは[gRPC](https://grpc.io)を使用してステータスを確認できます。これらはデフォルトで有効になっており、[`app.toml`](../run-node/run-node.md#のフィールド` grpc.enable`と `grpc.address`でノード使用アプリケーションを設定できます。プログラムの構成)。

gRPCの `Query`サービスは、モジュールのProtobuf定義ファイル、特に` query.proto`で定義されています。 `query.proto`定義ファイルは` Query`[Protobufサービス](https://developers.google.com/protocol-buffers/docs/proto#services)を公開します。各gRPCクエリエンドポイントは、rpcキーワードで始まり、クエリサービスにあるサービスメソッドに対応します。

Protobufは、すべてのサービスメソッドを含むモジュールごとに `QueryServer`インターフェースを生成します。モジュールの[`keeper`](#keeper)は、各サービスメソッドの特定の実装を提供することにより、この` QueryServer`インターフェースを実装する必要があります。この特定の実装は、対応するgRPCクエリエンドポイントハンドラーです。

最後に、各モジュールは、[`AppModule`インターフェース](#application-module-interface)の一部として` RegisterServices`メソッドも実装する必要があります。このメソッドは、生成されたProtobufコードによって提供される `RegisterQueryServer`関数を呼び出す必要があります。 

### ゴールキーパー

[`Keepers`](../building-modules/keeper.md)は、モジュールストレージのゲートキーパーです。モジュールのストレージで読み取りまたは書き込みを行うには、その `keeper`メソッドの1つを使用する必要があります。これは、Cosmos SDKの[object-capabilities](../core/ocap.md)モデルによって保証されます。ストレージキーを保持しているオブジェクトのみがアクセスでき、モジュールの「キーパー」のみがモジュールによって保存されているキーを保持する必要があります。

`Keepers`は通常、` keeper.go`という名前のファイルで定義されます。これには、 `keeper`の型定義とメソッドが含まれています。

`keeper`タイプの定義には、通常、次のものが含まれます。

-**マルチストア内のモジュールのストアへのキー**。
-**その他のモジュール**の「キーパー」を参照してください。 `keeper`が他のモジュールのストレージにアクセスする必要がある場合にのみ必要です(それらの読み取りまたは書き込み)。
-アプリケーションの**コーデック**への参照。ストレージは値として `[] bytes`のみを受け入れるため、` keeper`は、構造を保存する前に構造をマーシャリングするか、構造を取得するときに構造をアンマーシャルする必要があります。

タイプ定義とともに、 `keeper.go`ファイルの次の重要なコンポーネントは` keeper`のコンストラクターである `NewKeeper`です。この関数は、上記で定義されたタイプの新しい「キーパー」を「コーデック」でインスタンス化し、「キー」と他のモジュール「キーパー」への潜在的な参照をパラメーターとして格納します。 `NewKeeper`関数は、[アプリケーションコンストラクター](#constructor-function)から呼び出されます。ファイルの残りの部分は、主にゲッターとセッターである `keeper`のメソッドを定義します。

###コマンドライン、gRPCサービスおよびRESTインターフェース

各モジュールは、コマンドラインコマンド、gRPCサービス、およびRESTルートを定義します。これらは、[application-interfaces](#application-interfaces)を介してエンドユーザーに公開されます。これにより、エンドユーザーは、モジュールで定義されたタイプのメッセージを作成したり、モジュールによって管理されている状態のサブセットを照会したりできます。

#### コマンドラインインターフェイス

通常、[モジュールに関連するコマンド](../building-modules/module-interfaces.md#cli)は、モジュールフォルダー内の `client/cli`という名前のフォルダーで定義されます。 CLIは、コマンドをトランザクションとクエリの2つのカテゴリに分類します。これらは、それぞれ `client/cli/tx.go`と` client/cli/query.go`で定義されています。どちらのコマンドも[Cobraライブラリ](https://github.com/spf13/cobra)に基づいています。

-トランザクションコマンドを使用すると、ユーザーは新しいトランザクションを生成して、ブロックに含め、最終的にステータスを更新できます。モジュールで定義されている[message-types](#message-types)ごとにコマンドを作成する必要があります。このコマンドは、エンドユーザーから提供されたパラメーターを使用してメッセージのコンストラクターを呼び出し、それをトランザクションにラップします。 Cosmos SDKは、署名やその他のトランザクションメタデータの追加を処理します。
-クエリを使用すると、ユーザーはモジュールによって定義された状態のサブセットをクエリできます。 queryコマンドは、クエリを[アプリケーションのクエリルーター](../core/baseapp.md#query-routing)に転送し、[querier](#querier)が提供する適切な `queryRoute`パラメーターにルーティングします。

#### gRPC

[gRPC](https://grpc.io)は、複数の言語をサポートする最新のオープンソースの高性能RPCフレームワークです。これは、外部クライアント(ウォレット、ブラウザー、その他のバックエンドサービスなど)がノードと対話するための推奨される方法です。

各モジュールは、[サービスメソッド](https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition)および[モジュールのProtobuf`クエリ。定義 `ファイルで呼ばれるgRPCエンドポイントを公開できます。 in proto](#grpc-query-services)。サービスメソッドは、その名前、入力パラメーター、および出力応答によって定義されます。次に、モジュールには次のものが必要です。

-`AppModuleBasic`で `RegisterGRPCGatewayRoutes`メソッドを定義して、クライアントのgRPCリクエストをモジュール内の正しいハンドラーに接続します。
-サービスメソッドごとに、対応するハンドラーを定義します。ハンドラーは、gRPCリクエストを処理するために必要なコアロジックを実装し、 `keeper/grpc_query.go`ファイルにあります。

#### gRPCゲートウェイRESTエンドポイント

一部の外部クライアントはgRPCを使用したくない場合があります。この場合、Cosmos SDKはgRPCゲートウェイサービスを提供し、各gRPCサービスを対応するRESTエンドポイントとして公開します。詳細については、[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/)ドキュメントを参照してください。

RESTエンドポイントは、Protobufアノテーションを使用して、gRPCサービスとともにProtobufファイルで定義されます。 RESTクエリを公開するモジュールは、 `rpc`メソッドに` google.api.http`アノテーションを追加する必要があります。デフォルトでは、SDKで定義されているすべてのRESTエンドポイントには、 `/cosmos/`プレフィックスで始まるURLがあります。

Cosmos SDKは、これらのRESTエンドポイントの[Swagger](https://swagger.io/)定義ファイルを生成するための開発エンドポイントも提供します。このエンドポイントは、[`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml)構成ファイルの` api.swagger`キーで有効にできます。 

## アプリケーションプログラムインターフェイス

[インターフェース](#command-line-grpc-services-and-rest-interfaces)エンドユーザーがフルノードクライアントと対話できるようにします。これは、フルノードからデータをクエリするか、フルノードによって中継されて最終的にブロックに含まれる新しいトランザクションを作成して送信することを意味します。

メインインターフェイスは[コマンドラインインターフェイス](../core/cli.md)です。 Cosmos SDKアプリケーションのCLIは、集約アプリケーションで使用される各モジュールで定義された[CLIコマンド](#cli)によって構築されます。アプリケーションのCLIはデーモン( `appd`など)と同じであり、` appd/main.go`という名前のファイルで定義されています。ファイルには次のものが含まれます。

-** `main()`関数**。 `appd`インターフェースを使用してクライアントを構築するために使用されます。この関数は、各コマンドを準備し、ビルドする前にそれらを `rootCmd`に追加します。この関数は、 `appd`のルートディレクトリに、` status`、 `keys`、` config`などの一般的なコマンド、クエリコマンド、txコマンド、 `rest-server`を追加します。
-**クエリコマンド**は、 `queryCmd`関数を呼び出すことで追加されます。この関数は、各アプリケーションモジュールで定義されたクエリコマンド( `main()`関数から `sdk.ModuleClients`配列として渡されます)を含むCobraコマンドと、ブロックなどの他の低レベルのクエリコマンドを返します。バリデータークエリ。 CLIコマンド `appd query[query]`を使用して、queryコマンドを呼び出します。
-`txCmd`関数を呼び出して**取引コマンド**を追加します。 `queryCmd`と同様に、この関数はCobraコマンドを返します。このコマンドには、各アプリケーションモジュールで定義されたtxコマンドと、トランザクション署名やブロードキャストなどの低レベルのtxコマンドが含まれています。 CLIコマンド `appd tx[tx]`を使用して、Txコマンドを呼び出します。

[nameserviceチュートリアル](https://github.com/cosmos/sdk-tutorials/tree/master/nameservice)のアプリケーションマスターコマンドラインファイルの例を参照してください。

+++ https://github.com/cosmos/sdk-tutorials/blob/86a27321cf89cc637581762e953d0c07f8c78ece/nameservice/cmd/nscli/main.go

##依存関係とMakefile

::: 暖かい
`go-grpc v1.34.0`で導入されたパッチにより、gRPCが` gogoproto`ライブラリと互換性がなくなり、[gRPCクエリ](https://github.com/cosmos/cosmos-sdk/issues/8426)パニックが発生しました。したがって、Cosmos SDKでは、go.modにgo-grpc <= v1.33.2をインストールする必要があります。

gRPCが正しく機能するようにするには、アプリケーションの `go.mod`に次の行を追加することを**強くお勧めします**。

```
replace google.golang.org/grpc => google.golang.org/grpc v1.33.2
```

詳細については、[issue#8392](https://github.com/cosmos/cosmos-sdk/issues/8392)を参照してください。
:::

開発者は依存関係マネージャーとプロジェクトの構築方法を自由に選択できるため、この部分はオプションです。つまり、最も一般的に使用されるバージョン管理フレームワークは[`go.mod`](https://github.com/golang/go/wiki/Modules)です。これにより、アプリケーション全体で使用されるすべてのライブラリが正しいバージョンでインポートされます。[nameserviceチュートリアル](https://github.com/cosmos/sdk-tutorials/tree/master/nameservice)の例を参照してください。

+++ https://github.com/cosmos/sdk-tutorials/blob/c6754a1e313eb1ed973c5c91dcc606f2fd288811/go.mod#L1-L18

アプリケーションを構築するには、通常、[Makefile](https://en.wikipedia.org/wiki/Makefile)が使用されます。 Makefileは主に、アプリケーションをビルドするための2つのエントリポイント[`appd`](#node-client)と[` appd`](#application-interface)の前に `go.mod`を実行することを保証します。[nameserviceチュートリアル](https://tutorials.cosmos.network/nameservice/tutorial/00-intro.html)のMakefileの例を参照してください。

+++ https://github.com/cosmos/sdk-tutorials/blob/86a27321cf89cc637581762e953d0c07f8c78ece/nameservice/Makefile

## 次へ{hide}

[トランザクションライフサイクル](./tx-lifecycle.md){hide}の詳細 