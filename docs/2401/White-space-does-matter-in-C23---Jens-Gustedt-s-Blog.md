<!--yml
category: 未分类
date: 2024-05-27 14:53:29
-->

# White space does matter in C23 – Jens Gustedt's Blog

> 来源：[https://gustedt.wordpress.com/2024/01/16/white-space-does-matter-in-c23/](https://gustedt.wordpress.com/2024/01/16/white-space-does-matter-in-c23/)

Usually in C identifiers are not directly followed by strings. But when `U` prefixed literals were introduced in C. there still were some rare clashes with existing code. This happened were a macro `U` that expanded to a string was used to add some sort of leading character sequence to a string. Prior, this usage was not sensible to whether or not there was a space between the two. By introducing the prefix the two usages (with and without space) became distinct and code changed its meaning or became invalid. So for this situation, space is in fact already significant.

Generally, it is often assumed that in C spaces don’t contribute much to the interpretation of programming text, but for C23 this has become an over-simplification that does not reflect the situation anymore.

In addition to problems internal to C, there is a problem of interfacing with C++, where some of the rules are different.

| syntax | meaning, C23 | different meaning, C++ |
| --- | --- | --- |
| `# define X(A)` | function like macro, empty |  |
| `# define X (A)` | object macro, expands to `(A)` |  |
| `0x4'7'a` | hex number with digit separators |  |
| `0x4 '7'a` | number, character literal, and identifier | number, character literal with suffix |
| `0x4 '7' a` | number, character literal, and identifier |  |
| `"%" PRIx64` | valid format string for `printf` |  |
| `"%"PRIx64` | valid format string for `printf` | string literal with suffix |
| `R "(hör)"` | identifier followed by multi-byte string |  |
| `R"(hör)"` | identifier followed by multi-byte string | raw multi-byte string, contains just `hör` |
| `R "hör"` | identifier followed by multi-byte string |  |
| `R"hör"` | identifier followed by multi-byte string | invalid raw string |
| `U "hör"` | identifier, followed by multi-byte string |  |
| `U"hör"` | UTF-32 string |  |

It would be good if there could be some coordination here between C and C++ about such simple questions concerning lexing and the preprocessor.

Any use of identifiers that are adjacent to character and string literals should be discouraged; I think that such constructs should just be excluded by the syntax. If we want this to be diagnosed it should be before compilation phase 4, in particular before macro expansion. Best would be if this is diagnosed in phase 3, lexing.

Compilers and preprocessor implementations could start to warn about such possible collision immediately, as of today. I don’t see much of a reason to still allow it. Even where there are valid uses, such as for the `printf` format string macros, the code becomes much easier to read if these are separated from the surrounding format string.