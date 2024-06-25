<!--yml

类别：未分类

日期：2024-05-27 15:08:44

-->

# Babashka

> 来源：[https://babashka.org/](https://babashka.org/)

### 瞬间启动

利用 GraalVM native-image 和 Small Clojure Interpreter，babashka 是一个自包含、瞬间启动的脚本环境。

Babashka 自带脚本编写工具：tools.cli, cheshire, babashka.fs, babashka.process, java.time 以及许多其他库和类。

### 跨平台

Babashka 脚本可以在 Linux、macOS 和 Windows 上运行。

除了内置库外，babashka 还能从源代码加载库，利用已有的 Clojure 库世界。

### 多线程

Babashka 支持真实的 JVM 线程，并且像 Clojure 一样支持 futures 和动态线程局部绑定变量。

Babashka 特色内置任务运行器，涵盖了 make、just 和 npm 脚本的最常见用例。

Babashka 可以像在 bash 中那样调用其他 CLI 程序。它更进一步，通过 pod 协议无缝集成其他二进制文件。Pods 可以用任何语言实现，包括 Clojure、Rust 和 Go。
