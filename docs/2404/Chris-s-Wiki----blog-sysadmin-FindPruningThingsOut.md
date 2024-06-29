<!--yml

category: 未分类

date: 2024-05-27 13:33:39

-->

# Chris's Wiki :: blog/sysadmin/FindPruningThingsOut

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/FindPruningThingsOut](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/FindPruningThingsOut)

假设您需要扫描您的文件系统并传递一些具有特定名称、所有权或其他内容的文件，但您希望排除扫描/tmp和/var/tmp下的文件（作为说明性示例）。也许您正在将文件名馈送给shell脚本，特别是在管道中，这意味着您希望筛选出具有（常见）问题字符（如空格）的目录和文件名。

（如果您可以在shell脚本中使用Bash，后一个问题可以解决，因为您可以让Bash读取由 'find ... -print0' 生成的NUL终止行。）

从find的 -prune 操作中排除结果，当您想要排除绝对路径时（好吧，总之，这通常有点棘手；请参见[此 SO 问题和答案](https://stackoverflow.com/questions/1489277/how-to-use-prune-option-of-find-in-sh)）。首先，您需要生成一个文件系统列表，然后通过绝对路径扫描它们：

> ```
> FSES="$(... something ...)"
> for fs in $FSES; do
>     find "$fs" -xdev [... magic ...]
> done
> 
> ```

从文件系统的绝对路径开始（而不是cd到文件系统根目录并执行 'find . -xdev [...]` 的方式，意味着我们现在可以在find的 -path 参数中使用绝对路径，而不是相对于文件系统根目录的路径：

> ```
> find "$fs" -xdev '(' -path /tmp -o -path /var/tmp ')' -prune -o ....
> 
> ```

使用绝对路径，我们不必担心/var或/tmp（或/var/tmp）是否是单独的文件系统，而不是根文件系统上的目录。尽管没有进行实验，但-xdev和-prune的组合方式正是我们想要的。

（如果我们在一个不包含/tmp或/var/tmp的文件系统上运行 'find'，尽管永远不可能匹配它们，我们会浪费一点CPU时间让 'find' 一直评估这些 -path 参数。与拥有一个更简单、更少错误的脚本相比，这并不重要。）

如果我们想要排除路径中带有空格的情况，可以使用 '`-name "* *"`' 轻松实现。如果我们想要获取所有空白字符，我们需要使用 [GNU Find](https://man7.org/linux/man-pages/man1/find.1.html) 及其 '-regex' 参数，最好在 ["Regular Expressions" in the info documentation](https://www.gnu.org/software/findutils/manual/html_mono/find.html#Regular-Expressions) 中有详细说明。因为我们想要使用 [字符类](https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html) 来匹配空白字符，所以我们需要使用其中包含的正则表达式类型之一：

> ```
> find "$fs" -regextype grep ... -regex '.*[[:space:]].*' ...
> 
> ```

总体而言，对于这种过滤任务，'`find`' 是一个使用起来很笨拙的工具。不幸的是，有时这是我们不得不使用的工具，因为其他选择可能涉及编写消耗和过滤NUL终止文件路径的程序。

（而让'`find`'跳过整个目录树比让其进入目录树、打印所有文件路径，然后稍后再过滤文件路径更有效率。）

附言：Unix中对系统管理员的一些小烦恼之一是，许多事情在标准Unix环境中一旦人们开始在文件名中添加奇怪的字符，就会崩溃，除非你极度小心并使用非常规的工具。这通常影响到系统管理员，因为我们经常不得不处理其他人几乎任意选择的文件和目录名称，而且我们可能正在处理积极恶意的攻击者，这更值得关注。

### 侧边栏：在Bash中读取以NUL结尾的行

[Bash中的'`read`'内建版本](https://www.gnu.org/software/bash/manual/bash.html#index-read)支持一个'-d'参数，用于读取以NUL结尾的行：

> ```
> while IFS= read -r -d '' line; do
>   [ ... use "$line" ... ]
> done
> 
> ```

在每次使用时仍然必须正确引用`"$line"`，特别是在您这样做的情况下，因为您期望您的行（或文件名）有时会包含问题字符。您应该绝对使用[Shellcheck](http://www.shellcheck.net/)，并且密切关注它的警告（[它们对您有好处](/~cks/space/blog/programming/ShellcheckGoodForMe)）。
