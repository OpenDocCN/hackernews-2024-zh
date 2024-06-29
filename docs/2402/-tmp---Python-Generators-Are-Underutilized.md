<!--yml

类别：未分类

日期：2024 年 5 月 29 日 13:20:18

-->

# /tmp | Python 生成器被低估了

> 来源：[https://www.slashtmp.io/posts/generators/](https://www.slashtmp.io/posts/generators/)

<main>

#### 2024 年 1 月 12 日

# Python 生成器被低估了

在 Python 生态系统中我最喜欢的功能之一是一个我并不经常看到被利用的功能：

*生成器*

## 简介

Python 生成器 *在达到 yield 语句时* 生成值。与循环不同，生成器一次生成一个值。考虑以下代码片段：

```
def square_vals(x : list):
    return [val * val for val in x]

nums_to_square = list(range(500))
print(square_vals(nums_to_square)) 
```

这将产生以下输出：

```
[0, 1, 4, 9, 16, 25, 36, ...] 
```

让我们引入一个叫做 [tracemalloc](https://docs.python.org/3/library/tracemalloc.html#tracemalloc.get_traced_memory) 的实用库。这是一个 Python 内置库，带有一些非常简单的用于跟踪 Python 程序内存分配的实用工具。让我们修改我们的程序来跟踪为整个程序分配的内存量：

```
def square_vals(x : list):
    to_return = [val * val for val in x]
    return to_return

tracemalloc.start()
nums_to_square = list(range(500))
squares = square_vals(nums_to_square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

这将输出：

```
[<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<2>>,)> size=19616 count=484>,
 <Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<5>>,)> size=11832 count=245>]
<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=7>,)> size=64 count=2>]
'Peak usage:31648 
```

这里我们看到了每一行分配的内存量，以及峰值内存。因此，在这个代码块中，我们进行了以下分配：

+   `py to_return = [val * val for val in x]` 分配了 19616 字节

+   `py squares = square_vals(nums_to_square)` 分配了 11832 字节

+   `py to_return = initial_mem, peak_mem = tracemalloc.get_traced_memory()` 分配了 64 字节

在这个程序中，我们总共利用了 31,468 字节的内存。

现在添加一个生成器。

考虑以下代码：

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()

nums_to_square = list(range(500))
squares = gen_square_vals(nums_to_square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

这将输出：

```
[<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<7>>,)> size=11932 count=484>,
 <Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<8>>,)> size=208 count=245>]
<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=9>,)> size=64 count=2>]
'Peak usage:12040 
```

这很有趣！注意，与我们的第一个示例相比，提供平方数 `yield val * val` 的行为实际上不会导致任何内存分配，而 `to_return = [val * val for val in x]` 则会。相反，在后一个代码块中，我们在调用 `gen_square_values()` 的行上最终会分配 208 字节的内存。这就是 Python 生成器的美妙之处！

那么这里发生了什么？

在我们的第一个示例中，当我们调用 `square_vals()` 时，我们在内存中创建包含所有平方值的列表，并从方法返回该内存块。当我们将其转换为生成器时，我们实际上创建了一个 *生成器对象*。这个生成器对象会实时生成值，也就是说：直到我们准备使用它们之前，我们不会分配任何平方值。

如果我们回到我们的代码块，通过添加一个新的循环，我们可以看到它的运行情况：

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()
nums_to_square = list(range(500))
squares = gen_square_vals(nums_to_square)
gen_to_list = [square for square in squares]
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

这个代码块只是将我们的生成器取出，并将所有值 yield 到一个名为 `gen_to_list` 的列表中。当我们运行这个代码时，我们会得到额外的输出：

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=5>,)> size=15456 count=483>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=2>,)> size=11832 count=245>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=4160 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=6>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=64 count=2>]
'Peak usage:31856' 
```

我们成功地将我们的峰值内存使用量提高了！请注意，生成值的行再次分配了 11832 字节，当我们将所有值 yield 到列表中时。这揭示了生成器的基本功能：它作为一个小型、轻量级的对象存在，*逐个生成* 值。

## 扩展

那么这里的实用性是什么？当我们稍微改变一下事物时，这就变得更加明显了。让我们修改我们的代码，生成 100,000 个数字而不是 500 个。另外，我将修改以下行：

```
nums_to_square = list(range(100000)) 
```

至：

```
nums_to_square = range(100000) 
```

在我们的生成器示例中。（Python 中的 range 内置函数在行为上与生成器非常相似，但略有不同）

使用列表运行我们的示例给出以下基准：

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=6>,)> size=4000384 count=99984>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=14>,)> size=3991832 count=99745>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=16>,)> size=64 count=2>]
'Peak usage:7992416' 
```

对于不做太多工作的 7,992,416 个字节来说，这是很多的字节。我们已经使用了 7MB 的内存。让我们来看一个生成器的实现：

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=80 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=9>,)> size=64 count=2>]
'Peak usage:288' 
```

288 字节？好吧，但我们没有对这些值做任何事情，让我们实际对生成器做些事情：

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()

nums_to_square = range(100000)
squares = gen_square_vals(nums_to_square)
for square in squares:
    print(square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

这将打印：

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=80 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=11>,)> size=64 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=3>,)> size=32 count=1>]
'Peak usage:784' 
```

这仍然是微不足道的！

## 为什么？

显然，这只是使用生成器将数据输出到控制台的一个极为简单的示例，但是生成器在许多应用中非常有用。Python 常被用作 ETL 流水线和批处理作业的粘合语言，许多工程师通过在其应用程序中创建大型列表对象来实现这些工作。考虑一个我们解析成 Python 对象的简单 JSON 模式：

```
{
  "user_id": 123456789,
  "username": "john_doe",
  "full_name": "John Doe",
  "email": "john.doe@example.com",
  "age": 30,
  "gender": "male",
  "country": "United States",
  "city": "New York",
  "occupation": "Software Engineer",
  "education": "Bachelor's Degree",
  "interests": ["programming", "hiking", "reading"],
  "languages": ["English", "Spanish"],
  "last_login": "2024-02-22T08:30:00Z",
  "subscription": {
    "type": "premium",
    "start_date": "2023-01-01",
    "end_date": "2024-12-31"
  }
} 
```

如果我们正在摄取成千上万的这些对象，将它们全部存储在内存中会很快变得昂贵。生成器使我们能够避免一次性在内存中创建所有这些对象。在减少成本和提高内存有限的云计算环境中的性能方面，这可能是非常宝贵的。

## 注意事项

尽管方便，生成器确实有一些必须在使用它们之前考虑的限制：

### 生成器不能在没有额外代码的情况下倒回或查看。

一旦一个值被生成，它就不能从同一个生成器中重新生成。同样，没有办法计算生成器的下一个值而不消耗它。

### 生成器可能很难调试。

逐行调试器通常很难检查底层对象，因为它们是不可倒回的。我目前不知道任何 Python 可视化调试应用程序，允许检查生成的值而不消耗该值。可以通过在生成器上实现自定义代码来解决这个问题。

### 生成器可能增加了理解代码的心理负担。

生成器的低利用率意味着它们对于那些对 Python 经验有限的人来说通常很难理解。人们可能需要一些时间来开发“当迭代时，这实际上变成了一个列表”的心理抽象。最初，明显的“无序”执行可能很难解析。拿这个例子来说：

```
def do_hello():
    print("hello")
    yield 5
    print("world")

hi = do_hello()
print("goodbye")
for test in hi:
    pass 
```

如果没有对生成器有扎实的了解，解析此脚本中输出的顺序可能会让人困惑。

</main>
