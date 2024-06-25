<!--yml

类别：未分类

日期：2024-05-27 14:53:57

-->

# Rust 的`collect::<vec>()`内存泄漏问题的识别 | 对代码编写的思考

> 来源：[`blog.polybdenum.com/2024/01/17/identifying-the-collect-vec-memory-leak-footgun.html`](https://blog.polybdenum.com/2024/01/17/identifying-the-collect-vec-memory-leak-footgun.html)

在周末，我正在进行一个个人的 Rust 项目，当时遇到了过多的内存使用问题。经过一晚的试错，我找到了一个解决内存使用问题的变通方法，但我仍然不明白问题如何可能出现，所以我又花了一个晚上研究 Rust 标准库的源代码以理解根本原因。

这是我如何识别出错误的故事。（简而言之：`collect::<Vec<_>>()`有时会重新使用分配的内存，导致`Vec`具有很大的过剩容量，即使长度事先完全已知，所以如果要释放额外的内存，您需要调用`shrink_to_fit`。）

# 背景

我正在开发的项目涉及构建稀疏加权图，然后对图进行各种计算。周一下午，我终于完成了代码的初始版本，并尝试运行它，但在到达任何地方之前，它就耗尽了我计算机的 32gb RAM。具体来说，最终的图应该有大约三百万个节点，但在图的构建过程中，当它达到 300k 个节点时，进程内存已经上升到 18gb 以上，并且很快会用尽整个 RAM，使一切都减速并迫使我杀死进程。

起初，我以为我尝试做的计算只是太大了，我的项目是不可能的，但我还没有放弃，花了晚上的时间试图弄清楚哪些部分使用了如此多的内存，是否有任何优化的方法。

# 缩小内存泄漏范围

代码中有几个地方我使用了[memoization](https://en.wikipedia.org/wiki/Memoization)，所以我最初怀疑这些是内存使用暴增的原因。我修改了图构建函数，使其在 300k 个节点后停止（当内存使用量达到 18gb 时，通过`top`查看，虽然很大，但还不足以使计算机崩溃），并添加了`println!`来记录记忆结果的预计总大小。然而，它们只在 100mb 范围内，远远不足以解释观察到的内存使用情况。

接下来，我尝试运行图构建*两次*（但每次都设置为在 300k 个节点时尽早停止），以尝试缩小原因范围。我的主函数看起来像这样（`denom`和`optimistic`是算法的参数）。

```
let denom = 257;
let optimistic = false;
let graph = build_graph(denom, optimistic);
let graph = build_graph(denom, optimistic); 
```

在运行过程中，我仔细观察了进程的内存使用情况，以查看是否会停留在 18gb 或不会停留。我认为也许是由于内存碎片化导致记忆使用的内存比预期的要多得多而爆炸性地增加堆。

由于代码是确定性的，调用 `build_graph` 两次不会导致任何新的结果被记忆，因此内存使用量应该在 18gb 处停止，即使运行两次也是如此。相反，在第二次 `build_graph` 调用期间，内存使用量继续迅速增加，这毫无道理。

幸运的是，之后我尝试在中间删除了 `graph`：

```
let graph = build_graph(denom, optimistic);
std::mem::drop(graph);
let graph = build_graph(denom, optimistic); 
```

突然，这次时间进程内存使用量在第一次 `build_graph` 调用后从 18gb 下降到了约 500mb，然后又回升到了 18gb。显然，内存某种程度上是由 `graph` 本身浪费的。和以前一样，我尝试打印出图的大小，但远远不足以解释内存使用量。

但是，我的 `graph` 数据结构由 `Vec` 组成，而 Vec 可以使用比大小指示的更多的内存，因为它们预先分配一个缓冲区，最多可容纳 2x 的容量，以使插入的摊销 O(1) 时间。我将其更改为记录总 *容量* 而不是 *长度*，突然图的估计大小为 18gb。中了！我在图构建代码中添加了 `shrink_to_fit()` 调用（它们删除了过量的容量），突然内存问题消失了。

# 等等，但是为什么？

我最终确定了 `shrink_to_fit` 的解决方法是在周一晚上晚些时候，并计划在随后的周末继续项目工作。我第二天早上在工作中向同事提到了我的经历，作为关于 `shrink_to_fit` 重要性的 PSA，但他们不相信我，当我再仔细考虑时，我自己也不相信。

首先，`Vec` 每次仅以 2x 的速度增长缓冲区，这意味着（忽略小 vecs 的初始分配），容量最多始终是长度的两倍（假设您不删除元素），因此来自过量容量的内存浪费应该始终最多为 2x，但我看到的却是超过 200x。

其次，内存泄漏来自于图的边缘列表，我将其存储为 `Vec<Vec<(u32, u32)>>`（一个表，其中一个 `Vec` 存储每个节点的边缘列表）。在将它们插入之前，当我对边缘列表调用 `shrink_to_fit` 时，内存问题消失了。然而，生成边缘列表的最后一步是调用 `vec.into_iter().map(..).collect()` 将它们转换为适当的格式。由于 `collect` 分配一个新的向量，并且长度完全已知，它只为元素分配足够的空间，绝不会有 *任何* 多余的容量，更不用说是 200 倍的多余容量了。

# Vec 源代码

显然，我的关于 `Vec` 实现的假设在某种程度上是不正确的，因此出于好奇，周二晚上下班后，我深入研究了 `Vec` 源代码，试图弄清楚到底发生了什么。有几层间接和特殊化，但很快就可以追踪所有相关代码（或者我认为是的）。

当您调用`collect()`时，它调用`FromIterator::from_iter`。 [`Vec`的`FromIterator`实现](https://github.com/rust-lang/rust/blob/c58a5da7d48ff3887afe4c618dc04defdee3dab5/library/alloc/src/vec/mod.rs#L2791)又调用`SpecFromIter`，这是一个内部 trait。

[`SpecFromIter`](https://github.com/rust-lang/rust/blob/c58a5da7d48ff3887afe4c618dc04defdee3dab5/library/alloc/src/vec/spec_from_iter.rs) 对于`IntoIter`有一个专门化（即，如果您调用`vec.into_iter().collect()`而没有中间的迭代器适配器，它将直接返回原始 vec 而不是创建新的）。 否则，它调用第二个内部 trait，`SpecFromIterNested`。

[`SpecFromIterNested`](https://github.com/rust-lang/rust/blob/c58a5da7d48ff3887afe4c618dc04defdee3dab5/library/alloc/src/vec/spec_from_iter_nested.rs) 又有两个实现，一个用于一般迭代器，一个用于`TrustedLen`迭代器（这里是相关情况）。 但是，这些实现基本上符合您的预期。 它只保留了与迭代器的`size_hint`相等的容量，没有任何可能导致多余容量的迹象。

我还检查了 Vec 分配代码，并确认每次仅将缓冲区增加 2x。

# 记录前后

在源代码中达到了死胡同后，我尝试向我的代码添加日志记录（并再次注释掉`shrink_to_fit`）以查看问题有多严重。 我记录了最终的`into_iter().map().collect()`步骤之前和之后边缘列表的长度和容量，以及长度和容量的累积总数。 （这里的`get_cull_all`是计算边缘列表的函数，而`main_edges`是边缘列表的最终列表。）

```
let res = get_cull_all(ds, max_denom as u128, optimistic);
println!("precol {} {}", res.len(), res.capacity());

let mut res: Vec<_> = res
    .into_iter()
    .map(|(ds2, p)| (cull_map.get(ds2), p as u32))
    .collect();

main_len_tot += res.len();
main_cap_tot += res.capacity();
println!("{} {} tot {} {}", res.len(), res.capacity(), main_len_tot, main_cap_tot);
// res.shrink_to_fit();
assert!(main_edges.len() == id);
main_edges.push(res); 
```

这产生了如下输出

```
precol 46 11400
46 34200 tot 10410429 2330729253
precol 54 11340
54 34020 tot 10410483 2330763273
precol 2 5520
2 16560 tot 10410485 2330779833 
```

至 300k 节点累积的浪费空间为 2330779833/10410485 或超过**223x**的浪费空间。 这也证实了边缘列表在`collect()`调用后具有显着的多余容量。 实际上，由`collect()`产生的新 vec 的容量始终恰好是原始 vec 的容量的三倍，而与长度无关。

# 游乐场

确认了尽管源代码看起来不可能这样，但`collect()`在某种程度上确实会产生具有多余容量的 vec，我接下来尝试在 Rust 游乐场上重现此问题。 我无法复现我在代码中遇到的 3x 增加，但我确实注意到了一些有趣的事情。

```
let mut v = vec![1,2,3];
v.push(4);
println!("len {} cap {}", v.len(), v.capacity());
let v = v.into_iter().map(|x| x+1).collect::<Vec<_>>();
println!("len {} cap {}", v.len(), v.capacity());
let v = v.iter().copied().map(|x| x+1).collect::<Vec<_>>();
println!("len {} cap {}", v.len(), v.capacity()); 
```

[运行此代码](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=547a67d6857b726120f73e03efd703ed)将打印出

```
len 4 cap 6
len 4 cap 6
len 4 cap 4 
```

因此，在这里的向量确实有多余的容量，即使在 playground 上也是如此，尽管只有一点点。此外，如果您使用 `iter().copied()` 而不是 `into_iter()`，多余的容量就会消失。不幸的是，这一部分让我分心了一段时间，因为我认为 `SpecFromIter` 的 `IntoIter` 特化可能与之相关。虽然它*不应该*，因为这里的迭代器是 `Map<IntoIter>` 而不是 `IntoIter` 本身，但我找不到其他相关内容，花了很长时间盯着源代码，希望能找到答案。

由于原始问题在 playground 上没有出现，我以为可能是特定版本的问题。在我的代码中，我正在使用夜间编译器，因此可以使用`Vec::is_sorted`，但幸运的是，我只是用它来进行了一些断言，因此在注释掉这些断言之后，我能够切换到了 beta 编译器（1.76），在那里出现了完全相同的行为。至于稳定版，由于与 `impl Trait` 的一个无关问题导致的编译器错误，我无法尝试稳定版（1.74）。 （事实证明，问题只存在于 beta 版本中，而不是稳定版，所以这是一个不幸的错过。）我还确认了该问题在非发布构建中重现，因此与优化无关。

# 向量类型变更

我继续尝试在 playground 上缩小并重现我的问题。经过更多的试验和错误，我发现将向量映射到*更小*类型会导致多余的容量，甚至在 playground 上也是如此，但*仅限于* beta 和 nightly 版本，而不是稳定版。

```
let mut v = vec![1,2,3,4];
v.push(1u128);
println!("len {} cap {} v={:?}", v.len(), v.capacity(), v);
let v = v.into_iter().map(|x| x as u64).collect::<Vec<_>>();
println!("len {} cap {} v={:?}", v.len(), v.capacity(), v);
let v = v.into_iter().map(|x| x as u32).collect::<Vec<_>>();
println!("len {} cap {} v={:?}", v.len(), v.capacity(), v);  
let v = v.into_iter().map(|x| x as u8).collect::<Vec<_>>();
println!("len {} cap {} v={:?}", v.len(), v.capacity(), v);  
let v = v.into_iter().map(|x| x as u16).collect::<Vec<_>>();
println!("len {} cap {} v={:?}", v.len(), v.capacity(), v); 
```

```
len 5 cap 8 v=[1, 2, 3, 4, 1]
len 5 cap 16 v=[1, 2, 3, 4, 1]
len 5 cap 32 v=[1, 2, 3, 4, 1]
len 5 cap 128 v=[1, 2, 3, 4, 1]
len 5 cap 5 v=[1, 2, 3, 4, 1] 
```

此外，容量与类型大小的减小成比例地增加。在此示例中，从`u128`到`u64`再到`u32`等，每次都会使容量翻倍。另一方面，正如最后一个示例中（`u8` -> `u16`）所示，增加类型大小会立即删除多余的容量。

# 分配重用

由于容量随元素大小的减小而增加，这意味着新向量的支持数组大小可疑地与旧向量相同。我假设它以某种方式重用了旧向量的内存作为优化。这也解释了为什么只在 `into_iter()` 中发生这种情况，以及为什么使用 `iter().copied()` 可以消除多余的容量。使用 `iter()` 时，旧向量不会被丢弃，因此其内存无法重用。在我的代码中，插入之前的最后一步将边缘列表从 `(u64, u128)` 映射到 `(u32, u32)`，大小为原来的三分之一，这就是为什么转换后容量始终增加了 3 倍的原因。

这暗示的另一件事是，尽管 `collect()` 会使向量的容量增加三倍，但实际的内存使用量并未改变，这意味着 `collect()` 本身并不是内存泄漏的原因。相反，它仅仅是*未能减少*现有内存使用量。为了理解这是如何导致进程内存使用量激增的，需要对我的代码进行一点解释。

我的边缘列表计算逻辑在内部使用 `u128` 来增加精度，然后在最后将权重截断为 `u32`。此外，在某些情况下，它会生成一个潜在边缘列表，然后在返回之前过滤掉大部分边缘，这导致了大量的过剩容量。作为一个极端的例子，上面显示的日志输出中的最后一次迭代长度为 2，但容量为 5520。

通常情况下，这不会成为问题，因为`into_iter().map().collect()` 行用于将它们打包成 `(u32, u32)`，将分配一个新的向量，其空间仅仅是所需的确切量。然而，由于 Rust 1.76 添加的分配重用优化，新的 vec 共享了输入 vec 的后备存储器，因此容量为 16560，这意味着它使用 132480 字节的内存来存储仅有 16 字节的数据。

由于边缘列表计算逻辑是按顺序逐个节点运行的，因此它暂时使用 132kb 的内存通常不会有任何问题。然而，新的 `collect()` 优化意味着内部存储被保留在最终的 vec 中，这个 vec 被插入到边缘列表的列表中，因此永远保留着那 132kb 的过时内存。将其乘以 300k 个节点，突然间你就泄露了计算机的整个 RAM。

* *从技术上讲，Rust 有时会甚至在稳定版上重用分配，就像在 playground 上显示的初始 len=4,cap=6 的例子一样。Rust 1.76 只是使优化更智能，因此在更多情况下触发。*

# 源代码

现在我知道我特别在寻找一个在 Rust 1.76 中添加的提交，查看 Github 上 `Vec` 源代码的最近变化并找到 [可能的罪魁祸首](https://github.com/rust-lang/rust/commit/13a843ebcbe536257a8442bf5c26b227d1c2f7c9) 并不困难。原来有 [一个专门用于实现这个优化的 412 行文件](https://github.com/rust-lang/rust/blob/c58a5da7d48ff3887afe4c618dc04defdee3dab5/library/alloc/src/vec/in_place_collect.rs) 我之前忽略了。

这是怎么可能的？Rust 中更烦人的特性之一是 Trait impls 可以*在同一个 crate 中的任何地方*添加，而不仅仅是在你逻辑上期望它们的文件中，并且没有办法找到隐藏的 impls，除非进行全面的代码搜索。

在这种情况下，`SpecFromIter` trait 在 [spec_from_iter.rs](https://github.com/rust-lang/rust/blob/c58a5da7d48ff3887afe4c618dc04defdee3dab5/library/alloc/src/vec/spec_from_iter.rs) 中被定义，连同该 trait 的两个实现，因此我自然而然地假设那就是所有的实现。我没有办法猜测到实际上还有一个*第三个*实现，它隐藏在完全不同的文件中（in_place_collect.rs）。

从技术上讲，有一个小小的提示。`spec_from_iter.rs` 包含以下注释：

```
/// Specialization trait used for Vec::from_iter
///
/// ## The delegation graph:
///
/// ```text

/// +-------------+

/// |FromIterator |

/// +-+-----------+

///   |

///   v

/// +-+-------------------------------+  +---------------------+

/// |SpecFromIter                  +---->+SpecFromIterNested   |

/// |where I:                      |  |  |where I:             |

/// |  Iterator (default)----------+  |  |  Iterator (default) |

/// |  vec::IntoIter               |  |  |  TrustedLen         |

/// |  SourceIterMarker---fallback-+  |  +---------------------+

/// +---------------------------------+

/// ``` 
```

这段注释列出了文件中找到的两个 impl，以及一个看似缺失的“SourceIterMarker”的第三个 impl。事实上，我曾经试图在某个时候搜索“SourceIterMarker”，但唯一找到的东西就是*在 spec_from_iter.rs 中的相同注释*，由于文件中没有第三个 impl 的迹象，我认为这只是一个错误或过时的注释。回过头来看，我应该在 Github 上对“SourceIterMarker”进行全面的代码搜索，以防它被偷偷隐藏在另一个文件中。

# Box<[T]>

如前所述，在将边列表存储到最终图形中之前添加对`shrink_to_fit()`的调用消除了内存泄漏问题。这在作为一个快速的 hack 的情况下是可以的，但这并不是正确的解决方案。*真正*的修复方法是调用`into_boxed_slice()`。

基本问题在于`Vec`被优化为*可变*列表的情况，即您计划在将来*添加和删除*元素。然而，人们也经常将它们用于*固定长度*列表数据，即使这不是 Vec 设计的目的，只是因为 Vec 是在 Rust 中存储列表数据的最简单和最*直观*的方式。固定长度列表的*适当*数据结构是`Box<[T]>`而不是`Vec<T>`，但即使*知道*这是一种选择，语法看起来也很奇怪，需要很多的 Rust 经验。

在 Java 中，不可变字符串称为`String`，而可变字符串（类似于 Rust 的`String`）则称为`StringBuilder`。这会促使人们采取适当的行为，因为如果你将名为`StringBuilder`的东西存储在长期不可变的数据结构中，你看起来会很愚蠢。与此同时，在 Rust 中，几乎没有人使用`Box<str>`而不是`String`。

当然，我并不是说 Rust 应该将`Vec`称为“ListBuilder”或其他什么。可变数组列表是算法和数据结构的**极其**常见的组件，因此保持名称简短且易记是很重要的。只是有点不幸的是，Rust 设计的一个副作用是，它促使人们*也*使用`Vec`来存储固定长度的数据。

事实证明，我实际上已经在其中一个我使用记忆化的函数中使用了`Box<[T]>`来保存结果。我的主要动机是避免浪费额外的字来存储 *值* 方面的容量，但作为副作用，这也方便地使其免受任何潜在的多余容量错误的影响。（我使用记忆化的另一个地方只是出于懒惰使用了`Vec`，但幸运的是，在那里并不重要。）

# 紧凑数组

我打算以后尝试的另一个可能的优化是避免首先存储单独的列表。相反，有一个单一的“环境” Vec，并且将 *所有* 列表的数据 *连续地* 存储在该 Vec 中，只需传递一个（索引，长度）对到该 Vec 中，而不是真正的`Vec`/`Box<[]>`等。

这种设计有时可用（以这种方式无法自由地释放或传递单独的列表，并且在代码设计方面更具侵入性（您必须通过所有代码传递对环境 vec 的引用）），但除了使您自动免受多余容量错误的影响之外，它还避免了具有许多小的单独分配的列表可能引起的潜在内存碎片问题。此外，它还可以让您将列表表示为仅为`(u32, u32)`，假设总长度小于 2³²，在值方面也可以节省 2x，尽管需要增加边界检查的成本。

这个技巧的适用性有限，但对于诸如记忆化（其中结果永远存在并存储在单个位置）或图数据结构的边列表（在创建后不需要添加或删除边）等情况非常适合。

# 结论

这是一个令人惊讶和恼人的错误调查，但至少在这个过程中我学到了很多关于 Rust 的知识。我不知道标准库正在底层进行这种优化（并且由于这是基于代码的，所以即使在调试构建中也会发生）。这也说明了一个地方的优化如何通过违反程序员的期望而导致下游错误。

无论如何，我希望写下我的经验能帮助其他人避免将来这种“踩坑”，或者至少能教给人们一些有关 Rust 的有趣知识。
