<!--yml
category: 未分类
date: 2024-05-27 14:30:54
-->

# Dennis Ritchie on & | vs. ==

> 来源：[https://www.lysator.liu.se/c/dmr-on-or.html](https://www.lysator.liu.se/c/dmr-on-or.html)

From decvax!harpo!npoiv!alice!research!dmr Fri Oct 22 01:04:10 1982
Subject: Operator precedence
Newsgroups: net.lang.c

The priorities of `&& ||` vs. `==` etc. came about in the following way.

Early C had no separate operators for `&` and `&&` or `|` and `||`. (Got that?) Instead it used the notion (inherited from [B](msb-on-b.html#main) and [BCPL](alan-on-bcpl.html#bcpl-main)) of "truth-value context": where a Boolean value was expected, after "if" and "while" and so forth, the & and | operators were interpreted as && and || are now; in ordinary expressions, the bitwise interpretations were used. It worked out pretty well, but was hard to explain. (There was the notion of "top-level operators" in a truth-value context.)

The precedence of `&` and `|` were as they are now.

Primarily at the urging of Alan Snyder, the `&&` and `||` operators were added. This successfully separated the concepts of bitwise operations and short-circuit Boolean evaluation. However, I had cold feet about the precedence problems. For example, there were lots of programs with things like

```
	if (a==b & c==d) ...

```

In retrospect it would have been better to go ahead and change the precedence of `&` to higher than `==`, but it seemed safer just to split `&` and `&&` without moving `&` past an existing operator. (After all, we had several hundred kilobytes of source code, and maybe 3 installations....)

[Dennis Ritchie](http://www.cs.bell-labs.com/who/dmr/index.html)