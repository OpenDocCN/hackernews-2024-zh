<!--yml

category: 未分类

date: 2024-05-27 15:03:20

-->

# Laurence Tratt: Parsing: The Solved Problem That Isn't

> 来源：[https://tratt.net/laurie/blog/2011/parsing_the_solved_problem_that_isnt.html](https://tratt.net/laurie/blog/2011/parsing_the_solved_problem_that_isnt.html)

**Updated (2014-10-24): If you find this article interesting, you may be interested in [the follow-up article on an editor for composed programs](/laurie/blog/entries/an_editor_for_composed_programs).**

解析是将一系列字符进行推导，确定它们是否符合某种基础语法的过程。例如，“Bill hits Ben”符合英语语法的`名词 动词 名词`部分。解析关注于揭示结构；虽然这只能部分指示句子的意义，但完整的意义只能在后续的处理阶段揭示。像“Bill evaporates Ben”这样明显无意义的句子突显了这一点（句子仍然是`名词 动词 名词`，但找到两个人对其含义达成一致将是一场挑战）。作为人类，我们自然地在不经意间一直在解析文本；事实上，我们甚至有能力解析我们以前从未见过的结构。

在计算机领域，解析也很常见；虽然语法是合成的（例如特定编程语言的语法），但总体思想与人类语言相同。尽管不同社区在解析的实际处理上有不同方法 - C 程序员会使用`lex` / `yacc`；函数式程序员会使用解析器组合子；其他人会用到诸如[ANTLR](http://www.antlr.org)或基于[Packrat / PEG](http://pdos.csail.mit.edu/~baford/packrat/)的方法 - 但它们通常依赖于相同的基础知识领域。

After the creation of programming languages themselves, parsing was one of the first major areas tackled by theoretical computer science and, in many peoples eyes, one of its greatest successes. The 1960s saw a concerted effort to uncover good theories and algorithms for parsing. Parsing in the early days seems to have shot off in many directions before, largely, converging. Context Free Grammars (CFGs) eventually won, because they are fairly expressive and easy to reason about, both for practitioners and theorists.

不幸的是，鉴于上世纪六十年代计算机硬件极其有限（并未得到有效算法的帮助），任意上下文无关文法（CFG）的解析速度过慢，无法实用化。诸如LL、LR和LALR的解析算法识别了CFG的部分子集，可以高效解析。后来，出现了相对实用的任意CFG解析算法，其中最显著的是[Earley的1973年解析算法](http://dl.acm.org/citation.cfm?id=358005)。很容易忽视那时和现在性能上的相对差异：1964年至1969年间世界上*最快的计算机*是CDC6600，运行速度约为10 MIPS；我的2010年手机处理器运行速度超过2000 MIPS。等到计算机速度足够快以运行Earley算法时，LL、LR及其类似算法已经树立了文化上的主导地位，这种主导地位直到最近才开始受到严肃挑战 - 许多最广泛使用的工具仍然使用这些算法（或其变种）进行解析。然而在像[ACCENT / ENTIRE](http://accent.compilertools.net/)和最新版本的`bison`等工具中，我们可以使用高效的解析器解析任何CFG，如果需要的话。

因此，普遍的共识是解析问题已经解决。如果你有合成语言的解析问题，现有的工具应该可以胜任。一些英雄人物 - 如[Terence Parr](http://www.cs.usfca.edu/~parrt/)、[Adrian Johnstone](http://www.cs.rhul.ac.uk/~adrian/)和[Elizabeth Scott](http://www.cs.rhul.ac.uk/People/staff/scott.html) - 依然在不懈努力，以确保解析变得更加高效，但最终这将被工具透明地采纳，而无需显著改变解析通常的方式。

## 语言组合

在过去几年中，我越来越明显地意识到，普遍的共识在一个关键的新兴趋势面前不再适用：语言组合。组合是一个长期复杂但常常模糊的术语，在理论工作中频繁出现。幸运的是，对于我们的目的，它意味着一个简单的事实：语法组合，即我们将一个语法添加到另一个语法中，并使得合并的语法可以解析新语言中的文本（这恰好是我们想要在特定领域语言（DSL）中做的事情）。举个经典的例子，想象我们希望将类似Java的语言扩展为能直接编写SQL的语言：

```
for (String s : SELECT name FROM person WHERE age > 18) {
 ... } 
```

让我们假设有人为我们提供了两个独立的语法：一个是类似Java的语言，另一个是SQL的语法。语法组合看起来应该相对容易。实际上，事实证明这相当令人沮丧，现在我将解释其中一些原因。

### 语法组合

尽管语法组合在理论上是微不足道的，简单地将两个语法压在一起在实践中很少有用。通常，语法有一个单一的起始规则；因此需要选择这两个语法中的哪一个有起始规则。更混乱的是，两个语法相互引用的机会很小；实际上，需要指定第三批数据 - 常常被误导地称为"粘合剂" - 它实际上将这两个语法连接在一起。在我们的示例中，类似Java的语言有主要语法；粘合剂将指定在Java表达式中SQL语句可以被引用的位置。

对于那些使用旧的解析算法如LR（以及LL等）的人来说，存在一个更基本的问题。如果拿两个LR兼容的语法并组合它们，得到的语法不一定是LR兼容的（即LR解析器可能无法使用它来解析）。因此，这些算法在语法组合方面的用处很小。

此时，像Earley算法的用户显得有些得意洋洋。因为我们从语法理论上知道，合并两个CFG总是导致一个有效的CFG，这样的算法总是可以解析语法组合的结果。但是，或许不可避免地，还存在问题。

### 标记化

解析通常是一个两阶段过程：首先我们将输入拆分成标记（标记化）；然后我们解析这些标记。标记是我们日常语言中所谓的单词。在英语中，单词很容易定义（大致：一个单词以空格或标点符号字符开始和结束）。然而，不同的计算机语言对它们的标记有相当不同的概念。有时，标记化规则很容易组合；然而，由于标记化是在不知道标记将来如何使用的情况下进行的，有时很难。例如，在SQL中SELECT可能是一个关键字，但在Java中它也是一个有效的标识符；在传统解析器中很难，甚至不可能将这些标记化规则组合起来。

幸运的是有一个解决方案：无扫描仪解析（例如SDF2 [scannerless parsing](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.37.7828&rep=rep1&type=ps)）。对于我们的目的而言，也许更好称为无标记解析；不同的名称反映了不同解析学派的命名惯例。无扫描仪解析消除了单独的标记化阶段；现在语法包含了动态标记文本所需的信息。现在可以将具有明显不同标记规则的语法组合在一起了。

### 细粒度组合

在实践中，早前提到的简单粘合剂用于合并两个文法通常是不够的。文法之间可能存在微妙的冲突，即合并后的语言可能与预期结果不符。考虑合并具有不同关键字的两个文法。无扫描解析允许我们合并这两个文法，但我们可能希望确保合并后的语言不允许用户将另一种语言的关键字用作标识符。在正常的上下文无关文法中难以表达这一点。早先提到的SDF2论文允许使用拒绝产生式作为解决方案；不幸的是，这使得SDF2文法[稍微上下文敏感](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.56.883&rep=rep1&type=pdf)。据我所知，这种做法的确切后果尚未被探讨，但这意味着至少一部分上下文无关文法理论将无法适用；这足以让人有点紧张，至少是这样（尽管[埃尔科·维瑟](https://eelcovisser.org/)及其它人使用SDF2形式主义创造出的优秀作品不在此列）。

最近，尽管相对较为不知名，但另一种选择是[布尔文法](http://users.utu.fi/aleokh/boolean/)。这些文法是上下文无关文法的一种泛化，包括合取和否定，乍看之下，这些构造正是使文法组合实际化所需的（允许表达像标识符是除了SELECT之外的任意ASCII字符序列）。至少对我来说，布尔文法似乎有很大的潜力，[亚历山大·奥赫廷](http://users.utu.fi/aleokh/)正在为此做出英勇努力。然而，据我所知，它们还没有被实际应用，因此理解其实用性远非易事。关于布尔文法还有[几个未解问题](http://users.utu.fi/aleokh/boolean/nine_open_problems.html)，其中一些在解答之前可能会阻碍广泛应用。特别是一个问题涉及到歧义性，这需要更多的讨论。

### 歧义性

通过严格限制它们接受的上下文无关文法，与传统解析算法（LL、LR等）兼容的文法始终是无歧义的（尽管，如我们将看到的，这并不意味着所有不兼容的文法都是有歧义的：许多是无歧义的）。因此，文法的歧义性没有被广泛理解。考虑标准算术的以下文法：

```
E ::= E "+" E
 | E "-" E
 | E "/" E
 | E "*" E 
```

使用这种语法，像`2 + 3 * 4`这样的字符串可以有两种模棱两可的解析方式：等同于`(2 + 3) * 4`；或者等同于`2 + (3 * 4)`。像Earley算法这样的解析算法会生成所有可能性，尽管我们通常只需要其中一种（由于算术约定，本例中我们需要后者的解析）。有几种不同的方法可以消除语法歧义，例如优先级（在本例中，较高的优先级在面对歧义时胜出）：

```
E ::= E "+" E  %precedence 1 | E "-" E  %precedence 1 | E "/" E  %precedence 2 | E "*" E  %precedence 3 
```

这可能暗示我们可以相对轻松地控制歧义：不幸的是，解析理论告诉我们实际情况相当棘手。基本问题在于，通常情况下我们无法静态分析上下文无关文法（CFG）并确定它是否具有歧义。要发现给定的CFG是否具有歧义，我们必须尝试每种可能的输入：如果没有输入触发歧义解析，则CFG没有歧义。然而，这在一般情况下是不可行的：大多数CFG描述无限语言，并且无法进行详尽的测试。有各种技术旨在为歧义提供良好的启发式方法（参见[Bas Basten的硕士论文](https://sites.google.com/site/basbasten/publications/BastenMaster.pdf)以获取良好的总结；我也正在与同事合作开发新方法，尽管目前来看是否有用还为时过早）。然而，这些启发式方法本质上是有限的：如果它们表示CFG存在歧义，则肯定存在；但如果它们找不到歧义，则只能说CFG*可能*是无歧义的。

由于理论问题并不总是实际问题，一个好问题是：这是一个真正的问题吗？根据我迄今的经验，使用Earley解析为编程语言定义独立语法（即可能存在歧义的解析算法），这并不是一个巨大的问题：作为语法设计者，我经常能够理解危险的歧义可能存在的地方，并及时制止。我有几次受到影响，但并不足以真正担心。

然而，我不认为我的经验能够在广泛的语法组合面前保持不变。理论原因很容易说明：组合两个无歧义的语法可能导致一个有歧义的语法（正如之前所述，我们通常无法一般性地静态确定）。考虑合并来自不同作者的两个语法，两者都无法预见特定组合：在这种情况下，我认为歧义更有可能出现。直到不幸的用户发现触发歧义的输入之前，它将保持未被检测到。对看似有效输入失败的编译器不太可能受欢迎。

### PEGs

正如前文所述，LL和LR等明确的解析算法在语法组合中并不容易使用。最近，重新发现的一种解析方法引起了广泛关注：[Packrat / PEG parsing](http://pdos.csail.mit.edu/~baford/packrat/)（我将其简称为PEGs）。PEGs与先前提到的所有方法都不同：它们与CFGs没有正式关系。这主要是因为PEGs的*有序选择*运算符，在PEGs中消除了任何歧义的可能性。PEGs很有趣，因为与LL和LR不同，它们在组合时是封闭的：换句话说，如果你有两个PEGs并将它们组合，你将得到一个有效的PEG。

PEGs是解决我们问题的答案吗？唉，至少目前情况下，我对此表示怀疑。首先，PEGs相当不具表达力：与LL和LR解析类似，PEGs在实践中经常令人沮丧。这主要是因为它们不支持左递归；Alex Warth提出了[一种增加左递归的方法](http://www.vpri.org/pdf/tr2007002_packrat.pdf)，但是[我发现似乎存在问题](/laurie/research/pubs/html/tratt__direct_left_recursive_parsing_expression_grammars/)，尽管我应该指出，关于这一点尚未达成普遍共识（我正在与一位同事合作，试图确切理解PEGs中左递归应该意味着什么）。其次，虽然PEGs始终是无歧义的，但取决于在组合过程中使用的粘合剂，有序选择运算符可能导致先前在各自语言中被接受的字符串在组合语言中不被接受——这，姑且说是不太可能是期望的行为。

## 结论

如果你一直看到这里，做得很好。这篇文章的长度比我最初预期的要长得多 - 尽管远远比我真正详细讨论这些要点时可能的长度要短！重要的是要注意，**我不是一个解析专家**：我只是想成为解析的一个*用户*，而不是像我现在这样知道一些内部工作的人。发生的情况是，我希望更多地使用解析，我逐渐意识到了我能找到的东西的局限性。重点是逐渐：有关解析的知识分散在几十年间（从60年代一直到今天）；许多出版物（其中一些难以获取）；以及许多人的头脑（其中一些人不再从事计算机工作，更不用说解析领域了）。因此，要理解各种方法或它们的局限性是很困难的。本文是我尝试写下我当前理解的文章，特别是在编写语法时当前方法的局限性；我欢迎那些比我更有知识的人的更正。预测未来是个鸡蛋砸的游戏，但我开始想知道，如果我们不能提出更合适的解析算法，未来希望允许语法扩展的编程语言将会完全绕过解析，并使用语法直接编辑。许多人认为解析是一个解决了的问题 - 我认为它并不是。

[更早的](/laurie/blog/2011/problems_whose_solution_are_easy_to_state.html)

2011-03-15 08:00

[更旧的](/laurie/blog/2010/in_praise_of_the_imperfect.html)

如果你想获得新博客文章的更新：请在以下平台上关注我

[Mastodon](https://mastodon.social/@ltratt)

或者

[Twitter](https://twitter.com/laurencetratt)

；或者

[订阅RSS提要](../blog.rss)

；或者

[订阅电子邮件更新](/laurie/newsletter/)

：

### 注释
