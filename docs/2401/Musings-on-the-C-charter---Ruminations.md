<!--yml
category: 未分类
date: 2024-05-27 14:27:08
-->

# Musings on the C charter | Ruminations

> 来源：[https://blog.aaronballman.com/2023/12/musings-on-the-c-charter/](https://blog.aaronballman.com/2023/12/musings-on-the-c-charter/)

Within the C committee, there’s been a lot of talk about what the [C charter](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2611.htm) means and whether it should be updated or not. My personal musing on the topic is that the C charter needs to be updated in order for C to remain relevant long-term. Whether C should remain relevant long-term is an exercise left to the reader. Note, these thoughts may change over time as I gather more feedback from more places.

 ## Existing code is important, existing implementations are not.
A standard is a treaty between implementor and programmer.
Migration of an existing code base is an issue.
Minimize incompatibilities with C90.

I think we could combine these principles into one bullet instead of four because they’re all effectively about code migration to newer standard versions.

Each new release of C should provide a simple upgrade path from the previous release of C so that users can gradually transition between adjacent versions of C, but incompatibilities between releases are acceptable. In other words, it’s fine to deprecate old functionality even without replacement, or remove previously deprecated functionality (again, even without replacement), but this should only be done with significant justification given the costs to users of C.

Changing the standard a code base compiles against is a new agreement between the programmer and the implementer; it is akin to changing your code (as is changing your compiler), and there should be no expectation that everything remains identical to how it was before. Older functionality will be deprecated, newer functionality will be added, bugs will be fixed, etc. New standard == new treaty.

I think we should be careful about blind adherence to this principle. It’s obviously true that there’s a lot of economic value tied up in existing C code bases and that rewriting that code is both expensive and error-prone. However, C has undergone multiple revisions in its long history. A good idea in 1978 is not necessarily even acceptable practice in 2024\. I would love to see this principle updated to set a time limit, along the lines of: existing code that has been maintained to not use features marked deprecated, obsolescent, or removed in the past ten years is important; unmaintained code and existing implementations are not. If you cannot update your code to stop relying on deprecated functionality, your code is not actually economically important — people spend time and money maintaining things that are economically important.

## C code can be portable.
C code can be non-portable.

I think both of these principles are still reasonable today. While commodity hardware has trended towards more homogeneity, there is still plenty of heterogeneous hardware solutions out there (DSPs, FPGAs, etc) and domains like AI, ML, cryptography, etc are pushing the boundaries for new hardware solutions. We should continue to aim to be as portable as possible while not constraining ourselves to the lowest common denominator of hardware.

## Avoid quiet changes.

I still strongly believe in this principle. It directly impacts transitions between standard versions — quiet changes make it much harder for users to upgrade to the latest standard.

## Keep the spirit of C.

I do not think this principle has much value because it is too subjective, though I understand the desire behind it. I think people want C to remain familiar between releases instead of being reinvented. For example, I think users do not want to see a Perl 5/Perl 6- or Python 2/Python 3-style bifurcation of the language and community. However, I think the principle that existing code is important is sufficient to cover that need.

Most especially, I think concepts like “trust the programmer”, “don’t prevent the programmer from doing what needs to be done”, “make it fast”, etc are problematic given the increased need for improved security in system’s level programming languages.

I would prefer to see this bullet removed because it’s not an actionable principle and I think other bullets cover the main thrust of what it was trying to say.

## Support international programming.

I still strongly believe in this principle, though I worry that WG14 lacks enough expertise to really help validate designs in this space. I appreciate that WG21’s SG16 members have been willing to provide advice, and that the Unicode consortium folks are actively engaging with programming language committees as well. But there’s more to internationalization than simply text (for example, units of measure, date/time functionality, and input methods are also part of internationalization). I think the goal is excellent, but hopefully we have enough humility as a committee to recognize when we could use outside advice.

## Codify existing practice to address evident deficiencies.
Unlike for C99, the consensus at the London meeting was that there should be no invention, without exception.

I think these bullets are saying the same thing and we only need to say this once if we wish to say it at all. However, I do not think the committee agrees with the idea of no invention without exception and we do everyone a disservice by stating it as a principle. For starters, we’ve shown we’re perfectly comfortable with invention during the C11 and C23 cycles. Invention is necessary if we want to unify divergent existing practice in the wild into something standardized. “No new invention” is why we still have no solution for a “pointer + size” type or a “string” type. There are a lot of different ways to solve those kinds of problems and until the committee picks one and says “This is The Way”, we’ll continue to see more ad hoc solutions.

That said, I think there are really important points hiding in these two bullets. For example, having a principled way to separate “need” from “want” is critical to avoid spending too much committee time on work that does not benefit enough users. What makes something a good feature to standardize vs not? How integrated should a feature be (for example, does adding a new arithmetic type to the language also require it be supported in I/O operations in the standard library, etc)? When two implementations support the same notional feature with slightly differing semantics, should the committee use undefined behavior to resolve the conflict so no users have to change code, or should the committee push for well-defined behavior despite knowing it will break some users? I think we want to have answers to these sorts of questions, but they’re less about charter and more about feature design guidance. Perhaps we want a second document to help authors understand what the committee needs in a proposal and why?

## Minimize incompatibilities with C++

[C++ inherits the C standard library almost directly from C,](https://eel.is/c++draft/library.c#2) but the core C++ language is specified entirely independently of the C language specification. Despite the languages being distinct in terms of specification, every production C++ compiler also claims to be a conforming C compiler. Further, C++ standard library implementations eventually need to use C interfaces to communicate with the host operating system. So I think it is in the best interests of WG21, users of C++, and C++ implementers for WG14 to minimize incompatibilities with C++.

Based on my experience as the former C and C++ compatibility study group chair and from being on both of these committees for about a decade, I think WG14 spends a significantly larger percentage of committee time on C and C++ compatibility than WG21 does. WG14 has a charter that mandates we minimize these incompatibilities (WG21 has no such mandate as they do not have a charter). The WG14 committee spends a considerable amount of meeting time deliberating how changes to C will impact C++ and our convener regularly reminds us of our obligations to minimize incompatibilities where possible. Comparatively, WG21 spends relatively little committee time on C compatibility and WG21 leadership does not proactively attempt to steer the committee to minimize incompatibilities. Given the difference in size between our committees (WG21 is approximate 15x larger than WG14), this is not a tenable situation. The committee with the least resources is spending the most time on compatibility despite reaping the least benefits.

I believe compatibility with C++ should not be a chartered requirement for features brought into C that are present in C++. Without closer cooperation between WG21 and WG14 (such as [a shared base document between the two languages](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2644.pdf)), I do not think WG14, C implementers, or C users get sufficient benefit for the amount of WG14 committee effort spent on compatibility with C++. Unfortunately, an inability for the committees to effectively collaborate hurts the people least represented on either committee: users working in this shared space. Therefore, I do believe compatibility with C++ should continue to be a goal as a design principle, especially for features that are likely to be used in header files shared between a mixed C and C++ code base.

## Maintain conceptual simplicity.

I still believe in this principle, though it’s a bit too ambiguous for my tastes. What one committee members finds to be simple, another might find to be too complex. For example, is a feature for overloading operators maintaining conceptual simplicity? I suspect asking five committee members that exact question will net about seven different answers.

## Trust the programmer, as a goal, is outdated in respect to the security and safety programming communities.

I don’t think this goes far enough. I think “trust the programmer” has lead to decades of security vulnerabilities, to the point that major government agencies are now [recommending against](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/0/CSI_SOFTWARE_MEMORY_SAFETY.PDF) [using C](https://advocacy.consumerreports.org/wp-content/uploads/2023/01/Memory-Safety-Convening-Report-1-1.pdf) (and C++). I would prefer we take a stronger stance along the lines of: do not trust the programmer. Design facilities such that their misuse becomes almost impossible outside of pathological code. Prefer “defined” behavior (well-defined, implementation-defined, or unspecified behaviors) to undefined behavior where possible. When misuse is possible, require strong error detection mechanisms which can prevent undefined behavior. I think it’s perfectly reasonable to expect “trust me, I know what I’m doing” users to have to step outside of the language to accomplish their goals and use facilities like inline assembly or implementation extensions.

## Application Programming Interfaces (APIs) should be self-documenting when possible.

I agree with the idea, but I don’t agree with the way it is presented regarding preference for VLA interfaces. That’s reasonable guidance for new APIs unrelated to any existing APIs, but it is not defensible when adding new APIs that are related to existing ones. e.g., adding `memset_explicit` should not (and thankfully did not) rearrange parameter order to fit this guidance, because it would have been very hard for people to migrate existing uses of `memset` to `memset_explicit` without introducing memory-related errors. I would prefer we made the guidance more general and less about VLAs.

## What’s Missing?

To me, the biggest thing we’re missing is a plan. ISO standards committees are “obligated” to consider all proposals from committee members, and so the approach the committee takes is to take whatever papers arrive and try to give the author feedback so that they can make progress. That said, our obligations do not include a mandate on when to schedule discussion of papers — we could elect to consider papers that aren’t in the committee’s plan only after considering all other in-scope work. But the committee has never gone to those lengths and so we release standards with a hodge podge of unrelated features and bug fixes. I would love to see the committee charter require that we come up with a roadmap of what we want to accomplish in a given release cycle and then stick to that road map. I don’t mind considering occasional out-of-scope work during the cycle, but that should not come at the expense of in-scope work. I think having a roadmap will also help the relationship between the committee, implementers, and users because it gives everyone more time to plan for what’s coming in the next few years.

The #1 thing I want to see on that roadmap is a goal to address the memory safety issues that plague C. We don’t have to fix everything at once, but we do need to start making inroads into solving the problem.

## Closing Thoughts

Regardless of whether others on the committee agree with my current thinking, I think it’s a great thing that committee is reflecting on our charter. I think C is at an inflection point in many ways the should spend time carefully considering what we want the future of the language to look like. I don’t think C or C++ will be irrelevant any time soon, but I do worry they’re heading that direction and will continue to do so without course correction. No programming language is “too big to fail”, not even something as venerable and ubiquitous as C.