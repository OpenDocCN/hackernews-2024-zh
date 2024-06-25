<!--yml

类别：未分类

日期：2024-05-27 14:48:42

-->

# ChatGPT vs Advent of Code - The Motte

> 来源：[https://www.themotte.org/post/797/chatgpt-vs-advent-of-code](https://www.themotte.org/post/797/chatgpt-vs-advent-of-code)

# ChatGPT参加Advent of Code 2023

LLM风头正劲，人们担心它们很快就会取代程序员（或者说是每一个可能的办公室工作），所以我决定进行一个实验，看看ChatGPT-4在Advent of Code 2023中的表现如何。

## 什么是Advent of Code

[Advent of Code](https://twitter.com/ericwastl)（以下简称AoC）是由[Eric Wastl](https://twitter.com/ericwastl)举办的年度编程“事件”，在每年的12月的头25天举行。每天午夜，一个问题解锁，包括一个输入文件和对所需解决方案的描述（可以是一个数字或一个字母和数字的序列），需要通过处理输入文件来确定。要解决问题，您必须向网站提交正确的解决方案。一旦您完成了问题的第2部分，通常是问题第1部分的更难的版本就会解锁。理论上，您不必提交任何代码，因此您可以通过手工解决所有问题，然而，通常情况下这是不可行的，编写一个程序来为您完成工作是解决问题的唯一简单方法。

还有一个排行榜，参与者根据提交解决方案的速度得分。

问题从第1天开始非常简单（有时仅仅是要求编写一个程序来对输入中的所有数字求和），并逐渐进展到更困难的问题，但它们从来没有变得非常困难：一个计算机科学专业的毕业生应该能够在几个小时内解决所有问题，也许除了1或2个例外。

## 先前历史

这不是ChatGPT（或LLMs）第一次参加Advent of Code。事实上，去年（2022年），ChatGPT的用户能够在多个日子里达到全球排行榜的顶端，这成为了一个大新闻。这引起了一些担忧，以至于Eric明确禁止ChatGPT用户在全球排行榜填满之前提交解决方案（当然，他也没有任何办法来实际执行这个禁令）。[一些人](https://old.reddit.com/r/adventofcode/comments/18515qh/2023_the_year_of_gpt/)甚至预期GPT-4能完成整个一年的挑战。

去年GPT-3.5在AoC中的表现引起了很多关注，但实际结果却相当平平，LLM（大型语言模型）爱好者表现得非常不科学，夸耀成功的时候，却在它开始失败时变得非常安静。事实上，ChatGPT在第3天和第5天开始遇到困难，可能在第5天之后无法解决任何问题。

## 为什么要使用GPT来解决AoC问题？

我认为这是最接近完美基准的方法。问题大致按照难度递增的顺序排列，因此你可以看到它在哪里无法解决。因为几乎每年的问题都可以在几个小时内由计算机科学毕业生解决，所以这是 AGI 的一个很好的基准。由于所有问题都是新颖的，所以解决方案不能来自过拟合。

此外，在发布时，人们尝试了 GPT-4 在 AoC 2022 上，并 [发现它的性能更好](https://twitter.com/geoffreylitt/status/1635757456377917440)，因此很有趣看到改进有多少来自过拟合而不是实际改进

## 方法论

我没有购买 ChatGPT Plus，我只有一个付费的 API 密钥，所以我使用了一个命令行客户端 [chatgpt-cli](https://github.com/marcolardera/chatgpt-cli/) 并手动运行了输出的程序。我用于第一部分的提示是：

```
Write a python program to solve the following problem, the program should read its input from a file passed as argument on the command line: 
```

然后是问题的复制文本。我手动从提示中删除了所有 Eric 写的故事性内容，这对于 ChatGPT 来说是一些小帮助。如果输出有微不足道的语法错误，我会手动修复它们。

如果程序在 15 分钟内未终止，我就放弃了解决方案，并让 ChatGPT 失败 3 次后放弃。失败包括无效的程序或运行到完成但返回错误输出值的程序。

如果程序运行到完成但答案错误，我使用以下提示：

```
There seems to be a bug can you add some debug output to the program so we can find what the bug is? 
```

如果程序遇到错误，我会指出并复制错误消息。

如果第一部分正确解决，则第二部分的提示将是：

```
Very good, now I want you to write another python program, that still reads input from a command line argument, same input as before, and solves this additional problem: 
```

我决定在 ChatGPT 连续 4 天无法解决第一部分后停止实验。

## ChatGPT Plus

因为我意识到 ChatGPT Plus 可能会更好，所以我用另外两个来源补充了我的实验。第一个是 [Martin Zikmund 的 Youtube 频道](https://youtube.com/@mzikmund)（以下简称 "youtuber"），他在视频中讲解了如何用 C# 解决问题，以及如何使用 ChatGPT（使用 Plus 帐户）尝试解决问题。

第二个是 ChatGPT 爱好者的博客 ["Advent of AI"](https://medium.com/@dretyr)（以下简称爱好者），他尝试使用 ChatGPT Plus 解决问题，然后还用 ChatGPT Plus 写了一篇关于此的博客。由于博客是由 ChatGPT 生成的，所以它的质量很差，可能含有幻觉，然而 [带有转录的 github 仓库](https://github.com/dretyr/adventofai) 是有价值的。

结果表明爱好者完全无用：它经常通过逐步引导 ChatGPT 到结果来解决问题，而且他在第 6 天就放弃了。

youtuber 提供了更多的信息，大部分时间他坚持让 ChatGPT 自己解决问题。然而，他有时会给出一些重要提示，要么通过调试 ChatGPT 的解决方案，要么解释给它如何解决问题。我已在结果中备注了这一点。

## 结果

|  | 第一部分 | 第二部分 | 注释 |
| --- | --- | --- | --- |
| 第一天 | 通过 | 失败 |  |
| 第2天 | OK | OK |  |
| 第3天 | FAIL | N/A |  |
| 第4天 | OK | OK | 用蛮力方法解决了第2部分 |
| 第5天 | OK | FAIL |  |
| 第6天 | FAIL | N/A | ChatGPT Plus 解决了两部分 |
| 第7天 | FAIL | N/A |  |
| 第8天 | OK | FAIL | ChatGPT Plus如果你告诉它解决方案，它就解决了第2部分 |
| 第9天 | FAIL | N/A | ChatGPT Plus 解决了两部分 |
| 第10天 | FAIL | N/A |  |
| 第11天 | FAIL | N/A | ChatGPT Plus 能够用一个重要的提示解决第1部分 |
| 第12天 | FAIL | N/A |  |

今年 GPT-4 的性能比去年的 GPT-3.5 差了*一点*。去年 GPT-3.5 能够自己解决 3 天（1、2 和 4），而今年 GPT-4 只能解决 2 天（2 和 4）。

但是 ChatGPT Plus 做得*稍微好一些*，自己解决了4天（2、4、6 和 9）。这可能是因为它能够看到问题输入（作为附件），而不仅仅是问题提示和示例输入，以更好地系统提示，并能够通过代码解释器进行更多次的往返（我在3~4个错误/错误输出后放弃了）。

一个人不应该过分看重它解决第9天问题的能力，问题难度并不是单调递增的，第9天恰巧非常容易。

## 结论

总的来说，我的主观印象是并没有太多改变，它解决不了任何需要比只是遵循指示更复杂的东西，而且它很擅长遵循简单的指示。

可能是语言模型已经达到了它们的顶峰。或者也许明年 Q* 或 Bard Ultra 或 Grok Extra 会像今年应该由 GPT-4 做的那样。很难不对炒作周期感到厌倦。

我对 ChatGPT 在 AoC 上的表现有一些观察，我会无序地在这里报告。

### 调试/世界模型

大多数人在第一次尝试解决 AoC 问题时都无法做到没有错误，所以我也不指望人类水平的 AI 能够做到（如果它能够做到，那就是超人类定义的）。

我的一些提示策略是朝着让 ChatGPT 调试其有缺陷的解决方案的方向发展的。我要求它添加调试打印以找出解决方案的逻辑出了什么问题。

ChatGPT *从未*做到过这一点：它的调试技能完全不存在。如果它遇到错误，它会简单地重写整个函数，或者更常见的是整个程序，从头开始。

这与程序员的观点截然不同。

这很有趣，因为调试技术实际上并没有被教授。总的来说，编程教科书教你如何编程，而不是如何修复你写的错误。然而，人们确实会隐含地掌握调试技巧。

ChatGPT 具有与人类相同的编程教科书访问权限，但它却不学习调试。我认为这表明了 ChatGPT 实际上还没有学会编程，它没有“世界模型”，也就是在编程时它并不理解自己在做什么。

让 ChatGPT 学习调试的蛮力方式我认为是从 twitch 上抓取数百小时的编程直播，并在视频上进行 OCR 和音频上进行语音转文字，然后将其馈送给训练程序。这是我能想到的唯一大量调试示例的来源。

### 难度

这一届 AoC 是不是比去年的难度更大，这就是为什么 GPT-4 表现不佳的原因？也许吧。

难度很难客观评估。排行榜填充时间有 [散点图](https://aoc-stats.fastbee.box.ca/)，但是完成时间不一定等同于难度，而且今年和去年的差异也不大（注意：散点图不是按比例绘制的，很遗憾）。

我个人的主观印象也是，今年（迄今为止）并没有更困难。

增加难度的最佳证据是第 1 天的第 2 部分，其中包含一个小陷阱，人类参与者和 ChatGPT 都掉进了。

我认为这指向了一个问题，那就是用大量训练数据训练的 AI 存在问题：你无法真正判断它们到底有多好。理想情况下，你会在 AoC 2022 上测试 GPT-4，但 GPT-4 的训练集 *包含* 了大量 AoC 2022 的解决方案的副本，所以它实际上已经不再是一个好的基准了。

通常你会从训练集中取出一部分作为测试集，但是对于大规模的训练集来说，这是不可能的，因为没有人知道其中包含什么，也没有人知道每个单独的训练示例在其中被复制了多少次。

我想知道 OpenAI 是否有一个秘密的测试数据集，他们不会在任何地方放在互联网上以避免训练集污染。

[有人](https://news.ycombinator.com/item?id=38483271#38488265) 甚至推测今年的问题是故意设计的，以阻止 ChatGPT，但 Eric 实际上 [否认了这一点](https://old.reddit.com/r/adventofcode/comments/18bp8id/why_does_aoc_care_about_llms/kc646i4/)。

### 过拟合

GPT 4 比 GPT 3.5 大 10 倍，它在许多标准测试中表现更好，例如 [司法考试](https://law.stanford.edu/2023/04/19/gpt-4-passes-the-bar-exam-what-that-means-for-artificial-intelligence-tools-in-the-legal-industry/)。

为什么在 AoC 上表现并不好？如果不是因为难度，那可能是过拟合。它简单地记住了一堆标准化测试的答案。

是这样吗？我的 AoC 第 7 天的经历支持这一点。该问题要求编写一个自定义字符串排序函数，问题中的字符串代表卡牌的手（A25JQ 是 A、2、5、J 和 Q），它所要求的顺序与扑克牌得分 *类似*。但这 *不是* 扑克牌。

这是一个非常简单的一天，我以为 ChatGPT 能够毫无问题地解决它，因为你只需按照说明操作。但是它却不能，它不可避免地被拉向了为扑克而写解决方案，而不是为这个问题而写。

我猜测这是过拟合的一个例子。它在训练集中看到了太多扑克的例子，以至于无法解决这个类似扑克的事物。
