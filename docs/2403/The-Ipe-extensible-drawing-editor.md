<!--yml

category: 未分类

date: 2024-05-27 14:37:49

-->

# Ipe 可扩展绘图编辑器

> 来源：[https://ipe.otfried.org/](https://ipe.otfried.org/)

# Ipe 可扩展绘图编辑器

Ipe 是用于创建 PDF 格式图形的绘图编辑器。它支持创建小图形以包含到 LaTeX 文档中，还支持创建多页 PDF 演示文稿。

Ipe 的主要特点包括：

+   作为 LaTeX 源代码输入文本。这使得输入数学表达式并重用主文档的 LaTeX 宏变得容易。在显示中，文本显示为它将在图中显示的样子。

+   生成纯 PDF，包括文本。Ipe 在保存文件时将 LaTeX 源代码转换为 PDF。

+   可以通过各种捕捉模式轻松地相对于其他对象对齐对象（例如，在两条线的交点上放置一个点，或者通过三个给定点绘制一个圆）。

+   用户可以提供 ipelets（Ipe 插件）以增强 Ipe 的功能。这样，Ipe 可以根据具体任务进行扩展。

+   可编译为 Unix、Windows 和 OSX 版本。

+   Ipe 使用标准 C++ 和 Lua 5.4 编写。

您可以在 [手册](https://otfried.github.io/ipe) 中找到更多关于 Ipe 特性的信息。

该手册也以 [epub 格式](https://ipe.otfried.org/ipe-manual.epub) 和 [pdf 格式](https://ipe.otfried.org/ipe-manual.pdf) 的书籍形式提供。

如果你想开发 Ipe 脚本或 ipelets，你需要 [Ipe 库文档](https://ipe.otfried.org/ipelib)。

#### 下载当前的 Ipe 版本

当前的 Ipe 版本是 Ipe 7.2.29。（旧版本可以在 [这里](https://github.com/otfried/ipe/releases)找到，甚至更旧的版本在 [这里](https://github.com/otfried/old-ipe-releases/releases)。）

我提供了四种 Ipe 下载选项：Windows 的二进制分发包，Mac OS X 的二进制包，几个 Linux 发行版的二进制包，以及应该能够在任何最新 Unix 系统上编译的源码包。

你也可以将 Ipe 安装为 [flatpak](https://flathub.org/apps/org.otfried.Ipe) —— 这将适用于许多 Linux 发行版，包括不再构建包的旧版本。

##### Windows 二进制包

[ipe-7.2.29-win64.zip](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-win64.zip)

在你的 Windows 计算机上解压这个包到某个地方。这个包至少需要 Windows 7。

##### Mac OS X 二进制包

下载以下磁盘映像，并将 Ipe.app 复制到运行 Mac OS 10.10 或更高版本的 Mac 上。

安全警告：当您尝试正常打开新下载的 Ipe 版本时，MacOS 将显示一个安全警告，指出它无法验证开发者并且无法检查恶意软件。这是 [MacOS 的正常行为](https://support.apple.com/en-au/guide/mac-help/mh40616/mac)。按住 Control 键点击 Ipe 图标，然后从快捷菜单中选择“打开”。

对于配备Intel处理器的Apple电脑：[ipe-7.2.29-mac-intel.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-mac-intel.dmg)

对于配备ARM处理器的Apple电脑（“Apple Silicon”）：[ipe-7.2.29-mac-arm.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-mac-arm.dmg)

对于MacOS，[IpePresenter](https://ipepresenter.otfried.org)提供作为一个单独的应用程序。将IpePresenter.app复制到您的Mac上：

对于配备Intel处理器的Apple电脑：[ipepresenter-7.2.29-mac-intel.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipepresenter-7.2.29-mac-intel.dmg)

对于配备ARM处理器的Apple电脑（“Apple Silicon”）：[ipepresenter-7.2.29-mac-arm.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipepresenter-7.2.29-mac-arm.dmg)

##### Linux二进制包

几个Linux发行版（包括Debian、Ubuntu、Linux Mint、Fedora、Arch Linux）提供了一个ipe软件包，您可以通过发行版的软件包管理器安装。

新版Ipe需要一段时间才能进入发行版，特别是长期稳定的发行版。多亏了美妙的[openSuse构建服务](https://build.opensuse.org)，我可以为一些最新的Linux发行版（目前包括Debian、Ubuntu、Mint、Fedora和openSuse）提供Ipe安装包。（在Arch Linux上，您可以依赖于[官方软件包](https://aur.archlinux.org/packages/ipe/)，该软件包总是非常及时更新。）

在安装这些软件包之前，请删除通过您发行版的软件包管理器安装的任何旧版本Ipe！

#### 其他Linux发行版和旧版本（例如Ubuntu 20.04、Debian 11、Fedora 36）

如果您的Linux发行版未在下面列出，或者如果您的Linux发行版较旧，您可以将Ipe安装为[flatpak](https://flathub.org/apps/org.otfried.Ipe)。（例如，自Ipe 7.2.27以来，Ipe依赖于Qt6，我们无法再为Ubuntu 20.04、Debian 11或Fedora 36提供软件包。）

#### Debian、Ubuntu、Mint

对于这些基于Debian的发行版，请根据以下表格下载DEB软件包。

然后通过以下方式安装软件包：

```
$ sudo gdebi ipe_7.2.29-1_amd64.deb

```

要再次移除Ipe，请使用以下命令：

```
$ sudo apt-get remove ipe

```

#### Fedora和openSuse

对于这些发行版，您需要根据以下表格下载RPM软件包。

##### 愿源代码与您同在

[ipe-7.2.29.tar.gz](https://github.com/otfried/ipe/archive/refs/tags/v7.2.29.tar.gz)

这包括构建Ipe的源代码以及Ipe文档。查看install.txt文件以获取安装说明。

#### Ipe商品

您是Ipe的粉丝吗？您可以通过佩戴[Ipe T-shirt](https://www.shirtee.com/en/store/ipe)向所有人展示，并同时赞助Ipe的开发。

#### 赞助Ipe的开发

现在您有机会成为支持Ipe开发的[社区成员](donate.html)。

#### 维基

在手册之后，您可以从[Ipe 7 Wiki](https://github.com/otfried/ipe-wiki/wiki)获取有用的信息源，例如示例文件或者对常见问题的回答。Ipe 用户可以在这里添加有用的技巧和窍门，或者与 Ipe 相关的任何内容。

#### 邮件列表

Ipe 有两个邮件列表。第一个邮件列表仅用于公布新版本的 Ipe，以及可能对广大用户有兴趣的新 ipelets。这个列表的流量非常小，因为这个列表上的大多数消息都来自我。您可以在[这里](https://mailman.science.uu.nl/mailman/listinfo/ipe-announce)订阅公告列表。

第二个列表用于讨论 Ipe。您可以在[这里](https://mailman.science.uu.nl/mailman/listinfo/ipe-discuss)订阅。请不要用它来报告 bug—bug 跟踪器更适合这样做。

这两个列表由 René van Oostrum 维护。谢谢！

#### IpePresenter

现在的 Ipe 配备了一个伴随程序 IpePresenter。IpePresenter 是一个展示工具，用于展示 PDF 演示文稿（使用 Ipe 制作或者使用 beamer Latex 包制作）—详细信息请参见[IpePresenter 主页](https://ipepresenter.otfried.org)。

IpePresenter 包含在 Windows 和 Linux 上的 Ipe 二进制包中，并且也可以作为单独的 Windows 和 MacOS 二进制包（不包括 Ipe）使用。

#### 报告 bug

在报告 bug 之前，请确认您拥有最新的 Ipe 版本，并检查问题是否在[FAQ](https://otfried.github.io/ipe/96_faq.html)中有解释。请不要直接将 bug 报告发送给我（如果您报告的第一件事就是将其输入 bug 跟踪器）。

要报告 bug，请使用[Ipe bug 跟踪器](bugzilla.html)（点击 New issue）。

#### 版权

可扩展的绘图编辑器 Ipe 是"自由"的，这意味着每个人都可以在特定条件下自由使用和再分发它。Ipe 不属于公共领域；它受版权保护，并且有关其分发的限制如下：

版权 © 1993–2024 Otfried Cheong

此程序是自由软件；您可以根据自由软件基金会发布的 Gnu 通用公共许可证的第 3 版或（按您的选择）任何更高版本来重新分发和/或修改它。

作为特殊例外，您有权限将 Ipe 与 CGAL 库链接并分发可执行文件，只要您遵循 Gnu 通用公共许可证在除 CGAL 之外的所有软件中的要求。

这个程序是希望它对您有所帮助，但没有任何保证；甚至没有适销性或特定用途的默示保证。请查看[Gnu 通用公共许可证](https://www.gnu.org/copyleft/gpl.html)获取更多详细信息。

#### 其他下载

可在 GitHub 上的[ipe-tools](https://github.com/otfried/ipe-tools)存储库中获取多个独立程序，这些程序仅以源代码形式提供。

##### Svgtoipe

svgtoipe 将 SVG 图形转换为 Ipe 格式。它无法处理 SVG 中的所有内容，但对于几何对象和渐变应该可以处理。这个程序实际上是一个 Python 脚本。

##### Poweripe

你更喜欢在 Ipe 中创建你的演示文稿吗？但是你的老板/同事/客户总是要求 Powerpoint 格式的文件？

不再担心！Poweripe 是一个 Python 脚本，将 Ipe 演示文稿转换为 Powerpoint 演示文稿。

##### Pdftoipe

pdftoipe 尝试将任意 PDF 文件转换为 Ipe 的 XML 格式。你需要 poppler 库来编译它。

##### Ipe Python 模块

ipepython 是一个 Python 3 的扩展模块，允许你读取和写入 Ipe 文档，并处理文档中的所有信息。

##### Matplotlib 后端

Matplotlib 是一个用于科学绘图的 Python 模块。使用这个后端，你可以直接从 matplotlib 创建 Ipe 图形。

##### Figtoipe

figtoipe 将用 Xfig 制作的图形转换为 Ipe 的 XML 格式。它不能处理 Xfig 的所有功能。figtoipe 已由 Alexander Bürger 改进并目前正在维护。谢谢！

##### ipe5toxml

如果你仍然有用 Ipe 5 制作的图形，你可以使用这个程序将它们转换为 Ipe 6 可理解的格式。然后可以运行 ipe6upgrade 将它们转换为 Ipe 7 格式。ipe5toxml 的源代码位于公共领域。
