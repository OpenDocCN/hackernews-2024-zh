<!--yml

category: 未分类

date: 2024-05-29 12:43:48

-->

# 范围分区：从零到一

> 来源：[https://www.aspiring.dev/range-partitioning/](https://www.aspiring.dev/range-partitioning/)

在构建分布式系统中使用的许多分区方法中，范围分区（将数据分割成逻辑键范围）迅速成为构建最可扩展系统的首选方法。

然而，如果你去搜索，关于如何实现这一点的资源明显不足：它被深藏在 FoundationDB 等实现中，并在论文中被略过作为一个次要细节。

范围分区已经成为许多规模最大的数据库部署的基础，例如 FoundationDB（Apple）、BigTable（Google）、Spanner（Google）和 CockroachDB。

在本文中，我们将介绍范围分区的工作原理，为什么选择它而不是其他分区方法，并为键值存储构建一个简单的实现。

为了保持本文集中在分区上，我们不会涉及到像复制、范围映射、领导选举等在范围分区分布式数据库中的相关话题。

你可以在 [Github 上找到这段代码](https://github.com/danthegoodman1/RangePartitioningPost?ref=aspiring.dev)

## 如何工作的范围分区：极速介绍

为了确保我们都理解得很清楚，我将快速介绍一下范围分区的工作方式。

很简单：数据按键的范围进行分割。

假设你有一个键值存储，其中有键 `a`、`b`、`d` 和 `e`。

使用范围分区的数据库，如 CockroachDB，将从单一范围开始，并根据需要进行分割。在这些数据库中，范围通常有它们自己的低级索引（例如 B 树），因此将它们放在单独的索引中减少了竞争，并允许数据在集群中的节点之间的粒度可移植性。

分割范围的两个最常见的条件是：

1.  超过某个大小阈值（例如 512MB）

1.  高负载，通常称为“热范围”

在我们上面的例子中，我们从单一范围 `[0, inf]` 开始。这个范围表示“我拥有包括无字节和无限字节在内的所有内容。通常它们用字节范围表示，但为了简单起见，我将显示字符串值。

但是我们的数据库有问题！键 `b` 收到了大量的 `get()` 操作，以至于它减慢了对其他键的选择性能！*是时候分割这个范围了*。

所以，我们将一个范围分成两个部分：`[0, d)` 和 `[d, inf)`。我们在 `d` 处进行分割，因此现在 `[0, d)` 范围可以处理 `b` 的负载，而减少了来自 `[d, inf)` 数据的竞争。

*注：在范围命名方面，`[` 或 `]` 表示包括（>= 或 <=），`(` 或 `)` 表示不包括（> 或 <）。*

如果我们现在插入，它落入的范围取决于键：如果我们插入 `c`，那将落入 `[0, d)` 范围，因为 `c >= 0` 且 `c < d`。如果我们插入 `z`，那将在 `[d, inf)` 范围内，因为 `z >= d` 且 `z < inf`。

这就是范围分区的工作方式！看，这很简单！

## 为什么选择范围分区

有许多理由更喜欢范围分区而不是其他方法，如哈希分区。

### 规模

尽管一些显著可扩展的数据库使用基于哈希的分区（例如Cassandra，Scylla），但它们在文档中都有相同的警告：类似于“在创建键空间时必须选择`num_tokens`”，这意味着一旦选择了分区数（通常称为“token”），**就不能在不重写数据的情况下更改它。**

这是因为如何为给定键路由操作的理解基于对该键进行哈希和对令牌数进行取模，例如 `hash('abc') % 256`。

这也意味着在插入和选择之间哈希数的变化会导致失败！

例如，也许我们插入了 `"blah"`，它被放在令牌 `42` 中，但在将令牌数从 `256` 更改为 `1024` 后，后续的选择会在令牌 `731` 中寻找它，找不到！*大问题*。

使用范围分区，您可以轻松地调整范围的数量，因为节点通信的是范围所有权，而不是令牌所有权。正如我们从第一节的解释中看到的那样，对等体可以根据范围所有权知道哪个节点拥有给定的键，因为算法不会改变！范围可以调整它们的高低来适应分割和合并（尽管合并不会经常发生，所以我们会停止谈论它）。

### 扫描性能

数据库不仅仅是关于单个键值操作，还涉及如何快速选择一个连续行的可变数量。

使用基于哈希的分区，只要所有数据共享相同的分区键，您就处于良好状态。然而，一旦必须跨越分区，就有很高的概率需要从多个节点选择，因为哈希令牌通常在节点之间随机分布。集群越大，下一个分区需要跳转网络的概率就越高。

*这慢多了... 在数据库世界中，我们不喜欢慢。*

使用范围分区，我们可以获得相关数据在同一逻辑分区的好处，但是当我们需要跳转分区时，我们很有可能需要的数据仍然在同一个节点上！仅仅因为一个范围被分割，并不意味着它会被扔到另一个节点上：基于负载的分割通常是锁定或索引性能的一个因素，而不是节点性能，因此当我们分割范围时，通常会保持它们都在同一个节点上。这也意味着，如果我们必须访问许多范围，那么我们必须访问多个节点累积数据的机会较小，尽管这种概率在扩展集群时并不会真正改变。

*在数据库世界中，随着集群的增长，我们喜欢线性扫描性能！*

一个很好的例子是CQL与SQL的比较。注意Cassandra和Scylla没有丰富的聚合函数生态系统，但CockroachDB（以及其他NewSQL数据库）有？

*扫描性能胜利*。

## 为什么不使用范围分区

范围分区并非万能之策，某些情况下效果并不理想。

**第一种情况是高度顺序工作负载，比如时间。**

如果你正在非常快速地插入时间序列数据，显然你总是一次只针对一个范围。这是有问题的，并且会造成热点。

哈希是一种防止热点的机制之一（事实上，[CockroachDB在本地包含基于哈希的索引](https://www.cockroachlabs.com/docs/stable/hash-sharded-indexes?ref=aspiring.dev#:~:text=Hash%2Dsharded%20indexes%20contain%20a,be%20seen%20with%20SHOW%20COLUMNS%20.))，但会导致对时间范围的选择变得非常缓慢，因为需要访问许多哈希范围...

**与哈希分区相比，范围分区的实现可能也更加复杂一些。**

哈希分区很简单：对键进行哈希，然后找出哪个节点拥有该令牌（例如通过流言蜚语共享的令牌所有权）。它们通常在创建时被委派给拥有它们的节点，并且只有在集群大小发生变化时才会更改。

使用范围分区时，稍微有些麻烦，因为我们必须弄清楚何时分割范围，如何分割范围，并在负载下选择哪个节点拥有范围。在节点上存储范围元数据通常使用更复杂的数据结构，如B树（用于快速键->范围查找），每次分割或移动范围时都会更改。在一致性分布式系统中，当范围移动到另一个节点时，情况变得更加复杂。如果你仔细听，你可以听到分布式系统工程师在豆袋和空咖啡杯之间哭泣，因为他们的第107次一致性测试仍然失败。

哈希环可以只使用一个很少编辑的映射，由于大多数这些数据库最终是一致的，它们可以通过流言蜚语同步令牌所有权。*真是太好了*。

**范围分区的数据库通常从单个范围开始。**

这意味着随着时间的推移，写入性能会变得更好（这很奇怪），因为范围通常受限于锁定或IO的性能（例如RAFT组）。许多数据库可以预先分割范围，但如果我听到过的话，那真是个麻烦事。通常情况下，你不会遇到这种情况，因为大多数人在小规模并且没有太多写入负载时会选择新数据库。如果你正在迁移，这是一个重要考虑因素，如果处理不当可能会使迁移变得*超级慢*。

## 构建范围分区

我们将建立一个简单的单节点键值数据库，采用范围分区。为了专注于范围分区，而不是分布范围到节点和复制等其他方面，我们将设置合成约束：当一个范围有N/2个或更多键时，我们将分割它（N是我们可以在创建数据库时传递的参数）。

我想强调，这种构建KV数据库的方法完全不是实际数据库的实现方式，这只是一个快速的实现，展示范围分区的效果！

我们将从定义一个简单的`Database`结构体开始：

```
type Database struct {
	Ranges  []*Range
	maxRangeSize int
}
```

在典型的实现中，你会使用某种形式的二分搜索，如b树，快速找到范围所有权。

为了保持代码简单、易于理解且无依赖，我们将仅使用数组和映射（牺牲性能）。

然后，我们可以创建`Range`结构体：

```
type Range struct {
	Low  string
	High string

	KV map[string]string // optimally a btree
}
```

我们将范围保持为字符串，以便阅读调试数据，但通常你会希望将其编码为字节。请注意，这也不是线程安全的：并发操作可能会导致恐慌！

我们将`Low`定义为范围的*包含*，`High`定义为*不包含*。

### 数据库功能

我们需要一些显著的数据库函数：获取（Get）、设置（Set）和删除（Delete）。

要从我们的数据库中获取数据，我们需要遍历所有范围，找到拥有当前键的范围，并检查该范围是否拥有该键。如果拥有，我们将检查该范围是否有该键。

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

删除数据很简单，只需找到范围，如果键存在，就从映射中删除它：

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

最有趣的部分留到最后：设置。

首先，我们需要能够对范围进行一些简单的设置：

```
func (r *Range) Set(key, value string) {
	r.KV[key] = value
}
```

范围分裂功能在数据库层面实现：

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

如果我们检测到范围太大，那么我们将分裂该范围。分裂非常简单：在中点创建一个新范围，移动KVs，添加到数据库中！现在我们做出了将一个未排序映射使用数组进行排序的异端行为，但实际的数据库会根据分裂的原因（例如基于负载或大小）评估分裂的位置，并找到相应的“中点”。

请注意，范围并没有决定分裂，数据库做了决定。这一点很重要。我们的范围应该只专注于执行命令，别无他求。数据库的可靠性和持久性非常重要，因此最好确保实际存储数据的代码简洁。关注点分离使得测试和关注功能变得更加容易，也给我们更多的控制权来决定何时分裂范围。

## 测试数据库

我们可以编写一些简单的测试，确认我们的怀疑是正确的，这个方法确实有效。

我们将插入零填充的键，这样我们就可以使用整数进行字符串排序，来展示我们获得预期的行为。

我们可以先插入500条记录，以展示它们都进入当前范围：

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

然后，我们可以插入更多记录，导致第一个范围分裂：

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

现在我们有了两个范围！

这里发生了什么：

1.  我们插入了更多的记录（确切地说是101次）。

1.  我们超过了每个范围的最大记录数（600），所以我们将其分裂为300和301范围。

1.  我们将最后的99条记录插入到顶部范围，因为这些键大于我们创建的新范围的高边界（我们的代码会向下创建范围）。

如果我们插入更多值，我们可以看到范围被进一步分割：

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

真是太酷了，看看所有这些范围！

用数字进行运行，让我们看看它们的词典排序：

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

您可以在[Github的这个代码](https://github.com/danthegoodman1/RangePartitioningPost?ref=aspiring.dev)上玩耍，只需按照自述文件中的说明进行操作！

## 我们没有涵盖的一些事情

我们构建了世界上最简单但最糟糕的范围分区内存数据库！

分布式数据库的一个关键实现是在范围分割和移动时的可用性。

这不是我们实现时需要处理的问题，因为我们的范围如此小，以至于当我们（b）锁定数据库以分割范围时，客户端无法察觉。但是对于FoundationDB和CockroachDB等数据库来说，它们协调一种复杂的舞蹈，以最小化数据范围不可用的时间。

另一个主要关注的领域是确保节点在范围所有权的最新信息上是最新的。这不是我们希望通过八卦来做的事情，因为可能会有短暂的（甚至可能不那么短暂的）时间段，节点将操作路由到错误的节点，认为它们拥有一个它们实际上并没有的范围。

这其实是另一种复杂的舞蹈，结合了复制魔法以及Paxos和RAFT等共识协议。这些数据库非常努力，确保你永远不会知道它发生了什么。

## 范围分区的替代使用案例

数据库并不是唯一能够使用范围分区的地方，我们可以把范围分区塞进我们构建的任何东西中。

其中一个地方实际上是计算调度！事实上，在数据库的背景下，你可以把范围分区看作是一种“调度数据”到节点的方式。开始有点意义了，对吧？

当你需要在机器上运行长时间的工作负载时，使用范围分区将它们放置在工作者上实际上是一个*非常好的主意，请听我解释*。

假设我们有一个名为[Cloudflare Durable Object](https://developers.cloudflare.com/durable-objects/?ref=aspiring.dev)，我们希望能够将它们调度到机器上，并对谁负责运行哪些实例（通过它们所拥有的范围）达成共识。这样我们就可以轻松地将请求路由到实例，并确保每次只有一个实例在运行（至少可以接收请求）。

如果一个节点死了并恢复，或者不回来（而我们需要将对象移动到其他机器上），如果有一组已定义对象的一致性隔离（对象ID的范围）将非常有帮助。能够使用粗粒度锁（在范围上）比每个对象的锁定*要高效得多*。

使用范围分区，这意味着集群可以通过添加更多机器和分割范围来轻松扩展，以适应Durable Objects的采用。

实际上，性能的一个潜在优化可能是，如果将持久化对象实例与自定义分布式KV数据库的选定领导者放置在一起（我相信他们目前使用FoundationDB），在写入时去除一个额外的网络跳跃，并且在读取时去除唯一的网络跳跃。

很棒的东西，对吧？

*免责声明：我不在Cloudflare工作，也不知道他们是否在使用范围分区，但这似乎是一个合理的猜测。*

范围分区不仅对数据库有用，而且对于任何需要定义粗粒度所有权的地方都是有用的。范围分区可以非常优雅地进行扩展和收缩，而且非常简单易懂！

[在HackerNews上讨论本文](https://news.ycombinator.com/item?id=39725649&ref=aspiring.dev)
