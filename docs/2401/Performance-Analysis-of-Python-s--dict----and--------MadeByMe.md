<!--yml
category: 未分类
date: 2024-05-27 15:00:46
-->

# Performance Analysis of Python's `dict()` and `{}` - MadeByMe

> 来源：[https://madebyme.today/blog/python-dict-vs-curly-brackets/](https://madebyme.today/blog/python-dict-vs-curly-brackets/)

<main class="main">

Some time ago, during a code review, I had a discussion with a colleague of mine about preferring `dict()` over `{}` in new Python code. They argued that `dict()` is more readable — and expresses intent more clearly — therefore should be preferred. I wasn’t convinced by that, but at that time I didn’t have any counterarguments, so I passed.

Yet that made me wonder: what’s the difference between the `dict` type and `{}` literal expression?

Let’s go down the rabbit hole.

## [#](#pythons-dictionaries) Python’s dictionaries

First, let’s check what’s the performance difference between the two methods. I’ll use the [`timeit`](https://docs.python.org/3.12/library/timeit.html) module for that.

```
[1](#hl-0-1)$ python -m timeit "dict()" [2](#hl-0-2)10000000 loops, best of 5: 40 nsec per loop [3](#hl-0-3)$ python -m timeit "{}" [4](#hl-0-4)20000000 loops, best of 5: 19.6 nsec per loop 
```

On my MacBook M1 the difference is **almost 2x**. A non-trivial difference, especially knowing that both of these expressions produce the exact same object.

Where does the difference come from?

Before we get into that, we need to sidetrack a little bit and talk about what happens when you execute Python code. Python is an [interpreted language](https://en.wikipedia.org/wiki/Scripting_language) — it needs to have an interpreter. [CPython](https://github.com/python/cpython/tree/v3.12.0) is the most wildly used interpreter of Python; it’s a reference implementation, meaning that other interpreters use it to determine the “correct” behavior. If you’ve installed Python from a default distribution, you have CPython installed on your machine.

Interestingly, CPython is both a compiler and an interpreter. When executing Python code it first compiles it into [bytecode](https://docs.python.org/3.12/library/dis.html#python-bytecode-instructions) instructions and interprets them.

To better understand the performance difference between `dist()` and `{}` let’s compare the bytecode instructions they compile into. Python has a built-in module called [dis](https://docs.python.org/3.12/library/dis.html) which does exactly that.

```
 [1](#hl-1-1)>>> import dis [2](#hl-1-2)>>> def a(): [3](#hl-1-3)...   return dict() [4](#hl-1-4)... [5](#hl-1-5)>>> def b(): [6](#hl-1-6)...   return {} [7](#hl-1-7)... [8](#hl-1-8)>>> dis.dis(a) [9](#hl-1-9) 1           0 RESUME                   0 [10](#hl-1-10) [11](#hl-1-11) 2           2 LOAD_GLOBAL              1 (NULL + dict) [12](#hl-1-12) 12 CALL                     0 [13](#hl-1-13) 20 RETURN_VALUE [14](#hl-1-14)>>> dis.dis(b) [15](#hl-1-15) 1           0 RESUME                   0 [16](#hl-1-16) [17](#hl-1-17) 2           2 BUILD_MAP                0 [18](#hl-1-18) 4 RETURN_VALUE 
```

Although they produce the same end result these two expression, quite clearly, execute different code.

### [#](#bytecode-instructions) Bytecode instructions

The output of `dis.dis` isn’t very hard to understand.

```
[1](#hl-2-1) (1) |    (2)    |          (3)          | (4) |      (5) [2](#hl-2-2)-----|-----------|-----------------------|-----|--------------- [3](#hl-2-3) 1 |         0 | RESUME                | 0   | [4](#hl-2-4) |           |                       |     | [5](#hl-2-5) 2 |         2 | LOAD_GLOBAL           | 1   | (NULL + dict) [6](#hl-2-6) |        12 | CALL                  | 0   | [7](#hl-2-7) |        20 | RETURN_VALUE          |     | 
```

Each column has a purpose:

1.  Line number in the source code.
2.  The address of the instruction — byte index in the compiled bytecode.
3.  The bytecode name (opcode).
4.  Operation parameters.
5.  Human-readable interpretation of the operation parameters.

Alright, equipped with this knowledge and by cross-referencing the [opcode set](https://docs.python.org/3.12/library/dis.html#python-bytecode-instructions) we know that:

1.  [`RESUME`](https://docs.python.org/3.12/library/dis.html#opcode-RESUME) — does nothing. It performs internal tracking, debugging and optimization checks.
2.  [`LOAD_GLOBAL`](https://docs.python.org/3.12/library/dis.html#opcode-LOAD_GLOBAL) — loads a global variable onto the stack. From the human-readable interpretation of the opcode parameters we know that it loads `dict` (ignore `NULL` for now).
3.  [`CALL`](https://docs.python.org/3.12/library/dis.html#opcode-CALL) — calls a callable object with the number of arguments specified by `argc` — in our case it’s zero.
4.  [`RETURN_VALUE`](https://docs.python.org/3.12/library/dis.html#opcode-RETURN_VALUE) — returns with the last element from the stack to the caller.

Compiling `return {}` yields one more opcode:

5.  [`BUILD_MAP`](https://docs.python.org/3.12/library/dis.html#opcode-BUILD_MAP) — pushes a new dictionary object onto the stack. Pops `2 * count` items so that the dictionary holds count entries.

Already we have a few obvious differences between the two cases. The `{}` expression builds a dictionary directly, while `dict()` delegates that to a callable object. Before that can even happen it first needs to load the global value (`dict`) onto the stack — it actually does that every single time we call that function.

Why?

Because `dict` is not constant: it can be overwritten — and therefore — produce a different value.

```
[1](#hl-3-1)>>> def a(): [2](#hl-3-2)...   return dict() [3](#hl-3-3)... [4](#hl-3-4)>>> dict = lambda: 42 [5](#hl-3-5)>>> [6](#hl-3-6)>>> assert a() == 42 
```

It can happen. This is why Python needs to generate this overhead of loading and calling a callable for every call of the function.

OK, it all sound very neat. Calling a callable does create an overhead and it’s reasonable to assume that the difference we saw at the beginning of this post is a result of this overhead. However, are we sure that `dict()` internally calls the same code, as `{}` does? `dict` is a class and, luckily, the [`dis.dis`](https://docs.python.org/3/library/dis.html#dis.dis) function can compile classes to bytecode.

###### *Example*

```
 *`[1](#hl-0-1)import dis [2](#hl-0-2) [3](#hl-0-3)class Foo: [4](#hl-0-4) def bar(self): [5](#hl-0-5) return 42 [6](#hl-0-6) [7](#hl-0-7) def __str__(self): [8](#hl-0-8) return self.__class__.__name__ [9](#hl-0-9) [10](#hl-0-10)dis.dis(Foo)`* 
```

*It prints out disassembly for all of the classes’s methods:*

```
 *`[1](#hl-1-1)Disassembly of __str__: [2](#hl-1-2) 8           0 RESUME                   0 [3](#hl-1-3) [4](#hl-1-4) 9           2 LOAD_FAST                0 (self) [5](#hl-1-5) 4 LOAD_ATTR                0 (__class__) [6](#hl-1-6) 24 LOAD_ATTR                2 (__name__) [7](#hl-1-7) 44 RETURN_VALUE [8](#hl-1-8) [9](#hl-1-9)Disassembly of bar: [10](#hl-1-10) 5           0 RESUME                   0 [11](#hl-1-11) [12](#hl-1-12) 6           2 RETURN_CONST             1 (42)`* 
```

*Let’s try this for `dict`:*

*&mldr; it doesn’t print anything? Why?*

*Well, the [dis](https://docs.python.org/3.12/library/dis.html) module isn’t very useful for internal types and `dict` is one of those types — defined within the interpreter’s source code.*

### *[#](#getting-lost-in-the-cpython-source-code) Getting lost in the CPython source code*

*To tap into the forbidden magic, we need to clone the [CPython repository](https://github.com/python/cpython/tree/main) first. We don’t need all the history, so let’s [`--depth 1`](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt---depthltdepthgt) this.*

```
*`[1](#hl-5-1)git clone --depth 1 --branch v3.12.0 https://github.com/python/cpython.git [2](#hl-5-2)# or [3](#hl-5-3)git clone --depth 1 --branch v3.12.0 git@github.com:python/cpython.git`* 
```

*Then we need to find what we’re actually looking for — a place where opcode instructions are interpreted. After a cup of coffee and a lot of greps later we find the [`Python/bytecodes.c`](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c) file. It consists of a single `switch` statement and it seems like all bytecode instruction are interpreted there.*

#### *[#](#the-dict-type) The `dict` type*

*Let’s tackle `dict` first. All internal types are defined as `PyTypeObject` objects and the `dict` type is defined in the [`dictobject.c`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L3839) file. It has a lot of attributes defined, but only two are of interest for us:*

```
 *`[1](#hl-6-1)PyTypeObject PyDict_Type = { [2](#hl-6-2) PyVarObject_HEAD_INIT(&PyType_Type, 0) [3](#hl-6-3) "dict", [4](#hl-6-4) sizeof(PyDictObject), [5](#hl-6-5) // ... [6](#hl-6-6) dict_init,                                  /* tp_init */ [7](#hl-6-7) // ... [8](#hl-6-8) dict_new,                                   /* tp_new */ [9](#hl-6-9) // ... [10](#hl-6-10)};`* 
```

*The pair (`dict_new` and `dict_init`) will tell us what happens when someone creates a new dictionary.*

###### **Note**

**The constructor in Python is called `__new__`. It’s a static method that returns a new object of its type.**

**The initializer method is called `__init__`. It takes a newly created object and initializes its attributes. The `__init__`method is called after the `__new__` method.**

**The `dict_new` function is defined here: [dictobject.c#L3751](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L3751).**

```
 **`[1](#hl-7-1)static PyObject * [2](#hl-7-2)dict_new(PyTypeObject*type, PyObject *args, PyObject*kwds) [3](#hl-7-3){ [4](#hl-7-4) assert(type != NULL); [5](#hl-7-5) assert(type->tp_alloc != NULL); [6](#hl-7-6) // dict subclasses must implement the GC protocol [7](#hl-7-7) assert(_PyType_IS_GC(type)); [8](#hl-7-8) [9](#hl-7-9) PyObject *self = type->tp_alloc(type, 0); [10](#hl-7-10) if (self == NULL) { [11](#hl-7-11) return NULL; [12](#hl-7-12) } [13](#hl-7-13) PyDictObject *d = (PyDictObject *)self; [14](#hl-7-14) [15](#hl-7-15) d->ma_used = 0; [16](#hl-7-16) d->ma_version_tag = DICT_NEXT_VERSION( [17](#hl-7-17) _PyInterpreterState_GET()); [18](#hl-7-18) dictkeys_incref(Py_EMPTY_KEYS); [19](#hl-7-19) d->ma_keys = Py_EMPTY_KEYS; [20](#hl-7-20) d->ma_values = NULL; [21](#hl-7-21) ASSERT_CONSISTENT(d); [22](#hl-7-22) [23](#hl-7-23) // ... [24](#hl-7-24)  [25](#hl-7-25) return self; [26](#hl-7-26)}`** 
```

**We see that first, it allocates a new object via the provided allocator (9^(th) line) and then sets some of its internal fields (15^(th), 19^(th), and 20^(th)). Interestingly for us, it **does not** pre-allocate the dictionary’s internal memory (see marked lines). It might seems strange at first, but please remember: an object’s initialization — like memory pre-allocation — is not a responsibility of the `__new__` method, that what the `__init__` method is for.**

**With the new object in memory, the `dict_init` function can insert entries to it. The insertion logic is delegated to the `dict_update_common` function, which can be found here: [dictobject.c#L2678](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2678).**

```
 **`[1](#hl-8-1)static int [2](#hl-8-2)dict_update_common(PyObject *self, PyObject *args, PyObject *kwds, [3](#hl-8-3) const char *methname) [4](#hl-8-4){ [5](#hl-8-5) PyObject *arg = NULL; [6](#hl-8-6) int result = 0; [7](#hl-8-7) [8](#hl-8-8) if (!PyArg_UnpackTuple(args, methname, 0, 1, &arg)) { [9](#hl-8-9) result = -1; [10](#hl-8-10) } [11](#hl-8-11) else if (arg != NULL) { [12](#hl-8-12) result = dict_update_arg(self, arg); [13](#hl-8-13) } [14](#hl-8-14) [15](#hl-8-15) if (result == 0 && kwds != NULL) { [16](#hl-8-16) if (PyArg_ValidateKeywordArguments(kwds)) [17](#hl-8-17) result = PyDict_Merge(self, kwds, 1); [18](#hl-8-18) else [19](#hl-8-19) result = -1; [20](#hl-8-20) } [21](#hl-8-21) return result; [22](#hl-8-22)}`** 
```

**It updates the dictionary from both args and kwargs.**

##### **[#](#args--dict_update_arg) args & `dict_update_arg`**

**For `args` it calls the `dict_update_arg` function.**

```
 **`[1](#hl-9-1)static int [2](#hl-9-2)dict_update_arg(PyObject *self, PyObject *arg) [3](#hl-9-3){ [4](#hl-9-4) if (PyDict_CheckExact(arg)) { [5](#hl-9-5) return PyDict_Merge(self, arg, 1); [6](#hl-9-6) } [7](#hl-9-7) PyObject *func; [8](#hl-9-8) if (_PyObject_LookupAttr(arg, &_Py_ID(keys), &func) < 0) { [9](#hl-9-9) return -1; [10](#hl-9-10) } [11](#hl-9-11) if (func != NULL) { [12](#hl-9-12) Py_DECREF(func); [13](#hl-9-13) return PyDict_Merge(self, arg, 1); [14](#hl-9-14) } [15](#hl-9-15) return PyDict_MergeFromSeq2(self, arg, 1); [16](#hl-9-16)}`** 
```

**The function checks if `arg` is a dictionary, if so it merges the two ([`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988)), otherwise it adds new entries from a sequence of pairs ([`PyDict_MergeFromSeq2`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2722)).**

##### **[#](#kwargs--pydict_merge) kwargs & `PyDict_Merge`**

**The `kwargs` path ends in the same place — [`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988). Let’s take a look.**

**Internally, it delegates all logic to the [`dict_merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2807) function.**

```
 **`[1](#hl-10-1)static int [2](#hl-10-2)dict_merge(PyInterpreterState *interp, PyObject *a, PyObject *b, int override) [3](#hl-10-3){ [4](#hl-10-4) PyDictObject *mp, *other; [5](#hl-10-5) [6](#hl-10-6) assert(0 <= override && override <= 2); [7](#hl-10-7) [8](#hl-10-8) /* We accept for the argument either a concrete dictionary object, [9](#hl-10-9) * or an abstract "mapping" object.  For the former, we can do [10](#hl-10-10) * things quite efficiently.  For the latter, we only require that [11](#hl-10-11) * PyMapping_Keys() and PyObject_GetItem() be supported. [12](#hl-10-12) */ [13](#hl-10-13) if (a == NULL || !PyDict_Check(a) || b == NULL) { [14](#hl-10-14) PyErr_BadInternalCall(); [15](#hl-10-15) return -1; [16](#hl-10-16) } [17](#hl-10-17) mp = (PyDictObject*)a; [18](#hl-10-18) if (PyDict_Check(b) && (Py_TYPE(b)->tp_iter == (getiterfunc)dict_iter)) { [19](#hl-10-19) [20](#hl-10-20) // ... [21](#hl-10-21) else { [22](#hl-10-22) /* Do it the generic, slower way */ [23](#hl-10-23) [24](#hl-10-24) // ... [25](#hl-10-25) } [26](#hl-10-26) ASSERT_CONSISTENT(a); [27](#hl-10-27) return 0; [28](#hl-10-28)}`** 
```

**From the comment, we know that the function has been optimized for being called with another dictionary as a parameter. If the parameter is not a dictionary, then it’s a generic [`Mapping`](https://docs.python.org/3.12/library/collections.abc.html#collections.abc.Mapping) — a container object that supports arbitrary key lookups.**

#### **[#](#the--expression) The `{}` expression**

**The literal expression should be easier to reason about. Let’s go back to the [bytecode.c](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c) file and find mapping for the `BUILD_MAP` opcode.**

```
 **`[1](#hl-11-1)inst(BUILD_MAP, (values[oparg*2] -- map)) { [2](#hl-11-2) map = _PyDict_FromItems( [3](#hl-11-3) values, 2, [4](#hl-11-4) values+1, 2, [5](#hl-11-5) oparg); [6](#hl-11-6) if (map == NULL) [7](#hl-11-7) goto error; [8](#hl-11-8) [9](#hl-11-9) DECREF_INPUTS(); [10](#hl-11-10) ERROR_IF(map == NULL, error); [11](#hl-11-11)}`** 
```

***See sources here: [bytecode.c#L1541](https://github.com/python/cpython/blob/v3.12.0/Python/bytecodes.c#L1541).***

**We see that the dictionary is fully constructed and returned by [`_PyDict_FromItems`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L1616). Let’s go there.**

```
 **`[1](#hl-12-1)PyObject * [2](#hl-12-2)_PyDict_FromItems(PyObject *const *keys, Py_ssize_t keys_offset, [3](#hl-12-3) PyObject *const *values, Py_ssize_t values_offset, [4](#hl-12-4) Py_ssize_t length) [5](#hl-12-5){ [6](#hl-12-6) bool unicode = true; [7](#hl-12-7) PyObject *const *ks = keys; [8](#hl-12-8) PyInterpreterState *interp = _PyInterpreterState_GET(); [9](#hl-12-9) [10](#hl-12-10) for (Py_ssize_t i = 0; i < length; i++) { [11](#hl-12-11) if (!PyUnicode_CheckExact(*ks)) { [12](#hl-12-12) unicode = false; [13](#hl-12-13) break; [14](#hl-12-14) } [15](#hl-12-15) ks += keys_offset; [16](#hl-12-16) } [17](#hl-12-17) [18](#hl-12-18) PyObject *dict = dict_new_presized(interp, length, unicode); [19](#hl-12-19) if (dict == NULL) { [20](#hl-12-20) return NULL; [21](#hl-12-21) } [22](#hl-12-22) [23](#hl-12-23) ks = keys; [24](#hl-12-24) PyObject *const *vs = values; [25](#hl-12-25) [26](#hl-12-26) for (Py_ssize_t i = 0; i < length; i++) { [27](#hl-12-27) PyObject *key = *ks; [28](#hl-12-28) PyObject *value = *vs; [29](#hl-12-29) if (PyDict_SetItem(dict, key, value) < 0) { [30](#hl-12-30) Py_DECREF(dict); [31](#hl-12-31) return NULL; [32](#hl-12-32) } [33](#hl-12-33) ks += keys_offset; [34](#hl-12-34) vs += values_offset; [35](#hl-12-35) } [36](#hl-12-36) [37](#hl-12-37) return dict; [38](#hl-12-38)}`** 
```

**I’ve marked the most interesting line: in opposition to `dict_new`, it **pre-allocates** the dictionary, so it already has a capacity for all of its entries. After that it inserts key-value pairs one-by-one.**

## **[#](#conclusions) Conclusions**

**To sum things up.**

**When you do `dict(a=1, b=2)`, Python needs to:**

*   **allocate a new `PyObject`,**
*   **construct a `dict` via the `__new__` method,**
*   **call its `__init__` method, which internally calls [`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988),**
*   **because `kwargs` is not `dict`, [`PyDict_Merge`](https://github.com/python/cpython/blob/v3.12.0/Objects/dictobject.c#L2988) needs to use the slower method, which inserts entries one-by-one.**

**Whereas doing `{}` causes Python to:**

*   **construct a new **pre-allocated** dictionary,**
*   **insert entries one-by-one.**

**To be fair, unless you construct dictionaries in a loop, I don’t believe there’s a lot to gain performance-wise by switching from `dict` to `{}`. There’s one thing I’d like you to remember after reading this post:**

###### ***tl;dr***

***The `{}` is always faster than `dict`.***

## ***[#](#appendix) Appendix***

***Similar analysis can be done for lists, sets, and (*possibly*) tuples. However, this post is already too long, so I’ll drop all the other resources I’ve gathered here.***

### ***[#](#lists) Lists***

```
***`[1](#hl-13-1)$ python -m timeit "list((1, 2, 'a'))" [2](#hl-13-2)5000000 loops, best of 5: 53.4 nsec per loop [3](#hl-13-3)$ python -m timeit "[1, 2, 'a']" [4](#hl-13-4)10000000 loops, best of 5: 29.7 nsec per loop`*** 
```

#### ***[#](#list-type) `list` type***

***The Python’s `list` type is defined in the `listobject.c` file:***

```
***`[1](#hl-14-1)PyTypeObject PyList_Type = { [2](#hl-14-2) // ... [3](#hl-14-3) (initproc)list___init__, /* tp_init */ [4](#hl-14-4) PyType_GenericAlloc, /* tp_alloc */ [5](#hl-14-5) PyType_GenericNew, /* tp_new */ [6](#hl-14-6) // ... [7](#hl-14-7)}`*** 
```

***`PyType_GenericNew` does nothing — it ignores `args` and `kwargs`, and return an object allocated by `type->typ_alloc`. Source: [https://github.com/python/cpython/blob/main/Objects/typeobject.c#L1768](https://github.com/python/cpython/blob/main/Objects/typeobject.c#L1768)***

***`PyType_GenericAlloc` allocated a new object and (if it’s garbage-collectable) marks it as “to-be-collected”.***

***`list___init__` is a convenience wrapper around the `list___init___impl` function, which does the actual initialization. It does some basic pre-steps:***

*   ***fails if any `kwarg` has been passed,***
*   ***fails if `args` has more than 1 element,***
*   ***converts the first argument of `args` to an iterable.***

***`list___init___impl` clears the list (if not empty) and extends it by the provided iterable (by calling the `list_extend` function).***

###### ****Question****

****I think the two:****

```
****`[1](#hl-0-1)list((1, 2, 3)) [2](#hl-0-2) [3](#hl-0-3)_tmp = list() [4](#hl-0-4)_tmp.extend((1, 2, 3))`**** 
```

****are equivalent in every way (but the generated opcodes sequence) and are calling the same underlying C code.****

#### ****[#](#-literal-expression) `[]` literal expression****

****Opcode `BUILD_LIST` uses the `_PyList_FromArraySteal` function to construct a list object and return it.****

###### *****Note*****

****_PyList_FromArraySteal is used since 3.12\. See [3.12 changelog](https://docs.python.org/3.12/whatsnew/changelog.html) and [gh-100146](https://github.com/python/cpython/pull/100147). Before 3.12, the opcode was repeatedly calling `POP()` to get items from the stack, and then inserting them into the list.****

```
****`[1](#hl-0-1)while (--oparg >= 0) { [2](#hl-0-2) PyObject *item = POP(); [3](#hl-0-3) PyList_SET_ITEM(list, oparg, item); [4](#hl-0-4)}`**** 
```

****See [change in gh-100146 PR](https://github.com/python/cpython/pull/100147/files#diff-729a985b0cb8b431cb291f1edb561bbbfea22e3f8c262451cd83328a0936a342L1447-L1450).****

****The `_PyList_FromArraySteal` helper creates an empty list (if there are no items), or preallocates the list before inserting items to it.****

```
****`[1](#hl-15-1)PyObject **dst = list->ob_item; [2](#hl-15-2)memcpy(dst, src, n * sizeof(PyObject *));`**** 
```

### ****[#](#sets) Sets****

###### *****Warning*****

*****AFAIK there’s no way to construct a new set with a literal expression, hence the following analysis is performed for sets with two elements: `1` and `2`.*****

```
*****`[1](#hl-16-1)$ python -m timeit "set((1, 2))" [2](#hl-16-2)5000000 loops, best of 5: 82.6 nsec per loop [3](#hl-16-3)$ python -m timeit "{1, 2}" [4](#hl-16-4)5000000 loops, best of 5: 45.5 nsec per loop`***** 
```

#### *****[#](#set-type) `set` type*****

*****The Python’s `set` type is defined in the `setobject.c` file:*****

```
*****`[1](#hl-17-1)PyTypeObject PySet_Type = { [2](#hl-17-2) // ... [3](#hl-17-3) (initproc)set_init, /* tp_init */ [4](#hl-17-4) PyType_GenericAlloc, /* tp_alloc */ [5](#hl-17-5) set_new, /* tp_new */ [6](#hl-17-6) // ... [7](#hl-17-7)}`***** 
```

*****`set_new` uses the allocated to create a new empty set object. Internally it calls the `make_new_set` function.*****

*****`set_init` performs some pre-steps:*****

*   *****fails if any `kwarg` has been passed,*****
*   *****fails if `args` has more then 1 element,*****
*   *****converts the first argument of `args` to an iterable,*****
*   *****if the set object has already been filled, then it clears it*****

*****&mldr; and calls the `set_update_internal` function, to insert values into the set object.*****

#### *****[#](#1-2-literal-expression) `{1, 2}` literal expression*****

*****The opcode handler has no dedicated helpers. It constructs a new set (empty), and then iterates over all items from the stack and insets them one-by-one.*****

```
 *****`[1](#hl-18-1)inst(BUILD_SET, (values[oparg] -- set)) { [2](#hl-18-2) set = PySet_New(NULL); [3](#hl-18-3) if (set == NULL) [4](#hl-18-4) GOTO_ERROR(error); [5](#hl-18-5) int err = 0; [6](#hl-18-6) for (int i = 0; i < oparg; i++) { [7](#hl-18-7) PyObject *item = values[i]; [8](#hl-18-8) if (err == 0) [9](#hl-18-9) err = PySet_Add(set, item); [10](#hl-18-10) Py_DECREF(item); [11](#hl-18-11) } [12](#hl-18-12) if (err != 0) { [13](#hl-18-13) Py_DECREF(set); [14](#hl-18-14) ERROR_IF(true, error); [15](#hl-18-15) } [16](#hl-18-16)}`***** 
```

*****Source: [https://github.com/python/cpython/blob/main/Python/bytecodes.c#L1627](https://github.com/python/cpython/blob/main/Python/bytecodes.c#L1627).*****

###### ******Warning******

*****Warning The generated bytecode is different if we add another argument to the literal expression: `{1, 2, 3}`. Then it becomes:*****

```
*****`[1](#hl-0-1) 2 BUILD_SET                0 [2](#hl-0-2) 4 LOAD_CONST               1 (frozenset({1, 2, 3})) [3](#hl-0-3) 6 SET_UPDATE               1 [4](#hl-0-4) 8 RETURN_VALUE`***** 
```

### *****[#](#tuples) Tuples*****

*****This one’s tricky. Constructing a tuple with the `tuple` type requires an iterable. The easiest way of getting one it to create a tuple literal, which is obviously less efficient, than using a tuple literal syntax in the first place.*****

```
 *****`[1](#hl-19-1)>>> import sys [2](#hl-19-2)>>> sys.version_info [3](#hl-19-3)sys.version_info(major=3, minor=12, micro=1, releaselevel='final', serial=0) [4](#hl-19-4)>>> [5](#hl-19-5)>>> def a(): [6](#hl-19-6)...   return tuple((1, 2, [])) [7](#hl-19-7)>>> [8](#hl-19-8)>>> def b(): [9](#hl-19-9)...   return (1, 2, []) [10](#hl-19-10)>>> [11](#hl-19-11)>>> import dis [12](#hl-19-12)>>> [13](#hl-19-13)>>> dis.dis(a) [14](#hl-19-14) 1           0 RESUME                   0 [15](#hl-19-15) [16](#hl-19-16) 2           2 LOAD_GLOBAL              1 (NULL + tuple) [17](#hl-19-17) 12 LOAD_CONST               1 (1) [18](#hl-19-18) 14 LOAD_CONST               2 (2) [19](#hl-19-19) 16 BUILD_LIST               0 [20](#hl-19-20) 18 BUILD_TUPLE              3 [21](#hl-19-21) 20 CALL                     1 [22](#hl-19-22) 28 RETURN_VALUE [23](#hl-19-23)>>> dis.dis(b) [24](#hl-19-24) 1           0 RESUME                   0 [25](#hl-19-25) [26](#hl-19-26) 2           2 LOAD_CONST               1 (1) [27](#hl-19-27) 4 LOAD_CONST               2 (2) [28](#hl-19-28) 6 BUILD_LIST               0 [29](#hl-19-29) 8 BUILD_TUPLE              3 [30](#hl-19-30) 10 RETURN_VALUE`***** 
```

*****It might not make much sense to include this example in the article.*****

#### *****[#](#tuple-type) `tuple` type*****

*****The Python’s `tuple` type is defined in `tupleobject.c`. Interestingly, it has no `tp_alloc`, nor `tp_init`.*****

```
*****`[1](#hl-20-1)PyTypeObject PyTuple_Type = { [2](#hl-20-2) // ... [3](#hl-20-3) 0, /* tp_init */ [4](#hl-20-4) 0, /* tp_alloc */ [5](#hl-20-5) tuple_new, /* tp_new */ [6](#hl-20-6) // ... [7](#hl-20-7)}`***** 
```

*****The `tuple_new` function is responsible for allocating a new tuple object, with the items passed as a parameter.*****

</main>