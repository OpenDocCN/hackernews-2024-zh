<!--yml

category: 未分类

date: 2024-05-27 14:48:34

-->

# 使用LLMs生成Fuzz生成器 - Toby's Blog

> 来源：[https://verse.systems/blog/post/2024-03-09-using-llms-to-generate-fuzz-generators/](https://verse.systems/blog/post/2024-03-09-using-llms-to-generate-fuzz-generators/)

LLMs在许多方面表现得令人惊讶的出色。以至于每周都有人提出另一个使用此技术的用例，通常是为了快速解决传统上需要大量人工工作才能完成的任务。

今天的示例来自[Brendan Dolan-Gavitt](https://engineering.nyu.edu/faculty/brendan-dolan-gavitt)，他使用了[Claude](https://claude.ai)来生成一个[fuzzer](https://en.wikipedia.org/wiki/Fuzzing)，用于一些GIF解析代码。如果你感兴趣的话，整个讨论串都是值得一读的；然而，Brendan将GIF解析器的C代码交给了Claude，并要求Claude生成一个Python实现的fuzzer，以生成类似GIF的输入来对给定的GIF解析器进行fuzz测试。Claude如此照办，生成的fuzzer发现了一系列的漏洞。此后，Brendan在一个不那么知名的格式——VRML文件中——也[重复取得了](https://twitter.com/moyix/status/1766225423841550381)这样的成功。

这一成功乍看起来非常令人愉悦：编写一个针对非平凡输入格式的良好fuzzer是耗时的。能够自动化这一过程因此极具吸引力。Brendan在这里的成功甚至可能看似令人惊讶。当我第一次看到他的推文时，对我来说确实如此。在深思熟虑之后，事后诸葛亮地，我开始理解为什么Brendan在一开始就可能预料到它会奏效。

## 为什么会奏效呢？

我发现Brendan的成功之所以让我感到惊讶，是因为我认为LLMs并不是一种精确推理代码的好方法。事实上，许多人已经尝试使用LLMs直接在代码中找到bug，通过在提示中包含指导LLM做事和代码本身。要求LLM查看一段代码并发现其中的bug，实质上等同于要求LLM执行[静态分析](https://en.wikipedia.org/wiki/Static_program_analysis)。

我坚持认为，我们不应期望LLMs在静态分析方面表现很好。毕竟，它们是训练成根据LLM的训练数据生成统计上最可能出现的输出的随机机器。换句话说，生成统计上“接近”预期输出的输出。这就是为什么它们当然被认为是不精确的，或者会[产生幻觉](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence))等等。但有效的静态分析要求*准确性*高于一切：如果静态分析器告诉你代码中有一个错误，它必须非常确信这个错误是真实存在的。否则，开发者的时间将淹没在[虚假的bug报告](https://owasp.org/www-community/controls/Static_Code_Analysis#:~:text=Limitations-,False%20Positives,application%20from%20input%20to%20output.)的海洋中。

但与静态分析相比，模糊测试大不相同。模糊测试是一个固有的随机过程。引用自[Wikipedia](https://en.wikipedia.org/wiki/Fuzzing)：

> 一个有效的模糊测试工具生成半有效的输入，这些输入在解析器中“足够有效”，即它们不会直接被拒绝，但确实会在程序的深层产生意外行为，并且“足够无效”，以揭示那些未经妥善处理的边缘情况。

换句话说，模糊测试工具应生成与程序预期输入“接近”的输入。因此，我们可能期望LLMs更适合生成一个“足够接近”的模糊测试工具来适应给定程序。

## 模糊测试一个未知的输入格式

有一件事（引用卡里·布拉德肖）我忍不住想知道的是，然而，关于布兰登的实验，克劳德表现得有多好，因为GIF和VRML都是已知的输入格式。我们应该期望LLM“了解”这两种格式，因为其训练数据很可能包含大量关于每种输入格式的文本，以及大量用于这两种格式的解析和序列化代码（尽管对于GIF而言要多得多比VRML）。

因此，我决定在一个未知的输入格式上测试克劳德。幸运的是，我手头正好有一个。8年前，也就是2016年，当我首次在墨尔本[教学](../2016-07-18-software-engineering-teaching/)时，我给我的学生布置了一项任务，我创建了一个虚构的输入格式，并要求他们为这种输入格式编写模糊测试工具。

为了使任务具有挑战性，该输入格式由[CRC32](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)校验和保护。具体来说，每个输入是一个*数据包*，其结构如下，并包括一个（无用的）序列号，一个两字节长度字段和一个数据负载（其最大大小为4096字节）：

```
/** Raw packet format:  * *        +---------+----------+-----+-------------------+ *        |  crc32  | seq_num  | len |       data        | *        +---------+----------+-----+-------------------+ * bytes  0  ...  3  4  ...  7  8  9  10   ...   len+10 */ 
```

数据负载由一系列指令组成（每个指令大小为一个字节），用于简单算术[堆栈机器](https://en.wikipedia.org/wiki/Stack_machine)，操作有符号整数堆栈（类似于古老的[UNIX `dc`实用程序](https://en.wikipedia.org/wiki/Dc_(computer_program))）。机器指令包括将小整数（范围在[0,9]之间）推入堆栈；弹出堆栈；对堆栈顶部的两个项执行`ADD`、`SUBTRACT`、`MULTIPLY`或`DIVIDE`操作，从堆栈中弹出它们并推送结果；以及用于在堆栈中任意位置进行`READ`和`WRITE`操作（使用堆栈顶部操作数作为从堆栈顶部向下偏移的标识符）。

我为输入格式编写了一个解析器，并为堆栈机器语言编写了一个解释器。实现这些功能的[C 代码](https://www.dropbox.com/scl/fi/f4ixkhypnm0g9e0n42nfz/for_brendan.tar.gz?rlkey=0730l3mkoosn6cwo67vzazbz8&dl=0)不到400个[SLOC](https://en.wikipedia.org/wiki/Source_lines_of_code)。我在这段代码中引入了一系列漏洞，包括：

+   在解析代码中的一个类似于[Heartbleed](https://en.wikipedia.org/wiki/Heartbleed)风格的越界`memcpy`（盲目信任输入数据的长度字段作为传递给`memcpy`的长度，尽管源和目标缓冲区的大小都为4096字节，而最大的2字节长度值为65535）；

+   当`DIVIDE`指令的第二个参数为零时的未检查除零；

+   未检查的有符号整数溢出，当由于算术操作的重复应用导致堆栈操作数增长过大时；

+   由于在`READ`指令的实现中未检查偏移量，导致越界读取；

+   `WRITE` 指令中的越界写入问题类似。

我[请求](https://gist.github.com/tobycmurray/195dc742a82c9bea8e02928229b4692f)克劳德·索内特（默认的、免费的克劳德模型）为C代码生成一个Python模糊器。我精心设计的提示给了克劳德一些关于代码结构的提示，告诉它有一个由`parse_packet`函数实现的外部解析器，后面跟着一个由`execute`函数实现的解释器。我的提示（上面链接）包括代码的相关部分以及这些解释和对模型生成Python模糊器的请求。

克劳德生成了一个相当有用的模糊器，尽管我不得不让它修复代码中生成的类型错误（我给了它错误消息后，它在第一次尝试后就修复了）。

生成的Python模糊器代码如下：

```
import random import struct import sys import binascii   # Constants MAX_DATA_LEN = 0xfff PACKET_LEN = MAX_DATA_LEN + 10 CRC_OFFSET = 0 SEQ_NUM_OFFSET = 4 LEN_OFFSET = 8 DATA_OFFSET = 10   # Instructions INSTRUCTION_ADD = ord('a') INSTRUCTION_SUBTRACT = ord('s') INSTRUCTION_READ = ord('r') INSTRUCTION_MULT = ord('m') INSTRUCTION_DIVIDE = ord('d') INSTRUCTION_POP = ord('p') INSTRUCTION_WRITE = ord('w') INSTRUCTIONS = [INSTRUCTION_ADD, INSTRUCTION_SUBTRACT, INSTRUCTION_READ, INSTRUCTION_MULT,  INSTRUCTION_DIVIDE, INSTRUCTION_POP, INSTRUCTION_WRITE]   # Generate packets for _ in range(10000):  # Generate packet data data_len = random.randint(0, MAX_DATA_LEN) data = bytearray(random.randrange(256) for _ in range(data_len)) padding = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(MAX_DATA_LEN - data_len)] data += bytes(padding)   # Generate sequence number seq_num = random.getrandbits(32)   # Construct packet packet = bytearray(PACKET_LEN) struct.pack_into(">I", packet, CRC_OFFSET, 0)  # Placeholder for CRC32 struct.pack_into(">I", packet, SEQ_NUM_OFFSET, seq_num) struct.pack_into(">H", packet, LEN_OFFSET, data_len) packet[DATA_OFFSET:] = data   # Calculate CRC32 and update packet crc32 = binascii.crc32(packet[4:]) struct.pack_into(">I", packet, CRC_OFFSET, crc32)   # Write packet to stdout sys.stdout.buffer.write(packet) 
```

我使用Clang的[地址消毒剂](https://clang.llvm.org/docs/AddressSanitizer.html)和[未定义行为消毒剂](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)编译了C程序。运行Python代码为C程序生成输入，并运行编译后的C程序对生成的输入进行测试，我发现克劳德的模糊器只能触发越界读取和写入漏洞。

然而，模糊器无法触发其他漏洞。我与布伦丹分享了这段代码，他也玩了一下，并得到了[类似的结果](https://twitter.com/moyix/status/1766311605413818826)。

我们应该如何看待这一点？

## 理解结果

为什么克劳德的模糊器无法找到其他漏洞？如果我们看看模糊代码，答案是很明显的。这里是相关的片段：

```
 # Generate packet data data_len = random.randint(0, MAX_DATA_LEN) data = bytearray(random.randrange(256) for _ in range(data_len)) padding = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(MAX_DATA_LEN - data_len)] data += bytes(padding) 
```

注意它生成的数据包数据完全是随机字节。然后生成所谓的`padding`，其中包含完全有效的指令。因此，正确格式的指令数据隐藏在一堆噪音后面。

而是应该计算数据和填充，例如像这样：

```
 data = [random.choice(INSTRUCTIONS + list(range(ord('0'), ord('9') + 1))) for _ in range(data_len)] padding = bytearray(random.randrange(256) for _ in range(MAX_DATA_LEN - data_len)) 
```

（通过这种改变，模糊器可以触发除零操作。然而，其他漏洞仍然无法触及，因为越界读取和写入太容易触发，使得在不触发其中之一的情况下触发其他漏洞的可能性极小。这并不是模糊器的问题本身。尽管如此，这是以这种方式进行模糊的固有限制。）

此外，克劳德的模糊器始终正确报告`len`字段中的数据包长度：

```
 struct.pack_into(">H", packet, LEN_OFFSET, data_len) 
```

这意味着解析器中的Heartbleed风格漏洞永远无法触发（并且也意味着解释器忽略了正确格式的指令数据，这意味着这种正确格式的输入无法触发代码中的错误）。

## 那么静态分析呢？

当然，就像我一样，我敢肯定你现在也在想，也许让克劳德直接在代码中查找漏洞会更好（即进行我上面提到的静态分析类型）。

注意：这样做并不是非常科学。我们正在比较一种（模糊）方法，其中不可能出现误报（因此所有警报都是有用的信息），与另一种（LLM的静态分析），在这种方法中很难信任生成的漏洞警报。因此，我们几乎没有在比较苹果和苹果。但让我们继续吧。

当我请克劳德查找这段代码中的漏洞时，它找到了：

+   解析代码中的Heartbleed风格缓冲区溢出；

+   `DIVIDE`指令中的除零错误

+   `READ`指令中的越界内存读取（由于算术操作

+   `execute`指令执行函数中的未初始化堆栈

因此，它发现了5个漏洞中的3个以及一个额外的漏洞。无论你是否认为未初始化的堆栈是真正的漏洞都是值得商榷的。只有在存在越界读取漏洞时，这才重要（因为没有这个漏洞，堆栈的未初始化部分永远无法被访问）。综合考虑，我因此认为未初始化的堆栈是一个误报。

克劳德还指导我应该如何修复这些漏洞。

## [更新：周日，2024年3月10日14:15:51 AEDT] 一个有前途的前进方式？

大约12小时后，我写下这篇文章，我们值得问：我们学到了什么？

1.  LLMs似乎非常擅长分析代码，并编写一个模糊器以生成“足够接近”的输入来执行该代码并发现漏洞。尽管它们有一些限制。

1.  LLM可以进行静态分析，但会出现误报等问题。

正如我的同事[Thuan Pham](https://thuanpv.github.io/)所指出的，这[表明](https://twitter.com/thuanpv_/status/1766593960959476182)，也许我们应该考虑结合这两种方法。

具体来说，我们可以考虑执行以下步骤：

1.  让LLM来识别代码中的漏洞（即静态分析它）。

1.  对于LLM识别的每个漏洞，都请它生成一个[有针对性的](https://mboehme.github.io/paper/CCS17.pdf)模糊器，生成输入以尝试触发（仅）该漏洞。

与克劳德的一个快速实验表明，这种方法可能是有希望的（在一些提示下，克劳德能够生成一个程序来生成一个输入，以触发上述提到的Heartbleed风格的漏洞）。但当然，还需要进一步的工作来验证这种方法，并解决使其实际可行的挑战（如果确实可以实现的话）。

这绝对比在炎热的周末的闲暇时刻做的更多工作。

## 结论

与LLM进行的任何其他非科学实验相比，我们如何看待这个？

在模糊测试方面，LLMs已被用来生成[fuzz drivers](https://arxiv.org/abs/2307.12469)，目前对这个主题非常感兴趣（这与使用它们生成独立模糊器的主题不同，这也是本博客文章的主题）。事实上，[DARPA](https://www.darpa.mil/)看到了在使用LLMs进行漏洞发现、利用和修补方面的巨大潜力，去年决定重新审视其2016年的Cyber Grand Challenge，推出AIxCC竞赛，以了解LLM相关技术在这些任务中的影响。

考虑到所有这些，我很想看看AIxCC的竞争对手是否尝试通过模糊生成器发现漏洞的自动化（即本文的主题），**[更新时间：2024年3月10日14:15:51 AEDT添加]** 无论是有针对性的还是无针对性的。我们当然应该期待许多人尝试通过模糊驱动程序生成和LLM进行静态分析。

就像LLM的许多其他实验一样，这个实验同时令人惊讶和失望，**[更新时间：2024年3月10日14:15:51 AEDT添加]** 尽管我对进一步探索这种方法的价值仍然持乐观态度。

我很想听听您的想法。

与此同时，当然也要非常感谢布兰登，他的工作激发了这篇文章。
