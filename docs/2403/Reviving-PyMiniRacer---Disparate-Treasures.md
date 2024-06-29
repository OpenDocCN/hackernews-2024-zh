<!--yml

category: 未分类

date: 2024-05-29 12:36:07

-->

# 复苏PyMiniRacer - 不同的宝藏

> 来源：[https://bpcreech.com/post/mini-racer/](https://bpcreech.com/post/mini-racer/)

在[最后一篇博客文章](https://bpcreech.com/post/python-nodejs-eval/)中，我创建了一个[助手](https://pypi.org/project/nodejs-eval/)，使用NodeJS边车进程从Python调用JavaScript。在文章中，我[评论道](https://bpcreech.com/post/python-nodejs-eval/#alternatives-considered)，*在*进程中进行JS评估可能会更好。旧的[Sqreen `PyMiniRacer`项目](https://github.com/sqreen/PyMiniRacer)最近*才*开始失修。我们能复活它吗？

**简而言之：是的！**只需几周的努力和新的所有权（就是我）。现在我拥有从Python程序运行JavaScript的*两*种最佳方式：[`PyMiniRacer`](https://github.com/bpcreech/PyMiniRacer)和[`nodejs-eval`](https://github.com/bpcreech/nodejs-eval)。

```
from py_mini_racer import MiniRacer   mr = MiniRacer()   # Let's run some JavaScript from Python! >>> mr.eval("Math.pow(7, 3);") 343   # Updated, for the first time since 2021! >>> mr.v8_version "b'12.2.281.23'"   # Now supported: the Intl API! >>> mr.eval('Intl.DateTimeFormat(["ban", "id"]).format(new Date())') '16/3/2024'   # As of the v0.9.0 release, async execution (as a side effect, Control+C works): >>> mr.eval('while (1) {}') ^CTraceback (most recent call last): ... KeyboardInterrupt 
```

*其他新功能可以在[发布说明页面上找到](https://bpcreech.com/PyMiniRacer/history/)，v0.7.0是自2021年以来的第一个新版本。*

## 一个血统[](#a-lineage)

1.  **[`therubyracer`](https://github.com/rubyjs/therubyracer) (2009-2018)** [查尔斯·洛厄尔](https://github.com/cowboyd)，来自[Frontside Software](https://frontside.com/)，创建了[The Ruby Racer](https://github.com/rubyjs/therubyracer)，用于将[Chrome](https://v8.dev/)、NodeJS等使用的JavaScript引擎[V8](https://v8.dev/)直接嵌入[Ruby](https://www.ruby-lang.org/)，以便从Ruby程序中执行JS代码。

    +   不幸的是，Ruby Racer与V8之间的深度集成成为Ruby Racer的痛点，因为升级V8通常意味着重新调整Ruby Racer以适应接口变化。因此，这个项目最终被存档，并被&mldr;

1.  **[`mini_racer`](https://github.com/rubyjs/mini_racer) (2016-)** [山姆·萨弗隆](https://github.com/SamSaffron)和其他人创建了[`mini_racer`](https://github.com/rubyjs/mini_racer)，一个新的Ruby/V8集成版本，相对于Ruby Racer精简了许多。此版本仍在维护中。

1.  **[`sqreen/PyMiniRacer`](https://github.com/sqreen/PyMiniRacer) (2016-2021)** [Sqreen](https://github.com/sqreen)，一家网络应用安全初创公司，创建了[`PyMiniRacer`](https://github.com/sqreen/PyMiniRacer)，这是一个Python模块，仿照了Ruby的`mini_racer`。这与V8的接口最小化模型相似，并使用Python的`ctypes`集成（而不是Python扩展模块），进一步减少了与*Python*的接口，从而实现了JS/Python集成，而支持负担相对较少。

1.  **[`bpcreech/PyMiniRacer`](https://github.com/bpcreech/PyMiniRacer) (2024-)** 这就是你现在正在阅读的内容。 :)

在与Sqreen（现为DataDog）的讨论后，我们决定将我对他们的`PyMiniRacer`项目的复苏作为分支进行托管，位于这里：

## 一般更新[](#general-updates)

除了升级V8——这有自己的小节——我还有机会整理这个项目的各个部分。

### Python 生态系统更新[](#python-ecosystem-updates)

特别是，在 Python 的世界里发生了许多事情！

#### Python 版本（删除 Python 2，添加到 3.12）[](#python-versions-drop-python-2-add-up-to-312)

首先，我们可以放弃 Python 2，它在 [全球于 2020 年达到了 EOL](https://www.python.org/doc/sunset-python-2/)（在超过十年的逐步弃用计划之后！）。虽然世界很大，仍然有人在使用 Python 2，但我们不需要为他们维护最新的 V8 集成。

同时，截至本文撰写时，Python 已经更新到 3.12，在 `PyMiniRacer` 中增加了一些次要的破坏。例如，Python 的 `memoryview` 系统现在要求显式的 `memoryview` 强制转换。

我还添加了对新的 Python [`importlib.resources`](https://docs.python.org/3/library/importlib.resources.html) 规范的支持，这使得 Python 可以直接运行模块并加载它们的依赖，例如 `PyMiniRacer` 的 DLL 文件，从不寻常的非文件系统位置（例如 `zip` 文件内）加载数据文件。这一系统在 Python 3.7 中添加，Python 3.9 中进行了重新设计；`PyMiniRacer` 现在支持从包中加载数据文件的两种实现方式。

#### 使用 Hatch 打包[](#packaging-with-hatch)

`PyMiniRacer` 最初使用 `setuptools` 构建二进制分发，并使用手工编写的 `Makefile` 管理其各种自动化任务。Python 现在有了一个标准化的可插拔打包系统来构建二进制分发，[Hatch](https://github.com/pypa/hatch) 是其中最流行的实现。根据 [其作者](https://github.com/ofek) 的说法，*“Hatch 试图成为 Python 的 Cargo 或 Go CLI 的等价物”*。通过使用 Hatch（并接受其各种观点），我们可以删除 `PyMiniRacer` 中的大量开发工具配置。

Hatch 包含一个捆绑的、有意见的代码检查工具和代码格式化器 [Ruff](https://github.com/astral-sh/ruff)，它让我们可以删除开发依赖项 [`flake8`](https://flake8.pycqa.org/en/latest/) 和 [`isort`](https://pycqa.github.io/isort/) 以及它们的配置文件。我唯一改变的默认设置是行长度，从 120 改为 Black 的默认值 88。我认为这场毫无意义的辩论 [最终被 Black 解决](https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html#line-length)，但由于某些原因，Hatch [覆盖了这个设置到 120](https://github.com/pypa/hatch/discussions/1117)，所以我将其恢复到 Black（和 Ruff）的默认值。

Hatch还包含了Python版本矩阵测试的内置支持。它的表现非常好（但在[Alpine](https://www.alpinelinux.org/)上不行，原因见[此处](https://github.com/indygreg/python-build-standalone/issues/86)），并且让我们可以放弃[`tox`](https://tox.wiki/en/4.14.1/)作为开发依赖。

### `pytest`代替`unittest`[](#pytest-instead-of-unittest)

如今这更像是一种无脑的选择。`unittest`从Python一开始就内置了，但如今每个人（至少在开源社区中是这样）似乎都在转向`pytest`。因此，我将所有测试转换为`pytest`，这使得测试看起来稍微简单些，而且控制台输出更漂亮，赞！

### 文档！[](#docs)

受[这篇文章](https://matklad.github.io//2021/02/06/ARCHITECTURE.md.html)的启发，我觉得我们应该有一个`ARCHITECTURE.md`，所以[我写了一个](https://github.com/bpcreech/PyMiniRacer/blob/main/ARCHITECTURE.md)（[*或者在`mkdocs`网站上查看*](https://bpcreech.com/PyMiniRacer/architecture/)）。

我还添加了大量注释。`PyMiniRacer`的V8构建（见下文）充满了配置调整、补丁和一些额外步骤的解决方案。这些解决方案*并不*有助于向前兼容性，因为每个构建过程的小调整都可能成为未来当上游V8构建过程发生变化时的破坏源。现在，至少我们有一个记录这些调整来源的记录！

最后，从外观上最为显著的变化，我将`PyMiniRacer`的文档从[Sphinx](https://www.sphinx-doc.org/en/master/)迁移到[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)，并创建了一个文档构建流水线（根据我所知，之前没有！）。Sphinx一直在那里并且一直在工作，但当前的生态系统关注度似乎最近都集中在`mkdocs-material`上。

我有点担心可维护性，因为`mkdocs-material`是`mkdocs`的一个复杂且承载重要功能的*插件*，而且只有当与[`mkdocs-material`自身的*其他*插件系统](https://squidfunk.github.io/mkdocs-material/setup/extensions/)结合使用时才能发挥最佳效果。这种设置容易导致[这种情况发生](https://xkcd.com/2347/)。但是，受同行的压力，我还是选择了它，新的文档配置非常简单，看起来很棒，因为`mkdocs-material`确实很出色。新的文档位于[这里](https://bpcreech.com/PyMiniRacer)。

## 实际上在构建V8[](#actually-building-v8)

好的，主要工作在于更新V8\. 最后一个`PyMiniRacer`的Sqreen版本，从2021年开始使用的V8 8.9，现在已经无法构建。而现在V8版本已经更新到12.2了。

### 在构建V8方面的一般挑战[](#general-challenges-in-building-v8)

V8 库没有官方的独立二进制分发版本。使用 V8 的唯一方法是自行构建。*除非，也许，您使用整个 NodeJS 二进制文件，这让我们回到了[我的最新博客文章](https://bpcreech.com/post/python-nodejs-eval/)——也许，毕竟，使用 V8 的最佳方式是通过在 NodeJS 内运行的服务器？*

但是构建 V8 很难！在构建 V8 中的有趣挑战：

1.  **V8 是庞大的，构建过程缓慢：** V8 包含 *或者动态下载* 6.6 GB 的 *源代码*！（好吧，并非全部是“源代码”，实际上包括某些操作系统根目录的供应商副本，比如来自各种 Debian 发行版的 `/usr`。）大约有 2.4k 个构建步骤，其中包括构建由 [Torque 编译器](https://v8.dev/docs/torque) 生成的大量代码。在 [免费的 GitHub Actions 托管器](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners) 上，从头构建需要超过一个小时（目前是一个适用于 Linux 等的 4-CPU 机器）。为了构建适用于 Linux `aarch64`，GitHub 没有提供免费的托管器，所以我们通过 [模拟](https://github.com/uraimo/run-on-arch-action) 运行，并且需要 *数天* 的时间。这远远超过了 [GitHub Actions 提供的最长 6 小时的限制](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#usage-limits)，因此由于时间限制，构建会失败。我们可以通过使用 [`sccache`](https://github.com/mozilla/sccache) 缓存并在足够多次重试后最终成功使我们的 GitHub Actions 构建 *确实* 成功。（希望有一天，GitHub 会提供免费的 `aarch64` Linux 托管器！）

1.  **V8 想要建立自己的构建生态系统：** 要 [构建 V8](https://v8.dev/docs/build)，首先要 [下载另一组实用程序](https://v8.dev/docs/source-code) 称为 [`depot_tools`](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)。`depot_tools` 包括为一些但不是所有目标平台构建的自有二进制文件，用于诸如 Python、[Goma](https://chromium.googlesource.com/infra/goma/client/)（我们未使用的构建缓存）、[Ninja](https://ninja-build.org/)（我们 *确实* 使用的构建系统）、[GN](https://github.com/o-lim/generate-ninja)（我们也使用的元构建系统）等。`depot_tools` 的 `fetch` 工具充当递归依赖模块抓取器（类似 Git 子模块，但更复杂）。一旦我们获取了所有源代码，V8 使用一系列 Python 脚本包装 GN，而 GN 又包装 Ninja。

1.  **构建生态系统，以及总体上的构建，实际上不适用于 *Alpine 或者 Linux-on-Arm*：** 对于 `PyMiniRacer`，我们希望至少针对 `{ Windows, Mac, Linux [glibc], Linux [musl] } × { x86_64, aarch64 }` 进行目标设置（`aarch64` 是 [受欢迎的需求](https://github.com/sqreen/PyMiniRacer/issues/154)）。尽管 V8 不支持在 Linux-on-arm64 上进行构建，但它支持用于 *交叉编译*。V8 不支持 `musl`（Alpine 的 `libc`），无论是在主机上构建 *还是* 交叉编译。因此，我们需要进行各种有趣的配置调整，以确保其实际可用性。

1.  **V8 和构建系统经常变化：** V8 在 Google 进行大规模开发，用于多种产品（当然包括 Chromium，还有 ChromeOS 等）。可用的和默认的配置选项会随时间变化，这意味着我们在 `PyMiniRacer` 中进行的任何复杂构建设置可能会随着新的 V8 版本而破坏。因此，我们希望 *尽量减少* 在 `PyMiniRacer` 中的构建配置量，以尽可能未来证明其耐用性。

1.  **V8 需要一个尖端的 LLVM（特别是 `clang`），并且需要自己的 `libstdc++`：** V8 使用 `clang` 的全新功能，包括基于机器学习的优化模型。它附带了 llvm 工具链的构建，但仅适用于支持的平台（因此不包括 Alpine，也不包括在 Linux `aarch64` 上进行构建）。我们通过安装来自 [LLVM 项目](https://llvm.org/) 的最新版本来解决这个问题，其中 V8 自身供应的二进制文件不能正常工作。但即使是这个版本对于 V8 来说也不够新！我们 *仍然* 必须调整构建配置，以确保即使是最新的 *稳定* LLVM 也能成功构建。

与此同时，我们通过使用免费的 GitHub Actions 运行器面临另一个挑战：不幸的是，GitHub Actions 没有原生的 `aarch64` 托管运行器，也没有原生的 Alpine 运行器。我们通过使用 Umberto Raimondi 出色的 [`run-on-arch-action`](https://github.com/uraimo/run-on-arch-action) 来解决这个问题。这个 GitHub Actions 插件帮助我们在 Linux 发行版和架构中构建 Docker 容器，然后在那里构建 `PyMiniRacer`。

除了 [从 v8.9 到 v12.2 的所有 V8 更新](https://chromium.googlesource.com/v8/v8/+log/branch-heads/8.9..branch-heads/12.2/?n=1000)，我还添加了以下内容，这些内容在之前的 `PyMiniRacer` 构建中已被禁用：

这两者都需要将生成的数据文件与编译后的 DLL 放入 Python 包中。

### 潜在的未来工作是简化 V8 构建[](#potential-future-work-in-simplifying-the-v8-build)

前面提到的 Ruby 项目 [`mini_racer`](https://github.com/rubyjs/mini_racer) 实际上将 V8 的构建拆分为一个独立的项目，[`libv8-node`](https://github.com/rubyjs/libv8-node)：“*一个项目，用于以源代码和二进制形式打包 v8 运行时库和头文件，作为一种语言无关的 zip 和 Ruby gem 发布*”。该项目通过重用 NodeJS 的默认供应商副本和构建 V8 的方法来解决这个问题，而不是按照[谷歌的指导](https://v8.dev/docs/build)构建 V8。我们*可能*可以通过将 `PyMiniRacer` 重新基于 `libv8-node` 的 V8 构建来简化。我们需要确保 `libv8-node` 是最新且稳定的（我不完全清楚它是否是），并且由于我们完全放弃了 V8 的构建，将 [`PyMiniRacer` 的自定义 C++ 代码的编译](https://github.com/bpcreech/PyMiniRacer/tree/main/src/v8_py_frontend)从 `GN`+`ninja` 转移到另一个待确定的构建系统中。

另外，如果 V8 能够与通用的 C/C++ 包系统兼容，那将是一件好事。如今，跨平台的获胜者似乎是 [https://conan.io](https://conan.io)。使 V8 与 Conan 兼容（足以正式上传到 [Conan Center](https://conan.io/center)）将会很困难，因为除了 [上述所有的困难](#general-challenges-in-building-v8)外，V8 还喜欢下载自己的依赖项，这违反了 Conan 的常识性“一次定义原则”（ODR）。

## `PyMiniRacer` 中的其他未来工作[](#other-future-work-in-pyminiracer)

未来的工作可能包括：

+   对新的 V8 版本的更新可以假定会不断出现。

+   对 [Python `asyncio`](https://docs.python.org/3/library/asyncio.html) 的支持。

+   来自[旧 GitHub 问题列表](https://github.com/sqreen/PyMiniRacer/issues)的其他内容。

+   标准库内容。`PyMiniRacer` 没有 `console.log`（也没有 `window` 对象可以存放 `console`），没有 `setTimeout` 等。提供这些函数会很方便，但如果我们不小心可能会违反 `PyMiniRacer` 提供的安全沙箱，会偏离我们所追求的最小接口规则，并可能趋向于“只是成为 NodeJS”，其具有[丰富的标准库](https://nodejs.org/docs/latest-v12.x/api/)。此时，我们最好是嵌入 NodeJS，或者仅[作为旁路运行](https://pypi.org/project/nodejs-eval/)。

如果您正在阅读此内容并希望做出贡献，请尽管去做！参见[贡献指南](https://bpcreech.com/PyMiniRacer/contributing/)。
