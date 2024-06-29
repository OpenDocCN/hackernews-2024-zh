<!--yml

category: 未分类

date: 2024-05-27 15:00:26

-->

# 20 Years in the Making, GnuCOBOL Is Ready for Industry - The New Stack

> 来源：[https://thenewstack.io/20-years-in-the-making-gnucobol-is-ready-for-industry/](https://thenewstack.io/20-years-in-the-making-gnucobol-is-ready-for-industry/)

取出那些打孔卡！

开源项目GnuCOBOL经过20年的发展，“已经达到了工业成熟，并且可以在所有环境中与专有产品竞争”，如[GnuCOBOL贡献者](https://fabrice.lefessant.net/)和[OCamlPro创始人](https://ocamlpro.com/) [法布里斯·勒费桑特](https://fabrice.lefessant.net/)在[FOSDEM演讲](https://ftp.heanet.ie/mirrors/fosdem-video/2024/h2215/fosdem-2024-3249-gnucobol-the-free-industrial-ready-alternative-for-cobol-.av1.webm)中所说。

[GnuCOBOL](https://gnucobol.sourceforge.io/)将[COBOL源代码](https://thenewstack.io/how-cobol-code-can-benefit-from-machine-learning-insight/)转换为可执行应用程序。它非常跨平台，可以在Linux、BSD、许多专有Unix、macOS和Windows上运行，甚至是Android。最新版本v.32被用于许多商业设置中。

## 谁仍在使用COBOL？

COBOL（通用商业导向语言）于1959年发布，是一种主要用于大型组织的财务和人力资源部门的[高级语言](https://www.techtarget.com/searchitoperations/definition/COBOL-Common-Business-Oriented-Language)。现在是ISO标准，最新版本（v 35.060）[于2023年发布](https://standards.iteh.ai/catalog/standards/iso/31d28b41-4fef-4527-9083-0b65e495b32e/iso-iec-1989-2023)。

COBOL在一个关键方面是第一个现代语言：它被设计成跨平台。资助COBOL开发的美国国防部希望摆脱为每个供应商的计算机品牌支持不同编程语言的做法。可移植性是COBOL早期成功的关键。

尽管经常被视为遗留语言，Cobol仍然广泛使用，据估计仍然有高达800亿行代码存在。最惊人的部分是它还在以每年15%的速度增长。

当你使用ATM卡时，在幕后发生的大部分事情，如果不是[Java](https://thenewstack.io/we-can-have-nice-things-upgrading-to-java-21-is-worth-it/)，可能是COBOL，如[GnuCOBOL项目](https://github.com/GitMensch)领导者西蒙·索比希在同一场FOSDEM演讲中所说。

许多组织有大量COBOL代码库，太笨重无法迁移。但他们为什么要这样做呢？它快速可靠。

现在，COBOL部署主要由商业供应商主导。[IBM](https://www.ibm.com?utm_content=inline+mention)将COBOL捆绑到其主机中。[Micro Focus](https://thenewstack.io/back-future-micro-focus-can-make-hpe-software-purchase/)为PC提供COBOL。[Fujitsu NetCOBOL](https://www.fujitsu.com/global/products/software/developer-tool/netcobol/)在PC和主机上运行。

尽管如此，Sobisch指出，GnuCOBOL正在看到许多商业部署，例如用于银行后端应用程序，其中许多正在从Micro Focus迁移，用户报告由此带来的性能改进。法国[DGFIP](https://www.economie.gouv.fr/files/files/directions_services/dgfip/Rapport/Rapport_2011_version_anglaise.pdf)联邦机构从GCOS主机迁移到了GnuCOBOL，得益于Le Fessant的公司帮助。

## COBOL中的‘Hello World’

项目最初被称为OpenCOBOL，始于2002年，并于2013年改名为GnuCOBOL。在过去的三年中，它吸引了13名贡献者，提交了460个提交。

大多数Linux软件包管理器都有GnuCOBOL的可下载程序。

以下是COBOL中的“Hello World”程序，分为三个部分：

|  |
| --- |

**IDENTIFICATION  DIVISON**

**PROGRAM-ID.  prog**

**DATA       DIVISION**

**WORKING-STORAGE-SECTION**

**01  var-string  PIC  X(20)  VALUE  "Hello World"**

**PROCEDURE       DIVISION**

**DISPLAY  var-string**

END  PROGRAM  prog

|

identification division用于标识程序名称。data division保存数据（“Hello World”），procedure division包含函数。

## GnuCOBOL为企业提供了什么

当然，对于熟悉[Unix环境](https://thenewstack.io/fosdem-24-can-the-unix-shell-be-improved-hell-yes/)的人来说，GnuCOBOL很直观。它可以编译为C代码（C89+），从主机到Raspberry Pi都非常便携，Sobisch说。

有GnuCOBOL代码实现运行数千个处理器，这使得项目开发者有机会针对大型用例的性能和内存使用进行调整。

在合规方面，它通过了97%的COBOL 85一致性测试，这一成功率尚未被专有供应商实现，Sobisch夸耀道。它有19种方言，包括IBM和Micro Focus的扩展。

GnuCOBOL尚不支持对象或消息。

对象是“COBOL 22中的一个很好的功能，但使用并不多”，Sobisch说。

消息功能最近重新实施，对于COBOL群体来说仍然是一个新功能，因此GnuCOBOL中尚无支持。

还有一个新的[SuperBOL](https://get-superbol.com/)，是Le Fessant的OCamlPro开发的[GnuCOBOL开发工作室](https://github.com/OCamlPro/superbol-studio-oss)。它作为VSCode扩展运行，并具有完整的COBOL处理器（用[OCaml](https://ocaml.org/)编写）。然而，该软件仍处于早期开发阶段。

最后，GnuCOBOL 将成为即将到来的 [Google Summer of Code](https://gnucobol.sourceforge.io/gsoc.html) 活动中的特色语言之一，因此，整个新一代的编程人员都将能够说：“这不仅仅是 COBOL。这是 GnuCOBOL。”

<template class="youtube-subscribe-block-template"></template><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 68 31" version="1.1"><title>Group</title> <desc>Created with Sketch.</desc></svg>
