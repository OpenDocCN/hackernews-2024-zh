<!--yml
category: 未分类
date: 2024-05-27 13:38:45
-->

# Good Ideas in Computer Science ・ Daniel Hooper

> 来源：[https://danielchasehooper.com/posts/good-ideas-in-cs/](https://danielchasehooper.com/posts/good-ideas-in-cs/)

Programmers love arguing for their favorite technologies. C++ vs Rust. Mac vs PC. These arguments overshadow the victories of Computer Science — the ideas that we all agree on. To unearth these ideas, I recently [asked a simple question](https://twitter.com/DanielcHooper/status/1778795850107424827) on Twitter/X:

> What ideas in computer science are universally considered good?

By “universally considered good” I mean they *aren’t* debated. Ideas so widespread and effective that you might not even think of them as being invented. Each idea may not be suitable in all situations, but you won’t find a programmer that thinks you should *never* use them. I intentionally focus on *ideas*, not *implementations*. For example: Unix contains many good ideas, but is not on the list because it is an implementation.

Here’s my list, including the year each idea appeared:

Intentionally excluded:

*   Garbage Collection

    There are many examples of teams fighting the garbage collector to hit performance goals^(. The [CPU/Memory performance gap](http://gec.di.uminho.pt/discip/minf/ac0102/1000Gap_Proc-Mem_Speed.pdf) necessitates control over memory to have performant code.)

*   Databases

    Databases are more than just one idea, with many ways to combine those ideas into a “database shape”. Some good ideas in databases: [Structured query language](https://en.wikipedia.org/wiki/SQL), [B-trees](https://en.wikipedia.org/wiki/B-tree), [ACID transactions](https://en.wikipedia.org/wiki/ACID).

*   Other data structures and algorithms

    There are too many to list. Few are as universal as arrays and hashmaps, which appear in almost all programs.

*   Object Oriented Programming

    There is a large group of programmers that do not consider Object Oriented Programming good^(. I recommend [data oriented design](https://github.com/dbartolini/data-oriented-design) as a replacement worldview.)

By 1974, 50 years ago, we had most of what we call modern computing. Today’s fundamentals are the same — a C programmer from 1974 would feel at home on a modern computer except for the alien-like speed. I hope we have new ideas that in 50 years will be universally considered good.

Discuss on [Twitter](https://twitter.com/DanielcHooper/status/1782446647047311466)
Discuss on [Lobste.rs](https://lobste.rs/s/kruxyr/good_ideas_computer_science)