<!--yml

category: 未分类

日期：2024-05-27 14:35:17

-->

# ts_zip：使用大型语言模型进行文本压缩

> 来源：[https://bellard.org/ts_zip/](https://bellard.org/ts_zip/)

# ts_zip：使用大型语言模型进行文本压缩

`ts_zip` 实用程序可以使用大型语言模型压缩（并希望能够解压缩）文本文件。与其他压缩工具相比，其压缩比显著更高。当然，也有一些注意事项：

+   获取合理的速度需要 GPU。需要 4 GB RAM。

+   它比传统压缩器慢（压缩和解压速度：在 RTX 4090 上最高达每秒 1 MB）。

+   仅支持文本文件。二进制文件的压缩效果不会很好。当前使用的语言模型（RWKV 169M v4）主要训练于英文文本。支持其他语言，包括源代码。

+   这是实验性质的，因此不应期望各个版本之间有向后兼容性。

## 压缩比率

压缩比率以每字节位数（bpb）给出。

| 文件 | 原始大小（字节） | [xz](https://en.wikipedia.org/wiki/XZ_Utils)（字节）（bpb） | ts_zip（字节）（bpb） |
| --- | --- | --- | --- |
| [alice29.txt](https://en.wikipedia.org/wiki/Canterbury_corpus) | 152089 | 48492 | 2.551 | 21713 | 1.142 |
| [book1](https://en.wikipedia.org/wiki/Calgary_corpus) | 768771 | 261116 | 2.717 | 137477 | 1.431 |
| [enwik8](http://mattmahoney.net/dc/text.html) | 100000000 | 24865244 | 1.989 | 13825741 | 1.106 |
| [enwik9](http://mattmahoney.net/dc/text.html) | 1000000000 | 213370900 | 1.707 | 135443237 | 1.084 |
| [linux-1.2.13.tar](https://mirrors.edge.kernel.org/pub/linux/kernel/v1.2/linux-1.2.13.tar.xz) | 9379840 | 1689468 | 1.441 | 1196859 | 1.021 |

其他程序在 enwik8 和 enwik9 上的结果和速度可在[大文本压缩基准测试](http://www.mattmahoney.net/dc/text.html)查看。

## 下载

## 技术信息

+   `ts_zip` 使用的是[RWKV 169M v4](https://github.com/BlinkDL/RWKV-LM)语言模型，这是速度和压缩比之间的一个良好折衷方案。该模型的参数量化为每个参数 8 位，并使用 BF16 浮点数进行评估。

+   语言模型预测下一个标记的概率。然后，算术编码器根据这些概率对下一个标记进行编码。

+   该模型以确定性和可重现的方式进行评估。因此，结果不依赖于确切的 GPU 或 CPU 型号，也不依赖于配置的线程数。这一关键点确保可以使用不同的硬件或软件配置解压缩压缩文件。

* * *

Fabrice Bellard - [https://bellard.org/](https://bellard.org)
