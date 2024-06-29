<!--yml

category: 未分类

date: 2024-05-27 14:55:08

-->

# 识别软件 — 2024 — 博客 — GNU Guix

> 来源：[https://guix.gnu.org/en/blog/2024/identifying-software/](https://guix.gnu.org/en/blog/2024/identifying-software/)

## 识别软件

Ludovic Courtès, Maxim Cournoyer, Jan Nieuwenhuizen, Simon Tournier — 2024年3月4日

“识别软件”需要什么？我们如何确定机器上运行的软件，以确定例如可能影响其安全性漏洞？

2023年10月，美国国家网络安全与基础设施安全局（CISA）发布了名为[*软件识别生态系统选项分析*](https://www.cisa.gov/resources-tools/resources/software-identification-ecosystem-option-analysis)的白皮书，探讨了解决这些问题的现有选项。该出版物随后引发了[评论请求](https://www.regulations.gov/document/CISA-2023-0026-0001)；我们作为Guix开发者的[评论](https://git.savannah.gnu.org/cgit/guix/maintenance.git/plain/doc/cisa-2023-0026-0001/cisa-2023-0026-0001.pdf)由于时间问题未能及时发布，但我们想在这里分享。

软件识别对于网络安全至关重要，正如白皮书在其介绍中所述：

> 有效的漏洞管理要求软件以一种可跟踪的方式进行，这样可以与其他信息（如已知漏洞）进行关联[...]。只有当不同的网络安全专业人员知道他们在谈论同一款软件时，这种关联才可能实现。

[通用平台标识（CPE）](https://en.wikipedia.org/wiki/Common_Platform_Enumeration)标准旨在填补此角色；它用于作为[通用漏洞和曝光（CVE）](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures)过程的一部分识别软件。但CPE作为*外在识别机制*显示出其局限性：CPE选择的人类可读标识符未能捕捉“软件”复杂性。

我们认为由Nix和Guix实施的功能性软件部署，结合由Software Heritage进行的源代码识别工作，为这些问题提供了独特的视角。

# 关于软件识别

美国国家网络安全与基础设施安全局（CISA）于2023年10月发布的[*软件识别生态系统选项分析*](https://www.cisa.gov/resources-tools/resources/software-identification-ecosystem-option-analysis)白皮书研究了朝向定义*可用于全球所有关键网络安全用例的软件识别生态系统*的选项。

我们的经验在于设计和开发[GNU Guix](https://guix.gnu.org)，一个包管理器、软件部署工具和GNU/Linux发行版，强调三个关键元素：**可再现性、来源追踪和可审计性**。在接下来的章节中，我们解释了我们的方法及其与前述白皮书目标的关系。

Guix从源代码中生成各种复杂程度的二进制作品：软件包二进制、应用程序捆绑包（供Docker及相关工具消费的容器映像）、系统安装、系统捆绑包（容器和虚拟机映像）。

所有这些作品都可以被归类为“软件”，源代码也是如此。其中一些“软件”来自于明确定义的上游软件包，有时会被打包者（补丁）在下游进行修改；二进制作品本身是构建过程的副产品，包管理器在此过程中使用*其他*先前构建的二进制作品（编译器、库等），以及更多的源代码（软件包定义）来构建它们。如何以那种意义上识别“软件”？

软件是双重的：它以*源*形式存在，也以*二进制*形式存在，后者是将源代码和中间二进制文件作为输入进行的复杂计算过程的结果。

我们的论文可以总结如下：

> **我们认为，用于识别源代码标识符的要求不同于识别二进制作品的要求。**
> 
> 我们的观点体现在GNU Guix中：
> 
> 1.  **源代码**可以通过诸如加密哈希等*固有标识符*以一种明确和分布式的方式进行识别。
> 1.  
> 1.  **二进制作品**则需要作为一个*全面且可验证的构建过程的副产品本身可用的源代码*。

在接下来的章节中，为了澄清这一声明的背景，我们展示了Guix如何识别源代码，如何定义*从源到二进制*路径并确保其可验证性，以及如何提供溯源跟踪。

# 源代码识别

Guix包含了近30,000个包的[包定义](https://guix.gnu.org/manual/en/html_node/Defining-Packages.html)。每个包定义都标识其[起源](https://guix.gnu.org/manual/en/html_node/origin-Reference.html)——其“主”源代码以及补丁。起源是**内容寻址的**：它包括代码的SHA256加密哈希（一个*固有标识符*），以及下载它的主要URL。

由于源代码是内容寻址的，因此可以将URL视为提示。事实上，**我们将Guix与[软件遗产](https://www.softwareheritage.org)源代码存档连接起来**：当源代码从其原始URL消失时，Guix将退而从存档中下载它。这得益于Guix和软件遗产都使用固有（或内在）标识符。

更多信息可以在这篇[2019年的博客文章](https://guix.gnu.org/en/blog/2019/connecting-reproducible-deployment-to-a-long-term-source-code-archive/)和[软件哈希标识符（SWHID）](https://www.swhid.org/)工作组的文件中找到。

# 可再现构建

Guix通过确保[可重现构建](https://reproducible-builds.org)提供了从源码到二进制的**可验证路径**。为了实现这一点，Guix建立在Eelco Dolstra的开创性研究工作基础上，该工作导致了[Nix软件包管理器](https://nixos.org)的设计，二者共享相同的概念基础。

Guix依赖于*隔离构建*：在仅包含显式声明的依赖项的隔离环境中执行构建，其中“依赖项”可以是另一个构建过程或源码，包括构建脚本和补丁。

一个重要的结果是**可以独立验证构建**。例如，对于Guix的特定版本，`guix build gcc`应该生成完全相同的二进制文件。为了促进独立验证，`guix challenge gcc`比较由不同方发布的GNU编译器集合（GCC）的二进制工件。用户还可以使用`guix build gcc --check`与本地构建进行比较。

与Nix类似，构建过程通过*衍生物*标识，这些是低级别的内容地址化构建指令；衍生物可以引用其他衍生物和源代码。例如，`/gnu/store/c9fqrmabz5nrm2arqqg4ha8jzmv0kc2f-gcc-11.3.0.drv`唯一标识了构建GNU编译器集合（GCC）版本11.3.0的特定变体的衍生物。改变软件包定义（应用补丁、构建标志、依赖集）或类似地改变其依赖的一个软件包，会导致不同的衍生物（更多信息可以在[Eelco Dolstra的博士论文](https://edolstra.github.io/pubs/phd-thesis.pdf)中找到）。

衍生物形成一个图形，**记录导致二进制工件的所有构建过程**。相比之下，仅有软件包名称/版本对如`gcc 11.3.0`无法捕获导致二进制工件的广度和深度元素。这是像**公共平台枚举**（CPE）标准的缺陷之一：它不能表达一个适用于`gcc 11.3.0`的漏洞是否适用于无论如何构建、打补丁和配置的情况，或者是否需要特定条件。

# 全源码引导（Full-Source Bootstrap）

仅依靠可复现构建无法确保源码到二进制的对应关系：编译器可能包含后门，正如肯·汤普森在《*Reflections on Trusting Trust*》中展示的那样。为了解决这个问题，Guix进一步实现了所谓的**全源码引导**：第一次，分发中的每一个软件包都是从源码编译而来的，[从一个非常小的二进制种子开始](https://guix.gnu.org/en/blog/2023/the-full-source-bootstrap-building-from-source-all-the-way-down/)。这提供了前所未有的透明度，允许在所有层面进行代码审计，并提高了对肯·汤普森描述的“信任攻击”的鲁棒性。

欧盟通过 [NLnet 隐私与信任增强技术 (NGI0 PET) 拨款](https://nlnet.nl/project/GNUMes-fullsource/) 认识到了这项工作的重要性，该拨款于 2021 年分配给 Jan Nieuwenhuizen，以进一步在 GNU Guix、GNU Mes 及相关项目中开展全源码引导工作，随后于 2022 年分配了[另一项拨款](https://nlnet.nl/project/GNUMes-ARM_RISC-V/)，以扩展对 Arm 和 RISC-V CPU 架构的支持。

# 溯源跟踪

我们将溯源跟踪定义为**将二进制构件映射回其完整对应的源码的能力**。溯源跟踪是必要的，以允许二进制构件的接收者访问相应的源代码，并在希望这样做时验证源码与二进制的对应关系。

[`guix pack`命令](https://guix.gnu.org/manual/en/html_node/Invoking-guix-pack.html) 可用于构建例如容器镜像。运行 `guix pack -f docker python --save-provenance` 将生成一个*自描述 Docker 镜像*，其中包含 Python 及其运行时依赖的二进制文件。这个镜像是自描述的，因为 `--save-provenance` 标志导致包含一个*清单*，描述了用于生成此二进制文件的 Guix 修订版。第三方可以检索 Guix 的这个修订版，并从此查看 Python 的整个构建依赖图，查看其源代码和任何应用的补丁，以及递归地查看其依赖项。

总结来说，只需捕获使用的 Guix 修订版，即可*复制*特定的二进制构件。这由 [时间机器命令](https://guix.gnu.org/manual/en/html_node/Invoking-guix-time_002dmachine.html) 所说明。以下示例在任何时间、任何机器上部署`python`包的特定构建构件，就像在这个 Guix 提交中定义的一样：

```
guix time-machine -q --commit=d3c3922a8f5d50855165941e19a204d32469006f \
  -- install python
```

换句话说，由于 Guix 本身定义了构件的构建方式，**Guix 源码的修订版与包名称的结合能够唯一标识该包的二进制构件**。作为科学家，我们基于这一特性来实现可复制的研究工作流程，正如解释在这篇[2022年的《自然科学数据》文章](https://doi.org/10.1038/s41597-022-01720-9)中；作为工程师，我们重视这一特性，以分析我们正在运行的系统，并确定适用的已知漏洞和错误。

再次说明，作为仅仅列出包名称/版本对的软件材料清单 (SBOM) 将无法捕获这么多信息。而 [OmniBOR 的构件依赖图 (ADG)](https://omnibor.io/) 虽然不那么模棱两可，但在两个方面不足：对于典型的网络安全应用来说太细粒度（在单个源文件的级别），并且仅捕获个别文件的所谓源码/二进制对应关系，而不涉及从源码到二进制的过程。

# 结论

固有标识符非常适合于明确的源代码识别，正如软件遗产、Guix 和 Nix 所展示的那样。

然而，我们认为二进制文件应被视为计算过程的结果；需要完全记录该过程以支持对源代码/二进制对应关系的**独立验证**。为了网络安全目的，接收二进制文件的人必须能够将其映射回源代码（*来源追踪*），并且还要能够重现整个构建过程以验证源代码/二进制的对应关系（*可再现构建和全源引导*）。只要二进制文件是由可重现的构建过程生成的，那么**识别二进制文件归结为识别其构建过程的源代码**。

这些思想在 2022 年的科学论文 [*使用 GNU Guix 构建安全的软件供应链*](https://doi.org/10.22152/programming-journal.org/2023/7/1) 中得到了发展。

除非另有声明，本站点上的博客文章版权归各自的作者所有，并根据 [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) 许可证和 [GNU 自由文档许可证](https://www.gnu.org/licenses/fdl-1.3.html)（版本 1.3 或更新版本，无不变部分、无封面文字和无封底文字）发布。
