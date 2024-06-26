<!--yml

类别：未分类

日期：2024-05-27 14:31:42

-->

# 人们不理解面向对象编程 - Sigma's Blog

> 来源：[https://blog.sigma-star.io/2024/01/people-dont-understand-oop/](https://blog.sigma-star.io/2024/01/people-dont-understand-oop/)
> 
> “对我来说，面向对象编程意味着仅仅是消息传递、局部保留和保护及隐藏状态-过程，以及所有事物的极限后期绑定。”
> 
> 艾伦·凯（发明“面向对象编程”术语的人）

看起来很多人都不喜欢面向对象编程。听到这三个字母时，脑海中首先浮现的是汽车、继承、获取器、设置器以及ObjectFactoryFactorySingleton。

这总让我感到有些奇怪。我不仅喜欢面向对象编程，我甚至认为它经常是解决问题的最佳/最明显的方式。以下是我认为如此的原因：

## 面向对象是啥？

我认为在进一步进行任何操作之前，我们可能应该定义一下我们正在谈论的内容。不幸的是，面向对象编程并没有被很好地定义。因此，为了保持连贯性，让我们首先确定一个清晰和明确的定义。

我们将经常谈论“对象”。那么它们是什么？大多数面向对象编程的入门文本使用汽车和动物等物理事物来说明对象的概念。虽然这不是错误的（这实际上是对象隐喻的来源；艾伦·凯是以生物细胞和网络的方式思考的），但显然是误导的，因为对象远远不止于此。

彼得·韦格纳写道：“对象是共享状态的操作集合。”

Mark Stefik和Daniel Bobrow对对象的定义如下：

“对象是将过程和数据属性结合起来的实体，因为它们执行计算并保存局部状态。对象的统一使用与传统编程中的独立过程和数据使用形成对比。”

下面是“四人帮”提供的另一种定义：“面向对象程序由对象组成。对象封装了既能处理数据又能操作数据的数据和过程。过程通常称为方法或操作。”

好吧，这是一个很好的开始，但我仍然觉得少了一个重要的对象特征。也许蒂姆·伦奇可以帮助一下：对象是状态单位，通常对外部是不透明的（我认为这是重要的部分。稍后我们将看到为什么）。然而，对象可以通过消息传递（即“方法”）与其状态进行交互。

等一下。“[…] 共享状态的操作集合”？“[…] 结合 […] 过程和数据 […] 的实体”？“[…] 状态单位”？

这到底是什么意思？

嗯，这意味着“对象”是一个抽象的术语。对象可以是任何东西 - 任何带有状态的东西。它可以是一个物理项目，如汽车，一个抽象概念，也可以是任何带有某种行为附加的随机数据片段。面向对象编程只是意味着我们使用这些对象来建模我们的问题。就是这样。

### 一类特殊的类

你可能会想：“等等，我们甚至没有触及类就定义了面向对象编程。这是怎么回事？”

答案很简单：类对于面向对象编程并非绝对必需。是不是有点震惊？

当然，我们需要能够构建新对象，而基于类的语言显然更为普遍。但是，这不是实现此目标的唯一方法。

像JavaScript（尽管ES6引入了语言中的类）或Lua这样的语言使用一种称为基于原型或原型面向对象编程（prototypal OOP）的概念。不是提供一个用于构造新对象的模式，而是使用现有对象作为原型。这种方法甚至可以在现实世界中带来好处，因为它降低了语言的复杂性。

（只是顺便提一下：类不需要被称为类。像Go或Rust - 以及在某种程度上的C++ - 例如称它们为structs。）

### 这是遗传的

另一个术语，虽然在技术上并不必需，但通常与面向对象编程相关联的是继承。

使用继承有两个原因：

第一个原因是重用现有代码。但是，在现代编程中，通常鼓励使用对象组合（一个对象内嵌到另一个对象）。

第二个原因（对我来说更重要的一个）是用于抽象和多态性。这种技术术语称为“子类型化”。

### ‘嘿，打字中？

（是的，我认为这个主题足够重要，值得有自己的标题。😅）

子类型化不仅限于面向对象编程，但在这里具有特殊意义，因为它是建模多态性的主要方式。其思想是将共享共同消息（即具有类似语义的方法）的多个不同类组合成一个定义这些消息的超类型。现在，可以使用超类型而不是指定子类型。

我最喜欢的子类型化在实践中的例子是Java集合框架。它定义了用于列表、队列、集合、映射等常见用例的接口（稍后我们将详细讨论接口是什么）以及具有不同特性的不同实现，支持这些不同用例。

wpg_div_collectionmap

（图表是从JavaDocs生成的，通过检索所有已知的[Collection](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/Collection.html)和[Map](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/Map.html)的子类并删除非相关节点而生成。）

所以，假设我想处理一个数据列表，我可以在所有地方使用List接口。在我实例化List的时候，我选择ArrayList，因为它通常是更高性能的实现。后来发现程序在数组的开头做了很多插入/删除操作，这在数组上非常慢。为了加速程序，我可以切换到LinkedList而不改变任何类型签名。

旁注：在调用方法时，我们需要知道对象的实际类，而不仅仅是它声明的类，否则子类型化将无法正常工作。这称为“迟绑定/动态绑定”。其技术执行有点棘手，这也是为什么在C++中对象和对象指针行为不同的主要原因（参见vtables）。

### 奇怪的行为

我认为我们不能（也不应该）谈论子类型化而不提到行为子类型化和Barbara Liskov。行为子类型化的基本思想是，子类型应该以与父类型类似的方式行为。

Barbara Liskov（后来因其在编程语言和面向对象编程方面的工作而获得图灵奖）在1987年将这一概念正式形成为“强行为子类型化”：子类型应该可以在其父类型可用的任何情况下使用。

“子类型要求：让 φ(x) 为可证明的关于类型 T 的对象 x 的属性。那么对于类型 S 的对象 y ，其中 S 是 T 的子类型， φ(y) 应该为真。”

这称为Liskov替换原则。我这里不想详细讨论，但基本思想是，任何参数的前提条件（对于类型、数据或状态）都不能比超类型更强，而结果的任何后置条件都不能比超类型更弱。这个概念与设计按契约方法论相关联，该方法论也在大约同一时间开始出现。

### 过于抽象

在某些情况下，我们不关心继承的代码共享方面，但仍希望从子类型化中获益 - 我们可能根本不会实际使用方法的超类型实现，因此可以完全省略它。这实际上是如此常见，甚至有一个名字：虚拟或抽象方法。

我们甚至可能会从我们的抽象超类型中移除所有状态，并且仅将其用作定义方法的模板。这被称为接口。

一些语言甚至进一步解耦接口和类。有两种不同的思维方式：

结构类型化（与通常的名义类型化相对）是接口实现根本不声明时的情况。只要定义了所有必要的方法，您就可以简单地将对象用作实现。这在编译时进行静态检查。支持结构类型化的语言示例包括Go（用于接口本身和类型约束）和C++（用于概念）。

鸭子类型类似，但仅在运行时检查方法的存在。使用此模式的语言包括Python和JavaScript。经常被提到的一个缺点是，在程序的特定点更复杂地确定可以预期的类。

第二个模式似乎还没有确立的名称。这个想法是在类定义之后声明该类实现一个接口。一个支持这种特性的语言的例子是Rust与traits。不幸的是，“traits”对于这个概念来说是一个可怕的名称，因为“traits”通常只是指混入。我听说过术语“extension traits”——参考C#/Kotlin中的“extension methods”——但这似乎也不是很常见。另一个支持此特性的语言是Haskell（他们称其为“type classes”；但Haskell可以说不是面向对象的）。

### 躲猫猫

在面向对象编程中经常使用的一个术语是“封装”。实际上，这个术语有两个适用的定义。第一个是指将数据与行为捆绑在一起（即对象比喻）。第二个是指仅将对状态的访问限制在对象本身。我想集中讨论后者，因为我认为很多人并没有完全理解它。

> “封装是一种通过定义严格的外部接口来最小化分开编写的模块之间相互依赖的技术。”
> 
> Alan Snyder，1986

所以为什么限制对状态的访问很重要呢？好吧，有多个原因。我们可以说这违反了Liskov的历史约束。但我认为从想重构代码库的开发者的角度来看更为实际。假设我们想改变对象的内部结构（就像之前的List示例中一样，也许我们想从ArrayList切换到LinkedList）。但是如果其他组件依赖于内部状态（在ArrayList的情况下：这可能是内部原始数组），我们无法轻松更改它。我们需要找到类外引用内部结构的所有地方。当类被导出并由我们可能无法控制的模块使用时，问题会变得更加严重。

“（对象）耦合”和“（类）内聚”经常用来谈论封装。“对象耦合”描述了不同对象彼此依赖的程度。高“对象耦合”意味着相关对象之间的依赖很大，通常意味着它们应该是一个单一对象。如果对象依赖于彼此的内部结构，则它们耦合性很高。“类内聚”从不同的角度描述了相同的特性。它衡量了一个类的责任有多一致。一个类理想情况下应该代表一个想法，并且只执行与该想法相关的操作。低“类内聚”通常意味着高“对象耦合”，反之亦然。

我相信如果你有进行过任何面向对象编程，你一定听说过类似“不要使用公共属性[...]”的建议。这是真的，因为公共属性暴露了内部状态，可能会导致高对象耦合。然而，和任何教条一样，通常质疑它是一个好主意。在这种情况下，完整的“指导原则”是“不要使用公共属性，而是使用getter和setter。”这完全是错误的。在封装方面，getter和setter与公共属性一样糟糕，因为它们对阻止对象耦合没有任何作用。如果一个类除了getter和setter之外没有任何方法，它实际上并不符合我们的对象定义。用于这种情况的术语是“记录”。

### };

好的，那么面向对象编程（OOP）是什么？OOP是将相关状态和行为捆绑成单元（= 对象）。面向对象语言可能具有的其他特性包括：类、原型、封装、子类型、继承等。

让我们来看看一些现代语言（这些是来自[StackOverflow 2023开发者调查](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-programming-scripting-and-markup-languages)的前15种语言，不包括HTML等...）：

| 语言 | 对象 | 对象创建 | 封装 | 子类型 |
| --- | --- | --- | --- | --- |

| JavaScript | ✔️ | 类/原型

| ✔️（自ES2022起） | 继承/鸭子类型（Inheritance/ Duck typing） |
| --- | --- |
| Python | ✔️ | 类（Classes） | ❌（语言层面上没有） | 继承/鸭子类型（Inheritance/ Duck typing） |

| TypeScript | ✔️ | 类/原型（Classes/ Prototypes） | ✔️ | 继承/结构类型（Inheritance/ Structural typing） |

鸭子类型 |

| ShellScript | ❌ | ❌ | ❌ | ❌ |
| --- | --- | --- | --- | --- |
| Java | ✔️ | 类（Classes） | ✔️ | 继承/命名类型（Inheritance/ Nominal typing） |
| C# | ✔️ | 类（Classes） | ✔️ | 继承/命名类型（Inheritance/ Nominal typing） |

| C++ | ✔️ | 类 + 结构体（Classes + Structs） | ✔️ | 继承/命名类型（Inheritance/ Nominal typing） +

结构类型（概念） |

| C | ❌（没有方法） | 结构体（Structs） | ✔️（使用不完整类型） | ❌（通过嵌套结构体实现的单一“继承”，没有真正的子类型） |
| --- | --- | --- | --- | --- |
| PHP | ✔️ | 类（Classes） | ✔️ | 继承/鸭子类型（Inheritance/ Duck typing） |
| PowerShell | ✔️ | 类（Classes） | ❌ | 继承/鸭子类型*（Inheritance/ Duck typing*） |
| Go | ✔️ | 结构体（Structs） | ✔️（在包级别上） | 结构类型 |
| Rust | ✔️ | 结构体（Structs） | ✔️ | 扩展特性/命名类型（Extension Traits/ Nominal typing） |
| Kotlin | ✔️ | 类（Classes） | ✔️ | 继承/命名类型（Inheritance/ Nominal typing） |
| Ruby | ✔️ | 类（Classes） | ✔️（强制） | 继承/鸭子类型（Inheritance/ Duck typing） |
| Lua | ✔️ | 表（原型） | ❌ | 继承/鸭子类型（Inheritance/ Duck typing） |

*) 不确定

## 噢-噢，不好了……

好了。现在我们对面向对象编程是什么以及我们可以从实现OOP范式的语言中期待什么有了很好的理解，让我们来看看一些常见的批评点。（我厚颜无耻地通过询问朋友们讨厌OOP的原因来获得以下大部分内容。😋）

### 但是什么是对象？

所以，对象可以是任何东西，对吧？那么我如何知道什么应该是一个对象？什么时候应该将事物结合在一起，什么应该分开？

嗯，在最后，这只是实践和经验问题。随着时间的推移，你会对什么应该是对象，什么不应该是对象有所感觉。然而，为了开始，有一些技巧可能会对你有所帮助。

以下是《四人帮》的看法：

“面向对象设计方法偏爱多种不同的方法。你可以编写问题陈述，单独找出名词和动词，然后创建相应的类和操作。或者你可以专注于系统中的协作和责任。或者你可以对现实世界进行建模，并将分析中找到的对象转化为设计。对于哪种方法最好，总会存在分歧。”

### 快速的东西进去，快速的东西出来

编辑：有人指出我在设计基准测试时犯了几个错误。感谢Github上的NoNaeAbC指出我在面向对象编程和结构化编程测试中分配和清零了太多内存，以及Youtube上的u9vata批评了我的基准设计。关于后者，虽然我不同意他们的所有观点，但的确是我对编译器优化做出了一些无根据的假设。

我不确定何时会有时间重新设计基准测试，因此在这段时间内，请对以下内容保持怀疑态度。

顺便说一句，我也想出了另一个解释，解释为什么函数式编程测试如此缓慢：乐队被存储为越来越嵌套的闭包，这些闭包必须将它们的参数存储在堆上，而面向对象和结构化编程版本则可以完全在栈上工作。

面向对象编程（OOP）很慢。或者这是我听说的。理由是虚表查找比直接函数调用多余。我实际上不知道这是不是真的，所以我决定进行测试。

测试设置如下：我写了相同的程序（检查二进制回文的图灵机）三次，一次使用面向对象编程，一次使用结构化编程（仅使用函数、循环、元组、数组等等），还有一次为了充分考虑使用函数式编程。

我用C++实现了所有内容，这样是公平竞争的基础（此外，C++为函数式版本提供了一流的函数/lambda表达式）。有100,000个测试用例，总时间已测量。编译器是clang 14.0.3，目标平台是Apple Silicon（M1）。我用-O0和-O3分别运行了每个测试。

对于面向对象的实现，我确保不依赖堆分配，因为上下文切换可能会完全破坏运行时。但是，我确实使用了继承（具体来说是模板模式），以使虚表查找尽可能真实。

结构化版本还在堆栈上分配所有内容。我构建了两个不同的版本。第一个版本在转换表查找中使用了元组，但我不确定元组在内部是如何实现的，如果可能的话，我想避免意外使用对象，所以我写了另一个只依赖函数的版本。但结果显示两者非常接近，我分辨不出有什么区别。

<canvas id="myiChartbenchmarks--o0-1"></canvas>

<canvas id="myiChartbenchmarks--o3-2"></canvas>

正如我们所看到的，结构化版本在没有使用任何优化时，速度比面向对象版本稍快（大约5%）（虽然我应该提到，在运行之间观察到的值有很大的跳动）。当使用 -O3 时，性能基本上是相同的（约为1%），所以我猜测 C++ 优化器能够消除影响性能的任何东西。

函数式实现根本不接近。在某种程度上，这可能是由我选择的基准测试引起的。图灵机在本质上是有状态的，这在函数式模型中建模起来相当尴尬。另一个方面是，尽管我使用了支持返回类型推断的 C++14（但我被迫使用了 std::function 模板作为 lambda 表达式的包装器（匿名类型真是让人头疼），而根据我的测试，这比本地 lambda 表达式要慢得多。

我应该可能进行一些严格的统计测试，或者至少计算一下偏差。但老实说，我太懒了。也许以后我会写一个更新，进行适当的分析。

如果你想自己进行一些测试，请随时把结果发给我。😛 源代码在 [Github](https://github.com/overflowerror/oop-benchmarks/tree/blog-version) 上（同时我可能应该为那些可怕的代码道歉，C++ 不是我的母语，我大概只花了一个小时左右瞎搞了一通 😅）。

无论如何，缺乏严格的统计数据，我的测试结论是性能差异非常小。增加更多的抽象层（或者使用不同的数据结构）可能会产生更显著的影响。

然而，其他嵌入式系统上的基准测试发现与过程化实现相比，性能有约10%的降低。

另一篇比较面向对象编程不同方面性能的论文以及不同设计模式的性能表现显示，虚函数（我在我的实现中使用了）可能会对性能产生负面影响（大约5%）。模板模式（我也使用了）也可能会使性能降低约3-4%（但这可能只是因为它依赖于虚函数）。

### 抽象的胡说八道

由于某种原因，面向对象编程（OOP）导致我们把一切都复杂化了。我们不必要地在抽象之上再次建立抽象，看起来似乎只是为了制作漂亮的UML图表。

问题是：这是由我们如何使用工具引起的，而不是工具本身的问题。我怀疑大多数这些问题是开发人员想要聪明地构建通用解决方案来覆盖每一种可能的未来发展所致。

我认为通过调整工作流程，可以避免很多这样的情况。具体来说：如果开始时没有确定最终目标，那就不要从一开始就为每种可能性制定计划，只为你知道你将需要的部分制定计划。需求可能会后来改变，所以你为此费了4个星期连续工作的精彩且高度通用的解决方案最终可能不会被使用，这是在浪费时间。

### 获取和设置的威胁

面向对象编程太啰嗦了，有太多样板代码。例如，获取器和设置器。

*叹气* 这是我个人的小小怨念。我们之前提到过这个问题，但我真的很想强调一下：如果你真的需要为每个成员变量都有获取器和设置器，那它可能不是一个适合用对象表示的对象。我强烈建议重新考虑你的对象模型，试着减少耦合。如果真的是一个没有内部行为的记录类，一切都可以是公开的——使用获取器和设置器几乎没有意义。类似的情况（尽管在某种程度上更好）也适用于诸如C＃之类的属性，以及像臭名昭著的Lombok之类的代码生成器。

使用获取器和设置器而不是公共成员的唯一真正原因是，当存在一些额外的逻辑，比如不变量的验证时。

有点相关：如果你有一个值对象，没有设置器但有很多获取器，请确保不要意外地暴露内部状态的可修改引用。否则，你就有了设置器——只是不是故意的。

### ObjectFactoryFactorySingleton

我想这个标题适合两个话题。首先是已经在企业软件开发中确立的命名疯狂。虽然这与面向对象编程本身并无直接关系，但由于某种原因，这种情况似乎在面向对象编程中更为普遍。我碰巧是Kevlin Henney的粉丝，他在2015年的DevWeek上做了一场[关于编程中命名的精彩演讲](https://www.youtube.com/watch?v=CzJ94TMPcD8)。除其他内容外，他还谈到了命名如何影响建模。我强烈推荐观看这个视频。

第二个话题是设计模式的兔子洞，通常是盲目应用的，似乎没有思考为什么。具体来说，工厂模式确实有一些有效的用途，但因为人们过度使用这种模式，现在它已经成为不必要抽象的代名词。

当然也有已经确立的模式，你确实应该有一个非常好的理由来实际使用它——至少在严格的面向对象上下文中是这样。例如单例模式。“单例”本质上只是全局变量的一个花哨的名字——很好。有趣的小插曲：在Spring框架中，Bean默认为Singleton作用域，这意味着如果没有另有说明，每个Bean都是全局的。

### 春天的梦想

我注意到现代“企业”应用的另一件事是它们实际上并不是面向对象的。实体、DTOs，… 都是记录，而不是对象。Bean、服务、仓库，… 不保持状态，可以同样写成模块中的简单函数。

我们使用的语言迫使我们在不需要对象的架构中思考类 - Spring Boot 可以同样用 C 语言编写。

## 结论

太精彩了。我觉得这是我迄今为止最长的博客文章。或许有点太长了… 我会确保下一篇文章更短一些。😅

我还发现这个真的很有意思的 [关于抽象的 Barbara Liskov 的演讲](https://www.youtube.com/watch?v=dtZ-o96bH9A)，但我不确定应该放在哪，所以给你了。（我特别喜欢她讽刺 Python 丢掉封装的那部分 😆）

无论如何，希望我能为这个主题提供一些见解，也许你学到了什么，或者至少觉得我的胡言乱语有点娱乐性。

待会见，

西格玛

### 脚注
