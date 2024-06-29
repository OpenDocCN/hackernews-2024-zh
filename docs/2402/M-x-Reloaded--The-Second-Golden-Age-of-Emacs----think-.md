<!--yml

category: 未分类

date: 2024-05-29 13:24:57

-->

# M-x 重载：Emacs 的第二个黄金时代 - （想象）

> 来源：[https://batsov.com/articles/2024/02/27/m-x-reloaded-the-second-golden-age-of-emacs/](https://batsov.com/articles/2024/02/27/m-x-reloaded-the-second-golden-age-of-emacs/)
> 
> 生活在黄金时代的人们通常抱怨一切看起来多么黄。
> 
> – Randall Jarell

昨天我写道，我认为 Emacs 当前正在经历它的（第二个）黄金时代。^(今天我将进一步阐述这一观点，并谈谈导致这一局面的原因和因素。)

## 成功之路

昨天有人在 X 上提到了以下内容：

> 我认为 @emacs 在一定程度上因为 VSCode（LSP）和 Atom（tree-sitter）而复兴。

尽管拥有共享的编辑器基础设施确实有所帮助，但我认为这只是 Emacs 近年来“复兴”的一个小部分。我相信许多因素共同作用于此，我将尝试列出我认为最重要的因素。我将在接下来的部分中按照没有特定顺序列出它们。

### GitHub

2008 年 GitHub 的诞生对全球的开源软件开发者来说是一场革命。它是从 SourceForge 和 EmacsWiki 时代迈向前的巨大一步，大量的 Emacs 项目也在其中诞生。

在 GitHub 诞生之前，我很少为开源软件项目做贡献，但之后发生了巨大改变。GitHub 在我开源软件工作中占据了重要位置，我的所有项目仍然托管在那里。其规模和影响力使得项目能够与众多潜在的贡献者连接。最近历史上许多最有影响力的 Emacs 扩展包也是在 GitHub 上诞生和流行起来的。

我认为一个专有平台对自由开源软件社区做了这么多事情有点讽刺，但我认为没有人会反对其结果。我不会在这一部分花太多时间，因为我怀疑我能说的任何关于 GitHub 的话都不会是新闻。

### Clojure

大约在同一时间（2007 年）Clojure 被创建。这种语言的发布在编程社区引起了极大的兴趣和激情，这也转化为了对 Emacs 的增加兴趣。为什么呢？

Emacs 是第一个为 Clojure 编程提供了相当良好支持的编辑器 - `clojure-mode`（略微修改过的 `lisp-mode`）和 SLIME 的 Clojure 适配器 `swank-clojure`。CIDER（SLIME + `swank-clojure` 的后继者）至今仍然是最受欢迎和最强大的 Clojure 编程环境之一。

我注意到许多活跃在 Emacs 社区的贡献者在某种程度上与 Clojure 相关联。以下是一些例子：

+   Phil Hegelberg（化名 `technomancy`），一个多产的 Emacs 黑客。他当年也做了最著名的“Emacs 入门”之一 - [遇见 Emacs](https://www.pluralsight.com/courses/meet-emacs)。

+   Artur Malabarba of [无尽的括号](https://endlessparentheses.com/)

+   Magnar Sveen（Emacs Rocks，`dash.el`，`multiple-cursors.el` 等等）

+   Steve Purcell（许多Emacs包的作者/维护者，MELPA背后的推动力之一）

+   我 :-) 是的，我甚至在对Clojure感兴趣之前就是Emacs用户，但是对Emacs的热情大部分来自于我对Lisp的热爱。Clojure让我有更多理由花时间在Emacs上，并激励我更努力地改进Emacs。

我相当确信Clojure近年来在Emacs的成功中起到了重要作用，吸引了新用户和新贡献者加入项目。

故事还有更多内容。不过，Clojure也对Emacs核心API产生了一些影响，如`if-let`和`thread-first`等宏以及像`dash.el`和`seq.el`这样的库受到了它的影响。

### MELPA

不幸的是，几年前许多Emacs包真是一团糟。许多维护者从不发布版本，用户只能使用代码的最新版本。这使得只分发“打过标签”的版本的包仓库的概念问题重重。更不用说`package.el`当时非常新，没有足够的流行度来鼓励人们改变他们的做法。还记得许多Emacs包当时不使用VCS，只在EmacsWiki上分发吗？好时光！

[MELPA](https://melpa.org)自发布以来真正带来了一场革命 - 这个仓库从众多来源（GitHub、个人代码仓库、EmacsWiki）构建快照发布包。添加一个包到MELPA是非常简单的，“一劳永逸”（不像其前身Marmalade，那时你必须手动上传每个新版本）。你只需提交一个包的配方，MELPA会在需要时重新构建你的包。

今天，MELPA托管了令人惊叹的6000（！！！）个Emacs包，它是我们社区的真正支柱。为了更好的理解 - GNU ELPA和NonGNU ELPA仓库加起来约有650个包。我不知道你们怎么看，但我在浏览MELPA时发现了很多很酷的包，我无法想象Emacs社区没有它会是什么样子。

### Killer “Apps”

不详细展开：

+   [org-mode](https://orgmode.org/)

+   [Magit](https://magit.vc)

+   [Auctex](https://www.gnu.org/software/auctex/)（可能是撰写LaTeX文档的最佳方式）

+   [SLIME](https://slime.common-lisp.dev/)（惊人的Common Lisp编程环境）

+   [Geiser](https://github.com/emacsmirror/geiser)（惊人的Scheme编程环境）

+   [CIDER](https://cider.mx)（令人惊叹的Clojure开发）

你会把什么加入到这个列表中？

### Spacemacs（及其朋友们）

[Spacemacs](https://www.spacemacs.org/)向许多人展示了Emacs的全部潜力，并成功地把许多Vim用户转变为Emacs用户。这绝非小事！虽然它并非第一个Emacs发行版，但在当时无疑是最雄心勃勃的一个。它决定大量使用`evil-mode`并针对Vim用户，结果证明这是一个明智的选择！

Emacs的“发行版”（也称为“入门套件”）在很大程度上有助于弥合原始Emacs体验和其他编辑器用户通常期望的差距。今天我们有[许多](https://github.com/emacs-tw/awesome-emacs?tab=readme-ov-file#starter-kit)，但我仍然认为Spacemacs对整个社区影响最大。

### 共享编辑器基础设施（LSP，TreeSitter）

Emacs因缺乏一些“现代”编辑器和IDE的高级代码分析和重构能力而受到许多批评。它采用行业标准的LSP（语言服务器协议）和TreeSitter改变了这一局面，确保Emacs开发人员无需花时间解决其他地方已经解决的问题。我猜想从长远来看，这将使Emacs的维护者能够更多地专注于使Emacs独特的功能，这在我的书中是一件好事。

完全过渡到LSP和TreeSitter不会一夜之间完成，我们需要数年时间来完成它。尽管如此，迄今为止的进展令人惊叹。未来将会是令人兴奋的时代！

### 渐进式的Emacs维护者

理查德·斯托曼在2008年辞去了Emacs的首席维护者职务，接替他的是一系列更为进步的Emacs维护者。我认为斯蒂芬·蒙尼尔、约翰·韦格利和伊莱·扎雷茨基帮助打造了一个合作共赢、欢迎贡献的环境。

在这里，我们可以列举几个重要的里程碑：

+   在2008年从CVS切换到Bazaar

+   2014年切换到Git

+   采用`package.el`作为标准包管理器（Emacs 24中自带于2012年）

+   创建NonGNU ELPA（官方包仓库，对包含条件有松弛要求）

+   原生JSON支持、原生编译、内置LSP支持、TreeSitter等

或许还有其他非常重要的成就是由首席维护者的行动带来的，我可能忘记了。欢迎在评论中提及。

### 寻找不同的人们

> 你做自己永远不会错。

虽然当今的编辑器（如VS Code）能完成任务，但并不一定适合每个人。总会有一些人因为各种原因在寻找不同的东西。

鉴于今天大多数编辑器的相似性，那些对现状不满意的人实际上选择并不多。如果你想要一个与主流不同并且有强大社区支持的编辑器，基本上只能选择Emacs或Vim。

如果你想要打造一个在各方面都能满足你个人偏好的编辑器，那很可能就是Emacs。

## 总结思考

> 成功发生在机会遇到准备的时候。
> 
> —— 奇奇·兹格拉

正如你所看到的——这是一段跨越15年的旅程。如果我必须选择一个最重要的年份，那可能是2008年：

+   GitHub和Clojure的兴起

+   Emacs领导层的变动

+   Magit的诞生

真是不可思议的一年！

所以，这就是我的看法 - 关于导致Emacs第二个黄金时代的因素和事件。有什么是你不同意的吗？有什么重要的事情你认为我漏掉了吗？请在评论中告诉我！

## 结语

> 你来这里是因为你知道一些事情。你知道的事情无法解释，但你感受得到。你一生都感受到，世界有些不对劲。你不知道具体是什么，但它就在那里，像是心中的一根刺，让你发疯。正是这种感觉把你带到了我这里。
> 
> – 摩菲厄斯（《黑客帝国》）

所以，为什么你应该在2024年尝试Emacs呢？

+   你是一个充满好奇心和不断摸索的人，喜欢玩弄（超酷的）复古软件。

+   你想体验生活在主流之外。

+   你一直想学一点Lisp。

+   你想构建一个完全适合你需求和偏好的编辑器。

+   你有大量时间可消磨。

现在你已经准备好开始你终生的Emacs掌握之旅。Meta-x永存！在括号中，我们信赖！
