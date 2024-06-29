<!--yml

category: 未分类

date: 2024-05-29 12:35:14

-->

# Edge URL 方案 – text/plain

> 来源：[https://textslashplain.com/2022/07/18/edge-url-schemes/](https://textslashplain.com/2022/07/18/edge-url-schemes/)

### `microsoft-edge:` 应用程序协议

Microsoft Edge 实现了一个[应用程序协议](https://textslashplain.com/2019/08/29/web-to-app-communication-app-protocols/)，其方案为 `microsoft-edge:`，旨在启动 Microsoft Edge 并传递 Web 方案的 URL 和/或附加参数。一个基本的调用可能只需简单地：

`microsoft-edge:http://example.com/`

然而，正如我选择写的事物经常的情况一样，可能存在一些隐藏的复杂性，这些可能并不是显而易见的。

#### 非公开

该 URL 方案的目的是使 Windows 和协作应用程序能够调用 Edge 浏览器中的特定用户体验。

这个方案并不被视为“公开” ——没有该方案的官方文档，并且 Edge 团队对其行为不作任何保证。我们可以（并且已经）根据需要添加或修改功能，以实现所需的行为。

在过去几年中，我们为该方案增加了各种功能，包括调用 UX 功能、启动特定用户配置文件，以及实现其他集成方案。例如，Windows 可能会宣传 Edge Surf 游戏，如果用户选择播放，则通过 Shell 执行以下 URL 启动游戏 `microsoft-edge:?ux=surf_game`。

由于此 URL 方案的非公开性和固有的不稳定性（不向后兼容），它不是一个*可扩展性点*，不支持配置处理程序为非 Microsoft Edge。

*底层实现：处理此方案的方法可以在 Edge 的非公开版本的[StartupBrowserCreator::ProcessCmdLineImpl](https://source.chromium.org/chromium/chromium/src/+/main:chrome/browser/ui/startup/startup_browser_creator.cc;l=911;drc=97f94c6631e327c2c1a9891774b44fbba9e8e3bf)功能中找到，我最近在关于[Chromium 启动](https://textslashplain.com/2022/06/15/chromium-startup/)的文章中提到过此部分。*

#### 繁琐之处

对 `microsoft-edge` 方案的一个（也许令人惊讶的）限制是它不能从 Edge 内部启动。如果用户在 Edge 内部点击 `microsoft-edge:` 方案的链接，则不会发生任何可见的操作。只有当他们打开 F12 控制台时，才会看到错误消息：

Edge浏览器本身会阻止`microsoft-edge`协议，以避免“*导航清洗*”问题，即通过外部处理程序路径进行导航可能导致上下文丢失。导航上下文的丢失可能会在安全性和滥用方面引入漏洞。例如，当Android Chrome未能阻止该协议的Chrome版本时，Android上存在弹出窗口拦截器绕过漏洞。同样，对于`SameSite=Strict`Cookie的CSRF防护限制可以被规避，因为这些Cookie在“跨站点”导航时会被故意抑制，但在“外部”导航中不会被抑制。

Edge WebView2控件还会阻止导航到`microsoft-edge`协议，尽管我认为希望允许这种导航的应用程序可能可以通过适当的事件处理程序实现。

另一个棘手的问题是，用户可能安装了多个不同的Edge通道（稳定版、Beta版、开发版、Canary版），但`microsoft-edge`协议只能由其中一个来声明。如果用户选择了不同的通道来处理`HTTPS:`和`microsoft-edge`链接，这可能会令人困惑：

…因为一些链接将在Edge Canary中打开，而其他链接将在Edge Beta中打开。

* * *

### `edge:`内置方案

除了上述的应用程序协议之外，Microsoft Edge还支持一个名为[edge:](https://textslashplain.com/2022/01/21/adding-protocol-schemes-to-chromium/)的[内置方案](https://textslashplain.com/2022/01/21/adding-protocol-schemes-to-chromium/)。与`microsoft-edge:`应用程序协议相比，此方案仅在浏览器内部可用。您不能在Windows的其他地方调用`edge:` URL，也不能将其作为命令行参数传递给Edge。

`edge:`方案只是Chromium中用于支持内部页面的`chrome:`和`about:`方案的别名，比如`about:flags`，`about:settings`等（请查看`edge:about`以获取列表）。

出于安全考虑，常规网页无法导航到或加载`edge/chrome`方案中的子资源。多年前，常见的攻击模式是导航到`chrome:downloads`，然后滥用其特权的WebUI绑定来逃离浏览器沙盒。还有一些特殊的调试链接，比如`about:inducebrowsercrashforrealz`，它们会如字面所述地执行。
