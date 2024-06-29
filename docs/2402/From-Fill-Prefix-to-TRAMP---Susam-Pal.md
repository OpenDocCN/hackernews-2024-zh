<!--yml

类别：未分类

日期：2024-05-27 15:06:28

-->

# 从填充前缀到 TRAMP - Susam Pal

> 来源：[https://susam.net/from-fill-prefix-to-tramp.html](https://susam.net/from-fill-prefix-to-tramp.html)

<main>

# 从填充前缀到 TRAMP

由 **Susam Pal** 于 2023 年 12 月 30 日撰写

我们的 [小型书籍俱乐部](cc/mastering-emacs/) 通常在周末和假期聚会，并讨论书籍 [Mastering Emacs](https://www.masteringemacs.org/)，2022 年版今天结束了。在今天的 [最后一次会议](cc/mastering-emacs/log.html#72) 中，我们首先讨论了如何在同一个 Dired 缓冲区中跨多个目录工作。然后我们演示了 Emacs 自带的各种 Shell 和终端模式。这完成了我们对第 6 章的讨论。然后我们转向第 7 章（最后一章），首先重申了使用 describe-system 向 Emacs 提问其自身的重要性，然后提出了关于第三方包和在线 Emacs 社区的一些建议。完成这一章也结束了我们的书籍俱乐部讨论。

非常感谢 Mickey Petersen 写了这本书，并且在我们讨论时非常慷慨地允许我分享他的书籍。

这个读书俱乐部始于 2022 年 12 月 16 日，当时我们在 Jitsi 上举行了第一次会议。开始这些会议大约 3 个半月后，我在另一篇名为 [从月相到摇摇晃晃](from-lunar-phases-to-yank-pop.html) 的博文中发布了关于这个书籍俱乐部的更新。如果你还没有阅读过那篇博文，我建议你在回到这篇文章之前先阅读一下。特别是如果你最近开始学习 Emacs，我认为你会发现那篇文章很有用。

当我发布最后一次更新时，我们一共在 36 次会议中共同度过了约 26 小时，当时我们正在阅读这本书的第 5 章。完成这一章节和剩下的两章共计 36 次会议。在总共 72 次会议后，我们今天完成了讨论该书第 7 章，这也结束了这一系列书籍俱乐部会议。总计，我们共度过了略多于 52 小时来讨论这本书，尝试了书中介绍的每一个概念和命令，并与彼此分享了对材料的见解。

在这篇文章中，我将分享自上次更新以来我们会议的一些亮点。这些亮点分享了我们学习到的一些概念和命令，这些概念和命令大多数我们的书籍俱乐部成员之前并不熟悉，但在学习后发现非常有用。

## 目录

## 填充前缀

我们书籍讨论小组中的大多数人都知道如何使用 `M-q` 来填充段落。考虑以下格式混乱、行长短不一的段落：

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
incididunt ut labore et dolore
magna aliqua.  Arcu dui vivamus arcu felis bibendum ut tristique et egestas.
Bibendum arcu vitae
elementum curabitur vitae.

```

现在将光标放在段落的任何位置，然后键入 `M-q`。 段落将被整齐地格式化为以下内容：

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
eiusmod tempor incididunt ut labore et dolore magna aliqua.  Arcu dui
vivamus arcu felis bibendum ut tristique et egestas.  Bibendum arcu
vitae elementum curabitur vitae.

```

键序列 `M-q` 调用 `fill-paragraph` 命令，重新格式化段落，使每行尽可能长，但不超过填充宽度（默认为70列）。我们中的大多数人在写作和编辑文本时经常使用此命令。但是，一些人不知道的是在填充段落时会考虑填充前缀的存在。为了说明这个概念，我们首先考虑这个格式不良的段落：

```
:::: Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
:::: incididunt ut labore et dolore
:::: magna aliqua.  Arcu dui vivamus arcu felis bibendum ut tristique et egestas.
:::: Bibendum arcu vitae
:::: elementum curabitur vitae.

```

每行都有一个由四个冒号和一个空格组成的前缀。使用 `M-q` 重新格式化这个段落后，我们得到类似这样的结果：

```
:::: Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
eiusmod tempor :::: incididunt ut labore et dolore :::: magna aliqua.
Arcu dui vivamus arcu felis bibendum ut tristique et egestas.  ::::
Bibendum arcu vitae :::: elementum curabitur vitae.

```

这不是我们想要的。我们希望段落的格式不要超过70个字符的长度（事实上，我们已经做到了），每一行都包含四个冒号和一个空格作为前缀（上面的格式已经破碎了）。我们能做到吗？可以，通过设置填充前缀。键入 `C-/` 或 `C-x u` 来撤消刚才做的不良格式化，并让我们再试一次。这次将光标移动到 `Lorem` 的 `L` 上，键入 `C-x .` 来设置填充前缀为当前行直到光标的内容。确认消息会在回显区打印，告知 `":::: "` 已被设置为填充前缀。然后键入 `M-q`，段落现在整齐地格式化如下：

```
:::: Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
:::: eiusmod tempor incididunt ut labore et dolore magna aliqua.  Arcu
:::: dui vivamus arcu felis bibendum ut tristique et egestas.
:::: Bibendum arcu vitae elementum curabitur vitae.

```

注意每行尽可能长，但不超过70个字符的长度，并且每行都有填充前缀。Emacs 负责从结果的每一行中移除填充前缀，从每行的最大字符预算中减去填充前缀的长度，重新格式化行，并在结果的每一行上重新插入填充前缀。

要关闭填充前缀，只需将其设置为空前缀，通过在行首键入 `C-x .`。因此 `C-a C-x .` 成为关闭填充前缀的惯用语。

## 替换字符串中的 Elisp 表达式

书讨论小组中没有人会对键序列 `C-M-% f.. RET bar RET` 开始的搜索和替换操作感到意外，该操作用于将与正则表达式模式 `f..` 匹配的字符串替换为文本 `bar`。

大多数人也知道反向引用的概念。例如，`C-M-% \(f..\)-\(b..\) RET \2-\1 RET` 搜索与给定正则表达式模式匹配的字符串，并用新字符串替换它，交换第一个捕获组和第二个捕获组的位置。反向引用 `\1` 指的是第一个捕获组 `\(f..\)` 匹配的字符串，类似地 `\2` 指的是第二个捕获组 `\(b..\) `匹配的字符串。在这个例子中，像 `foo-bar` 将被替换为 `bar-foo`，或者 `playful-banter` 将被替换为 `playban-fulter`。

然而，令我们一些人感到惊讶的是，我们还可以在替换字符串中使用 Elisp 表达式。这样做的语法是在替换字符串中写入 `\,`，然后跟随替换字符串中的 Elisp 表达式。例如，考虑键序列 `C-M-% f.. RET \,(upcase \&) RET`。请注意，我们使用了反向引用 `\&`，它将整个匹配作为参数传递给 Elisp 函数 `upcase`，将其转换为大写形式。这个示例搜索与模式 `f..` 匹配的字符串，并用匹配项的大写形式替换它。例如，字符串 `foo-bar` 将被替换为 `FOO-bar`。

这里是另一个稍微复杂一些的示例：`C-M-% port-\([0-9]+\) RET port-\,(+ 1000 \#1)`。反向引用 `\#1` 指的是第一个捕获组 `\([0-9]+\)` 匹配的字符串作为*数字*。替换模式中的 Elisp 表达式简单地将1000添加到该数字，并用结果替换匹配的字符串。例如，字符串 `port-80` 变成 `port-1080`。

## 保留行和清除行

我们小组成员喜欢学习的一组巧妙的命令是保留和清除行的命令。在过滤大型日志文件时，这些命令非常有用。以下是这些命令的简要说明：

+   `M-x keep-lines RET f.. RET`: 保留区域内与正则表达式 `f..` 匹配的行，删除其余行。如果没有活动区域，则保留指针和缓冲区末尾之间匹配的行，并删除其余行。删除的行不会复制到 kill ring。

+   `M-x flush-lines RET f.. RET`: 删除区域内与正则表达式 `f..` 匹配的行。如果没有活动区域，则删除指针和缓冲区末尾之间匹配的行。删除的行不会复制到 kill ring 中。

请注意上述每个要点都提到删除的行不会复制到 kill ring 中。如果我们想快速将删除的行粘贴到另一个缓冲区，这可能有些不便。Emacs 28.1 引入了一些更多命令，在一定程度上解决了这种情况。以下是这些命令：

+   `M-x copy-matching-lines RET f.. RET`: 复制区域内与正则表达式 `f..` 匹配的行。如果没有活动区域，则复制指针和缓冲区末尾之间匹配的行。

+   `M-x kill-matching-lines RET f.. RET`: 删除区域内与正则表达式 `f..` 匹配的行到 kill ring 中。如果没有活动区域，则删除指针和缓冲区末尾之间匹配的行。

## 键盘宏

我们小组中大多数有经验的 Emacs 用户都知道键盘宏。然而，一些人在我们的会议中第一次了解到这一优秀的自动化功能，因此我认为这在本文中值得单独一节。

键盘宏是一个庞大的主题，最好从手册中的[键盘宏](https://www.gnu.org/software/emacs/manual/html_node/emacs/Keyboard-Macros.html)部分学习。在 Emacs 中，键入 `M-: (info "(emacs) Keyboard Macros") RET` 来使用 Info 文档浏览器打开此部分。在本博文中，我们将简要讨论足以让初学者快速入门的键盘宏。

假设我们有一个如下所示的缓冲区：

```
foo:bar:baz
bar:baz:qux
quux:corge:grault
garply:waldo:fred

```

现在假设我们想要交换每行中由冒号分隔的前两个字段。当然，我们可以使用正则表达式来完成，例如使用键序列 `C-M-% ^\(.+?:\)\(.+?:\) RET \2\1 RET`。但我们也可以通过简单地对一行执行必要的编辑来解决这个问题。然后要求 Emacs 在其他行上重复我们所做的。以下是步骤：

1.  首先将光标移动到第一行的某处。

1.  然后输入 `C-x (` 开始录制键盘宏。

1.  然后输入 `C-a M-d C-d M-f : C-y C-n` 来交换第一行的第一个和第二个字段，并将光标移动到下一行。这只是实现交换的一种方式。您可以使用任何您熟悉的编辑命令来进行交换并移动光标到下一行。

1.  现在输入 `C-x )` 来停止宏录制。

1.  现在输入 `C-x e` 来在第二行重放宏。一旦我们键入此键序列，第二行中的交换就会发生，光标移动到第三行。重复此键序列以在后续行上重复交换操作。

总结一下，`C-x (` 开始录制新的键盘宏，`C-x )` 停止录制键盘宏，`C-x e` 重放最后一个键盘宏。或者，我们还可以使用功能键 `F3` 和 `F4`。要开始录制键盘宏，请键入 `F3`。键入 `F4` 来停止录制键盘宏。然后再次键入 `F4` 来重放最后一个宏。

特别注意上述步骤 3。我们以 `C-a` 开始键序列，将光标移动到第一列。当光标已经位于第一列时，这可能会感觉多余。然而，在我们的会议中，我经常强调这一点的重要性。在定义要记录的键盘宏的其余部分时，键入 `C-a` 确保我们不会带入任何关于光标位置的假设。通过键入 `C-a`，我们确保无论光标位于行的何处，当我们重放宏时，光标首先会移动到行的开头。这一保证使我们能够自信地定义执行交换所需的其余编辑操作。

类似地，最后我们输入`C-n`将光标移动到下一行。在我们的会议中，我曾强调过这样做的重要性。将光标移动到下一行可以确保光标处于一个良好的位置，以便立即再次重复键盘宏。这就是为什么我们可以一遍又一遍地输入`C-x e`（或者`F4`）来重放宏。实际上，如果我们对键盘宏感到自信，我们可以使用数字参数自动重复执行多次。例如，输入`C-3 C-x e`（或者`C-3 F4`）来重复执行键盘宏3次。我们也可以输入`C-0 C-x e`（或者`C-0 F4`）来重复执行键盘宏，直到出现错误（例如，达到缓冲区末尾）。

## DAbbrev

DAbbrev 代表*dynamic abbrevation*。这是一个非常有用的包，我们中的许多人只是从我们的书籍俱乐部会议中了解到它。我们讨论了这个包支持的两个简单的键序列：

+   `M-/`: 将光标前的单词扩展到最近的前导词，其中当前单词是前缀。如果找不到合适的前导词，则将当前单词扩展到最近的后继单词，其中当前单词是前缀。

+   `C-M-/`: 查找缓冲区中所有具有光标前的当前单词作为前缀的单词，并将当前单词扩展到所有这些匹配单词的最长公共前缀。然而，如果匹配单词的最长公共前缀与光标前的单词相同，则将它们作为完成建议呈现。如果只有一个匹配单词，则将光标前的单词扩展为该单词。

让我们看一些例子。假设缓冲区中有以下一行文本：

```
abacus apple appliance application
```

现在，如果我们在下一行输入`ap`并键入`M-/`，DAbbrev 将自动将部分输入的单词扩展为`application`，因为这是具有`ap`前缀的最近单词。

但是，如果我们输入`ap`并键入`C-M-/`，则该单词扩展为`appl`，因为这是所有匹配单词中的最长公共前缀。如果我们再次键入`C-M-/`，则`apple`、`appliance`和`application`将作为可能的完成选项出现在名为`*Completions*`的临时缓冲区中。如果我们输入`ic`，使光标前的单词变为`applic`，然后键入`C-M-/`，它将扩展为`application`，因为现在这是唯一可能的完成选项。

这两个命令比上面段落中详细描述的内容听起来简单得多。当我们实际开始使用它们时，它们会立即变得直观。粗略地说，`M-/` 将扩展到光标前的最近的前导词，`C-M-/` 则会考虑文件中所有匹配的单词进行扩展，并在发现多个匹配时向用户呈现完成选项。

## TAB vs M-i

当我们在 Emacs 中键入 `TAB` 键时，它的行为对初学者来说可能会令人惊讶。在大多数其他主流编辑器中，该键要么插入一个制表符字符，要么插入足够数量的空格，以便将光标移动到下一个制表位。但在 Emacs 中，`TAB` 键大多数情况下根据缓冲区中启用的主要模式实现的语法规则对当前行进行缩进。

在其他编辑器上是一个简单的键，在 Emacs 中却是一个复杂的功能。`TAB` 的确切行为由诸如 `tab-always-indent`、`indent-line-function` 等变量控制。某些主要模式可能会参考其他类似的特殊变量来决定 `TAB` 的操作。然而，作为 Emacs 的用户，这并不是我们通常需要担心的事情。大多数主要模式适当设置所有这些变量，以便 `TAB` 几乎总是按照经验丰富的 Emacs 用户的期望行为，即正确缩进当前行的代码。

但是，如果我们真的只想插入一个制表符或足够数量的空格以将光标移动到下一个制表位列中，该怎么办？这可以通过按下 `M-i` 来实现。如果变量 `indent-tabs-mode` 设置为 `t`，那么 `M-i` 将插入一个字面上的制表符字符。如果设置为 `nil`，那么 `M-i` 将插入足够数量的空格，以将光标移动到下一个制表位列中。

总结一下，`M-i` 的行为类似于我们在其他编辑器中观察到的 `TAB` 行为。但实际上，在实践中，很少需要使用键序列 `M-i`。大多数人只需键入 `TAB` 即可自动缩进代码。事实上，我们也可以选择代码区域，然后键入 `TAB` 来重新缩进整个区域。

## 项目管理

出自名为 `project.el` 的包的默认项目管理命令使一些人感到惊讶。事实上，我们组中一些之前从未使用过项目管理命令的成员，现在在我们的会议中学习到它们后经常使用它们。

当我们使用类似 `C-x p f` 的项目命令来访问当前项目中的文件时，该命令会自动检测项目的顶级目录，方法是检查父目录中的版本控制系统工件（例如 `.git` 目录），并在该顶级目录中显示文件作为自动完成选项。

有关 Emacs 默认提供的项目管理功能，可以写很多东西。以下列表仅介绍了一些非常简单的功能，以便让初学者开始使用：

+   `C-x p f logger RET`：在当前项目中查找与 `logger` 匹配的文件。这会递归搜索所有子目录。如果只有一个匹配的文件（例如 `src/logger.cc`），则打开该文件。如果有多个匹配文件，则它们作为完成选项呈现。运行此命令或者事实上运行任何项目命令都会发现当前项目，并将发现的项目条目添加到 `~/.emacs.d/projects`。这对于像下一个点中提到的命令非常有用。

+   `C-x p f foo TAB RET logger TAB RET`：当我们在访问不属于任何项目的文件时键入`C-x p f`，它首先提示输入项目路径。在这个例子中，我们输入`foo TAB RET`来自动扩展到已知的项目路径，如`~/git/foo/`并进入其中。然后，我们输入`logger TAB RET`来自动扩展到文件名，如`src/logger.cc`并访问它。

+   `C-x p p bar TAB RET f logger TAB RET`：假设我们在项目`~/git/foo/`中，但我们想要切换到另一个先前发现的项目`~/git/bar/`并在那里找到一个文件。为此，我们首先输入`C-x p p`来切换项目。在项目选择提示符处，我们输入`bar TAB`来自动完成已知项目`~/git/bar/`的目录路径。然后，另一个提示出现以选择多个操作中的一个。在这种情况下，我们输入`f`来在我们切换到的项目中查找文件。最后，我们输入`logger TAB RET`来自动扩展部分输入的名称到路径，例如`src/logger.cc`并访问它。当当前文件属于一个项目但我们想要在另一个项目上运行项目命令时，关键序列`C-x p p`非常有用。

+   `C-x p p ... RET ~/git/baz/ RET f logger TAB RET`：这个奇怪的键序列发现了一个位于`~/git/baz/`的新项目目录，然后在其中找到一个文件。尽管如此，关键序列`C-x p p ... RET`很少需要。请参阅列表结束后的注释以了解原因。

+   `C-x p g ^key\> RET`：在当前项目中查找正则表达式`^key\>`的所有匹配项。结果显示在`*xref*`缓冲区中。

+   `C-x p s`：在当前项目的根目录中启动shell。

+   `C-x p d`：在当前项目的根目录中启动Dired。

+   `C-x p k yes RET`：关闭属于当前项目的所有缓冲区。

为了简洁起见，我们将以上项目命令列表到此为止。请注意第二点，即如果当前文件不属于任何项目，我们首先被提示输入项目名称。这是所有项目命令的共同主题。每当我们调用项目命令时，它会在当前项目上执行。但是，如果没有当前项目，则会自动提示我们输入项目名称以执行命令。

`C-x p p ... RET` 这个关键序列在日常编辑活动中很少需要。一旦一个项目被发现（例如因为早些时候在该项目上运行了一个项目命令并将其添加到`~/.emacs.d/projects`的已知项目列表中），我们就不需要再次发现它了。我们可以使用其他键序列来切换或处理已知项目。大多数日常项目活动涉及处理已知项目。

此外，即使我们想要发现一个新项目并将其添加到已知项目列表中，更自然的方法是在访问项目目录中的文件时运行项目命令。在大多数情况下，我们已经在当前缓冲区中打开了某个项目的文件。因此，直接运行项目命令，比如 `C-x p f`、`C-x p g` 等，而不是显式地通过 `C-x p p ... RET` 发现项目更有意义。仅仅在打开了某个项目文件时运行项目命令就会自动发现当前项目。几乎从不需要显式通过 `C-x p p ... RET` 发现项目。

## Eshell 与 TRAMP

我们团队中的许多成员分别了解了 Eshell 和 TRAMP。例如，`M-x eshell RET` 启动 Eshell。Eshell 完全使用 Elisp 实现，我们可以像使用常规 shell 一样使用它。以下是一个示例会话：

```
~ $ `cd /tmp/`
/tmp $ `echo hello > hello.txt`
/tmp $ `cat hello.txt`
hello
/tmp $ `python3 --version`
Python 3.11.5
/tmp $ `which cd echo cat python3 which`
eshell/cd is a byte-compiled Lisp function in ‘em-dirs.el’.
eshell/echo is a byte-compiled Lisp function in ‘em-basic.el’.
eshell/cat is a byte-compiled Lisp function in ‘em-unix.el’.
/usr/bin/python3
eshell/which is a byte-compiled Lisp function in ‘esh-cmd.el’.

```

我们也了解了 TRAMP。例如，当我们输入键序列 `C-x C-f /ssh:alice@box:/tmp/foo.txt RET` 时，TRAMP 察觉到我们打算通过 SSH 连接到名为 `box` 的远程系统，用户名为 `alice`，并编辑远程系统上名为 `/tmp/foo.txt` 的文件。TRAMP 然后会为我们透明地建立 SSH 连接。如果公钥身份验证已设置，则连接会立即成功建立。否则，它会提示输入密码。最终，我们得到一个编辑远程文件 `/tmp/foo.txt` 的缓冲区。一旦我们有了这个缓冲区，就无需额外操作即可在远程文件上工作。所有 Emacs 命令都能无缝地在这个远程文件的缓冲区上运行。例如，当我们键入 `C-x C-s` 时，TRAMP 会使用已建立的 SSH 连接保存文件到远程系统。如果我们键入 `C-x d`，TRAMP 会为远程目录 `/tmp/` 创建一个 Dired 缓冲区。所有我们熟悉的用于文件和目录操作的 Emacs 命令都可以正常与远程文件或目录一起使用。

因此，我们知道了 Eshell 和 TRAMP。然而，令我们许多人感到惊喜的是，Eshell 和 TRAMP 的协同工作效果如此显著。以下是一个示例 Eshell 会话，用以说明这一点：

```
~ $ `cd /ssh:alice@box:/tmp/`
/ssh:alice@box:/tmp $ `echo foo > foo.txt`
/ssh:alice@box:/tmp $ `ls`
foo.txt
/ssh:alice@box:/tmp $ `cd /tmp/`
/tmp $ `echo bar > bar.txt`
/tmp $ `ls`
bar.txt
/tmp $ `cp /ssh:alice@box:/tmp/foo.txt .`
/tmp $ `ls`
bar.txt  foo.txt
/tmp $

```

看看上面的命令 `cd /ssh:alice@box:/tmp/` 是如何无缝透明地将 shell 的当前目录设置为远程目录的。在此之后，如果我们创建一个文件，它会被创建在远程目录中！我们也可以在使用多个 TRAMP 方法打开的目录间工作。例如，首先考虑这样一个会话，当前本地用户没有权限写入本地 `/etc/` 目录：

```
~ $ `cp /ssh:alice@box:/etc/wgetrc /etc/`
Opening output file Permission denied /etc/wgetrc

```

但是，如果当前用户具有 `sudo` 特权，我们可以这样做：

```
~ $ `cp /ssh:alice@box:/etc/wgetrc /sudo::/etc/`
~ $ `ls /etc/wgetrc`
/etc/wgetrc

```

我们使用`sudo`特权从远程系统复制文件，并将其写入本地系统上受保护的目录。我们使用`ssh`方法读取远程文件，并使用`sudo`方法将文件写入受保护的本地目录。TRAMP确实名副其实：*透明远程访问，多协议*！

## 感谢

今年来主持这些Emacs图书俱乐部会议真是一大乐事。我非常享受讨论书中的内容，仔细研究每个新概念的引入，并进行演示来说明这些概念。特别感谢Libera和Matrix网络上的Emacs社区对这些会议的关注，参与讨论，并帮助这些会议取得成功！

</main>
