<!--yml

类别：未分类

日期：2024-05-27 13:21:32

-->

# Bash中的自然语言处理

> 来源：[https://massimo-nazaria.github.io/nlp.html](https://massimo-nazaria.github.io/nlp.html)

让我们实现一个Bash工具链，通过使用*n*-gram语言模型生成类似文本语料库的随机散文！

基本上，[NLP](https://en.wikipedia.org/wiki/Natural_language_processing)旨在通过使用不同的技术，教会计算机理解和处理人类语言。

接下来，我们将通过使用小说[*白鲸*](https://gutenberg.org/ebooks/15)训练我们的模型来一窥工具链。

首先，让我们从仓库中获取`bash-textgen/`文件夹并进入它：

```
$ git clone https://github.com/massimo-nazaria/bash-textgen.git
$ cd bash-textgen/ 
```

### 预处理

在典型的NLP过程中，文本语料库（即输入数据）在训练数学模型之前被预处理成结构化数据。

这包括诸如分词（即将文本分解为单词）或数据清洗（即去除任何非相关文本）的任务。

例如，让我们从`moby-dick.txt`中提取单词，通过删除不必要的字符并将结果放入`words.txt`：

```
$ cat moby-dick.txt | ./words.sh > words.txt 
```

初始提取的前10个单词：

```
$ cat words.txt | head -n 10
moby
dick
by
herman
melville
chapter
loomings
call
me
ishmael 
```

正如您所见，初始词语包括：标题，作者全名，第1章标题，以及开头的*叫我以实玛利*。

具体来说，`words.sh`执行以下文本转换：

1.  将所有输入文本转换为小写；

1.  移除除句号`.`外的非字母字符；

1.  移除多余的空格和句点字符；

1.  每行输出一个单词（或句号）。

### 训练

在训练过程中，NLP模型从预处理的训练数据中学习语言中的模式和关系。

这样，生成的训练模型可以提供各种服务，如情感分析和文本生成。

#### *N*-Gram语言模型

*N*-gram语言模型易于理解和实现。

虽然在NLP早期被认为是*最先进*技术，但现在更先进的NLP技术比*n*-gram在文本生成方面表现更好。

尽管如此，它们在文本编辑器中的下一个词建议或自动完成方面仍然非常有效。

训练数据准备工作包括将文本语料库中预处理的单词组织成连续的*n*-元组，即*n*-grams。

让我们通过计算[bigrams](https://en.wikipedia.org/wiki/Word_n-gram_language_model#Bigram_model)（即2-gram）来做到这一点：

```
$ cat words.txt | ./ngrams.sh 2 > bigrams.txt 
```

计算的前10个bigrams：

```
cat bigrams.txt | head -n 10
moby dick
dick by
by herman
herman melville
melville chapter
chapter loomings
loomings call
call me
me ishmael
ishmael . 
```

在训练过程中，*n*-gram语言模型学会根据前*n*-1个词预测句子中的下一个词。

因此，使用bigrams，它们学会根据句子中仅有的前一个词预测下一个词。

### 文本生成

让我们从初始单词“the”开始，基于计算的bigrams生成文本。

```
$ ./textgen.sh bigrams.txt "the" 
```

*输出：*

> 对于他退休的捕鲸人来说，备用桅杆和更确定的野生动物，他从未告诉过码头头领，对此倾斜是为了强加的，我将它们作为无尽的雕塑。

生成的散文实际上非常不错！它模仿了我们最初用作文本语料库的赫尔曼·梅尔维尔的风格。

让我们再次尝试初始词“man”：

```
$ ./textgen.sh bigrams.txt "man" 
```

*输出：*

> 从回忆中的男人，即使对于泰晤士隧道，然后拖线不必去鲸鱼，选择我们的半球。

看到我们的工具如何在几行Bash代码中模拟如此文学巨匠的诗意，这有点令人惊讶（而且有趣！）。

特别是，在上面的例子中，`textgen.sh`从给定的初始词开始生成句子如下：

让“man”成为初始给定的单词。

#### *步骤 1：获取所有以“man”开头的二元组*

```
cat bigrams.txt | grep -e "^man " 
```

上述命令的结果是一个二元组列表，例如：

```
man on
man receives
man enter
man distracted
man travelled
... 
```

请注意，二元组列表通常包含大量重复项。其中，一些二元组将比其他二元组频繁出现得多。

并且最常见的二元组将是在接下来的步骤中最有可能被选中的，以提取我们的下一个词。

#### *步骤 2：随机洗牌所有这样的二元组*

```
cat bigrams.txt | grep -e "^man " | shuf 
```

我们随机重新排列大量二元组列表，以避免总是提取相同的下一个词。洗牌列表的示例：

```
man from
man who
man prefers
man interested
man slipped
... 
```

```
cat bigrams.txt | grep -e "^man " | shuf | head -n 1 
```

从上述洗牌列表中，我们选择第一个二元组：

在我们的例子中，“from”是生成句子中的下一个词！

请注意，我们可以从洗牌列表中选择任何二元组，不失一般性。

事实上，无论在洗牌列表中的位置如何，最常见的二元组都将始终最有可能被提取。

#### *最终步骤：从二元组中获取第二个词*

```
cat bigrams.txt | grep -e "^man " | shuf | head -n 1 | cut -d ' ' -f2` 
```

显然，管道中的最终命令 `cut -d ' ' -f2` 提取了二元组中的第二个词：

上述生成步骤迭代地逐字生成，直到下一个词是句点字符或没有下一个词为止。

让我们通过使用[三元组](https://en.wikipedia.org/wiki/Word_n-gram_language_model#Trigram_model)，即*n*-grams（*n*=3），再玩一会儿：

```
cat words.txt | ./ngrams.sh 3 > trigrams.txt 
```

一般来说，`ngrams.sh`接受作为其参数任何从2开始的自然整数。

让我们从初始的两个词“a man”开始生成一个随机句子。

```
$ ./textgen.sh trigrams.txt "a man" 
```

*输出：*

> 一个人从这个胖子里脱身，我们将从鲸鱼的心脏开始，总是在极大的和非凡的困难下，每个人都知道他们挂在他们仍然悬挂的小艇的精美康乃馨。

现在让我们尝试初始的两个词“by falling”：

```
$ ./textgen.sh trigrams.txt "by falling" 
```

*输出：*

> 通过跌落在南塔基特最好的捕鲸者中，我的两位同伴不是我所有的美丽对我来说都是痛苦的，我们要感谢这艘鲸鱼船的老手工艺，特别是像昨天我毁了你和错误不是船长阿哈布如果它可能落在所有这是一个新生的视野。

让我们试试“一刻钟”：

```
$ ./textgen.sh trigrams.txt "one moment" 
```

*输出：*

> 一瞬间观看同样的无数粗鲁划痕的原因完全是在面对死亡的一种巨大实用决心这位老鲸鱼猎人的字，为什么你们不都在深处摇摆？然而有没有理由拥有最古老的现存肖像无论如何都试图在不被所有航行中拿出来时进行测试现在还是从一个低垂的果园树枝上。  

多么有趣!

## 实施  

[***获取工具链使用和代码！***](https://github.com/massimo-nazaria/bash-textgen "GitHub Repository")  

请阅读也*[“Unix Philosophy with an Example”](/unix-philosophy.html)*来学习如何在本教程中看到的多个命令一起组成。  
