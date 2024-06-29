<!--yml

分类：未分类

日期：2024-05-29 12:32:08

-->

# 我是如何将我的 Rust 编译时间提高了 75% 的。

> 来源：[https://benw.is/posts/how-i-improved-my-rust-compile-times-by-seventy-five-percent](https://benw.is/posts/how-i-improved-my-rust-compile-times-by-seventy-five-percent)
> 
> 现在有第二部分，我将介绍更多选项。在这里可以查看 [链接](https://benw.is/posts/how-i-improved-my-rust-compile-times-part2)

Rust 经常提到的一个痛点是编译时间慢。为了拥有像借用检查器、安全性和零成本抽象等好东西，我们要花费大量时间编译。Web 开发者，尤其是那些来自 JavaScript 框架的开发者，非常非常喜欢快速的重新加载时间。我在 Remix 的 Discord 早期非常活跃，很多人都在问关于 HMR（热模块重载）的问题[超级多](https://github.com/remix-run/remix/discussions/2384)。热模块重载是指当网页框架在现有页面上更改 JavaScript、HTML 和 CSS 模块时，不会丢失页面状态。而另一种方法，即实时页面重新加载，会在检测到更改时触发页面重新加载。对于 Remix，这两者之间的时间差异显著小于在 Rust 中的时间差异，因为在 Rust 中，我们必须重新编译。

作为 Leptos 团队成员，我们经常听到相同的讨论。作为前端 Rust 网络框架，Leptos 位于这两个世界的交汇处。我们经常在 [Leptos Discord](https://discord.gg/x8NhWWYTV2) 上看到这种情况。

> "现在，如果重建速度更快就好了... 🫣"
> 
> "有人有一些改善编译时间的建议吗？Leptos 真的很棒，但是等待 5-10 秒来渲染额外的 div 实在是很烦人"

因此，Leptos 基本上有两种方法来解决这个问题：改善 Rust 的编译时间本身，和/或者想出一种方法来在不需要首先重新编译的情况下输出页面的部分变化。在本文中，我们将探讨这两种方法。

为了测试不同的更改，我们需要建立一个基准。在我的机器上，默认的 Rust 环境，编译我的 Web 应用需要多长时间？

我有一台非常强大的测试机器，通常用来编译。在我的机器上，我将编译一个更简单的项目，我的朋友 Alex 友好地同意测试他的创业公司 Houski 的 [网站](https://www.houski.ca/)。

+   AMD 5950x 处理器，

+   72GB RAM

+   SATA SSD 系统驱动器。

+   7200RPM 旋转硬盘存储驱动器

+   NVME 驱动器

+   NixOS Linux 发行版

+   Rust 1.75 nightly

+   AMD 7900x 处理器

+   128GB RAM

+   NVME 驱动器

+   Ubuntu Linux 发行版

+   Rust 1.75 nightly

Leptos Web 应用通常经历两个阶段的构建过程。

1.  使用 cargo 构建 WebAssembly 前端模块。使用 warm-bindgen 进行优化和打包。

1.  使用 cargo 构建服务器二进制文件。

我选择为大多数演示项目进行整体过程基准测试，但由于实质上是两个 cargo 步骤，任何相对变化都会影响任何 cargo 命令。

> ~~注意：Discord 用户 @PaulH（还有几位其他人）向我反映了关于这个 [bug](https://github.com/leptos-rs/cargo-leptos/pull/203) 的情况，该 bug 导致服务器构建无法使用之前编译的依赖项，因为 RUSTFLAGS 发生了变化。您可以通过将 cargo-leptos 设置为 > 0.2.1 并将以下内容添加到您的 Cargo.toml 文件中的 Leptos 设置块中来修复此问题。我没有启用过这个选项，但这现在是 cargo-leptos 的默认行为。该功能已被弃用。~~
> 
> TOML 标记语言
> 
> ```
> *[**package**.**metadata**.**leptos**]*
> *separate-front-target-dir* *=* *true*
> 
> ```

这些建议来自 Bevy，建议在开发中设置更高的 opt-level 以可能减少开发编译时间并提高性能。默认情况下，Rust 编译器为开发构建设置 opt-level 为 0。我们将为我们的代码设置 opt-level 为 1，并为所有代码的依赖项设置 opt-level 为 3。

这带来了一个缺点，如果错误消息来自我们的依赖项，它们的有用性会大大降低。因此，如果遇到棘手的 bug，您可能需要调整级别。

由于优化需要额外的时间，我预计清洁编译时间会增加，但我们可能会看到增加的优化使代码在增量构建中更容易编译。我们在 Discord 上也有一些积极的案例。

> "fwiw @gbj @Alex Wilkinson 我通过将这个放入我的工作空间 Cargo.toml 中，将我的编译时间从5到30分钟缩短到了6秒。

我们可以在我们的 `Cargo.toml` 文件中启用添加这些块。简单易行。

TOML 标记语言

```
*[**profile**.**dev**]*
*opt-level* *=* 1
*[**profile**.**dev**.**package**.**"*"**]*
*opt-level* *=* 3

```

对于那些不熟悉 Rust 编译器内部工作的人，它通常遵循几个基本步骤。它读取源代码，将其转换为各种类型的中间表示（IR），并在此转换过程中执行优化。然后将该 IR 提交给由 LLVM 提供的代码生成器，将 IR 转换为目标文件，然后链接器将这些目标文件及其他系统库链接成一个可执行二进制文件。关于其工作原理的更多细节可以在 [这里](https://rustc-dev-guide.rust-lang.org/overview.html) 找到。这是一篇引人入胜的阅读，但对我们的讨论来说有些深入。

[Mold](https://github.com/rui314/mold) 是由 Rui Ueyama 开发的一款新链接器，旨在通过尽可能并行化加载来提高链接器的性能，基准测试显示它比 Rust 的默认链接器要快得多。

对于 Linux 和 Mac，默认的链接器是 ld，由 cc 运行。Windows 则使用 Microsoft 的 MVC link.exe。如果您在运行 Linux，则可以直接使用 mold。如果您在 Mac 上，可以购买一个名为 Sold 的 mold 的付费版本，价格非常实惠。如果 Mold 为您带来了好处，我建议您在他的 Github Sponsors [页面](https://github.com/sponsors/rui314) 上购买 Sold 或赞助 Rui。目前不支持 Windows 用户。Sold 中对 Windows 的支持正在开发中。

在增量构建过程中，链接您的Rust二进制文件需要相当大的时间，因此这可能会带来一些真正的好处。

在Linux上，实际上非常容易使用，只需[安装Mold](https://github.com/rui314/mold)，然后在cargo命令之前加上`mold -run`。例如，`mold -run cargo build`。也可以像这样在`.cargo/config.toml`中启用它：

TOML标记

```
*[**target**.**x86_64-unknown-linux-gnu**]*
*linker* *=* *"clang"*
*rustflags* *=* *[**"-C"**,* *"link-arg=-fuse-ld=/path/to/mold"**]*

```

其中`/path/to/mold`是到模具可执行文件的绝对路径。这也是您启用Sold的方法，只需将模具路径替换为Sold路径，将目标替换为您的Mac的目标。

在前面的优化中，我们替换了Rust编译器使用的链接器。让我们尝试替换代码生成器，Cranelift是一个替代代码生成器，在构建步骤中代替LLVM使用。虽然它在执行许多优化方面不如LLVM，但它能够快速生成代码。最近，在Rust 1.73夜间版本中，Cranelift作为x86_64 Linux目标代码生成的一个选项进行了集成。其他平台需要单独设置cranelift，请参阅它们的[README](https://github.com/rust-lang/rustc_codegen_cranelift)。

bash

```
rustup update nightly #install nightly if you haven't already
rustup component add rustc-codegen-cranelift-preview --toolchain nightly

```

要在Cargo中使用它，可以通过启用不稳定的`codegen-backend`功能，然后为一个配置文件设置`codegen-backend= "cranelift"`值来启用它。可以像这样在`.cargo/config.toml`中完成：

TOML标记

```
*[**unstable**]*
*codegen-backend* *=* *true*
*[**profile**.**server-dev**]*
*codegen-backend* *=* *"cranelift"*

```

如果您不需要担心多目标构建，也可以在下面使用一个目标来启用它。这与在生产构建过程中不使用Cranelift的愿望相冲突，因此我不建议这样做。Rust不支持每个目标的配置文件构建，因此创建自己的配置文件更加灵活。

TOML标记

```
*[**target**.**x86_64-unknown-linux-gnu**]*
*rustflags* *=* *[**"-Zcodegen-backend=cranelift"**]*

```

请记住，Cranelift仍在开发中，可能存在一些缺少的内部函数。因此，您的crate在其中可能工作或不工作。如果您发现缺少内部函数，我建议您在它们的仓库[这里](https://github.com/rust-lang/rustc_codegen_cranelift)创建一个问题，可能会有可行的解决方法。

我主要关注优化Rust项目在开发中的构建时间。这意味着我不关心文件大小、优化或运行速度，只要它们不受到显著影响即可。我们关注的是从干净状态开始构建我们的Rust Leptos应用程序的时间，以及增量编译的时间。

为此，我们将使用[hyperfine](https://github.com/sharkdp/hyperfine)，一个命令行基准测试工具，运行 'cargo leptos build'，正如前面描述的那样，Leptos的两步编译。对于清理构建，我们将在每次构建之前运行`cargo clean`，以删除Rust缓存的任何文件。增量构建稍微困难一些。

对于增量构建，我们将模拟开发人员在Leptos组件中修改单个HTML标记。为此，我们将运行我们便捷的Unix朋友`sed`，将当前日期和时间插入HTML标记中。为此，我选择了`<dfn>`标记，因为我从未见过它被使用过，因此我可能不必担心多次替换标记。经过一些调整，我最终得到了这个sed命令：

bash

```
sed -i -e "s|<dfn>[^<]*</dfn>|<dfn>$(date +%m%s)</dfn>|g" src/routes/index.rs

```

每个配置都经过六次测试，其中包括两次预热运行以填充任何缓存并减少输出变异性。没有预热运行时，变异性会略微加大，但结果仍然是一致的。对于这个测试来说，六次运行可能有点多余，但我能说什么呢，我有大量的CPU时间，或者更确切地说是夜间时间。

我们将测量完成六次运行所需的时间，包括两次预热运行以设置文件系统缓存，并希望提供一致的结果。

我将编译我的[网站](https://benwis)，而Alex将编译[Houski的网站](https://houski.ca)。Houski的网站比这个博客复杂得多，大量使用Polars、Serde的序列化/反序列化以及更多的路由。我们两个都没有尝试大幅缩短这些时间，也没有进行任何特殊的优化。

对于我们两个来说，这些将是要超越的时间。

| 清洁编译 |  |  |
| --- | --- | --- |
| 网站 | 平均(s) | 标准偏差(s) |
| benw.is 7200RPM | 88.387 | 1.817 |
| benw.is NVME | 80.057 | 0.268 |
| houski.ca | 124.993 | 0.483 |
| 增量编译 |  |  |
| --- | --- | --- |
| 网站 | 平均(s) | 标准偏差(s) |
| benw.is 7200RPM | 20.461 | 0.801 |
| benw.is NVME | 19.097 | 0.078 |
| houski.ca | 40.818 | 0.252 |

不算太差的基线。我觉得有趣的是在7200转/分钟的构建时间变化，无论是干净构建还是增量构建，这可能表明硬盘在保持一致的数据提供速率方面可能有困难，或者可能有其他因素正在争夺IO访问的基准。

20或40秒在dev配置文件中重新构建网站是一个相当大的时间，希望我们能改进。

让我们按照之前描述的方式启用Mold链接器，看看这将如何改变事情！

| 清洁编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 网站 | 平均(s) | 标准偏差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 74.35 | 0.96 | 14.037 | 15.88129476 |
| benw.is NVME | 67.206 | 0.386 | 12.851 | 16.05231273 |
| houski.ca | 102.892 | 0.416 | 22.101 | 17.68179018 |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 网站 | 平均(s) | 标准偏差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 5.842 | 0.268 | 14.619 | 71.44812082 |
| benw.is NVME | 5.615 | 0.051 | 13.482 | 70.59747604 |
| houski.ca | 19.6 | 0.067 | 21.218 | 51.98196874 |

哇！这快了不少。对于我的站点，旋转磁盘的清洁编译时间减少了12.04秒（13.6%），NVME减少了12.80秒（16.0%）。Houski减少了22.10秒（17.7秒）。这是一个不错的提升。

增量编译时间也发生了显著变化。我的旋转磁盘站点减少了14.62秒（71.4%），NVME减少了13.5秒（70.6%）。这是一个根本性的改进，这在某种程度上是合理的。增量编译的大部分时间都花在链接步骤上，因此在这里进行任何升级将会不成比例地影响它。

至于Leptos用户，Mold仅支持x84_64_linux的编译，因此WebAssembly的编译时间保持不变。这些优化大多数只影响服务器构建，但我会注意哪些不受其影响。

除了一些小缺点外，如此巨大的优势，Mold/Sold似乎是一个明智的选择，特别是如果你经常进行增量构建！

添加更多优化将如何影响编译时间？

| 清洁编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均时间(s) | 标准偏差(s) | 基准时间差(s) | % 基准时间差 |
| benw.is 7200RPM | 174.399 | 0.488 | -86.012 | -97.31295326 |
| benw.is NVME | 166.847 | 0.41 | -86.79 | -108.4102577 |
| houski.ca | 288.562 | 0.674 | -163.569 | -130.8625283 |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均时间(s) | 标准偏差(s) | 基准时间差(s) | % 基准时间差 |
| benw.is 7200RPM | 16.201 | 1.303 | 4.26 | 20.82009677 |
| benw.is NVME | 14.334 | 0.147 | 4.763 | 24.94109022 |
| houski.ca | 32.489 | 0.348 | 8.329 | 20.40521339 |

我确实预料到清洁编译会慢一些，但它确实慢了很多。与基准相比，清洁构建时间是原来的两倍。增量构建的确实获得了20%的改进，这是令人欣慰的。在列出的构建时间中，包含了WebAssembly和服务器构建时间的翻倍，这是有道理的，但应该注意到。对我来说，设定这些的好处并不值得延迟清洁编译时间。

| 清洁编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均时间(s) | 标准偏差(s) | 基准时间差(s) | % 基准时间差 |
| benw.is 7200RPM | 67.989 | 0.763 | 20.398 | 23.07805447 |
| benw.is NVME | 63.645 | 0.247 | 16.412 | 20.50039347 |
| houski.ca |  |  |  |  |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均时间(s) | 标准偏差(s) | 基准时间差(s) | % 基准时间差 |
| benw.is 7200RPM | 9.201 | 0.092 | 11.26 | 55.03152339 |
| benw.is NVME | 9.345 | 0.212 | 9.752 | 51.0656124 |
| houski.ca |  |  |  |  |

Cranelift，我对你寄予了很高的期望，而你也没有辜负！与基准相比，它在清洁和增量编译中都有所改善。你可能会注意到这里的houski应用行是空白的，这是由于缺少llvm内置函数而无法编译。

我的测试完成后，他们合并了这个 [PR](%5B#1417%5D(https://github.com/rust-lang/rustc_codegen_cranelift/pull/1417))，其中添加了缺失的 `llvm.x86.avx.ldu.dq.256` 指令，@bjorn3 提到我可以通过 rust 标志 `RUSTFLAGS="--cfg aes_force_soft"` 避免缺少的 aes 内部功能。要么这样，要么这个 [PR](https://github.com/rust-lang/rustc_codegen_cranelift/pull/1425) 会被合并到 rustup nightly 中。我希望很快更新这一部分。

如果你缺少内部功能，你可以连接调试器查看引发陷阱的原因，或者你可以使用 `cargo vendor` 并搜索内部功能名称或部分内部功能名称。务必在 [rustc cranelift 仓库](https://github.com/rust-lang/rustc_codegen_cranelift/issues/171#issuecomment-1803136179) 中提及任何你缺少的内容，并偶尔查看是否有修复的计划。@bjorn3 在这方面表现不俗。

| 干净编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均值(s) | 标准差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 170.241 | 1.016 | -81.854 | -92.60864154 |
| benw.is NVME | 168.047 | 1.018 | -87.99 | -109.9091897 |
| houski.ca | 271.949 | 0.735 | -146.956 | -117.571384 |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均值(s) | 标准差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 5.789 | 0.051 | 14.672 | 71.70715019 |
| benw.is NVME | 5.727 | 0.041 | 13.37 | 70.01099649 |
| houski.ca | 18.15 | 0.202 | 22.668 | 55.53432309 |

我们的第一个特征组合。不幸的是，这似乎具有 O3 的缓慢干净编译时间，没有比单独使用模具减少编译时间更多。

| 干净编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均值(s) | 标准差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 107.401 | 1.05 | -19.014 | -21.51221333 |
| benw.is NVME | 103.544 | 0.88 | -23.487 | -29.33784678 |
| houski.ca |  |  |  |  |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均值(s) | 标准差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 4.992 | 0.322 | 15.469 | 75.60236548 |
| benw.is NVME | 4.767 | 0.073 | 14.33 | 75.03796408 |
| houski.ca |  |  |  |  |

让我们启用所有功能！增量编译比之前测试的任何选项都要快，但干净编译仍然因 O3 而受到惩罚。Cranelift 似乎减少了来自 O3 的惩罚，但时间会告诉我们它是否比仅 Mold + Cranelift 更好。我怀疑这是因为 Cranelift 的优化比 llvm 少，这与他们的文档一致。

| 干净编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 站点 | 平均值(s) | 标准差(s) | 与基线的差异(s) | % 与基线的差异 |
| benw.is 7200RPM | 110.251 | 0.687 | -21.864 | -24.737 |
| benw.is NVME | 106.732 | 0.526 | -26.675 | -33.323 |
| houski.ca |  |  |  |  |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 网站 | 平均时间(s) | 标准偏差(s) | 与基线的增量(s) | % 增量 |
| benw.is 7200RPM | 9.096 | 0.351 | 11.365 | 55.54469479 |
| benw.is NVME | 8.73 | 0.076 | 10.367 | 54.28601351 |
| houski.ca |  |  |  |  |

Adding further support to the Cranelift optimization hypothesis, we see similiar performance in the clean compilation, but slower performance in incremental without Mold enabled. It is interesting that this ran faster than the previous test with Mold enabled, although I am not sure the difference is signifigant. Perhaps Mold and Cranelift or Mold and O3 have some negative interaction I don't recognize.

| 干净编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 网站 | 平均时间(s) | 标准偏差(s) | 与基线的增量(s) | % 增量 |
| benw.is 7200RPM | 65.343 | 1.243 | 23.044 | 26.07170738 |
| benw.is NVME | 59.777 | 0.191 | 20.28 | 25.33195098 |
| houski.ca |  |  |  |  |
| 增量编译 |  |  |  |  |
| --- | --- | --- | --- | --- |
| 网站 | 平均时间(s) | 标准偏差(s) | 与基线的增量(s) | % 增量 |
| benw.is 7200RPM | 4.831 | 0.251 | 15.63 | 76.38922829 |
| benw.is NVME | 4.654 | 0.067 | 14.443 | 75.62968005 |
| houski.ca |  |  |  |  |

Here's the big money, Cranelift and Mold together. This offered the fastest clean compilation times, with a roughly 25% reduction, and incremental, sitting pretty at around 75% reduction. Interestingly enough, there seemed to be little difference between the NVME and the 7200RPM drives here, suggesting that these two might be better at using disk I/O than some of the other configurations.

For Rust web frameworks, we can try to simulate the performance of HMR, but we won't be able to substitute JS modules. Our attempt to do this is in cargo-leptos, and can be used with the `--hot-reload` flag. It will attempt to send html and css patches to the browser over a web socket connection when a change inside a `view` macro is detected that does not involve a change to a rust code block, and compile in the background. For me it works fairly well.

I didn't try to instrument and test it, as hot reload times are nearly instant. Any changes to the rust code will require a incremental recompilation though, which is why it's so useful to improve that time. JS frameworks will probably have an edge there for the foreseeable future.

启用Mold和Cranelift使我减少了75%的增量和25%的干净编译时间，这是相当显著的。使用Cranelift需要nightly版本，或者在rustup之外设置它，这可能会让一些项目感到不理想。由于Mold需要Linux，Sold需要Mac（并且需要支付费用），对我来说，这进一步增强了Rust Web开发应该在某种Linux或Mac上进行，而不是在Windows上的论点。WSL2可能有助于改善情况，但可能会[更慢一些](https://medium.com/for-linux-users/wsl-2-why-you-should-use-real-linux-instead-4ee14364c18)。因为我有一台Mac笔记本和一台Linux工作站，所以我将为我的Mac购买Sold，并在可以使用Cranelift的项目中使用它。

如果你想在你的硬件或者你的项目上运行这些测试，我已经在Github上自动化了部分过程，并将那个仓库放在[这里](https://github.com/benwis/customs)。

让我知道你的经历如何，并且这是否通过在mastodon上联系`@benwis@hachyderm.io`对你的项目有所帮助。
