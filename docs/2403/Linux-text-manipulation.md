<!--yml

类别：未分类

日期：2024-05-29 12:44:45

-->

# Linux文本操作

> 来源：[https://yusuf.fyi/posts/linux-text-manipulation](https://yusuf.fyi/posts/linux-text-manipulation)

今天，我想为我的AwesomeWM状态栏制作一个模块，显示当前播放的Spotify歌曲，如果能显示类似这样易于理解的内容就更好了：*Wild World by Yusuf/Cat Stevens*。

通过快速搜索，我找到了一个小巧的bash脚本，[sp](https://gist.github.com/streetturtle/fa6258f3ff7b17747ee3)，可以完成我需要的功能。不过输出必须进行清理。

```
joe@popos> sp current
Album Tea For The Tillerman (Remastered 2020)
AlbumArtist Yusuf / Cat Stevens
Artist Yusuf / Cat Stevens
Title Wild World 
```

我可以让chatgpt格式化这篇文章或编写一个Lua脚本，但那就没什么乐趣了，*对吧？*在本文中，我们将学习如何使用`awk`、`sed`和其他命令来获得所需的输出。

预期读者

这篇文章适用于那些已经使用过Linux但对终端仍然新手的人，如果你至少能从命令行创建一个文件，那就可以了，我会在整篇文章中覆盖其他所有内容。

如果你想跟着学，可以下载bash脚本，并尝试我们将学习的命令，但这并非必须。

### 迷你问题＃1

要开始使用，`sp current`返回专辑名称、专辑艺术家、歌曲艺术家和歌曲标题，但我们只需要艺术家和标题。我们可以使用`sed`来选择我们需要的特定行。

`sed`通过循环处理输入的每一行并执行给定的操作， 默认情况下将结果行打印到标准输出。我们可以给`sed`提供一个要处理的文件，但更常见的是将一个命令的输出管道传递给`sed`，正如我们稍后将会做的那样。

> <details><summary>你所说的***管道***是什么意思？</summary>通常在Linux中，命令的输入来自键盘，并将其输出（以及任何错误）打印到屏幕上。然而，我们可以改变这种行为。</details>
> 
> 管道本质上允许你将一个命令的输出传递到另一个命令的输入。程序可以选择是否支持管道，但大多数基本的Linux程序（以及我们今天将讨论的所有程序）都支持它。

让我们更详细地了解`sed`。

```
sp current | sed '' # no action given, sed will print the lines as is
sp current | sed 's/Cat/Dog/g' # replace Cat with Dog and print every line 
```

```
# the n flag suppresses the automatic printing
# this will replace Cat with Dog but print nothing!
sp current | sed  -n 's/Cat/Dog/g' 
```

```
# p prints a line
# so this will print each line twice (default + p action)
sp current | sed  'p'

# if we combine -n with the p action we can do this
# this is equivalent to plain sed ''
sp current | sed -n  'p'

# we can print a specific line too (we'll need this later!)
# this will print every line AND print the second line an extra time
sp current | sed '2p' 
```

```
# finally, we can chain actions.
# this substitutes Cat with Dog and prints ONLY the 2nd line, because of the n flag.
sp current | sed -n 's/Cat/Dog/g;2p' 
```

现在——你可能已经发现——我们可以仅仅这样打印艺术家和歌曲名：

```
sp current | sed -n '3p;4p' 
```

然后我们得到了

```
Artist       Yusuf/Cat Stevens
Title        Wild World 
```

### 迷你问题＃2

接下来，我们需要去掉列标签，我们可以使用`cut`来做这件事。与`sed`类似，`cut`接受文件作为其输入，如果没有提供文件，则将从标准输入读取。

这是我们通常使用`cut`的方式，你给它一个分隔符和一组要打印的列，它将每一行按照该分隔符分割，并返回选定的列。

在我们的情况下，列是用空格分隔的，因此我们可以这样做。

```
# we're splitting the input by spaces and picking the 2nd column
sp current | cut  -d ' '  -f2 
```

我们的列由多个空格分隔，所以cut将每个空白之间的空白视为其自己的列，为了避免这种情况，我们可以使用`tr`命令来修剪额外的空格：

```
# the s flag specifies what to trim
sp current |  tr -s ' ' | cut -d ' ' -f2 
```

但这还不是完全正确。如果歌曲标题包含多个单词，我们将会得到这样的结果。

```
Artist       Yusuf/Cat Stevens
Title        Wild World

Yusuf/Cat
Wild 
```

你可以向`cut`传递一个列范围以打印，例如`cut -d ' ' -f2-4`来选择第2、第3和第4列，**如果范围是开放的**，`cut` **将打印直到行尾的每一列**，所以让我们将它们合并到我们的脚本中。

```
sp current | tr -s ' ' | cut -d ' '  -f2- | sed -n '3p;4p' 
```

输出：

```
Yusuf/Cat Stevens
Wild World 
```

**注意**：正如我向你展示的那样，在`cut`之前需要应用`tr`，相反，`cut`和`sed`可以交换，因为它们提供完全不同的功能。

### 小问题＃3

现在，剩下的就是简单地连接这些行并进行良好的格式化，我们可以使用`awk`来实现这一点。

`awk`基于条件脚本模型，类似于`sed`，它检查当前行是否符合条件并执行相应的脚本，尽管我觉得`awk`的语法更加表达丰富。

awk脚本用引号括起来写成这样：

```
sp current | awk '{print $1}' 
```

让我们看几个例子。任何这些都可以用引号传递给awk。

```
{print} # no condition given, prints every line.

# NR==1 matches the 1st line only
# thus, this will print the 1st line
NR==1{print} 

# any subsequent script isn't part of the condition. 
# despite them being on the same line.
# this will print the print every line + the first line again
NR==1{print}{print} 
```

```
# awk has pre-defined globals corresponding to columns in your input
# this behaves like sed and will print the first column.
{print $1}

# $0 matches the entire line
# this is equivalent to {print}
{print $0} 
```

最后，你可以将全局变量赋值给变量，

这将`a`赋给第1行的第1列，然后对输入的每一行打印`x`。请注意，第一个脚本只运行一次，因此`a`对于每一行都保持不变。

```
NR==1{x=$1}{print x} 
```

现在，我们可以结合我们所学的一切来得到这个脚本：

```
sp current | ... | awk 'NR==1{a=$0; next}{print $0 " by " a}' 
```

这做了以下几件事：

1.  我们将`a`赋给艺术家的名字（整个第一行）

1.  `skip`告诉`awk`跳过第二个脚本的执行并移动到下一行。

1.  我们现在在第二行上，因此`NR==1`不匹配并且第一个脚本被跳过。

1.  打印歌曲名（`$0`），然后是字符串文字`by`，然后是艺术家的名字（变量`a`的内容）

```
Yusuf/Cat Stevens
Wild World

Wild World by Yusuf/Cat Stevens 
```

## 最后的想法

至此，我们的脚本结束了，这就是我们最终得到的结果：

```
sp current | tr -s ' ' | cut -d ' ' -f2- | sed -n '3p;4p' | awk 'NR==1{a=$0; next}{print $0 " by " a}' 
```

现在，这个脚本可以以不同的方式进行优化，但是我不会在这里向你展示如何做，我把它留给读者作为一个练习。这里有一些你可以做的事情，从最简单到最难：

+   用额外的`sed`操作替换`cut`命令

+   从记忆中重写整个内容

+   使脚本尽可能简短

+   在`awk`中编写整个脚本

最后，我希望今天你学到了新东西，如果你尝试了其中的任何一个，请分享你做了什么，无论是通过Mastodon回复还是在你的博客文章中。我承诺会看一看，而且我可能也会学到一些新东西！
