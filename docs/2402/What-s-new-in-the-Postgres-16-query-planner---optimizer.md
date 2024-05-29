<!--yml
category: 未分类
date: 2024-05-27 14:44:18
-->

# What’s new in the Postgres 16 query planner / optimizer

> 来源：[https://www.citusdata.com/blog/2024/02/08/whats-new-in-postgres-16-query-planner-optimizer/](https://www.citusdata.com/blog/2024/02/08/whats-new-in-postgres-16-query-planner-optimizer/)

PostgreSQL 16 introduces quite a few improvements to the query planner and makes many SQL queries run faster than they did on previous versions of PostgreSQL.

If you look at the [PG16 release notes](https://www.postgresql.org/docs/16/release-16.html), you’ll see some of these planner improvements. But with the volume of changes made in each PostgreSQL release, it’s not possible to provide enough detail about each and every change. So maybe you might need a bit more detail to know what the change is about—before you understand if it’s relevant to you.

In this blog post, assuming you’ve already got a handle on the [basics of EXPLAIN](https://www.postgresql.org/docs/16/using-explain.html#USING-EXPLAIN-BASICS), you’ll get a deep dive into the 10 improvements made in the PostgreSQL 16 query planner. For each of the improvements to the PG16 planner (the planner is often called an optimizer in other relational databases), you’ll also get comparisons between PG15 and PG16 planner output—plus examples of what changed, in the form of a self-contained test you can try for yourself.

Let’s dive into these 10 improvements to the PostgreSQL planner in PG16:

1.  [Incremental sorts for DISTINCT queries](#distinct-queries)
2.  [Faster ORDER BY / DISTINCT aggregates](#faster-orderby-distinct)
3.  [Memoize for UNION ALL queries](#union-all)
4.  [Support Right Anti Join](#right-anti-join)
5.  [Parallel Hash Full and Right Joins](#parallel-hashfull-rightjoin)
6.  [Optimize window function frame clauses](#frame-clauses)
7.  [Optimize various window functions](#window-functions)
8.  [JOIN removals for partitioned tables](#join-removals)
9.  [Short circuit trivial DISTINCT queries](#short-circuit)
10.  [Incremental Sort after Merge Join, in more cases](#merge-joins)

[Incremental sorts](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commit;h=d2d8a229bc58a2014dce1c7a4fcdb6c5ab9fb8da) were first added in PostgreSQL 13\. These incremental sorts reduce the effort required to get sorted results. How? By exploiting the knowledge that some given result set is already sorted by 1 or more leading columns—and only performing a sort on the remaining columns.

For example, if there’s a btree index on column `a` and we need the rows ordered by `a`,`b`, then we can use the btree index (which provides presorted results on column `a`) and sort the rows seen so far only when the value of `a` changes. With the quicksort algorithm used by PostgreSQL, sorting many smaller groups is more efficient than sorting one large group.

The PostgreSQL 16 query planner now considers performing incremental sorts for `SELECT DISTINCT` queries. Prior to PG16, when the sorting method was chosen for `SELECT DISTINCT` queries, the planner only considered performing a full sort (which is more expensive than an incremental sort.)

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

In the PostgreSQL 16 `EXPLAIN` output above, you can see the planner chose to use the `distinct_test_a_idx` index on the `a` column and then performed an `Incremental Sort` to sort all of the equal values of `a` by `b`. The `Presorted Key: a` indicates this. Because the `INSERT` statements above only added a single value of `b` for each value of `a`, each batch of tuples sorted by incremental sort only contains a single row.

The `EXPLAIN` output for PostgreSQL 16 above shows that the `Peak Memory` for the `Incremental Sort` was just 26 kilobytes, whereas the hashing method used by PostgreSQL 15 needed much memory, so much so that it needed to spill about 30 megabytes of data to disk. **The query executed 63% faster on PostgreSQL 16**.

In PostgreSQL 15 and earlier, aggregate functions containing an `ORDER BY` or `DISTINCT` clause would result in the executor always performing a sort inside the `Aggregate` node of the plan. Because the sort was always performed, the planner would never try to form a plan to provide presorted input to aggregate the rows in order.

The PostgreSQL 16 query planner now tries to form a plan which feeds the rows to the plan’s `Aggregate` node in the correct order. And the executor is now smart enough to recognize this and forego performing the sort itself when the rows are already pre-sorted in the correct order.

```
-- Setup
CREATE TABLE aggtest (a INT, b text);
INSERT INTO aggtest SELECT a,md5((b%100)::text) FROM generate_series(1,10) a, generate_series(1,100000)b;
CREATE INDEX ON aggtest(a,b);
VACUUM FREEZE ANALYZE aggtest;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF, BUFFERS)
SELECT a,COUNT(DISTINCT b) FROM aggtest GROUP BY a; 
```

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

Aside from PostgreSQL 16 executing the query over twice as fast as in PG15, the only indication of this change in the `EXPLAIN ANALYZE` output above is from the `temp read=4540 written=4560` that’s not present in the PostgreSQL 16 output. In PG15, this is caused by the implicit sort spilling to disk.

`Memoize` plan nodes were first introduced in PostgreSQL 14\. The `Memoize` plan node acts as a cache layer between a parameterized `Nested Loop` and the Nested Loop’s inner side. When the same value needs to be looked up several times, Memoize can give a nice performance boost as it can skip executing its subnode when the required rows have been queried already and are cached.

The PostgreSQL 16 query planner will now consider using `Memoize` when a `UNION ALL` query appears on the inner side of a parameterized `Nested Loop`.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

In the PostgreSQL 16 EXPLAIN output above, you can see the `Memoize` node is put atop of the `Append` node—which caused a reduction in the number of `loops` in the `Append` from 1 million in PG15 down to 10 in PG16\. Each time the `Memoize` node has a cache hit, there’s no need to execute the `Append` to fetch records. This results in the **query running around 6 times faster on PostgreSQL 16**.

When performing a `Hash Join` for an `INNER JOIN`, PostgreSQL prefers to build the hash table on the smaller of the two tables. Smaller hash tables are better as it’s less work to build them. Smaller tables are also better as they’re more cache-friendly for the CPU, and it’s less likely that the CPU will stall while waiting for data to arrive from main memory.

In PostgreSQL versions before 16, an `Anti Join`—as you might see if you use `NOT EXISTS` in your queries—would always put the table mentioned in the `NOT EXISTS` part on the inner side of the join. This meant there was no flexibility to hash the smaller of the two tables, resulting in possibly having to build a hash table on the larger table.

The PostgreSQL 16 query planner can now choose to hash the smaller of the two tables. This can now be done because PostgreSQL 16 supports `Right Anti Join`.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

You can see from the `EXPLAIN ANALYZE` output above that due to PG16’s planner opting to use a `Hash Right Anti Join`, the `Memory Usage` in PostgreSQL 16 was much less than in PostgreSQL 15 and the `Execution Time` was almost halved.

PostgreSQL 11 saw the introduction of `Parallel Hash Join`. This allows multiple parallel workers in a parallel query to assist in the building of a single hash table. In versions prior to 11, each worker would have built its own identical hash table, resulting in additional memory overheads.

In PostgreSQL 16, `Parallel Hash Join` has been improved and now supports `FULL` and `RIGHT` join types. This allows queries that have a `FULL OUTER JOIN` to be executed in parallel and also allows `Right Joins` plans to execute in parallel.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

The `EXPLAIN` output shows that PostgreSQL 16 was able to perform the join in parallel and this resulted in a significant reduction in the query’s `Execution Time`.

When a query contains a window function such as `row_number()`, `rank()`, `dense_rank()`, `percent_rank()`, `cume_dist()` and `ntile()`, if the [window clause](https://www.postgresql.org/docs/16/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS) did not specify the `ROWS`option, then PostgreSQL would always use the default `RANGE` option. The `RANGE` option causes the executor to look ahead until it finds the first “non-peer” row. A peer row is a row in the window frame which compares equally according to the window clause’s `ORDER BY` clause. When there is no `ORDER BY` clause, all rows within the window frame are peers. When processing records which have many rows which sort equally according to the window clause’s `ORDER BY` clause, the additional processing to identify these peer rows could be costly.

The window functions mentioned above don’t behave any differently whether `ROWS` or `RANGE` is specified in the query’s window clause. However, the executor in PostgreSQL versions prior to 16 didn’t know that, and because **some** window functions do care about the `ROWS`/`RANGE` option, the executor had to perform checks for peer rows in all cases.

The PostgreSQL 16 query planner knows which window functions care about the `ROWS`/`RANGE` option and it passes this information along to the executor so that it can skip the needless additional processing.

This optimization works particularly well when `row_number()` is being used to limit the number of results in the query as shown in the example below.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

The `Index Scan` node in the `EXPLAIN` output for PG15 above shows that 50410 rows had to be read from the `scores_score_idx` index before execution stopped. While in PostgreSQL 16, only 11 rows were read as the executor realized that once the row_number got to 11, there’d be no more rows matching the `<= 10` condition. Both this and the executor using the `ROWS` window clause option led to this query running over 500 times faster on PostgreSQL 16.

This change expands on work done in PostgreSQL 15\. In PG15 the query planner was modified to allow the executor to stop processing `WindowAgg` executor nodes early. This can be done when an item in the `WHERE` clause filters a window function in a way that once the condition becomes false, it will never be true again.

`row_number()` is an example of a function which can offer such guarantees as it’s a monotonically increasing function, i.e. subsequent rows in the same partition will never have a row_number lower than the previous row.

The PostgreSQL 16 query planner expands the coverage of this optimization to also cover `ntile()`, `cume_dist()` and `percent_rank()`. In PostgreSQL 15, this only worked for `row_number()`, `rank()`, `dense_rank()`, `count()` and `count(*)`.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

From the PostgreSQL 16 `EXPLAIN` output above, you can see that the planner was able to use the `pr <= 0.01` condition as a `Run Condition`, whereas, with PostgreSQL 15 this clause appeared as a `Filter` on the subquery. In PG16, the run condition was used to abort the execution of the `WindowAgg` node early. This resulted in the `Execution Time` in PG16 being more than 4 times faster than in PG15.

For a long time now, PostgreSQL has been able to remove a `LEFT JOIN` where no column from the left joined table was required in the query and the join could not possibly duplicate any rows.

However, in versions prior to PostgreSQL 16, there was no support for left join removals on partitioned tables. Why? Because the proofs that the planner uses to determine if there’s any possibility any inner-side row could duplicate any outer-side row were not present for partitioned tables.

The PostgreSQL 16 query planner now allows the `LEFT JOIN` removal optimization with partitioned tables.

This join elimination optimization is more likely to help with views, as it’s common that not all columns that exist in a view will always be queried.

```
-- Setup
CREATE TABLE part_tab (id BIGINT PRIMARY KEY, payload TEXT) PARTITION BY HASH(id);
CREATE TABLE part_tab_p0 PARTITION OF part_tab FOR VALUES WITH (modulus 2, remainder 0);
CREATE TABLE part_tab_p1 PARTITION OF part_tab FOR VALUES WITH (modulus 2, remainder 1);
CREATE TABLE normal_table (id INT, part_tab_id BIGINT);

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT nt.* FROM normal_table nt LEFT JOIN part_tab pt ON nt.part_tab_id = pt.id; 
```

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

```
 QUERY PLAN
-----------------------------------------------------
 Seq Scan on normal_table nt (actual rows=0 loops=1)
 Planning Time: 0.244 ms
 Execution Time: 0.015 ms
(3 rows) 
```

The important thing to notice here is that the PostgreSQL 16 plan does not include a join to `part_tab` meaning all there is to do is scan `normal_table`.

The PostgreSQL query planner is able to avoid including plan nodes to de-duplicate the results when it can detect that all rows contain the same value. Detecting this is trivial and when the optimization can be applied it can result in huge performance gains.

```
-- Setup
CREATE TABLE abc (a int, b int, c int);
INSERT INTO abc SELECT a%10,a%10,a%10 FROM generate_series(1,1000000)a;
VACUUM ANALYZE abc;

EXPLAIN (ANALYZE, COSTS OFF, TIMING OFF)
SELECT DISTINCT a,b,c FROM abc WHERE a = 5 AND b = 5 AND c = 5; 
```

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

If you look carefully at the SQL query, you’ll notice that each column in the `DISTINCT` clause also has an equality condition in the `WHERE` clause. This means that all the output rows in the query will have the same values in each column. The PostgreSQL 16 query planner is able to take advantage of this knowledge and simply `LIMIT` the query results to 1 row. PostgreSQL 15 produced the same query result by reading the entire results and using the `Unique` operator to reduce all the rows down to a single row. The `Execution Time` for PostgreSQL 16 was more than 1200 times faster than PostgreSQL 15.

Before PostgreSQL 16, when the query planner considered performing a `Merge Join`, it would check if the sort order of the merge would suit any upper-level plan operation (such as `DISTINCT`, `GROUP BY` or `ORDER BY`) and only use that order if it matched the requirements for the upper-level exactly. This choice was a little outdated as `Incremental Sorts` can be used for these upper-level operations and incremental sorts can take advantage of results that are presorted by only some of the leading columns that the results need to be sorted by.

The PostgreSQL 16 query planner adjusted the rule used when considering `Merge Join` orders from “the order of the rows must match exactly” to “there must be at least 1 leading column correctly ordered”. This allows the planner to use `Incremental Sorts` to get the rows into the correct order for the upper-level operation. We know from earlier in this blog that incremental sorts, when they’re possible, require less work than a complete sort as incremental sorts are able to exploit partially sorted inputs and perform sorts in smaller batches, resulting in less memory consumption and fewer sort comparisons overall.

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

### PG15 EXPLAIN output

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

### PG16 EXPLAIN output

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

In the PG16 EXPLAIN output above, you can see that an `Incremental Sort` was used (compared to PG15 which instead used a `Sort`) and that resulted in a small reduction of the query’s `Execution Time` in PG16 and a large reduction in the memory used to perform the sort.

## Conclusion

A lot of engineering work was done in PostgreSQL 16 to improve the query planner by many engineers from all around the world. I’d like to thank all the people who helped by reviewing the pieces that I worked on and all the people who gave feedback on the changes.

Each of the 10 improvements to the PostgreSQL 16 planner above are enabled by default—and are either applied in all cases where the optimization is possible, or are selectively applied by the query planner when it thinks the optimization will help.

If you’re running an older version of PostgreSQL, I encourage you to try your workload on PostgreSQL 16 to see which of your queries are faster. And as always, feedback about real-world usage of PostgreSQL in the wild is welcome on the [pgsql-general@postgresql.org](mailto:pgsql-general@postgresql.org?subject=PG16%20query%20planner%20feedback) mailing list—you don’t have to just file issues, you can always share the positive experiences too. So please drop us a line about your experience with the PostgreSQL 16 planner.

### Enjoy what you’re reading?

If you want to read more posts from our Citus database and Postgres teams, sign up for our monthly newsletter and get the latest content delivered straight to your inbox.