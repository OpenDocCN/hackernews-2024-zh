<!--yml

类别：未分类

日期：2024-05-27 13:03:41

-->

# 使用 Explore Logs 查找您的日志数据：无需 LogQL！ | Grafana Labs

> 来源：[https://grafana.com/blog/2024/04/09/使用-explore-logs-查找您的日志数据-无需-logql/](https://grafana.com/blog/2024/04/09/使用-explore-logs-查找您的日志数据-无需-logql/)

我们很高兴地宣布预览 Explore Logs，这是一种新的浏览日志的方式，无需编写 LogQL。在本文中，我们将介绍为什么要构建 Explore Logs，并深入探讨其一些功能，包括标签快速概览、检测字段以及我们的新模式检测。最后，我们将告诉您如何今天就试用 Explore Logs。

但让我们从头开始 —— 用老牌的 LogQL。

## 我们热爱 LogQL。

Grafana Loki，Grafana Labs 的 [开源日志聚合项目](https://github.com/grafana/loki)，提供了一个强大的查询语言，称为 [LogQL](/blog/2022/05/12/10-things-you-didnt-know-about-logql/)。站点可靠性工程师（SREs）和其他 Loki 专家喜欢使用它来过滤特定关键字的日志、通过选择特定标签减少噪音，并执行其他操作以了解其系统。

可以从日志创建度量，将其放入仪表板以可视化关键洞察。可以设置警报，并将其连接到 [Grafana 事故响应与管理（IRM）](/products/cloud/irm/)，包括 [Grafana OnCall](/products/cloud/oncall/) 和 [Grafana 事故](/products/cloud/incident/)，以确保在事情出错时及时获得警告。

那些相同的查询可以用于 [Grafana 仪表板可视化](/docs/grafana/latest/panels-visualizations/visualizations/)，这样您就可以理解日志内容。LogQL 真的很强大！但是有一个小问题……

## ‘但我不懂 LogQL！’

如果您不是每天深入研究日志，您可能没有理由去学习 LogQL。也许您偶尔会深入了解，或者可能只在事件发生时才这样做。即便在这些时刻，不得不记住查询语言的细节也会减慢响应时间。

重点是，您来到日志平台不是因为您喜欢日志，而是因为您 *需要* 日志来完成工作。无论您是需要查看部署是否顺利、调查延迟问题，还是在凌晨 4 点处理页面，您最不想做的事情就是再与另一种查询语言纠缠。

如果您能够享受 Loki + Grafana 提供的所有优势，而无需学习 LogQL，那该多好啊。是的，现在您可以做到！

## 推出 Explore Logs，一个新的开源应用程序

Explore Logs 是我们的新开源应用程序，让您可以像事先整理好一样浏览日志。您可以在 [GitHub 上找到源代码](https://github.com/grafana/explore-logs)，但在您跳转之前，让我们给您进行一个快速导览。

当您进入 Explore Logs（在导航中，转到 **探索** > **日志**）时，会显示检测到的服务或应用程序列表。工程师不再需要与组织中的团队争论统一规范。相反，我们接受混乱。

服务显示在其日志量和最近日志行预览的旁边，因此您可以一目了然地看到哪些服务交流最频繁，以及它们发出的日志种类。

没有看到您正在寻找的服务？只需在搜索栏中搜索（纯文本搜索，而非 LogQL）。

就好像自动服务检测还不够，当您选择一个服务时，事情开始变得真正有趣。

## 如何分解服务中的日志

Explore Logs 提供了进一步可视化和按标签、检测到的字段和模式分解服务日志的工具，同时确保日志行本身始终只有一次点击即可。

+   **标签** 是可以附加到日志行的键值对，例如：`level=error`、`environment=prod`、`app=nginx`、`team=loki`

+   **检测到的字段** 包含在查询时从日志行自动提取的结构化键值对。

+   **模式** 是从日志流自动派生的模板，用于匹配相同类型的行。此处的新创新带来了下文描述的一些强大功能。

让我们逐个（get it?）探讨这些内容，并看看 Explore Logs 如何比以往任何时候都更容易使用这些功能。

### 标签

在 Loki 中，[标签](/docs/loki/latest/get-started/labels/) 是用于组织和识别日志流的键值对。标签附加到日志流上，帮助用户高效地查询、过滤和聚合日志。它们类似于其他日志系统中的标签或元数据。

Explore Logs 为每个标签创建了一个日志量图表，让您轻松查看哪些标签活动最频繁。

选择一个标签会显示每个值的日志分解。例如，日志级别分解为调试、信息、警告、错误和严重。

选择一个值会应用过滤器（在后台为您编写 LogQL），并立即跳转到一个经过精心筛选的视图，显示相关的日志行。

### 检测到的字段

Loki 中的检测到的字段指的是在日志消息被摄取时自动提取的结构化键值对。Loki 不会索引日志行的内容，但可以在查询时从日志中解析和提取字段。此功能允许用户通过这些字段更高效地查询日志，而无需将每个可能的属性都标记为标签，这可能既昂贵又低效。

处理检测到的元数据时，Explore Logs 提供了无缝体验。一些字段的选择会自动展示为网格视图，并且您可以从中过滤日志。

对最终用户来说，处理标签和检测字段几乎相同，减少了需要记住哪个是哪个（以及与每个查询语法相关联的认知负担）的认知负担。

### 模式匹配

Loki 提取日志模式，帮助您理解服务生成的不同类型的日志行。过滤后进行模式提取。为了获得最佳结果，我们建议您首先按已知值（如标签、检测字段和时间范围）进行过滤，以缩小搜索空间并使模式更有用。

您可以使用模式匹配来：

1.  理解系统生成的日志行的类型

1.  隐藏嘈杂或无关的行，快速缩小重点

1.  通过过滤与一个或多个模式匹配的日志行，专注于特定趋势或异常

探索日志允许您轻松地向过滤器添加多个模式，这些模式将包括与所选模式中任何一个匹配的日志行。您还可以排除一个模式，以隐藏该类日志行。

#### 模式匹配的示例用例

假设您正在查找大量 HTTP 流量日志中的错误。很难找到您要找的个别行。有了模式，您可以通过点击按钮简单地静音有问题的行来排除这类行。

考虑下面的示例日志行。您可以清楚地看到两种不同的模式：

```
code=200 method=get path=/page1 trace_id=123
code=500 method=post path=/page2 trace_id=124
err: code=500 msg=”db connection error” trace_id=124
code=201 method=post path=/page2 trace_id=125
code=200 method=get path=/page1 trace_id=126
code=500 method=post path=/page2 trace_id=127
err: code=500 msg=”db connection error” trace_id=127
code=200 method=get path=/page2 trace_id=128
code=200 method=get path=/page2 trace_id=129
code=500 method=post path=/page2 trace_id=130
err: code=500 msg=”db connection error” trace_id=130
```

第一个模式描述了 HTTP 请求：

```
code=<_> method=<_> path=/<_> trace_id=<_>
```

第二个模式描述了错误行：

```
err: code=<_> msg=”<_>” trace_id=<_>
```

值被剥离，只留下字符串常量和构成模板或模式的 LogQL 占位符。

> **您知道吗？** 在 [Loki 3.0](https://github.com/grafana/loki) 中，您可以使用 [模式过滤器](/docs/loki/latest/query/#pattern-match-filter-operators) 代替正则表达式，以提高查询速度。

## 探索日志还可以做什么呢？

除了我们已经讨论过的所有内容之外，还有一些其他显著的探索日志功能需要注意：

### 搜索

或许 LogQL 很难，但文本搜索不是。当您已经缩小了搜索范围时，您渲染的日志行的纯文本、区分大小写的搜索可能会派上用场。

### 复制 URL

轻松与队友共享您当前的探索日志上下文，以帮助在团队环境中进行故障排除。

### 在探索中打开

探索中是否有您真正喜欢但“探索日志”中尚不存在的功能？此时使用 LogQL 会非常有帮助吗？轻松跳转到 [探索](/docs/grafana/latest/explore/) —— 我们的 UI，用于探索数百种数据源，包括 Loki —— 同时保留当前的上下文。

## 探索日志的工作原理

我们可能会认为这看起来很酷，而且这与探索的体验非常不同。但它究竟能如何帮助开发人员呢？

好吧，快速解决可观测性问题听起来怎么样？更快地关闭 P1，更快地知道要推送到生产环境的热修复，更快地回到床上。让我们来展示给你看。

假设您需要识别服务中的一个行为不当的 Pod。您可以通过 LogQL 和 Explore 获得相关信息 — 构建一个按服务计算错误次数的 LogQL 指标查询，看看哪个会弹出来。但这是最 *简单* 的方法吗？

不再是！现在您可以：

+   开始选择您的服务

+   选择 `level` 标签选择器，然后将 `level=error` 添加到您的过滤条件中

+   选择 `pod` 标签选择器，然后查看所有受影响的 Pod 及其错误率

+   只需再点击一下，查看与这些 Pod 相关的日志行

只需少数几次点击，在每个步骤中都有视觉提示，而且 — 再来一次 — *无需编写 LogQL*。

## 特别鸣谢

探索日志是共享的同理心、创造力和辛勤工作的结果。我们要感谢迄今为止所有对探索日志做出贡献的人！

## 试一试吧！

今天可预览“探索日志”。您可以在 [探索日志 GitHub 仓库](https://github.com/grafana/explore-logs) 中了解更多信息。您也可以通过从仓库安装并在 [Grafana 11](/blog/2024/04/09/grafana-11-release-all-the-new-features/) 和 [Loki 3.0](/blog/2024/04/09/grafana-loki-3.0-release-all-the-new-features/) 中使用探索日志来试用它，这两者都在 [GrafanaCON 2024](/blog/2024/04/09/grafanacon-2024-a-guide-to-all-the-announcements-from-grafana-labs/) 上宣布。

或者您还可以在 [Grafana Play 环境](https://play.grafana.org/a/grafana-lokiexplore-app/explore?mode=start&patterns=&var-patterns=&logId=&var-filters=) 中体验探索日志。

请告诉我们您希望通过 Explore Logs 中的 **提供反馈** 按钮改进或添加的内容，或者您可以在仓库中更深入地参与！

我们很高兴与我们的社区合作，共同打造最简单的 Loki + Grafana 体验！

*了解有关 [Loki 3.0 的最新功能](/blog/2024/04/09/grafana-loki-3.0-release-all-the-new-features)，我们的开源日志聚合工具。*
