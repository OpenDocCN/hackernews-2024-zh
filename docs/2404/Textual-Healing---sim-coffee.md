<!--yml

分类：未分类

日期：2024-05-27 13:06:54

-->

# 文本疗法 - sim.coffee

> 来源：[https://sim.coffee/textual-healing/](https://sim.coffee/textual-healing/)

[这是一个令牌问题](#tokenizer)

[结果](#results)

[惊喜](#surprise)

最近我们的 [Discord](https://discord.gg/7fpmbqfXcJ) 上活跃度很高。人们正在使用 [Codea](http://itunes.apple.com/app/id439571171?mt=8)，当它不能正常工作时，他们会抱怨！这*太棒了*。

在制作应用程序的14年之后，我最感激的是当有人花时间和注意力关注您所创造的内容。尤其是当他们因为它不如他们期望的那样工作而感到困扰并告诉您时。您应该感激每一个这样的人，即使是那些因为耐心不足而感到沮丧的人。

有人发布说，双击以下代码中的 `_bar` 一词并没有选择`_bar`这段文本，而是选择了“bar”（不包括下划线）

```
function  setup()
 foo._bar  =  5
end
```

我立即知道为什么会发生这种情况。我们的 `UITextInputTokenizer` 就是它发生的原因。我检查了我自定义的 `UITextInputStringTokenizer` 的子类^([1](#165e6f52-80ea-42ba-9a36-d82931aa4386))

```
//
//  JAMCodeTokenizer.m
//  Jam
//
//  Created by Simeon on 11/11/2013.
//  Copyright (c) 2013 Two Lives Left. All rights reserved.
//
```

*2013*。我十一年前开始这个文件。

## 这是一个令牌问题

`UITextInputTokenizer` 是 iOS 文本系统用来查询您的自定义文本输入系统中不同粒度的文本单元的工具。它有一个包含以下四个方法的复杂且神秘的 API：

```
func  isPosition(UITextPosition, 
 atBoundary: UITextGranularity, 
 inDirection: UITextDirection) ->  Bool
 func  isPosition(UITextPosition, 
 withinTextUnit: UITextGranularity, 
 inDirection: UITextDirection) ->  Bool
 func  position(from: UITextPosition, 
 toBoundary: UITextGranularity, 
 inDirection: UITextDirection) -> UITextPosition?
 func  rangeEnclosingPosition(UITextPosition, 
 with: UITextGranularity, 
 inDirection: UITextDirection) -> UITextRange?
```

`UITextPosition` 和 `UITextRange` 对象是不透明类型，您可以使用具有您的文本系统中有意义的自定义类型来实现它们。换句话说，Apple 的文本系统不关心它们是什么，只要它能把它们交给您并获取到有意义的结果即可。例如，Codea 使用 `AVAudioPlayer` 作为 `UITextPosition`，使用 `CLLocation2D` 作为 `UITextRange`^([2](#9df35c91-6dc2-4036-9ce0-a9f104e5dc70)))

这些方法基本上是查询您的文本系统，以了解在不同方向上的“边界”与“粒度”。边界是文本在指定粒度（如单词、句子、段落、文档等）中的“边缘”。例如，“如果我向前移动，这个位置是不是单词的边缘？”你可能回答：“是的，它是。”或者“不是。”

这对键盘导航和选择非常重要。当您按住 Option 键并按箭头键时，您会跳到下一个单词。这就是文本系统理解“单词”及其位置的方式。双击单词选择，三次点击选择整行。

Apple 提供了 `UITextInputStringTokenizer`（注意其中的 `String`），它是 `UITextInputTokenizer` 协议的一个具体子类，因此您不必自己编写。我们曾经基于此作为我们代码标记器的基础，因为我们懒得写很久了。

当时，我会在 Xcode 的代码编辑器中找到我喜欢的功能（下面我将记录一些），然后在标记器的上下文中找出如何实现它们，当我不想特别处理时，则返回到基本的字符串标记器

双击一个单词是我们回退到字符串标记器的其中一种情况。问题是，字符串标记器设计用于自然语言而不是代码。在英语中，单词通常不包括下划线，因此它们不会被选中，因为它们在单词粒度级别上形成边界，并且`rangeEnclosingPosition`方法不会将它们包括在单词中

另一个问题是，我把*Jam*写成一个通用的代码编辑器，而不是特定于 Lua 的，所以上述代码标记器不知道 Lua 标识符是如何形成的，或者符号边界应该在哪里。我把我们的代码标记器集中在如何导航空白和允许精确插入符位置^([3](#51675be5-f78f-475a-9933-fae14827f803))

## 结果

我决定编写一个新的、特定于 Lua 的标记器，现在所有的 Codea Lua 编辑器都在使用。但是，`UITextInputTokenizer` 的 API 设计要避免特殊情况成为一团糟非常困难！以下是我处理过的情况，其他情况都回退到原始代码标记器：

#### 命令后箭头用于缩进代码

转到 Xcode 缩进代码行的末尾

按下 ⌘←

插入符跳转到行的开始 — 但*不是空白的开始！*

按下 ⌘←

插入符跳转到行的开始，在空白处的开始

这是在 Codea 中的样子^([4](#acc4a566-1df1-40d3-ae0a-1fe2517dd8c7))

#### 精确的插入符位置

同样是之前的一个特性，在 iOS 模拟器中记录下来（以展示点按发生的位置）

#### 尊重符号边界

下面是使用选项键跳转到“单词”（符号）的导航。丑陋的 `_main_Test` 成员与原始的错误报告相关联。您可以看到我们现在将符号作为一个单独的实体遍历

#### 为符号提供范围作为“单词”

演示选择“_”作为标识符一部分时的固定行为

#### 回退到自然语言的标记化

当然，因为引号字符串在代码编辑器中被视为符号，双击其中一个在这个系统中通常会选择整个字符串。在这种情况下，我们排除这些符号，并推迟到`UITextInputStringTokenizer`以获取正常的文本编辑体验，当在字符串、注释等内部时

## 附录

当我坐下来写这篇文章时，我昨晚重新编写了 Codea 的标记器，2024 年 4 月 7 日星期日发布了一个新的测试版本，然后迅速睡着了

而我开始写这篇文章时，我谈到了用户的*投诉*。但就在今晚我写这篇文章的一半时，我收到了一封非常可爱和亲切的电子邮件。它如下所示

> *我怀疑您没有足够多这类的测试反馈，所以..：*
> 
> *除非我大错特错，你在某个时候改进了编辑器中的光标定位和行选择，比过去要好得多，至少对我来说是这样。*
> 
> *我以前经常遇到麻烦，当使用行号侧边栏选择行时，或者试图将光标置于行首时（换句话说，这两个操作会混在一起），但现在感觉要容易得多 / 需要的精度或者之前导致我失败的任何因素都没有了。*
> 
> *往往是这些微小的生活质量改善，起到了决定性作用，特别是当你经常使用它们时（即大多数代码编辑器中的功能，如果放在整个用户群体中来看）。*
> 
> *我似乎记得至少跟你抱怨过几次（我的墙壁肯定听过很多），所以我理所当然地也花时间从心底挖掘以下内容（或者说从我的心底残存的一块黑炭中）：*
> 
> *非常感谢你！*

有人注意到了！
