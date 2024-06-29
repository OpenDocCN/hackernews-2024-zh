<!--yml

category: 未分类

date: 2024-05-27 14:54:18

-->

# 如何退出 vi

> 来源：[https://liw.fi/vi/](https://liw.fi/vi/)

对于一些莫名其妙的原因，人们经常在退出`vi`这个古老而强大的UNIX编辑器时遇到困难。考虑到`vi`的用户界面是逻辑的，因此很容易学习，这本手册应该是不必要的。然而，世界并非如此。

22 September 2002.

如果你想退出并保存缓冲区中的内容，命令序列是：

> Control-Q Control-C ESC Z Z

如果你想退出，但不想保存缓冲区中的内容，命令序列是：

> Control-Q Control-C ESC : q ! ENTER

欢迎将忘了`vi`的用户引导到这个页面。希望这能让他们转向这个编辑器的优雅和简单。
