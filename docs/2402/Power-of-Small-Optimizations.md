<!--yml
category: 未分类
date: 2024-05-27 14:45:26
-->

# Power of Small Optimizations

> 来源：[https://maksimkita.com/blog/power-of-small-optimizations.html](https://maksimkita.com/blog/power-of-small-optimizations.html)

## Overview

In this post, I will talk about the importance of small optimizations that can provide a lot of performance improvements.

Generally, when I think about different optimizations, I place them in a hierarchy of different levels:

1.  Architecture level optimizations.

2.  Algorithms and data structures level optimizations.

3.  Source code level optimizations.

Typically, optimizations from higher levels of this hierarchy have more impact on your application than those from lower levels. It does not make a lot of sense to spend time on lower-level optimizations if, on the architecture level, your application is designed in a suboptimal way or the algorithms and data structures choices are wrong.

For lower-level optimizations, sometimes people think that those optimizations always require the usage of some hardcore algorithms or writing code in inline assembly, but a lot of them are fairly simple. In fact, from my practice, clear and simple code is generally fast, and optimizations do not always make code extremely hard to read or think about. It is also great if your application components are well isolated so you can change the internal implementation of some components without affecting other components or the rest of the application.

When I talk about “small” optimizations, they are most likely placed on a lower level of optimizations hierarchy, more often source code level optimizations, and if we measure them in lines of code, they are relatively small. It does not mean that such optimizations are easy because sometimes writing a “right” couple lines of code can be extremely hard and take a lot of time.

## How to find places for optimizations

Generally, you need to have good introspection for your application and always profile your application, both in production and during development. You also need to be curious to explore every possible performance optimization opportunity. Even highly optimized places in your application can be optimized even further.

Examples of some optimizations for algorithms and data structures choice:

1.  Use hybrid algorithms. Some algorithms or data structures can work well when the amount of data is small, but when the amount of data grows, the underlying algorithm or data structure needs to be changed.

2.  Use statistics for run-time optimizations. All algorithm’s performance is affected by data distribution. For example, if you know the cardinality of your data in advance, it is possible to choose a faster algorithm or data structure.

3.  Use specializations. You can specialize your algorithms and data structures for specific data types. For example, if you know that you need to sort integers, it makes sense to use radix sort instead of general comparison-based sort algorithms like pdqsort. Another example is if you need to sort integers, but they are always sorted or almost sorted, radix sort will be much slower than pdqsort.

There is no silver bullet or best algorithm for any task. You need to choose the fastest possible algorithm for your specific task.

Examples of some source code level optimizations:

1.  Avoid memory copying.

2.  Avoid unnecessary allocations.

3.  Change data layout to reduce memory usage and improve cache locality.

4.  Use manual SIMD instructions or manual loop unrolling.

It is also important to have performance tests as part of your CI/CD pipeline that can help you verify your optimization results or find regressions. Performance tests can also be very helpful in [finding places that you can optimize](cpu-dispatch-in-clickhouse.html#cpu-dispatch-optimization-places).

You need to constantly think about “How the performance of your application can be improved” and always check all available introspection for potential places that can be improved. For example, you can always check all hot functions in `perf-top` during development to understand if anything can be improved. It is also great to check the [flame graphs](https://www.brendangregg.com/flamegraphs.html) and try to understand the big picture of how your application works and where it spends time.

You always need to start optimization with places that take most of the time during application execution. But usually, after you optimize every such place, it is hard to understand which places to optimize next. For example, when you investigate some potential place, and you already know that this place takes 3-5% of the whole application execution time, it is important to still improve this place, even if you can improve this place’s performance only by a small amount. If you optimize many such places, the compound result of such optimizations will be visible for the whole application.

For multithreaded applications, it is also important to understand in which places your application processes data in a single thread. Such single-thread execution stage is common for many algorithms like sorting and aggregation. You sort/aggregate data by multiple threads, but the final merge stage is single-threaded. This also applies to the distributed execution of such algorithms. Even a small improvement in that single-thread merge stage is a big win because those places do not scale with an increased amount of threads or servers.

## Example

I will provide an example of one of such optimizations, where [small source code level change](https://github.com/ClickHouse/ClickHouse/pull/57717) lead to great performance improvement.

In December 2023, during the development of some ClickHouse features, when I ran some queries that read a lot of String columns, I noticed in the `perf-top` and flame graphs that we can spend around 20-40% of query execution time on strings deserialization. I knew that string deserialization place was already heavily optimized in ClickHouse, but I decided to dig deeper.

In `perf-top`, I saw something like this:

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

Most of the time is spent in `DB::deserializeBinarySSE2`, and it is expected. What is not expected that we see `DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>` and `DB::PODArrayDetails::byte_size` methods. If you check the `DB::deserializeBinarySSE2` assembly, you can notice that the PODArray `resize` function is called from it, and that function call is not inlined.

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

Now, if we check the `DB::PODArrayBase<1ul, 4096ul, Allocator<false, false>, 63ul, 64ul>::resize<>` assembly, we will notice it calls `DB::PODArrayDetails::byte_size` function and we can also see that PODArray `resize` function call overhead is high.

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

In C++ code `deserializeBinarySSE2` function looked like this:

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

The solution was simple: in this specific deserialization place, we work with `PODArray` as just a characters buffer, and we can manually control the resize process and resize buffer with some constant resize factor, for example, 2\. We also use the `resize_exact` function to reduce memory allocation size. So inside the loop, we replace:

```
data.resize(offset);

```

with:

```
if (unlikely(offset > data.size()))
    data.resize_exact(roundUpToPowerOfTwoOrZero(std::max(offset, data.size() * 2)));

```

As a result, we have such performance improvement for a query that I used for the optimization test:

Before:

```
SELECT max(length(value)) FROM test_table FORMAT Null

0 rows in set. Elapsed: 0.855 sec. Processed 1.50 billion rows, 19.89 GB (1.75 billion rows/s., 23.27 GB/s.)
Peak memory usage: 1.24 MiB.

```

After:

```
SELECT max(length(value)) FROM test_table FORMAT Null

0 rows in set. Elapsed: 0.691 sec. Processed 1.50 billion rows, 19.89 GB (2.17 billion rows/s., 28.79 GB/s.)
Peak memory usage: 1.17 MiB.

```

Results of performance tests from ClickHouse CI:

| Query | Old (s) | New (s) | Ratio of speedup(-) or slowdown(+) | Relative difference (new - old) / old |
| --- | --- | --- | --- | --- |
| SELECT count() FROM empty_strings WHERE NOT ignore(s) | 0.449 | 0.27 | -1.662x | -0.399 |
| select anyHeavy(OpenstatSourceID) from hits_100m_single where OpenstatSourceID != '' group by intHash32(UserID) % 1000000 FORMAT Null | 0.088 | 0.066 | -1.328x | -0.247 |
| select anyHeavy(OpenstatCampaignID) from hits_100m_single where OpenstatCampaignID != '' group by intHash32(UserID) % 1000000 FORMAT Null | 0.088 | 0.066 | -1.32x | -0.243 |
| SELECT count() FROM hits_100m_single WHERE NOT ignore(format('{}Hello{}', MobilePhoneModel, PageCharset)) | 0.321 | 0.28 | -1.147x | -0.129 |
| SELECT count() FROM hits_100m_single WHERE NOT ignore(concat(MobilePhoneModel, SearchPhrase)) | 0.337 | 0.298 | -1.131x | -0.117 |
| SELECT str FROM test_full_10 FORMAT Null | 0.074 | 0.058 | -1.282x | -0.22 |
| SELECT count() FROM hits_100m_single WHERE NOT ignore(substring(PageCharset, 1, 2)) | 0.135 | 0.116 | -1.16x | -0.138 |

As a result, this optimization improved performance for queries that spend a lot of execution time on string deserialization by 10-20% on average. For some queries, even for 60%.

## Conclusion

Sometimes, you can achieve significant performance improvement not only by using some high-level complex optimizations but also by using better algorithms and data structures for your specific tasks or by small source code level optimizations.