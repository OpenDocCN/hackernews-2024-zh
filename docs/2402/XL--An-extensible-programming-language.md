<!--yml

category: 未分类

date: 2024-05-27 15:03:43

-->

# XL：一种可扩展的编程语言

> 来源：[https://xlr.sourceforge.io/](https://xlr.sourceforge.io/)

一个值的生命周期是其存在于程序中的时间长度，换句话说，是其 [创建](#creation) 到 [销毁](#destruction) 之间的时间。

如果实体已创建但尚未销毁，则称其为 *存活*。否则称其为 *死亡*。

|  | 有些实体可能在当前上下文中存活但无法从中访问，因为它们不可见。这适用于声明在调用者上下文中的变量的情况。 |
| --- | --- |

编译器关于实体 `X` 的生命周期信息表示为编译时常量 `lifetime X`。生命周期值带有部分顺序 `<`，使得表达式 `lifetime X < lifetime Y` 为 `true`，是编译器保证 `Y` 在 `X` 存活期间始终存活的保证。`lifetime X > lifetime Y` 或 `lifetime X < lifetime Y` 都可能为假。这种 `lifetime` 特性用于实现类似 [Rust](https://doc.rust-lang.org/1.8.0/book/ownership.html) 的 [访问类型限制](#lifetime)，即以零运行时成本实现内存安全的一种方式。

XL 值的生命周期可归类为以下几种：

+   *全局值* 在程序的 [声明阶段](#declaration-phase) 开始，正好在其 [评估阶段](#evaluation-phase) 之前变得存活，并且在该评估阶段结束之前保持存活。全局值通常在程序评估之前静态预分配在内存的保留区域中，由一个称为链接器的程序执行。

+   *本地值* 在评估阶段与 [模式匹配](#pattern-matching) 相对应的声明体的 [局部声明阶段](#nested-declarations) 中变为存活。本地值通常动态分配在为每个线程分配的 [调用栈](https://en.wikipedia.org/wiki/Call_stack) 上。该栈具有有限的 "深度"，这可能限制程序允许的递归深度。

+   *动态值* 是使用 "堆" 动态分配的，并且只要某些其他值 [拥有它们](#ownership)，它们就会保持存活。拥有它们的值可以是全局值、本地值或动态值。堆通常是程序中最大的可用内存空间。XL 提供了多种设施来帮助您管理动态分配，包括在这是有效管理策略时构建 [垃圾收集器](#garbage-collection) 的设施。

+   *临时值* 在表达式评估期间创建，并且在被使用后可以立即丢弃。例如，假设 `x+y` 已定义，则表达式 `a+b+c+d` 将被处理为 `((a+b)+c)+d`，并且在评估 `(a+b)+c` 后，评估 `a+b` 的结果可以被销毁。临时值通常也分配在堆栈上。

例如，考虑以下代码片段：

```
use XL.CONSOLE.TEXT_IO ***(1)**
use XL.TEXT.FORMAT

print "Starting printing Fibonacci sequences" ***(2)**

fib 0 is 1 ***(3)**
fib 1 is 1
fib N is (fib(N-1) + fib(N-2)) ***(4)**

for I in 1..5 loop ***(5)**
    F is fib I ***(6)**
    print format("Fib(%1) is %2", I, F) ***(7)*********
```

******| ***1*** | `use` 语句导入其他文件中定义的全局值。在这里，第一个语句导入了 `XL` 模块及其子模块 `XL.CONSOLE` 和第三级子模块 `XL.CONSOLE.TEXT_IO`，第二个语句导入了 `XL.TEXT` 和 `XL.TEXT.FORMAT`（假设 `XL` 已经被导入） |

| ***2*** | `print` 的声明是在 `XL.CONSOLE.TEXT_IO` 中定义的全局值。这个 `print` 语句是程序评估期间执行的第一件事。它的评估将调用 `print` 实现的代码，从而在调用堆栈上增加一个新的上下文。正如我们之前指出的（见[#print_instances](#print_instances)），这可能涉及进一步调用，使调用堆栈更深 |
| --- | --- |
| ***3*** | 带有 `fib` 前缀的三个定义是三个不同的全局值，即使由于动态调度的原因，它们可能被认为是实现单一实体。评估 `fib N` 或 `fib I` 可能需要考虑所有三个全局值作为候选。 |

| ***4*** | 表达式 `(fib(N-1)+fib(N-2))` 的评估将需要创建一些临时变量，例如计算 `N-1` 或 `N-2`。临时变量可能会在不再需要时被销毁。例如，评估表达式的代码可能类似于以下代码，其中 `tmp1`、`tmp2`、`tmp3`、`tmp4` 和 `tmp5` 是所需的临时变量：

```
tmp1 is N-1
tmp2 is fib(tmp1)
delete tmp1
tmp3 is N-2
tmp4 is fib(tmp3)
delete tmp3
tmp5 is tmp2+tmp4
delete tmp2
delete tmp4
tmp5
```

|

| ***5*** | `for` 循环创建了一个名为 `I` 的局部变量，该变量将依次取值 `1`、`2`、`3`、`4`、`5`。`I` 的值只在循环的一个迭代中有效。换句话说，执行将与以下内容相同（对于不同值的 `I` 使用一个[closure](#closure)）：

```
tmpBody is
    F is fib I
    print format("Fib(%1) is %2", I, F)
{ I is 1 } ( tmpBody )
{ I is 2 } ( tmpBody )
{ I is 3 } ( tmpBody )
{ I is 4 } ( tmpBody )
{ I is 5 } ( tmpBody )
```

|

| ***6*** | `F is fib I` 定义的值是一个局部值，将在评估封闭块的持续时间内有效。它将在每个块的末尾被销毁。换句话说，在上面例子中，对 `tmpBody` 的更准确的描述将在末尾有一个 `delete F` 语句，如下所示：

```
tmpBody is
    F is fib I
    print format("Fib(%1) is %2", I, F)
    delete F
```

|

| ***7*** | 对于习惯于 C 或 C++ 的人来说，可能会感到惊讶的是，XL 并不要求在这一点之前调用 `fib I`。定义 `F is fib I` 可以解读为常量初始化为 `fib I`，或者作为返回 `fib I` 的函数。如果编译器可以确定调用评估 `F` 的结果始终相同，则允许实现记忆化，即在类似 `F+F` 的表达式中存储第一次计算 `F` 的值 |
| --- | --- |

|  | 通常，优秀的编译器还会利用机器 *寄存器*，作为处理器中非常快速的存储，作为逻辑上属于调用栈的值的缓存。一般来说，我们只会讨论堆栈，理解上包括适用的寄存器。 |******  *******创建* 是为使用准备值的过程。XL 语言规则保证在值存活时，通过在适当时间调用由程序员提供的代码来保证值永不未定义。

值 `V` 的生命周期始于隐式评估 *创建* 语句 `create V`。特别是当您创建一个未初始化的局部变量时会发生这种情况。

例如，请考虑以下代码：

```
Add Z:complex is
   T:complex
   Z+T
```

上述代码实际上等同于以下代码，其中隐式生成的代码已放在括号内：

```
Add Z:complex is
   T:complex
   (create T)
   Z+T
```

通过创建，[容器](#generic-containers) 中的值通过创建接收到明确定义的值。例如，如果创建一个 `array[1..5] of complex`，则在访问它们之前，这 5 个 `complex` 值已经被创建。

`create` 操作必须带有单一的 `out` 参数。定义体开始前，所有该 `out` 参数中的值都已被创建。例如，请考虑以下内容：

```
create Z:out complex is
    print "Creator called"
```

此代码不会导致未初始化的值，因为实际上等同于以下内容：

```
create Z:out complex is
   (create Z.Re)        // Implicit creation of complex fields
   (create Z.Im)
   print "Creator called"
```

[基本类型](#basic-types) 的 `create` 操作被称为 *零初始化* 它们，如下所示：

+   `type` 和 `nil` 值接收 `nil`。

+   所有整数类型接收值 `0`。

+   所有实数类型接收值 `0.0`。

+   所有字符类型接收 `character 0`。

+   `text` 接收 `""`。

+   `boolean` 接收 `false`。

程序员可以调用 `create` 操作，因此如果多次调用它，它必须表现正确。这是因为默认情况下 `out` 参数在调用之前会被[销毁](#destruction)。

例如，如果你显式调用 `create` 如下代码：

这实际上等同于以下代码，其中隐式语句放在括号内：

```
Z:complex
(create Z)      // because of the declaration above
(delete Z)      // because Z is passed as out argument to 'create'
create Z
```

编译器可能能够在这些情况下省略一些调用。另一个重要的情况是编译器应该省略创建调用的情况，称为 *构造*，基于为类型定义的形状。例如，我们如下定义了一个 `complex` 类型：

```
type complex is matching complex(Re:real, Im:real)
```

这意味着类似 `complex(2.3, 5.6)` 的形状是一个 `complex`。这也意味着 *唯一* 构建任意 `complex` 值的基本方法是创建这样的形状。因此，不可能在 `complex` 中有未初始化的元素，例如 `complex(X, Y)` 除非 `X` 和 `Y` 都是有效的 `real` 值。

然而，在某些情况下，*默认初始化* 复杂值是有必要的，例如创建容器时，这时 `create` 操作就显得必要。

明确使用类型给定的形状称为该类型的*构造函数*，可以在定义中或在具有初始值的变量声明中使用。构造函数永远不会失败或构建部分对象。如果在评估过程中一个参数返回[error](#errors)，那么该`error`值将无法匹配预期的参数，除非构造函数被编写为接受`error`值。

开发者经常会提供创建给定类型值的替代方法。这些替代助手除了返回该类型的值外，并无其他不同。

例如，对于`complex`类型，你可以创建一个虚数单位`i`，但需要一个构造函数来定义它。你还可以识别常见的表达式，如`2+3i`并将其转换为构造函数。

```
i   is complex(0.0, 1.0)

syntax { POSTFIX 190 i }
Re:real + Im:real i                 is complex(Re, Im)      // Case 1
Re:real + Im:real * [[i]]           is complex(Re, Im)      // Case 2
Re:real + [[i]] * Im:real           is complex(Re, Im)      // Case 3
Re:real as complex                  is complex(Re, 0.0)     // Case 4
X:complex + Y:complex as complex    is ...

2 + 3i              // Calls case 1 (with explicit concersions to real)
2 + 3 * i           // Calls case 2 (with explicit conversions to real)
2 + i * 3           // Calls case 3
2 + 3i + 5.2        // Calls case 4 to convert 5.2 to complex(5.2, 0.0)
2 + 3i + 5          // Error: Two implicit conversions (exercise: fix it)
```

唯一创建类型的基本方法是通过构造函数来说明，如下面的代码所示：

```
type large is matching (N when N > 42)
A:large := 44   // OK
B:large := 99.1 // OK
C:large         // Error
to create V:out large is
    print "Creator called"
```

`A:large`和`B:large`的初始化是可接受的，因为可以验证初始值是否匹配`large`模式。然而，`C:large`的定义是不可接受的，尽管存在`create`操作。原因是无法`create V.N`，首先因为无法推断要使用的类型，其次因为如果选择像`integer`这样的类型，那么默认初始值`0`将不会匹配模式。

但是，通过在创建者内部使用`large`类型的构造函数，可以修复上述代码，这意味着提供与类型模式匹配的值。以下是`create`函数的几乎可接受版本：

```
type large is matching (N when N > 42)
C:large         // OK
to create V:out large is
    print "Creator called"
    V := 44     // Creator for a large value
```

|  | 上述内容*几乎*可以接受，因为它调用了`print`，而`print`可能会失败。出于效率原因，默认情况下不允许`create`的签名返回`error`。如果有必要，你必须显式地标记`create`为返回`mayfail`。 |
| --- | --- |

类型实现可能会*隐藏*在[模块接口](#modules)中，在这种情况下，模块接口还应该提供一些函数来创建该类型的元素。以下示例说明了基于Unix风格文件描述符的`file`接口：

```
module MY_FILE with
    type file
    to open(Name:text) as file
    to close F:io file

module MY_FILE is
    type file matches file(fd:integer)
    to open(Name:text) as file is
        fd:integer := libc.open(Name, libc.O_RDONLY)
        file(fd)
    to close F:inout file is
        if fd >= 0 then
            libc.close(F.fd)
            F.fd := -2
    to delete F:inout file is close F    // Destruction, see below
```

如果接口提供了`create`操作，那么它必须准备接受所有其他模块函数中以默认创建的值为输入。然而，在上述模块中，获取`file`类型的唯一方法是使用`open`函数。这也意味着你不能在未初始化的情况下创建`file`类型的变量。

|  | **RATIONALE** 这种机制类似于Ada语言中的*elaboration*或C++中的*constructors*。它使得程序员能够在值被使用之前提供关于内部状态的强有力保证。这是诸如封装、编程契约或[RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)等编程技术的基本组成部分。 |
| --- | --- |

当值 `V` 的生命周期结束时，语句 `delete V` 会自动评估。声明的实体按其声明顺序的相反顺序销毁。一个 `delete X:T` 的定义被称为类型 `T` 的 *析构函数*。它通常有一个[inout](#inout)参数用于销毁值，以便能够修改其参数，即析构函数通常具有如 `delete X:inout T` 的签名。

对于 [creation](#field-creation) 的对称性，在定义体的退出时，`delete V` 会自动调用 `delete V.X` 来删除 `V` 中的任何字段 `X`。

例如，考虑下面的定义：

```
to delete Z:inout complex is
    print "Deleting complex ", Z
```

该定义实际上等价于以下定义：

```
to delete Z:inout complex is
    print "Deleting complex ", Z
    (delete Z.Im)
    (delete Z.Re)
```

对于该语句，有一个内置的默认定义，其没有效果并匹配任何值，并且仅删除字段：

```
to delete Anything is nil
```

可能有多个与给定表达式匹配的析构函数。当发生这种情况时，将遵循普通的查找规则。这意味着，与像 C++ 这样的语言不同，程序员可以有意地覆盖对象的销毁，并控制销毁过程。更重要的是，这意味着销毁过程遵循全局类型语义。

例如，考虑上面 `MY_FILE` 模块中定义的 `file` 类型的删除。由于负值的特殊情况，实现可能如下所示：

```
to delete F:inout file when F.fd < 0  is ... // Invalid flie
to delete F:inout file                is ... // Valid file
```

然而，这也意味着程序员可以创建一个 `valid_file` 类型，对应于 `F.fd<0` 为假的情况。如果有一个要 `delete` 的 `valid_file` 值，正常的类型系统和查找规则确保将选择第二种情况。

考虑另一个有趣的例子，您有以下声明：

```
type positive is matching (N when N > 0)
integers is string of integer
N:integers > 0 as boolean is
    for I in N loop
        if not (I > 0) then
            return false
    return true

delete N:inout positive is
    print "Deleting positive: ", N

delete N:integer is
    print "Deleting integer: ", N

delete N:integers is
    print "Deleting integers with size: ", size N

example is
   print "Beginning example"
   A:integers := string(1,8,4)
   B:integers := string(-1,0,5)
   print "End of example"
```

在这个例子中，我们创建了一个基于 `string of integers` 的 `integers` 类型，为其实现 `N>0` 操作符，意味着 `string` 中的所有元素都是正数。这反过来意味着某些具有 `integers` 类型的值也具有 `positive` 类型。在 `example` 的体中，`A` 是 `positive`，但 `B` 不是。

如果考虑 [implicit inheritance](#implicit-inheritance) 和隐式插入字段销毁，则上述 `integers` 类型和 `delete` 操作的代码实际上等价于以下内容：

```
type integers is matching(base:string of integer)

delete P:inout positive is
    print "Deleting positive: ", P
    (delete P.N)        // Delete the N bound in 'positive'

delete N:integer is
    print "Deleting integer: ", N
    (delete N.base)     // For 'integer', this is a no-op

delete S:integers is
    print "Deleting integers with size: ", size S
    (delete S.base)     // Delete the underlying 'string of integer'
```

因此，该程序的输出应该类似于：

```
Beginning example
End of example
Deleting integers with size 3 ***(1)**
Deleting positive: 5 ***(2)**
Deleting integer: 5 ***(3)**
Deleting integer: 0 ***(4)**
Deleting integer: -1
Deleting positive: string(1,8,4) ***(5)**
Deleting integers with size 3 ***(6)**
Deleting positive: 4
Deleting integer: 4
Deleting positive: 8
Deleting integer: 8
Deleting positive: 1
Deleting integer: 1******
```

******| ***1*** | 这是使用类型 `integers` 删除本地变量 `B`，知道它因值 `-1` 未通过 `positive` 测试。 |

| ***2*** | 这删除了局部变量`B`中`string of integer`容器中的值，从最后一个开始。容器可以以任何顺序销毁其值，但对于`string`，一个有效的算法可能从容器的末尾开始，以便能够通过改变`string`中的“项目数量”在每个元素被简单移除之前截断。局部的`delete`定义对于在`example`结尾处进行的`string of integer`类型的调用的`delete`实例是可见的。值`5`的第一个匹配定义是为`positive`类型。 |
| --- | --- |
| ***3*** | 这实际上删除了上述代码中称为`P.N`的`integer`值。 |
| ***4*** | 对于`B`中的`string of integer`值中的`0`值，`positive`测试失败，因此第一个可用的析构函数是为`integer`。 |
| ***5*** | 这会删除局部变量`A`。由于`A`是正数，因此调用了正数的析构函数。 |
| ***6*** | 与`B`发生的情况不同，对于`B`，`integers`的析构函数不是直接调用的，而是隐式调用的`P.N`。 |

可以创建局部析构函数定义。当存在这样的局部定义时，它可以覆盖更一般的定义。可以使用link:#enclosing context[`super` lookup]访问一般定义，并且通常应该这样做以保留语言语义。

```
show_destructors is
    delete Something is
        print "Deleted", Something
        super.delete Something
    X is 42
    Y is 57.2
    X + Y
```

这应该输出类似于以下内容：

```
Deleted 42.0
Deleted 57.2
Deleted 42
```

输出的第一个值是由于从`integer`到`real`的必要隐式转换而创建的临时值。注意，根据编译器执行的优化，可能会出现额外的临时值。函数返回的值不应被销毁，因为它会传递给调用者。

任何销毁代码都必须能够以相同的值被多次调用，仅因为你不能阻止程序员编写：

在这种情况下，`Value`将被销毁两次，一次是显式的`delete`，第二次是当`Value`超出范围时。显然，对象可能经历无数次的销毁。

```
for I in 1..LARGE_NUMBER loop
    delete Value
```

还要记住，将一个值作为`out`参数传递会隐式销毁它。这在特定情况下特别适用于赋值的目标。******  ******在XL中，错误由具有`error`类型的值或从错误继承的任何类型表示。错误类型具有一个构造函数，它接受一个简单的错误消息或一个简单的消息和一个负载。

```
type error is one_of
    error Message:text
    error Message:text, Payload
```

消息通常是一个本地化的格式文本，以类似于[`format`函数](#text-format)的方式使用负载中的元素作为编号参数：

```
log X:real as error when X <= 0 is
    error "Logarithm of negative value %1", X
```

可能失败的函数通常会有一个`T 或 error`的返回值。对此，有一个特定的快捷方式，即`mayfail T`：

```
mayfail T:type as type is T or error
```

例如，对于非正值，对数函数会返回错误，因此`log`函数的签名是：

```
log X:real as mayfail real     is ... // May return real or error
```

如果可能的话，错误检测应该推迟到函数的接口。对于 `log` 函数，已知仅在负值或空值时失败，因此更好的接口是：

```
log X:real as real  when X > 0.0    is ... // Always return a real
log X:real as error                 is ... // Always return an error
```

根据上述定义，如果已知 `X > 0.0`，`log X` 的类型将是 `real`，如果已知条件为假，则是 `error`，在更一般的情况下是 `real or error`，即 `mayfail real`。以这种方式编写代码的好处是编译器可以更轻松地确定以下代码是正确的，不需要任何形式的错误处理：

```
if X > 0.0 then
    print format("Log(%1) is %2", X, log X)
```

|  | **RATIONALE** 通过返回错误来处理失败条件，XL 强制程序员简单地处理错误以满足类型系统的要求。它们不像 C 返回值或 C++ 异常可以被简单地忽略。可能从函数返回的错误是其类型的基本部分，错误处理不是可选的。 |
| --- | --- |

一些类型从基本 `error` 类型[派生](#inheritance)，以具备额外的属性：

+   `range_error` 表示给定值超出范围。提供的默认消息会补充有关将值与期望范围进行比较的信息。

    ```
    T:text[I:offset] as character or range_error is
        if I >= length T then
            range_error "Text index %2 is out of bounds for text %2", I, T
        else
            P : memory_address[character] := memory_address(T.first)
            P += I
            *P
    ```

+   `logic_error` 表示程序中的意外条件，并且可以通过像 `assert`、`require` 和 `ensure` 这样的合约检查返回。

    ```
    if X > 0 then
        print "X is positive"
    else if X < 0 then
        print "X is negative"
    else
        logic_error "Some programmer forgot to consider this case"
    ```

+   `storage_error` 每当创建一个[动态值](#dynamic_value)时返回，尤其是每次创建一个 `own T` 对象时，但也是为容器需要额外存储时返回。

    ```
    S : string of integer           // The string requires storage
    loop
        V : own integer := 3        // This allocates an integer, freed each loop
        S &= V                      // Accumulate integers in an unbounded way
    ```

+   `file_error` 报告在打开文件时出错，例如因为文件不存在。

+   `permission_error` 报告资源访问被拒绝的情况，无论是文件还是其他资源。

+   `compile_error` 帮助编译器为导致无效程序的情况提供更好的诊断。如果编译器能够检测到它们将无条件发生，则所有错误都可以在编译时发出，但 `compile_error` 明确表明这意味着意图在编译时检测到错误。变体 `compile_warning` 发出消息但允许编译继续。

    ```
    // Emit a specific compile-time error if assigning text to an integer
    X:integer := Y:text is
        compile_error "Cannot assign text %1 to integer %2", Y, X

    // Emit a specific warning when writing a real into an integer
    X:integer := Y:real is
        compile_warning "Assigning real to integer may lose data"
        T is integer Y
        if real T = Y then
            X := T
        else
            range_error "Assigned real value %1 is out of range for integer", Y
    ```

如果一个值在其生命周期中可以更改，则称其为*可变*。如果一个值不可变，则称其为*常量*。称为*变量*的可变命名实体。称为*命名常量*的不可变命名实体。

`X:T` 类型注释表示 `X` 是类型 `T` 的可变值，除非类型 `T` 明确标记为常量。当 `X` 是名称时，该注释声明 `X` 是一个变量。`X as T` 类型注释表示 `X` 是类型 `T` 的常量值，除非类型 `T` 明确标记为变量。当 `X` 是名称时，这可能声明一个命名常量或者一个没有参数的函数，这取决于主体的形状。

```
StartupMessage : text := "Hello World"  // Variable
Answer as integer is 42                 // Named constant
```

可变值可以使用`:=`操作符进行初始化或修改，这被称为[*赋值*](##assignments-and-moves)。还有一些派生操作符，如`+=`，结合了频繁的算术操作和赋值。

```
X : integer := 42       // Initialize with value 42
X := X or 1             // Binary or, X is now 43
X -= 1                  // Subtract 1 from X, now 42
```

有些实体可能会提供[访问](#access-types)到单独内部值的方式。例如，`text`值在概念上由多个可以单独访问的`character`值组成。这一点是无论`text`如何表示都成立的。此外，`text`值的`slice`本身也是一个`text`值。`text`值的可变性显然会影响对`text`中访问的元素的可变性。

以下示例展示了如何直接改变`text`值（1）、使用计算赋值（2）、通过更改一个`slice`（3）或更改一个单独元素（4）来修改可变值。

```
Greeting : text := "Hello"              // Variable text
Person as text is "John"                // Constant text
Greeting := Greeting & " " & Person     // (1) Greeting now "Hello John"
Greeting &= "!"                         // (2) Greeting now "Hello John!"
Greeting[0..4] := "Good m0rning"        // (3) Greeting now "Good m0rning John!"
Greeting[6] := 'o'                      // (4) Greeting now "Good morning John!"
```

这些操作中的任何一种都不适用于代码中的常量`Person`等文本。例如，`Person[3]:='a'`是无效的，因为`Person`是一个常量值。

|  | 在情况（3）中，通过访问类型修改`text`值可以改变其长度。这是因为`Greeting[0..4]`不是一个独立的值，而是一个访问类型，具体是一个`slice`，它跟踪`text`（这里是`Greeting`）和索引范围（在这种情况下是`0..4`），带有一个`:=`操作符来修改访问的`text`值。 |
| --- | --- |

常量值在其生命周期内不会改变，但在程序的生命周期内可能会改变。更精确地说，常量的生命周期最多与计算其值的生命周期一样长。例如，在下面的代码中，常量`K`在循环的每次迭代中具有不同的值，但常量`L`在所有`I`的迭代中具有相同的值。

```
for J in 1..5 loop
    for I in 1..5 loop
        K is 2*I + 1
        L is 2*J + 1
        print "I=", I, " K=", K, " L=", L
```

|  | **RATIONALE** 常量和无参数函数在语法上没有区别。如果更有效，实现可以自由地将常量实现为函数，或者在适当时使用更智能的策略。 |
| --- | --- |

有些数据类型可以由固定数量的连续内存位置表示。例如，`integer`或`real`就是这种情况：所有`integer`值占用相同数量的字节。这些数据类型被称为*紧凑*。

另一方面，`text`值可以是任意长度，因此可能需要可变数量的字节来表示诸如`"Hi"`和`"There once was a time where text was represented in languages such as Pascal by fixed-size character array with a byte representing the length. This meant that you could not process text that was longer than, say, 255 characters. More modern languages have lifted this restriction."`等值。这些值被称为*散布的*。

散列类型总是通过*解释*紧凑类型来构建。例如，文本的表示可以由两个值组成，即第一个字符的内存地址和文本的大小。当然，这不是唯一可能的表示，但任何表示都要求解释固定大小的内存位置并为其提供逻辑结构。

尽管这不总是情况，紧凑类型的赋值通常进行[复制](#copy)，而散列类型的赋值通常进行[移动](#move)。

计算机提供多种*资源*：内存、文件、锁、网络连接、设备、传感器、执行器等。此类资源的常见问题是控制它们的*所有权*。换句话说，在任何给定时间，谁对给定资源负责。

在 XL 中，就像在 Rust 或 C++ 中一样，所有权主要由类型系统确定，并且在很大程度上依赖于它提供的保证，特别是关于[创建](#creation)和[销毁](#destruction)的保证。在 C++ 中，该机制称为[RAII](https://en.wikipedia.org/wiki/RAII)，即资源获取即初始化。其核心思想是，在值的生命周期内资源的所有权是不变的。换句话说，在构造期间，值获取资源的所有权，并在销毁期间释放此所有权。这在模块`MY_FILE`的`file`类型中有所体现[之前给出](#my_file)。

设计用于拥有相关值的类型称为*所有者类型*。通常每个受控资源在任何给定时间最多有一个活跃的所有者，该所有者在构造时获取资源，并在销毁时释放它。可能可以通过`delete Value`提前释放所拥有的资源。

[标准库](#standard-library)提供了多种旨在拥有常见类资源的类型，包括：

+   拥有（`own`）值拥有在动态存储中分配的单个项目。请注意，值`nil`不是有效的`own`值（除了`own nil`）。如果需要`nil`作为值，则必须使用`own T or nil`。

+   数组（`array`）、缓冲区（`buffer`）和字符串（`string`）都拥有相同类型的连续项目序列。

    +   数组（`array`）在其生命周期内具有固定大小，并直接分配项目，例如在执行栈上。

    +   缓冲区（`buffer`）在其生命周期内具有固定大小，并且动态分配项目，通常来自堆。

    +   字符串（`string`）在其生命周期内大小可变，因此可能会在特定操作的结果下在内存中移动项目。

+   文本（`text`）拥有可变数量的`character`项目，并继承自`string of character`类型。

+   文件（`file`）拥有一个打开的文件。

+   互斥体（`mutex`）在其活动期间由单个线程拥有执行。

+   定时器（`timer`）拥有可用于测量时间和调度执行的资源。

+   线程（`thread`）拥有执行线程和相关的调用堆栈。

+   一个`task`拥有一个操作，可以分派给可用线程执行之一。

+   一个`process`拥有一个操作系统进程，包括其线程和地址空间。

+   一个`context`捕获一个执行上下文。

并非所有类型都意图成为所有者类型。许多类型将所有权委托给另一种类型。这些类型称为*访问类型*。当访问类型被销毁时，它所访问的资源*不会*被处理，因为访问类型不拥有该值。访问类型的值仅仅提供对关联所有者类型的特定值的*访问*。

例如，如果`T`是一个`text`值，如果`A`和`B`是`integer`值，那么`T[A..B]`是一种特定的访问值，称为*slice*，表示基于从`0`开始的位置`A`和`B`之间的文本片段。根据构造，`T[A..B]`只能访问`T`，而不能访问任何其他`text`值。类似地，可以轻松实现对`A`和`B`的边界检查，以确保没有操作会访问`T`之外的任何`character`值。因此，这种访问值是完全安全的。

访问类型泛化了在其他语言中找到的*指针*或*引用*，因为它们可以描述一类更广泛的访问模式。指针只能访问单个元素，而访问类型没有这样的限制，正如`T[A..B]`的示例所示。访问类型还可以强制执行比简单指针更严格的所有权规则。

|  | C语言通过滥用所谓的“指针算术”来解决指针仅访问单个元素的限制，特别是用来实现数组。在C中，`A[I]`仅仅是`*(A+I)`的一种快捷方式。这意味着在C中，`3[buffer]`是访问`buffer`第三个元素的有效方式，并且存在`ptr[-1]`这样的情况，以访问`ptr`之前的元素。不幸的是，这种技巧在机器只有32K内存时可能很聪明，现在却成为一类编程错误的根源，称为*缓冲区溢出*，这在很大程度上导致C语言完全没有内存安全性的声誉。 |
| --- | --- |

[标准库](#standard-library)提供了许多用于访问常见所有者类型的类型，包括：

+   一个`ref`是对一个活动的`own`值的引用。

+   一个`slice`可以用来访问连续序列中的一系列项，包括`array`、`buffer`或`string`（因此`text`被视为`string of character`）。

+   一个`reader`或`writer`可以用来访问一个`file`，无论是用来读取还是写入。

+   一个`lock`使用一个`mutex`来防止多个线程执行给定的代码片段。

+   几种类型，如`timing`、`dispatch`、`timeout`或`rendezvous`，将结合`timer`、`thread`、`task`和`context`值。

+   `in`、`out` 和 `inout` 类型表达式有时可以等同于访问类型，如果这是传递参数的最有效方式。然而，这对程序员来说大多数情况下是看不见的。

+   `XL.SYSTEM.MEMORY.address` 引用内存中特定的地址，它是 XL 中最接近原始 C 指针的表达方式。它特意冗长且不方便使用，以避免在非绝对必要时使用它。

如果一个类型 *继承* 另一个类型（称为其 *基础类型*），则它可以使用所有其操作。然后该类型被称为从基础类型 *派生* 出来。在 XL 中，这通过在派生类型和基础类型之间提供 *隐式转换* 来实现：

```
Derived:derived as base is ...
```

根据这种方法，一个类型可以从任意数量的其他类型派生，有时这种特性被称为 *多重继承*。此外，基础类型和派生类型之间也没有必要共享任何特定的数据表示，尽管这在实践中经常发生（参见[#data-inheritance](#data-inheritance)）。例如，虽然 `i16` 和 `i32` 的机器表示不同，但在 XL 中可以说 `i16` 派生自 `i32`。

有时需要表示派生自特定类型的类型。例如，如果要为 `complex` 创建一个受限泛型类型，您可能希望它仅接受 `number` 作为其类型参数。以下代码是不正确的做法，因为它创建了一个仅接受 `number` *值* 作为参数的类型，即它将接受 `complex[3.5]` 但不接受 `complex[real]`：

```
type complex is matching complex[real:number] // WRONG
```

要表示派生自基础类型的类型，可以使用 `derived like base` 符号。实现上述限制的正确方法如下，指示参数是一种类型而不是值，并且该类型必须派生自 `number`：

```
type complex is matching complex[real:type like number]
```

`like` 的优先级低于 `:`，因此上述实际解析为 `(real:type) like number`。`like` 操作符可以应用于声明的任何位置，特别是在作用域中。`like` 的另一种拼写是 `inherits`，在全局范围内通常更易读。例如，您可以指示 `complex` 类型本身继承自 `arithmetic`（提供算术操作）以及从 `compact`（使用连续内存范围）继承，如下所示：

```
type complex inherits arithmetic
type complex inherits compact
```************
