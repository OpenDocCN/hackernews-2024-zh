<!--yml

category: 未分类

date: 2024-05-27 13:00:23

-->

# 函数柯里化 - HaskellWiki

> 来源：[https://wiki.haskell.org/Currying](https://wiki.haskell.org/Currying)

函数柯里化是将一个接受元组作为参数的多参数函数转换为一个仅接受单个参数并返回另一个函数的过程，后者逐个接受原始函数在元组中接收的其余参数。

<haskell>```
f :: a -> (b -> c)     -- which can also be written as    f :: a -> b -> c 
```

是**柯里化**形式的

<haskell>你可以使用 Prelude 中的函数 <hask>`curry` 和 <hask>`uncurry` 在这两种类型之间进行转换。</hask></hask>

<haskell>```
f = curry g
g = uncurry f 
```

这两种形式同样具有表达能力。它确实成立

<haskell>然而，柯里化形式通常更方便，因为它允许进行[部分应用](/Partial_application "Partial application")。

在 Haskell 中，*所有* 函数都被视为柯里化的：也就是说，*Haskell 中的所有函数都只接受一个参数*。这在符号表示中大多是隐藏的，因此对于新手来说可能不明显。可以说类型符号中的箭头是向*右*关联的，所以 <hask>`f :: a -> b -> c` 实际上是 <hask>`f :: a -> (b -> c)`。相应地，函数应用是向*左*关联的： <hask>`f x y` 实际上是 <hask>`(f x) y`，因此类型吻合。</hask></hask></hask></hask>

作为例证，让我们看一个函数 <haskell>```
div :: Int -> Int -> Int    -- which is actually Int -> (Int -> Int) 
```

它执行整数除法。表达式 <hask>`div 11 2` 可以预计地计算为 <hask>`5`。

但这里发生的比初看起来更复杂。它可能是一个两部分的过程。

单独使用 <hask>`div 11` 是一个类型为 <hask>`Int -> Int` 的函数。然后，该结果函数可以应用到值 <hask>`2` 上，因此 <hask>`(div 11) 2` 得到 <hask>`5`。当然，优化编译器可能会一次处理整个表达式，但在概念上，这就是发生的事情。</hask></hask></hask></hask></hask>

你会注意到类型的符号表示反映了这一点：你可能会错误地将 <hask>`Int -> Int -> Int` 读作“接受两个 <hask>`Int` 并返回一个 <hask>`Int`”，但它*真正*表示的是“接受一个 <hask>`Int` 并返回类型为 <hask>`Int -> Int` 的东西”-- 也就是说，它返回一个接受一个 <hask>`Int` 并返回一个 <hask>`Int` 的函数。 （如果你确实指的是前者，可以写成 <hask>`Int x Int -> Int`，但由于 Haskell 中所有函数都是柯里化的，这在 Haskell 中是非法的。或者，可以使用元组，你可以写成 <hask>`(Int, Int) -> Int`，但要记住元组构造器 <hask>`(,)` 本身也可以被柯里化。）</hask></hask></hask></hask></hask></hask></hask></hask></hask></hask>

大部分情况下，新程序员可以忽略柯里化。将所有函数视为柯里化的主要优势在于理论上：当所有函数统一处理（一个参数进，一个结果出）时，形式化证明更容易。话虽如此，有*Haskell*惯用法和技术需要理解柯里化。

参见

## 练习

+   简化 `<hask>`curry id`</hask>`

+   简化 `<hask>`uncurry const`</hask>`

+   使用基本的 Prelude 函数和不使用 lambda 表达式，表达 `<hask>`snd` 使用 `curry` 或 `uncurry` 和其他基本 Prelude 函数</hask>`

+   写出函数 `<hask>`\(x,y) -> (y,x)` 不使用 lambda，仅使用 Prelude 函数
