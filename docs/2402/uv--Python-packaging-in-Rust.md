<!--yml

category: 未分类

date: 2024-05-27 14:52:40

-->

# uv: Python packaging in Rust

> 来源：[https://astral.sh/blog/uv](https://astral.sh/blog/uv)

**TL;DR:** [uv](https://github.com/astral-sh/uv) 是一个**极快的 Python 包安装和解析器**，用 Rust 编写，旨在成为 `pip` 和 `pip-tools` 工作流的可替代品。

[uv](https://github.com/astral-sh/uv) 代表了我们在追求["Python 的 Cargo"](https://blog.rust-lang.org/2016/05/05/cargo-pillars.html#pillars-of-cargo)过程中的一个里程碑：一个全面的 Python 项目和包管理器，快速、可靠且易于使用。

作为本次发布的一部分，我们还将接管 [Rye](https://github.com/mitsuhiko/rye)，这是一个来自 [Armin Ronacher](https://github.com/mitsuhiko) 的实验性 Python 包装工具。我们将继续维护 [Rye](https://github.com/mitsuhiko/rye)，同时将 [uv](https://github.com/astral-sh/uv) 扩展为统一的后续项目，以实现我们对 Python 包装的 [共同愿景](https://rye-up.com/philosophy/)。

* * *

Astral 在 Python 生态系统中构建高性能开发者工具。我们以 [Ruff](https://github.com/astral-sh/ruff) 著称，这是一个极快的 Python [linter](https://notes.crmarsh.com/python-tooling-could-be-much-much-faster) 和 [formatter](https://astral.sh/blog/the-ruff-formatter)。

今天，我们发布了 Astral 工具链中的下一个工具：**[uv](https://github.com/astral-sh/uv)，一个极快的 Python 包解析和安装器，用 Rust 编写**。

<svg xmlns="http://www.w3.org/2000/svg" 

viewBox="0 0 422 250"><g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-frame uv-warm-vertical-benchmark_svg__root" 

aria-roledescription="group mark container" 

fill="none" 

stroke-miterlimit="10"><g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-scope uv-warm-vertical-benchmark_svg__cell" 

aria-roledescription="group mark container"><g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-axis" 

aria-roledescription="axis" 

aria-label="X 轴为线性刻度，从 0.0 到 3.5"><g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="middle"

transform="translate(90.5 105.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0s</text> <text text-anchor="middle" 

transform="translate(176.214 105.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">1s</text> <text text-anchor="middle" 

transform="translate(261.929 105.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">2s</text> <text text-anchor="middle" 

transform="translate(347.643 105.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">3s</text></g></g> <g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-axis"

aria-roledescription="axis"

aria-label="Y-axis for a discrete scale with 4 values: 

uv, poetry, pip-compile, pdm"><g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="end"

transform="translate(80.5 15.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel"

font-weight="bold">uv</text> <text text-anchor="end"

transform="translate(80.5 37.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">poetry</text> <text text-anchor="end"

transform="translate(80.5 60.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">pip-compile</text> <text text-anchor="end"

transform="translate(80.5 82.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">pdm</text></g></g> <g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-mark uv-warm-vertical-benchmark_svg__child_layer_1_marks"

aria-roledescription="text mark container"><text aria-label="Sum of time: 

0.60278702674;

tool:

poetry;

timeFormat:

0.60s"

aria-roledescription="text mark"

transform="translate(147.667 37.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0.60s</text> <text aria-label="Sum of time: 

1.55616658094;

tool:

pip-compile;

timeFormat:

1.56s"

aria-roledescription="text mark"

transform="translate(229.386 60.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">1.56s</text> <text aria-label="Sum of time: 

3.37404433084;

tool:

pdm;

timeFormat:

3.37s"

aria-roledescription="text mark"

transform="translate(385.204 82.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">3.37s</text></g> <g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-mark uv-warm-vertical-benchmark_svg__child_layer_2_marks"

aria-roledescription="text mark container"><text aria-label="Sum of time: 

0.0134756369786;

tool:

uv;

timeFormat: 

0.01s"

aria-roledescription="text mark"

transform="translate(97.155 15.25)"

font-family="Roboto Mono,monospace"

font-size="12"

font-weight="bold"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0.01s</text></g> <g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-axis"

aria-roledescription="axis"

aria-label="X-axis for a linear scale with values from 0 to 5"><g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="middle" 

transform="translate(90.5 248.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0s</text> <text text-anchor="middle" 

transform="translate(210.5 248.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">2s</text> <text text-anchor="middle" 

transform="translate(330.5 248.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">4s</text></g></g> <g class="uv-warm-vertical-benchmark_svg__mark-group uv-warm-vertical-benchmark_svg__role-axis" 

aria-roledescription="axis" 

aria-label="Y-axis for a discrete scale with 4 values: 

uv, poetry, pdm, pip-sync"><g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="end" 

transform="translate(80.5 158.25)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel" 

font-weight="bold">uv</text> <text text-anchor="end" 

transform="translate(80.5 180.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">poetry</text> <text text-anchor="end" 

transform="translate(80.5 203.25)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">pdm</text> <text text-anchor="end" 

transform="translate(80.5 225.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">pip-sync</text></g></g> <g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-mark uv-warm-vertical-benchmark_svg__child_layer_1_marks" 

aria-roledescription="text mark container"><text aria-label="Sum of time:

0.9872183659; 

tool: 

poetry; 

timeFormat: 

0.99s" 

aria-roledescription="text mark" 

transform="translate(155.233 180.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0.99s</text> <text aria-label="Sum of time: 

1.8969612492; 

tool: 

pdm; 

timeFormat: 

1.90s" 

aria-roledescription="text mark" 

transform="translate(209.818 203.25)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">1.90s</text> <text aria-label="Sum of time: 

4.6313483826; 

tool: 

pip-sync; 

timeFormat: 

4.63s" 

aria-roledescription="text mark" 

transform="translate(373.88 225.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">4.63s</text></g> <g class="uv-warm-vertical-benchmark_svg__mark-text uv-warm-vertical-benchmark_svg__role-mark uv-warm-vertical-benchmark_svg__child_layer_2_marks" 

aria-roledescription="text mark container"><text aria-label="Sum of time: 

0.0576289908; 

tool: 

uv; 

timeFormat: 

0.06s" 

aria-roledescription="text mark" 

transform="translate(99.458 158.25)"

font-family="Roboto Mono,monospace"

font-size="12"

font-weight="bold"

class="uv-warm-vertical-benchmark_svg__benchmarkLabel">0.06s</text></g></g></g></svg><svg xmlns="http://www.w3.org/2000/svg"

viewBox="0 0 454 139"><g class="uv-resolve-warm-benchmark_svg__mark-group uv-resolve-warm-benchmark_svg__role-frame uv-resolve-warm-benchmark_svg__root"

aria-roledescription="分组标记容器"

fill="none"

stroke-miterlimit="10"><g class="uv-resolve-warm-benchmark_svg__mark-group uv-resolve-warm-benchmark_svg__role-axis"

aria-roledescription="轴"

aria-label="一个包含从0.0到3.5值的线性刻度的X轴"><g class="uv-resolve-warm-benchmark_svg__mark-text uv-resolve-warm-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="middle"

transform="translate(106.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">0s</text> <text text-anchor="middle"

transform="translate(192.214 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">1s</text> <text text-anchor="middle"

transform="translate(277.929 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">2s</text> <text text-anchor="middle"

transform="translate(363.643 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">3s</text></g></g> <g class="uv-resolve-warm-benchmark_svg__mark-group uv-resolve-warm-benchmark_svg__role-axis"

aria-roledescription="轴"

aria-label="一个包含4个值的离散刻度的Y轴：

uv, poetry, pip-compile, pdm"><g class="uv-resolve-warm-benchmark_svg__mark-text uv-resolve-warm-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="end"

transform="translate(96.5 31.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel"

font-weight="bold">uv</text> <text text-anchor="end"

transform="translate(96.5 53.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">poetry</text> <text text-anchor="end"

transform="translate(96.5 76.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">pip-compile</text> <text text-anchor="end"

transform="translate(96.5 98.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">pdm</text></g></g> <g class="uv-resolve-warm-benchmark_svg__mark-text uv-resolve-warm-benchmark_svg__role-mark uv-resolve-warm-benchmark_svg__layer_1_marks"

aria-roledescription="文本标记容器"><text aria-label="时间总和：

0.60278702674;

tool:

poetry;

timeFormat:

0.60s"

aria-roledescription="文本标记"

transform="translate(163.667 53.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">0.60秒</text> <text aria-label="时间总和： 

1.55616658094; 

工具： 

pip-compile; 

时间格式： 

1.56秒" 

aria-roledescription="文本标记" 

transform="translate(245.386 76.25)" 

font-family="Roboto Mono,monospace" 

font-size="12"

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">1.56秒</text> <text aria-label="时间总和： 

3.37404433084; 

工具： 

pdm; 

时间格式： 

3.37秒" 

aria-roledescription="文本标记" 

transform="translate(401.204 98.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">3.37秒</text></g> <g class="uv-resolve-warm-benchmark_svg__mark-text uv-resolve-warm-benchmark_svg__role-mark uv-resolve-warm-benchmark_svg__layer_2_marks" 

aria-roledescription="文本标记容器"><text aria-label="时间总和： 

0.0134756369786; 

工具： 

工具： 

时间格式： 

0.01秒" 

aria-roledescription="文本标记" 

transform="translate(113.155 31.25)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

font-weight="bold" 

class="uv-resolve-warm-benchmark_svg__benchmarkLabel">0.01秒</text></g></g></svg><svg xmlns="http://www.w3.org/2000/svg" 

viewBox="0 0 454 139"><g class="uv-install-warm-benchmark_svg__mark-group uv-install-warm-benchmark_svg__role-frame uv-install-warm-benchmark_svg__root" 

aria-roledescription="组标记容器" 

fill="none" 

stroke-miterlimit="10"><g class="uv-install-warm-benchmark_svg__mark-group uv-install-warm-benchmark_svg__role-axis" 

aria-roledescription="轴" 

aria-label="线性刻度0到5的X轴"><g class="uv-install-warm-benchmark_svg__mark-text uv-install-warm-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="middle" 

transform="translate(84.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-install-warm-benchmark_svg__benchmarkLabel">0秒</text> <text text-anchor="middle" 

transform="translate(204.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-install-warm-benchmark_svg__benchmarkLabel">2秒</text> <text text-anchor="middle" 

transform="translate(324.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-install-warm-benchmark_svg__benchmarkLabel">4秒</text></g></g> <g class="uv-install-warm-benchmark_svg__mark-group uv-install-warm-benchmark_svg__role-axis" 

aria-roledescription="轴" 

aria-label="离散刻度的Y轴，共4个值： 

诗歌, pdm, pip-sync"><g class="uv-install-warm-benchmark_svg__mark-text uv-install-warm-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="end" 

transform="translate(74.5 31.25)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-install-warm-benchmark_svg__benchmarkLabel" 

font-weight="bold">uv</text> <text text-anchor="end" 

transform="translate(74.5 53.75)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-install-warm-benchmark_svg__benchmarkLabel">诗歌</text> <text text-anchor="end" 

transform="translate(74.5 76.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-install-warm-benchmark_svg__benchmarkLabel">pdm</text> <text text-anchor="end"

transform="translate(74.5 98.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-install-warm-benchmark_svg__benchmarkLabel">pip-sync</text></g></g> <g class="uv-install-warm-benchmark_svg__mark-text uv-install-warm-benchmark_svg__role-mark uv-install-warm-benchmark_svg__layer_1_marks"

aria-roledescription="文本标记容器"><text aria-label="时间总和：

0.9872183659;

工具：

poetry;

时间格式：

0.99s"

aria-roledescription="文本标记"

transform="translate(149.233 53.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-install-warm-benchmark_svg__benchmarkLabel">0.99s</text> <text aria-label="时间总和：

1.8969612492;

工具：

pdm;

时间格式：

1.90s"

aria-roledescription="文本标记"

transform="translate(203.818 76.25)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-install-warm-benchmark_svg__benchmarkLabel">1.90s</text> <text aria-label="时间总和：

4.6313483826;

工具：

pip-sync;

时间格式：

4.63s"

aria-roledescription="文本标记"

transform="translate(367.88 98.75)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-install-warm-benchmark_svg__benchmarkLabel">4.63s</text></g> <g class="uv-install-warm-benchmark_svg__mark-text uv-install-warm-benchmark_svg__role-mark uv-install-warm-benchmark_svg__layer_2_marks"

aria-roledescription="文本标记容器"><text aria-label="时间总和：

0.0576289908;

工具：

uv;

时间格式：

0.06s"

aria-roledescription="文本标记"

transform="translate(93.458 31.25)"

font-family="Roboto Mono,monospace"

font-size="12"

字体粗细="bold"

class="uv-install-warm-benchmark_svg__benchmarkLabel">0.06s</text></g></g></svg>

通过**预热缓存**来解析（左）并安装（右）[Trio](https://github.com/python-trio/trio)依赖项，模拟重新创建虚拟环境或向现有项目添加依赖项（[来源](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)）。

解析（顶部）和安装（底部）[Trio](https://github.com/python-trio/trio)依赖项，通过预热缓存模拟重新创建虚拟环境或向现有项目添加依赖项（[来源](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)）。

[uv](https://github.com/astral-sh/uv)被设计为`pip`和`pip-tools`的即插即用替代品，并已准备好在围绕这些工作流程构建的项目中投入生产使用。

像[Ruff](https://github.com/astral-sh/ruff)一样，uv的实现基于我们的核心产品原则：

1.  **专注于性能优化。** 在上述[基准测试](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)中，uv 在没有缓存的情况下比`pip`和`pip-tools`快8-10倍，在使用了热缓存（例如重新创建虚拟环境或更新依赖项）时比它们快80-115倍。uv 使用全局模块缓存以避免重复下载和重建依赖项，并在支持的文件系统上利用写时复制和硬链接来最小化磁盘空间使用。

1.  **为采用而优化。** 虽然我们对 Python 包装的未来抱有远大的期望，但 uv 的初次发布集中于支持我们的`uv pip`接口后面的`pip`和`pip-tools`API，使其可以被现有项目在零配置的情况下使用。同样，uv 可以作为“仅”解析器（`uv pip compile`用于锁定依赖项），“仅”虚拟环境创建器（`uv venv`），“仅”包安装器（`uv pip sync`）等等。它既统一又模块化。

1.  **一个简化的工具链。** uv 作为一个单独的静态二进制文件发行，能够替代`pip`、`pip-tools`和`virtualenv`。uv 没有直接依赖于 Python，因此你可以单独安装它，避免在多个 Python 版本（例如`pip`、`pip3`、`pip3.7`）之间管理`pip`安装的需要。

虽然 uv 将发展成为一个**完整的 Python 项目和包管理器**（类似于["Cargo for Python"](https://blog.rust-lang.org/2016/05/05/cargo-pillars.html#pillars-of-cargo)），但较窄的`pip-tools`范围允许我们解决构建此类工具（如包安装）中涉及的低级问题，同时提供一个立即有用且最小化采纳障碍的解决方案。

你可以通过我们的独立安装程序或从[PyPI](https://pypi.org/project/uv/)安装[uv](https://github.com/astral-sh/uv)。

[uv](https://github.com/astral-sh/uv) 支持现代 Python 包管理工具应具备的所有功能：可编辑安装、Git 依赖项、URL 依赖项、本地依赖项、约束文件、源分发、自定义索引等等，所有这些设计都围绕着与你现有工具的兼容性无缝集成。

[uv](https://github.com/astral-sh/uv) 支持 **Linux**、**Windows** 和 **macOS**，并已在公共 PyPI 索引中进行了大规模测试。

### 一个可直接兼容的 API [#](#a-drop-in-compatible-api)

此次初始发布集中于我们称之为 uv `pip` API 的内容。对于那些过去使用过`pip`和`pip-tools`的人来说，这将是非常熟悉的：

+   不再使用`pip install`，而是运行`uv pip install`来从命令行、要求文件或`pyproject.toml`安装 Python 依赖项。

+   不再使用`pip-compile`，而是运行`uv pip compile`来生成锁定的`requirements.txt`。

+   不再使用`pip-sync`，而是运行`uv pip sync`来将虚拟环境与锁定的`requirements.txt`同步。

通过将这些“低级”命令置于`uv pip`下，我们为CLI保留了空间，以便将来我们计划推出更类似于[Rye](https://github.com/mitsuhiko/rye)，[Cargo](https://github.com/rust-lang/cargo)，或[Poetry](https://github.com/python-poetry/poetry)的“主观”项目管理API（比如`uv run`，`uv build`等）。

uv可以作为虚拟环境管理器使用，通过`uv venv`。它比`python -m venv`快大约80倍，比`virtualenv`快7倍，并且不依赖于Python。

<svg xmlns="http://www.w3.org/2000/svg" 

viewBox="0 0 454 282"><g class="uv-venv-vertical-benchmark_svg__mark-group uv-venv-vertical-benchmark_svg__role-frame uv-venv-vertical-benchmark_svg__root" 

aria-roledescription="group mark container" 

fill="none" 

stroke-miterlimit="10"><g class="uv-venv-vertical-benchmark_svg__mark-group uv-venv-vertical-benchmark_svg__role-scope uv-venv-vertical-benchmark_svg__cell" 

aria-roledescription="group mark container"><g class="uv-venv-vertical-benchmark_svg__mark-group uv-venv-vertical-benchmark_svg__role-axis" 

aria-roledescription="axis"

aria-label="X轴为线性刻度，值从0.00到0.08"><g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="middle" 

transform="translate(107.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0s</text> <text text-anchor="middle" 

transform="translate(182.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0.02s</text> <text text-anchor="middle" 

transform="translate(257.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0.04s</text> <text text-anchor="middle" 

transform="translate(332.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0.06s</text> <text text-anchor="middle" 

transform="translate(407.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0.08s</text></g></g> <g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-mark uv-venv-vertical-benchmark_svg__child_layer_1_marks" 

aria-roledescription="text mark container"><text aria-label="Sum of time: 

0.0744; 

tool: 

virtualenv; 

timeFormat: 

74.4ms" 

aria-roledescription="text mark" 

transform="translate(392 65)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">74.4ms</text> <text aria-label="Sum of time: 

0.0241; 

tool: 

venv; 

timeFormat: 

24.1ms" 

aria-roledescription="text mark" 

transform="translate(203.375 95)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">24.1毫秒</text></g> <g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-mark uv-venv-vertical-benchmark_svg__child_layer_2_marks" 

aria-roledescription="text mark container"><text aria-label="时间总和： 

0.0041； 

工具： 

uv； 

时间格式： 

4.1毫秒" 

aria-roledescription="text mark" 

transform="translate(128.375 35)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

font-weight="bold"

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">4.1毫秒</text></g> <g class="uv-venv-vertical-benchmark_svg__mark-group uv-venv-vertical-benchmark_svg__role-axis" 

aria-roledescription="axis" 

aria-label="线性刻度的X轴，值从0.0到1.6"><g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="middle" 

transform="translate(107.5 264.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0秒</text> <text text-anchor="middle" 

transform="translate(201.25 264.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">0.5s</text> <text text-anchor="middle" 

transform="translate(295 264.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">1秒</text> <text text-anchor="middle" 

transform="translate(388.75 264.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">1.5s</text></g></g> <g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-mark uv-venv-vertical-benchmark_svg__child_layer_1_marks" 

aria-roledescription="text mark container"><text aria-label="时间总和： 

0.1414； 

工具： 

virtualenv； 

时间格式： 

141.4毫秒" 

aria-roledescription="text mark" 

transform="translate(139.512 208)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">141.4毫秒</text> <text aria-label="时间总和： 

1.54； 

工具： 

venv； 

时间格式： 

1.54秒" 

aria-roledescription="text mark" 

transform="translate(401.75 238)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">1.54秒</text></g> <g class="uv-venv-vertical-benchmark_svg__mark-text uv-venv-vertical-benchmark_svg__role-mark uv-venv-vertical-benchmark_svg__child_layer_2_marks" 

aria-roledescription="text mark container"><text aria-label="时间总和： 

0.0182; 

工具：

uv； 

时间格式： 

18.2毫秒" 

aria-roledescription="text mark" 

transform="translate(116.412 178)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

font-weight="bold" 

class="uv-venv-vertical-benchmark_svg__benchmarkLabel">18.2毫秒</text></g></g></g></svg><svg xmlns="http://www.w3.org/2000/svg" 

viewBox="0 0 463 139"><g class="uv-venv-seed-benchmark_svg__mark-group uv-venv-seed-benchmark_svg__role-frame uv-venv-seed-benchmark_svg__root" 

aria-roledescription="分组标记容器" 

fill="none" 

stroke-miterlimit="10"><g class="uv-venv-seed-benchmark_svg__mark-group uv-venv-seed-benchmark_svg__role-axis" 

aria-roledescription="坐标轴" 

aria-label="X轴，线性刻度，范围从0.0到1.6"><g class="uv-venv-seed-benchmark_svg__mark-text uv-venv-seed-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="middle" 

transform="translate(99.5 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">0秒</text> <text text-anchor="middle" 

transform="translate(193.25 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">0.5秒</text> <text text-anchor="middle" 

transform="translate(287 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">1秒</text> <text text-anchor="middle" 

transform="translate(380.75 121.5)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">1.5秒</text></g></g> <g class="uv-venv-seed-benchmark_svg__mark-group uv-venv-seed-benchmark_svg__role-axis" 

aria-roledescription="坐标轴" 

aria-label="Y轴，离散刻度，3个值： 

uv, virtualenv, venv"><g class="uv-venv-seed-benchmark_svg__mark-text uv-venv-seed-benchmark_svg__role-axis-label" 

pointer-events="none"><text text-anchor="end" 

transform="translate(89.5 35)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel" 

font-weight="bold">uv</text> <text text-anchor="end" 

transform="translate(89.5 65)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">virtualenv</text> <text text-anchor="end" 

transform="translate(89.5 95)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">venv</text></g></g> <g class="uv-venv-seed-benchmark_svg__mark-text uv-venv-seed-benchmark_svg__role-mark uv-venv-seed-benchmark_svg__layer_1_marks" 

aria-roledescription="文本标记容器"><text aria-label="时间总和： 

0.1414; 

tool: 

virtualenv; 

timeFormat: 

141.4毫秒" 

aria-roledescription="文本标记" 

transform="translate(131.512 65)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">141.4毫秒</text> <text aria-label="时间总和： 

1.54秒; 

tool: 

venv; 

timeFormat:

1.54秒" 

aria-roledescription="文本标记" 

transform="translate(393.75 95)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-seed-benchmark_svg__benchmarkLabel">1.54秒</text></g> <g class="uv-venv-seed-benchmark_svg__mark-text uv-venv-seed-benchmark_svg__role-mark uv-venv-seed-benchmark_svg__layer_2_marks" 

aria-roledescription="文本标记容器"><text aria-label="时间总和：

0.0182；

tool：

uv;

timeFormat：

18.2ms"

aria-roledescription="文本标记"

transform="translate(108.412 35)"

font-family="Roboto Mono,monospace"

font-size="12"

font-weight="bold"

class="uv-venv-seed-benchmark_svg__benchmarkLabel">18.2ms</text></g></g></svg><svg xmlns="http://www.w3.org/2000/svg"

viewBox="0 0 463 139"><g class="uv-venv-no-seed-benchmark_svg__mark-group uv-venv-no-seed-benchmark_svg__role-frame uv-venv-no-seed-benchmark_svg__root"

aria-roledescription="组标记容器"

fill="none"

stroke-miterlimit="10"><g class="uv-venv-no-seed-benchmark_svg__mark-group uv-venv-no-seed-benchmark_svg__role-axis"

aria-roledescription="轴"

aria-label="线性尺度X轴，值从0.00到0.08"><g class="uv-venv-no-seed-benchmark_svg__mark-text uv-venv-no-seed-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="middle"

transform="translate(99.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">0s</text> <text text-anchor="middle"

transform="translate(174.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">0.02s</text> <text text-anchor="middle"

transform="translate(249.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">0.04s</text> <text text-anchor="middle"

transform="translate(324.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">0.06s</text> <text text-anchor="middle"

transform="translate(399.5 121.5)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">0.08s</text></g></g> <g class="uv-venv-no-seed-benchmark_svg__mark-group uv-venv-no-seed-benchmark_svg__role-axis"

aria-roledescription="轴"

aria-label="离散尺度Y轴，共3个值：

uv，虚拟环境，venv"><g class="uv-venv-no-seed-benchmark_svg__mark-text uv-venv-no-seed-benchmark_svg__role-axis-label"

pointer-events="none"><text text-anchor="end"

transform="translate(89.5 35)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel"

font-weight="bold">uv</text> <text text-anchor="end"

transform="translate(89.5 65)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">虚拟环境</text> <text text-anchor="end"

transform="translate(89.5 95)"

font-family="Roboto Mono,monospace"

font-size="12"

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">venv</text></g></g> <g class="uv-venv-no-seed-benchmark_svg__mark-text uv-venv-no-seed-benchmark_svg__role-mark uv-venv-no-seed-benchmark_svg__layer_1_marks"

aria-roledescription="text mark container"><text aria-label="时间总和：

0.0744;

tool：

虚拟环境；

timeFormat：

74.4ms"

aria-roledescription="text mark" 

transform="translate(384 65)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">74.4ms</text> <text aria-label="时间总和： 

0.0241; 

工具： 

venv; 

timeFormat: 

24.1ms" 

aria-roledescription="text mark" 

transform="translate(195.375 95)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">24.1ms</text></g> <g class="uv-venv-no-seed-benchmark_svg__mark-text uv-venv-no-seed-benchmark_svg__role-mark uv-venv-no-seed-benchmark_svg__layer_2_marks" 

aria-roledescription="text mark container"><text aria-label="时间总和： 

0.0041;

工具： 

uv; 

timeFormat: 

4.1ms" 

aria-roledescription="text mark" 

transform="translate(120.375 35)" 

font-family="Roboto Mono,monospace" 

font-size="12" 

font-weight="bold" 

class="uv-venv-no-seed-benchmark_svg__benchmarkLabel">4.1ms</text></g></g></svg>

创建虚拟环境，使用（顶部）和不使用（底部）种子包如 pip 和 setuptools（[来源](https://github.com/astral-sh/uv/blob/ea13d94c57149a8fc6ebfcef46149252e869269f/scripts/benchmarks/venv.sh)）。

创建虚拟环境，使用（左侧）和不使用（右侧）种子包如 pip 和 setuptools（[来源](https://github.com/astral-sh/uv/blob/ea13d94c57149a8fc6ebfcef46149252e869269f/scripts/benchmarks/venv.sh)）。

uv 的虚拟环境符合标准，并可以与其他工具互换使用 — 没有锁定或定制。

从头开始构建我们自己的软件包管理堆栈还打开了新功能的空间。例如：

+   **uv 支持替代解析策略。** 默认情况下，uv 遵循标准的 Python 依赖解析策略，即首选每个包的最新兼容版本。但是通过传递 `--resolution=lowest`，库作者可以针对其依赖项的最低兼容版本进行测试。（类似于 Go 的 [Minimal version selection](https://go.dev/ref/mod#minimal-version-selection)）。

+   **uv 允许根据任意目标 Python 版本进行解析。** 虽然 `pip` 和 `pip-tools` 总是根据当前安装的 Python 版本进行解析（例如，在 Python 3.12 下运行时生成 Python 3.12 兼容的解析），uv 接受 `--python-version` 参数，允许您生成例如在更新版本下也兼容 Python 3.7 的解析。

+   **uv 允许进行依赖的“覆盖”。** 通过 overrides (`-o overrides.txt`)，uv 进一步扩展了 pip 的“约束”概念，允许用户通过覆盖包的声明依赖来指导解析器。覆盖为用户提供了一种绕过错误的上限和其他错误声明依赖的方法。

在其当前形式下，uv 并不适合所有项目。`pip` 是一个成熟且稳定的工具，支持极其广泛的用例，并专注于兼容性。虽然 uv 支持 `pip` 接口的大部分功能，但它缺乏对一些旧有特性的支持，比如 `.egg` 分发。

同样，uv 目前还没有生成一个平台无关的锁文件。这与 `pip-tools` 一致，但与 Poetry 和 PDM 不同，这使得 uv 更适合围绕 `pip` 和 `pip-tools` 工作流建立的项目。

对于深入包装生态系统的人士，uv 还包括符合标准的 Rust 实现的 [PEP 440](https://peps.python.org/pep-0440/)（版本标识符）、[PEP 508](https://peps.python.org/pep-0508/)（依赖说明符）、[PEP 517](https://peps.python.org/pep-0517/)（独立于构建系统的构建前端）、[PEP 405](https://peps.python.org/pep-0405/)（虚拟环境）等。

### 一个“Cargo for Python”：uv 和 Rye [#](#a-cargo-for-python-uv-and-rye)

uv 代表着我们追求的一个中间里程碑，即一个统一的 Python 包和项目管理器，极其快速、可靠且易于使用，类似于 ["Cargo for Python"](https://blog.rust-lang.org/2016/05/05/cargo-pillars.html#pillars-of-cargo)。

想象一下：一个单一的二进制文件，引导您的 Python 安装，并为您提供一切工作所需，不仅捆绑了 `pip`、`pip-tools` 和 `virtualenv`，还包括 `pipx`、`tox`、`poetry`、`pyenv`、`ruff` 等。

Python 工具可能让人信心不足：启动新项目或维护现有项目需要大量工作，并且命令失败时表现令人困惑。相比之下，在 Rust 生态系统中工作时，您可以信任工具的成功。Astral 工具链旨在将 Python 从低信心体验提升到高信心体验。

Python 包装的这一愿景与 [Rye](https://github.com/mitsuhiko/rye) 提出的愿景非常接近，后者是来自 [Armin Ronacher](https://github.com/mitsuhiko) 的实验性项目和包管理工具。

在与 Armin 的交谈中，很明显我们的愿景非常一致，但要实现这些愿景需要在基础工具上进行重大投资。例如：构建这样的工具需要一个极快、端到端集成、跨平台的解析器和安装程序。**在 uv 中，我们构建了这样的基础工具。**

我们认为这是一个难得的机会合作，避免瓦解 Python 生态系统。**因此，与 Armin 的合作中，我们非常高兴接管 [Rye](https://github.com/mitsuhiko/rye)。** 我们的目标是将 uv 发展成为一个成熟的 ["Cargo for Python"](https://blog.rust-lang.org/2016/05/05/cargo-pillars.html#pillars-of-cargo)，并在合适的时机为从 Rye 迁移到 uv 提供顺畅的迁移路径。

在此之前，我们将继续维护 Rye，并将其迁移到使用 uv 作为核心，而且更广泛地将其视为我们正在构建的终端用户体验的实验性试验平台。

虽然合并项目本身就有其挑战，但我们致力于在 Astral 旗下构建一个统一的工具，并支持现有的 Rye 用户，同时将 uv 进化为一个合适而全面的后继项目。

### 我们的路线图 [#](#our-roadmap)

在此发布后，我们的首要任务是支持用户考虑使用 [uv](https://github.com/astral-sh/uv)，重点是改善跨平台的兼容性、性能和稳定性。

接下来，我们将着眼于将 uv 扩展为一个完整的 Python 项目和包管理器：一个单一的二进制文件，为你提供一切你需要的，让你在 Python 中高效工作。

对于 uv，我们有着雄心勃勃的路线图。但即使在当前形式下，我认为它将为 Python 提供一种非常不同的体验。希望你能试一试。

### 致谢 [#](#acknowledgements)

最后，我们要感谢所有直接或间接为 uv 的发展做出贡献的人。其中，最重要的是 [Jacob Finkelman](https://github.com/Eh2406) 和 [Matthieu Pizenberg](https://github.com/mpizenberg)，他们是 [pubgrub-rs](https://github.com/pubgrub-rs/pubgrub) 的维护者。uv 使用 PubGrub 作为其底层版本解决器，我们感谢 Jacob 和 Matthieu 在过去对 PubGrub 的工作，以及他们作为合作者在整个项目中与我们合作的方式。

我们还要感谢那些在打包领域启发我们的项目，特别是来自 JavaScript 生态系统的 [Cargo](https://github.com/rust-lang/cargo)，以及 [Bun](https://github.com/oven-sh/bun)，[Orogene](https://github.com/orogene/orogene)，[pnpm](https://github.com/pnpm/pnpm)，以及来自 Python 生态系统的 [Posy](https://github.com/njsmith/posy)，[Monotrail](https://github.com/konstin/monotrail-resolve)，和 [Rye](https://github.com/mitsuhiko/rye)。特别感谢 [Armin Ronacher](https://github.com/mitsuhiko) 在此次合作中与我们的协作。

最后，我们要感谢 [pip](https://github.com/pypa/pip) 的维护者和 PyPA 的成员们，因为他们为使 Python 的打包成为可能而做出了所有努力。
