<!--yml
category: 未分类
date: 2024-05-29 12:34:59
-->

# C++ creator rebuts White House warning, but there’s no smoke without fire :) – CppDepend:

> 来源：[https://cppdepend.com/blog/c-creator-rebuts-white-house-warning-but-theres-no-smoke-without-fire/](https://cppdepend.com/blog/c-creator-rebuts-white-house-warning-but-theres-no-smoke-without-fire/)

In a March 15 response to an inquiry from [InfoWorld](https://www.infoworld.com/article/3714401/c-plus-plus-creator-rebuts-white-house-warning.html), Stroustrup pointed out strengths of C++. “I find it surprising that the writers of those government documents seem oblivious of the strengths of contemporary C++ and the efforts to provide strong safety guarantees,” Stroustrup said. 

And Stroustrup cited a fact about the origin of the issue :

> There are two problems related to safety. Of the billions of lines of C++, few completely follow modern guidelines, and peoples’ notions of which aspects of safety are important differ.

This highlights a significant problem with C++. When any programming language permits the execution of potentially harmful actions, it shouldn’t come as a surprise that a considerable portion of developers may misuse it.

And when confronted about writing bad code, developers may offer various arguments to justify their actions, though these are often excuses rather than valid reasons:

 1.  **Tight Deadlines**: “I had to rush because of tight deadlines. There wasn’t enough time to write clean code.”
2.  **Legacy Code**: “The existing codebase is messy and poorly structured. My changes just blend in with the existing mess.”
3.  **Scope Creep**: “The requirements kept changing throughout the project, making it difficult to maintain clean code.”
4.  **Technological Constraints**: “The technology stack we’re using isn’t suitable for writing clean code. We’re limited by what we have.”

So yes the developers have a responsibility to ensure they develop their code properly in C++. However, this kind of approach could be risky for the futur of C++. Just few years ago we have suprisingly assisted to the Nokia failure. Indeed, Nokia’s decline from being the world’s leading mobile phone manufacturer to struggling in the market is a story marked by several factors and strategic missteps. One of Nokia’s critical mistakes was its decision to stick with its Symbian operating system for too long. While Symbian was once a dominant platform, it struggled to compete with the user experience offered by iOS and Android.

In C++ we stick for too long with the current memory management mechanism and no radical solution is suggested, only few improvements that need to be applied by the developers.

**C++ vs .Net startegy**:

C# was developed in 2000 primarily for Windows machines. Miguel de Icaza created Mono to enable its use on Linux and Mac OSX. However, after over a decade of predominantly using the standard .Net framework on Windows, a significant issue arose regarding the language’s portability. To address this, Microsoft collaborated with Miguel to create .Net Core, a subset of .Net designed to function on other operating systems.

This is not a step by step solution to resolve the portability issue, but a radical one even if the legacy code is not compatible. at this time  Miguel de Icaza describes .NET Core as a “**redesigned version of .NET** that is based on the simplified version of the class libraries”,and Microsoft’s Immo Landwerth explained that .NET Core would be “**the foundation of all future .NET platforms**“.

And finally this solution worked perfetly and .Net core become widely used, and the big portability issue is resolved.

Ultimately, this solution proved highly effective, with .Net Core becoming widely adopted, successfully resolving the significant portability issue.

Why not take a similar approach with C++ to definitively address the safety concerns? Why not develop a safe subset of C++ and provide the option to work with this subset through the compiler?

clang –safe

**Conclusion**:

If C++ continues to allow developers to engage in unsafe memory practices, this significant safety concern will persist, potentially leading to other languages such as Rust or Go being preferred for new projects. Maybe, it’s time to think to a radical solution rather than step by step improvements. Certainly, experience has demonstrated that despite the availability of modern features in C++ aimed at addressing safety concerns for over a decade, the issue persists due to the language’s continued allowance of legacy bad practices.