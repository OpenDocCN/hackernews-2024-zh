<!--yml

category: 未分类

日期：2024-05-27 13:33:10

-->

# 优化您的 Arm 平台程序 - 架构和处理器博客 - Arm 社区博客 - Arm 社区

> 来源：[https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/optimizing-your-programs-for-arm-platforms](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/optimizing-your-programs-for-arm-platforms)

今天的编译器在自动产生高度优化的代码方面表现相当出色。然而，作为程序员，您可以帮助编译器生成更好的代码的情况还有几种。本博客文章涵盖了一些技术和提示，这些对于创建性能更好的程序非常有用，无论您是创建 Android、桌面还是服务器应用程序。

## 内存别名和 'restrict' 关键字

每当编译器自动向量化代码时，它首先需要确保这样做是安全的。进行的安全性检查之一是指针别名检查。此检查用于查看编译器读取和写入的指针是否可能指向相同的数据。当编译器无法静态确定时，它必须插入运行时检查。

如果这些检查无法插入运行时检查，您的程序速度可能会显著减慢，甚至无法完全进行向量化。然而，作为程序员，您可以通过告诉编译器假定指针不别名的方式来解决这个问题。阅读以下 URL 中的学习路径，详细说明了在 C 中正确使用 'restrict' 关键字的重要性。

[CTAToken URL = "https://learn.arm.com/learning-paths/cross-platform/restrict-keyword-c99/" target="_blank" text="了解 restrict 关键字" class ="green"]

## 内存延迟

加载和存储数据到内存是 CPU 完成的活动，需要一定时间。这个时间取决于几个因素，但是作为程序员，有很多事情可以做来提高访问时间。通常编译器无法为您完成这些工作，因此了解这些技术可以让您的程序具备必要的优势。阅读以下 URL 中的 Memory Latency 学习路径，以更好地理解在 Arm 平台上的高速缓存、预取和数据对齐。

[CTAToken URL = "https://learn.arm.com/learning-paths/cross-platform/memory-latency/" target="_blank" text="了解内存延迟" class ="green"]

## 利用整数与浮点数

使用整数算术而不是浮点算术执行操作通常可以显著提高程序速度，因为 CPU 倾向于具有更多整数算术的带宽。然而，有些情况下，由于编程语言的语义，您可能会不经意地进行浮点运算。一个常见的陷阱是隐式转换为浮点数。阅读以下 URL 中的 Speed benefit of integer vs float 学习路径，了解如何避免这些陷阱，并利用整数性能的力量来实现更快的程序。

[CTAToken URL = "https://learn.arm.com/learning-paths/cross-platform/integer-vs-floats/" target="_blank" text="了解整数与浮点性能差异" class ="green"]

## 在编译器中利用自动向量化

现代编译器通常被称为优化编译器，因为它们对输入程序执行各种优化和转换以获得更好的性能。其中一种优化是将程序从标量转换为向量。向量化的行为指的是将程序从一次处理一个值转换为一次可以处理多个值的操作。

虽然编译器在这方面非常出色并不断改进，但仍然有各种方法可以结构化程序的流程，以便更容易地进行自动向量化并利用先进的 SIMD 和 SVE 指令的能力。

有兴趣吗？在以下网址阅读更多：

[CTAToken URL = "https://learn.arm.com/learning-paths/cross-platform/loop-reflowing/" target="_blank" text="了解如何利用自动向量化" class ="green"]

## 修改循环布局以支持自动向量化

编写自动向量化友好程序同样重要的是数据布局。在自动向量化时，编译器转换循环时，它能够顺序加载数据或者需要跳过某些元素（例如每隔一个元素加载）。即使像在数组中读取结构体中的字段，如data[i].x，也可能导致步进访问。

高效的数据布局可以决定程序是慢还是非常快。这是编译器通常无法提供足够上下文帮助的领域之一，程序员了解如何帮助编译器非常重要。有兴趣将程序性能提升到下一个级别吗？请在下面的链接中继续阅读。

[CTAToken URL = "https://learn.arm.com/learning-paths/cross-platform/vectorization-friendly-data-layout/data-layout-basics/" target="_blank" text="了解数据布局" class ="green"]

## 总结

Arm 架构具有许多出色的功能，如果正确使用，可以显著提高程序的性能。如果您记住了提示和背景知识，就很容易利用它们。

一定要阅读这里的其他学习路径：[https://learn.arm.com](https://learn.arm.com)，获取有关如何充分利用 Arm 平台提供的所有信息和提示。

[CTAToken URL = "https://learn.arm.com/" target="_blank" text="阅读更多 Arm 学习路径" class ="green"]
