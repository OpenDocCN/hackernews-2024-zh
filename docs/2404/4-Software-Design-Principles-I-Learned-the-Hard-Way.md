<!--yml
category: 未分类
date: 2024-05-27 13:36:12
-->

# 4 Software Design Principles I Learned the Hard Way

> 来源：[https://read.engineerscodex.com/p/4-software-design-principles-i-learned](https://read.engineerscodex.com/p/4-software-design-principles-i-learned)

*Engineer’s Codex is a publication about real-world software engineering.*

* * *

I recently built and designed a massive service that (finally) launched successfully last month. During the design and implementation process, I found that the following list of “rules” kept coming back up over and over in various scenarios.

These rules are common enough that I daresay that at least one of them will be useful for a project that any software engineers reading this are *currently* working on. But if you can’t apply it directly now, I hope that these principles are a useful thought exercise that you are free to comment on below or challenge directly too.

One thing I will note here is that of course - each “principle” has a time and place. Nuance is necessary, as always. These are conclusions that I find myself erring towards *in general* because oftentimes, *the opposite that is the default* that I see when I’m reviewing code or reading design docs.

* * *

To be a top software engineer, you need to know a lot. But how do you know what you don't know? **[SWE Quiz](https://swequiz.com/?utm_source=codex) is a compilation of 450+ software engineering and system design questions covering databases, authentication, caching, etc.**

**They’ve been created by engineers from Google, Meta, Apple, and more.**

It’s helped many of my peers (including myself) pass the “software trivia questions” during interviews and feel more confident at work.

[Get Lifetime Access to SWE Quiz](https://swequiz.com)

* * *

If there’s two sources of truth, one is probably wrong. If it’s not wrong, it’s not wrong… yet. 

Basically, if you’re trying to maintain a piece of state in two different locations within the same service… just don’t. It’s better to try to just reference the same state wherever you can. For example, if you’re maintaining a frontend application and have a bank balance that is from the server, I’ve seen enough sync bugs in my time that I always want to get that balance from the server. If there is some balance that is derived from that, such as “spendable balance” versus “total” (for example, some banks make you keep a minimum balance), then that “spendable balance” should be derived on-the-fly rather than stored separately. Else, you’ll now have to update both balances whenever a transaction happens.

In general, if there is a piece of data that is derived from another value, then that value should be derived rather than stored. Storing that value leads to synchronization bugs. (Yes, I know this isn’t always possible. There will always be other factors in play, like the expense of the derivation. At the end of the day, it’s a tradeoff.)

We’ve heard of DRY (Don’t Repeat Yourself) and now I present to you PRY (Please Repeat Yourself).

Far too many times I’ve seen code that looks *mostly* the same try to get abstracted out into a “re-usable” class. The problem is, this “re-usable” class gets one method added to it, then a special constructor, then a few more methods, until it’s this giant Frankenstein of code that serves multiple different purposes and the original purpose of the abstraction no longer exists.

A pentagon may be *similar-looking* to a hexagon, but there is still enough of a difference that they are *absolutely not the same*. 

I’m also guilty of spending way too much time trying to make things reusable, when a bit of code duplication works perfectly fine. (Yes, you have to write more tests and it doesn’t scratch the “refactoring” itch, but oh well.)

Mocks. I have a love-hate relationship with mocks. My favorite one-liner from a [Reddit](https://www.reddit.com/r/programming/comments/1cckf07/comment/l1b66ok/) discussion about this post was “with mocks, we sell test fidelity for ease of testing.”

Mocks are great when I have to write unit tests to test something quickly and don’t want to mess with “prod-level” code. Mocks are not great when prod breaks because as it turns out - something you mocked broke deeper down the stack, even though that “deeper down the stack” is owned by another team. It doesn’t matter because it was your service that broke so it's your responsibility to fix it.

Writing tests is hard. The line between unit tests and integration tests is blurrier than you think. Knowing what to mock and not mock is subjective.

It’s much nicer to find things while developing rather than in prod. As I continue writing software, I try to stay away from mocks if possible. Tests being a bit more heavyweight is completely worthwhile when it comes to a much higher reliability. If mocks are really required by my code reviewer, I’d rather write more (and maybe even redundant) tests rather than skip out on tests. Even if I can’t use a real dependency in a test, I will still try to use other options first before mocks, like a local server.

[Google’s “Testing on the Toilet” has a good note on this from 2013.](https://testing.googleblog.com/2013/05/testing-on-toilet-dont-overuse-mocks.html) They note that overusing mocks causes:

Computers are VERY fast. In the optimization game, it’s super popular to instantly throw caching and store everything in a database immediately. I think this is probably the *end state* of most successful software products and services. Of course, most services will need some sort of state, but it’s important to figure out what is truly necessary storage-wise versus what can be derived on-the-fly.

In the “v1” of something, I’ve found that minimizing as much mutable state as possible gets you pretty far. It lets you develop faster because you don’t have to worry about sync bugs, conflicting data, and stale state. It also lets you develop functionality piece-by-piece, rather than introducing too much at once. Machines are fast enough today where doing a few redundant calculations is totally fine. If machines are supposedly “replacing us” soon, then they can handle a few extra work units of calculations.