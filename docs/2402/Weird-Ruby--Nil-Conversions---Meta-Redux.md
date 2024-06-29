<!--yml

类别：未分类

日期：2024-05-29 13:20:45

-->

# 奇怪的 Ruby：空值转换 | Meta Redux

> 来源：[https://metaredux.com/posts/2024/02/23/weird-ruby-nil-conversions.html](https://metaredux.com/posts/2024/02/23/weird-ruby-nil-conversions.html)

大多数 Ruby 程序员可能都知道，在 Ruby 中 `nil` 是 [NilClass](https://ruby-doc.org/3.3.0/NilClass.html) 的一个实例。因为 `nil` 和其他任何对象一样，我们实际上可以在其上调用方法，而 `NilClass` 提供了一些方法，主要是转换到其他 Ruby 类型的方法。让我们看看其中一些的作用：

```
irb(main):001:0> nil.to_h
=> {}
irb(main):002:0> nil.to_a
=> []
irb(main):003:0> nil.to_s
=> ""
irb(main):004:0> nil.to_f
=> 0.0
irb(main):005:0> nil.to_i
=> 0
irb(main):006:0> nil.to_c
=> (0+0i) 
```

使用这些方法，你可以从无中创造出一些东西（例如一个空的哈希表）！但是否真的应该这样做则是另外一个完全不同的问题。根据类文档：

> 转换方法将空值概念传递给其他类。

我不确定“空值概念”如何定义，但我一直觉得将缺少值等同于某些锚定值（例如空集合或0）很奇怪。我知道在某些情况下这可能很有用，但我非常确定你可以在不诉诸这种转换的情况下获得类似的结果。

记住，你可以用不同的方式进行类似的转换：

```
irb(main):001:0> Array(nil)
=> []
irb(main):002:0> Hash(nil)
=> {}
irb(main):003:0> Integer(nil)
# `Integer': can't convert nil into Integer (TypeError)
irb(main):004:0> Float(nil)
# `Float': can't convert nil into Float (TypeError) 
```

注意，“安全”的数值转换方法 `Integer` 和 `Float` 在处理 `nil` 时会有不同的操作，并在传递 `nil` 时引发异常。这也是为什么经常建议避免使用 `to_i` 和 `to_f` 的一部分原因，以避免转换意外的值。

[Python 之禅](https://peps.python.org/pep-0020/) 著名地说：

> 明确胜于隐晦。
> 
> 错误永远不应该悄悄地通过。
> 
> 面对歧义，拒绝猜测的诱惑。

但在 Ruby 的世界中，似乎我们喜欢过于冒险！

今天就告诉你这些了。让 Ruby 保持奇异！
