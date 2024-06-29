<!--yml

category: 未分类

日期：2024-05-27 12:54:06

-->

# std::launder：C++17 中最隐晦的新特性 · miyuki's 博客

> 来源：[https://miyuki.github.io/2016/10/21/std-launder.html](https://miyuki.github.io/2016/10/21/std-launder.html)

# std::launder：C++17 中最隐晦的新特性

2016年10月21日

提案 [P0137R1](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0137r1.html) 在 C++ 标准库中引入了一个名为 `std::launder` 的新函数。不幸的是，这个提案对普通人来说很难理解。

这是 Botond Ballo 在他向 C++ 标准委员会会议的[旅行报告](https://botondballo.wordpress.com/2016/07/06/trip-report-c-standards-meeting-in-oulu-june-2016/)中写到 `std::launder` 的内容。

> 不要问。如果你不是全球约5个人中的一员，已经知道这是什么，那么你不需要知道或者想知道。

但是，让我们试着理解一下 `std::launder` 是如何工作的。原始提案提供了一个例子：

```
struct X { const int n; };
union U { X x; float f; };
void tong() {
  U u = {{ 1 }};
  u.f = 5.f;               // OK, creates new subobject of 'u'
  X *p = new (&u.x) X {2}; // OK, creates new subobject of 'u'
  assert(p->n == 2);       // OK
  assert(*std::launder(&u.x.n) == 2); // OK 
  // undefined behavior, 'u.x' does not name new subobject
  assert(u.x.n == 2);
}
```

这里我们首先用 `1` 初始化 `u`（所以 `x.n` 被设置为 1，并且 `x` 成为联合的活动成员）。接下来，`u.f` 被赋值为 5，`f` 变为活动成员。最后，我们进行了放置新的操作，并用新值 `2` 覆盖了 `u`。这里的问题在于 `n` 是 const 限定的，优化器可以假设它将始终等于 1。因此，`std::launder` 充当了优化障碍，防止优化器执行常量传播。

这个例子在 Stack Overflow 上对问题的回答中有更详细的解释：[What is the purpose of std::launder?](http://stackoverflow.com/questions/39382501/what-is-the-purpose-of-stdlaunder)。

Alisdair Meredith 也在他的演讲“C++17 全览”中简要提到了一个现实生活中的用例，[链接](https://youtu.be/-rIixnNJM4k?t=583)。他告诉我们，当处理包含常量限定元素的容器时，这个函数可能会很方便。

最后，让我们看另一个例子。GCC 维护者最初将 `std::launder` [实现](https://gcc.gnu.org/ml/libstdc++/2016-10/threads.html#00152)为无操作，即类似于：

```
 template<typename _Tp>
   constexpr _Tp*
   launder(_Tp* __p) noexcept
   {
     return __p;
   }
```

但是 Richard Smith（Clang 的首席开发者）提出了一个例子，Clang 正确编译，但 GCC 编译错误：

```
void *operator new(size_t, void *p) { return p; }

struct A {
  virtual int f();
};

struct B : A {
  virtual int f() { new (this) A; return 1; }
};

int A::f() { new (this) B; return 2; }

int h() {
  A a;
  int n = a.f();
  int m = std::launder(&a)->f();
  return n + m;
}
```

优化器将 `h` 折叠成类似于以下内容：

```
int h() {
  return 4;
}
```

（正确编译时，此函数应返回 3）。

这个例子定义了一个类层次结构，包括两个类：`A` 和 `B`。`A` 定义了一个虚成员函数 `f`，在 `B` 中被重写。函数 `f` 在 `A` 中对 `this` 指针进行了放置新操作，并用 `B` 的新实例替换对象，然后返回 1。`B` 中的 `f` 做相反的操作，即用 `A` 的实例替换对象，然后返回 2。

函数 `h` 在堆栈上分配了一个 `A` 的实例，并两次调用 `f`。注意第二次调用是在“laundered”指针上执行的。

这个例子类似于第一个例子，但这次突变的常量值是`a`的`vptr`（即指向`A`或`B`的虚表）。GCC有一种称为“虚拟化”的优化，即优化器尝试确定每个虚拟调用中每个对象的动态类型，如果类型已知，编译器将此类虚拟调用替换为直接调用。这次优化器过于乐观，未能意识到即使是堆栈分配的对象现在也可以改变它们的动态类型。

注意，GCC中的C++17支持仍然处于实验阶段，因此可能会出现此类错误。已经发现了这个问题，并且已经存在[修复方案](https://gcc.gnu.org/ml/libstdc++/2016-10/msg00194.html)。
