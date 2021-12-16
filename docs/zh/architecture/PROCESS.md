# ADR 创建过程

1.复制`adr-template.md`文件。使用以下文件名模式:`adr-next_number-title.md`
2. 如果您想获得早期反馈，请创建一个拉取请求草案。
3. 确保上下文和解决方案清晰且有据可查。
4. 在 [README](./README.md) 文件的列表中添加一个条目。
5. 创建一个 Pull Request 来提出一个新的 ADR。

## ADR 生命周期

ADR 创建是一个**迭代**过程。与其试图在单个 ADR 拉取请求中解决所有决策，我们必须首先了解问题并通过 GitHub 问题收集反馈。

1. 每个提案都应该以一个新的 GitHub 问题开始或者是现有问题的结果。该问题应仅包含一个简短的提案摘要。

2. 一旦动机得到验证，GitHub 拉取请求 (PR) 将创建一个基于 `adr-template.md` 的新文档。

3. ADR 不必在单个 PR 中以 _accepted_ 状态到达`master`。如果动机明确且解决方案合理，我们应该能够合并它并保持 _proposed_ 状态。最好采用迭代方法，而不是长时间的、未合并的拉取请求。

4. 如果 _proposed_ ADR 被合并，那么它应该在 ADR 文档说明或 GitHub 问题中清楚地记录未决问题。

5. PR 应该总是被合并。在 ADR 有问题的情况下，我们仍然倾向于将其与 _rejected_ 状态合并。 ADR 不应该合并的唯一时间是作者放弃它。

6. 不应修剪合并的 ADR。

### ADR 状态

状态有两个组成部分: 

```
{CONSENSUS STATUS} {IMPLEMENTATION STATUS}
```

实施状态是“已实施”或“未实施”。 

#### Consensus Status

```
DRAFT -> PROPOSED -> LAST CALL yyyy-mm-dd -> ACCEPTED | REJECTED -> SUPERSEDED by ADR-xxx
                  \        |
                   \       |
                    v      v
                     ABANDONED
```

+ `草案`:[可选] 正在进行中的 ADR，尚未准备好进行一般审查。这是为了在草案拉取请求表单中展示早期工作并获得早期反馈。
+ `PROPOSED`:涵盖完整解决方案架构的 ADR，仍在审查中 - 项目利益相关者尚未达成一致。
+ `LAST CALL <上次通话的日期>`:[可选] 明确通知我们即将接受更新。将状态更改为“LAST CALL”意味着(Cosmos SDK 维护者的)社会共识已经达成，我们仍然希望给它时间让社区做出反应或分析。
+ `ACCEPTED`:ADR 将代表当前已实施或将要实施的架构设计。
+ `REJECTED`:如果项目利益相关者之间达成共识，ADR 可以从提议或接受变为拒绝。
+ `SUPERSEEDED by ADR-xxx`:已被新 ADR 取代的 ADR。
+ `ABANDONED`:原作者不再追求 ADR。

## ADR 中使用的语言

+ 上下文/背景应该用现在时写。
+ 避免使用第一种个人形式。 