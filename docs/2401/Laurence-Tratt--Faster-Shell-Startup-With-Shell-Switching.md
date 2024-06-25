<!--yml

类别：未分类

日期：2024-05-27 14:49:51

-->

# Laurence Tratt: Faster Shell Startup With Shell Switching

> 来源：[https://tratt.net/laurie/blog/2024/faster_shell_startup_with_shell_switching.html](https://tratt.net/laurie/blog/2024/faster_shell_startup_with_shell_switching.html)

几天前，Thorsten Ball 写了一篇文章，敦促 Unix 用户[优化他们的 shell 启动时间](https://registerspill.thorstenball.com/p/how-fast-is-your-shell)。这是一些建议值得遵循的。

Unix shell 有两个主要用例：作为一个交互提示（我们通常称之为“命令行”）；以及作为“脚本”或“非交互式命令”语言。我们通常选择一个 shell（bash、fish、zsh 等）并将其用于两种用例。

然而，我们可以针对每种用例使用不同的 shell。人们通常不会这样做，因为这样做没有多少功能性的效用。然而，有一个小小的性能原因需要这样做，我将在这篇文章中进行讨论。我将描述本文中描述的技术为“shell 切换”，因为我不知道它是否已经有一个现有的名称。

## 最小 Shell 开销

让我们首先看一下在没有用户配置时启动和停止非交互式 Unix shell 的绝对最低成本。如果我创建一个没有配置的全新用户帐户（除了一个空的 `~/.zshrc` 以阻止 zsh 要求我进行设置），bash、fish 和 zsh 在我的 OpenBSD 笔记本上运行需要以下时间：

```
$ hyperfine --shell=none "bash -c exit" Benchmark 1: bash -c exit
 Time (mean ± σ):       5.1 ms ±   0.2 ms    [User: 0.7 ms, System: 3.5 ms] Range (min … max):     4.6 ms …   7.3 ms    472 runs  $ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      14.7 ms ±   0.6 ms    [User: 6.3 ms, System: 7.4 ms] Range (min … max):    13.3 ms …  17.8 ms    184 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):       7.5 ms ±   3.1 ms    [User: 1.3 ms, System: 4.2 ms] Range (min … max):     5.9 ms …  21.1 ms    364 runs 
```

笔记本电脑上的基准数据已经够糟糕了，但我也在使用一个奇怪的操作系统——这些数据显然是不可信的。如果我在 Debian 服务器上的一个全新用户帐户上运行相同的基准测试，情况会怎样：

```
$ hyperfine --shell=none "bash -c exit" Benchmark 1: bash -c exit
 Time (mean ± σ):       3.3 ms ±   0.1 ms    [User: 1.5 ms, System: 1.4 ms] Range (min … max):     3.1 ms …   3.9 ms    828 runs   $ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      14.0 ms ±   0.9 ms    [User: 8.8 ms, System: 4.8 ms] Range (min … max):     8.7 ms …  15.9 ms    206 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):       4.2 ms ±   0.2 ms    [User: 1.5 ms, System: 2.3 ms] Range (min … max):     3.6 ms …   5.0 ms    718 runs 
```

两台机器之间的数字讲述了一个相当一致的故事：bash 和 zsh 非常快，但 fish 慢几倍。这对我来说似乎很糟糕，因为我在我的本地机器上使用 fish，但我真的应该担心仅有的 3-15ms 的最小开销吗？Thorsten 认为我应该关心交互式提示，但就我个人而言，我对“非交互式命令”执行同样关心。

我运行许多程序，它们本身运行许多命令。这可以从我在我的编辑器中运行外部命令到在远程机器上运行命令（使用 `ssh example.com "cmd arg1 ... argn"`）的范围。尽管我们经常忽视这一点，但我们指定的命令通常传递给一个 shell 以 `shell -c cmd "arg1 ... argn"` 的方式运行它们，换句话说，这些命令是一行 shell 脚本。shell 的开销是我每次运行这样的命令时被迫承担的成本。

一个明显的问题是：用于运行子命令的 shell 是哪个？令人惊讶的是，这并没有一个简单的答案，不同的程序使用不同的 shell 来运行子命令。有些使用 `$SHELL` 环境变量的内容；有些使用用户数据库中设置的 shell（即 `/etc/passwd` 和其它文件）；有些使用硬编码的 shell（例如 `/bin/sh`）。在大多数情况下，`$SHELL` 和用户数据库中的 shell 是相同的，通常是一个“大” shell，如 bash、fish 或 zsh。在我关心的许多情况下，我的非交互式命令是使用这些大 shell 之一运行的，因此它们的开销是我关心的。

在实践中，当我运行非交互式命令时，我经历的开销远远高于最低限度所示的。幸运的是，我的 fish 和 zsh 配置相当可比（一两年前我从后者迁移到前者）。以我的普通用户帐户执行命令可以显示出 shell 加上我的配置的开销：

```
$ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      88.2 ms ±   3.6 ms    [User: 38.1 ms, System: 92.9 ms] Range (min … max):    83.1 ms …  99.5 ms    31 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):      49.5 ms ±   1.5 ms    [User: 14.7 ms, System: 75.4 ms] Range (min … max):    46.9 ms …  53.0 ms    57 runs 
```

不仅这些时间比最低限度高 10-20 倍，而且它们足够慢，以至于慢慢地对我这个老年人的眼睛可见！而且，这还会因机器而异。我经常使用的服务器之一相当慢（在 CPU 和 IO 方面）：如果相关文件不在缓存中，fish 和 zsh 加上我的配置会增加 300 毫秒（三分之一秒！）的开销。

## 减少 Shell 开销

去年，当我本地和远程执行大量命令时，我特别沮丧。我的普通用户 shell（根据机器不同，可能是 fish 或 zsh）增加了可观的开销。

我所做的第一件事是更改一些配置，以便在非交互式（即“脚本”）模式下跳过它。这有点帮助，但比看起来更难。例如，对于 zsh，我总是被加载 `~/.zlogin`、`~/.zprofile`、`~/.zshenv` 和 `~/.zshrc` 配置文件的时间以及顺序搞混。fish 有一个不同的文件执行模型，让我这可怜的大脑不堪重负。不管我将来可能使用的是什么 shell，它们可能会再次做出不同的改变。

然后我有了不同的想法。我想要一个像 fish 或 zsh 这样功能齐全的 shell 用于交互提示，但我不在乎非交互式命令使用什么。事实上，在基本水平上，大多数 Unix shell 都实现了[相同的基础语言](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)。即使 fish 明确不与此标准兼容，实际上它大部分是兼容的。

大多数现代 Unix 系统都安装了一个小型 shell，位于 `/bin/sh` —— 例如，在 OpenBSD 中是 ksh，在 Debian 中是 dash。这是我们在我的 OpenBSD 笔记本电脑上运行 `/bin/sh` 的新用户帐户：

```
$ hyperfine --shell=none "/bin/sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.0 ms ±   0.2 ms    [User: 0.3 ms, System: 1.1 ms] Range (min … max):     1.6 ms …   3.3 ms    996 runs 
```

即使只有几毫秒，但这比任何“大” shell 都要快。在 Linux 服务器上的性能情况类似：

```
$ hyperfine --shell=none "/bin/sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.0 ms ±   0.1 ms    [User: 1.2 ms, System: 0.6 ms] Range (min … max):     1.6 ms …   2.5 ms    1462 runs 
```

ksh和dash都太简陋了，我不想将它们作为交互式Shell使用，所以我不想彻底切换到它们。相反，我想在非交互式命令中使用`/bin/sh`，在交互提示符中使用fish（或另一个“大型”Shell）。

## Shell切换

结果证明，使用一个Shell来运行非交互式命令，另一个Shell来运行交互提示符非常简单。

首先，我将用户Shell设置为`/bin/sh`（使用`chsh -c /bin/sh`）。然而，当我在终端登录时，我现在使用的是ksh或dash，而我想使用fish或zsh。

`/bin/sh`总是从`~/.profile`加载并执行shell脚本。在该文件中，我们需要确定我们是在运行非交互式命令还是交互式提示符。这让我困惑了一段时间，但事实证明这相当容易。Shell设置了一个环境变量`$-`，如果Shell是交互式提示符，则其中将包含字母“i”。

有了这个知识，最简单的Shell切换版本是将以下内容放入`~/.profile`中：

```
case $- in
 *i* ) exec fish ;; esac 
```

如果我启动一个交互提示符，这个`exec`执行fish（即用`fish`覆盖`/bin/sh`进程），否则它会保持`/bin/sh`。有趣的是，因为`login`设置了`$SHELL`，而fish没有更改该值，所以我的fish shell报告`$SHELL`的值为`/bin/sh`。因此，无论一个程序使用`$SHELL`还是用户数据库中设置的shell来运行子命令，它们总是得到`/bin/sh`的值，这正是我想要的！

我的小`.profile`文件执行起来非常快，对我们的基准测试没有实质性影响：

```
$ hyperfine --shell=none "sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.1 ms ±   0.1 ms    [User: 0.4 ms, System: 1.1 ms] Range (min … max):     1.8 ms …   2.7 ms    1111 runs 
```

我甚至可以看到Shell切换执行fish的交互提示符需要多长时间：

```
$ hyperfine --shell=none "sh -il -c exit" Benchmark 1: sh -il -c exit
 Time (mean ± σ):     101.5 ms ±   7.7 ms    [User: 44.1 ms, System: 96.6 ms] Range (min … max):    94.5 ms … 123.2 ms    29 runs 
```

如果我直接用fish运行这个基准测试，得到的基准数值在统计上是无法区分的。Shell切换很快！

一个问题是：我们在`~/.profile`中是否*需要*一些配置？我个人不需要。如果你需要，好消息是大多数时间都花在shell启动上，通常用于执行人类在交互提示符中想要的东西。在`~/.profile`中添加一两行代码，例如将目录添加到`$PATH`中，似乎不太可能增加太多开销。

## 防止Shell执行失败

这些年来，有几次我发现自己在使用一个shell二进制文件不起作用的系统上：要么它不存在；要么它无法执行。在这两种情况下，这通常发生在系统更新期间，最终结果是我无法登录。

最小程度地扩展“Shell切换”以便`~/.profile`捕获这些明显情况是可能的，这意味着如果我的“大型”Shell不起作用，`/bin/sh`最终会成为我的备用交互提示符：

```
case $- in
 *i* )
  if command -v fish > /dev/null; then
  fish --version > /dev/null && exec fish  echo "Couldn't run 'fish'" > /dev/stderr
  fi
 ;; esac 
```

你可能会注意到这里有一定程度的偏执。首先，它会快速检查一个名为“fish”的二进制文件是否存在（如果不存在，则`command -v fish`返回非零）。其次，它尝试实际运行 fish 看看二进制文件是否至少能够最小化地运行而不退出（使用`--version`标志作为代理）。只有在成功运行后，它才会`exec fish`。

## 摘要

如果你关心非交互式命令执行的速度，那么 shell 切换是一种简单的优化技术，可以在不影响交互提示配置的情况下进行优化。它涉及：

1.  设置你的 shell 为`/bin/sh`。

1.  将代码放入`~/.profile`中，当检测到交互式提示时，`exec`另一个 shell。

我已经使用这个设置运行了将近12个月，它没有给我造成任何问题，但却给了我一个有用的速度提升！

*更新（2024-01-16）* 我在`~/.profile`中提供的原始代码包含`[[...]]`，这不符合 POSIX 规范，并导致 dash（以及可能是其他严格的 shell）出错。我已将其替换为 POSIX 兼容的`case ... esac`。我重新运行了受影响的两个基准测试，但两者都没有在统计上显著变化。

*更新（2024-01-18）* 修正拼写错误：`login`设置`$SHELL`，而不是`/bin/sh`。

[更新的](/laurie/blog/2024/some_reflections_on_writing_unix_daemons.html)

2024-01-16 14:50

[更旧的](/laurie/blog/2024/choosing_what_to_read.html)

如果你想获取新博客文章的更新：关注我

[长毛象](https://mastodon.social/@ltratt)

或

[Twitter](https://twitter.com/laurencetratt)

；或

[订阅 RSS feed](../blog.rss)

；或

[订阅电子邮件更新](/laurie/newsletter/)

：

### 脚注

### 评论
