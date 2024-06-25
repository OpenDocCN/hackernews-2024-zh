<!--yml

分类：未分类

日期：2024-05-27 14:34:21

-->

# 苹果文档糟糕的一个具体例子 | 苹果文档糟糕的一个具体例子

> 来源：[https://www.amimetic.co.uk/blog/a-concrete-example-of-why-apples-docs-are-terrible/](https://www.amimetic.co.uk/blog/a-concrete-example-of-why-apples-docs-are-terrible/)

苹果的文档声名狼藉。特别是经常缺失或不完整。[没有概述可用](https://nooverviewavailable.com/) 给出了一些相关数据，但我认为那实际上低估了问题。即使有时可用，也经常很糟糕。这是我最近在做一些 MacOS 开发时遇到的一个小的具体例子（这是我不太熟悉的领域）。

我想提示用户打开一个目录。我该怎么做？显然我要谷歌一下（加入 `Swift`，避免古老的 `Obj-C` 答案）。

好像我想要的是叫做 `NSOpenPanel` 的东西。

我找到了[苹果的文档](https://developer.apple.com/documentation/appkit/nsopenpanel)，看起来相当清晰。我用一些简单的选项进行配置。

好的，我到底该如何使用它？

嗯。这远非清晰。页面上甚至没有解释最基本的用例。叹气。再次谷歌发现我想要调用 `begin`，而且这接受一个给出结果的回调。

这个示例已经过时了（Swift 的一个永恒问题），但最终在编辑器中解决了这个问题（现在我们有了一个带有结果的漂亮枚举）。

好吧，我该如何获取实际选择的目录？成功的值是`.OK`。有什么数据吗？没有。只是一个结果。

嗯……

有一个 [delegate](https://developer.apple.com/documentation/appkit/nsopensavepaneldelegate) 我可以设置。也许那就是得到结果的方法。

浪费了几分钟，基本上看起来是用于配置。

哦，它继承自 `NSSavePanel`！这一点完全不明显（我很抱歉，面向对象的爱好者，原则上这根本就没有任何意义）。哦，看，有大量新的函数和属性。啊，所以我应该查询 NSOpenPanel 对象以获取目录。

不太清楚怎么做。Xcode 提供了 `url`、`urls`、`directoryUrl`、`representedUrl` 等等（我没有编造这些，大部分在文档中都找不到）。让我们乐观点，选择 `url`。成功了！看起来能用了。

哦，还缺失一个（在我继续之前）：在最近的 MacOS 版本中，需要添加一个能够读取（和写入）用户选择目录的功能。文档中完全没有提及，这对于对该生态系统不太熟悉的人来说又是一个挫折。

这可以做得更好。苹果的开发者嘲笑 Electron，但看看 [Electron 的打开对话框文档多么清晰](https://electronjs.org/docs/api/dialog#dialogshowopendialogbrowserwindow-options)。

## 苹果的文档有什么问题？

1.  缺失和不完整

1.  过时了（Obj-C 或较旧的 Swift 版本文档很常见）

1.  写得很糟糕（假设了一种熟悉程度，使得阅读文档变得不必要）。缺乏解释。

1.  缺乏甚至基本用法的示例。

1.  设计不良的、遗留的面向对象编程API：通过良好的类型使用，Swift 有时可以在不需要太多文档的情况下运行。不幸的是，许多 MacOS 的API是一种老式面向对象编程与令人惊讶的低级细节的混合体。

## 如果你在进行苹果开发呢？

避免使用苹果的文档（即使它们是谷歌搜索的前几个结果）。它们可能会很糟糕。有时候 Stack Overflow 还不错（但往往是旧版本的 Swift）。一些第三方文档比较好（Paul Hudson、Ray Wenderlich）。
