# ドキュメントの更新

Cosmos SDKでPRを開いてドキュメントを更新する場合は、次のガイドラインに従ってください。[`CONTRIBUTING.md`](https://github.com/cosmos/cosmos-sdk/tree/master/CONTRIBUTING.md#updating-documentation)

## 国際化

- ドキュメントの翻訳は `docs/<locale>/`フォルダーにあります。ここで、 `<locale>`は特定の言語の言語コードです。 たとえば、中国語の場合は「zh」、韓国語の場合は「ko」、ロシア語の場合は「ru」などです。
- 各 `docs/<locale>/`フォルダは `docs/`内の同じフォルダ構造に従う必要がありますが、次のフォルダのコンテンツのみを翻訳して、それぞれの `docs/<locale>/`フォルダに含める必要があります。
    - `docs/basics/`
    - `docs/building-modules/`
    - `docs/core/`
    - `docs/ibc/`
    - `docs/intro/`
    - `docs/migrations/`
    - `docs/run-node/`
- 各 `docs/<locale>/`フォルダーには、ルートレベル[`README.md`](https://github。 com/cosmos/cosmos-sdk/tree/master/docs/README.md)。 `README.md`で定義されたレイアウトは、ホームページの構築に使用されます。
- 特定のリリースのドキュメントを改訂する場合を除き、常に `master`にあるコンテンツを翻訳してください。 ルートレベルのドキュメントのような翻訳されたドキュメントは、意味的にバージョン管理されています。
- その他の構成オプションについては、[VuePress Internationalization](https://vuepress.vuejs.org/guide/i18n.html)を参照してください。

## ドキュメントビルドワークフロー

Cosmos SDKのドキュメントはhttps://docs.cosmos.network/でホストされており、`/docs`ディレクトリ内のファイルから構築されています。

### 仕組み

`master`ブランチとサポートされている各バージョンタグ(` v0.39`と `v0.42`)の`/docs`ディレクトリの変更をリッスンするCircleCIジョブがあります。 `/docs`ディレクトリ内のファイルを更新すると、Webサイトの展開が自動的にトリガーされます。 内部的には、プライベートWebサイトリポジトリには、そのリポジトリ内のCircleCIジョブによって消費される `makebuild-docs`ターゲットがあります。

## README

[README.md](./README.md)は、リポジトリのREADMEであると同時に、ランディングページのレイアウトの構成でもあります。

## Config.js

[config.js](./.vuepress/config.js)は、サイドバーと目次を生成します
ウェブサイトのドキュメントで。相対リンクの使用との省略に注意してください
ファイル拡張子。外観を改善するための追加機能を利用できます
サイドバーの。

## リンク

**注:**既存のリンクを強く検討してください-両方ともこのディレクトリ内にあります
およびWebサイトのドキュメントへ-ファイルを移動または削除する場合。

相対リンクは、次のことを発見して評価した上で、ほぼすべての場所で使用する必要があります。

### 相対的

現在のファイルと比較して、他のファイルはどこにありますか？

- GitHubとVuePressビルドの両方で機能します
- 混乱する/煩わしい: `../../../../myfile.md`
- ファイルが再シャッフルされるときに、より多くの更新が必要です

### 絶対

リポジトリのルートを指定すると、他のファイルはどこにありますか？

- GitHubで動作し、VuePressビルドでは動作しません
- これははるかに優れています: `/docs/hereitis/myfile.md`
- そのファイルを移動すると、その中のリンクは保持されます(もちろん、ファイルへのリンクは保持されません)。

### 満杯

ファイルまたはディレクトリへの完全なGitHubURL。意味があるときに時々使用されます
ユーザーをGitHubに送信します。

## ローカルで構築する

`docs`ディレクトリにいることを確認し、次のコマンドを実行します。

```sh
rm -rf node_modules
```

このコマンドは、古いバージョンのビジュアルテーマと必要なパッケージを削除します。 このステップはオプションです。

```sh
npm install
```

テーマとすべての依存関係をインストールします。

```sh
npm run serve
```

`pre`フックと` post`フックを実行し、ホットリロードWebサーバーを起動します。 URLについては、このコマンドの出力を参照してください(多くの場合、https://localhost:8080です)。

ドキュメントを静的Webサイトとしてビルドするには、 `npm runbuild`を実行します。 Webサイトは `.vuepress/dist`ディレクトリにあります。

## RPCドキュメントを作成する

まず、リポジトリのルートから `make tools`を実行して、swagger-uiツールをインストールします。

次に、 `swagger.yaml`を手動で編集します。 [ここ](https://github.com/cosmos/cosmos-sdk/blob/master/client/lcd/swagger-ui/swagger.yaml)にあります

最後に、リポジトリのルートから `makeupdate_gaia_lite_docs`を実行します。

## 検索

全文検索を強化するために[Algolia](https://www.algolia.com)を使用しています。 これは、 `config.js`のパブリックAPI検索専用キーと[cosmos_network.json](https://github.com/algolia/docsearch-configs/blob/master/configs/cosmos_network.json)を使用します PRで更新できる構成ファイル。

## 一貫性

ビルドプロセスは(ここに含まれる情報と同様に)同一であるため、このファイルは次のように同期を維持する必要があります。
[Tendermint Coreリポジトリのカウンターパート](https://github.com/tendermint/tendermint/blob/v0.34.0/docs/DOCS_README.md)で可能な限り。

### RPCドキュメントを更新してビルドする

1. ルートディレクトリで次のコマンドを実行して、swagger-ui生成ツールをインストールします。

   ```bash
   make tools
   ```

2. APIドキュメントを編集します
    1. APIドキュメントを手動で直接編集します: `client/lcd/swagger-ui/swagger.yaml`。
    2. [Swagger Editor](https://editor.swagger.io/)内でAPIドキュメントを編集します。 `.yaml`の正しい構造については、この[ドキュメント](https://swagger.io/docs/specification/2-0/basic-structure/)を参照してください。
3. `swagger.yaml`をダウンロードし、fold` client/lcd/swagger-ui`の下にある古い` swagger.yaml`を置き換えます。
4. gaiacliをコンパイルします

   ```bash
   make install
   ```
