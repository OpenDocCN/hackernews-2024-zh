<!--yml
category: 未分类
date: 2024-05-27 14:56:54
-->

# memory leak proof every C program

> 来源：[https://flak.tedunangst.com/post/memory-leak-proof-every-C-program](https://flak.tedunangst.com/post/memory-leak-proof-every-C-program)

Memory leaks have plagued C programs for as long as the language has existed. Many solutions have been proposed, even going so far as to suggest we should rewrite C programs in other languages. But there’s a better way.

Presented here is a simple solution that will eliminate the memory leaks from every C program. Link this into your program, and memory leaks are a thing of the past.

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
Every allocated pointer is saved in the big bucket, where it remains accessible. Even if no other references to the pointer exist in the program, the pointer has not leaked.

It is now entirely optional to call `free`. If you don’t call free, memory usage will increase over time, but technically, it’s not a leak. As an optimization, you may choose to call free to reduce memory, but again, strictly optional.

Problem sovled!

Posted 19 Jan 2024 16:55 by tedu Updated: 19 Jan 2024 16:55

Tagged:

[c](/t/c) [programming](/t/programming) [rants](/t/rants)