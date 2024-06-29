<!--yml

类别：未分类

日期：2024-05-27 13:19:11

-->

# TS | Lions OS: 安全 – 快速 – 可适应

> 来源：[https://trustworthy.systems/publications/papers/Heiser_24:eo.abstract](https://trustworthy.systems/publications/papers/Heiser_24:eo.abstract)

<main id="page-content">

# Lions OS：安全 – 快速 – 可适应

## 作者

[Gernot Heiser](/people/?cn=Gernot+Heiser)

计算机科学与工程学院

UNSW,

悉尼 2052，澳大利亚

## 摘要

seL4 是世界上最安全的操作系统（OS）内核，但它与操作系统还有很大差距。作为微内核，它提供了安全地复用硬件的核心机制，但没有应用程序员期望的任何服务，因此采用者被迫自行开发这些服务。此外，设计在 seL4 上实现高性能系统需要高水平的专业知识。结果往往是设计不佳，人们放弃的情况太多。

澳大利亚新南威尔士大学（UNSW）的可信系统团队目前已决定构建 Lions OS，这是一个完整的基于 seL4 的操作系统，旨在支持物联网、物理计算和其他嵌入式系统开发者的需求。Lions OS 的命名灵感来自 John Lions（以 Lions Book 而闻名，可以说是开源运动的奠基者之一），从头开始设计和实现，旨在实现高性能、高安全性，并适应广泛的用例。具体而言，我们计划证明其关键组件的正确实现。

在演讲中，我解释了为什么我认为这不仅是可以实现的，而且将在2-3年内实现，严格遵循古老的工程原则 KISS —— keep it simple, stupid! 我将展示我们的初步结果，表明通过正确的设计，这种简洁不仅可以实现出色的性能（超越 Linux 网络），还可以实现实现正确性的证明。这次演讲标志着 Lions OS 的公开发布 —— 毋庸置疑，它是开源的。

## BibTeX 条目

```
  @inproceedings{Heiser_24:eo,
    address          = {Gladstone, QLD, AU},
    author           = {Gernot Heiser},
    booktitle        = {Everything Open},
    month            = apr,
    organization     = {Linux Australia},
    title            = {{Lions OS}: Secure -- Fast -- Adaptable},
    year             = {2024}
  }
```

## 下载

</main>
