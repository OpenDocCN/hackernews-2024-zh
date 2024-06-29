<!--yml

分类：未分类

日期：2024-05-27 13:36:42

-->

# 如果在Java中null是一个对象会怎样？ | Medium

> 来源：[https://donraab.medium.com/what-if-null-was-an-object-in-java-3f1974954be2](https://donraab.medium.com/what-if-null-was-an-object-in-java-3f1974954be2)

# 如果在Java中null是一个对象？

请做好准备，准备尝试红色药丸。

照片由 [Brett Jordan](https://unsplash.com/@brett_jordan?utm_source=medium&utm_medium=referral) 拍摄于 [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

# 面向对象的纯度

Smalltalk通常被称为一种纯粹的面向对象编程语言。Smalltalk由Alan Kay创建，他也是“面向对象”这个术语的创造者。Smalltalk的纯面向对象的部分是指在Smalltalk中，“一切皆为对象”。因为一切都是对象，你通过向对象发送消息来完成编程任务。

Java也是面向对象的，但不是纯粹的面向对象编程语言。Java中有一些不是对象的东西（例如基本类型），需要特殊机制来处理它们。Java还有一个专门处理`null`的特殊情况。

在这篇博客中，我将演示和解释Java和Smalltalk中处理*缺失*值的方式，Java中称为`null`，Smalltalk中称为`nil`。本博客不会提供Java或Smalltalk语法的全面解释。如果你想了解更多，可以在互联网上找到相关资源。本博客仅涵盖解释我分享的代码示例所需的内容。我将尽力详细解释示例，以便语法不成为理解的障碍。本博客旨在让您思考更广阔的可能性，而不是与语言语法的琐碎斗争。

在接下来的几节中，我将比较使用Java和Smalltalk解决同样简单问题的方法。我将使用Java 21和IntelliJ IDEA CE 2023.3.2进行Java代码示例。我将使用 [Pharo Smalltalk](https://pharo.org/) 11.0 进行Smalltalk代码示例。

# Null vs. Nil

## Java中的字面量null

在Java中，有一个叫做`null`的字面量。你可以将`null`赋给任何具有`Object`类型的变量，但是变量指向的引用不是`Object`的实例。我喜欢将`null`比作电影《黑客帝国》中的勺子的实例。那里没有勺子。

以下测试代码展示了Java中`null`的一些属性。

```
@Test
public void nullIsNotAnObject()
{
    // Assert things about null
    Assertions.assertFalse(null instanceof Object);
    final Set<?> set = null;
    Assertions.assertFalse(set instanceof Set);

    // Assert things about exceptions thrown when calling methods on null
    Assertions.assertThrows(
            NullPointerException.class,
            () -> set.getClass());
    Assertions.assertThrows(
            NullPointerException.class,
            () -> set.toString());

    // Assert things about a non-null variable named set2
    final Set<?> set2 = new HashSet<>(Set.of(1, 2, 3));
    set2.add(null);
    Assertions.assertNotNull(set2);
    Assertions.assertNotNull(set2.getClass());
    Assertions.assertTrue(set2 instanceof Set);

    // Filter all of the non-null values from set2 in the Set named set3
    // Uses a static method refererence from the Objects class
    final Set<?> set3 = set2.stream()
            .filter(Objects::nonNull)
            .collect(Collectors.toUnmodifiableSet());
    Assertions.assertEquals(Set.of(1, 2, 3), set3);
}
```

在这个测试中，我断言`null`不是`Object`的实例。我初始化了一个名为`set`且类型为`Set<?>`的最终变量为`null`，并进一步`assert`，即`set`（即`null`）不是`Set`的实例。我`assert`，当我在`set`上调用`getClass()`或`toString()`时（它仍然是`null`），会抛出`NullPointerException`。这是因为`null`不是`Object`。注意，我在这里将`set`变量的第一个声明设置为`final`，以便在第一部分的两个lambda中引用该变量，并且我`assert`，`NullPointerException`会被抛出。如果我不尝试重新设置值，我本可以将其留作“有效最终值”，但我觉得这样做更明确。

在第二部分中，我创建了一个可变的`Set`并将其存储在名为`set2`的变量中。我向`set`添加了`null`。`Set.of()`不会接受空值，因此我不得不将不可变的`Set`转换为`HashSet`，它可以接受`null`值。我手动将`null`添加到`set2`中。我断言`set2`不是`null`，它的类不是`null`，并且`set2`确实是`Set`的一个实例，正如编译器所说的那样。

最后，我使用`filter`方法过滤`set2`中的实例，只要它们对`Objects::nonNull`方法引用返回true即可。该方法引用了`Objects`上的静态方法，返回一个`Predicate`，检查`object != null`。再次强调，由于`null`不是对象，因此无法调用任何方法来构造有效的方法引用作为`Predicate`。

这就是我们在Java中所熟悉的`null`。它不是`Object`。我们在编写Java代码时都学会了如何处理`null`。以前用过Smalltalk的人会知道还有一种不同的方式。

## Smalltalk中的字面`nil`

在Smalltalk中，有一个名为`nil`的单例对象实例。字面`nil`是`UndefinedObject`类的一个实例。`UndefinedObject`是`Object`的子类。`Object`类是`nil`的一个子类……这种循环定义使许多程序员的大脑瘫痪，包括我自己的。不知何故，这一切都能正常工作。这是Smalltalk的神奇之处之一。顶部有一只乌龟，坐在另一只乌龟上，这只乌龟一直在下面。

以下测试代码通过了[Pharo Smalltalk](https://pharo.org/) 11.0。

```
testNilIsAnObject
 |set setNoNils|

 # Assert things about nil
 self assert: nil isNil.
 self assert: 'nil' equals: nil printString.

 # Assert things about the set variable which is nil
 set := nil.
 self assert: set isNil.
 self assert: set equals: nil.
 self assert: set class equals: UndefinedObject.
 self assert: (set ifNil: [ true ]).
 self assert: set isEmptyOrNil.

 # Assert things about the set variable which is not nil
 set := Set with: 1 with: 2 with: 3 with: nil.
 self deny: set isNil.
 self assert: set isNotNil.
 self deny: set equals: nil.
 self deny: set isEmptyOrNil.
 self assert: set class equals: Set.

 # Select all the non-nil values into a new Set name setNoNils
 setNoNils := set select: #isNotNil.
 self assert: (Set with: 1 with: 2 with: 3) equals: setNoNils.
```

上面是在Smalltalk中方法定义的样子。我写了一个名为`testNilIsAnObject`的单元测试方法。我通过在方法名后面的两个竖线之间声明来定义了两个临时变量`set`和`setNoNils`，就像这样`|set setNoNils|`。在第一段代码中，我断言了关于`nil`的一些事实，以证明它实际上是`Object`的一个实例。字面量`self`在Smalltalk中等同于Java中的`this`，指的是`testNilIsAnObject`所定义的类的实例，该实例继承了名为`assert:`、`deny:`和`assert:equals:`的方法。我断言`nil`可以像Smalltalk中的任何其他对象一样响应消息。我断言`nil`对`isNil`的响应为`true`。我还断言，在`nil`上调用`printString`会返回`String` `'nil'`。

`:=`操作符在Smalltalk中用于变量赋值。我将字面量`nil`引用的实例赋给名为`set`的变量。我断言当向`set`发送消息`isNil`时，`set`响应`true`。我断言`set`变量中包含的对象引用是`UndefinedObject`的一个实例。我断言在`set`上调用`ifNil:`消息返回`true`。代码`[ true ]`是一个零参数块或lambda。在Java中，相当于`Supplier`类型的lambda。最后，我断言当向`set`发送消息`isEmptyOrNil`时，`set`响应`true`。

在第二段代码中，我通过使用名为`with:with:with:with:`的类方法创建了一个`Set`的实例，该方法接受四个参数。然后，我否定了集合`set`是`nil`。我断言集合`set`不为`nil`。我断言`set`不等于`nil`。我否定了集合`set`既不是`nil`也不是空。然后，我断言`set`的类是`Set`。

在第三段代码中，如果实例返回`true`给消息`isNotNil`，我会将所有包含在`set`中的实例选择到一个名为`setNoNils`的新`Set`中。重要的是，Smalltalk中`Object`的每个子类对消息`isNotNil`都会返回`true`或`false`。

这是Smalltalk中所有对象（包括`nil`）可用的方法的最后一个示例。该方法名为`ifNil:ifNotNil:`。它是一个控制结构方法，接受两个块（也称为lambda）参数。结果由类型多态确定。这里的`set`知道它不是`nil`，因此它将自动执行第二个块并返回结果，这里是`String` `'not nil'`。字面量`nil`将知道它是`nil`，因此它将自动执行第一个块并返回结果，这里是`String` `'nil'`。

```
# Use the built in control structures around nil on all objects
self assert: (set ifNil: [ 'nil' ] ifNotNil: [ 'not nil']) equals: 'not nil'. 
self assert: (nil ifNil: [ 'nil' ] ifNotNil: [ 'not nil']) equals: 'nil'.
```

## 如果在Java中null是一个Object会怎样？

下面的代码是推测性的，不会编译，未经Java语言专家测试和证明。但是如果`null`是一个名为`Null`的某个类的实例，下面的代码*可能*是可能的。

```
@Test
// NOTE THIS CODE WILL NOT COMPILE AND IS PURELY SPECULATIVE!!!
public void nullIsNotAnObject()
{
    // Assert things about null
    Assertions.assertTrue(null instanceof Object);
    final Set<?> set = null;
    Assertions.assertFalse(set instanceof Set);

    // Assert things about calling methods on null as an object
    Assertions.assertTrue(set.isNull());
    Assertions.assertEquals(Null.class, set.getClass());
    Assertions.assertEquals("null", set.toString());

    // Assert things about a non-null variable
    final Set<?> set2 = Set.of(1, 2, 3);
    Assertions.assertNotNull(set2);
    Assertions.assertNotNull(set2.getClass());
    Assertions.assertTrue(set2 instanceof Set);

    // Filter all of the non-null values from set2 in the Set named set3
    // Uses an instance method refererence from the Object class
    final Set<?> set3 = set2.stream()
            .filter(Object::isNotNull)
            .collect(Collectors.toUnmodifiableSet());

    // Calling methods defined on Object, that would be overridden in Null
    Assertions.assertEquals("null", null.ifNull(() -> "null")); 
    Assertions.assertNull(null.ifNotNull(() -> "notNull"));
    Assertions.assertEquals("not null", set3.ifNotNull(() -> " not null"));
}
```

如果 `null` 是一个名为 `Null` 的类的实例，那么就可以向 `Object` 和 `Null` 添加诸如 `ifNull`、`isNotNull`、`ifNotNull`、`isEmptyOrNull` 等方法，就像 Smalltalk 做的那样。`ifNull` 和 `ifNotNull` 方法将接受诸如 `Supplier`、`Consumer` 或 `Function` 等功能接口类型，然后与 lambda 和方法引用一起工作。在上面的示例中，我修改了 `filter` 代码，使用了一个从方法引用生成的 `Predicate`，该方法引用将在 `Object` 上定义的名为 `isNotNull` 的方法。

最棘手的部分将是如何使 `null` 能够代表任何接口和类，并且为 `null` 不理解的方法调用一个名为 `doesNotUnderstand` 的方法进行调度。`Null` 类必须像代理一样行为，对存储在其中的任何类型转发到一个单一方法，该方法可以适当处理“我不是一个 Set”响应。

或许在 Java 中将 `null` 作为一个对象，就可以彻底摆脱各种 `NullPointerException` 的问题。

## 为什么 `nil` 作为一个对象很有用呢？

Smalltalk 中的 `nil` 是 `UndefinedObject` 类的一个实例，这使得它可以像 Smalltalk 中的所有其他对象一样对待。它可以响应所有对象支持的基本方法。`nil` 实例在对象层次结构中拥有一流的待遇，每个对象都知道它是不是 `nil`。这为语言和库带来了一种良好的对称性。这并没有通过避免或排除来处理 `nil` 的需要，但处理 `nil` 使用的机制与处理或排除其他类型的机制相同，通过调用诸如 `isNil`、`isNotNil`、`ifNil:`、`ifNotNil:`、`ifNil:ifNotNil:` 等方法。

在像 Smalltalk 这样的语言中，控制结构并不作为语言中的语句定义，而是作为库中的方法定义。在 Smalltalk 中，`nil` 存在如同其他所有对象一样，作为一个对象，这带来了令人惊叹的一致性和清晰度。在我的下一篇博客中，我将更深入地探讨 Smalltalk 中其他控制结构是如何在类库中定义的。

# 最后思考

我并不打算改变 Java，因为关于 `null` 的讨论早已很久以前就有了定论。我所期望的是，我可以帮助那些可能还没有看到其他面向对象编程语言中处理 `null` 不同方式的 Java 开发者。

本博客旨在简单解释 Java 中的 `null` 与 Smalltalk 中的 `nil` 之间的区别。学习多种使用不同策略解决同一问题的语言是有益的。学习一门全新的语言需要时间。我希望这个简要比较能帮助你理解不同方法的利弊。

在我的下一篇博客中，我计划介绍 Smalltalk 中的另外两个文字对象，分别命名为`true`和`false`，并将它们与它们的原始 Java 等效物进行比较，也命名为`true`和`false`。我还将它们与 Java 对象等效物命名为`Boolean`进行比较。我还将探讨如何在库中有效地定义控制结构，而不需要特殊的语法和保留字，比如`if`、`for`、`while`语句。

敬请关注，感谢阅读！

*我是* [*Eclipse Collections*](https://github.com/eclipse/eclipse-collections) *开源项目的创建者和提交者，该项目由* [*Eclipse Foundation*](https://projects.eclipse.org/projects/technology.collections) *管理。Eclipse Collections 欢迎* [*贡献*](https://github.com/eclipse/eclipse-collections/blob/master/CONTRIBUTING.md)*。*
