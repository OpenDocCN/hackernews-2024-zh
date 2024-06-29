<!--yml

类别：未分类

日期：2024年5月27日 15:01:07

-->

# Linux桌面终于很棒（包括Ubuntu和Firefox）

> 来源：[https://punkx.org/jackdoe/linux-desktop.html](https://punkx.org/jackdoe/linux-desktop.html)

# TLDR：Ubuntu和Firefox都非常出色，试试它们吧。

*Ubuntu 23.10，Firefox 118.0.1*

*2024年3月16日 下午1:25:58 CET*

*作者 [jackdoe](https://punkx.org/jackdoe)*

自从Chrome发布以来，我几乎只用它；它滚动更顺畅，之后加入了翻译功能，并且比Firefox更省CPU。

我从Slack 3.6开始使用Linux，期间使用过FreeBSD，在2.2.8和8.0之间（包括几个月的NetBSD/OpenBSD/DragonFly/OpenSolaris），但之后我主要在我的笔记本上使用macOS。最近的3-4年里，我在PC上安装了Windows，主要是为了玩魔兽世界（有时候通过wine能正常运行，但有些补丁会破坏，我厌倦了每次更新都要修复的问题）。

我从未设法让我的Windows PC感觉像家一样；我总是把我的Windows当作一个“黑客”公共计算机，并且由于没有全盘加密，我从不在上面放置任何敏感信息，因此也从未在上面工作过。几天前我的SSD坏了，我在新的SSD上安装了Ubuntu 23.10，全盘加密，并且完美地设置了Emacs。

### 以下是完美运作的事物（没有特定顺序）：

+   安装本身

+   多显示器，其中一个旋转了

+   Emacs，这一直都是如此，只是声明没有退步

+   Firefox - 现在有翻译功能了

+   显示应用程序使用麦克风的情况（包括arecord(1)和当然是Discord）

+   Discord，Slack

+   通知

+   Espressif工具链

+   RISC-V工具链（我在制作新桌游[PROJEKT:OVERFLOW](https://punkx.org/overflow)时使用它，在Windows上使用起来非常烦人）

+   Pulseview（甚至比macOS更好，比Windows好得多）

+   usbpcap + Wireshark

+   屏幕录制

+   llama.cpp with Alpaca

+   Ubuntu的默认终端

+   Inkscape（cli 和 gui）

+   当你从Gnome Files复制文件并在终端中粘贴时，它实际上粘贴了其绝对路径

Firefox给我最大的惊喜；它如此流畅，现在还有翻译功能，这对我来说是一个巨大的障碍，因为我在荷兰，不会说荷兰语，所以我不得不不断翻译东西，因为大多数网站只根据IP地址而不是Accept-Language标头推测语言。

### 不那么出色的事物：

+   Lutris - 这是winehacks + wine安装最新的Battlenet.exe存在一些问题，所以我不得不搜索如何修复它，但之后，WoW在3080上的帧率达到了200-300，没有输入延迟，所有的weakauras和插件都可以正常工作，尽管文字转语音很差（我用它来告诉我一些重要的冷却时间准备好了）。

+   文字转语音 - 听起来像个坏掉的机器人

+   ~~安装字体仍然需要 *cp z.ttf /usr/share/fonts/truetype && sudo fc-cache -fv*~~ [[popey@lobsters](https://lobste.rs/s/qurpee/linux_desktop_is_finally_great_both#c_jqzcl4) 告诉我，如果你双击字体，实际上有一个安装按钮，我只是没看到它]

我没有改过任何设置，也没有从 superuser.com 复制过任何一行。过去，我主要使用 EXWM、i3 和 awesome，但现在我只使用 Ubuntu 默认的窗口管理器；我甚至不知道它的名字。我也没有重新编译内核。

我甚至不知道它是否在使用 Wayland，我也不在乎 :)

我觉得多年来，苹果对待用户的态度越来越像对待小孩子，比如，当你编译一个程序并且第一次运行时，它启动很慢，因为“某些东西”必须决定是否“允许”或应该隔离，或者你不能 *gdb -p* 进程。每次它这样做，我就感到非常愤怒。自从 M1 推出以来，我可以说关于硬件的好话，我的 Mac 经常运行 100-200 天，电池续航 10+ 小时... 我只是无法忍受这种傲慢的态度。

电脑能不能按照我写的代码运行一下？问得太多了吗？

我曾以为 Linux 桌面远远落后于 macOS，结果证明我错了。现在我迫不及待地想回到我的 Linux PC，只想打开我的 emacs。它感觉像家一样。

注：我把这个故事告诉了我的朋友，他说 Fedora 更好。

* * *

*echo 'int main(void) { return 0; }' > aa.c && gcc -o aa aa.c && time ./aa && time ./aa*

**macOS**:

./aa 0.00s user 0.00s system 1% cpu 0.281 total

./aa 0.00s user 0.00s system 54% cpu 0.001 total

**Linux**:

real 0m0.001s

user 0m0.001s

sys  0m0.000s

real 0m0.001s

user 0m0.001s

sys  0m0.000s

看起来现在它只是在取笑我。
