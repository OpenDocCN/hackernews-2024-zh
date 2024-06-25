<!--yml

分类：未分类

日期：2024-05-27 14:25:18

-->

# 使用20行Python生成可读的胡言乱语

> 来源：[https://benhoyt.com/writings/markov-chain/](https://benhoyt.com/writings/markov-chain/)

# 使用20行Python生成可读的胡言乱语

2023年11月

> **转到：**[算法](#the-algorithm) | [示例](#some-better-examples) | [Python实现](#python-implementation) | [结论](#conclusion)

我最近学会了如何使用简单的[马尔科夫链](https://zh.wikipedia.org/wiki/%E9%A9%AC%E5%B0%94%E7%A7%91%E5%A4%AB%E9%93%BE)生成文本。生成的文本可读，但也完全是胡言乱语；作为散文，它没有多大价值，但对于预测下一个词，就像你手机键盘的建议一样，它非常有用。

我是在Kernighan和Pike的书[*编程实践*](https://www.cs.princeton.edu/~bwk/tpop.webpage/)第3章中学到这个算法的，他们在各种编程语言中实现了这样一个生成器，来讨论程序设计和数据结构。

注意，这个算法只是马尔科夫链的一个小应用，而马尔科夫链是一个更一般的统计概念。

## 这个算法

这个算法很容易解释：从一个输入文本开始，其中的短语将被用于生成输出。对于输入中的每一对词，记录可能出现在这对词后面的单词列表。

一旦你建立了那个数据结构，你就可以生成任意多或任意少的输出。从输入中任选一对词开始，然后随机选择可能的第三个词。接着继续，第二个词是刚刚生成的词，再随机选择另一个词，依此类推。

就这样！

这个算法使用的是单词对，也称为[双字组](https://zh.wikipedia.org/wiki/%E5%8F%8C%E5%AD%97%E7%BB%84)。你可以将算法推广为使用单个单词或三字组而不是对子。然而，正如《编程实践》指出的那样：

> 使前缀更短会产生不太连贯的散文；使其更长会基本复制输入文本。对于英文文本，使用两个词来选择第三个词是一个不错的折中；它似乎能够再现输入的风格，同时又添加了自己的古怪触摸。

让我们通过一个例子来演示一下。对于小输入，你希望有一些重复才能使其工作，所以我们将使用《圣经》（《国王詹姆士圣经》）中最后五条十诫作为输入：

> 不可杀人。
> 
> 不可奸淫。
> 
> 不可偷窃。
> 
> 不可作假见证。
> 
> 不可垂涎邻居的房屋，也不可垂涎邻居的妻子，他的男仆、女仆，他的牛、他的驴，以及他的一切所有。

下面是“可能的第三个词”数据结构的样子，显示了左边的词对和右边以`|`分隔的可能性的前几个条目：

```
shalt not         kill. | commit | steal. | bear | covet | covet
Thou shalt        not | not | not | not | not
nor his           manservant, | maidservant, | ox, | ass,
not covet         thy | thy
covet thy         neighbours | neighbours
thy neighbours    house, | wife,
not kill.         Thou
kill. Thou        shalt
not commit        adultery.
... 
```

所以如果你以“shall not”开头，你随机选择“kill.”、“commit”、“steal.”、“bear”或“covet”作为下一个词。因为“covet”在可能性列表中出现了两次，所以它被选择为下一个词的可能性是其他词的两倍。

你可以看到我们在单词中包含了大写字母和标点符号。这使得分词更简单，但这也意味着生成的输出将自动生成在输入中实际出现的具有大写和标点符号的“句子”。这是一个相当巧妙的技巧！

生成的输出是什么样的？让我们试试使用上面的十诫输入：

> 你不可作假见证，害你的邻居。你不可通奸。你不可偷盗。你不可贪恋你邻居的房屋，你不可通奸。你不可贪恋你邻居的房屋，你不可偷盗。你不可通奸。你不可偷盗。你不可贪恋你邻居的……

正如你所见，使用如此小的输入文本的问题在于输出通常是输入文本的逐字复制，或者是随机接近它的东西。

如果你使用一个更大的输入文本，你会得到更有趣的输出。

## 一些更好的例子

下面是使用更大的输入文本生成的一些更有趣的输出。我们将从每个输入中生成100个词。这些书籍来自[古腾堡计划](https://www.gutenberg.org/)。

### 使用我的文章

使用[我写的文章](/writings/)作为输入：

> 特别是如果你需要返回一个值，但现在非常平坦——几乎快了5倍。不过，如果你需要自定义异常；如果合适的话，使用内置异常。只有当你在Linux上时，这使用x86的SSE/SIMD指令来使用AWK处理CSV，比如`NF`在AWK中是只读的。什么是“Pythonic”？那太有趣了。我的建议：自己编写类型安全的`OrderedMap<int>`。但因为Lox字符串不支持转义序列，所以没有`null`或`undefined`，而你却有它！一个极简的Make在其收入的一部分来自于……

我喜欢它如何捕捉到我大部分的技术兴趣：性能，成语代码，向后兼容性，AWK，CSV，小语言等等。它不是ChatGPT，但如果你眯起眼睛看，文本看起来相当合理！

注意输出甚至包括了输入中的有效Markdown（尽管我删除了一些链接）。

### 使用*爱丽丝梦游仙境*

这里使用刘易斯·卡罗的*爱丽丝梦游仙境*作为输入。这本书本来就是胡说八道的，所以这几乎能行：

> 大门口附近有一棵大玫瑰树，她高兴地发现她的火烈鸟匆匆离去；“他们的名字叫艾尔希、莱茜和蒂莉；他们不能证明我做了：否认是毫无意义的。我想迪娜接下来会给我送去消息！” 她打开了门，突然打了一个喷嚏。睡鼠又闭上了眼睛，看着池塘里的大扇子，“她坐在手边那么好地发出咕噜声，就匆匆离开了，不等待被有限的替代权…

### 使用旧约圣经（来自于《英王詹姆士圣经》）

注意输出包含了（无意义的）诗句编号，这些是输入的一部分。

寻找巴比伦王的烤肉！

> 本雅明、贝拉基雅的儿子雅西勒、摩西的儿子约雅敬，至于圣神是在你的意志里。22:30 在海门的南边，在主的门廊上，对着那出自主的山，他的浪潮袭击着它。107:26 他们堆积起来埋葬他们，他们，他们的孩子，向祭司们展示；13:17 巴比伦王在尘土的名字下被烤了。18:1 其中一个回答说，约书亚拿了，把他的手放在我父亲扫罗的仆人身上，我生命不会从…

### 使用新约圣经（来自于《英王詹姆士圣经》）

有趣的是，新约的输出以 “摩西” 开头，他是旧约中的一个关键人物（但他的名字在新约中也出现了 80 次）。

我的电脑看起来也有些异端邪说：“我们已经传播了人的意志的话”。

是的，[“thoroughly”](https://en.wiktionary.org/wiki/throughly) 是一个（古语）词。

> 摩西，你们要住在那里。2:11 因此你们要记念，要凭我的声音起来反对他们。20:20 他们说，这不是他使人谦卑的吗？他会彻底地清除他的地，把小麦收集到海中，得到粮食：因为我们已经传播了人的意志的话，它将是如此的：但是我们有没有派人来找我？10:30 耶稣进入了神的国度。5:6 不要让人说，你这个傻瓜，他离约帕不远，另一边。10:33 但是无论是谁…

### 使用《世界大战》

再加一个，以确保准确。这是使用 H. G. 威尔斯的《世界大战》。

> 在 Halliford，我看起来像是德比赛马日上的黑暗一样。我兄弟朝向海德公园的铁门走去。我曾见过两具人类骷髅 —— 不是尸体，而是被清理干净的骨架 —— 在深坑里 —— 男人开车经过并停在逃亡者面前，却没有提供帮助。客栈关门了，就像是有人骑自行车经过一样，孩子们去寻找食物，并告诉他那对他来说将是一份铅的抵押。事实上，那是面向河边通往谢珀顿的房子和其他房子的黎明。一种疯狂的决心占据了…

怀着疯狂的决心，现在让我们看一下我用来实现这一切的 Python 代码。

## Python 实现

首先，这是完整的程序 - 有24行空格和注释，没有空格和注释的话只有16行：

```
import collections, random, sys, textwrap

# Build possibles table indexed by pair of prefix words (w1, w2) w1 = w2 = ''
possibles = collections.defaultdict(list)
for line in sys.stdin:
    for word in line.split():
        possibles[w1, w2].append(word)
        w1, w2 = w2, word

# Avoid empty possibles lists at end of input possibles[w1, w2].append('')
possibles[w2, ''].append('')

# Generate randomized output (start with a random capitalized prefix) w1, w2 = random.choice([k for k in possibles if k[0][:1].isupper()])
output = [w1, w2]
for _ in range(int(sys.argv[1])):
    word = random.choice(possibles[w1, w2])
    output.append(word)
    w1, w2 = w2, word

# Print output wrapped to 70 columns print(textwrap.fill(' '.join(output))) 
```

你可以自行判断，我认为它相当可读。

该程序期望以命令行参数的形式输入单词的数量（`sys.argv[1]`），并从标准输入接收输入文本。要运行它，使用如下命令：

```
$ python3 markov.py 100 <AliceInWonderland.txt 
A large rose-tree stood near the entrance of the cakes, and was
... 
```

该代码也可作为GitHub的Gist：[markov.py](https://gist.github.com/benhoyt/619cf3113866450aa31d8a2c1f8e01dc)。这个程序并不是原创 - 我在这里允许你对它做任何你想做的事情。

## 结论

我发现简单的数据结构和随机数生成器能产生如此有趣的输出，真是令人着迷。

我认为在记录的“单词”中包含大写字母和标点符号的技巧非常聪明。当然，这并不是一个技巧，而是一种优雅的设计选择，使程序更简单 *且* 输出更好。我想知道还有多少类似的设计选择隐藏在现有的程序中，等待简化？

快来尝试用你自己的输入生成文本吧！

如果你愿意在GitHub上给我[赞助](https://github.com/sponsors/benhoyt/)，这将激励我继续开发我的开源项目并写更多优质内容。谢谢！