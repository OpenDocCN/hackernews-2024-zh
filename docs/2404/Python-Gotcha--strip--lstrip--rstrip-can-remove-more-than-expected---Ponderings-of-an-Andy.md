<!--yml

类别：未分类

日期：2024-05-27 13:35:04

-->

# Python 的坑：`strip`，`lstrip`，`rstrip` 可以移除比预期更多的内容 · Andy 的思考

> 来源：[https://andrewwegner.com/python-gotcha-strip-functions-unexpected-behavior.html](https://andrewwegner.com/python-gotcha-strip-functions-unexpected-behavior.html)

## 介绍

作为一名软件工程师，你已经处理过很多脏字符串。删除前导或尾随空格可能是对用户输入最常见的操作之一。

在 Python 中，可以通过 [`.strip()`](https://docs.python.org/3.10/library/stdtypes.html#str.strip)，[`.lstrip()`](https://docs.python.org/3.10/library/stdtypes.html#str.lstrip) 或 [`.rstrip()`](https://docs.python.org/3.10/library/stdtypes.html#str.rstrip) 函数来实现，通常是这样的：

```
`>>> "     Andrew Wegner     ".lower().strip()
'andrew wegner'
>>> "     Andrew Wegner     ".lower().lstrip()
'andrew wegner     '
>>> "     Andrew Wegner     ".lower().rstrip()
'     andrew wegner'` 
```

这非常直接，没有出现意外情况。

## 坑

坑是每个函数都可以移除一个可以移除的字符列表。

```
`>>> "Andrew Wegner".lower().rstrip(" wegner")
'and'` 
```

发生了什么？为什么结果不仅是

## 解释

再次仔细阅读文档中的这一行：

> 一个**字符**列表

不是字符串列表。

这在文档中是明确说明的，并附有一个示例，展示了其影响。然而，对于新开发者来说，这是意外的行为。毕竟，这些函数看起来是直观的。

例子中的操作如下：

1.  接收一个要移除的字符列表。在这种情况下，是我的姓氏中的所有字母，加上空格字符：`wegner`

1.  将输入字符串中的所有字母转换为小写字母，结果为 `andrew wegner`

1.  从字符串的右侧开始移除属于输入列表中的字符。遇到不在列表中的字符时停止。在这种情况下，`rengew wer` 被移除（从右到左），然后在 `andrew` 中遇到了 `d`，因此 `rstrip` 函数停止。

1.  返回剩余的字符串 `and`

## 解决方案

Python 有两个函数可以正确地移除一个**字符串** - [`.removesuffix()`](https://docs.python.org/3.10/library/stdtypes.html#str.removesuffix) 和 [`.removeprefix()`](https://docs.python.org/3.10/library/stdtypes.html#str.removeprefix) 用于右侧和左侧的移除。

```
`>>> "Andrew Wegner".lower().removesuffix(" wegner")
'andrew'` 
```

这两个函数是作为 Python 3.9 的一部分引入的，作为 [PEP-616](https://peps.python.org/pep-0616/) 的一部分。在 PEP 中，明确指出了用户对 `*strip()` 函数以及它们行为的混淆。引入这两个函数是为了允许所需的行为。

一个重要的说明是这两个 `remove*` 函数只会最多移除一个实例的字符串。

```
`>>> "Andrew Wegner Wegner".lower().removesuffix(" wegner")
'andrew wegner'` 
```
