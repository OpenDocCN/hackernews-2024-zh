<!--yml
category: 未分类
date: 2024-05-27 14:42:46
-->

# My talk on CSS runtime performance | Read the Tea Leaves

> 来源：[https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/](https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/)

17 Jan

## My talk on CSS runtime performance

Posted January 17, 2023 by Nolan Lawson in [performance](https://nolanlawson.com/tag/performance/), [Web](https://nolanlawson.com/category/web/). [3 Comments](https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/#comments)

A few months ago, I gave a talk on CSS performance at [performance.now](https://perfnow.nl/) in Amsterdam. The recording is available online:

 [https://www.youtube.com/embed/nWcexTnvIKI?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent](https://www.youtube.com/embed/nWcexTnvIKI?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent)

VIDEO

(You can also read [the slides](https://nolanlawson.github.io/css-talk-2022/).)

This is one of my favorite talks I’ve ever given. It was the product of months (honestly, years) of research, prompted by a couple questions:

1.  What is the fastest way to implement [scoped styles](https://rfcs.lwc.dev/rfcs/lwc/0116-light-dom-scoped-styles)? (Surprisingly few people seem to ask this question.)
2.  Does using shadow DOM improve style performance? (Short answer: [yes](https://nolanlawson.com/2022/06/22/style-scoping-versus-shadow-dom-which-is-fastest/).)

To answer these questions (and more), I did a bunch of research into how browsers work under the hood. This included combing through [old Servo discussions from 2013](https://github.com/servo/servo/wiki/Css-selector-matching-meeting-2013-07-19), reaching out to browser developers like [Manuel Rego Casasnovas](https://blogs.igalia.com/mrego/) and [Emilio Cobos Alvarez](https://github.com/emilio), reading [browser PRs](https://github.com/WebKit/WebKit/commit/596fdf7c2cec599f8c826787363c54c4b008a7fe), and writing lots of benchmarks.

In the end, I’m pretty satisfied with the talk. My main goal was to shine a light on all the heroic work that browser vendors have done over the years to make CSS so performant. Much of this stuff is intricate and arcane (like [Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter)), but I hoped that with some simple diagrams and animations, I could bring this work to life.

The two outcomes I’d love to see from this talk are:

1.  Web developers spend more time thinking about and measuring CSS performance. (Pssst, check out the `SelectorStats` section of [my guide to Chrome tracing](https://nolanlawson.com/2022/10/26/a-beginners-guide-to-chrome-tracing/)!)
2.  Browser vendors provide better DevTools to understand CSS performance. (In the talk, I pitch this as a [`SQL EXPLAIN`](https://www.postgresql.org/docs/current/sql-explain.html) for CSS.)

What I *didn’t* want to do in this talk was rain on anybody’s parade who is trying to do sophisticated things with CSS. More and more, I am seeing [ambitious usage](https://www.bram.us/2023/01/12/sibling-scopes-in-css-thanks-to-has/) of new CSS features like [`:has`](https://developer.mozilla.org/en-US/docs/Web/CSS/:has) and [container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries), and I don’t want people to feel like they should avoid these techniques and limit themselves to classes and IDs. I just want web developers to consider the cost of CSS, and to get more comfortable with using the DevTools to understand which kinds of CSS patterns may be slowing down their website.

I also got some good feedback from browser DevTools folks after my talk, so I’m hopeful for the future of CSS performance. As techniques like shadow DOM and [native CSS scoping](https://css-tricks.com/early-days-for-css-scoping/) become more widespread, it may even mitigate a lot of my worries about CSS perf. In any case, it was a fascinating topic to research, and I hope that folks were intrigued and entertained by my talk.