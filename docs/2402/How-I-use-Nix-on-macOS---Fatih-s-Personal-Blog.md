<!--yml

分类：未分类

日期：2024-05-29 13:23:17

-->

# 我如何在 macOS 上使用 Nix | Fatih 的个人博客

> 来源：[https://blog.6nok.org/how-i-use-nix-on-macos/](https://blog.6nok.org/how-i-use-nix-on-macos/)

## [](#my-personal-history-with-package-managers)我的软件包管理经历

从早在 2005 年起，我就一直使用 GNU/Linux ^(作为我的主要操作系统。我喜欢它的很多东西，但有一件事我不喜欢，那就是它的粗糙。经过多年的摸索和崩溃，我开始寻找更加稳定的东西。这就是为什么当我换工作，公司 ^(能负担得起 Macbook 的时候，我对切换到 Mac 感到兴奋。))

然而，我错过了我以前 Linux 系统的一些东西，其中最主要的是软件包管理。熟悉 Linux 的人会知道，大多数发行版都带有一个预配置的中央软件包仓库。macOS 有 App Store，但它只限于图形用户界面。作为来自 Linux 的用户，我习惯于命令行工具，并发现它们在我的工作中不可或缺。

我很快发现了 macOS 的一个非官方软件包管理器及其附带的软件包仓库，总称为 [Homebrew](https://brew.sh)。我使用了一段时间，效果还可以。然而，我的同事们却在使用 [NixOS](https://nixos.org)，看到他们的工作流程从根本上改变了我对软件包管理器的期望。幸运的是，Nix 是跨平台的，并且在 macOS 上运行良好。自然地，我也试用了一下。

## [](#whats-nix)什么是 Nix

我会尽量简短，以免令人畏惧，但 Nix 可能指的是一些事情。

+   [Nix](https://zero-to-nix.com/concepts/nix-language)，纯函数式编程语言

+   [Nix](https://zero-to-nix.com/concepts/package-management)，这个软件包管理器

+   [Nixpkgs](https://zero-to-nix.com/concepts/nixpkgs)，软件包仓库，包的定义是用 Nix 语言编写的

+   [NixOS](https://zero-to-nix.com/concepts/nixos)，由上述所有支持的 Linux 发行版

在我这种情况下，我开始使用 Nix 软件包管理器和 Nixpkgs 仓库在我的 Mac 上安装软件。我没有打算更换操作系统，也不需要立即编写 Nix 表达式。

## [](#package-management-with-nix)使用 Nix 进行软件包管理

这里有一些基本的软件包管理命令，你可以把它当作备忘单使用。

+   搜索一个软件包（或者，您可以使用 [Web 界面](https://search.nixos.org/packages)）：

    ```
    > nix search nixpkgs ripgrep

    * legacyPackages.aarch64-darwin.ripgrep (14.1.0)
    A utility that combines the usability of The Silver Searcher with the raw speed of grep
    ```

+   安装一个软件包：

    ```
    > nix profile install nixpkgs#ripgrep # or by full path e.g. legacyPackages.aarch64-darwin.ripgrep
    ```

+   列出已安装的软件包：

    ```
    > nix profile list

    Index:              0
    Flake attribute:    legacyPackages.aarch64-darwin.ripgrep
    Original flake URL: flake:nixpkgs
    Locked flake URL:   github:NixOS/nixpkgs/98b00b6947a9214381112bdb6f89c25498db4959
    Store paths:        /nix/store/agbdqb8yjnji2p7hc1ckg0q2fq5a7yn5-ripgrep-14.1.0 
    ```

+   升级一个软件包：

    ```
    > nix profile upgrade legacyPackages.aarch64-darwin.ripgrep # or by index e.g. 0
    ```

+   删除一个软件包：

    ```
    > nix profile remove legacyPackages.aarch64-darwin.ripgrep # or by index e.g. 0
    ```

+   更新注册表（您的本地软件包索引）到最新版本

    ```
    > nix registry pin nixpkgs
    ```

和一些高级命令，展示了 Nix 的真正威力。

如果您可以在 [Nixpkgs GitHub 仓库](https://github.com/NixOS/nixpkgs) 中找到特定版本的软件包，您可以复制提交哈希并将注册表设置为那个时间点。

如果你想撤消影响你个人配置文件的操作，你可以撤销最近的更改，或者回滚到一个已知的良好版本。Nix 包是不可变的，因此升级或卸载不会造成破坏。

这改变了游戏规则，你不必污染系统来运行一次性命令，或者你可以在安装前尝试一个包。由于 Nix 包是确定性的，它们只在第一次运行时构建，之后从缓存中获取并且安装是即时的。

这是另一个改变游戏规则的地方。你可以打开一个带有你需要的所有工具的 shell，而不必安装它们。一个项目可能使用 Node 18，而另一个使用 Node 20。不再需要版本管理器。你甚至可以通过像[devenv](https://devenv.sh)这样的工具进一步扩展功能。

+   清理你的 Nix 存储：

    ```
    > nix store gc

    3196 store paths deleted, 2074.36 MiB freed
    ```

你可以定期运行这个命令来清理掉没有任何依赖的包，从而释放磁盘空间。感觉很不错。

## [](#what-you-may-miss-from-homebrew)你可能会错过的 Homebrew 功能

我想到的唯一一件事是[Homebrew Cask](https://formulae.brew.sh/cask/)。目前我找不到在 Nix 上安装预打包 GUI 应用程序的好方法。我目前使用 App Store 和磁盘镜像来解决这个问题。

## [](#how-to-install-nix-on-macos)如何在 macOS 上安装 Nix

我强烈推荐使用[Determinate Nix Installer](https://github.com/DeterminateSystems/nix-installer)。你甚至可以使用[图形安装程序](https://determinate.systems/posts/graphical-nix-installer/)。[官方安装程序](https://nixos.org/download#nix-install-macos)也可以，但它在操作系统升级后无法继续运行，并且不生成安装收据，这使得如果需要卸载它的话更加困难。总体上，我觉得有一个收据会更加令人放心。

## [](#where-to-go-from-there)接下来如何进行

你完全可以在这里停下来，利用 Nix 赋予你的新能力。但如果你想更进一步，可以尝试使用[nix-darwin](https://github.com/LnL7/nix-darwin)和[home-manager](https://github.com/nix-community/home-manager)，有些人用它们来更声明式地管理系统。我觉得这对我来说有点复杂。你也可以使用 Nix 来管理项目的依赖；我发现[devenv](https://devenv.sh)在这方面很有用，并希望未来能写更多关于它的内容。

* * *

我希望我已经充分展示了使用 Nix 作为 macOS 系统包管理器的好处。这仅仅是我的工作流程，其他人在使用 Nix 时可能以独特和创造性的方式进行方便的使用。这里有一些我阅读过并可以推荐的赞美 Nix 的文章：

希望你会像我一样觉得 Nix 既迷人又强大。如果你遇到任何问题或者只是想分享你有多喜欢它，请告诉我！

*感谢[Aycan](https://twitter.com/aycanirican)和[Taylan](https://twitter.com/taylan_dgn)审阅本文稿件。*
