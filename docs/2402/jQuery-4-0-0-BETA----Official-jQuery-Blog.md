<!--yml

类别：未分类

日期：2024-05-27 14:41:21

-->

# jQuery 4.0.0 BETA！| jQuery 官方博客

> 来源：[https://blog.jquery.com/2024/02/06/jquery-4-0-0-beta/](https://blog.jquery.com/2024/02/06/jquery-4-0-0-beta/)

# jQuery 4.0.0 BETA！

* * *

jQuery 4.0.0 已经在长期的开发中了，但现在已准备好发布 beta 版本！有很多内容需要介绍，团队对发布感到兴奋。我们进行了错误修复，性能改进，并引入了一些重大变更。终于，我们移除了对 IE<11 的支持！尽管如此，我们预计影响会很小。

许多重大变更是团队多年来一直想要进行的，但无法在补丁或次要版本中进行。我们已精简了遗留代码，移除了一些之前不推荐使用的 API，删除了一些内部函数的仅限公共功能参数，这些参数从未有文档记录，并放弃了一些过于复杂的“神奇”行为。

在最终版本发布之前，我们将发布一份全面的升级指南，概述移除的代码及迁移方法。[jQuery Migrate 插件](https://github.com/jquery/jquery-migrate)也将准备就绪以提供帮助。现在，请尝试 beta 版本，并[告诉我们是否遇到任何问题](https://github.com/jquery/jquery/issues)。

如往常一样，此版本可在[我们的 CDN](https://jquery.com/download/)和 npm 包管理器上获取。第三方 CDN 将不会托管此 beta 版本，但稍后将会托管 4.0.0 正式发布版。以下是 jQuery 4.0.0 beta 的一些亮点。

## 告别 IE<11

jQuery 4.0 放弃了对 IE 10 及更早版本的支持。有人可能会问为什么我们没有移除对 IE 11 的支持。我们计划分阶段移除支持，并且下一步将在 jQuery 5.0 中发布。目前，我们将开始删除专门支持低于 11 版本的 IE 的代码，这使我们在一个 PR 中减少了[-867 个已压缩字节](https://github.com/jquery/jquery/pull/4347)！

我们还放弃了对其他非常老旧的浏览器的支持，包括 Edge Legacy、iOS <11、Firefox <65 和 Android Browser。在你的端上不需要做任何更改。如果你需要支持这些浏览器中的任何一个，只需继续使用 jQuery 3.x 即可。

## 已移除不推荐使用的 API。

这些功能已经在多个版本中被弃用。现在我们已经达到了主要发布版本，是时候[移除它们了](https://github.com/jquery/jquery/issues/4056)。这些功能要么一直是内部使用的，要么现在在所有支持的浏览器中都有原生的替代方案。被移除的功能包括：

## `push`、`sort` 和 `splice` 已移除。

jQuery原型长期以来一直有不像其他任何jQuery方法那样行为的数组方法，总是只用于内部使用。这些方法包括 `push`、`sort` 和 `splice`。我们将这些方法的使用切换为数组函数而不是jQuery原型。例如，`$elems.push( elem )` 变成了 `[].push.call( $elems, elem )`。我们在这里提到它，以防有任何插件依赖于这些方法。

## `focusin` 和 `focosout` 事件顺序

长期以来，浏览器在焦点和失焦事件的顺序上意见不一致，其中包括 `focusin`、`focusout`、`focus` 和 `blur`。最终，所有jQuery 4.0支持的最新版本浏览器都收敛到了共同的事件顺序。不幸的是，这与多年前jQuery选择的一致顺序不同，这使得这是一个破坏性变更。至少现在每个人都在同一个页面上了！

jQuery在早期版本中所有四个事件的顺序如下：

```
1\. focusout
2\. blur
3\. focusin
4\. focus
```

从jQuery 4.0开始，我们[不再覆盖原生行为](https://github.com/jquery/jquery/pull/4362)。这意味着除了IE之外的所有浏览器都将遵循当前的W3C规范，即：

```
1\. blur
2\. focusout
3\. focus
4\. focusin
```

对于那些好奇的人，W3C规范先前定义了不同的顺序：

```
1\. focusout
2\. focusin
3\. blur
4\. focus
```

但是，少数人认为直观和[规范](https://www.w3.org/TR/uievents/#events-focusevent-event-order)在2023年[已更改](https://github.com/w3c/uievents/issues/88)，以匹配浏览器已经实现的内容。具有讽刺意味的是，唯一遵循旧规范的浏览器是Internet Explorer。

## `FormData` 支持

`jQuery.ajax` 已经[增加了对二进制数据的支持，包括`FormData`](https://github.com/jquery/jquery/pull/5197)。以前，二进制数据不是一种已知的数据类型，并且会被转换为字符串。可以通过禁用数据转换并手动处理数据来禁用此行为，但我们决定使其自动工作。这在技术上是一个破坏性的变化，但应该更接近预期的行为。

## 自动JSONP提升已移除

以前，带有提供的回调的 `dataType: "json"` 的 `jQuery.ajax` 请求会转换为JSONP请求。今天，与跨域后端交互的首选方式是使用[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，这在所有jQuery 4.0支持的浏览器中都有效。这应该有助于避免开发人员不知道代码可以通过JSONP从远程域执行而导致的意外行为。

## jQuery源码迁移到ES模块

当 jQuery 源代码从[AMD](https://requirejs.org/docs/whyamd.html)迁移到[ES modules](https://github.com/jquery/jquery/pull/4541)时，这是一个特殊的日子。jQuery 源代码始终以 jQuery 发布版的形式发布在 npm 和 GitHub 上，但在没有 [RequireJS](https://requirejs.org/) 的情况下无法直接导入为模块，而 RequireJS 是 jQuery 的构建工具选择。我们已经切换到 [Rollup](https://rollupjs.org/introduction/) 来打包 jQuery，并且我们确实对 ES 模块运行所有测试。

## Trusted Types 和 CSP

jQuery 4.0 添加了对[Trusted Types](https://twitter.com/kkotowicz/status/1445713282128515074)的支持，确保以不违反 `require-trusted-types-for` 内容安全策略指令的方式将包含在 [TrustedHTML](https://developer.mozilla.org/en-US/docs/Web/API/TrustedHTML) 中的 HTML 作为输入用于 jQuery 操作方法。

除此之外，尽管某些 AJAX 请求已经使用了 `<script>` 标签来维护 `crossdomain` 等属性，我们自从[将大多数异步脚本请求切换到使用 `<script>` 标签](https://github.com/jquery/jquery/pull/4763)以避免使用内联脚本导致的任何 CSP 错误之后。仍有少数情况下会使用 XHR 进行异步脚本请求，例如传递了 `"headers"` 选项（请使用 `scriptAttrs` 替代！），但我们现在尽可能使用 `<script>` 标签。

## 更新瘦身版本

在 jQuery 4.0.0 中，瘦身版本因移除 Deferreds 和 Callbacks（现在压缩后不到 20k 字节！）而变得更小。Deferreds 长期支持[Promises A+ 标准](https://promisesaplus.com/)，因此在大多数情况下可以使用原生 Promise，并且它们在 jQuery 支持的所有浏览器中都可用，除了 IE11。Deferreds 还具有一些额外功能，而原生 Promise 不支持这些功能，但大多数用法可以迁移到 Promise 方法上。如果需要支持 IE11，最好使用主版本或者为原生 Promise 添加一个 polyfill。

## 下载

您可以从 jQuery CDN 获取这些文件，或直接链接到它们：

[https://code.jquery.com/jquery-4.0.0-beta.js](https://code.jquery.com/jquery-4.0.0-beta.js)

[https://code.jquery.com/jquery-4.0.0-beta.min.js](https://code.jquery.com/jquery-4.0.0-beta.min.js)

您还可以从 npm 获取此版本：

`npm install jquery@4.0.0-beta`

### Slim build

有时您不需要ajax，或者您更喜欢使用专注于ajax请求的许多独立库。通常，使用CSS和类操作结合进行Web动画更简单。最后，除了IE11之外，jQuery支持的所有浏览器现在都全面支持原生Promise，因此在大多数情况下不再需要Deferreds和Callbacks。除了包含所有内容的常规版本外，我们还发布了一个“slim”版本，不包括这些模块。如今，jQuery的大小几乎不再是加载性能的问题，但“slim”版本比常规版本小约8k压缩字节。这些文件也可在npm包和CDN上获得：

[https://code.jquery.com/jquery-4.0.0-beta.slim.js](https://code.jquery.com/jquery-4.0.0-beta.slim.js)

[https://code.jquery.com/jquery-4.0.0-beta.slim.min.js](https://code.jquery.com/jquery-4.0.0-beta.slim.min.js)

这些更新已经作为当前版本在npm和Bower上提供。有关获取jQuery的所有方式的信息，请访问[https://jquery.com/download/](https://jquery.com/download/)。公共CDN今天会收到它们的副本，请给它们几天时间发布文件。如果您急于快速开始，请使用我们CDN上的文件，直到它们有机会更新。

## 谢谢

感谢所有参与此版本发布的人员，包括提交补丁、报告错误或进行测试的[Alex](https://github.com/sashashura)，[Ahmed S. El-Afifi](https://github.com/aelafifi)，[fecore1](https://github.com/fecore1)，[Dallas Fraser](https://github.com/fras2560)，[Richard Gibson](https://github.com/gibson042)，[Michał Gołębiowski-Owczarek](https://github.com/mgol)，[Pierre Grimaud](https://github.com/pgrimaud)，[Gabriela Gutierrez](https://github.com/gabibguti)，[Jonathan](https://github.com/vanillajonathan)，[Necmettin Karakaya](https://github.com/Necmttn)，[Anders Kaseorg](https://github.com/andersk)，[Wonseop Kim](https://github.com/wonseop)，[Simon Legner](https://github.com/simon04)，[Shashanka Nataraj](https://github.com/ShashankaNataraj)，[Pat O’Callaghan](https://github.com/patocallaghan)，[Christian Oliff](https://github.com/coliff)，[Dimitri Papadopoulos Orfanos](https://github.com/DimitriPapadopoulos)，[Wonhyoung Park](https://github.com/wonhyoung05)，[Bruno PIERRE](https://github.com/bubbatls)，[Baoshuo Ren](https://github.com/renbaoshuo)，[Beatriz Rezener](https://github.com/beatrizrezener)，[Sean Robinson](https://github.com/skrobinson)，[Ed Sanders](https://github.com/edg2s)，[Timo Tijhof](https://github.com/Krinkle)，[Tom](https://github.com/gaohuia)，[Christian Wenz](https://github.com/wenz)，[ygj6](https://github.com/ygj6)和整个jQuery团队。

## 我们在Mastodon上！

jQuery现在有自己的Mastodon账号。我们将从现在开始在Twitter和Mastodon上进行交叉发布。您可能还对关注一些拥有Mastodon账号的团队成员感兴趣。

jQuery: [https://social.lfx.dev/@jquery](https://social.lfx.dev/@jquery)

mgol: [https://hachyderm.io/@mgol](https://hachyderm.io/@mgol)

timmywil: [https://hachyderm.io/@timmywil](https://hachyderm.io/@timmywil)

### Ajax

+   不将数组数据视为二进制（[992a1911](https://github.com/jquery/jquery/commit/992a1911d0b6195012edc25fd5a48810d4be64b5)）

+   即使是二进制数据，也允许 `processData: true`（[ce264e07](https://github.com/jquery/jquery/commit/ce264e0789116e37fe371503537a217c038dfae8)）

+   支持二进制数据（包括 FormData）（[a7ed9a7b](https://github.com/jquery/jquery/commit/a7ed9a7b6364273b1b964fd2cf9691dec2cbec6b)）

+   即使跨域，也支持脚本传输的 `headers`（[#5142](https://github.com/jquery/jquery/issues/5142)，[6d136443](https://github.com/jquery/jquery/commit/6d1364431b63b0d3bbe1c5fd604131f9db453396)）

+   在 `jQuery.get` 中支持 `null` 作为成功函数（[#4989](https://github.com/jquery/jquery/issues/4989)，[74978b7e](https://github.com/jquery/jquery/commit/74978b7e892537559850cda7332bdab8106e6354)）

+   除非提供 dataType，否则不自动执行脚本（[#4822](https://github.com/jquery/jquery/issues/4822)，[025da4dd](https://github.com/jquery/jquery/commit/025da4dd343e6734f3d3c1b4785b1548498115d8)）

+   使 responseJSON 在错误的同域 JSONP 请求中起作用（[#4771](https://github.com/jquery/jquery/issues/4771)，[a1e619b0](https://github.com/jquery/jquery/commit/a1e619b03a557b47c3e26a5e74af12b63a0d5e73)）

+   执行 JSONP 错误脚本响应（[#4771](https://github.com/jquery/jquery/issues/4771)，[a1e619b0](https://github.com/jquery/jquery/commit/a1e619b03a557b47c3e26a5e74af12b63a0d5e73)）

+   避免在异步请求的脚本传输中出现 CSP 错误（[#3969](https://github.com/jquery/jquery/issues/3969)，[07a8e4a1](https://github.com/jquery/jquery/commit/07a8e4a177550025c1a08d7ac754839733943f55)）

+   移除 JSON 到 JSONP 的自动提升逻辑（[#1799](https://github.com/jquery/jquery/issues/1799)，[#3376](https://github.com/jquery/jquery/issues/3376)，[e7b3bc48](https://github.com/jquery/jquery/commit/e7b3bc488d01d584262e12a7c5c25f935d0d034b)）

+   如果存在，则用 content-type 头值覆盖 s.contentType（[#4119](https://github.com/jquery/jquery/issues/4119)，[7fb90a6b](https://github.com/jquery/jquery/commit/7fb90a6beaeffe16699800f73746748f6a5cc2de)）

+   废弃 AJAX 事件别名，将事件/别名内联到废弃的位置（[#1967](https://github.com/jquery/jquery/issues/1967)，[abdc89ac](https://github.com/jquery/jquery/commit/abdc89ac2e581392b800c0364e0f5f2b6a82cdc6)）

+   不要为不成功的 HTTP 响应执行脚本（[#4250](https://github.com/jquery/jquery/issues/4250)，[50871a5a](https://github.com/jquery/jquery/commit/50871a5a85cc802421b40cc67e2830601968affe)）

+   简化 jQuery.ajaxSettings.xhr（[#1967](https://github.com/jquery/jquery/issues/1967)，[abdc89ac](https://github.com/jquery/jquery/commit/abdc89ac2e581392b800c0364e0f5f2b6a82cdc6)）

### 属性

+   削减几个字节大小（[b40a4807](https://github.com/jquery/jquery/commit/b40a4807b604efbde51faf075d11e25441af1990)）

+   不在 setter 中转换属性值为字符串 ([#4948](https://github.com/jquery/jquery/issues/4948), [4250b628](https://github.com/jquery/jquery/commit/4250b628783d7bfa92ec6c5550c6e4b22fab6034))

+   删除 `toggleClass(boolean|undefined)` 签名 ([#3388](https://github.com/jquery/jquery/issues/3388), [a4421101](https://github.com/jquery/jquery/commit/a4421101fd6d9d7b0550210f8e8690641733dd9a))

+   重构 val(): 不去掉换行符，隔离 IE 的工作区 ([ff281991](https://github.com/jquery/jquery/commit/ff2819911da6cbbed5ee42c35d695240f06e65e3))

+   在 IE 之外完全不设置 type 属性钩子 ([9e66fe9a](https://github.com/jquery/jquery/commit/9e66fe9acf0ef27681f5a21149fc61678f791641))

### 构建

+   设置定期代码扫描分析 ([39c5778c](https://github.com/jquery/jquery/commit/39c5778c649ad387dac834832799c0087b11d5fe))

### 构建

+   将 grunt 作者迁移到自定义脚本 ([af79c999](https://github.com/jquery/jquery/commit/af79c99939628255f46f30bced000eba9aa6711f))

+   升级 follow-redirects 从 1.15.1 到 1.15.4 ([56139394](https://github.com/jquery/jquery/commit/56139394705022e4f6756440030ad6f3bf35f5a6))

+   升级 actions/setup-node 和 github/codeql-action ([99151d7a](https://github.com/jquery/jquery/commit/99151d7ab0923aa3aeeb1b957a9063e4e20d31ae))

+   重格式化 GitHub 工作流 YAML 文件 ([c98597ea](https://github.com/jquery/jquery/commit/c98597eaf5e144ee5e549cb41984687cd1033068))

+   升级 @babel/traverse 和多个操作 ([fb0cc272](https://github.com/jquery/jquery/commit/fb0cc272916dc909552a1b7bc1a39295e564d3a8))

+   不为 dependabot 分支运行 CI 推送工作流 ([635cb152](https://github.com/jquery/jquery/commit/635cb152e7daac658223455aaab2f81204b5b215))

+   更新与 ESLint 相关的包，修复 lint 错误 ([f47c6a83](https://github.com/jquery/jquery/commit/f47c6a83370675af0eff227d0266b40f9f45514a))

+   在 test:* npm 脚本之前运行 pretest ([1ad66aeb](https://github.com/jquery/jquery/commit/1ad66aeb6d7d94f8e4c8e2286569722ca41f9868))

+   按照比较大小排序分支；最后运行最后 ([a7fa303f](https://github.com/jquery/jquery/commit/a7fa303fda11ad298875676ffff78143cc49ce95))

+   在 jenkins 脚本中运行 pretest ([cb763072](https://github.com/jquery/jquery/commit/cb763072fee1eb9ec3d4037c50cb0d07836b7af6))

+   修复 Node 20 中不一致的构建问题 ([7ef9099d](https://github.com/jquery/jquery/commit/7ef9099d328e90d19bc40b64148747e854b13e20))

+   在比较中添加提交 SHAs 和最后运行信息 ([09972bcc](https://github.com/jquery/jquery/commit/09972bcc680e89e38f56d83043bb368eb7fbda91))

+   添加新的工厂文件到 dist eslint ([79223841](https://github.com/jquery/jquery/commit/792238410dc16ba0cc53c2740c47c314ea65d822))

+   升级 qs、socket.io-parser、socket.io 和 json5 ([b923047d](https://github.com/jquery/jquery/commit/b923047d29d37f2d5c96f8b33992f322bc7b7944))

+   将大多数 grunt 任务迁移到 grunt 之外 ([2bdecf8b](https://github.com/jquery/jquery/commit/2bdecf8b7bd10864e5337a4e24e39476c78cf23a))

+   更新 actions/checkout、actions/setup-node 和 github/codeql-action（[42e50f8c](https://github.com/jquery/jquery/commit/42e50f8c21fbfd08092ad81add4ac38982ef0841)）

+   更新 Krinkle 的 mailmap 条目（[699bcd39](https://github.com/jquery/jquery/commit/699bcd396fa342c546905805a0cdfedd1959b7ce)）

+   在缩小期间将 CRLF 替换为 LF（[48cc402a](https://github.com/jquery/jquery/commit/48cc402a917d6011c7d3e75f779f11ef91b474fb)）

+   在 package.json 中添加 `exports`，导出 slim 和 esm 构建（#4592）（[8be4c0e4](https://github.com/jquery/jquery/commit/8be4c0e4f89d6c8f780e5937a0534921d8c7815e)）

+   从 Terser 切换到 SWC 作为 JS 缩小工具（#5286）（[#5285](https://github.com/jquery/jquery/issues/5285)，[e2421875](https://github.com/jquery/jquery/commit/e24218758bb21bfdc296731d70f3d48ab786e5f5)）

+   确保 `*.cjs` 和 `*.mjs` 文件也使用 UNIX 换行符（[198b41c8](https://github.com/jquery/jquery/commit/198b41c8c2cd726b875615023b2b37b213040ad3)）

+   切换 timmywil 的首选电子邮件（[2b6b5e0a](https://github.com/jquery/jquery/commit/2b6b5e0a3ba3029ec3ad1525a178920765e3adf1)）

+   更新 github/codeql-action 和 actions/checkout（[4a13266e](https://github.com/jquery/jquery/commit/4a13266efd262a92f05d86b71d715885de103e6d)）

+   放弃单独的 AMD 模块（[5701957b](https://github.com/jquery/jquery/commit/5701957b7223659c52a43f8c2c5465fdf2803df4)）

+   通过提交 SHA 引用 GitHub Actions（#5266）（[784b9ba6](https://github.com/jquery/jquery/commit/784b9ba6e403997161113aa56d1747baed4e0767)）

+   从 UglifyJS 切换到 Terser 作为缩小工具（[27303c6b](https://github.com/jquery/jquery/commit/27303c6be09b8fc24c13454deae234e480cbf995)）

+   确保 `eslint:dev` 任务不再 lint `dist/` 文件夹（[44906a83](https://github.com/jquery/jquery/commit/44906a83d28a81f0107f8418a430db7e040a776b)）

+   在 Node.js 20 上进行测试，停止对 Node.js 14 和 19 的测试（[6616acff](https://github.com/jquery/jquery/commit/6616acff0a6c144c3eac3afae8578085bd325834)）

+   仅在需要时安装 Playwright 依赖项（[e77bd9d6](https://github.com/jquery/jquery/commit/e77bd9d64fc696cadfe1f8c9ebb50d7609a97b07)）

+   更新 actions/setup-node 从 3.5.1 到 3.6.0（[7e7bd062](https://github.com/jquery/jquery/commit/7e7bd062070b3eca8ee047136ea8575fbed5d70f)）

+   在 Playwright WebKit 上运行 GitHub Action 浏览器测试（[b02a257f](https://github.com/jquery/jquery/commit/b02a257f98688aa890e06a85672cd1a54c3ffa3a)）

+   将 middleware-mockserver 迁移到现代 JS（[ce90a484](https://github.com/jquery/jquery/commit/ce90a48450ba40586a6567235abb8fd2df84da97)）

+   从自定义构建中删除过时的 Insight 包（[c66d4700](https://github.com/jquery/jquery/commit/c66d4700dcf98efccb04061d575e242d28741223)）

+   限制 GitHub 工作流的权限（[c909d6b1](https://github.com/jquery/jquery/commit/c909d6b1ff444e68618b6da13d9c21714f681925)）

+   在 Node.js 18 和 19 上进行测试，停止对 Node 12 的测试（[f62d8e21](https://github.com/jquery/jquery/commit/f62d8e2159baf1eabf3b760b85b2dda56d569a09)）

+   从 3.5.0 升级 actions/setup-node 到 3.5.1（[0208224b](https://github.com/jquery/jquery/commit/0208224b5b76d54a39986f78aac97dbf1cccbe38)）

+   从 1.4.1 升级 Grunt 到 1.5.3（[aa231cd2](https://github.com/jquery/jquery/commit/aa231cd21421503d319ad6068e7df0fb3baa7fea)）

+   从 3.4.1 升级 actions/setup-node 到 3.5.0（[25400750](https://github.com/jquery/jquery/commit/25400750fb2e08b0a7e1a752a3ca0e9eaec16163)）

+   更新 GitHub Actions（[52f452b2](https://github.com/jquery/jquery/commit/52f452b2e8881e5ec5c9e880e277c8ecf633e8dc)）

+   添加 dependabot.yml 配置（GitHub Actions）（[3f8bb2a4](https://github.com/jquery/jquery/commit/3f8bb2a46daf76f1427f49810d06a210ffbc7016)）

+   测试在 Node 17 上，更新 Grunt 和 `karma-*` 包（[2525cffc](https://github.com/jquery/jquery/commit/2525cffc42934c0d5c7aa085bc45dd6a8282e840)）

+   在 GitHub Actions 中将安装步骤与运行测试分开（[eef97250](https://github.com/jquery/jquery/commit/eef972508c8be6cc3cd0039d34dc9fe16bac916c)）

+   从核心中删除 travis.yml 和 travis 提及（#4983）（[5f4d449a](https://github.com/jquery/jquery/commit/5f4d449aa836f376eab2d2ca7585d5476d12f095)）

+   将 CI 迁移到 GitHub Actions（[e23190e6](https://github.com/jquery/jquery/commit/e23190e63cb121da79b92e6641a81a44dcea9252)）

+   更新 ESLint 和 eslint-plugin-import，修复构建问题（[9735edd5](https://github.com/jquery/jquery/commit/9735edd5cb7b5ef30bb8acc4d7596a1410a971cc)）

+   改为在 Node.js 16 上测试而不是 15（[0f623fdc](https://github.com/jquery/jquery/commit/0f623fdc8db128657716290cb7d57430e224c977)）

+   从外部目录中也使用 core-js-bundle（[345cd22e](https://github.com/jquery/jquery/commit/345cd22e5664655ed315958ed2056610607c12ef)）

+   恢复外部目录（[a684e6ba](https://github.com/jquery/jquery/commit/a684e6ba836f7c553968d7d026ed7941e1a612d8)）

+   将存储库中的 master 重命名为 main（[8ae477a4](https://github.com/jquery/jquery/commit/8ae477a432f0924cd4bd3bdeaef2c4c15e483a8f)）

+   在 Node.js 15 上进行测试（[6984d174](https://github.com/jquery/jquery/commit/6984d1747623dbc5e87fd6c261a5b6b1628c107c)）

+   明确排除 slim 构建中的 queue 模块（[a503c691](https://github.com/jquery/jquery/commit/a503c691dc06c59acdafef6e54eca2613c6e4032)）

+   修复 WebStorm 中的 import/no-unused-modules ESLint 规则（[8612018d](https://github.com/jquery/jquery/commit/8612018d4eb570b39532fca8344a187d2e95f98e)）

+   将 .eslintignore 路径追加到 grunt eslint 路径中（[a22b43ba](https://github.com/jquery/jquery/commit/a22b43bad41592ec61e5fa0fcd2b3a3d44f8bfd3)）

+   使用 “favor”的美国拼写（[fa0058af](https://github.com/jquery/jquery/commit/fa0058af426c4e482059214c29c29f004254d9a1)）

+   修复 commitplease husky 配置（[#4735](https://github.com/jquery/jquery/issues/4735)，[3a1b338a](https://github.com/jquery/jquery/commit/3a1b338a7a579a45543b031a003abdce4dc6ac67)）

+   更新依赖项（[b5028669](https://github.com/jquery/jquery/commit/b502866960b30863e56968bd35e720905ac58025)）

+   事件：确保使用所有源模块的导出 (#4648) ([40c3abd0](https://github.com/jquery/jquery/commit/40c3abd0ab049449acab5f2e12c34b9cc3199482))

+   更新 eslint-config-jquery，修复 lint 违规问题 ([ef4d6ca6](https://github.com/jquery/jquery/commit/ef4d6ca6c3a1b276fedc27b1f3a18823276f01a3))

+   在引入通过 Rollup 编译的 ES 模块后进行后续操作 ([55cd3a44](https://github.com/jquery/jquery/commit/55cd3a44368d529326b6d9b16ff7c0204fee5516))

+   根据 jQuery 风格指南正确缩进代码 ([3d62d570](https://github.com/jquery/jquery/commit/3d62d5704989f17d3a20ae7521d52e9c8c60b4ee))

+   减少 slim 构建的头部注释和 jQuery.fn.jquery ([812b4a1a](https://github.com/jquery/jquery/commit/812b4a1a837c049b85efb73603105b4245cb0e5c))

+   将 ESLint max-len 禁用指令移动到 dist/.eslintrc.json 中 ([34296ec5](https://github.com/jquery/jquery/commit/34296ec547f0ab6577e8c31815447990a9c01b31))

+   在 Node.js 14 上进行测试，停止在 Node.js 8 和 13 上测试 ([88eb22e0](https://github.com/jquery/jquery/commit/88eb22e0599d546f98f6145c53deb086e1d82857))

+   在 ESLint 中启用 reportUnusedDisableDirectives ([46f9810b](https://github.com/jquery/jquery/commit/46f9810b73a7ad446d7c3711faf92f56b67df3c1))

+   解决 Travis 配置警告 ([5b94a4f8](https://github.com/jquery/jquery/commit/5b94a4f847fe2328b1b8f2340b11b6031f95d2d1))

+   在浏览器代码中启用 ESLint one-var 规则用于 var 声明 ([4a7fc854](https://github.com/jquery/jquery/commit/4a7fc8544e2020c75047456d11979e4e3a517fdf))

+   将 Christian Oliff 添加到 .mailmap 和 AUTHORS.txt 中 ([721744a9](https://github.com/jquery/jquery/commit/721744a9fab5b597febea64e466272eabfdb9463))

+   同时 lint 压缩后的 jQuery 文件 – Gruntfile 修复 ([#3075](https://github.com/jquery/jquery/issues/3075), [338f1fc7](https://github.com/jquery/jquery/commit/338f1fc77483a1bc1456e1f4ba1db2049bb45b45))

+   同时 lint 压缩后的 jQuery 文件 ([#3075](https://github.com/jquery/jquery/issues/3075), [89a18de6](https://github.com/jquery/jquery/commit/89a18de64cec73936507ea9feca24d029edea24f))

+   为 Travis 作业添加直观名称 ([e1fab109](https://github.com/jquery/jquery/commit/e1fab10911dfe3b93bf8bd5d276e30e6fc69f780))

+   修复 Karma 在开发模式下的工作问题，从磁盘上提供源文件 ([437f389a](https://github.com/jquery/jquery/commit/437f389a24a6bef213d4df507909e7e69062300b))

+   修复自定义构建测试，验证 Travis 上的测试 ([0f780ba7](https://github.com/jquery/jquery/commit/0f780ba7cc5968d53bba386bdcb59b8d9410873b))

+   为 Slim 构建创建一个 `grunt custom:slim` 别名 (#4578) ([9b9ed469](https://github.com/jquery/jquery/commit/9b9ed469b43e9fa6e2c752444470ae4c87d03d57))

+   使 Karma 在 ES 模块模式下工作 ([341c6d1b](https://github.com/jquery/jquery/commit/341c6d1b5abe4829f59fbc32e93f6a6a1afb900f))

+   自动转换源代码为 AMD 格式 ([f37c2e51](https://github.com/jquery/jquery/commit/f37c2e51f36c8f8bab3879064a90e86a685feafc))

+   修复 Windows 构建问题 ([#4548](https://github.com/jquery/jquery/issues/4548), [9fd2fa53](https://github.com/jquery/jquery/commit/9fd2fa5388dba5c71129a1d9e3bb8e4fe6e4eb0b))

+   要求 ES6 导入的扩展名，防止导入循环 ([44ac8c85](https://github.com/jquery/jquery/commit/44ac8c8529173711b66046ae5cfefa5bd4892461))

+   修复从 ajax.js 到 serialize.js 的导入路径问题 ([07532014](https://github.com/jquery/jquery/commit/075320149ae30a5c593c06b2fb015bdf033e0acf))

+   只在 Travis 上测试定义在配置中的浏览器 ([bcbcdd2b](https://github.com/jquery/jquery/commit/bcbcdd2b2c1bb7075f4f73dc89ca7d65db2a09ed))

+   在 Firefox ESR 上运行测试 ([2d5ad6d2](https://github.com/jquery/jquery/commit/2d5ad6d23e0f57c733ce4556d3f2ee93ca99cadb))

+   在 Node.js 13 上进行测试，除了 8、10 和 12 ([830976e6](https://github.com/jquery/jquery/commit/830976e690b5fffeac860e2fdd07986d087ce824))

+   在 Travis 上也使用 FirefoxHeadless 运行测试 ([584835e6](https://github.com/jquery/jquery/commit/584835e68239ce55d1fc007b284e8ef4ed2817c2))

+   通过 ESLint 在 Node.js 脚本中要求使用严格模式 ([bbad821c](https://github.com/jquery/jquery/commit/bbad821c399da92995a11b88d6684970479d4a9b))

+   支持 jquery-release 的 --dry-run 标志 ([d7d0b52b](https://github.com/jquery/jquery/commit/d7d0b52bda74486f2351baa9d03ca4534de0d733))

+   发布时停止将 src/core.js 复制到 dist 中 ([#4489](https://github.com/jquery/jquery/issues/4489), [9a4d9806](https://github.com/jquery/jquery/commit/9a4d980639dd804ad320685a25b8ff4572e3f595))

+   移除外部目录，直接从 node_modules 中读取 ([d7e64190](https://github.com/jquery/jquery/commit/d7e64190efc411e3973a79fd027bf1afe359f429))

+   ESLint: 禁止未使用的函数参数 ([438b1a3e](https://github.com/jquery/jquery/commit/438b1a3e8a52d3e4efd8aba45498477038849c97))

+   修复 AMD var-modules 中的正则表达式解析问题 (#4389) ([9ec09c3b](https://github.com/jquery/jquery/commit/9ec09c3b4aa5182c2a8b8f51afb861b685a4003c))

+   修复 curCSS 中的 AMD 依赖问题 ([b220f6df](https://github.com/jquery/jquery/commit/b220f6df88d34dd908f55d57417fcec377787e5c))

+   在 Node.js 12 上进行测试，停止在 Node.js 6 和 11 上测试 ([b8d47128](https://github.com/jquery/jquery/commit/b8d4712825a26a7f24c2bdb5a71aa3abcd345dfd))

+   修复 finalPropName 中未解析的 jQuery 引用问题 ([#4358](https://github.com/jquery/jquery/issues/4358), [87403058](https://github.com/jquery/jquery/commit/874030583c9b94603de467124420e6c7a1c3c8ac))

+   将 Sizzle 从 2.3.3 更新到 2.3.4 ([#1756](https://github.com/jquery/jquery/issues/1756), [#4170](https://github.com/jquery/jquery/issues/4170), [#4249](https://github.com/jquery/jquery/issues/4249), [0b2c36ad](https://github.com/jquery/jquery/commit/0b2c36adb4e2c048318659e4196e0925da10ead2))

+   将主版本更新为 4.0.0-pre ([c4f2fa2f](https://github.com/jquery/jquery/commit/c4f2fa2fb33d6e52f7c8fad9f687ef970f442179))

+   将 Sinon 从 2.3.7 更新到 7.3.1，其他更新 ([fea7a2a3](https://github.com/jquery/jquery/commit/fea7a2a328f475048b4450c5c02a60832fdcfc3c))

### 核心

+   添加有关命名导出的更多信息 ([5f869590](https://github.com/jquery/jquery/commit/5f869590924b7dea6a16d176b18700939f4b5290))

+   简化代码在浏览器支持减少后 ([93ca49e6](https://github.com/jquery/jquery/commit/93ca49e6d1ac23fee33b3bc3b7f4d93dd1a25cb7))

+   将工厂移到单独的导出项 ([46f6e3da](https://github.com/jquery/jquery/commit/46f6e3da796ee9d28c7c1428793b72d66bcbb0b7))

+   在 `src/` 中使用命名导出 ([#5262](https://github.com/jquery/jquery/issues/5262), [f75daab0](https://github.com/jquery/jquery/commit/f75daab09102a4dd5107deadb55d4a169f86254a))

+   修复 jQuery.text() 在 HTMLDocument 对象上的回归问题 ([#5264](https://github.com/jquery/jquery/issues/5264), [a75d6b52](https://github.com/jquery/jquery/commit/a75d6b52fad212820358e8ada3154f2f634e699b))

+   选择器：将 jQuery.contains 从选择器移到核心模块 ([024d8719](https://github.com/jquery/jquery/commit/024d87195ac46690023e2b0b308d4406a8a5a27e))

+   删除 jQuery.fn.init 的 root 参数 ([d2436df3](https://github.com/jquery/jquery/commit/d2436df36a4b2ef556907e734a90771f0dbdbcaf))

+   不要依赖输入中存在的 splice 方法 ([9c6f64c7](https://github.com/jquery/jquery/commit/9c6f64c7b51d50e334ef1183e2937ad77c0a68b0))

+   操作：添加基本的 TrustedHTML 支持 ([#4409](https://github.com/jquery/jquery/issues/4409), [de5398a6](https://github.com/jquery/jquery/commit/de5398a6ad088dc006b46c6a870a2a053f4cd663))

+   在 parseXML 中报告浏览器错误 ([#4784](https://github.com/jquery/jquery/issues/4784), [89697325](https://github.com/jquery/jquery/commit/8969732518470a7f8e654d5bc5be0b0076cb0b87))

+   使 jQuery.isXMLDoc 接受虚假输入 ([#4782](https://github.com/jquery/jquery/issues/4782), [fd421097](https://github.com/jquery/jquery/commit/fd421097c56696e4c1c4a99c1aae44c59a722be4))

+   放弃对 Edge Legacy（即非 Chromium Microsoft Edge）的支持 ([#4568](https://github.com/jquery/jquery/issues/4568), [e35fb62d](https://github.com/jquery/jquery/commit/e35fb62db4fb46f031056bb53e393982c03972a1))

+   在其上下文中触发 iframe 脚本，全局执行中添加 doc 参数 ([#4518](https://github.com/jquery/jquery/issues/4518), [4592595b](https://github.com/jquery/jquery/commit/4592595b478be979141ce35c693dbc6b65647173))

+   在 slim 构建中也排除回调和延迟模块 ([fbc44f52](https://github.com/jquery/jquery/commit/fbc44f52fe76e1b601da76a1d7f8ef27884c06da))

+   从 AMD 迁移到 ES 模块 🎉 ([d0ce00cd](https://github.com/jquery/jquery/commit/d0ce00cdfa680f1f0c38460bc51ea14079ae8b07))

+   在支持的地方使用 Array.prototype.flat ([#4320](https://github.com/jquery/jquery/issues/4320), [9df4f1de](https://github.com/jquery/jquery/commit/9df4f1de12728b44a4b0f91748f12421008d9079))

+   从 jQuery 原型中删除私有副本的 push、sort 和 splice 方法 ([b59107f5](https://github.com/jquery/jquery/commit/b59107f5d7451ac16a7c8755128719be6ec8bf12))

+   实现`.even()`和`.odd()`以替换POS中的`:even`和`:odd`（[78420d42](https://github.com/jquery/jquery/commit/78420d427cf3734d9264405fcbe08b76be182a95))

+   弃用jQuery.trim（[#4363](https://github.com/jquery/jquery/issues/4363), [5ea59460](https://github.com/jquery/jquery/commit/5ea5946094784f68437ef26d463dfcfbbbaff1f6))

+   移除IE特定支持测试，依赖于document.documentMode（[#4386](https://github.com/jquery/jquery/issues/4386), [3527a384](https://github.com/jquery/jquery/commit/3527a3840585e6a359cd712591c9c57398357b9b))

+   不再支持IE <11、iOS <11、Firefox <65、Android浏览器和PhantomJS（[#3950](https://github.com/jquery/jquery/issues/3950), [#4299](https://github.com/jquery/jquery/issues/4299), [cf84696f](https://github.com/jquery/jquery/commit/cf84696fd1d7fe314a11492606529b5a658ee9e3))

+   移除已弃用的jQuery API（[#4056](https://github.com/jquery/jquery/issues/4056), [58f0c00b](https://github.com/jquery/jquery/commit/58f0c00bed695f934bb205c6115e5fe99dd5c27b))

### CSS

+   修复对最初隐藏的iframe的reliableTrDimensions支持测试（[#b1e66a5f](https://github.com/jquery/jquery/commit/b1e66a5faaf46ffcbcc27c79a9a224aaf851a987))

+   选择器：与3.x版本对齐，移除外部的`selector.js`包装（[#3950](https://github.com/jquery/jquery/issues/3950), [#4299](https://github.com/jquery/jquery/issues/4299), [cf84696f](https://github.com/jquery/jquery/commit/cf84696fd1d7fe314a11492606529b5a658ee9e3))

+   使reliableTrDimensions支持测试与Bootstrap CSS一起工作（[#5270](https://github.com/jquery/jquery/issues/5270), [65b85031](https://github.com/jquery/jquery/commit/65b85031fb5688361c077bc04e641e4b502671e1))

+   使`offsetHeight( true )`等包括负边距（[#3982](https://github.com/jquery/jquery/issues/3982), [bce13b72](https://github.com/jquery/jquery/commit/bce13b72c1753e16cc0db53ebf0f0456bdcf6b48))

+   返回空白CSS变量值时返回`undefined`（#5120）([7eb00196](https://github.com/jquery/jquery/commit/7eb0019640a5856c42b451551eb7f995d913eba9))

+   不要修剪未定义自定义属性的空白（[#5105](https://github.com/jquery/jquery/issues/5105), [ed306c02](https://github.com/jquery/jquery/commit/ed306c0261ab63746040e5d58bb4477c3069a427))

+   在`addClass( array )`中跳过假值，压缩代码（[#4998](https://github.com/jquery/jquery/issues/4998), [a338b407](https://github.com/jquery/jquery/commit/a338b407f2479f82df40635055effc163835183f))

+   在CSS属性值周围使用rtrim的理由（[655c0ed5](https://github.com/jquery/jquery/commit/655c0ed5e204b1f6427e09d615a49586a7bc84eb))

+   修剪围绕CSS自定义属性值的空白（[#4926](https://github.com/jquery/jquery/issues/4926), [efadfe99](https://github.com/jquery/jquery/commit/efadfe991a5c287af561a9326bf1427d726c91c1))

+   在jQuery slim版本中包括`show`、`hide`和`toggle`方法（[297d18dd](https://github.com/jquery/jquery/commit/297d18dd13f7b810ea5a4afeefa4cb15d9e16e16))

+   移除不透明度CSS钩子（[#865469f5](https://github.com/jquery/jquery/commit/865469f5e60f55feb28469bb0a7526dd22f04b4e))

+   解决 IE/Edge 中表行的 getComputedStyle 问题 ([#4490](https://github.com/jquery/jquery/issues/4490), [26415e08](https://github.com/jquery/jquery/commit/26415e081b318dbe1d46d2b7c30e05f14c339b75))

+   不自动为少数属性添加“px”（[#2795](https://github.com/jquery/jquery/issues/2795), [00a9c2e5](https://github.com/jquery/jquery/commit/00a9c2e5f4c855382435cec6b3908eb9bd5a53b7))

### 数据

+   重构以减少体积 ([805cdb43](https://github.com/jquery/jquery/commit/805cdb43fd02c3a5783c06b5ec2c9519be0682ab))

+   事件：操作：防止与 Object.prototype 冲突 ([#3256](https://github.com/jquery/jquery/issues/3256), [9d76c0b1](https://github.com/jquery/jquery/commit/9d76c0b163675505d1a901e5fe5249a2c55609bc))

+   分离数据与 CSS/效果的 camelCase 实现 ([#3355](https://github.com/jquery/jquery/issues/3355), [8fae2120](https://github.com/jquery/jquery/commit/8fae21200e80647fec4389995c4879948d11ad66))

### 延迟

### 废弃的

+   使用非废弃方法定义 `.hover()` ([fd6ffc5e](https://github.com/jquery/jquery/commit/fd6ffc5eb2c12562f2656d2f33865448420252be))

+   移除 jQuery.trim ([0b676ae1](https://github.com/jquery/jquery/commit/0b676ae12d20721e2df6f6f32f37f7302f8805bf))

+   修复 AMD 参数顺序 ([f810080e](https://github.com/jquery/jquery/commit/f810080e8e92278bb5288cba7cc0169481471780))

### 尺寸

+   为不可靠的 TR 尺寸在 FF 中添加偏移属性回退 ([#4529](https://github.com/jquery/jquery/issues/4529), [3bbbc111](https://github.com/jquery/jquery/commit/3bbbc11111840d6fd5160db13f2c1a9acb05c4c4))

### 文档

+   修复包中 README 的模块链接 ([ace646f6](https://github.com/jquery/jquery/commit/ace646f6e83e653f666ba715c200739f1cbdba52))

+   在 CONTRIBUTING.md 中更新 watch 任务 ([77d6ad71](https://github.com/jquery/jquery/commit/77d6ad7172db3ae11573df7b322d410b161eb43e))

+   通过 codespell 发现并修复拼写错误 ([620870a1](https://github.com/jquery/jquery/commit/620870a1af5287d29c77ec6d5f973116b23793a7))

+   从 readme 中删除过时的 gitter 徽章 ([67cb1af7](https://github.com/jquery/jquery/commit/67cb1af7740a956e150e8d93266c4e601f55e8a4))

+   从 PR 模板中删除“Grunt build”部分 ([988a5684](https://github.com/jquery/jquery/commit/988a56847de301ce18a653f84b07c5af432a269f))

+   从 README 中删除过时徽章 ([bcd9c2bc](https://github.com/jquery/jquery/commit/bcd9c2bc3ddaa04f89f550681ca9c1ec5efc4328))

+   更新已发布包的 README ([edccabf1](https://github.com/jquery/jquery/commit/edccabf10d37b57cbd4eeebc44f3acb67cb2739c))

+   从 GitHub Actions 评论中删除 git.io ([016872ff](https://github.com/jquery/jquery/commit/016872ffe03ab9107b1bc62fae674a4809c3b23f))

+   在 README 中更新 webpack 网站 ([01819bc3](https://github.com/jquery/jquery/commit/01819bc3bcc44282e5bb9301c3478d837d1e5152))

+   添加链接到 patchwelcome 和 help wanted 问题 ([924b7ce8](https://github.com/jquery/jquery/commit/924b7ce825962bfe4c16e02eb411c7f66ee75a55))

+   添加链接以预览新的 CLA（[683ceb8f](https://github.com/jquery/jquery/commit/683ceb8ff067ac53a7cb464ba1ec3f88e353e3f5)）

+   修正不正确的 `trac-NUMBER` 引用 ([eb9ceb2f](https://github.com/jquery/jquery/commit/eb9ceb2facbeff1c66a41824bd0ac0c56d0c5c62))

+   从旧的 jQuery 源中删除已过期的链接 (#4997) ([ed066ac7](https://github.com/jquery/jquery/commit/ed066ac70270b4bb20b5717501d2d268ef144bd3))

+   从源代码中移除对 Web Archive 的链接 ([#4981](https://github.com/jquery/jquery/issues/4981), [e24f2dcf](https://github.com/jquery/jquery/commit/e24f2dcf3f6bda1a672502e0233c732065cbbe89))

+   将 `#NUMBER` Trac 问题引用替换为 `trac-NUMBER` ([5d5ea015](https://github.com/jquery/jquery/commit/5d5ea015114092c157311c4948f7cc3d8c8e7f8a))

+   在 CONTRIBUTING.md 中更新到最新的 jQuery 构建的 URL ([9bdb16cd](https://github.com/jquery/jquery/commit/9bdb16cd19097da67950a707baac3980bda873f3))

+   从拉取请求模板中移除 CLA 复选框 ([e1248931](https://github.com/jquery/jquery/commit/e124893132d7a979d7987f978e968a1f889348b6))

+   更新 IRC 到 Libera 并修复 LAMP 死链接 ([175db73e](https://github.com/jquery/jquery/commit/175db73ec7938e774d9e93d3afdfb35a24466b47))

+   在 GitHub 问题模板中更新经常报告的问题 ([7a6fae6a](https://github.com/jquery/jquery/commit/7a6fae6a7e51ae30a9f3177e8639fbf523ed0915))

+   将 JS Foundation 的提及更改为 OpenJS Foundation ([11611967](https://github.com/jquery/jquery/commit/11611967adf2bd9ff4304132f917629ec1134049))

+   添加 SECURITY.md，并显示安全电子邮件地址 ([2ffe54ca](https://github.com/jquery/jquery/commit/2ffe54ca53b4ba2de2012f83c3faf262c1003af9))

+   修正拼写错误 ([1a7332ce](https://github.com/jquery/jquery/commit/1a7332ce83cdee7d6cd9d45c2a4b83067f53f14b))

+   更新指向 jsdom 仓库的链接 ([a62309e0](https://github.com/jquery/jquery/commit/a62309e01b3c76d2b73560ca666c454b7bbfcb77))

+   在 README 中使用 HTTPS 进行超链接 ([73415da2](https://github.com/jquery/jquery/commit/73415da25d964ee31ec1804d55f5af0199a1378e))

+   从 README 中删除对事件/alias.js 模块的提及 ([3edfa1bc](https://github.com/jquery/jquery/commit/3edfa1bcdc50bca41ac58b2642b12f3feee03a3b))

+   更新指向通过 Web Archive 访问 EdgeHTML 问题的链接 ([1dad1185](https://github.com/jquery/jquery/commit/1dad1185e0b2ca2a13bf411558eda75fb2d4da88))

+   将用户直接引导到 GitHub 文档以克隆仓库 ([f1c16de2](https://github.com/jquery/jquery/commit/f1c16de29689d2cfaf629f00d682148e99753509))

+   在 README 中将 OS X 更改为 macOS ([5a3e0664](https://github.com/jquery/jquery/commit/5a3e0664d261422f11a78faaf101d70c73b3a5a8))

+   将大多数 URL 更新为 HTTPS ([f09d9210](https://github.com/jquery/jquery/commit/f09d92100ffff6208211b200ed0cdc39bfd17fc3))

+   将 Homebrew 的链接从 HTTP 转换为 HTTPS ([e0022f23](https://github.com/jquery/jquery/commit/e0022f23144fd1dc6db86a5d8c18af47bc14f0f3))

### 效果

+   修复 .stop() 中不必要的条件语句 ([#4374](https://github.com/jquery/jquery/issues/4374), [110802c7](https://github.com/jquery/jquery/commit/110802c7f22b677ef658963aa95ebdf5cb9c5573))

### 特效

### 事件

+   避免 jQuery.event.special 和 Object.prototype 之间的冲突 ([bcaeb000](https://github.com/jquery/jquery/commit/bcaeb000b777c018ad5d18e01be5060caa8cb158))

+   简化 leverageNative 中对保存数据的检查 ([dfe212d5](https://github.com/jquery/jquery/commit/dfe212d5a1eed6b4a67d1cbd04ece09bbac33699))

+   使 trigger(focus/blur/click) 能够与原生处理程序一起工作 ([#5015](https://github.com/jquery/jquery/issues/5015), [6ad3651d](https://github.com/jquery/jquery/commit/6ad3651dbfea9e9bb56e608f72b4ef2f97bd4e70))

+   通过 focusin/focusout 模拟 IE 中的焦点/模糊 ([#4856](https://github.com/jquery/jquery/issues/4856), [#4859](https://github.com/jquery/jquery/issues/4859), [#4950](https://github.com/jquery/jquery/issues/4950), [ce60d318](https://github.com/jquery/jquery/commit/ce60d31893deab7d3da592b5173e90b5d50e7732))

+   在`.on(focus).off(focus)`后不要破坏焦点触发 ([#4867](https://github.com/jquery/jquery/issues/4867), [e539bac7](https://github.com/jquery/jquery/commit/e539bac79e666bba95bba86d690b4e609dca2286))

+   使焦点重新触发时不再聚焦到原始元素 ([#4382](https://github.com/jquery/jquery/issues/4382), [dbcffb39](https://github.com/jquery/jquery/commit/dbcffb396c2db61ff96edc4162602e850797d61f))

+   如果元素在失焦时被移除则不崩溃 ([#4417](https://github.com/jquery/jquery/issues/4417), [5c2d0870](https://github.com/jquery/jquery/commit/5c2d08704e289dd2745bcb0557b35a9c0e6af4a4))

+   移除 event.which 的 shim ([#3235](https://github.com/jquery/jquery/issues/3235), [1a5fff4c](https://github.com/jquery/jquery/commit/1a5fff4c169dbaa2df72c656868bcf60ed4413d0))

+   删除 jQuery.event.global ([18db8717](https://github.com/jquery/jquery/commit/18db87172cffbe48b92e30b70249e304863a70f9))

+   仅将事件附加到接受数据的对象上 - 真实 ([#4397](https://github.com/jquery/jquery/issues/4397), [d5c505e3](https://github.com/jquery/jquery/commit/d5c505e35d8c74ce8e9d99731a1a7eab0e0d911c))

+   停止 shim focusin 和 focusout 事件 ([#4300](https://github.com/jquery/jquery/issues/4300), [8a741376](https://github.com/jquery/jquery/commit/8a741376937dfacf9f82b2b88f93b239fe267435))

+   防止 leverageNative 注册重复的虚拟处理程序 ([eb6c0a7c](https://github.com/jquery/jquery/commit/eb6c0a7c97b1b3cf00144de12d945c9c569f935c))

+   修复多个异步焦点事件处理 ([#4350](https://github.com/jquery/jquery/issues/4350), [ddfa8376](https://github.com/jquery/jquery/commit/ddfa83766478268391bc9da96683fc0d4973fcfe))

### 操作

+   将测试泛化以支持 IE ([88690ebf](https://github.com/jquery/jquery/commit/88690ebfc8b5ef8b1e444326c664b590ecc0b888))

+   支持 `$el.html(selfRemovingScript)`（#5378）（[#5377](https://github.com/jquery/jquery/issues/5377), [937923d9](https://github.com/jquery/jquery/commit/937923d9ee8dfd19008447b5059cbd13ee5a23ac)）

+   将 `domManip` 提取到一个单独的文件中（[ee6e8740](https://github.com/jquery/jquery/commit/ee6e874075ba1fcd8f9e62cd1ee5c04f6518b6d6)）

+   不要从脚本中移除 HTML 注释（[#4904](https://github.com/jquery/jquery/issues/4904), [2f8f39e4](https://github.com/jquery/jquery/commit/2f8f39e457c32c454c50545b0fdaa1d7a4a2f8bd)）

+   在 DOM 操作中尊重脚本 `crossorigin` 属性（[#4542](https://github.com/jquery/jquery/issues/4542), [15ae3614](https://github.com/jquery/jquery/commit/15ae361485056b236a9484a185238f992806e1ff)）

+   避免在 `buildFragment` 中串联字符串（[9c98e4e8](https://github.com/jquery/jquery/commit/9c98e4e86eda857ee063bc48adbc1a11bb5506ee)）

+   将 `jQuery.htmlPrefilter` 设为标识函数（[90fed4b4](https://github.com/jquery/jquery/commit/90fed4b453a5becdb7f173d9e3c1492390a1441f)）

+   使用 `nodeName` 工具来尽可能节省大小（[4504fc3d](https://github.com/jquery/jquery/commit/4504fc3d722dd029d861cb47aa65a5edc651c4d3)）

### 发布

+   直接使用 `buildDefaultFiles` 并传递版本信息（[b507c864](https://github.com/jquery/jquery/commit/b507c8648f701acd1c48b3c38054ad38d76fd1ca)）

+   同时复制 `dist-module` 文件夹（[63767650](https://github.com/jquery/jquery/commit/63767650b5b171b4671304fd2bb2f2890431929f)）

+   仅发布版本化的文件到 cdn（[3a0ca684](https://github.com/jquery/jquery/commit/3a0ca684eb21d64a13d7591ce1891b1990e0339c)）

+   从 dist package.json 中移除脚本和开发依赖项（[7eac932d](https://github.com/jquery/jquery/commit/7eac932da7177104546abef595adf4429eb829b3)）

+   在 Release.generateArtifacts 中更新构建命令（[3b963a21](https://github.com/jquery/jquery/commit/3b963a21662061e0f39ad90f146e73e2223c2b86)）

+   在 windows 中增加对 md5 校验和的支持（[f088c366](https://github.com/jquery/jquery/commit/f088c36631df3d5dc98408debd147ea5d3618557)）

+   不再需要全局安装 grunt（[b2bbaa36](https://github.com/jquery/jquery/commit/b2bbaa36d4d37bd48f954ed3cdbd50d3461a523d)）

+   升级发布依赖项（[967af732](https://github.com/jquery/jquery/commit/967af73203378db0cc3637adee85c442e246e05a)）

+   移除未使用的 chalk 依赖项（[bfb6897c](https://github.com/jquery/jquery/commit/bfb6897c558dfdccff7ac5fc377b08e806525be3)）

+   在仓库中使用一个 dist README fixture（[358b769a](https://github.com/jquery/jquery/commit/358b769a00c3a09a8ec621b8dcb2d5e31b7da69a)）

+   更新 `AUTHORS.txt`（[1b74660f](https://github.com/jquery/jquery/commit/1b74660f730d34bf728094c33080ff406427f41e)）

+   更新 `AUTHORS.txt`（[cf9fe0f6](https://github.com/jquery/jquery/commit/cf9fe0f6a104a0f527c7c3f441485c19e2b19c69)）

### Selector

+   使 `selector.js` 模块依赖于 `attributes/attr.js`（[#5379](https://github.com/jquery/jquery/issues/5379), [e06ff088](https://github.com/jquery/jquery/commit/e06ff08849057cd099365bf43598c8952fe9956d)）

+   从各种模块中消除对 `selector.js` 的依赖 ([e8b7db4b](https://github.com/jquery/jquery/commit/e8b7db4b0f1e1b8e08578641b30a92b955ccc4ec))

+   重新公开 jQuery.find.{tokenize,select,compile,setDocument} ([#5259](https://github.com/jquery/jquery/issues/5259), [338de359](https://github.com/jquery/jquery/commit/338de3599039a3ba906214e656bcbe637430c37d))

+   不再依赖于 CSS.supports( “selector(…)” ) ([#5194](https://github.com/jquery/jquery/issues/5194), [68aa2ef7](https://github.com/jquery/jquery/commit/68aa2ef7571e2d9f91fad1aa9e5f956c04dc9ee9))

+   将 jQuery 选择上下文逻辑回退到选择器本地化 ([#5185](https://github.com/jquery/jquery/issues/5185), [2e644e84](https://github.com/jquery/jquery/commit/2e644e845051703775b35b358eec5d3608a9465f))

+   让选择器列表再次与 `qSA` 协同工作 ([#5177](https://github.com/jquery/jquery/issues/5177), [09d988b7](https://github.com/jquery/jquery/commit/09d988b774e7ff4acfb69c0cde2dab373559aaca))

+   实现 `uniqueSort` 可链式化方法 ([#5166](https://github.com/jquery/jquery/issues/5166), [5266f23c](https://github.com/jquery/jquery/commit/5266f23cf49c9329bddce4d4af6cb5fbbd1e0383))

+   重新引入 selector-native.js ([4c1171f2](https://github.com/jquery/jquery/commit/4c1171f2ed62584211250df0af8302d34c04621a))

+   操作：修复模板内容内的 DOM 操作 ([#5147](https://github.com/jquery/jquery/issues/5147), [3299236c](https://github.com/jquery/jquery/commit/3299236c898136dc1aa57dc5148811203e931895))

+   放弃对旧伪类的支持，测试自定义伪类 ([8c7da22c](https://github.com/jquery/jquery/commit/8c7da22caeae8c2c3f7e9869d5f47414669f106c))

+   如果 `CSS.supports(selector(…))` 不兼容，则使用 jQuery 的 `:has` ([#5098](https://github.com/jquery/jquery/issues/5098), [d153c375](https://github.com/jquery/jquery/commit/d153c375e67f2c2dba82c2fb079c36b8d795e66a))

+   移除 Chrome <=77 中对 “a:enabled” 的回避方式 ([c1ee33ad](https://github.com/jquery/jquery/commit/c1ee33aded44051b8f1288b59d2efdc68d0413cc))

+   让空属性选择器在 IE 中再次生效 ([#4435](https://github.com/jquery/jquery/issues/4435), [05184cc4](https://github.com/jquery/jquery/commit/05184cc448f4ed7715ddd6a5d724e167882415f1))

+   在 uniqueSort 中使用浅层文档比较（[#4441](https://github.com/jquery/jquery/issues/4441), [15750b0a](https://github.com/jquery/jquery/commit/15750b0af270da07917b70457cf09bda97d3d935))

+   添加一个在逗号后无效选择器上抛出异常的测试 ([6eee5f7f](https://github.com/jquery/jquery/commit/6eee5f7f181f9ebf5aa428e96356667e3cf7ddbd))

+   让带有前置组合器的选择器再次使用 qSA ([ed66d5a2](https://github.com/jquery/jquery/commit/ed66d5a22b37425abf5b63c361f91340de89c994))

+   使用浅层文档比较来避免 IE/Edge 崩溃（[#4441](https://github.com/jquery/jquery/issues/4441), [aa6344ba](https://github.com/jquery/jquery/commit/aa6344baf87145ffc807a527d9c1fb03c96b1948)）

+   减小体积，简化 setDocument ([29a9544a](https://github.com/jquery/jquery/commit/29a9544a4fb743491a42f827a6cf8627b7b99e0f))

+   在可能的情况下利用 :scope 伪类 ([#4453](https://github.com/jquery/jquery/issues/4453), [df6a7f7f](https://github.com/jquery/jquery/commit/df6a7f7f0f615149266b1a51064293b748b29900))

+   恢复 querySelectorAll 的快捷使用方式 ([cef4b731](https://github.com/jquery/jquery/commit/cef4b73179b8d2a38cfd5e0730111cc80518311a))

+   将 Sizzle 内联到选择器模块中 ([47835965](https://github.com/jquery/jquery/commit/47835965bd100a3661d8299d8b769ceeb8b6ce48))

+   将 Sizzle 测试移植到 jQuery ([79b74e04](https://github.com/jquery/jquery/commit/79b74e043a4ee737d44a95094ff1184e40bd5b16))

### 支持

+   确保支持 div 的显示设置为块级 ([#4832](https://github.com/jquery/jquery/issues/4832), [09f25436](https://github.com/jquery/jquery/commit/09f254361f1fe8a563b8a90fe6a4d269f4b11514))

### 测试

+   禁用 “:lang respects escaped backslashes” 测试 ([#5271](https://github.com/jquery/jquery/issues/5271), [62b9a258](https://github.com/jquery/jquery/commit/62b9a2583460c2384f6de1d0a6aeaf05e51d523b))

+   指示 Chrome 112 和 Safari 16.4 通过 cssHas 支持测试 ([89ef81f8](https://github.com/jquery/jquery/commit/89ef81f86f8f371154e9fd3173be5fb57cb72d5e))

+   正确地测试 AJAX 废弃的事件别名 ([cff28998](https://github.com/jquery/jquery/commit/cff2899885c314d32eea42e9eef6ead6e5da5c2f))

+   指示 Firefox 106+ 通过 `cssSupportsSelector` 测试 ([716130e0](https://github.com/jquery/jquery/commit/716130e094caf780100a39cfd4526adbd7673b12))

+   移除 Firefox XML 解析问题的解决方法 ([e7ffe1f1](https://github.com/jquery/jquery/commit/e7ffe1f135dfa68ce3065b2bd319a29a57866dc6))

+   修复到 QUnit CSS 文件的链接 ([8cf39b78](https://github.com/jquery/jquery/commit/8cf39b78e6c36b360dd81268a7f84fb4ca218e15))

+   基于编译标志而不是 API 存在来排除测试 ([#5069](https://github.com/jquery/jquery/issues/5069), [fae5fee8](https://github.com/jquery/jquery/commit/fae5fee8b435cc20352d28b0a384b9784b1ad9ed))

+   解决 Firefox XML 解析 bug 的问题 ([af1cd6f2](https://github.com/jquery/jquery/commit/af1cd6f218f699abc34b1582a910c0df00312aee))

+   锁定 colors 版本为 1.4.0 ([9603b3c8](https://github.com/jquery/jquery/commit/9603b3c899af354a4f538fa5b15f9dac3fcc0f55))

+   在 TestSwarm 上跳过 ETag AJAX 测试 ([00c060d1](https://github.com/jquery/jquery/commit/00c060d1619d472a2d8c5b104ed76fa3afc2ce97))

+   允许 AJAX 测试中 statusText 为 "success" ([19ced963](https://github.com/jquery/jquery/commit/19ced963c63372eae5aca9e1a4baec80b78a2b8e))

+   使 Karma 浏览器超时时间大于 QUnit 的超时时间 ([4fd6912b](https://github.com/jquery/jquery/commit/4fd6912bfd8fffbfabc98a9b0789d28f10af0914))

+   在 mock.php 的 cspClean 操作中不要移除 csp.log ([1019074f](https://github.com/jquery/jquery/commit/1019074f7b1df96ee9d6409ada3dc0562046f6c7))

+   通过 HTTPS 加载 TestSwarm 监听器（[d225639a](https://github.com/jquery/jquery/commit/d225639a8ea62863482bd20249077688f60235db)）

+   将背景图片从在线文件切换到本地 1×1.jpg（[482f8462](https://github.com/jquery/jquery/commit/482f846203e82b1c2620f580e483bf41d11f9f49)）

+   从 mock.php 中去除非典型的回调参数字符（[a7027463](https://github.com/jquery/jquery/commit/a70274632dc19ff4a64d7bb7657a2cc647ff38b9)）

+   使更多测试在 Chrome 和 Firefox 上本地运行（[50e8e846](https://github.com/jquery/jquery/commit/50e8e84621ff7a314fca253ce73f0519322d8a4d)）

+   修复不带 dataType 的脚本不自动执行的测试（[d38528b1](https://github.com/jquery/jquery/commit/d38528b17a846b7ca4513b41150a05436546292d)）

+   在 Node.js 模拟服务器中识别带有点的回调（[df6858df](https://github.com/jquery/jquery/commit/df6858df2ed3fc5c424591a5e09b900eb4ce0417)）

+   在 Safari 中跳过“jQuery.ajax() 在卸载时的测试”（[c18dc496](https://github.com/jquery/jquery/commit/c18dc49699bc27481a4af36ed1a0ee1b19c6eb03)）

+   删除未使用的局部变量（[82b87f6f](https://github.com/jquery/jquery/commit/82b87f6f0e45ca4e717b4e3a4a20a592709a099f)）

+   删除剩余的 jQuery.cache 引用（[d96111e1](https://github.com/jquery/jquery/commit/d96111e18b42ae1bc7def72a8a0d156ea39e4d0e)）

+   解决 iOS 8 – 12 最近 XSS 测试的失败（[11066a9e](https://github.com/jquery/jquery/commit/11066a9e6ac183dd710d1bc7aa74a3f809757136)）

+   添加最近修复的操作 XSS 问题的测试（[dc06d68b](https://github.com/jquery/jquery/commit/dc06d68bdc4c2562b5cc530f21e668a17d78ee2d)）

+   每个匹配窗口和文档仅使用一个 focusin/out 处理程序（[9b732043](https://github.com/jquery/jquery/commit/9b7320435059e30af71d648ab34ac6c00c80f5ef)）

+   修复“jQuery.ajax() – JSONP – 同一域”测试中的不稳定性（[7b0864d0](https://github.com/jquery/jquery/commit/7b0864d0539bbfbb01d88d9bc95369580ffd99bc)）

+   传递必要的 done() 调用数量给 assert.async()（[364476c3](https://github.com/jquery/jquery/commit/364476c3dc1231603ba61fc08068fa89fb095e1a)）

+   删除过时的 jQuery 数据测试（[eb35be52](https://github.com/jquery/jquery/commit/eb35be528fdea40faab4d89ac859d38dfd024271)）

+   在 Firefox 中跳过“带有幻影边框的表行的宽度/高度”测试（[a612733b](https://github.com/jquery/jquery/commit/a612733be0c68d337647a6fcc8f8e0cabc1fc36b)）

+   不在 Chrome 中测试同步 XHR 在卸载时（[323575fb](https://github.com/jquery/jquery/commit/323575fb9bb330a852718d89e323f7ec79549100)）

+   在测试中停止使用 jQuery.find（[1d624c10](https://github.com/jquery/jquery/commit/1d624c10b4a6b97ac254bcefffa91058556075d2)）

+   从 Sizzle 移植更改（[ac5f7cd8](https://github.com/jquery/jquery/commit/ac5f7cd8e29ecc7cdf21c13199be5472375ffa0e)）

+   修复 testinit.js 中的注释（[7bdf307b](https://github.com/jquery/jquery/commit/7bdf307b51e4d4a891b123a96d4899e31bfba024)）

+   更新 npo.js 并包含未经缩小的源代码 ([b334ce77](https://github.com/jquery/jquery/commit/b334ce7735ae453bd5643b251f36c3cedce4b3de))

+   限制事件测试回退到 TestSwarm ([bde53edc](https://github.com/jquery/jquery/commit/bde53edcf4bd6c975d068eed4eb16c5ba09c1cff))

+   修复 IE 中的新 focusin/focusout 测试 ([6f2fae7c](https://github.com/jquery/jquery/commit/6f2fae7c410dcb5876814866a03fc819f0502290))

+   修复 core-js polyfill 包含方法 ([2e4b79ab](https://github.com/jquery/jquery/commit/2e4b79ab8f7c43d36537a743c4c1a1a5b17e130e))

### 遍历

+   修复 IE 中 `<object>` 元素有子元素时的 `contents()` 方法 ([ccbd6b93](https://github.com/jquery/jquery/commit/ccbd6b93424cbdbf86f07a86c2e55cbab497d7a3))

+   修复在 `<object>` 元素有子元素时的 `contents()` 方法（[#4384](https://github.com/jquery/jquery/issues/4384), [4d865d96](https://github.com/jquery/jquery/commit/4d865d96aa5aae91823c50020b5c19da79566811))
