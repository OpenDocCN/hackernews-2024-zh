<!--yml

category: 未分类

date: 2024-05-27 13:10:34

-->

# Ruby 元编程：一切都在于 Self

> 来源：[https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/](https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)

在写完我上一篇关于 Rails 插件惯用法的文章后，我意识到 Ruby 元编程实际上非常简单。

核心问题在于，所有的 Ruby 代码都是执行代码--没有单独的编译或运行时阶段。在 Ruby 中，每一行代码都是针对特定的 `self` 执行的。考虑以下五个片段：

```
class Person
  def self.species
    "Homo Sapien"
  end
end

class Person
  class << self
    def species
      "Homo Sapien"
    end
  end
end

class << Person
  def species
    "Homo Sapien"
  end
end

Person.instance_eval do
  def species
    "Homo Sapien"
  end
end

def Person.species
  "Homo Sapien"
end 
```

所有这五个片段定义了一个 `Person.species`，返回 `Homo Sapien`。现在考虑另一组片段：

```
class Person
  def name
    "Matz"
  end
end

Person.class_eval do
  def name
    "Matz"
  end
end 
```

所有这些片段都定义了一个叫做 `name` 的方法，并且在 `Person` 类上。所以 `Person.new.name` 将返回 "Matz"。对于熟悉 Ruby 的人来说，这不是新闻。在学习元编程时，每个片段都是单独呈现的：另一种机制来获取方法的所属位置。然而，事实上，所有这些片段之所以工作方式如此，有一个统一的原因。

首先，理解 Ruby 的元类是非常重要的。当你第一次学习 Ruby 时，你会学习类的概念，以及 Ruby 中每个对象都有一个：

```
class Person
end

Person.class #=> Class

class Class
  def loud_name
    "#{name.upcase}!"
  end
end

Person.loud_name #=> "PERSON!" 
```

`Person` 是 `Class` 的一个实例，因此任何添加到 `Class` 上的方法也可以在 `Person` 上使用。然而，他们没有告诉你的是，Ruby 中的每个对象也有自己的**元类**，一个可以有方法的 `Class`，但仅附加到对象本身。

```
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end 
```

这里发生的情况是，我们正在向 `matz` 的**元类**添加 `speak` 方法，而 `matz` 对象继承自其**元类**，然后继承自 `Object`。之所以不够清晰，是因为在 Ruby 中，元类是看不见的：

```
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end

matz.class #=> Object 
```

实际上，`matz` 的 "class" 就是它看不见的元类。我们甚至可以访问这个元类：

```
metaclass = class << matz; self; end
metaclass.instance_methods.grep(/speak/) #=> ["speak"] 
```

在这一主题的其他文章中，你可能正在努力记住所有的细节；似乎有这么多的规则。那么 `class << matz` 到底是什么呢？

事实证明，所有这些奇怪的规则都归结为一个概念：在代码的某个部分控制 `self`。让我们回顾一下之前看过的一些片段：

```
class Person
  def name
    "Matz"
  end

  self.name #=> "Person"
end 
```

在这里，我们正在向 `Person` 类添加 `name` 方法。一旦我们说 `class Person`，直到块的结束，`self` 就是 `Person` 类本身。

```
Person.class_eval do
  def name
    "Matz"
  end

  self.name #=> "Person"
end 
```

在这里，我们正在做完全相同的事情：向 `Person` 类的实例添加 `name` 方法。在这种情况下，`class_eval` 将 `self` 设置为 `Person`，直到块结束。在处理类时，这一切都非常直截了当，但在处理元类时同样如此：

```
def Person.species
  "Homo Sapien"
end

Person.name #=> "Person" 
```

就像之前的`matz`示例中一样，我们在`Person`的元类上定义了`species`方法。我们没有操纵`self`，但是您可以看到在对象上使用`def`附加方法到对象的元类。

```
class Person
  def self.species
    "Homo Sapien"
  end

  self.name #=> "Person"
end 
```

在这里，我们打开了`Person`类，将`self`设置为`Person`，持续整个块的时间，就像上面的例子一样。但是，在这里我们在`Person`的元类上定义了一个方法，因为我们在对象（`self`）上定义了方法。此外，您可以看到，在`person`类内部使用`self.name`与在外部使用`Person.name`是相同的。

```
class << Person
  def species
    "Homo Sapien"
  end

  self.name #=> ""
end 
```

Ruby提供了一种直接访问对象元类的语法。通过`class << Person`，我们将`self`设置为`Person`的元类，持续整个块的时间。因此，`species`方法被添加到`Person`的元类中，而不是类本身。

```
class Person
  class << self
    def species
      "Homo Sapien"
    end

    self.name #=> ""
  end
end 
```

在这里，我们结合了几种技术。首先，我们打开了`Person`，使得`self`等于`Person`类。接下来，我们执行了`class << self`，使得`self`等于`Person`的元类。然后我们定义了`species`方法，该方法被定义在`Person`的元类上。

```
Person.instance_eval do
  def species
    "Homo Sapien"
  end

  self.name #=> "Person"
end 
```

最后一种情况，`instance_eval` 实际上做了一些有趣的事情。它将`self`分解为用于执行方法的`self`和用于定义新方法时的`self`。当使用`instance_eval`时，新方法被定义在**元类**上，但`self`是对象本身。

在某些情况下，通过多种方式实现相同结果自然而然地出现在Ruby语义中。在这个解释之后，应该清楚`def Person.species`，`class << Person; def species`和`class Person; class << self; def species`并不是**设计上**达到相同目的的三种方式，而是由于Ruby在程序的任何给定点上指定`self`的灵活性而产生的。

另一方面，`class_eval` 有些许不同。因为它接受一个块而不是作为关键字，它捕获周围的局部变量。这不仅可以提供强大的DSL功能，还可以控制代码块中使用的`self`。但除此之外，它们与这里使用的其他结构完全相同。

最后，`instance_eval`将`self`分解为两部分，并允许您访问其外部定义的局部变量。

在下表中，*定义新的作用域* 意味着块内的代码**不能**访问块外的局部变量。

| 机制 | 方法解析 | 方法定义 | 新作用域？ |
| --- | --- | --- | --- |
| class Person | Person | 相同 | 是 |
| class << Person | Person 的元类 | 相同 | 是 |
| Person.class_eval | Person | 相同 | 否 |
| Person.instance_eval | Person | Person 的元类 | 否 |

注意，`class_eval` 仅适用于 `Modules`（请注意，Class 继承自 Module），并且是 `module_eval` 的别名。此外，`instance_exec` 在 Ruby 1.8.7 中添加，与 `instance_eval` 的作用完全相同，唯一的区别是它还允许你将变量传递到块中。

**更新：** 感谢 Ruby 核心团队的 Yugui [纠正原始发布内容](http://yugui.jp/articles/846?ref=yehudakatz.com)，该内容忽略了在 `instance_eval` 的情况下 `self` 被分为两部分这一事实。
