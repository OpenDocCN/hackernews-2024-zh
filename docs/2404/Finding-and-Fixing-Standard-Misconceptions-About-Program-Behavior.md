<!--yml
category: 未分类
date: 2024-05-27 13:09:55
-->

# Finding and Fixing Standard Misconceptions About Program Behavior

> 来源：[https://blog.brownplt.org/2024/04/12/behavior-misconceptions.html](https://blog.brownplt.org/2024/04/12/behavior-misconceptions.html)

*Posted on 12 April 2024.*

A large number of modern languages — from Java and C# to Python and JavaScript to Racket and OCaml — share a common semantic core:

*   variables are lexically scoped
*   scope can be nested
*   evaluation is eager
*   evaluation is sequential (per “thread”)
*   variables are mutable, but first-*order*
*   structures (e.g., vectors/arrays and objects) are mutable, and first-*class*
*   functions can be higher-order, and close over lexical bindings
*   memory is managed automatically (e.g., garbage collection)

We call this the *Standard Model of Languages* (SMoL).

SMoL potentially has huge pedagogic benefit:

*   If students master SMoL, they have a good handle on the core of several of these languages.

*   Students may find it easier to port their knowledge between languages: instead of being lost in a sea of different syntax, they can find familiar signposts in the common semantic features. This may also make it easier to learn new languages.

*   The *differences* between the languages are thrown into sharper contrast.

*   Students can see that, by going beyond syntax, there are several big semantic ideas that underlie all these languages, many of which we consider “best practices” in programming language design.

We have therefore spent the past four years working on the pedagogy of SMoL:

*   Finding errors in the understanding of SMoL program behavior.

*   Finding the (mis)conceptions behind these errors.

*   Collating these into clean instruments that are easy to deploy.

*   Building a *tutor* to help students correct these misconceptions.

*   Validating all of the above.

We are now ready to present a checkpoint of this effort. We have distilled the essence of this work into a tool:

[The SMoL Tutor](https://smol-tutor.xyz/)

It identifies and tries to fix student misconceptions. The Tutor assumes users have a baseline of programming knowledge typically found after 1–2 courses: variables, assignments, structured values (like vectors/arrays), functions, and higher-order functions (`lambda`s). Unlike most tutors, instead of *teaching* these concepts, it investigates how well the user actually *understands* them. Wherever the user makes a mistake, the tutor uses an educational device called a *refutation text* to help them understand where they went wrong and to correct their conception. The Tutor lets the user switch between multiple syntaxes, both so they can work with whichever they find most comfortable (so that syntactic unfamiliarity or discomfort does not itself become a source of errors), and so they can see the semantic unity beneath these languages.

Along the way, to better classify student responses, we invent a concept called the *misinterpreter*. A misinterpreter is an intentionally incorrect interpreter. Concretely, for each misconception, we create a corresponding misinterpreter: one that has the same semantics as SMoL *except* on that one feature, where it implements the misconception instead of the correct concept. By making misconceptions executable, we can mechanically check whether student responses correspond to a misconception.

There are many interesting lessons here:

*   Many of the problematic programs are likely to be startlingly simple to experts.

*   The combination of state, aliasing, and functions is complicated for students. (Yet most *introductory* courses plunge students into this maelstrom of topics without a second thought or care.)

*   Misinterpreters are an interesting concept in their own right, and are likely to have value independent of the above use.

In addition, we have not directly studied the following claims but believe they are well warranted based on observations from this work and from experience teaching and discussing programming languages:

*   In SMoL languages, local and top-level bindings behave the same as the binding induced by a function call. However, students often do not realize that these have a uniform semantics. In part this may be caused by our focus on the “call by” terminology, which focuses on calls (and makes them seem special). We believe it would be an improvement to replace these with “bind by”.

*   We also believe that the terms “call-by-value” and “call-by-reference” are so hopelessly muddled at this point (between students, instructors, blogs, the Web…) that finding better terminology overall would be helpful.

*   The way we informally talk about programming concepts (like “pass a variable”), and the syntactic choices our languages make (like `return x`), are almost certainly also sources of confusion. The former can naturally lead students to believe variables are being aliased, and the latter can lead them to believe the *variable*, rather than its *value*, is being returned.

For more details about the work, see [the paper](https://cs.brown.edu/~sk/Publications/Papers/Published/lk-smol-tutor/). The paper is based on an old version of the Tutor, where all programs were presented in parenthetical syntax. The Tutor now supports multiple syntaxes, so you don’t have to worry about being constrained by that. Indeed, it’s being used right now in a course that uses Scala 3.

Most of all, the [SMoL Tutor](https://smol-tutor.xyz/) is free to use! We welcome and encourage instructors of programming courses to consider using it — you may be surprised by the mistakes your students make on these seemingly very simple programs. But we also welcome learners of all stripes to give it a try!