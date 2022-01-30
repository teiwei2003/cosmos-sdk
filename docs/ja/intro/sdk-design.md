# CosmosSDKの主要コンポーネント

Cosmos SDKは、Tendermint上での安全なステートマシンの開発を容易にするフレームワークです。 Cosmos SDKのコアは、Golangでの[ABCI](./sdk-app-architecture.md#abci)のプロトタイプ実装です。 データを永続化するための[`multistore`](../core/store.md#multistore)と、トランザクションを処理するための[` router`](../core/baseapp.md#routing)が付属しています。

以下は、Cosmos SDK上に構築されたアプリケーションが、[DeliverTx]を介してTendermintから送信するときにトランザクションを処理する方法の簡略図です。

1. Tendermintコンセンサスエンジンから受信した `transactions`をデコードします(Tendermintは` [] bytes`のみを処理することに注意してください)。
2. `transactions`から` messages`を抽出し、基本的な整合性チェックを実行します。
3. 各メッセージを適切なモジュールにルーティングして、処理できるようにします。
4. ステータス変更を送信します。

## `基本的なアプリケーション`

`baseapp`は、CosmosSDKアプリケーションの定型的な実装です。 基盤となるコンセンサスエンジンとの接続を処理するためのABCIの実装が付属しています。 通常、Cosmos SDKアプリケーションは、[`app.go`](../basics/app-anatomy.md#core-application-file)に[baseapp]を埋め込むことで拡張します。 CosmosSDKアプリケーションチュートリアルの例を参照してください。

+++ https://github.com/cosmos/sdk-tutorials/blob/c6754a1e313eb1ed973c5c91dcc606f2fd288811/app.go#L72-L92

`baseapp`の目標は、ストレージと拡張可能なステートマシンの間に安全なインターフェイスを提供すると同時に、可能な限り少ないステートマシンを定義することです(ABCIに忠実)。

`baseapp`の詳細については、[こちら](../core/baseapp.md)をクリックしてください。

## 複数のストレージ

Cosmos SDKは、永続状態用の[`multistore`](../core/store.md#multistore)を提供します。 multistoreを使用すると、開発者は任意の数の[`KVStores`](../core/store.md#base-layer-kvstores)を宣言できます。 これらの `KVStores`は、値として` [] byte`タイプのみを受け入れるため、カスタム構造は、保存する前に[a codec](../core/encoding.md)でマーシャリングする必要があります。

マルチストレージ抽象化は、状態を異なるコンパートメントに分割するために使用され、各コンパートメントは独自のモジュールによって管理されます。 マルチストアの詳細については、[ここをクリック](../core/store.md#multistore)をクリックしてください。

## モジュール

Cosmos SDKの力は、そのモジュール性にあります。 Cosmos SDKアプリケーションは、相互運用可能なモジュールのセットを集約することによって構築されます。 各モジュールは状態のサブセットを定義し、独自のメッセージ/トランザクションプロセッサを含み、CosmosSDKは各メッセージをそれぞれのモジュールにルーティングする役割を果たします。

有効なブロックで受信されたときに、各フルノードアプリケーションがトランザクションを処理する方法の簡略化されたビューを次に示します。

```
                                      +
                                      |
                                      |  Transaction relayed from the full-node's
                                      |  Tendermint engine to the node's application
                                      |  via DeliverTx
                                      |
                                      |
                +---------------------v--------------------------+
                |                 APPLICATION                    |
                |                                                |
                |     Using baseapp's methods: Decode the Tx,    |
                |     extract and route the message(s)           |
                |                                                |
                +---------------------+--------------------------+
                                      |
                                      |
                                      |
                                      +---------------------------+
                                                                  |
                                                                  |
                                                                  |  Message routed to
                                                                  |  the correct module
                                                                  |  to be processed
                                                                  |
                                                                  |
+----------------+  +---------------+  +----------------+  +------v----------+
|                |  |               |  |                |  |                 |
|  AUTH MODULE   |  |  BANK MODULE  |  | STAKING MODULE |  |   GOV MODULE    |
|                |  |               |  |                |  |                 |
|                |  |               |  |                |  | Handles message,|
|                |  |               |  |                |  | Updates state   |
|                |  |               |  |                |  |                 |
+----------------+  +---------------+  +----------------+  +------+----------+
                                                                  |
                                                                  |
                                                                  |
                                                                  |
                                       +--------------------------+
                                       |
                                       | Return result to Tendermint
                                       | (0=Ok, 1=Err)
                                       v
```

各モジュールは、小さなステートマシンと見なすことができます。 開発者は、モジュールによって処理される状態のサブセットと、状態を変更するためのカスタムメッセージタイプを定義する必要があります(*注:* `messages`は` transactions`から `baseapp`によって抽出されます)。 一般的に、各モジュールは、定義した状態のサブセットを永続化するために、 `multistore`で独自の` KVStore`を宣言します。 ほとんどの開発者は、独自のモジュールを構築するときに、他のサードパーティモジュールにアクセスする必要があります。 Cosmos SDKはオープンフレームワークであるため、一部のモジュールは悪意のあるものである可能性があります。つまり、モジュール間の相互作用について推論するにはセキュリティの原則が必要です。 これらの原則は、[object-capabilities](../core/ocap.md)に基づいています。 実際には、これは、各モジュールに他のモジュールのアクセス制御リストを保持させる代わりに、各モジュールが[キーパー]と呼ばれる特別なオブジェクトを実装し、他のモジュールに渡して事前定義された機能のセットを付与できることを意味します。

Cosmos SDKモジュールは、CosmosSDKの `x/`フォルダーで定義されています。 いくつかのコアモジュールは次のとおりです。

-`x/auth`:アカウントと署名を管理するために使用されます。
-`x/bank`:トークンとトークン転送を有効にするために使用されます。
-`x/stakeing` + `x/slashing`:プルーフオブステークブロックチェーンを構築するために使用されます。

`x/`にすでに存在するモジュールに加えて、誰でもアプリケーションでそれらを使用できます。CosmosSDKを使用すると、独自のカスタムモジュールを構築できます。 [チュートリアルの例](https://tutorials.cosmos.network/)を確認できます。

## 次へ{hide}

[Cosmos SDKアプリケーションの構造](../basics/app-anatomy.md){hide}の詳細