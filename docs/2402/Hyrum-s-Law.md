<!--yml

分类：未分类

日期：2024-05-27 14:57:46

-->

# Hyrum的法则

> 来源：[https://www.hyrumslaw.com](https://www.hyrumslaw.com)

# Hyrum的法则

*关于软件工程的观察*

简而言之，这一观察是：

> 通过足够数量的API用户，
> 
> 在合同中承诺什么并不重要：
> 
> 你系统的所有可观测行为
> 
> 将会被某人依赖。

在过去几年里，在地球上最复杂的软件系统之一进行了低级基础设施迁移，我对接口和其实现之间的差异做了一些观察。我们通常认为接口是与系统交互的抽象（就像汽车中的方向盘和踏板），实现是系统完成工作的方式（车轮和发动机）。这是有用的出于很多原因，其中首要的是，大多数有用的系统很快变得过于复杂，以至于单个个人或团队无法完全了解，而抽象对于管理这种复杂性至关重要。

确定正确的抽象级别是一个完全独立的讨论（参见《人月神话》），但我们认为一旦定义了一个抽象，它就是具体的。换句话说，一个接口理论上应该提供一个清晰的分隔，使系统的消费者和实现者之间达到隔离。实际上，当一个系统的使用增长并且其用户开始有意识地依赖通过接口故意暴露或通过常规使用推断出的实现细节时，这一理论就会变得不成立。Spolsky的“漏洞抽象法则”体现了使用者对内部实现细节的依赖。

如果推到逻辑的极致，这就得出了以下观察，俗称“隐式接口法则”：在使用足够多的情况下，私有实现就不复存在。也就是说，如果一个接口有足够的使用者，他们将共同依赖于实现的每个方面，不管是有意还是无意。这种作用限制了对实现的更改，现在必须符合明确记录的接口以及由使用捕获的隐式接口。我们经常将这种现象称为“bug对bug兼容性”。

隐式接口的创建通常是逐渐发生的，接口使用者通常并不知道正在发生什么。例如，一个接口可能对性能不做任何保证，但使用者经常期望其实现达到一定的性能水平。这些期望成为了对系统的隐式接口的一部分，系统的变化必须维持这些性能特征，以继续为其使用者提供服务。

并非所有的消费者都依赖相同的隐式接口，但是随着消费者的增多，隐式接口最终会精确地匹配实现。在这一点上，接口已经消失：实现已成为接口，对其任何更改都将违反消费者的期望。有些运气的话，广泛的、全面的、自动化的测试可以检测到这些新的期望，但不能减轻它们。

隐式接口是大型系统有机发展的结果，虽然我们希望这个问题不存在，但设计师和工程师在构建和维护复杂系统时应该考虑到它。因此，请注意隐式接口如何限制您的系统设计和演进，并了解对于任何相当流行的系统来说，接口影响的深度远远超出您的想象。

### 海伦姆是谁？

我是谷歌的软件工程师，负责大规模代码变更工具和基础设施。在此之前，我花了五年时间改进谷歌的核心C++库。上述观察源于当时即使是最简单的库变更也会导致某些远程系统的失败经验。

虽然可能是我做出了这一观察，但功劳应归于Titus Winters，他将其正式命名为“海伦姆定律”并广泛普及了这一概念。

### 历史

必看的XKCD漫画：[https://xkcd.com/1172/](https://xkcd.com/1172/)
