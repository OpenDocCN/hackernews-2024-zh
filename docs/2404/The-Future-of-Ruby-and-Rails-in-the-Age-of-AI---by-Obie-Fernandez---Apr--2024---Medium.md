<!--yml

分类: 未分类

日期: 2024-05-27 13:21:34

-->

# [AI时代下的Ruby与Rails的未来](https://obie.medium.com/the-future-of-ruby-and-rails-in-the-age-of-ai-8f1acea31bc2) | 作者 Obie Fernandez | 2024年4月 | Medium

> 来源：[https://obie.medium.com/the-future-of-ruby-and-rails-in-the-age-of-ai-8f1acea31bc2](https://obie.medium.com/the-future-of-ruby-and-rails-in-the-age-of-ai-8f1acea31bc2)

# AI时代下的Ruby与Rails的未来

## Ruby程序员将通过以提示驱动的开发引领AI革命

人工智能（AI）正在以前所未有的速度改变我们构建、调试和优化代码的方式。作为长期以来的Ruby和Rails开发者，我通过[我的创业公司](https://olympia.chat)，[我的新书](/patterns-of-application-development-using-ai-fbb660fa9ae7)以及[开源项目](https://github.com/OlympiaAI/open_router)兴奋地参与到这些发展中。

我要分享的好消息是：不仅Ruby *和* Rails会在这场AI革命中生存下来，Ruby程序员将开创未来时代最革命性的应用开发方法。正是吸引我们进入Ruby和Rails生态系统的特质 —— 其表达能力、可读性和对开发者幸福感的重视 —— 恰恰是我们能够引领软件开发AI驱动未来的位置所在。

假如你对此不甚了解，Ruby的神奇之处在于其动态且富有表现力的特性。它是一种让你专注于解决问题而非与语法纠缠的语言。再加上Rails的约定大于配置的方法，你将拥有一个强大的组合，可以实现快速迭代开发。这也是为什么20年后，Rails仍然是无可争议的Web应用框架之王。

> 正是吸引我们进入Ruby和Rails生态系统的特质 —— 其表达能力、可读性和对开发者幸福感的重视 —— 恰恰是我们能够引领软件开发AI驱动未来的位置所在。

正如我在过去一年中亲身体会到的那样，这些“神奇”的特质非常适合于AI增强的软件开发。我设想不久的将来，我将能够以简单、几乎是自然语言（实际上是伪代码）来声明我的应用功能，而不是通过大量的代码、测试和文档。这才是AI为软件开发带来的真正承诺，而熟悉Ruby本质的我们正处于独特的位置，可以实现这一承诺。

# 以提示驱动的开发：一个新的范式

在这个由AI驱动的开发的新世界中，我不仅仅是在谈论使用AI生成代码片段或自动完成方法。我谈论的是一种根本性的转变。以提示驱动的开发：一个范式，其中应用行为不再由成千上万行繁琐编写和维护的代码决定，而是由高级、声明性的提示定义。在早期阶段，它将被编织到普通的Ruby代码中，但最终，这只会是谈论而已。

想象用简单的会话语言描述复杂的工作流程或新功能。你的 AI 助手将此描述实时转化为现实。不再花费数小时抓住复杂实现或担心低级细节。你专注于业务目标，AI 处理如何实现它。

*这个愿景不是为了生成存储和维护的代码。* 它是关于在你的声明性提示与应用程序的实际行为之间创建动态、运行时的关系。你的提示是你活跃、充满活力的代码库。

```
You are an intelligent account manager for Olympia. The user will request
changes to their account, and you will process those changes by invoking one
or more of the functions provided. Escalate to human support rep if you
encounter any problems.

Do not allow the user to change their account or add a new AI assistant
unless their account subscription status is active.

Make sure to notify the account owner of the result of the change request
whether or not it is successful.

Always end by calling the 'finished' function so that we save the state
of the change request as completed.
```

上述提示并非幻想。正如在[我的新书](https://leanpub.com/patterns-of-application-development-using-ai)中描述的那样，这是供 [Olympia](https://olympia.chat/?utm_source=medium) 中 AccountManager 组件使用的实际系统提示。该组件通过提示与我的应用程序的其余部分进行交互。再说一遍，这个组件的内部 API 是纯文本。我正在让 AI 以黑匣子方式处理本应是数十或数百行代码的工作。已经，今天。

> 在你的项目中采取类似的方法吗？让我们[交流](mailto:obiefernandez@gmail.com)吧！

# 挑战与机遇

当然，这种方法也带来了一系列挑战。当需要确定性结果时，我们如何确保？如何确保生成的代码高效且安全？当代码抽象时如何调试问题？这些是我们作为 Ruby 程序员独特装备处理的问题。我们社区的务实、创造力和专注于开发者体验的混合将在塑造这种新范式的最佳实践和工具方面非常宝贵。

潜力巨大，开发周期更快，bug 更少，代码库更易维护和自我记录。而且借助 Ruby 的表现力和可读性，我们有机会创建使用即时 DSL（领域专用语言）的快捷驱动工具。这是一个开发者角色从低级代码编写者转变为高级即时驱动组合者的未来。

# 声明性编码与 BDD 的历史

在计算历史上，并非首次有人试图使用自然语言建立声明性范式。过去曾多次尝试创建编程语言或框架，允许开发者使用类似自然语言的语法表达他们的意图。

**HYPERCARD是一种交互式系统，旨在作为独特的信息中心运行**。它可以用来搜索和存储文本、自定义图形和数字化的照片。与书籍不同，书籍具有线性格式（您从一页翻到下一页），HyperCard 的交互能力使您能够将任何一条信息与任何其他信息关联起来。通过模仿我们的思维过程（关联过程），HyperCard 让您能够以最简单、最快的方式找到所需知识。摘自*1988年1月的[《音乐技术》](https://www.muzines.co.uk/mags/MT/88/01/73)*文章。

例如，在20世纪80年代，有一种名为HyperTalk的语言，用于苹果的HyperCard，旨在通过使用类似英语的命令使编程更加容易。最近也有像[Inform 7](https://ganelson.github.io/inform-website/)这样的尝试，它是一种用于创建交互式虚构的编程语言，使用英语语法的子集。

Inform是一种使用自然语言语法创建交互式虚构的编程语言。利用语言学和文学编程的思想，Inform广泛用作文学写作的媒介，在游戏行业中作为原型工具，以及在教育领域，包括学校和大学水平（Inform通常作为数字叙事课程的分配材料）。根据TIOBE指数，它已多次排名前100位最有影响力的编程语言之一。Inform于2006年4月创建，于2022年4月开源。

然而，这些方法都依赖于结构化语言，虽然它们使用类似自然语言的单词和短语，但仍要求开发者遵循特定预定义的语法和语法。开发者必须学习这种“伪英语”并在其约束内工作。

在21世纪中期，一种新的软件开发方法开始获得关注：行为驱动开发（BDD）。BDD作为测试驱动开发（TDD）的演进而出现，其主要区别在于：它不再专注于测试单个代码单元，而是通过一种可被所有利益相关者理解的语言来定义系统的期望行为，包括非技术人员。

对于我和我在Thoughtworks的同事们来说，对BDD的兴奋促使我们创建了像RSpec、Cucumber和Pickle这样的工具，这些工具允许开发者使用结构化的自然语言来定义应用程序的行为。我朋友Aslak Hellesoy的Gherkin语法类似于英语句子，组织成场景和步骤。例如：

```
Given I am on the home page
When I click on the "Login" button
Then I should see the login form
```

这些场景既作为文档，又作为自动化测试。开发者编写代码，将每个步骤与应用程序中相应的操作或断言关联起来。当运行测试时，工具读取 Gherkin 场景并执行关联的代码，确保应用程序的行为与描述一致。

This approach had a significant impact, especially in the Ruby community, *where it was born*. It allowed teams to bridge the gap between business requirements and technical implementation. Business analysts, product managers, and even clients could contribute to writing these scenarios, ensuring that the development team had a clear, shared understanding of what needed to be built.

Moreover, by tying the application code directly to these human-readable scenarios, BDD made the codebase more maintainable and easier to understand. Developers could see not just what the code was doing, but why it was doing it. This made it easier to make changes and avoid introducing bugs.

The problem was that while BDD and tools like Cucumber and Pickle represented a significant step towards using natural language in software development, they still required a level of structure and specificity in how the scenarios were written. They also require extensive implementation of “steps”.

```
Given("I have entered {int} into the calculator") do |number|
  @calculator = Calculator.new if @calculator.nil?
  @calculator.enter(number)
end

When("I press add") do
  @result = @calculator.add
end

Then("the result should be {int} on the screen") do |expected|
  expect(@result).to eq(expected)
end
```

I can tell you from personal experience that even though they look simple, steps are a nightmare to write and maintain. The AI-powered approach I’m envisioning takes BDD a couple, ahem, steps further.

Instead of requiring scenarios to be written in a specific format, an AI system can understand and generate code from free-form descriptions, or even from a conversation about the system requirements. Imagine the following:

> Human: “When a user clicks the login button, they should be taken to a login page where they can enter their username and password. If they enter the correct details, they should be logged in and redirected to their dashboard. If they enter incorrect details, they should see an error message.”
> 
> AI: “Do we support Oauth-based logins?
> 
> Human: Sure, but just Google for now.
> 
> AI: And how should I style it?
> 
> Human: Show me a few options based on our typical style guide.

And so on and so forth…

In the aformentioned conversation (and in contrast to BDD), the AI system is generating not just the test scenarios, but also specification, code and visual design for this behavior. This is the kind of natural, conversational interaction that the advancements in AI are starting to make possible.

> The AI-powered future of software development promises to take the concept of natural language programming to a whole new level. It’s an exciting prospect, and one that the Ruby community, with its history of embracing innovative approaches to software development, is well-positioned to explore and pioneer — we were literally the ones that pioneered BDD.

因此，尽管BDD和像Cucumber和Pickle这样的工具在它们的时代具有开创性并且今天仍被广泛使用，但AI驱动的软件开发未来承诺将把自然语言编程的概念提升到一个全新的层次。这是一个令人兴奋的前景，而且是Ruby社区所能够探索和开拓的——毕竟我们可是BDD的先驱者。

# Ruby和Rails的AI融入之路

目前我已经在Ruby on Rails上工作了近20年，几乎每天都在用。因此，我如何实现上述“神奇未来”的视角包括将AI紧密融入到Ruby和Rails的基础层级中。LLM最终可能会在Ruby解释器内部得到利用，从而实现神奇的体验——智能推断方法名称和参数，无缝连接不同的服务和API，并实时优化代码，但*首先*我计划在我用来构建Ruby on Rails应用程序的基础组件中加入越来越多的AI集成。

例如，在定义传递给LLM的函数时，[我正在开发的框架](https://github.com/OlympiaAI/raix-rails)会提供一个智能参数对象，而不是冗长地描述参数，该对象能处理映射、推导和计算。使用智能参数对象等AI组件的代码将不可避免地开始看起来更像伪代码。

有些人可能会批评这太神奇，太抽象。但他们在Rails早期也对此提出了同样的批评，现在显然他们是错的。“他们”会对我所倡导的方法提出批评吗？当然会。让我们再次开始牙齿咬碎的抱怨吧。
