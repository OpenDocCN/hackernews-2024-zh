<!--yml

category: 未分类

date: 2024-05-27 14:56:54

-->

# memory leak proof every C program

> 来源：[https://flak.tedunangst.com/post/memory-leak-proof-every-C-program](https://flak.tedunangst.com/post/memory-leak-proof-every-C-program)

自从C语言诞生以来，内存泄漏一直困扰着C程序。已经有许多解决方案被提出，甚至有人建议我们应该用其他语言重写C程序。但有一种更好的方法。

这里提供了一种简单的解决方案，可以消除每个C程序中的内存泄漏。将此链接到您的程序中，内存泄漏将成为过去时。

<details><summary>leakproof.c</summary>

```
#include <dlfcn.h>
#include <stdio.h>

struct leaksaver {
        struct leaksaver *next;
        void *pointer;
} *bigbucket;

void *
malloc(size_t len)
{
        static void *(*nextmalloc)(size_t);
        nextmalloc = dlsym(RTLD_NEXT, "malloc");
        void *ptr = nextmalloc(len);
        if (ptr) {
                struct leaksaver *saver = nextmalloc(sizeof(*saver));
                saver->pointer = ptr;
                saver->next = bigbucket;
                bigbucket = saver;
        }
        return ptr;
}
```</details>

每个分配的指针都保存在大桶中，仍然可以访问。即使程序中没有对指针的其他引用，指针也不会泄漏。

现在完全可以选择不调用`free`。如果您不调用free，内存使用量将随时间增加，但从技术上讲，这不是泄漏。作为优化，您可以选择调用free以减少内存，但再次强调，这是完全可选的。

问题已解决！

由tedu于2024年1月19日16:55发布 更新：2024年1月19日16:55

Tagged:

[c](/t/c) [programming](/t/programming) [rants](/t/rants)
