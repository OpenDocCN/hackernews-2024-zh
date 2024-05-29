<!--yml
category: 未分类
date: 2024-05-27 14:40:57
-->

# RustPython

> 来源：[https://rustpython.github.io/](https://rustpython.github.io/)

# Why RustPython?

There are many implementations of Python. For example:

Each of these implementations offer some benefits: Jython, for example, compiles Python 2 source code to Java byte code, then routes it to the Java Virtual Machine. Because Python code is translated to Java bytecode, it looks and feels like a true Java program at runtime and so it integrates well with Java applications.

IronPython is well-integrated with .NET, which means IronPython can use the .NET framework and Python 2 libraries or vice versa.

We want to unlock the same possibilities that Jython and IronPython enable, but for the Rust programming language. In addition, thanks to Rusts’ minimal runtime, we’re able to compile RustPython to WebAssembly and allow users to run their Python code easily in the browser.