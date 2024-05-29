<!--yml
category: 未分类
date: 2024-05-27 14:29:40
-->

# Need for an Undefined Behavior Annex to C++

> 来源：[https://community.intel.com/t5/Blogs/Tech-Innovation/Tools/Why-do-we-need-a-Undefined-Behavior-Annex-for-the-C-standard/post/1574397](https://community.intel.com/t5/Blogs/Tech-Innovation/Tools/Why-do-we-need-a-Undefined-Behavior-Annex-for-the-C-standard/post/1574397)

Intel's developer software engineers are not only working on the latest optimizations and improvements in the Intel® oneAPI DPC++/C++ Compiler. We actively contribute to the advancements of Khronos Group's [SYCL*](https://www.khronos.org/sycl/), the [LLVM*](https://llvm.org/) Project, and the latest C and [C++ language standard](https://isocpp.org/std/the-standard)s .

We would like to share a more detailed picture of what is happening in the C and C++ world with the software engineering community and how Intel actively contributes to the latest standards revisions.

## A Blog Series on Our C/C++ Standards Contributions

To that end, we are starting a new series of blog posts highlighting Intel's work on the C and C++ standards.

In particular, safety and security are one of the most active focus areas of our work on the next language standard revisions. This article will discuss the effort to create a much-needed Undefined Behavior Annex for the C++ Standard. This will strengthen coding standards for functional determinism and security, even for uncommon or unanticipated application code paths.

There are areas we know are important to the [Intel® oneAPI DPC++/C++ Compiler](https://www.intel.com/content/www/us/en/developer/tools/oneapi/dpc-compiler.html) user community and the oneAPI and C/C++ community at large. We know that because C and C++ are continuously evolving languages. Keeping track of the fast pace of change within the community is hard if you are not actively involved. Intel's C and C++ Compiler team is involved in this evolution at many points, participating in standardization through

*   writing proposals
*   attending meetings 
*   chairing committees
*   Submitting open-source contributions to LLVM and the Clang compiler.
*   Being active in the community at large.

We would like to take this engagement with the community to the next level by using this format to encourage a dialog about the latest developments and focal points in C and C++ programming.   

## Undefined Behavior Scenario Annex

I am Shafik Yaghmour, a Compiler Engineer at Intel supporting C++. I am active in the C++ community in many forms:

This blog highlights some of my work on the C++ standard committee.

My proposal, [P1705R1: Enumerating Core Undefined Behavior](http://wg21.link/P1705R1) [2], discusses adding an *undefined behavior* annex to the C++ standard. 

Invoking *undefined behavior* has all sorts of unintuitive consequences, such as [the removal of safety checks](https://blog.regehr.org/archives/970), [turning finite loops infinite](https://stackoverflow.com/questions/32506643/c-compilation-bug), [booleans that can both be false and true](https://markshroyer.com/2012/06/c-both-true-and-false/), and apparently [undefined behavior can even time travel](https://devblogs.microsoft.com/oldnewthing/20140627-00/?p=633) [3]  🤯. These consequences are not desirable and can lead to safety and security issues.

## Safety Implications of Undefined Behavior 

To avoid the consequences of undefined behavior in code, many industries employ two strategies:

1.  Rigorous and well-documented dynamic functional safety and security attack vector verification testing. Regulatory bodies often mandate this for safety-sensitive industries such as the medical devices, aviation, manufacturing and automotive sectors.
2.  Strict adherence to coding standards (such as MISRA and AUTOSAR) that may limit the developer's ability to take advantage of the latest C++ language features.    

Developers do this, because the consequence of undefined behavior causing non-deterministic functional outcomes may well mean the loss of not just property but could also endanger people's safety! 

So I ask myself:

*   What if we took advantage of the rich and long history of C and C++  and documented cases of undefined behavior to define a clear, complete, authoritative list of potential undefined behavior scenarios to avoid and recommended fixes?
*   What if this list was part of the C++ standard itself?

This could introduce a new level of functional safety and protections against many security exploits to a wider range of use cases without the restrictive overhead of DOE-178 or ISO 26262 type regulations.

## A Comprehensive Reference for Undefined Behavior

Currently, this proposed comprehensive reference for *undefined behavior* does not exist. If you are a developer attempting to understand what *undefined behaviors* you need to avoid, you must refer to the 2000+ page C++ standard document and the 20+ years of defect reports. To add to this challenge, *undefined behavior* can be referred to in the document using several different phrases:

*   the behavior of the program is undefined
*   has undefined behavior
*   results in undefined behavior
*   the behavior is undefined
*   have undefined behavior
*   is undefined
*   result has undefined behavior

Then, after finding each *undefined behavior* entry, there may not be an example. Therefore, it may be unclear how the *undefined behavior* works. This is a daunting task even for a very knowledgeable C++ developer. This also hinders tool developers from building tools to make software more secure, as well as researchers in software safety and security.

We are also ignoring ***implicit undefined behavior***, which is behavior that violates the rules in the standard but is not explicitly called out as *undefined behavior*. It is just as bad as the explicit kind but much harder to find.

So, one of the main goals of *P1705R1* is to create an ***undefined behavior annex*** or listing of ***explicit* *undefined behavior ***in the C++ standard. Each entry would include a description in accessible language and an example(s) of the *undefined behavior* in code. This should allow developers to learn about undefined behavior. While we don't expect everyday developers to refer to the C++ standard themselves, having the material will allow blog authors, conference presenters, and trainers to develop more comprehensive material covering *undefined behavior*.

This will also allow the C++ committee to track the growth or shrinkage of *undefined behavior*. We also will require future proposal authors to keep the annex updated if they add or remove *undefined behavior*. They are making *undefined behavior* an explicit topic to be discussed when reviewing a proposal. It also gives explicit targets for future proposal writers to attempt to shrink *undefined behavior* in the standard.

Another goal that came up during a recent review of the proposal is that we want to turn implicit *undefined behavior* into ***explicit undefined behavior***. This requires identifying *i**mplicit undefined behavior*** and then having the [Core Language Wording Group](https://isocpp.org/std/the-committee)  update the wording to make the behavior explicitly *undefined*. The benefit here is that we document more *undefined behavior*. It is also an opportunity to determine if the intent was for this to be *undefined* and, if not, document that.

In [P3075R0: Adding an Undefined Behavior and IFNDR Annex](https://wg21.link/P3075R0) [4], I also extend this idea to include adding an ill-formed no diagnostic required (IFNDR) annex. *IFNDR* differs from *undefined behavior* in that while your program is still *undefined,* it is a [static property of your program and not a run-time property](https://shafik.github.io/c++/2023/07/09/lets-enumerate-ub-part-1.html). While this does not have the unrestricted runtime consequences of *undefined behavior,* it can still lead to safety and security issues since programs containing *IFNDR* may not be diagnosed and may have unintuitive results.

## Share Your Thoughts

We are looking forward to your feedback on proposal [P3075R0: Adding an Undefined Behavior and IFNDR Annex](https://wg21.link/P3075R0).

What do you think should be considered. Let us work together to advance the C++ standard set it on the path to fully embrace safety and security paradigms. 

Please feel free to leave comments below.

### References

**[1]** S. Yaghmour, [What is the Strict Aliasing Rule and Why do we care?,](https://gist.github.com/shafik/848ae25ee209f698763cffee272a58f8) Gist.GitHub, 2020

**[2]** S. Yaghmour, P1075R1: [Enumerating Core Undefined Behavior](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1705r1.html), 2019

**[3]** R. Chen, [Undefined behavior can result in time travel](https://devblogs.microsoft.com/oldnewthing/20140627-00/?p=633), Microsoft* DevBlogs, 2014

**[4]** S. Yaghmour, P3075R0: [Adding an Undefined Behavior and IFNDR Annex,](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p3075r0.pdf)  2023

### Further Reading