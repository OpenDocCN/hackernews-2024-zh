<!--yml

category: 未分类

date: 2024-05-27 14:45:26

-->

# 小优化的力量

> 来源：[https://maksimkita.com/blog/power-of-small-optimizations.html](https://maksimkita.com/blog/power-of-small-optimizations.html)

## 概览

在本文中，我将讨论一些能够提供大量性能改进的小优化的重要性。

一般来说，当我考虑不同的优化时，我会将它们放在不同层次的层次结构中：

1.  架构级别的优化。

1.  算法和数据结构级别的优化。

1.  源代码级别的优化。

通常，来自层次结构较高的优化对应用程序的影响比来自较低层次的优化更大。如果在架构层面上，你的应用程序设计不够优化，或者算法和数据结构的选择是错误的，那么花费时间在低级别的优化上就没有多大意义。

对于低级别的优化，有时人们认为这些优化总是需要使用一些复杂的算法或者在内联汇编中编写代码，但其中许多都相当简单。事实上，根据我的实践，清晰简洁的代码通常也是快速的，并且优化并不总是会使代码变得极其难以阅读或思考。如果你的应用程序组件良好隔离，这也很好，这样你可以在不影响其他组件或应用程序其余部分的情况下更改某些组件的内部实现。

当我谈论“小”优化时，它们很可能位于优化层次结构的较低层次上，更多是源代码级别的优化，如果我们以代码行数来衡量，它们相对较少。这并不意味着这些优化很容易，因为有时编写“正确”的几行代码可能非常困难，耗时很长。

## 如何找到优化的位置

通常，你需要对应用程序进行良好的自省，并始终在生产和开发过程中对其进行性能分析。你还需要保持好奇心，探索每一个可能的性能优化机会。即使在你的应用程序中高度优化的地方，也可以进一步进行优化。

一些算法和数据结构选择的优化示例：

1.  使用混合算法。某些算法或数据结构在数据量较小时可能效果良好，但当数据量增大时，底层算法或数据结构需要进行更改。

1.  使用统计数据进行运行时优化。所有算法的性能都受数据分布的影响。例如，如果你事先知道你的数据的基数，就可以选择更快的算法或数据结构。

1.  使用特化。你可以为特定的数据类型专门设计算法和数据结构。例如，如果你知道你需要对整数进行排序，那么使用基数排序而不是通用的基于比较的排序算法如pdqsort是有意义的。另一个例子是，如果你需要对整数进行排序，但它们总是或几乎总是有序的，那么基数排序要比pdqsort慢得多。

没有一种万能的解决方案或最佳算法适用于所有任务。你需要为你特定的任务选择最快的可能算法。

一些源代码级别优化的示例：

1.  避免内存复制。

1.  避免不必要的分配。

1.  改变数据布局以减少内存使用和改善缓存局部性。

1.  使用手动SIMD指令或手动循环展开。

在你的CI/CD流水线中，拥有性能测试是非常重要的一部分，它可以帮助你验证优化结果或发现回归。性能测试也可以在[寻找可以优化的地方](cpu-dispatch-in-clickhouse.html#cpu-dispatch-optimization-places)时非常有帮助。

你需要不断思考“如何提高你的应用程序的性能”，并始终检查所有可用的内省，寻找潜在的可以改进的地方。例如，在开发过程中，你可以始终检查`perf-top`中的所有热函数，以了解是否可以进行改进。查看[火焰图](https://www.brendangregg.com/flamegraphs.html)并尝试理解你的应用程序的整体运行情况以及它在哪里花费了时间，也是非常好的。

优化始终需要从应用程序执行中占用大部分时间的地方开始。但通常，在优化了每个这样的地方之后，很难理解接下来应该优化哪些地方。例如，当你调查某个潜在的地方时，如果你已经知道这个地方占整个应用程序执行时间的3-5%，那么仍然改善这个地方非常重要，即使你只能通过少量的性能提升来优化这个地方。如果你优化了许多这样的地方，这些优化的综合结果将对整个应用程序产生影响。

对于多线程应用程序，了解你的应用程序在哪些地方以单线程处理数据也很重要。这种单线程执行阶段对于许多算法如排序和聚合都很常见。你可以通过多线程对数据进行排序/聚合，但最终的合并阶段是单线程的。这也适用于这些算法的分布式执行。即使在单线程合并阶段进行了轻微的改进，这也是一个巨大的胜利，因为这些地方不会随着线程或服务器数量的增加而扩展。

## 示例

我将提供一个这类优化的例子，其中[小的源代码级别改变](https://github.com/ClickHouse/ClickHouse/pull/57717)导致了性能显著提升。

在 2023 年 12 月，在开发某些 ClickHouse 特性期间，当我运行一些读取大量字符串列的查询时，我注意到在 `perf-top` 和火焰图中，我们可以花费大约 20-40% 的查询执行时间在字符串反序列化上。我知道字符串反序列化位置在 ClickHouse 中已经被大幅优化，但我决定深入挖掘。

在`perf-top`中，我看到了这样的内容：

```
Samples: 1M of event 'cycles', 4000 Hz, Event count (approx.): 756969039682 lost: 0/0 drop: 0/17041
Overhead  Shared Object                   Symbol
  39.00%  clickhouse                      [.] DB::deserializeBinarySSE2<1>
  15.12%  clickhouse                      [.] DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>
  13.57%  clickhouse                      [.] DB::PODArrayDetails::byte_size
   9.36%  clickhouse                      [.] LZ4::(anonymous namespace)::decompressImpl<16ul, true>
   4.36%  [kernel]                        [k] copy_user_generic_string
   2.60%  clickhouse                      [.] DB::FunctionStringOrArrayToT<DB::LengthImpl, DB::NameLength, unsigned long, false>::executeImpl
   2.55%  clickhouse                      [.] CityHash_v1_0_2::CityHash128WithSeed
   2.31%  clickhouse                      [.] LZ4::(anonymous namespace)::decompressImpl<16ul, false>
   1.40%  clickhouse                      [.] memcpy
   1.15%  clickhouse                      [.] DB::findExtremeImplAVX2<unsigned long, DB::MaxComparator<unsigned long>, true, false>
   0.52%  [kernel]                        [k] filemap_get_read_batch
   0.52%  clickhouse                      [.] LZ4::(anonymous namespace)::decompressImpl<8ul, true>
   0.40%  clickhouse                      [.] LZ4::(anonymous namespace)::decompressImpl<32ul, false>

```

大部分时间都花在了`DB::deserializeBinarySSE2`上，这是可以预期的。令人意外的是，我们看到了 `DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>` 和 `DB::PODArrayDetails::byte_size` 方法。如果你检查 `DB::deserializeBinarySSE2` 的汇编代码，你会注意到 `PODArray` 的 `resize` 函数从中调用，并且该函数调用没有被内联。

```
  ...

  0.32 │    │  inc    %r13
  1.25 │    │  mov    %r13,(%rax)
  5.19 │    │  add    $0x8,%rax
  0.02 │    │  mov    %rax,0x8(%rbp)
  0.23 │    │  mov    0x30(%rsp),%r12
  0.36 │    │  mov    %r12,%rdi
  0.14 │    │  mov    %r13,%rsi
  2.67 │    │→ callq  DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>
  2.95 │    │  mov    0x28(%rsp),%rdx
  1.70 │    │  test   %rdx,%rdx
  1.51 │    │↑ je     51
  0.00 │    │  lea    0x11(%r15),%rax

  ...

```

现在，如果我们检查 `DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>` 的汇编代码，我们会注意到它调用了 `DB::PODArrayDetails::byte_size` 函数，我们还可以看到 `PODArray` 的 `resize` 函数调用开销很大。

```
  ...

  0.10 │104:   mov   $0x1,%esi
  0.71 │       mov   %r14,%rdi
  5.92 │     → callq DB::PODArrayDetails::byte_size
  5.63 │       add   %r12,%rax
  0.10 │       mov   %rax,0x8(%rbx)
 30.30 │       pop   %rbx
  0.76 │       pop   %r12
  1.96 │       pop   %r13
  4.42 │       pop   %r14
  2.89 │       pop   %r15
 11.24 │     ← retq

  ...

```

在 C++ 代码中，`deserializeBinarySSE2` 函数看起来像这样：

```
template <int UNROLL_TIMES>
static NO_INLINE void deserializeBinarySSE2(ColumnString::Chars & data,
    ColumnString::Offsets & offsets,
    ReadBuffer & istr,
    size_t limit)
{
    size_t offset = data.size();
    for (size_t i = 0; i < limit; ++i)
    {
        if (istr.eof())
            break;

        UInt64 size;
        readVarUInt(size, istr);

        ...

        data.resize(offset);

        if (size)
        {

#ifdef __SSE2__
            /// An optimistic branch in which more efficient copying is possible.
            if (offset + 16 * UNROLL_TIMES <= data.capacity() &&
                istr.position() + size + 16 * UNROLL_TIMES <= istr.buffer().end())
            {
                ...
            }
            else
#endif
            {
                istr.readStrict(reinterpret_cast<char*>(&data[offset - size - 1]), size);
            }
        }

        data[offset - 1] = 0;
    }
}

```

解决方案很简单：在这个特定的反序列化位置，我们将 `PODArray` 作为一个字符缓冲区处理，并且我们可以手动控制调整过程，并使用某个常数调整因子，例如 2。我们还使用 `resize_exact` 函数来减少内存分配大小。因此，在循环内部，我们替换为：

```
data.resize(offset);

```

其中：

```
if (unlikely(offset > data.size()))
    data.resize_exact(roundUpToPowerOfTwoOrZero(std::max(offset, data.size() * 2)));

```

结果是，在优化测试中使用的查询性能有了显著改善：

之前：

```
SELECT max(length(value)) FROM test_table FORMAT Null

0 rows in set. Elapsed: 0.855 sec. Processed 1.50 billion rows, 19.89 GB (1.75 billion rows/s., 23.27 GB/s.)
Peak memory usage: 1.24 MiB.

```

后来：

```
SELECT max(length(value)) FROM test_table FORMAT Null

0 rows in set. Elapsed: 0.691 sec. Processed 1.50 billion rows, 19.89 GB (2.17 billion rows/s., 28.79 GB/s.)
Peak memory usage: 1.17 MiB.

```

来自 ClickHouse CI 的性能测试结果：

| 查询 | 旧 (s) | 新 (s) | 加速比(-)或减速比(+) | 相对差异 (新 - 旧) / 旧 |
| --- | --- | --- | --- | --- |
| 从 empty_strings 中选择 count() WHERE NOT ignore(s) | 0.449 | 0.27 | -1.662x | -0.399 |
| 从 hits_100m_single 中选择 anyHeavy(OpenstatSourceID) WHERE OpenstatSourceID != '' GROUP BY intHash32(UserID) % 1000000 格式 Null | 0.088 | 0.066 | -1.328x | -0.247 |
| 从 hits_100m_single 中选择 anyHeavy(OpenstatCampaignID) WHERE OpenstatCampaignID != '' GROUP BY intHash32(UserID) % 1000000 格式 Null | 0.088 | 0.066 | -1.32x | -0.243 |
| 从 hits_100m_single 中选择 count() WHERE NOT ignore(format('{}Hello{}', MobilePhoneModel, PageCharset)) | 0.321 | 0.28 | -1.147x | -0.129 |
| 从 hits_100m_single 中选择 count() WHERE NOT ignore(concat(MobilePhoneModel, SearchPhrase)) | 0.337 | 0.298 | -1.131x | -0.117 |
| 从 test_full_10 中选择 str 格式 Null | 0.074 | 0.058 | -1.282x | -0.22 |
| 从 hits_100m_single 中选择 count() WHERE NOT ignore(substring(PageCharset, 1, 2)) | 0.135 | 0.116 | -1.16x | -0.138 |

由于优化，这种优化平均提高了那些在字符串反序列化上花费大量执行时间的查询的性能，平均提高了10-20%。对于某些查询，甚至达到60%。

## 结论

有时，你可以通过使用一些高级复杂的优化手段，不仅仅是通过使用更好的算法和数据结构来完成你特定任务，或者是通过小的源代码层次优化，来实现显著的性能提升。
