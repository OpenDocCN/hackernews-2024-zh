<!--yml

类别：未分类

日期：2024-05-27 13:32:38

-->

# Ruby might be faster than you think - John Hawthorn

> 来源：[https://www.johnhawthorn.com/2024/ruby-might-be-faster-than-you-think/](https://www.johnhawthorn.com/2024/ruby-might-be-faster-than-you-think/)

几周前，我看到一个项目允许在 Ruby 文件中内联写入和运行 Crystal 方法。

这是一个很棒的项目，我不想从中削减，但是 README 中的某些内容看起来对我来说有些不对劲。

require 'crystalruby' require 'benchmark' module Fibonnaci crystalize [n: :int32] => :int32 def fib_cr(n) a = 0 b = 1 n.times { a, b = b, a + b } a end module_function def fib_rb(n) a = 0 b = 1 n.times { a, b = b, a + b } a end end puts(Benchmark.realtime { 1_000_000.times { Fibonnaci.fib_rb(30) } }) puts(Benchmark.realtime { 1_000_000.times { Fibonnaci.fib_cr(30) } })

我的基准运行看起来与 README 中的类似。"水晶化" 版本比纯 Ruby 版本运行快大约 4 倍。

```
$ ruby test.rb
2.113752398872748
0.6535678738728166 
```

但这里有点不对劲。Ruby 实现中有一个细微的错误，导致它比必要的工作要多得多。

Ruby 的多重赋值 `a, b = b, a + b` 等同于 `a, b = [b, a + b]`。大多数情况下，这种数组分配实际上并没有发生，但因为在这种情况下它是块的最后一行，并且因为 Ruby 在块的末尾有一个隐式返回，所以数组是必需的（尽管 `Integer#times` 不使用返回，我们还没有一个能“知道”这一点的优化）。

让我们看看如何避免这种情况...（稍微有些不雅观的 `; nil` 替代返回）

def fib_rb(n) a = 0 b = 1 n.times { a, b = b, a + b; nil } a end

```
$ ruby test.rb
1.245176327880472
0.6385665240231901 
```

不错。我们正在弥合差距。现在我们只比水晶慢大约 2 倍。

为了让这个过程变得更快一点，我们可以内联循环，而不是调用 `Integer#times`。

def fib_rb(n) a = 0 b = 1 while n > 0 a, b = b, a + b n -= 1 end a end

```
$ ruby test.rb
0.7689670620020479
0.638991191983223 
```

现在几乎持平！最后，让我们启用 YJIT，Ruby 内置的 JIT 编译器，以查看真正的比较。

```
$ ruby --yjit test.rb
0.10502525512129068
0.5002881051041186 
```

现在 Ruby 的速度是 Crystal 的 5 倍！！！比我们原来的版本快了 20 倍。尽管很可能这是来自于 FFI 的一些成本，或者类似的东西，尽管这似乎是一个令人惊讶的开销量。

我认为值得注意的是，通过对 Ruby 代码进行一些小的调整，它现在可以在某些情况下胜过预编译的静态类型语言。我希望将来随着 Ruby JIT 技术的进步，甚至可能不再需要这些小调整。
