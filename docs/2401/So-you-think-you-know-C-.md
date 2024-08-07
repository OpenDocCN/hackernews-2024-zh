<!--yml

分类：未分类

日期：2024-05-27 14:56:47

-->

# 那么你认为你懂 C 语言吗？

> 来源：[`wordsandbuttons.online/so_you_think_you_know_c.html`](https://wordsandbuttons.online/so_you_think_you_know_c.html)

很多程序员声称他们懂 C 语言。好吧，它有着最著名的语法，已经存在了 44 年，而且没有被混乱的特性所淹没。很容易！

如果你觉得是的话——做这个测试。它只有 5 个问题。每个问题基本上都一样：返回值会是什么？每个问题都有四个选项，其中只有一个是正确的。

因此，正确答案是：

你得了 0 分，抱歉。你不知道你不知道的。你得了 1 分，这还不错。你知道有些东西你不知道。你得了 2 分，这还不错。你知道有些你不知道的事情。你得了 3 分，相当不错。你知道你不知道。你得了 4 分，非常好。你知道你不知道什么。你得了 5 分，恭喜！你准确地知道你不知道什么。

是的，每个问题的正确答案都是“I don’t know”。

现在我们一一来解决它们。

**第一个**问题实际上是关于结构填充的。C 编译器知道在 RAM 中存储不对齐的数据可能是昂贵的，所以它会为你填充数据。如果你的结构中有 5 个字节的数据，它可能会将其变为 8、16、6 或任何它想要的。还有一些像 GCC 属性 aligned 和 packed 这样的扩展，它们让你在这个过程中获得一些控制，但它们不是标准的。C 本身并不定义填充属性，所以正确答案是：“I don’t know”。

**第二个**问题是关于整数提升。一个短整型和一个具有最大整数值的表达式的类型是相同的，这是合理的。但是对于 C 语言来说，合理并不意味着正确。有一个规则是，每个整数表达式都会提升为 int 类型。实际上，情况比这复杂得多。去查看标准，你会喜欢的。

但是即使这样，我们不是比较类型，我们比较大小。标准只保证了 short int 的范围不应该超过 int。它们的范围可能相等，而且由于填充位，short int 的大小甚至可能大于 int 的大小。因此正确答案是：“I don’t know”。

**第三个**问题是关于黑暗角落的。从没有一个整数溢出，也没有一个字符类型的符号由标准定义的地方开始。第一个是未定义行为，第二个是实现特定的。但更重要的是，*char*类型本身的大小也没有指定位数。在历史上，有些平台上它是 6 位（还记得三字符序列吗？），今天有些平台上所有五种整数类型都是 32 位。甚至空格字符的值也是由实现定义的。如果没有这些细节被指定，那么对结果的所有推测都是无效的，所以答案是：“I don’t know”。

第四个看起来很棘手，但回想起来并不那么难，因为你已经知道 int 大小在标准中没有直接指定。它很容易是 16 位，然后第一个操作就会导致过度移位，这就是明显的未定义行为。这不是 C 的错，在某些平台上，甚至在汇编中也是未定义的，所以编译器简单地无法在不消耗大量性能的情况下给你有效的保证。

所以再次，“我不知道”是正确的答案。

最后一个是经典之作。结果取决于未定义行为：[后置增量是否被延迟以及首先获取哪个参数](https://gynvael.coldwind.pl/?id=372)。在一个平台上可能会像你期望的那样工作，在另一个平台上可能很容易失败。通常，它只是评估为*2*，所以你习惯了，直到有一天它不再是这样。这就是未定义事物的问题。当你遇到一个时，正确的答案总是：“我不知道”。

## 附言。

到这一步，我只需要道歉。这个测试显然是挑衅性的，甚至可能有点冒犯。如果它引起了任何不快，我很抱歉。

问题是，我大约在 1998 年左右学会了 C，然后整整 15 年以来都以为自己很擅长它。这是我在大学期间选择的语言，并且我在第一份全职工作中用 C 完成了一些成功的项目，甚至在我主要使用 C++ 的时候，我仍然认为它是臃肿的 C。

2013 年是一个关键时刻，当我开始参与一些安全关键的 PLC 编程时。这是一个核电厂自动化研究项目，绝对不能容忍任何不明确的地方。我必须学会，虽然我确实了解很多关于 C 编程的知识，但我所知道的绝大部分都是错误的。而且我必须以艰难的方式学习。

最终，我不得不学会依靠标准而不是民间传说；相信测量而不是假设；对“事情看起来正常”的态度持怀疑态度，——我必须学会一种工程态度。这才是最重要的，而不是某些特定的 WAT 轶事。

我只希望这个小测试能帮助像我过去那样的人在大约 15 分钟而不是 15 年的时间里学会这种态度。
