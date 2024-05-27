<!--yml
category: 未分类
date: 2024-05-27 12:53:16
-->

# Every dunder method in Python - Python Morsels

> 来源：[https://www.pythonmorsels.com/every-dunder-method/](https://www.pythonmorsels.com/every-dunder-method/)

You've just made a class. You made a `__init__` method. Now what?

Python includes *tons* of [dunder methods](https://www.pythonmorsels.com/what-are-dunder-methods/) ("double underscore" methods) which allow us to deeply customize how our custom classes interact with Python's many features. What dunder methods could you add to your class to make it friendly for other Python programmers who use it?

Let's take a look at **every dunder method in Python**, with a focus on when each method is useful.

Note that the Python documentation refers to these as [special methods](https://docs.python.org/3/glossary.html#term-special-method) and notes the synonym "magic method" but *very* [rarely](https://docs.python.org/3/reference/lexical_analysis.html#reserved-classes-of-identifiers) uses the term "dunder method". However, "dunder method" is a fairly common Python colloquialism, as noted in my [unofficial Python glossary](https://www.pythonmorsels.com/terms/).

You can use the links scattered throughout this page for more details on any particular dunder method. For a list of all of them, see the cheat sheet in the final section.

## The 3 essential dunder methods 🔑

There are 3 dunder methods that *most* classes should have: [`__init__`](https://www.pythonmorsels.com/what-is-init/), [`__repr__`](https://www.pythonmorsels.com/customizing-string-representation-your-objects/), and [`__eq__`](https://www.pythonmorsels.com/overloading-equality-in-python/).

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `T(a, b=3)` | `T.__init__(x, a, b=3)` | `None` |
| `repr(x)` | `x.__repr__()` | `str` |
| `x == y` | `x.__eq__(y)` | Typically `bool` |

The [`__init__`](https://www.pythonmorsels.com/what-is-init/) method is the **initializer** (not to be confused with the [constructor](#construction-and-finalizing)), the [`__repr__`](https://www.pythonmorsels.com/customizing-string-representation-your-objects/) method customizes an object's string representation, and the [`__eq__`](https://www.pythonmorsels.com/overloading-equality-in-python/) method customizes what it means for objects to be *equal* to one another.

The `__repr__` method is particularly helpful at the [the Python REPL](https://www.pythonmorsels.com/using-the-python-repl/) and when debugging.

## Equality and hashability 🟰

In addition to the `__eq__` method, Python has 2 other dunder methods for determining the "value" of an object in relation to other objects.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `x == y` | `x.__eq__(y)` | Typically `bool` |
| `x != y` | `x.__ne__(y)` | Typically `bool` |
| `hash(x)` | `x.__hash__()` | `int` |

Python's `__eq__` method typically returns `True`, `False`, or [`NotImplemented`](https://www.pythonmorsels.com/when-to-use-notimplemented/) (if objects can't be compared). The default `__eq__` implementation relies on the `is` operator, which checks for **[identity](https://www.pythonmorsels.com/equality-vs-identity/)**.

The default implementation of `__ne__` calls `__eq__` and negates any boolean return value given (or returns `NotImplemented` if `__eq__` did). This default behavior is usually "good enough", so **you'll almost never see `__ne__` implemented**.

Hashable objects can be used as keys in dictionaries or values in sets. All objects in Python are [hashable](https://www.pythonmorsels.com/what-are-hashable-objects/) by default, but if you've written a custom `__eq__` method then your objects *won't* be hashable without a custom `__hash__` method. But **the hash value of an object must never change** or [bad things will happen](https://pym.dev/p/2ysgz/) so **typically only *immutable* objects implement `__hash__`**.

For implementing equality checks, see [`__eq__` in Python](https://www.pythonmorsels.com/overloading-equality-in-python/). For implementing hashability, see [making hashable objects in Python](https://www.pythonmorsels.com/making-hashable-objects/).

## Orderability ⚖️

Python's comparison operators (`<`, `>`, `<=`, `>=`) can all be overloaded with dunder methods as well. The comparison operators also power functions that rely on the relative ordering of objects, like `sorted`, `min`, and `max`.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `<` | `__lt__` | Typically `bool` |
| `>` | `__gt__` | Typically `bool` |
| `<=` | `__le__` | Typically `bool` |
| `>=` | `__ge__` | Typically `bool` |

If you plan to implement all of these operators in the *typical* way (where `x < y` would be the same as asking `y > x`) then the [`total_ordering` decorator](https://docs.python.org/3/library/functools.html#functools.total_ordering) from Python's `functools` module will come in handy.

## Type conversions and string formatting ⚗️

Python has a number of dunder methods for converting objects to a different type.

| Function | Dunder Method Call | Returns |
| --- | --- | --- |
| `str(x)` | `x.__str__()` | `str` |
| `bool(x)` | `x.__bool__()` | `bool` |
| `int(x)` | `x.__int__()` | `int` |
| `float(x)` | `x.__float__()` | `float` |
| `bytes(x)` | `x.__bytes__()` | [`bytes`](https://pym.dev/p/2ysgz/) |
| `complex(x)` | `x.__complex__()` | [`complex`](https://docs.python.org/3/glossary.html#term-complex-number) |
| `f"{x:s}"` | `x.__format__(s)` | `str` |
| `repr(x)` | `x.__repr__()` | `str` |

The `__bool__` function is used for [truthiness](https://www.pythonmorsels.com/truthiness/) checks, though `__len__` is used as a fallback.

If you needed to make an object that acts like a number (like [`decimal.Decimal`](https://docs.python.org/3/library/decimal.html) or [`fractions.Fraction`](https://docs.python.org/3/library/fractions.html)), you'll want to implement `__int__`, `__float__`, and `__complex__` so your objects can be converted to other numbers. If you wanted to make an object that could be used in a `memoryview` or could otherwise be converted to `bytes`, you'll want a `__bytes__` method.

The `__format__` and `__repr__` methods are different string conversion flavors. Most string conversions rely the `__str__` method, but the default `__str__` implementation simply calls `__repr__`.

The `__format__` method is used by all [f-string conversions](https://www.pythonmorsels.com/string-formatting/), by the `str` class's `format` method, and by the (rarely used) built-in `format` function. This method allows `datetime` objects to [support custom format specifiers](https://www.pythonmorsels.com/string-formatting/#formatting-datetime-objects).

## Context managers 🚪

A context manager is an object that can be used in a `with` block.

| Use | Dunder Method Call | Returns |
| --- | --- | --- |
| `with` block enter | `x.__enter__()` | A value given to `as` |
| `with` block exit | `x.__exit__(exc_type, exc, traceback)` | Truthy/falsey value |

For more on context managers see, [what is a context manager](https://www.pythonmorsels.com/what-is-a-context-manager/) and [creating a context manager](https://www.pythonmorsels.com/creating-a-context-manager/).

## Containers and collections 🗃️

Collections (a.k.a. containers) are essentially data structures or objects that act like data stuctures. Lists, dictionaries, sets, strings, and tuples are all examples of collections.

| Operation | Dunder Method Call | Return Type | Implemented |
| --- | --- | --- | --- |
| `len(x)` | `x.__len__()` | integer | Very common |
| `iter(x)` | `x.__iter__()` | iterator | Very common |
| `for item in x: ...` | `x.__iter__()` | iterator | Very common |
| `x[a]` | `x.__getitem__(a)` | any object | Common |
| `x[a] = b` | `x.__setitem__(a, b)` | None | Common |
| `del x[a]` | `x.__delitem__(a)` | None | Common |
| `a in x` | `x.__contains__(a)` | bool | Common |
| `reversed(x)` | `x.__reversed__()` | iterator | Common |
| `next(x)` | `x.__next__()` | any object | Uncommon |
| `x[a]` | `x.__missing__(a)` | any object | Uncommon |
| `operator.length_hint(x)` | `x.__length_hint__()` | integer | Uncommon |

The `__iter__` method is used by the `iter` function *and* for all forms of iteration: [`for` loops](https://www.pythonmorsels.com/writing-a-for-loop/), [comprehensions](https://www.pythonmorsels.com/what-are-list-comprehensions/), [tuple unpacking](https://www.pythonmorsels.com/tuple-unpacking/), and [using `*` for iterable unpacking](https://www.pythonmorsels.com/unpacking-iterables-iterables/).

While the `__iter__` method is necessary for creating a custom iterable, the `__next__` method is necessary for creating a custom iterator (which is much less common). The `__missing__` method is only ever called by the `dict` class on itself, unless another class decides to implement `__missing__`. The `__length_hint__` method supplies a length guess for structures which do not support `__len__` so that lists or other structures can be pre-sized more efficiently.

Also see: [the iterator protocol](https://www.pythonmorsels.com/iterator-protocol/), [implementing `__len__`](https://www.pythonmorsels.com/making-the-len-function-work-on-your-python-objects/), and [implementing `__getitem__`](https://www.pythonmorsels.com/supporting-index-and-key-lookups/).

## Callability ☎️

Functions, classes, and all other [callable objects](https://www.pythonmorsels.com/callables/) rely on the `__call__` method.

| Operation | Dunder Method Call | Return Type |
| --- | --- | --- |
| `x(a, b=c)` | `x.__call__(a, b=c)` | any object |

When a class is called, its [metaclass](https://docs.python.org/3/glossary.html#term-metaclass)'s `__call__` method is used. When a class *instance* is called, the class's `__call__` method is used.

For more on callability, see [Callables: Python's "functions" are sometimes classes](https://www.pythonmorsels.com/class-function-and-callable/).

## Arithmetic operators ➗

Python's dunder methods are often described as a tool for "operator overloading". Most of this "operator overloading" comes in the form of Python's various arithmetic operators.

There are two ways to break down the arithmetic operators:

*   Mathematical (e.g. `+`, `-`, `*`, `/`, `%`) versus bitwise (e.g. `&`, `|`, `^`, `>>`, `~`)
*   Binary (between 2 values, like `x + y`) versus unary (before 1 value, like `+x`)

The mathematical operators are much more common than the bitwise ones and the binary ones are a bit more common than the unary ones.

These are the binary mathematical arithmetic operators:

| Operation | Left-Hand Method | Right-Hand Method | Description |
| --- | --- | --- | --- |
| `x + y` | `__add__` | `__radd__` | Add / Concatenate |
| `x - y` | `__sub__` | `__rsub__` | Subtract |
| `x * y` | `__mul__` | `__rmul__` | Multiply |
| `x / y` | `__truediv__` | `__rtruediv__` | Divide |
| `%` | `__mod__` | `__rmod__` | Modulo |
| `x // y` | `__floordiv__` | `__rfloordiv__` | [Integer division](https://www.pythonmorsels.com/integer-division/) |
| `**` | `__pow__` | `__rpow__` | Exponentiate |
| `x @ y` | `__matmul__` | `__rmatmul__` | Matrix multiply |

Each of these operators includes left-hand and right-hand methods. If `x.__add__(y)` returns [`NotImplemented`](https://www.pythonmorsels.com/when-to-use-notimplemented/), then `y.__radd__(x)` will be attempted. See [arithmetic dunder methods](https://www.pythonmorsels.com/arithmetic-dunder-methods/) for more.

These are the binary bitwise arithmetic operators:

| Operation | Left-Hand Method | Right-Hand Method | Description |
| --- | --- | --- | --- |
| `x & y` | `__and__` | `__rand__` | AND |
| `x &#124; y` | `__or__` | `__ror__` | OR |
| `x ^ y` | `__xor__` | `__rxor__` | XOR |
| `x >> y` | `__rshift__` | `__rrshift__` | Right-shift |
| `x << y` | `__lshift__` | `__rlshift__` | Left-shift |

These are Python's unary arithmetic operators:

| Operation | Dunder Method | Variety | Description |
| --- | --- | --- | --- |
| `-x` | `__neg__` | Mathematical | Negate |
| `+x` | `__pos__` | Bitwise | Affirm |
| `~x` | `__invert__` | Bitwise | Invert |

The unary `+` operator typically [has no effect](https://stackoverflow.com/questions/16819023/whats-the-purpose-of-the-pos-unary-operator-in-python), though some objects use it for a specific operation. For example [using `+` on `collections.Counter` objects](https://www.pythonmorsels.com/using-counter/#removing-negative-counts) will remove non-positive values.

Python's arithmetic operators are often used for non-arithmetic ends: [sequences](https://www.pythonmorsels.com/sequence/) use `+` to concatenate and `*` to self-concatenate and [sets](https://www.pythonmorsels.com/practical-uses-of-sets/) use `&` for intersection, `|` for union, `-` for asymmetric difference, and `^` for symmetric difference. Arithmetic operators are sometimes overloaded for more creative uses too. For example, `pathlib.Path` objects [use `/` to create child paths](https://docs.python.org/3/library/pathlib.html#operators).

## In-place arithmetic operations ♻️

Python includes many dunder methods for **in-place** operations. If you're making a [mutable](https://www.pythonmorsels.com/terms/#mutable) object that supports any of the arithmetic operations, you'll want to implement the related in-place dunder method(s) as well.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `x += y` | `x.__iadd__(y)` | Typically `self` |
| `x -= y` | `x.__isub__(y)` | Typically `self` |
| `x *= y` | `x.__imul__(y)` | Typically `self` |
| `x /= y` | `x.__itruediv__(y)` | Typically `self` |
| `x %= y` | `x.__imod__(y)` | Typically `self` |
| `x //= y` | `x.__ifloordiv__(y)` | Typically `self` |
| `x **= y` | `x.__ipow__(y)` | Typically `self` |
| `x @= y` | `x.__imatmul__(y)` | Typically `self` |
| `x &= y` | `x.__iand__(y)` | Typically `self` |
| `x &#124;= y` | `x.__ior__(y)` | Typically `self` |
| `x ^= y` | `x.__ixor__(y)` | Typically `self` |
| `x >>= y` | `x.__irshift__(y)` | Typically `self` |
| `x <<= y` | `x.__ilshift__(y)` | Typically `self` |

All of Python's binary arithmetic operators work in **augmented assignment statements**, which involve using an operator followed by the `=` sign to assign to an object while performing an operation on it.

Augmented assignments on **mutable objects** are [expected to mutate the original object](https://www.pythonmorsels.com/augmented-assignments-mutate/), thanks to the mutable object implementing the appropriate dunder method for in-place arithmetic.

When no dunder method is found for an in-place operation, Python performs the operation followed by an assignment. **Immutable objects typically do *not* implement dunder methods for in-place operations**, since they should return a new object instead of changing the original.

## Built-in math functions 🧮

Python also includes dunder methods for many math-related functions, both [built-in functions](https://www.pythonmorsels.com/built-in-functions-in-python/#type) and some functions in the `math` library.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `divmod(x, y)` | `x.__divmod__(y)` | 2-item tuple |
| `divmod(x, y)` | `y.__rdivmod__(x)` | 2-item tuple |
| `abs(x)` | `x.__abs__()` | `float` |
| `sequence[x]` | `x.__index__()` | `int` |
| `round(x)` | `x.__round__()` | Number |
| `math.trunc(x)` | `x.__trunc__()` | Number |
| `math.floor(x)` | `x.__floor__()` | Number |
| `math.ceil(x)` | `x.__ceil__()` | Number |

Python's `divmod` function performs [integer division](https://www.pythonmorsels.com/integer-division/) (`//`) and a modulo operation (`%`) at the same time. Note that, just like the many binary arithmetic operators, `divmod` will also check for an `__rvidmod__` method if it needs to ask the *second* argument to handle the operation.

The `__index__` method is for making integer-like objects. This method losslessly converts to an integer, unlike `__int__` which may perform a "lossy" integer conversion (e.g. from `float` to `int`). It's used by operations that require *true* integers, such as [slicing](https://www.pythonmorsels.com/slicing/), indexing, and `bin`, `hex`, and `oct` functions ([example](https://pym.dev/p/2k9zt/)).

## Attribute access 📜

Python even includes dunder methods for controlling what happens when you access, delete, or assign any attribute on an object!

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `x.missing` | `x.__getattr__("missing")` | Attribute value |
| `x.anything` | `x.__getattribute__("anything")` | Attribute value |
| `x.thing = value` | `x.__setattr__("thing", value)` | `None` |
| `del x.thing` | `x.__delattr__("thing")` | `None` |
| `dir(x)` | `x.__dir__()` | List of strings |

The `__getattribute__` method is called for *every* attribute access, while the `__getattr__` method is only called after Python *fails* to find a given attribute. All method calls and attribute accesses call `__getattribute__` so implementing it correctly is challenging (due to accidental [recursion](https://www.pythonmorsels.com/what-is-recursion/)). See [`__getattr__` versus `__getattribute__`](https://www.pythonmorsels.com/getattr-vs-getattribute/) for examples demonstrating the difference between these two methods.

The `__dir__` method should return an iterable of attribute names (as strings). When [the `dir` function](https://www.pythonmorsels.com/built-in-functions-in-python/#dir) calls `__dir__`, it converts the returned iterable into a sorted list (like [`sorted`](https://www.pythonmorsels.com/sorting-in-python/) does).

The built-in `getattr`, [`setattr`](https://www.pythonmorsels.com/python-setattr/), and `delattr` functions correspond to the dunder methods of the same name, but they're only intended for dynamic attribute access (not *all* attribute accesses).

Now we're getting into the really unusual dunder methods. Python includes many dunder methods for metaprogramming-related features.

| Implemented on | Operation | Dunder Method Call | Returns |
| --- | --- | --- | --- |
| Metaclasses | `class T: ...` | `type(base).__prepare__()` | mapping |
| Metaclasses | `isinstance(x, T)` | `T.__instancecheck__(x)` | `bool` |
| Metaclasses | `issubclass(U, T)` | `T.__subclasscheck__(U)` | `bool` |
| Any class | `class U(T): ...` | `T.__init_subclass__(U)` | `None` |
| Any class | (Called manually) | `T.__subclasses__()` | `list` |
| Any class | `class U(x): ...` | `x.__mro_entries__([x])` | `tuple` |
| Any class | `T[y]` | `T.__class_getitem__(y)` | an item |

The `__prepare__` method customizes the dictionary that's used for a class's initial namespace. This is used to pre-populate dictionary values or customize the dictionary type ([silly example](https://pym.dev/p/23wfv/)).

The `__instancecheck__` and `__subclasscheck__` methods override the functionality of `isinstance` and `issubclass`. Python's ABCs use these to practice [goose typing](https://www.pythonmorsels.com/goose-typing/) ([duck typing](https://www.pythonmorsels.com/duck-typing/) *while* type checking).

The `__init_subclass__` method allows classes to hook into subclass initialization ([example](https://pym.dev/p/246z6/)). Classes *also* have a `__subclasses__` method (on their [metaclass](https://docs.python.org/3/glossary.html#term-metaclass)) but it's not typically overridden.

Python calls `__mro_entries__` during [class inheritance](https://www.pythonmorsels.com/inheriting-one-class-another/) for any base classes that are not *actually* classes. The [`typing.NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple) function uses this to pretend it's a class ([see here](https://pym.dev/p/2qgzd/)).

The `__class_getitem__` method allows a class to be subscriptable (*without* its metaclass needing a `__getitem__` method). This is typically used for enabling fancy type annotations (e.g. `list[int]`).

## Descriptors 🏷️

[Descriptors](https://docs.python.org/3/glossary.html#term-descriptor) are objects that, when attached to a class, can hook into the access of the attribute name they're attached to on that class.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `class T: x = U()` | `T.x.__set_name__(T, 'x')` | `None` |
| `t.x` | `T.x.__get__(t, T)` | The value |
| `t.x = y` | `T.x.__set__(t, y)` | `None` |
| `del t.x` | `T.x.__delete__(t)` | `None` |

The descriptor protocol is *mostly* a feature that exists to make Python's `property` decorator work, though it is also used by a number of third-party libraries.

## Buffers 💾

Implementing a low-level memory array? You need Python's [buffer protocol](https://docs.python.org/3/reference/datamodel.html#emulating-buffer-types).

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `memoryview(x)` | `x.__buffer__(flags)` | `memoryview` |
| `del memoryview(x)` | `x.__release_buffer__(m)` | `None` |

The `__release_buffer__` method is called when the buffer that's returned from `__buffer__` is deleted.

Python's buffer protocol is typically implemented in C, since it's meant for low level objects.

## Asynchronous operations 🤹

Want to implement an asynchronous context manager? You need these dunder methods:

*   `__aenter__`: just like `__enter__`, but it returns an awaitable object
*   `__aexit__`: just like `__exit__`, but it returns an awaitable object

Need to support asynchronous iteration? You need these dunder methods:

*   `__aiter__`: must return an asynchronous iterator
*   `__anext__`: like `__next__` or non-async iterators, but this must return an awaitable object and this should raise `StopAsyncIteration` instead of `StopIteration`

Need to make your own awaitable object? You need this dunder method:

*   `__await__`: returns an iterator

I have little experience with custom asynchronous objects, so look elsewhere for more details.

## Construction and finalizing 🏭

The last few dunder methods are related to object creation and destruction.

| Operation | Dunder Method Call | Returns |
| --- | --- | --- |
| `T(a, b=3)` | `T.__new__(T, a, b=3)` | New instance (`x`) |
| `T(a, b=3)` | `T.__init__(x, a, b=3)` | `None` |
| `del x` | `x.__del__()` | `None` |

Calling a class returns a new class instance thanks to the `__new__` method. The `__new__` method is Python's **constructor method**, though unlike constructors in many programming languages, you should almost *never* define your own `__new__` method. To control object creation, prefer the initializer (`__init__`), not the constructor (`__new__`). [Here's an odd `__new__` example](https://pym.dev/p/28r9m/).

You could think of `__del__` as a "destructor" method, though it's actually called the **finalizer method**. Just before an object is deleted, its `__del__` method is called ([example](https://pym.dev/p/2hexg/)). Files implement a `__del__` method that closes the file and any binary file buffer that it may be linked to.

## Library-specific dunder methods 🧰

Some standard library modules define custom dunder methods that aren't used anywhere else:

*   [dataclasses](https://www.pythonmorsels.com/dataclasses/) support a `__post_init__` method
*   `abc.ABC` classes have a `__subclasshook__` method which `abc.ABCMeta` calls in its `__subclasscheck__` method (more in [goose typing](https://www.pythonmorsels.com/goose-typing/))
*   Path-like objects have a `__fspath__` method, which returns the file path as a string
*   Python's `copy` module will use the `__copy__` and `__deepcopy__` methods if present
*   Pickling relies on `__getnewargs_ex__` or `__getargs__`, though `__getstate__` and `__setstate__` can customize further and `__reduce__` or `__reduce_ex__` are even lower-level
*   `sys.getsizeof` relies on the `__sizeof__` method to get an object's size (in bytes)

## Dunder attributes 📇

In addition to dunder methods, Python has many non-method **dunder attributes**.

Here are some of the more common dunder attributes you'll see:

*   `__name__`: name of a function, classes, or module
*   `__module__`: module name for a function or class
*   `__doc__`: [docstring](https://www.pythonmorsels.com/docstrings/) for a function, class, or module
*   `__class__`: an object's class (call [Python's `type` function](https://www.pythonmorsels.com/built-in-functions-in-python/#type) instead)
*   `__dict__`: most objects store their attributes here (see [where are attributes stored?](https://www.pythonmorsels.com/where-are-attributes-stored/))
*   `__slots__`: classes using this are more memory efficient than classes using `__dict__`
*   `__match_args__`: classes can define a tuple noting the significance of positional attributes when the class is used in structural pattern matching (`match`-`case`)
*   `__mro__`: a class's method resolution order used when for attribute lookups and `super()` calls
*   `__bases__`: the direct parent classes of a class
*   `__file__`: the file that defined the module object (though not always present!)
*   `__wrapped__`: functions decorated with [`functools.wraps`](https://docs.python.org/3/library/functools.html#functools.wraps) use this to point to the original function
*   `__version__`: commonly used for noting the version of a package
*   `__all__`: modules can use this to customize the behavior of `from my_module import *`
*   `__debug__`: running Python with `-O` sets this to `False` and disables Python's `assert` statements

Those are only the more commonly seen dunder attributes. Here are some more:

*   Functions have `__defaults__`, `__kwdefaults__`, `__code__`, `__globals__`, and `__closure__`
*   Both functions and classes have `__qualname__`, `__annotations__`, and `__type_params__`
*   Instance methods have `__func__` and `__self__`
*   Modules may also have `__loader__`, `__package__`, `__spec__`, and `__cached__` attributes
*   Packages have a `__path__` attribute
*   Exceptions have `__traceback__`, `__notes__`, `__context__`, `__cause__`, and `__suppress_context__`
*   Descriptors use `__objclass__`
*   Metaclasses use `__classcell__`
*   Python's `weakref` module uses `__weakref__`
*   [Generic aliases](https://docs.python.org/3/library/stdtypes.html#type-annotation-types-generic-alias-union) have `__origin__`, `__args__`, `__parameters__`, and `__unpacked__`
*   The `sys` module has `__stdout__` and `__stderr__` which point to the original `stdout` and `stderr` versions

Additionally, these dunder attributes are used by various standard library modules: `__covariant__`, `__contravariant__`, `__infer_variance__`, `__bound__`, `__constraints__`. And Python includes a built-in `__import__` function which you're not supposed to use (`importlib.import_module` is preferred) and CPython has a `__builtins__` variable that points to the `builtins` module (but this is an implementation detail and `builtins` should be explicitly imported when needed instead). Also importing from the `__future__` module can enable specific Python feature flags and Python will look for a `__main__` module within packages to make them runnable as CLI scripts.

And that's just *most* of the dunder attribute names you'll find floating around in Python. 😵

## Every dunder method: a cheat sheet ⭐

This is every Python dunder method organized in categories and ordered very roughly by the **most commonly seen** methods first. Some caveats are noted below.

| Category | Operation | Dunder Method Call | Returns |
| --- | --- | --- | --- |
| Object Creation | `x = T(a, b)` | `x.__init__(a, b)` | `None` |
| Object Creation | `x = T(a, b)` | `T.__new__(T, a, b)` | New instance (`x`) |
| Finalizer | `del x` (ish) | `x.__del__()` | `None` |
| Comparisons | `x == y` | `x.__eq__(y)` | Typically `bool` |
| Comparisons | `x != y` | `x.__ne__(y)` | Typically `bool` |
| Comparisons | `x < y` | `x.__lt__(y)` | Typically `bool` |
| Comparisons | `x > y` | `x.__rt__(y)` | Typically `bool` |
| Comparisons | `x <= y` | `x.__le__(y)` | Typically `bool` |
| Comparisons | `x >= y` | `x.__ge__(y)` | Typically `bool` |
| Hashability | `hash(x)` | `x.__hash__()` | `int` |
| Conversions | `repr(x)` | `x.__repr__()` | Always `str` |
| Conversions | `str(x)` | `x.__str__()` | Always `str` |
| Conversions | `bool(x)` | `x.__bool__()` | Always `bool` |
| Conversions | `int(x)` | `x.__int__()` | Always `int` |
| Conversions | `float(x)` | `x.__float__()` | Always `float` |
| Conversions | `bytes(x)` | `x.__bytes__()` | Always `bytes` |
| Conversions | `complex(x)` | `x.__complex__()` | Always `complex` |
| Conversions | `format(x, s)` | `x.__format__(s)` | Always `str` |
| Context Managers | `with x as c:` | `x.__enter__()` | The `c` object |
| Context Managers | `with x as c:` | `x.__exit__()` | Truthy/falsey value |
| Collections | `len(x)` | `x.__len__()` | `int` |
| Collections | `iter(x)` | `x.__iter__()` | An iterator |
| Collections | `x[a]` | `x.__getitem__(a)` |  |
| Collections | `x[a] = b` | `x.__setitem__(a, b)` | `None` |
| Collections | `del x[a]` | `x.__delitem__(a)` | `None` |
| Collections | `a in x` | `x.__contains__(a)` | `bool` |
| Collections | `reversed(x)` | `x.__reversed__()` | An iterator |
| Collections | `next(x)` | `x.__next__()` | Next iterator item |
| Collections | `x[a]` | `x.__missing__(a)` |  |
| Collections |  | `x.__length_hint__()` | `int` |
| Arithmetic | `x + y` | `x.__add__(y)` |  |
| Arithmetic | `x + y` | `y.__radd__(x)` |  |
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
| Math functions | `divmod(x, y)` | `x.__divmod__(y)` | 2-item tuple |
| Math functions | `abs(x)` | `x.__abs__()` | `float` |
| Math functions |  | `x.__index__()` | `int` |
| Math functions | `round(x)` | `x.__round__()` | Number |
| Math functions | `math.trunc(x)` | `x.__trunc__()` | Number |
| Math functions | `math.floor(x)` | `x.__floor__()` | Number |
| Math functions | `math.ceil(x)` | `x.__ceil__()` | Number |
| Assignment | `x += y` | `x.__iadd__(y)` | Typically `self` |
| Assignment | `x -= y` | `x.__isub__(y)` | Typically `self` |
| Assignment | `x *= y` | `x.__imul__(y)` | Typically `self` |
| Assignment | `x /= y` | `x.__itruediv__(y)` | Typically `self` |
| Assignment | `x %= y` | `x.__imod__(y)` | Typically `self` |
| Assignment | `x //= y` | `x.__ifloordiv__(y)` | Typically `self` |
| Assignment | `x **= y` | `x.__ipow__(y)` | Typically `self` |
| Assignment | `x @= y` | `x.__imatmul__(y)` | Typically `self` |
| Assignment | `x &= y` | `x.__iand__(y)` | Typically `self` |
| Assignment | `x &#124;= y` | `x.__ior__(y)` | Typically `self` |
| Assignment | `x ^= y` | `x.__ixor__(y)` | Typically `self` |
| Assignment | `x >>= y` | `x.__irshift__(y)` | Typically `self` |
| Assignment | `x <<= y` | `x.__ilshift__(y)` | Typically `self` |
| Attributes | `x.y` | `x.__getattribute__('y')` |  |
| Attributes | `x.y` | `x.__getattr__('y')` |  |
| Attributes | `x.y = z` | `x.__setattr__('y', z)` | `None` |
| Attributes | `del x.y` | `x.__delattr__('y')` | `None` |
| Attributes | `dir(x)` | `x.__dir__()` | An iterable |
| Descriptors | `class T: x = U()` | `T.x.__set_name__(T, 'x')` | `None` |
| Descriptors | `t.x` | `T.x.__get__(t, T)` |  |
| Descriptors | `t.x = y` | `T.x.__set__(t, y)` | `None` |
| Descriptors | `del t.x` | `T.x.__delete__(t)` | `None` |
| Class stuff | `class U(T): ...` | `T.__init_subclass__(U)` | `None` |
| Class stuff | `class U(x): ...` | `x.__mro_entries__([x])` | `tuple` |
| Class stuff | `T[y]` | `T.__class_getitem__(y)` |  |
| Metaclasses | `class T: ...` | `type(base).__prepare__()` | `dict`/mapping |
| Metaclasses | `isinstance(x, T)` | `T.__instancecheck__(x)` | `bool` |
| Metaclasses | `issubclass(U, T)` | `T.__subclasscheck__(U)` | `bool` |
| Async | `await x` (ish) | `x.__await__()` | An iterator |
| Async | `async with x:` | `x.__aenter__()` | An awaitable |
| Async | `async with x:` | `x.__aexit__()` | An awaitable |
| Async | `async for a in x:` | `x.__aiter__()` | An awaitable |
| Async | `async for a in x:` | `x.__anext__()` | An awaitable |
| Buffers | `memoryview(x)` | `x.__buffer__(flags)` | `memoryview` |
| Buffers | `del memoryview(x)` | `x.__release_buffer__(m)` | `None` |

The above table has a slight but consistent *untruth*. Most of these dunder methods are not *actually* called on an object directly but are instead called on the *type* of that object: `type(x).__add__(x, y)` instead of `x.__add__(y)`. This distinction mostly matters with metaclass methods.

I've also purposely excluded library-specific dunder methods (like `__post_init__`) and dunder methods you're unlikely to ever define (like `__subclasses__`). See those below.

| Category | Operation | Dunder Method Call | Returns |
| --- | --- | --- | --- |
| Dataclasses | `x = T(a, b)` | `T.__post_init__(a, b)` | `None` |
| Copying | `copy.copy(x)` | `x.__copy__()` | New object |
| Copying | `copy.deepcopy(x)` | `x.__deepcopy__(memo)` | New object |
| [Pickling](https://docs.python.org/3/library/pickle.html#pickling-class-instances) | `pickle.dumps(x)` | `x.__getnewargs__()` | A 2-item tuple |
| Pickling | `pickle.dumps(x)` | `x.__getnewargs_ex__()` | A 2-item tuple |
| Pickling | `pickle.dumps(x)` | `x.__getstate__()` | A meaningful state |
| Pickling | `pickle.dumps(x)` | `x.__reduce__()` | A 2-6 item tuple |
| Pickling | `pickle.dumps(x)` | `x.__reduce_ex__(4)` | A 2-6 item tuple |
| Pickling | `pickle.loads(b)` | `x.__setstate__(state)` | `None` |
| pathlib | [`os.fspath(x)`](https://docs.python.org/3/library/os.html#os.fspath) | `p.__fspath__()` | `str` or `bytes` |
| sys | `sys.getsizeof(x)` | `x.__sizeof__()` | `int` (size in bytes) |
| Class stuff | None? | `x.__subclasses__()` | Subclasses iterable |
| ABCs | `issubclass(U, T)` | `T.__subclasshook__(U)` | `bool` |

So, Python includes 103 "normal" dunder methods, 12 library-specific dunder methods, and at least 52 other dunder attributes of various types. That's over 150 unique `__dunder__` names! I **do not recommend** memorizing these: let Python do its job and look up the dunder method or attribute that you need to implement/find whenever you need it.

Keep in mind that **you're not meant to invent your own dunder methods**. Sometimes you'll see third-party libraries that *do* invent their own dunder method, but this isn't encouraged and it can be quite confusing for users who run across such methods and assume they're "*real*" dunder methods.