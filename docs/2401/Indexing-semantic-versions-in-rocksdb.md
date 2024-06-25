<!--yml

分类：未分类

日期：2024-05-27 14:31:42

-->

# 在rocksdb中索引语义版本

> 来源：[https://blog.aawadia.dev/2024/01/02/index-semver-rocksdb/](https://blog.aawadia.dev/2024/01/02/index-semver-rocksdb/)

## 介绍

最近出现的一个需求是需要为一堆记录索引一个属性，该属性的类型是语义版本。该属性包含一个`major.minor.patch`类型的值，表示某个外部事物的版本。要求是能够进行过滤版本`>`或`<`某个特定版本的行。

这个功能需求可以简化为技术定义，即为记录的语义版本属性提供一个允许进行范围查询和点查找（高效）的索引。

那么，如何创建索引呢？

## 基本理念

当需要在数据片段上创建索引时，第一种方法始终是将键保存在一个哈希映射中，其中键是排序的。这里的技术原始数据结构是一个哈希映射，其中键是排序的。为什么要排序？因为排序允许基于`lg(n)`的范围查询。这就是我们得到点查找和范围查询的方式。

那么，我怎么创建一个键排序的哈希映射？

## 实施

键排序的哈希映射称为`SortedMap`接口，其中一些实现是`TreeMap`和`ConcurrentSkipListMap`。

## 把它带回来

因此，将所有这些重新提升到需求级别 - 我可以创建一个语义版本的排序映射，指回记录的主键，然后通过查询层将其暴露给世界。

## Rocksdb

Rocksdb是一个可嵌入的kv存储，是一个持久化到磁盘的排序映射。我们可以利用这个功能，让rocksdb负责保持键排序。

然而，这里有一个需要注意的地方 - 让我们看一个代码示例，看看在插入语义版本然后再打印出来时会发生什么。

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26

```

|

```
fun main() {
 val emptyByte = byteArrayOf()
 RocksDB.loadLibrary()
 val options = Options().setCreateIfMissing(true)
 val rdb = RocksDB.open(options, "/tmp/rdb-semver")
 rdb.put("1.2.4".encodeToByteArray(), emptyByte)
 rdb.put("10.2.4".encodeToByteArray(), emptyByte)
 rdb.put("2.2.5".encodeToByteArray(), emptyByte)
 rdb.put("1.3.4".encodeToByteArray(), emptyByte)
 rdb.put("1.2.4-rc.1".encodeToByteArray(), emptyByte)
 rdb.put("2.2.4-beta-2".encodeToByteArray(), emptyByte)
 rdb.put("2.2.4-alpha-2".encodeToByteArray(), emptyByte)
 printDb(rdb)
 rdb.close()
}
 === DB start ===
1.2.4
1.2.4-rc.1
1.3.4
10.2.4
2.2.4-alpha-2
2.2.4-beta-2
2.2.5
=== DB end === 
```

|

排序完全乱了。`10.2.4`应该是最后一个，而`1.2.4-rc.1`应该在`1.2.4`之前。

这是因为rocksdb实际上不知道我正在将语义版本字符串放入键中 - 这没有意义 - 它只是一个字节数组，因此被按字典顺序排序。这也发生在linux上，当执行`ls`列出文件夹时，名称10在2之前。

我们需要做的是告诉rocksdb如何执行比较，以便它可以根据我们提供的逻辑进行排序。

编写rocksdb可以理解的比较器的接口是经典的 -1 [a < b]，0 [ a == b]，1 [ a > b]样式的返回，当a和b被比较时

最棒的部分是这些比较器可以直接在kotlin/java代码中编写并使用 - 无需修补c++ rocksdb代码。这是一个令人惊奇的功能，也是开发工作的巨大成功。

## 语义版本比较器

使用库`semver4j`来执行比较和解析的繁重工作。一个可以被输入到 Rocksdb 中的`SemverCompare`对象的实现看起来像是

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

```

|

```
class SemVerCompare(co: ComparatorOptions): AbstractComparator(co) {
 override fun name(): String {
 return "semver.comparator"
 }
 override fun compare(x: ByteBuffer, y: ByteBuffer): Int {
 val xAsString = String(x.array(), 0, x.limit())
 val yAsString = String(y.array(), 0, y.limit())
 val xAsSemver = Semver(xAsString)
 val yASSemver = Semver(yAsString)
 return xAsSemver.compareTo(yASSemver)
 }
} 
```

|

这样就可以在启动时作为`options`传递给 rocksdb。`val options = Options().setCreateIfMissing(true).setComparator(SemVerCompare(ComparatorOptions().setUseDirectBuffer(false)))`

与之前相同的示例现在看起来是这样的

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29

```

|

```
 fun main() {
 val emptyByte = byteArrayOf()
 RocksDB.loadLibrary()
 val options = Options().setCreateIfMissing(true)
 .setComparator(SemVerCompare(ComparatorOptions().setUseDirectBuffer(false)))
 val rdb = RocksDB.open(options, "/tmp/rdb-semver")
 rdb.put("1.2.4".encodeToByteArray(), emptyByte)
 rdb.put("10.2.4".encodeToByteArray(), emptyByte)
 rdb.put("2.2.5".encodeToByteArray(), emptyByte)
 rdb.put("1.3.4".encodeToByteArray(), emptyByte)
 rdb.put("1.2.4-rc.1".encodeToByteArray(), emptyByte)
 rdb.put("2.2.4-beta-2".encodeToByteArray(), emptyByte)
 rdb.put("2.2.4-alpha-2".encodeToByteArray(), emptyByte)
 printDb(rdb)
 rdb.close()
}
 === DB start ===
1.2.4-rc.1
1.2.4
1.3.4
2.2.4-alpha-2
2.2.4-beta-2
2.2.5
10.2.4
=== DB end === 
```

|

它奏效了！现在密钥根据实际语义版本规则进行排序。
