<!--yml

category: 未分类

date: 2024-05-27 14:51:51

-->

# Meta披露他们的无服务器平台每天处理数万亿个函数调用

> 来源：[https://read.engineerscodex.com/p/meta-xfaas-serverless-functions-explained](https://read.engineerscodex.com/p/meta-xfaas-serverless-functions-explained)

*Engineer’s Codex是一封解释实际软件工程的新闻简报。*

* * *

Meta的[XFaaS](https://dl.acm.org/doi/abs/10.1145/3600006.3613155)是他们的无服务器平台，“在数十个数据中心地区的10万台服务器上每天处理数万亿次函数调用。”

XFaaS是Meta内部版本的公共Function-as-a-Service（FaaS）选项，例如AWS Lambda、Google Cloud Functions和Azure Functions。

我很幸运能够提前阅读他们写的[论文](https://drive.google.com/file/d/1FXejqjUgBfW_OSy3Q6FfwgnHEoV4FZPC/view)。

> *计算机科学家在命名方面并不是最有创意的，因为今年有两篇论文介绍了“XFaaS”。一篇是Meta发表的，另一篇是IISC班加罗尔的科学家发表的。*
> 
> *两者完全是不同的系统，所以如果你谷歌“XFaaS”，请知道这篇论文是在谈论Meta的系统版本。*

在这篇文章中，我从高层次的摘要和教训开始，针对那些想要快速了解的人。 *(预计阅读时间：3分钟)*

然后，我会对那些想要了解XFaaS背后架构的人进行更详细的论文解读。 *(预计阅读时间：5分钟)*

+   这篇论文的一个关键点是硬件使用可以通过软件优化以提高无服务器性能。

+   Meta意识到了无服务器函数的启动开销浪费，并旨在模仿**通用工作人员，即任何工作人员都可以立即执行任何函数而无需启动开销。**

+   XFaaS **仅用于非用户界面函数**。无服务器函数的可变延迟太大，无法一致地用于用户界面函数。

+   XFaaS的客户以高度峰值的方式提交函数调用。 **高峰需求比低峰需求高4.3倍。**

    +   他们展示的负载示例之一是**在15分钟内向XFaaS提交了2000万次函数调用。**

    +   Meta发现即使是他们的高峰函数也有模式，他们利用这些模式使高峰函数在其工作负载上更加可预测。

**XFaaS的日均CPU利用率达到了66%，** **比行业平均水平要好得多。**

它通过时间（延迟函数）和空间（将其发送到负载较轻的数据中心）有效地**扩散负载。**

> *Meta正在继续将他们的许多功能过渡到非高峰时段进行调度，这样负载和成本更可预测。*

由于这是内部云，Meta能够执行许多独特的优化，例如在同一进程中运行来自不同用户的多个函数。

大多数函数在不到一秒的时间内执行，但并非所有函数都是如此。

* * *

> 友情插播：**[SWE Quiz](https://swequiz.com/?utm_source=codex) 是涵盖数据库、身份验证、缓存等 450+ 软件工程和系统设计问题的汇编。**
> 
> **它们由来自 Google、Meta、Apple 等公司的工程师创建。**
> 
> 它帮助了我和许多同行（包括我自己）在面试中通过了“软件琐碎问题”，并在工作中感到更自信。

[查看 SWE Quiz](https://swequiz.com)

* * *

**问题：冷启动时间过长。**

+   如果一个容器关闭得太早，整个容器必须为下一次调用重新初始化。

+   如果一个容器关闭得太晚，它就会闲置，浪费宝贵的计算资源。

+   **解决方案：XFaaS 使用即时编译等方法来近似每个工作者都可以立即执行任何函数。**

**问题：负载的高方差。**

+   导致额外的硬件成本，因为过度配置或系统速度较慢时的情况。

+   **解决方案：XFaaS 推迟运行延迟容忍函数，直到非高峰时间，并在数据中心区域全球范围内分发函数调用。**

**问题：过载下游服务。**

+   例子：曾经，来自非用户界面功能的呼叫激增导致用户在线服务的中断。

+   **解决方案：XFaaS 使用类似于 TCP 拥塞控制的机制来调节函数的执行。**

XFaaS 的技术可能有助于公共云，包括：

+   允许调用者指定函数执行开始时间。

+   允许功能所有者设置关于完成期限的服务水平目标（SLOs）。 （低 SLOs 可以延迟到更合适的时间。）

+   允许功能所有者为函数分配关键性级别。

虽然公共云可能不会像 XFaaS 一样在同一个进程中运行来自不同用户的函数，但大型云客户可以在其虚拟专用云中采用 XFaaS 方法。

少数 Meta 团队消耗了 XFaaS 容量的大部分。 类似的大型客户在公共云中也可能受益于 XFaaS 的策略。

> *这是 XFaaS 详细概述的开始。预计阅读时间：5 分钟*

如前所述，XFaaS 处理非常尖锐的负载。 在需求高峰时，只有一个功能就可以每分钟接收 130 万个呼叫。

一个关键的见解是**大多数 XFaaS 功能是由自动化工作流触发的，这些工作流对延迟可以接受。** 再次，这使得 XFaaS 能够在时间（通过延迟功能）和空间（通过将其发送到负载较轻的数据中心）上平衡负载。

XFaaS 支持 PHP、Python、Erlang、Haskell 的运行时，以及用于任何语言的通用基于容器的运行时。

一个函数有几个开发人员可以设置的属性，如函数名、参数、运行时、关键性、执行开始时间、执行完成期限、资源配额、并发限制和重试策略。 执行完成期限可以从几秒钟到 24 小时不等。

**最后，XFaaS 在不同区域具有不同的硬件容量，因此负载均衡必须考虑这一点。**

Meta 在 XFaaS 上有三种工作负载类型：**队列触发函数**、**事件触发函数**（来自数据仓库和数据流系统中的数据更改事件）、以及**定时触发函数**（在预设时间自动触发）。

XFaaS 用于非用户界面的功能，例如异步推荐系统、日志记录、生产力机器人、通知等。

> 客户端通过向提交者发送请求来发起函数调用，在将其传递给 QueueLB（队列负载均衡器）之前应用速率限制。
> 
> QueueLB 将函数调用指向 DurableQ 进行存储，直到完成。
> 
> 定期，调度器将这些调用移动到其 FuncBuffer（内存中的函数缓冲区），按重要性、截止日期和配额进行排序。
> 
> 就绪执行的调用会移动到 RunQ 中，然后 WorkerLB（工作负载均衡器）将其分派给适当的工作者。

提交者、QueueLB、调度器和 WorkerLB 都是无状态的、非分片的，并且没有指定的领导者，因此它们的副本发挥相同的作用。

DurableQ 是有状态的，并且拥有分片式、高可用的数据库。

通过仅添加更多副本，扩展无状态组件变得很容易。如果无状态组件失败，它的工作可以被同级接管。

中央控制器与函数执行路径分开。它们通过更新关键配置参数来**持续优化系统**，这些参数由关键路径组件（如工作者和调度器）使用。

这些配置被缓存在组件本身中，因此如果中央控制器失败，系统仍然正常运行（但无法重新配置）。

这使得中央控制器可以停机数十分钟。这是 Meta 如何将弹性构建到该系统中的一个例子。

需要强隔离以确保安全性或性能的函数被分配到不同的命名空间，每个命名空间使用不同的工作者池实现物理隔离。

由于私有云中的信任、强制同行审查和现有的安全措施，多个功能可以在单个 Linux 进程中运行。数据只能从较低的分类级别流向较高的分类级别。

客户端通过向提交者发送调用来发送调用。提交者通过批处理调用并将其作为一个操作写入 DurableQ 来提高效率。

提交者通过分布式键值存储管理大型参数存储，并具有内置的速率限制策略。为了管理客户端提交速率的变化，各区域都有两个提交者集用于常规和高频客户端。

XFaaS 积极监控高频客户端，需要人工干预来进行限流或服务水平目标（SLO）调整。

提交者然后将函数调用发送到QueueLB。[Configerator](https://research.facebook.com/publications/holistic-configuration-management-at-facebook/)（配置管理系统中央控制器）为QueueLB提供了路由策略，以在不同区域之间平衡负载，因为DurableQ的硬件容量各不相同。

使用UUID对DurableQ的调用进行分片，以实现均匀分布。每个DurableQ通过调用方设置的计划执行时间对函数调用进行分类和存储。

调度器不断查询存储在DurableQ中到期的函数调用。当DurableQ将函数调用交给调度器时，除非执行失败，否则该函数调用将专属于该调度器。

调度器与DurableQ进行通信：

+   **发送成功执行的ACK消息。**然后，函数调用将从DurableQ中永久删除。

+   **对于不成功的尝试，发送NACK。**函数调用再次在DurableQ中可用，供另一个调度器处理。

调度器的主要作用是根据其重要性、截止日期和容量配额对函数调用进行优先级排序。

它从DurableQ中提取函数调用，将它们合并到FuncBuffers中（按重要性和执行截止日期排序），然后将它们安排在RunQ中进行执行。FuncBuffers和RunQ都是内存数据结构。

为了管理有效执行并防止系统滞后，RunQ控制函数调用的流量。该系统还确保负载平衡，全局流量调度器（中央控制器）根据需求和容量指导函数调用流量跨区域流动，优化整体工作器利用率。

调度器中的RunQ将函数发送给WorkerLB（工作负载均衡器），WorkerLB将函数发送到工作池中的工作器中运行。

在XFaaS系统中，使用相同编程语言的函数具有隔离性，具有专用运行时和工作器池。

**该系统的设计旨在允许任何工作器立即执行函数，而无需初始延迟。**XFaaS保持始终活动的运行时，并在本地SSD上更新函数代码。

XFaaS引入了合作式JIT编译，周期性地将函数代码打包并推送到工作器。由于代码经常更新，这是必要的。

JIT编译有三个执行阶段：

+   少数工作器测试新代码。

+   2%的工作器进一步测试代码；一些进行JIT编译的性能分析。

+   JIT编译是在接收函数调用之前完成的，从而消除了延迟。

有了合作式JIT编译，与没有JIT编译相比，工作器在函数代码更新后仅在3分钟内达到了每秒最大请求量，而不是在21分钟内。

由于内存限制，存储每个函数的即时编译（JIT）代码是不可行的。

本地优化器（中央控制器）将函数和工作器分成组，确保内存消耗大的函数得到分发。

使用局部组可将内存消耗降低 11-12%，并且允许工作者高效、一致地利用内存。

+   **函数配额**：每个函数都有一个由其所有者设定的配额，该配额定义了其每秒的 CPU 循环次数。该配额被转换为每秒请求数（RPS）速率限制。中央速率限制器根据函数的 RPS 限制确定是否限制函数调用。

+   **函数的动态 RPS 限制**：为了防止给下游服务带来过大负担，XFaaS 利用类似 TCP 的[AIMD（加性增加，乘性减少）方法](https://en.wikipedia.org/wiki/Additive_increase/multiplicative_decrease#:~:text=The%20additive%2Dincrease%2Fmultiplicative%2D,reduction%20when%20congestion%20is%20detected.)动态调整 RPS 限制。它可以使用慢启动方法设置并管理并发级别和 RPS 转移。

过去的挑战，比如一个 XFaaS 函数过载了[TAO 数据库](https://research.facebook.com/publications/tao-facebooks-distributed-data-store-for-the-social-graph/)，导致服务级联故障，凸显了这些防护措施的必要性。现在，XFaaS 已经证明了其在保护下游服务方面的能力，在真实世界的事件中，其反压力机制预防性地抑制了潜在的过载，确保了服务的稳定性。

**这是一篇很棒的论文。**Meta 为我们提供了对他们自己的无服务器平台的深入，诚实的了解，同时为那些希望优化自己的无服务器函数使用的开发人员和公司提供了明确的教训。[您可以在此处找到论文，尽管您可能需要机构访问权限才能免费阅读。](https://dl.acm.org/doi/abs/10.1145/3600006.3613155)

此外，使用软件进行硬件优化，例如有效的 CPU 使用，在行业中并没有得到足够的关注。尽管谷歌、Facebook 等公司为其自己的系统进行了优化，但与软件优化相比，它并没有得到太多讨论。

与此同时，如果您对其他真实世界的分布式系统感兴趣，**[我还写过关于 Meta 如何通过创建世界上最大的 Memcached 系统来处理数十亿请求的文章。点击此处阅读。](https://engineercodex.substack.com/p/how-facebook-scaled-memcached)**

如果您对论文或系统的任何部分有任何疑问，请在下方留下评论。我会回答它们。