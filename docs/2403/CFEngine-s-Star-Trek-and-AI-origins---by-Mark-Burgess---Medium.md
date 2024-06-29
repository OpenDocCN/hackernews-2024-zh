CFEngine的星际迷航与人工智能起源

30周年纪念致敬

<!--yml

1993年，我与奥斯陆大学物理系的系统管理员深入讨论之后，编写了CFEngine。当时，我们正使用shell和Perl脚本来自动化约五十台服务器的操作。物理系是该大学Unix-like操作系统中最苛刻的IT环境之一。那时候的Unix有很多种：SunOS4、后来成为Solaris的SunOS5、带有类似Sinclair Spectrum的橡皮键盘的HP-UX、Apollo工作站、IBM的多版本AIX、DEC Alpha上的OSF1、Ultrix等等。Linux当时还只是一个梦想。

# 如今，CFEngine广为人知的是一款配置管理工具，曾经被Puppet、Chef，然后是Ansible所取代。*“我知道工程师，他们喜欢改变事物！”*（在《星际迷航：电影版》中Dr McCoy曾如此说）。确实，我们喜欢新工具，但说CFEngine被取代了并不准确。它依然活跃且健康。更重要的是，仅仅称CFEngine为配置管理也不完全准确。CFEngine的设想远远超出了我们现在理解的配置管理范畴。它设计之时还没有虚拟机和云计算，但其原则至今仍然合乎逻辑，适用于现代IT世界的很多方面。要在现代IT系统中实现CFEngine多年来的所有功能（配置、监控、网络路由、文本处理、网络编排等），你需要半打不同的软件，并需要进行一些复杂的集成。

> CFEngine的星际迷航与AI起源 | 作者：Mark Burgess | Medium

# -->

## 我和CFEngine的Nick Anderson合影

日期: 2024-05-29 12:39:45

来源：[https://mark-burgess-oslo-mb.medium.com/cfengines-star-trek-and-ai-origins-e99096fe845b](https://mark-burgess-oslo-mb.medium.com/cfengines-star-trek-and-ai-origins-e99096fe845b)

类别: 未分类

*CFEngine今年已经30岁了。对于一款软件来说，这算是很长的生命周期了。它的发展历程中发生了很多事情，但大多数用户现在已经忘记了它是如何从一个年轻人对人工智能的梦想中诞生的。我在我的书籍* [*《寻找确定性》*](http://markburgess.org/certainty.html)* 中详细探讨了这门科学，甚至还有一些* [*传记*](/list/youre-not-a-scientist-maybe-a-dj-e09240fbaaa6) *笔记。但是，抓杯咖啡过来，围坐一起，我会告诉你更多。*

这些Unix的风味彼此非常不同 —— 有些基于[BSD Unix](https://en.wikipedia.org/wiki/Berkeley_Software_Distribution)，有些基于[System V](https://en.wikipedia.org/wiki/UNIX_System_V)。而且，大学里的每个人都有特殊需求！今天我们试图使系统尽可能相似以便管理规模，但CFEngine设计用于处理多样性和变异性，而不违反人力的界限。它把用户需求放在管理方便性的限制之前。它不仅仅是关于*知识管理*，也是关于配置管理。生物学可以处理多样性，为什么技术不能？

大学计算服务USIT的一位同事编写了一些令人印象深刻的日常和每周shell脚本来管理大学计算服务在机器上的兴趣。定期运行相同软件的想法，就像一个错误校正循环，是令人着迷的。大多数软件安装一次，然后进入无人监控模式。我对这些维护脚本很感兴趣 —— 至少也是因为它们处理不同系统之间不同语法的巨大复杂性。有一半的脚本是if-then-else测试，以确定精确的版本。人们几乎无法看到它们究竟在做什么。

我们讨论的是一个不生存在物理世界的*软件机器人*，而是其环境是操作系统的抽象状态空间。声明性语言可以用来表达期望的意图，但每台机器都必须具备自己状态的机器人技能。那时候，你从未知道网络是联通还是断开。这个机器人不会像80年代人们所想的那样操纵块或棋子，而是系统文件和进程。

其中一个主要挑战是配置困难的子系统，比如[伯克利Sendmail](https://en.wikipedia.org/wiki/Sendmail)，涉及复杂的文本编辑和进程管理。不能简单地安装标准文件，因为它们在所有机器上都不起作用，并且不会考虑本地用户所需的特殊需求。最好让人们不要碰主机，但你永远不能保证，所以强行改变是不好的做法。从这一经验中，自主性和*承诺*与*强制*的概念最终演变成了[承诺理论](https://en.wikipedia.org/wiki/Promise_theory)。CFEngine的文本编辑语言仍然是自动编辑文件的最复杂模型之一，以收敛方式运作，同时避免其他人可能已经完成的工作（我们现在倾向于称之为幂等，这并不完全准确）。

在CFEngine中关于收敛的旧笔记。今天新一代的名字放弃了半格和幂等性。

仅仅运行Shell命令并不能如今为相对同质的Linux所做到。 其中一个问题是，不同的Unix系统中的所有Shell命令选项都是不同的。 软件代理人需要像人类一样具备复杂的操纵技能，并尽可能少地造成停机中断。

答案显而易见：应该分离认知或*感知*环境问题，并隐藏所有这些检查，使其与意图的语言相分离，以揭示目的。 我们讨论了使用特定领域语言的想法，以使一切变得清晰明了，在圣诞节假期期间我写了一篇。 它起初是一个临时性的事务，经历了几次修订，但它完成了揭示意图而非技术细节的任务。 简单是当时系统管理员们所期望的（如今的情况已经不同了，在开发者时代）。

2009年会议中我做的一个涂鸦准确地展示了CFEngine作为在更大的配置游戏中整合独立部件的整合者 :)

脚本中无尽的if-then-else语句被一个类似于Prolog的策略规则相关性的集合代数评估器所取代。 当时我对人工智能研究了解不多，除了在[Douglas Hoftadter的《哥德尔，艾舍尔，巴赫》](https://en.wikipedia.org/wiki/G%C3%B6del,_Escher,_Bach)中读到的内容。我当时凭直觉开始，直到后来才尝试将这些想法放在[适当的学术基础](http://markburgess.org/sysadmin.html)上。 尽管如此，我仍然惊讶于如何经常重新发现，解决现代问题的正确方式是以我在CFEngine中直觉做的方式。

作为一名物理学家，对计算机感兴趣，管理系统的挑战有多个维度。 我不仅仅想要运行命令然后被遗忘，就像[马尔科夫过程](https://en.wikipedia.org/wiki/Markov_chain)一样。 我想要理解机器的动态稳定性并测量它们的行为。 我对[人工生命](https://en.wikipedia.org/wiki/Artificial_life)的概念很感兴趣，我想象过像星际迷航中的那种系统，你可以向计算机询问诊断，然后一切都能自动修复。 在[早期的在线手册](https://www.gnu.org/software/cfengine/daystrom.html)中，我甚至引用了《星际迷航：究极计算机》这一原始剧集中关于Dr Daystrom的M5“双电子管”计算机的一些自嘲。

> *Kirk*: “我很好奇，博士，为什么它被称为M5？”
> 
> *Daystrom*: “嗯，你看，M1到M4并不完全成功。 而这一款确实成功了。 M5已准备好控制你的飞船。”
> 
> *Kirk*: “全面控制？”
> 
> *Daystrom*: “这就是它的设计初衷。”
> 
> *Kirk*: “有些事情人类必须做才能继续保持人性，你的计算机却把这些都夺走了。”
> 
> *Daystrom*: “计算机可以做你的工作…… 一台机器可以完成现在人类所做的所有事情。 人类可以继续做更伟大的事情……”

我对AI并不是专家，但我刚与老同事艾伦·麦克拉克兰合写了一篇论文，他已经离开物理学转向AI研究。AI仍然新鲜在我的脑海中。

我意识到CFEngine需要具备广泛的感知技能，以了解它所工作的Unix系统的类型，并具备强大的操作器，能够改变系统配置，特别是在编辑Unix上随处可见的文本文件时。在90年代中期，我受到免疫学和[群体智能](https://en.wikipedia.org/wiki/Swarm_intelligence)的启发。Polly Matzinger的人类免疫系统的[危险模型](https://en.wikipedia.org/wiki/Danger_model)在BBC的纪录片中亮相，我开始开发[自愈能力](https://www.usenix.org/conference/lisa-98/computer-immunology)，以参考我们免疫系统所采用的环境调节的巧妙方法。

最终，我在大学内部发布了当时称为cfengine 1.0的版本，并在1993年春季发布了用于CERN的[cfengine 2.0](http://markburgess.org/papers/cfengine_history.pdf)，并在那里向CERN unix小组做了一个演讲。我还发表了几篇关于它的论文，并希望它可能对他人有用，但并没有真正的期望。我不知道它会有多快地找到归属。此时，理查德·斯托曼的[自由软件基金会](https://en.wikipedia.org/wiki/Free_Software_Foundation)已经启动了GNU自由软件项目。我认为公开代码并从他人那里学习是个好主意。我将代码发布为cfengine 2.x，并且这个项目成为了[GNU cfengine](https://www.gnu.org/software/cfengine/)的公共版本1.0.0。CFEngine的重新命名CFEngine发生在2009年的CFEngine 3.0时，当时公司成立。

安全问题开始成为一个热门话题，人们的记忆中还新鲜着[互联网蠕虫](https://www.gnu.org/software/cfengine/)的记忆。我意识到实现一个分布式代理系统真正安全的唯一途径是在设计本身中构建安全性。CFEngine被设计为*绝不*接受来自其它来源的命令或指令，只接受本地策略。一些人对不能随心所欲感到愤怒，而另一些人则看到了设定安全限制的智慧。

据我所知，从未有任何CFEngine安装被攻破或者说能被攻破。像所有软件一样，它偶尔会出现缓冲区溢出问题（具有讽刺意味的是，这些问题通常出现在预计的安全中心OpenSSL加密中），但由于一切都被隔离的方式，它们无法被利用。[理查德·斯托曼](https://en.wikipedia.org/wiki/Richard_Stallman)鼓励我使强大的功能更难以调用，以防止意外发生。这些原则对我来说是有意义的，尽管后来因政治分歧我们分道扬镳，但我从他的经验中吸取了教训。

在圣地亚哥的Town & Country Hotel举办的USENIX LISA大会。

1997年，我参加了[USENIX](https://en.wikipedia.org/wiki/USENIX) Lisa大会，向一千多名与会者演讲CFEngine的稳定性机制。之后，我在小房间举行了一场关于这款软件的非正式“鸟羽会谈”，数百人蜂拥而至听我讲解。人们围坐在我脚下的地板上，我则固定在几平方英尺的空间里讲述。这款软件的使用规模已超出我的预期，但我的原始愿景中仍有许多遗漏之处，即智能自治代理系统。在1998年，我在关于CFEngine和人工免疫系统愿景的后续演讲后，编写了GNU cfengine 2。这包括行为模式的机器学习，具备对抗服务拒绝攻击的能力。在观众中，有一位来自新墨西哥大学的成员向我介绍了[斯蒂芬妮·福雷斯特](https://en.wikipedia.org/wiki/Stephanie_Forrest)的工作；我们的团队共同发展了人工免疫系统的不同概念。

我尝试过符号人工智能和贝叶斯机器学习方法，以使CFEngine尽可能自包含。我对任何复杂逻辑推理的东西都非常怀疑。我从物理学中知道，稳定性是任何自动系统的首要关注点，而逻辑推理则非常脆弱。我希望确保没有任何事情能阻止CFEngine对陷入麻烦的主机进行自我修复。我围绕“[不动点](https://en.wikipedia.org/wiki/Fixed_point_(mathematics))”思维构建了CFEngine：系统的每个期望状态都应是可达的不动点，否则应予以抑制。

尽管花了多年时间才理解所有细节，但我的选择出奇的幸运。符号方法是确定和强大的，而机器学习虽然价值有限，但它们使得计算机的首次持续监控研究得以实现异常检测。机器学习很快变得昂贵并且存在一个基本缺陷：*通常*发生的事情未必是您希望您的系统将来要做的。自动驾驶系统擅长直线飞行或在特定机场降落，但如果天气恶劣或发动机掉落，它们无法告诉您该怎么做。在我试图深入理解问题时，我偶然发现了承诺理论和算子代数（有时被称为半格等）。许多这些方法被新一代研究人员（经常在谷歌工作）在21世纪重新发现。

> CFEngine 的严格的自主代理模型意味着不能有推送数据的客户端服务器协议。现在我们所称的[发布订阅](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)方法作为一项政策基础上的所有通信方法。基于发布-订阅消息传递设计了一个版本的[自愿合作基于 RPC](https://www.usenix.org/legacy/events/lisa05/tech/burgess.html)。其效果类似于为互联网服务创建一种类似于[命名数据网络](https://en.wikipedia.org/wiki/Named_data_networking)（NDN）概念。主机们会进行某种形式的“请人来电”以检查消息，而不承诺任何事情或暴露自己于漏洞之中。

这些想法为一个通用平台奠定了基础，但对于现代应用来说一切发展得太慢了。我将 CFEngine 用作通用分布式平台，并与惠普的[克劳迪奥·巴尔托利尼](https://scholar.google.com/citations?user=j6eORnoAAAAJ&hl=en)和[约翰·威尔克斯](https://www.usenix.org/conference/lisa13/speaker-or-organizer/john-wilkes-google)讨论过其在欧洲学术卓越网络项目 EMANICS 中的广泛网络方法应用。我知道这些想法中有些应该放到其他地方，但从未完全有时间在现代化架构中重建它们。最终实施的云计算方式最初违反了我研究和认可的许多原则，需要另外10年才能回归基本的安全原则，例如通过重新发现[Kubernetes](https://en.wikipedia.org/wiki/Kubernetes)来实现。

我也尝试了一些愚蠢的想法，比如将*熵*作为衡量配置的方法（正如大多数学者在某些时候所做的），但通过系统化的研究我尽可能将它们消除。我撰写了一本研究文本《分析网络与系统管理》（J. Wiley & Sons, 2002），并开始攻读硕士学位，并且被授予系统管理领域的第一个教授职位。

到了2000年代中期，GNU cfengine 2 已经无处不在。你可以翻开任何 IT 数据中心的石头，你会发现 CFEngine 就像一个秘密的蚂蚁殖民地在下面工作。在大多数情况下，用户通常只使用其能力的一小部分，并且许多人并不知道在设计上进行了多少精心思考以确保其稳健性。

大约在这个时候，曾经是CFEngine用户的Luke Kanies开始了自己的公司，使用了一个叫做Puppet的工具。我们在2001年的LISA会议上见过面，那时我是会议主席。当我们为组织者准备晚宴时，他一个人在那里徘徊，看起来很迷茫，于是我邀请他加入我们的“高桌”宴会。有一次他失业，斯坦福大学要求我为他们提供建议时，我把这份工作转给了他，他使用Puppet将他们迁移，开始了他的公司Reductive Labs。Luke很直言不讳，并且善于在网上社区中发动群众，这是我所不擅长的。我是一个病态的反社交内向者，有太多矛盾的兴趣。我当时是计算机科学教授，维护CFEngine基本上是一个人的活，非常疲惫。[2008年我创办了一家公司](https://www.aftenposten.no/norge/i/P96nJ/it-mygg-sikter-mot-stjernene)，部分原因是看到Puppet的发展，逃避了我在大学的死胡同工作，另一部分原因是能够把CFEngine的支持负担交给别人。就在CFEngine公司成立之前，Adam Jacob（一位前Puppet用户）和Jesse Robbins也开始了Chef软件。

许多用户转向使用Puppet和Chef而非CFEngine，因为它们是最新的工具。它们早期获得了风险投资，并投入大量资金进行社区建设。其中一些用户之间存在着奇怪的敌意。我开始在网上收到Puppet用户的辱骂。这是我第一次尝试软件社区的情况。它揭示了一种部落主义，让我不太愿意过多参与社区。多年来，我收到了大约30%来自Puppet和Chef用户的恶意邮件。有时是因为我无法融入这个部落之一。

商业利益导致这三种工具竞争激烈。新出现的工具在功能或效率上都不如CFEngine强大，但它们在可用性上更受欢迎，部分原因是基于Ruby代码的开发受到Web一代的鼓励。我着手重写CFEngine 3，以解决代码基础中存在的一些深层限制。我尝试使用C++进行了六个月的实验，最终由于沮丧而放弃，回到了C语言。我在三个月内重新编写了整个代码基础，大多数限制都得到了解决。

当CFEngine 3准备就绪时，已经有一批在Web商务中成长起来的新一代IT人员，他们不具备系统管理员以前所具有的深入系统知识。当时，我承认我并不认为我们能够销售配置管理：毕竟这已经是一个解决的问题，而且已经是自由和开放的——但那只是我商业上的不成熟观念在作祟。我看到竞争市场分析师把前沿研究拉回到了最低级的商品基础上。

CFEngine Nova的知识图和机器学习图表

我为公司的想法是采用知识管理方法，包括语义网络和机器学习，以理解系统。毕竟，配置系统相对容易，但理解你所创造的“怪物”却很难。我在这方面付出了很多努力，与我的朋友[Tufts大学的Alva Couch](https://engineering.tufts.edu/cs/people/faculty/alva-couch)合作。我们认为语义网（及其RDF语言）的方法不正确，因此我们提出了[更简单的替代方案](https://www.researchgate.net/publication/224146282_Human-understandable_inference_of_causal_relationships)来支持因果推理。它首次出现在第一个CFEngine商业产品中，名为CFEngine Nova。那时，我已经在某种程度上发展了[Promise Theory](https://en.wikipedia.org/wiki/Promise_theory)，并且承诺模型严格应用于CFEngine 3，以确保每个操作的可靠性、安全性，尤其是*确定性*。

我们CFEngine商业产品的宏伟计划是从*CFEngine Nova*（星星网络）开始，然后将它们组成*CFEngine Constellation*集群，最后用*CFEngine Galaxy*来完善整个系统！然而，在公司接受风险投资后，新任管理层的聪明头脑去掉了与CFEngine相关的知识和人工智能部分 — 部分是因为开发团队尚未准备好理解它们，部分是因为当前市场缺乏需求。今天，15年后，这些理念正在逐步成熟。

在经历了内部斗争和风险投资的艰难旅程后，我最终决定离开我在2014年创办的公司。我继续开发一些机器学习和语义特性，并将[Semantic Spacetime](https://en.wikipedia.org/wiki/Semantic_spacetime)作为[虚拟过程的因果结构](https://www.researchgate.net/publication/351492269_Motion_of_the_Third_Kind_I_Notes_on_the_causal_structure_of_virtual_processes_for_privileged_observers)的Promise Theory正式化表示。在我希望的情况下，再过十年，我的AI功能版本开始成功运行，我尽力在其他工作间隙中开发了一个小型图形数据库来支持它。然后，与[CAIDA](https://www.caida.org/)和更强大的独立图形数据库[ArangoDB](https://en.wikipedia.org/wiki/ArangoDB)合作，我[在Semantic Spacetime中绘制了整个互联网的图像](https://medium.com/@mark-burgess-oslo-mb/semantic-spacetime-and-data-analytics-aabbb811cb26)。今天，我们正处于人工智能的新时代，一切旧事物都再次焕发生机。

CFEngine赢得了进入JPMC创新名人堂的奖杯。

由于Puppet和Chef的到来带来的商业环境，将叙事重心从我自己的技术抱负转移到了更符合主要以Linux和LAMP堆栈为主的Web一代基础的基本事务上。只有少数高级客户真正能看到CFEngine无限潜力的可能性——正如我决定离开时，J.P.摩根大通将CFEngine纳入他们著名的创新殿堂。

我从未将CFEngine的知识相关特性称为“AI”，但在我心中，始终有着星舰企业号自动飞行和自我愈合的愿景。这些工具比以往更加手动化，更加偏向于开发者而非自我治理。CFEngine的一些设计原则被引入了Kubernetes。Open Policy Agent基本上是CFEngine的分类系统自动化执行，但我愿意认为CFEngine在这个领域仍然占据一席之地。

20 年前，我记得读过一篇文章，作者写道：关于语音识别的利弊，好消息是我们在达到《星际迷航》中理解水平之前还有很长的路要走。坏消息是，至少我们有到23世纪来弄清楚它。在过去的20年中，我们在IT硬件方面取得了长足的进步，但在系统自动化方面并没有取得多大进展。在某些方面，我们甚至出现了倒退。每一代都有新工具需要构建，因为基础设施在变化，但每一代也都必须重新学习过去的经验教训。我们在IT领域并不擅长*原则*。

令人惊讶的是，GNU cfengine 2和CFEngine 3依然在野外运行，管理着一些最大的企业——虽然看不见，但它们确保着系统群在一条直线上飞行。我的朋友尼克·安德森，他仍然在公司工作，并且一直是CFEngine的坚定支持者，告诉我目前最大的单一部署约为300万个节点。很少有公司愿意公开宣布他们做了什么，但知道这些年的努力是值得的，让我感到一些安慰。

生日快乐，CFEngine。中年已经悄然而至。
