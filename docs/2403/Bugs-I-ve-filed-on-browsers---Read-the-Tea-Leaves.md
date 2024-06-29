<!--yml

类别：未分类

日期：2024-05-27 14:36:57

-->

# 我在浏览器上提交了的 bug | 读懂茶叶

> 来源：[https://nolanlawson.com/2024/03/03/bugs-ive-filed-on-browsers/](https://nolanlawson.com/2024/03/03/bugs-ive-filed-on-browsers/)

我认为在浏览器上提交 bug 是 Web 开发人员可以做的最有用的事情之一。

当面对跨浏览器兼容性问题时，很多人习惯于只是寻找一些快速的解决方案，或者在各种替代方案之间反复尝试，直到某些方法奏效。这确实是我早期职业生涯中所做的事情。但我认为这样做是短视的。

浏览器开发团队就像 Web 开发团队一样 - 他们有优先事项和积压问题，有时会让一些 bug 溜过去。此外，一个写得很好、有明确重现步骤的 bug 报告通常可以迅速解决问题 - 尤其是如果你成功地[nerd-snipe](https://xkcd.com/356/)了某个无聊或好奇的工程师。

多年来，我在浏览器上提交了很多 bug。无论出于何种原因 - 固执、沮丧，或者是某种为大众服务的高傲感 - 我已经养成了每天缠着浏览器供应商解决我遇到的任何障碍的习惯。他们通常会修复！

因此，我认为分析我在主要浏览器引擎 - Chromium、Firefox 和 WebKit 上提交的 bug 在我大约 10 年的 Web 开发生涯中是一件有趣的事情。我排除了旧的和较少知名的浏览器引擎，我从未在其上提交过 bug，并且我还排除了 Trident/EdgeHTML，因为原始的“Microsoft Connect” bug 追踪器似乎已经下线。（另外，我确实曾经被雇佣来提交 EdgeHTML 的 bug 大约 2 年时间，所以这有点不公平。）

在人们开始得出结论之前，关于这组数据集的一些说明：

+   Chromium 的数据略有过高，因为我倾向于更多使用与 Chromedriver 相关的工具（例如 Puppeteer），而不是其他浏览器自动化工具。

+   WebKit 有点奇怪，因为许多这些 bug 最终发现是在专有的苹果系统中（如 Safari、iOS 等），而不是真正的 WebKit。至少，我认为这个谜一般的 `rdar://` 响应是这样的。[^([1])](#footnote-1)

+   我在这里排除了一个实际上在 MDN 上的 Firefox 缺陷（MDN 使用相同的 bug 追踪器）。

## 数据

因此，不多说了，这里是数据集：

| 浏览器 | 已提交 | 待处理 | 已解决 | 无效 | 解决率% |
| --- | --- | --- | --- | --- | --- |
| Chromium | 27 | 4 | 14 | 9 | 77.78% |
| Firefox | 18 | 3 | 8 | 7 | 72.73% |
| WebKit | 25 | 6 | 12 | 7 | 66.67% |
| **总计** | **70** | **13** | **34** | **23** | **72.34%** |

**注释：** 对于“无效”，我很宽容地包括“重复”，“过时”，“不打算修复”等情况。对于“解决率%”，我仅计算有效 bug 中已解决的数量的比例。

从数据集中我注意到的一些事情：

+   “已修复%” 在所有三个浏览器中都相似，尽管 WebKit 的稍低一些。当我查看未解决的 WebKit bug 时，有 2 个与 iOS 相关，一个在 WebSQL 中（已过时），其余 3 个实际上相当次要。所以我真的不能责怪 WebKit 团队。（而[其中一个次要问题](https://bugs.webkit.org/show_bug.cgi?id=263663)最终出现在 [Interop 2024](https://wpt.fyi/results/?label=master&label=experimental&aligned&view=interop&q=label%3Ainterop-2024-accessibility)，所以可能很快会修复。）

+   对于 3 个未解决的 Firefox 问题，其中 2 个非常古老且重现步骤糟糕（我的错），剩下的一个是与阴影 DOM 相关的[次要 CSS 问题](https://bugzilla.mozilla.org/show_bug.cgi?id=1739682)。

+   对于 4 个未解决的 Chromium 问题，[其中一个](https://issues.chromium.org/issues/40890306) 实际上已经过时（我已经回复了该线程），2 个相当次要，而[剩下的一个](https://issues.chromium.org/issues/40656738) 部分修复（在启用 [BFCache](https://web.dev/articles/bfcache) 时可用）。

+   我很惊讶在 Firefox 上提交的 bug 总数居然不低。我模糊的记忆是我几乎没有在 Firefox 上提交过任何 bug，但实际上，当我这样做时，通常是因为它们遵循了规范，而其他浏览器没有。（我学会了在提交 bug 之前真正三次检查我的工作！）

+   我在 WebKit 上提交的 6 个 bug 是关于 IndexedDB 的，这完全符合我骚扰他们关于 IDB 的 bug 报告的记忆。（相比之下，我在 Chromium 上提交了 3 个 IDB bug，而在 Firefox 上一个都没有。）

+   和预期一样，我在 Chromium 上提交的 5 个问题是由于 ChromeDriver、DevTools 等引起的。

如果你想查看原始数据，可以在下面找到：

<details><summary>Chromium 数据</summary>

| 状态 | ID | 标题 | 日期 |
| --- | --- | --- | --- |
| 已修复 | [41495645](https://crbug.com/41495645) | Chromium 在使用基于 CDP 的堆快照导航多页面站点时泄漏 Document/Window 等 | 2024年2月22日 凌晨01:02 |
| 新问题 | [40890306](https://crbug.com/40890306) | 反映的 ARIA 属性不应将 undefined 设置为 null 处理 | 2024年3月2日 下午05:50 |
| 新问题 | [40885158](https://crbug.com/40885158) | 设置 DocumentFragment 的子元素 outerHTML 抛出错误 | 2024年1月8日 晚上10:52 |
| 已修复 | [40872282](https://crbug.com/40872282) | aria-label 在 a 上不应被计算为可访问名称 | 2024年1月8日 晚上11:24 |
| 重复 | [40229331](https://crbug.com/40229331) | 对多个 s 进行样式计算比一个大 s 要花费更多时间 | 2023年6月20日 上午09:14 |
| 已修复 | [40846966](https://crbug.com/40846966) | customElements.whenDefined() 返回 undefined 而非构造函数 | 2024年1月8日 晚上10:13 |
| 已修复 | [40827056](https://crbug.com/40827056) | ariaInvalid 属性未反映到 aria-invalid 属性 | 2024年1月8日 晚上08:33 |
| 废弃 | [40767620](https://crbug.com/40767620) | 堆快照包括由 DevTools 控制台引用的对象 | Jan 8, 2024 11:53PM |
| 新 | [40766136](https://crbug.com/40766136) | 恢复选择范围导致文本输入忽略按键 | Jan 8, 2024 07:20PM |
| 修复 | [40759641](https://crbug.com/40759641) | 属性选择器的样式计算性能差于类选择器 | May 5, 2021 01:40PM |
| 废弃 | [40149430](https://crbug.com/40149430) | performance.measureMemory() 在无头模式下不允许 | Feb 27, 2023 12:26AM |
| 废弃 | [40704787](https://crbug.com/40704787) | 在 Retainers 图中禁用 WeakMap 键和循环引用的选项 | Jan 8, 2024 05:52PM |
| 修复 | [40693859](https://crbug.com/40693859) | DevTools 正在录制跟踪时，Chrome 崩溃由于 WASM 文件 | Jun 24, 2020 07:20AM |
| 修复 | [40677812](https://crbug.com/40677812) | npm 安装 chrome-devtools-frontend 因预安装脚本失败 | Jan 8, 2024 05:17PM |
| 新 | [40656738](https://crbug.com/40656738) | 返回导航不会将焦点恢复到单击的元素 | Jan 8, 2024 03:26PM |
| 废弃 | [41477958](https://crbug.com/41477958) | 混合器动画在每帧空的 requestAnimationFrame 上主线程重新计算样式 | Aug 29, 2019 04:06AM |
| 重复 | [41476815](https://crbug.com/41476815) | OffscreenCanvas convertToBlob() 比 | Feb 18, 2020 11:51PM 慢 300ms |
| 修复 | [41475186](https://crbug.com/41475186) | 插入和移除 overflow:scroll 元素导致大量样式计算回归 | Aug 26, 2019 01:34AM |
| 废弃 | [41354172](https://crbug.com/41354172) | IntersectionObserver 使用根元素的填充框而不是边框框 | Nov 12, 2018 09:43AM |
| 废弃 | [41329253](https://crbug.com/41329253) | word-wrap:break-word 与奇特 Unicode 字符导致长布局 | Jul 11, 2017 01:50PM |
| 修复 | [41327511](https://crbug.com/41327511) | IntersectionObserver boundingClientRect 的宽度/高度不准确 | Jun 1, 2019 08:28PM |
| 修复 | [41267419](https://crbug.com/41267419) | Chrome 52 在所有作者标头均为 CORS 安全列表时，发送带空 Access-Control-Request-Headers 的 CORS 预检请求 | Mar 18, 2017 02:27PM |
| 修复 | [41204713](https://crbug.com/41204713) | IndexedDB 阻塞 DOM 渲染 | Jan 24, 2018 04:03PM |
| 修复 | [41189720](https://crbug.com/41189720) | chrome://inspect/#devices 闪烁 “待授权” 适用于 Android 设备 | Oct 5, 2015 03:59AM |
| 废弃 | [41154786](https://crbug.com/41154786) | Chrome for iOS：openDatabase 导致 DOM 异常 11 或 18 | Feb 9, 2015 08:30AM |
| 修复 | [41151574](https://crbug.com/41151574) | 空 IndexedDB Blob 在 ajax 获取时导致 404 | Mar 16, 2015 11:37AM |

| 修复 | [40400696](https://crbug.com/40400696) | 存储在 IndexedDB 中的 Blob 导致 FileReader 返回 null 结果 | Feb 9, 2015 10:34AM |</details> <details><summary>Firefox 数据</summary>

| ID | Summary | Resolution | Updated |
| --- | --- | --- | --- |
| [1704551](https://bugzilla.mozilla.org/show_bug.cgi?id=1704551) | 属性选择器的样式计算性能较类选择器差 | FIXE | 2021-09-02 |
| [1861201](https://bugzilla.mozilla.org/show_bug.cgi?id=1861201) | 支持 ariaBrailleLabel 和 ariaBrailleRoleDescription 反射 | FIXE | 2024-02-20 |
| [1762999](https://bugzilla.mozilla.org/show_bug.cgi?id=1762999) | 具有 id 的干预 div 向 NVDA 报告不正确的列表框选项计数 | FIXE | 2023-10-10 |
| [1739154](https://bugzilla.mozilla.org/show_bug.cgi?id=1739154) | 当焦点集中在宿主上时，delegatesFocus 会更改焦点的内部元素 | FIXE | 2022-02-08 |
| [1707116](https://bugzilla.mozilla.org/show_bug.cgi?id=1707116) | 替换 shadow DOM 样式导致计算样式不一致 | FIXE | 2021-05-10 |
| [1853209](https://bugzilla.mozilla.org/show_bug.cgi?id=1853209) | ARIA 反射应将设置为 null/undefined 视为删除属性 | FIXE | 2023-10-20 |
| [1208840](https://bugzilla.mozilla.org/show_bug.cgi?id=1208840) | IndexedDB 阻止 DOM 渲染 | — | 2022-10-11 |
| [1531511](https://bugzilla.mozilla.org/show_bug.cgi?id=1531511) | Service Worker 在“install”阶段的 fetch 请求阻塞了主线程的 fetch 请求 | — | 2022-10-11 |
| [1739682](https://bugzilla.mozilla.org/show_bug.cgi?id=1739682) | 裸 ::part(foo) CSS 选择器选择影子根内部的部件 | — | 2024-02-20 |
| [1331135](https://bugzilla.mozilla.org/show_bug.cgi?id=1331135) | 性能用户定时条目缓冲区限制为 150 | DUPL | 2019-03-13 |
| [1699154](https://bugzilla.mozilla.org/show_bug.cgi?id=1699154) | :focus-visible - 后退导航上的基于 JS 的 focus() 被视为键盘输入 | FIXE | 2021-03-19 |
| [1449770](https://bugzilla.mozilla.org/show_bug.cgi?id=1449770) | position:fixed 内的 position:sticky 在 Firefox for Android 中不进行异步滚动（并在调试版本中在 ActiveScrolledRoot::PickDescendant() 中断言） | WORK | 2023-02-23 |
| [1287221](https://bugzilla.mozilla.org/show_bug.cgi?id=1287221) | WebDriver:Navigate 导致性能.timing 指标下降 | DUPL | 2023-02-09 |
| [1536717](https://bugzilla.mozilla.org/show_bug.cgi?id=1536717) | document.scrollingElement.scrollTop 不正确 | DUPL | 2022-01-10 |
| [1253387](https://bugzilla.mozilla.org/show_bug.cgi?id=1253387) | Safari 不支持在 worker 中使用 IndexedDB | FIXE | 2016-03-17 |
| [1062368](https://bugzilla.mozilla.org/show_bug.cgi?id=1062368) | 即使加载成功，针对 blob URL 的 Ajax 请求返回 .status 为 0 | DUPL | 2014-09-04 |
| [1081668](https://bugzilla.mozilla.org/show_bug.cgi?id=1081668) | Blob URL 返回 xhr.status 为 0 | DUPL | 2015-02-25 |
| [1711057](https://bugzilla.mozilla.org/show_bug.cgi?id=1711057) | :focus-visible 在鼠标点击后对程序化键盘焦点不匹配 | FIXE | 2021-06-09 |

| [1471297](https://bugzilla.mozilla.org/show_bug.cgi?id=1471297) | fetch() 和 importScripts() 不共享HTTP缓存 | WORK | 2021-03-17 |</details> <details><summary>WebKit data</summary>

| ID | Resolution | Summary | Changed |
| --- | --- | --- | --- |
| [225723](https://bugs.webkit.org/show_bug.cgi?id=225723) | — | 恢复选择范围导致文本输入忽略按键 | 2023-02-26 |
| [241704](https://bugs.webkit.org/show_bug.cgi?id=241704) | — | 预解析器在运行内联脚本之前不下载样式表 | 2022-06-23 |
| [263663](https://bugs.webkit.org/show_bug.cgi?id=263663) | — | 支持ariaBrailleLabel和ariaBrailleRoleDescription反射 | 2023-11-01 |
| [260716](https://bugs.webkit.org/show_bug.cgi?id=260716) | FIXE | adoptedStyleSheets（ObservableArray）的长度非可写 | 2023-09-03 |
| [232261](https://bugs.webkit.org/show_bug.cgi?id=232261) | FIXE | :host::part(foo)选择器不选择影子DOM中的元素 | 2021-11-04 |
| [249420](https://bugs.webkit.org/show_bug.cgi?id=249420) | DUPL | :host(.foo, .bar) 应为无效选择器 | 2023-08-07 |
| [249737](https://bugs.webkit.org/show_bug.cgi?id=249737) | FIXE | 在DocumentFragment的子元素上设置outerHTML会抛出错误 | 2023-07-15 |
| [251383](https://bugs.webkit.org/show_bug.cgi?id=251383) | INVA | 反射的ARIA属性不应将设置未定义视为null | 2023-10-25 |
| [137637](https://bugs.webkit.org/show_bug.cgi?id=137637) | — | 空字符导致Web SQL中字符串提前终止 | 2015-04-25 |
| [202655](https://bugs.webkit.org/show_bug.cgi?id=202655) | — | iOS Safari：连续rAF回调的时间戳可能相同 | 2019-10-10 |
| [249943](https://bugs.webkit.org/show_bug.cgi?id=249943) | — | 使用COLR字体时，表情符号水平对齐不正确 | 2023-01-04 |
| [136888](https://bugs.webkit.org/show_bug.cgi?id=136888) | FIXE | IndexedDB onupgradeneeded事件中的oldVersion值不正确 | 2019-07-04 |
| [137034](https://bugs.webkit.org/show_bug.cgi?id=137034) | FIXE | 运行时禁用IDB时完全删除所有IDB属性/构造函数 | 2015-06-08 |
| [149953](https://bugs.webkit.org/show_bug.cgi?id=149953) | FIXE | 现代IDB：WebWorker支持 | 2016-05-11 |
| [151614](https://bugs.webkit.org/show_bug.cgi?id=151614) | FIXE | web worker中location.origin未定义 | 2015-11-30 |
| [156048](https://bugs.webkit.org/show_bug.cgi?id=156048) | FIXE | 重新验证后，有时我们未能从磁盘缓存中删除过时条目，而资源不再可缓存 | 2016-04-05 |
| [137647](https://bugs.webkit.org/show_bug.cgi?id=137647) | FIXE | 使用XHR获取Blob URL时返回空内容类型和内容长度 | 2017-06-07 |
| [137756](https://bugs.webkit.org/show_bug.cgi?id=137756) | INVA | WKWebView：JavaScript加载失败，可能是由于解码错误 | 2014-10-20 |
| [137760](https://bugs.webkit.org/show_bug.cgi?id=137760) | DUPL | WKWebView：openDatabase 导致 DOM 异常 18 | 2016-04-27 |
| [144875](https://bugs.webkit.org/show_bug.cgi?id=144875) | INVA | WKWebView 在应用关闭后不保留 IndexedDB 数据 | 2015-05-28 |
| [149107](https://bugs.webkit.org/show_bug.cgi?id=149107) | FIXE | IndexedDB 对唯一键不会抛出 ConstraintErrors | 2016-03-21 |
| [149205](https://bugs.webkit.org/show_bug.cgi?id=149205) | FIXE | IndexedDB 的 openKeyCursor() 返回主键的顺序错误 | 2016-03-30 |
| [149585](https://bugs.webkit.org/show_bug.cgi?id=149585) | DUPL | 大量使用本地存储可能导致页面冻结 | 2016-12-14 |
| [156125](https://bugs.webkit.org/show_bug.cgi?id=156125) | INVA | 使用查询参数获取 blob URL 导致 404 | 2022-05-31 |

| [169851](https://bugs.webkit.org/show_bug.cgi?id=169851) | FIXE | Safari 在预检请求中发送空的 “Access-Control-Request-Headers” | 2017-03-22 |

## -   结论

-   我认为跨浏览器兼容性在过去几年中有了很大改善。我们有像 [Interop](https://github.com/web-platform-tests/interop) 和 [Web Platform Tests](https://github.com/web-platform-tests/wpt/) 这样的项目，使得浏览器团队更容易找出有问题的地方并确定他们应该优先考虑的事项。

-   所以，如果你还没有开始在浏览器上提交 bug 报告，现在是个绝佳时机！我建议首先在正确的 bug 追踪器中搜索你的问题（[Chromium](https://issues.chromium.org/issues)，[Firefox](https://bugzilla.mozilla.org)，[WebKit](https://bugs.webkit.org)），然后创建一个最小的重现场（CodePen，JSBin，纯 HTML 等），最后尽可能包含尽可能多的细节（浏览器版本，操作系统版本，截图等）。我还建议阅读 [“如何提交良好的浏览器 bug”](https://web.dev/articles/how-to-file-a-good-bug)。

-   开心地寻找 bug！

## -   脚注

-   一些人指出，`rdar://` 链接可以意味着几乎任何东西。我一直以为它意味着 bug 被重定向到某个内部团队，但看来并非如此。
