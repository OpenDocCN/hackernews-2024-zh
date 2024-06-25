<!--yml

类别：未分类

日期：2024-05-27 14:45:30

-->

# Felix Prasanna

> 来源：[`fprasx.github.io/articles/type-system-arithmetic/`](https://fprasx.github.io/articles/type-system-arithmetic/)

# 在 Rust 的类型系统中进行一年级数学

2024-01-04

算术很难。幸运的是，我们可以使用 Rust 的表现型系统来解决这个问题™️。

### 数学

让我们从 Church 编码开始。函数式编程中有一个概念，即函数是好的，类型理论家将其推得相当远。到目前为止，他们有时将数字定义为看起来像这样的函数：

```
0 = |f, x| x                             # f applied 0 times 1 = |f, x| f(x)                          # f applied 1 times 2 = |f, x| f(f(x))                       # f applied 2 times 3 = |f, x| f(f(f(x)))                    # f applied 3 times ...                                      # f applied . times n = |f, x| f( .. n - 2 more f's .. f(x)) # f applied n times 
```

第 n 个自然数 *字面上* 被定义为一个接受值和函数的函数，并将该函数应用 n 次。

还有数学中的另一个概念叫做 Peano 公理。这也是一种非常有用的方式来表征自然数，基本上如下（省略了一些细节）：

```
0\. 0 is a natural number. 1\. The successor, S(n), of a natural number is a natural number. 
```

这以归纳的方式定义了自然数：

```
0 = 0 1 = S(0) 2 = S(1) = S(S(0)) 3.= S(2) = S(S(S(0))) ... n = S( .. n - 2 more S's .. S(0)) 
```

其中`S`称为后继函数。它实际上是什么并不重要，因为我们正在追求更...抽象的事物视图。

注意到与 Church 编码的任何相似之处了吗？

主要思想是我们可以将一个数字表示为*重复的函数应用*。

### 进入 Rust

那么我们如何在 Rust 中复制这个？我们如何使其性能良好？我们可以只是使用常规的 lambda 来做，但那可能会很慢，因为闭包是装箱的，等等等等。

最终策略是扮演上帝并在编译时计算我们的数字！毕竟，这就是 Rust 的方式。但我们如何在编译时调用函数呢？嗯，我们可以借鉴类型理论的另一个想法，并注意到泛型结构体在某种程度上类似于函数。考虑：`struct Wrapper<T>(PhantomData<T>)`。实例化`T`有点像在类型上计算一个函数，从类型`T`开始，以类型`Wrapper<T>`结束。而且可以想象构造一种类型`Wrapper<Wrapper<Wrapper<T>>>`...

我敢打赌这开始变得重复了。让我们定义以下类型：

```
struct Zero; struct Succ<T>(PhantomData<T>); 
```

是的，我知道`Succ`听起来像一个滑稽的词，但它实际上只是`Successor`的惯例。现在我们可以定义一些自然数：

```
type One = Succ<Zero>; type Two = Succ<Succ<Zero>>; type Three = Succ<Succ<Succ<Zero>>>; ... 
```

这将变得很烦人。让我们写一个小宏来帮助我们：

```
#[macro_export] macro_rules! encode {
 () => { Zero }; ($_a:tt $($tail:tt)*) => {
 Succ<encode!($($tail)*)>
 }; } 
```

基本上，我们将令牌传递给这个坏男孩，它将逐个将它们吞噬掉，每次向其构造的类型添加一个新的`Succ`层。这是一个示例扩展：

```
 encode!(* ***) ^ these tokens correspond to $tail = Succ<encode!(* **)> = Succ<Succ<encode!(* *)>> = Succ<Succ<Succ<encode!(*)>>> = Succ<Succ<Succ<Succ<encode!()>>>>
 ^ input is empty, so encode!() expands to Zero = Succ<Succ<Succ<Succ<Zero>>>> 
```

现在我们可以使用`encode!(***** ... *****)`构造相当大的数字，尽管编译器由于递归在大约 3200 个令牌处会溢出其堆栈。

### 评估

现在，请记住，我们正在尝试使用这些数字进行算术运算，所以我们需要能够在某个时候将它们从编码形式中提取出来。希望我们可以在编译时执行所有计算（所以无限快？），然后将它们作为常量提取到我们的最终二进制文件中。这意味着我们需要一个从类型到...值的函数？嗯。如果你仔细看，特质有点像是从类型到值的函数。

```
trait SpecialNumber {
  const MAGIC: usize; }   impl SpecialNumber for Foo {
  const MAGIC: usize = 13; }   impl SpecialNumber for Bar {
  const MAGIC: usize = 29; } 
```

从某种意义上说，我们将类型 `Foo` 映射到 `13`，将类型 `Bar` 映射到 `29`。让我们继续这个想法，定义 `Value` 特性：

```
trait Value {
  const VALUE: usize; }   impl Value for Zero {
  const VALUE: usize = 0; }   impl<T> Value for Succ<T> where T: Value {  const VALUE: usize = 1 + T::VALUE; } 
```

现在我们可以递归地 "评估" 一个类型 - 或者获取 `Succ` 被应用了多少次，因为第二种情况在每次迭代中剥离一个 `Succ` 层。

### 加法和减法

好了，我们现在有点有趣的东西了。主要思想仍然是一样的。我们将使用这种奇怪的类型递归来构造新类型。我们可以使用特性将类型映射到值，也可以使用它们将类型映射到其他类型，使用关联常量。让我们这样定义特性 `Add`：

```
trait Add {
  type Sum; } 
```

现在我们可以为我们的递归定义一些基本情况：

```
impl Add for (Zero, Zero) { // 0 + 0 = 0
  type Sum = Zero; }   impl<T> Add for (Succ<T>, Zero) { // x + 0 = x
  type Sum = Succ<T>; }   impl<T> Add for (Zero, Succ<T>) { // 0 + x = x
  type Sum = Succ<T>; } 
```

现在轮到我们的递归规则了：

```
impl<T, U> Add for (Succ<T>, Succ<U>) where
 (T, Succ<Succ<U>>): Add, {
  type Sum = <(T, Succ<Succ<U>>) as Add>::Sum; } 
```

让我们来分解一下。我们首先要取两个非零数 - 因为两个类型参数都包含在 `Succ` 中，并且也可能是 `Succ<..>` 本身。这个规则的想法是从第一个类型参数中取下一个 `Succ` 层，并将其放在另一个上。然后最终，第二个类型参数将拥有所有的 `Succ` - 正好是它们最初共同拥有的 `Succ` 的总数，这意味着它表示了两个起始数的和。让我们做一个小例子（我不会为所有做这个）：

```
 Add<Succ<Succ<Zero>>, Succ<Succ<Zero>>> = Add<Succ<Zero>, Succ<Succ<Succ<Zero>>>> // move one Succ to the right = Add<Zero, Succ<Succ<Succ<Succ<Zero>>>>> // again = Succ<Succ<Succ<Succ<Zero>>>>>           // base case :) 
```

最后，还有一些特性约束 - 我们告诉 rustc 我们只会在可以相加的类型上调用这个特性（是不是很奇怪？）。

减法很相似（除法真的很疯狂）。我们定义了我们的基本情况：

```
trait Sub {
  type Diff; }   impl Sub for (Zero, Zero) { // 0 - 0 = 0
  type Diff = Zero; }   impl<T> Sub for (Succ<T>, Zero) { // x - 0 = x
  type Diff = Succ<T>; }   impl<T> Sub for (Zero, Succ<T>) { // 0 - x = 0, we'll saturate
  type Diff = Zero; } 
```

这里唯一有趣的事情是，我们将减法定义为 *饱和的*。这将有助于实现除法。如果完全省略这个规则，编译器实际上会阻止您发生下溢！

无论如何，这是我们的递归规则：

```
impl<T, U> Sub for (Succ<T>, Succ<U>) where
 (T, U): Sub, {
  type Diff = <(T, U) as Sub>::Diff; } 
```

我们基本上只是从每个类型参数中剥离一个 `Succ` 层，直到其中一个为零。剩下的那个就是我们的差异。你可以想象成这样：

```
5 [***] ** - 3 [***] = 2 [   ] ** 
```

我们逐个消除括号中的共同项。

# 乘法

好的，这个任务有点复杂，但我们可以使用之前的构建块。像往常一样，我们定义一个特性函数和一些基本情况：

```
trait Mul {
  type Product; }   impl<T> Mul for (T, Zero) { // x * 0 = 0
  type Product = Zero; }   impl<T> Mul for (Zero, Succ<T>) { // 0 * x = 0
  type Product = Zero; } 
```

现在我们必须更加小心地实现 `Mul`。如果我们对 `(Zero, T)` 进行第二次实现，那么我们将有两个实现涵盖 `(Zero, Zero)`。因此，我们将其实现在 `(Zero, Succ<T>)` 上，因为 `Succ<T>` 永远不等于 `Zero`。现在对于递归，我们有了这个宝贝怪物：

```
impl<T, U> Mul for (Succ<T>, Succ<U>) where
 (T, Succ<U>): Mul, (Succ<U>, <(T, Succ<U>) as Mul>::Product): Add, {
  type Product = <(
 Succ<U>,                       // y <(T, Succ<U>) as Mul>::Product // (x - 1) * y ) as Add>::Sum; } 
```

在这里，我们利用了 `x * y = y + (x - 1) * y` 这个事实。我们递归地计算第二项，然后将其加到第一项上！记住，如果 `Succ<T>` 表示 `T + 1`，那么 `T` 表示 `Succ<T> - 1`。

最后...

### 除法

我不会撒谎，这个真的是个怪物。我们首先需要定义 `GreaterThanEq` 来检查 `x >= y`，因为有些除法不干净，比如 `7 / 3`。稍后我会详细解释。我们将以与我们做 `Subtraction` 相似的方式来做这个：

```
trait GreaterThanEq {
  type Greater; }   impl GreaterThanEq for (Zero, Zero) { // 0 >= 0
  type Greater = Succ<Zero>; }   impl<T> GreaterThanEq for (Zero, Succ<T>) { // 0 >!= { 1 .. }
  type Greater = Zero; }   impl<T> GreaterThanEq for (Succ<T>, Zero) { // { 1 .. } >= 0
  type Greater = Succ<Zero>; }   impl<T, U> GreaterThanEq for (Succ<T>, Succ<U>) where
 (T, U): GreaterThanEq, // x >= y -> x - 1 >= y - 1 {
  type Greater = <(T, U) as GreaterThanEq>::Greater; } 
```

有了这个，让我们处理除法吧。我们首先从完美商开始，比如`6 / 3`。这是我们的样板代码：

```
trait Div {
  type Quotient; }   impl<T> Div for (Zero, T) { // 0 / x = 0
  type Quotient = Zero; }   impl<T> Div for (Succ<T>, Succ<Zero>) { // x / 1 = x
  type Quotient = Succ<T>; } 
```

再次，我们必须小心处理我们的类型参数。我们不是在`(T, Succ<Zero>)`上执行第二个实现，而是在`(Succ<T>, Succ<Zero>)`上实现它，以避免与在`(Zero, Succ<Zero>)`上的第一个实现重叠。我们现在可以利用恒等式`x / y = 1 + (x - y) / y`。直观地说，这意味着我们只需从`x`中取出一个`y`，然后递归计算其余部分。在代码中，它看起来像：

```
type RawQuotient<T, U> = <(
 Succ<Zero>, // 1 <( <(Succ<T>, Succ<Succ<U>>) as Sub>::Diff, Succ<Succ<U>> ) as Div>::Quotient, // (x - y) / y ) as Add>::Sum; 
```

唯一的问题是，如果`x < y`，那么此表达式是不正确的，并且会返回 1。减法会饱和到 0，所以我们得到`1 + 0 / y = 1 + 0 = 1`。

我们只需要将这个`RawQuotient`与`x / y`是 0 的事实相结合，如果`x < y`。由于我们将此数字作为布尔值，因此我们可以通过用乘法替换条件跳转来进行一个技巧（或者在汇编中进行 cmov）。请注意以下表格：

```
 Case       Boolean of Gte       RawQuotient       Desired +--------+  +----------------+   +-------------+   +---------+ | x >= y |: |       1        | x |   nonzero   | = | nonzero | +--------+  +----------------+   +-------------+   +---------+ | x < y  |: |       0        | x |     ???     | = |    0    | +--------+  +----------------+   +-------------+   +---------+ 
```

我们只需将我们的`RawQuotient`乘以`x >= y`的结果！如果`x >= y`，我们将得到我们的`RawQuotient`。如果`x < y`，我们将乘以 0，并且无论如何都会得到 0，正如我们所期望的那样。这导致了除法的最终形式：

```
impl<T, U> Div for (Succ<T>, Succ<Succ<U>>) where
 (T, Succ<U>): Sub, (<(T, Succ<U>) as Sub>::Diff, Succ<Succ<U>>): Div, ( Succ<Zero>, <(<(T, Succ<U>) as Sub>::Diff, Succ<Succ<U>>) as Div>::Quotient, ): Add, ( <(Succ<T>, Succ<Succ<U>>) as GreaterThanEq>::Greater, <( Succ<Zero>, <(<(T, Succ<U>) as Sub>::Diff, Succ<Succ<U>>) as Div>::Quotient, ) as Add>::Sum, ): Mul, (Succ<T>, Succ<Succ<U>>): GreaterThanEq, {
  // Hi I'm the important part
  type Quotient = <(
 <(Succ<T>, Succ<Succ<U>>) as GreaterThanEq>::Greater, RawQuotient<T, U>, ) as Mul>::Product; } 
```

我们有一堆特质边界，因为我们有更多的子表达式需要确保可以计算。但归根结底，所有这些基本上都是将`Succ`链传递到底部的`Zero`。

您还可能注意到我们在`(Succ<T>, Succ<Succ<U>>)上实现了`Div`的递归情况。这意味着分母大于 1，因此不会与任何其他实现重叠。

按照真正的 Rust 风格，这个实现实际上不会允许我们除以 0，因为我们从未为`(*, Zero)`实现过`Div`！

有点令人失望的是……就是这样了！我不知道如何表达小绿色测试用例点通过的喜悦，所以您可以通过从[Arithmetic in Rust's Type System](https://github.com/fprasx/arts)获取代码或简称为 arts 来自行运行。

# 注：

我应该指出，我不是第一个这样做的人——尽管当我刚开始时可能会认为我是。有关先前的作品，请参阅 [typenum](https://crates.io/crates/typenum) 和 [peano](https://crates.io/crates/peano) crates。`typenum`速度要快得多，并且在实践中实际上是您会使用的。

再次，我强烈建议您自行运行[代码](https://crates.io/crates/peano)。看到这些抽象特征在您的终端上转化为数字真是一种很酷的感觉。也请随意给个星星！

我就说这么多了。祝您今天愉快 :)
