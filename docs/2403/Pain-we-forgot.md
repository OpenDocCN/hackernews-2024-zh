<!--yml

category: 未分类

date: 2024-05-27 14:32:02

-->

# Pain we forgot

> 来源：[https://www.scattered-thoughts.net/writing/pain-we-forgot/](https://www.scattered-thoughts.net/writing/pain-we-forgot/)

Much of the pain in programming is taken for granted. After years of repetition it fades into the background and is forgotten. The first step in making programming easier is to be concious of what makes it hard. So let's put ourselves in the shoes of a smart but inexperienced end user trying to build, test and maintain a simple application.

Anon 实习生负责管理午餐订单，并很快意识到他们的工作可以由计算机完成：每天上午10点，向每位员工发送一封电子邮件，其中包含一个链接，员工可以通过链接选择午餐内容。中午12点，收集回复并将列表通过电子邮件发送给餐饮公司。每个月末，统计每个人应付的费用并将列表发送给会计部门。虽然这是一个简单的程序，但涵盖了所有基础知识：数据输入、验证、错误处理、计算、呈现、沟通、反应性、调度、部署等。可能有专门的应用程序可以覆盖这个特定示例，但我们更关注的是普通终端用户如何解决这类问题以及他们在此过程中可能遇到的困难。

我们将遇到许多看似不可避免的问题 - 它们是由外部约束强加给我们的。然而，大多数这些约束并不是刻意选择的结果，而是历史的偶然。我们仍然像在1960年编程一样，因为有强大的路径依赖性，这鼓励我们假装你的太空时代计算机实际上是一个80字符的tty终端。我们被困在局部最大值中。

也可以说一旦学会使用这些工具，它们就足够简单了。但我只想指出，实际上，这个门槛太高了。尽管有明显的好处，但世界的绝大部分仍然选择保持文盲。即使是有明显需求的工具（例如版本控制），也基本上未能产生影响。显然，需要一个不那么敌对的编程环境。

相信这是我们能做到的最好，并且编程天然复杂。但是当我们处理午餐应用程序时，请考虑我们实际上要做的工作与指定应用程序的问题有多少关系。

## Getting started

首先，我们需要设置一个开发环境。让我们尝试一下 Clojure：

```
lein new lunch_app cd lunch_app mkdir resources touch resources/index.html LightTable resources/index.html # insert script tag for web repl firefox resources/index.html LightTable clj/lunch_app/core.clj # server side, fire up compiler, connect to repl LightTable cljs/lunch_app/core.cljs # client side, connect to web repl 
```

到了这一步，你已经失去了99%的人口，而我们甚至还没有涉及CSS或模板。更糟糕的是，这一切都是无法发现的。我碰巧已经知道如何设置一个简单的客户端-服务器Web应用程序，所以所有这些步骤对我来说似乎显而易见。但是，实习生安儿需要能够打开Programming™并点击“新建Web表单”。Intellij在这方面做得相当不错——你可以启动一个新的Web项目，在几次按钮点击后编译并在浏览器中打开结果。但在大多数环境中，你需要一个教程来开始一个新项目。

## 寻求帮助

现在，匿名者正盯着空白编辑器页面上闪烁的光标。接下来怎么办？如何制作网页表单或发送电子邮件？对于常见任务，谷歌可能会找到整个代码示例，或者至少是一些Javadoc。这些示例将缺少许多隐含信息，比如如何安装必要的库以及如何处理缺失的依赖项和版本冲突。转录和修改这些示例可能会导致耗费时间的错误。这并不可怕，主要归功于像stackoverflow这样的网站，但这仍然会让人分散注意力，远离手头的任务。

我想只需输入'email'，就能看到与电子邮件相关的函数和库的列表。如果我从自动完成中选择一个函数，它的依赖项应该自动添加到项目中，不需要任何麻烦。丢失的依赖项或版本冲突应该与解决建议一起呈现（点击此处选择版本A）。[必应代码搜索](http://blogs.msdn.com/b/visualstudio/archive/2014/02/17/introducing-bing-code-search-for-c.aspx)将这一理念推向了更高层次，并自动完成常见任务的代码。

## 编写代码

即使对于专家来说，编程也是一个探索过程。我们尝试使用库，运行示例并逐步构建功能。初学者必须学会的最痛苦的教训之一是每个人关于每件事都是多么错误。缩短编写代码和查看结果之间的反馈循环可以减少错误假设造成的损害，减轻追踪应该发生的事情的认知负担，并帮助建立系统的准确心理模型。后者对初学者尤为重要，他们经常对语言的基本语义甚至产生误解。不幸的是，你最有可能得到的就是自动刷新你的浏览器。如果你运气好的话，可能会得到一个REPL。

想象一下电子表格，每次更改都必须打开终端，运行编译器，并扫描打印输出中的单元格/值对，以查看更改的影响。我们不会容忍任何其他工具中这种糟糕的用户体验，但不知何故这仍然是编程工具的最新技术水平。我怀疑很大一部分责任在于我们未能找到像纯文本那样灵活和可组合的 GUI 工具模型。我看到保罗·基乌萨诺关于[杀死应用程序](http://pchiusano.blogspot.com/2013/05/the-future-of-software-end-of-apps-and.html)以及埃斯基尔·斯坦伯格的[Verse](https://www.youtube.com/watch?feature=player_detailpage&v=f90R2taD1WQ#t=1837)的想法有很大潜力。

Light Table至少提供了[内联评估](https://www.youtube.com/watch?v=gtXpOD6jFls)、[监视](https://www.youtube.com/watch?v=d8-b6QEN-rk)和[instarepl](https://www.youtube.com/watch?v=YY6B9EHbH24)。像[调试模式是唯一模式](http://gbracha.blogspot.com/2012/11/debug-mode-is-only-mode.html)和[示例中心编程](http://www.subtext-lang.org/OOPSLA04.pdf)这样的思想将此类交互推向更前沿。与单独的编辑器、编译器、REPL、调试器等不同，您通过在调试器中编辑代码来开发一切。这类似于JIT编译器的思想——IDE在运行时拥有比编译时更多的信息，因此可以做出更好的决策并提供更好的反馈（例如，在编写函数时生成示例输入和输出）。

纯文本也非常有限。语言很擅长传达意义，但在显示数据方面不太好。能够快速展示图表和图示（比如在[rhizome](https://github.com/ztellman/rhizome)、[automat](https://github.com/ztellman/automat)或[lamina](https://github.com/ztellman/lamina/wiki/Channels)中）非常有用。Light Table的[内联图表](http://www.chris-granger.com/images/040/ipygraphs.png)是个开始，但我们在可视化方面的利用还不多。

## 运行代码

令人惊讶的是，我们从初学者那里听到的最常见困难之一就是运行代码。即使我们把整个 lunch_app 的源代码交给 Anon，他们可能仍然难以真正使用它。他们必须安装依赖项、编译代码、启动服务器和打开端口。每一步骤中的错误都很难诊断并且耗时修复。旨在解决这些问题的工具通常自身也更糟（每次我在 octopress 中写博客文章时，我发现 rvm 总是再次出问题）。像 Intellij 和 Visual Studio 这样的 IDE 做了合理的工作来标准化构建过程和捕获依赖项，因此通常可以导入一个项目并且只需点击运行，但这只适用于开发阶段。对于部署，我们有 Docker 这样的工具使得部署高度可重复，但在捕获流程方面并没有太多帮助。这些工具都不会真正帮助 Anon 实习生部署 lunch_app。

午餐应用程序也需要计划任务、错误记录和监控。如果邮件发送失败或者没有回应，Anon 需要被警告。即使是设置最简单的日志记录、监控和重新启动对专业程序员来说也是一件麻烦事。

Wolfram 的语言[工作流](https://www.wolfram.com/universal-deployment-system/)非常接近理想状态。你可以在一个笔记本中编写代码，代码可以即时运行和更新，没有手动编译的步骤。部署到云服务器只需要一个函数调用，它会自动收集代码和依赖项，并返回一个 URL，你的程序现在正在运行。无需考虑文件或库，没有项目文件，没有构建产物，也不需要设置服务器和打开端口。

从这里想象一下，添加简单的任务调度和错误记录器不需要多少想象力，当出现问题时会给 Anon 发送电子邮件。这些都不需要放弃控制权。你也可以轻松地用 '部门服务器' 或 '我们互联网附带的小黑盒子' 替换 '云服务器'。重要的是部署有合理的默认设置，并且在语言或IDE中是 '即插即用'。

## 什么？

我们可以询问我们的应用程序的最简单问题是“当前状态是什么”。奇怪的是，很少有编程环境在这方面给你任何帮助。许多程序员只能靠打印语句。如果你幸运的话，可能会有调试器或观察点，但你仍然通过钥匙孔看你的应用程序。你必须手动插入仪器来查看应用程序每个小部分的状态。如果要修改该状态，您必须逆向思考并构造正确的代码片段来查找和更改正在查看的变量。它甚至可能没有一个可以从repl访问的名称（例如，匿名闭包中的变量）。查看和修改应用程序的状态应该是一种基本的交互，但由于我们的语言和工具，这变得不合理困难。

与诸如Excel或[Django管理](http://www.youtube.com/watch?v=kvFDV1oM-ZA)之类的工具相比，*所有*数据都可以轻松浏览，用户无需任何主动操作，只需点击和输入即可*直接*修改。工具本身并不难，但需要重新思考我们在编程语言中管理状态的方式。所有主流语言，无论采用何种范式，都鼓励[匿名本地状态](/writing/local-state-is-harmful/)，这些状态无法轻松观察和修改。

一旦我们管理了状态，无论是使用像[Bloom](http://bloom-lang.net/)这样的关系模型，还是更传统的函数数据结构像[Opis](http://infoscience.epfl.ch/record/136776)，我们也可以轻松记录历史。像[时间旅行调试器](http://chrononsystems.com/products/chronon-time-travelling-debugger)这样在传统语言中需要大量工程工作的工具，在你可以便宜地记录或重建过去时变得微不足道。当你可以快速获取各种组件如何通信的视觉概述时，从源代码中确定数据流拓扑结构的工具像Bloom和Opis也都能做到（示例分别隐藏在[这里](http://db.cs.berkeley.edu/papers/cidr11-bloom.pdf)和[这里](http://infoscience.epfl.ch/record/136776/files/DagandETAL09Opis.pdf)）。

## 为什么？

传统的调试器完全专注于*what* - 一次只能走过一小部分状态。但调试时通常先问的问题是*why*？为什么午餐选项顺序错了？午餐邮件为什么没有发出？为什么每个人这个月的账单都是零？这些问题通常涉及从效果到原因的逆向推理，而调试器则是从原因到效果的前进。结果是，调试主要不是找问题，而是通过设置孤立的测试用例并在调试器中重复运行它们来手动沿着原因链向后走。

[Theseus](http://blog.brackets.io/2013/08/28/theseus-javascript-debugger-for-chrome-and-nodejs/)通过捕获每个事件回调入口处的参数稍微改进了这一点，因此您无需重复运行。Ko和Myer的[因果调试器](http://repository.cmu.edu/cgi/viewcontent.cgi?article=1165&context=hcii)通过追踪每个状态变化的原因树来明确回答“为什么”和“为什么不”的问题，因此从效果到原因的反向行走过程完全自动化，您可以专注于找出问题出在哪里。

随着规模的扩大，问题变得更加严重。在大型系统中，通过跟随控制流进行调试效果不佳，真正重要的是*数据流*。回答诸如“为什么订单有时会丢失”这样的问题需要追踪一个庞大的图形，大多数系统甚至没有记录这种图形，而是必须从日志中推断，就像从打破的陶器中拼凑古代文明一样。[BOOM分析](http://db.cs.berkeley.edu/papers/eurosys10-boom.pdf)通过将所有数据反映到关系表中来处理这个问题，包括错误日志、持久数据、消息队列和分析数据，这些表可以供overlog（系统中运行的分布式查询语言）使用。这意味着您可以直接在因果图上运行查询，例如“对于每个已输入但未完成的订单，请列出通过一系列规则与该订单相关联的每条消息”。由于这些数据的记录本身受到overlog规则的控制，您可以在运行时为特定类型的数据（例如“记录所有关于由集群C发出的订单197的消息并将其转发给我”）打开详细日志记录。

## 变更

在软件世界之外，版本控制和协作软件仅限于保存lunch_app.v07并将其附加到电子邮件中。在单一项目上进行协作是困难和缓慢的。程序员的标准工具（如git、mercurial等）则更加强大，并解决了一个紧迫的问题，但它们的界面[让许多用户感到困惑和沮丧](http://steveko.wordpress.com/2012/02/24/10-things-i-hate-about-git/)。其基础模型优雅而强大，但即使是图形界面也需要投入大量时间和精力才能理解。

匿名需要的是介于[undo-tree](http://www.emacswiki.org/emacs/UndoTree)（没有ASCII艺术）和[etherpad](http://etherpad.org/)之间的地方。更改应自动记录，并（可选地）用提交消息进行后期标记。实时协作应该只需点击同事的面孔就可以简单实现。撤销更改和查看不同版本只需在[提交图形](https://github.com/blog/39-say-hello-to-the-network-graph-visualizer)上移动即可。将代码片段从编辑器中拖出应生成指向该代码的VCS页面的链接。如果编辑器理解代码的结构，我们甚至可以跟踪单个代码单元的语义更改（例如重命名函数、重新排序表达式），而不是比较文件中的文本，这样自动和手动合并都更容易，因为我们有更好的变更意图记录。

同样地，当匿名2号的会计师想要修改他们客户端的午餐表单以记住他们最喜欢的午餐时，这应该是一个简单的过程。不需要追踪和重新编译系统二进制文件，也不需要从文件系统安装greasemonkey脚本。只需点击打开源代码，根据你的喜好进行修改，点击保存即可。我从未见过任何接近这种基本互动的东西。[OLPC查看源代码按钮](http://blog.printf.net/articles/2006/10/29/the-view-source-key/)承诺了完全相同的体验，但据我所知从未实现过（它在我的设备上肯定没有工作）。

## 学习

编程工具通常很少关注生成有帮助的错误消息（与[一个](http://cs.brown.edu/%7Esk/Publications/Papers/Published/mfk-measur-effect-error-msg-novice-sigcse/paper.pdf)或[两个](http://research.microsoft.com/pubs/64590/helium.pdf)例外）。有些[证据](http://www.amazon.com/Man-Who-Lied-His-Laptop-ebook/dp/B003YUC7BI/ref=sr_1_1?ie=UTF8&qid=1400099030&sr=8-1&keywords=man+who+lied+to+his+laptop)表明，人们与计算机的交互方式就像对待人一样。这些研究结果中的许多都是令人惊讶和反直觉的，例如[使编译器拟人化](http://faculty.washington.edu/ajko/papers/Lee2011Gidget.pdf)可以提高学生的学习效率。鉴于此，你真的想花很多时间与那种在你面前不断喊着“无法调用未定义的未定义方法！”而没有任何提示如何解决问题或从何处开始查找的人打交道吗？

我们的编程环境非常具有敌意。界面要么详细到令人不知所措，要么隐藏了一切。大多数操作无法撤销（例如更改变量、定义函数、安装库）。运行时默认在未捕获异常时退出，丢弃了解决问题所需的所有上下文，迫使用户试图重新创建崩溃状态。当任何操作都可能导致混乱和工作破坏时，经验不足的用户会因恐惧和犹豫而无法实验。这削弱了他们学习的能力。

错误消息至少应该指出可能导致错误的原因，并最好提供修复选项。例如，Intellij会突出拼写错误并建议更正（"did you mean..."）。良好的终端用户应用程序会将常见错误链接到常见问题解答页面。建议不必完美，但必须准确。大家都讨厌 Clippy，因为它的建议毫无用处、重复且缺乏上下文。黄金法则是，如果没有有用的东西可说，请不要说什么。一个雄心勃勃的项目（参考文献？）通过众包收集了造成类型检查错误及其解决方案的示例。大规模数据收集和测试可能是提供有用反馈的最佳途径。

环境也需要更加主动。未捕获的错误应该将用户带入调试器，用户可以[编辑并继续](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html)。编辑器可以发现常见错误并建议更正（Intellij 在这方面非常擅长，[kibit](https://github.com/jonase/kibit) 也不错，但很多人仍在使用连拼写错误和变量屏蔽都不提示的编辑器）。分析器可以启发式地探索瓶颈并建议解决方案（例如，如果 foo 被索引了，此查询将快10倍）。与其依赖用户自己创建测试，我们可以提示他们提供示例和不变属性，并[搜索反例](http://www.scs.stanford.edu/11au-cs240h/notes/testing-slides.html)。[Opis](http://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=0CD0QFjAC&url=http%3A%2F%2Finfoscience.epfl.ch%2Frecord%2F136776&ei=gNpzU7yeKMTesASLr4KoCA&usg=AFQjCNGyqXOAavdVfGxGuBFZbTzobRmXtQ&sig2=jvF_qeyNTyCOffdJkC-twA&bvm=bv.66917471,d.cWc) 配备了一个自动估算模型中每个函数渐近复杂度的分析器和一个有限状态模型检查器，可以通过高效且详尽地检查每个可能的状态来证明不变量始终成立。Bloom 提供了一个[生成测试框架](http://db.cs.berkeley.edu/papers/dbtest12-bloom.pdf)，使用 SMT 求解器高效地探索可能的空间，并提供了一个[静态分析插件](http://www.bloom-lang.net/calm/)，用于警告分布式程序中未利用的协调点。你的 IDE 是否能为你运行单元测试？

最后，环境不能是黑盒子。初学者需要简单的体验，但如果他们要成为专家，他们需要能够脱掉训练轮并打开引擎盖。许多面向最终用户的编程尝试失败，因为它们假设用户很愚蠢，所以将一切包裹在棉花中。每当我们提供简化的体验时，应该有一种简单的方法来打开它并了解其工作原理。没有什么东西能永远是魔法。确保用户的好奇心永远不被挫败，他们就不需要长时间教导。
