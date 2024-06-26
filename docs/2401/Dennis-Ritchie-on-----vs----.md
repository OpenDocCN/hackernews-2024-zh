<!--yml

类别：未分类

日期：2024-05-27 14:30:54

-->

# 丹尼斯·里奇谈& | vs. ==

> 来源：[`www.lysator.liu.se/c/dmr-on-or.html`](https://www.lysator.liu.se/c/dmr-on-or.html)

来自 decvax!harpo!npoiv!alice!research!dmr Fri Oct 22 01:04:10 1982

主题：运算符优先级

新闻组：net.lang.c

`&& ||` 与 `==` 的优先级等问题是这样产生的。

早期的 C 语言没有单独的 `&` 和 `&&` 或 `|` 和 `||` 运算符。（明白了吗？）相反，它使用了“真值上下文”的概念：在预期布尔值的地方，在“if”和“while”等之后，`&` 和 `|` 运算符被解释为现在的 `&&` 和 `||`；在普通表达式中，使用了位操作的解释。这种方式效果还不错，但解释起来很困难。（在真值上下文中有“顶层运算符”的概念。）

`&` 和 `|` 的优先级与现在一样。

在艾伦·斯奈德的督促下，`&&` 和 `||` 运算符被添加了进来。这成功地区分了位操作和短路布尔运算的概念。然而，我对优先级问题有所顾虑。例如，有很多程序的代码像这样

```
	if (a==b & c==d) ...

```

回顾起来，改变 `&` 的优先级高于 `==` 更好，但只是将 `&` 和 `&&` 分开而不将 `&` 移动到现有运算符之后似乎更安全一些。（毕竟，我们有数百千字节的源代码，可能有 3 个安装实例....）

[丹尼斯·里奇](http://www.cs.bell-labs.com/who/dmr/index.html)
