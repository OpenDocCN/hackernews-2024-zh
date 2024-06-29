<!--yml

category: 未分类

date: 2024-05-27 14:55:19

-->

# Erlang/OTP 27.0 Release Candidate 1 - Erlang/OTP

> 来源：[https://www.erlang.org/news/167](https://www.erlang.org/news/167)

## OTP 27.0-rc1 [#](#otp-270-rc1)

Erlang/OTP 27.0-rc1是在OTP 27.0发布之前的三个发布候选版本中的第一个。

此版本的目的是从用户那里获得反馈。我们欢迎任何反馈，即使只是告诉我们它对您有效。我们鼓励用户尝试并通过在[https://github.com/erlang/otp/issues](https://github.com/erlang/otp/issues)上创建问题或在[Erlang Forums](https://erlangforums.com/)上发布来提供反馈。

所有发布版本的构件均可从[Erlang/OTP Github](https://github.com/erlang/otp/releases/tag/OTP-27.0-rc1)下载，您可以在[https://erlang.org/documentation/doc-15.0-rc1/doc](https://erlang.org/documentation/doc-15.0-rc1/doc/)查看新文档。您还可以像这样使用[kerl](https://github.com/kerl/kerl)安装最新版本：

```
kerl build 27.0-rc1 27.0-rc1. 
```

Erlang/OTP 27是一个新的主要发布版本，具有新功能、改进以及一些不兼容性。下面突出显示了一些新功能。

衷心感谢所有贡献者！

# 亮点 [#](#highlights)

## 文档 [#](#documentation)

[EEP-59](https://www.erlang.org/eeps/eep-0059)已经实现。现在可以在源文件中使用文档属性来记录函数、类型、回调函数和模块。

整个Erlang/OTP文档现在使用新文档系统。

## 新语言特性 [#](#new-language-features)

+   三引号字符串已根据[EEP 64](https://www.erlang.org/eeps/eep-0064)实现，允许字符串包含整段文字。

+   相邻的字符串文字没有中间空白现在是语法错误，以避免可能与三引号字符串混淆。

+   字符串文字（普通和三引号）上的Sigils已根据[EEP 66](https://www.erlang.org/eeps/eep-0066)实现。例如，`~"Björn"`或`~b"Björn"`现在等效于`<<"Björn"/utf8>>`。

## 编译器和JIT改进 [#](#compiler-and-jit-improvements)

+   编译器现在将合并同一记录的连续更新。

+   编译器和运行时系统中已实现安全破坏性元组更新。这使得VM可以在安全的情况下原地更新元组，从而通过减少复制和产生更少垃圾来提高性能。

+   `maybe` 表达式现在默认启用，不再需要启用 `maybe_expr` 特性。

+   JIT中已实现原生覆盖支持。 `cover` 工具将自动使用它来减少运行覆盖编译代码时的执行开销。还有新的API支持原生覆盖，而不使用 `cover` 工具。

+   编译器现在会在更新记录/映射文字时引发警告，以捕捉常见错误。例如，编译器现在会对`#r{a=1}#r{b=2}`发出警告。

## ERTS [#](#erts)

+   现在`erl`命令支持`-S`标志，类似于`-run`标志，但修正了一些粗糙的边缘。

+   默认情况下，现在将编译 escript 脚本而不是解释。这意味着必须安装`compiler`应用程序。

+   默认进程限制已提升至`1048576`个进程。

+   现在可以通过新函数`erlang:system_monitor/2`监控系统中的长消息队列。

+   已删除通过将原子（或字符串）作为`open_port()`的第一个参数来打开到外部资源的端口的过时且未记录的支持，该功能自 OTP 26 发布以来已计划在 OTP 27 中删除。

+   `erlang:fun_info/1,2`中的`pid`字段已被移除。

+   现在支持多个跟踪会话。

## STDLIB [#](#stdlib)

+   向模块`timer`添加了几个接受 fun 的新函数。

+   函数`is_equal/2`、`map/2`和`filtermap/2`已添加到模块`sets`、`ordsets`和`gb_sets`中。

+   新的有效`ets`遍历函数，保证原子性。例如，现在可以将`ets:next/2`后跟`ets:lookup/2`替换为`ets:next_lookup/1`。

+   新函数`ets:update_element/4`类似于`ets:update_element/3`，但在没有带有该键的先前记录时，将以默认元组作为第四个参数插入。

+   `binary:replace/3,4`现在支持使用 fun 作为替换二进制的提供者。

+   新函数`proc_lib:set_label/1`可以用于为任何没有注册名称的进程添加描述性术语。此名称将显示在诸如`c:i/0`和`observer`等工具中，并包含在使用`gen_server`、`gen_statem`、`gen_event`和`gen_fsm`的进程生成的崩溃报告中。

+   添加了从`gb_trees`和`gb_sets`检索下一个更高或更低键/元素的函数，以及返回以给定键/元素开始的迭代器的功能。

## common_test [#](#common_test)

+   调用`ct:capture_start/0`和`ct:capture_stop/0`现在是同步的，以确保捕获所有输出。

+   如果浏览器偏好，默认的CSS现在将包括基本的深色模式处理。

## crypto [#](#crypto)

+   在 Erlang/OTP 25 中标记为已弃用的函数`crypto_dyn_iv_init/3`和`crypto_dyn_iv_update/3`已被移除。

## dialyzer [#](#dialyzer)

+   Dialyzer的`--gui`选项已删除。

## ssl [#](#ssl)

+   `ssl`客户端现在可以协商和处理证书状态请求（客户端侧的OCSP stapling支持）。

+   新工具`tprof`结合了`eprof`和`cprof`的功能在一个界面下。它还添加了堆分析。

## xmerl [#](#xmerl)

+   作为`xmerl_xml`的替代方案，新增了一个名为`xmerl_xml_indent`的导出模块，它提供了即插即用的缩进输出功能。

更多关于新功能和潜在不兼容性的详细信息，请参阅[README](https://erlang.org/download/otp_src_27.0-rc1.readme)。
