<!--yml

类别：未分类

日期：2024-05-27 13:26:18

-->

# Ruby与Python的区别归结为for循环

> 来源：[https://softwaredoug.com/blog/2021/11/12/ruby-vs-python-for-loop.html](https://softwaredoug.com/blog/2021/11/12/ruby-vs-python-for-loop.html)

Ruby和Python之间的许多区别归结为`for`循环。

Python拥抱`for`。对象告诉`for`如何与它们一起工作，而`for`循环的主体处理对象返回的内容。Ruby则相反。在Ruby中，`for`本身（通过`each`）是对象的一个方法。调用者将for循环的主体传递给这个方法。

在习惯用法的Python中，对象模型适应于for循环。在Ruby的情况下，for循环适应于对象模型。

在Python中，如果你希望自定义迭代，一个对象告诉语言应该如何迭代它：

```
class Stuff:
    def __init__(self):
        self.a_list = [1,2,3,4]
        self.position = 0
    def __next__(self):
        try:
            value = self.a_list[self.position]
            self.position += 1
            return value
        except IndexError:
            self.position = 0
            raise StopIteration
    def __iter__(self):
        return self 
```

这里的`Stuff`使用方法`__next__`和`__iter__`使自己可迭代。

```
for data in Stuff():
    print(data) 
```

然而，在习惯用法的Ruby中，你做的事情完全相反。你将`for`本身创建为一个方法，并且它接受要运行的代码（主体）。Ruby将过程代码放入块中，以便可以将其传递。然后在你的`each`方法中，你使用`yield`与块交互，将值传递到块中以执行需要的操作（块在任何方法上都是一种隐式参数）。

如果我们重写上面的代码，它会是这样的

```
class Stuff
  def initialize
    @a_list = [1, 2, 3, 4]
  end

  def each
    for item in @a_list
      yield item
    end
  end
end 
```

使用`each`进行迭代：

```
Stuff.new().each do |item|
  puts item
end 
```

不是将数据传递回for循环（Python），而是将代码传递给数据（Ruby）。

但问题不仅如此：

Python基于类似`for`的构造来进行各种处理；Ruby将其他类型的数据处理工作推到方法中。

Pythonic代码使用列表和字典推导式来实现map和filter，其核心是相同的for/迭代语义。

```
In [2]: [item for item in Stuff()]
Out[2]: [1, 2, 3, 4]

In [3]: [item for item in Stuff() if item % 2 == 0]
Out[3]: [2, 4] 
```

Ruby继续沿着首先方法的方法前进，除了`each`之外，我们还有一组通常在集合上实现的新方法，如下所示：

```
class Stuff
  ...

  def select
    out = []
    each do |e|
      # If block returns truthy on e, append to out
      if yield(e)
        out << e
      end
    end
    out
  end

  def map
    out = []
    # One line block syntax, append output of block processed on e to out
    each {|e| out << yield(e) } 
    out
end 
```

```
puts Stuff.new().map {|item| item}
puts Stuff.new().select{|item| item.even?} 
```

Python说“你告诉我们如何迭代你的实例，我们决定如何处理你的数据。” Python有一些基于语言的迭代和处理原语，要定制该迭代，我们只需在for循环的（或表达式）主体中添加正确的代码。

Ruby颠覆了脚本，使对象具有更深层次的可定制性。是的，在某些情况下，我们可以简单地在块内部添加更多的控制流。是的，我们可以弯曲我们对`each`的使用以基本上做`map`。但是Ruby让对象能够提供不同的`map`和`each`实现（也许`each`的实现如果用于`map`将是非常次优的，甚至是不安全的）。Ruby对象可以更加前卫地处理其数据的最佳方法。

在Ruby中，对象控制可能性。在Python中，语言控制。

Python 的成语风格对数据处理有着强烈的见解。Python 说：“看，你的 90% 代码将很好地符合这些思想，只要遵循它并完成你的工作。”只需使您的对象可迭代并结束我的困扰。但是 Ruby 说：“有些重要情况下，我们不想给调用者那么大的权力。”因此，Ruby 鼓励对象控制它们如何被处理，开发人员被鼓励按照对象希望互动的方式行事。Ruby 选择了更少关于数据的意见的表达方式。

Python 更像是 C-基础“面向对象”编程的扩展。在基于 C 的 OO 中，例如[posix 文件描述符](https://en.wikipedia.org/wiki/File_descriptor)或[Win32 窗口句柄](https://stackoverflow.com/questions/2334966/win32-application-arent-so-object-oriented-and-why-there-are-so-many-pointers)，语言不强制将“方法”与对象本身捆绑在一起。相反，对象到方法的捆绑是出于约定。Python 认为这种过程式世界可以进化 - 它将这种思维升级以使其更安全。自由函数存在，并且确实经常比方法更受鼓励。对象存在，但方式更为犹豫。方法将“self”作为其第一个参数接受，几乎与 Win32 或 Posix API 中的 C 函数相同，后者接受句柄。在传递函数时，它们几乎像 C 函数指针一样对待。过程式范式优先，并作为一切的关键基础，面向对象的语义则层叠在其上。

Ruby，然而，完全颠覆了这一点。Ruby 将面向对象作为金字塔的基础。Ruby 将混乱的过程式世界放在块中，让对象与这些过程式块一起工作。与其按照语言的过程式基础来破坏对象，Ruby 让过程式代码适应对象对世界的视角。Ruby 有真正的私有成员，不像 Python，后者仅通过约定具有私有方法/参数。

不难理解为什么当我从系统编程的角度来看 Python 时，它对我的大脑感觉如此自然。它进化并使那个世界更安全，有能力在需要时编写 C 代码。也许这就是它在系统资源密集型数值计算空间找到家园的原因。

不难理解为什么 Ruby 对于构建更流畅、也许更安全的 API 和 DSL 的开发人员来说是一个自然选择。Ruby 希望程序员模仿领域，而不是编程环境，对许多工作来说，这似乎是正确的方法。

像我这样的搜索开发人员，在[一个 Ruby 店铺](http://engineering.shopify.com)工作，需要应对这些差异。也许你想加入我，来体验这场 Ruby-Python-搜索冒险？那么，也许你会[申请这份工作](https://jobs.smartrecruiters.com/ni/Shopify/bedf9119-9a23-4fd3-8d8a-7fcbf168eca9-senior-relevance-engineer-search-discovery) :-p

特别感谢 [Felipe Besson](https://felipebesson.com/)、[Simon Eskildsen](https://sirupsen.com/) 和 [John Berryman](http://blog.jnbrymn.com/) 对本文进行审阅，并提供了实质性的编辑和反馈！
