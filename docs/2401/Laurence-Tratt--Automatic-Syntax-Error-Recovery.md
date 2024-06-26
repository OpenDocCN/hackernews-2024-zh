<!--yml

category: 未分类

日期：2024 年 5 月 27 日 14:39:17

-->

# Laurence Tratt: 自动语法错误恢复

> 来源：[`tratt.net/laurie/blog/2020/automatic_syntax_error_recovery.html`](https://tratt.net/laurie/blog/2020/automatic_syntax_error_recovery.html)

编程是我碰到的傲慢的最好解药 — 我犯了这么多错误，以至于我不断地被提醒到我的自身缺陷。广义地说，我认为错误分为严重和轻微。严重的错误是我对我正在创建的系统的某些事物有根本性的误解。每个严重的错误都是一个定制问题，通常需要定制工具来理解和修复它。相比之下，轻微的错误是重复性的，很快就可以修复。但是，它们也比严重的错误*要*多得多：即使是缩短程序员修复一类轻微错误所需的时间几秒钟，考虑到它们经常发生，也是值得的。

最微小的错误之一，我怀疑也是最频繁的，就是语法错误。它们出现的主要原因有三个：粗心大意；打字不准确；或者对语言语法的理解不完整。后一种情况通常是我们经历的一个简短学习阶段的一部分，我不确定对于它会有什么好的解决方案。然而，前两种情况非常普遍。当我犯了一个小错别字时，我希望编译器或 IDE 中的解析器能准确地指出语法错误的位置，然后从中恢复，并像我没有犯错误一样继续。由于编译通常远非瞬间完成，而且我经常犯多个错误（不仅仅是语法错误），良好质量的语法错误恢复提高了我的编程效率。

不幸的是，LR 解析器 – 我特别喜欢的 – 在语法错误恢复方面声誉不佳。我将在本文中展示，这并不是不可避免的，而且对于任何 LR 语法来说，都可以做出出乎意料的好的自动语法错误恢复。如果你想了解更多细节，你可能会对我和[Lukas Diekmann](https://diekmann.co.uk/) 最近发表的论文《别慌！更好，更少，LR 解析器的语法错误》感兴趣。该论文还有一个相当简短的附带演讲，如果你觉得这样的东西有帮助的话：

[`www.youtube.com/embed/PGRnk-bzTdU`](https://www.youtube.com/embed/PGRnk-bzTdU)

视频

对于其他人，让我们继续。为了让我们的生活更轻松，在本文的剩余部分中，我将“语法错误恢复”缩写为“错误恢复”。

## 概述问题

让我们在一个广泛使用的编译器中看看错误恢复的效果 — `javac`：

[点击开始/停止动画]

作为一个快速提醒，'`int x y;`'不是有效的 Java 语法。`javac`在‘`x`’标记之后正确检测到语法错误，然后尝试从该错误中恢复。由于'`int x;`'是有效的 Java 语法，`javac`假定我是想在'`x`'之后放一个分号，*修复*我的输入，并继续解析。这是错误恢复的好处：我的愚蠢的语法错误没有阻止编译器继续它的工作。然而，错误恢复的坏处立即显现出来：'`y;`'不是有效的 Java 语法，因此`javac`立即输出第二个虚假的语法错误，这并不是我的错。

当然，我故意挑选了一个`javac`表现不佳的例子，但我遗憾地告诉你，我没花多少时间就找到了它。许多解析器在错误恢复方面做得很差，以至于经验丰富的程序员经常会滚动到第一个语法错误的位置，忽略修复和任何后续的语法错误。错误恢复不仅没有帮助，而且很容易产生相反的效果，使我们在一大堆虚假错误中寻找真正的错误时变慢下来。

让我们来看看我最常用的现代编译器，也是一个良好错误消息的典范，`rustc`。它在面对语法错误时通常做得很好：

[点击开始/停止动画]

然而，即使是`rustc`在面对简单的语法错误时也可能会出错：

[点击开始/停止动画]

有些语言实现甚至不费力地尝试从语法错误中恢复。例如，即使我在一个文件中制造了两个容易修复的语法错误，CPython 一旦遇到第一个错误就会停止：

[点击开始/停止动画]

正如这一切可能暗示的那样，良好的错误恢复很难做到，并且很难完全符合人类关于如何修复语法错误的直觉。问题的根源在于，当我们在解析时遇到错误时，一般来说有无限多种方式可以尝试恢复解析。由于探索无限需要一段时间，错误恢复必须使用某种启发式规则。

解析器对其解析语言的了解程度越深，其启发式规则就越精细。手写解析器在这方面有着根本的优势，因为你可以向解析器添加关于语言语义的任意多知识。然而，以这种方式扩展手写解析器并不是一件小事，特别是对于语法庞大的语言而言。很难得到准确的数据，但我见过不止一个解析器耗费了少数人年的努力，其中大部分用于错误恢复。我们中没有多少人有那么多时间来解决这个问题。

相比之下，自动生成的解析器处于明显的劣势：它们对语言的唯一了解就是通过其语法表达的。尽管如此，自动生成的 LL 解析器通常能够做到对错误的容忍性工作得还不错。

不幸的是，LR 分析器因错误恢复而声名狼藉。例如，Yacc 要求用户在他们的语法中散布`error`标记，以便在生成的解析器中进行错误恢复：我想我只看到过一个真正使用了这个功能的语法，我对其是否能够很好地工作持怀疑态度。*恐慌模式* 是 LR 分析中错误恢复的一种全自动方法，但它是通过逐渐删除解析堆栈来工作的，这导致它在语法错误之前删除输入，以尝试在之后进行恢复。坦率地说，恐慌模式的修复效果非常糟糕，我认为在现代计算机上，它比根本没有错误恢复还要糟糕。

## 解决方案的根源

在处理[grmtools](https://github.com/softdevteam/grmtools/)时，我意识到我应该考虑错误恢复，这是我之前只作为普通用户遇到过的东西。关于 grmtools 的简短介绍：它旨在成为 Rust 中与解析相关的库的集合。目前，对用户最有用的部分是[lrpar](https://crates.io/crates/lrpar)——与[Yacc](http://dinosaur.compilertools.net/yacc/index.html)兼容的解析器——以及 [lrlex](https://crates.io/crates/lrlex)——与[Lex](http://dinosaur.compilertools.net/lex/index.html)相似的词法分析器。在本文的其余部分，我几乎只会谈论 lrpar，因为那是与错误恢复相关的部分。

幸运的是，我很快就找到了[Carl Cerecke 的博士论文](https://ir.canterbury.ac.nz/bitstream/handle/10092/5492/cerecke_thesis.pdf)，它让我眼界大开，看到了一种完全不同的错误恢复方式。他的论文值得仔细阅读，并且里面有一些非常好的想法。最终，我意识到 Cerecke 的论文是我现在称之为[Fischer *等人*](https://minds.wisconsin.edu/bitstream/handle/1793/58168/TR363.pdf?sequence=1&isAllowed=y)家族的 LR 分析错误恢复算法的一员，因为它们都可以追溯到那篇论文。

当 Fischer *等* *人* 家族的错误恢复算法遇到语法错误时，它们会尝试找到一个*修复序列*，将其应用于输入，使解析重新回到正轨。不同的算法有不同的修复方式和不同的创建修复序列的机制。例如，我们最终采用了[Corchuelo *等人*](https://idus.us.es/bitstream/11441/65631/1/Repairing%20syntax%20errors.pdf)的方法——Fischer *等人* 家族中最新的成员之一——作为我们的智力基础。

## CPCT+的使用

我们采用了 Corchuelo *等人* 的算法，对其进行了修复和半正式化，并将其扩展为生成新的错误恢复算法 CPCT+，现已成为 lrpar 的一部分。我们可以使用[`nimbleparse`](https://softdevteam.github.io/grmtools/master/book/nimbleparse.html)——一个简单的命令行语法调试工具——在我们的原始 Java 示例中查看 CPCT+的运行情况：

[点击开始/停止动画]

与 `javac` 的错误恢复类似，当 lrpar 在 ‘`y`’ 标记处遇到语法错误时，CPCT+ 会启动。不同于 `javac`，CPCT+ 向用户呈现 3 种不同的*修复序列*（编号为 1、2、3），按顺序，可以将输入修复为等效于：‘`int x, y;`’、‘`int x = y;`’ 或 ‘`int x;`’。重要的是，修复序列可以包含多个修复：

[点击开始/停止动画]

由于你可能不想无休止地观看动画，我将在这里报告的修复序列放在这里：

```
Parsing error at line 2 column 13\. Repair sequences found:
 1: Insert ), Shift {, Delete if, Delete true 2: Insert ), Shift {, Shift if, Insert (, Shift true, Insert ) 
```

此示例显示了 CPCT+ 可生成的所有单个修复类型：

+   ‘`插入 x`’ 意味着 ‘在当前位置插入标记 `x`’；

+   ‘`移动 x`’ 意味着 ‘保持当前位置的标记 `x` 不变并继续搜索’；

+   ‘`删除 x`’ 意味着 ‘删除当前位置的标记 `x`’。

一个修复序列就是这样的：一个有序的修复序列。例如，上面的第一个修复序列意味着输入将被修复为等效于：

```
class C {
  void f() {
 { } } } 
```

而第二个修复序列将修复输入以等效于：

```
class C {
  void f() {
  if (true) { }
 } } 
```

正如这个示例所显示的，CPCT+ 与传统的错误恢复完全不同：它会一次修复跨多个标记的输入。这与在文件中的不同点处修复语法错误完全补充，正如这个示例所示：

[点击开始/停止动画]

尽管 CPCT+ 可以返回多个修复序列，但并行运行所有这些可能性是不切实际的 —— 我也怀疑用户是否能够解释出结果的错误！因此，lrpar 只采用 CPCT+ 返回的第一个修复序列，将其应用于输入，并继续解析。

到了这一点，你可能已经对 Java 示例感到厌烦了。幸运的是，CPCT+ 并没有与 Java 有关。如果我提供 Lua 的 Lex 和 Yacc 文件以及错误的输入，它也会乐意修复：

[点击开始/停止动画]

的确，CPCT+ 会愉快地对任何你可以写出 Yacc 语法的其他语言进行错误恢复。

最终，CPCT+ 相对于 Fischer *等人* 族的先前成员有一个主要的新奇之处：它向用户呈现了*最小成本修复序列的完整集合*，而其他方法是以非确定性方式向用户呈现该集合的一个成员。换句话说，对于我们的原始 Java 示例，当 CPCT+ 向用户呈现以下内容时：

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Insert , 2: Insert = 3: Delete y 
```

例如 Corchuelo *等人* 的方法只会向用户呈现一个修复序列。由于这些方法是非确定性的，每次运行它们时，它们可以呈现与之前不同的修复序列，这相当令人困惑。所谓“最小成本修复序列”的直觉是，我们要优先考虑对用户输入进行最小数量的更改的修复序列：插入和删除修复会增加修复序列的成本，尽管移动修复是成本中性的。

在我看来，在上面的例子中，‘`Insert ,`’和‘`Insert =`’同样有可能代表程序员的意图，并且展示这两者是有帮助的。‘`Delete y`’有点不太可能代表程序员的意图，但这并不是一个荒谬的建议，在其他类似的情境中，它可能是所呈现的三种修复序列中更可能的。

## 在引擎盖下

[这篇论文](https://soft-dev.org/pubs/html/diekmann_tratt__dont_panic/)和/或[这段代码](https://github.com/softdevteam/grmtools/)是你想要了解 CPCT+工作原理的地方，但我会在这里尝试给你一个非常粗略的概念。

当 lrpar 遇到语法错误时，CPCT+会使用文法的状态表（LR 解析器将文法转换为的状态机；参见[此示例](https://en.wikipedia.org/wiki/LR_parser#Action_and_goto_tables)），当前的解析栈（告诉我们我们在状态表中的位置以及我们是如何到达那里的），以及用户输入的当前位置。根据定义，解析栈的顶部将指向状态表中的一个错误状态。CPCT+的主要任务是返回一个解析栈和位置给 lrpar，以使 lrpar 能够继续解析；生成修复序列是这个过程的一个愉快副产品。

因此，CPCT+实际上是一个伪装的路径查找算法，我们将其建模为[Dijkstra 算法](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)的一个实例。实质上，图中的每一条边都是一个修复，具有成本；我们正在寻找一条通往成功的路径。在这种情况下，“成功”可能以两种方式发生：在文件末尾附近发生错误的罕见情况下，我们可能会达到状态表的唯一的*接受*状态；更常见的是，我们会选择连续移动 3 个令牌（即我们已经到达了可以解析用户一部分输入而不会导致进一步错误的点）。正如这可能暗示的那样，核心搜索是相当简单的。

CPCT+ 的大部分复杂性来自于我们试图找到通往成功的所有最小成本路径，并且我们需要优化搜索的方法。我们在论文中描述了一些提高性能的技术，所以我将使用可能是最有效的一个作为例子。我们的基本观察是，当搜索时，一度不同的路径通常最终达到同一节点，此时我们可以将它们模拟为一个。因此，我们识别 *兼容* 节点并将它们 *合并* 成一个。然后，挑战在于如何有效地识别兼容节点。我们利用了哈希映射经常被遗忘的一个方面：节点的哈希行为只需要是其相等行为的一个子集。在我们的情况下，节点具有三个属性（解析堆栈、剩余输入、修复序列）：我们基于其中两个属性（解析堆栈、剩余输入）进行哈希，这迅速地告诉我们节点可能是兼容的；然后相等性检查（相对较慢）所有三个属性以确认确定的兼容性。这是一个强大的优化，特别是在最难的情况下，将平均性能提高了 2 倍。

## 修复排序

CPCT+ 收集了完整的最小成本修复序列，因为我认为这样做能最好地满足用户对错误恢复算法的期望。创建完整的最小成本修复序列的成本早就清楚了，但令我惊讶的是，还有额外的好处。

所有 Fischer *等* 家族方法面临的根本问题是搜索空间的大小是无界的。这就是为什么仅从用户输入中移动 3 个标记就足以使我们宣布修复序列成功的原因：理想情况下，我们希望检查剩余的全部输入，但这会导致除了少数输入之外的所有情况都会导致组合爆炸。换句话说，CPCT 的核心搜索本质上是局部的：它创建的修复序列仍然可能导致 CPCT+ 搜索的输入的一小部分之外的后续错误。

完整的最小成本修复序列使我们能够轻松地将非常局部的搜索转变为区域性搜索，从而使我们能够在更广泛的上下文中对修复序列进行排名。我们利用了 CPCT+ 的核心搜索通常只能找到少量修复序列的事实。然后我们暂时将每个修复序列应用到输入中，并查看它可以在不引起错误的情况下解析多远（最多 250 个标记）。然后我们选择已解析最远的（非严格）子集并丢弃其余部分。考虑这个 Java 示例：

```
class C { int x z() { } }  
```

当通过“全” CPCT+ 算法运行时，向用户报告了两个修复序列：

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Delete z 2: Insert ; 
```

但是，如果关闭排序，我们可以看到 CPCT+ 的“核心”实际上生成了三个修复序列：

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Insert = 2: Insert ; 3: Delete z Parsing error at line 2 column 15\. Repair sequences found:
 1: Insert ; 
```

在这个特定的运行中，lrpar 选择对输入应用‘`Insert =`’修复序列。这导致了在 CPCT+搜索区域之外有一个虚假的第二个错误。然而，其他两个修复序列允许整个文件成功解析（尽管解析结果存在语义问题，但那是另一回事）。也许不是立即显而易见的，但传统的 Fischer *et al.*算法无法丢弃‘`Insert =`’修复序列并继续寻找更好的东西，因为它们没有可以让它们意识到可以做得更好的比较点。换句话说，最小成本修复序列的完整集合的意外优势正是它允许我们对修复序列进行相对排名并丢弃不太好的修复序列。

随着时间的推移，我也意识到排名过程（需要大约 20 行代码）确实一石二鸟。首先，最明显的是，它减少了虚假的语法错误。其次——这让我花了一段时间才意识到——它减少了我们向用户呈现的低质量修复序列的数量，使得用户更有可能实际检查呈现的修复序列。

## 词法错误

与大多数解析系统一样，grmtools 将词法分析（将输入拆分成标记）与语法分析（确定/如何一个标记序列符合语法）分开。本文不是讨论这一点的场所，但有一个相关的因素：一旦词法分析器在输入中遇到错误，它就会终止。这意味着语法分析不会开始，而且，由于我们迄今定义的错误恢复是解析器的一部分，用户将看不到错误恢复。这导致了这样令人沮丧的情况：

[点击开始/停止动画]

虽然很长一段时间我都没有想到，但将词法错误转换为语法错误，然后进行正常的错误恢复是微不足道的。更令人惊讶的是，这并不需要 lrpar 或 CPCT+的任何支持。用户只需通过在其 Lex 文件末尾添加以下规则来捕获否则不会识别的输入即可：

```
. "ERROR" 
```

这匹配一个单个字符（‘`.`’）作为一个名为‘`ERROR`’的新标记类型。然而，grmtools 如果在 Lexer 中定义了标记但在 Yacc 语法中没有使用，它会抱怨，所以让我们通过向语法中添加一个虚拟规则来关闭它：

```
ErrorRule: "ERROR" ; 
```

现在当我们运行 nimbleparse 时，CPCT+会在我们的“词法错误”上启动：

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Delete #, Delete y 2: Insert ,, Delete # 3: Insert =, Delete # 
```

我觉得这个很令人满意，有两个原因：第一，因为用户在 lrpar 方面没有任何额外的努力就得到了一个有用的功能；第二，因为词法和语法错误现在以统一的方式呈现给用户，而之前它们是令人困惑地分开的。也许把这个作为核心功能会是一个好主意，这样我们就可以做一些事情，比如合并连续的错误标记，但这不会改变底层的技术。

## 将错误恢复集成到操作中

就我所知，没有一个“高级”的错误恢复算法曾经进入一个持久的解析系统：我找不到一个我可以运行的单一实现。事实上，令人惊讶的是，许多错误恢复论文甚至没有提到相应的实现，尽管肯定在某个时候至少有一个研究原型。

无论存在还是不存在任何软件，我阅读过的所有论文都没有提到错误恢复如何影响解析器的使用。想想你最喜欢的编译器：当它遇到语法错误并从中恢复时，它不仅仅继续解析，还运行诸如类型检查器之类的东西（尽管它可能拒绝生成代码）。当然，你最喜欢的编译器这样做的原因可能是因为它有一个手写的解析器。在 Yacc-ish 设置中，我们应该如何处理这个问题呢？

grmtools 的解决方案非常简单：动作代码（即成功解析一个产生式时执行的代码）不直接访问令牌，而是通过一个`enum`，允许将错误恢复插入的令牌与用户输入中的令牌区分开来。这样做的原因可能最好通过一个例子来看，比如这个 [非常简单的计算器语法](https://github.com/softdevteam/grmtools/blob/master/lrpar/examples/calc_actions/src/calc.y) ，它在解析时计算数字：

[点击开始/停止动画]

在这种情况下，我在编写计算器评估器时选择的做法是继续评估带有语法错误的表达式，除非插入了整数。这样做的原因是我根本不知道如果 CPCT+ 生成了一个‘`Insert INT`’修复，我应该插入什么值：0 合理吗？-1 呢？或者是 1？正如这个例子所示，插入令牌可能会引发很多问题：虽然人们可能对当 CPCT+ 删除了‘`2 + + 3`’中的第二个‘`+`’时是否应该继续评估有所争议，但至少该情况不需要评估器从空中捡取一个整数值。

这是我所谓的错误恢复的*语义* *后果*：更改用户输入的语法通常会改变其语义，而 grmtools 无法知道哪些更改具有可接受的语义后果，哪些不具有。例如，在 Python 中插入一个缺失的‘`:`’可能没有语义后果，但在计算器表达式中插入整数 999 将具有重大的语义后果。

好消息是，lrpar 为用户提供了灵活性，以处理令牌插入的语义后果。例如，这是用于计算器语言的与 grmtools 兼容的语法：

```
Expr -> Result<u64, Box<dyn Error>>:
 Expr '+' Term {
 Ok($1?.checked_add($3?)
 .ok_or_else(||
 Box::<dyn Error>::from("Overflow detected."))?)
 } | Term { $1 } ;   Term -> Result<u64, Box<dyn Error>>:
 Term '*' Factor {
 Ok($1?.checked_mul($3?)
 .ok_or_else(||
 Box::<dyn Error>::from("Overflow detected."))?)
 } | Factor { $1 } ;   Factor -> Result<u64, Box<dyn Error>>:
 '(' Expr ')' { $2 }
 | 'INT' {
  parse_int($lexer.span_str($1.map_err(|_|
 "<evaluation aborted>")?.span()))
 } ; 
```

如果你不习惯 Rust，那可能看起来有点吓人，所以让我们从一些非错误恢复的部分开始。首先，计算器语法在解析时评估数学表达式，并且仅处理无符号 64 位整数。其次，与传统的 Yacc 不同，lrpar 要求每个规则都指定其返回类型。在本例中，每个规则都有一个返回类型 `Result<u64, Box<dyn Error>>`，它表示“如果成功，则返回 `u64`；如果不成功，则返回指向堆上错误描述的指针”。换句话说，‘`dyn Error`’ 是 Rust 中“此事将引发异常”的粗略等效物。

与传统的 Yacc 类似，动作代码中的 ‘`$n`’ 表达式引用了符号的产生式，其中 *n* 从 1 开始。符号引用的要么是规则，要么是标记。与大多数解析系统一样，引用规则的符号具有该规则的静态类型（在本例中为 `Result<u64, Box<dyn Error>>`）。

`grmtools` 与现有系统不同的地方在于，标记始终具有静态类型 `Result<Lexeme, Lexeme>`。如果你习惯于 Rust，这可能看起来有点令人惊讶，因为 `Result` 类型几乎总是包含两个不同的子类型，但在这种情况下，我们说在“好”的情况下你会得到一个 `Lexeme`，而在“坏”的情况下你也会得到一个 `Lexeme`。这样做的原因是“好”情况（如果你熟悉 Rust 术语，则为 ‘`Ok`’ 情况）表示用户实际输入的标记，而“坏”情况（‘`Err`’）表示插入的标记。因为 `Result` 是 Rust 的常见类型，所以可以使用 Rust 程序员熟悉的所有标准习语。

让我们首先看一下计算器语法中第一个规则的简化版本：

```
Expr -> Result<u64, Box<dyn Error>>:
 Expr '+' Term { $1? + $3? }
 | Term { $1 } 
```

`Expr` 规则有两个产生式。其中第二个（‘`Term`’）简单地将另一个规则的评估结果原样传递。第一个产生式尝试将其他规则产生的两个整数相加，但如果其中任何一个规则产生了 `dyn Error`，那么‘`?`’ 操作符会将该错误向上传播（粗略地说：将异常抛出到调用堆栈上）。

现在让我们看一下语法中第三个规则的简化（甚至有些不正确）版本：

```
Factor -> Result<u64, Box<dyn Error>>:
 '(' Expr ')' { $2 }
 | 'INT' {
  parse_int($1.map_err(|_| "<evaluation aborted>")?)
 } ; 
```

第二个产生式引用了 `INT` 标记类型。动作代码然后包含了 `$1.map_err(|_| “<evaluation aborted>”)?`，它的英文含义是“如果标记 `$1` 是由错误恢复插入的，则抛出 `dyn Error`”。换句话说，当计算器语法遇到插入的整数时，就停止评估表达式。然而，如果标记来自用户的输入，则将其转换为 `u64`（使用 `parse_int`）并继续评估。

如果你回顾原始语法，你会发现这个语法已经做出了一个决定，那就是*只有*插入的整数具有不可接受的语义后果：例如插入‘`*`’允许继续评估。

在解析完成后，会将解析错误（及其修复）的列表返回给用户，以便他们决定是否要继续计算。因此，不存在 lrpar 修复输入并且解析消费者无法判断是否发生了错误恢复的危险。但是，您可能会想知道为什么 lrpar 只允许对插入修复进行精细控制。当然，它也可以允许用户在删除修复面前做出精细决策，但我不认为这会是用户一个很普遍的需求，也不知道如何为他们处理这种情况提供一个良好的界面。因此，lrpar 拥有的是一种务实的妥协。值得注意的是，尽管上面的内容似乎非常与 Rust 相关，但我相信其他语言可以找到一种不同的、自然的编码方式来表达相同的思想。

## 它足够快而且足够好吗？

此时，您可能已经被说服 CPCT+ 在理论上是一个好主意，但不确定它在实践中是否可用。对我来说，有两个重要的问题：CPCT+ 的速度足够快以便可用吗？CPCT+ 的质量足够好以便可用吗？回答这样的问题并不容易：在卡尔·塞雷克的论文出版之前（以及之后...），我不知道有任何一个错误恢复方法曾经有过一个模糊令人信服的评估。

第一个问题是，我们需要语法错误的代码来用于评估，但你几乎找不到任何野外的源代码是语法错误的，这并不奇怪。尽管已经有一些工作在人工创建文件中的语法错误，但我的经验是程序员产生了如此令人费解的各种语法错误，以至于很难想象有一个工具能够准确地模拟它们。

不同于大多数先前的方法，我们很幸运，因为这些天 BlueJ Java 编辑器有一个可选择的数据收集框架叫做 [Blackbox](https://bluej.org/blackbox/)，它记录了程序员（主要是初学者）在编辑程序时的操作。至关重要的是，这包括了他们尝试编译语法错误程序的情况。因此，我们提取了一个由 200,000 个程序员认为值得编译的语法错误文件的语料库（即我们没有在用户还在输入代码的时候就取得了文件）。如果没有 Blackbox 的访问权限，我不知道我们会做什么：我不知道还有其他哪种语言存在这样丰富的数据集。

“足够快”的问题有很多角度可以看待。在我们的语料库中，CPCT+ 每个文件在错误恢复上平均花费的时间是 0.014 秒。将其放入背景中，这比 Corchuelo *等人* 要快 3 倍，尽管 CPCT+ 计算了完整的最小成本修复序列集，而 Corchuelo *等人* 一旦找到一个最小（ish）成本修复序列就结束了！我还认为最坏情况很重要。由于各种原因，像 CPCT+ 这样的算法确实需要超时来阻止它们永远运行下去，我们将其设置为每个文件最多 0.5 秒，因为假设即使是最苛刻的用户也会容忍错误恢复需要 0.5 秒。我们的语料库比较中，CPCT+ 在超时内完全修复了 98.4% 的文件，而 Corchuelo *等人* 修复了 94.5% 的文件。总之，在大多数情况下，CPCT+ 运行速度足够快，你可能都不会注意到它。

一个更难回答的问题是 CPCT+ 是否足够好。在某种意义上，唯一重要的度量标准是真实程序员是否发现报告的错误和修复有用。不幸的是，这是一个不切实际的评估标准，无法在任何合理的时间内进行评估。试图这样做的错误恢复论文通常只有少于 20 个文件，这导致评估不令人信服。实际上，我们必须找到一种替代的、更容易测量的度量标准，作为我们真正关心的事情的代理。

在我们的情况下，我们使用发现的语法错误总数作为度量标准：数量越少越好。尽管我们知道我们的语料库至少有 200,000 个错误（每个文件至少 1 个），但我们不确定还有多少错误。因此，没有绝对的方法使用这个度量标准来绝对衡量错误恢复算法：我们只能进行相对比较。为了给您一个比较的基准，panic mode 报告了 981,628 个语法错误，而 CPCT+ 报告了 435,812 个。把这放入背景中，如果您使用 panic mode，那么平均每个 CPCT+ 报告的语法错误就会额外多出一个虚假的语法错误，也就是说，panic mode 在这个指标上比 CPCT+ 要糟糕得多。与 Corchuelo *等人* 比较 CPCT+ 更难，因为尽管 Corchuelo *等人* 在语料库中发现的语法错误比 CPCT+ 少 15%，但它也在更多的文件上失败了。这几乎可以肯定地解释为 Corchuelo *等人* 无法完成解析最难的文件，这些文件有时包含惊人数量的语法错误（例如，由于重复的复制和粘贴错误）。

最终，“CPCT+ 是否足够好？”这个问题的真正令人满意的答案是不可能的 —— 我们甚至无法用我们的指标对 CPCT+ 和 Corchuelo *等人* 进行有意义的比较。然而，我们可以非常明确地说，CPCT+ 比 panic mode 要好得多，后者是唯一另一个曾经有些广泛使用的等价算法。

## 限制和未来工作

生活中很少有事情是完美的，CPCT+ 明显也不是。还有很明显的改进空间，如果我有一两年的闲暇时间来解决这个问题，我会看一些事情来使错误恢复变得更好。

CPCT+ 仅在语法错误点尝试修复输入：然而，这往往比人类认为自己出错的点要晚。期望 CPCT+ 或其某个变体能够处理“原因”和“结果”相距甚远的情况是不现实的。然而，我的经验是，错误的原因往往就在被识别为实际错误的点的前面 1 或 2 个标记处。尝试“倒退” CPCT+ 输入 1 或 2 个标记，并观察这是否是一个好的权衡是很有趣的实验。在一般情况下这并不是微不足道的（主要是由于解析堆栈），但在许多实际情况下可能是相当可行的。

正如我们之前所说，CPCT+ 的搜索本质上是局部的，即使有修复排序，它也可能建议导致虚假错误的修复序列。我认为有两种有前途且互补的可能性可以减轻这个问题。第一种是利用一种鲜为人知且美妙的处理语法错误的方法：[不纠正错误恢复](https://dl.acm.org/doi/10.1145/3916.4019)。这通过丢弃语法错误点之前的所有输入，并使用修改后的解析器来解析剩余部分来实现：它不告诉用户如何修复输入，但会报告语法错误的位置。简化一下，像 CPCT+ 这样的算法会过度近似真正的语法错误（即它们报告（几乎）所有“真实”语法错误以及一些“假”语法错误），而不纠正错误恢复则会欠近似（即它会漏掉一些“真实”语法错误，但永远不会报告“假”语法错误）。我认为可以将不纠正错误恢复作为 CPCT+ 当前排名系统的一个更好的替代方案，甚至可能用于从一开始就指导搜索。不幸的是，我认为不纠正错误恢复目前尚未被“移植”到 LR 解析，但我认为这个任务并不是无解的。第二种可能性是利用机器学习（见例如[这篇论文](https://ieeexplore.ieee.org/document/8330219)）。在你激动之前，我怀疑机器学习是否是错误恢复的灵丹妙药，因为搜索空间太大，人类犯的语法错误的种类非常惊人。然而，我的直觉是，机器学习方法在恢复非局部错误方面会比像 CPCT+ 这样的算法更好。

更不明显的是，一些 Yacc 语法比其他语法更容易进行良好的修复。不点名地说，一些语法出人意料地宽容，允许通过“不正确”的语法，后续编译器的某个部分（或者有时是解析器的语义动作）会拒绝。这样做的问题在于搜索空间变得非常大，导致 CPCT+ 要么生成大量令人惊讶的修复序列，要么在最坏的情况下无法在分配的时间内完成搜索。解决此问题的一种方法是重写语法的这些部分，以更准确地指定可接受的语法，尽管这对我来说更容易建议，而对其他人来说实际执行起来可能更困难。另一种解决方法可能是向 CPCT+ 提供额外的提示（类似于 lrpar 的 [`%avoid_insert`](https://softdevteam.github.io/grmtools/master/book/errorrecovery.html#biasing-repair-sequences) 指令），使其能够缩小搜索范围。

程序员无意中在他们的输入中留下提示（例如缩进），语言具有重要的结构组件（例如块标记，例如花括号），错误恢复可以考虑到这些（参见例如[这种方法](https://repository.tudelft.nl/islandora/object/uuid%3Ac46d95b0-aae6-4dd9-aa9a-759bee35d577)）。为了利用这一点，语法作者需要提供提示，例如“块的开始/结束标记是什么”。这些提示是可选的，但我猜想大多数语法作者会发现由此产生的改进足够值得他们投入必要的时间去理解如何使用它们。

最后，一些语法部分必然允许大量的替代方案，在这些点进行错误恢复是很辛苦的工作。最明显的例子是二元或逻辑表达式，其中许多不同的运算符（例如‘`+`’、‘`||`’等）都是可能的。这可能会导致搜索空间爆炸，偶尔导致错误恢复失败，或者更经常地，导致 CPCT+ 生成数量庞大的修复序列。我最喜欢的例子是这个——这是直接从一个真实的例子中摘录的，尽管变量名已经修改了！——看起来无害，但显然语法不正确的 Java 表达式 `x = f(""a""b);`。CPCT+ 为其生成了一个滑稽的 23,607 个修复序列。用户应该如何处理所有这些信息？我不知道！我有时尝试的一件事是将组合方面明确化，以便不是：

```
Insert x, Insert +, Insert y Insert x, Insert +, Insert z Insert x, Insert -, Insert y Insert x, Insert -, Insert z 
```

用户将被呈现：

```
Insert x, Insert {+, -}, Insert {y, z} 
```

由于各种乏味的原因，该功能在某个时候被移除了，但写下这些让我觉得它可能应该重新引入。这并不能完全解决“修复序列数量过多”的问题，但可以减少它，可能会相当大程度上减少。

## 总结

分析是一个会使聚会上的谈话陷入僵局的话题。然而，由于每个程序员都依赖于分析，我们能够帮助他们快速修复错误的工作越好，他们的生活就越好。CPCT+ 永远不会超越最好的手写错误恢复算法，但它确实为任何 LR 文法带来了相当不错的错误恢复能力。我希望并期待未来会出现更好的错误恢复算法，但 CPCT+ 现在就在这里，如果你使用 Rust，你可能想看看 grmtools — 我建议从 [grmtools 书中的快速入门指南](https://softdevteam.github.io/grmtools/master/book/quickstart.html) 开始。希望其他语言的 Yacc 解析器可以移植 CPCT+，或者类似的东西到它们的实现中，因为 CPCT+ 并没有任何非常 Rust 特定的东西，而且它是一个相当容易实现的算法（在 Rust 版本中不到 500 行代码）。

最后，有些人很合理地反对 LR 分析的一个论点是它的错误恢复质量较差。这很遗憾，因为在我看来，LR 分析是一种优美的分析方法。我希望 CPCT+ 的错误恢复能够减轻对使用 LR 分析的这一特定障碍。

**致谢：** 感谢 Edd Barrett 和 Lukas Diekmann 的评论。

更新的文章

2020-11-17 08:00

更早的文章

如果你想要获取新博客文章的更新：请关注我

[长毛象](https://mastodon.social/@ltratt)

或者

[推特](https://twitter.com/laurencetratt)

；或者

订阅 RSS 源

；或者

订阅邮件更新

：

### 脚注

### 评论
