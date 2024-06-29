<!--yml

category: 未分类

日期：2024-05-27 13:01:10

-->

# ESLint v9.0.0 发布 - ESLint - 可插拔 JavaScript Linter

> 来源：[https://eslint.org/blog/2024/04/eslint-v9.0.0-released/](https://eslint.org/blog/2024/04/eslint-v9.0.0-released/)

## 亮点

这是升级从 ESLint v8.x 到 ESLint v9.0.0 时需要了解的重要变更（破坏性和非破坏性）的摘要。

### 安装

由于这是一个重大发布，您可能无法通过 npm 自动升级。要确保您正在使用此版本，请运行：

```
npm i eslint@9.0.0 --save-dev 
```

### 迁移指南

由于有许多更改，我们创建了一个[migration guide](/docs/latest/use/migrate-to-9.0.0)，详细描述了破坏性更改及解决方法。我们预计大多数用户在无需任何构建更改的情况下即可升级，但如果遇到问题，迁移指南应该是一个有用的资源。

### Node.js < v18.18.0，不再支持 v19

至此时，Node.js v20.x 是 LTS 版本，因此我们将[停止支持](https://github.com/eslint/eslint/issues/17595)所有早于 v18.18.0 和 v19.x 的 Node.js 版本。

### 平面配置现在是默认配置并有一些更改

平面配置现在是 ESLint 的默认配置格式，官方已废弃 eslintrc。要继续使用 eslintrc 配置文件，您需要将 `ESLINT_USE_FLAT_CONFIG` 环境变量设置为 `false`。

此更改影响用户、插件开发者和集成者，因为 ESLint 的许多方面都必须更改以实现这一点。请参阅我们之前的[博客文章](/blog/2023/10/flat-config-rollout-plans/)了解更多详情。

此版本还引入了[配置检查器](/blog/2024/04/eslint-config-inspector/)，可以通过命令行使用 `--inspect-config` 启动。

已移除以下格式化程序：

+   `checkstyle`

+   `compact`

+   `jslint-xml`

+   `junit`

+   `tap`

+   `unix`

+   `visualstudio`

如果您当前正在使用这些格式化程序，则需要安装[独立包](https://github.com/fregante/eslint-formatters)以供 ESLint v9.0.0 使用。

### 移除了 `valid-jsdoc` 和 `require-jsdoc` 规则

我们已移除了[`valid-jsdoc` 和 `require-jsdoc`](https://github.com/eslint/eslint/issues/15820)。我们建议使用 [`eslint-plugin-jsdoc`](https://github.com/gajus/eslint-plugin-jsdoc) 插件代替。

### 移除了 `context` 和 `SourceCode` 上的已废弃方法

正如我们在九月份[宣布的](https://eslint.org/blog/2023/09/preparing-custom-rules-eslint-v9/)，我们已从 `context` 中移除了大量已废弃的方法，并将其替换为 `SourceCode` 上的方法。

### 更新了 `eslint:recommended`

`eslint:recommended` 配置已更新，包含我们认为重要的新规则，并移除了已废弃和不那么重要的规则。

### 新规则：`no-useless-assignment`

ESLint v9.0.0 引入了一个新规则，[`no-useless-assignment`](https://eslint.org/docs/latest/rules/no-useless-assignment)，旨在捕获您将值赋给变量但从未使用该值的情况。例如：

```
let id = 1234; 
id = calculateId();
```

### 现有规则的更新：

+   [`complexity`](/docs/latest/rules/complexity) 规则现在还考虑可选链和解构模式及参数中的默认值。

+   [`no-fallthrough`](/docs/latest/rules/no-fallthrough) 规则有一个新选项 `reportUnusedFallthroughComment`。

+   [`no-inner-declarations`](/docs/latest/rules/no-inner-declarations) 规则有新的默认行为。在 v8.x 中，该规则会标记在块内定义的任何函数为错误，因为这种行为在早期版本的 JavaScript 中是未定义的。从 ES 2015 开始，块范围的函数声明是定义良好的，因此我们更改了默认行为，不再在块范围函数上发出警告。

+   [`no-misleading-character-class`](/docs/latest/rules/no-misleading-character-class) 规则现在突出显示正则表达式中有问题的字符，而不是整个正则表达式。

+   [`no-restricted-imports`](/docs/latest/rules/no-restricted-imports) 规则更改了 `paths` 的行为。在 v8.x 中，如果配置中 `paths` 数组中的多个条目具有相同的 `name` 属性，则仅应用最后一个。在 v9.0.0 中，所有条目都适用，允许为不同的导入名称指定不同的错误消息。

+   [`no-restricted-imports`](/docs/latest/rules/no-restricted-imports) 规则有新选项 `allowImportNames` 和 `allowImportNamePattern`。

+   [`no-unused-vars`](/docs/latest/rules/no-unused-vars) 规则的 `varsIgnorePattern` 选项不再适用于捕获的错误变量。

+   [`no-unused-vars`](/docs/latest/rules/no-unused-vars) 规则的 `caughtErrors` 选项有新的默认值（从 `"none"` 更改为 `"all"`）。

+   [`no-unused-vars`](/docs/latest/rules/no-unused-vars) 规则有新选项 `ignoreClassWithStaticInitBlock`。

+   [`no-unused-vars`](/docs/latest/rules/no-unused-vars) 规则有新选项 `reportUsedIgnorePattern`。

+   [`no-useless-computed-key`](/docs/latest/rules/no-useless-computed-key) 规则对 `enforceForClassMembers` 选项的默认值进行了更改（从 `false` 更改为 `true`）。这旨在帮助避免由重构引起的误导性注释。当此选项设置为 `true` 时，如果情况永远不会跌落，规则将禁止跌落注释。

### 新的 API `loadESLint()`

ESLint 现在从其主入口导出一个新函数 [`loadESLint()`](/docs/latest/integrate/nodejs-api#loadeslint)。集成可以使用此函数获取 `ESLint` 类（以前是 `FlatESLint` 类）或 `LegacyESLint` 类（以前是 `ESLint` 类），从而轻松在平面配置和 eslintrc API 之间进行切换。

### 对于编写规则方式的更改

为了帮助减少规则中的错误，我们已经进行了好几个变动：

1.  v9.0.0版本中的函数式规则将停止工作。函数式规则是指通过从文件中导出一个函数，而不是导出一个带有`create()`方法的对象而创建的规则。

1.  如果规则没有指定`meta.schema`，则将应用默认的 `[]`模式。这意味着没有模式的规则将被认为是没有选项的，从而在提供了选项时，验证就会失败。

### 更严格的`RuleTester`验证

此次发布在`RuleTester`中增加了更多的检查：

+   消息中不能包含未替换的占位符。

+   建议必须改变代码。

+   建议信息必须对同一种语法检查问题具有唯一性。

+   建议必须生成有效的语法。

+   测试用例的`output`必须与`code`不同。

+   测试错误对象必须明确指定`message`或`messageId`。

+   测试错误对象必须明确指定`suggestions`，如果实际错误提供了建议的话。

+   测试建议对象必须明确指定`desc`或`messageId`。

+   测试建议对象必须明确指定`输出`。

+   测试对象的`filename`和`only`属性必须是预期类型（`string`和`boolean`）。

+   定义重复的测试会导致错误。

### `--output-file`标志现在确保输出文件。

`--output-file`命令行标志旨在将ESLint运行的结果输出到指定文件。在此之前，如果代码检查无错误或警告，将不会输出任何文件。在v9.0.0版本中，代码检查无错误或警告时将输出一个空文件。

### 更好的范围分析

在ESLint v9.0.0中，我们通过修复了一些长期的bug更新了`eslint-scope`的行为：

1.  之前，ESLint会将`("use strict")`视为严格模式指令，即使它并非如此。 我们修正了行为的实现，只对有效的严格模式指令给予信任。

1.  类`extends`子句包含的包含范围错误地被设置为类的包含范围，而不是类本身的范围。这个问题已经得到了修正。

## 移除了`CodePath#currentSegments`

如我们在[之前的帖子](https://eslint.org/blog/2023/09/preparing-custom-rules-eslint-v9/#codepath%23currentsegments)中所公告的，规则API中的`CodePath#currentSegments`已被移除，请参阅帖子以获取更多详情。

### 预计算代码路径

在ESLint v9.0.0中，在规则使用的遍历之前，就已经计算了代码路径信息。因此，无论在哪里访问规则内部的代码路径信息，信息现在都是完整的。

在ESLint v8.x版本中，如果所检查的文件包含多次相同的`/* eslint */`配置注释，那么只会应用最后的注释，而其他的则会被忽略。

在ESLint v9.0.0中，前一个规则被应用，而其他规则则作为语法检查错误报告。

### 使用 `--quiet` 选项更加高效

`--quiet`选项在ESLint控制台中隐藏所有警告。在v9.0.0中，我们通过不执行任何设置为`"warn"`的规则来进行性能改进。

### 运行`eslint`时没有文件参数

如果您使用的是扁平配置并且没有向CLI传递任何文件参数，则CLI将默认为对当前目录进行lint操作，这意味着您可以输入`npx eslint`，它将正常工作。（使用eslintrc配置文件进行相同操作将导致错误。）

### 未使用的禁用指令默认会引发警告

ESLint长期以来能够标记未使用的禁用指令。在此版本中，我们已默认启用未使用的禁用指令的警告。您可以在配置文件中使用`linterOptions.reportUnusedDisableDirectives`或在命令行中使用`--report-unused-disable-directives-severity`来修改此值。

### 通过`--stats`在格式化程序中提供的性能统计信息

[规则分析器](https://eslint.org/docs/latest/extend/custom-rules#profile-rule-performance)现在在CLI中使用[`--stats` flag](/docs/latest/extend/stats#cli-usage)时，其信息现在可以在格式化程序内获取。这使得任何人都可以创建自定义可视化性能信息的方式，这些信息是ESLint跟踪的。

## Breaking Changes

## Features

## Bug修复

## 文档

+   [`e151050`](https://github.com/eslint/eslint/commit/e151050e64b57f156c32f6d0d1f20dce08b5a610) 文档：更新入门指南到新的`@eslint/create-config`（[#18217](https://github.com/eslint/eslint/issues/18217))（唯然）

+   [`94178ad`](https://github.com/eslint/eslint/commit/94178ad5cf4cfa1c8664dd8ac878790e72c90d8c) 文档：在扁平配置中提到`name`字段（[#18252](https://github.com/eslint/eslint/issues/18252))（Anthony Fu）

+   [`1765c24`](https://github.com/eslint/eslint/commit/1765c24df2f48ab1c1565177b8c6dbef63acf977) 文档：添加故障排除页面（[#18181](https://github.com/eslint/eslint/issues/18181))（Josh Goldberg ✨）

+   [`96607d0`](https://github.com/eslint/eslint/commit/96607d0581845fab19f832cd435547f9da960733) 文档：版本选择器同步（[#18260](https://github.com/eslint/eslint/issues/18260))（Milos Djermanovic）

+   [`651ec91`](https://github.com/eslint/eslint/commit/651ec9122d0bd8dd08082098bd1e1a24892983f2) 文档：从规则示例中删除`/* eslint-env */`注释（[#18249](https://github.com/eslint/eslint/issues/18249))（Milos Djermanovic）

+   [`950c4f1`](https://github.com/eslint/eslint/commit/950c4f11c6797de56a5b056affd0c74211840957) 文档：更新自述文件（GitHub操作机器人）

+   [`12f5746`](https://github.com/eslint/eslint/commit/12f574628f2adbe1bfed07aafecf5152b5fc3f4d) 文档：添加有关扁平配置中点文件和目录的信息（[#18239](https://github.com/eslint/eslint/issues/18239))（Tanuj Kanti）

+   [`b93f408`](https://github.com/eslint/eslint/commit/b93f4085c105117a1081b249bd50c0831127fab3) 文档：更新共享设置示例（[#18251](https://github.com/eslint/eslint/issues/18251))（Tanuj Kanti）

+   [`26384d3`](https://github.com/eslint/eslint/commit/26384d3367e11bd4909a3330b72741742897fa1f) 文档：修复一个示例中的 `ecmaVersion`，增加检查（[#18241](https://github.com/eslint/eslint/issues/18241)）（Milos Djermanovic）

+   [`7747097`](https://github.com/eslint/eslint/commit/77470973a0c2cae8ce07a456f2ad95896bc8d1d3) 文档：更新 PR 审查流程（[#18233](https://github.com/eslint/eslint/issues/18233)）（Nicholas C. Zakas）

+   [`b07d427`](https://github.com/eslint/eslint/commit/b07d427826f81c2bdb683d04879093c687479edf) 文档：修复拼写错误（[#18246](https://github.com/eslint/eslint/issues/18246)）（Kirill Gavrilov）

+   [`778082d`](https://github.com/eslint/eslint/commit/778082d4fa5e2fc97549c9e5acaecc488ef928f5) 文档：添加词汇表页面（[#18187](https://github.com/eslint/eslint/issues/18187)）（Josh Goldberg ✨）

+   [`239a7e2`](https://github.com/eslint/eslint/commit/239a7e27209a6b861d634b3ef245ebbb805793a3) 文档：澄清 [`sort-imports`](/docs/rules/sort-imports) 选项的描述（[#18198](https://github.com/eslint/eslint/issues/18198)）（gyeongwoo park）

+   [`4769c86`](https://github.com/eslint/eslint/commit/4769c86cc16e0b54294c0a394a1ec7ed88fc334f) 文档：修复 [`no-lone-blocks`](/docs/rules/no-lone-blocks) 中的错误示例（[#18215](https://github.com/eslint/eslint/issues/18215)）（Tanuj Kanti）

+   [`5251327`](https://github.com/eslint/eslint/commit/5251327711a2d7083e3c629cb8e48d9d1e809add) 文档：更新 README（GitHub Actions Bot）

+   [`1dc8618`](https://github.com/eslint/eslint/commit/1dc861897e8b47280e878d609c13c9e41892f427) 文档：更新 README（GitHub Actions Bot）

+   [`ba1c1bb`](https://github.com/eslint/eslint/commit/ba1c1bbc6ba9d57a83d04f450566337d3c3b0448) 文档：更新 README（GitHub Actions Bot）

+   [`337cdf9`](https://github.com/eslint/eslint/commit/337cdf9f7ad939df7bc55c23d953e12d847b6ecc) 文档：解释 RuleTester 修复测试的限制（[#18175](https://github.com/eslint/eslint/issues/18175)）（Nicholas C. Zakas）

+   [`c7abd89`](https://github.com/eslint/eslint/commit/c7abd8936193a87be274174c47d6775e6220e354) 文档：解释 Node.js 版本支持（[#18176](https://github.com/eslint/eslint/issues/18176)）（Nicholas C. Zakas）

+   [`d961eeb`](https://github.com/eslint/eslint/commit/d961eeb855b6dd9118a78165e358e454eb1d090d) 文档：在规则文档示例中显示红色下划线（[#18041](https://github.com/eslint/eslint/issues/18041)）（Yosuke Ota）

+   [`558274a`](https://github.com/eslint/eslint/commit/558274abbd25ef269f4994cf258b2e44afbad548) 文档：更新 README（GitHub Actions Bot）

+   [`2908b9b`](https://github.com/eslint/eslint/commit/2908b9b96ab7a25fe8044a1755030b18186a75b0) 文档：更新发布文档（[#18174](https://github.com/eslint/eslint/issues/18174)）（Nicholas C. Zakas）

+   [`1f1260e`](https://github.com/eslint/eslint/commit/1f1260e863f53e2a5891163485a67c55d41993aa) 文档：用 GitHub 咨询替换 HackerOne 链接（[#18165](https://github.com/eslint/eslint/issues/18165)）（Francesco Trotta）

+   [`e5ef3cd`](https://github.com/eslint/eslint/commit/e5ef3cd6953bb40108556e0465653898ffed8420) 文档：在 [`no-fallthrough`](/docs/rules/no-fallthrough) 中添加内联案例条件 ([#18158](https://github.com/eslint/eslint/issues/18158)) (Tanuj Kanti)

+   [`450d0f0`](https://github.com/eslint/eslint/commit/450d0f044023843b1790bd497dfca45dcbdb41e4) 文档：修复 `ignore` 选项文档 ([#18154](https://github.com/eslint/eslint/issues/18154)) (Francesco Trotta)

+   [`5fe095c`](https://github.com/eslint/eslint/commit/5fe095cf718b063dc5e58089b0a6cbcd53da7925) 文档：在下拉列表中显示 v8.57.0 作为最新版本 ([#18142](https://github.com/eslint/eslint/issues/18142)) (Milos Djermanovic)

+   [`7db5bb2`](https://github.com/eslint/eslint/commit/7db5bb270f95d1472de0bfed0e33ed5ab294942e) 文档：在下拉列表中显示预发布版本 ([#18135](https://github.com/eslint/eslint/issues/18135)) (Nicholas C. Zakas)

+   [`73a5f06`](https://github.com/eslint/eslint/commit/73a5f0641b43e169247b0000f44a366ee6bbc4f2) 文档：更新 README (GitHub Actions Bot)

+   [`f95cd27`](https://github.com/eslint/eslint/commit/f95cd27679eef228173e27e170429c9710c939b3) 文档：禁止在同一示例中使用多个规则配置注释 ([#18116](https://github.com/eslint/eslint/issues/18116)) (Milos Djermanovic)

+   [`d8068ec`](https://github.com/eslint/eslint/commit/d8068ec70fac050e900dc400510a4ad673e17633) 文档：更新模式示例的链接 ([#18112](https://github.com/eslint/eslint/issues/18112)) (Svetlana)

+   [`f1c7e6f`](https://github.com/eslint/eslint/commit/f1c7e6fc8ea77fcdae4ad1f8fe1cd104a281d2e9) 文档：切换到 Ethical Ads ([#18090](https://github.com/eslint/eslint/issues/18090)) (Strek)

+   [`15c143f`](https://github.com/eslint/eslint/commit/15c143f96ef164943fd3d39b5ad79d9a4a40de8f) 文档：PR 模板中的 JS Foundation 改为 OpenJS Foundation ([#18092](https://github.com/eslint/eslint/issues/18092)) (Nicholas C. Zakas)

+   [`6ea339e`](https://github.com/eslint/eslint/commit/6ea339e658d29791528ab26aabd86f1683cab6c3) 文档：在 v9 迁移指南中添加更严格的规则测试验证 ([#18085](https://github.com/eslint/eslint/issues/18085)) (Milos Djermanovic)

+   [`3c816f1`](https://github.com/eslint/eslint/commit/3c816f193eecace5efc6166efa2852a829175ef8) 文档：从 CLI 到核心概念使用相对链接 ([#18083](https://github.com/eslint/eslint/issues/18083)) (Milos Djermanovic)

+   [`9458735`](https://github.com/eslint/eslint/commit/9458735381269d12b24f76e1b2b6fda1bc5a509b) 文档：修复规则示例中格式错误的 `eslint` 配置注释 ([#18078](https://github.com/eslint/eslint/issues/18078)) (Francesco Trotta)

+   [`07a1ada`](https://github.com/eslint/eslint/commit/07a1ada7166b76c7af6186f4c5e5de8b8532edba) 文档：从 `--fix` CLI 文档链接到相关的核心概念 ([#18080](https://github.com/eslint/eslint/issues/18080)) (Bryan Mishkin)

+   [`b844324`](https://github.com/eslint/eslint/commit/b844324e4e8f511c9985a96c7aca063269df9570) 文档：更新团队责任 ([#18048](https://github.com/eslint/eslint/issues/18048)) (Nicholas C. Zakas)

+   [`aadfb60`](https://github.com/eslint/eslint/commit/aadfb609f1b847e492fc3b28ced62f830fe7f294) 文档：为上下文更新语言选项和其他 v9 更改进行文档化 ([#18074](https://github.com/eslint/eslint/issues/18074)) (fnx)

+   [`857e242`](https://github.com/eslint/eslint/commit/857e242584227181ecb8af79fc6bc236b9975228) 文档：微调 meta.docs 规则属性的解释 ([#18057](https://github.com/eslint/eslint/issues/18057)) (Bryan Mishkin)

+   [`10485e8`](https://github.com/eslint/eslint/commit/10485e8b961d045514bc1e34227cf09867a6c4b7) 文档：建议使用 messageId 而不是 message 来报告规则违规 ([#18050](https://github.com/eslint/eslint/issues/18050)) (Bryan Mishkin)

+   [`98b5ab4`](https://github.com/eslint/eslint/commit/98b5ab406bac6279eadd84e8a5fd5a01fc586ff1) 文档：更新 README (GitHub Actions Bot)

+   [`505fbf4`](https://github.com/eslint/eslint/commit/505fbf4b35c14332bffb0c838cce4843a00fad68) 文档：更新 [`no-restricted-imports`](/docs/rules/no-restricted-imports) 规则 ([#18015](https://github.com/eslint/eslint/issues/18015)) (Tanuj Kanti)

+   [`c25b4af`](https://github.com/eslint/eslint/commit/c25b4aff1fe35e5bd9d4fcdbb45b739b6d253828) 文档：更新 README (GitHub Actions Bot)

+   [`33d1ab0`](https://github.com/eslint/eslint/commit/33d1ab0b6ea5fcebca7284026d2396df41b06566) 文档：增加平坦配置忽略文档的更多示例 ([#18020](https://github.com/eslint/eslint/issues/18020)) (Milos Djermanovic)

+   [`e6eebca`](https://github.com/eslint/eslint/commit/e6eebca90750ef5c7c99d4fe3658553cf737dab8) 文档：更新 [sort-keys](/docs/rules/sort-keys) 选项属性计数 ([#18025](https://github.com/eslint/eslint/issues/18025)) (LB (Ben Johnston))

+   [`1fedfd2`](https://github.com/eslint/eslint/commit/1fedfd28a46d86b2fbcf06a2328befafd6535a88) 文档：优化平坦配置忽略文档 ([#17997](https://github.com/eslint/eslint/issues/17997)) (Nicholas C. Zakas)

+   [`38b9b06`](https://github.com/eslint/eslint/commit/38b9b06695f88c70441dd15ae5d97ffd8088be23) 文档：更新 [valid-typeof](/docs/rules/valid-typeof) 规则 ([#18001](https://github.com/eslint/eslint/issues/18001)) (Tanuj Kanti)

+   [`b4abfea`](https://github.com/eslint/eslint/commit/b4abfea4c1703a50f1ce639e3207ad342a56f79d) 文档：更新关于 ECMAScript 支持的说明 ([#17991](https://github.com/eslint/eslint/issues/17991)) (Francesco Trotta)

+   [`6788873`](https://github.com/eslint/eslint/commit/6788873328a7f974d5e45c0be06ca0c7dd409acd) 文档：更新发布博客模板 ([#17994](https://github.com/eslint/eslint/issues/17994)) (Nicholas C. Zakas)

+   [`1f37442`](https://github.com/eslint/eslint/commit/1f3744278433006042b8d5f4e9e1e488b2bbb011) 文档：添加关于非 npm 插件配置的部分 ([#17984](https://github.com/eslint/eslint/issues/17984)) (Nicholas C. Zakas)

+   [`96307da`](https://github.com/eslint/eslint/commit/96307da837c407c9a1275124b65ca29c07ffd5e4) 文档：迁移指南中的 [`no-inner-declarations`](/docs/rules/no-inner-declarations) 条目 ([#17977](https://github.com/eslint/eslint/issues/17977)) (塔努吉·坎蒂)

+   [`40be60e`](https://github.com/eslint/eslint/commit/40be60e0186cdde76219df4e8e628125df2912d8) 文档：更新 README (GitHub Actions Bot)

+   [`d31c180`](https://github.com/eslint/eslint/commit/d31c180312260d1a286cc8162907b6a33368edc9) 文档：修复自定义规则页面中代码路径事件的数量 ([#17969](https://github.com/eslint/eslint/issues/17969)) (理查德·亨特)

+   [`1529ab2`](https://github.com/eslint/eslint/commit/1529ab288ec815b2690864e04dd6d0a1f0b537c6) 文档：重新排列 v9 迁移指南中的条目 ([#17967](https://github.com/eslint/eslint/issues/17967)) (米洛什·杰尔马诺维奇)

+   [`9507525`](https://github.com/eslint/eslint/commit/95075251fb3ce35aaf7eadbd1d0a737106c13ec6) 文档：说明如何组合配置项 ([#17947](https://github.com/eslint/eslint/issues/17947)) (尼古拉斯·C·扎卡斯)

+   [`7c78576`](https://github.com/eslint/eslint/commit/7c785769fd177176966de7f6c1153480f7405000) 文档：在迁移到 v9 指南中添加更多已删除的 `context` 方法 ([#17951](https://github.com/eslint/eslint/issues/17951)) (米洛什·杰尔马诺维奇)

+   [`3a877d6`](https://github.com/eslint/eslint/commit/3a877d68d0151679f8bf1cabc39746778754b3dd) 文档：更新已删除的 CLI 标志迁移指南 ([#17939](https://github.com/eslint/eslint/issues/17939)) (尼古拉斯·C·扎卡斯)

+   [`4a9cd1e`](https://github.com/eslint/eslint/commit/4a9cd1ea1cd0c115b98d07d1b6018ca918a9c73f) 文档：为 v9 更新 Linter API ([#17937](https://github.com/eslint/eslint/issues/17937)) (米洛什·杰尔马诺维奇)

+   [`2a8eea8`](https://github.com/eslint/eslint/commit/2a8eea8e5847f4103d90d667a2b08edf9795545f) 文档：更新 v9.0.0-alpha.0 的文档 ([#17929](https://github.com/eslint/eslint/issues/17929)) (米洛什·杰尔马诺维奇)

+   [`7f0ba51`](https://github.com/eslint/eslint/commit/7f0ba51bcef3e6fbf972ceb20403238f0e1f0ea9) 文档：在版本选择器中显示 `NEXT` ([#17911](https://github.com/eslint/eslint/issues/17911)) (米洛什·杰尔马诺维奇)

+   [`0a7911e`](https://github.com/eslint/eslint/commit/0a7911e09adf2aca4d93c81f4be1cd80db7dd735) 文档：在 v9 迁移指南中添加 flat 配置默认值 ([#17927](https://github.com/eslint/eslint/issues/17927)) (米洛什·杰尔马诺维奇)

+   [`94f8065`](https://github.com/eslint/eslint/commit/94f80652aca302e2715ea51c10c3a1010786b751) 文档：将 CLI 更新添加到迁移到 v9 指南中 ([#17924](https://github.com/eslint/eslint/issues/17924)) (尼古拉斯·C·扎卡斯)

+   [`16187f2`](https://github.com/eslint/eslint/commit/16187f23c6e5aaed3b50ff551a66f758893d5422) 文档：将导出和字符串配置注释添加到迁移到 v9 指南中 ([#17926](https://github.com/eslint/eslint/issues/17926)) (尼古拉斯·C·扎卡斯)

+   [`3ae50cc`](https://github.com/eslint/eslint/commit/3ae50cc788c3cdd209e642573e3c831dd86fa0cd) 文档：将 RuleTester 更改添加到迁移到 v9 指南中 ([#17923](https://github.com/eslint/eslint/issues/17923)) (Nicholas C. Zakas)

+   [`0831b58`](https://github.com/eslint/eslint/commit/0831b58fe6fb5778c92aeb4cefa9ecedbbfbf48b) 文档：将规则更改添加到 v9 迁移指南中 ([#17925](https://github.com/eslint/eslint/issues/17925)) (Milos Djermanovic)

+   [`037abfc`](https://github.com/eslint/eslint/commit/037abfc21f264fca3a910c4a5cd23d1bf6826c3d) 文档：更新 API 文档 ([#17919](https://github.com/eslint/eslint/issues/17919)) (Milos Djermanovic)

+   [`afc3c03`](https://github.com/eslint/eslint/commit/afc3c038ed3132a99659604624cc24e702eec45a) 文档：将 function-style 和 `meta.schema` 更改添加到 v9 迁移指南中 ([#17912](https://github.com/eslint/eslint/issues/17912)) (Milos Djermanovic)

+   [`1da0723`](https://github.com/eslint/eslint/commit/1da0723695d080008b22f30c8b5c86fe386c6242) 文档：在迁移到 v9.x 中更新 `eslint:recommended` 部分 ([#17908](https://github.com/eslint/eslint/issues/17908)) (Milos Djermanovic)

+   [`f55881f`](https://github.com/eslint/eslint/commit/f55881f492d10e9c759e459ba6bade1be3dad84b) 文档：移除 [configuration-files-new.md](http://configuration-files-new.md) ([#17907](https://github.com/eslint/eslint/issues/17907)) (Milos Djermanovic)

+   [`63ae191`](https://github.com/eslint/eslint/commit/63ae191070569a9118b5972c90a98633b0a336e1) 文档：迁移到 v9.0.0 ([#17905](https://github.com/eslint/eslint/issues/17905)) (Nicholas C. Zakas)

+   [`e708496`](https://github.com/eslint/eslint/commit/e7084963c73f3cbaae5d569b4a2bee1509dd8cef) 文档：默认切换到平面配置 ([#17840](https://github.com/eslint/eslint/issues/17840)) (Nicholas C. Zakas)

+   [`fdf0424`](https://github.com/eslint/eslint/commit/fdf0424c5c08c058479a6cd7676be6985e0f400f) 文档：更新创建插件的平面配置 ([#17826](https://github.com/eslint/eslint/issues/17826)) (Nicholas C. Zakas)

+   [`e6a91bd`](https://github.com/eslint/eslint/commit/e6a91bdf401e3b765f2b712e447154e4a2419fbc) 文档：切换共享配置文档以使用平面配置 ([#17827](https://github.com/eslint/eslint/issues/17827)) (Nicholas C. Zakas)

+   [`3831fb7`](https://github.com/eslint/eslint/commit/3831fb78daa3da296b71823f61f8e3a4556ff7d3) 文档：更新 [`max-lines`](/docs/rules/max-lines) 规则的示例 ([#17898](https://github.com/eslint/eslint/issues/17898)) (Tanuj Kanti)

+   [`cd1ac20`](https://github.com/eslint/eslint/commit/cd1ac2041f48f2b6d743ebf671d0279a70de6eea) 文档：更新 README (GitHub Actions Bot)

+   [`26010c2`](https://github.com/eslint/eslint/commit/26010c209d2657cd401bf2550ba4f276cb318f7d) 构建：为 9.0.0-rc.0 更新变更日志 (Jenkins)

+   [`b91f9dc`](https://github.com/eslint/eslint/commit/b91f9dc072f17f5ea79803deb86cf002d031b4cf) 构建：修复 prism-eslint-hooks.js 中的 TypeError ([#18209](https://github.com/eslint/eslint/issues/18209)) (Francesco Trotta)

+   [`d7ec0d1`](https://github.com/eslint/eslint/commit/d7ec0d1fbdbafa139d090ffd8b42d33bd4aa46f8) 构建：更新 9.0.0-beta.2 的更改日志（Jenkins）

+   [`fd9c0a9`](https://github.com/eslint/eslint/commit/fd9c0a9f0e50da617fe1f2e60ba3df0276a7f06b) 构建：更新 9.0.0-beta.1 的更改日志（Jenkins）

+   [`c9f2f33`](https://github.com/eslint/eslint/commit/c9f2f3343e7c197e5e962c68ef202d6a1646866e) 构建：更新 8.57.0 的更改日志（[#18144](https://github.com/eslint/eslint/issues/18144)）（Milos Djermanovic）

+   [`1bbc495`](https://github.com/eslint/eslint/commit/1bbc495aecbd3e4a4aaf54d7c489191809c1b65b) 构建：更新 9.0.0-beta.0 的更改日志（Jenkins）

+   [`96f8877`](https://github.com/eslint/eslint/commit/96f8877de7dd3d92ac5afb77c92d821002d24929) 构建：更新 9.0.0-alpha.2 的更改日志（Jenkins）

+   [`52d5e7a`](https://github.com/eslint/eslint/commit/52d5e7a41d37a1a6d9aa1dffba3b688573800536) 构建：更新 9.0.0-alpha.1 的更改日志（Jenkins）

+   [`c2bf27d`](https://github.com/eslint/eslint/commit/c2bf27def29ef1ca7f5bfe20c1306bf78087ea29) 构建：发布预发版时更新文档文件（[#17940](https://github.com/eslint/eslint/issues/17940)）（Milos Djermanovic）

+   [`e91d85d`](https://github.com/eslint/eslint/commit/e91d85db76c7bd8a5998f7ff52d2cc844d0e953e) 构建：更新 9.0.0-alpha.0 的更改日志（Jenkins）

## 杂务

+   [`19f9a89`](https://github.com/eslint/eslint/commit/19f9a8926bd7888ab4a813ae323ad3c332fd5d5c) 杂务：为 v9.0.0 更新依赖项（[#18275](https://github.com/eslint/eslint/issues/18275)）（Nicholas C. Zakas）

+   [`7c957f2`](https://github.com/eslint/eslint/commit/7c957f295dcd97286016cfb3c121dbae72f26a91) 杂务：package.json 更新至 @eslint/js 发布版（Jenkins）

+   [`d73a33c`](https://github.com/eslint/eslint/commit/d73a33caddc34ab1eb62039f0f661a338836147c) 杂务：忽略 `/docs/v8.x` 在链接检查中（[#18274](https://github.com/eslint/eslint/issues/18274)）（Milos Djermanovic）

+   [`44a81c6`](https://github.com/eslint/eslint/commit/44a81c6151c58a3f4c1f6bb2927b0996f81c2daa) 杂务：升级 knip（[#18272](https://github.com/eslint/eslint/issues/18272)）（Lars Kappert）

+   [`e80b60c`](https://github.com/eslint/eslint/commit/e80b60c342f59db998afefd856b31159a527886a) 杂务：删除测试版本选择器代码（[#18266](https://github.com/eslint/eslint/issues/18266)）（Milos Djermanovic）

+   [`a98babc`](https://github.com/eslint/eslint/commit/a98babcda227649b2299d10e3f887241099406f7) 杂务：添加 npm 脚本以运行 WebdriverIO 测试（[#18238](https://github.com/eslint/eslint/issues/18238)）（Francesco Trotta）

+   [`9b7bd3b`](https://github.com/eslint/eslint/commit/9b7bd3be066ac1f72fa35c4d31a1b178c7e2b683) 杂务：更新依赖 markdownlint 至 ^0.34.0（[#18237](https://github.com/eslint/eslint/issues/18237)）（renovate[bot]）

+   [`297416d`](https://github.com/eslint/eslint/commit/297416d2b41f5880554d052328aa36cd79ceb051) 杂务：package.json 更新至 eslint-9.0.0-rc.0（[#18223](https://github.com/eslint/eslint/issues/18223)）（Francesco Trotta）

+   [`d363c51`](https://github.com/eslint/eslint/commit/d363c51b177e085b011c7fde1c5a5a09b3db9cdb) chore: 更新 package.json 以支持 @eslint/js 发布 (Jenkins)

+   [`1b841bb`](https://github.com/eslint/eslint/commit/1b841bb04ac642c5ee84d1e44be3e53317579526) chore: 修正一些注释 ([#18213](https://github.com/eslint/eslint/issues/18213)) (avoidaway)

+   [`29c3595`](https://github.com/eslint/eslint/commit/29c359599c2ddd168084a2c8cbca626c51d0dc13) chore: 移除重复的单词 ([#18193](https://github.com/eslint/eslint/issues/18193)) (cuithon)

+   [`acc2e06`](https://github.com/eslint/eslint/commit/acc2e06edd55eaab58530d891c0a572c1f0ec453) chore: 引入 Knip ([#18005](https://github.com/eslint/eslint/issues/18005)) (Lars Kappert)

+   [`7509276`](https://github.com/eslint/eslint/commit/75092764db117252067558bd3fbbf0c66ac081b7) chore: 升级 @eslint/js@9.0.0-beta.2 ([#18180](https://github.com/eslint/eslint/issues/18180)) (Milos Djermanovic)

+   [`96087b3`](https://github.com/eslint/eslint/commit/96087b33dc10311bba83e22cc968919c358a0188) chore: 更新 package.json 以支持 @eslint/js 发布 (Jenkins)

+   [`925afa2`](https://github.com/eslint/eslint/commit/925afa2b0c882f77f6b4411bdca3cb8ad6934b56) chore: 移除部分使用了`lodash.merge`的地方 ([#18179](https://github.com/eslint/eslint/issues/18179)) (Milos Djermanovic)

+   [`972ef15`](https://github.com/eslint/eslint/commit/972ef155a94ad2cc85db7d209ad869869222c14c) chore: 修正 @eslint/js 中的无效类型 ([#18164](https://github.com/eslint/eslint/issues/18164)) (Nitin Kumar)

+   [`32ffdd1`](https://github.com/eslint/eslint/commit/32ffdd181aa673ccc596f714d10a2f879ec622a7) chore: 升级 @eslint/js@9.0.0-beta.1 ([#18146](https://github.com/eslint/eslint/issues/18146)) (Milos Djermanovic)

+   [`e41425b`](https://github.com/eslint/eslint/commit/e41425b5c3b4c885f2679a3663bd081911a8b570) chore: 更新 package.json 以支持 @eslint/js 发布 (Jenkins)

+   [`bb3b9c6`](https://github.com/eslint/eslint/commit/bb3b9c68fe714bb8aa305be5f019a7a42f4374ee) chore: 升级 @eslint/eslintrc@3.0.2 ([#18145](https://github.com/eslint/eslint/issues/18145)) (Milos Djermanovic)

+   [`e462524`](https://github.com/eslint/eslint/commit/e462524cc318ffacecd266e6fe1038945a0b02e9) chore: 升级 eslint-release@3.2.2 ([#18138](https://github.com/eslint/eslint/issues/18138)) (Milos Djermanovic)

+   [`8e13a6b`](https://github.com/eslint/eslint/commit/8e13a6beb587e624cc95ae16eefe503ad024b11b) chore: 修正 [README.md](http://README.md) 中的拼写错误 ([#18128](https://github.com/eslint/eslint/issues/18128)) (Will Eastcott)

+   [`66f52e2`](https://github.com/eslint/eslint/commit/66f52e276c31487424bcf54e490c4ac7ef70f77f) chore: 移除未使用的工具规则 rule-types.json，更新 rule-types.js ([#18125](https://github.com/eslint/eslint/issues/18125)) (Josh Goldberg ✨)

+   [`bf0c7ef`](https://github.com/eslint/eslint/commit/bf0c7effdba51c48b929d06ce1965408a912dc77) ci: 修复 pr-labeler 的 sync-labels 值 ([#18124](https://github.com/eslint/eslint/issues/18124)) (Tanuj Kanti)

+   [`cace6d0`](https://github.com/eslint/eslint/commit/cace6d0a3afa5c84b18abee4ef8c598125143461) CI：添加 PR 标签动作（[#18109](https://github.com/eslint/eslint/issues/18109)）（Nitin Kumar）

+   [`1a65d3e`](https://github.com/eslint/eslint/commit/1a65d3e4a6ee16e3f607d69b998a08c3fed505ca) 日常事务：从 `eslint-config-eslint` 导出 `base` 配置（[#18119](https://github.com/eslint/eslint/issues/18119)）（Milos Djermanovic）

+   [`9aa4df3`](https://github.com/eslint/eslint/commit/9aa4df3f4d85960eee72923f3b9bfc88e62f04fb) 重构：移除 `globals` 依赖项（[#18115](https://github.com/eslint/eslint/issues/18115)）（Milos Djermanovic）

+   [`e40d1d7`](https://github.com/eslint/eslint/commit/e40d1d74a5b9788cbec195f4e602b50249f26659) 日常事务：升级 @eslint/js@9.0.0-beta.0（[#18108](https://github.com/eslint/eslint/issues/18108)）（Milos Djermanovic）

+   [`9870f93`](https://github.com/eslint/eslint/commit/9870f93e714edefb410fccae1e9924a3c1972a2e) 日常事务：更新 package.json 以适配 @eslint/js 发布（Jenkins）

+   [`2c62e79`](https://github.com/eslint/eslint/commit/2c62e797a433e5fc298b976872a89c594f88bb19) 日常事务：升级 @eslint/eslintrc@3.0.1（[#18107](https://github.com/eslint/eslint/issues/18107)）（Milos Djermanovic）

+   [`81f0294`](https://github.com/eslint/eslint/commit/81f0294e651928b49eb49495b90b54376073a790) 日常事务：升级 espree@10.0.1（[#18106](https://github.com/eslint/eslint/issues/18106)）（Milos Djermanovic）

+   [`5e2b292`](https://github.com/eslint/eslint/commit/5e2b2922aa65bda54b0966d1bf71acda82b3047c) 日常事务：升级 eslint-visitor-keys@4.0.0（[#18105](https://github.com/eslint/eslint/issues/18105)）（Milos Djermanovic）

+   [`ce838ad`](https://github.com/eslint/eslint/commit/ce838adc3b673e52a151f36da0eedf5876977514) 日常事务：使用 npm-run-all2 替换依赖项 npm-run-all ^5.0.0（[#18045](https://github.com/eslint/eslint/issues/18045)）（renovate[bot]）

+   [`54df731`](https://github.com/eslint/eslint/commit/54df731174d2528170560d1f765e1336eca0a8bd) 日常事务：更新依赖项 markdownlint-cli 到 ^0.39.0（[#18084](https://github.com/eslint/eslint/issues/18084)）（renovate[bot]）

+   [`8f06a60`](https://github.com/eslint/eslint/commit/8f06a606845f40aaf0fea1fd83d5930747c5acec) 日常事务：更新依赖项 shelljs 到 ^0.8.5（[#18079](https://github.com/eslint/eslint/issues/18079)）（Francesco Trotta）

+   [`93ffe30`](https://github.com/eslint/eslint/commit/93ffe30da5e2127e336c1c22e69e09ec0558a8e6) 日常事务：更新依赖项 file-entry-cache 到 v8（[#17903](https://github.com/eslint/eslint/issues/17903)）（renovate[bot]）

+   [`6ffdcbb`](https://github.com/eslint/eslint/commit/6ffdcbb8c51956054d3f81c5ce446c15dcd51a6f) 日常事务：升级 @eslint/js@9.0.0-alpha.2（[#18038](https://github.com/eslint/eslint/issues/18038)）（Milos Djermanovic）

+   [`2c12715`](https://github.com/eslint/eslint/commit/2c1271528e88d0c3c6a92eeee902001f1703d5c9) 日常事务：更新 package.json 以适配 @eslint/js 发布（Jenkins）

+   [`cc74c4d`](https://github.com/eslint/eslint/commit/cc74c4da99368b97494b924dbea1cb6e87adec53) 维护：升级 espree@10.0.0 ([#18037](https://github.com/eslint/eslint/issues/18037)) （Milos Djermanovic）

+   [`dfb68b6`](https://github.com/eslint/eslint/commit/dfb68b63ce6e8df6ffe81bd843e650c5b017dce9) 维护：为文档站点使用 Node.js 20 ([#18026](https://github.com/eslint/eslint/issues/18026)) （Milos Djermanovic）

+   [`8c1b8dd`](https://github.com/eslint/eslint/commit/8c1b8dda169920c4e3b99f6548f9c872d65ee426) 测试：为忽略文件和目录添加更多测试 ([#18018](https://github.com/eslint/eslint/issues/18018)) （Milos Djermanovic）

+   [`60b966b`](https://github.com/eslint/eslint/commit/60b966b6861da11617ddc15487bd7a51c584c596) 维护：更新依赖 @eslint/js 至 v9.0.0-alpha.1 ([#18014](https://github.com/eslint/eslint/issues/18014)) （renovate[bot]）

+   [`c893bc0`](https://github.com/eslint/eslint/commit/c893bc0bdf1bca256fbab6190358e5f922683249) 维护：更新 `markdownlint` 至 `v0.33.0` ([#17995](https://github.com/eslint/eslint/issues/17995)) （Nitin Kumar）

+   [`c5e50ee`](https://github.com/eslint/eslint/commit/c5e50ee65cf22871770b1d4d438b9056c577f646) 维护：更新 package.json 至 @eslint/js 发布版本（Jenkins）

+   [`1bf2520`](https://github.com/eslint/eslint/commit/1bf2520c4166aa55596417bf44c567555bc65fba) 维护：将文档 CI 从核心 CI 中分离 ([#17897](https://github.com/eslint/eslint/issues/17897)) （Nicholas C. Zakas）

+   [`320787e`](https://github.com/eslint/eslint/commit/320787e661beb979cf063d0f8333654f94ef9efd) 维护：删除 relative-module-resolver.js ([#17981](https://github.com/eslint/eslint/issues/17981)) （Francesco Trotta）

+   [`4926f33`](https://github.com/eslint/eslint/commit/4926f33b96faf07a64aceec5f1f4882f4faaf4b5) 重构：使用 `Object.hasOwn()` ([#17948](https://github.com/eslint/eslint/issues/17948)) （Milos Djermanovic）

+   [`df200e1`](https://github.com/eslint/eslint/commit/df200e147705eb62f94b99c170554327259c65d4) 重构：使用 `Array.prototype.at()` 获取最后的元素 ([#17949](https://github.com/eslint/eslint/issues/17949)) （Milos Djermanovic）

+   [`750b8df`](https://github.com/eslint/eslint/commit/750b8dff6df02a500e12cb78390fd14814c82e5b) 维护：更新依赖 glob 至 v10 ([#17917](https://github.com/eslint/eslint/issues/17917)) （renovate[bot]）

+   [`74794f5`](https://github.com/eslint/eslint/commit/74794f53a6bc88b67653c737f858cfdf35b1c73d) 维护：移除未使用的 eslintrc 模块 ([#17938](https://github.com/eslint/eslint/issues/17938)) （Milos Djermanovic）

+   [`10ed29c`](https://github.com/eslint/eslint/commit/10ed29c0c4505dbac3bb05b0e3d61f329b99f747) 维护：移除未使用的依赖 rimraf ([#17934](https://github.com/eslint/eslint/issues/17934)) （Francesco Trotta）

+   [`903ee60`](https://github.com/eslint/eslint/commit/903ee60ea910aee344df7edb66874f80e4b6ed31) CI：在安装 eslint 时使用 `--force` 标志 ([#17921](https://github.com/eslint/eslint/issues/17921)) （Milos Djermanovic）

+   [`17fedc1`](https://github.com/eslint/eslint/commit/17fedc17e9e6e39ad986d917fb4e9e4835c50482) 日常：升级 @eslint/js 到 9.0.0-alpha.0（[#17928](https://github.com/eslint/eslint/issues/17928)）（Milos Djermanovic）

+   [`cb89ef3`](https://github.com/eslint/eslint/commit/cb89ef373fffbed991f4e099cb255a7c116889f9) 日常：更新 package.json 以配合 @eslint/js 发布（Jenkins）

+   [`f6f4a45`](https://github.com/eslint/eslint/commit/f6f4a45680039f720a2fccd5f445deaf45babb3d) 日常：放弃 v9 的 structuredClone 兼容性补丁（[#17915](https://github.com/eslint/eslint/issues/17915)）（Kevin Gibbons）

+   [`412dcbb`](https://github.com/eslint/eslint/commit/412dcbb672b5a5859f96753afa7cb87291135a1b) 日常：升级 eslint-plugin-n 到 16.6.0（[#17916](https://github.com/eslint/eslint/issues/17916)）（Milos Djermanovic）

+   [`02a8baf`](https://github.com/eslint/eslint/commit/02a8baf9f2ef7b309c7d45564a79ed5d2153057f) 日常：重命名带有下划线的文件（[#17910](https://github.com/eslint/eslint/issues/17910)）（Nicholas C. Zakas）

+   [`c0f5d91`](https://github.com/eslint/eslint/commit/c0f5d913b0f07de332dfcecf6052f1e64bf3d2fb) 日常：移除在测试中创建未使用的 Linter 实例（[#17902](https://github.com/eslint/eslint/issues/17902)）（Milos Djermanovic）

+   [`3826cdf`](https://github.com/eslint/eslint/commit/3826cdf89294d079be037a9ab30b7506077b26ac) 日常：使用 jsdoc/no-multi-asterisks 并允许空格：true（[#17900](https://github.com/eslint/eslint/issues/17900)）（Percy Ma）

+   [`a9a17b3`](https://github.com/eslint/eslint/commit/a9a17b3f1cb6b6c609bda86a618ac5ff631285d2) 日常：修复在测试中获取作用域的问题（[#17899](https://github.com/eslint/eslint/issues/17899)）（Milos Djermanovic）

+   [`595a1f6`](https://github.com/eslint/eslint/commit/595a1f689edb5250d8398af13c3e4bd19d284d92) 测试：确保 CLI 测试与 FlatESLint 一起运行（[#17884](https://github.com/eslint/eslint/issues/17884)）（Francesco Trotta）

+   [`c7eca43`](https://github.com/eslint/eslint/commit/c7eca43202be98f6ff253b46c9a38602eeb92ea0) 日常：更新依赖 markdownlint-cli 到 ^0.38.0（[#17865](https://github.com/eslint/eslint/issues/17865)）（renovate[bot]）

+   [`cc0c9f7`](https://github.com/eslint/eslint/commit/cc0c9f707aa9da7965b98151868b3c249c7f8f30) 持续集成：将 github/codeql-action 从 2 升级到 3（[#17873](https://github.com/eslint/eslint/issues/17873)）（dependabot[bot]）
