<!--yml

category: 未分类

date: 2024-05-27 14:30:53

-->

# 无驱动用户空间文件系统适用于 Windows、macOS 和 Linux

> 来源：[https://thelig.ht/user-space-file-systems/](https://thelig.ht/user-space-file-systems/)

*TLDR：[userspacefs](https://pypi.org/project/userspacefs/) Python 软件包允许您轻松地为所有三个主要桌面平台编写用户空间文件系统。[dbxfs](https://pypi.org/project/dbxfs/) Python 软件包现在为 Windows 提供一个[可用的可执行文件](https://www.dropbox.com/scl/fo/5bjakuwingc6vk9kcgcy0/h?rlkey=bgab24afqq8rgb78qo9hwsum8&dl=0)。*

多年来，我一直被用户空间文件系统的想法所吸引。我第一次接触这个概念肯定是在 2002 年左右，当时我了解了 GNU Hurd。当然，能够在用户空间运行文件系统有许多好处，但对于我内心中正在萌芽的系统设计师来说，它最强大的吸引力是作为文件系统为基础的操作系统范式的自然、逻辑和优雅扩展。

GNU Hurd 只是冰山一角。GNU Hurd 在很大程度上仍然是 Unix，用户空间文件系统只是一个附加功能，而 Plan 9 则是围绕这个概念作为核心原语重新构建的 Unix。最终的结果是一个更不复杂的系统，整个堆栈都有统一的 API。将文件系统作为核心系统 API，我们可以进一步将网络协议、图形界面和其他硬件驱动程序实现为用户空间服务器，而不是一些自定义内核和用户空间组件的临时组合。今天我们将这称为[narrow waist](http://www.oilshell.org/blog/2022/02/diagrams.html)设计原则。

这个想法激发了一代程序员和系统设计师的想象力。自然地，这个想法传播开来，原生用户空间文件系统功能最终出现在 Linux 和 BSD 中，以 FUSE 的形式。这使得[数十个关键应用程序](https://en.wikipedia.org/wiki/Filesystem_in_Userspace#Applications)在引入 FUSE 之前根本不可行。

在开源 POSIX 系统之外，一直在努力将类似 FUSE 的 API 带到 Windows 和 macOS，但这些努力总是需要安装第三方内核驱动程序。对内核驱动程序的依赖使得部署用户空间文件系统变得更加复杂，通常几乎是不可行的。事实证明，不需要驱动程序。

## WebDAV

在2011年的某个时候，当我随意翻阅Mac OS X的man页面时，我注意到它有能力将WebDAV服务器挂载为本地文件系统。我早已知道这一点，但对我来说并不算什么有趣的事情，只是有一个命令行界面可以实现这一点。几天后，我恍然大悟，“你可以用这种方式实现用户空间文件系统！” 这个想法很简单，只需在本地主机上托管一个WebDAV服务器并运行一个命令来挂载它。这个想法是一个有趣的技巧，但我有一个更有趣的技巧的想法：编写一个模拟FUSE ABI但使用WebDAV作为后端透明挂载在Mac OS X上的库。我说ABI，是因为我不仅在源码级别上这样做，我会使用`DYLD_INSERT_LIBRARIES`欺骗预编译程序，让它们使用我的库。davfuse的工作于2012年7月17日开始。

接下来的一年里，我花了很多晚上和周末实现了这个想法。最终，它顺利运作。我欣喜若狂。davfuse于2013年9月11日发布，不幸的是，没有引起太多关注。我并没有花太多精力宣传这个库，因为davfuse只是用来证明概念的。在我开始着手davfuse的时候，我了解到Windows也原生支持挂载WebDAV文件系统。突然间，绝大多数桌面计算用户可以体验到用户空间文件系统，这使得这项技术非常强大。我的目标随之变成了利用这项技术在Windows上生产一款杀手级应用。

## Safe

在此期间，我是[EncFS](https://vgough.github.io/encfs/)的忠实用户。这是我在Dropbox上存储加密文件的便捷方式。因此，这个想法很简单，就是创建一个可以在Windows和Mac OS X上运行的用户友好版本的EncFS。工作于2013年7月26日开始，并且[于2014年4月14日发布](https://news.ycombinator.com/item?id=7588369)。

在构建项目时，你希望[专注于一项创新的事情](https://boringtechnology.club/)，这样就不会分心，可以按时发布产品。但在这种情况下，我专注于了错误的事情。由于各种原因，尽管它足够好用，EncFS并不适合在Windows上长期使用。对于交付这种类型的产品，我本应该专注于设计一个在Windows和Mac OS X上无缝运行的可信加密系统，而不是简单地用用户友好的方式包装EncFS。实际上，构建Safe并不像简单地用用户友好的方式包装EncFS那样简单。事实上，EncFS最终成为了我的一种束缚，而非我生产力的助力，因为我承诺兼容EncFS。这限制了我改进加密的能力。Safe实际上并没有任何用户，我也不想处理已经废弃的产品，所以我就关闭了它。

尽管davfuse和Safe的代码已经变得无用，但在构建它们的过程中，我学到了很多关于软件工程和产品开发的知识。

## SMB和dbxfs

尽管WebDAV运行得相当不错，但对于这种用途来说并不高效。在实现davfuse时，一位同事问我为什么决定使用WebDAV而不是SMB。我知道SMB，但我一直认为它是一种复杂和神秘的网络技术，所以一开始根本没考虑使用它。他向我保证它是一种简单的二进制协议，比WebDAV更高效。我把这个想法记在心里。

在2015年的某个时候，我转而开始使用[ARM Linux桌面](https://en.wikipedia.org/wiki/Novena_(computing_platform))。这迫使我寻找一个Dropbox的替代品。我从来不喜欢官方Dropbox应用程序的一点是它基于同步。我实际上并不需要同步功能，因为我的桌面永远是在线的，我的网络连接速度也很快，所以我不介意实时下载文件。我也认为构建一个实时的Dropbox文件系统不会太困难。所有这些因素导致我试图建立一个。不过这一次我打算用Python写（为了开发效率），而且我会使用SMB而不是WebDAV（这最终是一个错误）。dbxfs的工作始于2015年8月22日。

我将dbxfs分为两个部分。通用的用户空间文件系统功能被包含在一个名为[userspacefs](https://pypi.org/project/userspacefs/)的Python包中，而Dropbox特定的功能被包含在一个名为[dbxfs](https://pypi.org/project/userspacefs/)的包中。dbxfs将是userspacefs包的第一个使用者。我这样做是为了以防其他人想要使用dbxfs使用的用户空间文件系统功能（我认为没有人真的使用过）。

dbxfs于2018年10月3日公开[发布](https://news.ycombinator.com/item?id=18133450)。它花了很长时间的原因是我一直在私下使用它，而且确实不想处理其他人使用它的问题。它始终是一个供黑客使用的工具，以[suckless](https://www.suckless.org/)的方式。就像我提到的，它是一个被动的副业，只是为了在我做其他事情的时候支持我。最初发布时仅支持Linux和macOS，那时我使用的唯一的平台。两天后，也就是2018年10月5日，我加入了OpenBSD支持，我有时用。我最初计划在不久之后添加Windows支持，但我不用Windows，也没有人要求，所以我没做。

## Windows支持

快进到 2023 年下半年。由于生活中的各种变化，我有一些空闲时间，决定对 dbxfs 进行一些清理。这一次，我决定确保整个用户空间文件系统包在mypy的严格类型检查下通过。这需要一些对SMB服务器代码的重构。为了确保在重构过程中 SMB 服务器代码仍然工作正常，我将我的 Windows VM 指向它。这揭示了一个我长时间忘记了的问题：Windows 不允许你在任意端口挂载SMB服务器。这意味着你无法运行用户空间SMB服务器，因为你必须绑定到仅允许管理员用户使用的端口445。另一个问题是，Windows 的最近版本默认禁用了对SMBv1的支持。有一些额外的支持代码和UAC提示可以解决这两个问题，但这将大大降低用户空间文件系统方法的实用性。

这杀死了我对于轻松让这在Windows上运作的希望。这让我很困扰。这就像我想象中的一个松散的尾端变得更松了，我不喜欢这种感觉。另一个让我失望的是，SMB最终并没有成为我所希望的WebDAV的可行高效替代方案。不管怎样，我觉得我必须解决这个问题。我知道从我在davfuse和Safe工作时，WebDAV在Windows上是可行的，所以我开始把我的davfuse WebDAV服务器代码移植到Python，最终的目标是将dbxfs完全移植到Windows。这是在 2023 年 10 月底。

大部分工作是在 2023 年末完成的。在这个过程中，我清理了用户空间文件系统 API，以便与 Python 的 os 模块没有任何阻力。也就是说，你可以使用几乎零胶水代码使用 Python 的 os 模块实现 [用户空间文件系统API](https://thelig.ht/code/userspacefs/file/userspacefs/stdlibfs.py.html)。我还将文件系统API全面[用代码记录下来](https://thelig.ht/code/userspacefs/file/userspacefs/abc.py.html)，这对于mypy的类型检查非常有帮助。我还为dbxfs添加了一个基本的GUI，这样Windows用户就不必使用命令行了。我对一切的成果感到满意。

## 一个重大的不幸的情况

正当我在添加Windows支持的过程中，微软突然决定停用WebDAV支持。为了解决这个问题，用户空间文件系统将显示一个UAC提示，以重新启用服务，如果服务被禁用的话。除此之外，并没有太多可说的了，Windows确实应该允许在任何端口挂载SMB。如果有微软工程师看到这个消息，请与我联系。

*2024-01-05编辑：一位朋友告诉我，Windows 现在提供了一个显式的用户空间文件系统 API，名为[Projfs](https://learn.microsoft.com/en-us/windows/win32/projfs/projected-file-system)。看起来很有前途。*

*2024-01-05: [Hacker News 上的 marwis](https://news.ycombinator.com/item?id=38887069) 指出，Windows 11 的最新版本允许在[备用端口上挂载SMB](https://techcommunity.microsoft.com/t5/storage-at-microsoft/smb-alternative-ports-now-supported-in-windows-insiders/ba-p/3974509)，这是个好消息。*

## 未来

现在[userspacefs](https://pypi.org/project/userspacefs/)已经在所有主要桌面平台上运行，并且完全通过了类型检查，我计划最终让它与mypyC一起编译。这与为dbxfs添加全面严格的类型检查是一致的。这些代码已经有8年的历史了，我并不喜欢重写。我不想为了效率而不得不将这些代码重写成C++。再说，这只是一个被动的旁门左道的项目 :)

我认为userspacefs现在处于一个良好的状态，并且有广泛的用途。现在它加入了对Windows的支持，这为用户空间文件系统打开了一个庞大的用户群。如果你对如何使用它有疑问，随时与我联系：[@cejetvole](https://twitter.com/cejetvole)。

Rian Hunter

2024-01-05
