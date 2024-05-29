<!--yml
category: 未分类
date: 2024-05-27 14:45:04
-->

# Heresy II – Comments Are Code – Responsible Automation

> 来源：[https://responsibleautomation.wordpress.com/2018/02/01/heresy-ii-comments-are-code/](https://responsibleautomation.wordpress.com/2018/02/01/heresy-ii-comments-are-code/)

I hold an unpopular stance: code comments are as important as the code itself. OK, maybe that’s a little overstated, but I maintain that code comments are critical to code readability and maintainability.

Before you see if I weigh more than a duck, please hear me out…

When I was a college undergrad, I had the privilege of working on a project with a graduate professor and some grad students. The core of the project was creating a software package to perform type checking on a non-typed, functional language; this was both weird and cool…but I digress. As part of the project, we were required to comment our code. My teammates and I diligently went about documenting each function and other interesting parts of the code we’d created. When we’re finished, we proudly submitted our code to the professor.

The next day, the professor asked us to come to her office. She proceeded to scold us on our commenting…I’ll paraphrase: I can read code, don’t tell me what the code is doing, tell me why the code is doing that. She sent us home to think and told us to come back the next day and re-comment everything.

This was some of the best advice I’ve ever received regarding coding. Let me tell you why.

As you can probably deduce, our code comment explained *what* the code was doing. “Go through the list, added each name to the pending transaction list” or some such literal thing. As my professor correctly stated, she (and presumably whoever else is looking at the code) can read code; she understood what it was doing, but not necessarily *why*. The comments were missing *context*; they didn’t explain why we’d chosen that implementation and, just as importantly, why we’d not chosen a different implementation.

Please, before you get out the duck-weighing equipment, allow me to continue…

I’m a big fan of self-documenting code. I’m like most developers: I don’t want to document my code. I want to write code that is self-explanatory, straightforward, easy to use, and to maintain. One of the last things I want to do with code is go through and write a bunch of comments. Self-documenting code is nice and all, but at some point, as with most things it can be taken to extremes. For example, I could write a method name something like

```
searchListInReverseOrderBecauseTheValueWeWantIsUsuallyAtTheEnd()
```

I consider that a preposterous function or method name. Sure, it’s descriptive. Sure, nowadays, most of us use IDEs that have code completion so we don’t really need to type all those characters. I still think, however, that it is far clearer to include a comment before that “reverse search” loop that says something like

```
// Search list in reverse; the value we want is usually at the end
```

What’s the big deal about going through the list backward anyway? Perhaps nothing. But if it’s not important, then why not go through the list in a more intuitive way, i.e. starting at index 0? If it is important to go through backward, perhaps it’s important enough to document why. If we don’t comment the why, the reason, the context, we run the risk of a future author changing the algorithm and unwittingly causing some kind of issue because the algorithm ran “better” when searching the list in reverse.

But wait! The comments will get out of sync with the code! Well, they shouldn’t. When we perform our code reviews, we need to review the comments as well as the code. Comments should be correct, understandable, and valuable; we need to keep these points in mind during our reviews.

When should we consider using code comments?

*   The code is straightforward but not intuitive (e.g. the *why* is not evident from the code itself)
*   When the code is not straightforward for deliberate reasons, such as writing it in a more straightforward or self-documenting manner would sacrifice performance
*   A comment will increase readability or maintainability
*   A comment will remind us where the code came from, such as a book or StackOverflow URL.
*   Generating documentation (see example below)

When should we consider not commenting?

*   When both the behavior and the context are evident in the code
*   When simplifying the code or abstracting into a “helper” method would make the code self-documenting without sacrificing value (e.g. performance, maintainability, readability, etc.)

At the risk of being hypocritical, I detest standard comment blocks at the beginning of each file, class, and method. It seems like valueless busy work, especially since I do try to make my code rather descriptive. Then one day, when I was grousing about this subject, a less experienced member of my team (yes, we can learn from people of all experience levels) said: “you know, those comment blocks are where that convenient IntelliSense documentation comes from, right?” I’d not considered that. I silently cursed him for being right. I still detest doing that kind of commenting, but I now understand the value it provides, so I keep the grousing to myself…well, except for now.

Yes, it’s generally better for code to be self-documenting. In lieu of being dumb for dumb’s sake, however, perhaps add some code comments to provide context for others…or perhaps your future self.

*Like this? Catch me at an [upcoming event](https://responsibleautomation.wordpress.com/upcoming-events/)!*