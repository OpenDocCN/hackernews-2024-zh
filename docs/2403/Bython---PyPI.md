<!--yml

类别：未分类

日期：2024-05-27 14:50:34

-->

# Bython · PyPI

> 来源：[https://pypi.org/project/Bython/](https://pypi.org/project/Bython/)

# Bython

使用大括号的 Python。因为 Python 很棒，但空白很可怕。

Bython 是一个 Python 预处理器，它将大括号翻译成缩进。

## README 的内容：

## 主要特点

+   "忘记"缩进。您仍然应该编写美观的代码，但是如果您在制表符/空格上出错，或者将一个代码片段复制到使用不同缩进样式的另一个代码片段中，它不会破坏。

+   使用 Python 进行解释，这意味着您所有现有的模块，如 NumPy 和 Matplotlib，仍然可以正常工作。

## 代码示例

```
def print_message(num_of_times) {
    for i in range(num_of_times) {
        print("Bython is awesome!");
    }
}

if __name__ == "__main__" {
    print_message(10);
}

```

## 安装

您可以直接从 PyPI 安装 Bython，使用 pip（具体取决于您的 Python 安装，是否需要 `sudo -H`）：

```
$ sudo -H pip3 install bython 
```

如果您因某种原因想要从 git 仓库安装它，可以使用 `git clone` 进行本地安装：

```
$ git clone https://github.com/mathialo/bython.git
$ cd bython
$ sudo -H pip3 install . 
```

git 版本有时会比 PyPI 版本稍微领先一点，但差距不大。

要卸载，只需运行

```
$ sudo pip3 uninstall bython 
```

将取消所有更改。

## 快速介绍

Bython 的工作方式是首先将 Bython 文件（建议文件结尾为 .by）翻译成 Python 文件，然后使用 Python 运行它们。因此，您需要安装工作正常的 Python 才能使用 Bython。

要运行 Bython 程序，只需键入

```
$ bython source.by arg1 arg2 ... 
```

以带有参数 arg1、arg2 等的形式运行 `source.by`。如果您想要关于如何运行 Bython 文件（标志等）的更多详细信息，请键入

```
$ bython -h 
```

打印内置帮助页面的方法。您也可以通过键入命令查阅 man 页面

```
$ man bython 
```

Bython 还包括一个从 Python 到 Bython 的转换器。可以通过 `py2by` 命令找到：

```
$ py2by test.py 
```

这将创建一个名为 `test.by` 的 Bython 文件。关于 `py2by` 的完整说明可以通过键入命令找到

```
$ py2by -h 
```

或通过查阅 man 页面：

```
$ man py2by 
```

要获取更详细的介绍，请参阅 [Bython 介绍](INTRODUCTION.md)

## 仓库的结构

目前，Bython 是用 Python 编写的。git 仓库结构化为 4 个目录：

+   `bython` 包含一个 Python 包，其中包含解析器和主脚本使用的其他实用程序

+   `etc` 包含手册页面和其他辅助文件

+   `scripts` 包含可从 shell 运行的可执行 Python 脚本

+   `testcases` 包含一些用于测试实现的示例 *.by 和 *.py 文件
