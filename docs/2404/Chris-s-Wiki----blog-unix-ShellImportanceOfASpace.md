<!--yml

category: 未分类

date: 2024-05-27 13:34:40

-->

# Chris's Wiki :: blog/unix/ShellImportanceOfASpace

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/unix/ShellImportanceOfASpace](https://utcc.utoronto.ca/~cks/space/blog/unix/ShellImportanceOfASpace)

在[昨天的文章](/~cks/space/blog/sysadmin/FindPruningThingsOut)的侧边栏中，我（最初）因为一个无意的 Unix 命令行错误而犯了一个错误，我不经意地漏掉了一个看起来非常普通和无害的空格（在 Emilio 的评论中指出后，我已经在当前版本的文章中将其纠正）。这个看似无辜的错误及其后果说明了 Unix shell 命令行中的一些问题，尽管我不确定具体是什么问题，所以我打算写出来。

故事始于[Bash 的‘`read`’内建](https://www.gnu.org/software/bash/manual/bash.html#index-read)的一般参数：

> `read [-ers] [-a *aname*] [-d *delim*] [-i *text*] [-n *nchars*] [-N *nchars*] [-p *prompt*] [-t *timeout*] [-u *fd*] [*name* …]`

‘`read`’ 内建遵循 Unix 命令的一般标准行为，选项 '-d *delim*' 和其他需要参数的选项可以缩写以忽略空格，因此 '-d*delim*'。所以你可以写，例如：

> ```
> echo "a:b:c" | while IFS= read -r -d':' l; do echo "$l"; done
> 
> ```

Bash 还有一个特殊功能是 -d。通常，*delim* 的第一个字符被视为“行”终止符，但如果 *delim* 为空白，`read` 将在读取 NUL 字符（0 字节）时终止行，这正是你处理例如 'find ... -print0' 输出所需的方式。

在 Bash 命令行中创建一个空字符串参数的方法是使用一对空引号：

> ```
> read -r -d '' line
> 
> ```

所以当我在[昨天的文章](/~cks/space/blog/sysadmin/FindPruningThingsOut)中编写原始命令行时，我心不在焉地把这两者混在一起，写成了：

> ```
> read -r -d'' line
> 
> ```

我使用''创建一个空参数，然后我标准地移除了 -d 和其参数之间的空格。所以清楚地我给了 -d 一个空参数，对吗？错了。

在 Bash 和其他传统 shell 中，'' 表示无内容。只有当它单独出现时，它才表示一个空字符串参数；这是 shell 添加的特殊解释，程序实际上不会看到这些 ''。如果你将 '' 放在其他非空白字符旁边，它们在程序所看到的命令行中会消失。因此，写 '-d''' 与写没有参数的 '-d' 是相同的，而 '`read`' 命令行会看到它实际上是：

> ```
> read -r -d line
> 
> ```

这会导致 '`read`' 以 'l' 作为行终止符。

在写这篇文章的过程中，我意识到了另一种更有趣地犯同样错误的方法，尽管这更深入 Unix 奥秘，看起来并不那么明显。在许多现代 shell 中，包括 Bourne shell，在命令行中你可以用 `$'\0'` 来表示 NUL 字符（0 字节）。因此，你会看到人们将 'read with NUL terminated lines' 命令行写成：

> ```
> IFS= read -r -d $'\0' line
> 
> ```

这很好地运作，而且不像''情况下我们明显有一个真实的参数，而不仅仅是一个空参数，因此显然我们可以将其缩短为：

> ```
> IFS= read -r -d$'\0' line
> 
> ```

如果你尝试这样做，你会发现它不起作用。根本问题在于**Unix命令行参数不能包含NUL字符**，因为Unix命令行API将参数作为以NUL结尾的(C风格)字符串数组传递。无论如何调用程序，参数中的第一个NUL字符都会被程序视为该参数的结束。因此，尽管从键盘输入时看起来非常不同，但从`read`的角度来看，我们所做的与以下操作相同：

> ```
> IFS= read -r -d line
> 
> ```

（然后它将产生与我的错误相同的效果。）

PS：这有点复杂，因为'`read`'是Bash的内置命令，理论上Bash不必遵守内核API的限制，但实际上我认为Bash确实遵守了这些限制。
