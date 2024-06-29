<!--yml

category: 未分类

date: 2024-05-27 14:49:57

-->

# foreach循环 - 为什么Rust选择不提供`for`理解？ - 编程语言设计与实现Stack Exchange

> 来源：[https://langdev.stackexchange.com/questions/2848/why-does-rust-choose-not-to-provide-for-comprehensions](https://langdev.stackexchange.com/questions/2848/why-does-rust-choose-not-to-provide-for-comprehensions)

### Python列表理解

在Python中，有3个受到祝福的集合 -- `list`、`set`和`dict` -- 其中语法糖内建到语言中，允许直接创建其中之一。Python中可用的列表理解语法允许创建：

```
new_list = [i + 1 for i in old_list]
new_set = {i + 1 for i in old_set}
new_dict = {k:v+1 for (k, v) in old_dict.items()} 
```

当您使用它们时很棒... 当您希望另一个时则不是那么棒。

### Equality: Rust中没有二等公民！

Rust是一种系统编程语言。

在Python中，大多数开发者既不了解，也不关心`dict`在幕后是如何工作的：内存布局是什么？使用的哈希图类型是什么？使用的哈希算法是什么？

然而，系统程序员关心。他们可能关心是由于性能原因，也可能由于在他们的领域（嵌入式、内核）中缺乏内存分配。这意味着与Python开发者不同，他们更有可能需要专门的集合，而不是一般的标准集合。

而问题的关键在于：所有这些专门的集合都被看作是二等公民，而且比3个受到祝福的标准集合更加笨拙，这在Rust开发者中并不受欢迎。

因此，与其创建只能用于3个标准集合的列表理解语法，他们反而专注于通过Rust强大的特性系统使您正在寻找的操作适用于任何集合：

1.  `Iterator` trait允许迭代任何东西，并提供许多过滤/转换正在产出的元素的方法。

1.  `FromIterator` trait允许任何实现它的类型从`Iterator`构建。

因此，完全可以从一个集合创建另一个集合：

```
let new: Vec<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect();
let new: VecDeque<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect();
let new: LinkedList<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect(); 
```

*注意：由于可能的失败性问题，不可能*直接*创建数组，虽然也许应该提交一个RFC来允许收集到`Result<[_; _], _>`，因为那样可以处理失败性问题。*

迭代器链非常强大：take、skip、filter、filter_map、enumerate等等，因此在许多情况下，列表理解将是一种下降（它们只能映射和过滤）。

为了轻松的情况值得吗？嗯，到目前为止，来自Rust开发者的答案是他们觉得不值得。

* * *

顺便说一句，相同的迭代器链也可用于*扩展*现有集合以添加更多元素：

```
my_collection.extend([1, 2, 3, 4, 5].into_iter().map(|i| i + 1)); 
```

`extend`方法接受任何产生与集合相同类型元素的迭代器。

### 宏！

还应注意Rust有宏。

在 [crates.io](https://crates.io) 上快速搜索表明，一些 Rust 用户感到需要快速且简便的功能，于是他们前去创建了相应的宏，比如 [`list_comprehension_macro`](https://crates.io/crates/list_comprehension_macro) crate（以及其他很多）。来自 README：

```
let even_squares = comp![x.pow(2) for x in arr if x % 2 == 0];
let flatten_matrix = comp![x for row in arr for x in row];
let dict_comp = comp!{x: x.len() for x in arr}; 
```

反过来说，这也是不鼓励将任何此类功能包含在标准库中的因素之一，因为很容易自己制作，或者使用第三方库。

*注意：相对较小的标准库也是 Rust 开发者的一个显式目标。*
