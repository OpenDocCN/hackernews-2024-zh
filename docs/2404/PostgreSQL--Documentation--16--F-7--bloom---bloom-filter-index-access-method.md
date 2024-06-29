<!--yml

category: 未分类

date: 2024-05-27 13:28:02

-->

# PostgreSQL：文档：16：F.7. bloom — 布隆过滤器索引访问方法

> 来源：[https://www.postgresql.org/docs/current/bloom.html](https://www.postgresql.org/docs/current/bloom.html)

布隆过滤器是一种空间高效的数据结构，用于测试元素是否属于集合的成员。在索引访问方法的情况下，它通过签名快速排除非匹配元组，签名的大小在索引创建时确定。

签名是索引属性的有损表示，因此容易报告假阳性；即当不在集合中时，可能会报告某个元素在集合中。因此，必须始终使用堆条目的实际属性值重新检查索引搜索结果。较大的签名降低假阳性的可能性，从而减少无用的堆访问次数，但这当然也会使索引更大，因此扫描速度更慢。

当表具有许多属性且查询测试它们的任意组合时，这种类型的索引最有用。传统的 btree 索引比布隆索引更快，但是为了支持所有可能的查询，可能需要多个 btree 索引，而只需要一个布隆索引。然而请注意，布隆索引仅支持等式查询，而 btree 索引还可以执行不等式和范围搜索。

这是创建布隆索引的示例：

```
CREATE INDEX bloomidx ON tbloom USING bloom (i1,i2,i3)
       WITH (length=80, col1=2, col2=2, col3=4);

```

索引使用签名长度为80位，属性 i1 和 i2 映射到2位，属性 i3 映射到4位。我们本可以省略`length`、`col1`和`col2`的规范，因为它们具有默认值。

这里有一个更完整的布隆索引定义和使用示例，以及与等效的 btree 索引的比较。布隆索引比 btree 索引要小得多，并且性能更好。

```
=# CREATE TABLE tbloom AS
   SELECT
     (random() * 1000000)::int as i1,
     (random() * 1000000)::int as i2,
     (random() * 1000000)::int as i3,
     (random() * 1000000)::int as i4,
     (random() * 1000000)::int as i5,
     (random() * 1000000)::int as i6
   FROM
  generate_series(1,10000000);
SELECT 10000000

```

在这个大表上的顺序扫描需要很长时间：

```
=# EXPLAIN ANALYZE SELECT * FROM tbloom WHERE i2 = 898732 AND i5 = 123451;
                                              QUERY PLAN
-------------------------------------------------------------------​-----------------------------------
 Seq Scan on tbloom  (cost=0.00..2137.14 rows=3 width=24) (actual time=16.971..16.971 rows=0 loops=1)
   Filter: ((i2 = 898732) AND (i5 = 123451))
   Rows Removed by Filter: 100000
 Planning Time: 0.346 ms
 Execution Time: 16.988 ms
(5 rows)

```

即使定义了 btree 索引，结果仍然是顺序扫描：

```
=# CREATE INDEX btreeidx ON tbloom (i1, i2, i3, i4, i5, i6);
CREATE INDEX
=# SELECT pg_size_pretty(pg_relation_size('btreeidx'));
 pg_size_pretty
----------------
 3976 kB
(1 row)
=# EXPLAIN ANALYZE SELECT * FROM tbloom WHERE i2 = 898732 AND i5 = 123451;
                                              QUERY PLAN
-------------------------------------------------------------------​-----------------------------------
 Seq Scan on tbloom  (cost=0.00..2137.00 rows=2 width=24) (actual time=12.805..12.805 rows=0 loops=1)
   Filter: ((i2 = 898732) AND (i5 = 123451))
   Rows Removed by Filter: 100000
 Planning Time: 0.138 ms
 Execution Time: 12.817 ms
(5 rows)

```

在表上定义布隆索引比 btree 更好地处理这种类型的搜索：

```
=# CREATE INDEX bloomidx ON tbloom USING bloom (i1, i2, i3, i4, i5, i6);
CREATE INDEX
=# SELECT pg_size_pretty(pg_relation_size('bloomidx'));
 pg_size_pretty
----------------
 1584 kB
(1 row)
=# EXPLAIN ANALYZE SELECT * FROM tbloom WHERE i2 = 898732 AND i5 = 123451;
                                                     QUERY PLAN
-------------------------------------------------------------------​--------------------------------------------------
 Bitmap Heap Scan on tbloom  (cost=1792.00..1799.69 rows=2 width=24) (actual time=0.388..0.388 rows=0 loops=1)
   Recheck Cond: ((i2 = 898732) AND (i5 = 123451))
   Rows Removed by Index Recheck: 29
   Heap Blocks: exact=28
   ->  Bitmap Index Scan on bloomidx  (cost=0.00..1792.00 rows=2 width=0) (actual time=0.356..0.356 rows=29 loops=1)
         Index Cond: ((i2 = 898732) AND (i5 = 123451))
 Planning Time: 0.099 ms
 Execution Time: 0.408 ms
(8 rows)

```

现在，btree 搜索的主要问题是，当搜索条件不限制前导索引列时，btree 效率低下。对 btree 的更好策略是在每列上创建单独的索引。然后计划程序将选择类似于这样的内容：

```
=# CREATE INDEX btreeidx1 ON tbloom (i1);
CREATE INDEX
=# CREATE INDEX btreeidx2 ON tbloom (i2);
CREATE INDEX
=# CREATE INDEX btreeidx3 ON tbloom (i3);
CREATE INDEX
=# CREATE INDEX btreeidx4 ON tbloom (i4);
CREATE INDEX
=# CREATE INDEX btreeidx5 ON tbloom (i5);
CREATE INDEX
=# CREATE INDEX btreeidx6 ON tbloom (i6);
CREATE INDEX
=# EXPLAIN ANALYZE SELECT * FROM tbloom WHERE i2 = 898732 AND i5 = 123451;
                                                        QUERY PLAN
-------------------------------------------------------------------​--------------------------------------------------------
 Bitmap Heap Scan on tbloom  (cost=24.34..32.03 rows=2 width=24) (actual time=0.028..0.029 rows=0 loops=1)
   Recheck Cond: ((i5 = 123451) AND (i2 = 898732))
   ->  BitmapAnd  (cost=24.34..24.34 rows=2 width=0) (actual time=0.027..0.027 rows=0 loops=1)
         ->  Bitmap Index Scan on btreeidx5  (cost=0.00..12.04 rows=500 width=0) (actual time=0.026..0.026 rows=0 loops=1)
               Index Cond: (i5 = 123451)
         ->  Bitmap Index Scan on btreeidx2  (cost=0.00..12.04 rows=500 width=0) (never executed)
               Index Cond: (i2 = 898732)
 Planning Time: 0.491 ms
 Execution Time: 0.055 ms
(9 rows)

```

尽管这个查询运行速度比任何单个索引都要快得多，但我们在索引大小上付出了代价。每个单列 btree 索引占用2 MB，因此所需的总空间为12 MB，比布隆索引的空间大8倍。
