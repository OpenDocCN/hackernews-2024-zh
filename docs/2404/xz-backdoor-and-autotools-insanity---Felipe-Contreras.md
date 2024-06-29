<!--yml

分类：未分类

日期：2024-05-27 12:55:37

-->

# xz后门和autotools的疯狂 | 费利佩·康特雷拉斯

> 来源：[https://felipec.wordpress.com/2024/04/04/xz-backdoor-and-autotools-insanity/](https://felipec.wordpress.com/2024/04/04/xz-backdoor-and-autotools-insanity/)

十多年来，我一直在争论GNU Autotools过于复杂、不必要且愚蠢。最新的xz后门只是火上加油。

首先，先来点背景。我已经在Linux上做与软件包相关的事情超过20年了。我熟悉Debian、RPM和Arch Linux的打包系统，还包括手动编译整个系统（[LFS](https://www.linuxfromscratch.org/lfs/)）以及为嵌入式设备交叉编译系统（[buildroot](https://buildroot.org/)）。我还维护过Windows安装程序。

另外，我还有使用autotools、make、ninja和几个自定义构建系统的经验。

所以我对图书馆的构建方式有很好的理解，但在我的经验中，几乎没有人知道如何去做。在这里，你将看到一个明显的例子。

这篇文章不是关于后门本身的，肯定有许多研究人员正在处理这个问题。这篇文章关于的是首次引入后门的原因以及如何确保它不再发生。

如果xz项目没有使用autotools，安装后门是不可能的。就是这么简单。

## GNU Autotools入门

要理解为什么后门如此容易被引入，你需要首先了解autotools是什么。基本上它是一个构建系统，但它是一个不必要复杂的构建系统。

我将描述典型的步骤序列，以便你大致了解：

1.  运行 `autogen.sh` 脚本（生成 `configure`）

1.  运行 `configure` 脚本（生成 `Makefile` 文件）

1.  运行 `make`

1.  运行 `make install`

现在，虽然 `autogen.sh` 脚本是典型的，但并不是必需的，大多数情况下你可以直接运行 `autoreconf --force --install`，它基本上会做同样的事情。

一切就位后，你可以运行 `configure` 脚本，它会进行运行时检查以查看系统的功能，并且它的主要目的是生成 `Makefile` 文件，这样你就可以执行下一步。

现在，我要揭开惊喜，告诉你，就在这一步，[引入了后门](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27#design-specifics)，特别是这一行：

```
AC_CONFIG_COMMANDS([build-to-host], [eval $gl_config_gt | $SHELL 2>/dev/null], [gl_config_gt="eval \$gl_[$1]_config"]) 
```

他们通过引入几层间接性很好地隐藏了这实际上做了什么，但本质上命令在于 `gl_localedir_config` 中，他们自己通过称之为 `gl_[$1]_config` （其中 `$1` 是 `localedir`）来引入它。

然后你执行 `make` 和 `make install`，但在那时代码已经被篡改了。

为什么没人发现这个问题呢？

是的，它被混淆了，但是 `eval $anything | $SHELL` 看起来非常可疑，即使对我这个未经安全分析训练的人也是如此。开源开发者是瞎了吗？

## “自动工具”的“魔力”

自动工具的一个非常酷的特性是，你不必每次都执行每个步骤。这意味着一个人在一台机器上运行 `autoreconf` 会生成 `configure` 脚本，然后打包结果，这样其他人只需运行该脚本，即使他们没有安装自动工具。

一台机器可能运行 `./configure` 并发现 [ifunc](https://sourceware.org/glibc/wiki/GNU_IFUNC) 不可用，但在另一台机器上可能可用。

因此，`configure` 脚本是通用的，但是生成的 `Makefile` 在不同机器上可能会有所不同。因此，当您执行 `make dist` 时，`Makefile` 不包括在压缩包中，只有从 `configure.ac` 生成的 `configure` 脚本。

相当巧妙，对吧？

不，我们很快就会看到为什么不是这样，但长话短说：如果压缩包 `xz-5.6.1.tar` 包含不在git存储库中的文件，但是已经生成了，有什么能阻止恶意行为者修改这些文件呢？**没有**。发行版必须相信创建压缩包的人没有包含 `make dist` 在干净的git检出中没有包含的任何内容，而且这个检查是非常困难的。

其中一个文件是 `m4/build-to-host.m4`，它应该来自[gnulib](https://git.savannah.gnu.org/gitweb/?p=gnulib.git;a=blob;f=m4/build-to-host.m4;h=f466bdbd84abdf60e8305fa7adc12c74d7f05a8a)，并且当您运行 `autoreconf --install` 时复制，但是卑劣的行为者对其进行了修改。这是很多分析师提到的文件，因为那里的修改更容易被发现，但实际上并不危险，因为用户不会运行它。危险的修改在于 `configure` 脚本中，用户会运行它。

例如，Debian 的维护者可能决定在他的机器上运行 `make dist`，将结果与官方压缩包进行比较，并发现 `configure` 脚本不同，但这可能是因为官方压缩包是使用更新版本的autoconf或gnulib生成的。因此，它们不同可能完全是无害的。

根据我的经验，由自动工具生成的文件在不同机器上是截然不同的，因为每个人都在使用不同版本的这些工具。

自动工具的方便之处在于生成一个不需要自动工具或gnulib开发版本就能运行的 `configure` 脚本，现在正在咬我们的后腿。

因此，`configure.ac` 文件包含一行 `AM_GNU_GETTEXT([external])`，该宏位于 `m4/gettext.m4` 中，调用了 `gl_BUILD_TO_HOST([localedir])`，该宏位于 `m4/build-to-host.m4` 中。恶意行为者在该宏中插入了代码，然后运行了 `autoreconf`，生成了安装了后门的 `configure` 脚本。

最初之所以可能是因为GNU Autotools实在是太复杂了，**没有人**真正检查它生成的混乱。

即使在一个未被妥协的代码树中，生成的`configure`脚本也有**25,697行**的混淆Shell脚本。祝你好运，在那里找到任何恶意内容。

恶意开发者没有在tarball中包含他们的`m4/build-to-host.m4`，因为`configure`脚本已经包含了受污染的代码，但如果某些维护者运行了`autogen.sh`脚本，`configure`将会用良性版本重新生成。包含m4脚本确保即使在这种情况下，后门也会保留。即使你执行`aclocal --force --install`，恶意的m4脚本仍然存在（由于某种原因，`aclocal`不会复制已经存在的脚本）。

## 完全不必要

这个悲伤故事中最令人难过的部分是，根本不需要自动工具。它**没有任何用处**。

让我们创建一个简单的库：

```
#include <hello.h>

const char *hello(void)
{
	return "hello";
} 
```

我们按照使用libtool和gettext构建库的步骤，并执行`make dist`。在tarball中生成的文件包含52,466行：这比一个只有一个函数的库更多，超过**五万行**。

实际上我们真正需要构建库的时候只需要这些：

```
gcc -shared -I. -fPIC main.c -Wl,-soname,libhello.so.0 -o libhello.so.0 
```

到底自动工具在做什么？好吧，这里有一个检查：

```
checking for gcc option to enable C11 features... none needed 
```

从几十行看似难以理解的代码来看，似乎这是试图使用C11特性（比如匿名结构体）编译C程序，而没有指定任何标志。如果失败，则尝试使用`-std=gnu11`，如果成功，则在后续的编译中使用该标志。

为什么？我不使用C11，并且我从未告诉自动工具要确保支持C11。

想象一下，自动工具正在进行无数不必要的检查，以及在上帝知道多少软件包构建中的情况。

好吧，也许我有些不公平，也许确实有些情况下这些检查是必要的，例如在交叉编译到其他架构时……但这实际上是我的专业领域，事实上自动工具使得交叉编译变得更加**困难**。像[scratchbox2](https://github.com/sailfishos/scratchbox2)这样的项目正是为了应对过多依赖于自动工具运行时检查的代码而创建的。只需谷歌“无法在交叉编译时运行测试程序”。

当我还在为[Nokia N9](https://en.wikipedia.org/wiki/Nokia_N9)开发时，我写了一篇博客文章，详细解释了为什么我们使用了[scratchbox](https://felipec.wordpress.com/2009/06/07/installing-scratchbox-1-and-2-for-arm-cross-compilation/)，并从一个在ARM交叉编译上`configure`脚本失败的案例开始。解决方案是手动设置系统的特性，例如`ac_cv_func_posix_getgrgid_r=yes`。但如果我们必须手动指定系统具有POSIX兼容的`getgrgid_r`，我们也可以**在Makefile上**做。

```
make HAVE_POSIX_GETGRGID_R=yes 
```

这实际上是你如何向git的构建系统传递配置的方式，而git的构建系统只是Makefiles：`make NO_POSIX_GOODIES=UnfortunatelyYes`。但这在Windows上是默认的，所以实际上你不需要指定这个。

感谢autotools一无是处。

## 解决愚蠢的问题

让我们来看看autoconf为xz生成的一个配置：

```
/* The size of 'size_t', as computed by sizeof. */
#define SIZEOF_SIZE_T 8 
```

`sizeof(size_t)`有什么问题？

根据生成的注释，HP的C编译器HP92453-01 B.11.11.23709.GP在解释类似于`int a3[[(sizeof (unsigned char)) >= 0]]`的声明时存在bug，因此需要将其强制转换为`long int`以获取正确的数字。没错，但那是在2001年，为什么我们要强制成千上万的软件包做这个不必要的检查？也许早就修复了。即使没有，如果在HP的错误C编译器中确实存在问题，那这与数组内的`sizeof()`无关。即使`sizeof(size_t)`出现问题，那也是**他们的问题**。如果真的有人试图为该平台编译xz，他们可以简单地修补源代码。为什么**每个人**都需要检查这个问题？

你不会因为那**一次**有人烫伤自己而在每杯咖啡上写上“小心，咖啡很烫”。

不。不要为最低公共分母设计。这是对一个永远不会发生的事情过早担心。

> “我编造问题，然后写过于复杂的烂代码来解决它们。”
> 
> [Linus Torvalds](https://lore.kernel.org/lkml/CAHk-=wgZEHwFRgp2Q8_-OtpCtobbuFPBmPTZ68qN3MitU-ub=Q@mail.gmail.com/)

当有人确实在使用HP的C编译器编译xz时才会担心。

一个简单的Makefile可以很好地构建liblzma。作为概念验证，我在自己的liblzma项目中创建了它：[Makefile](https://github.com/felipec/liblzma/blob/master/Makefile)。它构建并工作得非常好，尽管需要更多的微调来支持更多的配置，但对于几个小时的工作来说，这已经不错了。

## 后门的起源

玩弄xz的构建系统之后，我现在清楚地知道了一切的起因。

```
// Set up the locale and message translations.
tuklib_gettext_init(PACKAGE, LOCALEDIR); 
```

为了正确设置翻译，我们需要指定翻译文件的位置。在我的系统中，该位置是“/usr/share/locale”。但你应该用类似于C编译器标志`-DLOCALEDIR=\"/usr/share/locale\"`来定义这个变量，这就是xz所做的：`-DLOCALEDIR=\"$(localedir)\"`。

但根据GNU的[gettext手册](https://www.gnu.org/software/gettext/manual/html_node/src_002fMakefile.html)，应该是`-DLOCALEDIR=$(localedir_c_make)`。这是生成的Makefile的一部分：

```
prefix = /usr
datarootdir = ${prefix}/share
localedir = ${datarootdir}/locale
localedir_c = "/usr/share/locale"
localedir_c_make = \"$(localedir)\" 
```

我们可以清楚地看到这两件事是等效的。

gnulib的m4宏[build-to-host.m4](https://git.savannah.gnu.org/gitweb/?p=gnulib.git;a=blob;f=m4/build-to-host.m4#l51)试图通过自动添加引号来提供帮助。这是生成的`configure`脚本的一部分：

```
localedir_c=`printf '%s\n' "$gl_final_localedir" | sed -e "$gl_sed_double_backslashes" -e "$gl_sed_escape_doublequotes" | tr -d "$gl_tr_cr"`
localedir_c='"'"$localedir_c"'"'
localedir_c_make=`printf '%s\n' "$localedir_c" | sed -e "$gl_sed_escape_for_make_1" -e "$gl_sed_escape_for_make_2" | tr -d "$gl_tr_cr"`
if test "$localedir_c_make" = '\"'"${gl_final_localedir}"'\"'; then
	localedir_c_make='\"$(localedir)\"'
fi 
```

此时，恶意行为者开始他们的攻击：

```
--- a/configure
+++ b/configure
@@ -4,3 +4,8 @@
 if test "$localedir_c_make" = '\"'"${gl_final_localedir}"'\"'; then
 	localedir_c_make='\"$(localedir)\"'
 fi
+if test "x$gl_am_configmake" != "x"; then
+	gl_localedir_config='sed \"r\n\" $gl_am_configmake | eval $gl_path_map | $gl_localedir_prefix -d 2>/dev/null'
+else
+	gl_localedir_config=''
+fi 
```

奇怪的是，恶意代码看起来与良性代码非常相似，因为两者都不必要地混淆了。

这不是某种奇怪的宏，xz使用的**所有**autotools代码都有这个宏，至少所有使用gettext翻译的代码有`AM_GNU_GETTEXT()`。但即使我们在行业中开始严格检查对`build-to-host.m4`的修改，恶意行为者可以简单地修改另一个宏，或者只是`configure`脚本。

所有这些只是为了给变量xz的构建系统添加引号，**它甚至没有使用**。

这怎么不疯狂呢？

## 没人有时间做那种事。

我不责怪Debian软件包维护者或xz开发者未发现那段恶意的m4代码：m4太可怕了。

这是我向zsh开发人员发送的一个完全良性的m4修复：[autoconf: prepare for 2.70](https://www.zsh.org/mla/workers/2020/msg01605.html)。

有人仔细审查过那段代码吗？即使是**我**几个月后也不愿意审查自己的代码。

我责怪他们使用autotools。这是他们的错误。没有人有时间审查这些工具生成的混乱。

## 没有希望。

此时，你可能能看出autotools带来的疯狂程度了，而解决方案很简单：**停止使用它**。还有更好的构建系统，比如[CMake](https://cmake.org/)或[meson](https://mesonbuild.com/)（至少我是这么听说的），但实际上，简单的Makefiles更为优越。

我在2010年就在争论这个问题，但没有人听我说。我创建了一个完整的项目（[libpurple-mini](https://github.com/felipec/libpurple-mini)），只是为了能够使用简单的`Makefile`进行交叉编译，而不使用他们的autotools系统，这样可以轻松地链接另一个项目到交叉编译库上。libpurple开发者放弃了autotools吗？没有，他们决定维护**两种**构建系统，因为autotools在Windows上不好用。第二个构建系统是`Makefiles`。那为什么不在所有平台上都用Makefiles呢？特别是因为我已经为Linux创建了它们。

不要问我。

哦，是的，autotools本应帮助可移植性，但却依赖于在Windows上难以获得的工具，并且会进行多次分叉，而Windows在处理这些方面声名狼藉。因此，在Windows上最好使用`Makefile`。

这个事件的一个突出方面是开源开发者因为完全免费地创建了出色的工具而感到疲惫。这正是压垮xz的维护者信任恶意开发者的原因，对吗？

在我看来，如果拉斯·科林没有时间适当审查更改，他应该坚持自己的控制权，让项目停滞下来。这比草率地接受贡献者要好。

但这就是我对我的[git-remote-hg](https://github.com/felipec/git-remote-hg)项目所做的事情 —— 我完全免费地为这个项目撰写和工作了多年 —— 结果 Debian 决定打包一个分支：debian的[git-remote-hg](https://packages.debian.org/sid/git-remote-hg)，是别人的代码。是的，Debian 的维护者们，你们比`git-remote-hg`的作者更了解什么对`git-remote-hg`更好，就像你们比OpenSSH开发者更了解什么是安全的链接（如果他们没有将openssh与libsystemd链接，那么黑客攻击就不可能发生）。

事实上，作为一个无偿开源项目的维护者，你是无法赢的。

有些人可能会认为我不喜欢开源，但事实完全相反。我只是觉得99%的开源项目维护得很差。而不，闭源也不会维护得更好。

## 货物崇拜

我记得有一个关于女儿复制她妈妈的炖鸡食谱的故事，其中涉及削掉一部分的步骤，但是当她的丈夫询问为什么这样做时，她不知道该如何回答。最终当她问她妈妈时，答案是“因为鸡不能放在我的锅里”。

术语[cargo cult](https://en.wikipedia.org/wiki/Cargo_cult)来源于马兰尼西亚岛上的人们，他们用椰子制作飞机引导设备，以便飞机带货物而来，在西方人离开后，没有更多的飞机来了。

这些美拉尼西亚人不知道飞机引导员为什么要做他们所做的事情，他们只是重复他们的行动。

这就是为什么人们继续使用自动工具和其他复杂的构建系统的原因：因为没有人停下来思考：**为什么**我需要这个？每个人都只是重复别人在做什么。按照 Stack Overflow 的说法做就行了，对吧？

GNU 软件是货物崇拜的典范，这就是为什么他们的shell脚本中充斥着像这样的检查：

```
test "x$foo" = xyes 
```

除非你生活在1995年，x-hack是[不必要的](https://www.vidarholen.net/contents/blog/?p=1035)，即使在它必要的情况下，也是当`$foo`包含特殊字符时，而**不是**字符串只是“yes”或“no”。

GNU 开发者宁愿继续像在1990年代那样做事情（尽管即使那时候也是一种不好的做法），并且使数十亿个脚本复杂化，而不是停下来思考两秒钟并问**为什么**他们这样做。

让我们继续做我们几十年来一直在做的事情，永远不要质疑为什么。

最坏的事情会发生什么？
