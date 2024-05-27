<!--yml
category: 未分类
date: 2024-05-27 13:00:08
-->

# history - Did any processor implement an integer square root instruction? - Retrocomputing Stack Exchange

> 来源：[https://retrocomputing.stackexchange.com/questions/29787/did-any-processor-implement-an-integer-square-root-instruction](https://retrocomputing.stackexchange.com/questions/29787/did-any-processor-implement-an-integer-square-root-instruction)

The claim in [another answer](/a/29789) of the Quake trick being the most efficient has not been true for a long time, and was only true regarding low-quality results for floats on specific hardware. On pretty much every modern chip the native instructions are much, much faster, often a few clock cycles. (I'm the Chris Lomont that wrote an early, widely cited paper on the Quake trick, providing generalizaitions and an improvement that seems to have been copied everywhere, despite it being a terrible idea now).

A much quicker method, one used in hardware (with many more tricks), is to store a (non-equal spaced) table of values, use a quick method to pull two values, linear (or better) interpolate, shift base 2 exponent with any odd excess becoming a multiple by constant sqrt2, then, if needed, one iteration of methods better than Newton–Raphson.

Things like Halley's (and many others) converge quicker than Netwon–Raphson, and are often much faster depending on what time various operations take.

Approximations for square root on a fixed interval (since all the methods for computers are bounded) are often also faster, polynomial, Cheby stuff, Pade and higher versions, all can be done in software or hardware, depending on what tradeoffs you want.

If you only want integer, say 2^32, the same trick applies, do it in fixed point, and some not too hard analysis lets you bound tables very quickly. Another simple method used in hardware for integers is divide and conquer: each say 8 bits maps to a table of 256 fixed point values, instantly looked up in parallel, then 3 multiples (2 in parallel) give the 32 bit value (after a free truncate).

There's still plenty of research being done on speeding these up (e.g., [https://inria.hal.science/hal-03424131](https://inria.hal.science/hal-03424131)), so any technique over 10 years old is most surely obsolete for any metric: speed, power consumption, die size, etc.