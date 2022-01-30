# コマンドラインインターフェイス

このドキュメントでは、[** application **](../basics/app-anatomy.md)のコマンドラインインターフェイス(CLI)が高レベルでどのように機能するかについて説明します。 Cosmos SDK [** module **](../building-modules/intro.md)のCLIを実装する別のドキュメントCLIは、[here](../building-modules/module-interfacesにあります。 md#))。 {まとめ}

## コマンドラインインターフェイス

### コマンドの例

CLIを作成するための固定された方法はありませんが、Cosmos SDKモジュールは通常[Cobraライブラリ](https://github.com/spf13/cobra)を使用します。 Cobraを使用してCLIを構築するには、コマンド、パラメーター、およびフラグを定義する必要があります。 [**コマンド**](#commands)トランザクションを作成するための `tx`やアプリケーションをクエリするための` query`など、ユーザーが実行したいアクションを把握します。各コマンドには、特定のトランザクションタイプに名前を付けるために必要なネストされたサブコマンドを含めることもできます。ユーザーは、コインの送付先のアカウント番号などの**パラメーター**と、ガス価格やノードなどのコマンドのさまざまな側面を変更するための[**フラグ**](#flags)も提供します。にブロードキャストします。

以下は、ユーザーが入力できるコマンドの例です。このコマンドは、simappCLIの `simd`と対話して、いくつかのトークンを送信します。

```bash
simd tx bank send $ MY_VALIDATOR_ADDRESS $ RECIPIENT 1000stake --gas auto --gas-prices <gasPrices>
```

最初の4つの文字列は、コマンドを指定します。

-アプリケーション[simd]全体のルートコマンド。
-サブコマンド `tx`。これには、ユーザーがトランザクションを作成できるようにするすべてのコマンドが含まれています。
-サブコマンド `bank`は、コマンドをルーティングするモジュールを示します([` x/bank`](../../x/bank/spec/README.md)はこの場合のモジュールです)。
-トランザクションタイプ `send`。

次の2つの文字列は、ユーザーが送信する[from_address]、受信者の[to_address]、および送信する[amount]のパラメーターです。最後に、コマンドの最後のいくつかの文字列は、ユーザーが支払う意思がある金額を示すオプションのフラグです(トランザクションの実行に使用されるガスの量とユーザーが提供するガスの価格を使用して計算されます)。

CLIは[node](../core/node.md)と対話して、このコマンドを処理します。インターフェイス自体は、main.goファイルで定義されています。

### CLIを構築する

`main.go`ファイルにはrootコマンドを作成するための` main() `関数が必要であり、すべてのアプリケーションコマンドはサブコマンドとしてrootコマンドに追加されます。ルートコマンドは追加で処理されます。

-**構成ファイル(Cosmos SDK構成ファイルなど)を読み取って構成を設定します**。
-** `--chain-id`などのロゴを追加します**。
-**アプリケーションの `MakeCodec()`関数( `simapp`では` MakeTestEncodingConfig`と呼ばれます)を呼び出して、 `codec` **をインスタンス化します。 [`codec`](../core/encoding.md)は、アプリケーションのデータ構造をエンコードおよびデコードするために使用されます-ストレージは` [] byte`のみを永続化できるため、開発者はデータ構造のシリアル化を定義する必要がありますフォーマットまたは使用デフォルト値のProtobuf。
-** [トランザクションコマンド](#transaction-commands)や[query-commands](#query-commands)など、考えられるすべてのユーザーインタラクションのサブコマンドを追加します**。

`main()`関数は、最終的にエグゼキュータと[execute](https://godoc.org/github.com/spf13/cobra#Command.Execute)ルートコマンドを作成します。 [simapp]アプリケーションの[main()]関数の例を参照してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/main.go#L12-L24

ドキュメントの残りの部分では、各ステップで達成する必要があることを詳しく説明し、 `simapp`CLIファイルからのコードの小さな部分を含めます。

## CLIにコマンドを追加する

各アプリケーションCLIは、最初にルートコマンドを作成し、次に `rootCmd.AddCommand()`を使用してサブコマンドを集約することで機能を追加します(通常はさらにネストされたサブコマンドを使用します)。 アプリケーションのユニークな機能のほとんどは、それぞれ[TxCmd]および[QueryCmd]と呼ばれるトランザクションコマンドとクエリコマンドにあります。

### ルートコマンド

rootコマンド([rootCmd]と呼ばれる)は、ユーザーが最初にコマンドラインに入力して、対話するアプリケーションを指定するコマンドです。 コマンドの呼び出しに使用される文字列([使用]フィールド)は、通常、[simd]や[gaiad]など、接尾辞[-d]が付いたアプリケーション名です。 rootコマンドには通常、アプリケーションの基本機能をサポートするための次のコマンドが含まれています。

-Cosmos SDKrpcクライアントツールの** Status **コマンド。接続[`Node`](../core/node.md)のステータスに関する情報を出力します。ノードのステータスには、 `NodeInfo`、` SyncInfo`、および `ValidatorInfo`が含まれます。
-CosmosSDKクライアントツールの**キー** [コマンド](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/keys)(Cosmos SDKを使用するための暗号化ツールを含む)のキー機能には、新しいキーの追加とキーリングへの保存、キーリングに保存されているすべての公開キーの一覧表示、およびキーの削除が含まれます。たとえば、ユーザーは `simd keys add <name>`を入力して新しいキーを追加し、暗号化されたコピーをキーリングに保存し、フラグ `--recover`を使用してシードフレーズまたはフラグ`から複数のキーを取得できます。 --- multisig`それらを組み合わせて、マルチ署名キーを作成します。 `add`キーコマンドの詳細については、コード[こちら](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/client/keys/add.go)を参照してください。 `--keyring-backend`を使用してキーの資格情報を保存する方法の詳細については、[keyring docs](../run-node/keyring.md)を参照してください。
-CosmosSDKサーバーパッケージの** Server **コマンド。これらのコマンドは、ABCI Tendermintアプリケーションを起動するために必要なメカニズムを提供し、アプリケーションを完全に起動するために必要なCLIフレームワーク([cobra](github.com/spf13/cobra)に基づく)を提供する役割を果たします。このパッケージは、2つのコア関数 `StartCmd`と` ExportCmd`を公開します。これらは、それぞれアプリケーションを起動し、状態をエクスポートするためのコマンドを作成します。詳細については、[こちら](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/server)をクリックしてください。
-[** Transaction **](#transaction-commands)コマンド。
-[** Query **](#query-commands)コマンド。

次は、[simapp]アプリケーションの[rootCmd]関数の例です。ルートコマンドをインスタンス化し、[_ persistent_ flag](#flags)と各実行の前に実行される `PreRun`関数を追加し、必要なすべてのサブコマンドを追加します。

+++ https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/simapp/simd/cmd/root.go#L37-L150

`rootCmd`には` initAppConfig() `という関数があり、アプリケーションのカスタム構成を設定するのに便利です。
デフォルトでは、アプリケーションはCosmos SDKのTendermintアプリケーション構成テンプレートを使用します。これは、 `initAppConfig()`でオーバーライドできます。
これは、デフォルトの `app.toml`テンプレートをオーバーライドするサンプルコードです。

+++ https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/simapp/simd/cmd/root.go#L84-L117

`initAppConfig()`では、デフォルトのCosmos SDK [サーバー構成](https://github.com/cosmos/cosmos-sdk/blob/4eea4cafd3b8b1c2cd493886db524500c9dd745c/server/config/config.go#L199)をオーバーライドすることもできます。 例として、[min-gas-prices]構成があります。これは、トランザクションを処理するときにバリデーターが受け入れることができる最小ガス価格を定義します。 デフォルトでは、CosmosSDKはこのパラメーターを `" "`(空の文字列)に設定します。これにより、すべてのバリデーターが強制的に `app.toml`を調整し、空でない値を設定します。そうでない場合、ノードは起動時に停止します。 これはバリデーターに最適なUXではない可能性があるため、チェーン開発者はこの `initAppConfig()`関数でバリデーターのデフォルトの `app.toml`値を設定できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/aa9b055ddb46aacd4737335a92d0b8a82d577341/simapp/simd/cmd/root.go#L101-L116

ルートレベルの `status`および` keys`サブコマンドは、ほとんどのアプリケーションで一般的であり、アプリケーションのステータスとは相互作用しません。 アプリケーションのほとんどの機能(ユーザーが実際にアプリケーションで実行できること)は、その `tx`および` query`コマンドによって有効になります。

### 取引注文

[Transactions](./transactions.md)は、[`Msg`s](../building-modules/messages-and-queries.md#messages)をラップするオブジェクトであり、状態の変更をトリガーします。 CLIインターフェースを使用してトランザクションを作成できるようにするために、通常、関数 `txCmd`が` rootCmd`に追加されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L86-L92

この `txCmd`関数は、エンドユーザーが利用できるすべてのトランザクションをアプリケーションに追加します。 これには通常、次のものが含まれます。

-**署名コマンド**は[`auth`](../../x/auth/spec/README.md)モジュールから取得され、トランザクションでメッセージに署名するために使用されます。 マルチ署名を有効にするには、 `auth`モジュールの` MultiSign`コマンドを追加します。 すべてのトランザクションが有効であるためにはある種の署名が必要であるため、すべてのアプリケーションには署名コマンドが必要です。
-**Cosmos SDKクライアントツールからのブロードキャストコマンド**、トランザクションのブロードキャストに使用されます。
-**すべての[モジュールトランザクションコマンド](../building-modules/module-interfaces.md#transaction-commands)** [Basic Module Manager](../building- modules/module -managerを使用したアプリケーションの依存関係.md#basic-manager) `AddTxCommands()`関数。

これは、[simapp]アプリケーションからこれらのサブコマンドを集約する[txCmd]の例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L123-L149

### クエリコマンド

[**Queries**](../building-modules/messages-and-queries.md#queries)は、ユーザーがアプリケーションの状態に関する情報を取得できるようにするオブジェクトです。 CLIインターフェースを使用してトランザクションを作成できるようにするために、通常、関数 `txCmd`が` rootCmd`に追加されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L86-L92

この `queryCmd`関数は、エンドユーザーが利用できるすべてのクエリをアプリケーションに追加します。 これには通常、次のものが含まれます。

-**QueryTx**および/またはその他のトランザクションクエリコマンド]は `auth`モジュールから取得され、ユーザーはハッシュ値、タグリスト、またはブロックの高さを入力してトランザクションを検索できます。 これらのクエリにより、ユーザーはトランザクションがブロックに含まれているかどうかを確認できます。
-`auth`モジュールからの**アカウントコマンド**。特定のアドレスのアカウントのステータス(アカウントの残高など)を表示します。
-**バリデーターコマンド**は、指定された高さのバリデーターセットを表示するCosmos SDKrpcクライアントツールから取得されます。
-Cosmos SDK rpcクライアントツールの**ブロックコマンド**を使用して、特定の高さのブロックデータを表示します。
-**すべての[モジュールクエリコマンド](../building-modules/module-interfaces.md#query-commands)**アプリケーションの依存関係、[Basic Module Manager](../building-modules/module-managerを使用してください。 md#basic-manager) `AddQueryCommands()`関数。

これは、[simapp]アプリケーションからのサブコマンドを集約する[queryCmd]の例です。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L99-L121

## サイン

フラグはコマンドを変更するために使用されます。開発者はCLIを使用して、コマンドを `flags.go`ファイルに含めることができます。ユーザーは、コマンドに明示的に含めるか、[`app.toml`](../run-node/run-node.md#configuring-the-node-using-apptoml)で事前に構成することができます。通常、事前設定されたフラグには、ユーザーが操作したいブロックチェーンの[--node]と[--chain-id]が含まれます。

コマンドに追加された_persistent_フラグ(_local_フラグではなく)は、そのすべてのサブコマンドをオーバーライドします。サブコマンドは、これらのフラグの構成値を継承します。さらに、すべてのフラグには、コマンドに追加されたときにデフォルト値があります。一部はオフオプションですが、その他は空の値であり、ユーザーが有効なコマンドを作成するためにオーバーライドする必要があります。フラグを明示的に_required_としてマークして、ユーザーが値を指定しない場合にエラーを自動的にスローすることができますが、誤って欠落しているフラグを別の方法で処理することもできます。

フラグはコマンドに直接追加され(通常、モジュールコマンドを定義する[module CLIファイル](../building-modules/module-interfaces.md#flags)にあります)、 `rootCmd以外のフラグは必要ありません。 `永続フラグ追加はアプリケーションレベルで追加されます。通常、[-chain-id](アプリケーションが属するブロックチェーンの一意の識別子)は、rootコマンドに_persistent_フラグを追加します。このフラグの追加は、 `main()`関数で行うことができます。このアプリケーションCLIのコマンド間でチェーンIDを変更してはならないため、このフラグを追加することは理にかなっています。以下は、[simapp]アプリケーションの例です。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L118-L119

## 環境変数

各フラグは、独自の名前付き環境変数にバインドされています。 その場合、環境変数の名前は2つの部分で構成されます。大文字の[basename]の後にロゴのロゴ名が続きます。 `-`は` _`に置き換える必要があります。 たとえば、ベース名が[GAIA]であるアプリケーションのフラグ[--home]は[GAIA_HOME]にバインドされます。 これにより、通常の操作で入力されるフラグの数を減らすことができます。 たとえば、次の代わりに:

```sh
gaia --home=./--node=<node address> --chain-id="testchain-1" --keyring-backend=test tx ... --from=<key name>
```

これはより便利になります:

```sh
# define env variables in .env, .envrc etc
GAIA_HOME=<path to home>
GAIA_NODE=<node address>
GAIA_CHAIN_ID="testchain-1"
GAIA_KEYRING_BACKEND="test"

# and later just use
gaia tx ... --from=<key name>
```

## 構成

アプリケーションのルートコマンドは、 `PersistentPreRun()` cobraコマンド属性を使用してコマンドを実行することが重要です。これにより、すべてのサブコマンドがサーバーとクライアントのコンテキストにアクセスできるようになります。 これらのコンテキストは、最初はデフォルト値に設定されており、それぞれの[PersistentPreRun()]関数で変更できますが、スコープはコマンドに限定されます。 `client.Context`には通常、[デフォルト]の値が事前に入力されていることに注意してください。これは、必要に応じてすべてのコマンドを継承およびオーバーライドする場合に役立ちます。

`simapp`の` PersistentPreRun() `関数の例を次に示します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/simd/cmd/root.go#L54-L60

`SetCmdClientContextHandler`呼び出しは、` ReadPersistentCommandFlags`を介して永続性フラグを読み取り、 `client.Context`を作成して、ルートコマンドの` Context`に設定します。

`InterceptConfigsPreRunHandler`呼び出しは、バイパーテキスト、デフォルトの` server.Context`、およびレコーダーを作成し、ルートコマンドの `Context`に設定します。 `server.Context`は変更され、内部の` interceptConfigs`呼び出しを介してディスクに保存されます。この呼び出しは、提供されたメインパスに従ってTendermint構成を読み取りまたは作成します。 さらに、 `interceptConfigs`は、アプリケーション構成` app.toml`を読み取ってロードし、それを `server.Context`viperテキストにバインドします。 これは重要であるため、アプリケーションはCLIフラグだけでなく、このファイルによって提供されるアプリケーション構成値にもアクセスできます。

## 次へ{hide}

[events](./events.md){hide}を理解する