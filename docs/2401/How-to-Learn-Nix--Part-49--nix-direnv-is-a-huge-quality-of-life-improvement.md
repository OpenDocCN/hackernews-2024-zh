<!--yml

类别：未分类

日期：2024-05-27 15:20:01

-->

# 如何学习 Nix，第四十九部分：nix-direnv 是一个巨大的生活质量改善

> 来源：[`ianthehenry.com/posts/how-to-learn-nix/nix-direnv/`](https://ianthehenry.com/posts/how-to-learn-nix/nix-direnv/)

我之所以发现了一篇古老的博客文章，是因为我有了两年多以来第一次对 Nix 有新的看法。

我想说的是：[`nix-direnv`](https://github.com/nix-community/nix-direnv) 真的很棒。它解决了我遇到的每个`nix-shell`问题，而且比以前的临时解决方案要好得多。

这很重要，因为我*主要*只是使用 Nix 来记录和安装每个项目的本地依赖。我也会用它来安装“全局”工具，但那是[很少很有趣的](https://ianthehenry.com/posts/janet-game/how-to-patch-emacs/)，而我如今与 Nix 的大部分交互都是编辑小小的`shell.nix`文件。

但要让我感觉*好*使用 Nix 还需要一些努力。首先，shell 不会注册 GC 根，这意味着每次收集垃圾时，您都必须重新下载您正在处理的项目的所有依赖。我们在第三十七部分克服了这个障碍，通过制作一个围绕`nix-shell`的自定义包装器来正确设置 GC 根，但这实际上是相当困难的。

另外，Nix 相当坚持你使用*bash*作为交互式 shell。我在 Nix 经典中找到了一个解决方法，但在 nix-develop 中实际上未能使`nix develop`同样可用。

[`nix-direnv`](https://github.com/nix-community/nix-direnv) 解决了这两个问题。它不是生成一个新的 shell，而是将环境变量添加到您现有的 shell 中。并且当它评估`shell.nix`时，它会自动将结果注册为 GC 根。

它还仅在`shell.nix`实际更改时重新评估，这意味着在典型情况下没有启动时间。相比之下，我的 GC 根安装包装器需要大约 750ms 打开一个典型的 shell（原始的`nix-shell`，不涉及 GC 根评估过程，只需 400ms）。这听起来并不算很长，因为它确实不长——我在我只能称之为超级计算机的设备上运行 Nix。但最初我是在一台比起菌理论还早的笔记本电脑上安装了 Nix，它的启动延迟更加令人讨厌。

`nix-direnv` 还会在 `shell.nix` 更改时自动更新环境，所以每次添加依赖时都不需要关闭和重新打开`nix-shell`。这不仅在人体工程学上更好，而且意味着每次添加依赖或退出项目时都不会弄乱您的 shell 历史记录。

我以前从未使用过[`direnv`](https://direnv.net/)，直到现在我唯一使用它的东西就是管理我的 Nix shells。但它是一个通用的管理每个目录环境变量的工具，这基本上就是`nix-shell`的全部内容。`nix-shell`还可以注册 bash 函数 - 如果你使用的是 bash - 这对于想要用它调试推导很有用。但对我来说，环境变量是我真正需要的全部。

`direnv`对 Nix 有一些内置支持，但不是很好；[direnv 发布了一张表格，概述了一些使用`nix-direnv`的优点](https://github.com/direnv/direnv/wiki/Nix#some-factors-to-consider)。`nix-direnv`是一种什么样的插件(?)，用更好的东西替代了原生的 Nix 支持。而且非常棒。它使 Nix 的“可重复的开发环境”方面变得很顺畅™。而且使用起来很容易：

首先，安装`nixpkgs.direnv`和`nixpkgs.nix-direnv`。

我用`nix-env`安装了它们，使用的是我在 part 22 中编写的相同的声明式封装。如果你以不同的方式安装`nix-direnv`，下面的内容会有所不同。

安装`nix-direnv`不会“启用”插件；你必须单独告诉`direnv`它：

```
mkdir -p ~/.config/direnv
echo 'source ~/.nix-profile/share/nix-direnv/direnvrc' > ~/.config/direnv/direnvrc
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc 
```

一旦你这样做了，你就必须在每个想要 nix-shellify 的目录中运行以下命令：

```
echo 'use nix' > .envrc
direnv allow 
```

就这样。就这样！现在每次你进入那个目录，你都会有&mldr;

```
$ cd ~src/project
direnv: loading ~/src/project/.envrc
direnv: using nix
direnv: nix-direnv: using cached dev shell
direnv: export +CONFIG_SHELL +HOST_PATH +IN_NIX_SHELL +MACOSX_DEPLOYMENT_TARGET +NIX_BUILD_CORES +NIX_CFLAGS_COMPILE +NIX_COREFOUNDATION_RPATH +NIX_DONT_SET_RPATH +NIX_DONT_SET_RPATH_FOR_BUILD +NIX_ENFORCE_NO_NATIVE +NIX_IGNORE_LD_THROUGH_GCC +NIX_INDENT_MAKE +NIX_NO_SELF_RPATH +NIX_STORE +PATH_LOCALE +SOURCE_DATE_EPOCH +XDG_DATA_DIRS +__darwinAllowLocalNetworking +__impureHostDeps +__propagatedImpureHostDeps +__propagatedSandboxProfile +__sandboxProfile +buildInputs +builder +configureFlags +depsBuildBuild +depsBuildBuildPropagated +depsBuildTarget +depsBuildTargetPropagated +depsHostHost +depsHostHostPropagated +depsTargetTarget +depsTargetTargetPropagated +doCheck +doInstallCheck +dontAddDisableDepTrack +gl_cv_func_getcwd_abort_bug +name +nativeBuildInputs +nobuildPhase +out +outputs +patches +phases +propagatedBuildInputs +propagatedNativeBuildInputs +shell +shellHook +stdenv +strictDeps +system -PS2 ~PATH
```

噢。好像不太好。

默认情况下，`direnv`会打印它添加、删除或更改的每个环境变量。如果你用它来处理像凭据之类的东西，这是有意义的，但对于 Nix shells 来说，这只是浪费滚动条。

没有简单的方法来禁止打印那个巨大的`export`行，但是你可以通过在你的`.zshrc`中添加类似以下内容来将其屏蔽掉：

```
export DIRENV_LOG_FORMAT="$(printf "\033[2mdirenv: %%s\033[0m")"
eval "$(direnv hook zsh)"
_direnv_hook() {
  eval "$(direnv export zsh 2> >(egrep -v -e '^....direnv: export' >&2))"
}; 
```

(正则表达式中的`.`排除了行首的“暗文本”控制字符。)

这样就去掉了巨大的导出行而不移除其他输入。现在：

```
$ cd ~src/project
direnv: loading ~/src/project/.envrc
direnv: using nix
direnv: nix-direnv: using cached dev shell
```

啊。好多了。

我已经使用`nix-direnv`几个月了，我必须说：我希望我早点安装它。它比默认的`nix-shell`体验*好多了*，我很高兴我可以摆脱多年来积累的特殊技巧。

&mldr;几乎。这个方法唯一不帮助的是`nix-shell -p`。`nix-shell -p`是一种有用的方式，可以“临时”安装软件包而不实际将它们放在你的 PATH 上，我仍然使用我的 zsh hack，以便`nix-shell -p`不会将我置于 bash 会话中。虽然我很少这样做，但我可能只能忍受它。
