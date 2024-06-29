<!--yml

category: 未分类

date: 2024-05-27 15:01:28

-->

# 为什么需要 shuffle 支持？洗牌执行计划分析 | MatrixOrigin | Medium

> 来源：[https://medium.com/@matrixorigin-database/why-is-shuffle-support-needed-5df13e8edaf2](https://medium.com/@matrixorigin-database/why-is-shuffle-support-needed-5df13e8edaf2)

# 为什么需要 shuffle 支持？

# 洗牌执行计划分析

洗牌算法的基本原理是将输入数据分桶，每个桶的数据可以独立处理并输出结果。这样做可以在聚合数据时减小哈希表的大小。

以分组操作符为例，有两种实现方法：sort group 和 hash group。

在 hash group 中，通过多个并发操作，每个操作对输入数据进行哈希并在哈希表中处理。所有哈希表都需要合并在一起才能获得最终结果。这种方法称为 merge group。

另一种方法是首先洗牌和分发数据，每个并发操作接收和处理不同的数据，并直接输出结果，而无需将哈希表合并在一起。这种方法称为 shuffle group。

## **为什么需要 shuffle 支持？主要原因如下。**

**解决因大型哈希表导致的内存分配失败问题：** 以 merge group 为例，由于所有数据需要合并在一起，如果最终合并的哈希表的基数过大，所需的内存分配将非常大，导致内存分配失败。超出单个 CN（计算节点）的内存限制并且不考虑溢出情况还可能导致内存不足（OOM）错误。与此同时，shuffle 可以将一个大型哈希表分割成多个较小的哈希表进行处理。

**增强计算节点的水平可伸缩性：** 仍以 merge group 为例，由于所有数据需要合并在一起，处理在单个 CN 上的单一并发。如果哈希表的基数很大，此步骤将变慢且成为瓶颈，并且增加更多的 CN 也无法加速这样的查询。shuffle group 不具有这种单点瓶颈。

**提高性能：** 当哈希表过大时，对哈希表的随机访问会导致非常高的缓存未命中率，而缓存未命中显著降低性能。通过使用 shuffle 将哈希表分割成多个较小的哈希表进行处理，可以大大提高对每个小哈希表的随机访问缓存命中率，从而提高整体性能。

# Part2: Shuffle Implementation Principle

洗牌将数据分桶，桶的数量在流水线编译期间根据运行时环境中的 CNs 和 CPU 核心数动态确定。例如，如果当前租户的环境有 3 个 CN，每个 CN 有 10 个核心，洗牌操作符将自动计算桶的数量为 30。

shuffle 有两种分桶算法：range shuffle 和 hash shuffle。

哈希 shuffle 直接通过哈希函数获取数据的桶编号。

Range shuffle 要求优化器在编译阶段提供分桶策略。

对于均匀分布的数据，分桶策略相对简单。例如，以 tpch1T 的 lineitem 表为例，l_orderkey 的范围是从 1 到 60 亿，且均匀分布。对这一列进行 shuffle 的策略是基于最小值和最大值均匀分布，如 1 亿到 2 亿进入桶 1，2 亿到 4 亿进入桶 2，依此类推。

对于非均匀分布的数据，处理算法会更加复杂，稍后将详细介绍。

在编译阶段，MO 的优化器决定是否使用 range shuffle 或者 hash shuffle，通常优先选择 range shuffle。如果由于某些原因无法使用 range shuffle，则会使用 hash shuffle 来确保桶分布的均匀性。

**在 MO 管道中，有两个与 shuffle 算法对应的运算符：shuffle 和 dispatch。**

**shuffle 运算符的功能是对输入数据进行分桶**，并输出分桶后的数据。

例如，输入一个批次（MO 是列存储数据库，运算符将数据批次作为基本单位处理），批次包含 8192 行数据，值从 1 到 8192。如果需要将其分成两个桶，可能的分桶结果是将 1 到 4096 放入一个新批次，4097 到 8192 放入另一个新批次，然后发送这两个新批次出去。

由于可能存在非常高的并发性，每个桶中的数据在分桶后可能非常稀疏，因此直接输出是不适合的。因此，shuffle 运算符需要维护一个内部内存池，用于存储分桶后的数据，并在积累到一定量时发送出去。或者，在 shuffle 运算符读取完所有输入数据后，需要将所有数据内部发送出去。

在流水线中，大多数运算符的一个批次输入对应一个批次输出，但是 shuffle 运算符不是这样工作的。由于内部内存池的存在，shuffle 可能有一段时间没有输出，或者可能连续输出多个批次。内存池的策略对 shuffle 运算符的性能有重大影响。

**调度运算符负责将经过 shuffle 分桶的数据发送到指定位置**。shuffle 运算符在批处理中设置一个名为 shuffleIDX 的字段，调度运算符读取此字段，并根据指定的策略将批次发送到目的地。需要注意的是 shuffleIDX 字段不参与批次的序列化和反序列化过程，因此 shuffle 和 dispatch 运算符之间不应该跨 CN 网络传输，否则会导致错误。

dispatch 运算符有三种发送数据的策略。第一种策略在绝大多数情况下使用，而后两种策略用于稍后引入的混合 shuffle 中。

1\. 根据 shuffleIDX 直接找到唯一对应的目的地（可能在本地 CN 或远程 CN），并发送数据。可能会跨 CN 进行网络传输。

2\. 根据 shuffleIDX，在当前 CN 中找到相应的目的地并发送数据。在这种情况下，只会发送本地副本，不会跨 CN 进行网络传输。

3\. 根据 shuffleIDX，为每个 CN 找到相应的目的地并发送数据。在这种情况下，每个 CN 都会收到一份副本，肯定会跨 CN 进行网络传输。

**目前，MO 支持为四种操作符启用 shuffle：load、scan、group 和 join。** 然而，理论上，任何操作符都可以与 shuffle 无关，MO 将来会支持更多操作符的 shuffle 算法启用。

## 加载运算符

Load 运算符部分重用 shuffle 的分发逻辑，以确保写入 S3 的数据在多个并发读写时保持其原始顺序。

## scan 运算符

scan 运算符的 shuffle 在管道编译阶段完成，将需要读取的所有块分布到多个 CN 中。这里的 shuffle 是按块进行的，在实际块数据读取阶段之前完成。如果是哈希 shuffle，则对块名称进行哈希处理。如果是范围 shuffle，则取块 zonemap 中最小最大值的中间值进行范围计算。

## group 运算符

group 运算符的 shuffle 在管道编译期间静态地将 group 运算符展开为多个管道，对 scan 运算符实际读取的块数据进行洗牌，理论上是按行进行的。每个管道接收 shuffle 输出的数据。此时，group 运算符不需要合并组，因为每个独立的管道可以直接将数据输出给下一个管道。

## 连接运算符

join 运算符的 shuffle 在管道编译期间也静态地将 join 运算符展开为多个管道。join 的输入被分为探测和构建两侧，并且使用相同的 shuffle 算法来处理两侧的输入数据，以确保相同的数据分布到同一个桶中。每个 join 运算符也可以直接输出 join 结果，无需合并。

要检查语句的执行计划是否启用了 shuffle，可以使用 explain 语句查看执行计划中是否包含 shuffle 关键字。

例如，在tpch q4中，您可以看到lineitem表与orders表进行了连接，连接类型为右半连接，lineitem作为探测表，orders作为构建表。启用了Shuffle连接，并对lineitem表的l_orderkey列执行了范围Shuffle。由于连接两个表使用了相同的Shuffle算法，因此也将对orders表的o_orderkey列执行Shuffle。

```
QUERY PLAN
Project
  ->  Sort
        Sort Key: orders.o_orderpriority INTERNAL
        ->  Aggregate
              Group Key: orders.o_orderpriority
              Aggregate Functions: starcount(1)
              ->  Join
                    Join Type: RIGHT SEMI   hashOnPK
                    Join Cond: (lineitem.l_orderkey = orders.o_orderkey) shuffle: range(lineitem.l_orderkey)
                    ->  Table Scan on tpch_10g.lineitem
                          Filter Cond: (lineitem.l_commitdate < lineitem.l_receiptdate)
                          Block Filter Cond: (lineitem.l_commitdate < lineitem.l_receiptdate)
                    ->  Table Scan on tpch_10g.orders
                          Filter Cond: (orders.o_orderdate >= 1997-07-01), (orders.o_orderdate < 1997-10-01)
                          Block Filter Cond: (orders.o_orderdate >= 1997-07-01), (orders.o_orderdate < 1997-10-01)
```
