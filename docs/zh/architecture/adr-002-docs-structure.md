# ADR 002:SDK 文档结构

## 语境

需要 Cosmos SDK 文档的可扩展结构。 当前文档包含大量与 Cosmos SDK 无关的材料，难以维护且作为用户难以遵循。

理想情况下，我们应该:

- 所有与开发框架或工具相关的文档都位于各自的 github 存储库中(sdk 存储库将包含 sdk 文档，集线器存储库将包含集线器文档，乳液存储库将包含乳液文档等)
- 所有其他文档(常见问题解答、白皮书、关于 Cosmos 的高级材料)都将保存在网站上。

## 决定

重新构建Cosmos SDK github repo的`/docs`文件夹如下:

```
docs/
├── README
├── intro/
├── concepts/
│   ├── baseapp
│   ├── types
│   ├── store
│   ├── server
│   ├── modules/
│   │   ├── keeper
│   │   ├── handler
│   │   ├── cli
│   ├── gas
│   └── commands
├── clients/
│   ├── lite/
│   ├── service-providers
├── modules/
├── spec/
├── translations/
└── architecture/
```

每个子文件夹中的文件无关紧要，可能会更改。重要的是切片:

- `README`:文档的登陆页面。
- `intro`:介绍材料。目标是对 Cosmos SDK 有一个简短的解释，然后引导人们找到他们需要的资源。 [Cosmos SDK 教程](https://github.com/cosmos/sdk-application-tutorial/) 以及 `godocs` 将被突出显示。
- `concepts`:包含 Cosmos SDK 抽象的高级解释。它不包含特定的代码实现，不需要经常更新。 **它不是接口的 API 规范**。 API 规范是 `godoc`。
- `clients`:包含有关各种 Cosmos SDK 客户端的规范和信息。
- `spec`:包含模块的规范等。
- `modules`:包含指向 `godocs` 和模块规范的链接。
- `architecture`:包含与当前的架构相关的文档。
- `translations`:包含文档的不同翻译。

网站文档侧边栏将仅包含以下部分:

-`自述文件`
- `介绍`
-`概念`
-`客户`

`architecture` 不需要显示在网站上。

## 状态

公认

## 结果

### 目的

- Cosmos SDK 文档的组织更加清晰。
- `/docs` 文件夹现在只包含 Cosmos SDK 和 gaia 相关材料。稍后，它将只包含 Cosmos SDK 相关资料。
- 开发人员只需在打开 PR 时更新 `/docs` 文件夹(而不是 `/examples` 例如)。
- 由于重新设计的架构，开发人员更容易在文档中找到他们需要更新的内容。
- 网站文档的更清洁的 vuepress 构建。
- 将有助于构建可执行文档(参见 https://github.com/cosmos/cosmos-sdk/issues/2611)

### 中性的

- 我们需要将一堆不推荐使用的东西移到 `/_attic` 文件夹。
- 我们需要在`concepts`中集成`docs/sdk/docs/core`中的内容。
- 我们需要将当前存在于 `docs` 中并且不适合新结构(如 `lotion`、介绍材料、白皮书)的所有内容移动到网站存储库。
- 更新`DOCS_README.md`

## 参考

- https://github.com/cosmos/cosmos-sdk/issues/1460
- https://github.com/cosmos/cosmos-sdk/pull/2695
- https://github.com/cosmos/cosmos-sdk/issues/2611
