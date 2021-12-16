# 运行测试网

`simd testnet` 子命令可以轻松初始化和启动模拟测试网络以进行测试。 {概要}

除了用于 [运行节点](./run-node.html) 的命令之外，`simd` 二进制文件还包括一个 `testnet` 命令，允许您在进程中启动模拟测试网络或初始化文件在单独的进程中运行的模拟测试网络。

## 初始化文件

首先，让我们看一下 `init-files` 子命令。

这类似于初始化单个节点时的 `init` 命令，但在这种情况下，我们正在初始化多个节点，为每个节点生成创世交易，然后收集这些交易。

`init-files` 子命令初始化必要的文件以在单独的进程中运行测试网络(即使用 Docker 容器)。运行此命令不是`start` 子命令([见下文](#start-testnet))的先决条件。

为了初始化测试网络的文件，请运行以下命令:

```bash
simd testnet init-files
```

您应该在终端中看到以下输出: 

```bash
Successfully initialized 4 node directories
```

默认输出目录是一个相对的 `.testnets` 目录。让我们看看在`.testnets` 目录中创建的文件。

### 绅士

`gentxs` 目录包含每个验证器节点的创世交易。每个文件都包含一个 JSON 编码的创世交易，用于在创世时注册验证器节点。在初始化过程中，创世交易被添加到每个节点目录中的“genesis.json”文件中。

###节点

为每个验证器节点创建一个节点目录。每个节点目录中都有一个“simd”目录。 `simd` 目录是每个节点的主目录，其中包括该节点的配置和数据文件(即运行单个节点时，默认的 `~/.simapp` 目录中包含的相同文件)。

## 启动测试网

现在，让我们看一下 `start` 子命令。

`start` 子命令初始化并启动一个进程内测试网络。这是为了测试目的而启动本地测试网络的最快方法。

您可以通过运行以下命令来启动本地测试网络: 

```bash
simd testnet start
```

You should see something similar to the following:

```bash
acquiring test network lock
preparing test network with chain-id "chain-mtoD9v"


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
++       THIS MNEMONIC IS FOR TESTING PURPOSES ONLY        ++
++                DO NOT USE IN PRODUCTION                 ++
++                                                         ++
++  sustain know debris minute gate hybrid stereo custom   ++
++  divorce cross spoon machine latin vibrant term oblige  ++
++   moment beauty laundry repeat grab game bronze truly   ++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


starting test network...
started test network
press the Enter Key to terminate
```

第一个验证器节点现在正在进程中运行，这意味着一旦您关闭终端窗口或按下 Enter 键，测试网络就会终止。 在输出中，提供了第一个验证器节点的助记词用于测试目的。 验证器节点使用与初始化和启动单个节点时相同的默认地址(无需提供 `--node` 标志)。 

Check the status of the first validator node:

```
simd status
```

Import the key from the provided mnemonic:

```
simd keys add test --recover --keyring-backend test
```

Check the balance of the account address:

```
simd q bank balances [address]
```

使用此测试帐户对测试网络进行手动测试。

## 测试网选项

您可以使用标志自定义测试网络的配置。 要查看所有标志选项，请将 `--help` 标志附加到每个命令。 
