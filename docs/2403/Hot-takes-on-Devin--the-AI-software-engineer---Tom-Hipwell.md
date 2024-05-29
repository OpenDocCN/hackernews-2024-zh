<!--yml
category: 未分类
date: 2024-05-27 15:05:35
-->

# Hot takes on Devin, the AI software engineer | Tom Hipwell

> 来源：[https://tomhipwell.co/blog/devin/](https://tomhipwell.co/blog/devin/)

I thought [Devin](https://www.cognition-labs.com/introducing-devin) from Cognition looked super cool this week, the UX feels like a glimpse of a new era.

I wonder how deep the moat is though? 🤔

From staring a little bit too closely at the screenshots and videos I’ve seen so far, a hot take would be that it feels like most of the performance lift in the SWE benchmarks could come from a switch in prompting technique, i.e. the size in the performance lift in the benchmark looks similar to that of shifting from chain-of-thought to something like the [Self-Discover](https://arxiv.org/abs/2402.03620) technique documented by Google DeepMind (but tailored to the domain rather than generic reasoning as in that paper). This style of prompting (select-adapt-implement -> solve) would make sense for this type of reasoning task. Maybe there’s a way to get a proxy in place to intercept the API calls and find out.

I wonder if the [Cyborg AI style](https://www.oneusefulthing.org/p/centaurs-and-cyborgs-on-the-jagged) that Ethan Mollick describes in the [BCG paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4573321) is a more natural fit to developer flow, and a better approach would be to have something like this integrated into the IDEs we’re already using today.

I like the pause and step buttons in the bottom left of the screenshots for this reason. In my head I think of it a bit like turn-taking in a strategy game like Civ II, with the ability to skip over the boring bits and let it play out when you’re confident in running loose (the ability to step backwards and forwards through agent changes would also be cool).

It’ll be interesting to see how this plays out over the next little while. If it is prompting that’s driving the uplift then I guess what we’ve seen so far is a glimpse into the DX of the future.