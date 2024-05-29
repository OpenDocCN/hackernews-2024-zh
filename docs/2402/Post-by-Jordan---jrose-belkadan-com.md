<!--yml
category: 未分类
date: 2024-05-27 14:41:15
-->

# Post by Jordan, @jrose@belkadan.com

> 来源：[https://social.belkadan.com/@jrose/statuses/01HNRNHBQY4E5MC37KG14R50P7](https://social.belkadan.com/@jrose/statuses/01HNRNHBQY4E5MC37KG14R50P7)

[@jrose](https://social.belkadan.com/@jrose) (just for the frame: my background is computer science with a side of mathematics)

the nuance here is really what operations are allowed/available.

without any, each of the 2^32 bijections is an isomorphism so it's not unique.

with order ("<"), there is one isomorphism: 0 -> int32.min, 1 -> int32.min+1, ... this one is unique.

if you add basically anything else ("== 0" or "+"), they cease to be isomorphic.

two's complement is a bijection, but not an isomorphism for most sets of ops.