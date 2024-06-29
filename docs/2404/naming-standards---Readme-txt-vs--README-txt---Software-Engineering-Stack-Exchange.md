<!--yml

类别：未分类

日期：2024-05-27 13:07:07

-->

# 命名标准 - Readme.txt vs. README.txt - 软件工程Stack Exchange

> 来源：[https://softwareengineering.stackexchange.com/questions/301691/readme-txt-vs-readme-txt](https://softwareengineering.stackexchange.com/questions/301691/readme-txt-vs-readme-txt)

全大写字母使文件更易于识别，这是有道理的，因为这可能是新用户希望首先查看的内容。（或者，至少应该查看的内容……）正如其他人已经提到的，以大写字母开头的文件名在[ASCII排序](http://www.catb.org/~esr/jargon/html/A/ASCIIbetical-order.html)中会在小写字母之前列出（`LC_COLLATE=C`），这有助于在第一眼就找到文件。

`README`文件是用户在自由软件包中通常期望找到的文件之一。其他文件包括`INSTALL`（构建和安装软件的说明）、`AUTHORS`（贡献者列表）、`COPYING`（许可证文本）、`HACKING`（如何开始贡献，可能包括起始点的TODO列表）、`NEWS`（最近的变更）或`ChangeLog`（大部分与版本控制系统冗余）。

这是[*GNU编码标准*](https://www.gnu.org/prep/standards/html_node/Releases.html#Releases)对`README`文件的描述。

> 发行版应包含名为`README`的文件，提供包的概述：
> 
> +   包的名称；
> +   
> +   包的版本号，或者指出包中版本号的位置；
> +   
> +   包的一般描述；
> +   
> +   对应文件`INSTALL`的引用，该文件应包含安装过程的解释；
> +   
> +   对于任何不寻常的顶级目录或文件的简要说明，或者其他帮助读者找到源代码的提示；
> +   
> +   包含复制条件的文件的引用。如果使用GNU GPL，则应该在名为`COPYING`的文件中。如果使用GNU LGPL，则应该在名为`COPYING.LESSER`的文件中。

既然始终为用户提供最少惊喜是件好事，您应该遵循此约定，除非有充分的理由偏离。在UNIX世界中，文件名扩展通常使用得很少，因此文件的规范名称是`README`，没有任何后缀。但是，大多数用户可能不会对名为`README.txt`的文件有任何困扰。如果文件使用[*Markdown*](https://daringfireball.net/projects/markdown/)编写，像`README.md`这样的文件名也许也是合理的。但是，避免在`README`文件中使用像HTML这样的更复杂的标记语言，因为它应该在纯文本终端上方便阅读。您可以将用户引导至软件的手册或在线文档，这些文档可能采用更复杂的格式来详细说明`README`文件中的内容。
