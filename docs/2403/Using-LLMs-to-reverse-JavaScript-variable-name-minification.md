<!--yml

category: 未分类

date: 2024-05-27 15:03:12

-->

# 使用LLMs反转JavaScript变量名缩小

> 来源：[https://thejunkland.com/blog/using-llms-to-reverse-javascript-minification.html](https://thejunkland.com/blog/using-llms-to-reverse-javascript-minification.html)

这篇博客介绍了使用大型语言模型（LLMs），如ChatGPT和llama2来反转缩小的JavaScript的新方法，同时保持代码的语义完整性。该代码是开源的，可在[Github项目Humanify](https://github.com/jehna/humanify)找到。

## 什么是缩小？

缩小是减少JavaScript文件大小以便优化快速网络传输的过程。从逆向工程的角度来看，有几个不同类别的缩小提出了越来越大的挑战：

### 无损缩小

大多数缩小是无损的；当`true`转换为其缩小的替代项`!0`时，不会丢失数据。可以很容易地[编写一个Babel转换](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)来反转此过程。有[许多](https://www.npmjs.com/package/babel-plugin-transform-beautifier) [工具](https://www.npmjs.com/package/debundle)专门设计用于反转这些类型的无损转换。

### 不重要的数据丢失

在缩小过程中会丢失一些数据，但这些数据可能很容易重新创建。一个很好的例子是空白；使用[Prettier](https://prettier.io)（或类似工具）重新格式化缩小代码的缩进和空白到人类可读的格式是微不足道的。大多数情况下，开发人员在原始代码中也使用了类似的工具，因此可以高度自信地重新创建空白数据。

### 变量名称

在缩小过程中丢失的最重要信息是变量和函数名的丢失。当你运行一个缩小工具时，它会完全替换所有可能的变量和函数名，以节省字节。

到目前为止，还没有任何好的方法来反转这个过程；当你将变量从`crossProduct`重命名为`a`时，你几乎无法反转该过程。

## 人类如何反转缩小？

许多逆向工程师通过从代码的上下文中识别一些启发式规则来形成对代码用途的有教育意义的猜测。让我们看一个简单的例子：

```
function a(b) {
  return b * b;
}
```

如何重新命名函数`a`？根据上下文，我们可以猜测原始名称可能类似于`square`。但这需要了解函数的内部工作原理。

让我们尝试系统化地描述重命名函数的过程：

1.  阅读函数的主体

1.  描述函数的作用

1.  尝试提出一个符合该描述的名称

对于传统的计算机程序来说，从“将 `b` 与其自身相乘”到“计算一个数的平方”是非常困难的。幸运的是，LLM的最新进展使得这种转变不仅可能，而且几乎是微不足道的。

实际上，步骤 2\. 被称为“重新表述”（或者如果你把Javascript视为自然语言，则称为“翻译”），而LLMs在这方面表现非常出色。

另一个LLMs真正闪耀的任务是摘要，这几乎就是我们在步骤 3\. 所做的事情。这里唯一的专业化是输出需要足够短，并且格式化为驼峰式。

## 控制LLMs

使用LLM输出的问题在于它们不是确定性的。简言之，LLM是一个非常复杂的马尔可夫链；它尝试根据前面的单词猜测文本中的下一个单词。

这意味着即使我们有一个像这样好的提示：

```
Are all roses red? Please answer only "Yes" or "No".
```

即使如此，LLM可能仍会回答“不，但是...”，“我不知道”或者著名的“对不起，但作为一个AI语言模型，我不能……”。

这曾经是一个问题，但幸运的是现在有了控制LLM输出的方法，如 [guidance](https://guidance.readthedocs.io/) 和 [outlines](https://github.com/normal-computing/outlines)。这些工具使用不同的技术确保LLM的输出与所需的格式匹配。

幸运的是，Javascript变量只能有特定的格式，因此可以使用正则表达式匹配输出，以确保输出是有效的Javascript变量名。

## 不要让AI触碰代码

现在虽然LLM在重新表述和摘要方面非常出色，但在编码方面并不是很擅长（至少目前如此）。它们具有固有的随机性，这使它们不适合执行实际的重命名和代码修改。

幸运的是，在传统工具如Babel中，重新命名Javascript变量在其作用域内已经是一个解决的问题。Babel首先将代码解析成抽象语法树（AST，代码的机器表示），然后可以使用良好的算法轻松修改。

这比让LLM在文本级别修改代码要好得多；它确保只进行非常具体的转换，以确保重命名后代码的功能不会改变。代码保证具有原始功能，并且可以由计算机运行。

## 将所有内容整合在一起

那么，我们如何反向缩小Javascript呢？让我们把所有内容整合起来：

1.  使用 [webcrack](https://github.com/j4k0xb/webcrack) 解开webpack捆绑包

1.  将代码通过 [transform-beautifier](https://www.npmjs.com/package/babel-plugin-transform-beautifier) 和几个自定义的Babel插件运行，这些插件会反向无损缩小化。

1.  循环遍历代码中的所有变量，要求LLM描述它们的意图，并基于该描述生成更好的名称。

1.  使用Babel重命名变量

1.  最后运行一轮 [Prettier](https://prettier.io/) 确保良好的空白格式。

就这样了！给出以下代码：

```
function a(e,t){var n=[];var r=e.length;var i=0;for(;i<r;i+=t){if(i+t<r){n.push(e.substring(i,i+t))}else{n.push(e.substring(i,r))}}return n}
```

工具输出如下：

```
function chunkedString(inputStringToBeSliced, chunk) {
  var chunkBuffer = [];
  var sliceSize = inputStringToBeSliced.length;
  var currentCharIndex = 0;
  for (; currentCharIndex < sliceSize; currentCharIndex += chunk) {
    if (currentCharIndex + chunk < sliceSize) {
      chunkBuffer.push(
        inputStringToBeSliced.substring(
          currentCharIndex,
          currentCharIndex + chunk
        )
      );
    } else {
      chunkBuffer.push(
        inputStringToBeSliced.substring(currentCharIndex, sliceSize)
      );
    }
  }
  return chunkBuffer;
}
```

## 试试看吧！

工具名为[Humanify，可在Github找到](https://github.com/jehna/humanify)。请查看并确认它是否适合你！
