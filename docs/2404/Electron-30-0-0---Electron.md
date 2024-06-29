<!--yml

category: 未分类

日期：2024-05-27 13:28:49

-->

# 电子 30.0.0 | Electron

> 来源：[https://www.electronjs.org/blog/electron-30-0](https://www.electronjs.org/blog/electron-30-0)

电子 30.0.0 已发布！它包括对 Chromium `124.0.6367.49`、V8 `12.4` 和 Node.js `20.11.1` 的升级。

* * *

电子团队很高兴宣布发布电子 30.0.0！您可以通过 npm 安装它，命令为 `npm install electron@latest`，或从我们的[发布网站](https://releases.electronjs.org/releases/stable)下载。继续阅读以了解有关此版本的详细信息。

如果您有任何反馈，请在[Twitter](https://twitter.com/electronjs)或[Mastodon](https://social.lfx.dev/@electronjs)上分享，或加入我们的社区[Discord](https://discord.com/invite/electronjs)！Bug 和功能请求可以在 Electron 的[问题跟踪器](https://github.com/electron/electron/issues)中报告。

## 显著变更[​](#notable-changes "Direct link to Notable Changes")

### 亮点[​](#highlights "Direct link to Highlights")

+   现在在 Windows 上支持 ASAR 完整性融合。([#40504](https://github.com/electron/electron/pull/40504))

    +   如果未正确配置已启用 ASAR 完整性的现有应用，可能无法在 Windows 上运行。使用 Electron 打包工具的应用应升级到`@electron/packager@18.3.1`或`@electron/forge@7.4.0`。

    +   查看我们的[ASAR 完整性教程](https://www.electronjs.org/docs/latest/tutorial/asar-integrity)获取更多信息。

+   添加了[`WebContentsView`](https://www.electronjs.org/docs/latest/api/web-contents-view)和[`BaseWindow`](https://www.electronjs.org/docs/latest/api/base-window)主进程模块，弃用和替换`BrowserView` ([#35658](https://github.com/electron/electron/pull/35658))

    +   `BrowserView`现在是`WebContentsView`的一个包装，旧实现已被移除。

    +   查看我们的[Web 嵌入文档](https://www.electronjs.org/docs/latest/tutorial/web-embeds)比较新的`WebContentsView` API 与其他类似的 API。

+   实现了对[文件系统 API](https://developer.mozilla.org/zh-CN/docs/Web/API/File_System_API)的支持 ([#41827](https://github.com/electron/electron/commit/cf1087badd437906f280373decb923733a8523e6))

### 堆栈更改[​](#stack-changes "Direct link to Stack Changes")

+   Chromium `124.0.6367.49`

+   Node `20.11.1`

+   V8 `12.4`

电子 30 将 Chromium 从`122.0.6261.39`升级到`124.0.6367.49`，Node 从`20.9.0`升级到`20.11.1`，V8 从`12.2`升级到`12.4`。

### 新功能[​](#new-features "Direct link to New Features")

+   向 webviews 添加了一个`transparent` webpreference。([#40301](https://github.com/electron/electron/pull/40301))

+   在 webContents API 上添加了一个新的实例属性`navigationHistory`，并提供`navigationHistory.getEntryAtIndex`方法，使应用能够检索浏览历史中任何导航条目的 URL 和标题。([#41662](https://github.com/electron/electron/pull/41662))

+   添加了新的 `BrowserWindow.isOccluded()` 方法，允许应用程序检查遮挡状态。 ([#38982](https://github.com/electron/electron/pull/38982))

+   增加了在实用进程中使用 `net` 模块发出请求的代理配置支持。 ([#41417](https://github.com/electron/electron/pull/41417))

+   增加了对通过服务类 ID 请求蓝牙端口的支持，该功能通过 `navigator.serial` 实现。 ([#41734](https://github.com/electron/electron/pull/41734))

+   增加了对 Node.js [`NODE_EXTRA_CA_CERTS`](https://nodejs.org/api/cli.html#node_extra_ca_certsfile) CLI 标志的支持。 ([#41822](https://github.com/electron/electron/pull/41822))

### 破坏性变更[​](#breaking-changes "破坏性变更的直接链接")

#### 行为变更：跨源 iframe 现在使用权限策略来访问功能[​](#behavior-changed-cross-origin-iframes-now-use-permission-policy-to-access-features "跨源 iframe 现在使用权限策略来访问功能的直接链接")

跨源 iframe 现在必须通过 `allow` 属性指定给定 `iframe` 可用的功能，以访问它们。

查看更多信息，请参考[文档](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#allow)。

#### 移除：`--disable-color-correct-rendering` 命令行开关[​](#removed-the---disable-color-correct-rendering-command-line-switch "查看移除的 --disable-color-correct-rendering 命令行开关的直接链接")

此开关从未正式记录在案，但其移除在此被记录。Chromium 本身现在对颜色空间有更好的支持，因此不应需要此标志。

#### 行为变更：在 macOS 上，`BrowserView.setAutoResize` 的行为[​](#behavior-changed-browserviewsetautoresize-behavior-on-macos "在 macOS 上，BrowserView.setAutoResize 的行为的直接链接")

在 Electron 30 中，BrowserView 现在是围绕新的 [WebContentsView](https://www.electronjs.org/docs/latest/api/web-contents-view) API 的包装器。

以前，`BrowserView` API 的 `setAutoResize` 函数在 macOS 上由 [autoresizing](https://developer.apple.com/documentation/appkit/nsview/1483281-autoresizingmask?language=objc) 支持，在 Windows 和 Linux 上由自定义算法支持。对于简单的用例，例如使 BrowserView 填充整个窗口，这两种方法的行为是相同的。但在更复杂的情况下，macOS 和其他平台上的 BrowserViews 的自动调整大小行为可能不同，因为 Windows 和 Linux 的自定义调整大小算法与 macOS 的自动调整大小 API 的行为并不完全匹配。现在，自动调整大小行为在所有平台上都是标准化的。

如果您的应用程序使用 `BrowserView.setAutoResize` 进行比使 BrowserView 填充整个窗口更复杂的操作，那么您可能已经在处理此差异的 macOS 行为时具有自定义逻辑。如果是这样，在 Electron 30 中，由于自动调整大小行为是一致的，您将不再需要该逻辑。

`WebContents`的`context-menu`事件中的`params`对象的`inputFormType`属性已移除。请改用新的`formControlType`属性。

#### 移除：`process.getIOCounters()`[​](#removed-processgetiocounters "直达链接至removed-processgetiocounters")

Chromium已移除对这些信息的访问。

## 27.x.y 支持结束[​](#end-of-support-for-27xy "直达链接至27.x.y支持结束")

根据项目的[支持政策](https://www.electronjs.org/docs/latest/tutorial/electron-timelines#version-support-policy)，Electron 27.x.y已经达到支持结束状态。开发者和应用程序被鼓励升级到更新的Electron版本。

| E30 (Apr'24) | E31 (Jun'24) | E32 (Aug'24) |
| --- | --- | --- |
| 30.x.y | 31.x.y | 32.x.y |
| 29.x.y | 30.x.y | 31.x.y |
| 28.x.y | 29.x.y | 30.x.y |

## 下一步[​](#whats-next "直达链接至下一步")

短期内，您可以期待团队继续专注于跟进Electron的主要组件（包括Chromium、Node和V8）的开发。

你可以在[Electron的公共时间轴页面](https://www.electronjs.org/docs/latest/tutorial/electron-timelines)找到更多信息。

有关未来变更的更多信息，请参阅[计划的重大变更](https://github.com/electron/electron/blob/main/docs/breaking-changes.md)页面。
