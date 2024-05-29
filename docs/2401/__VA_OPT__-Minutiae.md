<!--yml
category: 未分类
date: 2024-05-27 15:18:38
-->

# __VA_OPT__ Minutiae

> 来源：[https://www.corsix.org/content/va-opt-minutiae](https://www.corsix.org/content/va-opt-minutiae)

Let's say we're writing C code, and would like to define a macro for `printf`-with-newline. We might start with:

```
#define printfln(fstr, ...) \
  printf(fstr "\n", __VA_ARGS__) 
```

So far so good; we can write `printfln("1+1 is %d", 1+1)` and it'll print `1 + 1 is 2` followed by a newline. However, simpler cases such as `printfln("Hello")` result in a syntax error, as this macro expands to `printf("Hello" "\n",)`, which is invalid due to the trailing comma.

**The non-standard solution**

One conventional solution is to rely on a non-standard language extension whereby inserting `##` between `,` and `__VA_ARGS__` has a *special effect*:

```
#define printfln(fstr, ...) \
  printf(fstr "\n", ## __VA_ARGS__) 
```

With this, both `printfln("1+1 is %d", 1+1)` and `printfln("Hello")` work fine. However, combining the two into something like `printfln("Wrote %d chars", printfln("Hello"))` results in a mysterious error. To figure out why, we need to look into exactly what *special effect* this non-standard language extension has. The [GNU C Preprocessor documentation on Variadic Macros](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/cpp/Variadic-Macros.html) says:

> The `##` token paste operator has a special meaning when placed between a comma and a variable argument. If you write [...] and the variable argument is left out [...], then the comma before the `##` will be deleted. This does not happen if you pass an empty argument, nor does it happen if the token preceding `##` is anything other than a comma.

Meanwhile, the [GCC documentation on Variadic Macros](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/Variadic-Macros.html) says:

> If the variable arguments are omitted or empty, the `##` operator causes the preprocessor to remove the comma before it. If you do provide some variable arguments in your macro invocation, GNU CPP does not complain about the paste operation and instead places the variable arguments after the comma. Just like any other pasted macro argument, these arguments are not macro expanded.

The last sentence of this 2^(nd) piece of documentation tells us what is going on: macro arguments are *usually* expanded before being substituted, but this *doesn't* happen to macro arguments used as operands to `#` or `##`. In either case, *after* substitution has been performed, the resultant tokens are rescanned for more expansions, albeit with the original macro deactivated during this rescan. The only major observable difference between expansion-before-substitution and expansion-during-rescan is that the original macro is active in the former but deactivated in the latter. Hence `printfln("Wrote %d chars", printfln("Hello"))` expands to `printf("Wrote %d chars" "\n", printfln("Hello"))` without expanding the inner `printfln`, which the compiler then tries to parse as a call to the non-existent *function* `printfln`.

The two pieces of documentation are contradictory as to what happens if the variable arguments are present but empty (as in `printfln("Hello",)`). In practice the 1^(st) piece of documentation is correct; the comma is kept if the variable arguments are present but empty.

The *nor does it happen if the token preceding `##` is anything other than a comma* phrase from the 1^(st) piece of documentation is interesting, as it turns out this is a slightly dynamic property: comma deletion can happen for things like `, ## x ## __VA_ARGS__` provided that `x` expands to nothing. Clang will also delete the comma in `, x ## __VA_ARGS__` when `x` expands to nothing. Things also seem to get funky if more pasting immediately follows `, ## __VA_ARGS__`. As examples:

| `#define F(x, ...)` as | `F()` | `F(1)` | `F(,)` | `F(1,)` | `F(1,2)` |
| `,##__VA_ARGS__` | empty | empty | `,` | `,` | `,2` |
| `,##__VA_ARGS__ x` | empty | `1` | `,` | `,1` | `,2 1` |
| `,##__VA_ARGS__##x` | `,` | error | `,` | error | error† or `,21`‡ |
| `,##x##__VA_ARGS__` | empty | error | `,` | error | error |
| `,x##__VA_ARGS__` | `,`† or empty ‡ | `,1` | `,` | `,1` | `,12` |

† According to gcc 13.2.
‡ According to clang 17.0.1.

In the cases where gcc and clang differ, who is to say which is correct? After all, there is no standard documenting the desired behaviour for non-standard language extensions.

**Enter `__VA_OPT__`**

Rather than standardise this mess, the [C](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3033.htm) and [C++](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0306r4.html) languages adopted a different solution, namely `__VA_OPT__`. With `__VA_OPT__`, our motivating example looks like:

```
#define printfln(fstr, ...) \
  printf(fstr "\n" __VA_OPT__(,) __VA_ARGS__) 
```

In this, `__VA_OPT__(,)` expands to nothing if the variable arguments are absent or empty or become empty after macro expansion, whereas it expands to `,` otherwise. There's no token pasting going on, so `printfln("Wrote %d chars", printfln("Hello"))` now works. The other behavioural difference is that the odd-looking `printfln("Hello",)` now expands to the valid `printf("Hello" "\n")`, as does `printfln("Hello", EMPTY)` in the context of `#define EMPTY /*nothing*/`.

The standard didn't stop there though; there can be more than just a `,` within `__VA_OPT__`. Any `(`-`)`-balanced token sequence is allowed, and it can even access the arguments of the macro invocation, so for example it is valid to have:

```
#define M(x, ...) (0 __VA_OPT__(-(x)) ) 
```

Then `M(1)` expands to `(0)` and `M(1,2)` expands to `(0 - (1))`.

**What about whitespace?**

Compilers seem to disagree on the behaviour of whitespace just after `__VA_OPT(` or just before the matching `)`. Consider:

```
#define TILDAS(...) ~__VA_OPT__( ~ )~
#define S2(x) #x
#define S1(x) S2(x)
const char* s = S1(TILDAS());
const char* sa = S1(TILDAS(a)); 
```

The observed results are:

|  | gcc 13.2 | clang 17.0.1 |
| s | `"~~"` | `"~ ~"` |
| sa | `"~~~"` | `"~ ~ ~"` |

One interpretation is that gcc is correct, based on this paragraph from the standard: ([N3096](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3096.pdf) 6.10.4.1¶7)

> The preprocessing token sequence for [...] a va-opt-replacement [...] consists of the results of the expansion of the contained pp-tokens as the replacement list of the current function-like macro before removal of placemarker tokens, rescanning, and further replacement.

Combined with an earlier paragraph about replacement lists: ([N3096](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3096.pdf) 6.10.4¶7)

> [...] Any white-space characters preceding or following the replacement list of preprocessing tokens are not considered part of the replacement list for either form of macro.

**What does `__VA_OPT__()` expand to?**

The short answer should be *nothing*, based on the rules for expanding it:

1.  If the variable arguments are absent or empty or become empty after macro expansion, the expansion of `__VA_OPT__()` is a single placemarker token.
2.  Otherwise, if used as an operand of `##`, the expansion of `__VA_OPT__()` is a single placemarker token (because an empty expansion becomes a placemarker in this context).
3.  Otherwise, the expansion of `__VA_OPT__()` is empty (though a single placemarker token would be equally valid, as it'll evaporate away in due course).

`#__VA_OPT__()` therefore becomes an obfuscated way of writing `""`. Meanwhile, `__VA_OPT__()` as an operand of `##` can be used to force the *other* operand to be in a `##` context. For example, the undesirable behaviour of `, ## __VA_ARGS__` can be reproduced via:

```
#define printfln(fstr, ...) \
  printf(fstr "\n" __VA_OPT__(,) __VA_OPT__() ## __VA_ARGS__) 
```

If `__VA_OPT__()` expands to nothing, regardless of whether the variable arguments are absent or empty or become empty after macro expansion, then it might seem rational to optimise it away. There's a corner case though: determining whether the variable arguments become empty after macro expansion requires macro expanding them, and macro expansion of the (non-standard) `__COUNTER__` macro has visible side-effects. Consider:

```
#define EMPTY_VA_OPT(...) __VA_OPT__()
int x = __COUNTER__;
EMPTY_VA_OPT(__COUNTER__)
int y = __COUNTER__;
return y - x; 
```

For the above, gcc 13.2 returns 2, whereas clang 17.0.1 returns 1, indicating that clang optimised away the `__VA_OPT__()`.

**Token pasting and `__VA_OPT__`**

The standard is careful to say that the expansion of `__VA_OPT__` can contain placemarker tokens: ([N3096](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3096.pdf) 6.10.4.1¶7, emphasis mine)

> The preprocessing token sequence for [...] a va-opt-replacement [...] consists of the results of the expansion of the contained pp-tokens as the replacement list of the current function-like macro **before removal of placemarker tokens**, rescanning, and further replacement.

These placemarker tokens were originally a fiction dreamt up by the standard to provide a succinct description of the semantics of `##`:

*   The operands of `##` always produce at least one token. If an operand were to produce zero tokens, it instead produces a single placemarker token.
*   If one operand of `##` is a placemarker token, the result of `##` is the other operand.
*   Placemarker tokens are removed after `##` processing and before rescan.

Before `__VA_OPT__`, it was possible to ignore this fiction, and [implement the preprocessing algorithm](https://www.spinellis.gr/blog/20060626/) with careful rules around `##` evaluation rather than producing and later removing placemarker tokens. It becomes harder to maintain this fiction with `__VA_OPT__`. Consider:

```
#define G(x, y, z, ...) 1 ## __VA_OPT__(x y ## y z) ## 5 
```

Then `G(,,)` expands to the single token `15`, while `G(2,3,4,-)` expands to the three tokens `12` `33` `45`. If `y` is empty, then the inner `y ## y` should produce a placemarker, and then things get interesting:

| Expansion of | Result | Notes |
| `G(2,,4,-)` | `12` `45` | The `__VA_OPT__` expands to `2``4`. |
| `G( ,,4,-)` | `1` `45` | The placemarker inhibits merging to `145`. |
| `G(2,, ,-)` | `12` `5` | The placemarker inhibits merging to `125`. |
| `G( ,, ,-)` | `1` `5`† or `15`‡ | The correct result is the merged `15`. |

† According to gcc 13.2.
‡ According to clang 17.0.1.

The other fun observation is that macro arguments within `__VA_OPT__` get macro expanded, even if the `__VA_OPT__` itself is an operand of `##`. Consider:

```
#define H1(...) x ##            __VA_ARGS__
#define H2(...) x ## __VA_OPT__(__VA_ARGS__) 
```

Then `H1(__LINE__)` expands to `x__LINE__`, whereas `H2(__LINE__)` expands to something like `x3`.

**Further reading**

*   [P1042R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1042r1.html) tweaked the `__VA_OPT__` semantics slightly a few years ago.
*   `__VA_OPT__` can be (ab)used for [recursive macros](https://www.scs.stanford.edu/~dm/blog/va-opt.html).