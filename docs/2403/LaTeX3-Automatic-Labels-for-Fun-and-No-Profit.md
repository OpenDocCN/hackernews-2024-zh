<!--yml

类别：未分类

日期：2024-05-29 12:30:04

-->

# LaTeX3 自动标签：有趣且无盈利

> 来源：[https://commutative.xyz/~miguelmurca/blog/x/autoref.html](https://commutative.xyz/~miguelmurca/blog/x/autoref.html)

# LaTeX3 自动标签：有趣且无盈利

我是一名物理学的博士生，这意味着我过去 10 年大部分时间都在写 LaTeX。对于不了解的人来说，LaTeX 是一个 40 年历史的超集，属于 46 年历史的排版系统 - 也就是说，它是一个基于宏的编程语言，用于生成印刷文档。值得注意的是，它主要*不*是作为一种编程语言；它最强的一面可以说是它美丽地排版数学公式，以及它解决复杂数学表达式的解决方案。例如，

```
\sqrt{\frac{a+b}{\vert{c}\vert} 
```

引用 <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><msqrt><mrow><mfrac><mrow><mi>a</mi> <mo>+</mo> <mi>b</mi></mrow> <mrow><mi>|</mi> <mrow><mi>c</mi></mrow> <mi>|</mi></mrow></mfrac></mrow></msqrt></mrow></math> 。确实，如果您曾经需要在计算机上写入数学表达式，您很可能使用了 TeX 或某种形式的简化 TeX。

但是，(La)TeX 确实是图灵完备的 - 它只是极为复杂。这使得(La)TeX 成为一个相当有趣的神秘编程语言来玩耍。另一方面，能够驾驭 (La)TeX 的宏系统让您能够自动化重复的任务，或者通常扩展 (La)TeX 的功能，这最终会变得非常实用。这也是为什么有一个社区努力改进 LaTeX 的编程（“宏编写”）的原因。

LaTeX3 是“LaTeX 的新核心，基于其自身一致的接口来控制 TeX 的所有必要函数。”^([[@expl3.pdf]](https://mirrors.up.pt/pub/CTAN/macros/latex/contrib/l3kernel/expl3.pdf)) “新”一词或许有争议，因为它自 1989 年以来一直在开发中，但简言之，现在我们可以访问一组更复杂且行为更可预测的 LaTeX 基本宏集。不幸的是，由于 Knuth 本人对文学编程的喜爱（毕竟他发明了这个概念），以及 LaTeX 输出的性质，^([需要引用]) 大多数关于 LaTeX3 功能的信息都埋藏在深入且带有零散散文和代码的长 PDF 中，只能通过 `texdoc <您必须猜测的标识符>` 访问。一个值得注意的（且受欢迎的）例外是[这篇 Alan Xiang 的文章，我推荐阅读](https://www.alanshawn.com/latex3-tutorial/)。无论如何，本文是我为了提供实用且易于理解的 LaTeX3 材料而做出的小小贡献，以便您也能通过编写非常复杂的 LaTeX 宏来拖延写文档。

## 问题陈述和目标

在 LaTeX 中，您可以 `\label{...}` 区段、方程式等，然后可以使用 `\ref{...}` 引用它们的标识符。所以，例如，

```
\documentclass{article}
\begin{document}

\section{First section}
\label{sec:first-section}

Some text here.

This is actually section~\ref{sec:first-section}.

\end{document} 
```

产生

```
1 First section

    Some text here.
    This is actually section 1. 
```

正如所说，这在方程式中同样适用，并且在数学文档中非常有用，你可以在文本中引用方程式。LaTeX2e提供了`equation`环境，它会在你的方程式旁边自动排版一个漂亮的`(eqno)`，你可以给它标签并进行引用：

```
\documentclass{article}
\let\implies\Longrightarrow % \implies == \Longrightarrow
\begin{document}

\begin{equation} \label{eq:a-implies-b}
    A \implies B
\end{equation}
\begin{equation} \label{eq:b-implies-c}
    B \implies C
\end{equation}

From eqs.~$(\ref{eq:a-implies-b}, \ref{eq:b-implies-c})$, we conclude that $A$ implies
$C$.

\end{document} 
```

得到的结果如下：

```
 A ⇒ B       (1)
            B ⇒ C       (2)

From eqs. (1,2), we conclude that A implies C. 
```

然而，标签必须在整个文档中是唯一的。因此，在撰写长篇文档时，为即将在下一行引用的方程式想出一个好的标签很快会让人沮丧。有没有办法让`\AutoLabel`和`\AutoRef`宏在被调用时自动生成并引用标签，那就太好了。

```
\begin{equation} \AutoLabel
    1 + 1 = 2
\end{equation}

Equation~\AutoRef{} can be proven with set theory. 
```

## 第一步

这个问题的最简单解决方法是让`\AutoLabel`在每次调用时以系统化的方式生成不同的标签，并让`\AutoRef`在调用时引用该标签。

```
\documentclass{article}
\let\implies\Longrightarrow

\ExplSyntaxOn
\int_new:N \g_autolabel_int
\NewDocumentCommand{\AutoLabel}{}{
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoRef}{}{
    \exp_args:Ne \ref {autolabelprefix-\int_use:N \g_autolabel_int}
}
\ExplSyntaxOff

\begin{document}

\begin{equation} \AutoLabel
    A \implies B
\end{equation}

I have just defined eq.~\AutoRef{}.

\end{document} 
```

让我们逐行分析：

在`\ExplSyntaxOn`和`\ExplSyntaxOff`之间的任何内容都遵循LaTeX3的语法规则，而不是LaTeX2e的规则。事实上，(La)TeX允许你在文档中动态重新定义语法规则；特别是，解析时每个字符的“功能”可以粗略地重新定义。例如，虽然`\`通常是一个特殊字符（表示接下来是一个控制序列），但你可以在文档的中途将其值更改为普通字符。在LaTeX2e中，这种机制的典型（滥用）用法是修改`@`的“字符类型”（即“字符码”或“类别码”），使其在特殊字符和普通字母之间转换，以保护内部命令不受最终用户的影响。这个过程可以通过`\makeatletter`和`\makeatother`命令来实现（分别将`@`的类别码设置为“普通字母”和“其他字符”），在LaTeX2e代码中，你经常会看到如下的代码：

```
\makeatletter
\newcommand{\@internalcommand}{Do something}
\newcommand{\endusercommand}{\@internalcommand\ and something else.}
\makeatother 
```

在`\makeatother`之后调用`\@internalcommand`会产生错误，因为在调用`\makeatother`之后，LaTeX不再知道如何解释字符`@`。这也突显了类别码切换发生在解析这些声明时，即使这些声明本身的表达式在声明时并不被评估。这在以下示例中得到了表现，这是我多次犯过的错误：

```
\newcommand{\boom}{\makeatletter\@destroytheworld\makeatother} 
```

在文档的后面调用`\boom`会导致错误，因为在声明时，`\@`被解析为声明中的命令，而不是`\@destroytheworld`。在宏声明时，解析器看到`\makeatletter`，但只将其视为声明的一部分的宏 - 毕竟，在声明时运行定义是没有意义的。因此，它根据当前的规则解析声明的其余部分，这些规则并不期望`@`是一个普通字符。因此，解析器将`\boom`最终解析为`[macro: \makeatletter] [macro: \@] [letters: destroytheworld] [macro: \makeatother]`。当最终调用`\boom`时，LaTeX会抱怨使用了`\@`。

这是一个相当长的离题，用来说明`\ExplSyntaxOn`修改类别码，以便更喜欢和更一致的宏名称，并提供更好的“编程”环境。有两个显著的效果：首先，在`expl3`规则内，空格被忽略。实际上，在LaTeX2e中处理宏声明中的空格是一个真正的痛点；不仅仅是你键入的空格，还有新行隐含的空格。这里有一个例子；你期望`\a`、`\b`、`\c`和`\d`之间的差异是什么？

```
\newcommand{\a}{foobar}
\newcommand{\b}{
    foobar
}
\newcommand{\c}{
    foo
    bar%
}
\newcommand{\d}{foo%
    bar} 
```

我通过在LaTeX文档中调用`.\a..\b..\c..\d.`来测试这些；句点用于突出可能的间距。结果如下：

```
.foobar..␣foobar␣..␣foo␣bar..foobar. 
```

(我已经为你突出显示了空格，使用了 `␣` 字符。) 本质上，在文本主体中你所期望的 – 单个换行变成空格 – 也发生在宏声明中，导致在任何地方插入多余的空格。这些空格只能通过在行尾插入注释字符 `%` 来避免，解析器将其解释为显式指令以忽略换行符。这导致宏声明中到处都是 `%` 的混乱，因为在某些时候，宏编写者开始在不想有空格的每行末尾放置 `%`。虽然在给定的示例中，空格不太重要（尽管不需要），但在复杂的宏声明中，多余的空格将导致一些非常恶劣的错误和崩溃。双重换行的情况更糟，它们会转换为段落，这将*真正*混乱你命令的解析。

另一个显著的类别码切换是使`:`和`_`成为“普通字母”。因此，在LaTeX2e中，宏名称通常由`a-zA-Z`（最终是`@`）组成，而在LaTeX3中，宏名称期望是形式为（`texdoc expl3`，第3.2节）

```
\⟨module⟩_⟨description⟩:⟨arg-spec⟩ 
```

其中`module`将是唯一的每个宏包前缀，以避免名称冲突，`description`将是宏的实际名称。`arg-spec`有点特殊，告诉你宏期望的参数 *类型*，尤其是命令是否会扩展它。可以将其视为仅略高于类型提示。宏扩展是一个大主题 – 在这里无法详细覆盖 – 但这里有一个说明性的例子：

```
\newcommand{\foobar}{A winner is you!}
\newcommand{\myname}{foobar}

\csname\myname\endcsname % The macro named `\myname`
                         % (so, `\\myname`, if it were possible.)

\expandafter\csname\myname\endcsname % \foobar 
```

关于扩展的更多信息，请[在这里](https://www.overleaf.com/learn/latex/Articles/How_does_%5Cexpandafter_work%3A_The_meaning_of_expansion)阅读。

在`expl3`中，宏的`arg-spec`部分将告诉您宏消耗的内容，以及它是否会展开所消耗的内容。主要关注的几个（虽然您可以在`texdoc expl3`的第3章和LaTeX3主要参考手册中找到完整列表）是：

+   `n` – 一组括号中的字符 ("记号列表")，

+   `N` – 一个单一字符 ("记号")；宏是一个单一的记号，

+   `c` – 一个宏 *名称* (`foobar`，而不是`\foobar`)；因此，一个记号列表，

+   `V` – 一个变量，即包含值的宏 (继续阅读),

+   `v` – 一个变量的 *名称*；类似于 `c`，但适用于 `V`，

+   `e` – 在消费前需要完全展开的一组花括号内的字符，

+   `o` – 在消费前需要展开一次的一组花括号内的字符，

+   `T` 和 `F` – 一个条件评估为真或假时插入的一组花括号内的字符。

在这里，“变量”的概念也出现了，因为 LaTeX3 尝试将一个做事情的宏（函数）与仅仅存储某个值的宏（变量）分开。因此，在 LaTeX3 的术语中，下面的 `\foo` 将是一个变量，而 `\baz` 则是一个函数：

```
\newcommand{\foo}{winner}
\newcommand{\baz}[1]{A #1 is you!}

\baz{\foo} 
```

虽然我真的应该说，

```
\ExplSyntaxOn
\cs_new:Nn \baz:N {
    A #1 is you!
}
\tl_new:Nn \foo {winner}
\baz:N \foo
\ExplSyntaxOff 
```

更加正确的说法将是

```
\ExplSyntaxOn
\cs_new:Nn \baz:N {
    A #1 is you!
}
\cs_generate_variant:Nn \baz:N {V}
\tl_new:Nn \foo {winner}
\baz:V \foo
\ExplSyntaxOff 
```

最后，看吧，`texdoc expl3` 指定了变量应该命名为 `\⟨scope⟩_⟨module⟩_⟨description⟩_⟨type⟩`，所以这里是一些真正的、老实的 LaTeX3：

```
\ExplSyntaxOn
\cs_new:Nn \custom_baz:N {
    A #1 is you!
}
\cs_generate_variant:Nn \custom_baz:N {V}
\tl_new:Nn \l_custom_foo_tl {winner}
\custom_baz:N \l_custom_foo_tl
\ExplSyntaxOff 
```

这是得到 LaTeX3 提供的神奇扩展控制工具所需付出的代价。

最后，我们移至以下行，

```
\int_new:N \g_autolabel_int 
```

在这里，我们声明一个新的整数变量。作为合格的 LaTeX3 使用者，我们使用正确的变量命名方案，并通过 `g_`（文档中的标签需要全局唯一，所以这是合理的）将变量声明为全局变量。我们期望这个变量被初始化为 0；参见 `texdoc interface3` 第 168 页：

> 这个 ⟨integer⟩ 最初等于 0。

我们的想法是可以通过选择一个唯一的前缀并将其后缀化为递增整数来生成无限唯一的标签；`\g_autolabel_int` 将保存这个整数。

下一行：

```
\NewDocumentCommand{\AutoLabel}{}{
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}
} 
```

在这里，我们声明了一个用户可见的命令，使用 `\NewDocumentCommand`（LaTeX3 版本的 `\newcommand`，使用起来更加友好，关于这点你可以阅读 `texdoc xparse`）。因此，无需遵循 LaTeX3 的命名约定。声明进一步指定它不从用户那里接受任何参数，并在调用时留下两条指令：`\g_autolabel_int` 变量的全局递增和 `\exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}`。关于 `\exp_args:Ne`，`interface3.pdf` 有以下说明：

> 此函数吸收两个参数（⟨function⟩ 名称和 ⟨tokens⟩），并穷尽地展开 ⟨tokens⟩。结果被插入花括号中重新插入 ⟨function⟩ 到输入流中。因此 ⟨function⟩ 可能需要多于一个参数：其他所有参数都保持不变。

理想情况下，我们希望有一个标签函数 `\label:n` 和一个变体 `\label:e`，在接受实际标签文本之前会展开其输入内容。但实际上，由于 `label` 是 LaTeX2e 的命令（见 `texdoc latex2e`，第 7.1 章），所以我们使用 `LaTeX3` 的 `\exp_args:Ne`，保持第一个记号（`\label`）不变，而其后的花括号内的记号不会完全展开。然后将这个记号列表作为 `\label` 的参数传递。

在标记列表内部，我们找到了`autolabelprefix- \int_use:N \g_autolabel_int`：因为`\int_use:N`将“恢复一个⟨integer⟩的内容并将其直接放入输入流”，我们在完全展开后得到了我们唯一的标签形式

```
autolabelprefix-⟨label number⟩ 
```

最后，我们有引用此标签的命令声明：

```
\NewDocumentCommand{\AutoRef}{}{
    \exp_args:Ne \ref {autolabelprefix-\int_use:N \g_autolabel_int}
} 
```

它与`\AutoLabel`非常相似，唯一的区别是相关的LaTeX2e函数现在是`\ref`，而不是`\label`。由于我们知道生成的标签形式，我们可以在引用时通过再次使用`\g_autolabel_int`变量和`\exp_args:Ne`来重建标签。

## 多个标签

上述解决方案很好用，但如果我们希望在*两个*声明后引用两个方程式时，它很快就会崩溃。

```
% snip
\begin{equation} \AutoLabel
    A \implies B
\end{equation}
\begin{equation} \AutoLabel
    B \implies C
\end{equation}

Eqs.~\AutoRef\ and \AutoRef\ imply $A \implies C$. 
```

将不起作用，因为输出将会读取

```
Eqs. (2) and (2) imply A ⇒ C. 
```

我们可以通过分别计数标签和引用来简单处理这个问题。

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\NewDocumentCommand{\AutoLabel}{} {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
\NewDocumentCommand{\AutoRef}{} {
    \int_gincr:N \g_autoref_int
    \exp_args:Ne \ref{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autoref_int}
}
\ExplSyntaxOff 
```

我们已经偷偷地将`\c_autoprefix_tl`作为常量标记列表插入，只因为它在源代码中看起来比重复的任意常量字符串更加美观，为了可读性，我们还取消了作用域。否则，一切仍然与之前的定义非常相似。

当使用例如`amsmath`包中的`gather`环境时，事情开始变得混乱起来：

```
% snip
\begin{gather} 
    A \implies B \AutoLabel \\
    B \implies C \AutoLabel
\end{gather} 
```

产生

```
Eqs. ?? and ?? imply A ⇒ C. 
```

发生了什么？LaTeX3确实具有一些控制台调试功能，例如`tl_show:N`等命令，但在这里更容易采用LaTeX的版本进行打印调试：直接在文本中放置值。因此，我们修改了`\AutoLabel`的定义，不仅标记方程式，还将标签名称放入文本流：

```
% snip
\NewDocumentCommand{\AutoLabel}{} {
    \int_gincr:N \g_autolabel_int
    % 👇 new addition
    {\tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int}
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
% snip 
```

再次编译，我们发现方程式现在读取为：

```
 A ⇒ Bautolabelprefix−3   (1)
    B ⇒ Cautolabelprefix−4   (2) 
```

哎呀？标签1和2去哪儿了？问题似乎只发生在`amsmath`环境中，稍微查看一下它的文档（`texdoc amsmath`）揭示了以下段落：

> `ifmeasuring@` - 所有显示环境都会进行两次排版——一次在“测量”阶段，然后在“生成”阶段；`\ifmeasuring@`将用于确定我们处于哪种情况，以便我们可以采取适当的行动。
> 
> `\newif\ifmeasuring@`

糟糕了。我们的`\AutoLabel`函数被调用了两次：一次在`amsmath`用于执行测量的盒子中，然后被丢弃，然后在实际排版发生时再次调用。更糟糕的是，`amsmath`正在使用一个Plain TeX布尔值`\ifmeasuring@`，*并且*布尔值的名称中包含一个`@`。这里有`interface3.pdf`关于其布尔值和Plain TeX的布尔值的相关信息：

> TeXhackers注意：与Plain TeX、LaTeX2e等不同，`bool`数据类型未使用`\iffalse`/`\iftrue`原语实现。程序员不应该基于特定的实现期望使用`bool`开关。

纯TeX布尔值非常棘手，如果布尔测试看起来与以下内容不同，可能会出现问题

我们可能可以将一些参数给予`\a`或`\b`，但不能超过这些；当然，像`\ifbool\a\b\c\else\d\e\f\fi`这样的东西肯定会给你带来麻烦。因此，我们的第一步是将整个`\AutoLabel`移入其自身的单个宏中：

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
\NewDocumentCommand{\AutoLabel}{}{
    \autolabel:
}
\NewDocumentCommand{\AutoRef}{}{
    \int_gincr:N \g_autoref_int
    \exp_args:Ne \ref{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autoref_int}
}
\ExplSyntaxOff 
```

现在，我们只需在`\AutoLabel`中测试`\ifmeasuring@`，并相应地处理；也就是说，如果正在测量，我们根本不想做任何事情。但请注意，LaTeX3和LaTeX2e对`@`的处理方式不同，LaTeX3根本不会正确解释`\ifmeasuring@`（它不期望宏名称中有`@`）。我们可以通过首先使用`c`类型参数和辅助定义来解决这个问题：

```
% snip
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc \amsmath_ifmeasuring {ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
% snip 
```

我们正在使用`\cs_set_eq:NN`的`Nc`变体，关于这一点，`interface3.pdf`告诉我们：

> 全局创建⟨控制序列1⟩并将其设置为与⟨控制序列2⟩或⟨标记⟩相同的含义。第二个控制序列可以在不影响副本的情况下进行修改。

这样做合适吗？绝对不是。这是有史以来最不诅咒的LaTeX宏吗？一点也不，而且它还有效。重新运行编译步骤，你会发现文档现在正确地读取

```
 A ⇒ B               (1)
        B ⇒ C               (2)

Eqs. 1 and 2 imply A ⇒ C. 
```

## 一些最终的改进

唯一缺少的是重复引用同一个方程的能力。目前，还没有办法排版类似

```
Eqs. 1 and 2 imply A ⇒ C, but eq. 1 does not imply B ⇒ A. 
```

让我们来修复这个问题。具体来说，让我们修改`\AutoRef`以接受一个可选的参数。如果参数存在，并且等于`n`，那么该命令应该引用之前的第`n`个方程。因此，我们之前的示例将对应于类似于以下内容

```
Eqs.~\AutoRef\ and \AutoRef imply $A \implies C$, but \AutoRef[2] does not imply $B \implies A$. 
```

我将再次给出最终答案，然后分解任何新元素：

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \c_autoprefix_tl \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \IfValueF{#1}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \IfValueTF{#1}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \IfValueT{#1}
        {\int_sub:Nn \l_tmpa_int {#1 - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
\ExplSyntaxOff 
```

首先注意，我还保护了`\g_autoref_int`计数器免受测量中的双重增加影响 – 这是我们先前忽略的，但在`\AutoRef`被调用于`amsmath`环境中时是合理的。否则，与先前定义相比的主要区别在于在声明`\AutoRef`时包含了可选（o）参数。当没有为这些参数提供值时，它们将采用特殊的标志值（通常在文档中用`-NoValue-`表示）。`xparse`提供了很好的`IfValue(TF)`宏来处理这些情况，分别处理参数有值和没有值的情况。

因此，当我们发现用户传入了一些可选值时，我们计算相应的“绝对标签数”；这由当前的自动标签数减去用户给出的数再减一得出（换句话说：如果`g_autolabel_int`当前为`2`，意味着`autoprefix-1`和`-2`已经定义，用户传入`\AutoRef[2]`，则他们引用的是`autoprefix-1`，即`1 = 2-(2-1)`）。

另一方面，如果用户没有传入任何值，则他们引用最近的标签；与以前一样，我们增加`g_autoref_int`，并设置`\l_tmpa_int`以保存这个数字：

```
\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi
\int_set_eq:NN \l_tmpa_int \g_autoref_int 
```

然后，最后一行正确地引用了最近的标签：

```
\exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int} 
```

这个*几乎*能正常工作，除了以下边缘情况：您期望以下代码会产生什么结果？

```
\begin{equation} \AutoLabel
    A = A
\end{equation}

\AutoRef[1] constitutes a ``tautology''.

\begin{equation}
    A = B
\end{equation}

Eq.~\AutoRef{} does not constitute a tautology. 
```

就目前而言，对`\AutoRef[1]`的第一次调用正确地引用了第一个方程的标签，但它*不会*推进`\g_autoref_int`，因为给定了一个值。因此，在第二段落中第二次调用`\AutoRef`时，它再次引用第一个方程。然而，这正是我们在以下情况下可能预期到的行为（抱歉，这更多是合成的情况）：

```
\begin{gather}
    A \AutoLabel \\
    B \AutoLabel \\
    C \AutoLabel
\end{gather}

Eq~\AutoRef[1] and \AutoRef{} and \AutoRef{}. 
```

确实，在这里，我们*不*希望`\AutoRef[1]`推进`\g_autoref_int`。

底线是，为了得到预期的行为，调用`\AutoRef[1]`应该与简单调用`\AutoRef`没有区别；但仅在这两种情况会产生相同结果的情况下。目前并非如此。但是！LaTeX3已经为我们提供了一些非常全面的算术工具。对`\AutoRef`的以下修改已经足够：

```
% snip
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    % 👇 main difference
    \tl_set:Nn \l_tmpb_tl {#1}
    \IfValueT{#1}
        {\int_compare:nNnT {#1}={1} {
            \int_compare:nNnT {\g_autoref_int}={\g_autolabel_int - 1} 
                {\tl_set:NV \l_tmpb_tl \c_novalue_tl}}}
    % 👆
    % 👇 But notice also the use of \l_tmpb_tl below, now:
    \exp_args:Ne \IfValueF{\tl_use:N \l_tmpb_tl}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \exp_args:Ne \IfValueTF{\tl_use:N \l_tmpb_tl}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \exp_args:Ne \IfValueT{\tl_use:N \l_tmpb_tl}
        {\exp_args:NNe \int_sub:Nn \l_tmpa_int {\tl_use \l_tmpb_tl - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
% snip 
```

这个想法并不完全不同；但现在，我们不是直接处理可选参数（`#1`），而是首先将其分配给`\l_tmpb_tl`。然后，如果我们发现用户确实传递了某个值，但该值为1，并且会导致与未提供任何值相同，我们将空值放入`\l_tmpb_tl`。因此，在这种情况下，该命令的其余部分被处理，就好像用户没有提供值一样。

使用`\l_tmpb_tl`代替`#1`需要更多注意，特别是要确保在测试之前扩展`\l_tmpb_tl`的值，但没有什么特别的。

因此，我们有了我们最终的LaTeX3宏集：

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \c_autoprefix_tl \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \tl_set:Nn \l_tmpb_tl {#1}
    \IfValueT{#1}
        {\int_compare:nNnT {#1}={1} {
            \int_compare:nNnT {\g_autoref_int}={\g_autolabel_int - 1} 
                {\tl_set:NV \l_tmpb_tl \c_novalue_tl}}}
    \exp_args:Ne \IfValueF{\tl_use:N \l_tmpb_tl}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \exp_args:Ne \IfValueTF{\tl_use:N \l_tmpb_tl}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \exp_args:Ne \IfValueT{\tl_use:N \l_tmpb_tl}
        {\exp_args:NNe \int_sub:Nn \l_tmpa_int {\tl_use \l_tmpb_tl - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
\ExplSyntaxOff 
```

虽然在[LaTeX包的非常庞大历史](https://ctan.math.illinois.edu/)中算是一个相当谦逊的贡献，但它仍然作为讨论LaTeX3宏编写的重要点的基础，例如语法差异，标记列表操作，受控展开，变量和函数，布尔测试以及LaTeX2e接口的基础。不算太糟糕。希望本帖也能作为一个足够好的介绍，让你可以直接参考重要的参考文档，比如`interface3.pdf`。

就个人而言，我在实践中也发现这个宏非常有用，并且我一直在我的科学写作中使用它。我还不知道Physical Review是否会对此感到不悦。

作为告别礼，并将感兴趣的用户指向属性列表，这里有一个给读者的练习：Physical Review强制要求的`revtex4-2`文档类不支持`\footnotetext`和`\footnotenumber`的使用。这意味着任何脚注文本实际上必须位于正文源代码的中间。您能写出`\FootnoteLater`和`\FootnoteNow`来纠正这一点吗？

* * *

**给HackerNews读者的附言：** 我喜欢把这些文章提交到HN，因为我觉得平均的HN用户就是我的目标读者群体。但是，1.众所周知，HN在某些回复中可能非常可预测（我认为这本质上是一种模因效应），2.之前我在HN上达到FP后也经历过一些意外的经历。所以，请把这视为我预期会被提到的一些点的预防性回应。如果你从HN进来并看到有人未能考虑到这些答案，你会知道他们甚至没有完整地阅读整篇文章。

1.  很奇怪我竟然需要说这个，但请不要跟踪我并给我发邮件到我的个人邮箱。这确实发生过，莫名其妙的事情。如果你想通过邮件联系我，请发邮件到 `miguelmurca+autoref [æt] cumperativa.xyz`。

1.  再次，很奇怪我需要指出这一点，但请不要假设我的国籍或语言。这篇文章是用英文写的。如果你想写信给我，请用英文写。

1.  是的，我们知道 [typst](https://github.com/typst/typst)。我觉得它很酷，但是C++没有取代C，Rust也没有取代C++，Typst也不太可能取代LaTeX。同样，很多人知道 [LuaTeX](https://www.luatex.org/)，但是一个有40多年历史的系统的根深蒂固是不可低估的。不管怎样，我还是支持 `typst`，希望它能找到自己的位置。如果他们真的想取代TeX，一个好的起点就是提供从 `typst` 到TeX的编译工具链。

1.  实际上，有很多理由为什么有些人不愿意只使用你最喜欢的Markdown加上一点点的TeX，其中至少一个原因是不是每个人都只是在做笔记，还因为不是每个人（我敢说，大多数人）使用LaTeX的人都是计算机科学/技术专业的。另外，从纯个人角度来看，如果你用Markdown+TeX来做课堂笔记，要么课程太简单了，要么你自己给自己增加了很多困难。

1.  是的，LaTeX看起来确实又丑又过时。它确实很老了。它是一个不断发展的、小众的、非常怪异的东西，也因此受到影响。然而，它似乎有它的位置。但是，如果你把它看作一个奇怪的编程语言，为什么这会成为问题呢？

1.  尽管我努力保持正确，但我可能会犯一些错误。HN上有很多人真的很擅长LaTeX，我期望他们指出任何错误（我会非常感激，并会及时更正）。但归根结底，LaTeX并不是我的主要工作，我觉得这篇文章能够给普通程序员带来对LaTeX的介绍，这比任何可能的小错误更重要，可以轻易通过查阅官方资料进行修正。

1.  是的，现在是`$YEAR`，我们仍然在制作PDF文档。同样的，出于历史原因，加上大多数从事数学工作的人并不一定对计算机非常感兴趣。

1.  99% 的时间，当你在写 LaTeX 时，你实际上不需要知道这篇博文中所包含的任何内容。

1.  (La)TeX 没有语法。解析 TeX 是图灵完备的。这并不意味着你不能为 TeX 写一个语法，粗略地说，特别是对于“许多”数学表达式。我怀疑这就是许多简易 TeX 相关程序所做的事情。

* * *
