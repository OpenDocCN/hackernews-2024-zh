<!--yml

category: 未分类

date: 2024年5月27日12:50:48

-->

# research!rsc: xz 开源攻击时间表

> 来源：[https://research.swtch.com/xz-timeline](https://research.swtch.com/xz-timeline)

# xz 开源攻击时间表

发表于2024年4月1日星期一。

更新于2024年4月3日星期三。

在超过两年的时间里，一个使用“贾坦”这个名字的攻击者作为 xz 压缩库的勤奋有效的贡献者工作，最终被授予提交访问权限和维护权限。利用这一访问权限，他在 liblzma 中安装了一个非常微妙、精心隐藏的后门，这是 xz 的一部分，也是 Debian、Ubuntu 和 Fedora 上的 OpenSSH sshd 的依赖项，以及其他使用 systemd 的 Linux 系统，这些系统已经对 sshd 进行了补丁以链接 libsystemd。（请注意，这不包括像 Arch Linux、Gentoo 和 NixOS 这样的系统，这些系统没有对 sshd 进行补丁。）该后门在 SSH 会话开始时监视攻击者发送的隐藏命令，使攻击者能够在目标系统上以非认证、有针对性的远程代码执行方式运行任意命令。

该攻击于2024年3月29日[公开披露](https://www.openwall.com/lists/oss-security/2024/03/29/4)，似乎是广泛使用的开源软件首次严重已知的供应链攻击。这标志着开源供应链安全的一个分水岭时刻，无论好坏。

本文详细记录了社会工程方面的攻击时间线，看起来可以追溯到2021年末。（另见我对[攻击脚本分析](xz-script)的分析。）

欢迎在[Bluesky](https://bsky.app/profile/swtch.com/post/3kp4my7wdom2q)，[Mastodon](https://hachyderm.io/@rsc/112199506755478946)，或[电子邮件](mailto:rsc@swtch.com)上进行更正或补充。[](#prologue)

## [序言](#prologue)

**2005–2008**：[拉斯·科林（Lasse Collin），在其他人的帮助下](https://github.com/kobolabs/liblzma/blob/87b7682ce4b1c849504e2b3641cebaad62aaef87/doc/history.txt)，设计了使用LZMA压缩算法的.xz文件格式，该算法将文件压缩至大约gzip的70%[1]。随着时间的推移，该格式广泛用于压缩tar文件、Linux内核映像和许多其他用途。[](#jia_tan_arrives_on_scene_with_supporting_cast)

## [贾坦到场，有支持角色](#jia_tan_arrives_on_scene_with_supporting_cast)

**2021-10-29**：贾坦（Jia Tan）向 xz-devel 邮件列表发送[第一个无害补丁](https://www.mail-archive.com/xz-devel@tukaani.org/msg00512.html)，添加“.editorconfig”文件。

**2021-11-29**：贾坦（Jia Tan）向 xz-devel 邮件列表发送[第二个无害补丁](https://www.mail-archive.com/xz-devel@tukaani.org/msg00519.html)，修复了显然的可重现构建问题。随后的更多补丁（即使回顾起来也如此）似乎都是可以接受的。

**2022-02-07**: 拉瑟·科林合并了[第一个提交](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=6468f7e41a8e9c611e4ba8d34e2175c5dacdbeb4)，其中 git 元数据中的作者为“jiat0218@gmail.com”（“liblzma: Add NULL checks to LZMA and LZMA2 properties encoders”）。

**2022-04-19**: 贾坦向 xz-devel 邮件列表发送了[又一个无关紧要的补丁](https://www.mail-archive.com/xz-devel@tukaani.org/msg00553.html)。

**2022-04-22**: “吉加尔·库马尔”发送了[第一封抱怨邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00557.html)，称贾坦的补丁还未被采纳。（“补丁在这个邮件列表上花了几年时间。没有理由认为很快会有什么结果。”）此时，拉瑟·科林已经合并了贾坦的四个补丁，在提交消息中标注“感谢贾坦”。

**2022-05-19**: “丹尼斯·恩斯”向 xz-devel 发送了[邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00562.html)，询问 XZ for Java 是否得到了维护。

**2022-05-19**: 拉瑟·科林[回复了](https://www.mail-archive.com/xz-devel@tukaani.org/msg00563.html)，并对迟缓表示歉意，并补充道：“贾坦已经在 XZ Utils 上线下帮助了我，他可能在未来在 XZ Utils 中扮演更大的角色。显然，我的资源太有限了（因此有很多等待回复的邮件），所以长期来看必须有所改变。”

**2022-05-27**: 吉加尔·库马尔向补丁线程发送了[催促的邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00565.html)。“一个多月过去了，距离合并还远着呢。一点都不奇怪。”

**2022-06-07**: 吉加尔·库马尔向 Java 线程发送了[催促的电子邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00566.html)。“在没有新维护者之前，进展不会发生。C 语言的 XZ 也是如此。丹尼斯，你最好等到有新的维护者或者自己分叉。目前在这里提交补丁已经没有意义了。当前的维护者已经失去了兴趣或者不再关心维护。看到这样一个仓库真是令人难过。”

**2022-06-08**: 拉瑟·科林[提出了反对意见](https://www.mail-archive.com/xz-devel@tukaani.org/msg00567.html)。“我并没有失去兴趣，但是由于长期的心理健康问题以及其他一些事情，我的关心能力相当有限。最近，我在 XZ Utils 上线下与贾坦合作了一些，也许他将来会扮演更大的角色，我们将见分晓。还要记住，这只是一个无偿的业余项目。”

**2022-06-10**: 拉瑟·科林合并了[第一个提交](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=aa75c5563a760aea3aa23d997d519e702e82726b)，其中 git 元数据中的作者为“贾坦”（“Tests: Created tests for hardware functions”）。还要注意，早在 2022-02-07，有一个提交，作者全名仅设置为 jiat75。

**2022-06-14**: Lasse Collin合并了[唯一的提交，作者为“jiat75@gmail.com”](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=0354d6cce3ff98ea6f927107baf216253f6ce2bb)。这可能是Jia Tan在git上的暂时性配置错误，忘记了他们的虚假电子邮件地址。

**2022-06-14**: Jugar Kumar发送[压力邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00568.html)。“以你目前的速度，我非常怀疑今年能否看到5.4.0版本的发布。自四月以来，唯一的进展是对测试代码的小改动。你忽略了许多在邮件列表上搁浅的补丁。现在你的存储库已经堵塞了。为什么要等到5.4.0才改变维护者？为什么延迟你的存储库需要的东西？”

**2022-06-21**: Dennis Ens发送[压力邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00569.html)。"对于你的心理健康问题我感到抱歉，但重要的是要意识到自己的极限。我理解这对所有贡献者来说都是一个爱好项目，但社区希望更多。为什么不把XZ for C的维护权交给别人，这样你就可以更多地关注XZ for Java？或者把XZ for Java交给其他人，专注于XZ for C？试图同时维护两者意味着两者都无法得到很好的维护。"

**2022-06-22**: Jigar Kumar向C补丁线程发送[压力邮件](https://www.mail-archive.com/xz-devel@tukaani.org/msg00570.html)。“这个事情有进展吗？Jia，我看到你最近有提交。为什么你不能自己提交这个呢？”

**2022-06-29**: Lasse Collin [回复道](https://www.mail-archive.com/xz-devel@tukaani.org/msg00571.html)：“正如我在之前的邮件中暗示的，Jia Tan可能在未来在项目中扮演更重要的角色。他在列表外已经帮了很多忙，实际上已经是共同维护者了。:-) 我知道在git仓库中实际上没有发生太多变化，但事情正在小步骤中发生。无论如何，至少对于XZ Utils来说，维护者的一些变化已经在进行中。”[](#jia_tan_becomes_maintainer)

## [Jia Tan成为维护者](#jia_tan_becomes_maintainer)

在这一点上，Lasse似乎已经开始与Jia Tan更密切地合作。Brian Krebs [观察到](https://infosec.exchange/@briankrebs/112197305365490518)，这些电子邮件地址在互联网上从未出现过，甚至没有出现在数据泄露中（也没有再次出现在xz-devel中）。看起来它们很可能是假的，用来推动Lasse给予Jia更多的控制权。这起作用了。在接下来的几个月里，Jia开始在xz-devel中权威地回复关于即将到来的5.4.0版本的讨论。

**2022-09-27**: Jia Tan为5.4.0版发布做了[摘要](https://www.mail-archive.com/xz-devel@tukaani.org/msg00593.html)。（“计划于12月发布包含多线程解码器的5.4.0版本。我正在追踪与5..4.0版本相关的一般性开放问题列表...”）

**2022-10-28**: Jia Tan被添加到了[Tukaani组织](https://github.com/JiaT75?tab=overview&from=2022-10-01&to=2022-10-31)中。成为组织成员并不意味着特殊访问权限，但这是授予维护者权限之前的必要步骤。

**2022-11-30**: Lasse Collin将错误报告电子邮件从他的个人地址更改为一个别名，该别名同时发送给他和Jia Tan，并在README中指出“项目维护者Lasse Collin和Jia Tan可通过[xz@tukaani.org](mailto:xz@tukaani.org)联系”。

**2022-12-30**: Jia Tan将[一批提交直接合并到xz仓库](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=8ace358d65059152d9a1f43f4770170d29d35754)（“CMake: Update .gitignore for CMake artifacts from in source build”）。此时我们知道他们拥有提交权限。有趣的是，在同一批次的几个提交后，有一个唯一的全名不同的提交：“Jia Cheong Tan”。

**2023-01-11**: Lasse Collin [标签并构建了最终版本](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=18b845e69752c975dfeda418ec00eda22605c2ee)，v5.4.1。

**2023-03-18**: Jia Tan [标签并构建了他们的第一个版本](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=6ca8046ecbc7a1c81ee08f544bfd1414819fb2e8)，v5.4.2。

**2023-03-20**: Jia Tan [更新了Google oss-fuzz配置](https://github.com/google/oss-fuzz/commit/6403e93344476972e908ce17e8244f5c2b957dfd)，以便向其发送漏洞报告。

**2023-06-22**: Hans Jansen发送了[一对](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=23b5c36fb71904bfbe16bb20f976da38dadf6c3b)[补丁](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=b72d21202402a603db6d512fb9271cfa83249639)，由Lasse Collin合并，利用“[GNU间接函数](https://maskray.me/blog/2021-01-18-gnu-indirect-function)”特性在启动时选择快速CRC函数。最终提交由[Lasse Collin重新调整](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=ee44863ae88e377a5df10db007ba9bfadde3d314)，并由Jia Tan合并。这一变更很重要，因为它提供了一个钩子，通过这个钩子，后门代码可以在全局函数表在重新映射为只读之前修改它们。虽然这一变更本身可能是无辜的性能优化，但Hans Jansen在2024年回来推广植入后门的xz，否则在互联网上不存在。

**2023-07-07**: Jia Tan [在oss-fuzz构建中禁用了ifunc支持](https://github.com/google/oss-fuzz/commit/d2e42b2e489eac6fe6268e381b7db151f4c892c5)，声称ifunc与地址检查器不兼容。这本身可能是无害的，但也是后续使用ifunc的更多准备工作。

**2024-01-19**: 贾坦 [将网站迁移到 GitHub 页面](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=c26812c5b2c8a2a47f43214afe6b0b840c73e4f5)，从而控制了 XZ Utils 网页。拉斯·科林（Lasse Collin）可能创建了指向 GitHub 页面的 xz.tukaani.org 子域名的 DNS 记录。在发现攻击后，拉斯·科林删除了此 DNS 记录，以恢复对他控制的 [tukaani.org](https://tukaani.org) 的控制。

## [攻击开始](#attack_begins)

**2024-02-23**: 贾坦 [合并了隐藏后门的二进制代码](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=cf44e4b7f5dfdbf8c78aef377c10f71e274f63c0)，这些代码被巧妙地隐藏在一些二进制测试输入文件中。README 文件早在贾坦出现很久之前就写道：“该目录包含一堆用于测试解码器实现中 .xz、.lzma（LZMA_Alone）和 .lz（lzip）文件处理的文件。许多文件是通过十六进制编辑器手动创建的，因此这些文件本身就是最好的‘源代码’。”这种类型的测试文件对于这种库来说非常常见。贾坦利用这一点添加了一些不会被仔细审查的文件。

**2024-02-24**: 贾坦 [标记并构建了 v5.6.0](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=2d7d862e3ffa8cec4fd3fdffcd84e984a17aa429)，并发布了带有额外恶意 build-to-host.m4 的 xz-5.6.0.tar.gz 发行版，当构建 deb/rpm 包时会添加后门。这个 m4 文件不在源代码仓库中，但在打包过程中添加了许多其他合法的文件，因此本身并不可疑。但是脚本已被修改，从通常的复制修改为添加后门。查看我的[xz攻击shell脚本详细过程文章](xz-script)了解更多。

**2024-02-24**: Gentoo [开始在 5.6.0 版本中遇到崩溃问题](https://bugs.gentoo.org/925415)。这似乎是一个真实的 ifunc bug，而不是隐藏后门的 bug，因为这是 Hans Jansen 的 ifunc 更改的第一个 xz 版本，而 Gentoo 没有打补丁来使用 libsystemd，因此没有后门。

**2024-02-26**: Debian [将 xz-utils 5.6.0-0.1 添加到不稳定版本](https://tracker.debian.org/news/1506761/accepted-xz-utils-560-01-source-into-unstable/)。

**2024-02-27**: 贾坦开始通过电子邮件更新 Fedora 40 给**理查德·W·M·琼斯**（Rich Jones）（由 Rich Jones 私下确认）。

**2024-02-28**: Debian [将 xz-utils 5.6.0-0.2 添加到不稳定版本](https://tracker.debian.org/news/1507917/accepted-xz-utils-560-02-source-into-unstable/)。

**2024-02-28**: Jia Tan 在配置脚本中 [通过在检查 landlock 支持的 C 程序中添加微妙的拼写错误](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=a100f9111c8cc7f5b5f0e4a5e8af3de7161c7975) 来破坏 landlock 检测。配置脚本尝试构建和运行 C 程序来检查 landlock 支持，但由于 C 程序有语法错误，永远无法构建和运行，因此脚本始终会决定不存在 landlock 支持。Lasse Collin 被列为提交者；他可能忽略了微妙的拼写错误，或者作者可能伪造了提交者。很可能是前者，因为 Jia Tan 没有在他的其他许多更改中伪造提交者。这个补丁似乎不仅仅是为了 sshd 的变更，因为 landlock 支持是 xz 命令的一部分，而不是 liblzma。具体意图尚不清楚。

**2024-02-29**: 在 GitHub 上，@teknoraver [发送了拉取请求](https://github.com/systemd/systemd/pull/31550)，停止将 liblzma 链接到 libsystemd 中。看起来这本来可能会阻止攻击。[Kevin Beaumont 推测](https://doublepulsar.com/inside-the-failed-attempt-to-backdoor-ssh-globally-that-got-caught-by-chance-bbfe628fafdd)，知道这件事即将发生可能加速了攻击者的时间表。@teknoraver 在 HN 上 [评论道](https://news.ycombinator.com/item?id=39916125)，liblzma PR 是 libsystemd 依赖精简系列更改中的一部分；在一月底有 [两次提到](https://github.com/systemd/systemd/pull/31131#issuecomment-1917693005)。

**2024-03-04**: RedHat 发行版开始看到 Valgrind 在 liblzma 的 `_get_cpuid`（后门的入口）中报错。这场赛跑是为了在 Linux 发行版深入挖掘之前修复这个问题。

**2024-03-05**: [libsystemd PR 被合并](https://github.com/systemd/systemd/commit/3fc72d54132151c131301fc7954e0b44cdd3c860)，移除 liblzma。另一场赛跑开始，为了在发行版完全破坏这种方法之前，尽快设置 liblzma 的后门。

**2024-03-05**: Debian 将 [xz-utils 5.6.0-0.2 添加到测试版](https://tracker.debian.org/news/1509743/xz-utils-560-02-migrated-to-testing/)。

**2024-03-05**: Jia Tan 提交了 [两个 ifunc 修复](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=ed957d39426695e948b06de0ed952a2fbbe84bd1)。这些看起来是针对实际 ifunc bug 的真正修复。一个提交链接到 Gentoo 的 bug，并且还打了一个 [上游 GCC 的 bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=114115) 的错字。

**2024-03-08**: Jia Tan [提交了所谓的 Valgrind 修复](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=82ecc538193b380a21622aea02b0ba078e7ade92)。这是一个误导，但却是有效的。

**2024-03-09**: Jia Tan [提交了更新的后门文件](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=74b138d2a6529f2c07729d7c77b1725a8e8b16f1)。这是实际的Valgrind修复，更改了包含攻击代码的两个测试文件。“原始文件是在本地机器上使用随机数生成的。为了今后更好地重现这些文件，使用了一个恒定的种子重新创建这些文件。”

**2024-03-09**: Jia Tan [打了标签并构建了v5.6.1版本](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=fd1b975b7851e081ed6e5cf63df946cd5cbdbb94)，发布了包含新后门的xz 5.6.1分发版。到目前为止，我还没有看到关于新旧后门有何不同的分析。

**2024-03-20**: Lasse Collin向LKML发送了一个补丁集，用Jia Tan一起作为内核xz压缩代码的维护者，[链接1](https://lkml.org/lkml/2024/3/20/1009)，[链接2](https://lkml.org/lkml/2024/3/20/1008)。在这里没有迹象表明Lasse Collin有恶意行为，只是清理了他作为唯一维护者的引用。当然，Jia Tan可能促使了这一举动，并且能够向Linux内核发送xz补丁将是Jia Tan未来工作的一个很好的支持点。虽然我们还没有达到[信任信任](nih)的水平，但这是更近一步。

**2024-03-25**: Hans Jansen回来了，[向Debian提交了一个bug报告](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1067708)，要求将xz-utils更新到5.6.1版本。就像在2022年的压力活动中一样，更多在互联网上并不存在的name###@mailhost地址出现以支持这一请求。

**2024-03-27**: Debian更新到了5.6.1版本。

**2024-03-28**: Jia Tan提交了一个Ubuntu漏洞报告，要求从Debian更新xz-utils到5.6.1版本，[报告链接](https://bugs.launchpad.net/ubuntu/+source/xz-utils/+bug/2059417)。

## [攻击检测到](#attack_detected)

**2024-03-28**: Andres Freund发现漏洞，私下通知了Debian和distros@openwall。RedHat分配了CVE-2024-3094编号。

**2024-03-28**: Debian回滚到5.6.1+really5.4.5-1版本，[Debian新闻链接](https://tracker.debian.org/news/1515519/accepted-xz-utils-561really545-1-source-into-unstable/)。

**2024-03-28**: Arch Linux [改变了从Git构建的5.6.1版本](https://gitlab.archlinux.org/archlinux/packaging/packages/xz/-/commit/881385757abdc39d3cfea1c3e34ec09f637424ad)。

**2024-03-29**: Andres Freund向公共oss-security@openwall邮件列表发布[后门警告](https://www.openwall.com/lists/oss-security/2024/03/29/4)，称他在“过去几周”发现了它。

**2024-03-29**: RedHat在Fedora Rawhide和Fedora Linux 40 beta中宣布xz存在后门，[官方公告链接](https://www.redhat.com/en/blog/urgent-security-alert-fedora-41-and-rawhide-users)。

**2024-03-30**: Debian关闭了构建过程，重新使用Debian稳定版重建他们的构建机器（以防恶意软件xz逃离他们的沙箱？），[社交媒体公告链接](https://fulda.social/@Ganneff/112184975950858403)。

**2024-03-30**：Haiku OS [迁移到 GitHub 源代码库快照](https://github.com/haikuports/haikuports/commit/3644a3db2a0ad46971aa433c105e2cce9d141b46)。[](#further_reading)

## [进一步阅读](#further_reading)

+   Evan Boehs，[关于 XZ 后门的一切我所知道的](https://boehs.org/node/everything-i-know-about-the-xz-backdoor)（2024-03-29）。

+   Filippo Valsorda，[Bluesky](https://bsky.app/profile/filippo.abyssdomain.expert/post/3kowjkx2njy2b) 关于后门操作的内容（2024-03-30）。

+   Michał Zalewski，[技术人员 vs 间谍：关于 XZ 后门的辩论](https://lcamtuf.substack.com/p/technologist-vs-spy-the-xz-backdoor)（2024-03-30）。

+   Michał Zalewski，[OSS 后门：易修复的错误](https://lcamtuf.substack.com/p/oss-backdoors-the-allure-of-the-easy)（2024-03-31）。

+   Connor Tumbleson，[远距离观察 XZ 的发展](https://connortumbleson.com/2024/03/31/watching-xz-unfold-from-afar/)（2024-03-31）。

+   nugxperience，[Twitter](https://twitter.com/nugxperience/status/1773906926503591970) 关于 awk 和 rc4 的讨论（2024-03-29）

+   birchb0y，[Twitter](https://twitter.com/birchb0y/status/1773871381890924872) 提到提交的时间与邪恶程度的关系（2024-03-29）

+   Dan Feidt，[揭露多年黑客阴谋的 'xz utils' 软件后门](https://unicornriot.ninja/2024/xz-utils-software-backdoor-uncovered-in-years-long-hacking-plot/)（2024-03-30）

+   smx-smz，[[进行中] XZ 后门分析和符号映射](https://gist.github.com/smx-smx/a6112d54777845d389bd7126d6e9f504)

+   Dan Goodin，[我们了解的几乎感染全球的 xz Utils 后门](https://arstechnica.com/security/2024/04/what-we-know-about-the-xz-utils-backdoor-that-almost-infected-the-world/)（2024-04-01）

+   Akamai 安全情报组，[XZ Utils 后门 — 你需要知道的一切以及你可以做的事情](https://www.akamai.com/blog/security-research/critical-linux-backdoor-xz-utils-discovered-what-to-know)（2024-04-01）

+   Kevin Beaumont，[全球尝试对 SSH 进行后门的失败尝试内幕 — 偶然被捕获](https://doublepulsar.com/inside-the-failed-attempt-to-backdoor-ssh-globally-that-got-caught-by-chance-bbfe628fafdd)（2024-03-31）

+   amlweems，[xzbot：关于 XZ 后门的笔记、蜜罐和漏洞演示](https://github.com/amlweems/xzbot)（2024-04-01）

+   Rhea Karty 和 Simon Henniger，[XZ 后门：时间、诅咒的时间和欺骗](https://rheaeve.substack.com/p/xz-backdoor-times-damned-times-and)（2024-03-30）

+   Andy Greenberg 和 Matt Burgess，[“贾谈”之谜：XZ 后门的幕后主谋](https://www.wired.com/story/jia-tan-xz-backdoor/)（2024-04-03）

+   [Risky Business #743 -- 与发现 XZ 后门的人的讨论](https://risky.biz/RB743/)（2024-04-03）
