# 概念

## 计划

`x/upgrade` 模块定义了一个 `Plan` 类型，其中安排了实时升级
发生。可以在特定的块高度安排“计划”。
一旦(冻结的)候选发布以及适当的升级，就会创建一个“计划”
`Handler`(见下文)已达成一致，其中 `Plan` 的 `Name` 对应于
特定的`处理程序`。通常，“计划”是通过治理提案创建的
流程，如果投票通过，将被安排。 “计划”的“信息”
可能包含有关升级的各种元数据，通常是特定于应用程序的
升级要包含在链上的信息，例如验证者可以进行的 git commit
自动升级到。

### Sidecar 流程

如果运行应用程序二进制文件的操作员也运行一个 sidecar 进程来协助
在二进制文件的自动下载和升级中，`Info` 允许这个过程
无缝衔接。即，`x/upgrade` 模块满足
[cosmosd 可升级二进制规范](https://github.com/regen-network/cosmosd#upgradeable-binary-specification)
可以选择使用规范和 `cosmosd` 来完全自动化升级
节点运营商的流程。通过用必要的信息填充“信息”字段，
二进制文件可以自动下载。见[这里](https://github.com/regen-network/cosmosd#auto-download) 

```go
type Plan struct {
  Name   string
  Height int64
  Info   string
}
```

## Handler

`x/upgrade` 模块有助于从主要版本 X 升级到主要版本 Y。
要做到这一点，节点运营商必须首先将他们当前的二进制文件升级到一个新的
具有新版本 Y 对应的 `Handler` 的二进制文件。假设
此版本已经过社区的全面测试和批准。 这
`Handler` 定义了在新的二进制 Y 之前需要发生什么状态迁移
可以成功运行链条。 当然，这个 `Handler` 是特定于应用程序的
并且不是在每个模块的基础上定义的。 注册`Handler`是通过
应用程序中的`Keeper#SetUpgradeHandler`。 

```go
type UpgradeHandler func(Context, Plan, VersionMap) (VersionMap, error)
```

在每次 `EndBlock` 执行期间，`x/upgrade` 模块会检查是否存在
应该执行的“计划”(在那个高度安排)。 如果是，对应的
`Handler` 被执行。 如果“计划”预期执行但未注册“处理程序”
或者如果二进制文件升级过早，节点将优雅地恐慌并退出。 

## StoreLoader

`x/upgrade` 模块还有助于作为升级一部分的存储迁移。 这
`StoreLoader` 设置在新二进制文件可以之前需要发生的迁移
成功运行链。 这个`StoreLoader`也是特定于应用程序的
不是在每个模块的基础上定义的。 注册这个`StoreLoader`是通过
应用程序中的`app#SetStoreLoader`。 

```go
func UpgradeStoreLoader (upgradeHeight int64, storeUpgrades *store.StoreUpgrades) baseapp.StoreLoader
```

如果有计划的升级并且达到升级高度，旧的二进制文件会在恐慌之前将“计划”写入磁盘。

此信息对于确保“StoreUpgrades”在正确的高度和
预期升级。 它消除了新二进制文件多次执行“StoreUpgrades”的机会
每次重启时的次数。 此外，如果计划在同一高度进行多次升级，“名称”
将确保这些 `StoreUpgrades` 仅在计划的升级处理程序中发生。 

## Proposal

通常，“计划”是通过“SoftwareUpgradeProposal”通过治理提出和提交的。
该提案规定了标准的治理流程。 如果提案通过，
以特定“处理程序”为目标的“计划”被持久化和调度。 这
可以通过更新新提案中的“Plan.Height”来延迟或加快升级。

```go
type SoftwareUpgradeProposal struct {
  Title       string
  Description string
  Plan        Plan
}
```

### Cancelling Upgrade Proposals

可以取消升级建议。 存在一个 `CancelSoftwareUpgrade` 提议
类型，可以投票通过，并将删除计划的升级“计划”。
当然，这需要在升级之前就知道升级是一个坏主意。
升级自己，留出时间进行投票。

如果需要这种可能性，升级高度应为
`2 * (VotingPeriod + DepositPeriod) + (SafetyDelta)` 从开始
升级建议。 “SafetyDelta”是一个成功的可用时间
升级提案并意识到这是一个坏主意(由于外部社会共识)。

一个`CancelSoftwareUpgrade` 提议也可以在原来的同时提出
`SoftwareUpgradeProposal` 仍在投票中，只要 `VotingPeriod`
在“SoftwareUpgradeProposal”之后结束。 