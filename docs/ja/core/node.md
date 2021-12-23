# ノードクライアント(デーモン)

Cosmos SDKアプリケーションのメインエンドポイントは、フルノードクライアントとも呼ばれるデーモンクライアントです。 フルノードは、ジェネシスファイルから開始してステートマシンを実行します。 同じクライアントを実行しているピアに接続して、トランザクションの受信と中継、提案のブロック、および署名を行います。 フルノードは、Cosmos SDKを使用して定義されたアプリケーションと、ABCIを介してアプリケーションに接続されたコンセンサスエンジンで構成されます。 {まとめ}

## 前提条件の読書

- [SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## `main`関数

Cosmos SDKアプリケーションのフルノードクライアントは、 `main`関数を実行することによって構築されます。クライアントは通常、アプリケーション名に `-d`サフィックスを追加することで名前が付けられ(たとえば、` appd`は `app`という名前のアプリケーションに使用されます)、` main`関数は `./appd/cmdで定義されます。/main。 `ファイルに移動します。この関数を実行すると、一連のコマンドを含む実行可能ファイル「appd」が作成されます。 `app`という名前のアプリケーションの場合、メインコマンドは[` appd start`](＃start-command)で、これはフルノードを開始します。

通常の状況では、開発者は次の構造を使用して `main.go`関数を実装します。

-まず、アプリケーションの[`appCodec`](./encoding.md)をインスタンス化します。
-次に、 `config`を取得し、構成パラメーターを設定します。これには主に、[アドレス](../basics/accounts.md＃addresses)のBech32プレフィックスの設定が含まれます。
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/config.go#L13-L24
-[cobra](https://github.com/spf13/cobra)を使用して、フルノードクライアントのルートコマンドを作成します。その後、 `rootCmd`の` AddCommand() `メソッドを使用して、アプリケーションのすべてのカスタムコマンドを追加します。
-`server.AddCommands() `メソッドを使用して、デフォルトのサーバーコマンドを` rootCmd`に追加します。これらのコマンドは、標準であり、Cosmos SDKレベルで定義されているため、上記で追加したコマンドとは別のものです。これらは、すべてのCosmosSDKベースのアプリケーションで共有する必要があります。最も重要なコマンド[`start`コマンド](＃start-command)が含まれています。
-`executor`を準備して実行します。
   +++ https://github.com/tendermint/tendermint/blob/v0.34.0-rc6/libs/cli/setup.go#L74-L78

デモンストレーションの目的で、「simapp」アプリケーション、CosmosSDKアプリケーションの「main」関数の例を参照してください。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/simd/main.go 

## `start` command

`start`コマンドは、CosmosSDKの`/server`フォルダーで定義されています。 これは、[`main`関数](＃main-function)のフルノードクライアントのルートコマンドに追加され、エンドユーザーによって呼び出されてノードを起動します。 

```bash
# For an example app named "app", the following command starts the full-node.
appd start

# Using the Cosmos SDK's own simapp, the following commands start the simapp node.
simd start
```

念のため、フルノードはネットワーク層、コンセンサス層、アプリケーション層の3つの概念層で構成されています。最初の2つは通常、コンセンサスエンジン(デフォルトではTendermint Core)と呼ばれるエンティティにバンドルされており、3つ目はCosmosSDKを使用して定義されたステートマシンです。現在、Cosmos SDKはデフォルトのコンセンサスエンジンとしてTendermintを使用しています。これは、startコマンドを実行してTendermintノードを起動することを意味します。

`start`コマンドの流れはとても簡単です。まず、 `context`から` config`を取得して `db`を開きます(デフォルトは[` leveldb`](https://github.com/syndtr/goleveldb)インスタンスです)。この `db`には、アプリケーションの最新の既知の状態が含まれています(アプリケーションを初めて起動した場合は空になります。

`db`、` start`コマンドを使用して、 `appCreator`関数を使用してアプリケーションの新しいインスタンスを作成します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L227-L228

`appCreator`は` AppCreator`の署名を満たす関数であることに注意してください。
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/types/app.go#L48-L50

実際には、[アプリケーションコンストラクター](../basics/app-anatomy.md＃constructor-function)は `appCreator`として渡されます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/simd/cmd/root.go#L170-L215

次に、 `app`のインスタンスを使用して、新しいTendermintノードをインスタンス化します。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L235-L244

テンダーミントノードは `app`で作成できます。後者は[` abci.Application`インターフェース](https://github.com/tendermint/tendermint/blob/v0.34.0/abci/types/application.go# L7- L32)( `app`が[` baseapp`](./baseapp.md)を拡張すると仮定)。 「NewNode」メソッドの一部として、Tendermintは、アプリケーションの高さ(つまり、作成からのブロック数)がTendermintノードの高さと等しいことを確認します。これら2つの高さの差は、常に負またはゼロである必要があります。厳密に負の場合、 `NewNode`は、アプリケーションの高さがTendermintノードの高さに達するまでブロックを再生します。最後に、アプリケーションの高さが `0`の場合、Tendermintノードはアプリケーションで[` InitChain`](./baseapp.md＃initchain)を呼び出して、ジェネシスファイルから状態を初期化します。

Tendermintノードがインスタンス化され、アプリケーションと同期されると、次のように開始できます。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L250-L252

起動後、ノードはRPCサーバーとP2Pサーバーを起動し、ダイヤルを開始します。ピアとのハンドシェイク中に、ノードが先行していることを認識すると、追いつくためにすべてのブロックにクエリを実行します。次に、新しいブロックの提案と検証者からのブロック署名が進行するのを待ちます。

## その他のコマンド

ノードを実行して操作する方法については、[実行中のノード、API、およびCLI](../run-node/README.md)ガイドを参照してください。

## 次へ{非表示}

[store](./store.md){hide}を理解する