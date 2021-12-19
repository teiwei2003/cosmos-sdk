# ADR 002:SDKドキュメント構造

## 環境

CosmosSDKドキュメントの拡張可能な構造が必要です。 現在のドキュメントには、Cosmos SDKに関連しない多くの資料が含まれています。これらの資料は、保守が難しく、ユーザーとしてフォローするのが困難です。

理想的には、次のことを行う必要があります。

-開発フレームワークまたはツールに関連するすべてのドキュメントは、それぞれのgithubリポジトリにあります(sdkリポジトリにはsdkドキュメントが含まれ、ハブリポジトリにはハブドキュメントが含まれ、ローションリポジトリにはローションドキュメントが含まれます)。
-他のすべてのドキュメント(FAQ、ホワイトペーパー、Cosmosに関する高度な資料)はWebサイトに保存されます。

## 決定

次のように、Cosmos SDKgithubリポジトリの `/docs`フォルダーを再構築します。 

```
docs/
├── README
├── intro/
├── concepts/
│   ├── baseapp
│   ├── types
│   ├── store
│   ├── server
│   ├── modules/
│   │   ├── keeper
│   │   ├── handler
│   │   ├── cli
│   ├── gas
│   └── commands
├── clients/
│   ├── lite/
│   ├── service-providers
├── modules/
├── spec/
├── translations/
└── architecture/
```

各サブフォルダー内のファイルは無関係であり、変更される可能性があります。重要なことはスライスすることです:

-`README`:ドキュメントのランディングページ。
-`intro`:紹介資料。目標は、Cosmos SDKの簡単な説明をしてから、必要なリソースに人々を導くことです。 [Cosmos SDKチュートリアル](https://github.com/cosmos/sdk-application-tutorial/)と `godocs`が強調表示されます。
-`concepts`:CosmosSDK抽象化の高レベルの説明が含まれています。特定のコード実装は含まれておらず、頻繁に更新する必要はありません。 **これはインターフェースのAPI仕様ではありません**。 API仕様は `godoc`です。
-`clients`:さまざまなCosmosSDKクライアントに関する仕様と情報が含まれています。
-`spec`:モジュール仕様などが含まれます。
-`modules`: `godocs`とモジュール仕様へのリンクが含まれています。
-`architecture`:現在のアーキテクチャに関連するドキュメントが含まれています。
-`translations`:ドキュメントのさまざまな翻訳が含まれています。

Webサイトのドキュメントのサイドバーには、次のセクションのみが含まれます。

-`Readme`
-`はじめに `
-`コンセプト `
-`顧客 `

「アーキテクチャ」をウェブサイトに表示する必要はありません。

## ステータス

受け入れられました

## 結果

### 目的

-CosmosSDKドキュメントの構成がより明確になりました。
-`/docs`フォルダーには、CosmosSDKとgaia関連の資料のみが含まれるようになりました。後で、CosmosSDK関連の資料のみが含まれるようになります。
-開発者は、PRを開くときに `/docs`フォルダを更新するだけで済みます(たとえば、`/examples`ではありません)。
-再設計されたアーキテクチャのおかげで、開発者はドキュメントで更新する必要があるものを簡単に見つけることができます。
-Webサイトドキュメント用のよりクリーンなvuepressビルド。
-実行可能ドキュメントの作成に役立ちます(https://github.com/cosmos/cosmos-sdk/issues/2611を参照)

### ニュートラル

-廃止されたものの束を `/_attic`フォルダに移動する必要があります。
-`docs/sdk/docs/core`のコンテンツを `concepts`に統合する必要があります。
-現在 `docs`に存在し、新しい構造に適していないすべてのもの(` lotion`、紹介資料、ホワイトペーパーなど)をWebサイトリポジトリに移動する必要があります。
-`DOCS_README.md`を更新します

## 参照する

-https://github.com/cosmos/cosmos-sdk/issues/1460
-https://github.com/cosmos/cosmos-sdk/pull/2695
-https://github.com/cosmos/cosmos-sdk/issues/2611 