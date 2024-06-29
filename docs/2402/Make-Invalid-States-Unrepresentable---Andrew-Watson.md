<!--yml

类别：未分类

日期：2024-05-27 14:28:28

-->

# 使无效状态不可表示 - Andrew Watson

> 来源：[https://www.awwsmm.com/blog/make-invalid-states-unrepresentable](https://www.awwsmm.com/blog/make-invalid-states-unrepresentable)

[[编辑历史记录]](https://github.com/awwsmm/awwsmm.com/commits/master/blog/make-invalid-states-unrepresentable.md)

假设您在程序中有一个`Person`类，并且一个`Person`有一个`age`。`age`应该是什么类型？

## `age`作为`String`

```
case class Person(age: String) 
```

"当然，它不应该是一个`String`"，你可能会想。但为什么呢？原因是我们可以得到类似以下的代码

```
val person = Person("Jeff") 
```

如果我们曾经想要*使用*`age: String`做任何事情，我们需要在所有地方验证它

```
def isOldEnoughToSmoke(person: Person): Boolean = {
  Try(person.age.toInt) match {
    case Failure(_) => throw new Exception(s"cannot parse age '${person.age}' as numeric")
    case Success(value) => value >= 18
  }
}

def isOldEnoughToDrink(person: Person): Boolean = {
  Try(person.age.toInt) match {
    case Failure(_) => throw new Exception(s"cannot parse age '${person.age}' as numeric")
    case Success(value) => value >= 21
  }
} 
```

这对编写代码的程序员来说很麻烦，同时也让阅读代码的任何程序员感到困难。

我们可以将此验证移到一个单独的方法中

```
def parseAge(age: String): Int = {
  Try(age.toInt) match {
    case Failure(_) => throw new Exception(s"cannot parse age '$age' as numeric")
    case Success(value) => value
  }
}

def isOldEnoughToSmoke(person: Person): Boolean =
  parseAge(person.age) >= 18

def isOldEnoughToDrink(person: Person): Boolean =
  parseAge(person.age) >= 21 
```

...但这仍然不理想。代码变得更清晰了，但我们仍然需要每次想要执行任何数字操作（比较、算术等）时将`String`解析为`Int`。这通常被称为["stringly-typed" data](https://en.wiktionary.org/wiki/stringly-typed)。

这也可以通过抛出`Exception`将程序移动到非法状态。如果我们最终会失败，我们应该[尽快失败](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)。我们可以做得更好。

## `age`作为`Int`

你的第一个直觉可能是将`age`作为`Int`，而不是`String`

```
case class Person(age: Int) 
```

如果是这样，你有很好的直觉。`age: Int`更容易操作

```
def isOldEnoughToSmoke(person: Person): Boolean =
  person.age >= 18

def isOldEnoughToDrink(person: Person): Boolean =
  person.age >= 21 
```

这

+   更容易编写

+   更容易阅读

+   快速失败

您*无法*使用`String`的`age`构造`Person`类的实例。这是一个无效状态。我们使用类型系统[使其不可表示](https://www.cs.rice.edu/~javaplt/411/23-spring/NewReadings/functional_programming_on_Wall_Street.pdf)。编译器不会允许这个程序编译。

问题解决了，对吧？

```
val person = Person(-1) 
```

这显然也是一个无效状态。一个人不能有负年龄。

```
val person = Person(90210) 
```

这也是无效的 -- 看起来有人意外地输入了他们的[ZIP code](https://en.wikipedia.org/wiki/Beverly_Hills,_California)而不是他们的年龄。

那么我们如何进一步约束这种类型？如何使更多的无效状态不可表示？

## `age`作为带约束的`Int`

### 在运行时

我们可以在任何静态类型语言中强制执行*运行时约束*

```
case class Person(age: Int) {
  assert(age >= 0 && age < 150)
} 
```

在Scala中，如果断言失败，`assert`将抛出`java.lang.AssertionError`。

现在我们可以确保`Person`的任何`age`都在`[0, 150)`的范围内。两者

```
val person = Person(-1) 
```

和

```
val person = Person(90210) 
```

现在将会失败。但它们会在运行时失败，停止程序的执行。

这与我们在"`age`作为`String`"中看到的类似。这仍然不理想。有更好的方式吗？

### 在编译时

许多语言也允许*compile-time*约束。通常通过宏来实现，它们在编译阶段检查源代码。这些通常被称为[refined types](https://en.wikipedia.org/wiki/Refinement_type)。

Scala在多个库中对[refined types](https://blog.rockthejvm.com/refined-types/)有[相当好的](https://github.com/Iltotore/iron)支持。使用[`refined` library](https://github.com/fthomas/refined)可能会看起来像这样

```
case class Person(age: Int Refined GreaterEqual[0] And Less[150]) 
```

此方法的一个限制是，要在编译时约束的字段必须是[*literal*, hard-coded](https://en.wikipedia.org/wiki/Literal_(computer_programming))值。例如，不能对用户提供的值强制执行编译时约束。在那时，程序已经编译完成。在这种情况下，我们可以始终退回到运行时约束，这通常是这些库所做的。

目前，我们将继续只使用运行时约束，因为通常这是我们能做到的最好的。

## `age` 作为具有约束的 `Age`

从图表中最简单到最复杂的实现，我们从左向右移动

```
String => Int => Int with constraints 
```

这种复杂性的增加直接与我们对这些数据建模的*准确性*相关

> "我们要解决的问题具有固有的复杂性，适当地对其进行建模需要一些努力。" [[source]](https://fasterthanli.me/articles/i-want-off-mr-golangs-wild-ride)

上述从左到右的移动应该由系统的要求驱动。例如，不要实现编译时的细化，除非您有很多硬编码的值需要在编译时验证：否则[你不需要它](https://martinfowler.com/bliki/Yagni.html)。每行代码都有实现和维护的成本。避免过早的规范化与避免[过早的泛化](https://www.codewithjason.com/premature-generalization)一样重要，尽管从更具体的类型向不太具体的类型移动**总是更容易的*。因此，请优先考虑具体性而不是泛化。

每个数据都有一个*context*。在宇宙中不存在“纯”`Int`值。年龄可以被建模为`Int`，但与重量不同，重量也可以被建模为Int。我们附加到这些原始值上的标签是*context*

```
case class Person(age: Int, weight: Int) {
  assert(age >= 0 && age < 150)
  assert(weight >= 0 && weight < 500)
} 
```

在这里我们还有一个问题要解决。假设我是81公斤，33岁

```
val me = Person(81, 33) 
```

虽然编译通过了，但...不应该。我颠倒了我的体重和年龄！

避免这种混淆的一种简单方法是定义一些更多的类型。在这种情况下，[*newtypes*](https://doc.rust-lang.org/rust-by-example/generics/new_types.html)

```
case class Age(years: Int) {
  assert(years >= 0 && years < 150)
}

case class Weight(kgs: Int) {
  assert(kgs >= 0 && kgs < 500)
}

case class Person(age: Age, weight: Weight) 
```

*newtype* 模式的名称[来自 Haskell](https://wiki.haskell.org/Newtype)。这是一种简单的方法，确保我们不会意外交换具有相同基础类型的值。例如，以下代码不会编译：

```
val age = Age(33)
val weight = Weight(81)

val me = Person(weight, age) 
```

我们也可以使用[*标记类型*](https://medium.com/iterators/to-tag-a-type-88dc344bb66c)。在Scala中，这个最简单的示例看起来像这样

```
trait AgeTag
type Age = Int with AgeTag

object Age {
  def apply(years: Int): Age = {
    assert(years >= 0 && years < 150)
    years.asInstanceOf[Age]
  }
}

trait WeightTag
type Weight = Int with WeightTag

object Weight {
  def apply(kgs: Int): Weight = {
    assert(kgs >= 0 && kgs < 500)
    kgs.asInstanceOf[Weight]
  }
}

case class Person(age: Age, weight: Weight)

val p0 = Person(42, 42) 
val p1 = Person(Age(42), 42) 
val p2 = Person(Age(42), Weight(42)) 
val p3 = Person(Weight(42), Weight(42)) 
```

这利用了函数应用`f()`在Scala中是`apply()`方法的语法糖的事实。因此，`f()`等同于`f.apply()`。

这种方法允许我们建模“年龄”/“体重”是一个`Int`的想法，但`Int`不是“年龄”/“体重”。这意味着我们可以将“年龄”/“体重”视为`Int`，并添加、减去或进行其他类似于`Int`的操作。

将这两种方法混合在一个示例中，你可以看到新类型和标记类型之间的区别。你必须从新类型中提取“原始值”。对于标记类型，你不需要这样做。

```
 trait AgeTag
type Age = Int with AgeTag

object Age {
  def apply(years: Int): Age = {
    assert(years >= 0 && years < 150)
    years.asInstanceOf[Age]
  }
}

case class Weight(kgs: Int) {
  assert(kgs >= 0 && kgs < 500)
}

assert(40 == Age(10) + Age(30))

Weight(10) + Weight(30) 

Weight(10).kgs + Weight(30).kgs 
```

在某些语言中，新类型的“解包”可以自动完成。这可以使新类型像标记类型一样符合人体工学。例如，在Scala中，可以通过*隐式转换*来完成这个操作。

```
implicit def weightAsInt(weight: Weight): Int = weight.kgs

Weight(10) + Weight(30) 
```

## 进一步的改进。

上述讨论的重点是，尽可能*使无效状态无法表示*。

+   "Jeff" 是一个无效的年龄。年龄不是字符串，而是一个数字。

+   -1 是一个无效的年龄。年龄不能为负数，应该是 0 或正数，并且可能小于约 150。

+   我的年龄不是 88\. 年龄应该很容易与其他整数值区分开，比如体重。

上述所有讨论都是关于“年龄”概念逐步实现的这些改进。

如果有必要，我们可以进一步改进这些改进。

例如，假设我们想在`Person`的生日时发送一封“生日快乐！”的电子邮件。现在我们需要的不是`Age`，而是出生日期。

```
case class Date(year: Year, month: Month, day: Day)

case class Year(value: Int, currentYear: Int) {
  assert(value >= 1900 && value <= currentYear)
}

case class Month(value: Int) {
  assert(value >= 1 && value <= 12)
}

case class Day(value: Int) {
  assert(value >= 1 && value <= 31)
}

case class Person(dateOfBirth: Date, weight: Weight) {
  def age(currentDate: Date): Age = {
    ??? 
  }
} 
```

`dateOfBirth`提供的信息量严格大于`Age`提供的信息量。我们可以根据某人的出生日期计算他们的年龄，但反过来却不行。

上述实现还有很多不足之处 -- 存在许多无效状态。更好的实现方式是让`Month`成为一个枚举，并且`Day`的有效性取决于`Month`（例如，2月从不有 30 天）

```
case class Year(value: Int, currentYear: Int) {
  assert(value >= 1900 && value <= currentYear)
}

sealed trait Month

case object January extends Month
case object February extends Month
case object March extends Month
case object April extends Month
case object May extends Month
case object June extends Month
case object July extends Month
case object August extends Month
case object September extends Month
case object October extends Month
case object November extends Month
case object December extends Month

case class Day(value: Int, month: Month) {
  month match {
    case February => assert(value >= 1 && value <= 28)
    case April | June | September | November => assert(value >= 1 && value <= 30)
    case _ => assert(value >= 1 && value <= 31)
  }
}

case class Date(year: Year, month: Month, day: Day) 
```

在可能的情况下，始终优先选择低基数类型而不是[高基数类型](https://www.seas.upenn.edu/~sweirich/types/archive/1999-2003/msg01233.html)。这限制了可能的无效状态数量。在大多数语言中，这里应该使用`enum`（在Scala 2中，可以使用`sealed trait`来模拟`enum`，如上所示）。但仍然有一些隐藏的无效状态。你能找到它们吗？

在某些情况下，使用正则表达式验证的字符串类型数据可以完全被`enum`替代。你能建模[加拿大邮政编码](https://www150.statcan.gc.ca/n1/pub/92-153-g/2011002/tech-eng.htm)，以确保无法构造出无效的邮政编码吗？

利用以上知识，继续前进，并且*使无效状态无法表示*。
