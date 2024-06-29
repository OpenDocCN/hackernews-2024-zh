<!--yml

category: 未分类

date: 2024-05-27 14:40:20

-->

# Modular: 社区聚焦：用 Mojo 🔥 战胜 Rust ⚙️ DNA 序列解析基准测试，性能提升50%

> 来源：[https://www.modular.com/blog/outperforming-rust-benchmarks-with-mojo](https://www.modular.com/blog/outperforming-rust-benchmarks-with-mojo)
> 
> 这是 Mohamed Mabrouk 的客座博客文章。Mohamed 是 [MojoFastTrim](https://github.com/MoSafi2/MojoFastTrim) 的创作者，一个 Mojo 🔥 社区项目。Mohamed 在Python上取得了100倍的性能提升，并比Rust最快的实现提高了50%。他很快学会了这门语言，第一次实现仅用了200行代码。继续阅读了解他应用的额外优化细节，以战胜现有最快基准测试！

### 生物信息学的大数据时代

当今生物信息学面临的挑战根源于大数据处理。成千上万台价值数百万美元的DNA测序机器在生物技术、医学和生物医学研究的所有领域中不停地工作。预计到2025年，每年的测序数据规模将达到[40艾字节](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4494865/)的原始序列数据。这相当于YouTube每年上传的数据量的**20倍**。

大多数最终分析是在高级语言如Python和R中进行的，但生物信息学的世界却是由C、C++和Java等低层黑魔法驱动的！这些高度优化的工具用于预处理和总结大量原始数据。这就产生了一个双重世界的问题，那些不熟悉低级语言的生物信息学家无法理解、定制和实现低级操作。此外，典型的生物信息学流水线是Bash和Python脚本与预编译二进制文件混合，还包括分析逻辑本身。对新老生物信息学家来说，这种复杂性和挫折感越来越严重。这与人工智能社区所面临的问题相同。

### Mojo 🔥 统治一切的工具

我第一次从[Jeremy Howard](https://www.youtube.com/watch?v=6GvB5lZJqcE)的演示视频中了解到Mojo。它的价值提议很简单，它是一种Pythonic语言，允许程序员在更低的层次上进行优化，以统一人工智能等领域的碎片化。对我来说，从Python世界来看，学习Mojo相对容易，只用了几天就习惯了额外的语法。我决定在一个严肃的项目中尝试Mojo 🔥，用于低级生物信息学任务：FASTQ解析和质量修剪。[FASTQ](https://en.wikipedia.org/wiki/FASTQ_format)是大多数DNA测序操作的基本格式，包含了每个碱基呼叫的基因组序列和机器的置信度分数。这是一个简单的解析格式，大多数记录看起来像这样：

FASTQ

@SEQ_ID # 新的读取头 + read_id GATTTGGGGTTCAAAGCAGTATCGATCAAATAGTAAA... # DNA 序列 + # 质量标头 + id（可选）！''*((((***+))%%%++)(%%%%).1***-+*'')... # Phred 质量分数

然而，典型的未压缩文件大小为1-50 GB，一个平均以序列为主的研究可能为单个文件生成超过1 TB 的数据。在解析和数据处理中，性能至关重要。

我尝试编写一个简单的解析器，它可以：

1\. 作为 *String* 读取文件的一个块。

2\. 根据换行符 *\n* 分隔字符串。

3\. 获取每4行，验证它们是否是一致且正确的 FASTQ 记录，并返回它。

4\. 反复执行直到达到 *EOF*。

在 [第一次尝试](https://github.com/MoSafi2/MojoFastTrim/tree/v0.1)，*MojoFastTrim 🔥* 的性能达到了 Python 的 [SeqIO](https://biopython.org/wiki/SeqIO) 的 **8x**。我对开发时间感到愉快的惊讶。我的代码仍然保持 Python 风格，简洁地约200行，并使用普通 Python 开发者能理解的功能。在质量修剪中，从每个读取中删除低质量碱基，它达到了行业标准工具 [Cutadapt](https://github.com/marcelm/cutadapt) 的50%-80%。对我投入到项目中的开发时间来说，这是惊人的性能水平。

### 沉迷于优化的兔子洞

Mojo 🔥 最强大的好处在于它为您提供了访问低级优化的能力。Mojo 标准库的初期状态意味着我不得不从头开始编写、测试和基准一些函数。Mojo 对 *SIMD* 向量化的一流支持非常有帮助，而且令人惊讶地直观。这里是在 Mojo 中实现向量化版本的函数，用于找到换行符索引的实现：

##### 迭代的

Mojo

@always_inline fn find_chr_next_occurance_iter[ T: DType ](in_tensor: Tensor[T], chr: Int = 10, start: Int = 0) -> Int: for i in range(start, in_tensor.num_elements()): if in_tensor[i] == chr: return i return -1

##### SIMD Vectorized

Mojo

@always_inline fn find_chr_next_occurance_simd[ T: DType ](in_tensor: Tensor[T], chr: Int = 10, start: Int = 0) -> Int: alias simd_width = simdwidthof[T]() let len = in_tensor.num_elements() - start let aligned = start + math.align_down(len, simd_width) for s in range(start, aligned, simd_width): let v = in_tensor.simd_load[simd_width](s) let mask = v == chr if mask.reduce_or(): return s + arg_true(mask) # Handling other elements iteratively return -1

向量化版本加载32个 *Int8* 元素，并使用更少的操作检查换行符的存在。

在以下图表中，您可以看到 SIMD 向量化的效果。它提供了高达 **4x** 的加速，平均加速度为 **3.2x**。同样，*SIMD* 从张量中的存储和加载提供了显著的性能提升。

此外，我探索了来自C/C++实现的优化。我担心加载的块没有为明确的内存缓冲区进行分配，但Mojo编译器已经照顾到了这一点，并避免了新的内存分配：

Mojo

fn test_chunk_ptr_index() raises: let f = open("large_fastq_file.fastq", "r") var offset = 0 var chunk_no = 0 while True: let t = f.read_bytes(Size) print(t._ptr.__as_index()) offset += Size chunk_no += 1 _ = f.seek(offset) if t.num_elements() == 0: break # chunk 0的内存地址：139711892942912 # chunk 1的内存地址：139711892942912 # chunk 2的内存地址：139711892942912 # chunk 3的内存地址：139711892942912 # chunk 4的内存地址：139711892942912

实现这些优化导致额外的**3x**速度提升，而*MojoFastTrim 🔥*平均比Python的*SeqIO*快了**24x**。此外，由于Mojo对引用和值语义的控制🔥，我应用了解析器的*FastParser*版本。在解析期间不会进行内存复制，并且各个读取都被传递为对内存中加载的块的引用。这种方法在Rust的[needletail](https://github.com/onecodex/needletail)解析器中实现。尽管Mojo仍然是一门年轻的语言，但我的实现在Apple Silicon上比Rust实现快了**50%**，比*SeqIO*快了**100x**。在质量修剪方面，*MojoFastTrim 🔥*平均比高度优化的Python/Cython的*Cutadapt*快了**2x**。

这个基准可以通过[按照这里的说明](https://github.com/MoSafi2/MojoFastTrim?tab=readme-ov-file#setup)来重现。

### 最后的思考

对于想要编写更具性能的Python程序员来说，Mojo是一个值得尝试和学习的好工具。然而，语言和生态系统仍在成长，我不得不使用打印调试来深入了解我遇到的错误。调试器仍处于预览状态且未经记录，尽管他们告诉我它很快将正式推出！

总之，我认为Mojo可以成为Python训练的科学家和研究人员在许多领域进行激进改变的工具。它可以让他们在性能和控制方面达到以前无法实现的水平。

感谢阅读！
