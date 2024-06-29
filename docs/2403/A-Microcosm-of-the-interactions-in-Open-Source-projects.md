<!--yml

category: 未分类

date: 2024-05-29 12:49:36

-->

# 开源项目互动的微观视角

> 来源：[https://robmensching.com/blog/posts/2024/03/30/a-microcosm-of-the-interactions-in-open-source-projects/](https://robmensching.com/blog/posts/2024/03/30/a-microcosm-of-the-interactions-in-open-source-projects/)

将会有大量分析 xz/liblzma 漏洞。然而，我发现大多数人跳过了攻击的第一步：

> 1.  原始维护者燃尽了自己，只有攻击者提供帮助（因此攻击者继承了原始维护者建立的信任）

令人惊讶的是，有人找到了一个包含有关世界状态的电子邮件线索的存档。让我们读读他们的话。

首先，我们从一个合理的请求开始。这个问题迫使维护者面对他的“失败”。我在这里用引号来表示“失败”，因为a. 维护者在这里实际上并没有欠任何东西，所以他实际上并没有失败，b. 我完全理解这种感受。让“社区”失望感觉真是糟透了。

> ”XZ for Java 还在维护吗？我一个星期前在这里问了个问题，但没有收到回复。” - [https://www.mail-archive.com/[email protected]/msg00562.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00562.html)

维护者承认自己“落后”，正在挣扎着跟上进度。这是一声痛苦的呼唤。这是一种求助的呼声。在这个帖子中帮助是不会到来的。

> ”是的，至少按某种定义来说，如果有人报告了一个 bug，它会被修复。但是开发新功能确实并不是非常活跃。:-(” - [https://www.mail-archive.com/[email protected]/msg00563.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00563.html)

哦！在同一条消息中，我们介绍了我们的 xz/liblzma 攻击者。这可不是你期待的帮助。

> ”贾坦帮助了我……并且他将来可能会扮演更重要的角色……显然我的资源太有限……所以长期来看必须有所改变。” - [https://www.mail-archive.com/[email protected]/msg00563.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00563.html)

反而，一个不助人的消费者说了不助人的话。这正是这类电子邮件线程的典型发展方向。

> ”在有新的维护者之前进展不会发生……现任维护者失去了兴趣或者不再关心维护。对于像这样的存储库，看到这种情况很令人沮丧。” - [https://www.mail-archive.com/[email protected]/msg00566.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00566.html)

*另外：考虑到 xz/liblzma 漏洞看起来像是 “贾坦” 故意的攻击，那么 “吉加尔·库马尔” 主动鼓励原始维护者放弃，是否应该认为他是帮凶呢？不确定？我们很快会再次见到这位不助人的消费者 “吉加尔·库马尔”。*

不可避免地，维护者试图为自己辩护。维护者们处理燃尽压力的方式各不相同。我倾向于生气，这最终变得愤世嫉俗。然而，这种反应是令人心碎的。

> ”我并没有失去兴趣，但由于长期的心理健康问题，以及其他一些事情，我关心的能力确实相当有限。” - [https://www.mail-archive.com/[email protected]/msg00567.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00567.html)

…然后维护者还提醒大家，现在的软件是如何构建的。

> ”还要记住，这是一个无偿的爱好项目” - [https://www.mail-archive.com/[email protected]/msg00567.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00567.html)

一周后，这位无益的消费者又回来了。**一周后。**

> ”你忽略了这个邮件列表上正在腐烂的许多补丁。现在你压制了你的仓库。为什么要等到5.4.0版本才更换维护者？为什么延迟你的仓库所需的改变？” - [https://www.mail-archive.com/[email protected]/msg00568.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00568.html)

这有什么意义呢？我无法告诉你这让维护者多么愤怒。

然后我们最初开始这封电子邮件线程的理性请求者决定现在是提出要求的好时机。

> ”我对你的心理健康问题感到抱歉，但重要的是要意识到自己的限制。我知道这对所有贡献者来说都是一个爱好项目，但社区希望更多” - [https://www.mail-archive.com/[email protected]/msg00569.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00569.html)

再读一遍最后一句，“社区希望更多”。消费者必须被满足。维护者的需求，其中显然有几个重要的需求，被忽视了。

我们这位不再理性的请求者还提供了一些建议。请注意，并没有提供实际帮助的提议。

> ”为什么不把XZ的C语言维护权移交出去，这样你就可以更关注XZ的Java版本？或者把XZ的Java版本交给其他人，专注于XZ的C语言？试图同时维护两者意味着两者都得不到良好的维护。” - [https://www.mail-archive.com/[email protected]/msg00569.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00569.html)

因此，维护者不得不解释现实情况。

> ”寻找一个共同维护者或者完全将项目移交给其他人已经在我的脑海中很久了，但这不是一件简单的事情。例如，某人需要具备技能、时间和足够的长期兴趣，专门用于此。” - [https://www.mail-archive.com/[email protected]/msg00571.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00571.html)

编写软件需要技能和知识。虽然许多技能和一些知识会转移，但参与一个新的软件项目不可避免地需要开发新的技能和更多的知识。

软件开发人员不是可以随意更换的可互换零部件。

邮件讨论最终以抱怨的消费者提供不了帮助，但继续提出要求而结束。只剩下攻击者了。

> ”贾坦将来可能在项目中扮演更重要的角色。他在幕后提供了很多帮助，几乎已经是一个共同维护者了。:-)” - [https://www.mail-archive.com/[email protected]/msg00571.html](https://www.mail-archive.com/xz-devel@tukaani.org/msg00571.html)

这个帖子是开源项目中互动的一个缩影。消费者向一个（很少是两个）维护者提出各种要求（有些礼貌，有些不那么礼貌）。

不要误会。这就是事情的运转方式。

[这需要改变。](/blog/posts/2024/03/31/what-could-be-done-to-support-open-source-maintainers/)
