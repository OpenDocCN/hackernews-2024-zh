<!--yml

类别：未分类

日期：2024-05-27 12:47:04

-->

# RSS 源组织策略和新的源成本

> 来源：[https://yukinu.com/blog/2024/02/06/rss-feed-processing-strategies.html](https://yukinu.com/blog/2024/02/06/rss-feed-processing-strategies.html)

我在[The New Leaf Journal](https://thenewleafjournal.com/)发布的一篇关于[组织 RSS 和 ATOM 源](https://thenewleafjournal.com/organizing-rss-and-atom-feed-sources/)的文章上阅读到的内容让我开始思考我的 RSS 源策略。从我理解的角度来看，该文章在高层次上描述了几种组织 RSS 源的策略，其中可以分解为四种不同的方法来组织源：

+   **分类**

+   源是根据内容与其他源的相似性进行组织的。

+   **时间**

+   源根据更新频率进行组织。

+   **排名**

+   源按照每个订阅者对其吸引力的排名/评分系统进行组织。

+   **社交图距离**

+   源是按照它们与你个人的接近程度（基于社交图的大致距离）进行组织。

反过来，一个源策略可能会使用这些方法的组合。例如，过去我的 RSS 源组织策略使用了分类和排名方法的组合；我将我的源分为 3 个等级（高、中、低优先级），并根据主题对它们进行分类。然而，随着时间的推移，随着我的源规模的增长，我开始遇到这种策略的规模问题。特别是：

+   大多数 RSS 阅读器使用树形数据结构来组织源，但有些源可能不太适合这种数据结构。例如，如果你有一个播客类别和一个技术类别，那么技术类播客应该如何在这棵树中合理安排？

+   随着添加更多源，可能需要更多类别，并且结果可能迅速发展成为一个难以导航的深度嵌套树。

+   如果你不能每天处理整个源，那么如何处理源排名方法中的较低排名就往往不明确。例如，如果你每天能处理 20 个源项，而某一天你收到了 20 个高优先级和 30 个低优先级的源项，那么未处理的 30 个低排名源项应该怎么处理呢？它们应该自动标记为已读，还是累积起来？或者应该完全丢弃这些源？

当我达到超过 100 个源^([[1](#cite-1)])时，上述问题开始显现，因此我决定重新调整我的策略，更多地朝数学方向发展，这将为分析我的源和管理其增长提供一个逻辑机制。所以这是我的 RSS 策略如何运作的。

在深入了解我的策略之前，我们必须首先提出三个问题：

1.  什么是 RSS 源？

1.  为什么我们要创建我们自己策划的源？

1.  处理源的资源是什么？

在最基本的层面上，RSS 源只是一组源项。每个源项可以被看作是两种可能类型的信息之一：

+   **信号** - 对我们有关的信息。

+   **噪音** - 对我们来说不相关的信息。

我们策划自己的饲料的原因是为了以比通过本地新闻频道背景播放（例如通过主动处理饲料而不是被动获取的形式）更有效的速率积累前者的信息，**信号**。然而，主动处理饲料并非免费，它需要我们花费一种资源：时间（由变量`m`表示）。因此，我们RSS饲料的总体目标通常是在尽可能短的时间内积累尽可能多的信号。考虑到这一点，我们现在已经有足够的定量信息可以开始制定我们的饲料处理逻辑和方程。

一个RSS饲料，作为饲料项目的集合，可以数学上表示为信号和噪音饲料项目的总和：

*图 1\. RSS饲料收集公式*

其中：

+   `t` = 总饲料项目数。

+   `s` = 信号项数。

+   `n` = 噪音项数

仅凭这些变量，我们计算一个指标来代理饲料的质量：**信号比率**，表示饲料中信号占比的分数：

*图 2\. 信号比率*

所以，例如，如果您刷新您的饲料，并且它拉取了10个新项目，其中有2个您感兴趣的项目，那么信号比率是：

*图 3\. 信号比率示例*

在这种情况下，更高的比率表示更高质量的饲料（信号比率为1.0表示每个项目对您都是相关的，因此全部为信号），而较低的比率表示质量较低的饲料（值为0.0表示纯噪音，因此您会丢弃的饲料）。

现在，通过我们衡量的质量衍生，我们需要一个变量来代理我们可以处理饲料的速率。为此，我们将使用变量`r`（饲料处理率），它将表示平均处理单个饲料项目的速率（以时间计量）。例如，假设您刷新您的饲料，获取了20个新项目，并且您可以以每分钟处理10个饲料项目的速率处理饲料，则您处理此饲料所需的总时间为（20 / 10）= 2分钟。有了这些，我们已经具备分析我们饲料的所有逻辑必要条件。

例如，假设：

+   我们有0个饲料订阅。

+   我们每天更新我们的订阅1次。

+   我们每天最多有10分钟来处理所有的饲料项目。

+   我们的饲料处理速率为每分钟5项。

根据这些假设，我们目前需要花费0分钟阅读饲料项目和清理饲料，因此根据我们的时间平衡10分钟，我们目前能够添加价值10分钟（或10 * 5 = 50项价值）的订阅到饲料中。现在，假设我们向饲料中添加一个新的订阅，具有以下特征：

+   新饲料，平均每个地方有5个新的饲料项目。

+   在这5项中，平均有2项是信号，3项是噪音。

给定这个源，我们的订阅表格如下所示：

| 源 | 信号 | 噪音 | 总数 | 时间 | 信号比 |
| --- | --- | --- | --- | --- | --- |
| 和 | 2 | 3 | 5 | 1 | 0.40 |
| 0 | 2 | 3 | 5 | 1 | 0.40 |

这个新源每天的增量处理成本为1分钟，低于我目前可用的10分钟，因此将会将我的信号比从0提高到0.40。因此，从增加信号和整体源质量的角度来看，添加此源是有益的。现在让我们将第二个订阅添加到源中：

| 源 | 信号 | 噪音 | 总数 | 时间 | 信号占总数比率 |
| --- | --- | --- | --- | --- | --- |
| 和 | 3 | 5 | 8 | 1.6 | 0.38 |
| 0 | 2 | 3 | 5 | 1 | 0.40 |
| 1 | 1 | 2 | 3 | 0.6 | 0.33 |

您会注意到这个新订阅的信号比与之前的相比有所降低，从0.40降至0.38，表明总体上反映出源的质量降低。然而，从添加此订阅到源的总时间成本仍然低于10分钟（当前总和为1.6分钟），因此尽管质量有所下降，将此订阅添加到我们的源仍然是有利的。

现在让我们再增加2个订阅，其更新频率和信号比显著不同：

| 源 | 信号 | 噪音 | 总数 | 时间 | 信号占总数比率 |
| --- | --- | --- | --- | --- | --- |
| 和 | 6.03 | 35 | 41.03 | 8.206 | 0.15 |
| 0 | 2 | 3 | 5 | 1 | 0.40 |
| 1 | 1 | 2 | 3 | 0.6 | 0.33 |
| 2 | 0.03 | 0 | 0.03 | 0.006 | 1.00 |
| 3 | 3 | 30 | 33 | 6.6 | 0.09 |

源2全是信号，没有噪音。然而，该订阅更新频率如此低（大约一个月一次），以至于对整个源的信号比影响微乎其微。但是，比率确实还是略有增加，而增量成本如此之低，将其添加到我们的源中将是有利的。

另一方面，尽管源3每天的信号数量最高，但噪音和总数都很高，导致信号比下降超过0.25个点。此外，该订阅目前占用了我们大约80%的时间，并且占用了最大允许时间的66%。尽管如此，将其添加到我们的源中仍然是有利的，因为信号总数增加，处理整个源所需的时间仍然低于10分钟。

最后，让我们再向源中添加1个订阅：

| 源 | 信号 | 噪音 | 总数 | 时间 | 信号占总数比率 |
| --- | --- | --- | --- | --- | --- |
| 和 | 9.03 | 42 | 51.03 | 10.206 | 0.18 |
| 0 | 2 | 3 | 5 | 1 | 0.40 |
| 1 | 1 | 2 | 3 | 0.6 | 0.33 |
| 2 | 0.03 | 0 | 0.03 | 0.006 | 1.00 |
| 3 | 3 | 30 | 33 | 6.6 | 0.09 |
| 4 | 3 | 7 | 10 | 2 | 0.30 |

现在我们遇到了一个问题。新的订阅源改善了我们的信号比，因此逻辑上我们希望将其添加到我们的订阅源中，但它也导致我们超过了最大可用时间（平均花费 10.2 分钟处理订阅源，但只有 10 分钟可用）。那么我们该怎么办？简单，我们放弃信号比最低的订阅源，直到总时间低于 10 分钟^([[2](#cite-2)])。在这种情况下，Feed 3，项目数最多但信号比最差的订阅源被放弃，产生了最终的订阅源：

| 订阅源 | 信号 | 噪音 | 总计 | 时间 | 信号到总比率 |
| --- | --- | --- | --- | --- | --- |
| 总计 | 6.03 | 12 | 18.03 | 3.606 | 0.33 |
| 0 | 2 | 3 | 5 | 1 | 0.40 |
| 1 | 1 | 2 | 3 | 0.6 | 0.33 |
| 2 | 0.03 | 0 | 0.03 | 0.006 | 1.00 |
| 4 | 3 | 7 | 10 | 2 | 0.30 |

这样一来，我们的供给就平衡了；在时间约束下，我们已经最大化了我们的供给质量。现在，如果我们想要重新添加 Feed 3，我们必须执行以下操作之一：

+   增加 `m`。

+   增加 `r`。

+   增加 `s`。

+   减少 `n`。

那么我们如何修改这些变量呢？在实践中，`s` 在短期内基本上是不可改变的，因为我们认为对我们相关的内容通常只在较长时期内发生变化。此外，`m` 在短期内要想不做其他妥协（例如，花更多时间处理订阅源而不是做其他爱好）是很难改变的。这就把 `r` 和 `n` 留作了短期优化的潜在目标。幸运的是，我们可以通过技术解决方案来提高这些值，在长期内是零成本的，允许我们在相同分配的时间内增加信号数。

Thunderbird 是一个流行的桌面邮件客户端，同时还内置了一个 RSS 订阅阅读器^([[3](#cite-3)])。要使用 RSS 功能，您需要创建一个新的 Feed 帐户。点击 `文件 -> 新建 -> Feed 帐户`，然后给帐户取一个名字。在最新的 Thunderbird ESR 版本（截至本文写作时为 115），您应该会看到如下界面：

*图 4\. Thunderbird Feed 帐户*

现在点击 `管理订阅源` 来添加一个新的订阅源。我们将添加 [copyleft NT-based operating system ReactOS](https://reactos.org/index.xml) 和 [extended LTS Debian project Freexian](https://www.freexian.com/index.xml) 的订阅源：

*图 5\. 添加新的订阅源*

*图 6\. 所有 Feed 项目*

撰写本文时，ReactOS订阅有523个订阅项目，Freexian订阅有1168个。订阅非常庞大，但更新并不频繁。然而，为了举例说明，假设每天订阅发送约1700个项目。以这种速率，如果我们的订阅处理速率（变量 `r`）过慢，可能会难以处理所有订阅。鉴于许多计算机用户经常使用鼠标，我决定进行一些快速速度测试，以确定我使用鼠标可以轻松点击项目的速度，最终得出每分钟处理约25个订阅项目的速率。以这种速率，处理整个订阅将需要约（1700 / 25）= ~68分钟。然而，通过使用键盘快捷键，我们可以显著提高我们的速率。Thunderbird中最重要的订阅导航键盘快捷键包括：

+   `n`: 移动到当前订阅中的下一个未读项目（或者如果当前订阅没有更多未读项目，则移动到下一个订阅）。

+   `f`: 前进1个订阅项目（已读和未读都包括）。

+   `b`: 后退1个订阅项目（已读和未读都包括）。

+   `s`: 给当前项目加星标。

+   `Shift+c`: 将当前订阅中的所有项目标记为已读。

使用这些键盘快捷键，我们可以非常快速地浏览所有未读的订阅项目，并标记那些有太多噪音的订阅为已读。通过切换到键盘快捷键，我能够将我的订阅处理速率提高到大约每分钟75个项目，将订阅处理时间减少到（1700 / 75）= ~22.6分钟。这是相当大的收益，在我的经验中，通过键盘驱动的导航是处理订阅项目时可以达到的最佳状态。

现在，优化了我们的订阅处理速率，我们接下来可以做的是减少订阅中的噪音，以提高我们的信号比并进一步减少总体订阅处理时间。为此，我们可以使用消息过滤器自动删除我们不感兴趣的新订阅项目。看一下ReactOS订阅。假设我们只对ReactOS的更新版本订阅感兴趣。在这种情况下，所有非发布项目都是噪音，因此我们希望将它们过滤掉。为此，我们将通过转到`工具 -> 消息过滤器 -> 新建`来创建一个消息过滤器，并添加以下过滤器：

*图7. ReactOS过滤器*

然后点击`立即运行`来过滤ReactOS订阅。

*图8. 消息过滤器菜单*

在过滤器运行后，ReactOS订阅中的反馈项目数量大大减少。在应用过滤器之前，信号比为（32 / 523）= ~0.06；在过滤器应用后增加到1.0（从我们的角度来看，我们已经过滤掉了所有的噪音）。此外，反馈处理时间从约7分钟减少到约0.4分钟。

*图9. 过滤后的ReactOS订阅*

同样地，对于Freexian订阅源，假设我们只对一般新闻感兴趣，而不是包安全更新通知。所有的安全更新通知都以`ELA-`开头，因此我们可以创建一个过滤器来移除这些项。创建过滤器后，订阅源从1168项减少到90项，并且我们显著增加了信号比例（尽管可能不是1.0，因为我们可能会收到与我们不相关的项目有关的订阅项，与ReactOS版本更新订阅项非常具体的范围形成对比）。

*图 10\. 筛选后的Freexian订阅源*

有了这两个过滤器，我们将订阅项数量从1689减少到122，并显著改善了整体订阅源的信号比例。结合我们在处理订阅源方面的效率提升，我们将处理订阅源的时间从大约68分钟降低到约1.6分钟。如果我们每天只有20分钟用于处理RSS订阅源，那么在额外的效率提升后，我们将有约18.4分钟来阅读所有的信号，足以使得使用RSS订阅这些订阅变得真正值得。

所以这是我每天处理我的RSS订阅源的策略。比上述组织方法衍生的策略稍微复杂一些，但是添加了数学和统计推理，为理解订阅源提供了相当坚实的基础，确保订阅源的质量保持高水平，项目数量保持较低，从而降低了长期疲劳的风险。

* * *

1.  ^([^](#ref-1)) 我目前订阅了超过300个源。

1.  ^([^](#ref-2)) 从技术上讲，在这种情况下的最佳解决方案应该是丢弃订阅源1，因为这将把总共花费的时间降低到9.2分钟（低于10分钟），平均信号数量为8（比移除订阅源3多2个）。然而，在实践中，我们每天可能有不同的处理时间，对于大量的订阅源来说，一个单独的订阅源成为新的订阅项的主要来源的可能性较小。因此，信号比率是一个更好的跟踪指标，因为它确保移除订阅源总是增加订阅源质量。

1.  ^([^](#ref-3)) 实际上，它的RSS订阅功能已有十多年历史。
