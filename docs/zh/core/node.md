# 节点客户端(守护进程)

Cosmos SDK 应用程序的主要端点是守护程序客户端，也称为全节点客户端。全节点运行状态机，从创世文件开始。它连接到运行相同客户端的对等点，以接收和中继交易、区块提案和签名。全节点由使用 Cosmos SDK 定义的应用程序和通过 ABCI 连接到应用程序的共识引擎组成。 {概要}

## 先决条件阅读

- [SDK 应用剖析](../basics/app-anatomy.md) {prereq}

## `main` 函数

任何 Cosmos SDK 应用程序的全节点客户端都是通过运行 `main` 函数来构建的。客户端通常通过在应用程序名称后附加`-d` 后缀来命名(例如`appd` 用于名为`app` 的应用程序)，并且`main` 函数定义在`./appd/cmd/main 中。去`文件。运行此函数会创建一个带有一组命令的可执行文件“appd”。对于名为`app`的应用程序，主要命令是[`appd start`](#start-command)，它启动全节点。

一般情况下，开发者会用以下结构实现`main.go`函数:

- 首先，为应用程序实例化一个 [`appCodec`](./encoding.md)。
- 然后，检索`config`并设置配置参数。这主要涉及为 [addresses](../basics/accounts.md#addresses) 设置 Bech32 前缀。
  +++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/types/config.go#L13-L24
- 使用[cobra](https://github.com/spf13/cobra)，创建全节点客户端的root命令。之后，使用`rootCmd`的`AddCommand()`方法添加应用程序的所有自定义命令。
- 使用 `server.AddCommands()` 方法将默认服务器命令添加到 `rootCmd`。这些命令与上面添加的命令分开，因为它们是标准的并在 Cosmos SDK 级别定义。它们应该由所有基于 Cosmos SDK 的应用程序共享。它们包括最重要的命令:[`start` 命令](#start-command)。
- 准备并执行`executor`。
   +++ https://github.com/tendermint/tendermint/blob/v0.34.0-rc6/libs/cli/setup.go#L74-L78

请参阅“simapp”应用程序中的“main”函数示例，用于演示目的的 Cosmos SDK 应用程序:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/simd/main.go 

## `start` command

`start` 命令定义在 Cosmos SDK 的 `/server` 文件夹中。 它被添加到 [`main` 函数](#main-function) 中的全节点客户端的 root 命令中，并由最终用户调用以启动他们的节点: 

```bash
# For an example app named "app", the following command starts the full-node.
appd start

# Using the Cosmos SDK's own simapp, the following commands start the simapp node.
simd start
```

提醒一下，全节点由三个概念层组成:网络层、共识层和应用层。前两个通常捆绑在一个称为共识引擎(默认为 Tendermint Core)的实体中，而第三个是在 Cosmos SDK 的帮助下定义的状态机。目前，Cosmos SDK 使用 Tendermint 作为默认共识引擎，这意味着执行 start 命令来启动 Tendermint 节点。

`start` 命令的流程非常简单。首先，它从`context` 中检索`config` 以打开`db`(默认为[`leveldb`](https://github.com/syndtr/goleveldb) 实例)。这个 `db` 包含应用程序的最新已知状态(如果应用程序从第一次启动，则为空。

使用 `db`，`start` 命令使用 `appCreator` 函数创建应用程序的新实例:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L227-L228

请注意，`appCreator` 是一个满足 `AppCreator` 签名的函数:
+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/types/app.go#L48-L50

在实践中，[应用程序的构造函数](../basics/app-anatomy.md#constructor-function) 作为`appCreator` 传递。

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/simapp/simd/cmd/root.go#L170-L215

然后，`app` 的实例用于实例化一个新的 Tendermint 节点:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L235-L244

Tendermint节点可以用`app`创建，因为后者满足[`abci.Application`接口](https://github.com/tendermint/tendermint/blob/v0.34.0/abci/types/application.go# L7-L32)(假设 `app` 扩展了 [`baseapp`](./baseapp.md))。作为“NewNode”方法的一部分，Tendermint 确保应用程序的高度(即自创世以来的块数)等于 Tendermint 节点的高度。这两个高度之间的差值应始终为负值或为零。如果严格为负，`NewNode` 将重放区块，直到应用程序的高度达到 Tendermint 节点的高度。最后，如果应用程序的高度为`0`，Tendermint 节点将调用应用程序上的[`InitChain`](./baseapp.md#initchain) 以从创世文件中初始化状态。

一旦 Tendermint 节点被实例化并与应用程序同步，就可以启动该节点:

+++ https://github.com/cosmos/cosmos-sdk/blob/v0.40.0-rc3/server/start.go#L250-L252

启动后，节点将引导其 RPC 和 P2P 服务器并开始拨号。在与对等方握手期间，如果节点意识到他们领先，它将顺序查询所有块以赶上。然后，它将等待来自验证者的新区块提议和区块签名以取得进展。

## 其他命令

要了解如何具体运行节点并与之交互，请参阅我们的 [运行节点、API 和 CLI](../run-node/README.md) 指南。

## 下一个 {hide}

了解 [store](./store.md) {hide} 