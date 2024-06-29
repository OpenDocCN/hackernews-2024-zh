<!--yml

类别：未分类

日期：2024-05-29 12:29:30

-->

# ELIZAGEN - 新闻

> 来源：[https://sites.google.com/view/elizagen-org/news#h.ykbzq5nuiccs](https://sites.google.com/view/elizagen-org/news#h.ykbzq5nuiccs)

## 20190121：Eliza 在 PDP-10 模拟器上的 DEC 10（Lars Brinkhoff）

几天前，我有幸搭乘一台真正的时间机器。

大约一周前，我突然收到 [Lars Brinkhoff](https://github.com/larsbrinkhoff) 通过 [ElizaGen 问题跟踪器](https://github.com/jeffshrager/elizagen) 的消息："我有一个移植到 Maclisp 的 ELIZA 版本，日期为 1977 年 9 月 5 日。不幸的是，它只是一个编译的二进制文件。这对你有兴趣吗？"

只是一个二进制文件可能不是太有趣，所以我的最初反应有所保留：“有没有办法运行它或提取嵌入的代码？”

原来 Lars 显然低估了这种情况：“这个二进制文件是一个编译的 FASL，快速加载文件，所以除了 PDP-10 指令之外，提取任何代码并不容易。可能可以做到，但是这需要大量工作，以重建大致相同的 Lisp 代码。”

但是，继续说道："...我们可以在 PDP-10 模拟器上运行 Maclisp。我有一个在线模拟器。"

Eliza 在 MacLisp 上运行在 PDP-10 模拟器上？这里难道不是缺少了一个层次吗？在什么操作系统上？

这个答案让我停下了：[MIT's ITS](https://en.wikipedia.org/wiki/Incompatible_Timesharing_System)。ITS 是计算机科学历史上最重要的操作系统之一。（除了许多其他有趣的历史事实，原始 EMACS 就是在 ITS 上开发的！）我建议在这里阅读更多：[Wikipedia 上的 ITS](https://en.wikipedia.org/wiki/Incompatible_Timesharing_System)。在我本科和研究生阶段，即 70 年代末和 80 年代初，我们通过早期 ARPANet 远程使用 MIT-AI 的 ITS。有趣的是，在某个时候（我不知道持续多久），MIT-AI 的 ITS 在其命令行驱动程序中已经被 Eliza 黑客入侵，所以除非你的命令前面有 "："，否则响应来自 Eliza！

Eliza 在 MacLisp 上运行在 ITS 上的 Dec 10 模拟器？真的吗！

是的，真的！

这里是模拟器：[https://github.com/PDP-10/its](https://github.com/PDP-10/its)

（Lars 的注释：“...比我涉及的人更多。首先是埃里克·斯文森（Eric Swenson），他曾在当年的 Maclisp 和 Macsyma 上工作。我们[...]更新模拟器以支持更多硬件，修复错误，找出如何构建和运行程序，添加新软件等。”）

如果你想体验一段伟大的软件历史，你可以自己登录这个 ITS 模拟器；这是免费且开放的。无需密码！

来自 Lars：

Telnet 到 "its.pdp10.se, port 10003"。输入 Control-Z 登录。然后输入 ":lisp games; eliza (init)" 开始 Maclisp 并加载 ELIZA。应该看起来像这样：

==================

你：

telnet its.pdp10.se 10003

尝试 88.99.191.74... 连接到 pdp10.se。转义字符是 '^]'。

连接到 KA-10 模拟器 MTY 设备，第 4 行

你：

输入 Control-Z

TT ITS.1648\. DDT.1546.  TTY 25  2\. Lusers, Fair Share = 0%  THIS IS TT ITS, A HIGHTY [sic] VOLATILE SYSTEM FOR TESTING

要获取简要信息，请输入？要获取冒号命令列表，请输入：？并按 Enter 键。要获取完整信息系统，请输入：INFO 并按 Enter 键。

管理层对使用缓慢的 TK10 终端复用器表示歉意。我们将在 Datapoint kludge 可用时进行升级，速度快得多。

然后你输入：

:lisp games; eliza（初始化）

（请登录）QUIT SPEAK UP！

看吧，你正在通过 40 多年的计算机历史输入反馈！

（使用双击回车键结束你的话题到 Eliza。）

==================

为了测试它，我粘贴了 Wiezenbaum 的 Eliza 论文中的“标准”示例对话。以下是论文的截图：

现在它在 Lars 的重生 ITS 上实时运行（底部的混乱是因为我不小心粘贴了相同的句子两次，然后又回退了。重复的字符显然是 ITS 显示回退的方式）：

关于这个版本的确切来源有点神秘。Lars 和我推测，由于 BBN 就在人工智能实验室对面，而且众所周知 BBN 是 ITS 系统的用户，这很可能就是 Bernie Cosell 的原始代码！
