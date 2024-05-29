<!--yml
category: 未分类
date: 2024-05-27 15:03:39
-->

# Dave Mason - Zag Smalltalk - 30 November 2022 on Vimeo

> 来源：[https://vimeo.com/802502826](https://vimeo.com/802502826)

Dave Mason ( [sarg.ryerson.ca/dmason/](http://sarg.ryerson.ca/dmason/) ) has been a professor of Computer Science at Toronto Metropolitan University (previously known as Ryerson) for 41 years. He has done research on operating systems, software reliability and programming languages. Current research is mostly around Smalltalk and other dynamic languages. If forced to program very low level projects such as virtual machines, he is willing to use Zig or Rust, but for any other purpose, he insists on using higher productivity languages - primarily Smalltalk.

Zag Smalltalk ( [github.com/dvmason/Zag-Smalltalk](https://github.com/dvmason/Zag-Smalltalk) ) is a principle-based Smalltalk VM. "Principled" means that the only 3 operations are: message send, assignment, and return.

1) it is designed from the ground up to use multi-processing, to leverage multi-core systems.
2) There is no special-casing of methods like ifTrue:ifFalse or whileTrue:, although of course some methods are implemented by primitive methods. It does aggressive inlining of methods and blocks.
3) “Source” code is maintained as ASTs.
4) Compiled code runs in a dual form of threaded code and JIT’ed machine code that seamlessly interoperate.
5) It has a partially-copying and partially-non-moving garbage collector.
6) It keeps many more values as immediate, including symbols.
7) It uses a single-level dispatch mechanism to implement Smalltalk dispatch semantics

The research question is, "Can this be made fast enough to be competitive?” Preliminary results are encouraging.

Recorded 30 November 2022.

The UK Smalltalk User Group is a group or professionals and hobbists who share an interest in the Smalltalk programming languages and related technologies. You can find us at
[uksmalltalk.org/](https://www.uksmalltalk.org/) and [twitter.com/uksmalltalk](https://twitter.com/uksmalltalk)