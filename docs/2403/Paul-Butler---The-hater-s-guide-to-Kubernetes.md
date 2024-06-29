<!--yml

category: 未分类

date: 2024-05-27 14:33:59

-->

# Paul Butler – Kubernetes 的仇恨者指南

> 来源：[https://paulbutler.org/2024/the-haters-guide-to-kubernetes/](https://paulbutler.org/2024/the-haters-guide-to-kubernetes/)

在某些技术圈内，Kubernetes 因为过度复杂和浪费时间而获得了声誉，创业公司应该避免使用。使用 Kubernetes 和小团队被视为过度工程化的标志。

我也曾经对自己进行[尖锐的挑剔](https://twitter.com/paulgb/status/1568257167882436608)。

尽管我的讽刺，“伟大的技术成就”确实是真诚的；在那篇文章发布时，我刚写过关于 Kubernetes 复杂性的文章[最近写的](https://driftingin.space/posts/complexity-kubernetes)，讲述了它的许多复杂性是必要的。

我们在[Jamsocket](https://jamsocket.com/)的生产环境中运行 Kubernetes 已经几年了，我发现了与之相处的良好方式。我们内部已经实现了 Kubernetes 的平静。其中一个重要因素是[切出一小块](https://twitter.com/paulgb/status/1743361919535260053) Kubernetes 功能，假装其他部分不存在。

这篇文章最初是作为我们使用 Kubernetes 的内部指南开始的，因此并不意味着适用于每个创业公司；尽管如此，我认为它是避开 Kubernetes 浩瀚海洋中许多陷阱的一个很好的起点。

## **为什么要使用 Kubernetes？**

在我看来，如果你想要这三样东西，Kubernetes 是最佳选择。

1.  用于运行多个进程/服务器/定时作业。

1.  为了使它们具备冗余性并在它们之间进行负载均衡。

1.  为了将它们配置为代码及其之间的关系。

最基本的来说，Kubernetes 只是一个抽象层，让你可以把一组计算机想象成一个（无头）计算机。如果这符合你的使用情况，并且你可以避开其他部分，你可以走得很远。

有些人告诉我，#2 是杀鸡用牛刀，创业公司不应专注于零停机部署或高可用性。但我们通常每天进行多次部署，当我们的产品出问题时，我们客户的产品也会出现问题，他们的用户会受到影响。哪怕是一分钟的停机时间也会被某人注意到。滚动部署使我们能够毫不庄重地频繁部署。

## 我们如何使用 Kubernetes

作为背景，[Jamsocket](https://jamsocket.com) 是一个为动态生成 Web 应用程序可以通信的进程的服务。有点像 AWS Lambda，但进程生命周期绑定到 WebSocket 连接而不是单个请求/响应。

我们使用 Kubernetes 运行需要支持的长时间运行的进程。API 服务器、容器注册表、控制器、日志收集器、一些 DNS 服务、指标收集等等。

一些我们**不**使用 Kubernetes 的事情：

+   这些短暂的进程本身。我们在很早的时候确实做过，但很快发现它限制了我们（稍后详述）。

+   静态/营销网站。我们使用 [Vercel](https://vercel.com/)。虽然它更昂贵，但在小型初创公司中，一小时工程时间的机会成本也是如此，而 Vercel 能为我们节省的时间比其成本更多。

+   任何直接存储数据的内容，我们会因丢失而感到遗憾。我们确实使用一些持久卷来缓存或派生数据，但除此之外，我们在集群外使用托管的 PostgreSQL 数据库和 Blob 存储。

还值得一提的是，我们并不自行管理 Kubernetes — 使用 Kubernetes 的主要优势在于我们可以外包基础设施级的运维工作！我们对 Google Kubernetes Engine 感到满意，虽然 [Google Domains 的问题](https://blog.pragmaticengineer.com/google-domains-to-shut-down/) 使我对 Google Cloud 的信心动摇，但我至少可以安心地知道迁移到 Amazon EKS 相对来说是相对简单的。

## 我们乐意使用的内容

这里有几种我们毫不犹豫使用的 Kubernetes 资源类型。我只在这里列出我们明确创建的资源；大多数这些资源隐含地创建其他资源（如 Pod），这些资源我们当然也（间接地）使用。

+   **[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)**：我们大部分的 Pod 都是通过部署创建的。每个对我们服务功能至关重要的部署都有多个副本和滚动更新。

+   **[服务](https://kubernetes.io/docs/concepts/services-networking/service/)**：特别是内部服务的 `ClusterIP` 和外部服务的 `LoadBalancer`。我们避免使用 `NodePort` 和 `ExternalName` 服务，更倾向于将我们的 DNS 配置放在 Kubernetes 外部。

+   [**CronJobs**](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)：用于清理脚本和类似操作。

+   **ConfigMaps** 和 **Secrets**：用于向上述资源传递数据。

## 我们谨慎使用的内容

+   **StatefulSet** 和 **PersistentVolumeClaim**：我们使用了一些 StatefulSets。配置比部署复杂一些，但它们可以在重启时保持持久卷。我们更喜欢将重要数据持久化在 Kubernetes 外的托管服务中。我们没有针对卷设定严格规则，因为有时持久化缓存跨服务重启很有用，但我尽可能避免使用它们，因为它们可能与滚动部署产生不良互动（死锁）。

+   **RBAC**：我们在几个地方使用了它，例如给服务权限来刷新一个密钥。它给我们较小的集群增加了足够的复杂性，所以我大多数情况下都会避免使用它。

## 我们积极避免的内容

+   **手写 YAML**。YAML 存在足够多的坑，以至于我尽量避免使用它。相反，我们的 Kubernetes 资源定义是通过 TypeScript 和 [Pulumi](https://www.pulumi.com/) 创建的。

+   **非内置资源和操作符**。 我之前写过关于[Kubernetes复杂性](https://driftingin.space/posts/complexity-kubernetes)的文章，控制循环模式是个双刃剑：它是使K8s强大的核心思想，但也是间接性和复杂性的源头。 [操作符模式](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)和[自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)允许第三方软件利用Kubernetes强大的基础设施进行自己的控制循环，理论上是个好主意，但我发现在实践中有些笨拙。 我们不使用[cert-manager](https://cert-manager.io/)，而是使用[Caddy](https://caddyserver.com/)的证书自动化。

+   **Helm**。 Helm不适合我，因为操作符和没有YAML规则，但我也认为使用非结构化字符串模板来生成机器可解析的东西，意味着引入脆弱性却无任何收益。 [`nindent`](https://v2.helm.sh/docs/charts_tips_and_tricks/#using-the-include-function)对我来说就像是在黑板上挠指甲一样让人难受，抱歉。

+   **任何名字中带有“mesh”的东西。** 我想它们对某些人可能有用，但对我来说不是，也对[这位先生](https://matduggan.com/k8s-service-meshes/)也不是。

+   **Ingress资源**。 我对这些没有任何“战斗伤痕”，我知道有些人能够有效地使用它们，但我们成功使用Kubernetes的主题是避免增加不必要的间接层。 配置Caddy对我们来说很合适，所以我们就这么做了。

+   **试图在本地复制整个k8s堆栈**。 我们不使用k3s或kind之类的工具来精确复制生产环境，而是仅使用Docker Compose或我们自己的脚本，在需要时启动我们实际关心的系统子集。

## 人类永远不应该等待Pod

上面我提到，我们曾在Kubernetes上短暂运行过临时的交互式会话生命周期进程。 我们很快意识到，Kubernetes设计的重点是容错性和模块化，而不是容器启动时间。 总的来说，我认为Kubernetes适合于想要冗余运行一些长时间运行的进程的场景，但如果人类等待Pod启动，则[Kubernetes是错误的选择](https://twitter.com/paulgb/status/1684718880353042432)。

我得承认我在这里表达了自己的观点，但至少这是一本开源的书：我们使用一个名为[Plane](https://plane.dev/)的MIT许可的Rust编排器，专门设计用于快速调度和运行人机交互工作负载的进程。

## 更高级的抽象

对于完整性，我还应该提到，一些出现的 Kubernetes 替代方案也相当不错。特别是如果您不需要或不想要我最初列表中的第三项要求（能够指定基础设施为代码）。对于[我们的一个产品](https://y-sweet.cloud/)，我们选择使用[Railway](https://railway.app/)而不是我们的 k8s 集群，主要是为了预览环境。我非常尊重的一些朋友非常推崇[Render](https://render.com/)（我尝试过，但个人认为Railway的环境模型更清晰）。我也偏爱[Flight Control](https://www.flightcontrol.dev/)的自带云方法。

对于许多SaaS类型的应用程序，您可能会在这些上取得相当大的进展。但是，如果您满足本文开头列出的三个需求，并且采取有纪律的方法对待它，那么不要让任何人告诉您，您对 Kubernetes 来说还为时过早。
