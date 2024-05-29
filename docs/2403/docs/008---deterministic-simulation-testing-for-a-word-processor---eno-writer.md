<!--yml
category: 未分类
date: 2024-05-29 12:38:17
-->

# 008 - deterministic simulation testing for a word processor | eno writer

> 来源：[https://www.eno-writer.com/008-deterministic-simulation-testing-for-a-word-processor/](https://www.eno-writer.com/008-deterministic-simulation-testing-for-a-word-processor/)

<main>

# 008 - deterministic simulation testing for a word processor

*15 Mar, 2024*

It's hard to overstate the cost of having bugs in your software. At best, they give your users little paper cuts during an otherwise positive experience, at worst they set your users back, leaving them wishing they had never used your software in the first place. Behind the scenes, the costs of bugs continue. The time it takes to fix a bug that has made its way into production vastly exceeds the time it takes to create it. It's not uncommon for a team to spend more time fixing bugs in a new feature than they spent actually writing it.

I've taken a lot of inspiration over the last couple years from [TigerBeetle](https://tigerbeetle.com/). TigerBeetle is a startup that is building a financial accounting database from scratch in zig. You can imagine that avoiding bugs is top of mind for a company that is building a database from scratch to store billions (trillions?) of dollars of financial transactions. Joran Dirk Greef, CEO/founder, appreciates the difficulty of the task more than anyone, pointing out that Postgres (an extremely popular database), went almost 20 years with an [undiscovered bug relating to data persistence](https://news.ycombinator.com/item?id=19119991).

Joran's team is taking a multi-prong approach to ensuring TigerBeetle is bulletproof by the time it enters production use. One of the most interesting prongs is something they call deterministic simulation testing. They have written a program called the VOPR ("vopper") which simulates the operation of a TigerBeetle cluster and runs a determined sequence of actions rapidly to test whether it behaves correctly. The sequence of actions is generated with a random seed which means this program can be run over and over again producing a different sequence each time. Joran says its a time machine that allows them to simulate years of usage in a matter of hours.

I wanted to find a way to bring deterministic simulation testing to eno. I also wanted to do this early, while it was still a fairly small undertaking. I also knew it needed to have a pithy name like the VOPR, so I called it Lincoln. Lincoln is a great writer but he is quite error prone. Every time he types something new in, he forgets a piece of it and accidentally adds some extra text. He always notices his mistakes eventually and goes back and fixes them. He also saves his work along the way, keeping a history of versions.

Here's Lincoln writing the Gettysburg address in eno. On the left is what the end user would see. On the right is eno's internal representation of documents. It uses a tree data structure which I call the BFT (the "T" stands for tree).

So, was writing Lincoln worth it? Absolutely yes. I am actually surprised with how effective this very humble script has been already. When I first got Lincoln working properly, it immediately uncovered two bugs which caused eno to crash. I was able to take the error output from these crashes and create a unittest that isolated the specific problems in the BFT. The bugs were then trivial to fix. Until now, my process for finding bugs has been to run eno and try typing stuff into it in various ways and see if it crashes. The issue with this is I have to invent ways in which I think eno might be broken. Any bug I am not creative enough to think of, I will miss. With Lincoln, I just run the program and before I know it, one of the random things it did has uncovered a bug.

Once Lincoln was able to complete the Gettysburg address with no crashes, I started to instrument some high level performance measurements. One of my fears is that the BFT will not scale well as documents receive more and more edits. This proved to be true - my first run of Lincoln with instrumentation produced the following output.

```
total time: 0.8370s
avg edit time: 0.0017s
max edit time (483/483): 0.0098s 
```

Note how the longest edit was the last edit (483/483) and took roughly 5x the length of the average. I had anticipated this and knew one thing I would need to do is simplify the underlying data structure every time the user commits a new version of their document. Up until now, the BFT was holding every edit in history in a distinct tree node. I went ahead and implemented this and then ran Lincoln again:

```
total time: 0.0544s
avg lap time: 0.0001s
max lap time (434/483): 0.0004s 
```

Well, the max time is still fairly near the end and it's still about 4x the average. The good news is that all the numbers are about 90% lower than the previous benchmark. I can live with that for now.

I've just scratched the surface of what is possible but I am quite excited about what the future holds for Lincoln!

* * *

If you liked this post, please consider sharing it with a friend.

We also have an [RSS feed](/feed/)

[#software](/blog/?q=software) [#startups](/blog/?q=startups) [#zig](/blog/?q=zig)

</main>