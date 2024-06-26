<!--yml

类别：未分类

日期：2024-05-27 15:03:50

-->

# “无用的 Ruby 糖”的结局：关于直觉和演变

> 来源：[`zverok.substack.com/p/the-end-of-useless-ruby-sugar-on`](https://zverok.substack.com/p/the-end-of-useless-ruby-sugar-on)

九月底，我回到了——让我们这样说——乌克兰的更为平静的地方。一个 Reddit 讨论激发了我开始写我预计将是一篇简短博客文章。

这次讨论是关于最近版本中的新的 Ruby 语法元素，像往常一样，很多参与者表达了各种不满。我不太关心“我喜欢/我不喜欢”这种讨论，但我经常发现自己被“我看不到任何解释为什么这可能是个好主意”或“这完全违背了我在 Ruby 中喜欢的一切”的话题吸引。

尽管这种说法可以——而且经常——只是另一种说“我不喜欢它”的方式，但我的脑海将其视为一种挑战：是否能理解并解释变化背后的推理？解决问题的设计空间是什么？

这就是“无用的 Ruby 糖”诞生的方式：作为寻找语法变化一致性解释的尝试，并分析这些变化是否具有深远影响。

> “语法糖”是另一个广泛使用的术语，以贬低的方式标记语法元素，暗示其唯一作用是节省几次按键，代价是使语言逻辑复杂化。名称选择为一种讽刺的标志，但我不得不遗憾地承认，一些读者把它当作字面意思（有时是欢呼“终于说出真相”，有时是争辩）。

当我开始写“那些无用的 Ruby 糖”时，我想象它是由简明的部分组成的简短博客文章（每个段落一两个），专门介绍了几种最新的语言变化，并调查它们的来源、设计方式以及它们如何改变代码的感知。

**意图并不是“为了捍卫”某个特定功能或整个语言。** 我想诚实地分享我的*视角*，我的表达方式以及将其融入我的思维模型中的方式。

我决定选择的每个语法特性（匿名块参数、模式匹配、参数转发等）都可以从相同的角度来讨论，从引入原因到后果和代码风格的变化，并与其他编程语言中类似的设计进行比较。

一个“小帖子”变成了“一个非常大的帖子”，然后成为在两个月内发布的一系列帖子（其中一个主题——模式匹配——分形地变成了[三个](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html) [连续](https://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html) [帖子](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html) ——相信我，我在祈祷当前文本不会变得太长以致于成为一篇单独的最终文章！）

尽管我低估了写作量（花了我大约两个月的每周帖子），但我还是有点喜欢这样做。看起来，它似乎吸引了一些感激和细心的读者（谢谢你们！）。对我来说，这不是关于特定语言特性的问题。而且，在某种程度上，也不是关于特定语言（尽管这成为了论点的一个重要支柱）。

我从一个稍微扩展的版本开始，其中包括我经常与同事分享的理解、猜测和记忆语言行为的“经验法则”。最终，这些文字演变成了一些“思维框架”：看待一种生动语言不可避免的变化的方式——它的原因和可能的实现设计空间，其后果和影响。

这个框架的基本属性是将编程语言视为首要的写作和阅读工具。我总是试图看到其演变背后的推动力，即更好地表达意图的意图。

这显然不是看待编程语言的唯一方式，但似乎对于这个系列来说足够有趣和富有洞见，以至于获得了一些适度的成功和大量的评论。这些单独的文章经常引发有趣的跨语言讨论，其中大多数参与者并非母语是 Ruby 的人。

虽然已经相当于一本小书，但这个系列有一个非常狭窄的前提：只是从广阔的语言宇宙中的一个角度看待其中一个语言的几个变化。不过，我认为其中两个主要的“推动力”对于 Ruby 以外的语言也是重要且相关的。

我认为这种推动力对许多主流语言和风格都是共通的，即从“经典 OO”风格的程序结构（和构想！）向“小的不可变对象 + 函数式算法”风格的“构造型变革”。显然，这并不是最新鲜的消息（整个 Go 和 Rust 这样的语言已经可以被认为是主流语言，它们的基础与“经典 OO”相去甚远）；但看到这些成熟的语言如何适应这种变化还是很有趣的。

变革的第二个推动力更为持久：这是对语言词库——任何语言的词库——的自然扩展，以应对以前不那么简单和复杂的事物变得日常化的趋势。曾经需要解释和教育的概念现在成为你提到一次就能理解并希望用一个词表达的东西。几十年前，并非每种编程语言都能正确处理字符串（甚至更不用说 Unicode 字符串）。如今，像 lambda、sum types 或 pattern matching 这样的东西几乎成了表达日常思想的必备元素。基线不断变化。

我在 Ruby 的上下文中讨论的许多变化更具普遍性：大多数广泛使用的语言继续调整以适应*我们正在思考的内容*的变化景观。这似乎是每种语言社区的一部分的普遍抵制：来自那些思考过程在其他层次进行的人的抵制，他们认为语言的结构应该保持稳定，以不“分散”已经形成的思想的实现，而不是引入新特性。

每种语言的演变可能都是这两种力量之间的一种妥协。

虽然我希望这系列文章中的一些观察和结论对更广泛的受众有好奇心，但我仍然在 Ruby 中思考和写作。

有人可能会说，Ruby 是我看待问题的“镜头”，既包括实用层面（编写代码以理解某些东西），也包括元层面/哲学层面的看待语言如何调整以更好地理解世界。

Ruby 的重要之处，既是它的诅咒也是它的祝福，**它并不是一门有趣的语言**。没有特别的“花招”或机智命名的概念，所有人都试图在其他语言中模仿（嗯，我认为 Ruby 引入了`map`等函数迭代器进入主流，但它们并非在这里发明）。

我相信这门语言的前提大致是“将现代思想与一定数量的基本概念结合在一起，尽可能清晰和表达，而不变得奇特”。

*我认为这门语言在 2000 年代赢得的“怪异和古怪”的声誉，无论是在其粉丝还是反对者中间，更多地反映了当时行业的状态。在那个行业中，只需一行代码就能完成“应该”需要一个类和一个包和几个方法和初始化和类型定义的工作——当这种愿望单独使开发人员看起来“不认真”时。*

在许多方面，Ruby（受 Smalltalk 启发，内部“几乎所有东西都是对象交换消息”）从一小部分核心概念中实现了显著的清晰度和一致性——但并不小到成为一个“语言构造器”而不是一个语言。

多年来，它设法保持*相当现代化*，而不失去内部一致性和可发现性（当然，某些 Ruby 爱好者可能会有强烈不同意见：我认为系列文章的某位评论者说过“我喜欢 Ruby 但自 2.0 以来没有什么好事情发生”——而 2.0 已经十年了）。

当然，并不是所有语言的最初设计决策和积累的权重都是“最佳”或可以证明最好的。

我可以列举很多我个人不太喜欢的事情。本文的早期草稿计划有一个部分专门讨论它们——但我最近对逐级扩展的文章产生了一种非理性的恐惧！虽然，举几个例子：

+   文本中的`require`和模块系统本可以更加清晰；作为间接结果，Ruby 社区最终采纳了“自动加载”的概念，虽然[当前的自动加载解决方案](https://github.com/fxn/zeitwerk)非常好地工程化了，但我个人不太相信这种方法；

+   虽然“代码块”即`Proc`是引入高级函数的机智方式，但 Ruby 中当前的“可调用对象”情况（非一级`Method`对象，`call`方法处理的半约定）使其不如可能的强大；

+   标准库的不均匀和令人困惑的支持情况（我最喜欢的例子是`Date`/`Time`类型的情况，其中`Time`是一个核心对象，`Date`大部分是标准库，主要用于科学计算，它们之间存在许多小的差异）；

+   （我可以继续）。

最好的事情仍然是一个相当小的核心概念，开发团队努力将任何新功能“因子化”为这些概念，并且灵活的语法对于使核心变小至关重要（比如能够在不加括号的情况下调用方法，使`foo.bar`这样的属性语法成为可能，并且消除了需要有单独的“方法”和“属性”/“获取器”概念的必要性）。

这些特征使 Ruby——依然——非常适合开发*领域特定词库*：语言的新的、任务特定的部分能够与其余部分自然地整合，并且能够使“在领域中思考”变得更加容易。我会将其与“领域特定语言”的概念区分开来，后者通常意味着一种*新的*表达系统，其逻辑未必与实现语言对齐。

> 我认为 Rails 是/是一个很好的示范，展示了 Ruby 在开发“领域特定词库”方面的自然优势。我只是有时候希望它能被当作许多其他项目开发的一个例子/灵感。相反，Rails 成为了 Ruby 的*标志*，在某个时刻，它的流行使 Ruby 进入了“开发和支持大型系统”的领域，其声誉和演进受到了这些新情况的严重影响。

尽管如此，对我而言，Ruby 仍然是一个用于尝试新事物和探索新思想的绝佳*建模黏土*。

***请在这里停一下。** 这是您定期的中文提示，我是来自乌克兰的活生生的人，有一些相关的有用信息。*

***关于新闻。** 通常，我会在这里放一个“新闻项目”。然而，说实话。很难选择自从我的[上一篇文章](https://zverok.space/blog/2023-12-28-advent-of-changelog-fin.html)以来发生的事情。我们发生了很多事情。一位杰出的年轻诗人[在前线被杀害](https://twitter.com/PenUkraine/status/1744075793855029721)（在这里，我[翻译了](https://twitter.com/zverok/status/1744416917392179531)他的一些诗）。在新年期间发生了大规模的导弹/无人机[轰炸](https://twitter.com/United24media/status/1742224719011594673)，包括在我家乡（自那以来一直经常遭到炮击；比如，上周一枚导弹[击中](https://twitter.com/FeministsUa/status/1747695174602420381)了一家私人妇科诊所，而前一周是一家[酒店](https://twitter.com/United24media/status/1745340529699696643)，那里有外国记者）。这甚至[没有](https://twitter.com/den_kazansky/status/1743684511320133876)提到在靠近前线的较小城市中对平民的[日常](https://twitter.com/United24media/status/1747896573990781438) [袭击](https://twitter.com/United24media/status/1748633966255686091)。我想知道这些在我的读者国家的主流媒体中有多少报道。*

***一个背景。** 今天（我完成这篇文章的日期是 1 月 20 日）是[顿涅茨克机场保卫者纪念日](https://twitter.com/DefenceU/status/1748647821803245895)。他们在 242 天内保卫了机场，这阻碍了俄罗斯通过该机场转移部队的计划，并防止了在 2014 年至 2015 年间失去更多的乌克兰领土。是的，俄罗斯当时已经在发动战争。*

***一个请求。** 如果你在美国，请打电话给你的代表！在华盛顿特区，你也可以参加[乌克兰集会](https://twitter.com/ukrainerallydc)，即使只有很短的时间。*

***一个筹款活动。** 你可以捐款给[UA 救护车](https://www.ambulanssi.org/en)倡议，该倡议购买救护车辆用于在前线附近执行撤离任务。*

虽然这个系列，如它被创造出来的那样，已经结束了，但有很多有趣的事情留给我去思考，至少对我来说是这样。也许其中一些事情会变成未来的文章！

一个思考方向是更深入地探讨“可读代码”的后果：我认为我对比较有趣的角度有一些想法——从“软件项目作为杂志”的隐喻（在[上一篇](https://zverok.space/blog/2023-12-01-syntax-sugar5-endless-methods.html)中略有涉及）到思考“代码页面”意味着什么（这个系列的重复动机），或者使用我们目前的知识对旧概念“文学编程”进行现代化处理。

另一个有前途的方向是将几种不同编程语言在同一领域所做的设计决策进行对比：在我涉及的少数语法特性中进行了比较后，我现在相信这是理解一些编程概念、它们的设计空间以及某些特定语言为了采纳它们所做出的选择和牺牲的强大工具。

总的来说，我对“语言直觉”这个概念以及其他半意识到的力量思考很多，这些力量使我们认为某些代码“好”或“坏”，选择解决问题的方法和方法，并随着时间的推移改变我们的观点。试图找出我们为什么有这些直觉和信念，以及揭示、挑战和正式化它们，似乎是一个有趣的话题。（尽管可以争论，这比“只需按照我的方式编写软件，你就会很好，完了”要少一些“市场价值”。）

毕竟，这不是一门语言研究；这是关于我们如何思考和写作的研究。至少，这是我诗人的心灵告诉我的。

“直觉”在我理解编程语言的语境中是一个重要的词（仿佛有人还没注意到！）。正如我在系列文章的开头所述：

> 我相信，一个设计良好、一致的语言对“通常如何做事情、这些元素组合可能意味着什么、这个特定名称与什么相关”等方面有很多隐含的*直觉*知识。获得这些直觉会使阅读和编写遵循它们的代码变得*流畅*。

自然地，“Ruby 直觉”成为了我计划在 2022 年写的书的名字。

写一本关于试图揭示 Ruby 的这种隐含知识并对其进行反思的语言的书似乎是个好主意。实际上看起来仍然是一个好主意。然而，这本书的想法变成了我“大白鲸”的东西：我需要很多运气、时间、专注和经验来处理它，好吧——看起来这个时机还没到！

但是，在撰写系列文章的同时，我想到了另一本更简单（希望如此！）的书的主意，这本书将使我能够将多年来对理解语言、记录它并观察其变化的工作汇集到一篇文章中。

我想出的标题只是“Ruby 进化”，目前，在我的脑海中，它看起来像是我 [Ruby Changes](https://rubyreferences.github.io/rubychanges/) 网站和诸如此类的文章的一个合理交叉：按版本/主题有序讨论最显著的变化，其背后的原因，以及它如何改变了语言。

我希望像这样的一本书对 Ruby 程序员和其他语言社区的人都会感兴趣，处于一个普遍的“思想史”背景下。

目前，我正在早期阶段思考这个想法，并草拟目录。但我想知道是否会有人对这样一本书感兴趣？

与此同时，我将写一些其他的话题，或多或少与 Ruby 有关，或者至少与编写代码有关。

***你可以订阅我的[Substack](https://zverok.substack.com/)，或者在[Twitter](https://twitter.com/zverok)上关注我。***

* * *

谢谢你的阅读。请用你的捐款和游说来支持乌克兰的军事和人道主义援助。在[这里](https://war.ukraine.ua/)，你会找到全面的信息来源和许多链接，这些链接指向接受捐款的国家和私人基金。

如果你没有时间处理这一切，捐款给[Come Back Alive](https://savelife.in.ua/en/)基金会总是一个不错的选择。

如果你觉得这篇文章（或者我之前的一些作品）有用，我现在有一个[Buy Me A Coffee 账户](https://www.buymeacoffee.com/zverok)。在战争结束之前，支付给它的所有款项（如果有的话）都将用于我或我兄弟的必要装备，或者寄给上述基金会之一。
