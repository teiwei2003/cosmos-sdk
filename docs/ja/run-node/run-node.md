# ノードの実行

アプリケーションの準備が整い、キーリングにデータが入力されたので、ブロックチェーンノードを実行する方法を確認します。 このセクションでは、実行しているアプリケーションを[`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/simapp)と呼び、対応するCLIバイナリ`simd`を使用します。 {synopsis}

## 前提条件の読み

- [CosmosSDKアプリケーションの構造](../basics/app-anatomy.md) {prereq}
- [キーリングの設定](./keyring.md) {prereq}

## チェーンを初期化する

::: 警告
独自のバイナリを作成できることを確認し、スニペットで`simd`をバイナリの名前に置き換えてください。
:::

ノードを実際に実行する前に、チェーン、そして最も重要なのはそのジェネシスファイルを初期化する必要があります。 これは、`init`サブコマンドで実行されます。

```bash
# The argument <moniker> is the custom username of your node, it should be human-readable.
simd init <moniker> --chain-id my-test-chain
```

上記のコマンドは、ノードの実行に必要なすべての構成ファイルと、ネットワークの初期状態を定義するデフォルトのジェネシスファイルを作成します。 これらの構成ファイルはすべてデフォルトで`〜/.simapp`にありますが、`--home`フラグを渡すことでこのフォルダーの場所を上書きできます。

`〜/.simapp`フォルダの構造は次のとおりです。

```bash
.                                   # ~/.simapp
  |- data                           # Contains the databases used by the node.
  |- config/
      |- app.toml                   # Application-related configuration file.
      |- config.toml                # Tendermint-related configuration file.
      |- genesis.json               # The genesis file.
      |- node_key.json              # Private key to use for node authentication in the p2p protocol.
      |- priv_validator_key.json    # Private key to use as a validator in the consensus protocol.
```

## いくつかのデフォルト設定の更新

構成ファイル(例：genesis.json)のフィールド値を変更する場合は、`jq`([installation](https://stedolan.github.io/jq/download/)＆[docs]( https://stedolan.github.io/jq/manual/#Assignment))＆`sed`コマンドでそれを行います。 ここにいくつかの例を示します。

```bash
# to change the chain-id
jq '.chain_id = "testing"' genesis.json > temp.json && mv temp.json genesis.json

# to enable the api server
sed -i '/\[api\]/,+3 s/enable = false/enable = true/' app.toml

# to change the voting_period
jq '.app_state.gov.voting_params.voting_period = "600s"' genesis.json > temp.json && mv temp.json genesis.json

# to change the inflation
jq '.app_state.mint.minter.inflation = "0.300000000000000000"' genesis.json > temp.json && mv temp.json genesis.json
```

## ジェネシスアカウントの追加

チェーンを開始する前に、州に少なくとも1つのアカウントを設定する必要があります。 これを行うには、最初に[test`キーリングバックエンドの下にある`my_validator`という名前の[キーリングに新しいアカウントを作成](./keyring.md＃adding-keys-to-the-keyring)(別の名前を選択して、 別のバックエンド)。

ローカルアカウントを作成したので、先に進んで、チェーンのジェネシスファイルにいくつかの`stake`トークンを付与します。 そうすることで、チェーンがこのアカウントの存在を認識していることも確認できます。

```bash
simd add-genesis-account $MY_VALIDATOR_ADDRESS 100000000000stake
```

`$ MY_VALIDATOR_ADDRESS`は、[keyring](./keyring.md＃adding-keys-to-the-keyring)の`my_validator`キーのアドレスを保持する変数であることを思い出してください。また、CosmosSDKのトークンは`{amount} {denom}`形式であることに注意してください。`amount`は18桁の精度の10進数であり、`denom`は金種キーを持つ一意のトークン識別子です(例：`atom`または`uatom`)。ここでは、`stake`が[`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/)でステーキングに使用されるトークン識別子であるため、`stake`トークンを付与しています。 simapp)。独自のステーキングデノムを持つ独自のチェーンの場合は、代わりにそのトークン識別子を使用する必要があります。

アカウントにいくつかのトークンがあるので、チェーンにバリデーターを追加する必要があります。バリデーターは、チェーンに新しいブロックを追加するために、コンセンサスプロセス([基になるコンセンサスエンジン](../intro/sdk-app-architecture.md＃tendermint)で実装)に参加する特別なフルノードです。どのアカウントもバリデーターオペレーターになる意思を宣言できますが、十分な委任があるアカウントのみがアクティブセットに入ることができます(たとえば、委任が最も多い上位125のバリデーター候補のみがCosmos Hubのバリデーターになります)。このガイドでは、ローカルノード(上記の`init`コマンドで作成)をチェーンのバリデーターとして追加します。バリデーターは、`gentx`と呼ばれるジェネシスファイルに含まれる特別なトランザクションを介してチェーンが最初に開始される前に宣言できます。

```bash
# Create a gentx.
simd gentx my_validator 100000000stake --chain-id my-test-chain --keyring-backend test

# Add the gentx to the genesis file.
simd collect-gentxs
```

`gentx`は3つのことを行います：

1. 作成した`validator`アカウントをバリデーターオペレーターアカウント(つまり、バリデーターを制御するアカウント)として登録します。
2. 提供されたステーキングトークンの「量」を自己委任します。
3. オペレーターアカウントを、ブロックの署名に使用されるTendermintノードのpubkeyにリンクします。`--pubkey`フラグが指定されていない場合、デフォルトで上記の`simdinit`コマンドを使用して作成されたローカルノードpubkeyになります。

`gentx`の詳細については、次のコマンドを使用してください。

```bash
simd gentx --help
```

##`app.toml`と`config.toml`を使用したノードの設定

Cosmos SDKは、`〜/.simapp/config`内に2つの構成ファイルを自動的に生成します。

-`config.toml`：Tendermintの設定に使用されます。詳細については、[Tendermintのドキュメント](https://docs.tendermint.com/master/nodes/configuration.html)をご覧ください。
-`app.toml`：Cosmos SDKによって生成され、状態プルーニング戦略、テレメトリ、gRPCおよびRESTサーバー構成、状態同期などのアプリの構成に使用されます。

両方のファイルはコメントが多いので、ノードを微調整するために直接参照してください。

微調整する設定の例の1つは、`app.toml`内の`minimum-gas-prices`フィールドです。これは、バリデーターノードがトランザクションの処理のために受け入れることができる最小ガス価格を定義します。 チェーンによっては、空の文字列である場合とそうでない場合があります。 空の場合は、必ず`10token`などの値でフィールドを編集してください。そうしないと、ノードは起動時に停止します。 このチュートリアルの目的のために、最小ガス価格を0に設定しましょう。

```toml
 # The minimum gas prices a validator is willing to accept for processing a
 # transaction. A transaction's fees must meet the minimum of any denomination
 # specified in this config (e.g. 0.25token1;0.0001token2).
 minimum-gas-prices = "0stake"
```

## ローカルネットを実行する

すべての設定が完了したので、最終的にノードを起動できます。

```bash
simd start
```

ブロックが入ってくるのが見えるはずです。

前のコマンドを使用すると、単一のノードを実行できます。 このノードとの対話に関する次のセクションではこれで十分ですが、複数のノードを同時に実行して、それらの間でコンセンサスがどのように発生するかを確認することもできます。

単純な方法は、同じコマンドを別々のターミナルウィンドウで再度実行することです。 これは可能ですが、Cosmos SDKでは、[Docker Compose](https://docs.docker.com/compose/)の機能を利用してローカルネットを実行します。 Docker Composeを使用して独自のローカルネットを設定する方法についてのインスピレーションが必要な場合は、Cosmos SDKの[`docker-compose.yml`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/docker-compose.yml)。

## 次へ{非表示}

[ノードとのやり取り](./interact-node.md){hide}について読む