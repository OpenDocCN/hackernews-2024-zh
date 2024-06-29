<!--yml

category: 未分类

date: 2024-05-27 14:35:17

-->

# 不使用生命周期进行借用检查 · baby steps

> 来源：[https://smallcultfollowing.com/babysteps/blog/2024/03/04/borrow-checking-without-lifetimes/](https://smallcultfollowing.com/babysteps/blog/2024/03/04/borrow-checking-without-lifetimes/)

本文探讨了Rust类型系统的另一种表述，它放弃了生命周期而采用*位置*。简而言之，与其让`'a`在代码中代表*生命周期*，不如让它代表一组*贷款*，比如`shared(a.b.c)`或`mut(x)`。如果这听起来很熟悉，那是因为它是[polonius](https://smallcultfollowing.com/babysteps/blog/2023/09/22/polonius-part-1/)的基础，但重新定义为类型系统而不是静态分析。本文只是提供高层次的思想。在后续文章中，我将深入探讨如何利用这一点来支持内部引用和其他高级借用模式。在实现方面，我已经做了一些模拟，但我打算开始扩展[a-mir-formality](https://github.com/rust-lang/a-mir-formality)以包含这一分析。

## 为什么要替换生命周期？

生命周期是Rust的优点和缺点。优点是它们能让你表达非常酷的模式，比如在数据结构中返回指向数据中间某些数据的指针。但它们也存在一些严重问题。首先，生命周期的概念相当抽象，人们难以理解（“`'a`究竟代表什么？”）。而且Rust无法表达一些重要的模式，尤其是内部引用，其中结构体的一个字段引用了另一个字段拥有的数据。

## 那么生命周期究竟是什么？

这里是来自非词法生命周期RFC的生命周期定义：

> 每当创建一个借用时，编译器都会为生成的引用分配一个生命周期。这个生命周期对应于引用可以使用的代码范围。编译器会推断这个生命周期，使其尽可能小，同时涵盖引用的所有使用。

[阅读RFC获取更多详情。](https://rust-lang.github.io/rfcs/2094-nll.html#what-is-a-lifetime)

## 将*生命周期*替换为*起源*

在这种表述下，`'a`不再表示*生命周期*，而是一个**起源**，即解释引用可能来自何处。我们定义一个起源为**贷款集合**。每笔贷款都捕获了一些**位置表达式**（例如`a`或`a.b.c`），以及它们被借用的模式（`shared`或`mut`）。

```
Origin = { Loan }

Loan = shared(Place)
     | mut(Place)

Place = variable(.field)*  // e.g., a.b.c 
```

## 定义类型

使用起源，我们可以大致定义Rust类型如下（显然我在这里忽略了一堆复杂性&mldr;）。

```
Type = TypeName < Generic* >
     | & Origin Type
     | & Origin mut Type

TypeName = u32 (for now I'll ignore the rest of the scalars)
         | ()  (unit type, don't worry about tuples)
         | StructName
         | EnumName
         | UnionName

Generic = Type | Origin 
```

第一件有趣的事情需要注意：这里没有`'a`标记！这是因为我还没有介绍泛型。与Rust不同，这种类型系统的具体语法(`Origin`)为`'a`代表的内容定义了一个具体的语法。

## 简单程序的显式类型

拥有完全明确的类型系统意味着我们可以轻松地编写所有类型完全指定的示例程序。这曾经是相当具有挑战性的，因为我们没有生命周期的表示法。让我们看一个简单的例子，一个应该会出错的程序：

```
let  mut  counter: u32 =  22_u32; let  p: & /*{shared(counter)}*/  u32  =  &counter; //       --------------------- //       no syntax for this today! counter  +=  1;  // Error: cannot mutate `counter` while `p` is live println!("{p}"); 
```

除了 `p` 的类型之外，这是有效的 Rust。当然，它不会编译，因为在存在活跃的共享引用 `p` 时，我们无法修改 `counter`（[playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1a05f0a4aad12c33345ca4adc1cd9bb2)）。随着我们继续，你将看到新的类型系统形式化如何得出相同的结论。

## 基本的类型判断

类型判断是描述类型系统的标准方式。我们将分阶段引入我们系统的类型判断。我们将从一个不包括借用检查的简单且相当标准的形式开始，然后展示如何引入借用检查。对于这个第一个版本，我们正在定义的类型判断的形式是：

```
Env |- Expr : Type 
```

这个说法是，“在环境 `Env` 中，表达式 `Expr` 是合法的，且具有类型 `Type`”。这里的*环境* `Env` 定义了作用域内的局部变量。我们用于我们的[示例程序](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1a05f0a4aad12c33345ca4adc1cd9bb2)的 Rust 表达式非常简单：

```
Expr = integer literal (e.g., 22_u32)
     | & Place
     | Expr + Expr
     | Place (read the value of a place)
     | Place = Expr (overwrite the value of a place)
     | ... 
```

由于我们仅支持一种标量类型（`u32`），因此 `Expr + Expr` 的类型判断非常简单：

```
Env |- Expr1 : u32
Env |- Expr2 : u32
----------------------------------------- addition
Env |- Expr1 + Expr2 : u32 
```

`Place = Expr` 赋值的规则基于子类型：

```
Env |- Expr : Type1
Env |- Place : Type2
Env |- Type1 <: Type2
----------------------------------------- assignment
Env |- Place = Expr : () 
```

`&Place` 的规则稍微有趣一些：

```
Env |- Place : Type
----------------------------------------- shared references
Env |- & Place : & {shared(Place)} Type 
```

这个规则只是说明我们确定了被借用的位置 `Place` 的类型（这里，位置是 `counter`，其类型将是 `u32`），然后我们得到了对该类型的引用。该引用的起源将是 `{shared(Place)}`，表示该引用来自于 `Place`：

```
&{shared(Place)} Type 
```

## 计算活跃性

要引入借用检查，我们需要逐步引入**活跃性**的概念。^(如果你对这个概念不熟悉，NLL RFC 中有一个[很好的介绍](https://rust-lang.github.io/rfcs/2094-nll.html#liveness)：)

> “活跃性”这个术语源自于编译器分析，但它相当直观。我们说一个变量是活跃的，如果它当前持有的值可能在稍后被使用。

与 NLL 不同，我们仅计算活跃的**变量**，我们将计算**活跃的位置**：

```
LivePlaces = { Place } 
```

为了计算活跃位置集合，我们将引入一个辅助函数 `LiveBefore(Env, LivePlaces, Expr): LivePlaces`。`LiveBefore()` 返回在评估表达式 `Expr` 前是活跃的位置集合，给定环境 `Env` 和表达式后活跃的位置集合。我不会详细定义这个函数，但大致看起来是这样的：

```
// `&Place` reads `Place`, so add it to `LivePlaces`
LiveBefore(Env, LivePlaces, &Place) =
    LivePlaces ∪ {Place}

// `Place = Expr` overwrites `Place`, so remove it from `LivePlaces`
LiveBefore(Env, LivePlaces, Place = Expr) =
    LiveBefore(Env, (LivePlaces - {Place}), Expr)

// `Expr1` is evaluated first, then `Expr2`, so the set of places
// live after expr1 is the set that are live *before* expr2
LiveBefore(Env, LivePlaces, Expr1 + Expr2) =
    LiveBefore(Env, LiveBefore(Env, LivePlaces, Expr2), Expr1)

... etc ... 
```

## 将活跃性整合到我们的类型判断中

为了检测借用检查错误，我们需要调整我们的类型判断以包含活跃性。结果如下：

```
(Env, LivePlaces) |- Expr : Type 
```

这个判断说，“在环境 `Env` 中，并且考虑到函数将来会访问 `LivePlaces`，`Expr` 是有效的，并且具有类型 `Type`”。这样整合活性就给我们一些关于将来会发生什么访问的想法。

对于像 `Expr1 + Expr2` 这样的复合表达式，我们必须调整活跃位置的集合以反映控制流：

```
LiveAfter1 = LiveBefore(Env, LiveAfter2, Expr2)
(Env, LiveAfter1) |- Expr1 : u32
(Env, LiveAfter2) |- Expr2 : u32
----------------------------------------- addition
(Env, LiveAfter2) |- Expr1 + Expr2 : u32 
```

我们从 `LiveAfter2` 开始，即整个表达式之后仍然活跃的位置。这些位置也是在评估表达式2之后活跃的位置，因为这个表达式本身不引用或覆盖任何位置。然后我们计算 `LiveAfter1` - 即在评估 `Expr1` 之后活跃的位置 - 通过查看在 `Expr2` 之前活跃的位置。这有点令人费解，我花了一些时间来理解。这里的棘手之处在于活性是*反向*计算的，但我们大多数的类型规则（和直觉）倾向于*正向*流动。如果有帮助的话，可以考虑 `+` 的“完全展开”版本：

```
let tmp0 = <Expr1>
    // <-- the set LiveAfter1 is live here (ignoring tmp0, tmp1)
let tmp1 = <Expr2>
    // <-- the set LiveAfter2 is live here (ignoring tmp0, tmp1)
tmp0 + tmp1
    // <-- the set LiveAfter2 is live here 
```

## 活性检查借贷

现在我们知道活性信息了，我们可以用它来进行借贷检查。我们引入一个“允许”判断：

```
(Env, LiveAfter) permits Loan 
```

指示“在给定环境和活跃位置的情况下，允许采取贷款 Loan”。这是赋值规则，修改以包括活性和新的“允许”判断：

```
(Env, LiveAfter - {Place}) |- Expr : Type1
(Env, LiveAfter) |- Place : Type2
(Env, LiveAfter) |- Type1 <: Type2
(Env, LiveAfter) permits mut(Place)
----------------------------------------- assignment
(Env, LiveAfter) |- Place = Expr : () 
```

在我深入讨论如何定义“允许”之前，让我们回到我们的例子，对这里正在发生的事情有一个直觉。我们想在这个赋值上声明一个错误：

```
let  mut  counter: u32 =  22_u32; let  p: &{shared(counter)}  u32  =  &counter; counter  +=  1;  // <-- Error println!("{p}");  // <-- p is live 
```

注意，由于下一行的 `println!`，`p` 将在我们的 `LiveAfter` 集合中。查看 `p` 的类型，我们看到它包括贷款 `shared(counter)`。因此，突变 `counter` 是不合法的，因为有一个活跃的贷款 `shared(counter)`，这意味着 `counter` 必须是不可变的。

重申直觉：

> 集合 `Live` 中的活跃位置*允许*贷款 `Loan1`，如果对于 `Live` 中的每个活跃位置 `Place`，该位置类型中的贷款与 `Loan1` 兼容。

更正式地写出来：

```
∀ Place ∈ Live {
    (Env, Live) |- Place : Type
    ∀ Loan2 ∈ Loans(Type) { Compatible(Loan1, Loan2) }
}
-----------------------------------------
(Env, Live) permits Loan1 
```

这个定义使用了两个辅助函数：

+   `Loans(Type)` - 出现在类型中的贷款集合

+   `Compatible(Loan1, Loan2)` - 定义两个贷款是否兼容。两个共享贷款总是兼容的。仅当位置不重叠时，可变贷款才与另一个贷款兼容。

## 结论

本文的目标是提供高层次的直觉。我是从记忆中写的，所以可能有一些遗漏。但是在接下来的文章中，我想更深入地探讨我正在研究的系统如何工作以及它可以支持的新事物。一些高层次的例子：

+   如何定义子类型化，特别是活性在子类型化中的作用

+   我们今天使用的重要借贷模式及其在新系统中的工作方式

+   指向由其他结构字段拥有的数据的内部引用及其支持方式
