<!--yml
category: 未分类
date: 2024-05-29 13:24:13
-->

# Disillusioned with Deno – Baldur Bjarnason

> 来源：[https://www.baldurbjarnason.com/2024/disillusioned-with-deno/](https://www.baldurbjarnason.com/2024/disillusioned-with-deno/)

This is a part of a series where I review the work I’ve done over the past couple of years.

1.  [*Two-year review: to plan a strategy you must first have a theory of how the hell things work*](/2024/2022-23-strategy-review-part-one/)
2.  [*Out of the Software Crisis: two year project review*](/2024/out-of-the-software-crisis-two-year-review/)
3.  [*Sunk Cost Fallacy: chasing a half-baked idea for much too long*](/2024/sunk-cost-fallacy-research-project/)
4.  [*The Intelligence Illusion: stepping into a pile of ‘AI’*](/2024/the-intelligence-illusion-stepping-into-a-pile-of-ai/)
5.  [*A print project retrospective: the biggest problem with selling print books is the software*](/2024/the-problem-with-print-is-software/)
6.  [*Thinking about print*](/2024/thinking-about-print/)
7.  [*Disillusioned with Deno*](/2024/disillusioned-with-deno/) (this page)
8.  [*An Uncluttered retrospective: Teachable is a mess and I need to pick a lane*](/2024/uncluttered-retrospective/)

* * *

It’s inevitable that once you decide to review your own business and media strategy, you start to rethink and review other decisions you’ve made in the past.

One of those decisions is [Deno](https://deno.com/), which I’ve used to most of my personal projects and experiments over the past couple of years. Most of these projects have been on the simpler side – command-line scripts or small websites. I made the [softwarecrisis.dev](https://softwarecrisis.dev/) newsletter landing page using [Lume and Deno](https://lume.land/), for example. I used Deno to make most of the prototypes for the research project that followed [Colophon Cards](https://www.colophon.cards/). It’s very useful.

There’s a lot to like.

Deno has:

*   Overall much better support for standard browser APIs than node.
*   APIs that aren’t based on standards hew a lot closer to browser conventions, making switching between the front and back end a lot less jarring.
*   An excellent standard library.
*   Much simpler installation. It comes in a single binary, making installation and deployment much simpler.
*   Most of the tooling you need: linter, test runner, benchmarking, code formatting, type checker, and documentation generation.
*   A sensible security model for running code.

If you have a task that only requires the standard library or the standard tooling, using Deno is a dream – simplicity itself.

If that task requires something *more* than that, you quickly find yourself in trouble.

The company behind Deno has tried to conquer this problem with a two-pronged attack:

1.  Build Deno-specific versions of most of the common tools and projects that web projects might need.
2.  Hastily roll out node and npm compatibility for their runtime.

This has meant that a single startup has taken on the responsibility of building their own [scalable hosting](https://deno.com/deploy), [persistent storage product](https://deno.com/kv), [persistent execution queue](https://docs.deno.com/kv/manual/queue_overview), [realtime notification](https://docs.deno.com/kv/manual/#watching-for-updates-in-deno-kv), [scheduled execution](https://docs.deno.com/kv/manual/cron), [front-end React framework](https://fresh.deno.dev/), and no doubt a few more by the time you read this post.

They’re even planning to build their own full-featured package registry for [Deno 2.0.](https://www.reddit.com/r/Deno/comments/15nv8yv/deno_20_previewed_at_seattlejs_conf/) (Also, it’s quite annoying how much information out there on the future of Deno is only available in video.)

Add to that the attempt to implement full node and npm compatibility for their runtime.

Basically:

1.  Your business model is hosting.
2.  But none of the tools and projects people commonly use are available for your new runtime and hosting environment, so you need to implement your own to fill in the gaps.
3.  Then you discover that the biggest demand for hosting is from clients with legacy code and legacy dependencies, so you kind of have to give them a way to bring them over. Compatibility it is.
4.  Turns out many of the annoying features of the “legacy” system existed for a reason, so you have to implement them yourself, but you obviously do so in newer, modern, more sensible (and less backwards-compatible) ways.
5.  This would be fine – great even – if it weren’t for the fact that you now have to maintain ***two*** versions of large parts of your runtime, either as a translation layer from the “legacy” system to your new one, or as two completely independent features that require maintenance.

We’ve seen this strategy before. It’s basically [Cloudflare’s](https://www.cloudflare.com/), give or take a few details. Most of the differences in execution are because Cloudflare started on the opposite end of this particular chain of sausages: cloud hosting first, then [the open source runtime](https://blog.cloudflare.com/workers-open-source-announcement/), not the other way around as Deno did.

I find Deno’s tooling to be better, and their runtime much, much easier to use, but the overall strategy for the two companies is the same.

[Bun](https://bun.sh/) has the same playbook, with their only innovation being that, since they began later, they understood the need for node and npm compatibility from the very beginning.

I don’t think this approach is going to work out that well for any of them.

The “legacy” compatibility effectively removes any incentive to make packages for Deno (or Cloudflare, or Bun). Why use [`dnt`](https://github.com/denoland/dnt) to create packages that are compatible with both Deno and Node when you can just make a Node package? ***It’s still going to be compatible with both Deno and Node.*** Deno themselves have seen to that. You’re going to get cross-runtime compatibility either way.

The risk is that Deno will effectively become a platform for running code and projects from the npm ecosystem. Except it will never be as good at “node” as node itself.

There are always going to be gaps in the node and npm compatibility layer because *node is a moving target*. It’s a living project that’s still changing.

* * *

If Deno were maintained by a foundation or open source community, I’d be less worried. It’d inevitably be smaller in scope, but that would be fine. You don’t need world domination to be a successful community-driven software project. It could find its niche, with time.

But you do need to conquer a big chunk of the world to be a successful VC-funded startup.

There is already a disconnect in the Deno ecosystem. Many of the third party modules seem to be stagnant and haven’t been updated in a while. The projects that do seem vibrant tend to be the ones that target browser-compatible platforms in general and so get Deno, Cloudflare, and Bun support almost for free through little specific effort of their own.

Node hasn’t been standing still either. They’re adding support for many of the features that made Deno special, such as built-in tooling, a [test runner with coverage](https://nodejs.org/docs/latest/api/test.html), and [HTTP imports](https://nodejs.org/docs/latest/api/esm.html#https-and-http-imports). Even [import maps](https://github.com/nodejs/loaders#milestone-3-usability-improvements) are on their roadmap.

The fate of Node also doesn’t depend on the fate of a single startup or tech company.

The npm ecosystem remains a strategic liability, but adding support for import maps and HTTP imports will go some way to mitigate that and in the meantime you can install node packages [directly from git](https://medium.com/pravin-lolage/how-to-use-your-own-package-from-git-repository-as-a-node-module-8b543c13957e) or use an npm-compatible third party repository. [Any git remote URL should work with `npm install`](https://docs.npmjs.com/cli/v10/commands/npm-install#:~:text=npm%20install%20%3Cgit%20remote%20url%3E) – no GitHub required.

Seeing money and talent drain out of the software ecosystem, companies switch en masse to inherently conservative LLMs for development, and a growing frustration among software developers in general, it’s hard to envision a future where Deno or Bun win out in a direct competition with Node.

I’m not convinced I want to be using either of them once their VC-backed startups become desperate.

For those of us who *do* like to work in JavaScript, this means that Node is our best bet for the back end in the long term.

It’s probably time to try to recreate in node what has made Deno such a joy to work with.

But Node’s improvements aren’t the only issue facing Deno.

The big one is that the logical alternatives to Node – the “no Node” work environments developers are likely to reach for *aren’t going to be based on JavaScript*. Import maps mean that browsers effectively have an API surface that non-JS projects can use to build a dependency management system. Much of the tooling surrounding JavaScript is now implemented in *Rust*, not JS – much of it driven by Deno itself – and that makes it more easily accessible outside both the Node and Deno ecosystems. [The WASM component model](https://component-model.bytecodealliance.org/) additionally promises to make many dependencies runtime-independent, with the Rust-based tooling around JS being logical initial targets.

The “no Node” alternative for many won’t be another JavaScript runtime, but instead *something completely different.*