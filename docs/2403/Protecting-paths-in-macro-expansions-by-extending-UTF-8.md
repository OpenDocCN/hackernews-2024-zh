<!--yml

类别：未分类

日期：2024-05-27 14:38:32

-->

# 通过扩展UTF-8保护宏扩展中的路径

> 来源：[https://nullprogram.com/blog/2024/03/05/](https://nullprogram.com/blog/2024/03/05/)

2024年3月5日

nullprogram.com/blog/2024/03/05/

一年后，我终于为一个令人烦恼的[u-config](/blog/2023/01/18/)问题找到了一个优雅的解决方案。pkg-config格式使用宏通过递归扩展生成构建标志。一些标志嵌入文件系统路径，但对于宏系统来说，它们仅仅是字符串。输出最终也只是一个大字符串，接收shell将其[分割成字段](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_06_05)。如果路径包含空格或shell元字符，u-config必须对它们进行转义，以便shell将它们视为标记的一部分。但是，u-config本身如何区分路径中的偶发空格与标志之间的意图空格？路径中的其他shell元字符呢？我的解决方案是扩展UTF-8以编码在宏扩展中生存的元数据。

通常，从一个具体的问题例子开始会有所帮助。以下是一个类似于你在自己系统上找到的传统`.pc`文件：

```
prefix=/usr
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: Example
Version: 1.0
Description: An example .pc file
Cflags: -I${includedir}
Libs: -L${libdir} -lexample 
```

它从定义库的安装前缀开始，从中派生额外的路径，最终用于生成构建标志（`Cflags`，`Libs`）的包字段。如果我对这个配置运行u-config：

```
$ pkg-config --cflags --libs example
-I/usr/include -L/usr/lib -lexample 
```

通常`prefix`由库的构建系统填充，它知道库应该安装在哪里。在某些情况下，这是不可能的，并且没有机会将`prefix`设置为有意义的路径。在这种情况下，pkg-config可以自动覆盖它（`--define-prefix`）为相对于`.pc`文件的路径，使安装可以重定位。这在[Windows上效果非常好，因为它是默认设置](/blog/2020/09/25/)的。

```
$ pkg-config --cflags --libs example
-IC:/Users/me/example/include -LC:/Users/me/example/lib -lexample 
```

这只是起作用…… *只要路径中不包含空格*。如果包含空格，它有可能会分裂成独立的字段。`.pc`格式支持引用来控制如何转义这样的输出。在引号之间的区域在输出中被转义，这样它们在shell中的字段分割时保留它们的空格。如果一个`.pc`文件的作者小心的话，他们会使用引号来编写：

```
Cflags: -I"${includedir}"
Libs: -L"${libdir}" -lexample 
```

路径被小心地放置在[引用区域](/blog/2021/12/04/)中，以便它们正确地输出：

```
$ pkg-config --cflags example
-IC:/Program\ Files/example/include 
```

*几乎没有人以这种方式编写他们的`.pc`文件*！约定是不使用引号。我的最初解决方案是在分配时隐式包裹`prefix`在引号中，这修复了绝大多数`.pc`文件。在“虚拟”的`.pc`文件中，它实际上看起来像这样：

```
prefix="C:/Program Files/example"
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include 
```

因此，重要区域被引用，其空格得以保留。然而，偶尔会有支持Windows的库作者遇到这个问题，他们系统的pkg-config实现不会引用`prefix`。他们很快就会意识到显式引用并应用它，这将削弱u-config的隐式引用。引号本质上会取消掉：

```
"$includedir" -> ""C:/Program Files/example"/include" 
```

引用的区域被颠倒，什么也没有发生。尽管这只是少数情况，但这些库和你在Windows上可能使用的库是相关的。我曾感到困惑：如何同时支持带引号和不带引号的`.pc`文件？

### 扩展UTF-8

我最近想到了这个问题：如果u-config以某种方式追踪哪些字符串区域是路径呢。`prefix`最初是一个路径区域，然后通过宏扩展和连接进行跟踪。不久后我意识到更简单的方法：**将路径中的空格编码为除空格外的值**，但也是不能出现在输入中的值。请记住，[UTF-8文本中绝对不能出现某些八位组](/blog/2017/10/06/)：其最高5位为设置位的8个值。这将是5字节或更长代码点的第一个八位组，但这些都是被禁止的。

当路径进入宏系统时，特殊字符被编码为这8个值之一。它们在输出编码时会被转换回它们原始的ASCII值，并进行转义。它与pkg-config的引用机制无关，因此不会发生引号取消。两种引号情况都得到了支持。

例如，如果空格映射到`\xff`（255），那么：

```
in:  C:/Program Files/foo    -> C:/Program\xffFiles/foo
out: C:/Program\xffFiles/foo -> C:/Program\ Files/foo 
```

无论`${includedir}`还是`"${includedir}"`都打印相同的内容。问题解决！

这不是唯一的复杂情况。输出可能会*故意*包含shell元字符，尽管通常这些是[Makefile](/blog/2017/08/20/)片段。例如，`${pc_top_builddir}`的默认值是`$(top_builddir)`，稍后`make`将会扩展它。虽然这些字符对shell来说是特殊的，对`make`来说也是特殊的，但它们不应该被转义。

如果路径包含这些字符怎么办？pkg-config的引用机制无济于事。它只关心空格，而`$(...)`无论是否引用都会打印相同的内容。与之前一样，u-config必须追踪来源——这些字符是否起源于路径。

如果设置了`$PKG_CONFIG_TOP_BUILD_DIR`，则`pc_top_builddir`将设置为此环境变量，当结果不被`make`处理时非常有用。在这种情况下，它是一个路径，并且应该转义`$(...)`。即使没有`$`，也必须加引号，因为括号仍然会调用一个子shell。但是谁会在路径中放括号？瞧瞧！

```
C:/Program Files (x86)/example 
```

再次，扩展UTF-8也解决了这个问题：使用三个被禁用的八位组来编码路径中的`$`、`(`和`)`，并在输出时对它们进行转义，允许未编码的实例直接通过。

```
in:  C:/Program\xffFiles\xff\xfdx86\xfe/example
out: C:/Program\ Files\ \(x86\)/example 
```

这使得`pc_top_builddir`非常直观：默认为原始字符串，否则为路径编码的环境变量（注意：`s8`[是字符串类型](/blog/2023/10/08/)，而`upsert`是[hash映射](/blog/2023/09/30/)）：

```
 s8 top_builddir = s8("$(top_builddir)");
    if (envvar_set) {
        top_builddir = s8pathencode(envvar, perm);
    }
    *upsert(&global, s8("pc_top_builddir"), perm) = top_builddir; 
```

对于一个特别复杂的情况，考虑有意使用`uname -m`命令替换来构造路径，即路径包含目标机器架构（`i686`，`x86_64`等）：

```
Cflags: -I${prefix}/$(uname -m)/include 
```

（不是说赞成这种无稽之谈。这只是真实世界`.pc`文件的现实。）自动设置如上述的`prefix`后，将打印出：

```
-IC:/Program\ Files\ \(x86\)/example/$(uname -m)/include 
```

路径括号被转义，因为它们来自路径，但命令替换会通过，因为它来自`.pc`源。非常酷！
