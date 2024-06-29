<!--yml

category: 未分类

日期：2024-05-27 14:30:33

-->

# 马尔可夫链是最初的语言模型

> 来源：[https://elijahpotter.dev/articles/markov_chains_are_the_original_language_models](https://elijahpotter.dev/articles/markov_chains_are_the_original_language_models)

# 马尔可夫链是最初的语言模型

![一个旧车内饰](img/bca28c3f1205854697bad5adf9ffa8c2.png)

> **注意：** 这篇文章是我高中毕业年为线性代数课程写的一篇文章的修订版（拼写、语法和布局有些调整）。因此，发布日期并不完全正确。

## AI热潮现在变得无聊了

我得出结论，个人大脑中当前AI炒作周期有四个阶段，至少在涉及大型语言模型方面是这样。至少，这是我经历的阶段。

### 第一阶段：惊奇

"哇！这太酷了！我可以像和真人一样与计算机交流！"

这是所有科幻幻想成真的地方。可能性似乎无限。我们现在可以坐下来放松了，对吗？

### 第二阶段：挫败

"嗯... 这并不像我最初想象的那么有效。"

看起来这种全新的技术确实只适用于那些无人愿意做的工作。它所能做的并没有给你带来太多价值。它经常出错，无法信任它处理任何事情。

### 第三阶段：困惑

第二阶段过后，你开始忘记这些。但炒作无处不在。你的朋友们提到它。你回家过节时，父母问你这个。甚至你的牙医也试图夸耀它的优点。

即使你对此有所行动，其他人却没有。这是否意味着你错了？

### 第四阶段：无聊

此时，新语言模型的发布速度已经超过了新JavaScript框架的速度（同样让人讨厌）。你想回到自己的起点，从头开始。你想要知道整个技术栈的自由。你不想要任何无效的魔法。

这就是我现在的状态。想回到我的起点。有些人喜欢修旧车，即使它们效率不高。与此同时，它们比新车更有趣。我决定研究一下马尔可夫链。

## 马尔可夫链

下面是我使用马尔可夫链实现的自动补全的演示。

尽管它是用Rust编写并编译为WebAssembly，但它并不特别高效。要找出原因，请继续查看页面底部我对实现的详细解释。

## 控制

您可以使用"选择词"或您的右箭头键[→]让系统选择下一个单词。或者，您可以轻击任何[可能的下一个单词]自行操作。

# 解释

马尔可夫链，以其发明者安德烈·马尔可夫命名，通常用于建模概率事件的序列。也就是说，无法以确定性方式建模的系统。

## 示例

爱丽丝在杂货店。她在那里每呆一小时，有70%的机会离开去天文馆。相反，她有30%的机会留下来。如果爱丽丝已经在天文馆，她有10%的机会离开去杂货店，90%的机会留在那里。我们可以将这些概率表示为一个表格，其中每一列属于起始位置，每一行属于结束位置：

|  |  |  |
| --- | --: | --: |
|  | 从杂货店开始 | 从天文馆开始 |
| 结束于杂货店 | 30% | 10% |
| 结束于天文馆 | 70% | 90% |

如果我们已经确切知道爱丽丝的位置，我们可以简单地进行表查找来预测她最可能的下一步动作。例如，我们*知道*她现在在杂货店。因此，通过查看第2行第1列，我们可以确信她有70%的概率下个小时会在天文馆。然而，如果我们不确定她的位置，或者我们想预测超过一小时的情况，这种方法就不适用了。如果我们不确定她当前的位置，我们该如何预测她的下一个动作？在后一种情况下，我们可能会把她当前的位置表达为另一个表格。

| 位置 | % 爱丽丝的存在 |
| --- | --: |
| 杂货店 | 25% |
| 天文馆 | 75% |

在这个新的可能性平面上，我们如何估计爱丽丝的位置？特别是，下个小时爱丽丝在天文馆的可能性有多大？因为有25%的概率爱丽丝在杂货店，我们将这个概率乘以她转移到天文馆的概率：<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>25</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>75</mn><mi mathvariant="normal">%</mi></mrow><annotation encoding="application/x-tex">25\% * 75\%</annotation></semantics></math>25%∗75%。接下来，我们将结果与她留在天文馆的概率乘以她留在那里的概率相加：<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>75</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>90</mn><mi mathvariant="normal">%</mi></mrow><annotation encoding="application/x-tex">75\% * 90\%</annotation></semantics></math>75%∗90%。总体来说，<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>25</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>75</mn><mi mathvariant="normal">%</mi><mo>+</mo><mn>75</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>90</mn><mi mathvariant="normal">%</mi><mo>=</mo><mn>85</mn><mi mathvariant="normal">%</mi></mrow><annotation encoding="application/x-tex">25\% * 75\% + 75\% * 90\% = 85\%</annotation></semantics></math>25%∗75%+75%∗90%=85%。将这些概率表示为表格：

| 下一个位置 | 计算 | % 爱丽丝的存在 |
| --- | --- | --: |
| 杂货店 | <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>25</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>30</mn><mi mathvariant="normal">%</mi><mo>+</mo><mn>75</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>10</mn><mi mathvariant="normal">%</mi></mrow><annotation encoding="application/x-tex">25\% * 30\% + 75\% * 10\%</annotation></semantics></math>25%∗30%+75%∗10% | 15% |
| 天文馆 | <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>25</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>70</mn><mi mathvariant="normal">%</mi><mo>+</mo><mn>75</mn><mi mathvariant="normal">%</mi><mo>∗</mo><mn>90</mn><mi mathvariant="normal">%</mi></mrow><annotation encoding="application/x-tex">25\% * 70\% + 75\% * 90\%</annotation></semantics></math>25%∗70%+75%∗90% | 85% |

-   那些注意力集中的人可能会注意到，这些运算看起来很像矩阵乘法。我们可以将这些可能的转换表示为矩阵<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>T</mi></mrow><annotation encoding="application/x-tex">T</annotation></semantics></math>T，并且将爱丽丝当前位置表示为向量<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">\vec{s}</annotation></semantics></math>s。

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>T</mi><mo>=</mo><mrow><mo fence="true">[</mo><mtable rowspacing="0.16em" columnalign="center center" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.3</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.1</mn></mstyle></mtd></mtr><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.7</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.9</mn></mstyle></mtd></mtr></mtable><mo fence="true">]</mo></mrow></mrow><annotation encoding="application/x-tex">T = \begin{bmatrix} 0.3 & 0.1 \\ 0.7 & 0.9 \end{bmatrix}</annotation></semantics></math>T=[0.30.7​0.10.9​] <math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mover accent="true"><mi>s</mi><mo>⃗</mo></mover><mo>=</mo><mrow><mo fence="true">[</mo><mtable rowspacing="0.16em" columnalign="center" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mi mathvariant="normal">.</mi><mn>25</mn></mrow></mstyle></mtd></mtr><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mrow><mi mathvariant="normal">.</mi><mn>75</mn></mrow></mstyle></mtd></mtr></mtable><mo fence="true">]</mo></mrow></mrow><annotation encoding="application/x-tex">\vec{s} = \begin{bmatrix} .25 \\ .75 \\ \end{bmatrix}</annotation></semantics></math>s=[.25.75​]

> **注意：** 每个元素的位置与表格相同，即使我们没有明确标记行和列。

找到下一个状态矩阵就像将当前位置向量 <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">\vec{s}</annotation></semantics></math>s 乘以 <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>T</mi></mrow><annotation encoding="application/x-tex">T</annotation></semantics></math>T 一样简单。要预测未来的更多小时，我们需要多次操作。例如，要预估三个小时后的情况：<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>T</mi><mi>T</mi><mi>T</mi><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">TTT\vec{s}</annotation></semantics></math>TTTs。我们可以使用指数简化这个过程：<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>T</mi><mn>3</mn></msup><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">T^3\vec{s}</annotation></semantics></math>T3s 或者推广到 <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>n</mi></mrow><annotation encoding="application/x-tex">n</annotation></semantics></math>n 小时：<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><msup><mi>T</mi><mi>n</mi></msup><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">T^n\vec{s}</annotation></semantics></math>Tns。

## 适用于文本完成

上述原则可以应用于各种概率情况。与此特定网页最相关的是文本完成。我们希望预估用户最可能的下一个词。给定最后一个词，最可能的下一个词是什么？首先，我们需要一个字典。

### 字典

从样本文本构建字典非常简单。为了解释的目的，我们将从任意的字典开始。

| 索引 | 单词 |
| --- | --- |
| 0 | 橙子 |
| 1 | 水果 |
| 2 | 激情 |
| 3 | 奶酪 |
| 4 | 不 |
| 5 | 是 |

### 构建转移矩阵

要构建我们的转换矩阵，我们需要计算字典中可能单词之间发生的所有转换。出于性能考虑，我的实现将字典转换为`HashMap<String, usize>`。接下来，我会遍历训练文本，并将每个单词与其在字典中的索引进行匹配， effectively transforming the `String` into a `Vec<usize>`。例如，短语“passion fruit is not orange, cheese is orange”，会变成`[ 2, 1, 5, 4, 0, 3, 5, 0 ]`。然后，实现会遍历这个向量中的每个元素，计算每个转换的次数。为了性能起见，这些计数会存储在另一个`HashMap`中，但最终会被转换成一个矩阵 <math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>C</mi></mrow><annotation encoding="application/x-tex">C</annotation></semantics></math>C。每一行都是输出单词的索引，而每一列是输入单词的索引。例如，转换“fruit”（索引1）-> “is”（索引5）恰好发生了一次，所以我们在列1，行5记录`1`。

\( C \)是一个矩阵，如下所示：

这个矩阵并不是很有趣，对吧？

每个元素都需要转换为概率。计算每列的和：

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mo fence="true">[</mo><mtable rowspacing="0.16em" columnalign="center center center center center center" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>2</mn></mstyle></mtd></mtr></mtable><mo fence="true">]</mo></mrow><annotation encoding="application/x-tex">\begin{bmatrix} 1 & 1 & 1 & 1 & 1 & 2 \end{bmatrix}</annotation></semantics></math>[1​1​1​1​1​2​]

创建一个由<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>D</mi></mrow><annotation encoding="application/x-tex">D</annotation></semantics></math>D组成的对角矩阵，其由<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mfrac><mn>1</mn><mtext>列求和</mtext></mfrac></mrow><annotation encoding="application/x-tex">\frac{1}{\text{column sum}}</annotation></semantics></math>列求和组成。

矩阵 **C** 是一个`6x6`的矩阵，表示如下：

要完成我们的马尔可夫（又名过渡）矩阵<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>M</mi></mrow><annotation encoding="application/x-tex">M</annotation></semantics></math>M，我们只需执行：

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>M</mi><mo>=</mo><mi>D</mi><mi>C</mi></mrow><annotation encoding="application/x-tex">M = DC</annotation></semantics></math>M=DC

### 使用过渡矩阵

有两种可能的情况：用户正在输入过程中，或者他们已经完成了最后一个单词。后者是最容易实现的。扫描用户的文本，并且分离出最后一个单词。对单词列表执行查找以确定其索引。创建一个新向量，其中包含`0`，除了该索引处应包含`1`。例如，如果最后一个单词是'is'，

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mover accent="true"><mi>s</mi><mo>⃗</mo></mover><mo>=</mo><mrow><mo fence="true">[</mo><mtable rowspacing="0.16em" columnalign="center center center center center center" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>1</mn></mstyle></mtd></mtr></mtable><mo fence="true">]</mo></mrow></mrow><annotation encoding="application/x-tex">\vec{s} = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 1 \end{bmatrix}</annotation></semantics></math>s=[0​0​0​0​0​1​]

将其通过我们的过渡矩阵运行：

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi>M</mi><mover accent="true"><mi>s</mi><mo>⃗</mo></mover><mo>=</mo><mrow><mo fence="true">[</mo><mtable rowspacing="0.16em" columnalign="center center center center center center" columnspacing="1em"><mtr><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.5</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0.5</mn></mstyle></mtd><mtd><mstyle scriptlevel="0" displaystyle="false"><mn>0</mn></mstyle></mtd></mtr></mtable><mo fence="true">]</mo></mrow></mrow><annotation encoding="application/x-tex">M\vec{s} = \begin{bmatrix} 0.5 & 0 & 0 & 0 & 0.5 & 0 \end{bmatrix}</annotation></semantics></math>Ms=[0.5​0​0​0​0.5​0​]

这意味着最有可能的下一个选择是在索引`0`和`4`，分别对应于"orange"和"not"。这对自动补全非常有用。我们可以简单地向用户列出最可能的选项。

### 文本生成和稳态

用这种方法自动产生文本会很棒，对吧？

#### 天真的解决方案

每次迭代，从集合中选择最可能的单词。也许可以稍微随机一些：从前五个选项中随机选择一个单词。不幸的是，这里有一个问题。所有马尔可夫链在足够的迭代次数后都保证会收敛到一个特定的概率状态。为了使文本生成工作变得不可预测且不收敛，我们需要一些更复杂的东西。

#### 我的解决方案

创建一个边长等于<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>s</mi></mrow><annotation encoding="application/x-tex">\vec{s}</annotation></semantics></math>s长度的方形对角矩阵<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>R</mi></mrow><annotation encoding="application/x-tex">R</annotation></semantics></math>R。用介于<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>0</mn></mrow><annotation encoding="application/x-tex">0</annotation></semantics></math>0和<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mn>1</mn></mrow><annotation encoding="application/x-tex">1</annotation></semantics></math>1之间的随机数填充对角元素。然后选择索引与<math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi>R</mi><mover accent="true"><mi>s</mi><mo>⃗</mo></mover></mrow><annotation encoding="application/x-tex">R\vec{s}</annotation></semantics></math>Rs的最高值相对应的单词。
