<!--yml

类别：未分类

日期：2024年05月27日 14:50:18

-->

# PEP 701 – f-string 语法形式化 | peps.python.org

> 来源：[https://peps.python.org/pep-0701/](https://peps.python.org/pep-0701/)

# PEP 701 – f-string 语法形式化

作者：

Pablo Galindo <pablogsal at python.org>, Batuhan Taskaya <batuhan at python.org>, Lysandros Nikolaou <lisandrosnik at gmail.com>, Marta Gómez Macías <cyberwitch at google.com>

讨论至：

[讨论线程](https://discuss.python.org/t/pep-701-syntactic-formalization-of-f-strings/22046)

状态：

已接受

类型：

标准轨道

创建日期：

2022年11月15日

Python 版本：

3.12

发布历史：

[2022年12月19日](https://discuss.python.org/t/pep-701-syntactic-formalization-of-f-strings/22046 "讨论线程")

决议：

[讨论消息](https://discuss.python.org/t/pep-701-syntactic-formalization-of-f-strings/22046/119)

* * *

本文提议放宽最初在[PEP 498](../pep-0498/ "PEP 498 – Literal String Interpolation")中制定的一些限制，并为可以直接集成到解析器中的 f-string 提供正式的语法。所提议的 f-string 语法形式化将在解析和解释 f-string 方面产生一些小的副作用，为终端用户和库开发人员提供了相当多的优势，同时显著降低了专用于解析 f-string 的代码的维护成本。

当最初在[PEP 498](../pep-0498/ "PEP 498 – Literal String Interpolation")中引入 f-string 时，并未提供 f-string 的正式语法。此外，规范包含几个限制，以便可以将 f-string 的解析实现到 CPython 中，而无需修改现有的词法分析器。这些限制先前已被认识到，并曾尝试在[PEP 536](../pep-0536/ "PEP 536 – Final Grammar for Literal String Interpolation")中取消，但是[这些工作从未得到实施](https://mail.python.org/archives/list/python-dev@python.org/thread/N43O4KNLZW4U7YZC4NVPCETZIVRDUVU2/#NM2A37THVIXXEYR4J5ZPTNLXGGUNFRLZ)。其中一些限制（最初由[PEP 536](../pep-0536/ "PEP 536 – Final Grammar for Literal String Interpolation")收集）包括：

1.  不可能在表达式部分内部使用引号字符来界定 f-string：

    ```
    >>> f'Magic wand: {  bag['wand']  }'
     ^
    SyntaxError: invalid syntax 
    ```

1.  以前考虑的一个绕过方法会导致执行代码中的转义序列，并且在 f-string 中被禁止使用：

    ```
    >>> f'Magic wand { bag[\'wand\'] } string'
    SyntaxError: f-string expression portion cannot include a backslash 
    ```

1.  即使在多行 f-string 中也禁止评论：

    ```
    >>> f'''A complex trick: {
    ... bag['bag']  # recursive bags!
    ... }'''
    SyntaxError: f-string expression part cannot include '#' 
    ```

1.  许多其他语言中都有可用的表达式任意嵌套而不扩展转义序列的字符串插值方法。例如：

    ```
    # Ruby
    "#{ "#{1+2}" }"

    # JavaScript
    `${`${1+2}`}`

    # Swift
    "\("\(1+2)")"

    # C#
    $"{$"{1+2}"}" 
    ```

从语言用户的角度看，这些限制毫无意义，并且可以通过给 f-string 文字一个没有例外的常规语法并使用专用解析代码来取消这些限制。

f-string的另一个问题是，当前CPython的实现依赖于将f-string标记为`STRING`标记，并对这些标记进行后处理。这带来了以下问题：

1.  它给CPython解析器增加了相当大的维护成本。这是因为解析代码需要手动编写，这在历史上导致了大量的不一致性和错误。在C中手动编写和维护解析代码一直被认为是容易出错和危险的，因为它需要处理原始词法分析器缓冲区的大量手动内存管理。

1.  f-string解析代码无法使用新的改进的错误消息机制，这些机制最初在[PEP 617](../pep-0617/ "PEP 617 – CPython的新PEG解析器")中引入，为此感到非常高兴。这些错误消息的改进受到了极大的欢迎，但不幸的是，f-string无法从中受益，因为它们在解析机制的一个独立部分中解析。这特别令人遗憾，因为f-string的几个语法特性可能会因为表达式部分内部发生的不同隐式标记化而令人困惑（例如`f"{y:=3}"`不是一个赋值表达式）。

1.  其他Python实现无法知道他们是否正确实现了f-string，因为与其他语言特性相反，它们不是[官方Python语法](https://docs.python.org/3/reference/lexical_analysis.html#f-strings "(在Python v3.12)")的一部分。这一点很重要，因为几个知名的替代实现正在使用CPython的PEG解析器，[如PyPy](https://foss.heptapod.net/pypy/pypy/-/commit/fe120f89bf07e64a41de62b224e4a3d80e0fe0d4/pipelines?ref=branch%2Fpy3.9)，和/或者基于官方PEG语法构建他们的语法。由于f-string使用一个单独的解析器，这些替代实现无法利用官方语法和从语法中得到的错误消息的改进。

这个提案的一个版本最初在[Python-Dev讨论过](https://mail.python.org/archives/list/python-dev@python.org/thread/54N3MOYVBDSJQZTU6MTCPLUPIFSDN5IS/#SAYU6SMP4KT7G7AQ6WVQYUDOSZPKHJMS)，并且[在2022年Python语言峰会上做过演讲](https://pyfound.blogspot.com/2022/05/the-2022-python-language-summit-f.html)，得到了热烈的反响。

通过建立在新的Python PEG解析器（[PEP 617](../pep-0617/ "PEP 617 – CPython的新PEG解析器")）之上，本PEP建议重新定义“f-strings”，特别强调字符串部分与表达式（或替换，`{...}`）部分的明确分离。[PEP 498](../pep-0498/ "PEP 498 – 字面字符串插值")总结了“f-strings”的语法部分如下：

> 在 Python 源代码中，f-string 是一个字面字符串，前缀为‘f’，其中包含大括号内的表达式。这些表达式将被它们的值替换。

然而，[PEP 498](../pep-0498/ "PEP 498 – Literal String Interpolation") 也包含了一份正式的排除列表，列出了表达式组件中可以包含或不包含的内容（主要是由于现有解析器的限制）。通过清晰地建立正式的语法，我们现在还能够定义 f-strings 的表达式组件为真正的“任何适用的 Python 表达式”（在特定上下文中），而不受我们实现细节所施加的限制束缚。

对于 Python 程序员来说，上述的正式化努力和前提也因其简化和消除晦涩限制的能力而带来了显著好处。这减少了 f-string 字面值的心理负担和认知复杂性（以及 Python 语言总体上的复杂性）。

1.  表达式组件可以包含任何普通 Python 表达式可以包含的字符串字面值。这打开了在 f-strings 的表达式组件中嵌套字符串字面值（格式化或非格式化）的可能性，使用相同的引号类型（及其长度）：

    ```
    >>> f"These are the things: {", ".join(things)}"

    >>> f"{source.removesuffix(".py")}.c: $(srcdir)/{source}"

    >>> f"{f"{f"infinite"}"}" + " " + f"{f"nesting!!!"}" 
    ```

    这种“特性”并非所有人都认为是理想的，一些用户认为这种写法难以阅读。有关此问题不同观点的讨论，请参阅关于引号重用的考虑部分。

1.  另一个大多数人觉得不直观的问题是，在 f-string 的表达式组件中不支持反斜杠。一个经常出现的例子是在表达式部分中包含换行符，以用于连接容器。例如：

    ```
    >>> a = ["hello", "world"]
    >>> f"{'\n'.join(a)}"
    File "<stdin>", line 1
     f"{'\n'.join(a)}"
     ^
    SyntaxError: f-string expression part cannot include a backslash 
    ```

    对此的一个常见解决方案是将换行符分配给一个中间变量，或者在创建 f-string 之前预先创建整个字符串：

    ```
    >>> a = ["hello", "world"]
    >>> joined = '\n'.join(a)
    >>> f"{joined}"
    'hello\nworld' 
    ```

    现在新的 PEG 解析器可以轻松支持，因此在表达部分允许使用反斜杠只是很自然的。

    ```
    >>> a = ["hello", "world"]
    >>> f"{'\n'.join(a)}"
    'hello\nworld' 
    ```

1.  在本文档提出的更改之前，f-strings 的嵌套方式并没有明确的限制，但事实上，由于字符串引号无法在 f-strings 的表达式组件中重复使用，导致无法任意嵌套 f-strings。事实上，这是可以编写的最嵌套的 f-string：

    ```
    >>> f"""{f'''{f'{f"{1+1}"}'}'''}"""
    '2' 
    ```

    正如这个 PEP 允许在 f-strings 的表达式组件中放置**任何**有效的 Python 表达式一样，现在可以重新使用引号并因此可以任意嵌套 f-strings：

    ```
    >>> f"{f"{f"{f"{f"{f"{1+1}"}"}"}"}"}"
    '2' 
    ```

    虽然这只是允许任意表达式的一个结果，但本 PEP 的作者并不认为这是一个基本的好处，我们已经决定语言规范将不会明确规定这种嵌套可以是任意的。这是因为允许任意深度的嵌套会给词法分析器的实现带来很多额外复杂性（特别是在词法分析器/语法分析器流水线需要允许“反解标记化”以支持“f-string 调试表达式”时，这种允许任意嵌套的情况尤为紧张）。因此，实现可以自由地对嵌套深度施加限制，如果需要的话。请注意，这并不是一个罕见的情况，因为 CPython 实现已经在各处施加了几个限制，包括括号和方括号的嵌套深度限制，在 `if` 语句中的分支数限制，在星号解包表达式中表达式数限制等等。

f-string 的正式提议 PEG 语法规范为（详见 [PEP 617](../pep-0617/ "PEP 617 – CPython 的新 PEG 解析器") 中的语法细节）：

```
fstring
    | FSTRING_START fstring_middle* FSTRING_END
fstring_middle
    | fstring_replacement_field
    | FSTRING_MIDDLE
fstring_replacement_field
    | '{' (yield_expr | star_expressions) "="? [ "!" NAME ] [ ':' fstring_format_spec* ] '}'
fstring_format_spec:
    | FSTRING_MIDDLE
    | fstring_replacement_field 
```

新的标记（`FSTRING_START`，`FSTRING_MIDDLE`，`FSTRING_END`）在[本文档的后续部分](#new-tokens)中定义。

本 PEP 将 f-string 允许的嵌套级别留给实现决定（在其他 f-string 的表达式部分中使用 f-string 嵌套），但**规定嵌套层级的下限为 5 级**。这是为了确保用户能够合理期望能够嵌套具有“合理”深度的 f-strings。这个 PEP 暗示限制嵌套**不是语言规范的一部分**，但也**不强制要求任意嵌套**。

类似地，这个 PEP 将格式说明符中的表达式嵌套级别留给实现决定，但**规定嵌套层级的下限为 2 级**。这意味着以下内容始终应该是有效的：

但是，以下内容可以有效或无效，具体取决于实现：

新的语法将保留当前实现的抽象语法树（AST）。这意味着，使用 f-string 的现有代码在本 PEP 引入的改变下不会引入语义上的变化。

自 Python 3.8 起，可以使用 `=` 运算符来使用 f-string 调试表达式。例如：

```
>>> a = 1
>>> f"{1+1=}"
'1+1=2' 
```

这些语义并未在 PEP 中正式引入，而是作为特例在当前字符串解析器中实现（[bpo-36817](https://bugs.python.org/issue?@action=redirect&bpo=36817)）并在[f-string 词法分析部分](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)中有所记录。

此功能不受本PEP中提出的更改影响，但重要的是指出，正式处理此功能要求词法分析器能够“解标记化”f-string的表达式部分。对于当前的字符串解析器来说，这不是问题，因为它可以直接操作字符串标记内容。然而，将此功能合并到给定的解析器实现中，要求词法分析器跟踪f-string表达式部分的原始字符串内容，并在为f-string节点构造解析树时向解析器提供这些内容。仅仅进行“解标记化”是不够的，因为按当前规范，f-string调试表达式保留表达式中的空白，包括`{`和`=`字符之后的空格。这意味着f-string表达式部分的原始字符串内容必须保持完整，而不仅仅是相关的标记。

如何处理这个问题是由解析器/词法分析器实现决定的。

引入了三个新的标记：`FSTRING_START`、`FSTRING_MIDDLE` 和 `FSTRING_END`。不同的词法分析器可能有不同的实现方式，可能比本文提出的更高效，具体取决于特定实现的上下文。然而，以下定义将作为CPython公共API的一部分（例如`tokenize`模块），也作为参考，使读者能更好地理解所提议的语法更改以及标记的使用方式：

+   `FSTRING_START`: 这个标记包括f-string前缀（`f`/`F`/`fr`）和开头的引号。

+   `FSTRING_MIDDLE`: 这个标记包括字符串中的一部分文本，不是表达式部分，也不是开放或关闭大括号。这可以包括在开头引号和第一个表达式大括号（`{`）之间的文本，两个表达式大括号（`}` 和 `{`）之间的文本，以及最后一个表达式大括号（`}`）和结束引号之间的文本。

+   `FSTRING_END`: 这个标记包括结束的引号。

这些标记始终是字符串部分，并且从语义上等同于带有指定限制的`STRING`标记。这些标记必须由词法分析器在词法分析f-strings时生成。这意味着**标记生成器不能再为f-strings生成单个标记**。词法分析器如何生成此标记**未指定**，因为这将在很大程度上取决于每个实现（即使标准库中的Python版本的词法分析器与PEG解析器使用的版本也有所不同）。

例如：

```
f'some words {a+b:.3f} more words {c+d=} final words' 
```

将被标记为：

```
FSTRING_START - "f'"
FSTRING_MIDDLE - 'some words '
LBRACE - '{'
NAME - 'a'
PLUS - '+'
NAME - 'b'
OP - ':'
FSTRING_MIDDLE - '.3f'
RBRACE - '}'
FSTRING_MIDDLE - ' more words '
LBRACE - '{'
NAME - 'c'
PLUS - '+'
NAME - 'd'
OP - '='
RBRACE - '}'
FSTRING_MIDDLE - ' final words'
FSTRING_END - "'" 
```

而`f"""some words"""`将被简单地标记为：

```
FSTRING_START - 'f"""'
FSTRING_MIDDLE - 'some words'
FSTRING_END - '"""' 
```

[`tokenize`](https://docs.python.org/3/library/tokenize.html#module-tokenize) 模块将被调整为在解析f-strings时如前述部分所述生成这些标记，因此工具可以利用这种新的标记化模式，避免必须实现自己的f-string词法分析器和解析器。

一种现有的词法分析器可以被调整以发出这些 token 的方法是，将“词法分析器模式”堆栈或不同词法分析器的堆栈整合在一起。这是因为在遇到 f-string 开始 token 时，词法分析器需要从“常规 Python 词法分析”切换到“f-string 词法分析”，并且由于 f-strings 可能是嵌套的，因此需要保留上下文直到 f-string 结束。此外，在 f-string 表达式部分内部的“词法分析器模式”需要表现为“常规 Python 词法分析”的“超集”（因为它需要能够在遇到`}`结束符以及处理 f-string 格式和调试表达式时切换回 f-string 词法分析）。供参考，这里是修改类似 CPython 的标记器以发出这些新 token 的算法草稿：

1.  如果词法分析器检测到 f-string 的开始（通过检测到字母‘f/F’和可能的引号之一），则继续前进直到检测到有效的引号（`"`, `"""`, `'` 或 `'''`）并发出一个 `FSTRING_START` token，并捕获其中的内容（‘f/F’ 和起始引号）。为“F-string 词法分析”推送一个新的词法分析器模式到词法分析器模式栈中。转到步骤 2。

1.  继续消耗 token，直到遇到以下情况之一：

    +   遇到与开引号相匹配的闭引号。

    +   如果处于“格式说明符模式”（见步骤 3），则遇到左大括号（`{`）、右大括号（`}`）或换行符（`\n`）。

    +   如果不处于“格式说明符模式”（见步骤 3），则遇到不是紧随另一个开/闭括号的左大括号（`{`）或右大括号（`}`）。

    在所有情况下，如果字符缓冲区不为空，则发出一个 `FSTRING_MIDDLE` token，捕获到目前为止的内容，但将任何双开/闭括号转换为单个开/闭括号。然后，根据遇到的字符进行如下操作：

    +   如果遇到与开引号匹配的闭引号，则转到步骤 4。

    +   如果遇到一个开括号（紧随另一个开括号之后），则转到步骤 3。

    +   如果遇到一个关闭括号（不是紧随另一个关闭括号之后），则发出一个关闭括号的 token 并转到步骤 2。

1.  为“f-string 内部的常规 Python 词法分析”推送一个新的词法分析器模式到词法分析器模式栈中，并使用它进行词法分析。此模式将按照“常规 Python 词法分析”词法分析，直到遇到与进入 f-string 部分时推送的开括号标记具有相同嵌套级别的 `:` 或 `}` 字符为止。使用此模式，发出 token 直到达到停止点之一。当发生这种情况时，发出相应的停止字符遇到的 token，并从词法分析器模式栈中弹出当前的词法分析器模式并转到步骤 2。如果停止点是 `:` 字符，则进入“格式说明符”模式的步骤 2。

1.  发出一个 `FSTRING_END` token，并捕获到目前为止的内容，并弹出当前的词法分析器模式（对应于“F-string 词法分析”），然后返回“常规 Python 模式”。

当然，正如前面提到的，对于任意的标记器来说，精确说明如何实现这一点是不可能的，因为它将取决于特定实现和词法分析器的性质。

所有在 PEP 中提到的对 f-string 文本的限制都被解除，具体解释如下：

+   表达式部分现在可以包含使用与界定 f-string 文本相同类型的引号界定的字符串。

+   反斜杠现在可以像在 Python 代码的任何其他地方一样出现在表达式中。在 f-string 文本中嵌套的字符串在评估内部字符串时会展开转义序列。

+   现在允许在表达式括号内换行。这意味着现在可以这样写：

    ```
    >>> x = 1
    >>> f"___{
    ...  x
    ... }___"
    '___1___'

    >>> f"___{(
    ...  x
    ... )}___"
    '___1___' 
    ```

+   可以在 f-string 的表达式部分使用 `#` 字符进行注释。请注意，注释要求表达式部分的闭合括号 (`}`) 必须在不同行上才能生效，否则它会被视为注释的一部分而被忽略。

这里提出的语法的一个后果是，正如上面提到的，f-string 表达式现在可以包含使用相同类型引号来界定的字符串。例如：

```
>>> f" something {  my_dict["key"]  } something else " 
```

在 [这个 PEP 的讨论线程中](https://discuss.python.org/t/pep-701-syntactic-formalization-of-f-strings/22046)，关于这个方面已经提出了几个问题，我们希望在这里收集它们，因为在接受或拒绝这个 PEP 时应考虑到这些问题。

其中一些反对意见包括：

+   许多人发现在同一个字符串中重复使用引号会让人感到困惑和难以阅读。这是因为允许引号重用将违反当前 Python 的一个属性：字符串由两个连续的相同类型引号完全界定，这本身是一个非常简单的规则。引号重用可能更难让人类解析，从而导致代码不太易读，因为引号字符既用作开始标记又用作结束标记（与其他分隔符相反）。

+   一些用户提出了引号重用可能会破坏依赖简单机制来检测字符串和 f-string 的词法分析器和语法高亮工具的担忧，例如正则表达式或简单的分隔符匹配工具。引入 f-string 中的引号重用将使得保持这些工具的工作变得更加棘手，或者完全破坏这些工具（例如，正则表达式无法解析带有分隔符的任意嵌套结构）。标准库中包含的 IDLE 编辑器就是一个可能需要一些工作才能正确应用语法高亮到 f-string 的工具的例子。

下面是一些支持的论点：

+   许多允许类似语法结构（通常称为“字符串插值”）的语言允许引号重用和任意嵌套。这些语言包括 JavaScript、Ruby、C#、Bash、Swift 等等。许多语言允许引号重用的事实可以成为支持在 Python 中引入这一特性的有力论据。这是因为它会使语言对从其他语言转换过来的用户更加熟悉。

+   由于许多其他流行语言允许在字符串插值构造中重用引号，这意味着支持这些语言的语法高亮编辑器已经具备了支持 Python 中具有引号重用的 f-strings 的必要工具。这意味着尽管需要更新处理 Python 的语法高亮的文件来支持这一新特性，但这并不是不可能或非常困难的。

+   允许引号重用的一个优点是它与其他语法的组合非常干净。有时这被称为“引用透明性”。一个例子是，如果我们有 `f(x+1)`，假设 `a` 是一个全新的变量，它应该与 `a = x+1; f(a)` 的行为相同。反之亦然。因此，如果我们有：

    ```
    def py2c(source):
        prefix = source.removesuffix(".py")
        return f"{prefix}.c" 
    ```

    应该预期，如果我们用其定义替换变量 `prefix`，答案应该是相同的：

    ```
    def py2c(source):
        return f"{source.removesuffix(".py")}.c" 
    ```

+   代码生成器（例如来自标准库的[ast.unparse](https://docs.python.org/3/library/ast.html#ast.unparse)）在当前形式上依赖复杂的算法，以确保 f-string 中的表达式在其被使用的上下文中适当地运行。这些非平凡的算法面临着挑战，例如找到未使用的引号类型（通过跟踪外部引号），并生成字符串表示，尽量不包含反斜杠。允许引号重用和反斜杠将极大简化处理 f-string 的代码生成器，因为可以在 f-string 内外都使用常规的 Python 表达式逻辑，而无需特殊处理。

+   限制引号重用将显著增加所提议更改的实现复杂性。这是因为它将迫使解析器在解析带有给定引号的 f-string 表达式部分时具有上下文，以便知道是否需要拒绝重用引号的表达式。在可以任意回溯的解析器（如 PEG 解析器）中传递这种上下文并不简单。如果考虑到 f-string 可以任意嵌套，因此可能需要拒绝多种引号类型，问题变得更加复杂。

    为了收集社区的反馈，已经启动了[一项民意调查](https://discuss.python.org/t/pep-701-syntactic-formalization-of-f-strings/22046/24)，以了解社区对 PEP 的这一方面的看法。

本 PEP 不会向 Python 语言引入任何向后不兼容的语法或语义更改。然而，[`tokenize`](https://docs.python.org/3/library/tokenize.html#module-tokenize "(在 Python v3.12 中)") 模块（标准库的准公共部分）需要更新以支持新的 f-string 令牌（以便让工具作者能够正确地令牌化 f-strings）。有关 `tokenize` 公共 API 如何受到影响的更多详细信息，请参见 [更改 tokenize 模块](#changes-to-the-tokenize-module)。

由于 f-strings 的概念已经在 Python 社区中普遍存在，因此用户基本上无需学习任何新内容。然而，由于正式化的语法允许一些新的可能性，因此需要将正式语法添加到文档中并详细解释，明确提到此 PEP 旨在避免混淆的构造。

还有利于为用户提供一个简单的框架，以理解可以放置在 f-string 表达式中的内容。在这种情况下，作者认为这项工作将使解释语言的这一方面更加简单，因为它可以总结为：

> 您可以在 f-string 表达式中放置任何有效的 Python 表达式。

通过这个 PEP 的改变，无需澄清字符串引号受限于与封闭字符串不同的引号，因为现在允许这样做：由于任意的 Python 字符串可以包含任何可能的引号选择，因此任何 f-string 表达式也可以。此外，无需澄清由于实现限制（如注释、换行符或反斜杠），某些事物在表达部分中是不允许的。

唯一的“惊人”区别在于，由于 f-strings 允许指定格式，允许在顶层使用 `:` 字符的表达式仍然需要用括号括起来。这对这项工作并不新鲜，但强调这一限制仍然存在非常重要。这允许更容易地修改摘要：

> 您可以在 f-string 表达式中放置任何有效的 Python 表达式，并且在顶层的 `:` 字符后的所有内容都将被识别为格式规范。

1.  尽管我们认为反对允许在 f-string 表达式中重用引号的可读性论点是有效且非常重要的，但我们决定在解析器级别上不拒绝 f-string 中引号的重用。原因是，这个 PEP 的基石之一是减少在 CPython 中解析 f-string 的复杂性和维护工作，这不仅会阻碍该目标的实现，甚至可能使实现比当前更复杂。我们认为禁止引号重用应该由 linter 和代码风格工具处理，而不是由解析器处理，就像今天处理语言中其他令人困惑或难以阅读的结构一样。

1.  我们决定不取消对某些表达式部分在顶层需要用括号包裹`':'`和`'!'`的限制，例如：

    ```
    >>> f'Useless use of lambdas: {  lambda  x: x*2 }'
    SyntaxError: unexpected EOF while parsing 
    ```

    这是因为这样做会引入相当多的复杂性，而实际上并没有多大好处。原因在于`:`字符通常用于分隔f-string的格式说明。当前，这个格式说明被标记为一个字符串。因为标记器必须将`:`右边的内容标记为字符串或者一串标记，这将不允许解析器区分不同的语义，因为这将要求标记器回溯并生成不同的标记集（即，首先尝试作为一串标记，如果失败，则尝试作为格式说明的字符串）。

    由于允许在顶层使用lambda和类似表达式并没有根本上的优势，我们决定保持这一限制，即如果需要的话，这些表达式必须使用括号括起来：

    ```
    >>> f'Useless use of lambdas: {  (lambda  x:  x*2)  }' 
    ```

1.  我们决定（目前）不允许使用转义大括号（`\{` 和 `\}`）以及`{{`和`}}`语法。尽管PEP的作者们认为允许转义大括号是个好主意，我们决定在这个PEP中不包括它，因为它对于这里提议的f-string的形式化并不是严格必要的，并且可以独立地在常规CPython问题中添加。

本文档位于公共领域或CC0-1.0-Universal许可证之下，以更加宽松的许可证为准。
