<!--yml

category: 未分类

date: 2024-05-29 12:47:20

-->

# rev.ng 反编译器开源 + UI 封闭测试开始 - rev.ng

> 来源：[https://rev.ng/blog/open-sourcing-renvg-decompiler-ui-closed-beta](https://rev.ng/blog/open-sourcing-renvg-decompiler-ui-closed-beta)

嘿！好久不见！我们已经有一段时间没有关于 rev.ng 的更新了，但我们一直在悄悄努力。所以，现在是时候带来一些新闻了！ 🚀

*tl;dr* 在这篇博文中，我们：

1.  宣布 rev.ng 反编译器的开源化，UI 封闭测试的开始，新网站的发布，rev.ng Hub 和文档。此外，我们提供[私人演示](mailto:info@rev.ng)。

1.  解释如何尝试 rev.ng。

1.  解释整个 rev.ng 的设计和目标。

1.  解释开源项目、云端 UI 和独立 UI 之间的关系。

1.  展示我们向 1.0 发布的路线图。

1.  描述如何与我们互动（例如，[X/Twitter](https://twitter.com/_revng)，[Discord](https://discord.gg/wEQtgKJxcX)，[Newsletter](/newsletter-subscribe)，[Discourse](https://discuss.rev.ng/)）。

## [](#1-what-happens-today)1\. 今天发生了什么？

相当多。

+   我们正在开源[`revng-c`](https://github.com/revng/revng-c)，我们反编译器的后端。

    截至今天，整个反编译引擎已开源！

    我们还发布了关于如何使用 rev.ng 的初步[文档](https://docs.rev.ng)。

+   我们即将邀请 newsletter 订阅者参加 rev.ng UI 的封闭测试。

    我们将按先到先得的顺序邀请人员，请确保[您已注册](/newsletter-subscribe)。

+   我们发布了新网站。您正在查看它。

+   我们发布了[rev.ng Hub](https://hub.rev.ng)，这是使用 rev.ng 云版本的入口。

    封闭测试用户将能够创建项目，在浏览器中运行 UI 并与其他用户协作，无需安装任何内容。

    如果您尚未参与封闭测试，仍然可以[探索现有的公共项目](https://hub.rev.ng/)。

+   我们开始寻找对 rev.ng 功能感兴趣的人，希望能提供私人演示（[有兴趣吗？](mailto:info@rev.ng)）。

## [](#2-how-to-try-revng)2\. 如何尝试 rev.ng

简要地，让我们安装`revng`。无需 root 权限，所有内容都将适合一个单独的目录，您随时可以丢弃它。

```
$  $ curl -L -s https://rev.ng/downloads/revng-distributable/master/install.sh |  bash  $ cd revng $ source ./environment 
```

好的，现在让我们来反编译一个简单的程序。考虑`example.c`：

```
int  main(int argc,  char  *argv[])  {   return argc *  3;  } 
```

你可以编译和反编译它：

```
$ gcc example.c -o example -O2 $ revng artifact \  --analyze \  --progress \  decompile-to-single-file \  example \   | revng ptml --color \   |  grep -A2 -B1 '[^_]main\b'  \   > decompiled.c 
```

你应该获得：

```
_ABI(SystemV_x86_64)  generic64_t  main(generic64_t _argument0)  {   return _argument0 *  3  &  0xFFFFFFFF;  } 
```

或者，看看[文档](https://docs.rev.ng)。特别是，确保正确地[设置工作环境](https://docs.rev.ng/user-manual/working-environment/)，按照教程[从头开始创建模型](http://docs.rev.ng/user-manual/model-tutorial/)或者直接尝试[分析教程](http://docs.rev.ng/user-manual/analyses/)，这将展示如何一次性调用反编译程序。

要尝试 UI，请[注册我们的 newsletter](/newsletter-subscribe)。正如之前提到的，我们很快将开始分批邀请人员。

请注意，此版本的目标是演示UI和对一些二进制文件的反编译结果，并在我们得到有趣结果的一些实际二进制文件上让人们进行工作，然后让人们尝试并获得反馈、错误报告等。

特别是，以下是我们用于测试的一些非平凡二进制文件（及其`.text`大小）：[hostname](https://rev.ng/downloads/binaries/hostname)（2K）、[ntfscat](https://rev.ng/downloads/binaries/ntfscat)（4K）、[umount](https://rev.ng/downloads/binaries/umount)（8K）、[chroot](https://rev.ng/downloads/binaries/chroot)（15K）、[nc.openbsd](https://rev.ng/downloads/binaries/nc.openbsd)（15K）、[apt-get](https://rev.ng/downloads/binaries/apt-get)（18K）、[df](https://rev.ng/downloads/binaries/df)（45K）、[parted](https://rev.ng/downloads/binaries/parted)（38K）、[gzip](https://rev.ng/downloads/binaries/gzip)（54K）、[updatedb.plocate](https://rev.ng/downloads/binaries/updatedb.plocate)（56K）、[resolvectl](https://rev.ng/downloads/binaries/resolvectl)（61K）、[ps](https://rev.ng/downloads/binaries/ps)（47K）。

rev.ng支持许多ABI和平台。但是，在这个阶段，我们在Linux x86-64二进制文件上执行了初始QA。rev.ng在封闭测试期间将有显著发展，因此请确保您关注我们的[X/Twitter账号](https://twitter.com/_revng)并订阅我们的[月刊](/newsletter-subscribe)以了解何时回来尝试一些新功能！

如果您遇到问题，请转到[Discourse](https://discuss.rev.ng/)寻求帮助。

## [](#3-our-goals)3\.我们的目标

现在是了解rev.ng反编译器的好时机。简而言之，我们专注于：

+   数据结构的自动恢复;

+   现代用户体验;

+   协作逆向工程;

+   广泛的平台支持;

+   可扩展性。

### [](#automatic-data-structures-recovery)自动数据结构恢复

通过数据布局分析，我们可以自动*过程间*恢复`struct`的布局。

考虑以下`struct`，一个带有5个整数数组的链表中的节点：

```
struct  node  {   int64_t data[5];   struct  node  *next;  }; 
```

一个对数组元素求和的函数:

```
int64_t  sum(struct  node  *n)  {   int64_t result =  0;   for  (int i =  0; i <  5;  ++i)  result += n->data[i];   return result;  } 
```

并且一个遍历链表的函数:

```
int64_t  compute(struct  node  *n)  {   int64_t result =  0;   while  (n)  {  result +=  sum(n);  n = n->next;   }   return result;  } 
```

rev.ng生产的输出默认情况下如下（除了注释）：

```
typedef  struct  _PACKED  {   

  generic64_t _offset_0[5];   

 _struct_1876 *_offset_40;  } _struct_1876;  

_ABI(SystemV_x86_64)  generic64_t  sum(_struct_1876 *_argument0)  {   generic64_t _var_0 =  0, _var_1 =  0;   do  {   

 _var_0 = _var_0 + _argument0->_offset_0[_var_1];  _var_1 = _var_1 +  1;   }  while  (_var_1 !=  5);   return _var_0;  }  
_ABI(SystemV_x86_64)  generic64_t  compute(_struct_1876 *_argument0)  {  _struct_1876 *_var_0 = _argument0;   generic64_t _var_1 =  0;   generic64_t _var_2;   do  {  _var_2 =  sum(_var_0);  _var_1 = _var_1 + _var_2;   

 _var_0 = _var_0->_offset_40;   }  while  (_var_0);   return _var_1;  } 
```

很不错，不是吗？您还可以直接在[Hub](https://hub.rev.ng/revng-demos/linked-list)上进行。

额外奖励：我们生成语法上有效的C代码。

**截至今日状态**：可用。

### [](#a-modern-ux)现代用户体验

我们的UI基于VSCode。如果您使用VSCode，您已经知道如何使用它。此外，UI可以在浏览器标签中运行，也可以作为独立应用程序运行。

快捷方式:

+   `Ctrl + 单击`，导航到光标下的函数/类型定义。

+   `N`重命名它。

+   `Y`让你编辑它的类型。

+   `X`显示引用。

rev.ng不仅仅是一个反编译器，它是一个*交互式*反编译器。这意味着您可以交互式地进行更改，只有受到您更改影响的内容才会重新计算，就像您所期望的那样。

**今日状态**：UI现在仅限于封闭测试的参与者。

### [](#collaborative-reversing)协作逆向

rev.ng UI采用客户端-服务器架构。

多个用户可以连接到同一个守护程序实例，并同时在同一个项目上工作。如果您使用云版本，还可以获得类似GitHub的应用程序来管理项目，[rev.ng Hub](https://hub.rev.ng)。

**今日状态**：运行正常，需要一些改进用户体验的工作（[路线图项目＃797](/roadmap#feature-797)）。

### [](#wide-architecture-support)广泛的架构支持

为了理解为什么我们可以轻松支持许多架构，让我们简要地讨论一下编译器和模拟器。

编译器通常分为前端、中间端和后端。前端处理特定于输入语言的细节。中间端在独立于输入语言和目标架构的中间表示（IR）上执行优化。最后，后端执行特定于架构的优化并输出可执行代码。

让我们考虑一下 LLVM 生态系统的一部分：

LLVM IR 是 LLVM 的中间表示。

现在让我们考虑一下 QEMU，这是一个著名的模拟器。当在仿真模式下运行时，与虚拟化模式（例如 KVM）相反，它工作方式类似。它有自己的前端，自己的IR（简明代码）和多个后端。主要区别在于输入不是 C 或 C++，而是 ARM 或 x86-64 可执行代码：

这种类型的仿真也被称为动态二进制翻译。

现在，rev.ng 的工作方式如下：

基本上我们使用 QEMU 管道的第一半将可执行代码提升为简明代码，转换为 LLVM IR，然后继续反汇编过程。

这意味着我们可以相对轻松地支持由QEMU支持的任何[架构](https://wiki.qemu.org/Documentation/Platforms)。

然而，架构支持只是故事的一部分。另一个重要议题是某个平台或更具体地说是ABI的支持有多好。在这方面，我们付出了很多努力，以便能够以声明性的方式描述ABI的所有属性。我们的[ABI描述格式](https://github.com/revng/revng/tree/f297712/share/revng/abi)足够通用，以至于支持新的ABI将会相当容易。

**今日状态**：我们针对x86、x86-64、ARM、AArch64、MIPS和s390x有不同程度的成熟支持。在二进制格式方面，我们支持ELF、PE/COFF和Mach-O。我们还可以从`.idb`、DWARF调试信息和PDB调试信息中导入。我们对Linux x86-64二进制文件进行了大部分的QA，其余都需要额外的QA。

如果您对某个平台有特殊兴趣（并且有预算），[我们很乐意交谈`:)`](mailto:info@rev.ng)。

跟踪[路线图项目＃58](/roadmap#feature-58)以进一步对更多平台进行QA。

### [](#extensibility)可扩展性

rev.ng 力求成为反向工程工具的首选框架。因此，整个项目都是开源的，除了我们的交互式 UI 外。

rev.ng 也很容易脚本化。我们的项目文件，[模型](https://docs.rev.ng/user-manual/model/)，只是一个 YAML 文档。以下是带有一些注释的示例。

```
 Architecture: x86_64 DefaultABI: SystemV_x86_64 
 Segments:   -  StartAddress:  "0x400000:Generic64"   VirtualSize:  7   StartOffset:  0   FileSize:  7   IsReadable:  true   IsWriteable:  false   IsExecutable:  true  
 Functions:   -  
  Entry:  "0x400000:Code_x86_64"   
  CustomName:  "Sum"   
  Prototype:  "/Types/1809-CABIFunctionType"  
 Types:   
  -  Kind: PrimitiveType  ID:  1288   PrimitiveKind: Unsigned  Size:  8  

  -  Kind: CABIFunctionType  ABI: SystemV_x86_64  ID:  1809   Arguments:   -  Index:  0   CustomName: FirstAddend  Type:   UnqualifiedType:  "/Types/1288-PrimitiveType"   -  Index:  1   CustomName: SecondAddend  Type:   UnqualifiedType:  "/Types/1288-PrimitiveType"   ReturnType:   UnqualifiedType:  "/Types/1288-PrimitiveType" 
```

该文档提供了一个 [逐步教程](http://docs.rev.ng/user-manual/model-tutorial/) 来理解上述模型。

基本上，如果您可以解析 JSON 或 YAML，您可以脚本化 rev.ng。

我们计划为 Python 和 TypeScript 提供易于使用的包装器，以便轻松进行更改并获取工件（例如反编译代码或汇编）。

Python 示例用法：

```
>>>  import yaml >>>  from revng import model >>>  from pathlib import Path >>> binary = yaml.load(Path("model.yml").read_text(), Loader=model.YamlLoader)  >>> function = binary.Functions[0]  >>> function.CustomName Sum
>>> function.CustomName +=  "42"  >>> function.CustomName Sum42
>>>   >>> function.get_artifact("decompile-to-single-file")  uint64_t Sum(uint64_t FirstAddend, uint64_t SecondAddend)  {  ... 
```

TypeScript 示例用法：

```
import  { parseModel }  from  "revng-model";  import  { readFileSync }  from  "fs";  
const file =  readFileSync("model.yml",  "utf-8");  const binary =  parseModel(file);  const the_function = binary.Functions[0];  console.log(the_function.CustomName.str)  the_function.CustomName.str  +=  "42"  console.log(the_function.CustomName.str)   the_function.get_artifact("decompile-to-single-file") 
```

```
orc shell npm install typescript @types/node revng-model
tsc example.ts
node example.js 
```

最后，由于我们广泛使用 LLVM IR 作为主要的内部表示形式，您可以在其生态系统中利用许多工具。例如，您可以使用 [KLEE](https://klee.github.io/) 进行符号执行，或者使用 [SanitizerCoverage](https://clang.llvm.org/docs/SanitizerCoverage.html) + [libFuzzer](https://llvm.org/docs/LibFuzzer.html)（或其他工具）进行覆盖引导模糊测试。此外，由于我们的反编译器的输出是有效的 C 代码，我们还能够使用 [clang 静态分析器](https://clang-analyzer.llvm.org/) 直接查找反编译代码中的常见漏洞。但这可能需要另一篇专门的博客文章。

**截至今日的状态**：我们有 Python 和 TypeScript 包装器来操作模型，但还没有简便的方式来运行分析和获取工件。[路线图项目 #17](/roadmap#feature-17) 跟踪构建一个完整的 Python 客户端。

## [](#4-open-source-vs-free-to-use-vs-premium)4\. 开源 vs 免费使用 vs 高级版

欲了解 rev.ng 各组件的快速比较，请查看 [功能比较页面](/pricing)。

简而言之：

1.  rev.ng 框架完全开源。您可以从 CLI 反编译任何您想要的内容。

1.  UI 将以以下形式提供：

    1.  公共项目在云端免费使用；

    1.  适用于云端私有项目的订阅；

    1.  作为完全独立、完全脱机应用程序的成本可用。

### [](#revng-in-the-cloud)rev.ng 云端版本？

是的。

1.  您可以通过 [rev.ng Hub](https://hub.rev.ng/) 创建项目并邀请协作者。

1.  UI 在浏览器中运行。

1.  后端在我们的云端运行。

这是交易的内容：

1.  公共项目免费。如果您愿意让您的项目文件公开，您可以免费使用 rev.ng 云版本，包括 UI。

1.  私有项目需要订阅。

如果您对 Kubernetes 感兴趣，我们也愿意讨论在本地安装 rev.ng 云服务的私有实例。[有兴趣吗？](mailto:info@rev.ng)

如上所述，我们还将提供一个完全独立的 UI 版本，您可以购买常规许可证并在任何地方运行它。

## [](#5-the-roadmap)5\. 发展路线图

rev.ng 是一个复杂的项目，要达到 1.0 版本需要相当大的努力。但不用担心，我们有详细的路线图来达成这一目标。

路线图分为 4 个层级：

1.  Tier 1（已完成）：alpha 版本，向朋友展示演示。

1.  Tier 2（今天开始）：beta 版本，向通讯订阅者提供云端版本的访问权限。

1.  Tier 3（即将推出）：公开 beta 版本。

1.  Tier 4（即将推出）：1.0 版本发布。

想了解更多？查看[详细路线图页面](/roadmap)。

## [](#6-how-to-get-in-touch-and-stay-up-to-date)6\. 如何联系和保持最新

我们发布内容并可以通过多种方式联系我们：

+   [X/Twitter](https://twitter.com/_revng)：频繁发布开发进展和重要公告。

+   [Discord](https://discord.gg/wEQtgKJxcX)：与开发团队进行随机聊天和实时讨论。

+   [Discourse](https://discuss.rev.ng/)：用户获取支持和报告错误的论坛。

+   [GitHub](https://github.com/revng)：我们所有开源开发都在这里进行，请在尝试扩展 rev.ng 时提出问题。

+   [每月通讯](/newsletter-subscribe)：订阅以参与封闭 beta 测试，并获取项目进展的每月通讯。

+   [电子邮件](mailto:info@rev.ng)：希望保持隐私？给我们发电子邮件。

今天的内容就到这里。敬请关注更多动态！ 🚀
