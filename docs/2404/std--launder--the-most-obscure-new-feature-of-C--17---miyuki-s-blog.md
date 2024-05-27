<!--yml
category: 未分类
date: 2024-05-27 12:54:06
-->

# std::launder: the most obscure new feature of C++17 · miyuki's blog

> 来源：[https://miyuki.github.io/2016/10/21/std-launder.html](https://miyuki.github.io/2016/10/21/std-launder.html)

# std::launder: the most obscure new feature of C++17

21 Oct 2016

Proposal [P0137R1](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0137r1.html) introduces a new function into the C++ standard library called `std::launder`. Unfortunately, this proposal is hard to understand for mere mortals.

This is what Botond Ballo writes about `std::launder` in his [trip report](https://botondballo.wordpress.com/2016/07/06/trip-report-c-standards-meeting-in-oulu-june-2016/) to the C++ Standards Commitee meeting:

> Don’t ask. If you’re not one of the 5 or so people in the world who already know what this is, you don’t want or need to know.

But nevertheless let’s try to get some intuition about how `std::launder` works. The original proposal provides an example:

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

Here we first initialize `u` with `1` (so `x.n` is set to 1, and `x` becomes the active member of the union). Next, 5 is assigned to `u.f` and `f` becomes active. Finally, we do a placement new and overwrite `u` with the new value 2\. The problem here is that `n` is const-qualified and the optimizer is allowed to assume that it will always remain equal to 1\. So, `std::launder` acts as an optimization barrier that prevents the optimizer from performing constant propagation.

This example is explained in more detail in the answer to a question on Stack Overflow: [What is the purpose of std::launder?](http://stackoverflow.com/questions/39382501/what-is-the-purpose-of-stdlaunder).

Alisdair Meredith also briefly [mentions](https://youtu.be/-rIixnNJM4k?t=583) a real-life use case in his talk “C++17 in Breadth” from CppCon 2016\. He tells us that this function might be handy when dealing with containers of const-qualified elements.

Finally, let’s look at another example. The GCC maintainers initially [implemented](https://gcc.gnu.org/ml/libstdc++/2016-10/threads.html#00152) `std::launder` as a no-op, i.e. something similar to:

```
 template<typename _Tp>
   constexpr _Tp*
   launder(_Tp* __p) noexcept
   {
     return __p;
   }
```

but Richard Smith (the lead developer of Clang) came up with an example which is compiled correctly by Clang, but miscompiled by GCC:

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

The optimizer folds `h` into something similar to:

```
int h() {
  return 4;
}
```

(when compiled correctly, this function should return 3).

This example defines a class hierarchy consisting of two classes: `A` and `B`. `A` defines a virtual member function `f` which is overridden in `B`. Function `f` in `A` does a placement new on the `this` pointer and replaces the object with a newly created instance of `B` and then returns 1\. `f` in `B` does the opposite, i.e. replaces the object with an instance of `A` and then returns 2.

Function `h` allocates an instance of `A` on the stack and invokes `f` twice. Note that the second call is performed on the “laundered” pointer.

This example resembles the first one, but this time the mutated const value is the `vptr` of `a` (i.e., a pointer to the vtable of either `A` or `B`). GCC has an optimization called “devirtualization”, i.e. the optimizer tries to figure out the dynamic type of each object in each virtual call and if the type is known, the compiler replaces such virtual call with a direct call. This time the optimizer is overly optimistic and fails to recognize that even stack-allocated objects can now change their dynamic type.

Note that C++17 support in GCC is still experimental, and such bugs are expected. This one has been spotted and a [fix](https://gcc.gnu.org/ml/libstdc++/2016-10/msg00194.html) already exists.