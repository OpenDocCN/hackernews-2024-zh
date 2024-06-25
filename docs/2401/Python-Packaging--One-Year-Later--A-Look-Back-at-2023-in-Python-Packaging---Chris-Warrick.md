<!--yml

category: 未分类

date: 2024-05-27 14:50:30

-->

# Python 打包，一年后：回顾 2023 年 Python 打包 | Chris Warrick

> 来源：[https://chriswarrick.com/blog/2024/01/15/python-packaging-one-year-later/](https://chriswarrick.com/blog/2024/01/15/python-packaging-one-year-later/)

一年前，我写了一篇关于 Python 打包状况不佳的文章。领域中有大量的工具，强调编写模糊的标准而不是围绕着一个真正的工具集结，以及复杂的基于 venv 的生态系统而不是类似于 `node_modules` 的解决方案。在过去的一年里发生了什么变化？有什么改进了吗，还是一切仍然如旧，甚至比以前更糟？

## 工具们

[原始帖子](https://chriswarrick.com/blog/2023/01/15/how-to-improve-python-packaging/) 列出了一堆打包工具，称其中至少有十四个工具过多。我的想法是大多数人会满意于一个可以完成所有任务的工具，但科学 Python 用户可能有特殊要求，最好使用第二个工具。

在去年的帖子中提到的工具中，它们似乎都在维护状态。除了 Flit（过去 30 天没有新提交）和 virtualenv（只有自动化和半自动化的版本更新），这些工具都有最新的提交、拉取请求和问题。

所有这些工具仍在使用中。[Françoise Conil 分析了所有 PyPI 包](https://framapiaf.org/@fcodvpt/111540079686191842)，并检查了它们的 PEP 517 构建后端：setuptools 是最受欢迎的（有 50k 个包），Poetry 是第二位，有 41k，Hatchling 是第三位，有 8.1k。其他超过 500 用户的工具包括 Flit（4.4k），PDM（1.3k），Maturin（1.3k，用于 Rust 包的构建后端）。

当然，也有一些新工具。我关注到的工具有[Posy](https://github.com/njsmith/posy)和[Rye](https://github.com/mitsuhiko/rye)。Posy 是由纳撒尼尔·J·史密斯（trio 的作者）开发的项目，Rye 则是由阿尔明·罗纳赫尔（Flask 的作者）开发的项目。它们的愿景都是管理 Python 解释器和项目，但不使用自定义的构建后端（而是使用类似孵化器的东西）。Posy 是建立在 PyBI 之上的（PyBI 是由史密斯在草案 [PEP 711](https://peps.python.org/pep-0711/) 中提出的用于分发 Python 解释器二进制文件的格式），Rye 则使用格雷戈里·索尔克（Gregory Szorc）预构建的 Python。Rye 看起来相当完整和可用，而 Posy 现在只是 PyBI 格式的 PoC，只提供一个带有预安装包的 REPL。

Posy和Rye都是用Rust编写的。一方面，管理Python解释器的部分不是用Python编写是有道理的，因为那将需要一个单独的Python，而不是由Posy/Rye管理的，来运行这些工具。但是Rye也有自己的pyproject.toml解析器，用Rust编写，并且它的许多命令大部分或主要是用Rust实现的（有时也会调用一次性的Python脚本；虽然创建虚拟环境、安装包和处理锁文件的主要任务分别交给了`venv`、`pip`和`pip-tools`）。

谈到Rust和Python，过去一年中还有另一个项目在这方面取得了很大的发展（并集聚了大量的资金）。那个项目是[Ruff](https://github.com/astral-sh/ruff)，它是一个代码规范化工具和代码格式化工具。Ruff可以格式化Python代码，并且是用Rust编写的。这意味着它比现有的用Python编写的工具快10-100倍（根据Ruff自己的基准测试）。快速是好的，我想，但是这对Python意味着什么呢？事实上，包工具（除了快速依赖解决器，可能还包括快速访问Python内部以执行其工作的工具）和代码格式化器（需要深入理解Python语法，并将Python源代码解析为AST，这对于`ast` Python模块来说是很容易的）都用另一种语言编写了？这个趋势是否使Python成为了一种玩具语言（因为它也经常被认为是NumPy和朋友们的“胶水语言”）？此外，为什么贡献给许多Python开发人员重要的工具需要学习Rust呢？

## 标准

上次我们讨论了包装标准，重点是[PEP 582](https://peps.python.org/pep-0582/)。它提出了引入`__pypackages__`的想法，这将是一个地方，第三方包可以在本地按项目安装，而不涉及虚拟环境，类似于`node_modules`对于node来说是什么。该PEP最终于2023年3月[被拒绝了](https://discuss.python.org/t/pep-582-python-local-packages-directory/963/430)。PEP并不完美，其中一些选择是有问题的或者不足的（比如不递归搜索父目录中的`__pypackages__`，或者只关注简单的用例）。迄今为止，没有提出类似的新标准（设计更好）。

另一个有争议的话题是锁文件。包装系统的锁文件对于可重复的依赖安装非常有用。锁文件记录了所有已安装的包（即包含传递依赖项）及其版本。锁文件通常包含已安装包的校验和（如sha512），它们通常支持区分通过不同依赖组（运行时、构建时、可选的、开发等）安装的包。

实现这一目标的经典方式是 `requirements.txt` 文件。它们是针对 pip 的，只包含包、版本和可能的校验和列表。这些文件可以由 `pip freeze` 生成，也可以使用第三方的 `pip-tools` 的 `pip-compile` 生成。`pip freeze` 非常基础，`pip-compile` 无法处理除了生成多个 `requirements.in` 文件、编译它们并希望没有冲突之外的其他依赖组。

Pipenv、Poetry 和 PDM 都有自己的锁定文件实现，彼此不兼容。Rye 基于 `pip-tools` 进行开发。Hatch 的核心没有任何内容；他们正在等待标准实现（尽管有一些插件）。[PEP 665](https://peps.python.org/pep-0665/) 在 2022 年 1 月被[拒绝](https://discuss.python.org/t/pep-665-take-2-a-file-format-to-list-python-dependencies-for-reproducibility-of-an-application/11736/140)。其作者 Brett Cannon [正在研究一个可能成为标准的 PoC](https://snarky.ca/state-of-standardized-lock-files-for-python-august-2023/)（名为 [mousebender](https://github.com/brettcannon/mousebender)）。

这就是 Python 包装世界采用的工作模型的危险所在。即使是像锁文件这样简单的事情，也至少有四种不兼容的标准。由于“冷淡的反应”，一个规范的尝试被拒绝了，尽管至少有四种实现达到了大致相同的目标，其他生态系统也经历过这样的情况。

Python 中另一个重要的事情是扩展模块。扩展模块是用 C 编写的，它们通常用于与其他语言编写的库进行交互（有时也用于提高性能）。Poetry、PDM 和 Hatchling 实际上不支持构建扩展模块。Setuptools 支持；[SciPy 和 NumPy 从他们的自定义 numpy.distutils 迁移到了 Meson](https://numpy.org/doc/stable/reference/distutils_status_migration.html)。负责 Python 的 PyO3 Rust 绑定的团队开发了 [Maturin](https://github.com/PyO3/maturin)，它允许构建基于 Rust 的扩展模块 — 但如果你使用的是 C，这并不有用。

在 2023 年没有太多关于包装的标准被接受。值得一提的标准是[PEP 668](https://peps.python.org/pep-0668/)，它允许发行商通过添加一个 `EXTERNALLY-MANAGED` 文件来阻止 pip 的工作（以避免破坏发行版拥有的站点包），该标准于 2022 年 6 月被接受，但 pip 直到 2023 年 1 月才实现了对其的支持，许多发行版在 2023 年已经启用了此功能。防止系统崩溃是一件好事。

但是一些标准确实通过了。除了次要和小的标准之外，最突出的 2023 年标准应该是 [PEP 723](https://peps.python.org/pep-0723/)：内联脚本元数据。它允许在文件顶部添加注释块，以一种可以被工具消费的方式指定依赖项和最低的 Python 版本。这非常有用吗？我不这么认为；使用 pyproject.toml 设置项目很容易就能让事情变得更加复杂。如果你通过 GitHub gist 发送某些东西，只需创建一个仓库。如果你通过电子邮件发送某些东西，只需将文件夹打包。这种方法促进了没有源代码控制的混乱编程。

### 学习曲线与“简单”的欺骗

Microsoft Word 很简单，是一个很好的初学者写作工具。你可以用一次点击将文本变粗。你也可以用两次点击将其变成蓝色。但很容易造成不一致的混乱。为了制作章节标题，许多用户可能只是将文本加粗并稍微放大一点，没有任何一致性或语义。在 Word 中制作一致的具有语义格式的文档很难。添加 [节编号](https://www.techrepublic.com/article/how-to-create-multilevel-numbered-headings-in-word-2016/) 需要你选择一个标题并将其变成列表。也有一些所谓的魔法参与其中，那种魔法对我不起作用，我必须告诉 Word 更新标题样式。即使你试图做事情的时候很好，Word 也会随机出错，破坏样式，混合样式和行内临时格式化，你的文档在不同的计算机上可能会看起来不同。

LaTeX 对初学者来说非常令人困惑，有着巨大的学习曲线。你肯定可以在各处写 `\textbf{hello}`。但是通过一些学习，你将能够制作出漂亮的文档。你将定义一个 `\code{}` 命令，使代码等宽并添加边框，但如果你愿意的话，明天它可能会改变背景并以 Comic Sans 字体排版。你将使用能够从外部文件渲染带有语法突出显示的代码的包。标题编号默认是开启的，但可以轻松禁用某个章节的编号。LaTeX 还可以自动将新的章节放在新的页面上，例如。LaTeX 是为科学出版而建立的，因此在数学和参考文献等方面具有出色的支持。

让我们现在谈论一下编程。Python 很简单，是一个很好的初学者编程语言。你可以用一行代码写出 *hello world*。语法更简单，没有令人困惑的 C 语言遗留问题（比如基于索引的 `for` 循环）或者机器级代码（比如 `switch` 中的 `break`），看不到任何指针。你也不需要完全写类；你不需要写一个类只是为了把 `public static void main(String[] args)` 方法放在那里。你不需要一个 IDE，你可以用任何编辑器（甚至 notepad.exe）来写代码，你可以把它保存为 .py 文件，并使用 `python whatever.py` 运行它。

你的代码变得更复杂了？别担心，你可以将其拆分成多个`.py`文件，使用`import name_of_other_file_without_py`，它就会正常工作。你需要更多的结构，也许是分组到文件夹中吗？嗯，忘记`python whatever.py`吧，你必须使用`python -m whatever`，并且你必须`cd`到你的代码所在的位置，或者搞定`PYTHONPATH`，或者使用`pip`安装你的东西。这个简单但常见的操作（将东西分组到文件夹中）大大增加了复杂性。

标准库不够用，你需要第三方依赖？你找到了一些教程告诉你要`pip install`，但现在`pip`会告诉你使用`apt`。`apt`可能有效，但它可能给你一个与你正在阅读的教程不匹配的古老版本。或者它可能没有这个软件包。或者网络会告诉你不要从`apt`使用Python软件包。所以现在你需要[了解关于虚拟环境的知识](https://chriswarrick.com/blog/2018/09/04/python-virtual-environments/)（这增加了更多的复杂性，需要记住更多的事情；大多数教程都教如何激活，但通过基本操作如重命名文件夹可能会弄乱venvs，并且你可能会在git中得到一个venv或在venv中得到你的代码）。或者你需要选择众多的一站式工具来管理事务。

在其他生态系统中，IDE通常是必不可少的，即使是对初学者也是如此。IDE将强制你使用项目系统（也许默认情况下不是最好或最常见的系统，但它仍将是一个连贯的项目系统）。Java将强制你使用“1个公共类 = 1个文件”的规则创建多个文件，并且这样做很容易，你甚至不需要一个`import`。

你想要文件夹吗？在Java或C#中，你只需在IDE中创建一个文件夹，然后在那里创建一个类。新文件可能有一个不同的`package`/`namespace`，但IDE将帮助你向代码库添加正确的`import`/`using`，而且你不会冒用太多目录（包括像`src`这样的东西）或使用太少（不为所有代码创建一个顶级包）的风险，这将需要修正所有导入。在Java或C#中添加文件夹的干扰是最小的。

项目系统还会处理第三方软件包，而无需考虑它们被下载到哪里或什么是虚拟环境以及如何从不同的上下文激活它们。点击几下，就完成了。如果你不喜欢IDE呢？在许多生态系统中，生活在CLI中当然是可能的，它们有合理的CLI工具用于常见的管理任务，以及构建和运行项目。

PEP 723 解决了一个非常特定的问题：单文件程序的依赖管理。对于一次性的事情和混乱的代码，改善生活显然比为大型项目做任何其他改进更重要对于打包社区来说。

顺便说一句，你可以将这个教训应用到静态和动态类型上。动态类型更容易上手，打字更少，但编译类型检查可以防止许多错误——这些错误需要更高的测试覆盖率才能通过动态类型进行捕获。这就是为什么 JS 世界有 TypeScript，这也是为什么 mypy/pyright/typing 在 Python 世界中赢得了很多关注度。

## 未来…

查看[Python Packaging Discourse](https://discuss.python.org/c/packaging/14)，有一些关于改进事物的讨论。

例如，这个[关于迁移setup.py的讨论](https://discuss.python.org/t/user-experience-with-porting-off-setup-py/37502)是由Gregory Szorc发起的，他有[一长串抱怨](https://gregoryszorc.com/blog/2023/10/30/my-user-experience-porting-off-setup.py/)，指出了包装世界中的沟通问题和文档混乱（他的帖子值得一读，或者至少浏览一下，因为它很长，而且充满了包装失败）。有一个页面推荐使用setuptools，另一个页面有四个选项，Hatchling作为默认选项，还有一个页面推广Pipenv。一年前我们已经看到了这一点，在这方面没有任何变化。一些人尝试找到解决方案，一些人分享了他们的观点……然后论坛的管理员决定保护他的 PyPA 朋友，不让他们阅读用户反馈，并锁定了主题。

还有许多关于愿景的其他讨论，比如关于[10年观点](https://discuss.python.org/t/the-10-year-view-on-python-packaging-whats-yours/31834)或[单一包装工具](https://discuss.python.org/t/wanting-a-singular-packaging-tool-vision/21141)。基于用户调查的策略讨论有[第二部分](https://discuss.python.org/t/python-packaging-strategy-discussion-part-2/23442)（[第一部分](https://discuss.python.org/t/python-packaging-strategy-discussion-part-1/22420)于2023年1月结束），但比第一部分少了很多帖子，讨论也没有继续（还有[关于如何进行讨论的讨论](https://discuss.python.org/t/structure-of-the-packaging-strategy-discussions/23478)）。计划[创建一个包装委员会](https://discuss.python.org/t/draft-update-to-python-packaging-governance/31608)——设计委员会的最佳实践。

但是所有这些讨论，即使没有被过度热心的版主锁定，也没有产生任何实质性的影响。打包生态系统仍然严重分散和令人困惑。[PyPA文档和教程](https://packaging.python.org/en/latest/tutorials/) 仍然彼此矛盾。与PyPA关联的工具仍然比未关联的竞争对手功能更少（甚至新秀Rye也有某种形式的锁文件，不像Hatch或Flit），并且根据PEP 517构建后端使用统计数据，它们比现代PyPA工具更受欢迎。类似但竞争的工具的作者没有联合起来制作出真正的打包工具。

### …看起来相当黯淡

另一方面，如果您查看大多数打包工具的2023年贡献图表，您可能会担心打包生态系统的状态。

+   [Pip](https://github.com/pypa/pip/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c) 的贡献者结构健康，有大量提交。

+   [Pipenv](https://github.com/pypa/pipenv/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c) 和 [setuptools](https://github.com/pypa/setuptools/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c) 有两位主要提交者，但仍然有大量提交。

+   [Hatch](https://github.com/pypa/hatch/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c)，然而，是一个**单人表演**：Ofek Lev（项目创始人）有184次提交，第二名是Dependabot有6次提交，第三名（一个人类贡献者）有5次提交。Hatch和Hatchling的巴士因子为1。

非PyPA工具的情况并不好：

+   [Poetry](https://github.com/python-poetry/poetry/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c) 有两位主要贡献者，但至少有四位人类贡献者的提交数量是两位数。

+   [PDM](https://github.com/pdm-project/pdm/graphs/contributors?from=2023-01-01&to=2023-12-31&type=c) 是像Hatch一样的单人表演。

+   [Rye](https://github.com/mitsuhiko/rye/graphs/contributors?from=2023-04-23&to=2023-12-31&type=c) 有一位主要贡献者，以及三位提交数量是两位数的贡献者；注意它相当新（于2023年4月下旬开始），并且不如其他工具那样受欢迎。

## 结论

我理解 PyPA 是一个由志愿者组成的松散协会。有时人们说 *Python Packaging Authority* 这个名字最初是一个笑话。然而，他们也是维护所有打包标准的组织，所以在打包方面他们 *是* 权威的。例如，[PEP 668](https://peps.python.org/pep-0668/) 以一个警告块开头，表示这是一个历史性文件，并且[规范的最新版本在 PyPA 的网站上](https://packaging.python.org/en/latest/specifications/externally-managed-environments/)（以及一堆其他[打包规范](https://packaging.python.org/en/latest/specifications/)）。

**PyPA 应该关闭或合并一些重复的项目，并与社区合作（包括非 PyPA 项目的维护者）共同构建一个真正的打包工具。** 为了让事情更简单。为了避免编写大部分功能相同的代码。为了确保成千上万的项目不依赖于只有 1 或 2 个关键人员的工具。将打包从一个难以解决的障碍转变为 *只是工作™*，是一个普通开发者不需要考虑的东西。

这不是火箭科学。许多语言，大大小小，都有一个连贯的打包生态系统（只需阅读[去年的帖子](https://chriswarrick.com/blog/2023/01/15/how-to-improve-python-packaging/)，了解一些它可以多么简单）。与其专注于规范和治理，不如专注于制作一个全面的、可用的、用户友好的工具。

在下面讨论或者[在 Hacker News 上讨论](https://news.ycombinator.com/item?id=39004600)。
