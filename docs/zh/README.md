<!--
layout: homepage
title: Cosmos SDK Documentation
description: Cosmos SDK is the world’s most popular framework for building application-specific blockchains.
sections:
  - title: Introduction
    desc: High-level overview of the Cosmos SDK.
    url: /intro/overview.html
    icon: introduction
  - title: Basics
    desc: Anatomy of a blockchain, transaction lifecycle, accounts and more.
    icon: basics
    url: /basics/app-anatomy.html
  - title: Core Concepts
    desc: Read about the core concepts like baseapp, the store, or the server.
    icon: core
    url: /core/baseapp.html
  - title: Building Modules
    desc: Discover how to build modules for the Cosmos SDK.
    icon: modules
    url: /building-modules/intro.html
  - title: Running a Node
    desc: Running and interacting with nodes using the CLI and API.
    icon: interfaces
    url: /run-node/
  - title: Modules
    desc: Explore existing modules to build your application with.
    icon: specifications
    url: /modules/
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

# Cosmos SDK 文档

## 开始

- **[Cosmos SDK Intro](./intro/overview.md)**:Cosmos SDK 的高级概述。
- **[Starport](https://docs.starport.network/)**:Cosmos SDK 的开发人员友好界面，用于在主权和安全的区块链上搭建、启动和维护任何加密应用程序。
- **[SDK 教程](https://tutorials.cosmos.network/)**:展示如何从头开始构建基于 Cosmos SDK 的区块链的教程，并解释了该过程中的基本 Cosmos SDK 原理。

## 参考文档

- **[Basics](./basics/)**:Cosmos SDK 的基本概念，包括应用的标准剖析、交易生命周期和账户管理。
- **[Core](./core/)**:Cosmos SDK 的核心概念，包括`baseapp`、`store` 或`server`。
- **[Building Modules](./building-modules/)**:模块开发人员的重要概念，如`message`、`keeper`和`querier`。
- **[IBC](./ibc/)**:IBC 协议集成和概念。
- **[运行节点、API、CLI](./run-node/)**:如何使用 CLI 和 API 运行节点并与节点交互。
- **[Migrations](./migrations/)**:更新到新版本 Cosmos SDK 的迁移指南。

## 其他资源

- **[模块目录](../x/)**:Cosmos SDK 模块实现及其各自的文档。
- **[Specifications](./spec/)**:Cosmos SDK 的模块和其他部分的规范。
- **[Cosmos SDK API 参考](https://godoc.org/github.com/cosmos/cosmos-sdk)**:Cosmos SDK 的 Godocs。
- **[REST 和 RPC 端点](https://cosmos.network/rpc/)**:与`gaia` 全节点交互的端点列表。
- **[Rosetta API](./run-node/rosetta.md)**:Rosetta API 集成。

## Cosmos Hub

Cosmos Hub (`gaia`) 文档已移至 [github.com/cosmos/gaia](https://github.com/cosmos/gaia/tree/master/docs)。

## 开发语言

Cosmos SDK 是用 [Golang](https://golang.org/) 编写的，尽管该框架可以用其他语言类似地实现。联系我们以获取有关为另一种语言的实施提供资金的信息。

## 贡献

有关构建过程的详细信息和进行更改时的注意事项，请参阅 [DOCS_README.md](https://github.com/cosmos/cosmos-sdk/blob/master/docs/DOCS_README.md)。 