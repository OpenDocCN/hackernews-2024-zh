<!--yml

category: 未分类

date: 2024-05-27 14:43:34

-->

# Swift.org - 在 Swift 6.0 中迭代参数包

> 来源：[https://www.swift.org/blog/pack-iteration/](https://www.swift.org/blog/pack-iteration/)

# 在 Swift 6.0 中迭代参数包

2024年3月7日

参数包，引入于 Swift 5.9，使得能够编写抽象化处理参数数量的泛型成为可能。这消除了需要为一个参数、两个参数、三个参数等的相同泛型函数进行重载的需求。通过 Swift 6.0，参数包迭代使得与参数包一起工作变得比以往任何时候都更加容易。本文将展示如何充分利用参数包迭代。

首先，让我们回顾一下参数包。考虑以下代码：

```
let areEqual = (1, true, "hello") == (1, false, "hello")
print(areEqual)
// false 
```

上述代码仅仅是比较两个元组。但是，如果元组包含7个元素，此代码将不起作用！

Swift 标准库长期以来仅为最多包含6个元素的元组提供了比较运算符：

```
func == (lhs: (), rhs: ()) -> Bool

func == <A, B>(lhs: (A, B), rhs: (A, B)) -> Bool where A: Equatable, B: Equatable

func == <A, B, C>(lhs: (A, B, C), rhs: (A, B, C)) -> Bool where A: Equatable, B: Equatable, C: Equatable

// and so on, up to 6-element tuples 
```

在上述通用函数的每个输入元组的每个元素中，都必须在函数的通用参数列表中声明其类型。因此，每当我们希望支持更大的元组大小时，都需要向通用参数列表中添加一个新元素。由于这个原因，6元素元组的人为限制就被强加了上去。

参数包增加了通过变量数量的类型参数来抽象化函数的能力。这意味着我们可以使用类似这样的`==`操作符来解除6个元素的限制：

```
func == <each Element: Equatable>(lhs: (repeat each Element), rhs: (repeat each Element)) -> Bool 
```

让我们分解一下上面签名中看到的类型：

+   注意列表中的每个`Element`。`each`关键字表示`Element`是一个*类型参数包*，这意味着它可以接受任意数量的通用参数。就像非包（*scalar*）通用参数一样，我们可以在类型参数包上声明符合要求。在这种情况下，我们要求每个`Element`类型符合`Equatable`协议。

+   此函数接收两个元组 `lhs` 和 `rhs` 作为参数。在这两种情况下，元组的元素类型都是 `repeat each Element`。这称为*包扩展类型*，它由`repeat`关键字后跟一个*重复模式*组成，该重复模式必须包含一个包引用。在我们的情况下，重复模式是 `each Element`。

+   在调用位置，用户为每个将替换到相应类型参数包的元组提供了*值参数包*。在运行时，重复模式将对替换包中的每个元素重复进行。

使用参数包实现的元组相等操作符，让我们再次查看调用位置以更好地理解这些概念。

```
let areEqual = (1, true, "hello") == (1, false, "hello")
print(areEqual)
// false 
```

对`==`的调用用类型包`{Int, Bool, String}`替换了类型包`Element`。注意`lhs`和`rhs`都具有相同的类型。最后，函数`==`用值包`{1, true, "hello"}`替换了`lhs`元组的值包，用值包`{1, false, "hello"}`替换了`rhs`元组的值包。

具有新元组比较运算符签名的示例看起来很棒，但实际上我们如何在函数体内使用`lhs`和`rhs`元组的值？请随时花点时间考虑这个问题。

结果表明，在Swift 6.0之前，实现此功能没有简洁的方法。一种解决方案涉及创建一个本地函数，用于比较来自两个元组的一对元素，然后使用包展开为每一对元素调用该函数，如下所示：

```
struct NotEqual: Error {}

func == <each Element: Equatable>(lhs: (repeat each Element), rhs: (repeat each Element)) -> Bool {
  // Local throwing function for operating over each element of a pack expansion.
  func isEqual<T: Equatable>(_ left: T, _ right: T) throws {
    if left == right {
      return
    }

    throw NotEqual()
  }

  // Do-catch statement for returning false as soon as two tuple elements are not equal.
  do {
    repeat try isEqual(each lhs, each rhs)
  } catch {
    return false
  }

  return true
} 
```

上述代码看起来不太好，对吧？为了简单地检查每对元素的条件，我们需要声明一个名为`isEqual`的局部函数，只是比较给定的元素。然而，这并不足以使函数提前返回，因为在扩展它们时，本地的`isEqual`函数仍将被调用于参数包`lhs`和`rhs`中的每一对元素。因此，`isEqual`必须标记为`throws`，并在找到不匹配的元素对时抛出错误。然后，我们在`catch`块中捕获错误以返回`false`。

Swift 6.0通过引入熟悉的`for`-`in`循环语法大大简化了此任务中的包迭代。

更具体地说，通过包迭代，`==`元组比较运算符的主体简化为一个简单的`for`-`in repeat`循环：

```
func == <each Element: Equatable>(lhs: (repeat each Element), rhs: (repeat each Element)) -> Bool {

  for (left, right) in repeat (each lhs, each rhs) {
    guard left == right else { return false }
  }
  return true
} 
```

在上述代码中，我们能够利用`for`-`in`循环功能来成对迭代元组。

注意，在迭代包时，我们使用新的`for`-`in repeat`语法，后跟要迭代的值参数包。在每次迭代时，循环将值参数包的每个元素绑定到一个局部变量。这意味着在这种情况下，`lhs`的第i个元素将在第i次迭代时绑定到局部变量`left`上。在循环体中，您可以像平常一样使用局部变量。在我们的情况下，我们比较每一对元素，并在找到`left != right`的一对时返回`false`，使用熟悉的`guard`语句。当然，我们不再需要像以前那样抛出任何错误了！

现在让我们通过一些示例更深入地探讨如何在Swift代码中利用包迭代。

首先考虑一种情况，您需要编写一个函数来检查给定值参数包中所有数组是否为空：

```
func allEmpty<each T>(_ array: repeat [each T]) -> Bool {
  for a in repeat each array {
    guard a.isEmpty else { return false }
  }

  return true
} 
```

上述函数是针对类型参数包`each T`泛型化的，并接受一个名为`array`的值参数包，其类型使用`repeat [each T]`包展开声明，其中`[each T]`是重复模式。在调用点，它为替换的每个元素重复，导致值扩展为数组字面量列表。

在每次`for`-`in repeat`循环迭代中，将`array`值参数包的一个元素绑定到本地变量`a`。请注意，使用包迭代时，值包的元素按需评估，这意味着我们可以在不检查值包的所有数组的情况下提前从函数返回。在这种情况下，我们利用了`guard`语句。

这里是您可能如何使用`allEmpty`函数的方法：

```
print(allEmpty(["One", "Two"], [1], [true, false], []))
// False 
```

现在，让我们看一个通过包迭代大大简化的参数包的高级使用示例。首先，让我们声明以下协议：

```
protocol ValueProducer {
  associatedtype Value: Codable
  func evaluate() -> Value
} 
```

上述协议`ValueProducer`要求`evaluate()`方法，其返回类型是关联类型`Value`，该类型符合`Codable`协议。

假设您获得了一个类型为`Result<ValueProducer, Error>`的值参数包，并且您只需迭代`success`元素并调用其值的`evaluate()`方法。此外，假设您需要将每次调用的结果保存到一个数组中。包迭代使得这个任务变得非常简单！

```
func evaluateAll<each V: ValueProducer, each E: Error>(result: repeat Result<each V, each E>) -> [any Codable] {
  var evaluated: [any Codable] = []
  for case .success(let valueProducer) in repeat each result {
    evaluated.append(valueProducer.evaluate())
  }

  return evaluated
} 
```

让我们首先注意`evaluateAll`函数的签名。在泛型参数列表中，它声明了两个类型参数包：`each V: ValueProducer`和`each E: Error`。包`each V`的每个元素必须符合上面声明的`ValueProducer`协议，而包`each E`的每个元素必须符合`Error`协议。函数接受一个名为`result`的参数，其类型是`repeat Result<each V, each E>`的扩展类型。这意味着`Result<each V, each E>`模式将在运行时为`each V`和`each E`的每个元素重复。

要实现函数体，我们首先初始化了`evaluated`数组。接下来，请注意我们如何使用`for case`模式仅对`Result`枚举的`success`情况执行循环体。我们可以获取`valueProducer`变量，它将包含`ValueProducer`类型的值。现在，我们可以将调用`evaluate()`方法的结果追加到我们的`evaluated`数组中，最后返回它。

下面是您可能使用此函数的方式：

```
struct IntProducer: ValueProducer {

  let contained: Int

  init(_ contained: Int) {
    self.contained = contained
  }

  func evaluate() -> Int {
    return self.contained
  }
}

struct BoolProducer: ValueProducer {

  let contained: Bool

  init(_ contained: Bool) {
    self.contained = contained
  }

  func evaluate() -> Bool {
    return self.contained
  }
}

struct SomeError: Error {}

print(evaluateAll(result:
                    Result<SomeValueProducer, SomeError>.success(SomeValueProducer(5)),
                    Result<SomeValueProducer, SomeError>.failure(SomeError()),
                    Result<BoolProducer, SomeError>.success(BoolProducer(true))))

// [5, true] 
```

我们很高兴将包迭代引入Swift 6.0！正如本文所示，包迭代使得与值参数包交互变得显著简单，使得这样一个高级特性更易于融入您的Swift代码。
