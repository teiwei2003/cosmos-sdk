# 运行一个节点

现在应用程序已准备就绪并且密钥环已填充，是时候看看如何运行区块链节点了。 在本节中，我们正在运行的应用程序称为 [`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/simapp)，以及其对应的 CLI 二进制文件 `simd` . {概要}

## 先决条件阅读

- [Cosmos SDK 应用剖析](../basics/app-anatomy.md) {prereq}
- [设置密钥环](./keyring.md) {prereq}

## 初始化链

::: 警告
确保您可以构建自己的二进制文件，并在代码段中将“simd”替换为您的二进制文件的名称。
:::

在实际运行节点之前，我们需要初始化链，最重要的是它的创世文件。 这是通过 `init` 子命令完成的: 

```bash
# The argument <moniker> is the custom username of your node, it should be human-readable.
simd init <moniker> --chain-id my-test-chain
```

上面的命令创建了节点运行所需的所有配置文件，以及一个默认的 genesis 文件，它定义了网络的初始状态。 默认情况下，所有这些配置文件都在“~/.simapp”中，但您可以通过传递“--home”标志来覆盖此文件夹的位置。

`~/.simapp` 文件夹具有以下结构: 

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

## 更新一些默认设置

如果您想更改配置文件中的任何字段值(例如:genesis.json)，您可以使用 `jq` ([installation](https://stedolan.github.io/jq/download/) & [docs]( https://stedolan.github.io/jq/manual/#Assignment)) 和 `sed` 命令来做到这一点。 这里列出了几个例子。

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

## 添加创世账户

在启动链之前，您需要使用至少一个帐户填充状态。 为此，首先[在密钥环中创建一个新帐户](./keyring.md#adding-keys-to-the-keyring) 在`test` 密钥环后端下命名为`my_validator`(可以随意选择另一个名称和 另一个后端)。

现在你已经创建了一个本地账户，继续在你的链的创世文件中授予它一些“stake”令牌。 这样做还可以确保您的链知道此帐户的存在: 

```bash
simd add-genesis-account $MY_VALIDATOR_ADDRESS 100000000000stake
```

回想一下，`$MY_VALIDATOR_ADDRESS` 是一个变量，它保存了 [keyring](./keyring.md#adding-keys-to-the-keyring) 中 `my_validator` 密钥的地址。另请注意，Cosmos SDK 中的代币具有 `{amount}{denom}` 格式:`amount` 是一个 18 位精度的十进制数，而 `denom` 是唯一的代币标识符及其面额密钥(例如`atom` 或 `uatom`)。在这里，我们授予 `stake` 代币，因为 `stake` 是用于在 [`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/ simapp)。对于您自己的拥有自己权益的链，应该使用该代币标识符。

现在您的帐户有一些代币，您需要向链中添加一个验证器。验证者是参与共识过程的特殊全节点(在 [底层共识引擎](../intro/sdk-app-architecture.md#tendermint) 中实现)，以便向链中添加新块。任何账户都可以声明其成为验证者运营商的意图，但只有那些拥有足够委托的人才能进入活跃集(例如，只有拥有最多委托的前 125 名验证者候选人才能成为 Cosmos Hub 中的验证者)。对于本指南，您将添加本地节点(通过上面的 `init` 命令创建)作为链的验证器。验证器可以在链首次启动之前通过创世文件中包含的特殊交易声明，称为“gentx”: 

```bash
# Create a gentx.
simd gentx my_validator 100000000stake --chain-id my-test-chain --keyring-backend test

# Add the gentx to the genesis file.
simd collect-gentxs
```

一个 `gentx` 做了三件事:

1. 将您创建的 `validator` 帐户注册为验证器操作员帐户(即控制验证器的帐户)。
2. 自行委托所提供的质押代币的“数量”。
3. 将操作员帐户与将用于签署区块的 Tendermint 节点公钥相关联。 如果没有提供 `--pubkey` 标志，则默认为通过上面的 `simd init` 命令创建的本地节点公钥。

有关 `gentx` 的更多信息，请使用以下命令: 

```bash
simd gentx --help
```

## 使用 `app.toml` 和 `config.toml` 配置节点

Cosmos SDK 在`~/.simapp/config` 中自动生成两个配置文件:

- `config.toml`:用于配置 Tendermint，在 [Tendermint 的文档](https://docs.tendermint.com/master/nodes/configuration.html) 上了解更多信息，
- `app.toml`:由 Cosmos SDK 生成，用于配置您的应用程序，例如状态修剪策略、遥测、gRPC 和 REST 服务器配置、状态同步...

这两个文件都有大量注释，请直接参考它们以调整您的节点。

一个需要调整的示例配置是 `app.toml` 中的 `minimum-gas-prices` 字段，它定义了验证器节点在处理交易时愿意接受的最低 gas 价格。根据链的不同，它可能是空字符串，也可能不是。如果它为空，请确保使用某个值编辑该字段，例如“10token”，否则节点将在启动时停止。出于本教程的目的，让我们将最低汽油价格设置为 0: 

```toml
 # The minimum gas prices a validator is willing to accept for processing a
 # transaction. A transaction's fees must meet the minimum of any denomination
 # specified in this config (e.g. 0.25token1;0.0001token2).
 minimum-gas-prices = "0stake"
```

## Run a Localnet

Now that everything is set up, you can finally start your node:

```bash
simd start
```

你应该看到块进来了。

前面的命令允许您运行单个节点。 这对于下一节关于与此节点交互的内容已经足够了，但您可能希望同时运行多个节点，并查看它们之间如何达成共识。

天真的方法是在单独的终端窗口中再次运行相同的命令。 这是可能的，但是在 Cosmos SDK 中，我们利用 [Docker Compose](https://docs.docker.com/compose/) 的强大功能来运行本地网络。 如果您需要有关如何使用 Docker Compose 设置自己的本地网络的灵感，可以查看 Cosmos SDK 的 [`docker-compose.yml`](https://github.com/cosmos/cosmos-sdk/blob /v0.40.0-rc3/docker-compose.yml)。

## 下一个 {hide}

阅读 [与您的节点交互](./interact-node.md) {hide} 
