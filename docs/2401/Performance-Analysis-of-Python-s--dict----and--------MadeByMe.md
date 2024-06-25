<!--yml

类别：未分类

日期：2024-05-27 15:00:46

-->

# Python 的 `dict()` 和 `{}` 的性能分析 - MadeByMe

> 来源：[`madebyme.today/blog/python-dict-vs-curly-brackets/`](https://madebyme.today/blog/python-dict-vs-curly-brackets/)

<main class="main">

不久前，在一次代码审查中，我和一个同事就在新的 Python 代码中更喜欢 `dict()` 而不是 `{}` 进行了讨论。他们认为 `dict()` 更易读 —— 并且更清晰地表达了意图 —— 因此应该优先使用。我对此并不完全信服，但当时我没有任何反驳的论据，所以我就过去了。

但这让我想知道：`dict` 类型和 `{}` 字面表达式之间有什么区别？

让我们深入探讨。

## # Python 的字典

首先，让我们来看看两种方法之间的性能差异。我将使用 [`timeit`](https://docs.python.org/3.12/library/timeit.html) 模块进行。

```
1$ python -m timeit "dict()" 210000000 loops, best of 5: 40 nsec per loop 3$ python -m timeit "{}" 420000000 loops, best of 5: 19.6 nsec per loop 
```

在我的 MacBook M1 上，差异几乎是**2 倍**。这是一个非常显著的差异，尤其是知道这两个表达式产生的是完全相同的对象。

这种差异来自哪里？

在我们深入讨论之前，我们需要偏离一点，谈谈当你执行 Python 代码时会发生什么。Python 是一种[解释语言](https://en.wikipedia.org/wiki/Scripting_language) —— 它需要一个解释器。[CPython](https://github.com/python/cpython/tree/v3.12.0) 是 Python 最广泛使用的解释器；它是一个参考实现，意味着其他解释器使用它来确定“正确”的行为。如果你从默认发行版安装了 Python，那么你的机器上安装了 CPython。

有趣的是，CPython 既是编译器又是解释器。在执行 Python 代码时，它首先将其编译成[字节码](https://docs.python.org/3.12/library/dis.html#python-bytecode-instructions)指令，然后解释它们。

为了更好地理解 `dist()` 和 `{}` 之间的性能差异，让我们比较一下它们编译成的字节码指令。Python 有一个内置模块叫做[dis](https://docs.python.org/3.12/library/dis.html)，它正是做这件事情的。

```
 1>>> import dis 2>>> def a(): 3...   return dict() 4... 5>>> def b(): 6...   return {} 7... 8>>> dis.dis(a) 9 1           0 RESUME                   0 10 11 2           2 LOAD_GLOBAL              1 (NULL + dict) 12 12 CALL                     0 13 20 RETURN_VALUE 14>>> dis.dis(b) 15 1           0 RESUME                   0 16 17 2           2 BUILD_MAP                0 18 4 RETURN_VALUE 
```

尽管它们产生相同的最终结果，但这两个表达式显然执行不同的代码。

### # 字节码指令

`dis.dis` 的输出并不难理解。

```
1 (1) |    (2)    |          (3)          | (4) |      (5) 2-----|-----------|-----------------------|-----|--------------- 3 1 |         0 | RESUME                | 0   | 4 |           |                       |     | 5 2 |         2 | LOAD_GLOBAL           | 1   | (NULL + dict) 6 |        12 | CALL                  | 0   | 7 |        20 | RETURN_VALUE          |     | 
```

每一列都有一个目的：

1.  源代码中的行号。

1.  指令的地址 — 编译后字节码中的字节索引。

1.  字节码名称（opcode）。

1.  操作参数。

1.  操作参数的人类可读解释。

好的，有了这些知识和通过交叉引用[操作码集](https://docs.python.org/3.12/library/dis.html#python-bytecode-instructions)我们知道：

1.  [`RESUME`](https://docs.python.org/3.12/library/dis.html#opcode-RESUME) — 什么都不做。它执行内部跟踪、调试和优化检查。

1.  [`LOAD_GLOBAL`](https://docs.python.org/3.12/library/dis.html#opcode-LOAD_GLOBAL) — 将全局变量加载到堆栈上。从可读的操作码参数解释中，我们知道它加载了`dict`（暂时忽略`NULL`）。

1.  [`CALL`](https://docs.python.org/3.12/library/dis.html#opcode-CALL) — 使用由`argc`指定的参数数量调用可调用对象——在我们的情况下是零。

1.  [`RETURN_VALUE`](https://docs.python.org/3.12/library/dis.html#opcode-RETURN_VALUE) — 将堆栈上的最后一个元素返回给调用者。

编译`return {}`会产生一个额外的操作码：

1.  [`BUILD_MAP`](https://docs.python.org/3.12/library/dis.html#opcode-BUILD_MAP) — 将一个新的字典对象推送到堆栈上。弹出`2 * count`项，以便字典包含 count 个条目。

我们已经有了两种情况之间的几个明显区别。`{}`表达式直接构建字典，而`dict()`将其委托给一个可调用对象。在这之前，它首先需要将全局值（`dict`）加载到堆栈上——实际上每次我们调用该函数都会这样做。

为什么？

因为`dict`不是常量：它可以被重写——因此——产生不同的值。

```
1>>> def a(): 2...   return dict() 3... 4>>> dict = lambda: 42 5>>> 6>>> assert a() == 42 
```

可能会发生。这就是为什么 Python 需要为每次函数调用生成加载和调用可调用对象的开销。

好吧，这听起来非常整洁。调用可调用对象确实会产生额外的开销，合理地假设我们在本文开头看到的差异是这种开销的结果。但是，我们确定了`dict()`内部是否调用了与`{}`相同的代码吗？`dict`是一个类，幸运的是，[`dis.dis`](https://docs.python.org/3/library/dis.html#dis.dis)函数可以将类编译成字节码。

###### *示例*

```
 *`1import dis 2 3class Foo: 4 def bar(self): 5 return 42 6 7 def __str__(self): 8 return self.__class__.__name__ 9 10dis.dis(Foo)`* 
```

*它打印出所有类方法的反汇编内容：*

```
 *`1Disassembly of __str__: 2 8           0 RESUME                   0 3 4 9           2 LOAD_FAST                0 (self) 5 4 LOAD_ATTR                0 (__class__) 6 24 LOAD_ATTR                2 (__name__) 7 44 RETURN_VALUE 8 9Disassembly of bar: 10 5           0 RESUME                   0 11 12 6           2 RETURN_CONST             1 (42)`* 
```

*让我们尝试对`dict`这样做：*

*&mldr; 什么也没打印？为什么？*

*嗯，[dis](https://docs.python.org/3.12/library/dis.html)模块对于内部类型并不是很有用，`dict`就是其中之一——在解释器源代码中定义的类型。*

### *# 在 CPython 源代码中迷失*

*要进入禁止的魔法，我们首先需要克隆[CPython 存储库](https://github.com/python/cpython/tree/main)。我们不需要所有的历史记录，所以让我们使用[`--depth 1`](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt---depthltdepthgt)。*

```
*`1git clone --depth 1 --branch v3.12.0 https://github.com/python/cpython.git 2# or 3git clone --depth 1 --branch v3.12.0 git@github.com:python/cpython.git`* 
```

*然后，我们需要找到我们实际正在寻找的东西——一个解释操作码指令的地方。喝一杯咖啡并且进行大量 grep 之后，我们找到了[`Python/bytecodes.c`](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c)文件。它由一个单独的`switch`语句组成，似乎所有的字节码指令都在那里解释。*

#### *# `dict`类型*

*首先让我们处理 `dict`。所有内部类型都定义为 `PyTypeObject` 对象，而 `dict` 类型在 [`dictobject.c`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L3839) 文件中定义。它有许多已定义的属性，但我们只关注两个：*

```
 *`1PyTypeObject PyDict_Type = { 2 PyVarObject_HEAD_INIT(&PyType_Type, 0) 3 "dict", 4 sizeof(PyDictObject), 5 // ... 6 dict_init,                                  /* tp_init */ 7 // ... 8 dict_new,                                   /* tp_new */ 9 // ... 10};`* 
```

*这对 (`dict_new` 和 `dict_init`) 将告诉我们当有人创建一个新字典时会发生什么。*

###### **注意**

**在 Python 中，构造函数被称为 `__new__`。它是一个静态方法，返回其类型的新对象。**

**初始化方法称为 `__init__`。它接收一个新创建的对象并初始化其属性。`__init__` 方法在 `__new__` 方法之后调用。**

**`dict_new` 函数在这里定义：[dictobject.c#L3751](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L3751)。**

```
 **`1static PyObject * 2dict_new(PyTypeObject*type, PyObject *args, PyObject*kwds) 3{ 4 assert(type != NULL); 5 assert(type->tp_alloc != NULL); 6 // dict subclasses must implement the GC protocol 7 assert(_PyType_IS_GC(type)); 8 9 PyObject *self = type->tp_alloc(type, 0); 10 if (self == NULL) { 11 return NULL; 12 } 13 PyDictObject *d = (PyDictObject *)self; 14 15 d->ma_used = 0; 16 d->ma_version_tag = DICT_NEXT_VERSION( 17 _PyInterpreterState_GET()); 18 dictkeys_incref(Py_EMPTY_KEYS); 19 d->ma_keys = Py_EMPTY_KEYS; 20 d->ma_values = NULL; 21 ASSERT_CONSISTENT(d); 22 23 // ... 24  25 return self; 26}`** 
```

**我们看到，首先它通过提供的分配器分配一个新对象（第 9 行），然后设置一些内部字段（第 15 行、第 19 行和第 20 行）。我们对此感兴趣，它 **不会** 预先分配字典的内部内存（见标记的行）。这乍一看可能很奇怪，但请记住：对象的初始化 —— 像内存预分配 —— 不是 `__new__` 方法的责任，这就是 `__init__` 方法的责任。**

**有了新的对象在内存中，`dict_init` 函数可以向其中插入条目。插入逻辑被委托给 `dict_update_common` 函数，可以在这里找到：[dictobject.c#L2678](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2678)。**

```
 **`1static int 2dict_update_common(PyObject *self, PyObject *args, PyObject *kwds, 3 const char *methname) 4{ 5 PyObject *arg = NULL; 6 int result = 0; 7 8 if (!PyArg_UnpackTuple(args, methname, 0, 1, &arg)) { 9 result = -1; 10 } 11 else if (arg != NULL) { 12 result = dict_update_arg(self, arg); 13 } 14 15 if (result == 0 && kwds != NULL) { 16 if (PyArg_ValidateKeywordArguments(kwds)) 17 result = PyDict_Merge(self, kwds, 1); 18 else 19 result = -1; 20 } 21 return result; 22}`** 
```

**它从 args 和 kwargs 更新字典。**

##### **# args 和 `dict_update_arg`**

**对于 `args`，它调用了 `dict_update_arg` 函数。**

```
 **`1static int 2dict_update_arg(PyObject *self, PyObject *arg) 3{ 4 if (PyDict_CheckExact(arg)) { 5 return PyDict_Merge(self, arg, 1); 6 } 7 PyObject *func; 8 if (_PyObject_LookupAttr(arg, &_Py_ID(keys), &func) < 0) { 9 return -1; 10 } 11 if (func != NULL) { 12 Py_DECREF(func); 13 return PyDict_Merge(self, arg, 1); 14 } 15 return PyDict_MergeFromSeq2(self, arg, 1); 16}`** 
```

**该函数检查 `arg` 是否为字典，如果是，则合并两个（[`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988)），否则，它从一系列的键值对中添加新条目（[`PyDict_MergeFromSeq2`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2722)）。**

##### **# kwargs 和 `PyDict_Merge`**

**`kwargs` 路径以相同的地方结束 —— [`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988)。让我们来看看。**

**内部，它将所有逻辑委托给 [`dict_merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2807) 函数。**

```
 **`1static int 2dict_merge(PyInterpreterState *interp, PyObject *a, PyObject *b, int override) 3{ 4 PyDictObject *mp, *other; 5 6 assert(0 <= override && override <= 2); 7 8 /* We accept for the argument either a concrete dictionary object, 9 * or an abstract "mapping" object.  For the former, we can do 10 * things quite efficiently.  For the latter, we only require that 11 * PyMapping_Keys() and PyObject_GetItem() be supported. 12 */ 13 if (a == NULL || !PyDict_Check(a) || b == NULL) { 14 PyErr_BadInternalCall(); 15 return -1; 16 } 17 mp = (PyDictObject*)a; 18 if (PyDict_Check(b) && (Py_TYPE(b)->tp_iter == (getiterfunc)dict_iter)) { 19 20 // ... 21 else { 22 /* Do it the generic, slower way */ 23 24 // ... 25 } 26 ASSERT_CONSISTENT(a); 27 return 0; 28}`** 
```

**从注释中，我们知道该函数已被优化为以另一个字典作为参数进行调用。如果参数不是字典，则它是一个通用的 [`Mapping`](https://docs.python.org/3.12/library/collections.abc.html#collections.abc.Mapping) —— 一个支持任意键查找的容器对象。**

#### **# `{}` 表达式**

**字面表达式应该更容易理解。让我们回到[bytecode.c](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c)文件，找到`BUILD_MAP`操作码的映射。**

```
 **`1inst(BUILD_MAP, (values[oparg*2] -- map)) { 2 map = _PyDict_FromItems( 3 values, 2, 4 values+1, 2, 5 oparg); 6 if (map == NULL) 7 goto error; 8 9 DECREF_INPUTS(); 10 ERROR_IF(map == NULL, error); 11}`** 
```

***查看源码：[bytecode.c#L1541](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c#L1541)。***

**我们看到字典是由[`_PyDict_FromItems`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L1616)完全构建并返回的。让我们去那里看看。**

```
 **`1PyObject * 2_PyDict_FromItems(PyObject *const *keys, Py_ssize_t keys_offset, 3 PyObject *const *values, Py_ssize_t values_offset, 4 Py_ssize_t length) 5{ 6 bool unicode = true; 7 PyObject *const *ks = keys; 8 PyInterpreterState *interp = _PyInterpreterState_GET(); 9 10 for (Py_ssize_t i = 0; i < length; i++) { 11 if (!PyUnicode_CheckExact(*ks)) { 12 unicode = false; 13 break; 14 } 15 ks += keys_offset; 16 } 17 18 PyObject *dict = dict_new_presized(interp, length, unicode); 19 if (dict == NULL) { 20 return NULL; 21 } 22 23 ks = keys; 24 PyObject *const *vs = values; 25 26 for (Py_ssize_t i = 0; i < length; i++) { 27 PyObject *key = *ks; 28 PyObject *value = *vs; 29 if (PyDict_SetItem(dict, key, value) < 0) { 30 Py_DECREF(dict); 31 return NULL; 32 } 33 ks += keys_offset; 34 vs += values_offset; 35 } 36 37 return dict; 38}`** 
```

**我标记了最有趣的一行：与`dict_new`相反，它**预先分配**了字典，因此它已经为所有条目分配了容量。之后，它逐个插入键值对。**

## **# 结论**

**总结一下。**

**当你执行`dict(a=1, b=2)`时，Python 需要：**

+   **分配一个新的`PyObject`，**

+   **通过`__new__`方法构造一个`dict`，**

+   **调用其`__init__`方法，该方法在内部调用[`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988)，**

+   **因为`kwargs`不是`dict`，所以[`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988)需要使用更慢的方法，逐个插入条目。**

**而使用`{}`会导致 Python：**

+   **构造一个新的**预先分配**字典，**

+   **逐个插入条目。**

**公平地说，除非你在循环中构造字典，否则我认为从`dict`切换到`{}`在性能上并没有太多好处。在阅读完这篇文章后，有一件事我想让你记住：**

###### ***太长不看***

***`{}`始终比`dict`快。***

## ***# 附录***

***类似的分析也可以用于列表、集合和（可能的）元组。然而，这篇文章已经太长了，所以我将放弃我收集的所有其他资源。***

### ***# 列表***

```
***`1$ python -m timeit "list((1, 2, 'a'))" 25000000 loops, best of 5: 53.4 nsec per loop 3$ python -m timeit "[1, 2, 'a']" 410000000 loops, best of 5: 29.7 nsec per loop`*** 
```

#### ***# `list`类型***

***Python 的`list`类型在`listobject.c`文件中定义：***

```
***`1PyTypeObject PyList_Type = { 2 // ... 3 (initproc)list___init__, /* tp_init */ 4 PyType_GenericAlloc, /* tp_alloc */ 5 PyType_GenericNew, /* tp_new */ 6 // ... 7}`*** 
```

***`PyType_GenericNew`什么也不做 —— 它忽略了`args`和`kwargs`，并返回由`type->typ_alloc`分配的对象。来源：[`github.com/python/cpython/blob/main/Objects/typeobject.c#L1768`](https://github.com/python/cpython/blob/main/Objects/typeobject.c#L1768)***

***`PyType_GenericAlloc`分配一个新对象，并（如果它是可垃圾收集的）标记为“待收集”。***

***`list___init__`是对`list___init___impl`函数的方便封装，它执行实际的初始化。它执行一些基本的预处理步骤：***

+   ***如果传递了任何`kwarg`，则失败，***

+   ***如果`args`有多于 1 个元素，则失败，***

+   ***将`args`的第一个参数转换为可迭代对象。***

***`list___init___impl`清除列表（如果不为空），并通过提供的可迭代对象扩展它（通过调用`list_extend`函数）。***

###### ****问题****

****我认为两者:****

```
****`1list((1, 2, 3)) 2 3_tmp = list() 4_tmp.extend((1, 2, 3))`**** 
```

****在所有方面都是等价的（但生成的操作码序列不同），并且调用相同的底层 C 代码。****

#### ****# `[]`文字表达式****

****操作码`BUILD_LIST`使用`_PyList_FromArraySteal`函数构造一个列表对象并返回它****

###### *****注意*****

****自 3.12 版开始使用`_PyList_FromArraySteal`。参见[3.12 更新日志](https://docs.python.org/3.12/whatsnew/changelog.html)和[gh-100146](https://github.com/python/cpython/pull/100147)。在 3.12 之前，操作码重复调用`POP()`从堆栈中获取项目，然后将它们插入列表中。****

```
****`1while (--oparg >= 0) { 2 PyObject *item = POP(); 3 PyList_SET_ITEM(list, oparg, item); 4}`**** 
```

****参见[gh-100146 PR 的更改](https://github.com/python/cpython/pull/100147/files#diff-729a985b0cb8b431cb291f1edb561bbbfea22e3f8c262451cd83328a0936a342L1447-L1450)****

****`_PyList_FromArraySteal`助手函数创建一个空列表（如果没有项目），或在插入项目之前预先分配列表。****

```
****`1PyObject **dst = list->ob_item; 2memcpy(dst, src, n * sizeof(PyObject *));`**** 
```

### ****# 集合****

###### *****警告*****

*****据我所知，没有办法用文字表达构造一个新的集合，因此以下分析是针对包含两个元素的集合进行的：`1`和`2`*****

```
*****`1$ python -m timeit "set((1, 2))" 25000000 loops, best of 5: 82.6 nsec per loop 3$ python -m timeit "{1, 2}" 45000000 loops, best of 5: 45.5 nsec per loop`***** 
```

#### *****# `set`类型*****

*****Python 的`set`类型在`setobject.c`文件中定义：*****

```
*****`1PyTypeObject PySet_Type = { 2 // ... 3 (initproc)set_init, /* tp_init */ 4 PyType_GenericAlloc, /* tp_alloc */ 5 set_new, /* tp_new */ 6 // ... 7}`***** 
```

*****`set_new`使用分配的空间创建一个新的空集合对象。在内部，它调用`make_new_set`函数*****

*****`set_init`执行一些预先步骤：*****

+   *****如果传递了任何`kwarg`，则失败*****

+   *****如果`args`有多于 1 个元素，则失败*****

+   *****将`args`的第一个参数转换为可迭代的*****

+   *****如果集合对象已经被填充，则清除它*****

*****...并调用`set_update_internal`函数，将值插入到集合对象中*****

#### *****# `{1, 2}`文字表达式*****

*****操作码处理程序没有专用的帮助程序。它构造一个新的集合（空的），然后迭代堆栈中的所有项目，逐个插入它们。*****

```
 *****`1inst(BUILD_SET, (values[oparg] -- set)) { 2 set = PySet_New(NULL); 3 if (set == NULL) 4 GOTO_ERROR(error); 5 int err = 0; 6 for (int i = 0; i < oparg; i++) { 7 PyObject *item = values[i]; 8 if (err == 0) 9 err = PySet_Add(set, item); 10 Py_DECREF(item); 11 } 12 if (err != 0) { 13 Py_DECREF(set); 14 ERROR_IF(true, error); 15 } 16}`***** 
```

*****来源：[`github.com/python/cpython/blob/main/Python/bytecodes.c#L1627`](https://github.com/python/cpython/blob/main/Python/bytecodes.c#L1627)*****

###### ******警告******

*****警告：如果我们在文字表达式中添加另一个参数`{1, 2, 3}`，生成的字节码就会不同。然后它变成了：*****

```
*****`1 2 BUILD_SET                0 2 4 LOAD_CONST               1 (frozenset({1, 2, 3})) 3 6 SET_UPDATE               1 4 8 RETURN_VALUE`***** 
```

### *****# 元组*****

*****这个有点棘手。使用`tuple`类型构造一个元组需要一个可迭代的。获得一个可迭代的最简单的方法是创建一个元组文字表达式，这显然比一开始就使用元组文字表达式语法不高效。*****

```
 *****`1>>> import sys 2>>> sys.version_info 3sys.version_info(major=3, minor=12, micro=1, releaselevel='final', serial=0) 4>>> 5>>> def a(): 6...   return tuple((1, 2, [])) 7>>> 8>>> def b(): 9...   return (1, 2, []) 10>>> 11>>> import dis 12>>> 13>>> dis.dis(a) 14 1           0 RESUME                   0 15 16 2           2 LOAD_GLOBAL              1 (NULL + tuple) 17 12 LOAD_CONST               1 (1) 18 14 LOAD_CONST               2 (2) 19 16 BUILD_LIST               0 20 18 BUILD_TUPLE              3 21 20 CALL                     1 22 28 RETURN_VALUE 23>>> dis.dis(b) 24 1           0 RESUME                   0 25 26 2           2 LOAD_CONST               1 (1) 27 4 LOAD_CONST               2 (2) 28 6 BUILD_LIST               0 29 8 BUILD_TUPLE              3 30 10 RETURN_VALUE`***** 
```

*****在文章中包含此示例可能没有太多意义*****

#### *****# `tuple`类型*****

*****Python 的`tuple`类型在`tupleobject.c`中定义。有趣的是，它没有`tp_alloc`，也没有`tp_init`*****

```
*****`1PyTypeObject PyTuple_Type = { 2 // ... 3 0, /* tp_init */ 4 0, /* tp_alloc */ 5 tuple_new, /* tp_new */ 6 // ... 7}`***** 
```

*****`tuple_new`函数负责分配一个新的元组对象，并传递参数作为项目*****

</main>
