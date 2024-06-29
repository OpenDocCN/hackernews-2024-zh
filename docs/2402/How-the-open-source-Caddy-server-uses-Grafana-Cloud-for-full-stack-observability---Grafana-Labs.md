<!--yml

category: 未分类

date: 2024-05-27 15:03:02

-->

# 开源 Caddy 服务器如何利用 Grafana Cloud 实现全栈可观测性 | Grafana Labs

> 来源：[https://grafana.com/blog/2024/02/21/how-the-open-source-caddy-server-uses-grafana-cloud-for-full-stack-observability/](https://grafana.com/blog/2024/02/21/how-the-open-source-caddy-server-uses-grafana-cloud-for-full-stack-observability/)

*穆罕默德·阿尔·萨哈夫在三星电子沙特阿拉伯公司担任技术产品经理。在日常工作之外，他与 Caddy 团队合作，解决第三千年面临的 Web 服务器问题。穆罕默德是 Kadeessh 的作者，前身是 caddy-ssh，也是多个 Caddy 模块的维护者。当他不在编程时，他会依靠咖啡试图赶上生活和睡眠。你可以在 [caffeinatedwonders.com](http://caffeinatedwonders.com) 找到他的咖啡奇迹。*

作为 [Caddy 服务器](https://caddyserver.com/) 的维护者，我们知道端到端的可观测性的价值，可以加速根本原因分析并减少 MTTR。但与其自己管理可观测性堆栈的所有重复性繁重工作，我的团队更希望尽可能多地专注于我们的核心任务：开发和优化我们的开源项目。

这些是我们最近迁移到 [Grafana Cloud](/docs/grafana-cloud/?pg=blog&plcmt=body-txt) 背后的一些最重要的原因。

在这篇博文中，我将更详细地介绍为什么 Caddy 团队选择了 [Grafana Cloud](https://grafana.com/blog/2024/02/21/how-the-open-source-caddy-server-uses-grafana-cloud-for-full-stack-observability/) 作为可观测性解决方案，以及它如何帮助我们推动 Caddy 项目的前进。

> ### 为您的开源项目获取免费的 Grafana Cloud Pro 账户
> ### 
> 对于任何开源项目来说，全栈可观测性是至关重要的，以促进和提升。这也是为什么像 Grafana Labs 这样一个具有开源软件基因的公司，向像 Caddy 这样的开源项目维护者提供免费的 Grafana Cloud Pro 账户。有兴趣获取您的 Cloud Pro 账户吗？您可以通过 [community@grafana.com](mailto:community@grafana.com) 联系我们。

## 我们选择 Grafana Cloud 的原因

[Caddy](https://caddyserver.com/) 是一个用 Go 编写的可扩展 Web 服务器，被称为第一个普遍可用的 HTTP/2 服务器。Caddy 的旗舰功能包括通过 [ACME 协议](https://caddyserver.com/features#automatic-https) 自动获取和管理 HTTPS/TLS，以及强大的 [OCSP](https://caddyserver.com/#gold-standard) 状态验证。

当Caddy在2015年推出时，用户可以通过[下载页面](https://caddyserver.com/download)创建一个独特的按需Caddy构建，使用自定义的插件集。随着对Caddy的需求增长，这种自定义构建经受住了时间的考验。“构建工作者”是Caddy开发团队称之的，它在2020年Caddy v2正式发布后不久被重写。重写是为了适应新的模块结构和使用[xcaddy](https://github.com/caddyserver/xcaddy)，一个方便的库和工具，在幕后生成自定义Caddy构建流程。新版本一直以来运行良好，几乎没有投诉，直到2023年初。

我们从这时开始听到无响应的下载请求和下载页面无法构建所请求的自定义Caddy构建的问题。我们依赖于我们的云服务提供商的仪表板来监视服务器健康状况，并根据需要手动检查[journald](https://www.freedesktop.org/software/systemd/man/latest/systemd-journald.service.html)以识别和解决问题。云服务提供商的仪表板提供了服务器资源利用情况的高级视图，但未能深入了解运行在服务器上的应用程序。

缺乏统一和用户友好的界面来检查日志也使得在远离键盘时繁琐进行故障排除，与团队成员分享有趣的日志行，以及跟踪服务边界上的问题请求。更糟糕的是，这是一种被动的方式：最终用户需要告诉我们，因此我们无法主动或有效地解决问题，由于缺乏警报。认识到这些成长烦恼，我们知道我们需要一个更成熟的可观察性解决方案来管理我们的基础设施。

因此，在2023年3月，我们联系了Grafana Labs，并了解到他们有一个计划，通过该计划他们向开源项目维护者提供免费的Grafana Cloud Pro账户（在此特别感谢Caddy项目的同事、Grafana Labs的高级软件工程师Dave Henderson，他帮助我们发现了这个计划）。这个Grafana Cloud Pro账户使几个Caddy项目维护者能够监视我们的基础设施，并且在尝试使用由[Linux服务器集成](/docs/grafana-cloud/monitor-infrastructure/integrations/integration-reference/integration-linux-node/?pg=blog&plcmt=body-txt)提供的仪表板后，我们几乎立即发现了价值。

## 使用Grafana Cloud加速根本原因分析

为了开始使用Grafana Cloud，我们按照集成入门步骤，在所有服务器上安装和配置[Grafana Agent](/docs/agent/latest/)。我们知道我们的构建工作者是瓶颈，我们定制了一个仪表板来检查构建工作者节点的各个角度。`node_exporter`对于了解我们的工作负载如何影响其运行的实例至关重要。

我们很快发现由于运行时间较长的构建过程，包括一些较重的模块，导致 [goroutine](https://go.dev/tour/concurrency/1) 泄漏问题。我们在时间序列图中观察到 goroutine 数量在长时间内增长，但从未触及零线：

直到最近，长时间运行的构建并不是问题，我们知道这可能是一个症状。构建请求设置了超时，所以我们知道至少一个触发服务器问题的原因。解决这一症状确保了被卡住的作业不会无限期运行或消耗资源。当我们修复了构建工作者的 goroutine 泄漏问题后，我们发现可能会有波动，但图表线条并不总是向上走：

尽管我们解决了 goroutine 泄漏问题，但我们仍需要找出并解决服务器性能问题的根本原因。问题仍然存在：为什么这些构建如此缓慢？编译过程根据阶段的不同需求 RAM、CPU 和 I/O 资源。我们观察到的无响应报告从未与过度使用 RAM 相关联，但 CPU 利用率一直很高 —— 甚至导致无法及时接受 SSH 登录。这对我们来说是一个关键的认识，我们发现我们的进程是 CPU 限制的，而不是内存限制的。这为我们提供了一个新的行动计划：构建工作者服务器升级应该优化 CPU。

## Caddy 和 Grafana Cloud 未来的发展方向是什么？

尽管我们的计划取得了成功，但我们远未结束。SRE 是一个持续改进的过程。

展望未来，Grafana Cloud 中的可观察性工具将使我们能够有效地诊断和处理问题。日志聚合允许我们关联不同应用程序和系统边界上的各种事件；持续监控使我们能够在活动突然下降和事件周围发生的情况下发出警报；性能分析显示资源利用不足的地方；而合成监控提供了一个外部视角，以黑盒方式观察我们的系统，使用定期的 PING 和/或 TRACEROUTE。我们期待通过 Grafana Cloud 推进我们的可观察性战略 —— 以及 Caddy OSS 项目。

*要了解更多有关您的 OSS 项目的 Grafana Cloud Pro 账户，请通过 [community@grafana.com](mailto:community@grafana.com) 联系我们。*
