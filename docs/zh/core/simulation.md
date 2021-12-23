# Cosmos 区块链模拟器

Cosmos SDK 提供了一个完整的模拟框架来模糊测试每个
由模块定义的消息。

在 Cosmos SDK 上，此功能由 [`SimApp`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/app.go) 提供，它是一个
用于运行 [`simulation`](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/simulation) 模块的 `Baseapp` 应用程序。
该模块定义了所有的仿真逻辑以及操作
随机参数，如账户、余额等。

## 目标

区块链模拟器测试区块链应用程序在以下情况下的行为方式
通过生成和发送随机消息来实现现实生活环境。
这样做的目的是检测和调试可能导致活动链停止的故障，
通过提供有关模拟器运行的操作的日志和统计信息作为
以及在发现故障时导出最新的应用程序状态。

它与集成测试的主要区别在于模拟器应用程序允许
您可以传递参数来自定义正在模拟的链。
这在尝试重现生成的错误时会派上用场
提供的操作(随机与否)。

## 模拟命令

模拟应用程序有不同的命令，每个命令测试不同的
故障类型:

- `AppImportExport`:模拟器导出初始应用程序状态，然后它
  使用导出的“genesis.json”作为输入创建一个新应用程序，检查
 存储之间的不一致。
- `AppSimulationAfterImport`:将两个模拟排在一起。第一个向第二个提供应用程序状态(_i.e_ genesis)。用于测试来自实时链的软件升级或硬分叉。
- `AppStateDeterminism`:检查所有节点是否以相同的顺序返回相同的值。
- `BenchmarkInvariants`:分析运行所有模块的不变量的性能(_i.e_ 顺序运行 [benchmark](https://golang.org/pkg/testing/#hdr-Benchmarks) 测试)。不变式检查
  存储和被动跟踪器上的值之间的差异。例如:账户持有的总代币与总供应跟踪器。
- `FullAppSimulation`:通用模拟模式。为给定数量的块运行链和指定的操作。测试模拟中没有“恐慌”。它还对每个“Period”运行不变检查，但它们没有进行基准测试。

每个模拟必须接收一组输入(_i.e_ 标志)，例如
运行模拟的块、种子、块大小等。
检查标志的完整列表 [此处](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/simapp/config.go#L32-L55)。

## 模拟器模式

除了各种输入和命令外，模拟器还以三种模式运行:

1. 初始状态、模块参数和仿真完全随机
   参数是**伪随机生成的**。
2. 来自定义初始状态和模块参数的 `genesis.json` 文件。
   此模式有助于在已知状态下运行模拟，例如需要测试应用程序的新(很可能是破坏性的)版本的实时网络导出。
3. 来自`params.json` 文件，其中初始状态是伪随机生成的，但可以手动提供模块和模拟参数。
   这允许更可控和确定性的模拟设置，同时允许状态空间仍然是伪随机模拟的。
   可用参数列表在 [here](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/simulation/params.go#L44-L52) 中列出。

::: 小费
这些模式并不相互排斥。所以你可以例如随机运行
生成的创世状态(`1`)和手动生成的模拟参数(`3`)。
:::

## 用法

这是模拟运行方式的一般示例。更具体的例子
检查 Cosmos SDK [Makefile](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/Makefile#L251-L287)。 

```bash
 $ go test -mod=readonly github.com/cosmos/cosmos-sdk/simapp \
  -run=TestApp<simulation_command> \
  ...<flags>
  -v -timeout 24h
```

## 调试技巧

以下是遇到模拟失败时的一些建议:

- 在发现失败的高度导出应用程序状态。你可以这样做
  通过将 `-ExportStatePath` 标志传递给模拟器。
- 使用`-Verbose` 日志。他们可以为您提供有关所有操作的更好提示
  涉及。
- 减少模拟`-Period`。这将运行不变量检查更多
  频繁地。
- 使用 `-PrintAllInvariants` 一次性打印所有失败的不变量。
- 尝试使用另一个 `-Seed`。如果它可以重现相同的错误并且失败
  您将花更少的时间运行模拟。
- 减少`-NumBlocks`。应用程序在之前高度的状态如何
  失败？
- 使用 `-SimulateEveryOperation` 在每个操作上运行不变量。 _注意_:这个
  会减慢你的模拟**很多**。
- 尝试将日志添加到未记录的操作中。你必须定义一个
  [记录器](https://github.com/cosmos/cosmos-sdk/blob/v0.40.0/x/staking/keeper/keeper.go#L66-L69) 在你的“Keeper”上。

## 在基于 Cosmos SDK 的应用程序中使用模拟

了解如何将模拟集成到基于 Cosmos SDK 的应用程序中:

- 应用仿真经理
- [建筑模块:模拟器](../building-modules/simulator.md)
- 模拟器测试 