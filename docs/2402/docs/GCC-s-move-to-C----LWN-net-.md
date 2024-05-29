<!--yml
category: 未分类
date: 2024-05-27 15:02:20
-->

# GCC's move to C++ [LWN.net]

> 来源：[https://lwn.net/Articles/542457/](https://lwn.net/Articles/542457/)

| **Please consider subscribing to LWN**Subscriptions are the lifeblood of LWN.net. If you appreciate this content and would like to see more of it, your subscription will help to ensure that LWN continues to thrive. Please visit [this page](/subscribe/) to join up and keep LWN on the net. |

March 13, 2013

This article was contributed by Linda Jacobson

The GNU Compiler Collection (GCC) was, from its inception, written in C and compiled by a C compiler. Beginning in 2008, an effort was undertaken to change GCC so that it could be compiled by a C++ compiler and take advantage of a subset of C++ constructs. This effort was jump-started by [a presentation by Ian Lance Taylor [PDF]](http://airs.com/ian/cxx-slides.pdf) at the June 2008 GCC summit. As with any major change, this one had its naysayers and its problems, as well as its proponents and successes.

#### Reasons

Taylor's slides list the reasons to commit to writing GCC in C++:

*   C++ is well-known and popular.
*   It's nearly a superset of C90, which GCC was then written in.
*   The C subset of C++ is as efficient as C.
*   C++ "supports cleaner code in several significant cases." It never requires "uglier" code.
*   C++ makes it harder to break interface boundaries, which leads to cleaner interfaces.

The popularity of C++ and its superset relationship to C speak for themselves. In stating that the C subset of C++ is as efficient as C, Taylor meant that if developers are concerned about efficiency, limiting themselves to C constructs will generate code that is just as efficient. Having cleaner interfaces is one of the main advantages of C++, or any object-oriented language. Saying that C++ never requires "uglier" code is a value judgment. However, saying that it supports "cleaner code in several significant cases" has a deep history, best demonstrated by gengtype.

[According to the GCC Wiki](http://gcc.gnu.org/wiki/gengtype):

As C does not have any means of reflection [...] gengtype was introduced to support some GCC-specific type and variable annotations, which in turn support garbage collection inside the compiler and precompiled headers. As such, gengtype is one big kludge of a rudimentary C lexer and parser.

What had happened was that developers were emulating features such as garbage collection, a vector class, and a tree class in C. This was the "ugly" code to which Taylor referred.

In his slides, Taylor also tried to address many of the initial objections: that C++ was slow, that it was complicated, that there would be a bootstrap problem, and that the Free Software Foundation (FSF) wouldn't like it. He addressed the speed issue by pointing out that the C subset of C++ is as efficient as C. As far as FSF went, Taylor wrote, "The FSF is not writing the code."

The complexity of a language is in the eye of the beholder. Many GCC developers were primarily, or exclusively, C programmers, so of necessity there would be a time period in which they would be less productive, and/or might use C++ in ways that negated all its purported benefits. To combat that problem, Taylor hoped to develop coding standards that limited development to a subset of C++.

The bootstrap problem could be resolved by ensuring that GCC version *N-1* could always build GCC version *N*, and that they could link statically against `libstdc++`. GCC version *N-1* must be linked against `libstdc++` *N-1* while it is building GCC *N* and `libstdc++` *N*; GCC *N*, in turn, will need `libstdc++` *N*. Static linking ensures that each version of the compiler runs with the appropriate version of the library.

For many years prior to 2008, there had been general agreement to restrict GCC code to a common subset of C and C++, according to Taylor (via email). However, there was a great deal of resistance to replacing the C compiler with a C++ compiler. At the 2008 GCC summit, Taylor took a poll on how large that resistance was, and approximately 40% were opposed. The C++ boosters paid close attention to identifying and addressing the specific objections raised by C++ opponents (speed, memory usage, inexperience of developers, and so on), so that each year thereafter the size of the opposition shrank significantly. Most of these discussions took place at the GCC summits and via unlogged IRC chats. Therefore, the only available record is in the [GCC mailing list archives](http://gcc.gnu.org/ml/gcc).

#### First steps

The first step, a proper baby step, was merely to try to compile the existing C code base with a C++ compiler. While Taylor was still at the conference, he [created a gcc-in-cxx branch](http://gcc.gnu.org/ml/gcc/2008-06/msg00385.html) for experimenting with building GCC with a C++ compiler. Developers were quick to announce their intention to work on the project. The initial build attempts encountered many errors and warnings, which were then cleaned up.

In June 2009, almost exactly a year from proposing this switch, Taylor reported that phase one was complete. He configured GCC with the switch `enable-build-with-cxx` to cause the core compiler to be built with C++. A bootstrap on a single target system was completed. Around this time, the separate cxx branch was merged into the main GCC trunk, and people continued their work, using the `enable-build-with-cxx` switch. (However, the separate branch was revived on at least one occasion for experimentation.)

In May 2010, there was a [GCC Release Manager Q&A on IRC](http://gcc.gnu.org/wiki/Release%20Manager%20Q%26A?action=AttachFile&do=view&target=rmqa-20100527.txt). The conclusion from that meeting was to request permission from the GCC Steering Committee to use C++ language features in GCC itself, as opposed to just compiling with a C++ compiler. Permission was granted, with agreement also coming from the FSF. Mark Mitchell [announced the decision](http://gcc.gnu.org/ml/gcc/2010-05/msg00705.html) in an email to the GCC mailing list on May 31, 2010\.

In that thread, [Jakub Jelinek](http://gcc.gnu.org/ml/gcc/2010-05/msg00746.html) and [Vladimir Makarov](http://gcc.gnu.org/ml/gcc/2010-05/msg00744.html) expressed a lack of enthusiasm for the change. However, as Makarov put it, he had no desire to start a flame war over a decision that had already been made. That said, he recently shared via email that his primary concern was that the GCC community would rush into converting the GCC code base to C++ "<q>instead of working on more important things for GCC users (like improving performance, new functionality and so on). Fortunately, it did not happen.</q>"

Richard Guenther was concerned about [creating a tree class hierarchy](http://gcc.gnu.org/ml/gcc/2010-05/msg00745.html):

It's a lot of work (tree extends in all three Frontends, middle-end and backends). And my fear is we'll only get a halfway transition - something worse than no transition at all.

The efforts of the proponents to allay concerns, and the "please be careful" messages from the opponents give some indication of the other concerns. In addition to the issues raised by Taylor at the 2008 presentation, Jelinek mentioned memory usage. Others, often as asides to other comments, worried that novice C++ programmers would use the language inappropriately, and create unmaintainable code.

There was much discussion about coding standards in the thread. Several argued for existing standards, but others pointed out that they needed to define a "safe" subset of C++ to use. There was, at first, little agreement about which features of C++ were safe for a novice C++ developer. Taylor proposed a set of coding standards. These were amended by Lawrence Crowl and others, and then were [adopted](http://gcc.gnu.org/codingconventions.html). Every requirement has a thorough rationale and discussion attached. However, the guiding principle on maintainability is not the coding standard, but one that always existed for GCC: the maintainer of a component makes the final decision about any changes to that component.

#### Current status

Currently, those who supported the changes feel their efforts provided the benefits they expected. No one has publicly expressed any dissatisfaction with the effort. Makarov was relieved that his fear that the conversion effort would be a drain on resources did not come to pass. In addition, he cites the benefits of improved modularity as being a way to make GCC easier to learn, and thus more likely to attract new developers.

As far as speed goes, Makarov noted that a bootstrap on a multi-CPU platform is as fast as it was for C. However, on uniprocessor platforms, a C bootstrap was 30% faster. He did not speculate as to why that is. He also found positive impacts, like converting to C++ hash tables, which sped up compile time by 1-2%. This last work is an ongoing process, that Lawrence Crowl last [reported](http://gcc.gnu.org/ml/gcc-patches/2012-10/msg00217.html) on in October 2012\. In keeping with Makarov's concerns, this work is done slowly, as people's time and interests permit.

Of the initial desired conversions (gengtype, tree, and vector), vector support is provided using C++ constructs (i.e., a class) and gengtype has been rewritten for C++ compatibility. Trees are a different matter. Although they have been much discussed and volunteered for several times, no change has been made to the code. This adds credence to the 2010 contention of Guenther (who has [changed his surname to Biener](http://comments.gmane.org/gmane.comp.gcc.patches/272850)) that it would be difficult to do correctly. Reached recently, Biener stated that he felt it was too early to assess the impact of the conversion because, compared to the size of GCC, there have been few changes to C++ constructs. On the negative side, he noted (as others have) that, because of the changes, long-time contributors must relearn things that they were familiar with in the past.

In 2008, 2009, and 2010, (i.e., at the beginning and after each milestone) Taylor provided formal plans for the next steps. There is no formal plan going forward from here. People will use C++ constructs in future patches as they deem necessary, but not just for the sake of doing so. Some will limit their changes to the times when they are patching the code anyway. Others approach the existing C code with an eye to converting code to C++ wherever it makes the code clearer or more efficient. Therefore, this is an ongoing effort on a meandering path for the foreseeable future.

As the C++ project has progressed, some fears have been allayed, while some developers are still in a holding pattern. For them it is too soon to evaluate things definitively, and too late to change course. However, the majority seems to be pleased with the changes. Only time will tell what new benefits or problems will arise.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/542457/)

to post comments)