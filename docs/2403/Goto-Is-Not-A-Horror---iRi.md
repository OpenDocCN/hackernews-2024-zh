<!--yml
category: 未分类
date: 2024-05-29 12:34:09
-->

# Goto Is Not A Horror - iRi

> 来源：[https://jerf.org/iri/post/2024/goto/](https://jerf.org/iri/post/2024/goto/)

In 1968, Edsger Dijkstra published a classic letter which was titled “Go To Statement Considered Harmful”. I think the headline buries the lede, because it’s actually an exhortation to structured programming in general, but it is not incorrect. `goto` is indeed considered harmful in the letter.

Dijkstra’s letter was completely correct. History has bourne him out. He won.

He *won*.

*Past tense*.

The winning is over. It has been accomplished.

`goto` is *dead*. Dijkstra killed it.

That is, the `goto` that Dijkstra is talking about is dead. Your language does not have it. Your language hasn’t had it for decades. You do not have Dijkstra-`goto`.

It lives on in assembler. It can live there. It doesn’t have to bother anyone else.

The `goto` in your modern language is not the one Dijkstra was talking about. Dijkstra was talking about a `goto` that could leap anywhere. Literally anywhere.

Your language does not have that `goto`. Your language has a `goto` that has been thoroughly constrained by the structured programming paradigm. You can not `goto` from one function into another, in the middle of an `if` statement, in the middle of a `for` loop.

Not only could the one Dijkstra was complaining about do exactly that, the real complaint of the letter if you read it carefully is that `goto` was itself the organizational principle of the programs of the time… which means one of the reasons `goto` is so harmful is that it prevented functions from existing in the first place! You can’t have the guarantees functions provide in the presence of an unconstrainted `goto`.

That is expressed from a modern point of view. The letter will talk about the ability to tell where the program is based on indices of execution. The letter generally assumes you understand structured programming and how that criticism applies. Fortunately, you do understand structured programming, since it is about all you do.

So stop applying the criticisms of the 1960s `goto` to modern `goto`. Stop acting like a single use of `goto` in a function means you’re a bad programmer who does not Get It. `DELETE FROM shibboleths WHERE value LIKE "%goto%"`. Dumping on modern `goto` is not smart and wise and a sign of a good programmer, it’s a sign you don’t understand why `goto` is… or rather, *was*… bad.

Does this mean you should use it a lot more? No. Structured programming has a multi-decade track record of success. The vast, vast majority of the time structured programming does the job quite well.

It’s hard to mess up a function in a modern programming language with `goto`. Probably a lot of the “but what if I…?” questions that leap to mind are already forbidden. You will find your language already forbids jumping into the middle of `for` loops from the outside. You will find your language already forbids jumping over variable initializations, if they are statements in your language.

You can of course make a mess of a function with extensive (over)use of `goto`, but whoop de do. I’ve got hundreds of options for making messes of functions; consult your local non-trivial codebase for thousands of examples. Good odds precisely *zero* of those thousands of examples involve `goto` in any form. `goto` is not a special menace. I’d say [stringly typing](https://www.hanselman.com/blog/stringly-typed-vs-strongly-typed) is *multiple orders of magnitude* a greater menace to your code than `goto`, at least a good 6 or 7\. There’s plenty of programmers who know to say words about how bad `goto` is but are blissfully unaware of how destructive it is to program structure to blast strings and ints everywhere without further qualification.

In fact… hang on to your hats… when I see modern code that uses `goto`, I actually find that to be a marker that it was probably written by *highly skilled* programmers. Because they are the ones who understand the issues, understand that the modern anti-`goto` consensus is wrong, and understood that they had a case where `goto` was the best solution.

Stop freaking out about `goto`. Stop acting like the 1960s `goto` is the same as today’s `goto`.

Is this important? Heck no. This is just a blog post topic of a personal pet peeve.

But if you want to take something practical away from it, [re-read the letter](https://dl.acm.org/doi/10.1145/362929.362947) (or read it for the first time), but make an exercise of editing out the references to `goto` and read everything else around it. You will learn a lot about the water through which you as a programming fish swim. You will learn a lot about why the entire programming industry collectively decided to swim in this water. You will learn what stack traces *really* mean. I’ve lost track of the link, but I once read a blog post about why stack traces should really have the for loop indices in them too; you will learn why that’s an interesting idea that may be something the entire programming community has overlooked for decades. Then at least this won’t be a pointless whiny blog post for you.