<!--yml

category: 未分类

日期：2024-05-27 15:01:43

-->

# 作为 Emacs Lisp 的另一种模式匹配条件 [LWN.net]

> 来源：[https://lwn.net/Articles/961682/](https://lwn.net/Articles/961682/)

| **LWN.net 需要您的支持！** 没有订阅者，LWN 就无法存在。请考虑[订阅](/subscribe/)并帮助维持 LWN 的出版活动。 |
| --- |

作者：**Jake Edge**

2024年3月1日

Emacs Lisp（Elisp）中使用 Common Lisp 特性的（极度）长时间讨论的结果之一，我们在十一月份[曾经看过](/Articles/951090/)，是开始从 Emacs 中移除一些这些使用。Emacs 中使用 Common Lisp 库（cl-lib）的 Elisp 的一些重写工作，由[理查德·斯托曼](/ml/emacs-devel/E1qvq5s-0003LC-O6@fencepost.gnu.org/) 开始，旨在减少维护 Emacs 所需的认知负荷。从那时起，他扩展了简化 Elisp 的工作，通过添加一个新的[模式匹配条件](https://www.gnu.org/software/emacs/manual/html_node/elisp/Pattern_002dMatching-Conditional.html)，这将与 [`pcase`](https://www.gnu.org/software/emacs/manual/html_node/elisp/pcase-Macro.html) 竞争，后者是他认为过于复杂的长期宏。

#### 复杂性

回到十一月中旬，斯托曼[指出](/ml/emacs-devel/E1r3SgH-0007Rb-BU@fencepost.gnu.org/)，他发现 `pcase` 定义的 "<q>小语言</q>" "<q>如此简洁，简直晦涩难懂</q>"。他认识到，尝试通过结合更简单的 Elisp 构造，如 [`cond`](https://www.gnu.org/software/emacs/manual/html_node/elisp/Conditionals.html#index-cond) 和 [`let`](https://www.gnu.org/software/emacs/manual/html_node/eintr/let.html)，解决相同集合的问题是 "<q>冗长且笨重</q>"，但 `pcase` 已经把简洁性的欲望推到了不合理的极端。他说，这给所有必须维护使用 `pcase` 代码的 Emacs 开发者带来了成本，因此他决定在其他结构中适应一些 `pcase` 特性。

可预见的是，这引发了一系列长篇讨论——对于 emacs-devel 邮件列表来说是标准操作——讨论是否需要 `pcase` 的替代品，可能的替代品是什么样子，等等。斯托曼[开启了一个新的子线程](/ml/emacs-devel/E1r4Bci-00043P-VJ@fencepost.gnu.org/) 来研究他对一个新宏 `cond*` 的想法，这将提供一个更简单的模式匹配构造，仍然比使用 "<q>老式 Lisp</q>" 更为简洁。他的新宏旨在结合条件形式 `cond`，处理多个不同值的检查——类似于其他语言中的 [switch 语句](https://en.wikipedia.org/wiki/Switch_statement)，以及 `let`，在有限的作用域内临时绑定变量。`pcase` 和 `cond*` 都旨在为 Elisp 提供一种[ML风格的模式匹配条件](https://en.wikibooks.org/wiki/Standard_ML_Programming/Expressions#Case_expressions_and_pattern-matching)机制。

一个简单的`pcase`示例可能会展示这些结构的一般风格，但它们可以做的远不止这些，包括模式匹配和拆解列表及其他数据结构，这被称为“解构”。这个示例来自`pcase`文档，处理了多种不同类型（例如字符串、符号）的返回码，为每种情况生成适当的消息：

```
    (pcase (get-return-code x)
      ;; string
      ((and (pred stringp) msg)
       (message "%s" msg))

      ;; symbol
      ('success       (message "Done!"))
      ('would-block   (message "Sorry, can't do it now"))
      ('read-only     (message "The schmilblick is read-only"))
      ('access-denied (message "You do not have the needed rights"))

      ;; default
      (code           (message "Unknown return code %S" code)))

```

Stallman 将他的方法描述为试图“<q>避免 pcase 的 '什么都有的铃铛和哨子' 方法的笨拙</q>”。自然地，这导致了对`pcase`的进一步争论。例如，Michael Heerdegen [说](/ml/emacs-devel/878r6u3s7f.fsf@web.de/)：“<q>在我看来，`pcase`非常接近其任务的最佳解决方案。</q>”

到了十二月中旬，Stallman 开始询问关于`cond*`所需功能的问题“<q>以便几乎不会遇到无法干净替换的 pcase</q>”。这场谈话发生在原始`pcase`替换讨论的[另一个分支](/ml/emacs-devel/E1rF4wi-0006Q3-Ne@fencepost.gnu.org/)中，这使得跟进起来稍显困难。在讨论串的这一部分中，其他人与 Stallman 共同努力推动`cond*`；再次，围绕这个功能，有许多有用的建议和澄清查询，还夹杂着一些抱怨。不过，显然 Stallman 在推进这个功能上取得了进展。

十一月中旬，Alan Mackenzie 提出了一个问题，似乎自`pcase`自2010年[添加](https://git.savannah.gnu.org/cgit/emacs.git/commit/?id=d02c9bcd096c44b4e3d5e2834c75967b56cdecdd)以来一直存在：文档问题。他指出了他在2015年发表的[帖子](https://lists.gnu.org/archive/html/emacs-devel/2015-12/msg00741.html)，描述了`pcase`文档中存在的问题；当时已解决了其中许多问题，但目前仍在进行一项由 Jim Porter 主导的[持续努力](/ml/emacs-devel/19a64b7a-cec0-ed8a-d413-096451cc7413@gmail.com/)，以改进这个宏的文档。

Porter 列出了需要注意的多个领域，包括移动反引号的呈现（“```") operator, which is used for pattern-matching and destructuring, earlier in the `pcase` [doc string](https://www.emacswiki.org/emacs/DocString). As Mackenzie had noted, one of the confusing things about `pcase` is its use of two punctuation marks, backquote and comma ("`,`"), that already have established uses in Elisp macro definitions: "<q>pcase complicated the meaning of ` and ,. Before pcase these had definite meanings. Afterwards, they became highly context dependent.</q>"

There were a few responses to Porter's message, largely in favor of his ideas; eventually, Emacs co-maintainer Stefan Kangas [copied the post](/ml/emacs-devel/CADwFkm=aZB29-Jc_WGBWj+SdwrQJ1KRCqvJUPgaqR=JpWDLVGA@mail.gmail.com/) to Stefan Monnier, who is a former Emacs maintainer and the developer of `pcase`. Monnier was [generally in favor](/ml/emacs-devel/jwvbkaejfzz.fsf-monnier+emacs@gnu.org/) as well; he thought that the doc string was not really the place for detailed backquote information, though some reorganization made sense. Stallman [took exception](/ml/emacs-devel/E1rILvo-0006PN-Ka@fencepost.gnu.org/) to Porter's other suggestion to possibly mention `pcase` in "[An Introduction to Programming in Emacs Lisp](https://www.gnu.org/software/emacs/manual/html_node/eintr/index.html)". "<q>We should not encourage people learning Emacs Lisp to use pcase.</q>"

#### First draft

In mid-January, Stallman [posted a first draft of `cond*`](/ml/emacs-devel/E1rQJDv-0001UG-2E@fencepost.gnu.org/), asking for more testing, "<q>constructive comments, bug reports, patches, and suggestions</q>". Andrea Corallo [asked](/ml/emacs-devel/yp1ttnave7z.fsf@fencepost.gnu.org/) about one particular feature of `cond*`:

> what is the reason for some of these cond* clauses to keep the binding in effect outside the clause itself and for the whole cond* construct? At first glance it doesn't look very idiomatic in Lisp terms to me.

Corallo is referring to the `bind*` sub-clause that can appear anywhere in the body of the `cond*` to bind variables with a scope that does not end with the `bind*` that contains them. Instead, the scope of those variables is the body of the `cond*` from that point onward, which is a bit of an oddity from the usual expectation in Lisp.

```

    `(cond*`

        `(CONDITION FORM)`

        `((bind* (x 42)))` ；将 42 绑定到 x

        `(CONDITION-using-x FORM-using-x)`

        ...)

```

根据 João Távora 的[注释](/ml/emacs-devel/CALDnm51H-3vPJLEm40YbX5K559O7mRTxeXg5hG0no73y2Wu4gw@mail.gmail.com/)，其他人已经询问过这种行为，但他“<q>不确定最终我们是否澄清了这一点</q>”。他还想知道是否可以使用`pcase`来构建`cond*`；如果它是`pcase`功能的严格子集，这样做可能有所帮助。“<q>实际上可能有助于 'cond*' 的采用路径（尽管这条路径对某些方面仍然可疑）</q>”。但 Stallman [并不这样看](/ml/emacs-devel/E1rRO8o-0001R8-Mx@fencepost.gnu.org/)；他列举了他认为`cond*`具有的优势，并表示计划将这个新宏添加到 Emacs 中：

> cond* 在四个基本方面超过了 pcase：创建覆盖其余部分的绑定，匹配不同数据对象的模式（不一定是相同的对象），将普通的 Lisp 表达式作为子句中的条件，以及[能力]在绑定后继续执行进一步的子句。
> 
> 我将进行更多测试，然后安装 cond*。

Adam Porter [要求](/ml/emacs-devel/86a97153-94eb-4df4-b135-4127d2a00057@alphapapa.net/) Stallman 重新考虑将 `cond*` 安装到 Emacs 中，建议他考虑增强 `pcase` 而不是添加一个完全新的工具，开发者需要学习。关于 `pcase` 的一个抱怨是它学习起来很繁琐；"<q>学习 Pcase 和 cond* 会如何减轻这种负担？</q>" (Adam) Porter 指出 `pcase` 可以处理 Stallman 列出的许多进展，因此可能有改进 `pcase` 的路径：

> 你写 cond* 的陈述原因是 Pcase 的各种缺陷。其中一些，比如文档，已经有志愿者站出来解决了。其他问题也可以通过各种方式解决。我提出了一些建议，但你还没有解释拒绝它们的原因。

正如可以猜到的那样，Stallman [不同意](/ml/emacs-devel/E1rSH01-00025m-B1@fencepost.gnu.org/)。

> 如果 pcase 缺少特定工作的功能，通过添加几个功能来修复它将是[容易]的。然而，pcase 的问题在于它对它所做的工作有太多功能。cond*以更少的功能完成相同的工作，因为它们更好地协作。

#### ELPA？

Kangas [说](/ml/emacs-devel/CADwFkmmCYZqGrF5KtSED2RHJuqR-nbo0CDL9gSrHFVBD7zUGfQ@mail.gmail.com/) 他没有密切关注讨论，但对听说 "<q>有计划安装 `cond*`，我会更早表态的</q>" 感到惊讶。添加 `cond*` 将必然使得维护 Emacs 的工作更加困难，因为 `pcase` 并不会消失，因此将会有 "<q>不只是一个，而是两个相对复杂的宏</q>" 他必须知道和理解。如果 `cond*` 提供了显著的好处，那么可能值得这样做，但他没有看到；"<q>我看到的只是 `pcase` 的一个版本。</q>" 因此，他建议在 [GNU Emacs Lisp 包存档](https://elpa.gnu.org/) (ELPA) 上制作一个新包，这将提供 "<q>探索现有宏的替代版本的好方法</q>"。

没有计划彻底用 `cond*` 替换 `pcase` 被 Kangas 视为一件好事。这样可以避免一堆自然而然产生的代码变动和错误。但 Mackenzie [持不同意见](/ml/emacs-devel/ZbD_WHup2fSmtYwi@ACM/)；他认为完全替换 `pcase` 是“<q>一举提高代码的可读性和可维护性</q>”的一个好目标。他认为将其放入 ELPA 库是“<q>确保它永远不会成为现实，只会被遗忘的一种方式</q>”；一个包含最终合并到 Emacs 主线的功能分支会是更好的方向。

像其他几位一样，Monnier [认为](/ml/emacs-devel/jwvv87kvtey.fsf-monnier+emacs@gnu.org/) `cond*` 只是进一步复杂化 Elisp。另一方面，他说，如果他反对它，那也有点虚伪：

> 所以，我对添加这样一种新形式并不是非常热衷，但多年来在 ELisp 中引入了几种这样的新构造后，我确实感觉有点像孤独奋斗者。
> 
> 所以，与其反对它，我只是指出一些我认为可以改进的事情（或者可能触及我的口味）。

随后 Stallman 和 Monnier 之间进行了长时间的来回讨论 `cond*`，它与 `pcase` 的重叠（以及这两者如何可能“中间会合”），以及常规 `cond` 形式在条件内部绑定方面的一些不足之处，等等。许多讨论都围绕 Monnier 的担忧展开，即这两种构造有重叠但仍然不同的模式语言：

> [...] 我看到一个相当广泛但硬编码的模式语言，与 Pcase 相比似乎是一个倒退，后者只从几个原语构建模式语言（通过 `pcase-defmacro` 定义其余部分）。最糟糕的部分是，这两种模式语言非常相似，我看不出有任何好理由造成这些差异。

Monnier [断言](/ml/emacs-devel/jwvttmt8x38.fsf-monnier+emacs@gnu.org/) Stallman 可以简单地重用 `pcase` 的低级模式语言用于 `cond*`，这将带来许多好处：“<q>减少工作量，减少代码重复，减少文档重复，减少编码人员学习的负担。而且您可能会改进 Pcase，所以每个人都会受益。</q>”

-   就目前而言，讨论仍在继续，尚不清楚Stallman将采取哪种模式语言。Monnier[说](/ml/emacs-devel/jwvcysksgl3.fsf-monnier+emacs@gnu.org/)，这种语言通过`match*`形式实现，将为`cond*`添加，很容易改为使用`pcase`机制；这种做法的补丁[已经发送](/ml/emacs-devel/jwvmsroqxhc.fsf-monnier+emacs@gnu.org/)。Mackenzie[/ml/emacs-devel/Zdt3TM4AzmcDSrVq@ACM/]反对`cond*`使用`pcase`模式处理，部分原因是低级`pcase`机制缺乏文档化。Alfred M. Szmidt[希望](/ml/emacs-devel/E1reIZh-0000Z1-56@fencepost.gnu.org/)最终`pcase`能够改为使用`cond*`，而不是反过来，但Monnier[表示](/ml/emacs-devel/jwvbk84qvii.fsf-monnier+emacs@gnu.org/)这在技术上不可行。

#### -   添加`cond*`

一月底，Emacs 的共同维护者Eli Zaretskii和Kangas[宣布](/ml/emacs-devel/CADwFkm=p5QWY4Quy1ihrd8u3FX+NALis+OAOnGG0RzWPMihbsQ@mail.gmail.com/)，`cond*`将作为`pcase`的替代安装到Emacs核心中；然而，并没有努力摆脱`pcase`，因为这是“<q>被视为风格偏好的问题</q>”。在帖子中，Kangas明确指出，政治而非严格技术考虑是决策过程的一部分：

> -   作为维护者，我们的责任首先是确保我们可以共同合作，并在共同的旗帜下团结起来。我们作为一个项目的成功取决于此。因此，我们最不希望做的事情就是排斥任何一组贡献者，无论其规模大小。
> 
> -   我们认为这比对`cond*`或`pcase`赞成或反对的论点更重要。简单的事实是，我们有不同的背景和经验，这些经验使我们在讨论中处于不同的立场。这种多样性是一种优势，而不是一种弱点。

-   即使如此，这也没有满足所有人，因为显然仍然有一些人因`pcase`在14年前安装在Emacs中而感到痛苦，至少对于Mackenzie来说[/ml/emacs-devel/ZbZ5m1AO5Ltqglvc@ACM/]。他认为维护者选择的中间道路不会帮助解决他和其他人在理解一些`pcase`用法方面的问题；他仍然主张全面使用`cond*`进行替换。另一方面，Stallman[并不认为应该替换`pcase`](/ml/emacs-devel/E1rUfF3-00068E-Jd@fencepost.gnu.org/)，但希望“<q>在Emacs内部鼓励使用`cond*`而不是`pcase`</q>”。当然，这一决定并没有阻止又一场[``cond*`与`pcase`讨论`](/ml/emacs-devel/DU2PR02MB10109F1CBA5F7A8D78A597A5D96472@DU2PR02MB10109.eurprd02.prod.outlook.com/)的爆发，自然地继续下去。

一月底，斯托尔曼在[此处](/ml/emacs-devel/E1rVO51-00077C-A0@fencepost.gnu.org/)表示他准备将代码提交到[Emacs Git存储库](/ml/emacs-devel/E1rVO51-00077C-A0@fencepost.gnu.org/)，尽管到目前为止还没有发生。扎雷茨基[要求](/ml/emacs-devel/86cytg1vcu.fsf@gnu.org/)同时向[Elisp参考手册](https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html)添加文档，这可能稍微拖延了进程。不过很快，`cond*`应该会出现在Emacs中，这将提供两种具有模式匹配和解构功能的条件选项——无论是好是坏。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/961682/)

发表评论)
