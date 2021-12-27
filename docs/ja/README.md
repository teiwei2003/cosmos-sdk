<!--
layout: homepage
title: Cosmos SDK Documentation
description: Cosmos SDK is the world’s most popular framework for building application-specific blockchains.
sections:
  - title: 介绍
    desc: 高级介绍 Cosmos SDK.
    url: /zh/intro/overview.html
    icon: introduction
  - title: 基础
    desc: Anatomy of a blockchain, transaction lifecycle, accounts and more.
    icon: basics
    url: /zh/basics/app-anatomy.html
  - title: 核心概念
    desc: Read about the core concepts like baseapp, the store, or the server.
    icon: core
    url: /zh/core/baseapp.html
  - title: 模块构筑
    desc: Discover how to build modules for the Cosmos SDK.
    icon: modules
    url: /zh/building-modules/intro.html
  - title: 运行一个节点
    desc: Running and interacting with nodes using the CLI and API.
    icon: interfaces
    url: /zh/run-node/
  - title: 模块
    desc: Explore existing modules to build your application with.
    icon: specifications
    url: /zh/modules/
stack:
  - title: Cosmos Hub
    desc: The first of thousands of interconnected blockchains on the Cosmos Network.
    color: "#BA3FD9"
    label: hub
    url: http://hub.cosmos.network
  - title: Tendermint Core
    desc: The leading BFT engine for building blockchains, powering Cosmos SDK.
    color: "#00BB00"
    label: core
    url: http://docs.tendermint.com
footer:
  newsletter: false
aside: false
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