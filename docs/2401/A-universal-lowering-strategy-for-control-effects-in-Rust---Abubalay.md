<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:47:39

-->

# Rust 中控制效果的通用降级策略 - Abubalay

> 来源：[https://www.abubalay.com/blog/2024/01/14/rust-effect-lowering](https://www.abubalay.com/blog/2024/01/14/rust-effect-lowering)

## Rust 中控制效果的通用降级策略

2024 年 1 月 14 日

Rust 语言已逐步发展了一套模式来支持*控制流效果*，包括错误处理、迭代和异步 I/O。在 [Rust 寄存器](https://without.boats/blog/the-registers-of-rust/) 中，boats 阐述了这种模式的四个方面，这种模式是 Rust 的三个效果共享的。如今，这些效果通常是孤立地使用，或者最多以定制的方式组合在一起，但 Rust 项目一直致力于找到更深入地将它们彼此整合的方法，比如 [`async gen` 块](https://github.com/rust-lang/rust/pull/118420)。

代数效果和处理程序的理论探索了这个设计空间，并提供了对 Rust 项目在这项工作中遇到的许多问题的答案。本文将相关 Rust 使用的模式与效果的术语和语义联系起来，以帮助建立共享词汇和对组合多个效果的影响的理解。

这种联系暗示了一个 API 模式，可以为任意组合的效果实例化。它以一种支持现有 Rust 习惯用法的方式概括了 Rust 现有的 `Iterator` 和 `Future` 特性，同时由于结构而保持了零成本抽象。 （换句话说，它与一个专门手写的版本的方式相同，没有额外的堆分配或间接性。）它还与假设的特性（如 `async Drop`、（取消）固定、借贷或效果多态性）向前兼容，但不需要。

### 一些定义及其在 Rust 中的应用

考虑一个“函数”的形式概念，独立于编程语言。在这个层面上，函数的概念主要是通过它与程序的其他部分的交互来定义的，而不是通过任何特定的语法或实现策略。也就是说，当我们称呼某个东西为函数时，我们指的是一个子程序，它可以应用于一些参数值，此时它的参数绑定到这些值，然后运行以产生结果。

代数效果是一个类似抽象的概念。在函数是以参数和应用为基础定义的地方，效果是以效果操作和处理程序为基础定义的。执行其中一个操作会暂停执行，并将控制流转移到调用栈中的某个处理程序。处理程序接收代表挂起计算（一个“续延”）的一流值，该值可以用于在操作后传递控制流并恢复执行。

> #### 类型和效果系统
> #### 
> 效应和函数都是纯粹动态的构造。它们可以完全不需要类型，就像动态类型语言中那样存在。然而，带有类型系统的效应语言通常会将其扩展为“类型和效应系统”，以跟踪每个子程序可能执行的效应操作集合。这样做的一个原因是确保在执行操作时始终有一个适当的处理程序可用。另一个原因是驱动代码生成，当编译器精确知道哪些交互可能发生时，可能会产生更有效的输出。然而，本文侧重于效应的动态方面。

可以用签名来描述一个效应，该签名指定了效应计算和其处理程序之间如何传递值。例如，我们可以发明一种符号，其中效应操作看起来像函数：

```
effect my_effect {
    operation_x(A) -> B;
} 
```

通过提供`A`来执行`operation_x`的效应计算。处理程序接收这个`A`，为了恢复计算，它提供了一个`B`，这个`B`看起来就像是`operation_x`的结果。在我们的符号中，这个过程看起来像这样：

```
fn my_computation() -> C {
    let b: B = operation_x(A { ... });

    C { ... }
}

fn main() {
    handle my_computation() {
        // The computation has performed an operation:
        operation_x(resume, a) => { resume(B { ... }) }

        // The computation has completed:
        return(c) => {}
    }
} 
```

这样的效应计算是执行一系列效应操作的运行子程序。重要的是，对于这个序列中的每个操作，处理程序都可以自由选择何时恢复计算，甚至是否完全恢复计算。这就是效应处理程序与必须始终恢复其调用者（或发散）的一流函数之间的区别。

通过将Rust程序解糖成我们的符号，我们可以将Rust的具体效应映射到这些抽象定义上。这并不是针对Rust本身的任何提议——仅仅是一种简洁地描述Rust行为的方式，在我们考虑其实现之前。

这是Rust迭代效应的签名：

```
effect gen<Item> {
    yield(Item) -> ();
} 
```

提出的`yield`运算符是它的一种操作。它将一个`Item`传递给处理程序，处理程序使用`()`恢复它。`for`循环是Rust用于处理这种效应的专用语法。以下是它的解糖过程：

```
for item in some_iter() {
    loop_body(item);
} 
```

```
handle some_iter() {
    yield(resume, item) => {
        loop_body(item);
        resume(())
    }
    return(()) => {}
} 
```

处理程序可以选择通过`break`中断循环而不恢复计算。它也可以选择先恢复另一个计算，就像`zip`组合器一样。Rust选择外部迭代等同于代数效应的一流控制流。

这是Rust的失败效应的签名：

```
effect try<E> {
    throw(E) -> !;
} 
```

计算将错误`E`传递给处理程序，该处理程序永远不会恢复计算。Rust没有专门的语法来处理这种效应，但它确实有`?`运算符。

在抽象层面上，这一点的原因来自于Rust如何选择可中断计算的边界。具有效应的典型语言将处理程序和操作之间的整个堆栈帧链视为一次要挂起和恢复的单个计算——例如，这就是其他语言中异常（与此效应签名相同）的工作方式。相反，Rust将单个函数体和块视为需要单独处理的不同计算。

为了方便起见，`?` 操作符被用于*传播*此效果穿过一系列计算，直到它到达实际处理程序。它的行为类似于一种立即重新执行效果的空操作处理程序：

```
handle try_work() {
    // Propagate the `try` effect by forwarding `throw` operations in `try_work`.
    throw(resume, e) => { throw(e) }
    return(x) => { x }
} 
```

Rust 的 `async` 效果的签名稍微有些模糊，因为它具有与任意外部事件交互的灵活性。一种可能性是给它多个操作，就像这样：

```
effect async {
    read(File, &mut [u8]);
    write(File, &[u8]);
    // ...
} 
```

然而，这使得处理程序负责实现所有这些操作，而在 Rust 中，该集合是可以扩展的。（它还忽略了其与故障效果的交互，下面会讨论。）从[使用代数效果进行结构化异步](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/05/asynceffects-msr-tr-2017-21.pdf)改编的更精确签名如下所示：

```
effect async {
    sleep(impl FnOnce(&Waker)) -> ();
} 
```

现在，计算提供了一个回调函数，该函数安排在 I/O 完成时发出 `Waker` 信号。`.await` 操作符与 `?` 类似，用于传播：

```
handle async_work() {
    // Propagate the `async` effect by forwarding `sleep` operations in `async_work`.
    sleep(resume, schedule) => { resume(sleep(schedule)) }
    return(x) => { x }
} 
```

### 合并效果

到目前为止，我们只考虑了具有单一类型效果的计算，但许多计算可以执行多种类型的效果。例如，迭代器可能会执行一些异步 I/O 来生成其输出，或者异步计算可能会失败。效果的抽象定义可以毫无困难地适应这一点——来自不同效果的操作简单地将控制传递给不同的处理程序。

结果，计算的合并效果形成了一个扁平的、无序的集合。这是一个极其有用的特性，因为它意味着程序可以统一且独立地处理每个效果，而不管它们是如何组合的。向计算添加新效果不需要改变其余计算的编写方式，也不需要改变其余效果的处理方式。这种可移植性是 Rust 项目对 `async` 的高级目标之一（即它是同步和异步之间可移植性的一般化），使代数效果成为思考此语言特性的一种吸引人的方式。

考虑 `async` 和 `gen` 的组合。第一个效果由执行器处理，而第二个效果由 `for` 循环处理。这两个处理程序可以以任何顺序指定，而不改变计算的任何内容，它自由地交错 `.await` 和 `yield`。例如，这是一个为拟议的 `for await` 循环进行解糖处理，处理 `gen` 效果并传播 `async` 效果的示例：

```
for await item in some_async_iter() {
    loop_body(item);
} 
```

```
handle (handle some_async_iter() {
    yield(resume, item) => {
        loop_body(item);
        resume(())
    }
    return(()) => {}
}) {
    sleep(resume, schedule) => { resume(sleep(schedule)) }
    return(()) => {}
} 
```

相反，这是一个组合器，通过处理 `async` 效果并传播 `gen` 效果将异步迭代器转换为同步迭代器的组合器：

```
handle (handle some_async_iter() {
    sleep(resume, schedule) => {
        /* ... block_on ... */
        resume(())
    }
    return(()) => {}
}) {
    yield(resume, item) => { resume(yield(item)) }
    return(()) => {}
} 
```

这两者也可以用相反的处理程序嵌套方式编写，将传播处理程序置于内部，而非传播处理程序置于外部，而不改变行为。处理一个效果只会产生一个更大的计算，其中该效果已从其集合中移除。

这种可交换性对于如何评估这些虚构的 `handle` 表达式有着重要的影响。一个好的例子是 `async` 和 `try` 的组合。像 Tokio 的 [`AsyncReadExt::read`](https://docs.rs/tokio/latest/tokio/io/trait.AsyncReadExt.html#method.read) 这样的 API 可能执行 `sleep` 或 `throw` 操作中的任何一个。在内在层次结构方面进行思考是很诱人的，外层的 `Future` 驱动至完成以达到内部的 `Result`，事实上这也是今天传播这些效果的唯一方式：`f.read(buf).await?`。但是这种限制仅仅是 Rust 如何降低这些效果的产物，而不是由这些效果本身所决定的。例如，这里是对 `f.read(buf)?.await` 的展开：

```
handle (handle f.read(buf) {
    throw(resume, e) => { resume(throw(e)) }
    return(len) => { len }
}) {
    sleep(resume, schedule) => { resume(sleep(schedule)) }
    return(len) => { len }
} 
```

`handle` 表达式的关键在于，在 `handle` 表达式之前，对于 `handle` 表达式的 `scrutinee` 并不完全进行评估，就像对于 `match` 的 `scrutinee` 一样。相反，`scrutinee` 和 `handle` 分支同时进行评估，以便进行效果操作时来回切换。由 `read` 执行的 `sleep` 操作穿越内部的 `handle` 表达式，以达到外部的 `handle` 表达式，就像 `break` 穿越任意数量的包含表达式以达到循环，或者 `return` 穿越其包含的表达式以达到 `fn` 一样。

在大多数情况下，Rust 中的具有效果的计算“衰变”为它们的降级，例如 `async { 42 }` 或调用 `async fn() -> i32` 的表达式的类型为 `impl Future<Output = i32>` 而不是 `i32`。但是作为 `async` 效果的 `handler` 的 `scrutinee`，这些表达式保持未衰变状态，并且移除最后效果的最外层 handler 表达式成为非具有效果、非衰变的计算。正确处理多种效果意味着推迟此衰变，直到程序实际要求为不是 handler 的包含表达式的点。

> 如果此推迟行为感觉奇怪或神奇，请考虑位置和值表达式（也称为左值和右值）的行为。像 `obj.field`、`*expr` 或 `slice[i]` 这样的位置表达式不会立即执行任何操作，而只会计算位置。如何处理该位置的选择要推迟到包含表达式需要特定行为的时候：取地址运算符 `&expr` 将该位置实现为引用；调用 `f(expr)` 将从该位置移动；赋值 `expr = v` 将向该位置写入。就像读取 vs 写入一样，运行 vs 衰变的选择是表达式使用的特性。

### Rust 如何实现效果

Rust通过将效果计算降级为无栈协程来实现效果。 （更准确地说，这是Rust的最一般策略；一些效果使用此方法的子集或特殊化。）编译器将每个计算转换为一个对象以保存其在挂起时的状态，并具有恢复计算的方法。在Rust中，此对象通常称为“状态机”，但为了避免歧义，我将使用更具体的术语 *协程帧*。

在 [为什么选择异步 Rust？](https://without.boats/blog/why-async-rust/) 中，boats 讨论了这一选择的历史和动机：尽管效果的典型实现使用传递或有栈协程，但这些实现也引入了Rust无法承受的分配和间接性。Rust的协程帧具有满足这些要求的两个关键特征：

+   它们是普通的值类型，具有静态可知的布局和静态可分派的恢复方法，可以组合成更大的计算，这些计算本身也是普通的值类型。从处理器到效果操作的整个帧链只使用一个分配，可以来自任何地方——堆、栈、全局、结构体字段等。

+   它们在原地被改变，为计算的局部变量提供了一个稳定的地址。如果协程帧在每次暂停时重新构建或移动，则对局部变量的活动引用会迫使这些局部变量生活在一个单独的分配中，尽管前面的观点如此。

Rust的降级过程为每个效果都提供了与其签名相对应的特质方法。将具有效果的计算（编码为协程帧）实现为该特质；处理器重复调用该方法并匹配结果。因此，带有操作 `e(A) -> B` 的效果签名将成为一个特质方法 `fn m(Pin<&mut Self>, B) -> Either<A, Output>`，直到通过返回 `Output` 完成计算为止。这一情景受到三个因素的影响而变得复杂：

+   在执行任何效果之前，此方法也用于启动计算。对于此初始调用，参数 `B` 没有意义。因此，Rust的迭代器和 futures 被称为“懒惰”，而失败（使用 `B = !`）无法在其初始调用中使用此方法，因此是“急切”的。

+   带有 `async` 效果的计算并不会字面上将 `impl FnOnce(&Waker)` 传递给处理器。相反，处理器主动将 `&Waker` 作为附加参数提供给恢复方法，并期望计算在将控制权转移给处理器之前使用它。这依赖于上面的观点——计算的初始部分与后续部分一样需要 `&Waker`。

+   迭代效果的特质是在语言添加对自引用协程帧的支持之前设计的。在这种情况下，`Pin<&mut Self>` 参数变为 `&mut self`。

`Either<A, Output>`返回类型也被实例化为每种效果的不同`enum`，将我们现有的Rust三种效果降低的形式作为此一般配方的特殊化：

```
trait Iterator {
    type Item;

    // isomorphic to Either<Item, ()>
    fn next(&mut self) -> Option<Self::Item>;
} 
```

```
trait Future {
    type Output;

    // isomorphic to Either<impl FnOnce(&Waker), Output>
    fn poll(Pin<&mut Self>, &mut Context<'_>) -> Poll<Self::Output>;
} 
```

```
// isomorphic to Either<E, Output>
fn /* ... */ -> Result<Output, E>; 
```

> #### 另一种历史：零成本的继续传递
> #### 
> 随着更深层次的语言和编译器集成，一种语言可以将无栈协程的效率带入继续传递风格。相关的见解是，传统的调用堆栈已经是一种继续传递的形式，通过线性限制来实现连续分配和原地变异。进一步限制继续传递到静态深度使得它们的布局在静态上是可知的，就像协程帧的链条一样。
> 
> 每个悬停点可以降低为一个包含实时值的不同闭包样式匿名结构体，其动态大小的尾部表示其调用者链。每个结构体的`resume`方法以对继续的调整结束，然后尾调用到调用者、调用者或处理器。处理器为这些对象提供了固定大小的空间，并且每个叶子悬停点将其对象借给处理器。
> 
> 这种方法具有几个优点：
> 
> +   恢复绕过调用者链，直接跳转到叶子帧。
> +   
> +   类似`?`或`.await`的传播运算符不会生成任何粘合代码。
> +   
> +   `Waker`在执行效果之前不会构造或传递。
> +   
> +   类型系统防止处理程序恢复已完成的计算。
> +   
> Rust的协程帧本质上是这种方法的一种仿真，是由于语言缺乏尾调用而必需的。上述复杂情况是这种仿真的一部分，不是零成本效果的关键。

### 一种绕行：通过分层结合效果

由于这些降低是如此专业化，它们如何组合并不总是明显的。Haskell以一种与Rust相关的方式降低效果：效果计算是根据一个普通的语言内接口来定义的，使得降低对程序的其他部分可观察。这个接口，`Monad`类型类，由处理器而不是计算实现，并且具有Rust称为`and_then`方法。

> 另一个在这里不直接相关的有趣细节：虽然Rust的效果计算可以与任何处理程序一起重复使用，但单子程序本质上默认情况下固定了它们的处理程序选择。可以以不这样做的方式编写它们，例如使用`Monad`类型类的特定实现（称为“自由单子”），将所有`and_then`的调用保存在AST样式的结构中，以便稍后由任意处理程序解释。

对于具有多个效应的计算，Haskell使`and_then`方法对表示外部处理程序的“包装”单子通用化。这些泛型，或“单子变换器”，然后可以以任意方式嵌套。不幸的是，这引入了一种对效应集的任意排序，这影响了计算和处理程序。处理程序必须在他们的泛型`and_then`旁实现一个`lift`方法，并且计算在执行外部效应操作时必须使用适当数量的`lift`调用。

值得注意的是，这种分层方法与Rust的关键字泛型倡议所提出的方法非常相似。在那里，像`Iterator`这样的特征将变得通用，以适应其方法可能执行的任何附加效应，支持诸如`async Iterator`的实例化。这对处理程序有类似的不幸含义，处理程序必须使用适当数量的嵌套`map`调用来获取他们关心的效应的表示。

这种层次化也干扰了Rust对协程帧的降低。虽然Haskell的单子计算为每个可以恢复的地方传递一个新的闭包给处理程序，但Rust依赖于这些续延（表示为协程帧）被线性使用，以便可以原地修改它们。当计算一次降低一个效应时，每个效应都被强制产生一个单独的协程帧来满足API。每个帧都必须从下一个借用，并且在任何它借用的帧被恢复后必须重建。

当协程帧被用作特征对象时，情况变得更糟。一个具有效应的恢复方法返回链中下一个帧的相关类型——它应该分配在哪里？每个效应操作的恢复调用数量从每个效应操作增加到最坏情况下的总效应数量*乘以*效应操作数量——这些都是动态分派的吗？可以通过预留足够的空间来缓解这些成本，以一次性地静态调度整个链的帧，并同时单态化一个新的方法，该方法调度恢复和重建链，但这不是我们可以期望编译器自动执行的事情，因为它会改变可观察的行为和特征对象的类型。

处处都要处理这种复杂性，我们至少能得到一些好处吗？不——计算和处理程序都不能真正利用层次化，因为它没有语义含义，纯粹是降低的副产品。相反，如果我们因为任何原因想要它，我们总是可以从未分层的降低中恢复这种层次化，通过利用我们选择的任意顺序处理代数效应的能力！例如，我们可以仅用几行代码从降低的`poll_next`生成一个`async fn next`，反之亦然，从手写的`async fn next`生成一个`poll_next`。

> 这并不意味着带效果的特性在一般情况下是一个坏主意。`Read` 和 `Write` 等特性不代表协程帧，因此不会以完全相同的方式遇到这些问题。但本文讨论的是如何降低效果，而不是一般的 API 设计。

正是因为这种类型俄罗斯方块的情况，Haskell 生态系统已经发展出了几个旨在表示效果的库，而不需要这种分层，并且 Haskell 编译器最近还[支持](https://gitlab.haskell.org/ghc/ghc/-/merge_requests/7942)了限定续延的功能，以使这类库更加高效。Rust 应该从这项工作中学习，并跳过这种分层。

### 组合效果的降级

运行的计算会产生一系列从集合中抽取的效果。它的处理程序（或等效地，处理程序集合）必须准备好接受其中的任何一个。Rust 的单效果降级已经使用具有一种变体用于效果操作和另一种变体用于信号操作已完成的`enum`。这自然地扩展到具有每种效果一个变体的更大的`enum`，以及一个用于初始调用和最终返回的变体。

给定一组效果签名，它们的组合降级如下所示：

```
effect eff1 { op1(A) -> B; }
effect eff2 { op2(C) -> D; }
// ...

trait /* ... */ {
    type Output;
    fn resume(Pin<&mut Self>, Either<(), B, D>) -> Either<A, C, Self::Output>;
} 
```

> 这进一步倾向于不在类型级别强制执行此协议的设计选择。就像在`Iterator::next`返回`None`后仍然可以调用一样，在`resume`中传递错误的变体也是可能的。当然，可以通过日益复杂的类型系统机制（如ZST标记和存在性或生成式生命周期）来防止这种情况，但这在这里真的不是重点。

`AsyncIterator`（以前称为`Stream`）特性结合了`async`和`gen`效果，再次是这种模式的一个特例：

```
trait AsyncIterator {
    type Item;
    fn poll_next(Pin<&mut Self>, &mut Context<'_>) -> Poll<Option<Self::Item>>;
} 
```

读取返回类型`Poll<Option<Item>>`时可能会很诱人，尤其是当 Rust 程序员习惯于在手动编写`try`降级时简单地在`Result`中包装`Output`时，但这是一个红色警告。`Poll<Option<Item>>`与一个具有三种变体`Pending`（`sleep`操作）、`Some(Item)`（`yield`操作）和`None`（已完成计算）的枚举是同构的。通用的做法是添加更多的变体，而不是添加更多的层次。

同样，`impl Future<Output = Result>`的返回类型是`Poll<Result<Output, E>>`，它与三种变体`Pending`（`sleep`）、`Err(E)`（`throw`）和`Ok(Output)`（完成）是同构的。即使`impl Iterator<Item = Result>`的返回类型`Option<Result<Item, E>>`也与三个变体`Ok(Item)`（`yield`）、`Err(E)`（`throw`）和`None`（完成）是同构的，尽管这个迭代器产生的`Result`的类型冲突。

这个配方不仅仅是一个“上帝特性”，它不加区分地将我们需要的一切累积在一个地方。它是对效果和处理器的可交换语义更忠实的表示，避免了由分层引起的复杂性，并恢复了处理效果的灵活性，而不必考虑它们在一个无关的排序中的位置。

我们可能想要的其他变体，比如（不）固定或借出，它们本身并不是效果，而是涉及协程帧的效果的方面或模式。如果我们将符号扩展到包括显式的继续类型（类似于显式接收器类型，如`&self`），我们可以将这些变体表达为效果签名的一部分。

例如，我们可能会这样写未固定迭代效果：

```
effect gen<Item> {
    yield(Item) -> (&mut resume, ());
} 
```

类似地，我们可能会这样写一个固定的、借出的迭代效果：

```
effect gen<Item<'a>> {
    yield<'a>(Item<'a>) -> (Pin<&'a mut resume>, ());
} 
```

结合这些继续类型需要与添加变体不同的方法。因为`Pin`保证其目标保持固定，所以我们不能使用带有`&mut self`和`Pin<&mut Self>`变体的`enum`（尽管这在另一种语言中可能有效）。相反，我们需要一个单一的接收器类型，将所有效果的限制组合在一起。如果其中任何一个被固定，则使用`Pin`；如果其中任何一个是借出，则添加一个生命周期。例如，这正是为异步迭代器使用`Pin<&mut Self>`的理由-迭代效果签名不使用固定，但异步效果签名确实使用固定。

最后，使用单个协程帧还简化了设计`async Drop`或`poll_progress`等功能的设计，这些功能会对效果计算的控制流图执行有趣的操作。如果是分层帧的链条，那么需要处理器的合作才能在层之间传播这些信号，而单个组合的协程帧的状态则表示了一个地方的整个控制流图。

### 未来的可能性

关键字通用性倡议考虑分层方法的动机之一是希望避免特性的组合爆炸。非分层的降低已经减少了这个数量，因为它不区分顺序。但如果必要，它甚至可以比`async Iterator`方法更进一步，通过使用更为适度和更广泛适用的语言扩展，其中许多已经在夜间实现或已被反复提出。请允许我进行一些狂野的推测，并随意地处理语法。

由于Rust现有的效果特性已经是上面描述的模式的实例化，它们可以被赋予到或从类似于不稳定的`Coroutine`的通用特性的桥接`impl`。进一步地，它们可以被制作成该特性的别名，使用类似于不稳定的特性别名功能的东西。要兼容地执行此操作，还需要能够`impl`一个特性别名，以及能够重新映射关联类型和方法名称，方式与相同别名可以重新映射通用参数的方式相同。

用别名的组合爆炸来替换特质的组合爆炸足够好吗？也许！但是这些别名的外观可以用类似于`Fn`特质边界糖一样进行改进。不是使用像`AsyncIterator`这样的名称，通用特质的适当实例化可以使用其效果关键字来引用。例如，`async gen<I> ()`会意味着`AsyncIterator<Item = I>`，或者等效地说是`Coroutine<Yield = Pending + Some(Item) + None>`，而`try<E> async T`会意味着`Future<Output = Result<T, E>>`，或者等效地说是`Coroutine<Yield = Pending + Err(E) + Ok(T)>`。这样会对所有效果进行统一处理，而不像`async Iterator`将其中一个作为基本特质。

如何在多个实例之间共享组合器？这涉及到对一些变体进行泛化，同时具体指定其他变体。我们可以使用另一种语言扩展称为开放`enum`，有时也称为开放多态变体，并与现有的`#[non_exhaustive]`特性相关联。例如，这里是`Iterator::map`以任何具有`yield`效果的计算进行泛化，使用`Ts...`来声明开放`enum`泛型参数，并使用`A + Bs`来添加一个变体：

```
trait Coroutine {
    type Resume...;
    type Yield...;
    type Output;
    fn resume(Pin<&mut Self>, () + Self::Resume) -> Self::Yield + Self::Output;
}

impl<I: Coroutine, F, B> Coroutine for Map<I, F> where
    F: FnMut(I::Item) -> B,
{
    type Resume = I::Resume;
    type Yield = Option<B>::Some + I::Yield;
    type Output = I::Output;

    fn resume(self: Pin<&mut Self>, r: () + Self::Resume) -> Self::Yield + Self::Output {
        match self.project().iter.resume(r) {
            Some(item) => { Some((self.f)(item)) }
            y => { y }
        }
    }
} 
```

另一个函数可以在效果上多态的意义是支持有效果的闭包。这里是`Iterator::map`以这种方式进行泛化：

```
impl<I: Iterator, F, B: Coroutine> Coroutine for Map<I, F> where
    F: FnMut(I::Item) -> B,
{
    type Resume = () + B::Resume;
    type Yield = Option<B::Output>::Some + B::Yield;
    type Output = Option<B::Output>::None;

    fn resume(self: Pin<&mut Self>, r: () + Self::Resume) -> Self::Yield + Self::Output {
        let state = self.project().state;
        match state {
            State::Done => { panic!() }
            State::Iter => match self.project().iter.next() {
                None => { *state = State::Done; None }
                Some(item) => { *state = State::Call(f(item)); self.resume(()) }
            }
            State::Call(c) => match c.resume(r) {
                item: B::Output => { *state = State::Iter; Some(item) }
                y => { y }
            }
        }
    }
} 
```

当然，当我们有能力有选择性地处理效果时，没有必要手动编写这两者中的任何一个，因此这里同时以两种方式进行泛化，使用最后的一种语言扩展形式——`do`运算符来传播未知效果：

```
gen do {
    for do item in /* ... */ {
        yield f(item).do;
    }
} 
```

为了好玩，这里再次使用`handle`表达式：

```
handle /* ... */ {
    yield(resume, item) => handle f(item) {
        eff(resume, x) => { resume(eff(x)) }
        return(x) => { resume(yield(x)) }
    }
    eff(resume, x) => { resume(eff(x)) }
    return(x) => { x }
} 
```

目前，我仍然不确定这种普适性对Rust是否真的很重要。我的观点是，非分层效果降级策略（由`poll_next`表示）与减少特质蔓延的机制是向前兼容的，并且它比分层策略更直接地实现了这一点。毕竟，代数效果就是为此而设计的。

### 结论

代数效果是设计Rust这些特性的一个吸引人的框架，因为它们比单子变换器或有效果的协程特质更具有可组合性。它们的许多便利性都来自效果是*可交换*的，并且可以以任何顺序处理。像Haskell这样的语言已经涵盖了类似的地盘。我们可以从他们的经验中学习，以解锁像`async gen`和`for await`这样的特性，同时保留通向更大灵活性的路径。
