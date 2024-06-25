<!--yml

category: 未分类

date: 2024-05-27 14:29:10

-->

# Lambdasort - Lucas Seiki Oshiro

> 来源：[https://lucasoshiro.github.io/software-en/2020-06-06-lambdasort/](https://lucasoshiro.github.io/software-en/2020-06-06-lambdasort/)

使用 lambda 编写的快速排序！

GitHub: [https://github.com/lucasoshiro/lambdasort](https://github.com/lucasoshiro/lambdasort)

## λ演算

Python，就像许多其他编程语言一样，支持**高阶函数**和**匿名函数**。这基本上意味着我们可以这样做：

```
 # anonymous function (lambda). This is equivaltent to sqrt: lambda n: n ** 0.5

# function that takes another function as parameter def apply_func(func, parameter):
    return func(parameter)

# function that returns another function. in this case, this is a mutiplier factory def multiplier_factory(n):
    return lambda x: n * x 
```

没错。这样一来，**函数就是值**，可以由其他函数创建并返回。

如果我们编写一个每个值的类型都是函数的代码会怎样？例如：

```
t = lambda a: lambda b: a
f = lambda a: lambda b: b
x = (lambda a: lambda b: a(b)(f))(t)(f) 
```

乍一看似乎毫无用处。然而，信不信由你（剧透：稍后会解释），这是 `True` 和 `False` 的布尔 `and` 运算！

这就是著名的**λ演算**。它只有接收函数作为参数并返回其他函数的函数。对我们来说，这已经足够了，但如果你想要了解更多，我真的建议你阅读[维基百科上的这篇文章](https://en.wikipedia.org/wiki/Lambda_calculus)。

尽管它看起来似乎不足以做任何事情，但实际上 λ演算可以解决任何可以用算法解决的问题，因为它是**图灵完备**的！[艾伦·图灵](https://www.cambridge.org/core/journals/journal-of-symbolic-logic/article/abs/computability-and-definability/FE8B4FC84276D7BACB8433BD578C6BFD#access-block)本人已经证明了这一点！

出色的[Tom Stuart 的“无限编程”](https://tomstu.art/programming-with-nothing)展示了如何使用 Ruby 中的 lambda 演算编写 fizzbuzz。第一次阅读时，我想要做类似的事情，所以我做了这个：**λ演算中的快速排序**。

我会告诉你我是如何做到的每一步的！

## 开始：一个普通的 Python 快速排序

第一步是用 Python 写一个快速排序，但与传统的快速排序相比有一个区别：它**不会改变**原始列表，而是**返回**一个新列表，就像 Python 中的 `sorted` 一样：

```
def car(A):
    return A[0]

def cdr(A):
    return A[1:]

def cons(A, x):
    return [x] + A

def concat(A, B):
    return A + B

def quicksort(A):
    if len(A) <= 1: return A
    L, R = partition(A)
    p = car(R)
    L = quicksort(L)
    R = quicksort(cdr(R))
    return concat(L, concat([p], R))

def partition(A):
    p = car(A)

    L, R = [], []

    for x in cdr(A):
        if x < p:
            L, R = cons(x, L), R
        else:
            L, R = L, cons(x, R)

    L, R = L, cons(p, R)

    return L, R 
```

看一看函数 `car`、`cdr` 和 `cons` 的用法。我遵循了 Lisp 对这些函数的命名约定。它们将进一步按照一些 Lisp 方言的方式实现，因此我尝试使它们在列表工作方面更接近它们而不是 Python 中的通常用法：

+   `car` 函数返回第一个元素

+   `cdr` 函数返回一个带有其余元素的列表。换句话说，除了第一个元素外的所有元素

+   到目前为止，我还将 `cons` 实现为一个函数，它返回一个看起来与提供的列表相同但在其开头**附加**一个新元素的新列表，但 `cons` 不仅仅是如此（我稍后会讨论它）

+   `concat` 函数**连接**两个列表。

其余部分只是标准的快速排序：

+   `partition` 函数**将列表分割**成两部分，左边的列表中的所有元素都小于右边的第一个元素（中轴），右边的列表中的所有元素都大于或等于中轴。

+   `quicksort` 函数调用 `partition` 将我们要排序的列表分成两部分，并分别使用 `quicksort` 对其进行排序，递归地，递归的基础是长度小于或等于 1 的列表。

至于奇怪的构造 `L, R = ...`，现在做这些并行赋值是没有用的，但在未来会很有帮助。

您可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/simple-quicksort/lambdasort.py)看到它。

## 重新定义类型

由于我们的想法是只使用 lambda 重写快速排序，我们需要以只使用函数的方式来**表示数据**。这里的类型有：

+   整数（我们要排序的列表中的值）

+   列表

+   对（由返回多个值的函数使用）

+   布尔值（由验证使用）

幸运的是，lambda演算的创始人**阿隆佐·丘奇**也向我们展示了如何做到这一点。维基百科上有一篇[关于此的文章](https://en.wikipedia.org/wiki/Church_encoding)。

### 布尔值

让我们从更容易的开始，**布尔值**：

```
#boolean constants LAMBDA_TRUE = lambda a: lambda b: a
LAMBDA_FALSE = lambda a: lambda b: b

#boolean opearations LAMBDA_OR = lambda a: lambda b: a(LAMBDA_TRUE)(b)
LAMBDA_AND = lambda a: lambda b: a(b)(LAMBDA_FALSE)
LAMBDA_NOT = lambda a: a(LAMBDA_FALSE)(LAMBDA_TRUE) 
```

是的，`true` 只是一个接受**两个参数**并返回**第一个参数**的函数，而 `false` 是一个接受两个参数并返回**第二个参数**的函数。

我知道你在想：“应该是`lambda a, b: a`和`lambda a, b: b`，为什么有两个`lambda`？”。这是因为在 lambda 演算的定义中，函数只能接受**一个参数**。与该定义相反，在 Python 中 `lambda` 可以接受零个或多个参数，但在这里我限制自己只使用接受一个参数的函数，以符合该定义。这样，原本会写成 `lambda a1, a2, a3, ..., an:` 的函数将被写成 `lambda a1: lambda a2: lambda a3: ... lambda an:`。在调用函数时，我们使用 `(a1)(a2)(a3)...(an)` 而不是 `(a1, a2, a3, ..., an)`。这种转换的名称是 **[柯里化](https://en.wikipedia.org/wiki/Currying)**，以 Haskell Curry 命名。一个本地使用柯里化处理函数的例子是 Haskell（你别说！）。

接着，我实现了三个基本的**布尔运算**：

+   `not`：接受一个 Church 布尔值，并使用 `false` 和 `true` 作为参数调用它。如果布尔值是`true`，则返回**第一个参数**（`false`）；如果是`false`，则返回**第二个参数**（`true`）。

+   `or`：接受两个 Church 布尔值，调用第一个并将`true`和第二个布尔值作为参数传递给它。如果`or`的第一个参数是`true`，则返回其**第一个参数**（`true`）；如果是`false`，则返回其**第二个参数**（`or`的第二个参数）。

+   `and`：很像 `or`。试着在脑中模拟它 ;-)

我们也可以定义一个`if`：

#### if

```
LAMBDA_IF = lambda c: lambda t: lambda e: c(t)(e) 
```

在 `if` 中，`c` 是 **条件**；`t` 是“then”块，当 **条件为真** 时发生的情况，`e` 是“else”块，当条件为 **假** 时发生的情况。

这样，`if` 更接近于 Python 中的 **if 表达式**（或三元运算符）而不是控制流 `if`：

```
use_5 = LAMBDA_TRUE
dont_use_5 = LAMBDA_FALSE

a = LAMBDA_IF(use_5)(5)(0) # a = 5 b = LAMBDA_IF(dont_use_5)(5)(0) # a = 0 
```

#### 转换

我还定义了两个函数将 Church 布尔值转换为 Python 布尔值（`l2b`）和反之亦然（`b2l`）：

```
#boolean conversion def l2b(l):
    return l(True)(False)

def b2l(b):
    return LAMBDA_TRUE if b else LAMBDA_FALSE 
```

思维练习：模拟它们！

你可以在 [这里](https://github.com/lucasoshiro/lambdasort/blob/booleans/lambdasort.py) 看到它

### 整数

**Church 数** 被定义为：

```
LAMBDA_ZERO = lambda p: lambda x: x
LAMBDA_ONE = lambda p: lambda x: p(x)
LAMBDA_TWO = lambda p: lambda x: p(p(x))
# etc 
```

换句话说，所有的整数都是以这种方式接受 **两个参数** `p` 和 `x` 的函数：

+   0 是一个返回 `x` 的函数（就像 `false` 一样）

+   1 是一个返回 `p(x)` 的函数

+   2 是一个返回 `p(p(x))` 的函数

依此类推。但是，我们可以使用这种编码表示负数。

#### 增量和减量

**增量** 定义起来很容易，它是一个添加另一层 `p(...)` 的函数：

```
LAMBDA_INCREMENT = lambda l: lambda p: lambda x: p(l(p)(x)) 
```

但是 **减少** 要困难得多。解释它需要一些时间，而且现在理解它对我们来说没有用。如果你真的想知道它是如何工作的，请自己试一试。如果你不在乎，就相信 ~~我~~ Church 它是有效的：

```
LAMBDA_DECREMENT = lambda n: lambda f: lambda x: n(lambda g: lambda h: h(g(f)))(lambda y: x)(lambda y: y) 
```

如果你自己尝试模拟它，你可能试图弄清楚如果你减少零会发生什么。你发现了这里 **零的减量是零**。不幸的是，这是一个限制，我们很快将在未来需要记住它。

#### 加法和减法

好了，我们有增加和减少了。如果我们增加 `n` 次数字 `m`，那么我们将得到 `m + n`，如果我们减少 `n` 次，我们将得到 `m - n`。

我们可以这样定义加法和减法：

```
LAMBDA_ADD = lambda m: lambda n: n(LAMBDA_INCREMENT)(m)
LAMBDA_SUB = lambda m: lambda n: n(LAMBDA_DECREMENT)(m) 
```

注意增量和减量将作为数字 `n` 的 `p` 参数传递，`m` 将是 `x` 参数。换句话说，`m` 将增加或减少与 `n` 中 `p` 调用次数相同的次数，这是，`n` 次。非常、非常美丽。

即使如此，零的减量是零的结果是，如果 `m > n`，则 `n - m` 将始终为零。

乘法和除法也是可能的，但它们对这个快速排序来说不是很有用。

#### 比较

实际上对于实现快速排序有用的整数运算是**比较**。让我们从定义告诉一个 Church 数是否为零的函数开始（思维练习：这为什么有效？）：

```
LAMBDA_EQZ = lambda n: n(lambda x: LAMBDA_FALSE)(LAMBDA_TRUE) 
```

只使用 **等于零** 操作，结合 **布尔** 和 **算术** 运算，我们可以定义其他运算：

+   `m <= n`: `(m - n) == 0` （记住：如果 `n > m`，则 `m - n = 0`）

+   `m == n`: `(m <= n) and (n <= m)`

+   `n < m`: `(m <= n) and not (m == n)`

或者，在 Python / lambda 演算中：

```
LAMBDA_LEQ = lambda m: lambda n: LAMBDA_EQZ(LAMBDA_SUB(m)(n))
LAMBDA_EQ = lambda m: lambda n: LAMBDA_AND(LAMBDA_LEQ(m)(n))(LAMBDA_LEQ(n)(m))
LAMBDA_LESS = lambda m: lambda n: LAMBDA_AND(LAMBDA_LEQ(m)(n))(LAMBDA_NOT(LAMBDA_EQ(m)(n))) 
```

#### 转换

如果我们想要将一个 Church 数转换为 Python 整数，我们只需要将增量函数作为第一个参数（`p`）传递，0 作为第二个参数：

```
def l2i(l):
    return l(lambda x: x + 1)(0) 
```

我们可以反过来做：我们可以多次 **增加** Church 零直到达到该数字：

```
def i2l(i):
    l = LAMBDA_ZERO
    for j in range(0, i):
        l = LAMBDA_INCREMENT(l)

    return l 
```

我们还可以定义函数将 Python 整数列表转换为 Church 数字列表，反之亦然：

```
def llist2pylist(L):
    return list(map(l2i, L))

def pylist2llist(L):
    return list(map(i2l, L)) 
```

#### 使用 Church 数字

因为我们可以将 Python 整数列表转换为 Church 数字列表，反之亦然，而且我们可以比较 Church 数字，所以我们可以对快速排序应用第一个改变：

```
def quicksort(A):
    if len(A) <= 1: return A
    L, R = partition_wrapper(A)
    p = car(R)
    L = quicksort(L)
    R = quicksort(cdr(R))
    return concat(L, concat([p], R))

def partition_wrapper(A):
    B = pylist2llist(A)
    L, R = partition(B)
    return llist2pylist(L), llist2pylist(R)

def partition(A):
    p = car(A)

    L, R = [], []

    for x in cdr(A):
        if l2b(LAMBDA_LESS(x)(p)):
            L, R = cons(x, L), R
        else:
            L, R = L, cons(x, R)
    L, R = L, cons(p, R)

    return L, R 
```

`partition`函数现在在**Church 数字列表**上运行。为了做到这一点，我们将`x < p`替换为`LAMBDA_LESS(x)(p)`，它返回一个 Church 布尔值，而不是`True`和`False`。我需要使用`l2b`将 Church 布尔值转换为 Python 布尔值，以便我们可以保持与`if`的兼容性。

函数`partition_wrapper`**调整**了新的`partition`，使其以采用 Python 整数的方式进行分区，但使用新的`partition`函数进行分区。

在接下来的章节中，我将通过 lambda 演算中的函数对类型、函数和操作符进行多次替换，就像我迄今为止所做的那样。我将尝试只改变每个步骤中相关的内容，必要时使用转换函数。

你可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/integers/lambdasort.py)看到它。

### 对和列表

我们的基本数据结构是**对**。这对真的是一对值，就像 Python 的大小为 2 的元组一样。在 Church 编码中，一对及其基本操作定义如下：

```
LAMBDA_CONS = lambda a: lambda b: lambda l: l(a)(b)
LAMBDA_CAR = lambda p: p(lambda a: lambda b: a)
LAMBDA_CDR = lambda p: p(lambda a: lambda b: b) 
```

第一个函数，`LAMBDA_CONS`，定义了这对。请注意，当将两个值作为其参数传递（例如`LAMBDA_CONS(15)(20)`）时，它将返回一个函数，该函数接受一个参数`l`并**返回**使用对元素作为**参数**的`l`调用（在我们的示例中，`l(15)(20)`）。也就是说：`LAMBDA_CONS(15)(20) = lambda l: l(15)(20)`。在 Python 和其他支持一等函数的语言中，这两个值存储在一个**[闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming))**中，我们甚至可以这样得到它们：

```
l = LAMBDA_CONS(15)(20)
a, b = (x.cell_contents for x in l.__closure__) # a = 15, b = 20 
```

关于`LAMBDA_CAR`和`LAMBDA_CDR`，它们分别返回对的**第一个**和**第二个**元素。

思维练习：试着理解为什么`LAMBDA_CAR`和`LAMBDA_CDR`有效！

#### 列表

如果你足够注意，你会注意到`car`、`cdr`和`cons`是我们用来定义在**列表**上操作的函数的相同名称。是的，它们是一样的！这是因为列表在 Church 编码中的实现方式。

**Church 列表**只是一对，其中：

+   第一个元素是列表的**第一个元素**

+   第二个元素是一个**剩余元素的列表**

这是一个递归定义，其中递归的基础（一个**空列表**）可以通过多种方式实现。在这里，我们使用布尔值`LAMBDA_FALSE`来表示一个空列表：

```
LAMBDA_EMPTY = LAMBDA_FALSE 
```

这样，具有值`[1, 2, 3]`的列表可以这样声明：

```
LAMBDA_CONS(1)(LAMBDA_CONS(2)(LAMBDA_CONS(3)(LAMBDA_FALSE))) 
```

在实践中，它们是递归对：

```
(1, (2, (3, LAMBDA_EMPTY))) 
```

如此多的括号！但请注意，此时，`LAMBDA_CAR`、`LAMBDA_CDR`和`LAMBDA_CONS`，当应用于**列表**时，具有与我们定义的用于操作Python列表的`car`、`cdr`和`cons`相同的行为：

+   `LAMBDA_CAR`返回第一个对的第一个元素，即列表的**第一个元素**（`1`）

+   `LAMBDA_CDR`返回第二对的第一个元素，即列表的**剩余部分**（`(2, (3, LAMBDA_EMPTY))`）

+   `LAMBDA_CONS`添加另一对，附加**另一个元素**

由于这些列表的定义是递归的，对它们的迭代也将以递归方式完成。我们将用于知道递归是否达到了结尾的函数是：

```
LAMBDA_ISEMPTY = lambda l: l(lambda h: lambda t: lambda d: LAMBDA_FALSE)(LAMBDA_TRUE) 
```

那就是：

+   如果`l`为空（等于`LAMBDA_EMPTY`），那么它返回第二个参数：`LAMBDA_TRUE`

+   如果`l`**不为空**，那么`l`是一对。 `l`以函数`(lambda h: lambda t: lambda d: LAMBDA_FALSE)`为参数调用。该函数丢弃所有内容并返回`LAMBDA_FALSE`。尝试模拟一下 ;-).

#### 转换

**对**可以从仅具有两个元素的 Python 列表转换为和从中转换。这样做的 pythonic 方法是使用大小为 2 的元组，但为了保持代码的一致性，我正在使用列表：

```
def l2p(l):
    return [LAMBDA_CAR(l), LAMBDA_CDR(l)]

def p2l(p):
    return LAMBDA_CONS(p[0])(p[1]) 
```

我们还可以转换 Python 列表和 Church 列表：

```
def ll2pl(l):
    if l2b(LAMBDA_ISEMPTY(l)): return []
    return [LAMBDA_CAR(l)] + ll2pl(LAMBDA_CDR(l))

def pl2ll(l):
    if len(l) == 0: return LAMBDA_EMPTY
    return LAMBDA_CONS(l[0])(pl2ll(l[1:])) 
```

#### 在`partition`中使用 Church 对

由于`partition`返回两个值（一个 Python 元组），因此我们可以在这里使用 Church 对：

```
 # before:
    return L, R

    # after:
    return LAMBDA_CONS(L)(R) 
```

#### 在`partition`中使用 Church 列表

让我们在快速排序中使用 Church 列表！首先，我们将转换`partition`以在 Church 列表上操作。目前，我们的情况是这样的：

```
def partition(A):
    p = car(A)

    L, R = [], []

    for x in cdr(A):
        if l2b(LAMBDA_LESS(x)(p)):
            L, R = cons(x, L), R
        else:
            L, R = L, cons(x, R)
    L, R = L, cons(p, R)

    return L, R 
```

因此，我们需要将`car`、`cdr`、`cons`和`[]`替换为它们在 lambda 演算中的对应物：

```
def partition(A):
    p = LAMBDA_CAR(A)
    L, R = LAMBDA_EMPTY, LAMBDA_EMPTY

    for x in lliterator(LAMBDA_CDR(A)):
        if l2b(LAMBDA_LESS(x)(p)):
            L, R = LAMBDA_CONS(x)(L), R
        else:
            L, R = L, LAMBDA_CONS(x)(R)
    L, R = L, LAMBDA_CONS(p)(R)

    return LAMBDA_CONS(L)(R) 
```

请注意，我创建了生成器`lliterator`来迭代 Church 列表：

```
def lliterator(l):
    while not l2b(LAMBDA_ISEMPTY(l)):
        yield LAMBDA_CAR(l)
        l = LAMBDA_CDR(l) 
```

#### 在`quicksort`中使用 Church 列表

现在我们将把**Church 列表**添加到函数`quicksort`中！我们仍然需要为 Church 列表定义函数`concat`。我们可以递归地实现它：

+   如果左侧的列表**为空**，我们需要使用右侧的列表

+   如果左侧的列表**不为空**，它将返回一个新的列表，其中：

    +   第一个元素是左侧列表的**第一个元素**（`car`）

    +   剩余列表是左侧列表的**剩余部分**（`cdr`）与右侧列表的**列表**的连接

那就是（注意柯里化）：

```
def LAMBDA_CONCAT(l1):
    def _LAMBDA_CONCAT(l2):
        if l2b(LAMBDA_ISEMPTY(LAMBDA_CDR(l1))):
            return LAMBDA_CONS(LAMBDA_CAR(l1))(l2)
        else:
            return LAMBDA_CONS(LAMBDA_CAR(l1))(LAMBDA_CONCAT(LAMBDA_CDR(l1))(l2))
    return _LAMBDA_CONCAT 
```

这与其他在单个表达式中编写的操作有些不同。剧透：我们将进一步处理这个！

一旦定义了`concat`，我们就可以在`quicksort`中用 Church 列表操作替换所有 Python 列表操作：

```
def quicksort(A):
    # len(A) <= 1
    if l2b(LAMBDA_ISEMPTY(A)): return A
    if l2b(LAMBDA_ISEMPTY(LAMBDA_CDR(A))): return A

    L, R = partition(A)
    p = LAMBDA_CAR(R)

    L = quicksort(L)
    R = quicksort(LAMBDA_CDR(R))

    return LAMBDA_IF(LAMBDA_ISEMPTY(L))(LAMBDA_CONS(p)(R))(LAMBDA_CONCAT(L)(LAMBDA_CONS(p)(R))) 
```

你可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/pairs-lists/lambdasort.py)看到它。

## 用递归函数替换循环

正如在 lambda 演算中**没有任何状态**，我们不能像在命令式语言中那样使用循环，即重复代码片段并更改变量的状态。

由于缺乏状态，我们不是更改现有值，而是返回一个**新**值，就像我们替换快速排序的标准行为以返回已排序列表而不是对原始列表进行排序时一样。

即使我们在支持函数式和命令式范式的语言中编写代码，例如Python，如果我们限制自己以函数式方式编写代码，我们不能使用循环。

我们如何解决使用循环解决的问题？根据情况有许多解决方案，例如，我们可以使用`reduce`、列表推导、`map`、`filter`、递归函数等等。在这个快速排序中，我们只有**一个循环**，在`partition`中。我们将其替换为一个**递归函数**。

### 用`while`替换`for`

目前，循环是这样的：

```
p = LAMBDA_CAR(A)
L, R = LAMBDA_EMPTY, LAMBDA_EMPTY

for x in lliterator(LAMBDA_CDR(A)):
    if l2b(LAMBDA_LESS(x)(p)):
        L, R = LAMBDA_CONS(x)(L), R
    else:
        L, R = L, LAMBDA_CONS(x)(R) 
```

删除迭代器`lliterator`并将`for`替换为`while`后，我们得到了这样的结果：

```
p = LAMBDA_CAR(A)
L, R = LAMBDA_EMPTY, LAMBDA_EMPTY

S = LAMBDA_CDR(A)
while True:
    if l2b(LAMBDA_ISEMPTY(S)): break
    x = LAMBDA_CAR(S)
    if l2b(LAMBDA_LESS(x)(p)):
        L, R = LAMBDA_CONS(x)(L), R
    else:
        L, R = L, LAMBDA_CONS(x)(R)
    S = LAMBDA_CDR(S) 
```

换句话说：初始时`S`等于输入列表去除第一个元素（枢轴`p`）后的内容。`while`的每次迭代**移除**`S`中的一个值，并将其存储在`x`中。如果`x < p`，我们将`x`添加到`L`的开头，否则我们将其添加到`R`的末尾。停止条件是当列表`S`为空时。

从现在开始，我们可以确定哪些元素将有助于编写这个循环作为一个递归函数：

+   输入**`L`和`R`**，初始为空的Church列表；

+   输入**`S`**，初始为`LAMBDA_CDR(A)`；

+   输出，即循环结束时的`L`和`R`的值；

+   停止条件，即`S`为空时。

### 将`while`转换为递归

很好，到目前为止，我们已经足以编写一个**递归函数**（我在这里称之为`_partition`），给定`while`循环代码：

```
p = LAMBDA_CAR(A)
L, R = LAMBDA_EMPTY, LAMBDA_EMPTY

S = LAMBDA_CDR(A)

def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return L, R
    x = LAMBDA_CAR(S)
    if l2b(LAMBDA_LESS(x)(p)):
        nL, nR = LAMBDA_CONS(x)(L), R
    else:
        nL, nR = L, LAMBDA_CONS(x)(R)
    S = LAMBDA_CDR(S)
    return _partition(S, nL, nR)

nL, nR = _partition(S, L, R) 
```

注意，`_partition`不会改变输入`L`和`R`的状态。与其*改变*原始输入数据，它返回*新*数据，这些数据存储在变量`nL`和`nR`中，它们是下一次递归调用的`L`和`R`参数。

我们还可以使该函数返回Church对而不是元组：

```
p = LAMBDA_CAR(A)
L, R = LAMBDA_EMPTY, LAMBDA_EMPTY

S = LAMBDA_CDR(A)

def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)
    x = LAMBDA_CAR(S)
    if l2b(LAMBDA_LESS(x)(p)):
        nL, nR = LAMBDA_CONS(x)(L), R
    else:
        nL, nR = L, LAMBDA_CONS(x)(R)
    S = LAMBDA_CDR(S)
    return _partition(S, L, R)

LR = _partition(S, L, R)
nL, nR = LAMBDA_CAR(LR), LAMBDA_CDR(LR) 
```

你可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/recursion/lambdasort.py)看到它。

## 用`let`替换变量

[let表达式](https://zh.wikipedia.org/wiki/Let%E8%A1%A8%E8%BE%BE%E5%BC%8F)允许我们在一个**作用域**内将一个值定义给一个变量，以便其值**永远不会被改变**。在Python中，这个概念意义不大，但是不同的语言以许多方式实现了它。我会给你展示一些例子。

从**Kotlin**开始，`let`是可以由任何对象调用的方法，因此我们可以为其分配一个临时名称：

```
val x = 2.let { a -> 
   3.let { b ->
      a + b * a
   }
} 
```

在**Haskell**中，`let`是一个表达式，其中前半部分是值的赋值，后半部分是我们要评估的表达式：

```
x = let a = 2
        b = 3
    in a + b * a 
```

在**[Hy](http://hylang.org)**（具有Lisp语法的Python）中，它与Haskell非常接近：首先我们将值赋给变量，然后声明将使用它们的表达式：

```
(setv  x  (let  [  a  2  b  3  ]  (+  a  (*  b  a))  )  ) 
```

（上述三个示例中的`x`都将是8）

这种构造在函数式语言中非常常见，因为它们的变量在作用域内有一个**固定的值**。此外，它们很容易使用**lambda演算**编写。我们可以像这样重写上面的示例：

```
def _f(a, b):
   return a + b * a
x = _f(2, 3)

# or using lambda and currying: 
x = (lambda a: lambda b: a + b * a)(2)(3) 
```

请注意，与以前一样，我们将 `a = 2` 和 `b = 3` 赋给变量，然后计算 `a + b * a`。

我们在这一步的任务是将**所有不是参数或常量的变量**放入 let 中，这样它们就可以在 lambda 演算中使用。

从现在开始，代码将变得非常难以阅读，但让我们专注于一个如何进行替换的示例。在我们之前定义的 `_partition` 函数中，我们将用 let 替换 `x`。到目前为止，这个函数看起来像这样：

```
def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)
    x = LAMBDA_CAR(S)
    if l2b(LAMBDA_LESS(x)(p)):
        nL, nR = LAMBDA_CONS(x)(L), R
    else:
        nL, nR = L, LAMBDA_CONS(x)(R)
    S = LAMBDA_CDR(S)
    return _partition(S, L, R) 
```

这里的 `if` 只改变 `L` 和 `R` 的值，我们可以将其写成一个**if表达式**：

```
def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)
    x = LAMBDA_CAR(S)
    nL, nR = (LAMBDA_CONS(x)(L), R) if l2b(LAMBDA_LESS(x)(p)) else (L, LAMBDA_CONS(x)(R))

    S = LAMBDA_CDR(S)
    return _partition(S, nL, nR) 
```

我们只是为了被那个 if 表达式使用而评估 `x`，所以我们可以在这里使用 `let`：

+   **赋值**：`x = LAMBDA_CAR(S)`

+   **表达式**：我们刚刚定义的 if 表达式

为此，让我们声明一个名为 `_partition2` 的新函数，它以 `x` 为参数，并在之后立即调用它，将 `x = LAMBDA_CAR(S)` 作为参数传递：

```
def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)

    def _partition2(x):
        return (LAMBDA_CONS(x)(L), R) if l2b(LAMBDA_LESS(x)(p)) else (L, LAMBDA_CONS(x)(R))

    nL, nR = _partition2(LAMBDA_CAR(S))

    S = LAMBDA_CDR(S)
    return _partition(S, nL, nR) 
```

我们有一个 `let`！我们还可以用一个**Church 对**替换元组：

```
def _partition(S, L, R):
    if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)

    def _partition2(x):
        return LAMBDA_CONS(LAMBDA_CONS(x)(L))(R) if l2b(LAMBDA_LESS(x)(p)) else LAMBDA_CONS(L)(LAMBDA_CONS(x)(R))

    LR = _partition2(LAMBDA_CAR(S))
    nL, nR = LAMBDA_CAR(LR), LAMBDA_CDR(LR)

    return _partition(LAMBDA_CDR(S), nL, nR) 
```

您可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/let/lambdasort.py)看到它。

## 使用 `lambda` 重写函数

到目前为止，将所有**变量替换为 let**，`partition` 和 `quicksort` 函数不再具有变量。它们只有一些 `if`、返回表达式以及由 lets 使用的**内部函数**的定义（它们具有相同的特征）。

看一看这个（是的，只看一看，因为这段代码是无法阅读的）：

```
def quicksort(A):
    if l2b(LAMBDA_ISEMPTY(A)): return A
    if l2b(LAMBDA_ISEMPTY(LAMBDA_CDR(A))): return A

    def _quicksort(A, LR):
        return LAMBDA_IF(LAMBDA_ISEMPTY(quicksort(LAMBDA_CAR(LR))))(LAMBDA_CONS(LAMBDA_CAR(LAMBDA_CDR(LR)))(quicksort(LAMBDA_CDR(LAMBDA_CDR(LR)))))(LAMBDA_CONCAT(quicksort(LAMBDA_CAR(LR)))(LAMBDA_CONS(LAMBDA_CAR(LAMBDA_CDR(LR)))(quicksort(LAMBDA_CDR(LAMBDA_CDR(LR))))))

    return _quicksort(A, partition(A))

def partition(A):
    def _partition(S, L, R, p):
        if l2b(LAMBDA_ISEMPTY(S)): return LAMBDA_CONS(L)(R)
        def _partition2(x, L, R, p):
            if l2b(LAMBDA_LESS(x)(p)): return LAMBDA_CONS(LAMBDA_CONS(x)(L))(R)
            else: return LAMBDA_CONS(L)(LAMBDA_CONS(x)(R))

        def _partition3(S, LR, p):
            return _partition(LAMBDA_CDR(S), LAMBDA_CAR(LR), LAMBDA_CDR(LR), p)

        return _partition3(S, _partition2(LAMBDA_CAR(S), L, R, p), p)

    def _partition4(LR):
        return LAMBDA_CONS(LAMBDA_CAR(LR))(LAMBDA_CONS(LAMBDA_CAR(A))(LAMBDA_CDR(LR)))

    return _partition4(_partition(LAMBDA_CDR(A), LAMBDA_EMPTY, LAMBDA_EMPTY, LAMBDA_CAR(A))) 
```

我们可以用**if表达式**替换所有的 `if`，并用我们使用 Church 布尔值定义的 `LAMBDA_IF` 替换那些 if 表达式。此外，**内部函数**可以使用 `lambda` 而不是 `def` 来定义，因为它们只有**返回表达式**。现在我们有了这段糟糕的代码：

```
def quicksort(A):
    _quicksort = lambda A: lambda LR: LAMBDA_CONCAT(quicksort(LAMBDA_CAR(LR)))(LAMBDA_CONS(LAMBDA_CAR(LAMBDA_CDR(LR)))(quicksort(LAMBDA_CDR(LAMBDA_CDR(LR)))))

    _quicksort2 = lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(A))(lambda A: A)(lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(LAMBDA_CDR(A)))(A)(_quicksort(A)(partition(A))))

    return _quicksort2(A)(A)

def partition(A):
    _partition2 = lambda x: lambda L: lambda R: lambda p: LAMBDA_IF(LAMBDA_LESS(x)(p))(LAMBDA_CONS(LAMBDA_CONS(x)(L))(R))(LAMBDA_CONS(L)(LAMBDA_CONS(x)(R)))

    _partition3 = lambda S: lambda LR: lambda p: _partition(LAMBDA_CDR(S))(LAMBDA_CAR(LR))(LAMBDA_CDR(LR))(p)

    _partition = (lambda S: LAMBDA_IF(LAMBDA_ISEMPTY(S))(lambda L: lambda R: lambda p: LAMBDA_CONS(L)(R))(lambda L: lambda R: lambda p: _partition3(S)(_partition2(LAMBDA_CAR(S))(L)(R)(p))(p)))

    _partition4 = (lambda A: lambda LR: LAMBDA_CONS(LAMBDA_CAR(LR))(LAMBDA_CONS(LAMBDA_CAR(A))(LAMBDA_CDR(LR))))(A)

    return _partition4(_partition(LAMBDA_CDR(A))(LAMBDA_EMPTY)(LAMBDA_EMPTY)(LAMBDA_CAR(A))) 
```

在这种情况下，内部函数（即使它们现在是变量）可以是**常量**。这样，它们就不需要在 `quicksort` 和 `partition` 中，后者只能有**返回表达式**。然后，它们可以使用 `lambda` 而不是 `def` 来编写：

```
_quicksort = lambda A: lambda LR: LAMBDA_CONCAT(quicksort(LAMBDA_CAR(LR)))(LAMBDA_CONS(LAMBDA_CAR(LAMBDA_CDR(LR)))(quicksort(LAMBDA_CDR(LAMBDA_CDR(LR)))))
_quicksort2 = lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(A))(lambda A: A)(lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(LAMBDA_CDR(A)))(A)(_quicksort(A)(partition(A))))
quicksort = lambda A: _quicksort2(A)(A)
_partition2 = lambda x: lambda L: lambda R: lambda p: LAMBDA_IF(LAMBDA_LESS(x)(p))(LAMBDA_CONS(LAMBDA_CONS(x)(L))(R))(LAMBDA_CONS(L)(LAMBDA_CONS(x)(R)))
_partition3 = lambda S: lambda LR: lambda p: _partition(LAMBDA_CDR(S))(LAMBDA_CAR(LR))(LAMBDA_CDR(LR))(p)
_partition = (lambda S: LAMBDA_IF(LAMBDA_ISEMPTY(S))(lambda L: lambda R: lambda p: LAMBDA_CONS(L)(R))(lambda L: lambda R: lambda p: _partition3(S)(_partition2(LAMBDA_CAR(S))(L)(R)(p))(p)))
_partition4 = (lambda A: lambda LR: LAMBDA_CONS(LAMBDA_CAR(LR))(LAMBDA_CONS(LAMBDA_CAR(A))(LAMBDA_CDR(LR))))
partition = lambda A:_partition4(A)(_partition(LAMBDA_CDR(A))(LAMBDA_EMPTY)(LAMBDA_EMPTY)(LAMBDA_CAR(A))) 
```

### 递归和 Y 组合子

其中一些 lambda 函数是**递归**的：

+   `quicksort` 调用 `_quicksort2`，后者调用 `quicksort`

+   `_partition` 调用 `_partition3`，后者调用 `_partition`

然而，lambda 演算的一个特性是，一个函数**不需要有一个名称**。但是一个函数如何在不知道它的名称的情况下引用自身呢？答案是**[Y 组合子](https://en.wikipedia.org/wiki/Fixed-point_combinator#Fixed-point_combinators_in_lambda_calculus)**。

为了说明 Y 组合子，请看这个阶乘函数：

```
def fac(n):
   return 1 if n == 0 else n * fac(n-1)

# using lambda: fac = lambda n: 1 if n == 0 else n * fac(n-1) 
```

然后我们可以使用 Y 组合子替换 `fac` 调用：

```
fac = (lambda f: f(f))(lambda f: lambda n: 1 if n == 0 else n * f(f)(n-1))

# we even don't need to name fac. This expression calculates 5! = 120 recursively: (lambda f: f(f))(lambda f: lambda n: 1 if n == 0 else n * f(f)(n-1))(5) 
```

在那个东西里发生了什么？请注意，我们有一个函数`(lambda f: lambda n: 1 if n == 0 else n * f(f)(n-1))`，非常类似于原始的`fac`，**除了**它接受一个参数`f`并调用`f(f)`而不是`fac`。这里Y组合子的思想是，`f`将始终是**相同的函数**，并且它递归地将**自己作为参数**传递，以便递归调用产生另一个递归调用。谁将保证递归的**基础**是`(lambda f: f(f))`，它将为该函数的第一个传递提供第一个传递。

心理练习：尝试模拟`fac(2)`，看看神奇的事情发生了。

#### 使用Y组合子

好的，现在我们可以在`quicksort`中用Y组合子替换递归调用。现在它看起来是这样的：

```
_quicksort2 = lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(A))(lambda A: A)(lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(LAMBDA_CDR(A)))(A)(_quicksort(A)(partition(A))))
quicksort = lambda A: _quicksort2(A)(A) 
```

如果我们用Y组合子替换它：

```
_quicksort2 = lambda r: lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(A))(lambda A: A)(lambda A: LAMBDA_IF(LAMBDA_ISEMPTY(LAMBDA_CDR(A)))(A)(_quicksort(r)(A)(partition(A))))
quicksort = (lambda r: r(r)) lambda A: _quicksort2(r)(A)(A) 
```

心理练习：`quicksort`调用**不在**`quicksort`本身中，而在`_quicksort2`中（由`quicksort`调用）。你能想象出Y组合子在这种情况下是如何使用的吗？

你可以在[这里](https://github.com/lucasoshiro/lambdasort/blob/y-combinator/lambdasort.py)看到它。

## 扩展一切！

在这一点上，所有的**值**、**数据结构**和`if`都是函数。此外，这些函数和所有其他函数都是可以写成单个表达式的**值**。

我们在这里的工作基本上是，将**所有的常量**替换为它们的**值**，以便`quicksort`可以成为一个单一的表达式。这可以通过文本编辑器的文本替换工具来完成。

最后我们有了这个可怕的东西：

`quicksort = (lambda r: r(r))(lambda r: lambda A: (lambda r: lambda A: (lambda c: lambda t: lambda e: c(t)(e))((lambda l: l(lambda h: lambda t: lambda d: (lambda a: lambda b: b))((lambda a: lambda b: a)))(A))(lambda A: A)(lambda A: (lambda c: lambda t: lambda e: c(t)(e))((lambda l: l(lambda h: lambda t: lambda d: (lambda a: lambda b: b))((lambda a: lambda b: a)))((lambda p: p(lambda a: lambda b: b))(A)))(A)((lambda r: lambda A: lambda LR: (lambda c: lambda t: lambda e: c(t)(e))((lambda l: l(lambda h: lambda t: lambda d: (lambda a: lambda b: b))((lambda a: lambda b: a)))(r(r)((lambda p: p(lambda a: lambda b: a))(LR))))((lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))((lambda p: p(lambda a: lambda b: b))(LR)))(r(r)((lambda p: p(lambda a: lambda b: b))((lambda p: p(lambda a: lambda b: b))(LR)))))(((lambda r: r(r)) (lambda r: lambda l1: (lambda c: lambda t: lambda e: c(t)(e))((lambda l: l(lambda h: lambda t: lambda d: (lambda a: lambda b: b))((lambda a: lambda b: a)))((lambda p: p(lambda a: lambda b: b))(l1)))(lambda l2: (lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))(l1))(l2))((lambda r: lambda l2: (lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))(l1))(r(r)((lambda p: p(lambda a: lambda b: b))(l1))(l2)))(r))))(r(r)((lambda p: p(lambda a: lambda b: a))(LR)))((lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))((lambda p: p(lambda a: lambda b: b))(LR)))(r(r)((lambda p: p(lambda a: lambda b: b))((lambda p: p(lambda a: lambda b: b))(LR)))))))(r)(A)((lambda A:((lambda A: lambda LR: (lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))(LR))((lambda a: lambda b: lambda l: l(a)(b))((lambda p: p(lambda a: lambda b: a))(A))((lambda p: p(lambda a: lambda b: b))(LR)))))(A)(((lambda r: r(r))(lambda r: lambda S: (lambda c: lambda t: lambda e: c(t)(e))((lambda l: l(lambda h: lambda t: lambda d: (lambda a: lambda b: b))((lambda a: lambda b: a)))(S))(lambda L: lambda R: lambda p: (lambda a: lambda b: lambda l: l(a)(b))(L)(R))(lambda L: lambda R: lambda p: (lambda r: lambda S: lambda LR: lambda p: r(r)((lambda p: p(lambda a: lambda b: b))(S))((lambda p: p(lambda a: lambda b: a))(LR))((lambda p: p(lambda a: lambda b: b))(LR))(p))(r)(S)((lambda x: lambda L: lambda R: lambda p: (lambda c: lambda t: lambda e: c(t)(e))((lambda m: lambda n: (lambda a: lambda b: a(b)((lambda a: lambda b: b)))((lambda m: lambda n: (lambda n: n(lambda x: (lambda a: lambda b: b))((lambda a: lambda b: a)))((lambda m: lambda n: n((lambda n: lambda f: lambda x: n(lambda g: lambda h: h(g(f)))(lambda y: x)(lambda y: y)))(m))(m)(n)))(m)(n))((lambda a: a((lambda a: lambda b: b))((lambda a: lambda b: a)))((lambda m

### 使用lambdasort

当然，我们不能单独在列表中使用这个`quicksort`，因为它是以Church编码方式操作的。我们需要一个包装器来将Python类型转换为Church编码，使用`quicksort`对Church列表进行排序，然后将其转换回Python列表。我们将使用之前的函数。

```
def quicksort_wrapper(A):
    church = pl2ll([i2l(x) for x in A])
    sorted_church = quicksort(church)
    return [l2i(x) for x in ll2pl(sorted_church)] 
```

现在你可以使用`quicksort_wrapper`来对你的列表进行排序，它将使用我们的lambdasort作为后端：

```
>>> from lambdasort import quicksort_wrapper
>>> x = [22, 33, 11, 55, 99, 11, 33, 77, 44]
>>> quicksort_wrapper(x)
[11, 11, 22, 33, 33, 44, 55, 77, 99] 
```

## 最后的思考

我在2017年（我大学的第三年）只用了两天写出了lambdasort，在参加了[古比教授](https://memorial.ime.usp.br/homenageados/5)关于lambda演算和Y组合子的课程后。他曾经提到过编程无所不用其极。我觉得那太令人印象深刻了，我想做点类似的事情，于是挑战自己写了一些比fizzbuzz更*困难*的东西，所以我们就在这里！

写下这些真的很有趣，起初我没有注意到我只用了两天就学到了这么多知识，而我花了几年时间才最终写下这篇解释我所做的事情的文字。所以，谢谢你阅读！

最后但并非最不重要的，英语不是我的母语，我正在尽最大努力保持我的个人页面同时用葡萄牙语和英语编写。所以，如果你发现有什么不对或者对这里的任何事情有意见，请随时[提出问题](https://github.com/lucasoshiro/lucasoshiro.github.io/issues)。
