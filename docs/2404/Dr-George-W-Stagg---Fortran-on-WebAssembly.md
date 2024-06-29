<!--yml

分类：未分类

日期：2024-05-27 12:56:28

-->

# George W Stagg 博士 - Fortran 在 WebAssembly 上的应用

> 来源：[https://gws.phd/posts/fortran_wasm/](https://gws.phd/posts/fortran_wasm/)

# 简介

[Fortran](https://fortran-lang.org) 是最古老的编程语言之一。它首次出现于 1957 年，早于 C 编程语言、Intel 4004 CPU 甚至 IBM System/360 系列大型计算机。Fortran 诞生之时，字节刚刚问世，计算机仍然采用真空管，情况艰难。

¹ 名称来源于 *Formula Translator*。Fortran 最初是全大写风格，但现代 Fortran 已经放弃了这种写法。

² System/360 大约在 1965 年发布。4004 则是在 1970 年左右。而 [K&R C](https://en.wikipedia.org/wiki/The_C_Programming_Language) 则于 1978 年出版。

³ 有观点认为，Fortran 的缺乏别名和使用本地数组而不是指针算术，使得优化器能够比等效的 C 程序生成更有效率的输出。不过，也有反对意见。

多年来，Fortran 在科学和工程计算密集应用中积累了丰富的历史。它推动了天气预测和气候模型的流体动力学，为我的 [博士论文](../../docs/thesis_gws.pdf) 提供了凝聚态模拟，并且在某些情况下被认为比 C 更有效率，特别是在数值计算工作中。现代 Fortran 的语法也出奇地易于上手。这不是你父辈使用的 Fortran 77 代码；大部分限制使得固定格式的 Fortran 很难使用，在现代 Fortran 中已不再存在。

在计算时代的碰撞中，这篇博文讨论了将现有 Fortran 代码编译为 WebAssembly，以便在 Web 浏览器中运行的方法。我将描述我们当前在 [webR](https://github.com/r-wasm/webr) 项目中使用的方法，使用了 [LLVM](https://llvm.org) 的 `flang-new` 编译器的修补版本。这篇文章也是一则寻求帮助的请求。我描述的方法不幸地依赖于一种技巧，这种技巧意味着我无法在没有更有经验的编译器开发者帮助的情况下向 LLVM 贡献这些改动。

⁴ “LLVM” 不是首字母缩写，而是项目的全称。

# 那么，问题出在哪里？

有许多潜在的方法和工具链可以将 Fortran 编译到 WebAssembly。不幸的是，所有可用的选项都不完备。每种方法都有其缺点，没有一种是简单的即插即用的解决方案。

回到2020年，克里斯托夫·霍纳尔的文章[FORTRAN in the Browser](https://chrz.de/2020/04/21/fortran-in-the-browser/)精彩地总结了当时的情况。即使到现在，这篇文章仍然值得一读，并为本文提供了良好的背景。我个人对克里斯托夫的文章很感激，特别是对其对[Dragonegg](https://dragonegg.llvm.org)工具链的描述。如果没有那篇文章，我很早就会放弃将Fortran用于WebAssembly了。

我们在本文结束时的目标是能够编译一个现代Fortran例程为WebAssembly，该例程接收一些数值参数，计算[BLAS](http://www.netlib.org/blas/)和[LAPACK](https://netlib.org/lapack/)例程的输出，并将结果返回或打印到控制台。据我记得，2020年时，克里斯托夫文章中概述的方法都无法令人满意地完成这个任务。Dragonegg和`f2c`虽然接近，但都有一些缺点，我将在下文描述。

⁵ LAPACK（线性代数包）是一个流行的例程库，用于数值解决线性代数问题。它用Fortran 90编写，并依赖于一个BLAS（基本线性代数子程序）库。

⁶ 通过[Pyodide](https://pyodide.org/en/stable/)和[webR](https://github.com/r-wasm/webr)。

BLAS和LAPACK例程一起提供了一个强大的数值平台。在浏览器中运行它们为几个依赖于它们的高级编程环境打开了大门，例如SciPy或R。这种方法的优势在于，它允许您将现有和经过充分测试的工具和库带到Web上，而无需将它们全部重写为Rust或JavaScript。

后面，我将展示一个机器学习演示示例，该示例直接使用从Fortran编译为WebAssembly的BLAS例程。我们无需在JavaScript中编写复杂的线性代数数值算法，而是直接使用可靠和高效的BLAS例程。

⁷ 尽管在Web浏览器中运行机器学习算法永远无法与使用专用硬件（如GPU）一样高效，但我仍然认为这是一个有趣的演示。

## 编译器概述

自从[FORTRAN in the Browser](https://chrz.de/2020/04/21/fortran-in-the-browser/)发布以来，一些事情有所变化，特别是当涉及到基于LLVM的Fortran编译器时。据我所知，这里是2024年的当前情况的简要概述。

### `f2c`实用程序

[`f2c`](https://en.wikipedia.org/wiki/F2c)程序将Fortran 77转换为C代码，然后Emscripten可以将其编译成WebAssembly。这是[Pyodide](https://pyodide.org/en/stable/)项目用来编译包含Fortran代码的Python包的方法。他们说这种方法“效果不佳”（https://pyodide.org/en/0.25.0/project/roadmap.html#find-a-better-way-to-compile-fortran）。该工具无法处理现代Fortran代码，即使转换后结果仍会出现致命错误，需要大量修补。

### LFortran

过去几年里，LFortran编译器取得了长足的进步。2020年时，它[缺少很多功能](https://gitlab.com/lfortran/lfortran/-/issues/121)，仅支持极小的Fortran子集。现在，它支持更广泛的语言特性，并可以用来编译相当数量的Fortran代码。它甚至可以直接编译到WebAssembly！

⁸ 请查看[LFortran演示](https://dev.lfortran.org)。尽管令人印象深刻，但请注意，我尝试的第一件事是将`x ** 2`改为`x ** 3`，结果发现当前代码生成器不支持这样的更改。

但是，LFortran仍然存在一些使用上的障碍。该项目目前被认为处于alpha阶段，开发人员表示预计会出现编译现实世界代码的问题。虽然它可以成功编译一些项目，例如[MINPACK](https://github.com/fortran-lang/minpack)，但完整的Fortran规范尚未完全支持，因此许多较大的项目仍无法编译。

LFortran开发人员正在努力支持完整的Fortran 2018，并且其显著特性是一个类似于交互式Jupyter的Fortran REPL。再经过几年的发展，我预计LFortran将成为编译Fortran代码到WebAssembly的优秀选择。

### Dragonegg

[Dragonegg](https://dragonegg.llvm.org)是GCC的一个插件，使用GNU编译器作为前端，并生成LLVM IR。通过这种方式，LLVM可以作为后端生成WebAssembly输出。这种技术可行，并且这是我为[webR项目](https://github.com/r-wasm/webr)编译Fortran源代码最初使用的方法。

然而，这种方法也存在一些严重的缺点。Dragonegg需要非常旧的GCC和LLVM版本。对大多数用户而言，这意味着需要设置虚拟机或Docker容器来提供必要的环境。Dragonegg生成的LLVM IR还需要一些相当复杂的后处理才能由LLVM生成WebAssembly输出。查看[webR最初使用的脚本](https://github.com/r-wasm/webr/blob/v0.1.0/tools/dragonegg/emfc.in)可以了解所需的额外处理。

⁹ 最新支持的版本是`gcc-4.8`和`llvm-3.3`

然而，在2020年，这是编译Fortran代码到WebAssembly的唯一真正方法。

### 经典Flang

[“经典”Flang](https://github.com/flang-compiler/flang)是另一个针对LLVM的Fortran编译器，基于开源的PGI/NVIDIA编译器`pgfortran`。经典Flang从未支持32位输出，因此对我们来说不是一个选择，因为我们将使用`wasm32`作为目标架构。在浏览器对64位Wasm内存的支持改善之前，这可能会一直是问题。

^(10) 以前是Flang或Flang-7。

^(11) 截至撰写时，Firefox、Chrome和Node都支持`wasm64`，但需要通过功能标志来锁定。

尽管如此，项目文档本身建议，选择在新项目中使用经典Flang可能不是一个好主意：

> 经典Flang[…]继续维护，但计划未来将其替换为新的Flang。

### LLVM Flang

[“LLVM Flang”](https://flang.llvm.org/docs/)是一个完全从头开始重新实现的用于LLVM的Fortran前端。它旨在取代由同一团队开发的经典Flang，并且已经作为LLVM项目的一部分被接受，从LLVM 11版本开始，Flang的源代码现在可以在[官方LLVM源码树](https://github.com/llvm/llvm-project/tree/main/flang)中找到。

^(12) 也被称为Flang、新Flang或`flang-new`。以前是F18。

Flang目前尚未被认为已经准备好用于生产，但其开发目前非常活跃，团队已经发布了`flang-new`编译器的预生产版本。近年来，该编译器在编译实际的Fortran代码方面变得非常可用。

目前，LLVM Flang无法直接生成WebAssembly输出。尽管如此，由于LLVM的模块化设计，我们很快将看到可以将Flang前端与LLVM的WebAssembly后端结合使用。通过这种方式，我们可以利用NVIDIA和PGI团队为编译Fortran到WebAssembly所做的所有开发工作。

这在2020年也是可能的，尽管需要对LLVM进行较大的补丁，注入自定义数学函数，并进行多步骤的编译过程。现在，由于`flang-new`前端的显著开发工作，只需对LLVM的源代码进行少量更改，就可以创建一个Fortran到WebAssembly的编译器。

# 构建并使用LLVM Flang用于WebAssembly

如果有兴趣尝试LLVM Flang，可以通过喜欢的包管理器获取LLVM的发布版本。然而，按照这条路线走会让我们失望，因为`flang-new`二进制文件并不包含在内。

^(13) 至少，在使用`brew`安装LLVM v17.0.6的macOS上不行。

[~/fortran]brew install llvm

==> 下载 https://formulae.brew.sh/api/formula.jws.json

################################################################################ 100.0%

==> 下载 https://ghcr.io/v2/homebrew/core/llvm/manifests/17.0.6_1

################################################################################ 100.0%

==> 获取 llvm

==> 下载 https://ghcr.io/v2/homebrew/core/llvm/blobs/sha256:8d739bdfa4152

################################################################################ 100.0%

==> 安装 llvm

==> 正在解压 llvm--17.0.6_1.arm64_sonoma.bottle.tar.gz

==> 总结

🍺 /opt/homebrew/Cellar/llvm/17.0.6_1: 7,207 files, 1.7GB

==> 检查升级公式的依赖项...

==> 没有需要重新安装的已损坏依赖项！

[~/fortran]flang-new

zsh: command not found: flang-new

既然我们无论如何都要修改 LLVM Flang 源码，我们需要从头开始编译。让我们获取 LLVM v18.1.1 的源码并从那里开始。请随时跟进；我会尽量提供所有必要的命令和信息。

^(14) 我假设您熟悉 [Emscripten](https://emscripten.org)，并且在您的路径上有该工具链的版本。如果不是这样，并且您希望参与其中，请从 [emsdk](https://github.com/emscripten-core/emsdk) 开始，在您的机器上设置 Emscripten，熟悉为 WebAssembly 编译 C 代码的过程，然后再回到这里继续。

[~/fortran]git clone --depth=1 --branch=llvmorg-18.1.1 https://github.com/llvm/llvm-project.git

正在克隆到 'llvm-project'...

远程：正在枚举对象：138937，完成。

接收对象：100%（138937/138937），199.81 MiB | 11.36 MiB/s，完成。

注意：切换到 '6009708b4367171ccdbf4b5905cb6a803753fe18'。

更新文件：100%（132077/132077），完成。

[~/fortran]cmake -G Ninja -S llvm-project/llvm -B build \

-DCMAKE_INSTALL_PREFIX=llvm-18.1.1 \

-DCMAKE_BUILD_TYPE=MinSizeRel \

-DLLVM_DEFAULT_TARGET_TRIPLE="wasm32-unknown-emscripten" \

-DLLVM_TARGETS_TO_BUILD="WebAssembly" \

-DLLVM_ENABLE_PROJECTS="clang;flang;mlir"

-- C 编译器标识为 AppleClang 15.0.0.15000100

-- 发现汇编程序：/Library/Developer/CommandLineTools/usr/bin/cc

-- 检测 C 编译器 ABI 信息 - 完成

-- 测试成功 HAVE_POSIX_REGEX --

-- 配置完成（29.0秒）

-- 生成完成（2.9秒）

-- 已编写构建文件：fortran/build

[~/fortran]cmake --build build

...

[6136/6136] 链接 CXX 可执行文件 bin/obj2yaml

拿杯茶吧，这一步会消耗大量资源，可能需要**非常**长的时间。

^(15) **cuppa** (/ˈkʌp.ə/): 名词，非正式用法，英国。一种热饮，通常是茶或咖啡。

## 插曲 —— 从 C 调用 Fortran 子程序

等待 LLVM 构建时，打开一个新的终端窗口，我们来回顾一下如何将 Fortran 子程序编译并链接到 C 程序中。这里的原理将有助于我们稍后在 JavaScript 中调用 Fortran 时使用。

首先，让我们编写一个简单的子程序，接受三个整数参数：`x`、`y` 和 `z`。它将把 `z` 的值设为 `x` 和 `y` 的和。命名我们的新子程序为 `foo`，并将包含您的子程序的文件保存为 `foo.f08`。

```
SUBROUTINE foo(x, y, z)
 IMPLICIT NONE
 INTEGER, INTENT(IN)  :: x, y
 INTEGER, INTENT(OUT) :: z
 z = x + y
END
```

*注意到，一般来说，Fortran 例程通过引用传递参数，并且我们可以使用 `INTENT()` 在子程序中声明参数的使用方式。假设您已经安装了类似 `gfortran` 的传统 Fortran 编译器，请将 Fortran 源码编译成目标文件。

^(16) 您不*必须*有一个本地的 Fortran 编译器才能跟着本文继续进行，但如果您想要一个，可以从您操作系统的常规软件包管理器中获取 `gfortran` 作为 GCC 编译器套件的一部分。如果您在 Intel CPU 上，还有 [`ifort`](https://www.intel.com/content/www/us/en/developer/tools/oneapi/fortran-compiler.html)。

[~/fortran]gfortran -c foo.f08 -o foo.o

[~/fortran]文件 foo.o

foo.o：Mach-O 64 位对象 arm64

[~/fortran]nm foo.o

0000000000000038 s EH_frame1

0000000000000000 T _foo_

0000000000000000 t ltmp0

0000000000000038 s ltmp1

我使用的是 M1 macOS 机器，所以我的生成对象是用于 ARM 的 Mach 对象。如果您是 Linux 用户，您应该看到类似 `ELF 64 位 LSB 共享对象，x86-64` 的内容。我还运行了 `nm` 命令来查看编译器生成的对象中的符号名称。请注意我们子例程的符号 —— 在我的机器上，它被命名为 `_foo_`。前导下划线是相当标准的，但是尾随下划线与 C 过程的通常情况不同。

让我们编写一个调用我们 Fortran 子例程的 C 程序。再次注意我们如何通过引用将参数传递给外部符号。此外，如果您的 Fortran 编译器添加了尾随下划线，我们在 C 中声明符号名称时也需要包含它。

^(17) 现代 Fortran 标准提供了一个 Fortran 模块 [`iso_c_binding`](https://fortranwiki.org/fortran/show/iso_c_binding) 和一个 C 头文件 `ISO_Fortran_binding.h`，以提高与 C 的互操作性，但我们今天的代码足够简单，可以不用这些。

```
#include <stdio.h>

extern void foo_(int*, int*, int*);

int main() {
 int x = 1, y = 1, z;
 foo_(&x, &y, &z);

 printf("%d + %d = %d\n", x, y, z);
 return 0;
}
```

*使用 `gcc` 或等效工具编译 C 源码，然后运行生成的二进制文件，观察令人惊叹的数值计算水平。

[~/fortran]gcc main.c foo.o -o main

[~/fortran]./main

1 + 1 = 2**  **## 返回 LLVM Flang

一旦 LLVM 编译完成，`flang-new` 二进制文件应该位于目录 `build/bin` 中。现在我们可以运行它，并确认已设置为生成 `wasm32` 和 Emscripten 的二进制文件。

[~/fortran]./build/bin/flang-new --version

flang-new 版本 18.1.1（https://github.com/llvm/llvm-project.git dba2a75e9c7ef81fe84774ba5eee5e67e01d801a）

目标：wasm32-unknown-emscripten

线程模型：posix

InstalledDir：.../fortran/build/bin

太棒了！让我们尝试使用我们新构建的编译器编译我们的 Fortran 子例程。

[~/fortran]./build/bin/flang-new -c foo.f08 -o foo.o

错误：fortran/llvm-project/flang/lib/Optimizer/CodeGen/Target.cpp:1162：

尚未实现：目标未实现

LLVM 错误：中止

啊，不太好。`flang-new` 编译器中遗憾地尚未实现 `wasm32-unknown-emscripten` 目标三元组。

现在我们来为 LLVM 的第一个补丁。我们将通过扩展 Flang 的已知目标特定列表来实现目标。下面显示的所需更改可以通过查看文件 `flang/lib/Optimizer/CodeGen/Target.cpp` 中已实现的其他目标来推断。

```
diff --git a/flang/lib/Optimizer/CodeGen/Target.cpp b/flang/lib/Optimizer/CodeGen/Target.cpp
index 83e7fa9b440b..49e73ec48e0a 100644
--- a/flang/lib/Optimizer/CodeGen/Target.cpp
+++ b/flang/lib/Optimizer/CodeGen/Target.cpp
@@ -1109,6 +1109,44 @@ struct TargetLoongArch64 : public GenericTarget<TargetLoongArch64> {
 };
 } // namespace

+//===----------------------------------------------------------------------===//
+// WebAssembly (wasm32) target specifics.
+//===----------------------------------------------------------------------===//
+
+namespace {
+struct TargetWasm32 : public GenericTarget<TargetWasm32> {
+  using GenericTarget::GenericTarget;
+
+  static constexpr int defaultWidth = 32;
+
+  CodeGenSpecifics::Marshalling
+  complexArgumentType(mlir::Location, mlir::Type eleTy) const override {
+    assert(fir::isa_real(eleTy));
+    CodeGenSpecifics::Marshalling marshal;
+    // Use a type that will be translated into LLVM as:
+    // { t, t }   struct of 2 eleTy, byval, align 4
+    auto structTy =
+        mlir::TupleType::get(eleTy.getContext(), mlir::TypeRange{eleTy, eleTy});
+    marshal.emplace_back(fir::ReferenceType::get(structTy),
+                         AT{/*alignment=*/4, /*byval=*/true});
+    return marshal;
+  }
+
+  CodeGenSpecifics::Marshalling
+  complexReturnType(mlir::Location loc, mlir::Type eleTy) const override {
+    assert(fir::isa_real(eleTy));
+    CodeGenSpecifics::Marshalling marshal;
+    // Use a type that will be translated into LLVM as:
+    // { t, t }   struct of 2 eleTy, sret, align 4
+    auto structTy = mlir::TupleType::get(eleTy.getContext(),
+                                          mlir::TypeRange{eleTy, eleTy});
+    marshal.emplace_back(fir::ReferenceType::get(structTy),
+                          AT{/*alignment=*/4, /*byval=*/false, /*sret=*/true});
+    return marshal;
+  }
+};
+} // namespace
+
 // Instantiate the overloaded target instance based on the triple value.
 // TODO: Add other targets to this file as needed.
 std::unique_ptr<fir::CodeGenSpecifics>
@@ -1158,6 +1196,9 @@ fir::CodeGenSpecifics::get(mlir::MLIRContext *ctx, llvm::Triple &&trp,
 case llvm::Triple::ArchType::loongarch64:
 return std::make_unique<TargetLoongArch64>(ctx, std::move(trp),
 std::move(kindMap), dl);
+  case llvm::Triple::ArchType::wasm32:
+    return std::make_unique<TargetWasm32>(ctx, std::move(trp),
+                                               std::move(kindMap), dl);
 }
 TODO(mlir::UnknownLoc::get(ctx), "target not implemented");
 }
```

*将上述差异内容保存为文件 `add-wasm32-target.diff`，然后使用 `git` 或 `patch` 实用程序将其应用于 `llvm-project` 目录。然后，重新构建 LLVM Flang。第二次构建应该会更快，因为大部分生成的对象不受此更改的影响。

[~/fortran]patch -p1 -d llvm-project < add-wasm32-target.diff

[~/fortran]cmake --build build

...

[180/180] 生成../../../../include/flang/ieee_arithmetic.mod

一旦LLVM重新编译，再次尝试编译我们的Fortran源。

[~/fortran]./build/bin/flang-new -c foo.f08 -o foo.o

[~/fortran]file foo.o

foo.o: WebAssembly (wasm) 二进制模块版本 0x1 (MVP)

[~/fortran]llvm-nm foo.o

00000001 T foo_

成功！我们可以确认这是一个真实的WebAssembly对象，使用`file`实用程序，`llvm-nm`可以看到其中的`foo`符号，对应我们的Fortran子程序。

^(18) 您可能需要使用Emscripten提供的WebAssembly感知版本的这个工具。如果您正在使用[emsdk](https://github.com/emscripten-core/emsdk)，请确保`.../emsdk/upstream/bin/`在您的`$PATH`中。

^(19) 在这里我正在使用Node v18，但我认为比Node v16更新的版本都应该可以工作。Emscripten捆绑了一个Node的版本，但我喜欢使用[nvm](https://github.com/nvm-sh/nvm)来管理我的Node安装。

让我们继续使用Emscripten编译我们的C函数为WebAssembly，并使用Node运行它。我们应该看到与我们的本机二进制相同的输出。

[~/fortran]emcc main.c foo.o -o main.js

[~/fortran]node main.js

1 + 1 = 2*  *## 幕间 — 从JavaScript调用Fortran例程

在前一节中，我们使用了一个C程序调用Fortran代码，但我们技术上不需要这样做。如果我们告诉Emscripten关于Fortran子程序，我们可以直接从JavaScript调用它，而无需编写任何C代码。

首先，让我们将我们的Fortran对象与Emscripten链接，生成一个脚本，将我们的WebAssembly二进制加载到内存中，但不执行任何例程。除了我们的符号`_foo_`之外，我们还将导出`_malloc`和`_free`，以便我们可以从JavaScript中使用它们。

^(20) 请查看[Emscripten文档](https://emscripten.org/docs/tools_reference/settings_reference.html)，以获取有关`emcc`命令行选项的更多详细信息。顺便说一句，如果您之前没有使用过Emscripten，可能会在本文的各个步骤中看到额外的`cache:INFO`行。它们无需担心，可以忽略不计。

[~/fortran]emcc foo.o -sEXPORTED_FUNCTIONS=_foo_,_malloc,_free -o foo.js

cache:INFO: 正在生成系统资产：symbol_lists/ae47d07bfa3321ac4dde96bd87821b4aa93f9a19.json...（这将被缓存到".../emsdk/upstream/emscripten/cache/symbol_lists/ae47d07bfa3321ac4dde96bd87821b4aa93f9a19.json"以供后续构建使用）

cache:INFO: - 完成

[~/fortran]node foo.js

[~/fortran]

注意，当我们直接运行脚本`foo.js`时......什么也不会发生。

接下来，我们将编写一个JavaScript文件，加载`foo.js`，然后调用我们的Fortran子例程。我们需要分配一些内存来保存我们的整数`x`、`y`和`z`，使用导出的`_malloc()`函数。我们还需要将我们的输入参数`x`和`y`设置为一些整数值，我们可以通过在分配的WebAssembly内存中设置值来实现，使用`Module.HEAPU32`。

```
var Module = require('./foo.js');

setTimeout(() => {
 const x = Module._malloc(4);
 const y = Module._malloc(4);
 const z = Module._malloc(4);
 Module.HEAPU32[x / 4] = 123;
 Module.HEAPU32[y / 4] = 456;

 Module._foo_(x, y, z);

 console.log("x = ", Module.HEAP32[x / 4]);
 console.log("y = ", Module.HEAP32[y / 4]);
 console.log("x + y = ", Module.HEAP32[z / 4]);

 Module._free(x);
 Module._free(y);
 Module._free(z);
}, 100);
```

*[~/fortran]node standalone.js

x = 123

y = 456

x + y = 579

您还应该能够在 web 浏览器中运行生成的 WebAssembly 二进制文件。从 `standalone.js` 中删除 `var Module = require('./foo.js');` 这一行，而是在您的 HTML 中加载脚本 `foo.js`。

```
<html>
 <head>
 <title>Fortran Demo</title>
 </head>
 <body>
 <script src="foo.js"></script>
 <script src="standalone.js"></script>
 </body>
</html>
```

*启动本地 web 服务器，访问页面，同样的输出应该出现在浏览器的 JavaScript 控制台中。

^(22) 类似 `Rscript -e 'httpuv::runStaticServer()'` 或 `python3 -m http.server` 应该很好用。**  **## Fortran 运行时库：通向“Hello, World!”的旅程

普遍的“Hello, World!”测试程序通常用来介绍编程语言，但我没有使用这样的程序来介绍 Fortran。正如您将看到的那样，这是有充分理由的。让我们看看当我们尝试在 Fortran 中构建一个“Hello, World!”子程序并从 C 调用它时会发生什么。与之前一样，我们将使用 `flang-new` 编译 Fortran 对象，并使用 Emscripten 编译和链接 C 代码。

```
SUBROUTINE hello()
 IMPLICIT NONE
 PRINT *, "Hello, World!"
END
```

*```
extern void hello_();

int main() {
 hello_();
 return 0;
}
```

*[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o -o hello.js

wasm-ld: error: hello.o: undefined symbol: _FortranAioBeginExternalListOutput

wasm-ld: error: hello.o: undefined symbol: _FortranAioOutputAscii

wasm-ld: error: hello.o: undefined symbol: _FortranAioEndIoStatement

emcc: error: 'wasm-ld -o hello.wasm [...] --no-entry --stack-first --table-base=1' 失败（返回 1）

构建失败，因为缺少一些符号。这是我们尚未为 WebAssembly 编译 LLVM Fortran 运行时库的结果。当前缺少一些库符号，包括打印输出所需的某些函数！

幸运的是，运行时库作为 LLVM 源代码树的一部分，以 C++ 编写在 `llvm-project/flang/runtime` 中。因此，原则上，我们只需使用 Emscripten 的 `em++` 编译器构建该库，然后在使用 Fortran 代码的 WebAssembly 程序中链接它。

这里有一个 `Makefile`，旨在使此步骤变得简单。将其保存在当前目录中，然后运行 `make`。它应该会使用您路径上的 Emscripten 版本，在 `build/flang/runtime/libFortranRuntime.a` 中构建一个静态 Fortran 运行时库。

^(24) 确保使用制表符而不是空格来缩进此文件中的规则。

```
ROOT =  $(abspath .)
SOURCE =  $(ROOT)/llvm-project
BUILD =  $(ROOT)/build

RUNTIME_SOURCES :=  $(wildcard  $(SOURCE)/flang/runtime/*.cpp)
RUNTIME_SOURCES +=  $(SOURCE)/flang/lib/Decimal/decimal-to-binary.cpp
RUNTIME_SOURCES +=  $(SOURCE)/flang/lib/Decimal/binary-to-decimal.cpp
RUNTIME_OBJECTS =  $(patsubst  $(SOURCE)/%,$(BUILD)/%,$(RUNTIME_SOURCES:.cpp=.o))

RUNTIME_CXXFLAGS += -I$(BUILD)/include -I$(BUILD)/tools/flang/runtime
RUNTIME_CXXFLAGS += -I$(SOURCE)/flang/include -I$(SOURCE)/llvm/include
RUNTIME_CXXFLAGS += -DFLANG_LITTLE_ENDIAN
RUNTIME_CXXFLAGS += -fPIC -Wno-c++11-narrowing -fvisibility=hidden
RUNTIME_CXXFLAGS += -DFE_UNDERFLOW=0 -DFE_OVERFLOW=0 -DFE_INEXACT=0
RUNTIME_CXXFLAGS += -DFE_INVALID=0 -DFE_DIVBYZERO=0 -DFE_ALL_EXCEPT=0

$(BUILD)/flang/runtime/libFortranRuntime.a:  $(RUNTIME_OBJECTS)
 @rm -f $@
 emar -rcs $@ $^

$(BUILD)%.o :  $(SOURCE)%.cpp
 @mkdir -p $(@D)
 em++ $(RUNTIME_CXXFLAGS) -o $@ -c $<

.PHONY: clean
clean:
 @rm $(RUNTIME_OBJECTS)  $(BUILD)/flang/runtime/libFortranRuntime.a
```

*[~/fortran]make

em++ .../ISO_Fortran_binding.o -c .../ISO_Fortran_binding.cpp

em++ .../allocatable.o -c .../allocatable.cpp

em++ .../array-constructor.o -c .../array-constructor.cpp

...

emar -rcs build/flang/runtime/libFortranRuntime.a .../binary-to-decimal.o

[~/fortran]file build/flang/runtime/libFortranRuntime.a

build/flang/runtime/libFortranRuntime.a: 当前的 ar 存档

让我们再试一次，在 Emscripten 构建步骤中链接我们闪亮的新库。

[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o build/flang/runtime/libFortranRuntime.a -o hello.js

wasm-ld: warning: 函数签名不匹配：_FortranAioOutputAscii

>>> 在 hello.o 中定义为 (i32, i32, i64) -> i32

>>> 在 build/flang/runtime/libFortranRuntime.a(io-api.o) 中定义为 (i32, i32, i32) -> i32

成功了吗？不完全。有一个警告被发出，让我们知道存在签名不匹配的问题。Emscripten已经编译了符号`_FortranAioOutputAscii`来接受三个`i32`参数。然而，`flang-new`编译了`hello.f08`，期望该符号接受两个`i32`参数和一个`i64`参数。

^(25) 这是[LLVM IR标记](https://llvm.org/docs/LangRef.html#integer-type)，表示大小为32位的整数。

^(26) 在编译webR包时，这种情况仍然会出现。软件包作者或供应的库可能使用了诸如`f2c`这样的工具，声明Fortran `SUBROUTINE`返回`int`，而其他库可能声明Fortran `SUBROUTINE`返回`void`。谁是正确的？我不确定，因为据我了解，早期的Fortran没有标准的C接口。个人认为返回`void`是最有意义的。

这真是不幸。尽管只被作为警告发出，如果您尝试使用Node运行生成的程序，您会发现问题是灾难性的。与许多目标系统不同，WebAssembly绝对要求在多个编译单元中定义的符号具有一致的函数签名，包括参数和返回类型。

[~/fortran]node hello.js

.../fortran/hello.js:128

throw ex;

^

RuntimeError: unreachable

在 wasm://wasm/001a0366:wasm-function[20]:0x15d9

在 removeRunDependency (/Users/georgestagg/fortran/hello.js:630:7)

Node.js v18.18.0

与其详细讨论最终导致这里发生了什么的调试过程，不如直接指出问题的原因。看看LLVM源代码中的这条评论：

```
flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
```

```
//===----------------------------------------------------------------------===//
// Type builder models
//===----------------------------------------------------------------------===//

// TODO: all usages of sizeof in this file assume build ==  host == target.
// This will need to be re-visited for cross compilation.

/// Return a function that returns the type signature model for the type `T`
/// when provided an MLIRContext*. This allows one to translate C(++) function
/// signatures from runtime header files to MLIR signatures into a static table
/// at compile-time.
///
/// For example, when `T` is `int`, return a function that returns the MLIR
/// standard type `i32` when `sizeof(int)` is 4.
```

*问题就在这里。对于我们来说，主机与目标不同，打破了LLVM源代码中的假设。令人惊讶的是，这并没有像你预期的那样造成太多混乱。据我所知，这个机制仅用于将用C++编写的Fortran运行时库函数提供给Fortran使用。使用了`sizeof()`的编译时计算，由于大多数大小匹配，它基本上运行良好。

^(27) [主机和目标的C数据模型](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models)定义了特定硬件架构和操作系统上如何表示某些基本C类型的位数。具体的大小可以根据硬件架构和操作系统而异。

不幸的是，假设你正在一台现代的64位类Unix系统，比如Linux或者macOS上跟随，`long`数据类型的大小并不匹配。在我们编译器的主机平台上，`sizeof(long)`的结果是8字节（`i64`），但对于`wasm32-unknown-emscripten`目标平台，返回的值应该是4字节（`i32`）。

当我们使用 Emscripten 编译 Fortran 运行时库的 C++ 代码时，一切都很顺利。生成的符号是带有签名的，以便 `long` 参数为 `i32`。然而，当我们使用 `flang-new` 编译我们的 Fortran 代码时，外部库符号声明的是 `long` 参数为 `i64`。这种差异导致了不一致的函数签名警告和运行时失败。

为什么在我们的“Hello, World!”程序中使用 `PRINT()` 会调用一个带有 `long` 类型参数的函数？嗯，在一些 Fortran 实现中，当您将 Fortran `CHARACTER` 类型传递给函数或子程序时，会添加所谓的“隐藏”参数。这些额外的参数传递字符串的长度。在 Fortran 运行时库中，隐藏参数声明为 `size_t` 类型，随后通过一系列 `typedef` 最终变为 `unsigned long`。这个隐藏的隐式参数就是尺寸不一致的参数。****  ***## 解决问题的临时办法

不幸的是，我对 LLVM 或 Flang 的内部了解不够，无法为这个问题实现真正的解决方案。理想情况下，当进行交叉编译时，无论编译器运行在哪个主机架构上，`flang-new` 都应该针对目标架构和数据模型正确地使用 `i32` 或 `i64`。

由于今天我解决不了这个问题，暂时让我们绕过它。我们将构建一个版本的 `flang-new`，将 `long` 的大小硬编码为我们在 `wasm32` 和 Emscripten 中需要的大小。我们还会做一些更改，以便从 Fortran 中对 `malloc()` 的调用发出带有 `i32` 参数。

^(28) 此外，这还修复了 Fortran 90 中引入的 `ALLOCATE()` 的动态分配功能。

所需的补丁如下所示。如果您在跟进，将其保存为名为 `force-4-byte-values.diff` 的文件，并使用 `git` 或 `patch` 实用程序将其应用于 `llvm-project` 目录。最后，重新编译 `flang-new` 一次。

```
diff --git a/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h b/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
index b3fe52f4b..c3c7326da 100644
--- a/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
+++ b/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
@@ -146,7 +146,7 @@ constexpr TypeBuilderFunc getModel<void **>() {
 template <>
 constexpr TypeBuilderFunc getModel<long>() {
 return [](mlir::MLIRContext *context) -> mlir::Type {
-    return mlir::IntegerType::get(context, 8 * sizeof(long));
+    return mlir::IntegerType::get(context, 8 * 4);
 };
 }
 template <>
@@ -187,7 +187,7 @@ constexpr TypeBuilderFunc getModel<long long *>() {
 template <>
 constexpr TypeBuilderFunc getModel<unsigned long>() {
 return [](mlir::MLIRContext *context) -> mlir::Type {
-    return mlir::IntegerType::get(context, 8 * sizeof(unsigned long));
+    return mlir::IntegerType::get(context, 8 * 4);
 };
 }
 template <>
diff --git a/flang/lib/Optimizer/CodeGen/CodeGen.cpp b/flang/lib/Optimizer/CodeGen/CodeGen.cpp
index ba5946415..2931753a8 100644
--- a/flang/lib/Optimizer/CodeGen/CodeGen.cpp
+++ b/flang/lib/Optimizer/CodeGen/CodeGen.cpp
@@ -1225,7 +1225,7 @@ getMalloc(fir::AllocMemOp op, mlir::ConversionPatternRewriter &rewriter) {
 return mlir::SymbolRefAttr::get(userMalloc);
 mlir::OpBuilder moduleBuilder(
 op->getParentOfType<mlir::ModuleOp>().getBodyRegion());
-  auto indexType = mlir::IntegerType::get(op.getContext(), 64);
+  auto indexType = mlir::IntegerType::get(op.getContext(), 32);
 auto mallocDecl = moduleBuilder.create<mlir::LLVM::LLVMFuncOp>(
 op.getLoc(), mallocName,
 mlir::LLVM::LLVMFunctionType::get(getLlvmPtrType(op.getContext()),
@@ -1281,6 +1281,7 @@ struct AllocMemOpConversion : public FIROpConversion<fir::AllocMemOp> {
 mlir::Type heapTy = heap.getType();
 mlir::Location loc = heap.getLoc();
 auto ity = lowerTy().indexType();
+    auto i32ty = mlir::IntegerType::get(rewriter.getContext(), 32);
 mlir::Type dataTy = fir::unwrapRefType(heapTy);
 mlir::Type llvmObjectTy = convertObjectType(dataTy);
 if (fir::isRecordWithTypeParameters(fir::unwrapSequenceType(dataTy)))
@@ -1291,9 +1292,10 @@ struct AllocMemOpConversion : public FIROpConversion<fir::AllocMemOp> {
 for (mlir::Value opnd : adaptor.getOperands())
 size = rewriter.create<mlir::LLVM::MulOp>(
 loc, ity, size, integerCast(loc, rewriter, ity, opnd));
+    auto size_i32 = integerCast(loc, rewriter, i32ty, size);
 heap->setAttr("callee", getMalloc(heap, rewriter));
 rewriter.replaceOpWithNewOp<mlir::LLVM::CallOp>(
-        heap, ::getLlvmPtrType(heap.getContext()), size, heap->getAttrs());
+        heap, ::getLlvmPtrType(heap.getContext()), size_i32, heap->getAttrs());
 return mlir::success();
 }
```

*[~/fortran]patch -p1 -d llvm-project < force-4-byte-values.diff

[~/fortran]cmake --build build

[0/60] 正在构建 CXX 对象 tools/flang/lib/Lower/Runtime.cpp.o

...

[49/49] 生成 ../../../../include/flang/ieee_arithmetic.f18.mod

一旦 LLVM 重新构建完成，再次尝试编译我们的程序。这次，它应该可以在 Node 下无警告地成功编译并运行了。

[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o build/flang/runtime/libFortranRuntime.a -o hello.js

[~/fortran]node hello.js

Hello, World!*********  ***# 为 WebAssembly 编译 BLAS 和 LAPACK

现在我们有一个可以输出 WebAssembly 对象的 Fortran 编译器，让我们构建一些 Fortran 项目。[BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)（基本线性代数子程序）是一组低级别例程，执行线性代数中许多常见操作，包括矩阵和向量乘法的工具。它们是数值计算中的事实标准，有几种不同的 BLAS 例程实现可用。一些实现已经针对特定硬件进行了优化，其他的则因为存在时间较长而进行了优化 —— 最初的 BLAS 例程于 1979 年发布！

让我们获取一份 BLAS 的最新版本“参考实现”，它是用 Fortran 90 编写的，并使用我们上面构建的补丁版 LLVM 进行编译。

[~/fortran]curl -L https://www.netlib.org/blas/blas-3.12.0.tgz > blas-3.12.0.tgz

% 总计 % 已下载 % 传输中 平均速度 时间 时间 时间 当前

Dload Upload Total Spent Left Speed

100 323k 100 323k 0 0 317k 0 0:00:01 0:00:01 --:--:-- 317k

[~/fortran]tar xzf blas-3.12.0.tgz

我们需要修改 `BLAS-3.12.0/make.inc`，告诉它关于我们的 `flang-new` 版本和 Emscripten 工具的信息。修改以下设置，保持该文件中的其他行不变，然后使用 `make` 来构建 BLAS。

```
FC = ../build/bin/flang-new
FFLAGS = -O2
FFLAGS_NOOPT = -O0

AR = emar
RANLIB = emranlib
```

*[~/fortran]cd BLAS-3.12.0

[~/fortran/BLAS-3.12.0]make

../build/bin/flang-new -O2 -c -o isamax.o isamax.f

../build/bin/flang-new -O2 -c -o sasum.o sasum.f

...

emar cr blas_LINUX.a isamax.o ... xerbla_array.o

emranlib blas_LINUX.a

[~/fortran/BLAS-3.12.0]cd ..

[~/fortran]

那进展相当顺利！让我们尝试在为 WebAssembly 编译的 Fortran 子程序中使用它。为了好玩，我们将尝试使用双精度复数。我们将使用 BLAS 2级例程 [`ZGEMV()`](https://netlib.org/lapack/explore-html-3.6.1/dc/dc1/group__complex16__blas__level2_gafaeb2abd9fffa7442b938dc384aeaf47.html#gafaeb2abd9fffa7442b938dc384aeaf47)，它执行矩阵-向量操作。

\[\begin{equation} \mathbf{y} \leftarrow \alpha\mathbf{A}\mathbf{x} + \beta\mathbf{y}, \end{equation}\]

这里 \(\alpha\) 和 \(\beta\) 是标量常数，\(\mathbf{x}\) 和 \(\mathbf{y}\) 是向量，\(\mathbf{A}\) 是矩阵。我们的 Fortran 程序将接收 `alpha`、`beta`、`A`、`X` 和 `Y`，并且有一个固定参数 `N`，使得 \(\mathbf{A}\) 是一个有三行和列的方阵。结果写回到 \(\mathbf{y}\) 中，因此我们声明 `Y` 的意图为 `INOUT`。

```
SUBROUTINE bar(alpha, A, X, beta, Y)
 IMPLICIT NONE
 INTEGER, PARAMETER :: N = 3
 COMPLEX(KIND=8), INTENT(IN) :: alpha, beta, A(N,N), X(N)
 COMPLEX(KIND=8), INTENT(INOUT) :: Y(N)
 EXTERNAL zgemv

 CALL zgemv('N', N, N, alpha, A, N, X, 1, beta, Y, 1)
END
```

*请注意，对于某些 BLAS 例程，长度为一的 `CHARACTER` 字符串控制配置设置。在这里，我们将 `'N'` 作为第一个参数传递。这是我们花费时间和精力在上面构建一个能处理 `CHARACTER` 参数及其 `wasm32` 目标的 `flang-new` 版本的原因之一。

接下来，我们将编写一个C程序来创建一些复杂变量，将它们发送到Fortran和BLAS进行处理，并打印结果。这将让我们知道将双精度复数传递给Fortran并调用BLAS例程是否按预期工作。

```
#include <stdio.h>
#include <complex.h>

extern void bar_(double complex*, double complex*, double complex*,
 double complex*, double complex*);

int main() {
 double complex alpha = 1.;
 double complex beta = 2.*I;
 double complex A[] = {
 1.*I, 4.  , 5.,
 7.  , 2.*I, 6.,
 8.  , 9.  , 3.*I
 };
 double complex X[] = { 0., 1., 2. };
 double complex Y[] = { 3., 4., 5. };

 bar_(&alpha, A, X, &beta, Y);

 printf("Y[0]: %f + %fi, Y[1]: %f + %fi, Y[2]: %f + %fi\n",
 creal(Y[0]), cimag(Y[0]),
 creal(Y[1]), cimag(Y[1]),
 creal(Y[2]), cimag(Y[2]));
 return 0;
}
```

*[~/fortran]./build/bin/flang-new -c bar.f08 -o bar.o

[~/fortran]emcc bar.c bar.o build/flang/runtime/libFortranRuntime.a BLAS-3.12.0/blas_LINUX.a -o bar.js

[~/fortran]node bar.js

Y[0]: 23.000000 + 6.000000i, Y[1]: 18.000000 + 10.000000i, Y[2]: 6.000000 + 16.000000i

至此，我们已经完成了从Fortran 90源代码编译的BLAS，并在WebAssembly下运行！最后，让我们亲自确认这个输出是正确的，

^(31) 请记住Fortran的列主要数组布局。

\[\begin{equation} \begin{split} \alpha\mathbf{A}\mathbf{x} & + \beta\mathbf{y} \\[0.5em] & = \begin{pmatrix} i & 7 & 8\\ 4 & 2i & 9\\ 5 & 6 & 3i \end{pmatrix} \cdot \begin{pmatrix} 0\\1\\2 \end{pmatrix} + 2i \begin{pmatrix} 3\\4\\5 \end{pmatrix}\\[0.5em] & = \begin{pmatrix} 23+6i\\18+10i\\6+16i \end{pmatrix}. \end{split} \end{equation}\]

## 示例：手写数字分类器

这个演示使用[多层感知器（MLP）](https://en.wikipedia.org/wiki/Multilayer_perceptron)人工神经网络来对手绘数字进行分类。用鼠标或触摸屏尝试一下吧！只需在框中绘制一个0-9的数字，分类器将尝试标记您写的是什么数字。根据网络的相对概率，在右侧的图中显示。

这不是一个完美的模型，但对我来说效果相当不错！这个模型的权重已经使用Python进行了预训练，但分类是在运行时使用JavaScript和WebAssembly进行的，在您的浏览器中实时运行。

对于MLP网络，分类过程实质上是重复应用矩阵-向量加法和乘法。在这个演示中，通过一个单独的Fortran子程序来完成重活，利用了BLAS第2级例程[`DGEMV()`](https://netlib.org/lapack/explore-html-3.6.1/d7/d15/group__double__blas__level2_gadd421a107a488d524859b4a64c1901a9.html)。

## 构建LAPACK

[LAPACK](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)（线性代数包）是用于数值解线性代数问题的软件库。它建立在BLAS之上，并且类似地已经成为许多针对特定硬件或系统设计的重新实现的标准。

让我们通过也构建LAPACK的“参考实现”来完成本文。

^(32) 也可从[netlib](https://www.netlib.org/lapack/)获取，发布在修改版BSD许可下。

[~/fortran]curl -L https://github.com/Reference-LAPACK/lapack/archive/refs/tags/v3.12.0.tar.gz > lapack-3.12.0.tgz

% 总计 % 下载 % Xferd 平均速度 时间 时间 当前

Dload Upload Total Spent Left Speed

100 7747k 0 7747k 0 0 4117k 0 --:--:-- 0:00:01 --:--:-- 6655k

[~/fortran]tar xzf lapack-3.12.0.tgz

与BLAS类似，我们需要修改一些配置选项，让LAPACK了解Emscripten和`flang-new`。将文件`lapack-3.12.0/make.inc.example`复制到`lapack-3.12.0/make.inc`，然后进行以下修改。确保用您机器上构建目录的完整路径替换`[...]`，并将文件中的其他选项保持不变。

^(33) 相对路径在这里不起作用。或者，简单地设置选项以读取`flang-new`并在您的`$PATH`上提供它。

```
FC = [...]/build/bin/flang-new
FFLAGS = -O2
FFLAGS_DRV =  $(FFLAGS)
FFLAGS_NOOPT = -O0

AR = emar
RANLIB = emranlib

TIMER = INT_CPU_TIME
```

*然后，使用`make lib`命令构建LAPACK，以创建WebAssembly静态库`liblapack.a`。

[~/fortran]cd lapack-3.12.0

[~/fortran/lapack-3.12.0]进行lib制作

make -C SRC

.../build/bin/flang-new -O2 -c -o sbdsvdx.o sbdsvdx.f

...

emar cr ../../libtmglib.a slatms.o ... dlarnd.o

emranlib ../../libtmglib.a

[~/fortran/lapack-3.12.0]cd ..

[~/fortran]

[~/fortran]文件lapack-3.12.0/liblapack.a

lapack-3.12.0/liblapack.a：当前ar归档

有了这个，可以像前面部分中BLAS例程示例中一样调用LAPACK例程。*  *## 例子：使用线性代数进行多项式插值

以下演示为一组点找到插值多项式，展示在您的Web浏览器中运行的LAPACK例程。

点击绘图以添加新点。将使用[Vandermonde方法](https://en.wikipedia.org/wiki/Vandermonde_matrix)找到一个插值多项式，该多项式将通过所有点。然后，使用[`DGELS()`](https://netlib.org/lapack/explore-html-3.6.1/d7/d3b/group__double_g_esolve_ga1df516c81d3e902cca1fc79a7220b9cb.html#ga1df516c81d3e902cca1fc79a7220b9cb)例程在LAPACK中对此方法给出的线性代数方程进行数值求解。

^(34) 总是可以找到一个包含\(n-1\)个数据点的\(n\)次多项式。然而，当\(n\)很大时，多项式在相邻数据点之间波动很大。这个问题被称为[Runge现象](https://en.wikipedia.org/wiki/Runge%27s_phenomenon)，可以通过使用[样条插值](https://en.wikipedia.org/wiki/Spline_interpolation)来避免。*******
