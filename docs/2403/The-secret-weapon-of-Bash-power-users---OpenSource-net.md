<!--yml

分类: 未分类

date: 2024-05-29 12:40:28

-->

# Bash 高级用户的秘密武器 - OpenSource.net

> 来源：[https://opensource.net/the-secret-weapon-of-bash-power-users/](https://opensource.net/the-secret-weapon-of-bash-power-users/)

我是那些喜欢从命令行开始每个项目的 Linux 恐龙之一。好吧，也许并不是每个项目，但是很多项目。

我也是偏爱[vi](https://en.wikipedia.org/wiki/Vi_(text_editor))或**vim**，而不是**emacs**作为我的文本编辑器的社区成员之一。

Vi，在 Bash shell 中更确切地说是 vi 模式，因为几个原因成为了 bash 高级用户的秘密武器。Vi 的按键设计用于在文本文件中进行快速导航和编辑，将这些按键应用于 Bash 命令历史让您能够快速浏览过去的命令，节省时间和按键。

很长一段时间以来，我通过在[bash](https://opensource.net/programming-with-bash-part-3-loops/) shell 中使用 vi 模式来结合这两种偏好。这意味着使用 vi 的按键，比如`k`来在命令历史中向上移动一行，`j`来向下移动一行。使用`0`（就是零）来移到命令行的开头，`$`来移到末尾；或者`cw`来更改从当前字符位置开始的单词，等等。

基本上，bash shell 中的 vi 模式将命令历史视为正在 vi 或 vim 中编辑的文本文件，而 vi 和 vim 的按键序列则在当前光标位置的历史行上操作。听起来有些笨重，但实际上非常棒。对于习惯于使用 vi 的人来说，大约10秒钟就能意识到它有多酷，之后…

在 bash shell 中，默认情况下用户处于“插入”模式，输入命令并按回车执行。要退出“插入”模式，用户按下`Esc`键进入“命令”模式，其中按键被解释为 vi 命令。编辑命令行完成后，用户再次按下`Esc`键退出“命令”模式并进入“插入”模式。

让 vi 模式与 bash 协同工作的秘诀是在主目录下创建一个名为 .inputrc 的文件，并在其中添加以下行：

`set editing-mode vi`

`set keymap vi-insert`

然后启动一个新的 shell，这些设置将会生效。

就是这样！对于那些对这个话题更感兴趣的人，请查看[stackoverflow 上的这篇信息性帖子](https://stackoverflow.com/questions/46161918/bash-vim-mode-instead-of-vi-mode)，并查看我们的一系列[Bash 教程](https://opensource.net/?s=bash)。
