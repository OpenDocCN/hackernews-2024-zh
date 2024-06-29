<!--yml

category: 未分类

date: 2024-05-27 13:30:00

-->

# PyPy v7.3.16发布 | PyPy

> 来源：[https://www.pypy.org/posts/2024/04/pypy-v7316-release.html](https://www.pypy.org/posts/2024/04/pypy-v7316-release.html)

## PyPy v7.3.16：发布Python 2.7、3.9和3.10

PyPy团队很荣幸地发布了PyPy的7.3.16版本。

此版本包含来自上游CPython的安全修复，以及垃圾收集器的bug修复，详见[gc bug-hunt博客文章](https://www.pypy.org/posts/2024/03/fixing-bug-incremental-gc.html)。

这次发布包括三种不同的解释器：

> +   PyPy2.7是一个解释器，支持Python 2.7的语法和特性，包括CPython 2.7.18+的标准库（`+`用于后向安全更新）。
> +   
> +   PyPy3.9是一个解释器，支持Python 3.9的语法和特性，包括CPython 3.9.19的标准库。
> +   
> +   PyPy3.10是一个解释器，支持Python 3.10的语法和特性，包括CPython 3.10.14的标准库。

这些解释器基于相同的代码库，因此有多个版本。这是一个微型发布，所有API与其他7.3版本兼容。它紧随2024年1月15日发布的7.3.15版本。

我们建议更新。您可以在这里找到下载v7.3.16版本的链接：

> [https://pypy.org/download.html](https://pypy.org/download.html)

我们要感谢我们的捐赠者对PyPy项目的持续支持。如果PyPy对您的需求不够满足，我们提供[直接咨询](https://www.pypy.org/pypy-sponsors.html)服务。如果PyPy帮助到您了，我们很乐意听到您的反馈，并鼓励通过向我们的[博客](https://pypy.org/blog)提交拉取请求至[https://github.com/pypy/pypy.org](https://github.com/pypy/pypy.org)。

我们还要感谢我们的贡献者，并鼓励新的人加入项目。PyPy有很多层面，我们需要各种帮助：bug修复，[PyPy](index.html)和[RPython](https://rpython.readthedocs.org)文档改进，或者帮助使RPython的JIT变得更好。

如果您是Python库的维护者，并且使用C扩展，请考虑制作一个在PyPy上表现良好的[HPy](https://hpyproject.org/) / [CFFI](https://cffi.readthedocs.io) / [cppyy](https://cppyy.readthedocs.io)版本的库。无论如何，[cibuildwheel](https://github.com/joerick/cibuildwheel)和[multibuild系统](https://github.com/matthew-brett/multibuild)都支持为PyPy构建轮子。

### 什么是PyPy?

PyPy是一个Python解释器，是CPython的即插即用替代品。由于其集成的追踪JIT编译器，它速度很快（查看[PyPy和CPython 3.7.4](https://speed.pypy.org)的性能比较）。

我们也欢迎其他[动态语言的开发者](https://rpython.readthedocs.io/en/latest/examples.html)来看看RPython可以为他们做些什么。

我们为以下提供二进制构建：

> +   **x86** 机器运行大多数常见操作系统（Linux 32/64 位，Mac OS 64 位，Windows 64 位）。
> +   
> +   64 位 **ARM** 机器运行 Linux (`aarch64`).
> +   
> +   Apple **M1 arm64** 机器 (`macos_arm64`).
> +   
> +   **s390x** 运行 Linux

PyPy 支持 Windows 32 位，Linux PPC64 大尾和小尾，以及 Linux ARM 32 位，但不发布二进制版本。如果您希望赞助这些平台的二进制版本发布，请与我们联系。下游打包者为 Debian、Fedora、conda、OpenBSD、FreeBSD、Gentoo 等提供二进制构建。

### 还有什么新功能？

欲了解更多关于 7.3.16 版本的信息，请参阅 [完整变更日志](https://doc.pypy.org/en/latest/release-v7.3.16.html#changelog)。

请更新，并继续帮助我们改进 PyPy。

致敬，PyPy 团队
