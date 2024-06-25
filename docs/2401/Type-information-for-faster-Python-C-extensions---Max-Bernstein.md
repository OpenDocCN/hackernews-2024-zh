<!--yml

类别：未分类

日期：2024年05月27日 14:46:58

-->

# 为更快的 Python C 扩展提供类型信息 | Max Bernstein

> 来源：[https://bernsteinbear.com//blog/typed-c-extensions/](https://bernsteinbear.com//blog/typed-c-extensions/)

**更新：** 这篇文章的纸质版本已被 PLDI SOAP 2024接受。请查看[预印本](/assets/img/dr-wenowdis.pdf)（PDF）。

PyPy 是 Python 语言的另一种实现。PyPy 的 C API 兼容层存在一些性能问题。[CF Bolz-Tereick](https://cfbolz.de/) 和我正在研究一种使 PyPy 的 C API 交互速度更快的方法。看起来非常有希望。以下是它的大致工作原理的草图。

## 作为通用语言的 C API

Python 的使用相当广泛。多年来，CPython 是唯一的实现，而且 CPython 并不是为了速度而设计的。Python 社区需要一些程序运行得更快，并确定最佳路径是在 C 中编写一些模块，并从 Python 中与它们交互。效果很好。

其他 Python 运行时，如 PyPy，也随之而来。PyPy 包含一个 JIT 编译器，它执行普通 Python 代码的速度*非常*快，至少在调用从 Python 到 C 扩展函数时是这样。然后事情有点走样了。首先，PyPy JIT 不能“看到”本地代码；它是由其他编译器在 JIT 之外生成的，因此是不透明的。其次，因为前述 C 模块的绑定 API（“C API”）需要完全不同的对象和内存表示，而不是 PyPy 内部所具有的。

PyPy 有自己的对象模型、运行时和移动垃圾收集器，这一切都是为了获得更好的性能。不幸的是，这意味着每当你从 PyPy 调用 C API 函数时，它都必须停下正在做的事情，设置一些 C API 脚手架，进行 C API 调用，然后拆除脚手架。

例如，C API 以 `PyObject` 指针为中心。在 Python 代码的正常执行过程中，PyPy 不使用 `PyObject`。当它与 C API 代码交互时，它必须分配一个 `PyObject`，使其指向 PyPy 堆，调用 C API 函数，然后（可能）释放 `PyObject`。（这忽略了 GIL 问题和异常检查，这也是一个问题。）

图 1 - C 扩展以`PyObject*`为中心，因此任何希望与它们交互的运行时也必须以`PyObject*`为中心。

这是很多额外开销。（而且还有更多。另请参阅 Antonio Cuni 的[卓越博文](https://www.pypy.org/posts/2018/09/inside-cpyext-why-emulating-cpython-c-8083064623681286567.html)。）这是一个棘手的问题，已经影响了多个替代 Python 运行时^。

除了将其装箱为 `PyObject` 的开销之外，C 扩展调用的底层 C 函数甚至可能*不需要*首先存在 `PyObject`。例如，很多 C API 函数的结构都是这样的：

```
long inc_impl(long num) {
    return num + 1;
}

PyObject* inc(PyObject* obj) {
    long num = PyLong_AsLong(obj);
    if (num < 0 && PyErr_Occurred()) {
        return NULL;
    }
    long result = inc_impl(num);
    return PyLong_FromLong(result);
} 
```

在这个示例中，`PyObject*` 代码 `inc` 只是另一个直接处理 C 整数的函数 `inc_impl` 的包装器。

图2 - 即使底层 C 代码对 Python 一无所知，运行时仍然必须制造 `PyObject*`。不幸的是，解包是另一个开销的来源。

JIT 和 C 实现之间的所有中间位（实际上整个 `inc` 函数）都是“浪费的工作”，因为这些工作对用户程序的实际执行是不需要的。

所以即使 PyPy JIT 做得很好，并且已经在 Python 代码中消除了内存分配（例如，PyPy 可能已经将一些堆分配的 Python 整数对象解包为 C 长整型），它仍然必须为 C API 堆分配一个 `PyObject`…只是不久之后就会丢弃它。

如果有一种方式可以告诉 PyPy `inc` 需要一个 `long` 并将其解包为 C `long`（还会返回一个 C `long`），那么它就不需要做所有这些小动作了。

是的，理想情况下根本不应该有 C API 调用。但有时你必须这样做（也许因为你无法控制代码），你也可以加快这个调用。让我们看看是否能做到。

## 波茨坦

这就是我参与其中的地方。我在 2022 年的 ECOOP 大会上，[CF](https://cfbolz.de/)介绍了我给一些其他替代 Python 运行时的实现者。我和 PyPy、ZipPy 和 GraalPython 的作者们在喝咖啡和啤酒的时候交谈了一下。他们人都很好。

他们一直在共同努力开发一个名为 HPy 的项目。HPy 是一个考虑到替代运行时的 C API Python 的新设计。作为这一设计的一部分，他们正在[研究一种方法来通过 C API 将类型信息](https://github.com/hpyproject/hpy/issues/129)从 C 扩展模块传递到主机运行时可以读取的地方。

这是一个棘手的问题，因为不仅有 C API，还有 C ABI（注意“B”代表“二进制”）。虽然 API 是调用方和被调用方之间如何调用函数的抽象约定，但 ABI 更具体。在 C ABI 的情况下，这意味着不改变结构布局或大小，添加函数参数等。这是一种相当严格的约束，不清楚添加类型信息的最佳向后兼容方式是什么。

在这次会议或不久之后的某个时候，我有了一个关于如何在不更改 API 或 ABI 的情况下完成它的想法，并决定为 [Cinder](https://github.com/facebookincubator/cinder/)（我当时正在开发的 Python 运行时）尝试实现它。

## 解决方案：不靠谱的 C 东西？

为了更好地理解问题，让我们看看我们想要添加的类型信息的类型。这是我们想要添加到每个有类型的方法中的类型元数据，表示为一个 C 结构。

```
struct PyPyTypedMethodMetadata {
  int arg_type;
  int ret_type;
  void* underlying_func;
};
typedef struct PyPyTypedMethodMetadata PyPyTypedMethodMetadata; 
```

在这个人为限制的示例中，我们存储了一个参数的类型信息（但是未来可能会添加更多），返回值的类型信息，以及底层（非`PyObject*`）C函数指针。

但不清楚将它放在`PyMethodDef`中的什么位置。现有的`PyMethodDef`结构看起来是这样的。它包含一些元数据和一个C函数指针（`PyObject*`）。在理想情况下，我们可以“只是”向这个结构体添加类型元数据，并且完成它。但出于ABI理由，我们无法改变其大小。

```
struct PyMethodDef {
    const char  *ml_name;   /* The name of the built-in function/method */
    PyCFunction  ml_meth;   /* The C function that implements it */
    int          ml_flags;  /* Combination of METH_xxx flags, which mostly
                               describe the args expected by the C func */
    const char  *ml_doc;    /* The __doc__ attribute, or NULL */
};
typedef struct PyMethodDef PyMethodDef; 
```

该怎么办？我决定搞得有点奇怪，并查看是否可以以某种方式将指针暗中放置到元数据中。我的最初想法是将整个`PyPyTypedMethodMetadata`结构体*放在*`PyMethodDef`结构体的后面（有点类似于`malloc`的工作方式），但那样不太好：`PyMethodDef`通常是静态分配在数组中的，我们无法更改这些数组的布局。

但我们*能*做的是将`ml_name`字段指向另一个结构体内部的缓冲区^。

然后，当我们注意到一个方法被标记为类型化（使用我们可以添加到`ml_flags`位集中的新`METH_TYPED`标志），我们可以倒退读取以找到`PyPyTypedMethodMetadata`结构体。下面是你在C中可能会这样做的方式：

```
struct PyPyTypedMethodMetadata {
  int arg_type;
  int ret_type;
  void* underlying_func;
  const char ml_name[100];  // New field!
};
typedef struct PyPyTypedMethodMetadata PyPyTypedMethodMetadata;

PyPyTypedMethodMetadata*
GetTypedSignature(PyMethodDef* def)
{
  assert(def->ml_flags & METH_TYPED);  // A new type of flag!
  return (PyPyTypedMethodMetadata*)(def->ml_name - offsetof(PyPyTypedMethodMetadata, ml_name));
} 
```

这里有一个示意图来说明这一点，因为它真的很奇怪和令人困惑。

我用C做了一个[模拟实现](https://gist.github.com/tekknolagi/f4acd2202f6448d4a5813a43eda90121)（没有Python C API，只是假的结构来勾勒出它）,它起作用了。所以我在Cinder中实现了一个妥协版本，但从未发布，因为我的与Cinder的集成有些太妥协了。我为了后代的记录[写下了这些想法](https://github.com/faster-cpython/ideas/issues/546)，以防有人想承担这个项目。

一年后，没有人采用，所以我决定找CF看看我们是否可以在PyPy中实现它。我们马上就会看到这个实现是什么样子。但首先，让我们来看看C扩展是从哪里来的。

## 所有的C扩展都是从哪里来的？

嗯，在PyPy中，标准库中没有。PyPy几乎完全用Python编写，以便JIT能够看到代码。但人们喜欢使用Python包，有些Python包包含C扩展。

编写C扩展有几种不同的方法。 “最简单的”（即，所有组件都是可见的，没有魔法，也没有外部依赖）是手写它。如果你不想这样做，你也可以使用绑定生成器来为你编写粘合代码。我在Cython上有最丰富的经验，但其他绑定生成器如nanobind，pybind11，甚至CPython自己的Argument Clinic也存在！

### 手写的

让我们回顾一下之前的 `inc`/`inc_impl` 函数。这是一个合理的例子，可以将其集成为手写的 C 扩展到 Python 中。为了使其可以从 Python 调用，我们必须创建一个完整的 C 扩展模块。在这种情况下，这只是一个函数指针列表以及如何调用它们。

```
#include <Python.h>  
long inc_impl(long arg) {
  return arg+1;
}

PyObject* inc(PyObject* module, PyObject* obj) {
  (void)module;
  long obj_int = PyLong_AsLong(obj);
  if (obj_int == -1 && PyErr_Occurred()) {
    return NULL;
  }
  long result = inc_impl(obj_int);
  return PyLong_FromLong(result);
}

static PyMethodDef mytypedmod_methods[] = {
    {"inc", inc, METH_O, "Add one to an int"},
    {NULL, NULL, 0, NULL}};

static struct PyModuleDef mytypedmod_definition = {
    PyModuleDef_HEAD_INIT, "mytypedmod",
    "A C extension module with type information exposed.", -1,
    mytypedmod_methods,
    NULL,
    NULL,
    NULL,
    NULL};

PyMODINIT_FUNC PyInit_mytypedmod(void) {
  PyObject* m = PyState_FindModule(&mytypedmod_definition);
  if (m != NULL) {
    Py_INCREF(m);
    return m;
  }
  return PyModule_Create(&mytypedmod_definition);
} 
```

我们有一个 `PyMethodDef` 结构体的数组，每个方法我们都想要进行包装。然后我们有一个 `PyModuleDef` 来定义模块，其中还可以包含属性和一些其他内容。然后我们提供了一种模块的 `__new__` 函数，以 `PyInit_` 函数的形式。这是由 Python 中内置的 C 扩展加载器中的 `dlopen` 发现的。

可以手动扩展此模块，方法是添加一个 `PyPyTypedMethodMetadata` 结构体和一个 `METH_TYPED` 标志。这有点繁琐，但如果加速与模块的交互…嗯，扩展作者可能会被说服添加类型信息，或者至少接受一个拉取请求。

但并非所有扩展都是手工编写的。许多是由诸如 Cython 之类的绑定生成器生成的。而且 Cython 很有趣，因为它可以自动生成类型签名…

### Cython

与许多其他用于 Python 的绑定生成器不同，Cython 提供了一个完全功能齐全的类似 Python 的编程语言，它可以编译为 C 语言。类型服从不同于普通 Python 代码的规则，并且可以用于优化。Cython 还具有原始类型。让我们看一个例子。

在这段 Cython 代码中，我们制作了一个将两个机器整数相加并返回一个机器整数的函数。

```
cpdef int add(int a, int b):
    return a + b 
```

Cython 将生成一个非常快速的 C 函数，用于将两个机器整数相加。对于这些函数的调用在 Cython 中在编译时进行类型检查，并且将尽可能快地进行，就像您的 C 编译器允许的那样。

```
static int add(int __pyx_v_a, int __pyx_v_b) {
  return __pyx_v_a + __pyx_v_b;
} 
```

由于我们使用了 `cpdef` 而不是 `cdef`，Cython 还会生成一个包装的 C 扩展函数，这样就可以从 Python 中调用该函数。

这意味着生成的 Cython 包装器代码看起来像（一个更加丑陋的版本）下面。**您不需要理解或者实际上甚至不需要阅读** 下面的大量整理过的和带有注释的生成代码。您只需要说“哦”和“啊”以及“哇，这么多的 if 语句和这么多的分配和这么多的函数调用”。

与上面的 `METH_O` 示例稍有不同，因为它必须展开一个快速调用参数的数组并进行一些参数处理。

```
static PyObject *add_and_box(CYTHON_UNUSED PyObject *__pyx_self,
                             int __pyx_v_a,
                             int __pyx_v_b) {
  int result = add(__pyx_v_a, __pyx_v_b, 0);
  // Check if an error occurred (unnecessary in this case)
  if (result == ((int)-1) && PyErr_Occurred()) {
    return NULL;
  }
  // Wrap result in PyObject*
  return PyLong_FromLong(result);
}

static PyObject *add_python(PyObject *__pyx_self,
                            PyObject *const *__pyx_args,
                            Py_ssize_t __pyx_nargs,
                            PyObject *__pyx_kwds) {
  // Check how many arguments were given
  PyObject* values[2] = {0,0};
  if (__pyx_nargs == 2) {
    values[1] = __pyx_args[1];
    values[0] = __pyx_args[0];
  } else if (__pyx_nargs == 1) {
    values[0] = __pyx_args[0];
  }
  // Check if any keyword arguments were given
  Py_ssize_t kw_args = __Pyx_NumKwargs_FASTCALL(__pyx_kwds);
  // Match up mix of position/keyword args to parameters
  if (__pyx_nargs == 0) {
    // ...
  } else if (__pyx_nargs == 1) {
    // ...
  } else if (__pyx_nargs == 2) {
    // ...
  } else {
    // ...
  }
  // Unwrap PyObject* args into C int
  int __pyx_v_a = PyLong_AsLong(values[0]);
  // Check for error (unnecessary if we know it's an int)
  if ((__pyx_v_a == (int)-1) && PyErr_Occurred()) {
    return NULL;
  }
  int __pyx_v_b = PyLong_AsLong(values[1]);
  // Check for error (unnecessary if we know it's an int)
  if ((__pyx_v_b == (int)-1) && PyErr_Occurred()) {
    return NULL;
  }
  // Call generated C implementation of add
  return add_and_box(__pyx_self, __pyx_v_a, __pyx_v_b);
} 
```

现在，要明确一点：这可能是与 CPython 交互的最快的方法了。Cython 已经开发了很多年了，它非常快。但是 CPython 不是唯一的运行时，而其他运行时具有不同的性能特征，正如我们上面所探讨的那样。

由于许多 C 扩展是由 Cython 生成的，这是一个很大的机会：如果我们设法让 Cython 编译器为其编译的函数生成带有类型的元数据，那么这些函数在 PyPy 等运行时下可能会变得 *非常* 快。

为了证明这样的代码更改是合理的，我们必须看看类型元数据使事情变得多快。所以让我们来进行基准测试。

## 一个小小的、没什么用的基准测试

让我们尝试用一个愚蠢的基准测试来测试与本地模块的解释器交互。这有点愚蠢，因为在我熟悉的用例中，不太常见（无论如何）在不在 C 中编写循环的情况下调用 C 代码。但这将是一个很好的参考，可以看到我们可以恢复多少性能。

```
# bench.py import mytypedmod

def main():
    i = 0
    while i < 10_000_000:
        i = mytypedmod.inc(i)
    return i

if __name__ == "__main__":
    print(main()) 
```

我们将先用 CPython 运行它，因为 CPython 不会出现这种问题，它不会制造 `PyObject`——这只是运行时的默认对象表示。

```
$  python3.10 setup.py build
$  time python3.10 bench.py
10000000
846.6ms $ 
```

好吧，文本输出有点模糊，因为实际上我是用`hyperfine`测量的，但你明白我的意思。CPython在*C回溯和前进*的过程中需要850ms，这是非常令人尊敬的。

现在让我们看看 PyPy 在时间上的表现，因为它在边界处做了更多的工作。

```
$  pypy3.10 setup.py build
$  time pypy3.10 bench.py
10000000
2.269s $ 
```

是的，好吧，所以在我们的改变之前，PyPy 做的所有额外不必要的工作最终真的会积少成多。我们对 `inc` 的基准测试比 CPython *慢三倍*。哎呀。但本文的重点是添加类型。如果我们给 C 模块添加类型并且*在*我们对 PyPy 的更改进行测量呢？

这是 C 模块的变化：

```
diff --git a/tmp/cext.c b/tmp/cext-typed.c
index 8f5b31f..52678cb 100644 --- a/tmp/cext.c +++ b/tmp/cext-typed.c @@ -14,8 +14,15 @@ PyObject* inc(PyObject* module, PyObject* obj) {
   return PyLong_FromLong(result);
 }

+PyPyTypedMethodMetadata inc_sig = {
+  .arg_type = T_C_LONG,
+  .ret_type = T_C_LONG,
+  .underlying_func = inc_impl,
+  .ml_name = "inc",
+};
+
 static PyMethodDef mytypedmod_methods[] = {
-    {"inc", inc, METH_O, "Add one to an int"}, +    {inc_sig.ml_name, inc, METH_O | METH_TYPED, "Add one to an int"},
     {NULL, NULL, 0, NULL}};

 static struct PyModuleDef mytypedmod_definition = { 
```

现在让我们用我们的新修补过的 PyPy 运行它：

```
$  pypy3.10-patched setup.py build
$  time pypy3.10-patched bench.py
10000000
168.1ms $ 
```

168ms！为了提醒你，这比 CPython 快**5倍**，比基准 PyPy 快**13倍**。当我看到这个数字时，我老实说不敢相信我的眼睛。我认为还有*更多的改进空间*，比如在 JIT 内部进行签名/元数据查找，而不是调用那个 C 函数。

这是非常有前途的。

但正如我之前所说，大多数应用程序不是由一个 Python 程序调用一个 C 函数并且只有一个 C 函数在一个紧密的循环中。重要的是要分析这种变化如何影响*代表性*工作负载。这将有助于激励像 Cython 这样的绑定生成器中包含这些类型签名。

与此同时，让我们来看看 PyPy 代码库中的这些变化是什么样的。

## 在 PyPy 中实现：PyPy 内部

PyPy 由两个主要部分组成：

+   一个 Python 解释器

+   将解释器转换为 JIT 编译器的工具

这意味着我们不是编写了一些花哨的 JIT 编译器更改来使其工作，而是编写了一个解释器更改。他们的 `cpyext`（C API）处理代码已经包含了一个小型的“解释器”，用于调用 C 扩展。例如，它查看 `ml_flags` 来区分 `METH_O` 和 `METH_FASTCALL`。

因此，我们添加了一个看起来像这样的新情况的伪代码：

```
diff --git a/tmp/interp.py b/tmp/typed-interp.py
index 900fa9c..b973f13 100644 --- a/tmp/interp.py +++ b/tmp/typed-interp.py @@ -1,7 +1,17 @@
 def make_c_call(meth, args):
     if meth.ml_flags & METH_O:
         assert len(args) == 1
+        if meth.ml_flags & METH_TYPED:
+            return handle_meth_typed(meth, args[0])
         return handle_meth_o(meth, args[0])
     if meth.ml_flags & METH_FASTCALL:
         return handle_meth_fastcall(meth, args)
     # ...
+
+def handle_meth_typed(meth, arg):
+    sig = call_scary_c_function_to_find_sig(meth)
+    if isinstance(arg, int) and sig.arg_type == int and sig.ret_type == int:
+        unboxed_arg = convert_to_unboxed(arg)
+        unboxed_result = call_f_func(sig.underlying_func, unboxed_arg)
+        return convert_to_boxed(unboxed_result)
+    # ... 
```

由于 JIT 可能已经知道 C 扩展函数的参数类型（可能也已经解开了它们），所有中间检查和分配都可以省略。这就减少了很多工作！

要查看PyPy的实际更改，请看[这些提交记录](https://github.com/pypy/pypy/compare/3a06bbe5755a1ee05879b29e1717dcbe230fdbf8...branches/py3.10-mb-typed-sig-experiments)。

## 下一步

这个项目还没有合并或完成。虽然我们有一个不错的小测试套件和一个微基准，但理想情况下，我们还应该做一些更多的事情：

+   使其他本地类型（其他整数类型，双精度）工作

+   使多个参数工作（快速调用）

+   在Cython中推敲一个这个想法的概念证明

+   使签名更具表达性

    +   或许我们应该有一种类似于CPython的Argument Clinic的迷你语言

+   将其集成到其他运行时，比如GraalPython甚至CPython

    +   虽然这暂时对CPython没有帮助*，但当他们在JIT中进行更多优化时，他们可能会觉得很有用

如果你有任何想法，请告诉我们！

## 更新和其他想法

*Hacker News*上的*lifthrasiir* [指出](https://news.ycombinator.com/item?id=38989823) 这个结构应该进行版本化以未雨绸缪。他们还建议可能避免使用`METH_TYPED`，而是为每个`foo`发货一种类似的兄弟符号`_PyPyTyped_foo`。这很有趣。

pybind11和nanobind的著名人士Wenzel Jakob指出，我们不应该在我们的实现中忽略重载（显然在野外相当常见）。显然，人们喜欢编写`myextension.abs(x)`，其中对`abs_float`或`abs_int`的调用在绑定胶水中被分派（我在GitHub上的搜索中找到的一个真实例子是[在此处可用](https://github.com/lief-project/LIEF/blob/58f499a22221cee7ab6682986b1504b90acd8556/api/python/src/ELF/init.cpp#L100)）。像PyPy这样的JIT可能会取消动态分派。

另一个想法，来自CF，可能更加困难：Cython编译器能否生成Python代码或字节码？又或者，PyPy是否能吸收Cython代码？
