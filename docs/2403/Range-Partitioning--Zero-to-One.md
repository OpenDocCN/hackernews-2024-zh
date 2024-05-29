<!--yml
category: 未分类
date: 2024-05-29 12:43:48
-->

# Range Partitioning: Zero to One

> 来源：[https://www.aspiring.dev/range-partitioning/](https://www.aspiring.dev/range-partitioning/)

Of the many partitioning methods used to build distributed systems, range partitioning (dividing data into logical key ranges), is quickly becoming the preferred method to build the most scalable systems.

Yet if you go searching, there's a clear lack of resources on how to implement this: It's buried away in implementations such as FoundationDB, and skimmed over as a minor detail in papers.

Range partitioning has become the foundation for many of the databases deployed at the largest scales, such as FoundationDB (Apple), BigTable (Google), Spanner (Google), and CockroachDB.

In this post, we'll go over how range partitioning works, why it's chosen over other partitioning methods, and build a simple implementation for a key-value store.

To keep this post focused on the partitioning, we won't go over tangential topics in range-partitioned distributed databases like replication, range mapping, leader election, etc.

You can find the [code on Github](https://github.com/danthegoodman1/RangePartitioningPost?ref=aspiring.dev)

## How Range Partitioning Works: A Speedrun

Just to make sure we're all on the same page, I'll quickly go over how range partitioning works.

It's pretty simple: Data is split by ranges of the key.

Let's say you have a key-value store, with keys `a`, `b`, `d`, and `e`.

Databases like CockroachDB that using range partitioning will start with just a single range, and split as necessary. In databases like these, ranges typically have their own low-level indexes (e.g. a b-tree), so having them in separate indexes reduces contention and allows granular portability of data across nodes in a cluster.

The two most common conditions to splitting ranges are:

1.  Exceeding some size threshold (e.g. 512MB)
2.  High load, commonly referred to as a "hot range"

In our example above, we start with a single range `[0, inf]`. This range says "I own everything between and including no bytes, and infinite bytes. Typically they are represented in byte ranges, but for simplicity I'll show the string values.

But there's an issue in our database! Key `b` is getting a *LOT* of `get()` operations, so much so that it's slowing down the performance of selections for other keys as well! *Time to split the range*.

So, we split the one range into two: `[0, d)` and `[d, inf)`. We split at `d`, so that now the `[0, d)` range can handle the load of `b` with less contention from the data in `[d, inf)`.

*Note: When it comes to range nomenclature, `[` or `]` means includes (>= or <=), and `(` or `)` means excludes (> or <).*

If we insert now, the range it lands in depends on the key: If we insert `c`, that would fall into the `[0, d)` range, since `c >= 0` and `c < d`. If we inserted `z`, that would be in the `[d, inf)` range, because `z >= d` and `z < inf`.

And that's how range partitioning works! See, it's simple!

## Why Range Partitioning

There are many reasons to prefer range partitioning over other methods such as hash partitioning.

### Scale

While some notably scalable databases use hash-based partitioning (e.g. Cassandra, Scylla), they all share the same warning in the docs: Something along the lines of "you must choose the `num_tokens` when creating the keyspace", meaning that once you select the number of partitions (often called "tokens") **you can't change it without rewriting data.**

This is because the understanding of how to route operations for a given key is based on hashing that key, and performing modulo over the number of tokens like `hash('abc') % 256`.

This also means that if the number of hashes changes between insert and select, it breaks!

For example, maybe we inserted `"blah"` which was put in token `42`, but after changing the number of tokens from `256` to `1024` subsequent selects look for it at token `731`, and can't find it! *Big problem*.

With range partitioning, you can easily scale the number of ranges up and down, as instead of nodes communicating token ownership, they communicate range ownership. As we can see from the explanation in the first section, it's easy for peers to know what node owns a given key based on range ownership because the algorithm doesn't change! Ranges can adjust their highs and lows to accomodate for splitting and merging (although merging doesn't happen nearly as often, so we'll stop talking about it).

### Scan performance

Databases aren't just about single key-value operations, it also matters how fast we can select variable numbers of contiguous rows in one fell swoop.

With hash based partitioning, as long as all your data shares the same partitioning key, you're in good shape. However, once you have to straddle partitions, there's a high probability to have to select from multiple nodes, as hash tokens are typically spread out randomly among nodes. The larger the cluster, the higher the probability you have to hop the network for that next partition.

*That's much slower... and we don't like slow in the database world.*

With range partitioning, we have the same benefits of related data being in the same logical partition, but when we have to hop partitions, there's a high chance that the data we need is still on the same node! Just because a range is split, it doesn't mean it's thrown onto another node: Load-based splitting is generally a factor of lock or index performance, not node performance, so when we split ranges we often keep both on the same node. It also means that if we have to visit many ranges, there's a smaller chance that we have to visit more than one node to accumulate that data, which the probability of doesn't really change as we scale the cluster.

*We like linear scan performance as the cluster grows in the database world!*

An excellent example of this is CQL vs SQL. Notice how Cassandra and Scylla don't have a rich ecosystem of aggregation functions, but CockroachDB (and other NewSQL DBs) do?

*Scan performance ftw.*

## Why Not Range Partitioning

Range partitioning is not a silver bullet, there are cases where it's sub-par.

**The first case is highly sequential workloads, like time.**

If you're inserting time-series data super fast, you're obviously always targeting one range at a time. This is problematic, and creates hotspots.

Hashing is one mechanism to defend against hotspots (in fact, [CockroachDB includes hash-based indexing natively](https://www.cockroachlabs.com/docs/stable/hash-sharded-indexes?ref=aspiring.dev#:~:text=Hash%2Dsharded%20indexes%20contain%20a,be%20seen%20with%20SHOW%20COLUMNS%20.)), but can cause selections over ranges of time to become very slow, as many hash ranges need to be visited...

**Range partitioning can also be a bit more tricky to implement than hash partitioning.**

Hash partitioning is simple: Hash the key, then find what node owns that token (e.g. token ownership shared via gossip). They're typically delegated to their owning nodes on creation, and that only changes if the size of the cluster changes.

With range partitioning, it's a bit tricker because we have to figure out when to split ranges, where to split them, and how to choose which node owns ranges while under load. Storing range metadata on a node typically uses more complex datastructures like a b-tree (for quick key -> range lookups) which change every time a range is split or moved. This gets even more complicated in consistent distributed systems when the range is moved to another node. If you listen cloesly, you can hear distributed system engineers crying among the beanbags and empty coffee cups as their 107th consistency test is still failing.

Hash rings can just use a map that's rarely edited, and as most of these databases are eventually consistent, they can just sync token ownership over gossip. *Must be nice*.

**Range-partitioned databases typically start with just a single range.**

This means that write performance gets better over time (that's weird), because ranges are typically performance limited as they're bound to locks or IO (e.g. RAFT groups). Many databases can pre-split ranges, but that's a PITA if I've ever heard one. Typically you don't encounter this though, as most people adopt a new database when they're small and don't have too much write load. It's a major consideration if you are migrating, as it can make migrations *super slow* if not handled properly.

## Building Range Partitioning

We'll build a simple single node key-value database with range partitioning. To focus on range partitioning, and not other aspects like distributing ranges across nodes and replication, we'll create synthetic constraints as to when we split ranges: When a range has N/2 keys or more, we split it (N being a parameter we can pass at DB creation time).

I want to stress that this method of building a KV database is in no way reflective of how real databases do it, this is just a quick implementation to show off range partitioning!

We'll start by defining a simple `Database` struct:

```
type Database struct {
	Ranges  []*Range
	maxRangeSize int
}
```

In typical implementations, you'd use some form of binary search like a b-tree for quickly finding range ownership.

To keep our code simple, easy to understand, and dependency free, we'll just use arrays and maps (rip performance).

Then, we can create the `Range` struct:

```
type Range struct {
	Low  string
	High string

	KV map[string]string // optimally a btree
}
```

We'll keep the ranges in terms of strings so it's easy to read debug data, but generally you'll want to encode this to bytes. Note that this is also not thread-safe: concurrent operations could panic!

We'll say that the `Low` of the range is always *inclusive*, and the `High` is always *exclusive.*

### Database Functions

We'll need a few notable functions for our database: Get, Set, and Delete

To get from our database, we'll need to iterate over all ranges, find the range that owns the current key, and check if that range owns the key. If it does, we'll check if that range has the key.

```
func (d *Database) Get(key string) *string {
	// range range range... lol
	for _, rng := range d.Ranges {
		if rng.OwnsKey(key) {
			return rng.Get(key)
		}
	}
	return nil
}

// Range functions

func (r *Range) OwnsKey(key string) bool {
	return key >= r.Low && key < r.High
}

func (r *Range) Get(key string) *string {
	val, exists := r.KV[key]
	if !exists {
		return nil
	}
	return &val
}
```

Deleting data is simple, just find the range and delete the key from the map if it exists:

```
func (d *Database) Delete(key string) {
	for _, rng := range d.Ranges {
		if rng.OwnsKey(key) {
			rng.Delete(key)
			return
		}
	}
}

// Range function

func (r *Range) Delete(key string) {
	delete(r.KV, key)
}
```

We saved the most interesting for last: set.

First, we need to be able to do some simple set into the range :

```
func (r *Range) Set(key, value string) {
	r.KV[key] = value
}
```

The range splitting functionality comes in at the database level:

```
func (d *Database) Set(key, value string) {
	for _, rng := range d.Ranges {
		if rng.OwnsKey(key) {
			// Insert the value
			rng.Set(key, value)

			// Check if the range is too large
			if len(rng.KV) > d.maxRangeSize {
				fmt.Println("\n### Range too large, splitting! ###\n")
				d.SplitRange(rng)
			}
			return
		}
	}
}

func (d *Database) SplitRange(rng *Range) {
	// Let's just split it ~half-way... rip performance
	var keys []string
	for k, _ := range rng.KV {
		keys = append(keys, k)
	}

	// sort that boi (rip performance)
	sort.Strings(keys)

	// We only need to make one new range, and delete from the old range
	newRangeKeys := keys[:d.maxRangeSize/2]
	newRangeKV := map[string]string{}
	for _, k := range newRangeKeys {
		newRangeKV[k] = *rng.Get(k)
		rng.Delete(k)
	}

	newRange := Range{
		Low: rng.Low,

		High: newRangeKeys[(d.maxRangeSize/2)-1],
		KV:   newRangeKV,
	}

	// Update the old range low
	rng.Low = newRangeKeys[(d.maxRangeSize/2)-1]

	// Add the new range to our DB
	d.Ranges = append(d.Ranges, &newRange)
}
```

If we detect that the range is too large, then we'll split the range. Splitting is as simple as creating a new range at the mid-point, moving the KVs over, and adding it to the database! Now we commit the heresy of sorting an unsorted map using an array, but real databases would evaluate where to split based on why it's splitting (e.g. based on load, or size) and find the respective "midpoint".

Note that the range did not decide to split, the database did. This is important. Our ranges should just focus on doing what they're told, and nothing else. Reliability and durability is pretty important for databases, so best to make sure the code that actually stores the data is simple. Separation of concerns makes it much easier to test and focus on functionality, and gives us more control over when we should split ranges.

## Testing the DB

We can write some simple tests to confirm our suspicions that this is actually working.

We'll insert zero-padded keys so we can use integers with string-sorting to show that we get the expected behavior.

We can start by putting in 500 records to show that they all go into the current range:

```
// First let's insert 500 records and see the DB

db := database.NewDB(600) // 600 record limit per range
for i := 1; i <= 500; i++ {
    v := fmt.Sprintf("%05d", i)
    // Just set itself in
    db.Set(v, v)
}

// Debug it
fmt.Printf("Current DB:\n%s\n", db.DebugRanges())

// Current DB:
// 	Range:
// 	Low: ''
// 	High: 'Inf'
// 	Size: 500
```

Then, we can insert more records to cause a the first range to split:

```
// Now, let's insert again to exceed the range
for i := 501; i <= 700; i++ {
    v := fmt.Sprintf("%05d", i)
    // Just set itself in
    db.Set(v, v)
}

// Debug it
fmt.Printf("Current DB:\n%s\n", db.DebugRanges())

// ### Range too large, splitting! ###
//
// Current DB:
// 	Range:
// 	Low: '00300'
// 	High: 'Inf'
// 	Size: 400
// 	Range:
// 	Low: ''
// 	High: '00300'
// 	Size: 300
```

We now have two ranges!

Here's what just happened:

1.  We inserted a bunch more (101 times to be exact)
2.  We exceeded the max number of records per range (600), so we split into 300 and 301 ranges
3.  We inserted the final 99 records into the top range, because those keys were larger than the high border of the new range that we created (our code creates the range down)

If we insert a bunch more values, we can see the ranges get further split:

```
// Insert a ton to cause lots of range splits all over
for i := 700; i < 2000; i++ {
    v := fmt.Sprintf("%05d", i)
    db.Set(v, v)
}

// Debug it
fmt.Printf("Current DB:\n%s\n", db.DebugRanges())

// Current DB:
// 	Range:
// 	Low: '01500'
// 	High: 'Inf'
// 	Size: 499

// 	Range:
// 	Low: ''
// 	High: '00300'
// 	Size: 300

// 	Range:
// 	Low: '00300'
// 	High: '00600'
// 	Size: 300

// 	Range:
// 	Low: '00600'
// 	High: '00900'
// 	Size: 300

// 	Range:
// 	Low: '00900'
// 	High: '01200'
// 	Size: 300

// 	Range:
// 	Low: '01200'
// 	High: '01500'
// 	Size: 300
```

Pretty cool, look at all those ranges!

For run, let's jam it with numbers and see how they lexicographically sort:

```
mangleDB := database.NewDB(10_000) // o7 godspeed memory
for i := 0; i < 50_000; i++ {
    mangleDB.Set(fmt.Sprintf("%d", i), "e")
}
fmt.Println(mangleDB.DebugRanges())
fmt.Println(*mangleDB.Get("42000"))
fmt.Println(*mangleDB.Get("20000"))

// Range:
// Low: '5497'
// High: 'Inf'
// Size: 5001
// Range:
//
// Low: '41497'
// High: '5497'
// Size: 9999
// Range:
//
// Low: ''
// High: '14497'
// Size: 5000
// Range:
//
// Low: '14497'
// High: '18998'
// Size: 5000
// Range:
//
// Low: '18998'
// High: '23497'
// Size: 5000
// Range:
//
// Low: '23497'
// High: '27998'
// Size: 5000
// Range:
//
// Low: '27998'
// High: '32497'
// Size: 5000
// Range:
//
// Low: '32497'
// High: '36998'
// Size: 5000
// Range:
//
// Low: '36998'
// High: '41497'
// Size: 5000

// Getting key 42000 from range with low 41497
// e
// Getting key 20000 from range with low 18998
// e
```

You can play with this [code from Github](https://github.com/danthegoodman1/RangePartitioningPost?ref=aspiring.dev), just follow the instructions in the readme!

## Some Things We Didn't Cover

We built the worlds simplest, but worst, range-partitioned memory DB ever!

A key implementation of distributed databases is the availability of ranges when they are split and moved.

This isn't an issue we had to deal with our implementation, as our ranges are so small that when we (b)lock the DB to split ranges a client can't tell. But with databases such as FoundationDB and CockroachDB, they coordinate a fancy dance to minimize the amount of time that a range of data is unavailable.

Another major area of concern is making sure that nodes are up to date on the latest in range ownership. This isn't something that we want to do over gossip, because there would be brief (potentially not-so brief) periods of nodes routing operations to the wrong nodes, thinking that they own a range that they don't.

This is yet another fancy dance that combines replication magic, along with consenus protocols like Paxos and RAFT. These databases work really hard to make sure you never know it happens.

## Alternative Use Cases for Range Partitioning

Databases aren't the only place where range partitioning can be useful, there are abundant opportunities to jam range partitioning into what ever we build.

One place is actually compute scheduling! In fact, in the context of databases, you can kind of think of range partitioning as a way to "scheduling data" to nodes. Starting to make sense, right?

When you need to have long-running workloads on machines, using range partitioning to place them on workers is actually a *very good idea, hear me out.*

Let's say we have something called a [Cloudflare Durable Object](https://developers.cloudflare.com/durable-objects/?ref=aspiring.dev), we'd want to be able to schedule them on machines, with consensus about who is responsible for running what instances (through the ranges they own). That makes it easy for us to route requests to the instance, and ensure that only one instance is running at a time (at least that's available to receive requests).

If a node dies and recovers, or doesn't come back (and we need to move the objects to other machines), it'd be great if there was a consistent silo of defined objects that we can transfer ownership of (range of object IDs). Being able to use coarse-grained locking (on the range) is *FAR* more performant than locking per-object.

Using range partitioning, it means that the cluster can easily scale to adjust to the adoption of Durable Objects by adding more machines and splitting ranges.

In fact, a potential optimization for performance could be if they colocated Durable Object instances with the elected leader for a custom distributed KV database (I believe they currently use FoundationDB) to remove an extra network hop when writing, and remove the only network hop when reading.

Neat stuff, right?

*Disclaimer: I don't work at Cloudflare, nor do I know if they are using range partitioning, but it seems like a fair guess.*

Range partitioning is useful for not only databases, but anywhere we need to define coarse-grained ownership. Range partitioning scales up and down very elegantly, and is simple to understand!

[Discuss this post on HackerNews](https://news.ycombinator.com/item?id=39725649&ref=aspiring.dev)