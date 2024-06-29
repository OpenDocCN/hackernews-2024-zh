<!--yml

category: 未分类

date: 2024-05-27 15:02:17

-->

# C++退出时析构函数 | MaskRay

> 来源：[https://maskray.me/blog/2024-03-17-c++-exit-time-destructors](https://maskray.me/blog/2024-03-17-c++-exit-time-destructors)

在ISO C++标准中，[basic.start.term]指定：

> 具有静态存储期的构造对象（[dcl.init]）将在调用std::exit ([support.start.term])时销毁，并调用注册的函数。对std::exit的调用在销毁和注册函数之前序列化。[注释1：从main返回会调用std::exit ([basic.start.main])。-结束注释]

例如，考虑以下代码：

对象a的析构函数将在程序终止时注册执行。

## `__cxa_atexit`

Itanium C++ ABI 使用 `__cxa_atexit` 而不是 `atexit` 来注册对象析构函数的主要原因有两点：

+   有限的 `atexit` 保证：ISO C（直到C23）保证支持32个注册函数，尽管大多数实现支持更多。

+   动态库卸载：`__cxa_atexit` 提供了一个机制，用于在程序终止前通过 `dlclose` 卸载动态库时处理析构函数。

包括glibc、musl和FreeBSD libc在内的几个标准库使用`__cxa_atexit`实现`atexit`。

+   在glibc中，`atexit` 返回 `__cxa_atexit ((void (*) (void *)) func, NULL, __dso_handle)`，其中 `__dso_handle` 是libc本身的一部分。

+   musl使用0代替`__dso_handle`。

[https://itanium-cxx-abi.github.io/cxx-abi/abi.html#dso-dtor-runtime-api](https://itanium-cxx-abi.github.io/cxx-abi/abi.html#dso-dtor-runtime-api)提供了关于对象销毁机制的详细文档。让我们以GCC和glibc示例来说明：

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19

```

|

```
cat > a.cc <<'eof'
 int main() {
 void *h = dlopen("./b.so", RTLD_NOW);
 ((void (*)())dlsym(h, "foo"))();
 dlclose(h);
}
eof
cat > b.cc <<'eof'
 struct A { ~A(); } ga;
A::~A() { printf("~A %p\n", this); }
extern "C" void foo() {
 static A a;
 puts("foo");
}
eof
g++ -fpic -shared b.cc -o b.so
g++ a.cc -o a 
```

|

一个调用产生：

|

```
1
2
3

```

|

```
foo
~A 0x7f70d66c4c79  // for the static-local variable
~A 0x7f70d66c4c78  // for the global variable

```

|

主要观点：

+   编译器使用 `__dso_handle` 符号注册析构函数与 `__cxa_atexit`。

+   `crtbeginS.o` 定义了 `.fini_array` 节（触发 `__do_global_dtors_aux`）和隐藏符号 `__dso_handle`。

+   自2017年起，如果crtbegin没有，lld [将`__dso_handle`定义为隐藏符号](https://reviews.llvm.org/D33856)。

+   `dlclose`调用`.fini_array`函数。 `__cxa_finalize(d)`迭代终止函数列表，根据DSO句柄调用匹配的析构函数。

+   `__cxa_atexit`实现通常动态分配内存并可能失败。这些失败会被简单地忽略。

注意：在glibc中，`DF_1_NODELETE`标志将共享对象标记为可卸载。另外，使用`STB_GNU_UNIQUE`进行符号查找会自动设置此标志。

musl提供了一个[无操作的实现 `dlclose` 和 `__cxa_finalize`](https://wiki.musl-libc.org/functional-differences-from-glibc.html#Unloading_libraries)。

## 线程存储期变量

具有非平凡析构函数的线程存储期变量将在构造期间使用 `__cxa_thread_atexit` 注册这些析构函数。

## 当不希望退出时析构函数

针对静态和线程存储期变量的退出时析构函数可能是不希望的，因为

+   不必要的开销和复杂性：包括操作系统内核和内存受限系统。

+   潜在的竞态条件：在线程终止期间，析构函数可能会执行，而其他线程仍然尝试访问对象。例如：[webkit](https://bugs.webkit.org/show_bug.cgi?id=21810)

Clang 提供了 `-Wexit-time-destructors`（默认情况下禁用），用于警告退出时析构函数。

|

```
1
2
3
4
5

```

|

```
% clang++ -c -Wexit-time-destructors g.cc
g.cc:1:20: warning: declaration requires an exit-time destructor [-Wexit-time-destructors]
 1 &#124; struct A { ~A(); } a;
 &#124;                    ^
1 warning generated.

```

|

## 禁用退出时析构函数

接下来，我将描述一些禁用退出时析构函数的方法。

### 指向动态分配对象的指针/引用

我们可以使用引用或指针来引用动态分配的对象。

|

```
1
2
3
4
5
6

```

|

```
struct A { int v; ~A(); };
A &g = *new A; 
A &foo() {
 static A &a = *new A;
 return a; 
}

```

|

该方法防止析构函数在程序退出时运行，因为指针/引用具有平凡析构函数。请注意，这不会导致内存泄漏，因为指针/引用是根集的一部分。

主要的缺点是在访问对象时产生不必要的指针间接性。此外，此方法在数据段中使用一个可变指针并需要内存分配。

|

```
1
2
3
4
5
6

```

|

```
# %bb.2:                                 // initializer
 movl    $4, %edi
 callq   _Znwm@PLT
 movq    %rax, _ZZ3foovE1a(%rip)  // store pointer of the heap-allocated object to _ZZ3foovE1a
...
 movq    _ZZ3foovE1a(%rip), %rax  // load a pointer from _ZZ3foovE1a

```

|

## 具有空析构函数的类模板

一个常见的方法，如 [P1247](https://wg21.link/p1247) 所述，是使用一个空析构函数的类模板来防止退出时析构：

|

```
1
2
3
4
5
6
7
8

```

|

```
template <class T> class no_destroy {
 alignas(T) std::byte data[sizeof(T)];
public:
 template <class... Ts> no_destroy(Ts&&... ts) { new (data) T(std::forward<Ts>(ts)...); }
 T &get() { return *reinterpret_cast<T *>(data); }
};
 no_destroy<widget> my_widget; 
```

|

libstdc++ 使用了一个使用联合成员的变体。

|

```
1
2
3
4
5
6
7
8
9
10
11
12

```

|

```
struct A { ~A(); };
 namespace {
 struct constant_init {
 union { A obj; };
 constexpr constant_init() : obj() { }
 ~constant_init() {  }
 };
 constinit constant_init global;
}
 A* get() { return &global.obj; } 
```

|

C++20 将支持 constexpr 析构函数：

|

```
1
2
3
4
5
6

```

|

```
template <class T> union no_destroy {
 template <typename... Ts>
 explicit constexpr no_destroy(Ts&&... args) : obj(std::forward(args)...) {}
 constexpr ~no_destroy() {}
 T obj;
};

```

|

类似 [`absl::NoDestructor`](https://github.com/abseil/abseil-cpp/blob/master/absl/base/no_destructor.h) 和 [`folly::Indestructible`](https://github.com/facebook/folly/blob/main/folly/Indestructible.h) 的库提供类似功能。absl 版本针对可平凡析构类型进行了优化。

### 用于无操作析构函数的编译器优化

理想情况下，编译器应该优化用户提供的空析构函数以排除退出时析构函数：

|

```
1
2

```

|

```
struct C { C(); ~C() {} };
void foo() { static C c; }

```

|

LLVM 自 2011 年以来已解决了这个问题 [commit link](https://github.com/llvm/llvm-project/commit/ee6bc70d2f1c2434ca9ca8092216bdeab322c7e5)。其 GlobalOpt 通过消除与空析构函数相关的 `__cxa_atexit` 调用来优化其他全局变量。

相比之下，自 2005 年以来，GCC 一直有一个 [开放的功能请求](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=19661) 来优化这一点。

### `no_destroy` 属性

Clang 支持 `[[clang::no_destroy]]`（替代形式：`__attribute__((no_destroy))`）来禁用静态或线程存储期变量的退出时析构函数。其 `-fno-c++-static-destructors` 选项允许全局禁用退出时析构函数。

此属性的标准化工作正在进行中 [P1247R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1247r0.html)。

最近，我遇到了一个场景，其中 `no_destroy` 属性将会很有用。我在了解到 GCC 不具备该属性后，已经提交了一个 GCC 功能请求 ([PR114357](https://gcc.gnu.org/PR114357))。

## 案例研究

LLVM 提供了 `ManagedStatic` 以便按需构造对象（有助于减少启动时间），并通过 `llvm_shutdown` 明确进行销毁。`ManagedStatic` 旨在在命名空间范围内使用。一个典型示例是 LLVM 的统计机制（`-stats` 和 `-time-passes`）。

使用 LLVM 的程序可以通过跳过一些析构函数来战略性地避免调用 `llvm_shutdown` 以进行快速拆卸。lld 链接器采用了这种方法，除非将 `LLD_IN_TEST` 环境变量设置为非零整数。

需要库卸载的 DSO 插件用户可能会发现 [使用 `ManagedStatic` 不合适](https://discourse.llvm.org/t/making-llvm-play-nice-r-when-used-as-a-shared-library-in-a-plugin-setting/63306)。这是因为：

+   DSO 可能无法确定进程中是否存在其他活动的 LLVM 用户，因此调用 `llvm_shutdown` 可能是不安全的。

+   如果将 `llvm_shutdown` 推迟到程序退出时附近，一旦 DSO 的代码被移除，执行析构函数就变得不安全。

`mold`链接器通过为链接任务生成一个单独的进程来提高链接速度的感知。这允许父进程（从 shell 或其他程序启动的进程）尽早退出。这种方法消除了与静态析构函数和其他操作相关的开销。
