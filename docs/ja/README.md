<!--
レイアウト：ホームページ
タイトル：CosmosSDKドキュメント
説明：Cosmos SDKは、アプリケーション固有のブロックチェーンを構築するための世界で最も人気のあるフレームワークです。
セクション：
  -タイトル：はじめに
    desc：CosmosSDKの概要。
    url：/intro/overview.html
    アイコン：はじめに
  -タイトル：基本
    desc：ブロックチェーン、トランザクションライフサイクル、アカウントなどの構造。
    アイコン：基本
    url：/basics/app-anatomy.html
  -タイトル：コアコンセプト
    desc：baseapp、ストア、サーバーなどのコアコンセプトについて読んでください。
    アイコン：コア
    url：/core/baseapp.html
  -タイトル：モジュールの構築
    desc：CosmosSDKのモジュールを構築する方法をご覧ください。
    アイコン：モジュール
    url：/building-modules/intro.html
  -タイトル：ノードの実行
    desc：CLIとAPIを使用してノードを実行および操作します。
    アイコン：インターフェース
    url：/ run-node /
  -タイトル：モジュール
    desc：アプリケーションを構築するための既存のモジュールを調べます。
    アイコン：仕様
    url：/ modules /
スタック：
  -タイトル：コスモスハブ
    desc：CosmosNetwork上の数千の相互接続されたブロックチェーンの最初のもの。
    色：「＃BA3FD9」
    ラベル：ハブ
    url：http：//hub.cosmos.network
  -タイトル：テンダーミントコア
    desc：Cosmos SDKを強化する、ブロックチェーンを構築するための主要なBFTエンジン。
    色：「＃00BB00」
    ラベル：コア
    url：http：//docs.tendermint.com
フッター：
  ニュースレター：false
余談：false
-->

# CosmosSDKドキュメント

## はじめに

- **[Cosmos SDK Intro](./intro/overview.md)**:CosmosSDKの概要。
- **[Starport](https://docs.starport.network/)**:Cosmos SDKの開発者向けのインターフェイスは、暗号化されたアプリケーションをソブリンで安全なブロックチェーン上で構築、起動、および保守するために使用されます。
- **[SDK Tutorials](https://tutorials.cosmos.network/)**:Cosmos SDKに基づいてブロックチェーンを最初から構築する方法を示し、その過程でCosmosSDKの基本原則を説明するチュートリアル。

## 参考資料

- **[Basics](./basics/)**:アプリケーション標準分析、トランザクションライフサイクル、アカウント管理など、CosmosSDKの基本概念。
- **[Core](./core/)**:`baseapp`、` store`、または `server`を含むCosmosSDKのコアコンセプト。
- **[Building Modules](./building-modules/)**:`message`、` keeper`、 `querier`などのモジュール開発者にとって重要な概念。
- **[IBC](./ibc/)**:IBCプロトコルの統合と概念。
- **[运行节点、API、CLI](./run-node/)**:ノードを実行し、CLIとAPIを使用してノードと対話する方法。
- **[Migrations](./migrations/)**:CosmosSDKの新しいバージョンに更新するための移行ガイド。

## その他のリソース

- **[Module Directory](../x/)**:CosmosSDKモジュールの実装とそれぞれのドキュメント。
- **[Specifications](./spec/)**:CosmosSDKのモジュールおよびその他の部分の仕様。
- **[Cosmos SDK API 参照](https://godoc.org/github.com/cosmos/cosmos-sdk)**:CosmosSDKのGodocs。
- **[RESTとRPCエンドポイント](https://cosmos.network/rpc/)**:`gaia`フルノードと対話するエンドポイントのリスト。
- **[Rosetta API](./run-node/rosetta.md)**:RosettaAPI統合。

## コスモスハブ

Cosmos Hub（ `gaia`）のドキュメントは[github.com/cosmos/gaia]（https://github.com/cosmos/gaia/tree/master/docs）に移動しました。

## 言語

Cosmos SDKは[Golang]（https://golang.org/）で記述されていますが、フレームワークは他の言語でも同様に実装できます。 別の言語での実装への資金提供については、お問い合わせください。

## 貢献する

ビルドプロセスの詳細と変更を行う際の考慮事項については、[DOCS_README.md]（https://github.com/cosmos/cosmos-sdk/blob/master/docs/DOCS_README.md）を参照してください。