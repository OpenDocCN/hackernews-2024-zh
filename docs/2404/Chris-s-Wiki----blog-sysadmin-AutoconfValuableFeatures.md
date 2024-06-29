<!--yml

category: 未分类

date: 2024-05-27 13:36:55

-->

# Chris's Wiki :: 博客/系统管理员/AutoconfValuableFeatures

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/AutoconfValuableFeatures](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/AutoconfValuableFeatures)

在[XZ Utils后门事件](https://en.wikipedia.org/wiki/XZ_Utils_backdoor)之后，涉及[GNU Autoconf](https://www.gnu.org/software/autoconf/manual/)，人们普遍建议Autoconf应该退出历史舞台。一些人建议这样做的同时，也提议Autoconf和生成的'`configure`'脚本的替代方案应该更简单些。作为一个与配置脚本（以及autoconf）交互，并处理构建项目如[OpenZFS](https://openzfs.org/wiki/Main_Page)的系统管理员，我认为，提出更简单替代方案的人可能没有看到像我这样的人在实践中发现的有价值的特性。

（对于这一点，我先不考虑[替换Autoconf的（浪费）成本](/~cks/space/blog/programming/AutoconfNotReplaceable).）

诸如[OpenZFS](https://openzfs.org/wiki/Main_Page)等项目依赖其配置系统来检测正在构建的系统的各个方面，这些方面不能简单地假定。对于OpenZFS来说，这包括（内部）内核'API'的各个方面；对于其他项目，比如[conserver](https://en.wikipedia.org/wiki/Conserver)，这涵盖了系统是否具有IPMI库可用等情况。作为构建这些项目的系统管理员，我希望它们能自动检测所有这些情况，而不是强迫我手工设置构建选项（或者要求我安装它们可能想要使用的所有库等）。

作为系统管理员，我发现`configure`的一个重要优点是[它不需要我更改软件中已经存在的任何内容来配置软件](/~cks/space/blog/programming/ConfigureNoSourceCodeChanges)。我可以使用命令行配置软件，这意味着我可以使用各种方式保存和调用该命令行，从'如何在这里构建'的文档到自动化脚本。

普通的配置脚本也让我和其他人能够设置软件的安装位置。对于可能安装为Linux发行版软件包、*BSD第三方软件包、本地系统管理员安装，或是个人用户放置在自己家目录中的程序来说，这是一个相对关键的功能，因为这四种情况通常需要不同的安装位置。如果替代的配置系统不接受至少`--prefix`参数或其等效项，那么实际上它的用处就大大减少了。

许多GNU configure脚本还允许配置软件的人设置各种选项，例如它将包含哪些特性，默认情况下的行为等等。这些选项的使用程度在程序之间（以及构建程序的人之间）差异很大，但有时它们对于选择默认值和启用（或禁用）并非每个人都需要的功能至关重要。一个不支持这些构建选项的替代configure系统对于想要用非标准选项构建这类软件的人来说不太有用，而且它可能会迫使软件完全放弃构建选项。

（有些人会说，软件不应该像运行时配置设置那样拥有构建选项，但这并不是一个特别流行的观点。）

这是我的清单，所以其他人可能会更看重Autoconf和configure支持的其他功能（例如，设置C编译器标志的能力，或者它在构建RPM包方面的良好支持）。
