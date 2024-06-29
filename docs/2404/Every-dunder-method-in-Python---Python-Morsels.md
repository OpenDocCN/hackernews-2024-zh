<!--yml

category: 未分类

date: 2024-05-27 12:53:16

-->

# Python 中的每个 dunder 方法 - Python Morsels

> 来源：[https://www.pythonmorsels.com/every-dunder-method/](https://www.pythonmorsels.com/every-dunder-method/)

您刚刚创建了一个类。您编写了一个 `__init__` 方法。接下来该做什么？

Python 包括*大量*[dunder methods](https://www.pythonmorsels.com/what-are-dunder-methods/)（双下划线方法），这些方法允许我们深度定制我们的自定义类与 Python 的许多功能交互的方式。您可以添加哪些 dunder 方法到您的类中，以使其对其他使用它的 Python 程序员更友好？

让我们详细了解 **Python 中的每个 dunder 方法**，重点介绍每种方法何时特别有用。

请注意，Python 文档将这些方法称为[special methods](https://docs.python.org/3/glossary.html#term-special-method)，并指出了“magic method”的同义词，但*很少*使用“dunder method”一词。然而，“dunder method”是 Python 中相当常见的口头用语，如我在[非官方 Python 术语表](https://www.pythonmorsels.com/terms/)中所述。

您可以使用本页面上散布的链接获取有关任何特定 dunder 方法的更多详细信息。要查看所有方法的列表，请参见最后一节的速查表。

## 3 个必要的 dunder 方法 🔑

大多数类应具有 3 个 dunder 方法：[`__init__`](https://www.pythonmorsels.com/what-is-init/)、[`__repr__`](https://www.pythonmorsels.com/customizing-string-representation-your-objects/) 和 [`__eq__`](https://www.pythonmorsels.com/overloading-equality-in-python/)。

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `T(a, b=3)` | `T.__init__(x, a, b=3)` | `None` |
| `repr(x)` | `x.__repr__()` | `str` |
| `x == y` | `x.__eq__(y)` | 通常为 `bool` |

[`__init__`](https://www.pythonmorsels.com/what-is-init/) 方法是**初始化器**（不要与[构造函数](#construction-and-finalizing)混淆），[`__repr__`](https://www.pythonmorsels.com/customizing-string-representation-your-objects/) 方法自定义了对象的字符串表示形式，[`__eq__`](https://www.pythonmorsels.com/overloading-equality-in-python/) 方法自定义了对象之间的*相等*关系。

`__repr__` 方法在[Python REPL](https://www.pythonmorsels.com/using-the-python-repl/)和调试时特别有帮助。

## 相等性和可哈希性 🟰

除了 `__eq__` 方法外，Python 还有其他 2 个 dunder 方法，用于确定对象在与其他对象的关系中的“值”。

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `x == y` | `x.__eq__(y)` | 通常为 `bool` |
| `x != y` | `x.__ne__(y)` | 通常为 `bool` |
| `hash(x)` | `x.__hash__()` | `int` |

Python 的 `__eq__` 方法通常返回 `True`、`False` 或 [`NotImplemented`](https://www.pythonmorsels.com/when-to-use-notimplemented/)（如果对象无法比较）。 默认的 `__eq__` 实现依赖于 `is` 操作符，用于检查**[身份](https://www.pythonmorsels.com/equality-vs-identity/)**。

默认的 `__ne__` 实现调用 `__eq__` 并对给定的布尔返回值取反（或者如果 `__eq__` 返回 `NotImplemented` 则返回 `NotImplemented`）。 这种默认行为通常是“足够好的”，因此**你几乎永远不会看到实现 `__ne__`**。

可哈希对象可以用作字典的键或集合中的值。Python 中的所有对象默认都是[可哈希的](https://www.pythonmorsels.com/what-are-hashable-objects/)，但如果你编写了自定义的 `__eq__` 方法，则你的对象在没有自定义的 `__hash__` 方法时是*不可哈希的*。但是，**对象的哈希值绝对不能改变**，否则将会发生[不好的事情](https://pym.dev/p/2ysgz/)，因此**通常只有*不可变*对象实现 `__hash__`**。

若要实现相等性检查，请参阅[Python 中的 `__eq__`](https://www.pythonmorsels.com/overloading-equality-in-python/)。 若要实现可哈希性，请参阅[在 Python 中创建可哈希对象](https://www.pythonmorsels.com/making-hashable-objects/)。

## 可排序性 ⚖️

Python 的比较运算符（`<`、`>`、`<=`、`>=`）也可以通过魔术方法进行重载。 比较运算符还为依赖对象相对顺序的函数提供支持，如 `sorted`、`min` 和 `max`。

| 操作 | 魔术方法调用 | 返回值 |
| --- | --- | --- |
| `<` | `__lt__` | 通常是 `bool` |
| `>` | `__gt__` | 通常是 `bool` |
| `<=` | `__le__` | 通常是 `bool` |
| `>=` | `__ge__` | 通常是 `bool` |

如果你计划按照*典型*方式实现所有这些运算符（其中 `x < y` 等同于询问 `y > x`），那么 Python 的 `functools` 模块中的 [`total_ordering` 装饰器](https://docs.python.org/3/library/functools.html#functools.total_ordering) 会很有帮助。

## 类型转换和字符串格式化 ⚗️

Python 有许多魔术方法用于将对象转换为不同的类型。

| 函数 | 魔术方法调用 | 返回值 |
| --- | --- | --- |
| `str(x)` | `x.__str__()` | `str` |
| `bool(x)` | `x.__bool__()` | `bool` |
| `int(x)` | `x.__int__()` | `int` |
| `float(x)` | `x.__float__()` | `float` |
| `bytes(x)` | `x.__bytes__()` | [`字节`](https://pym.dev/p/2ysgz/) |
| `complex(x)` | `x.__complex__()` | [`复数`](https://docs.python.org/3/glossary.html#term-complex-number) |
| `f"{x:s}"` | `x.__format__(s)` | `str` |
| `repr(x)` | `x.__repr__()` | `str` |

`__bool__` 函数用于进行[真值检查](https://www.pythonmorsels.com/truthiness/)，尽管 `__len__` 作为后备。

如果您需要创建一个像 [`decimal.Decimal`](https://docs.python.org/3/library/decimal.html) 或 [`fractions.Fraction`](https://docs.python.org/3/library/fractions.html) 这样的对象，该对象能像数值一样使用，您需要实现 `__int__`、`__float__` 和 `__complex__` 方法，以便能将您的对象转换为其他数值。如果您需要创建一个可以在 `memoryview` 中使用或能以其他方式转换为 `bytes` 的对象，您需要实现 `__bytes__` 方法。

`__format__` 和 `__repr__` 方法是不同的字符串转换方式。大多数字符串转换依赖于 `__str__` 方法，但默认的 `__str__` 实现只是简单地调用 `__repr__`。

`__format__` 方法被所有 [f-string 转换](https://www.pythonmorsels.com/string-formatting/)、`str` 类的 `format` 方法以及（很少使用的）内置 `format` 函数使用。该方法允许 `datetime` 对象支持自定义的格式说明符，详情参见 [支持自定义格式说明符](https://www.pythonmorsels.com/string-formatting/#formatting-datetime-objects)。

## 上下文管理器 🚪

上下文管理器是一个可以在 `with` 块中使用的对象。

| 使用 | 双下划线方法调用 | 返回值 |
| --- | --- | --- |
| `with` 块进入 | `x.__enter__()` | 作为给定的值 |
| `with` 块退出 | `x.__exit__(exc_type, exc, traceback)` | 真值/假值 |

关于上下文管理器的更多信息，请参见，[什么是上下文管理器](https://www.pythonmorsels.com/what-is-a-context-manager/)和[创建上下文管理器](https://www.pythonmorsels.com/creating-a-context-manager/)。

## 容器和集合 🗃️

集合（即容器）本质上是充当数据结构或者像数据结构一样的对象。列表、字典、集合、字符串和元组都是集合的例子。

| 操作 | 双下划线方法调用 | 返回类型 | 已实现 |
| --- | --- | --- | --- |
| `len(x)` | `x.__len__()` | 整数 | 非常常见 |
| `iter(x)` | `x.__iter__()` | 迭代器 | 非常常见 |
| `for item in x: ...` | `x.__iter__()` | 迭代器 | 非常常见 |
| `x[a]` | `x.__getitem__(a)` | 任何对象 | 常见 |
| `x[a] = b` | `x.__setitem__(a, b)` | 无 | 常见 |
| `del x[a]` | `x.__delitem__(a)` | 无 | 常见 |
| `a in x` | `x.__contains__(a)` | 布尔值 | 常见 |
| `reversed(x)` | `x.__reversed__()` | 迭代器 | 常见 |
| `next(x)` | `x.__next__()` | 任何对象 | 不常见 |
| `x[a]` | `x.__missing__(a)` | 任何对象 | 不常见 |
| `operator.length_hint(x)` | `x.__length_hint__()` | 整数 | 不常见 |

`__iter__` 方法被 `iter` 函数使用 *以及* 所有形式的迭代：[`for` 循环](https://www.pythonmorsels.com/writing-a-for-loop/)、[推导式](https://www.pythonmorsels.com/what-are-list-comprehensions/)、[元组解包](https://www.pythonmorsels.com/tuple-unpacking/) 和 [使用 `*` 进行可迭代解包](https://www.pythonmorsels.com/unpacking-iterables-iterables/)。

虽然`__iter__`方法对于创建自定义可迭代对象是必需的，但`__next__`方法对于创建自定义迭代器（这种情况较少见）是必需的。`__missing__`方法仅在`dict`类自身调用时才会被调用，除非其他类决定实现`__missing__`。`__length_hint__`方法为不支持`__len__`的结构提供长度猜测，以便更高效地预分配列表或其他结构。

另请参阅：[迭代器协议](https://www.pythonmorsels.com/iterator-protocol/)，[实现`__len__`](https://www.pythonmorsels.com/making-the-len-function-work-on-your-python-objects/)，以及[实现`__getitem__`](https://www.pythonmorsels.com/supporting-index-and-key-lookups/)。

## 可调用性 ☎️

函数、类以及所有其他[可调用对象](https://www.pythonmorsels.com/callables/)依赖于`__call__`方法。

| 操作 | 双下划线方法调用 | 返回类型 |
| --- | --- | --- |
| `x(a, b=c)` | `x.__call__(a, b=c)` | 任何对象 |

当一个类被调用时，它的[元类](https://docs.python.org/3/glossary.html#term-metaclass)的`__call__`方法会被使用。当一个类的*实例*被调用时，则使用该类的`__call__`方法。

欲了解更多关于可调用性的内容，请参见[可调用对象：Python的“函数”有时是类](https://www.pythonmorsels.com/class-function-and-callable/)。

## 算术运算符 ➗

Python的双下划线方法通常被描述为"运算符重载"的工具。大部分这种"运算符重载"是通过Python的各种算术运算符来实现的。

有两种方法可以分解算术运算符：

+   数学运算符（如`+`、`-`、`*`、`/`、`%`）与位运算符（如`&`、`|`、`^`、`>>`、`~`）之间的区别。

+   二进制运算符（作用于两个值，如`x + y`）与一元运算符（作用于一个值之前，如`+x`）。

数学运算符比位运算符更常见，而二进制运算符比一元运算符稍常见一些。

这些是二进制数学算术运算符：

| 操作 | 左操作数方法 | 右操作数方法 | 描述 |
| --- | --- | --- | --- |
| `x + y` | `__add__` | `__radd__` | 加法 / 连接 |
| `x - y` | `__sub__` | `__rsub__` | 减法 |
| `x * y` | `__mul__` | `__rmul__` | 乘法 |
| `x / y` | `__truediv__` | `__rtruediv__` | 除法 |
| `%` | `__mod__` | `__rmod__` | 取模 |
| `x // y` | `__floordiv__` | `__rfloordiv__` | [整数除法](https://www.pythonmorsels.com/integer-division/) |
| `**` | `__pow__` | `__rpow__` | 指数运算 |
| `x @ y` | `__matmul__` | `__rmatmul__` | 矩阵乘法 |

每个这些运算符都包括左操作数和右操作数方法。如果`x.__add__(y)`返回[`NotImplemented`](https://www.pythonmorsels.com/when-to-use-notimplemented/)，则将尝试`y.__radd__(x)`。更多内容请参见[算术双下划线方法](https://www.pythonmorsels.com/arithmetic-dunder-methods/)。

这些是二进制位运算的算术运算符：

| 操作 | 左操作数方法 | 右操作数方法 | 描述 |
| --- | --- | --- | --- |
| `x & y` | `__and__` | `__rand__` | 与 |
| `x &#124; y` | `__or__` | `__ror__` | 或 |
| `x ^ y` | `__xor__` | `__rxor__` | 异或 |
| `x >> y` | `__rshift__` | `__rrshift__` | 右移 |
| `x << y` | `__lshift__` | `__rlshift__` | 左移 |

这些是 Python 的一元算术运算符：

| 操作 | Dunder 方法 | 种类 | 描述 |
| --- | --- | --- | --- |
| `-x` | `__neg__` | 数学 | 取反 |
| `+x` | `__pos__` | 位运算 | 肯定 |
| `~x` | `__invert__` | 位运算 | 取反 |

一元 `+` 运算符通常[没有效果](https://stackoverflow.com/questions/16819023/whats-the-purpose-of-the-pos-unary-operator-in-python)，尽管某些对象使用它进行特定操作。例如，在[collections.Counter 对象上使用 `+`](https://www.pythonmorsels.com/using-counter/#removing-negative-counts)会移除非正值。

Python 的算术运算符经常用于非算术的目的：[序列](https://www.pythonmorsels.com/sequence/)使用 `+` 进行连接和 `*` 进行自连接，而[集合](https://www.pythonmorsels.com/practical-uses-of-sets/)使用 `&` 进行交集，`|` 进行并集，`-` 进行非对称差异，`^` 进行对称差异。算术运算符有时也被重载以实现更有创意的用途。例如，`pathlib.Path` 对象使用 `/` 来创建子路径。

## 原地算术运算 ♻️

Python 包含许多用于**原地**操作的双下划线方法。如果你正在创建一个支持任何算术操作的[可变对象](https://www.pythonmorsels.com/terms/#mutable)，你将需要实现相关的原地双下划线方法。

| 操作 | Dunder 方法调用 | 返回 |
| --- | --- | --- |
| `x += y` | `x.__iadd__(y)` | 通常是 `self` |
| `x -= y` | `x.__isub__(y)` | 通常是 `self` |
| `x *= y` | `x.__imul__(y)` | 通常是 `self` |
| `x /= y` | `x.__itruediv__(y)` | 通常是 `self` |
| `x %= y` | `x.__imod__(y)` | 通常是 `self` |
| `x //= y` | `x.__ifloordiv__(y)` | 通常是 `self` |
| `x **= y` | `x.__ipow__(y)` | 通常是 `self` |
| `x @= y` | `x.__imatmul__(y)` | 通常是 `self` |
| `x &= y` | `x.__iand__(y)` | 通常是 `self` |
| `x &#124;= y` | `x.__ior__(y)` | 通常是 `self` |
| `x ^= y` | `x.__ixor__(y)` | 通常是 `self` |
| `x >>= y` | `x.__irshift__(y)` | 通常是 `self` |
| `x <<= y` | `x.__ilshift__(y)` | 通常是 `self` |

所有的 Python 二进制算术运算符都适用于**增强赋值语句**，这涉及使用操作符后跟 `=` 符号来对对象赋值同时进行操作。

可变对象上的**增强赋值**预期会[改变原始对象](https://www.pythonmorsels.com/augmented-assignments-mutate/)，这要归功于可变对象实现了适当的原地算术双下划线方法。

当找不到就地操作的双下划线方法时，Python 执行操作，然后进行赋值。**不可变对象通常不实现就地操作的双下划线方法**，因为它们应该返回一个新对象而不是修改原始对象。

## 内置数学函数 🧮

Python 还包括了许多与数学相关的双下划线方法，包括[内置函数](https://www.pythonmorsels.com/built-in-functions-in-python/#type)和 `math` 库中的一些函数。

| 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- |
| `divmod(x, y)` | `x.__divmod__(y)` | 2-项元组 |
| `divmod(x, y)` | `y.__rdivmod__(x)` | 2-项元组 |
| `abs(x)` | `x.__abs__()` | `float` |
| `sequence[x]` | `x.__index__()` | `int` |
| `round(x)` | `x.__round__()` | 数字 |
| `math.trunc(x)` | `x.__trunc__()` | 数字 |
| `math.floor(x)` | `x.__floor__()` | 数字 |
| `math.ceil(x)` | `x.__ceil__()` | 数字 |

Python 的 `divmod` 函数同时执行[整数除法](https://www.pythonmorsels.com/integer-division/)（`//`）和取模运算（`%`）。请注意，与许多二元算术运算符一样，`divmod` 如果需要请求第二个参数处理操作，则也会检查 `__rvidmod__` 方法。

`__index__` 方法用于创建类似整数的对象。该方法将对象无损转换为整数，与 `__int__` 方法不同，后者可能会执行“有损”的整数转换（例如从 `float` 到 `int`）。它被要求需要*真正*整数的操作使用，例如[切片](https://www.pythonmorsels.com/slicing/)、索引和 `bin`、`hex`、`oct` 函数（[示例](https://pym.dev/p/2k9zt/)）。

## 属性访问 📜

Python 甚至包括了双下划线方法，用于控制访问、删除或赋值对象的任何属性！

| 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- |
| `x.missing` | `x.__getattr__("missing")` | 属性值 |
| `x.anything` | `x.__getattribute__("anything")` | 属性值 |
| `x.thing = value` | `x.__setattr__("thing", value)` | `None` |
| `del x.thing` | `x.__delattr__("thing")` | `None` |
| `dir(x)` | `x.__dir__()` | 字符串列表 |

`__getattribute__` 方法在*每次*属性访问时被调用，而 `__getattr__` 方法仅在 Python 未能找到特定属性时才被调用。所有方法调用和属性访问都调用 `__getattribute__`，因此正确实现它具有挑战性（由于意外的[递归](https://www.pythonmorsels.com/what-is-recursion/)）。查看[`__getattr__` versus `__getattribute__`](https://www.pythonmorsels.com/getattr-vs-getattribute/)，了解演示这两种方法之间差异的示例。

`__dir__` 方法应返回一个属性名称的可迭代字符串集合。当[ `dir` 函数](https://www.pythonmorsels.com/built-in-functions-in-python/#dir)调用 `__dir__` 时，它将返回的可迭代对象转换为排序列表（类似于[`sorted`](https://www.pythonmorsels.com/sorting-in-python/) 的操作）。

内置的 `getattr`、[`setattr`](https://www.pythonmorsels.com/python-setattr/) 和 `delattr` 函数对应于同名的 Dunder 方法，但它们仅用于动态属性访问（不是 *所有* 属性访问）。

现在我们进入了真正不寻常的 Dunder 方法。Python 包含许多用于元编程相关功能的 Dunder 方法。

| 实现于 | 操作 | Dunder 方法调用 | 返回 |
| --- | --- | --- | --- |
| 元类 | `class T: ...` | `type(base).__prepare__()` | 映射 |
| 元类 | `isinstance(x, T)` | `T.__instancecheck__(x)` | `bool` |
| 元类 | `issubclass(U, T)` | `T.__subclasscheck__(U)` | `bool` |
| 任意类 | `class U(T): ...` | `T.__init_subclass__(U)` | `None` |
| 任意类 | （手动调用） | `T.__subclasses__()` | `list` |
| 任意类 | `class U(x): ...` | `x.__mro_entries__([x])` | `tuple` |
| 任意类 | `T[y]` | `T.__class_getitem__(y)` | 一个项目 |

`__prepare__` 方法自定义了用于类初始命名空间的字典。它用于预先填充字典值或自定义字典类型（[愚蠢的例子](https://pym.dev/p/23wfv/)）。

`__instancecheck__` 和 `__subclasscheck__` 方法覆盖了 `isinstance` 和 `issubclass` 的功能。Python 的 ABCs 使用这些方法来实现[鹅类型](https://www.pythonmorsels.com/goose-typing/)（[鸭子类型](https://www.pythonmorsels.com/duck-typing/)在类型检查时的应用）。

`__init_subclass__` 方法允许类挂钩子类初始化（[示例](https://pym.dev/p/246z6/)）。类还有一个 `__subclasses__` 方法（在它们的[元类](https://docs.python.org/3/glossary.html#term-metaclass)上），但通常不会被覆盖。

Python 在[类继承](https://www.pythonmorsels.com/inheriting-one-class-another/)期间调用 `__mro_entries__` 处理任何实际上不是类的基类。[`typing.NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple) 函数使用此功能来假装它是一个类（[请见此处](https://pym.dev/p/2qgzd/)）。

`__class_getitem__` 方法允许类成为可索引的（*无需*其元类需要一个 `__getitem__` 方法）。这通常用于启用花式类型注释（例如 `list[int]`）。

## 描述符 🏷️

[描述符](https://docs.python.org/3/glossary.html#term-descriptor)是附加到类时可以挂钩到该类上的属性名称访问的对象。

| 操作 | Dunder 方法调用 | 返回 |
| --- | --- | --- |
| `class T: x = U()` | `T.x.__set_name__(T, 'x')` | `None` |
| `t.x` | `T.x.__get__(t, T)` | 该值 |
| `t.x = y` | `T.x.__set__(t, y)` | `None` |
| `del t.x` | `T.x.__delete__(t)` | `None` |

描述符协议主要用于使 Python 的 `property` 装饰器工作，虽然也被许多第三方库使用。

## 缓冲区 💾

实现低级内存数组？你需要 Python 的 [缓冲区协议](https://docs.python.org/3/reference/datamodel.html#emulating-buffer-types)。

| 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- |
| `memoryview(x)` | `x.__buffer__(flags)` | `memoryview` |
| `del memoryview(x)` | `x.__release_buffer__(m)` | `None` |

当从 `__buffer__` 返回的缓冲区被删除时，会调用 `__release_buffer__` 方法。

Python 的缓冲区协议通常由 C 实现，因为它适用于低级对象。

## 异步操作 🤹

想要实现异步上下文管理器？你需要这些双下划线方法：

+   `__aenter__`: 就像 `__enter__` 一样，但它返回一个可等待对象

+   `__aexit__`: 就像`__exit__`一样，但它返回一个可等待对象。

需要支持异步迭代？你需要这些双下划线方法：

+   `__aiter__`: 必须返回一个异步迭代器

+   `__anext__`: 像`__next__`或非异步迭代器一样，但它必须返回一个可等待对象，且应该引发`StopAsyncIteration`而不是`StopIteration`。

需要制作自己的可等待对象？你需要这个双下划线方法：

+   `__await__`: 返回一个迭代器

我在自定义异步对象方面经验有限，请查找其他地方获取更多详情。

## 构造和结束 🏭

最后几个双下划线方法与对象的创建和销毁有关。

| 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- |
| `T(a, b=3)` | `T.__new__(T, a, b=3)` | 新实例 (`x`) |
| `T(a, b=3)` | `T.__init__(x, a, b=3)` | `None` |
| `del x` | `x.__del__()` | `None` |

调用类返回一个新的类实例，这要归功于 `__new__` 方法。`__new__` 方法是 Python 的 **构造方法**，尽管与许多编程语言中的构造方法不同，你几乎*永远不应该*定义自己的 `__new__` 方法。要控制对象创建，请优先使用初始化方法 (`__init__`)，而不是构造方法 (`__new__`)。[这里有一个奇怪的 `__new__` 示例](https://pym.dev/p/28r9m/)。

你可以把 `__del__` 视为一个 "析构" 方法，尽管它实际上被称为 **终结器方法**。在对象被删除之前，其 `__del__` 方法会被调用（[示例](https://pym.dev/p/2hexg/)）。文件实现了一个 `__del__` 方法，用于关闭文件及其可能链接到的任何二进制文件缓冲区。

## 特定于库的双下划线方法 🧰

一些标准库模块定义了不在其他地方使用的自定义双下划线方法：

+   [dataclasses](https://www.pythonmorsels.com/dataclasses/) 支持 `__post_init__` 方法

+   `abc.ABC` 类具有 `__subclasshook__` 方法，`abc.ABCMeta` 在其 `__subclasscheck__` 方法中调用它（更多内容请参见 [goose typing](https://www.pythonmorsels.com/goose-typing/)）

+   类似路径的对象具有 `__fspath__` 方法，返回文件路径字符串

+   Python 的 `copy` 模块将使用 `__copy__` 和 `__deepcopy__` 方法（如果存在）

+   Pickling 依赖 `__getnewargs_ex__` 或 `__getargs__`，尽管 `__getstate__` 和 `__setstate__` 可进一步自定义，而 `__reduce__` 或 `__reduce_ex__` 则更低级

+   `sys.getsizeof` 依赖于 `__sizeof__` 方法来获取对象的大小（以字节为单位）

## 魔术属性 📇

除了魔术方法之外，Python 还有许多非方法的 **魔术属性**。

这里是一些常见的魔术属性：

+   `__name__`: 函数、类或模块的名称

+   `__module__`: 函数或类所在的模块名

+   `__doc__`: 函数、类或模块的 [文档字符串](https://www.pythonmorsels.com/docstrings/)

+   `__class__`: 对象的类（使用 [Python 的 `type` 函数](https://www.pythonmorsels.com/built-in-functions-in-python/#type) 代替）

+   `__dict__`: 大多数对象将其属性存储在此处（参见 [属性存储在哪里？](https://www.pythonmorsels.com/where-are-attributes-stored/)）

+   `__slots__`: 使用此特性的类比使用`__dict__`的类更节省内存

+   `__match_args__`: 类可以定义一个元组，记录类在结构化模式匹配 (`match-case`) 中使用时位置属性的重要性

+   `__mro__`: 类的方法解析顺序，用于属性查找和 `super()` 调用

+   `__bases__`: 类的直接父类

+   `__file__`: 定义模块对象的文件（尽管并非总是存在！）

+   `__wrapped__`: 使用 [`functools.wraps`](https://docs.python.org/3/library/functools.html#functools.wraps) 装饰的函数使用此特性指向原始函数

+   `__version__`: 常用于指定包的版本

+   `__all__`: 模块可以使用此特性自定义`from my_module import *`的行为

+   `__debug__`: 使用 `-O` 运行 Python 将其设置为 `False`，并禁用 Python 的 `assert` 语句

这些仅是更常见的魔术方法。以下是一些更多的例子：

+   函数具有 `__defaults__`、`__kwdefaults__`、`__code__`、`__globals__` 和 `__closure__`

+   函数和类均具有 `__qualname__`、`__annotations__` 和 `__type_params__`

+   实例方法具有 `__func__` 和 `__self__`

+   模块还可以具有 `__loader__`、`__package__`、`__spec__` 和 `__cached__` 属性

+   包具有 `__path__` 属性

+   异常具有 `__traceback__`、`__notes__`、`__context__`、`__cause__` 和 `__suppress_context__`

+   描述符使用 `__objclass__`

+   元类使用 `__classcell__`

+   Python 的 `weakref` 模块使用 `__weakref__`

+   [通用别名](https://docs.python.org/3/library/stdtypes.html#type-annotation-types-generic-alias-union)具有 `__origin__`、`__args__`、`__parameters__` 和 `__unpacked__`

+   `sys` 模块具有指向原始 `stdout` 和 `stderr` 版本的 `__stdout__` 和 `__stderr__`

此外，各种标准库模块使用这些双下划线属性：`__covariant__`、`__contravariant__`、`__infer_variance__`、`__bound__`、`__constraints__`。Python 还包含一个内置的 `__import__` 函数，不建议使用（推荐使用 `importlib.import_module`），而 CPython 有一个 `__builtins__` 变量指向 `builtins` 模块（但这是一个实现细节，应在需要时显式导入 `builtins`）。此外，从 `__future__` 模块导入可以启用特定的 Python 功能标志，Python 还会在包中寻找 `__main__` 模块以使其作为 CLI 脚本运行。

这只是 Python 中 *大多数* 双下划线属性名称的速查表。 😵

## 每个双下划线方法：速查表 ⭐

这是按类别组织的每个 Python 双下划线方法，并且非常粗略地按照 **最常见** 方法排序。以下是一些注意事项。

| 类别 | 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- | --- |
| 对象创建 | `x = T(a, b)` | `x.__init__(a, b)` | `None` |
| 对象创建 | `x = T(a, b)` | `T.__new__(T, a, b)` | 新实例 (`x`) |
| 终结器 | `del x`（类似） | `x.__del__()` | `None` |
| 比较 | `x == y` | `x.__eq__(y)` | 通常是 `bool` |
| 比较 | `x != y` | `x.__ne__(y)` | 通常是 `bool` |
| 比较 | `x < y` | `x.__lt__(y)` | 通常是 `bool` |
| 比较 | `x > y` | `x.__rt__(y)` | 通常是 `bool` |
| 比较 | `x <= y` | `x.__le__(y)` | 通常是 `bool` |
| 比较 | `x >= y` | `x.__ge__(y)` | 通常是 `bool` |
| 可哈希性 | `hash(x)` | `x.__hash__()` | `int` |
| 转换 | `repr(x)` | `x.__repr__()` | 总是 `str` |
| 转换 | `str(x)` | `x.__str__()` | 总是 `str` |
| 转换 | `bool(x)` | `x.__bool__()` | 总是 `bool` |
| 转换 | `int(x)` | `x.__int__()` | 总是 `int` |
| 转换 | `float(x)` | `x.__float__()` | 总是 `float` |
| 转换 | `bytes(x)` | `x.__bytes__()` | 总是 `bytes` |
| 转换 | `complex(x)` | `x.__complex__()` | 总是 `complex` |
| 转换 | `format(x, s)` | `x.__format__(s)` | 总是 `str` |
| 上下文管理器 | `with x as c:` | `x.__enter__()` | 对象 `c` |
| 上下文管理器 | `with x as c:` | `x.__exit__()` | 真值/假值 |
| 集合 | `len(x)` | `x.__len__()` | `int` |
| 集合 | `iter(x)` | `x.__iter__()` | 一个迭代器 |
| 集合 | `x[a]` | `x.__getitem__(a)` |  |
| 集合 | `x[a] = b` | `x.__setitem__(a, b)` | `None` |
| 集合 | `del x[a]` | `x.__delitem__(a)` | `None` |
| 集合 | `a in x` | `x.__contains__(a)` | `bool` |
| 集合 | `reversed(x)` | `x.__reversed__()` | 一个迭代器 |
| 集合 | `next(x)` | `x.__next__()` | 下一个迭代器项 |
| 集合 | `x[a]` | `x.__missing__(a)` |  |
| 集合 |  | `x.__length_hint__()` | `int` |
| 算术运算 | `x + y` | `x.__add__(y)` |  |
| 算术运算 | `x + y` | `y.__radd__(x)` |  |
| Arithmetic | `x - y` | `x.__sub__(y)` |  |
| Arithmetic | `x - y` | `y.__rsub__(x)` |  |
| Arithmetic | `x * y` | `x.__mul__(y)` |  |
| Arithmetic | `x * y` | `y.__rmul__(x)` |  |
| Arithmetic | `x / y` | `x.__truediv__(y)` |  |
| Arithmetic | `x / y` | `y.__rtruediv__(x)` |  |
| Arithmetic | `x % y` | `x.__mod__(y)` |  |
| Arithmetic | `x % y` | `y.__rmod__(x)` |  |
| Arithmetic | `x // y` | `x.__floordiv__(y)` |  |
| Arithmetic | `x // y` | `y.__rfloordiv__(x)` |  |
| Arithmetic | `x ** y` | `x.__pow__(y)` |  |
| Arithmetic | `x ** y` | `y.__rpow__(x)` |  |
| Arithmetic | `x @ y` | `x.__matmul__(y)` |  |
| Arithmetic | `x @ y` | `y.__rmatmul__(x)` |  |
| Arithmetic | `x & y` | `x.__and__(y)` |  |
| Arithmetic | `x & y` | `y.__rand__(x)` |  |
| Arithmetic | `x &#124; y` | `x.__or__(y)` |  |
| Arithmetic | `x &#124; y` | `y.__ror__(x)` |  |
| Arithmetic | `x ^ y` | `x.__xor__(y)` |  |
| Arithmetic | `x ^ y` | `y.__rxor__(x)` |  |
| Arithmetic | `x >> y` | `x.__rshift__(y)` |  |
| Arithmetic | `x >> y` | `y.__rrshift__(x)` |  |
| Arithmetic | `x << y` | `x.__lshift__(y)` |  |
| Arithmetic | `x << y` | `y.__rlshift__(x)` |  |
| Arithmetic | `-x` | `x.__neg__()` |  |
| Arithmetic | `+x` | `x.__pos__()` |  |
| Arithmetic | `~x` | `x.__invert__()` |  |
| Math functions | `divmod(x, y)` | `x.__divmod__(y)` | 二元组 |
| Math functions | `abs(x)` | `x.__abs__()` | `浮点数` |
| Math functions |  | `x.__index__()` | 整数 |
| Math functions | `round(x)` | `x.__round__()` | 数字 |
| Math functions | `math.trunc(x)` | `x.__trunc__()` | 数字 |
| Math functions | `math.floor(x)` | `x.__floor__()` | 数字 |
| Math functions | `math.ceil(x)` | `x.__ceil__()` | 数字 |
| Assignment | `x += y` | `x.__iadd__(y)` | 通常为`self` |
| Assignment | `x -= y` | `x.__isub__(y)` | 通常为`self` |
| Assignment | `x *= y` | `x.__imul__(y)` | 通常为`self` |
| Assignment | `x /= y` | `x.__itruediv__(y)` | 通常为`self` |
| Assignment | `x %= y` | `x.__imod__(y)` | 通常为`self` |
| Assignment | `x //= y` | `x.__ifloordiv__(y)` | 通常为`self` |
| Assignment | `x **= y` | `x.__ipow__(y)` | 通常为`self` |
| Assignment | `x @= y` | `x.__imatmul__(y)` | 通常为`self` |
| Assignment | `x &= y` | `x.__iand__(y)` | 通常为`self` |
| Assignment | `x &#124;= y` | `x.__ior__(y)` | 通常为`self` |
| Assignment | `x ^= y` | `x.__ixor__(y)` | 通常为`self` |
| Assignment | `x >>= y` | `x.__irshift__(y)` | 通常为`self` |
| Assignment | `x <<= y` | `x.__ilshift__(y)` | 通常为`self` |
| Attributes | `x.y` | `x.__getattribute__('y')` |  |
| Attributes | `x.y` | `x.__getattr__('y')` |  |
| Attributes | `x.y = z` | `x.__setattr__('y', z)` | `None` |
| Attributes | `del x.y` | `x.__delattr__('y')` | `None` |
| Attributes | `dir(x)` | `x.__dir__()` | 可迭代对象 |
| Descriptors | `class T: x = U()` | `T.x.__set_name__(T, 'x')` | `None` |
| Descriptors | `t.x` | `T.x.__get__(t, T)` |  |
| Descriptors | `t.x = y` | `T.x.__set__(t, y)` | `None` |
| 描述符 | `del t.x` | `T.x.__delete__(t)` | `None` |
| 类的相关内容 | `class U(T): ...` | `T.__init_subclass__(U)` | `None` |
| 类的相关内容 | `class U(x): ...` | `x.__mro_entries__([x])` | `tuple` |
| 类的相关内容 | `T[y]` | `T.__class_getitem__(y)` |  |
| 元类 | `class T: ...` | `type(base).__prepare__()` | `dict`/映射 |
| 元类 | `isinstance(x, T)` | `T.__instancecheck__(x)` | `bool` |
| 元类 | `issubclass(U, T)` | `T.__subclasscheck__(U)` | `bool` |
| 异步 | `await x`（近似） | `x.__await__()` | 一个迭代器 |
| 异步 | `async with x:` | `x.__aenter__()` | 一个可等待对象 |
| 异步 | `async with x:` | `x.__aexit__()` | 一个可等待对象 |
| 异步 | `async for a in x:` | `x.__aiter__()` | 一个可等待对象 |
| 异步 | `async for a in x:` | `x.__anext__()` | 一个可等待对象 |
| 缓冲区 | `memoryview(x)` | `x.__buffer__(flags)` | `memoryview` |
| 缓冲区 | `del memoryview(x)` | `x.__release_buffer__(m)` | `None` |

上述表格略微但是一致地*不真实*。 这些双下划线方法大多数情况下不是直接在对象上调用，而是在该对象的*类型*上调用：`type(x).__add__(x, y)`而不是`x.__add__(y)`。 这种区别在元类方法中非常重要。

我还特意排除了特定于库的双下划线方法（如`__post_init__`）和您不太可能定义的双下划线方法（如`__subclasses__`）。 请参阅下文。

| 分类 | 操作 | 双下划线方法调用 | 返回值 |
| --- | --- | --- | --- |
| 数据类 | `x = T(a, b)` | `T.__post_init__(a, b)` | `None` |
| 复制 | `copy.copy(x)` | `x.__copy__()` | 新对象 |
| 复制 | `copy.deepcopy(x)` | `x.__deepcopy__(memo)` | 新对象 |
| [Pickling](https://docs.python.org/3/library/pickle.html#pickling-class-instances) | `pickle.dumps(x)` | `x.__getnewargs__()` | 一个2项元组 |
| Pickling | `pickle.dumps(x)` | `x.__getnewargs_ex__()` | 一个2项元组 |
| Pickling | `pickle.dumps(x)` | `x.__getstate__()` | 一个有意义的状态 |
| Pickling | `pickle.dumps(x)` | `x.__reduce__()` | 一个2-6项元组 |
| Pickling | `pickle.dumps(x)` | `x.__reduce_ex__(4)` | 一个2-6项元组 |
| Pickling | `pickle.loads(b)` | `x.__setstate__(state)` | `None` |
| pathlib | [`os.fspath(x)`](https://docs.python.org/3/library/os.html#os.fspath) | `p.__fspath__()` | `str`或`bytes` |
| sys | `sys.getsizeof(x)` | `x.__sizeof__()` | `int`（字节大小） |
| 类的相关内容 | None？ | `x.__subclasses__()` | 子类的可迭代对象 |
| ABCs | `issubclass(U, T)` | `T.__subclasshook__(U)` | `bool` |

因此，Python包括了103个“普通”的双下划线方法，12个特定于库的双下划线方法，以及至少52个其他各种类型的双下划线属性。 这超过了150个独特的`__dunder__`名称！我**不建议**记忆这些：让Python完成其工作，并在需要实现/查找的双下划线方法或属性时查阅它。

记住，**不要自己发明双下划线方法**。有时你会看到一些第三方库确实发明了他们自己的双下划线方法，但这并不被鼓励，而且对那些偶然遇到这些方法并认为它们是“*真正*”双下划线方法的用户来说，可能会相当困惑。
