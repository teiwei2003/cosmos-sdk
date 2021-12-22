# 设置钥匙圈

本文档描述了如何为 [**application**](../basics/app-anatomy.md) 配置和使用密钥环及其各种后端。 {概要}

密钥环保存用于与节点交互的私有/公共密钥对。例如，在运行区块链节点之前需要设置验证器密钥，以便正确签署区块。私钥可以存储在不同的位置，称为“后端”，例如文件或操作系统自己的密钥存储。

## 密钥环的可用后端

从 v0.38.0 版本开始，Cosmos SDK 附带了一个新的密钥环实现
它提供了一组以安全方式管理加密密钥的命令。这
新的密钥环支持多个存储后端，其中一些可能无法在
所有操作系统。

### `os` 后端

`os` 后端依赖于操作系统特定的默认值来处理密钥存储
安全地。通常，操作系统的凭证子系统处理密码提示，
私钥存储，以及根据用户密码策略的用户会话。这里
是最流行的操作系统及其各自的密码管理器的列表:

- macOS(自 Mac OS 8.6 起):[钥匙串](https://support.apple.com/en-gb/guide/keychain-access/welcome/mac)
- Windows:[凭据管理 API](https://docs.microsoft.com/en-us/windows/win32/secauthn/credentials-management)
- GNU/Linux:
    - [libsecret](https://gitlab.gnome.org/GNOME/libsecret)
    - [kwallet](https://api.kde.org/frameworks/kwallet/html/index.html)

使用 GNOME 作为默认桌面环境的 GNU/Linux 发行版通常带有
[海马](https://wiki.gnome.org/Apps/Seahorse)。基于 KDE 的发行版的用户是
通常随 [KDE 钱包管理器](https://userbase.kde.org/KDE_Wallet_Manager) 提供。
虽然前者实际上是一个 `libsecret` 方便的前端，后者是一个 `kwallet`
客户。

`os` 是默认选项，因为操作系统的默认凭据管理器是
旨在满足用户最常见的需求，并为他们提供舒适的
在不影响安全性的情况下体验。

推荐用于无头环境的后端是 `file` 和 `pass`。

### `file` 后端

`file` 后端更类似于之前使用的 keybase 实现
v0.38.1。它将加密的密钥环存储在应用程序的配置目录中。这
keyring 每次访问都会请求密码，可能会出现多次
次在单个命令中导致重复的密码提示。如果使用 bash 脚本
要使用 `file` 选项执行命令，您可能需要使用以下格式
对于多个提示: 

```sh
# assuming that KEYPASSWD is set in the environment
$ gaiacli config keyring-backend file                             # use file backend
$ (echo $KEYPASSWD; echo $KEYPASSWD) | gaiacli keys add me        # multiple prompts
$ echo $KEYPASSWD | gaiacli keys show me                          # single prompt
```

::: tip
The first time you add a key to an empty keyring, you will be prompted to type the password twice.
:::

### `pass` 后端

`pass` 后端使用 [pass](https://www.passwordstore.org/) 实用程序来管理磁盘
密钥的敏感数据和元数据的加密。 密钥存储在“gpg”加密文件中
在特定于应用程序的目录中。 `pass` 可用于最流行的 UNIX
操作系统以及 GNU/Linux 发行版。 请参阅其手册页了解
有关如何下载和安装它的信息。

::: 小费
**pass** 使用 [GnuPG](https://gnupg.org/) 进行加密。 `gpg` 自动调用 `gpg-agent`
守护进程在执行时处理 GnuPG 凭据的缓存。 请参考`gpg-agent`
有关如何配置缓存参数(例如凭据 TTL 和
密码过期。
:::

密码存储必须在第一次使用之前设置: 

```sh
pass init <GPG_KEY_ID>
```

将 `<GPG_KEY_ID>` 替换为您的 GPG 密钥 ID。您可以使用您的个人 GPG 密钥或替代密钥
您可能希望专门用于加密密码存储。

### `kwallet` 后端

`kwallet` 后端使用 `KDE Wallet Manager`，它默认安装在
将 KDE 作为默认桌面环境提供的 GNU/Linux 发行版。请参阅
[KWallet 手册](https://docs.kde.org/stable5/en/kdeutils/kwallet5/index.html) 了解更多
信息。

### `test` 后端

`test` 后端是 `file` 后端的无密码变体。存储密钥
磁盘上未加密。

**仅用于测试目的。不建议在生产环境中使用 `test` 后端**。

### `memory` 后端

`memory` 后端将密钥存储在内存中。程序退出后，这些键会立即被删除。

**仅用于测试目的。不建议在生产环境中使用 `memory` 后端**。

## 将密钥添加到密钥环

::: 警告
确保您可以构建自己的二进制文件，并在代码段中将“simd”替换为您的二进制文件的名称。
:::

使用 Cosmos SDK 开发的应用程序带有 `keys` 子命令。出于本教程的目的，我们将运行 `simd` CLI，这是一个使用 Cosmos SDK 构建的应用程序，用于测试和教育目的。更多信息请参见[`simapp`](https://github.com/cosmos/cosmos-sdk/tree/v0.40.0-rc3/simapp)。

您可以使用 `simd keys` 获取有关 keys 命令的帮助，使用 `simd keys [command] --help` 获取有关特定子命令的更多信息。

::: 小费
您还可以使用 `simd completion` 命令启用自动完成功能。例如，在 bash 会话开始时，运行 `. <(simd completion)`，并且所有 `simd` 子命令都将自动完成。
:::

要在密钥环中创建新密钥，请运行带有 `<key_name>` 参数的 `add` 子命令。出于本教程的目的，我们将仅使用 `test` 后端，并调用我们的新密钥 `my_validator`。此键将在下一节中使用。 

```bash
$ simd keys add my_validator --keyring-backend test

# Put the generated address in a variable for later use.
MY_VALIDATOR_ADDRESS=$(simd keys show my_validator -a --keyring-backend test)
```

此命令生成一个新的 24 字助记词，将其持久化到相关后端，并输出有关密钥对的信息。 如果此密钥对将用于保存有价值的令牌，请务必在安全的地方写下助记词！

默认情况下，密钥环会生成一个“secp256k1”密钥对。 密钥环还支持“ed25519”密钥，可以通过传递“--algo ed25519”标志来创建。 一个密钥环当然可以同时保存两种类型的密钥，Cosmos SDK 的 `x/auth` 模块(特别是它的 [AnteHandlers](../core/baseapp.md#antehandler))本身就支持这两种公钥算法。

## 下一个 {hide}

阅读有关 [运行节点](./run-node.md) {hide} 