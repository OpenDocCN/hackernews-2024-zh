<!--yml

category: 未分类

date: 2024-05-27 15:04:50

-->

# SIGILL 错误的新代码有助于识别分支错误。

> 来源：[https://www.undeadly.org/cgi?action=article;sid=20240222183703](https://www.undeadly.org/cgi?action=article;sid=20240222183703)

由[Janne Johansson](http://www.inet6.se)于2024-02-22贡献，属于“不要对电围栏撒尿”部门。

如果在某些amd64或aarch64平台上运行最近版本的OpenBSD，对“意外”位置的间接分支将导致程序崩溃，以防止ROP攻击和类似方式使程序在不应执行代码的位置上执行代码。

在所有分支应该到达的地方，OpenBSD编译器将插入额外的指令，如果它们到达其他位置，则会引发CPU错误，并显示“非法指令”。

以前，这类崩溃看起来更像是执行随机数据或来自随机位置的任何其他类型的错误，但由于内核知道何时发生这种情况，我们可以明确指出，错误是由于缺少的分支目标指令造成的，这在调试时将大有帮助。

链接到提交内容[在此](https://marc.info/?l=openbsd-cvs&m=170853069811439&w=2)。
