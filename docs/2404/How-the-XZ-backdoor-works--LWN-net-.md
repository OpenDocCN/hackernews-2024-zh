<!--yml

category: 未分类

date: 2024-05-27 12:53:25

-->

# 如何XZ后门工作 [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/967192/6c39d47b5f299a23/](https://lwn.net/SubscriberLink/967192/6c39d47b5f299a23/)

| **LWN订阅者的好处** [订阅LWN](/subscribe/)的主要好处是帮助我们继续发布文章，此外，订阅者还可以立即访问所有站点内容和许多额外的站点功能。请今天就注册！ |
| --- |

By **Daroc Alden**

April 2, 2024

版本5.6.0和5.6.1的[XZ](https://git.tukaani.org/?p=xz.git;a=summary)压缩实用工具及库随[后门](/Articles/967180)一同发布，其目标为[OpenSSH](https://www.openssh.com/)。Andres Freund通过注意到[失败的SSH登录消耗了大量CPU时间](/Articles/967194/)进行微基准测试，并从那里追踪到了后门的存在。后门由XZ的共同维护者"Jia Tan"引入 —— 这很可能是一个未知个人或团体的别名。该后门是一个复杂的攻击，涉及构建系统、链接时间和运行时间多个部分。

社区对这次攻击的反应和技术细节一样引人关注。更多信息，请参考[此附属文章](/Articles/967866)。

#### 构建时间

后门由几个明显阶段组成，从软件包构建开始。Gynvael Coldwind 写了一篇关于后门构建时部分的[深入调查](https://gynvael.coldwind.pl/?lang=en&id=782)。XZ的发布通过GitHub提供，后来已禁用维护者的账户并下线了发布。与许多使用[GNU Autoconf](https://www.gnu.org/software/autoconf/)的项目一样，XZ发布了几个源代码版本供下载 —— 自动生成的tar压缩文件包含仓库中的源代码及相关文件，还包括生成的构建文件，如`configure`脚本和项目的makefile。发布包含生成文件的版本允许软件的下游用户无需安装Autoconf即可构建。

然而，在这种情况下，维护者提供的源代码tar压缩包中的脚本并非由Autoconf生成。相反，一个构建脚本包含了`m4/build-to-host.m4`中的攻击的第一阶段。该脚本最初来自[Gnulib](https://www.gnu.org/software/gnulib/)库；它提供了一个宏，用于转换构建环境和程序运行环境之间的路径名风格。这些XZ版本中的版本已修改，以提取下一个阶段的攻击，该攻击包含在`tests/files/bad-3-1corrupt_lzma2.xz`中。

该文件包含在代码库中，表面上是作为 XZ 的测试套件的一部分，尽管这些测试从未使用过该文件。它在版本 5.6.0 发布之前提交。这个文件据称是一个已损坏的 XZ 文件，实际上是一个有效的 XZ 流，其中一些字节被交换了位置 — 例如，`0x20` 与 `0x09` 的出现次序互换。解码后，它生成了一个[shell脚本](/Articles/967979)，用于解压并执行后门的下一个阶段。

后门的下一个阶段位于 `tests/files/good-large_compressed.lzma` 中。这是 Freund 消息中附带的 `injected.txt` 文件。该文件不仅包含脚本的下一个阶段，还包含形成实际后门本身的额外二进制数据。最终的脚本跳过了从中提取的文件的头部，然后使用 `awk` 解密文件的其余部分。最后，解密的流使用 XZ 命令行程序进行解压，以提取一个名为 `liblzma_la-crc64-fast.o` 的预编译文件，该文件也附在 Freund 的消息中。

#### 链接时间

提取出的文件是一个64位可重定位 ELF库。构建过程的其余部分将其链接到最终的 `liblzma` 库中，后者最终被一些发行版加载到 OpenSSH 中。这些发行版[修补](https://sources.debian.org/patches/openssh/1:9.2p1-2%2Bdeb12u2/systemd-readiness.patch/)了 OpenSSH 以使用 systemd 进行守护进程准备通知；而 `libsystemd` 反过来依赖于 liblzma 用于压缩日志文件。Lennart Poettering 后来[发布了](https://mastodon.social/@pid_eins/112202687764571433)一些示例代码（由 Luca Boccassi 编写），展示如何让应用程序在不拉取整个库的情况下使用 systemd 的准备通知。当恶意的 `liblzma` 被动态链接的进程使用时，它使用[间接函数](https://sourceware.org/glibc/wiki/GNU_IFUNC)机制介入[链接过程](/Articles/961117)。

GNU C 库（glibc）的间接函数是一个功能，允许开发人员包含多个版本的函数，并在动态链接时选择要使用的版本。间接函数对于包含依赖于特定硬件特性的优化版本的函数非常有用，例如。在这种情况下，后门提供了自己的版本的间接函数解析器 `crc32_resolve()` 和 `crc64_resolve()`，分别选择要使用的 `crc32()` 和 `crc64()` 的版本。`liblzma` 通常不使用间接函数，但使用更快的函数来计算校验和似乎是使用该功能的一个合理用途。这种合理性可能就是为什么漏洞本身存放在名为 `liblzma_la-crc64-fast.o` 的文件中。

当动态链接器最终确定这些函数的位置时，它会调用后门的解析器函数。此时，动态链接仍在进行中，因此许多链接器的内部数据结构尚未设为只读。这使得后门可以通过覆盖过程链接表（PLT）或全局偏移表（GOT）中的条目来操纵已加载的库。然而，`liblzma`在OpenSSH的链接顺序中相当早加载，这意味着后门的最终目标[OpenSSL](https://www.openssl.org/)的加密函数可能尚未加载。

为了应对这一情况，后门添加了一个[审计钩子](https://www.man7.org/linux/man-pages/man7/rtld-audit.7.html)。当动态链接器正在解析符号时，会调用所有注册的审计钩子。后门利用这一点等待[`RSA_public_decrypt@got.plt`](https://linux.die.net/man/3/rsa_public_decrypt)符号被解析。尽管其名称如此，但实际上这个函数是处理RSA签名（即解密操作）的一部分 — OpenSSH在连接过程中验证客户端提供的RSA证书时调用它。

#### 运行时

一旦后门检测到此函数被链接，它会替换成自己的版本。修改后的版本的功能仍在调查中，但至少其中一个功能是尝试从所提供的RSA证书的公钥字段中提取命令（这意味着在此攻击中使用的证书实际上不能正常用于认证）。后门检查命令是否由攻击者的私钥签名并具有有效的格式。如果是，后门会直接以运行`sshd`的用户身份（通常是root）运行给定的命令。

Anthony Weems已经组织了一个[解释](https://github.com/amlweems/xzbot)关于利用运行时部分的漏洞，包括一个蜜罐来检测使用漏洞的尝试，以及生成命令有效载荷的代码。使用后门涉及使用私钥对要执行的命令进行签名，但攻击者的私钥不可用，因此必须对安装了后门的服务器进行修补以使用另一个私钥。这也意味着远程检测到后门服务器几乎不可能，因为它们对不使用攻击者的私钥的连接没有任何反应。

总体而言，后门的影响似乎是，一个被入侵的SSH服务器，接收到一个经过特制的RSA证书进行身份验证的连接，可以被迫运行攻击者控制的代码。

#### 反分析

后门的设计使得它难以在不直接检查 `liblzma` 的情况下被注意到。例如，选择启用远程代码执行而不是认证绕过意味着利用漏洞不会检测到可以被传统管理工具注意到的登录会话。后门的代码还使用了几种技术来增加发现的难度。例如，用于审计挂钩的字符串"RSA_public_decrypt@got.plt"从未出现在漏洞二进制文件中。相反，它使用一个[trie](https://en.wikipedia.org/wiki/Trie)来保存各种字符串。Serge Bazanski发布了一个以这种方式编码的恶意 `liblzma` 字符串的[列表](https://gist.github.com/q3k/af3d93b6a1f399de28fe194add452d01)。

检查列表显示，`RSA_public_decrypt` 可能不是唯一被干扰的函数；还列出了其他几个密码学例程。它还显示了用于干扰 OpenSSH 记录的各种函数和字符串。目前尚未确认，但似乎一个被入侵的 SSH 服务器实际上不会记录使用漏洞的连接尝试。

后门还包括许多检查，以确保它在预期环境下运行 — 这是现代恶意软件的标准预防措施，旨在使反向工程变得更加困难。后门仅在特定情况下活动，包括：在非图形环境中运行，~~作为根用户~~（参见Freund的[此评论](/Articles/968045)），在位于`/usr/sbin/sshd`的二进制文件中，且 `sshd` 具有预期的ELF头部，并且没有其函数被调试器插入断点。尽管存在这些障碍，社区仍在努力进行反向工程并解释后门代码的其余部分，详见[gist.github.com/smx-smx/a6112d54777845d389bd7126d6e9f504](https://gist.github.com/smx-smx/a6112d54777845d389bd7126d6e9f504)。

~~后门还包括代码，用于修补 `sshd` 本身的二进制文件，禁用 [`seccomp()`](https://man7.org/linux/man-pages/man2/seccomp.2.html) 并阻止程序为其子进程创建[chroot沙箱](https://en.wikipedia.org/wiki/Chroot)~~（参见[此评论](/Articles/968317)）。总的来说，后门的代码有87KB，这足够容纳额外的不愉快惊喜。许多人已经总结了他们对漏洞的理解，包括[Sam James的这篇全面FAQ](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27)，其中链接到其他资源。

#### 保持安全

漏洞很快被发现，因此几乎没有用户受到影响。Debian sid、Fedora Rawhide、Fedora 40 beta、openSUSE Tumbleweed 和 Kali Linux 都短暂地发布了受损的软件包。NixOS 不稳定版也发布了受损的版本，但并不容易受到影响，因为它没有将 OpenSSH 补丁链接到 `libsystemd`。Tan 还对 XZ 代码进行了一些其他更改，使检测和缓解后门变得更加困难，比如[破坏](https://git.tukaani.org/?p=xz.git;a=blobdiff;f=CMakeLists.txt;h=d2b1af7ab0ab759b6805ced3dff2555e2a4b3f8e;hp=76700591059711e3a4da5b45cf58474dac4e12a7;hb=328c52da8a2bbb81307644efdb58db2c422d9ba7;hpb=eb8ad59e9bab32a8d655796afd39597ea6dcc64d)沙盒措施，并进行了预防性努力以重定向安全报告。尽管漏洞未能影响到稳定版本，但几个发行版仍在采取步骤，转向不含 Tan 的任何提交的 XZ 版本，因此用户应该预期很快会看到与此相关的安全更新。读者也许还希望参考其发行版的安全通告获取更具体的信息。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/967192/)

发表评论)
