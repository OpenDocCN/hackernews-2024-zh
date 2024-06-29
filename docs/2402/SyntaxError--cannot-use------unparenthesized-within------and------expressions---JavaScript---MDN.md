<!--yml

category: 未分类

date: 2024-05-29 13:20:57

-->

# SyntaxError: 在 JavaScript 中，不能在 `||` 和 `&&` 表达式中未括号化使用 `??`。

> 来源：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_use_nullish_coalescing_unparenthesized](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_use_nullish_coalescing_unparenthesized)

[运算符优先级](/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence)链如下所示：

```
|   >   &&   >   ||   >   =
|   >   ??   >   =

```

然而，`??` 与 `&&`/`||` 之间的优先级*故意*未定义，因为逻辑运算符的短路行为可能使表达式的评估反直觉。因此，以下组合都是语法错误，因为语言不知道如何给操作数加括号：

```
a ?? b || c;
a || b ?? c;
a ?? b && c;
a && b ?? c; 
```

相反，通过明确括号化每一侧来明确你的意图：

```
(a ?? b) || c;
a ?? (b && c); 
```
