<!--yml

category: 未分类

date: 2024-05-27 14:29:46

-->

# Windows 10用户在微软更新后报告应用程序问题 • The Register

> 来源：[https://www.theregister.com/2024/02/01/windows_10_users_errors_hardware/](https://www.theregister.com/2024/02/01/windows_10_users_errors_hardware/)

旧的Windows 10硬件正在努力打开一些最近更新的微软应用程序，这给那些在不支持的硬件上运行Windows 11的人们提供了一个他们潜在未来的一瞥。

微软定期通过其商店更新Windows的内置应用程序。它不必等待操作系统的发布，可以在准备好时推出修复和功能。例如，日历、照片和计算器在一月份刚刚进行了更新，但这一更新却导致许多Windows 10用户遇到了问题。

然后在公司的支持论坛上开启了[一篇漫长的帖子](https://answers.microsoft.com/en-us/windows/forum/all/microsoft-photos-file-system-error-2147219196/3bf0b7d0-4390-46f2-b948-a1e05d726e73?page=1)，用户们发现之前能够正常工作的应用程序突然出现了故障。用户报告称，他们尝试从资源管理器打开照片时遇到了错误，计算器等应用也出现了问题。即使重新安装Windows 10也没有带来缓解。

自然而然，用户会通过反馈中心报告问题，但是，哦天呐，这对他们来说也出了问题。

在Windows本身没有更新的情况下，缺乏微软的官方评论，受影响的用户们进行了一些侦探工作，试图弄清楚他们之前运行良好的实例发生了什么。

答案似乎在于使用老旧硬件。在一些报告中出现了Intel Core 2 Duo和Quad处理器，以及一些AMD Athlon芯片。Core 2系列首次出现在2006年，虽然不在Windows 10的[官方支持处理器列表](https://learn.microsoft.com/en-us/windows-hardware/design/minimum/supported/windows-10-22h2-supported-intel-processors)中，但这款处理器足够应对许多生产任务，直到现在。

一位*Register*的读者联系我们，以突出这一情况：“一个普遍的理论是，故障组件使用了一些Core 2不支持的指令扩展，比如SSE 4.2。”

尽管一些用户将此视为微软迫使客户购买新硬件的证据，但看起来更可能是由于疏忽而非阴谋。我们的读者指出：“我相信微软的某个开发人员在构建共享组件（某些证据指向Visual C++运行时）时错误地设置了编译器开关。”

尽管微软以保持向后兼容性而闻名，*The Register*质疑该公司对不支持的硬件进行回归测试的程度。一个愤世嫉俗者可能会说，在支持的硬件上，微软在质量方面已经有足够的困难了。

然而，由于Windows自动更新的特性，长期避开更新是有问题的，如果需要修复漏洞，可能会带来潜在的危险。

目前Windows 10用户面临的问题，也应让那些设法在不支持的硬件上运行最新版微软操作系统的Windows 11粉丝们深思。如果公司无意中从过时芯片下撤出Windows 10支持，同样的事情很容易发生在他们身上。

*The Register*联系了微软以获取其对此情况的看法。如果Windows供应商回应，我们将更新此报道。®
