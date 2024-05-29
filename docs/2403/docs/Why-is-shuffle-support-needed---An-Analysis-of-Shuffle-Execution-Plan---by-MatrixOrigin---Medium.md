<!--yml
category: 未分类
date: 2024-05-27 15:01:28
-->

# Why is shuffle support needed?. An Analysis of Shuffle Execution Plan | by MatrixOrigin | Medium

> 来源：[https://medium.com/@matrixorigin-database/why-is-shuffle-support-needed-5df13e8edaf2](https://medium.com/@matrixorigin-database/why-is-shuffle-support-needed-5df13e8edaf2)

# Why is shuffle support needed?

# An Analysis of Shuffle Execution Plan

The basic principle of the shuffle algorithm is to bucket the input data, where each bucket’s data can be processed independently and output results. This reduces the size of the hash table when aggregating data.

Taking the group operator as an example, there are two implementation methods: sort group and hash group.

In hash group, with multiple concurrent operations, each operation hashes the input data and processes it in a hash table. All hash tables need to be merged together to obtain the final result. This method is called merge group.

Another method is to first shuffle and distribute the data, where each concurrent operation receives and processes different data, and directly outputs the results, without needing to merge the hash tables together. This method is called shuffle group.

## **Why is shuffle support needed? The main reasons are as follows.**

**Solving the problem of memory allocation failure due to large hash tables:** Taking merge group as an example, since all data need to be merged together, if the cardinality of the final merged hash table is too large, the required memory allocation will be very large, leading to memory allocation failure. Exceeding the memory limit of a single CN (compute node) and not considering the spill scenario can also cause out-of-memory (OOM) errors. Meanwhile, shuffle can split a large hash table into multiple smaller hash tables for processing.

**Enhancing the horizontal scalability of compute nodes:** Still taking merge group as an example, since all data need to be merged together, processing is done in single concurrency on a single CN. If the hash table has a large cardinality, this step becomes slow and a bottleneck, and adding more CNs cannot speed up such queries. Shuffle group does not have this single-point bottleneck.

**Improving performance:** When hash tables are too large, random access to the hash table leads to a very high probability of cache misses, and cache misses significantly reduce performance. By using shuffle to split the hash table into multiple smaller hash tables for processing, the cache hit rate for random access to each small hash table is greatly increased, thereby improving overall performance.

# Part2: Shuffle Implementation Principle

Shuffle buckets data, and the number of buckets is dynamically determined by the number of CNs and CPU cores in the runtime environment during pipeline compilation. For example, if the current tenant’s environment has 3 CNs, each with 10 cores, the shuffle operator will automatically calculate the number of buckets as 30.

There are two bucketing algorithms for shuffle: range shuffle and hash shuffle.

Hash shuffle directly obtains the bucket number of the data through a hash function.

Range shuffle requires the optimizer to provide a bucketing strategy during the compilation stage.

For uniformly distributed data, the bucketing strategy is relatively simple. For example, take the tpch1T, lineitem table as an example. The range of l_orderkey is from 1 to 6 billion, and it is evenly distributed. The strategy for shuffling this column is to evenly distribute it based on the minimum and maximum values, with 100 million to 200 million going to bucket 1, 200 million to 400 million to bucket 2, and so on.

For non-uniformly distributed data, the processing algorithm is more complex and will be detailed later.

During the compilation stage, MO’s optimizer decides whether to use range shuffle or hash shuffle, usually prioritizing range shuffle. If, for some reason, range shuffle cannot be used, hash shuffle will be used to ensure the uniformity of bucket distribution.

**In the MO Pipeline, there are two operators corresponding to the shuffle algorithm: shuffle and dispatch.**

**The function of the shuffle operator is to bucket the input data** and output the data after bucketing.

For example, input a batch (MO is a columnar storage database, and the operator processes data in batches as the basic unit), the batch contains 8192 rows of data, valued from 1, 2, 3 … up to 8192\. If it needs to be split into two buckets, a possible bucketing result is to put 1–4096 into a new batch, and 4097–8192 into another new batch, and then send these two new batches out.

Due to the possibility of very high concurrency, the data in each bucket may be sparse after bucketing, so direct output is not suitable. Therefore, the shuffle operator needs to maintain an internal memory pool to store the bucketed data and send it out when a certain amount is accumulated. Or, when the shuffle operator has read all the input data, it needs to send out all the data internally.

In the pipeline, most operators have one batch input corresponding to one batch output, but the shuffle operator does not work this way. Due to the internal memory pool, shuffle may have no output for a while, or it may continuously output multiple batches. The strategy of the memory pool has a significant impact on the performance of the shuffle operator.

**The dispatch operator is responsible for sending the data bucketed** by shuffle to a specified location. The shuffle operator sets a field named shuffleIDX in the batch, and the dispatch operator reads this field and sends the batch to the destination according to a specified strategy. It is important to note that the shuffleIDX field does not participate in the serialization and deserialization of the batch, so there must not be cross-CN network transmission between the shuffle and dispatch operators, otherwise, it will cause errors.

The dispatch operator has three strategies for sending data. The first strategy is used in the vast majority of cases, while the latter two strategies are used in hybrid shuffle, which will be introduced later.

1\. Directly find the unique corresponding destination based on shuffleIDX (which may be in a local CN or a remote CN) and send it there. There may be cross-CN network transmission.

2\. Based on shuffleIDX, find the corresponding destination in the current CN and send it there. In this case, only a local copy is sent, and there will be no cross-CN network transmission.

3\. Based on shuffleIDX, find the corresponding destination for each CN and send it there. In this case, a copy is sent to every CN, and there will definitely be cross-CN network transmission.

**Currently, MO supports enabling shuffle for four operators: load, scan, group, and join.** However, in theory, any operator is agnostic to shuffle, and MO will support enabling the shuffle algorithm for more operators in the future.

## Load Operator

The Load operator partially reuses the distribution logic of shuffle to ensure that data written to S3 can retain its original order in the case of multiple concurrent reads and writes.

## Scan Operator

The shuffle for the scan operator is done during the pipeline compilation stage, distributing all blocks that need to be read across multiple CNs. The shuffle here is done by block, before the actual block data reading stage. If it’s hash shuffle, it hashes the block name. If it’s range shuffle, it takes the mid-value of the min max in the block zonemap for range calculation.

## Group Operator

The shuffle for the group operator statically expands the group operator into multiple pipelines during pipeline compilation, shuffling the block data actually read by the scan operator, theoretically on a per-row basis. Each pipeline receives the data outputted by the shuffle. At this point, the group operator does not need to merge groups, as each separate pipeline can directly output data to the next pipeline.

## Join Operator

The shuffle for the join operator also statically expands the join operator into multiple pipelines during the pipeline compilation. The input of a join is divided into the probe and build sides, and the same shuffle algorithm is used for the input data on both sides to ensure that the same data is distributed to the same bucket. Each join operator can also directly output the join result without needing to merge.

To check if a statement’s execution plan has shuffle enabled, you can use the explain statement to see if the execution plan contains the shuffle keyword.

For example, in tpch q4, you can see the lineitem table join the orders table, with the join type being right semi, lineitem as the probe table, and orders as the build table. Shuffle join is enabled, and a range shuffle is performed on the l_orderkey column of the lineitem table. Since the same shuffle algorithm is used for joining the two tables, a shuffle will also be performed on the o_orderkey column of the orders table.

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