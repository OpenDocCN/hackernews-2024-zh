<!--yml

category: 未分类

date: 2024-05-27 14:55:32

-->

# 多线程和突变 - Ryan Fleury 撰写 - Hidden Grove

> 来源：[https://www.rfleury.com/p/multi-threading-and-mutation](https://www.rfleury.com/p/multi-threading-and-mutation)

第一天开始编程时，程序员必须了解如何编写文本文件，并使用解释器、编译器或汇编器将其转换为执行程序。不久之后，他们必须大致了解内存是什么，以及如何使用地址和指针（即使他们在各种高级语言中，这些概念都有不同的名称）。这自然而然地导致对内存分配的基本理解。一个重要的下一步是学习编写能够操作数据批次而不是单个数据元素的代码，这既有助于提升性能，也更简化。然后他们可以理解如何[更简单地管理内存](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)，通过对这些批次进行组织，并在前期执行分配。他们可以更加熟悉CPU的底层机制，包括它如何获取内存、执行计算和写入内存。

在这些里程碑中，程序员对底层硬件的理解以及如何优雅地使用它有了更广泛的认识。由于现代硬件提供了多个独立执行的处理器单元，在这个旅程中下一个最关键的里程碑之一是编写能够*并行*执行的代码，即多个同时执行的指令流，而不仅仅是单一的指令流。这对软件性能和效用都至关重要。在像我和大多数开发者这样的多任务操作系统上编写用户模式代码时，需要使用多线程。

许多这些里程碑引入了新的约束条件，这些条件迫使代码以特定的方式进行塑造。如果程序员只解决了*某些约束条件*，那就像在组装家具时只完全拧紧一个特定的螺丝，而没有放置其他螺丝一样。代码，就像[许多其他事物](https://www.rfleury.com/p/you-get-what-you-measure)，部分取决于它的环境——即它经常面对的约束条件。

多线程是这样一个重要的里程碑。单线程代码可以使用不同的约束条件构建，因此，如果要在已经基本构建完成的“家具”上添加多线程的“螺丝”，通常会非常困难。换句话说，如果代码需要多线程，那么应该尽早在这些约束条件下构建它；越晚引入这些约束条件，就需要重写越多的代码。

有时候，更明智的是愿意接受重写的代价，尤其是如果很难预测问题的正确多线程设计，那么首先必须编写单线程版本来探索问题空间。我认为，人们经常对重写系统感到过度厌恶，并且在“可重用性”上投入了过多资源（在完全理解重用情况之前），但我岔开了话题。

对我来说，多线程最初是一个极具威胁性的里程碑，因为它引入了*如此多*的额外约束。它似乎为编程增加了一个全新的维度，并且许多在单线程空间内能够充分工作的技术，在多线程空间内突然变得无效。

但是在过去几年的学习中，我了解到，这并不意味着，到最后，一切都需要*困难*。在内化了这些额外的约束条件之后，事情反而可以变得相当容易，并相应地设计系统。设计这样的系统显然是棘手的，依赖于个人的创造力，但是多线程的复杂性更多是由于其*不透明性*，而不是其*本质难度*。换句话说，困难问题依然很难，但当我最初思考多线程时，许多*看似应该很简单*的问题也变得困难起来。

举例来说，每当有人编写着色器时，他们就在进行多线程编程。[Shadertoy](https://www.shadertoy.com/)证明了对成千上万的程序员来说，这可能是简单的，因为底层原语设计良好，大多数情况下是看不见的。我学会了，我也可以为特定问题设计底层原语，使多线程编程变得简单。

我之前在[这里写过](https://www.rfleury.com/p/a-taxonomy-of-computation-shapes)一些我学到的经验教训，这些帮助我内化了这些额外的约束条件，并设计了多线程系统。最近，我做了更多的多线程编程，我想在这篇文章中分享更多这些经验。

* * *

通常，当有人学习多线程编程时，他们会学习同步机制——例如，原子CPU操作及其经常用于实现的更高级抽象，如互斥锁、条件变量、信号量等等。尽管理解同步机制很重要，但编写多线程系统的真正任务不是弄清楚如何同步，而是如何*不需要同步*。显然，多线程代码的一个主要好处是允许多个任务同时进行。多线程系统需要的同步越多，这种好处就越少。当然，在两个系统必须共同工作时，必须存在*某种程度*的同步，在这些同步点上，理解同步机制的棘手细节至关重要——但理想情况下，绝大多数多线程代码根本不需要任何同步。

多线程代码的一个固有特性是可以在没有任何同步的情况下进行同时读取。另一方面，写操作可能需要同步（包括其他写操作*和*读取操作）——对于内存中的一个接触区域，一次只能有一个写操作是真正有意义的，因此必须有一些同步的写操作顺序，并且在进行写操作时不能发生读取操作。这意味着在多线程代码中，*读取操作*或*不冲突的写操作*比*潜在冲突的写操作*更适合。

因此，设计能够并行执行的多线程系统需要特别注意*读*发生的地方，*写*——*变异*发生的地方，以及*变异的内容*。写操作不应与其他频繁执行写操作的线程发生冲突，除非系统需要额外的同步。换句话说，每个线程执行的变异操作应尽可能*桶化*。

一个简单的*桶化变异*示例是当多个线程在堆栈上分配和变异一个局部变量时。这是毫无冲突的，因此不需要同步，因为术语“堆栈”实际上指的是*每个线程的堆栈*，因此每个线程在内存中变异完全不同的区域：

```
void ThreadEntryPoint(...)
{
  U64 x = 0;
  for(U64 idx = 0; idx < 1000; idx += 1)
  {
    x += idx; // will *never* conflict with other threads
  }
}

void EntryPoint(...)
{
  LaunchThread(ThreadEntryPoint, ...);
  LaunchThread(ThreadEntryPoint, ...);
}
```

一个简单的示例*非桶化变异*是多个线程写入内存中相同的固定地址，例如通过`static`变量：

```
`void ThreadEntryPoint(...)
{
  static U64 x = 0;
  for(U64 idx = 0; idx < 1000; idx += 1)
  {
    x += idx; // will *always* conflict with other threads
  }
}

void EntryPoint(...)
{
  LaunchThread(ThreadEntryPoint, ...);
  LaunchThread(ThreadEntryPoint, ...);
}`
```

在上述示例中，冲突的写操作——非桶化变异操作未经适当同步，因此`x`的值不能可靠地定义，因此也没有太大的用处。

* * *

变异的常见示例是变异某些数据以分配内存。因为分配是变异的一个子集，它在多线程系统中具有与其他变异相同的特征。因此，无论何时可能，*尽可能对变异进行桶化*都意味着应尽可能*对分配进行桶化*。

对于多线程目的进行桶式分配通常自然地导致许多按*生命周期*进行桶式分配的机会。因此，[竞技场分配器](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)是一个优秀的桶式分配工具。

许多人问我关于将同步机制内建到竞技场分配器中，以使得最低级别的基于竞技场的分配操作——`ArenaPush`——是线程安全的。这是一个可以理解的问题，因为其他分配器——例如`malloc`——被设计为可以同时从多个线程调用。

但使基本竞技场操作线程安全，这是本末倒置，希望我更喜欢在多线程系统中使用*只读访问*和*桶式突变*的原因是——它*假设*正确的架构要求对同一个竞技场进行同步访问。但更可取的替代方案，对于给定问题确实可能是这样的，是每个竞技场用作每个线程的桶式分配（因此是*桶式突变*）机制，以便只有一个线程可以轻松访问一个竞技场，不需要同步。

当然，竞技场可以与同步机制一起使用，这样竞技场就不是每个线程一个，而是每个数据结构，或每个哈希表条纹等等，但是在最低级别上内建同步是对理想多线程系统的误解，其中线程几乎完全不需要同步（因此几乎完全具有对共享区域的只读访问，或者非冲突的读写）。使用竞技场的同步机制是对这一理想的妥协。在许多情况下这是完全可以接受的，因为它可能是不同的理想（例如一次存储大资源，节省内存和计算时间，因此需要对共享缓存进行同步访问），但是无论如何，竞技场实现更加灵活，并且对两种情况都能够很好地运作，当同步是用户定义时，通常根本不需要同步。

* * *

在单线程空间中，通常将*几个代码路径*组合并生成*[一个有效代码路径](https://www.rfleury.com/i/112467756/effective-codepaths-vs-codepaths)*很方便或者很有用。但有时，这样做是为了使*一个低级代码路径*在*一个有效代码路径*中包含一个变异，而在相同*有效代码路径*中的*另一个低级代码路径*只包含一个读取。接下来的例子取自先前的帖子：

```
// retrieves value associated with `key` - if it does not
// exist inside `table`, then inserts it with initial value
// of `default_val`
Val ValFromKey(Table *table, Key *key, Val default_val);

Key key = ...;
Val val = ValFromKey(table, &key, default_val);
```

在先前的帖子中，我写道这种即时模式API如何在折叠特定问题的低级代码路径数量方面非常有用，特别是负责维护复杂状态机的低级代码路径数量。

但在多线程系统中，这种类型的API需要额外的分析。*有效代码路径*必须被视为具有其所有潜在的*低级代码路径*的变异属性。因此，调用`ValFromKey`的有效代码路径只能被理解为其`table`参数的变异。如果该表是跨线程共享的缓存，可能存在冲突的变异，因此需要同步。

正如我之前所讨论的，这可能是一个完全合适的设计——例如，如果我用`TextureHandle`替换`Val`，用`TextureFromKey`替换`ValFromKey`，并且该API用于在给定键的情况下快速查找共享纹理缓存，我期望*主要*由加载后的纹理的几个正在进行的读取组成（因此不需要与写入同步），那么同步就是一个完全合理的权衡。

但是细节可能会改变。假设这个有效的代码路径作为两个其他主要代码路径的辅助机制使用。在第一种情况下，对`ValFromKey`的调用在99%的情况下是*可变*的。在第二个任务中，对`ValFromKey`的调用是100%的非可变，因此仅执行只读查找。

这最近在我工作的一个项目上显现出来，该项目中使用`ValFromKey`机制来构建一个大型的去重哈希表，用于字符串，并且稍后使用相同的机制来查找该哈希表中的节点并从中收集信息。我最初编写的系统是单线程代码，并且我正在对系统进行改进以提高其性能，特别是通过将独立的工作流程移动到多个线程上并行执行，无需同步。

当然，在单线程上下文中，这段代码的功能是完全正确的——但是通过模糊可变写入和只读访问之间的界限，它为99%可变的低级代码路径和100%只读代码路径赋予了相同的名称。当然，如果我验证确实后者是100%只读的，我可以简单地*无论如何*将其称为只读有效的代码路径，但这与“作弊”非常接近——只需要进行少量完全无辜的更改就可以不可见地破坏具有新竞争条件的系统，因此我发现明确分离“显式变异”和“只读”代码路径，并相应地使用每个代码路径，更加合理。

这样做后，因为系统中的所有分配都是调用者使用`Arena`参数显式完成，我改为使用以下API：

```
`void TableInsert(Arena *arena, Table *table, Key *key, Val *val);
Val *TableLookup(Table *table, Key *key);`
```

使用这种类型的API时，调用者通过`Arena`参数控制分配的存储桶。因此，`TableLookup`是只读的事实是明确的（因为没有这样的参数），因此在与同时执行`TableLookup`的其他代码路径并行时也保证了其正确功能。

在前述问题中，这使得*第二个*代码路径的性能大幅提升——先前的单线程代码对表的100%只读查找现在可以重新组织为完全并行执行。

* * *

但是对于`TableInsert`的情况呢？很容易理解分析代码路径，得出结论：每次调用`TableInsert`都会改变`arena`和`table`，这与其他调用`TableInsert`冲突，因此所有的`TableInsert`必须进行同步。

在某些情况下，这可能是完全合理的。而在这些情况下，正如我所发现的那样，这并不一定是世界末日。我发现，通常很容易实现比仅仅在`table`和`arena`周围加锁更巧妙的同步机制。例如，如果`table`是一个哈希表，那么在取代`Arena`的情况下，`TableInsert`可以采用一个不同的机制——我称之为`Guard`——这样，插入机制可以将`(Guard, Hash)`对映射到一个`Arena`和一个读写锁，其中`Guard`可以拥有多个`Arena`和锁，细分哈希表。因此，为了实际需要写入锁，多个线程不仅需要写入相同的表，还需要写入相同表中的同一“条带”。

这已经比单一锁和arena提供了改进，但除此之外，往往有可能比那更好，采用精心设计的原子锁操作，而不是更重的锁抽象。但是对这些技术进行全面调查值得另一篇文章来阐述。从我所了解的聪明的锁定机制来看，似乎它们在已经具有同步的代码中是有用的，可以使代码再提高20%，30%或40%，但它们仍然比根本不需要同步的代码慢得多。

但是对问题的简单调整允许所有对`TableInsert`的调用进行分桶，因此不冲突。我不仅可以拥有*单个表*，还可以简单地拥有*许多表*，然后有一个单独的*“连接”步骤*，它允许我将*许多表*组合成一个*单一的连接表*。在许多情况下，一个多线程的“构建”步骤，每个线程在没有同步的情况下操作（并且在单独的arena上操作），然后是一个“连接”步骤，可能比需要对单一数据结构进行同步访问的多线程“构建”步骤更有效率。

假设`Table`是一个简单的链表链接哈希表：

```
struct Node
{
  Node *next;
  Key key;
  Val val;
};

struct Slot
{
  Node *first;
  Node *last;
};

struct Table
{
  U64 slots_count;
  Slot *slots;
};
```

那么对于具有相同`slots_count`的两个`Table`的“连接”操作，可以很容易地编写为每个槽位的链表连接。

```
Table *dst = ...;
Table *src = ...;
for(U64 idx = 0; idx < slots_count; idx += 1)
{
  if(dst->slots[idx].last && src->slots[idx].first)
  {
    dst->slots[idx].last->next = src->slots[idx].first;
    dst->slots[idx].last = src->slots[idx].last;
  }
  else if(src->slots[idx].first)
  {
    MemoryCopyStruct(&dst->slots[idx], &src->slots[idx]);
  }
}
```

如果需要对相同的`slots_count`进行类型系统强制，则API可以稍作调整如下：

```
struct Table
{
  // `slots_count` is omitted - chosen/passed by user
  Slot *slots;
};

void TableInsert(Arena *arena, Table *table, U64 slots_count, Key *key, Val *val);
Val *TableLookup(Table *table, U64 slots_count, Key *key);
void TableJoin(Table *dst, Table *src, U64 slots_count);
```

“连接”操作也可以通过许多其他功能进行扩展，例如对每个存储桶进行排序以确保确定性结果。它还可以轻松并行化，因为每个插槽的“连接”操作完全独立于所有其他插槽的连接工作。因此，“连接”操作的突变是分桶的。

* * *

类似于我在学习更简单的[内存管理](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)技术后的感受，我发现当我采用一种谨慎、踏实和有组织的方法，并且熟悉给定问题的细节，并且通过它们的低层级属性精细解开操作时，多线程编程变得显著简单，而不是仅仅依赖于高层抽象中讲述的故事。在这种情况下，这些细节包括*读取*、*冲突写入*和*非冲突写入*。

希望这篇文章也能为您提供类似的见解。

* * *

如果您喜欢这篇文章，请考虑订阅。感谢阅读。

-Ryan
