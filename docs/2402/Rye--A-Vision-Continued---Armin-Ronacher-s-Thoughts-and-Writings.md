<!--yml

category: 未分类

date: 2024-05-27 14:35:35

-->

# Rye: A Vision Continued | Armin Ronacher's Thoughts and Writings

> 来源：[https://lucumr.pocoo.org/2024/2/4/rye-a-vision/](https://lucumr.pocoo.org/2024/2/4/rye-a-vision/)

# Rye: A Vision Continued

written on Sunday, February 4, 2024

去年四月，我向公众发布了[Rye](https://rye-up.com/)。Rye，无论是当时还是现在，代表了我对改进的 Python 打包和项目管理解决方案的非常个人的愿景。它本质上是一种全面的用户体验，设计得让 Python 程序员与之交互的唯一工具就是 Rye 本身，并且它让你从零到一只需一分钟。它能够通过自动下载不同的 Python 版本引导 Python，创建虚拟环境，管理依赖关系，进行代码检查和格式化。最初是为了我自己的使用而开发的，我决定将其发布给公众，反馈非常积极。

当我介绍它时，我开启了一个讨论线程，标题为[“Rye 是否应该存在”](https://github.com/mitsuhiko/rye/discussions/6)，参考了著名的[XKCD #929](https://xkcd.com/927/)，这个漫画幽默地评论了竞争标准的泛滥。我不想再往 Python 打包工具的竞技场里扔另外一个东西。

然而，现在它存在并且有用户。然而，我认为这个标准问题在某种程度上得到了帮助，因为 Rye 本身实际上并不执行这些操作。它包装了已经建立的工具：

+   **下载 Python**：它提供了一个自动化的方式来访问令人惊叹的[Indygreg Python Builds](https://github.com/indygreg/python-build-standalone/)以及 PyPy 二进制发行版。

+   **Linting 和格式化**：它捆绑了[ruff](https://github.com/astral-sh/ruff)，并通过 rye lint 和 rye fmt 提供了这项功能。

+   **管理虚拟环境**：它在幕后使用了广为人知的[virtualenv](https://virtualenv.pypa.io/en/latest/)库。

+   **构建 Wheels**：它主要委托了这项工作给[build](https://pypi.org/project/build/)。

+   **发布**：它的发布命令使用[twine](https://pypi.org/project/twine/)来完成这项任务。

+   **锁定和依赖安装**：今天使用[unearth](https://pypi.org/project/unearth/)和[pip-tools](https://github.com/jazzband/pip-tools/)来实现。

如您所见，Rye并非革命性，也不打算如此。Rye本身并没有做太多事情，因为它将所有核心功能委托给生态系统中的其他工具。Rye以用户友好的方式打包这些工具，显著减少了开发人员的认知负荷。这种便利性消除了学习各种工具、阅读大量文档和独立集成这些组件的需要。Rye使您可以在不到一分钟的时间内从一台计算机上的无Python环境开始，迅速获得一个功能齐全的Python项目，其中包括linting、格式化和一切就绪。它的观点明确，许多重要决策由它代为处理。例如，它从使用pyproject.toml开始，并为您选择了一个wheel构建系统。它还挑选了代码检查器和格式化程序，以及首选的Python分发版本，并确定了构建工具。

## 默认设置很重要

Rye旨在选择最适合工作的工具 — 它选择优胜者。为什么这样做？这种方法的灵感来自于我对Rust生态系统中开发者体验的赞赏，特别是rustup和cargo的无缝集成。它们的功能使我渴望在Python社区中获得类似的体验。在Rust世界中，这种工作方式并不意味着cargo做了所有事情。当你运行cargo build时，它调用rustc；当你运行cargo doc时，它运行rustdoc。当你调用cargo clippy时，它为你运行clippy等等。Cargo是一个管理器，将重要工作委托给由有时完全不同的团队改进的定制工具。这也意味着如果发现某些工具不再是正确选择，可以轻松替换它们。在Rust世界的经验还向我展示，出色的Windows支持是必不可少的。这就是为什么Rye不仅在macOS和Linux上提供出色体验，而且在Windows上也表现出色。

我相信Python社区应该享有卓越的开发者体验，而Rye，就其目前的状态而言，提供了一个有前途的开端。我的信念得到了从面对面用户访谈和演示中收集的证据支持，在那里Rye受到了很好的接待。事实上，每个我能够向其展示Rye的人都对能够迅速开始使用Python印象深刻。由于它明确设计用于避免干扰任何现有Python配置，Rye允许平稳逐步地集成，并显示即使对于使用其他工具的人，接触情感障碍也很低。

话虽如此，Rye 是一个单人项目，并未解决我们在 Python 生态系统中一些问题的根本挑战。它没有解决多版本依赖性，也没有提供更好的依赖安装性能。它也无法帮助分发可执行文件给最终用户或类似的事情。然而，我收到了多个信号表明现在是 Rye 这样的工具出现的时机，不仅仅是存在，还要号召更多 Python 社区的人采纳一些这样的标准化想法。

## 下一步是什么？

[克里斯·沃里克](https://github.com/Kwpolska) 最近撰写了一篇博客文章，回顾了过去一年的 Python 包装，并在 Twitter 上引起了热议。文章有点感叹我们在包装方面没有取得多少进展，并且也谈到了 Rye，并正确指出 Rye 没有足够的贡献者（基本上只有我）。这并不是一个健康的设置。

我仍然不确定 Rye 是否应该存在。它尚未确立，还存在许多不完善之处。我个人非常喜欢使用它，但每次使用时，我都会提醒自己，如果不投入时间，它将停止存在，从某种意义上讲，这正是推动我继续下去的原因。

然而，我希望看到社区收敛到像 Rye 这样的解决方案，无论它来自何处。

## 了解更多

你对此感兴趣吗？如果你愿意试试并提供反馈，我会非常感激：

*一段 16 分钟的 Rye 介绍*

[https://www.youtube.com/embed/q99TYA7LnuA](https://www.youtube.com/embed/q99TYA7LnuA)

视频

这篇文章被标记为 [announcement](/tags/announcement/) 和 [python](/tags/python/)
