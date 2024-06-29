<!--yml

类别：未分类

日期：2024-05-27 14:49:33

-->

# [或许一切都是协程 - Adam Nelson](https://adam.nels.onl//blog/maybe-everything-is-a-coroutine/)

> 来源：[https://adam.nels.onl//blog/maybe-everything-is-a-coroutine/](https://adam.nels.onl//blog/maybe-everything-is-a-coroutine/)

在阅读优秀博客文章 [让 Futures 成为 Futures](https://without.boats/blog/let-futures-be-futures/) 后，我受到了作者在想象一个所有函数都是协程的语言中，如何表达异步性的启发。异步函数在等待某些异步操作时可以产生称为 `Pending` 的类型，而纯同步函数可以产生 `Never`，表示它们根本不会产生任何输出。

我越想到这个问题，越意识到在一个 *每个函数都是协程* 的强类型语言中，实际上可以有很多优点。我设计了一个带有几个很酷特性的语言：

+   一种类型系统，其中协程基本上是状态机

+   基于协程的类型化（代数）效果系统

    +   函数可以针对效果和纯度进行泛化

    +   调用者可以“拦截”效果并伪造它们

+   基于简单和类型和的强大异常系统

    +   一种感觉像是更合理的泛化检查异常的例外传递系统

    +   类似于 Common Lisp 风格的可恢复条件

+   不可变数据结构和默认纯函数，但可以在算法中使用突变，通过将所有突变保留在函数的作用域中并使用协程来控制状态

它是如何工作的？有三个关键特性：*序列类型*，*协程 for 循环* 和 *传递操作符*。

## 序列类型

普通函数具有 *参数类型* 和 *返回类型*：

```
fib(n: Int) -> Int 
```

在这种一切都是协程的语言中，每个函数没有返回类型，而是有一个 *序列类型*。这是一个代数类型，描述了函数可以产生的每种类型，以及在产生后可以继续执行的每种类型。

例如，`fib` 的协程版本，它不返回第 `n` 个斐波那契数，而是产生斐波那契数的无限序列：

```
fib() -> Int() 
```

`()` 表示在协程产生一个 `Int` 后，可以不带参数继续执行。因为没有基本情况（没有无法继续执行的分支），这个协程必须是无限的；它将继续执行，直到调用者选择不再继续执行它。

考虑代替一个产生 *有限* `Int` 序列的协程：

```
factors(n: Int) -> Int() | Void 
```

这个序列类型有两个 *情形*：它要么发出一个 `Int` 并可以不带参数继续执行，要么不发出任何东西并且无法继续执行。

注意 `Void` 的语法：没有括号的类型是有效的序列类型。这意味着普通函数 `fib(n: Int) -> Int` 也是有效的协程——它一次产生 `Int`，然后无法继续执行。纯函数是协程的一个子集！

这些强类型的协程本质上是状态机。让我们定义一个额外的语法片段：一个*符号* `:foo` 是一个唯一的单例对象，其类型也是 `:foo`。使用符号，我们可以为门定义一个简单的状态机：

```
door() ->
  | :open(:close)
  | :closed(:open | :lock)
  | :locked(:unlock) 
```

每个状态只接受一组特定的动作。一扇开着的门只能关闭；一扇锁着的门只能解锁；一扇关闭的门可以打开或者锁上。

## 协程`for`循环

但是我们如何*使用*这些协程呢？基本函数调用的语法保持不变；如果一个协程在其序列类型中只有无恢复类型，那么它可以直接用作参数或赋值给变量。

对于更复杂的协程，有一个`for`循环语法。考虑之前的门状态机；这是一个使用它的`for`循环的示例：

```
for (door()) {
  | :locked ->
    continue(:unlock)
  | :closed ->
    continue(:open)
  | :open ->
    break
} 
```

这个循环模式匹配`door`产生的值。分支使用熟悉的关键字`continue`和`break`，但现在`continue`带有参数：它使用新参数恢复`for`的协程，这些参数必须与序列类型的当前分支的参数类型匹配。直到遇到`break`，循环才结束。

我们可以为协程常见情况下产生值列表而不带任何恢复参数的情况增加一些便利。在恢复类型为`()`的块末尾推断出`continue()`，在没有恢复类型的块末尾推断出`break`，可以省略`Void`分支，假定通配符匹配不到`Void`。

```
def someNumbers() -> Int() | Void {
  yield 1
  yield 2
  yield 3
}

var total
for (someNumbers()) { n ->
  total += n
} 
```

## 传递操作符

将此协程语法扩展以支持代数效应和错误处理非常简单。错误可以作为普通值被产生，通常不会恢复：

```
divide(divisor: Int, dividend: Int) -> DivideByZeroError | Int 
```

代数效应可以被产生，预计一直到主函数，然后将以对给定效应的响应恢复。这些变得复杂，但我稍后会引入语法糖来处理这些巨大的签名。

```
readFile(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | Bytes 
```

实际上*使用*已经存在的机制（例如`for`循环和`yield`）与效应和错误将会非常繁琐。它看起来很像Go：到处手动检查错误状态或效应，只是为了将它们`yield`给堆栈中的下一个级别。

```
readConfigJson(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | JsonParseError
  | JsonFormatError
  | ConfigFile
{
  for (readFile(filename)) {
    | IOFileStat a -> continue(yield a)
    | IOFileRead a -> continue(yield a)
    | FileError e -> yield e
    | Bytes data ->
      // borrowing some Scala syntax
      parseJson(data) match {
        | JsonParseError e -> yield e
        | JsonValue json -> yield parseConfigFile(json)
      }
  }
} 
```

这里的常见模式是 `a -> continue(yield a)`（对于需要恢复的分支）和 `e -> yield e`（对于不需要恢复的分支）。这种模式在这种代码风格中非常常见，它应该有自己的语法糖：*传递操作符* `^`。

在表达式前面加上`^`会导致该表达式产生的任何值（可以直接作为`continue (yield a)`或者仅仅`yield a`产生）被产生。换句话说，它会导致从这个表达式传递到调用者的函数序列类型中包含的任何类型。

这大大简化了示例函数：

```
readConfigJson(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | JsonParseError
  | JsonFormatError
  | ConfigFile
{
  val data = ^readFile(filename)
  val json = ^parseJson(data)
  yield parseConfigFile(json)
} 
```

尽管序列类型仍然很混乱。还有另一个问题：如果一个函数产生了一堆效果类型，但同时也有一个你想在变量中捕获的类型……而且这个类型也恰好在函数的序列类型中？

下面是解决这些问题的一些变更：

+   关键字`infer Type`可以在序列类型的任何分支中使用；它的类型被推断为函数产生的所有`Type`的子类型。通过`^`运算符传递的值也包括在此推断中。

    +   `infer Type(*)`也推断恢复参数；这可以推断多个分支。

+   `^`运算符可以出现在序列类型分支的前面。这会导致函数体中的`^`只传递具有序列类型前面也有`^`的类型。

+   一个块或关键字，比如`nopass`，可以阻止某种类型被传递；这类似于捕获块，确保函数可以处理特定的产生类型。

使用这些变更，并假设`FileIO`和`Error`这两个超类型已存在，该函数变得更加简单！

```
readConfigJson(filename: String) -> ^infer IOFile(*) | ^infer Error | ConfigFile {
  val data = ^readFile(filename)
  val json = ^parseJson(data)
  yield parseConfigFile(json)
} 
```

## 将所有内容整合起来

这三个关键特性构成了一个出奇稳固的语言核心，但它们本身并不是一个完整的语言。它们可以适用于几种不同的语言设计，但在我看来，建立在这个基础上的理想语言应该具有：

+   除了代数效应和可变局部变量外，严格的函数纯度。只能通过从`main`中产生的效果对象执行副作用。

    +   可变局部变量仍然允许有效的可变数据结构，多亏了协程。可以存在一个可变的、可调整大小的数组类型，但它不能被产生，不能作为参数传递，也不能存储在结构中。这些数组的结构可以通过将协程传递到函数中，然后使用由该协程产生的操作来操作。

+   不可变的、名义类型的结构，带有子类型化。

+   一个通用的元组类型，允许像 Erlang 那样使用符号进行消息传递，但是是强类型的。

+   完整的高阶泛型。这允许函数在纯度和检查异常方面是泛型的，这两个特性在类型系统中非常罕见。

+   序列类型的别名，可以包括多个分支。其语法可能会使用`(*)`。例如，你可以定义`type Ints -> Int() | Void` ——`->`表示它是一个序列类型——然后像这样使用：`someFunction() -> Ints(*)`。

    +   使用这些类型，函数可以接受序列参数。例如，`sum(seq -> Ints(*)) -> Int`是一个对`Int`的有限序列求和函数。可以通过`sum(someFunction())`调用它，并且`someFunction`返回的*整个序列*将被传递给`sum`。

    +   应该也存在用于独立序列的语法，基本上是[IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)的协程等价物。例如，`-> { yield 1 }` 默认是类型为 `-> Int` 的序列，但也可以显式声明类型：`-> Int() | Void { yield 1 }`。

    +   序列字面量可以用方括号写出：`[1, 2, 3]` 会被解释为 `-> Int() | Void { yield 1; yield 2; yield 3 }` 的语法糖。

我完全不知道何时能抽出时间来真正工作，但我希望能看到这种语言的发展。我觉得这种一切皆协程的系统作为函数式编程的新架构具有很大潜力。
