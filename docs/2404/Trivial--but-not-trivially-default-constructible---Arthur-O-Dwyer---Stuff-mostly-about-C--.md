<!--yml

category: 未分类

date: 2024-05-27 13:02:07

-->

# Trivial，但不是trivially default constructible – Arthur O'Dwyer – Stuff mostly about C++

> 来源：[https://quuxplusone.github.io/blog/2024/04/02/trivial-but-not-default-constructible/](https://quuxplusone.github.io/blog/2024/04/02/trivial-but-not-default-constructible/)

我刚刚看完Jason Turner的CppCon 2023演讲[“Great C++ `is_trivial`.”](https://www.youtube.com/watch?v=bpF1LKQBgBQ) 大约在[41m00s](https://www.youtube.com/watch?v=bpF1LKQBgBQ&t=41m)，他挑剔[[class.prop]/2](https://eel.is/c++draft/class.prop#def:class,trivial)的措辞：

> 一个**trivial class**是一个既可以trivially复制，又有一个或多个合格的默认构造函数的类。

“那个措辞总是让我困扰。你怎么可能有多个默认构造函数？它们可能会受到`requires`子句的约束……但那么其中不超过一个是[eligible](https://eel.is/c++draft/special#def:special_member_function,eligible)。”

这里有个诀窍：Jason，你错误地假设一个trivial类必须在所有情况下都是默认构造的！[Godbolt](https://godbolt.org/z/EGjb17of3)：

```
template<class T>
struct S {
    S() requires (sizeof(T) > 3) = default;
    S() requires (sizeof(T) < 5) = default;
};

static_assert(std::is_trivial_v<S<int>>);
static_assert(not std::is_default_constructible_v<S<int>>); 
```

在这里，`S<int>`是trivial的，但由于它有*两个*合格的默认构造函数，任何尝试默认构造一个`S<int>`都将因为重载决议的歧义而失败。因此，`S<int>`根本不是默认构造的（当然也不是*trivially* default-constructible）；但它仍然满足*trivial class*的所有要求。

* * *

顺便说一句，我觉得当讲解triviality时，Jason有点掩盖要点。规则是：类型`T`是*trivially fooable*，当且仅当编译器知道它的*foo*操作等价于相同的`T`对象表示（即，一个`unsigned char`数组）。`T`是trivially copy-constructible，如果它的复制构造函数仅仅复制它的字节；如果它的默认构造函数仅仅默认初始化它的字节（回想一下，默认初始化一个`unsigned char`数组是无操作的）；它是trivially value-initializable（见[“PSA: Value-initialization is not merely default-construction”](/blog/2023/06/22/psa-value-initialization/)（2023-06-22）），如果它的值初始化仅仅是值初始化它的字节（为零）；等等。*这*就是关于副词“trivially”的大事情——它意味着“仿佛在对象表示上”。
