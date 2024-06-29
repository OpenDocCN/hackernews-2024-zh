<!--yml

category: 未分类

date: 2024-05-27 14:40:57

-->

# RustPython

> 来源：[https://rustpython.github.io/](https://rustpython.github.io/)

# 为什么选择 RustPython？

Python 有许多实现方式。例如：

这些实现方式各有其优点：例如，Jython 将 Python 2 源代码编译为 Java 字节码，然后路由到 Java 虚拟机。因为 Python 代码被翻译为 Java 字节码，在运行时它看起来和感觉像一个真正的 Java 程序，因此与 Java 应用程序集成良好。

IronPython 与 .NET 高度集成，这意味着 IronPython 可以使用 .NET 框架和 Python 2 库，反之亦然。

我们希望解锁与 Jython 和 IronPython 相同的可能性，但适用于 Rust 编程语言。此外，由于 Rust 的最小运行时，我们能够将 RustPython 编译为 WebAssembly 并允许用户轻松在浏览器中运行他们的 Python 代码。
