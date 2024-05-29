<!--yml
category: 未分类
date: 2024-05-29 13:20:57
-->

# SyntaxError: cannot use `??` unparenthesized within `||` and `&&` expressions - JavaScript | MDN

> 来源：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_use_nullish_coalescing_unparenthesized](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_use_nullish_coalescing_unparenthesized)

The [operator precedence](/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence) chain looks like this:

```
|   >   &&   >   ||   >   =
|   >   ??   >   =

```

However, the precedence *between* `??` and `&&`/`||` is intentionally undefined, because the short circuiting behavior of logical operators can make the expression's evaluation counter-intuitive. Therefore, the following combinations are all syntax errors, because the language doesn't know how to parenthesize the operands:

```
a ?? b || c;
a || b ?? c;
a ?? b && c;
a && b ?? c; 
```

Instead, make your intent clear by parenthesizing either side explicitly:

```
(a ?? b) || c;
a ?? (b && c); 
```