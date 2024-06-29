<!--yml

category: 未分类

date: 2024-05-27 14:44:18

-->

# Postgres 16 查询规划器 / 优化器的新特性

> 来源：[https://www.citusdata.com/blog/2024/02/08/whats-new-in-postgres-16-query-planner-optimizer/](https://www.citusdata.com/blog/2024/02/08/whats-new-in-postgres-16-query-planner-optimizer/)

PostgreSQL 16 引入了多项查询规划器改进，使得许多 SQL 查询比在之前的 PostgreSQL 版本中运行得更快。

如果您查看 [PG16 发布说明](https://www.postgresql.org/docs/16/release-16.html)，您将看到一些这些规划器改进。但是由于每个 PostgreSQL 版本的变更量很大，无法为每个变更提供足够详细的信息。因此，您可能需要更多详细信息，以了解每项变更的具体内容 —— 从而确定其是否与您相关。

在本博客文章中，假设您已经掌握了 [EXPLAIN 的基础知识](https://www.postgresql.org/docs/16/using-explain.html#USING-EXPLAIN-BASICS)，您将深入了解 PostgreSQL 16 查询规划器中的 10 项改进。对于 PG16 规划器的每个改进（在其他关系数据库中通常称为优化器），您还将看到 PG15 和 PG16 规划器输出的比较 —— 以及自包含测试的示例，您可以自行尝试。

让我们深入了解 PostgreSQL 16 查询规划器中的这 10 项改进：

1.  [增量排序用于 DISTINCT 查询](#distinct-queries)

1.  [更快的 ORDER BY / DISTINCT 聚合](#faster-orderby-distinct)

1.  [UNION ALL 查询的记忆化](#union-all)

1.  [支持右反连接](#right-anti-join)

1.  [并行 Hash 全连接和右连接](#parallel-hashfull-rightjoin)

1.  [优化窗口函数框架子句](#frame-clauses)

1.  [优化各种窗口函数](#window-functions)

1.  [为分区表优化 JOIN 移除](#join-removals)

1.  [短路基本的 DISTINCT 查询](#short-circuit)

1.  [合并连接后的增量排序，在更多情况下](#merge-joins)

[增量排序](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commit;h=d2d8a229bc58a2014dce1c7a4fcdb6c5ab9fb8da) 首次在 PostgreSQL 13 中添加。这些增量排序减少了获取排序结果所需的工作量。如何实现？通过利用某些给定结果集已按一个或多个前导列排序的知识，仅对剩余列进行排序。

例如，如果在列 `a` 上有 btree 索引，并且我们需要按 `a`, `b` 排序行，则可以使用 btree 索引（提供列 `a` 上的预排序结果），并且仅在 `a` 的值更改时才对到目前为止看到的行进行排序。使用 PostgreSQL 的快速排序算法，对许多较小的组进行排序比对一个大组进行排序更有效率。

PostgreSQL 16 查询规划器现在考虑为 `SELECT DISTINCT` 查询执行增量排序。在 PG16 之前，选择排序方法用于 `SELECT DISTINCT` 查询时，规划器只考虑执行完整排序（比增量排序更昂贵）。

```
-- Setup
CREATE TABLE distinct_test (a INT, b INT);
INSERT INTO distinct_test
SELECT x,1 FROM generate_series(1,1000000)x;
CREATE INDEX on distinct_test(a);
VACUUM ANALYZE distinct_test;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT DISTINCT a,b FROM distinct_test; 
```

### PG15 EXPLAIN 输出

```
 QUERY PLAN
---------------------------------------------------------------
 HashAggregate (actual rows=1000000 loops=1)
   Group Key: a, b
   Batches: 81  Memory Usage: 11153kB  Disk Usage: 31288kB
   ->  Seq Scan on distinct_test (actual rows=1000000 loops=1)
 Planning Time: 0.065 ms
 Execution Time: 414.226 ms
(6 rows) 
```

### PG16 EXPLAIN 输出

```
 QUERY PLAN
------------------------------------------------------------------
 Unique (actual rows=1000000 loops=1)
   ->  Incremental Sort (actual rows=1000000 loops=1)
         Sort Key: a, b
         Presorted Key: a
         Full-sort Groups: 31250  Sort Method: quicksort  Average Memory: 26kB  Peak Memory: 26kB
         ->  Index Scan using distinct_test_a_idx on distinct_test (actual rows=1000000 loops=1)
 Planning Time: 0.108 ms
 Execution Time: 263.167 ms
(8 rows) 
```

在上述 PostgreSQL 16 的 `EXPLAIN` 输出中，您可以看到规划器选择使用 `a` 列上的 `distinct_test_a_idx` 索引，然后执行 `Incremental Sort` 来按 `b` 的所有相等值对 `a` 进行排序。`Presorted Key: a` 表示这一点。因为上面的 `INSERT` 语句仅为每个 `a` 值添加了单个 `b` 值，所以每个增量排序排序的元组批次仅包含单行。

上述 PostgreSQL 16 的 `EXPLAIN` 输出显示，`Incremental Sort` 的 `Peak Memory` 仅为 26 千字节，而 PostgreSQL 15 使用的哈希方法需要更多内存，以至于需要将约 30 兆字节的数据溢出到磁盘。**在 PostgreSQL 16 上执行的查询速度提高了 63%**。

在 PostgreSQL 15 及更早版本中，包含 `ORDER BY` 或 `DISTINCT` 子句的聚合函数会导致执行器始终在计划的 `Aggregate` 节点内部执行排序。由于总是执行排序，规划器从不尝试形成一个计划，以提供按顺序预排序的输入来对行进行聚合。

PostgreSQL 16 查询规划器现在试图形成一个计划，以正确顺序将行提供给计划的 `Aggregate` 节点。执行器现在足够聪明，能够识别这一点，并在行已经按正确顺序预排序时放弃执行排序本身。

```
-- Setup
CREATE TABLE aggtest (a INT, b text);
INSERT INTO aggtest SELECT a,md5((b%100)::text) FROM generate_series(1,10) a, generate_series(1,100000)b;
CREATE INDEX ON aggtest(a,b);
VACUUM FREEZE ANALYZE aggtest;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF, BUFFERS)
SELECT a,COUNT(DISTINCT b) FROM aggtest GROUP BY a; 
```

### PG15 EXPLAIN 输出

```
 QUERY PLAN
---------------------------------------------------------------
 GroupAggregate (actual rows=10 loops=1)
   Group Key: a
   Buffers: shared hit=892, temp read=4540 written=4560
   ->  Index Only Scan using aggtest_a_b_idx on aggtest (actual rows=1000000 loops=1)
         Heap Fetches: 0
         Buffers: shared hit=892
 Planning Time: 0.122 ms
 Execution Time: 302.693 ms
(8 rows) 
```

### PG16 EXPLAIN 输出

```
 QUERY PLAN
---------------------------------------------------------------
 GroupAggregate (actual rows=10 loops=1)
   Group Key: a
   Buffers: shared hit=892
   ->  Index Only Scan using aggtest_a_b_idx on aggtest (actual rows=1000000 loops=1)
         Heap Fetches: 0
         Buffers: shared hit=892
 Planning Time: 0.061 ms
 Execution Time: 115.534 ms
(8 rows) 
```

除了在 PostgreSQL 16 中执行的查询速度比 PG15 快两倍之外，在上述的 `EXPLAIN ANALYZE` 输出中，这种变化的唯一指示是在 PostgreSQL 16 输出中不存在的 `temp read=4540 written=4560`。在 PG15 中，这是隐式排序溢出到磁盘引起的。

`Memoize` 计划节点首次在 PostgreSQL 14 中引入。`Memoize` 计划节点充当参数化 `Nested Loop` 和 Nested Loop 内部之间的缓存层。当多次查找相同值时，Memoize 可以显著提高性能，因为当所需的行已经被查询并缓存时，它可以跳过执行其子节点。

PostgreSQL 16 查询规划器现在会考虑在参数化的 `Nested Loop` 的内部出现 `UNION ALL` 查询时使用 `Memoize`。

```
-- Setup
CREATE TABLE t1 (a INT PRIMARY KEY);
CREATE TABLE t2 (a INT PRIMARY KEY);
CREATE TABLE lookup (a INT);

INSERT INTO t1 SELECT x FROM generate_Series(1,10000) x;
INSERT INTO t2 SELECT x FROM generate_Series(1,10000) x;
INSERT INTO lookup SELECT x%10+1 FROM generate_Series(1,1000000)x;

ANALYZE t1,t2,lookup;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT * FROM (SELECT * FROM t1 UNION ALL SELECT * FROM t2) t
INNER JOIN lookup l ON l.a = t.a; 
```

### PG15 EXPLAIN 输出

```
 QUERY PLAN
-------------------------------------------------------------------------------
 Nested Loop (actual rows=2000000 loops=1)
   ->  Seq Scan on lookup l (actual rows=1000000 loops=1)
   ->  Append (actual rows=2 loops=1000000)
         ->  Index Only Scan using t1_pkey on t1 (actual rows=1 loops=1000000)
               Index Cond: (a = l.a)
               Heap Fetches: 1000000
         ->  Index Only Scan using t2_pkey on t2 (actual rows=1 loops=1000000)
               Index Cond: (a = l.a)
               Heap Fetches: 1000000
 Planning Time: 0.223 ms
 Execution Time: 1926.151 ms
(11 rows) 
```

### PG16 EXPLAIN 输出

```
 QUERY PLAN
---------------------------------------------------------------------------------
 Nested Loop (actual rows=2000000 loops=1)
   ->  Seq Scan on lookup l (actual rows=1000000 loops=1)
   ->  Memoize (actual rows=2 loops=1000000)
         Cache Key: l.a
         Cache Mode: logical
         Hits: 999990  Misses: 10  Evictions: 0  Overflows: 0  Memory Usage: 2kB
         ->  Append (actual rows=2 loops=10)
               ->  Index Only Scan using t1_pkey on t1 (actual rows=1 loops=10)
                     Index Cond: (a = l.a)
                     Heap Fetches: 10
               ->  Index Only Scan using t2_pkey on t2 (actual rows=1 loops=10)
                     Index Cond: (a = l.a)
                     Heap Fetches: 10
 Planning Time: 0.229 ms
 Execution Time: 282.120 ms
(15 rows) 
```

在上面的 PostgreSQL 16 EXPLAIN 输出中，可以看到 `Memoize` 节点置于 `Append` 节点之上，导致 `Append` 中的 `循环次数` 从 PG15 的 100 万次减少到 PG16 的 10 次。每当 `Memoize` 节点命中缓存时，就无需执行 `Append` 来获取记录。这导致 **在 PostgreSQL 16 上查询运行速度大约快了 6 倍**。

在执行 `INNER JOIN` 的 `Hash Join` 时，PostgreSQL 更倾向于在两个表中较小的那个上构建哈希表。较小的哈希表更好，因为构建它们的工作量更小。较小的表也更适合 CPU 的缓存，CPU 等待主内存中的数据到达的可能性更小。

在 PostgreSQL 16 之前的版本中，如果在查询中使用 `NOT EXISTS`，`Anti Join` 会始终将 `NOT EXISTS` 部分提到连接的内侧。这意味着无法灵活地对两个表中较小的那个进行哈希，可能需要在较大的表上构建哈希表。

PostgreSQL 16 查询规划器现在可以选择对较小的两个表进行哈希。这可以在 PostgreSQL 16 支持 `Right Anti Join` 的情况下完成。

```
-- Setup
CREATE TABLE small(a int);
CREATE TABLE large(a int);
INSERT INTO small
SELECT a FROM generate_series(1,100) a;
INSERT INTO large
SELECT a FROM generate_series(1,1000000) a;
VACUUM ANALYZE small,large;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT * FROM small s
WHERE NOT EXISTS(SELECT 1 FROM large l WHERE s.a = l.a); 
```

### PG15 解释输出

```
 QUERY PLAN
---------------------------------------------------------------
 Hash Anti Join (actual rows=0 loops=1)
   Hash Cond: (s.a = l.a)
   ->  Seq Scan on small s (actual rows=100 loops=1)
   ->  Hash (actual rows=1000000 loops=1)
         Buckets: 262144  Batches: 8  Memory Usage: 6446kB
         ->  Seq Scan on large l (actual rows=1000000 loops=1)
 Planning Time: 0.103 ms
 Execution Time: 139.023 ms
(8 rows) 
```

### PG16 解释输出

```
 QUERY PLAN
-----------------------------------------------------------
 Hash Right Anti Join (actual rows=0 loops=1)
   Hash Cond: (l.a = s.a)
   ->  Seq Scan on large l (actual rows=1000000 loops=1)
   ->  Hash (actual rows=100 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 12kB
         ->  Seq Scan on small s (actual rows=100 loops=1)
 Planning Time: 0.094 ms
 Execution Time: 77.076 ms
(8 rows) 
```

从上面的 `EXPLAIN ANALYZE` 输出可以看出，由于 PG16 的规划器选择使用 `Hash Right Anti Join`，PostgreSQL 16 的 `内存使用量` 明显少于 PostgreSQL 15，并且 `执行时间` 几乎减半。

PostgreSQL 11 引入了 `并行哈希连接`。这允许并行查询中的多个并行工作者协助构建单个哈希表。在 11 版本之前，每个工作者都将构建自己的相同哈希表，导致额外的内存开销。

在 PostgreSQL 16 中，`Parallel Hash Join` 已经改进，现在支持 `FULL` 和 `RIGHT` 连接类型。这允许并行执行具有 `FULL OUTER JOIN` 的查询，并且还允许执行 `Right Joins` 计划并行化。

```
-- Setup
CREATE TABLE odd (a INT);
CREATE TABLE even (a INT);
INSERT INTO odd
SELECT a FROM generate_series(1,1000000,2) a;
INSERT INTO even
SELECT a FROM generate_series(2,1000000,2) a;
VACUUM ANALYZE odd, even;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT COUNT(o.a),COUNT(e.a) FROM odd o FULL JOIN even e ON o.a = e.a; 
```

### PG15 解释输出

```
 QUERY PLAN
-------------------------------------------------------------------
 Aggregate (actual rows=1 loops=1)
   ->  Hash Full Join (actual rows=1000000 loops=1)
         Hash Cond: (o.a = e.a)
         ->  Seq Scan on odd o (actual rows=500000 loops=1)
         ->  Hash (actual rows=500000 loops=1)
               Buckets: 262144  Batches: 4  Memory Usage: 6439kB
               ->  Seq Scan on even e (actual rows=500000 loops=1)
 Planning Time: 0.079 ms
 Execution Time: 220.677 ms
(9 rows) 
```

### PG16 解释输出

```
 QUERY PLAN
--------------------------------------------------------------------------------
 Finalize Aggregate (actual rows=1 loops=1)
   ->  Gather (actual rows=2 loops=1)
         Workers Planned: 1
         Workers Launched: 1
         ->  Partial Aggregate (actual rows=1 loops=2)
               ->  Parallel Hash Full Join (actual rows=500000 loops=2)
                     Hash Cond: (o.a = e.a)
                     ->  Parallel Seq Scan on odd o (actual rows=250000 loops=2)
                     ->  Parallel Hash (actual rows=250000 loops=2)
                           Buckets: 262144  Batches: 4  Memory Usage: 6976kB
                           ->  Parallel Seq Scan on even e (actual rows=250000 loops=2)
 Planning Time: 0.161 ms
 Execution Time: 129.769 ms
(13 rows) 
```

`EXPLAIN` 输出显示，PostgreSQL 16 能够并行执行连接操作，这显著减少了查询的 `执行时间`。

当查询包含诸如 `row_number()`、`rank()`、`dense_rank()`、`percent_rank()`、`cume_dist()` 和 `ntile()` 等窗口函数时，如果窗口子句没有指定 `ROWS` 选项，则 PostgreSQL 会始终使用默认的 `RANGE` 选项。`RANGE` 选项导致执行器向前查找，直到找到第一个“非对等”行。对等行是根据窗口子句的 `ORDER BY` 子句相等的窗口帧中的行。当没有 `ORDER BY` 子句时，窗口帧中的所有行都是对等行。在处理具有按窗口子句的 `ORDER BY` 子句等方式排序的许多行的记录时，用于识别这些对等行的额外处理可能会很昂贵。

上述提到的窗口函数在查询的窗口子句中指定 `ROWS` 或 `RANGE` 时不会有任何不同的行为。然而，在 PostgreSQL 16 之前的版本中，执行器并不知道这一点，因为**一些**窗口函数确实关心 `ROWS`/`RANGE` 选项，所以执行器必须在所有情况下执行对等行的检查。

PostgreSQL 16 查询规划器知道哪些窗口函数关心 `ROWS`/`RANGE` 选项，并将此信息传递给执行器，以便它可以跳过不必要的额外处理。

此优化在查询中使用 `row_number()` 来限制结果数量时特别有效，如下面的示例所示。

```
-- Setup
CREATE TABLE scores (id INT PRIMARY KEY, score INT);
INSERT INTO scores SELECT s,random()*10 FROM generate_series(1,1000000)s;
CREATE INDEX ON scores(score);
VACUUM ANALYZE scores;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT * FROM (
    SELECT id,ROW_NUMBER() OVER (ORDER BY score) rn,score
    FROM scores
) m WHERE rn <= 10; 
```

### PG15 的 EXPLAIN 输出

```
 QUERY PLAN
-------------------------------------------------------------------------------
 WindowAgg (actual rows=10 loops=1)
   Run Condition: (row_number() OVER (?) <= 10)
   ->  Index Scan using scores_score_idx on scores (actual rows=50410 loops=1)
 Planning Time: 0.096 ms
 Execution Time: 29.775 ms
(5 rows) 
```

### PG16 的 EXPLAIN 输出

```
 QUERY PLAN
----------------------------------------------------------------------------
 WindowAgg (actual rows=10 loops=1)
   Run Condition: (row_number() OVER (?) <= 10)
   ->  Index Scan using scores_score_idx on scores (actual rows=11 loops=1)
 Planning Time: 0.191 ms
 Execution Time: 0.058 ms
(5 rows) 
```

PG15 的 EXPLAIN 输出中的 `Index Scan` 节点显示，在执行之前需要从 `scores_score_idx` 索引中读取 50410 行。而在 PostgreSQL 16 中，只需读取 11 行，因为执行器意识到一旦 `row_number` 达到 11，就不会再有符合 `<= 10` 条件的行了。这和执行器使用 `ROWS` 窗口子句选项导致该查询在 PostgreSQL 16 上运行速度提高了 500 多倍。

这一变更是对 PostgreSQL 15 中的工作的扩展。在 PG15 中，查询规划器被修改，允许执行器在窗口函数在 `WHERE` 子句中过滤的条件一旦变为 false 时，可以提前停止处理 `WindowAgg` 执行节点。

`row_number()` 是一个函数的例子，它可以提供这样的保证，因为它是一个单调递增的函数，即同一分区中的后续行的 `row_number` 永远不会低于前一行。

PostgreSQL 16 查询规划器扩展了此优化的覆盖范围，还涵盖了 `ntile()`、`cume_dist()` 和 `percent_rank()`。在 PostgreSQL 15 中，这仅对 `row_number()`、`rank()`、`dense_rank()`、`count()` 和 `count(*)` 有效。

```
-- Setup
CREATE TABLE marathon (id INT PRIMARY KEY, time INTERVAL NOT NULL);
INSERT INTO marathon
SELECT id,'03:00:00'::interval + (CAST(RANDOM() * 3600 AS INT) || 'secs')::INTERVAL - (CAST(RANDOM() * 3600 AS INT) || ' secs')::INTERVAL
FROM generate_series(1,50000) id;
CREATE INDEX ON marathon (time);
VACUUM ANALYZE marathon;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT * FROM (SELECT *,percent_rank() OVER (ORDER BY time) pr
FROM marathon) m WHERE pr <= 0.01; 
```

### PG15 的 EXPLAIN 输出

```
 QUERY PLAN
-----------------------------------------------------------------------
 Subquery Scan on m (actual rows=500 loops=1)
   Filter: (m.pr <= '0.01'::double precision)
   Rows Removed by Filter: 49500
   ->  WindowAgg (actual rows=50000 loops=1)
         ->  Index Scan using marathon_time_idx on marathon (actual rows=50000 loops=1)
 Planning Time: 0.108 ms
 Execution Time: 84.358 ms
(7 rows) 
```

### PG16 的 EXPLAIN 输出

```
 QUERY PLAN
-----------------------------------------------------------------------
 WindowAgg (actual rows=500 loops=1)
   Run Condition: (percent_rank() OVER (?) <= '0.01'::double precision)
   ->  Index Scan using marathon_time_idx on marathon (actual rows=50000 loops=1)
 Planning Time: 0.180 ms
 Execution Time: 19.454 ms
(5 rows) 
```

根据上面的 PostgreSQL 16 EXPLAIN 输出，您可以看到规划器能够将 `pr <= 0.01` 条件作为 `Run Condition` 使用，而在 PostgreSQL 15 中，此子查询中的条件作为 `Filter` 出现。在 PG16 中，运行条件用于提前中止 `WindowAgg` 节点的执行。这导致 PG16 的执行时间比 PG15 快了超过 4 倍。

长期以来，PostgreSQL 已能够移除 `LEFT JOIN`，其中左连接表的任何列都不在查询中需要，并且连接不可能复制任何行。

然而，在 PostgreSQL 16 之前的版本中，对分区表的左连接移除没有支持。为什么？因为规划器用于确定是否存在任何可能使内侧行重复任何外侧行的证据在分区表中不存在。

PostgreSQL 16 查询规划器现在允许对分区表进行 `LEFT JOIN` 移除优化。

此连接消除优化更有可能帮助视图，因为视图中存在的所有列并不总是会被查询。

```
-- Setup
CREATE TABLE part_tab (id BIGINT PRIMARY KEY, payload TEXT) PARTITION BY HASH(id);
CREATE TABLE part_tab_p0 PARTITION OF part_tab FOR VALUES WITH (modulus 2, remainder 0);
CREATE TABLE part_tab_p1 PARTITION OF part_tab FOR VALUES WITH (modulus 2, remainder 1);
CREATE TABLE normal_table (id INT, part_tab_id BIGINT);

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT nt.* FROM normal_table nt LEFT JOIN part_tab pt ON nt.part_tab_id = pt.id; 
```

### PG15 的 EXPLAIN 输出

```
 QUERY PLAN
-------------------------------------------------------------------
 Merge Right Join (actual rows=0 loops=1)
   Merge Cond: (pt.id = nt.part_tab_id)
   ->  Merge Append (actual rows=0 loops=1)
         Sort Key: pt.id
         ->  Index Only Scan using part_tab_p0_pkey on part_tab_p0 pt_1 (actual rows=0 loops=1)
               Heap Fetches: 0
         ->  Index Only Scan using part_tab_p1_pkey on part_tab_p1 pt_2 (actual rows=0 loops=1)
               Heap Fetches: 0
   ->  Sort (actual rows=0 loops=1)
         Sort Key: nt.part_tab_id
         Sort Method: quicksort  Memory: 25kB
         ->  Seq Scan on normal_table nt (actual rows=0 loops=1)
 Planning Time: 0.325 ms
 Execution Time: 0.037 ms
(14 rows) 
```

### PG16 的 EXPLAIN 输出

```
 QUERY PLAN
-----------------------------------------------------
 Seq Scan on normal_table nt (actual rows=0 loops=1)
 Planning Time: 0.244 ms
 Execution Time: 0.015 ms
(3 rows) 
```

这里需要注意的重要一点是，PostgreSQL 16 计划不包括与 `part_tab` 的连接，这意味着只需扫描 `normal_table`。

PostgreSQL 查询规划器能够在能够检测到所有行包含相同值时避免包含计划节点以去重结果。检测这一点很简单，当优化可以应用时，可以获得巨大的性能提升。

```
-- Setup
CREATE TABLE abc (a int, b int, c int);
INSERT INTO abc SELECT a%10,a%10,a%10 FROM generate_series(1,1000000)a;
VACUUM ANALYZE abc;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT DISTINCT a,b,c FROM abc WHERE a = 5 AND b = 5 AND c = 5; 
```

### PG15 的 EXPLAIN 输出

```
 QUERY PLAN
------------------------------------------------------------------------
 Unique (actual rows=1 loops=1)
   ->  Gather (actual rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         ->  Unique (actual rows=1 loops=3)
               ->  Parallel Seq Scan on abc (actual rows=33333 loops=3)
                     Filter: ((a = 5) AND (b = 5) AND (c = 5))
                     Rows Removed by Filter: 300000
 Planning Time: 0.114 ms
 Execution Time: 30.381 ms
(10 rows) 
```

### PG16 的 EXPLAIN 输出

```
 QUERY PLAN
---------------------------------------------------
 Limit (actual rows=1 loops=1)
   ->  Seq Scan on abc (actual rows=1 loops=1)
         Filter: ((a = 5) AND (b = 5) AND (c = 5))
         Rows Removed by Filter: 4
 Planning Time: 0.109 ms
 Execution Time: 0.025 ms
(6 rows) 
```

如果仔细查看 SQL 查询，您会注意到 `DISTINCT` 子句中的每列在 `WHERE` 子句中也具有等式条件。这意味着查询中的所有输出行在每列中都具有相同的值。PostgreSQL 16 查询规划器能够利用这一知识简单地将查询结果限制为 1 行。PostgreSQL 15 通过读取整个结果并使用 `Unique` 运算符将所有行减少到单行来产生相同的查询结果。PostgreSQL 16 的执行时间比 PostgreSQL 15 快了超过 1200 倍。

在 PostgreSQL 16 之前，当查询规划器考虑执行 `Merge Join` 时，它会检查合并的排序顺序是否适合任何上层计划操作（如 `DISTINCT`、`GROUP BY` 或 `ORDER BY`），并且仅在它与上层要求完全匹配时才使用该顺序。这种选择有点过时，因为对于这些上层操作，可以使用增量排序，并且增量排序可以利用仅对一些前导列进行排序的结果。

PostgreSQL 16查询规划器调整了在考虑`Merge Join`顺序时使用的规则，从“行的顺序必须完全匹配”调整为“必须至少有1个前导列正确排序”。这使得规划器可以使用`Incremental Sorts`将行按正确顺序准备到上层操作。我们从本博客的早期部分知道，当可能时，增量排序比完全排序需要更少的工作，因为增量排序能够利用部分排序的输入并以较小的批次执行排序，从而导致更少的内存消耗和更少的排序比较。

```
-- Setup

CREATE TABLE a (a INT, b INT);
CREATE TABLE b (x INT, y INT);
INSERT INTO a SELECT a,a FROM generate_series(1,1000000) a;
INSERT INTO b SELECT a,a FROM generate_series(1,1000000) a;
VACUUM ANALYZE a, b;

SET enable_hashjoin=0;
SET max_parallel_workers_per_gather=0;
EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT a,b,count(*) FROM a INNER JOIN b ON a.a = b.x GROUP BY a,b ORDER BY a DESC, b; 
```

### PG15 EXPLAIN输出

```
 QUERY PLAN
---------------------------------------------------------------------------
 GroupAggregate (actual rows=1000000 loops=1)
   Group Key: a.a, a.b
   ->  Sort (actual rows=1000000 loops=1)
         Sort Key: a.a DESC, a.b
         Sort Method: external merge  Disk: 17664kB
         ->  Merge Join (actual rows=1000000 loops=1)
               Merge Cond: (a.a = b.x)
               ->  Sort (actual rows=1000000 loops=1)
                     Sort Key: a.a
                     Sort Method: external merge  Disk: 17664kB
                     ->  Seq Scan on a (actual rows=1000000 loops=1)
               ->  Materialize (actual rows=1000000 loops=1)
                     ->  Sort (actual rows=1000000 loops=1)
                           Sort Key: b.x
                           Sort Method: external merge  Disk: 11768kB
                           ->  Seq Scan on b (actual rows=1000000 loops=1)
 Planning Time: 0.175 ms
 Execution Time: 1010.738 ms
(18 rows) 
```

### PG16 EXPLAIN输出

```
 QUERY PLAN
---------------------------------------------------------------------------
 GroupAggregate (actual rows=1000000 loops=1)
   Group Key: a.a, a.b
   ->  Incremental Sort (actual rows=1000000 loops=1)
         Sort Key: a.a DESC, a.b
         Presorted Key: a.a
         Full-sort Groups: 31250  Sort Method: quicksort  Average Memory: 26kB  Peak Memory: 26kB
         ->  Merge Join (actual rows=1000000 loops=1)
               Merge Cond: (a.a = b.x)
               ->  Sort (actual rows=1000000 loops=1)
                     Sort Key: a.a DESC
                     Sort Method: external merge  Disk: 17672kB
                     ->  Seq Scan on a (actual rows=1000000 loops=1)
               ->  Materialize (actual rows=1000000 loops=1)
                     ->  Sort (actual rows=1000000 loops=1)
                           Sort Key: b.x DESC
                           Sort Method: external merge  Disk: 11768kB
                           ->  Seq Scan on b (actual rows=1000000 loops=1)
 Planning Time: 0.140 ms
 Execution Time: 915.589 ms
(19 rows) 
```

在上述的PG16 EXPLAIN输出中，您可以看到使用了`Incremental Sort`（与PG15使用的`Sort`相比），这导致了PG16中查询的`Execution Time`略微减少以及执行排序所使用的内存大幅减少。

## 结论

PostgreSQL 16进行了大量工程工作，以改进来自世界各地的许多工程师的查询规划器。我要感谢所有帮助审查我工作部分的人，以及对更改提供反馈的所有人。

上述对PostgreSQL 16规划器的10个改进都是默认启用的，并且在可能的情况下，会应用于所有优化，或者由查询规划器在认为优化有助于时进行选择性应用。

如果您正在运行较旧版本的PostgreSQL，我鼓励您尝试在PostgreSQL 16上运行您的工作负载，看看哪些查询更快。如往常一样，欢迎在[pgsql-general@postgresql.org](mailto:pgsql-general@postgresql.org?subject=PG16%20query%20planner%20feedback)邮件列表上分享有关PostgreSQL在真实世界中使用的反馈，您不仅可以提交问题，还可以分享积极的使用经验。所以，请告诉我们您使用PostgreSQL 16规划器的体验。

### 喜欢您正在阅读的内容吗？

如果您想阅读更多我们Citus数据库和Postgres团队的文章，请订阅我们的月度通讯，获取最新内容直接送到您的收件箱。
