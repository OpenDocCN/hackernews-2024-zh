<!--yml

category: 未分类

date: 2024-05-27 14:28:33

-->

# 分离关注点在交叉编译中的应用

> 来源：[https://nixcademy.com/2024/01/30/cross-compilation-with-nix/](https://nixcademy.com/2024/01/30/cross-compilation-with-nix/)

# 分离关注点在交叉编译中的应用

![博客文章的标题图像 \](img/6473a66ae1d8e85f6512f9ae6f1f520d.png)

📆

2024年1月30日，作者：Jacek Galowicz

在多平台上管理复杂的C++项目通常是令人沮丧和耗时的任务。然而，即使是经验丰富的软件开发人员也会面临这个共同挑战，这并不是一场不可避免的斗争。想象一下，交叉编译不仅可行，而且高效且不那么麻烦的世界。我注意到在与其他工程师的几次友好讨论中，许多人对Nix未必是发明，但也使用了分离关注点的类型不熟悉。

早期文章[*C++ with Nix in 2023, Part 2: Package Generation and Cross-Compilation*](../../../../2023/11/16/cpp-with-nix-in-2023-part-2-package/)主要集中在从头开始创建新包和解释`mkDerivation`等内容，而在本文中，我们将稍微高层次地看待在Nix中如何优雅地处理交叉编译，这可以改善整体项目健康和开发速度，同时降低成本。

## 使得交叉编译变得困难的原因是什么？

总体而言，交叉编译不应该很难：我们只需要一个能为目标架构生成代码的编译器，并让它将编译后的代码链接到同样为目标架构编译的库上。

所以为什么每当为巨大的现实项目执行它时，它变得困难？

典型的困难往往不是交叉编译本身的问题，而是与管理依赖项有关的步骤逐渐走下坡路：

+   **产品**：我们假设一个已经庞大并且有复杂构建系统的项目

+   **交叉编译器**：我们需要一个

+   **外部库**：我们需要它们的交叉编译版本

+   **分发**：如果我们使用共享库，我们将需要提供一种方法将其与应用程序打包在一起。或者，我们使用静态链接来获得一个更容易分发的大型二进制文件。

然后步骤是：

1.  现在可以通过两种方式获得交叉编译器：

    1.  某人创建了一个编译器引导脚本，旨在与Linux发行版无关。这个脚本将进入一个[Dockerfile](https://docs.docker.com/engine/reference/builder/)或者直接进入构建系统。

    1.  使用交叉编译器包。由于这将项目与某个包分发绑定，我们现在被迫使用该发行版或者使用Docker。

1.  包管理器不允许我们在同一系统上安装本地和外部目标库。开发人员也使用不同的包管理器。因此，我们通常会自行为目标平台构建外部库：

    1.  这些都进入同一个Docker镜像。

    1.  或者项目构建系统也为我们构建它们。

1.  由于我们没有包分发基础设施，通常我们选择静态构建路径，并最终将静态链接调整到我们的项目构建系统中。

结果是这两种之一：

1.  一个巨大、复杂的、单片式的构建系统，不仅管理我们实际项目的构建，还管理编译器和所有库的构建。

    +   安装起来很痛苦，而且编译代码需要很长时间。

    +   只有高级开发人员被允许触及脆弱部分。

    +   复杂的升级。

1.  一个Docker镜像，很好地将与普通非交叉编译不同的部分抽象出来。

变体*B*看起来最清晰，但在行业中很少见到其干净的执行。至少在我的经验中，开发团队最终会创建大量的复杂性，形成变体*A*的形式。

特别是在与静态链接结合使用时，开发人员经常决定构建系统仅构建静态二进制文件，因为同时维护动态和静态链接会使构建系统过于复杂。

[CMake](https://cmake.org/) 和 [meson](https://mesonbuild.com/) 通常都可以正确使用，使项目描述与链接方法无关，然后通过命令行参数简单选择方法。但是，我没有看到很多大型商业实际项目在不带来太多麻烦的情况下仍然可能使用这种方法。

我经常问自己：“为什么这些构建系统完全不按照官方文档和教程中建议的方式操作？” 根据我今天的经验，我认为答案很简单：

## 关注点分离。

**构建系统并非设计用于管理依赖关系。** 不明白情况的情况下，开发人员仍试图这样做。结果是难以更改、扩展、维护和升级。

如前言所述，我们将看看Nix如何使这一切变得更好。

## 示例应用

让我们构建一个示例应用程序，它在运行时依赖于[OpenSSL](https://www.openssl.org/)和[Boost](https://www.boost.org/)。它简单地从标准输入读取字符流，并使用OpenSSL计算[SHA256](https://en.wikipedia.org/wiki/SHA-2)哈希值。我们使用boost依赖来停止时间 - [C++ STL](https://en.cppreference.com/w/cpp/chrono)也可以为我们做到这一点，但那样我们就不会对一个巨大的外部库有另一个很好的依赖。

程序大约有60行LOC（源代码行数）。我将代码上传到GitHub仓库：[https://github.com/tfc/cpp-cross-compilation-example](https://github.com/tfc/cpp-cross-compilation-example)

让我们称这个应用为`minisha256sum`，并为它编写一个[`CMakeLists.txt`](https://cmake.org/)文件：

```
cmake_minimum_required(VERSION 3.27)
project(minisha256sum)

find_package(OpenSSL REQUIRED)
find_package(Boost REQUIRED COMPONENTS chrono)

add_executable(minisha256sum src/main.cpp)
target_link_libraries(minisha256sum Boost::chrono OpenSSL::SSL)
set_property(TARGET minisha256sum PROPERTY CXX_STANDARD 20)

install(TARGETS minisha256sum DESTINATION bin)
```

CMake为查找外部库提供了很好的[标准设施](https://cmake.org/cmake/help/latest/command/find_package.html)。这种方式使得构建系统可能保持简单（它看起来仍然相对于其他语言生态系统来说有些嘈杂，因为这就是C++构建系统的外观）。

现在可以通过典型的CMake步骤构建此项目：

```
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
```

让我们来检查它是否工作：

```
$ ./minisha256sum < src/main.cpp
cb8829956b86a05cd4bf374e95f4ae3928644f4b79cef82ef89529c2ef65f004 0 milliseconds
$ sha256sum < src/main.cpp
cb8829956b86a05cd4bf374e95f4ae3928644f4b79cef82ef89529c2ef65f004  -
```

它提供与来自GNU `coreutils`的[`sha256sum`应用程序](https://www.gnu.org/software/coreutils/)相同的哈希值，这应该足够好了。该应用程序并非真正优化，但对于本文的其余部分并不重要。

### 使用Nix打包

要获得良好的`nix build`和`nix run`工作流程，我们需要提供一些Nix表达式。让我们从一个已经反映了我们依赖结构的`package.nix`开始：

```
# file: package.nix
{ stdenv, lib, cmake, boost, openssl }:

stdenv.mkDerivation {
 name = "minisha256sum";

 src = lib.fileset.toSource {
 root = ./.;
 fileset = lib.fileset.unions [
 ./src
 ./CMakeLists.txt
 ];
 };

 nativeBuildInputs = [ cmake ];
 buildInputs = [ boost openssl ];
}
```

当跨平台编译此应用程序时，我们需要：

+   在构建主机上运行但为目标平台编译的编译器

+   在主机上运行的CMake

+   Boost和OpenSSL，但构建为目标

`nativeBuildInputs`意味着*“在构建主机上的编译时依赖项”*，而`buildInputs`意味着*“目标运行时依赖项”*。[`nixpkgs`文档在这方面有更详细的描述](https://nixos.org/manual/nixpkgs/stable/#ssec-stdenv-dependencies)。

要从此表达式创建可构建、可安装和可运行的软件包，我们需要应用`callPackage`函数，这是[Nix领域中的一种众所周知的模式](https://nixos.org/guides/nix-pills/callpackage-design-pattern)：它会自动从我们在`package.nix`第一行中可以看到的`pkgs`中填充所有函数参数，这些参数碰巧是我们的依赖项。

```
minisha256sum = pkgs.callPackage ./package.nix { }
```

在项目的`flake.nix`文件中，我们[在这一行进行此调用](https://github.com/tfc/cpp-cross-compilation-example/blob/main/flake.nix#L14)。有了这个设置，我们现在可以在不手动处理构建命令的情况下运行它（我这里没有隐藏代码：`mkDerivation`通常知道如何在提到CMake作为依赖项时构建CMake项目）。将其推送到存储库后，我们甚至可以从不同的计算机上进行此操作，而无需先克隆存储库：

```
$ nix build github:tfc/cpp-cross-compilation-example
$ file result/bin/minisha256sum
result/bin/minisha256sum:
ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), dynamically linked,
interpreter /nix/store/7jiqcrg061xi5clniy7z5pvkc4jiaqav-glibc-2.38-27/lib/ld-linux-x86-64.so.2,
for GNU/Linux 3.10.0, not stripped

# Run directly over via `nix run` without building it first:

$ result/bin/minisha256sym < ~/some-file.zip
f7cb5ade7906f364a0d4d11478a4b9f25c86d0b3381a5b3907e2c49b31a00fee 3 milliseconds

$ nix run github:tfc/cpp-cross-compilation-example < ~/some-file.zip
f7cb5ade7906f364a0d4d11478a4b9f25c86d0b3381a5b3907e2c49b31a00fee 3 milliseconds
```

## 跨编译

这是一个快速的方法：

假设我们在64位英特尔PC上，我们可以像这样创建多个静态/动态交叉编译的软件包：

```
static = pkgs.pkgsStatic.callPackage ./package.nix { };
aarch64 = pkgs.pkgsCross.aarch64-multiplatform.callPackage ./package.nix { };
aarch64-static = pkgs.pkgsCross.aarch64-multiplatform.pkgsStatic.callPackage ./package.nix { };
```

属性`pkgs.pkgsStatic`和`pkgs.pkgsCross.aarch64-multiplatform`包含它们自己版本的`callPackage`，但它们附带了针对所选目标平台调整过的整个`pkgs`软件包列表。

还有`pkgs.pkgsCross.mingwW64`，它使用[minimalist GNU环境 for Windows `mingw`](https://www.mingw-w64.org/)为[Microsoft Windows](https://en.wikipedia.org/wiki/Microsoft_Windows)编译二进制文件。

每当我们使用这些专门的`callPackage`实现之一来调用我们的`package.nix`函数时，就会发生这种情况：

![pkgs.callPackage可以分析不同类型的依赖关系](img/463b6e0a3b2c467ff9bcc92f2b2532b9.png)

`pkgs.callPackage`可以分析不同类型的依赖项

因为我们在编译时依赖和运行时依赖之间进行了结构化分离，交叉`callPackage`函数现在可以填充每个包依赖项的正确版本。

这样，构建系统不需要过多了解发生了什么：Nix为给定的包变体创建正确的构建环境（类似于干净的Docker镜像的方法，但只有确实需要的依赖项，并且在定义时具有更少的开销），构建系统简单地使用给定的编译器并通过CMake/meson特定的环境变量（由Nix设置）定位给定的依赖项，并构建项目。 （它也适用于类似于[GNU Automake/Autoconf](https://www.gnu.org/software/automake/)和[GNUMake](https://www.gnu.org/software/make/)等的构建系统组合）

我使用以下组合测试了这个例子：

| `x86_64-linux` | ✅ | ✅ | ❌ | ❌ | ✅ |
| --- | --- | --- | --- | --- | --- |
| `aarch64-linux` | ✅ | ✅ | ❌ | ❌ | ✅ |
| `x86_64-darwin` | ✅ | ✅ | ✅ | ❌ | ✅ |
| `aarch64-darwin` | ✅ | ✅ | ✅ | ✅ | ✅ |

有趣的是，如果我们在Linux上从相同架构交叉编译到相同架构，我们得到的包与本地`pkgs.callPackage`版本完全相同，所以Nix甚至不需要重新构建它。

符号 ✅ 在所有情况下包括静态/动态链接，但不适用于Windows。

在Nixpkgs存储库中未实现带有❌符号的条目。如果需要，可以实现它。通常，公司要么实现功能并上游它，要么提供资金以实现它。

## 概述

展示的技巧表明，我们有效地将**依赖管理**与**构建系统**分开。我与许多开发者谈论过这个问题，他们从未考虑过这种分离。原因可能很简单：因为没有一个好的依赖管理技术，实现这种分离并不容易。我觉得Docker在部署中的作用比在开发中更大。

这种分离的优势是巨大的：

+   更简单的构建系统，即使对于非高级开发者也易于扩展

+   根据构建系统参数自由选择动态和静态链接

+   广泛支持从不同主机架构和操作系统编译到/从的支持

+   不在一个构建系统中管理编译器、依赖项和项目使一切*模块化*：

    +   更新工作更少

    +   每位开发者的快速设置时间

    +   可缓存的依赖项

    +   在其他项目中更轻松地重用各个模块

这次我们只构建了二进制文件，没有[容器镜像](../../../../2023/09/04/nixos-multi-php-version-container/)，[`systemd-nspawn`镜像](../../../../2023/08/29/nixos-nspawn/)，虚拟机或磁盘镜像。不过，如果需要的话，这些很容易添加。

如果你想评估Nix，甚至在你的真实项目中使用它，请毫不犹豫地[联系我们](../../../../meet.html)！我们有丰富的经验，特别是在低级别的C++项目中。与我们联系，看看如何简化你项目的复杂性。
