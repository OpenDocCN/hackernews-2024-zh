<!--yml

分类：未分类

日期：2024-05-27 13:23:33

-->

# PostgreSQL社区讨论ALTER SYSTEM [LWN.net]

> 来源：[https://lwn.net/Articles/968300/](https://lwn.net/Articles/968300/)

| **请考虑订阅LWN**订阅是LWN.net的生命线。如果您欣赏这篇文章并希望看到更多类似的内容，您的订阅将有助于确保LWN继续繁荣。请访问[此页面](/subscribe/)加入我们，支持LWN的持续发展。 |
| --- |

作者：**Jonathan Corbet**

2024年4月8日

有时候，即使是最小的补丁也可能引发最激烈的讨论。一个典型例子就是PostgreSQL社区的处理过程——这个本来不倾向于长篇强烈言辞的团体——他们如何解决是否合并一个添加新配置参数的简短补丁的问题。有时候，一个看似安全补丁的提议实际上并非旨在成为安全补丁，但要让这一点得到理解却并不容易。

PostgreSQL服务器是一个复杂的工具，可以根据其运行环境进行广泛配置和调整。自然地，[有很多配置参数](https://www.postgresql.org/docs/current/runtime-config.html)可供管理员使用。有两种设置这些参数的方式。在大多数部署中，管理员可能会编辑`postgresql.conf`文件以根据需要配置系统。但是，也可以在运行中的数据库内调整参数，使用[`ALTER SYSTEM`命令](https://www.postgresql.org/docs/current/sql-altersystem.html)。这种方式进行的任何更改都将保存到一个单独的`postgresql.auto.conf`文件中，该文件也会在服务器启动时被读取；因此这些更改是持久的。

#### 禁用`ALTER SYSTEM`

在2023年9月初，Gabriele Bartolini [提出了这样的想法](/ml/pgsql-hackers/CA+VUV5rEKt2+CdC_KUaPoihMu+i5ChT4WVNTr4CD5-xXZUfuQw@mail.gmail.com/)，允许管理员通过命令行参数或配置选项来禁用`ALTER SYSTEM`（在PostgreSQL术语中称为“GUC”，即“grand unified configuration”参数）。

> 主要原因在于，这将有助于在Kubernetes/Cloud Native环境下改进Postgres的“默认安全”姿态，以及在任何VMs/bare metal环境中通过配置管理系统进行声明式和版本化更改的环境中的普遍适用性，比如Ansible Tower等。

PostgreSQL核心贡献者Tom Lane迅速地表达了对这一提议的不同意见："<q>我认为我们不需要向权限系统添加随机的临时解决方案。特别是我不相信像‘超级用户不再具有所有权限’这样的临时解决方案。</q>" 如果真的需要，他建议使用[event triggers](https://www.postgresql.org/docs/current/event-triggers.html)在本地实现这样的限制。

Alvaro Herrera [指出](/ml/pgsql-hackers/202309091514.fdjfxlr5kjrw@alvherre.pgsql/)，作为系统级命令的`ALTER SYSTEM`不会触发事件触发器。他还[更详细地解释了使用案例](/ml/pgsql-hackers/202309081455.mjyy7fyeb7xb@alvherre.pgsql/)，这是推动此请求的原因：

> 我已经读过，容器中的人们以一种不同于我们历史上看待它们的方式来考虑服务；他们说"对待它们如待牲畜，而非宠物"。这影响了你对这些服务的思考方式。postgresql.conf（实际上是所有 PG 配置）只是一个从外部数据库服务器描述中派生出来的文件。你不再逐个喂养你的 PG 服务器，而是它们像一群牲畜一样行动，而放牧者是某种容器监督程序（不管它叫什么）。
> 
> 确保配置状态不会从内部改变对于维护服务的完整性至关重要。如果用户想要进行更改，可以通过外部操作工具来实现。

然而，Lane [描述](/ml/pgsql-hackers/1882832.1694187082@sss.pgh.pa.us/) 这一功能为 "<q>伪安全</q>"，讨论逐渐冷却了一段时间。但基本的分歧已经显现：反对限制`ALTER SYSTEM`的理由是它被视为安全功能的想法。因此，它将是一个失败，因为对于能够运行`ALTER SYSTEM`的 PostgreSQL 用户来说，有许多其他方法来控制服务器。但正如Bartolini [所说](/ml/pgsql-hackers/CA+VUV5pK4N8FaGa0y47ZFVk9qWOwWMYOAUtX0J-o4A7hHfJRYA@mail.gmail.com/)，这种限制是作为*可用性*功能，关闭了一个不应在由声明性配置系统控制的系统中使用的配置机制。

#### 新的一年

Robert Haas [在一月重新开始了讨论](/ml/pgsql-hackers/CA+TgmoaJxzx+EqmfVQxtJMKor7WSminuHjfCaO1N2z=cNoHG6Q@mail.gmail.com/)，承认这个提议并非旨在作为安全功能，但担心它仍会被视为此类功能：

> 我不得不承认，我有点担心人们会把这个误认为是真正的安全功能，并提交关于超级用户能够绕过这些限制的 bug 报告或 CVE。如果我们添加这个功能，最好确保文档非常清楚地说明我们在保证什么，或者更确切地说，我们不在保证什么。

即使如此，他还担心，关于"<q>安全研究人员威胁在注册表中发布我们的邪恶</q>"。然后Lane [宣布](/ml/pgsql-hackers/2256675.1706642404@sss.pgh.pa.us/) 项目"<q>不应仅仅拒绝这项提议，还应该拒绝任何未来打算阻止超级用户执行超级用户通常可以执行的操作的提议</q>"。然而，Haas [回应说](/ml/pgsql-hackers/CA+TgmoYNBs68DQifceGb4SNS8iaLt_wQCKuVJ8rNqeSvU4LiKA@mail.gmail.com/)，原始提议可能具有一定的价值，应该认真对待。

对话没有得出结论继续进行；两个月后，Haas [抱怨说](/ml/pgsql-hackers/CA+TgmoaCwtTn7QEuRYFUWypXUv6La5_-oYiiMJgw4LeC-ZujBw@mail.gmail.com/)进展不大。尽管如此，他说：“<q>从阅读讨论串来看，大多数人同意有某种方式来禁用 ALTER SYSTEM 是合理的</q>”。然而，有六种竞争的方式可以实现这一目标。这些包括最初提出的命令行选项和配置参数，以及事件触发器、将其推入扩展模块、识别由管理员创建的哨兵文件，或者只是更改 `postgresql.auto.conf` 的权限。他建议配置选项和哨兵文件是最可行的选项。

Lane [回答道](/ml/pgsql-hackers/2366852.1710443616@sss.pgh.pa.us/)，任何通过上述任何机制实施的限制，仍然可以被恶意管理员绕过。Haas [回复说](/ml/pgsql-hackers/CA+TgmobRx215ussbQgjM4dgds8xAcMwKeKmHN4UQoSV5jP4uzQ@mail.gmail.com/)，这个提议不是一个安全特性，但Lane [却否认说](/ml/pgsql-hackers/2372973.1710446903@sss.pgh.pa.us/)这是“<q>一个用儿童友好颜色涂成的带子弹的脚枪</q>”，这将导致更多虚假的 CVE 编号针对项目。

#### 一个新的补丁

3月15日，Jelte Fennema-Nio 发布了[一个补丁](/ml/pgsql-hackers/CAGECzQRrLpLzoOHmmUzo4g_Ed0CchJrez90s1tzZNoH00v-eFA@mail.gmail.com/)，将限制作为配置参数实施。最终，这只是 Bartolini 在9月份发布的补丁的更新版本，带有一些文档调整。各种评论导致了许多更新版本；[第六个版本](/ml/pgsql-hackers/CAGECzQQttoyf5EtjZvG6Biyx=cTKON4zgo2Ok27HgmvkayCb_A@mail.gmail.com/)几天后出现。此时，该补丁主要包括文档，其中包括以下警告：

> 注意，此设置不能被视为安全功能。它仅禁用 `ALTER SYSTEM` 命令，不能阻止超级用户通过其他方式远程更改配置。

Haas [欢迎这个版本](/ml/pgsql-hackers/CA+TgmoZ26YPK0to4Ac-oTz56Ae_uU2zearWDwPLHAFeVtcjqsA@mail.gmail.com/)，并询问是否就继续进行达成共识。此时，Lane 似乎在某种程度上[改变了他的看法](/ml/pgsql-hackers/2517206.1711388833@sss.pgh.pa.us/)，他说：“<q>我从未反对能够禁用 ALTER SYSTEM 的想法</q>”。Bruce Momjian 则[担心](/ml/pgsql-hackers/ZgHNeR2qlxofPH_W@momjian.us/)，管理员可能输入一个禁用 `ALTER SYSTEM` 命令，此时恢复可能会很困难。事实上，正如 Fennema-Nio [回答的](/ml/pgsql-hackers/CAGECzQQ5CaNo=JnL5_sxFCmyV8_76A9L7X-hRQYzhGLpJYzt4Q@mail.gmail.com/)，不能以那种方式设置该参数，因此不存在那种特定的陷阱。

Momjian 还对参数名称（`externally_managed_configuration`）提出了异议，认为它并没有真正描述受限制的内容。他提议 `sql_alter_system_vars` 作为替代方案。Haas [同意](/ml/pgsql-hackers/CA+TgmoaX_XJMORzkTvG0D6+KZP4mYUSq1Paa8YOSmZUy0=tJSQ@mail.gmail.com/) 这个抱怨，但认为 `allow_alter_system` 更有意义。这最终是选择该选项名称的原因。

讨论还没有结束。Lane [希望](/ml/pgsql-hackers/3868891.1710859910@sss.pgh.pa.us/) 服务器确保如果禁用 `allow_alter_system`，则 `postgresql.conf` 和 `postgresql.auto.conf` 不可写入 `postgres` 用户。否则，数据库管理员仍然可以通过编辑其中一个文件来修改配置。Fennema-Nio [持不同意见](/ml/pgsql-hackers/CAGECzQQhvV1K-pDgrd8NVqXP0PUY9bCDs+CvE31KXX6hKqNB-Q@mail.gmail.com/)，称禁用 `ALTER SYSTEM` 对预期的用例已足够，但 Lane [认为](/ml/pgsql-hackers/3879337.1710864320@sss.pgh.pa.us/) 单独的配置参数只是 "<q>一个可能会愚弄无能审计员但不会有更多作用的挡箭牌</q>"。Fennema-Nio [提醒](/ml/pgsql-hackers/CAGECzQQoZC0tG1xpi_+-O7uc3ESA7vJ+-Gsnb18WF0e1X78JKA@mail.gmail.com/) Lane，`allow_alter_system` 的目的不是安全性。Haas [抱怨](/ml/pgsql-hackers/CA+TgmoYsyrCNmg+Yh6rgP7K8r-bYPjCeF1tPxENRFwD4VZAZvw@mail.gmail.com/) 说：“<q>我不明白为什么那些讨厌这个功能并希望它彻底消失的人可以决定它的运作方式。</q>” 最终并未添加文件权限检查。

即便如此，讨论仍未结束；Momjian [质疑](/ml/pgsql-hackers/ZgSceowq_HL3VI_l@momjian.us/) 在 PostgreSQL 开发周期的最后阶段合并此更改的决定。他说：“<q>我的观点是我们在提交周期的最后几周设计用户API，通常对我们不利。</q>” Fennema-Nio [指出](/ml/pgsql-hackers/CAGECzQSZ9Z1cs_Ld0YcMrF6HyPVvzvtKmq9RGuRVcwmnSH_9BQ@mail.gmail.com/) API 从九月初的形式基本未变，已经花了几个月讨论替代方案。Haas [表示](/ml/pgsql-hackers/CA+Tgmob3G5AL_fL0LGK8xBe1-TA6GhUCsJ5ohpaY348TLSv_-Q@mail.gmail.com/) 这样一个小补丁不会因为延迟到下一个发布周期而变得更好：“<q>我认为在大家都在思考这个问题并且问题在大家脑海中还很清晰的时候完成这件事情是正确的。</q>"

3月28日，Momjian [同意](/ml/pgsql-hackers/ZgXF7CEVb61aDRzY@momjian.us/) 可以合并该补丁。一天后，Haas [就这么做了](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commitdiff;h=d3ae2a24f265a028f4b9e8df79ea7b075c6cf016;hp=0075d78947e3800c5a807f48fd901f16db91101b)。有鉴于此，最近 PostgreSQL 历史上一场持续时间最长的开发讨论终于结束了。这一成功证明了少数开发人员的坚持，他们在数月的反对和“有益的”实现建议中坚持了下来。在决定了这个棘手的问题之后，该项目可以把注意力转向[即将到来的七月提交节中需要解决的一小部分简单主题](https://commitfest.postgresql.org/48/)。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/968300/)

发表评论）
