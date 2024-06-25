<!--yml

类别：未分类

日期：2024-05-27 14:44:53

-->

# ngn-k-tutorial/12-thinking-in-k.md at main · razetime/ngn-k-tutorial · GitHub

> 来源：[https://github.com/razetime/ngn-k-tutorial/blob/main/12-thinking-in-k.md](https://github.com/razetime/ngn-k-tutorial/blob/main/12-thinking-in-k.md)

# 以数组语言思考

[](#thinking-in-an-array-language)

*你可以在[GitHub](/razetime/ngn-k-tutorial/blob/main/code/matmul.k)上查看本章的完整源代码。*

由于你现在已经对 K 有了适当的了解，让我们来做一些编程吧。大多数 K 编程都发生在 REPL 中，因为迭代先前的代码非常有用。ngn/k 与 rlwrap 一起具有使用上/下箭头键进行历史记录的功能，这应该足以开始在 K 中开发更大的程序了。函数在 REPL 中进行测试，然后移至实际代码中。请注意，ngn/k 的漂亮打印始终返回有效的 K 数据，你可以预先计算一些东西以加快程序速度。

K 脚本始终像在 repl 中键入一样执行，也就是说：每一行都会被执行，并且如果行末不是分号，则打印其返回值。脚本还允许多行定义，这对于可读性很方便。通常情况下，你可能会在脚本中保存工作，并希望在 repl 中使用它。为了使用你存储的数据和函数，只需在 repl 中输入 `\l file.k`，你的文件将被执行，并加载其数据。你可以多次将文件加载到 REPL 中，覆盖旧数据。通过 `\` 访问的 repl 帮助还列出了更多有用的命令。

K 编程（以及数组编程一般来说）是一个不断简化你的模式的持续过程。一个庞大、难以管理的模式有一种或多种方法可以将其压缩为一个更小、更声明式、更易于阅读的模式。如果你想更好地理解这一点，可以在 [Patterns and Anti-patterns in APL: Escaping the Beginner's Plateau - Aaron Hsu - Dyalog '17](https://www.youtube.com/watch?v=9xCJ3BCIudI) 中详细讨论这个问题。

K 中大多数人遇到的常见问题是需要将常见的、众所周知的算法转换为 K，通常是从编程网站（如 geeksforgeeks）或维基百科文章中提取的。让我们以矩阵乘法为例。

根据[这篇维基百科文章](https://en.wikipedia.org/wiki/Matrix_multiplication_algorithm)，矩阵乘法的迭代算法如下：

```
Input: matrices A and B
Let C be a new matrix of the appropriate size
For i from 1 to n:
  For j from 1 to p:
    Let sum = 0
    For k from 1 to m:
      Set sum ← sum + Aik × Bkj
  Set Cij ← sum
Return C 
```

如果你愿意，可以尝试将此内容翻译成 K。直接翻译如下：

```
matmul: {
  A::x
  B::y
  n::#A
  m::#*A
  p::#*B
  C::(n;p)#0
  i::0
  j::0
  k::0
  sum::0
  {
    i::x
    {
      j::x
      sum::0
      {
        k::x
        sum::sum+A[i;k]*B[k;j] 
      }'!m
      C[i;j]::sum
    }'!p
  }'!n
  C} 
```

这是我写过的最糟糕的 K 代码，因为我们试图将 K 写成一种命令式语言，而 K 并不适合这种设计。主要问题是：

+   很多，很多全局变量被赋值

+   多重嵌套循环

+   大量修改

幸运的是，我们这里有很多可以简化的东西，我们可以逐个解决这些问题。

让我们从最内层的循环开始：

```
sum::0
{
  k::x
  sum::sum+A[i;k]*B[k;j] 
}'!m
C[i;j]::sum 
```

我们可以做的第一个和最简单的修复是使用 fold（`/`）来求和。

```
C[i;j]::+/{
  k::x
  A[i;k]*B[k;j] 
}'!m 
```

一个全局变量解决了，还有 9 个。

我们可以移除的下一个全局变量是`C`。由于`'`（每个）返回一个数组，C不需要修改。我们可以简单地返回嵌套循环的值。

```
 {
    i::x
    {
      j::x
      +/{
        k::x
        A[i;k]*B[k;j] }'!m }'!p }'!n} 
```

现在，我们有了三个没有修改的循环，这让我们的工作变得更容易。现在要查看的主要变量是`i`、`j`和`k`。

+   `i`索引A的每一行。

+   `j`索引B的每一列。

+   `k`索引A的每一列和B的每一行。

基本上，`k`负责将A的每一行与B的每一列配对，然后将它们相乘。因此，我们可以在这里消除中间人，并直接匹配它们而不需要`k`。这也消除了一个循环，并消除了对`m`的需求。

```
{
  j::x
  +/A[i]*B[;j] }'!p }'!n} 
```

接下来，要移除`j`，我们需要取`B`的每一列并将其与`A[i]`配对。为此，我们对`B`进行转置，并将每个元素与eachright (`/:`)配对。

```
matmul: {
  A::x
  B::y
  n::#A
  i::0
  {
    i::x
      A[i]{+/x*y}/:+B}'!n } 
```

为了移除`i`，我们做了类似的事情：使用eachleft将`A`的每一行与`B`的每一列配对。

```
matmul: {
  A::x
  B::y
  A{+/x*y}/:\:+B } 
```

我们不再需要全局变量了！

现在，*这就是*在K中的矩阵乘法。这是矩阵乘法的最直接的算法转换。现在我们将看看如何缩短它，并移除更多的循环。

`+`（转置）是昂贵的，我们可以移除它。我们当前正在做的事情是简单的。我们可以将B的每一行调整到A的整个范围内，隐式地执行相同的操作。

现在，我们有一个可以轻松转换为被动语态的函数。按照[第三章](#trains)的规则，我们得到最终结果：

一个值得自豪的矩阵乘法函数。这个过程可能看起来有很多步骤，但随着您在K中练习技能，压缩代码将变得更加容易和直观。

矩阵乘法是一个简单的过程，很适合K的数组支持。我们将在未来的章节中看到更多不适用于K的算法以及如何处理它们。
