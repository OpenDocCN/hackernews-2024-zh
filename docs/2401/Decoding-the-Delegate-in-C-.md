<!--yml

category: 未分类

date: 2024-05-27 14:44:59

-->

# 在 C# 中解码委托

> 来源：[`blog.upperdine.dev/decoding-the-delegate-in-csharp`](https://blog.upperdine.dev/decoding-the-delegate-in-csharp)

委托是 C# 的一种语言特性，已经存在了几年，通过利用它们，我们可以使我们的代码库更具组合性和适应性。您可能已经在您工作的代码库中遇到过这些，也可能没有，但是它们在通用代码中更为常见，例如库或框架代码。

那么，当我谈到委托时，我到底是什么意思呢？让我开始解释一下，先从谈论函数式编程开始。

## 函数式编程

函数式编程是一种 **编程范式**，或者说是一种强调将函数作为应用程序的主要构建块的编程理念。您可能熟悉的另一种范式是 **面向对象编程**，它反之强调使用对象作为应用程序的主要构建块。

例如，在使用 OOP 设计系统时，我们通常会考虑各种对象之间的相互作用。使用功能性方法时，我们会考虑函数。

在函数式编程中，有一个称为 **一级函数** 的概念。这意味着函数和任何其他类型一样，并且因此可以作为值传递。

函数可以作为参数传递给其他函数，甚至可以从其他函数返回。接受函数作为参数或返回函数的函数称为 **高阶函数**。

由于 C# 是一种强类型语言，我们需要一个表示一级函数的类型的名称-在这种情况下，该名称是 **Delegate**。

如果您对这个概念感到陌生，那么当您专业地使用 C# 时，您很可能在以前甚至不知道的情况下使用过这些。例如，当您使用 LINQ 查询集合时，通常会提供一个 lambda 表达式进行过滤或转换：

```
IEnumerable<int> FindEvenNumbers(IEnumerable<int> numbers)
{
    return numbers.Where(n => n % 2 == 0);
} 
```

请记住，lambda 表达式只是没有名称的函数，因此 LINQ 中的 `Where` 方法是高阶函数的一个示例。

那么，如果我们想要编写自己的高阶函数怎么办呢？

## Func

**Func** 类型是您将遇到的最常见的委托类型，用于返回值的函数。例如，如果我们想要接受一个函数作为参数，该函数接受一个类型为 *string* 的参数并返回一个 *integer*（例如 **int.Parse**），我们将其声明为如下所示：

```
int StringToInt (string toParse, Func<string, int> parser)
{
    return parser(toParse);
}

StringToInt("42", int.Parse); 
```

**Func** 类型不需要任何输入参数，但是*确实*需要一个返回类型，因此至少需要提供一个类型参数。

## Action

在需要一个不返回值的函数的情况下，您可以使用 **Action** 类型。

举个例子，想象一下我们正在编写一个方法，并且我们希望能够将 **Console.WriteLine** 作为参数传递进去。我们可以使用 Action 类型如下所示：

```
void PrintMyMessage(string message, Action<string> printer)
{
    printer(message);
}

PrintMyMessage("Hello, World!", Console.WriteLine); 
```

这种实现的美妙之处在于，我们可以将**Console.WriteLine**替换为任何接受字符串作为参数的 void 方法。在生产中，我们可能希望将其替换为日志记录器调用，并且因为我们**编程到接口而不是具体实现**- 我们不需要修改方法签名。

那么什么样的函数既不返回值也不接受任何参数呢？这些通常被称为**Thunks**，可以通过从**Action**中省略类型参数来简单地建模：

```
Action printHelloWorld = () => Console.WriteLine("Hello, World!"); 
```

.NET 还提供了一些其他类型的委托，例如**EventHandler**、**Predicate**、**MethodInvoker**和**ThreadStart**。这些要么是用于特定用例，要么主要用于内部使用，因此我不会在本博客文章中花时间详细介绍它们。

## 返回函数

正如我之前提到的，高阶函数不仅可以将函数作为参数传递，还可以返回一个函数。

既然我们知道了委托的主要类型，那么将函数返回就像将函数的返回类型更改为委托类型一样简单：

```
Func<int, int> MultiplyBy(int multiplier)
{
    return number => number * multiplier;
}

Func<int, int> MultiplyByThree = MultiplyBy(3);

var result = MultiplyByThree(10); 
```

这是一个相当简单的例子，但是我们的**MultiplyBy**方法允许我们创建一个新的函数，该函数接受一个整数并将其乘以指定的值。

## 创建自己的委托类型

如果你想要在代码库中创建通用的 Delegate 类型，而不必一遍又一遍地重复类型签名，C# 中有一个内置的**delegate**关键字。

举个例子，假设我们想要在代码库中为对象映射代码添加一些统一性。我们需要我们的产品类型能够通过提供的函数映射到任何类型，但我们希望一个集中定义的映射函数应该是什么样子：

```
public record Product(Guid SKU, decimal MSRP)
{
    public delegate T ProductMapper<out T>(Product product);

    public T MapTo<T>(ProductMapper<T> mapper)
    {
        return mapper(this);
    }
} 
```

你*可以*在这里使用**Func**- 它实际上在内部使用委托关键字，但我觉得将我们的委托类命名为与域相关的内容更好。

## 结论

通过利用委托，我们能够将函数视为任何其他类型，并编写具有适应性的代码。

面向对象编程利用重写和重载来实现多态性，但通过将对象或函数组合起来，通过注入函数来获得我们想要的行为，也可以实现类似的效果。

委托的一个重要事项是，虽然它们非常强大，但过度使用它们会给代码库增加不必要的复杂性，因此要节制使用它们。
