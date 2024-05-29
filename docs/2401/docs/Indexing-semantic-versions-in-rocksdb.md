<!--yml
category: 未分类
date: 2024-05-27 14:31:42
-->

# Indexing semantic versions in rocksdb

> 来源：[https://blog.aawadia.dev/2024/01/02/index-semver-rocksdb/](https://blog.aawadia.dev/2024/01/02/index-semver-rocksdb/)

## Introduction

A recent requirement that came up was the need to index an attribute for a bunch of records, which was of type semantic version. The attribute contained a `major.minor.patch` type value denoting the version of some external thing. The requirement was to be able do queries that would filter out rows with versions `>` or `<` than a specific version.

This functional requirement can be reduced to a technical definition, which is to provide an index on the semantic version attribute of the records that allows for range queries and point lookups (efficiently).

So, how do you create an index ?

## Foundational idea

When an index needs to be created on a piece of data, the first approach is to always hold the keys in a hash map where the keys are sorted. The technical primitive data structure here is a hashmap where the keys are sorted. Why sorted? Because the sorting allows for `lg(n)` based range queries. This is how we get both point lookups and range queries.

So, how do I create a hashmap where the keys are sorted?

## Implementation

A hashmap where the keys are sorted is called the `SortedMap` interface and some implementations of it are `TreeMap` and `ConcurrentSkipListMap`.

## Bringing it back

So bringing all of this back up the levels to the requirement - I can create a sorted map of semantic versions that point back to the record’s primary key, which would I can then expose via the query layer out to the world.

## Rocksdb

Rocksdb is an embeddable kv store that is a sorted map persisted to disk. We can piggy back on this feature and let rocksdb do the heavy lifting of keeping the keys sorted.

However, there is a gotcha here - Let’s look at a code example of what happens when the semantic versions are inserted and then printed back out

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

The ordering is all messed up. `10.2.4` should be the last one and `1.2.4-rc.1` should come before `1.2.4`.

This is happening because rocksdb doesn’t really know that I am putting semantic version strings in the key - there is no meaning to it - it is just a byte array and thus is getting ordered lexicographically. This also happens on linux when doing an `ls` to list out folders, the name 10 comes before 2

What we need to do is tell rocksdb how to do the comparision so that it can do the sorting based on our provided logic.

The interface to write a comparator that rocksdb can understand is the classic -1 [a < b], 0 [ a == b], 1 [ a > b] style return when a and b are compared

The best part is that these comparators can be writted directly in kotlin/java code and used - no need to patch the c++ rocksdb code. That is an amazing feature and a huge win for the development effort.

## Semantic Version comparator

Using the library `semver4j` to do the heavy lifting of the comparisions and parsing. An implementation of a `SemverCompare` object that can be fed into Rocksdb looks like

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

This can then be passed in as `options` to rocksdb at startup. `val options = Options().setCreateIfMissing(true).setComparator(SemVerCompare(ComparatorOptions().setUseDirectBuffer(false)))`

The same example as before now looks like

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

It works! The keys are now sorted based on actual semantic versioning rules.