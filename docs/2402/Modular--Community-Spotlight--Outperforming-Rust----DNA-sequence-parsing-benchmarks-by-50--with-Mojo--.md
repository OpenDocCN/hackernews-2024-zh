<!--yml
category: 未分类
date: 2024-05-27 14:40:20
-->

# Modular: Community Spotlight: Outperforming Rust ⚙️ DNA sequence parsing benchmarks by 50% with Mojo 🔥

> 来源：[https://www.modular.com/blog/outperforming-rust-benchmarks-with-mojo](https://www.modular.com/blog/outperforming-rust-benchmarks-with-mojo)

> This is a guest blog post by Mohamed Mabrouk. Mohamed is the creator of the [MojoFastTrim](https://github.com/MoSafi2/MojoFastTrim), a Mojo 🔥 community project. Mohamed achieved 100x benchmark improvements over Python, and a 50% improvement over the fastest implementation in Rust. He quickly learned the language, and it took only 200 lines of code for the first implementation. Read on for details about the extra optimizations he applied, to beat the fastest existing benchmarks!

### The era of big-data in bioinformatics

The challenges for bioinformatics in the modern day, are rooted in big-data manipulation. Thousands of multi-million dollar DNA-sequencing machines are working non-stop in all fields of biotechnology, medicine, and biomedical research. The annual sequencing data size is expected to be up to [40 exabytes](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4494865/) of raw sequences by 2025\. That's **20x** the data uploaded to YouTube every year.

While most of the final analysis is carried out in high-level languages like Python and R, the world of bioinformatics is powered by an underlayer of black magic! Highly-optimized tools written in C, C++, and Java that pre-process and summarize large amount of raw data. This creates a two-world problem where bioinformaticians who are not skilled in low-level languages, are prohibited from understanding, customizing, and implementing low-level operations. In addition, typical bioinformatic pipelines are a mixture of Bash and Python scripts calling into pre-compiled binaries, along with the analysis logic itself. It's becoming increasingly complex and frustrating for new and experienced bioinformaticians. This is the same issue that the AI community is facing.

### Mojo 🔥 one tool to rule them all

I first heard about Mojo from the demo video by [Jeremy Howard](https://www.youtube.com/watch?v=6GvB5lZJqcE). Its value offer is simple, a Pythonic language that allows the programmer to optimize at a much lower level, to unify the fragmentation in fields such as AI. Learning Mojo was relatively easy for me, coming from the Python world, I got used to the extra syntax in only a few days. I decided to try Mojo 🔥 in a serious project for a low-level bioinformatic task; FASTQ parsing and quality trimming. [FASTQ](https://en.wikipedia.org/wiki/FASTQ_format) is a basic format for most DNA sequencing operations, incorporating both the genomic sequence and confidence scores of the machine in each base call. It is a simple format to parse, with most records looking like this:

FASTQ

@SEQ_ID # New read head + read_id GATTTGGGGTTCAAAGCAGTATCGATCAAATAGTAAA... # DNA sequence + # Quality header + id (optional) !''*((((***+))%%%++)(%%%%).1***-+*'')... # Phred quality score

However, typical uncompressed file sizes are 1-50 GB, an average sequence-heavy study could generate north of 1 TB for a single file. Performance is critical in parsing and data manipulation.

I tried to write a simple parser that would:

1\. Read a chunk of the file as a *String*.
2\. Split the string on the newline *\n* separator.
3\. Take each 4 lines, validate that they are a consistent and correct FASTQ record, and return it.
4\. Rinse and repeat until reaching *EOF*.

On the [first try](https://github.com/MoSafi2/MojoFastTrim/tree/v0.1), *MojoFastTrim 🔥* achieved **8x** the performance of python's [SeqIO](https://biopython.org/wiki/SeqIO). I was pleasantly surprised with the development time. My code was still Pythonic, concise at around 200 lines, and using features the average python developer would understand. In quality trimming, where low quality bases are removed from each read, it achieved 50%-80% of the industry standard tool [Cutadapt](https://github.com/marcelm/cutadapt). This was a surprising level of performance for development time I put into the project.

### Going down the optimization rabbit hole

The most powerful benefit of Mojo 🔥 is that it gives you access to low-level optimizations. The nascent state of the Mojo standard library meant that I had to write, test, and benchmark some functions from the ground up. Mojo's first-class support of *SIMD* vectorization was really helpful and surprisingly intuitive. Here is the implementation of the vectorized version of a function to find the index of the newline separator in Mojo:

##### Iterative

Mojo

@always_inline fn find_chr_next_occurance_iter[ T: DType ](in_tensor: Tensor[T], chr: Int = 10, start: Int = 0) -> Int: for i in range(start, in_tensor.num_elements()): if in_tensor[i] == chr: return i return -1

##### SIMD Vectorized

Mojo

@always_inline fn find_chr_next_occurance_simd[ T: DType ](in_tensor: Tensor[T], chr: Int = 10, start: Int = 0) -> Int: alias simd_width = simdwidthof[T]() let len = in_tensor.num_elements() - start let aligned = start + math.align_down(len, simd_width) for s in range(start, aligned, simd_width): let v = in_tensor.simd_load[simd_width](s) let mask = v == chr if mask.reduce_or(): return s + arg_true(mask) # Handling other elements iteratively return -1

The vectorized version loads 32-elements of *Int8* and checks the presence of a new-line separator using fewer operations.

In the following graph, you can see the effect of SIMD vectorization. It provides up to **4x** speed up, with average speed up of **3.2x**. Similarly, *SIMD* storing and loading from tensors providers substantial performance gains.

In addition, I explored optimizations from C/C++ implementations. I was concerned that no explicit memory buffer was allocated for the loaded chunks, but the Mojo compiler was already taking care of that and avoiding new memory allocations:

Mojo

fn test_chunk_ptr_index() raises: let f = open("large_fastq_file.fastq", "r") var offset = 0 var chunk_no = 0 while True: let t = f.read_bytes(Size) print(t._ptr.__as_index()) offset += Size chunk_no += 1 _ = f.seek(offset) if t.num_elements() == 0: break # memory adddress of chunk 0 : 139711892942912 # memory adddress of chunk 1 : 139711892942912 # memory adddress of chunk 2 : 139711892942912 # memory adddress of chunk 3 : 139711892942912 # memory adddress of chunk 4 : 139711892942912

Implementing those optimizations resulted in an extra **3x** speedup, and *MojoFastTrim 🔥* was on average **24x** more performant than Python's *SeqIO*. In addition, due to control over reference and value semantics in Mojo 🔥 I applied a *FastParser* version of the parser. No memory copies are made during parsing and the individual reads are passed around as references to the loaded chunk in memory. This approach is implemented in Rust's [needletail](https://github.com/onecodex/needletail) parser. Although Mojo is still a young language, my implementation was **50%** faster than the Rust implementation on Apple Silicon, and **100x** faster than *SeqIO*. In quality trimming, *MojoFastTrim 🔥* was on average **2x** faster than the highly-optimized Python/Cython *Cutadapt*.

This benchmark can be reproduced by [following the instructions here.](https://github.com/MoSafi2/MojoFastTrim?tab=readme-ov-file#setup)

### Final thoughts

For Python programmers wanting to write more performant code, Mojo is a great tool to try, and easy to learn. However, the language and the ecosystem is still growing, I had to use print debugging to gain insight into the bugs I was encountering. The debugger is still in preview and undocumented, although they tell me it will be officially launching soon!

In conclusion, I think that Mojo can be a radical change for a wide range of Python trained scientists and researchers across many fields. It can enable them to have a level of performance and control, that was previously unachievable.

Thanks for reading!