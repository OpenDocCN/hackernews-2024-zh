<!--yml

category: 未分类

日期：2024-05-27 12:55:30

-->

# autoconf 让我觉得我们停滞不前了

> 来源：[https://rachelbythebay.com/w/2024/04/02/autoconf/](https://rachelbythebay.com/w/2024/04/02/autoconf/)

## autoconf 让我觉得我们停滞不前了

我收到了一些反馈，要求我对过去几天发生的整个 "xz后门" 事件表达我的想法和/或反应。我对此事的大多数想法都适用于autoconf和它的朋友们，而且并不乐观。

这些天我不经常使用那些工具，但是在相当长一段时间里，我经常从源代码构建东西，然后 ./configure --with-this --with-that 是理所当然的。当那个东西让我可以重用旧的configure调用，这对我来说是一种小小的喜悦，因为我不必再去挖掘具体细节了。

我明白autoconf这个傻乎乎的 "配方" 的整体原因是你想知道你所在的系统是否支持X，或者能否做Y，或者它具体支持的Z的哪个变种，这样你就可以用#ifdef之类的方式进行条件编译。虽然这在今天不太相关，但确实，曾经有一个时代存在许多不同的Unix系统，它们都有自己处理事务的方式，而且没有两个系统完全相同。

所以，好吧，某个时候运行程序来经验性地确定在给定系统上支持什么是有意义的。我不明白的是，为什么我们一直在反复运行那些愚蠢的小Shell片段和少量的C代码。就好像，好吧，我们确认这个特定系统用两个参数而不是三个执行<library function foobar>。那我们为什么还在一遍又一遍地测试它？

为什么我们最终没有出现一种只有少数可能值的标准事物，而这些值可以在某处为你设置？负责构建你的系统的人（操作系统公司，分发打包者，等等）可以在 /etc 中留下类似于 "X = flavor 1, Y = flavor 2" 的内容，依此类推。

好吧，我明白，可能会有很多 "真正的操作系统公司" 不愿意沦为肮脏的自由软件嬉皮士的水平。随便吧。那些同样的嬉皮士本来可以在每个平台/操作系统组合上运行测试一次，将结果放入 /etc 中，然后结束这一切。

然后，我们每次从源代码构建东西时，而不是每次都测试所有这些东西，我们只需引入现成的结果然后继续。这些结果不会随机改变。它们反映了内核、C库、API和用户空间运作的方式。除非这些发生了变化，否则结果也不会改变。

但是不，我们从未达到那一点，因此仍然可以正常地使用一个 .tar.gz，其中包含大量愚蠢的小宏文件，运行各种晦涩难懂的测试，给出的答案与它们上次在您的机器或任何类似您的机器上运行时的答案相同，并且将来也会给出相同的答案。

这意味着正常情况下可以发布各种看起来非常疯狂的东西，所以当有人注意到这一点并决定将其作为从一个所谓的“测试文件”中提取一些坏东西的机制时，这样做是否令人惊讶？对我来说，这似乎是不可避免的。

顺便说一句，我想看看如果人们开始从各种项目中获取 tarballs 并将它们与这些项目的源代码存储库进行 diff，会发生什么。任何在 tarball 中“出现”的文件，据说是由于在项目上运行了 auto[re]conf，最好与实际的 autoconf、automake、ranlib、gettext 或其他现在使用的怪异的元构建工具的树中的某些内容匹配。

```
$ find . -type f | sort | xargs sha1sum
7d963e5f46cd63da3c1216627eeb5a4e74a85cac  ./ax_pthread.m4
c86c8f8a69c07fbec8dd650c6604bf0c9876261f  ./build-to-host.m4
0262f06c4bba101697d4a8cc59ed5b39fbda4928  ./getopt.m4
e1a73a44c8c042581412de4d2e40113407bf4692  ./gettext.m4
090a271a0726eab8d4141ca9eb80d08e86f6c27e  ./host-cpu-c-abi.m4
961411a817303a23b45e0afe5c61f13d4066edea  ./iconv.m4
46e66c1ed3ea982b8d8b8f088781306d14a4aa9d  ./intlmacosx.m4
ad7a6ffb9fa122d0c466d62d590d83bc9f0a6bea  ./lib-ld.m4
7048b7073e98e66e9f82bb588f5d1531f98cd75b  ./lib-link.m4
980c029c581365327072e68ae63831d8c5447f58  ./lib-prefix.m4
d2445b23aaedc3c788eec6037ed5d12bd0619571  ./libtool.m4
421180f15285f3375d6e716bff269af9b8df5c21  ./lt~obsolete.m4
f98bd869d78cc476feee98f91ed334b315032c38  ./ltoptions.m4
530ed09615ee6c7127c0c415e9a0356202dc443e  ./ltsugar.m4
230553a18689fd6b04c39619ae33a7fc23615792  ./ltversion.m4
240f5024dc8158794250cda829c1e80810282200  ./nls.m4
f40e88d124865c81f29f4bcf780512718ef2fcbf  ./po.m4
f157f4f39b64393516e0d5fa7df8671dfbe8c8f2  ./posix-shell.m4
4965f463ea6a379098d14a4d7494301ef454eb21  ./progtest.m4
15610e17ef412131fcff827cf627cf71b5abdb7e  ./tuklib_common.m4
166d134feee1d259c15c0f921708e7f7555f9535  ./tuklib_cpucores.m4
e706675f6049401f29fb322fab61dfae137a2a35  ./tuklib_integer.m4
41f3f1e1543f40f5647336b0feb9d42a451a11ea  ./tuklib_mbstr.m4
b34137205bc9e03f3d5c78ae65ac73e99407196b  ./tuklib_physmem.m4
f1088f0b47e1ec7d6197d21a9557447c8eb47eb9  ./tuklib_progname.m4
86644b5a38de20fb43cc616874daada6e5d6b5bb  ./visibility.m4
$ 

```

……没有与那个 sha1sum 匹配的 build-to-host.m4，*除了*在 xz 发行版中的那个坏的。这一部分被发现了……但其他所有项目中的每个其他 auto* blob 呢？谁或什么在检查它们？

最后，是的，我肯定有偏见。我的个人构建系统有一个小文件，根据 libs 和其他工作方式在机器上安装。这意味着所有特定版本操作系统的 Mac 都会得到相同的文件。所有运行相同版本的 Debian 箱子也会得到相同的文件，依此类推。

我不会每次构建东西时都问相同的问题。那只是疯狂。
