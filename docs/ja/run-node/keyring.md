# キーリングの設定

このドキュメントでは、[**アプリケーション**](../basics/app-anatomy.md)のキーリングとそのさまざまなバックエンドを構成して使用する方法について説明します。 {synopsis}

キーリングは、ノードとの対話に使用される秘密/公開キーペアを保持します。 たとえば、ブロックチェーンノードを実行する前に検証キーを設定して、ブロックに正しく署名できるようにする必要があります。 秘密鍵は、ファイルやオペレーティングシステム自体の鍵ストレージなど、[バックエンド]と呼ばれるさまざまな場所に保存できます。

## キーリングに使用可能なバックエンド

v0.38.0リリース以降、CosmosSDKには新しいキーリングの実装が付属しています
これは、暗号化キーを安全な方法で管理するための一連のコマンドを提供します。 ザ
新しいキーリングは複数のストレージバックエンドをサポートしますが、その一部はで利用できない場合があります
すべてのオペレーティングシステム。

### `os`バックエンド

`os`バックエンドは、キーストレージを処理するためにオペレーティングシステム固有のデフォルトに依存しています安全に。通常、オペレーティングシステムのクレデンシャルサブシステムは、パスワードプロンプトを処理します。
秘密鍵の保管、およびユーザーのパスワードポリシーに従ったユーザーセッション。 ここ最も人気のあるオペレーティングシステムとそれぞれのパスワードマネージャーのリストです。

- macOS (since Mac OS 8.6): [Keychain](https://support.apple.com/en-gb/guide/keychain-access/welcome/mac)
- Windows: [Credentials Management API](https://docs.microsoft.com/en-us/windows/win32/secauthn/credentials-management)
- GNU/Linux:
    - [libsecret](https://gitlab.gnome.org/GNOME/libsecret)
    - [kwallet](https://api.kde.org/frameworks/kwallet/html/index.html)

デフォルトのデスクトップ環境としてGNOMEを使用するGNU/Linuxディストリビューションには、通常、[Seahorse](https://wiki.gnome.org/Apps/Seahorse)が付属しています。 KDEベースのディストリビューションのユーザーには、通常、[KDE Wallet Manager](https://userbase.kde.org/KDE_Wallet_Manager)が提供されます。
前者は実際には[libsecret]の便利なフロントエンドですが、後者は[kwallet]クライアントです。

オペレーティングシステムのデフォルトのクレデンシャルマネージャーは、ユーザーの最も一般的なニーズを満たし、セキュリティを損なうことなく快適なエクスペリエンスを提供するように設計されているため、`os`がデフォルトのオプションです。

ヘッドレス環境で推奨されるバックエンドは、`file`と`pass`です。

### `file`バックエンド

`file`バックエンドは、v0.38.1より前に使用されていたキーベースの実装によく似ています。 暗号化されたキーリングをアプリの構成ディレクトリに保存します。
このキーリングは、アクセスされるたびにパスワードを要求します。これは、1つのコマンドで複数回発生し、パスワードプロンプトが繰り返される場合があります。
bashスクリプトを使用して`file`オプションを使用してコマンドを実行する場合は、複数のプロンプトに次の形式を使用することをお勧めします。 

```sh
# assuming that KEYPASSWD is set in the environment
$ gaiacli config keyring-backend file                             # use file backend
$ (echo $KEYPASSWD; echo $KEYPASSWD) | gaiacli keys add me        # multiple prompts
$ echo $KEYPASSWD | gaiacli keys show me                          # single prompt
```

::: tip
The first time you add a key to an empty keyring, you will be prompted to type the password twice.
:::

### `pass`バックエンド

`pass`バックエンドは、[pass](https://www.passwordstore.org/)ユーティリティを使用して、キーの機密データとメタデータのディスク上の暗号化を管理します。
キーは、アプリ固有のディレクトリ内の`gpg`暗号化ファイル内に保存されます。
`pass`は、最も一般的なUNIXオペレーティングシステムとGNU/Linuxディストリビューションで使用できます。
ダウンロードとインストールの方法については、マニュアルページを参照してください。

::: ヒント
**pass**は、暗号化に[GnuPG](https://gnupg.org/)を使用します。`gpg`は、実行時に`gpg-agent`デーモンを自動的に呼び出します。このデーモンは、GnuPGクレデンシャルのキャッシュを処理します。
クレデンシャルTTLやクレデンシャルなどのキャッシュパラメータを設定する方法の詳細については、`gpg-agent`のマニュアルページを参照してください。
パスフレーズの有効期限。
:::

パスワードストアは、最初に使用する前に設定する必要があります。 

```sh
pass init <GPG_KEY_ID>
```

`<GPG_KEY_ID>`をGPGキーIDに置き換えます。 個人のGPGキーまたは代替を使用できます。パスワードストアを暗号化するために特に使用する場合があります。

### `kwallet`バックエンド

`kwallet`バックエンドは`KDE Wallet Manager`を使用します。これは、KDEをデフォルトのデスクトップ環境として出荷するGNU/Linuxディストリビューションにデフォルトでインストールされます。
詳細については、[KWalletハンドブック(https://docs.kde.org/stable5/en/kdeutils/kwallet5/index.html)を参照してください。

### `test`バックエンド

`test`バックエンドは、`file`バックエンドのパスワードなしのバリエーションです。 キーは暗号化されずにディスクに保存されます。

**テスト目的でのみ提供されます。`test`バックエンドを本番環境で使用することはお勧めしません**。

### `メモリ`バックエンド

`memory`バックエンドはキーをメモリに保存します。 プログラムが終了すると、キーはすぐに削除されます。

**テスト目的でのみ提供されます。`memory`バックエンドを本番環境で使用することはお勧めしません**。

## キーリングにキーを追加する

::: 警告
独自のバイナリを作成できることを確認し、スニペットで`simd`をバイナリの名前に置き換えてください。
:::

Cosmos SDKを使用して開発されたアプリケーションには、`keys`サブコマンドが付属しています。 このチュートリアルでは、テストと教育の目的でCosmosSDKを使用して構築されたアプリケーションである`simd`CLIを実行しています。 詳細については、[`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/simapp)を参照してください。

キーコマンドのヘルプには`simd keys`を使用し、特定のサブコマンドの詳細については`simd keys [command]-help`を使用できます。

::: ヒント
`simdcompletion`コマンドでオートコンプリートを有効にすることもできます。 たとえば、bashセッションの開始時に、`を実行します。 <(simd complete)`、およびすべての`simd`サブコマンドは自動補完されます。
:::

キーリングに新しいキーを作成するには、`<key_name>`引数を指定して`add`サブコマンドを実行します。 このチュートリアルでは、`test`バックエンドのみを使用し、新しいキー`my_validator`を呼び出します。 このキーは次のセクションで使用されます。

```bash
$ simd keys add my_validator --keyring-backend test

# Put the generated address in a variable for later use.
MY_VALIDATOR_ADDRESS=$(simd keys show my_validator -a --keyring-backend test)
```

このコマンドは、新しい24ワードのニーモニックフレーズを生成し、それを関連するバックエンドに保持し、キーペアに関する情報を出力します。 このキーペアを使用して価値のあるトークンを保持する場合は、ニーモニックフレーズを安全な場所に書き留めてください。

デフォルトでは、キーリングは`secp256k1`キーペアを生成します。 キーリングは`ed25519`キーもサポートします。これは、`--algoed25519`フラグを渡すことで作成できます。 もちろん、キーリングは両方のタイプのキーを同時に保持でき、CosmosSDKの`x/auth`モジュール(特に[AnteHandlers](../core/baseapp.md#antehandler))は、これら2つの公開鍵アルゴリズムをネイティブにサポートします。

## 次へ{hide}

[ノードの実行](./run-node.md){hide}について読む