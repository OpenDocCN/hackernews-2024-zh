<!--yml

分类：未分类

日期：2024-05-29 12:45:32

-->

# 超越Rust模式匹配的一步:: Radiki Dev — Chris Gioran的博客

> 来源：[https://radiki.dev/posts/match-and-bind-patterns/](https://radiki.dev/posts/match-and-bind-patterns/)

### 如果你不需要刷新基本的Rust模式匹配，请直接转到[新内容](#glowdust) [#](#if-you-dont-need-a-refresh-of-basic-rust-pattern-match-go-directly-to-the-new-stuffglowdust)

## Rust模式匹配的一个非常主观的提醒 [#](#a-very-opinionated-reminder-of-patterns-in-rust)

这里是Rust中的一个简单模式匹配：

相当反戏剧性，是吧？好吧，让我加点调味：

还不够好？好吧，让我们试试更花哨的东西：

```
let (a, 2) = (1, 2); // Does not compile  error[E0005]: refutable pattern in local binding  --> src/example.rs:3:9 | 3 |     let (a, 2) = (1, 2);  |         ^^^^^^ patterns `(_, i32::MIN..=1_i32)` and `(_, 3_i32..=i32::MAX)` not covered | = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant 
```

哇，事情升级得真快。

现在，`rustc`和我之间有分歧，但在这种情况下，我也能看到点端倪：这种模式在理论上可能会失败。我们渺小的人类眼睛可能看不到它 - 实际上，你可能会认为“嘿，我很确定2等于2。怎么可能失败？”

好点。但如果它是

```
let (a, 2) = (1, 1+1); // Does not compile either 
```

或者其他一些更复杂的表达式？不行，`rustc`说，只要存在等式（例如，在左手边有一个常量），我就不会无条件地让你这么做。

但是*有条件*？没问题。

那么让我们做吧

```
if let (a, 2) = (1, 2) {  println!("Made it, a is {}", a); }   // prints "Made it, a is 1" 
```

很好。我可以在常量上进行模式匹配并捕获变量。让我们做一些更复杂的事情

```
if let (a, 2) = (1, 2) &&  let (b, 4) = (3, 4) && println!("Made it, a = {}, b = {}", a, b); }   // prints "Made it, a = 1, b = 3" 
```

很好，我可以在模式匹配和变量绑定之间进行条件处理。上面的例子很傻，因为模式总是匹配的，但这里重要的是语法。

我甚至可以在此基础上构建，为已绑定的变量添加条件：

```
if let (a, 2) = (1, 2) &&  let (c, 4) = (3, 4) && b == a + 2 { println!("Made it, a = {}, b = {}", a, b); }   // still prints "Made it, a = 1, b = 3" 
```

## 那就是所有麻烦的开始 [#](#glowdust)

看到这一点，我忍不住觉得这看起来很像一个声明性查询……东西？

假设我有一个成对的向量：

```
let the_data = vec![  (1, 2),  (2, 2)  (2, 3),  (5, 7),  (8, 8)]; 
```

我可以从中抓取一个迭代器，并在for循环中进行相同的模式匹配：

```
for (a, b) in the_data {  println!("({}, {})", a, b); }   // prints // (1, 2) // (2, 3) // (5, 7) // (8, 8) 
```

这很琐碎，但显然也是一种模式匹配。

*我想在匹配本身中有条件地进行。*

让我们试试用`while`而不是`for`

```
while let Some((a, b)) = iterator.next() && *b == *a + 1  {  println!("({}, {})", a, b); }   // prints // (1, 2) // (2, 3) 
```

好吧，还不够好？好吧，让我们尝试一些更复杂的东西：

```
while let Some((a, a + 1)) = iterator.next() { // How cool would it be if this worked?  println!("({}, {})", a, b); } 
```

遗憾的是，它不起作用：

```
error: expected a pattern, found an expression  --> src/example.rs:11:24 | 11 |     while let Some((a, a + 1)) = iterator.next() {  |                        ^^^^^ arbitrary expressions are not allowed in patterns 
```

现在，我不是一个语言设计师，对Rust内部几乎一无所知（坦率地说，我对Rust了解甚少）。我不能说允许这样做会破坏语言中的其他东西，或者这是一个众所周知的坏主意。

幸运的是，我的无知是我的盾牌，我完全没有问题地让这个工作在我自己的语言中。

## Glowdust中的模式匹配 [#](#pattern-matching-in-glowdust)

Glowdust中的算法很简单。

就像在Rust中一样，如果模式的左手边有一个未绑定变量，那么它将绑定到右手边的任何内容。从那时起，它就被绑定了，并且像Rust一样进行匹配。

*（注：我说左手/右手边，但在Glowdust语法中，->运算符会交换这两边。在接下来的例子中请记住这一点）*

不同之处在于它在同一个模式中同时执行。这里有一个在Glowdust中的例子。

首先定义一个包含我们数据的函数：

然后使用与上面迭代器示例相同的数据填充并查询：

```
my(1) = (1, 2); my(2) = (2, 3); my(3) = (5, 7); my(4) = (8, 8);   match my(x) -> (y1, y1 + 1), return y1 
```

这按预期工作：

但我们不要止步于此。让我们使用这个能力来进行连接：

```
match my(_) -> (left, middle), my(_) -> (middle, right), return left, middle, right 
```

正是因为第一次遇到变量时它是绑定的，然后它作为一个*可反驳的*匹配使用。

在这种情况下，`middle`被绑定在外部循环中，然后它的值被用作内部循环中可反驳的模式。

它还具有非常声明式、模式匹配的感觉，但非常熟悉和可读。它隐含的谓词可以根据数据存储中的基数和其他统计数据进行移动和优化。它们甚至可以被推送到存储层，如果计算存储可用的话。

我提到过它能工作吗？你可以立即在[Glowdust](https://codeberg.org/glowdust/glowdust)中运行这些例子。

## 按示例查询并没有走得太远[#](#query-by-example-didnt-go-far-enough)

这可能让您想起MS Access或Hibernate中的按示例查询。

它并不是。

它是按模式查询，这样更酷。

这编译成了适当的字节码，而不仅仅是DSL。您可以在完全表达式中重用刚刚进入作用域的变量，将它们连接起来并在后续表达式中使用，如过滤器和（最终）聚合。

我认为与Rust（和其他语言的）模式匹配的比较很有意思，但我绝不会认为它更好。然而，它更适合作为数据库查询语言。

这非常幸运，因为这就是我正在构建的。

* * *

一如既往，请告诉我你对[Mastodon](https://fosstodon.org/@chrisg)的想法。

如果您觉得这足够有趣，您可能想要[捐赠](https://liberapay.com/chris.gioran/)以支持Glowdust的开发成本。
