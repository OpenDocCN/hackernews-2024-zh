<!--yml
category: 未分类
date: 2024-05-27 14:35:17
-->

# ts_zip: Text Compression using Large Language Models

> 来源：[https://bellard.org/ts_zip/](https://bellard.org/ts_zip/)

# ts_zip: Text Compression using Large Language Models

The `ts_zip` utility can compress (and hopefully decompress) text files using a Large Language Model. The compression ratio is much higher than with other compression tools. There are some caveats of course:

*   A GPU is necessary to get a reasonable speed. 4 GB of RAM is required.
*   It is slower than conventional compressors (compression and decompression speed: up to 1 MB/s on a RTX 4090).
*   Only text files are supported. Binary files won't be compressed much. The currently used language model (RWKV 169M v4) was trained mostly on English texts. Other languages are supported including source code.
*   It is experimental so no backward compability should be expected between the various versions.

## Compression Ratio

The compression ratio is given in bits per byte (bpb).

| File | Original size (bytes) | [xz](https://en.wikipedia.org/wiki/XZ_Utils) (bytes) (bpb) | ts_zip (bytes) (bpb) |
| --- | --- | --- | --- |
| [alice29.txt](https://en.wikipedia.org/wiki/Canterbury_corpus) | 152089 | 48492 | 2.551 | 21713 | 1.142 |
| [book1](https://en.wikipedia.org/wiki/Calgary_corpus) | 768771 | 261116 | 2.717 | 137477 | 1.431 |
| [enwik8](http://mattmahoney.net/dc/text.html) | 100000000 | 24865244 | 1.989 | 13825741 | 1.106 |
| [enwik9](http://mattmahoney.net/dc/text.html) | 1000000000 | 213370900 | 1.707 | 135443237 | 1.084 |
| [linux-1.2.13.tar](https://mirrors.edge.kernel.org/pub/linux/kernel/v1.2/linux-1.2.13.tar.xz) | 9379840 | 1689468 | 1.441 | 1196859 | 1.021 |

Results and speed for other programs on enwik8 and enwik9 are available at the [Large Text Compression Benchmark](http://www.mattmahoney.net/dc/text.html).

## Download

## Technical information

*   `ts_zip` uses the [RWKV 169M v4](https://github.com/BlinkDL/RWKV-LM) language model which is a good compromise between speed and compression ratio. The model is quantized to 8 bits per parameter and evaluated using BF16 floating point numbers.
*   The language model predicts the probabilities of the next token. An arithmetic coder then encodes the next token according to the probabilities.
*   The model is evaluated in a deterministic and reproducible way. Hence the result does not depend on the exact GPU or CPU model nor on the number of configured threads. This key point ensures that a compressed file can be decompressed using a different hardware or software configuration.

* * *

Fabrice Bellard - [https://bellard.org/](https://bellard.org)