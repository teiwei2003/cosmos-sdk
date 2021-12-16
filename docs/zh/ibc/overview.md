# IBC 概览

了解 IBC 是什么、它的组件和用例。 {概要}

## 什么是区块链间通信协议(IBC)？

本文档是为想要为自定义用例编写自己的 IBC 应用程序的开发人员提供的指南。

IBC 协议的模块化设计意味着 IBC 应用程序开发人员不需要深入了解客户端、连接和证明验证的低级细节。提供了对堆栈较低级别的简要说明，以便应用程序开发人员可以获得对 IBC 协议的高级理解。

通道和端口的抽象层详细信息与应用程序开发人员相关。您可以定义自己的自定义数据包和 IBCModule 回调。

模块必须满足以下要求才能通过 IBC 进行交互:

- 绑定到一个或多个端口

- 定义数据包数据

- 定义可选的确认结构和方法来编码和解码它们

- 实现 IBCModule 接口

## 组件概述

本节介绍 IBC 组件和存储库链接。

### [客户端](https://github.com/cosmos/ibc-go/blob/main/modules/core/02-client)

IBC 客户端是由唯一客户端 ID 标识的轻客户端。 IBC 客户跟踪其他区块链的共识状态，以及根据客户的共识状态正确验证证明所需的那些区块链的证明规范。一个客户端可以与多个链的任意数量的连接相关联。支持的 IBC 客户端是:

- [Solo Machine 轻客户端](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/06-solomachine):手机、浏览器或笔记本电脑等设备。
- [Tendermint 轻客户端](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/07-tendermint):基于 Cosmos SDK 的链的默认值。
- [Localhost (loopback) client](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/09-localhost):用于测试、模拟和将数据包中继到模块相同的应用程序。

### [连接](https://github.com/cosmos/ibc-go/blob/main/modules/core/03-connection)

Connections 将两个“ConnectionEnd”对象封装在两个独立的区块链上。每个“ConnectionEnd”都与另一个区块链(交易对手区块链)的客户端相关联。连接握手负责验证每个链上的轻客户端对于其各自的交易对手是否正确。连接一旦建立，负责促进 IBC 状态的所有跨链验证。一个连接可以与任意数量的通道相关联。

### [证明](https://github.com/cosmos/ibc-go/blob/main/modules/core/23-commitment) 和 [路径](https://github.com/cosmos/ibc- go/blob/main/modules/core/24-host)

在 IBC 中，区块链不会直接通过网络相互传递消息。

- 为了进行通信，区块链将一些状态提交到为特定消息类型和特定交易对手保留的精确定义的路径。例如，将特定 connectionEnd 作为握手或旨在中继到交易对手链上的模块的数据包的一部分存储的区块链。

- 中继器进程监视这些路径的更新并通过将存储在路径下的数据连同该数据的证明提交给交易对手链来中继消息。

- 所有 IBC 实现必须支持提交 IBC 消息的路径在 [ICS-24 主机要求](https://github.com/cosmos/ics/tree/master/spec/core/ics-024-host-要求)。

- 所有实现必须生成和验证的证明格式在 [ICS-23 实现](https://github.com/confio/ics23) 中定义。 

### [功能](./ocap.md)

IBC 旨在在模块不一定相互信任的执行环境中工作。 IBC 必须验证端口和通道上的模块操作，以便只有具有适当权限的模块才能使用通道。这种安全性是使用 [动态功能](../architecture/adr-003-dynamic-capability-store.md) 实现的。在绑定到端口或为模块创建通道时，IBC 返回一个动态功能，模块必须声明使用该端口或通道。此绑定策略可防止其他模块使用该端口或通道，因为这些模块不拥有相应的功能。

虽然此解释是有用的背景信息，但 IBC 模块根本不需要与这些较低级别的抽象进行交互。 IBC 应用程序开发人员的相关抽象层是通道和端口。

将您的 IBC 应用程序编写为独立的**模块**。一个区块链上的模块可以通过发送、接收和确认数据包来与其他区块链上的其他模块通信，这些数据包通过由 `(channelID, portID)` 元组唯一标识的通道。

一个有用的类比是将 IBC 模块视为计算机上的 Internet 应用程序。然后可以将通道概念化为 IP 连接，IBC portID 类似于 IP 端口，IBC channelID 类似于 IP 地址。 IBC 模块的单个实例可以在同一端口上与任意数量的其他模块通信，并且 IBC 使用“(channelID, portID)”元组将所有数据包正确路由到相关模块。通过在不同的唯一通道上发送每个`(portID<->portID)` 数据包流，IBC 模块还可以通过多个端口与另一个 IBC 模块通信。

### [端口](https://github.com/cosmos/ibc-go/blob/main/modules/core/05-port)

IBC 模块可以绑定到任意数量的端口。每个端口必须由唯一的“portID”标识。由于 IBC 旨在通过在同一账本上运行的互不信任的模块来确保安全，因此绑定端口会返回动态对象功能。要对特定端口执行操作，例如，使用其端口 ID 打开通道，模块必须向 IBC 处理程序提供动态对象功能。此要求可防止恶意模块打开带有它不拥有的端口的通道。

IBC 模块负责声明在`BindPort` 上返回的能力。

### [频道](https://github.com/cosmos/ibc-go/blob/main/modules/core/04-channel)

可以在两个 IBC 端口之间建立 IBC 通道。一个端口由一个模块独占所有。 IBC 数据包通过通道发送。正如 IP 数据包包含目标 IP 地址、IP 端口、源 IP 地址和源 IP 端口一样，IBC 数据包包含目标端口 ID、通道 ID、源端口 ID 和通道 ID。 IBC 数据包使 IBC 能够正确地将数据包路由到目标模块，同时还允许接收数据包的模块知道发送方模块。

- 通道可以是“ORDERED”的，因此来自发送模块的数据包必须由接收模块按照它们发送的顺序进行处理。

- 推荐，通道可以是“无序”，以便来自发送模块的数据包按照它们到达的顺序进行处理，这可能不是发送数据包的顺序。

模块可以选择他们希望通过哪些通道进行通信。 IBC 期望模块实现在通道握手期间调用的回调。这些回调可以执行自定义通道初始化逻辑。如果返回错误，则通道握手失败。通过在回调中返回错误，模块可以以编程方式拒绝和接受通道。

通道握手是一个 4 步握手。简而言之，如果给定的链 A 想要使用已经建立的连接打开与链 B 的通道:

1. 链 A 发送一个 `ChanOpenInit` 消息来通知链 B 的通道初始化尝试。
2. 链 B 发送 `ChanOpenTry` 消息尝试打开链 A 上的通道。
3. 链 A 发送 `ChanOpenAck` 消息以标记其通道结束状态为打开。
4. 链B发送`ChanOpenConfirm`消息将其通道结束状态标记为开放。

如果所有这些操作都成功发生，则通道在双方都是开放的。在握手的每一步，与“ChannelEnd”关联的模块都会为握手的那一步执行回调。所以在`ChanOpenInit`上，链A上的模块执行了它的回调`OnChanOpenInit`。

正如端口具有动态功能一样，通道初始化返回模块**必须**声明的动态功能，以便它们可以传递功能来验证通道操作，例如发送数据包。通道功能在握手的第一部分传递到回调中:初始化链上的“OnChanOpenInit”或另一条链上的“OnChanOpenTry”。 

### [数据包](https://github.com/cosmos/ibc-go/blob/main/modules/core/04-channel)

模块通过在 IBC 通道上发送数据包来相互通信。所有 IBC 数据包都包含:

- 目的地`portID`

- 目的地`channelID`

- 源`portID`

- 来源`channelID`

  这些端口和通道允许模块知道给定数据包的发送模块。

- 可选择强制排序的序列

- `TimeoutTimestamp` 和 `TimeoutHeight`

  当非零时，这些超时值确定接收模块必须在此之前处理数据包的最后期限。

  如果超时过去而没有成功接收到数据包，则发送模块可以使数据包超时并采取适当的措施。

模块在 IBC 数据包的“Data []byte”字段内相互发送自定义应用程序数据。数据包数据对 IBC 处理程序是完全不透明的。发送器模块必须将其特定于应用程序的数据包信息编码到数据包的“数据”字段中。接收器模块必须将该“数据”解码回原始应用程序数据。

### [收据和超时](https://github.com/cosmos/ibc-go/blob/main/modules/core/04-channel)

由于 IBC 在分布式网络上工作，并依赖潜在故障的中继器在分类账之间中继消息，因此 IBC 必须处理数据包没有及时或根本没有发送到目的地的情况。数据包必须指定超时高度或超时时间戳，在此之后，目标链上将无法再成功接收数据包。

如果达到超时，则可以向原始链提交数据包证明超时，然后原始链可以执行特定于应用程序的逻辑来使数据包超时，可能通过回滚数据包发送更改(退还发件人任何锁定的资金，以及很快)。

在 ORDERED 通道中，通道中单个数据包的超时将关闭通道。如果数据包序列‘n’超时，那么在不违反按发送顺序处理数据包的有序通道合同的情况下，无法成功接收序列‘k > n’的数据包。由于 ORDERED 通道强制执行此不变量，因此数据包 n 的指定超时未在目标链上接收到序列 n 的证据足以使数据包 n 超时并关闭通道。

在 UNORDERED 情况下，可以按任何顺序接收数据包。 IBC 为它在 UNORDERED 通道中收到的每个序列写入一个数据包接收。此接收不包含任何信息，只是一个标记，旨在表示 UNORDERED 通道已按指定顺序接收到数据包。要使 UNORDERED 通道上的数据包超时，需要在指定的超时时间内为数据包的序列证明不存在数据包接收。当然，在 UNORDERED 通道上超时数据包会触发该数据包的应用程序特定超时逻辑，并且不会关闭通道。

因此，建议使用大多数使用 UNORDERED 频道的模块，因为它们需要较少的活跃度保证才能为该频道的用户有效运行。

### [致谢](https://github.com/cosmos/ibc-go/blob/main/modules/core/04-channel)

处理数据包时，模块还会编写特定于应用程序的确认。可以进行确认:

- 如果模块在从 IBC 模块接收到数据包后立即处理数据包，则在 `OnRecvPacket` 上同步。

- 如果模块在接收数据包后的某个时间点处理数据包，则异步。

这个确认数据对 IBC 来说是不透明的，就像数据包“Data”一样，被 IBC 视为一个简单的字节串“[]byte”。接收器模块必须对其确认进行编码，以便发送器模块可以正确解码。如何编码确认应通过通道握手期间的版本协商来决定。

确认可以编码数据包处理是成功还是失败，以及允许发送方模块采取适当行动的附加信息。

在接收链写入确认后，中继器将确认中继回原始发送器模块，然后使用确认的内容执行特定于应用程序的确认逻辑。此确认可能涉及在确认失败(退款发送方)的情况下回滚数据包发送更改。

在原始发送者链上成功收到确认后，IBC 模块删除相应的数据包承诺，因为它不再需要。

## 进一步阅读和规格

要了解有关 IBC 的更多信息，请查看以下规格:

- [IBC 规范](https://github.com/cosmos/ibc/tree/master/spec)
- [Cosmos SDK 上的IBC 协议](https://github.com/cosmos/ibc-go/tree/main/docs)

## 下一个 {hide}

了解如何 [integrate](./integration.md) IBC 到您的应用程序 {hide} 
