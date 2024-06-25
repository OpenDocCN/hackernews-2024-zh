<!--yml

类别：未分类

时间：2024 年 5 月 27 日 14:37:48

-->

# Discord 如何在一个服务器上为 1500 万用户提供服务

> 来源：[`blog.bytebytego.com/p/how-discord-serves-15-million-users`](https://blog.bytebytego.com/p/how-discord-serves-15-million-users)

GenAI 如何影响软件开发？

加入 LinearB 和 ThoughtWorks 的全球 AI 软件交付负责人，探讨显示 AI 影响的指标，解析在软件开发中利用 AI 的最佳实践，并测量您自己的 GenAI 计划的投资回报率。

[这个研讨会](https://bit.ly/LinearB_010924) 包括：

**📊来自 LinearB 的新 GenAI 影响报告的数据见解**

**🗣️案例研究** 揭示了其他人是如何做到的

**🔎影响度量：**采用情况、效益和风险指标

✅**现场演示：** 如何今天就衡量您的 GenAI 计划的影响

加入 1 月 25 日或 30 日的讨论。

[立即注册](https://bit.ly/LinearB_010924)

* * *

在 2022 年初夏，Discord 运营团队注意到他们的仪表板上活动异常频繁。他们认为这是机器人攻击，但实际上是来自 MidJourney 的合法流量 - 这是一个新兴的、迅速增长的社区，用于根据文本提示生成 AI 图像。

要使用 MidJourney，您需要一个 Discord 账户。大多数 MidJourney 用户加入一个主要的 Discord 服务器。这个服务器增长得如此之快，以至于很快就达到了 Discord 以前的约 100 万用户的限制。

如果他们不迅速采取行动，Discord 就会面临失去这个重要新社区的风险。

这是 Discord 团队如何创造性地解决这个挑战的故事。他们找到了方法，大幅扩展了他们的基础设施能够处理的内容 - 让繁荣的 MidJourney 社区在 Discord 上保持活跃。

Discord 是一个受数亿人使用的流行聊天应用程序，用于连接。最初是为玩家设计的，现在各种类型的社区都在使用它 - 从徒步俱乐部到学习小组再到大型游戏社区。

在 Discord 中，“服务器”托管一个社区。它有聊天频道，讨论由服务器所有者选择的主题。

在内部，Discord 将这些服务器称为“公会” - 所以我们将在后文中使用这个术语。

最大的 Discord 公会（图片来源：[Discord](https://discord.com/servers)）

在 MidJourney 之前，最大的公会有大约 100 万成员 - 像 Roblox 和 Fortnite 这样的大型游戏社区。

Discord 的工程团队认为 100 万成员非常接近一个公会可以处理的最大数量。让我们探讨一下原因 - 但首先，让我们简要了解一下支撑 Discord 的技术。

Discord 的实时消息后端是用 Elixir 构建的。Elixir 运行在 BEAM 虚拟机上。BEAM 是为 Erlang 创建的 - 一种针对需要牢固可靠性和正常运行时间的大型实时系统进行优化的语言。

BEAM 提供的一个关键功能是极轻量级的并行进程。这使得单个服务器能够有效地同时运行成千上万个进程。

Elixir 将友好、受 Ruby 启发的语法带入了 BEAM 的经过大量测试的基础。它们的结合使得编写大规模可扩展、容错系统变得更加容易。

比较 Erlang 和 Elixir 语法的代码片段（图片来源：[elixirforum](https://elixirforum.com/t/code-snippets-to-compare-erlang-and-elixir-syntax/16443/3)）

因此，通过利用 BEAM 的轻量级进程，驱动 Discord 的 Elixir 代码可以同时向全球数十万用户“扇出”消息。然而，随着社区规模的扩大，限制也出现了。

正如前面提到的，Discord 使用高度并发的 BEAM 虚拟机上的 Elixir 进程处理所有实时通信。

消息通过 Discord 的实时基础设施传递到公会中的其他用户和机器人的路径（来源：[Discord 工程博客](https://discord.com/blog/maxjourney-pushing-discords-limits-with-a-million-plus-online-users-in-a-single-server)）

在内部，每个 Discord 社区被称为“公会”。一个专门的 Elixir “公会进程”处理每个公会的协调和路由。这跟踪所有连接到公会的用户。

每个在线用户都有一个单独的 Elixir “会话进程”。当公会进程收到新消息、事件或更新时，它会将这些信息扇出到相关的会话进程。然后，这些会话进程将更新通过 WebSocket 推送给 Discord 客户端。

这种架构为 Discord 在云基础设施中的大量 Linux 服务器上处理数百万个活跃公会提供了一种经济有效的方式。

然而，随着公会规模的扩大，扩展限制也出现了。向更多用户分发消息和事件会产生指数级的工作量。更大的公会也有更多的活动要分发。

因此，随着用户数量的增加，公会进程的负载增长速度会更快。BEAM 能提供巨大帮助，但是一个 BEAM 进程能处理的事情是有限的。

这就是为什么 Discord 认为每个公会同时在线用户突破 100 万会很困难。

* * *

如果你不是付费订阅者，这就是你本月错过的内容。

1.  [Netflix：当你按下播放按钮时会发生什么？](https://blog.bytebytego.com/p/netflix-what-happens-when-you-press)

1.  [另外 6 个微服务面试问题](https://blog.bytebytego.com/p/6-more-microservices-interview-questions)

1.  [7 个微服务面试问题](https://blog.bytebytego.com/p/7-microservices-interview-questions)

1.  [为什么互联网既稳健又脆弱](https://blog.bytebytego.com/p/why-the-internet-is-both-robust-and)

1.  [利用人工智能实现高度相关的搜索](https://blog.bytebytego.com/p/unlock-highly-relevant-search-with)

要获取所有完整文章并支持 ByteByteGo，请考虑订阅：

* * *

在建立了这样的背景之后，让我们回到主要故事。面对 Midjourney 的快速增长带来的规模危机，Discord 组建了一个由资深工程师组成的小团队来深入研究这些问题。这个团队被称为 MaxJourney。

这就是他们取得的成就。

在改进之前，了解系统花费时间和内存的位置至关重要。团队使用各种分析技术分析公会流程的性能。

最简单的方法是抽样堆栈跟踪以揭示昂贵的操作。这样可以快速凸显问题，而不需要太多努力。但是，需要更丰富的数据。

因此，他们对事件循环进行了仪表化，以记录每种消息类型的指标。这包括频率、最小/最大/平均处理时间。这种分析揭示了需要优化的成本最高的操作。可以忽略廉价的操作。

还检查了内存使用情况，因为它影响硬件需求和垃圾收集吞吐量。

为了相对快速地估计大型数据结构的大小，构建了一个辅助库来对地图和列表进行抽样。它避免了完全遍历所有元素。

这次抽样揭示了需要重构的内存密集型领域。

有了对这些时间和内存热点的可见性，团队现在可以有系统地针对性地优化代码。

团队的第一个优化是减少不必要的工作。他们意识到客户端应用程序并不总是需要在应用程序前台不活动时更新公会的每个更新。

因此，他们为这些公会实现了“被动”连接。被动连接跳过处理和数据传输，直到用户打开该公会。

超过 90% 的用户 - 公会连接变为大服务器的被动连接。这将所需的工作量减少了 90%，大大减轻了负载。

然而，MidJourney 不断增长。因此，单独做这些还不够。

中继已经存在，用于跨 BEAM 进程进行扩展的扇出工作。中继仅在大型公会中启用，它们代表公会维护会话连接。

每个中继处理扇出和权限，最多可容纳 15,000 名用户。这允许利用更多的 BEAM 进程来服务大型公会。

最初，中继复制了完整的成员列表。这很容易实现，但对于拥有数百万成员的大型公会来说，几十个复制列表浪费了大量的内存。

此外，创建中继会在序列化和传输成员数据时使大型公会停顿几秒钟。

团队优化了中继，仅跟踪每个中继所需的微小成员子集。

除了整体吞吐量，确保低延迟也至关重要。因此，团队分析了操作的每次调用持续时间高的情况，而不仅仅是总时间。

重要的问题是成员迭代需要花费几秒钟，阻塞了公会。解决方案是使用工作进程来卸载这些任务。工作进程利用 ETS，这是一个用于快速 BEAM 进程间数据共享的内存数据库。

成员存储在 ETS 中，最近的更改在公会的堆中。这种混合模型保持了公会的内存较小。

对于慢任务，会生成工作进程以异步方式运行它们，使用共享的 ETS 数据，从而使公会能够继续处理消息。

一个例子是处理机器之间的公会迁移的缓慢任务。将状态从旧的公会进程复制到新进程通常会使旧进程停滞数分钟。但是，将此任务卸载到工作程序中可避免阻塞旧的公会进程处理传入消息。

另一个想法是将公会的扇出从公会分离到单独的“发送者”进程，进一步减少公会的工作量，并使公会进程免受网络背压的影响。

然而，由于病态垃圾收集，性能意外下降。分析表明，这是由于释放堆外的小内存而触发的。

调整虚拟二进制堆大小解决了这个问题。现在可以启用卸载，显着提高吞吐量。

通过系统优化，MaxJourney 团队实现了看似不可能的事情 - 将公会容量扩展了 15 倍，以保持 MidJourney 在 Discord 上蓬勃发展。

[1] [Maxjourney：在单个服务器上推动 Discord 超过一百万在线用户的限制](https://discord.com/blog/maxjourney-pushing-discords-limits-with-a-million-plus-online-users-in-a-single-server)

[使用 Rust 将 Elixir 扩展到 1100 万并发用户](https://discord.com/blog/using-rust-to-scale-elixir-for-11-million-concurrent-users)

[2] [如何将 Discord 的 Elixir 扩展到 500 万并发用户](https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users)

[3] [Discord 开发者门户 — 文档 — 公会](https://discord.com/developers/docs/resources/guild)

[4] [GitHub - discord/manifold：Erlang/Elixir 之间的快速批量消息传递。](https://github.com/discord/manifold)

[5] [BEAM（艾伦虚拟机器） - 维基百科](https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine))

[6] [艾伦的虚拟机器，BEAM](https://www.erlang-solutions.com/blog/erlangs-virtual-machine-the-beam/)

[7] [介绍 — Elixir v1.16.0](https://hexdocs.pm/elixir/1.16/introduction.html)
