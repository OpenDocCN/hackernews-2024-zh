<!--yml
category: 未分类
date: 2024-05-27 15:07:27
-->

# Donor Bounties: A New Kind of Feature Bounty ⚡ Zig Programming Language

> 来源：[https://ziglang.org/news/announcing-donor-bounties/](https://ziglang.org/news/announcing-donor-bounties/)

# Donor Bounties: A New Kind of Feature Bounty

[We have previously written](/news/bounties-damage-open-source-projects/) about how bounties can be damaging to Open Source projects as, even when the subject of the bounty is in accordance with the wishes of the project maintainers, it still fosters competition at the expense of cooperation and more in general is a strictly worse version of just offering contract work, as bounties default to potentially unfair resolutions when things don’t go as planned, among other things.

As we also pointed out in the past, this is an issue with certain kinds of bounties, and not others. One positive example are bug bounties, where the un-coordinated nature of the work is nowhere near as big of a problem (or at the very least is a *problem*, not a *“feature”* of the system).

Comments on social media have pointed out how the kinds of bounties we have issues with are generally known as “feature bounties”.

So we did some thinking and figured out a way of mitigating all the flaws of feature bounties. The change is minimal but mutates drastically the nature of the bounty, here it is: **instead of offering money to the implementer, the money is offered as a donation to the open source project itself**.

By offering the bounty to the open source project, the cooperative nature of open source work is fully preserved. Of course giving money to the project and not to who is doing the work is very different in nature, but it makes a lot of sense when the project is run by a non-profit corporation that pays developers, like in the case for the Zig Software Foundation.

**[In 2023 we spent 92% of our money on paying contributors for their time](/news/2024-financials/)**, as a point of reference.

I want to call this new kind of bounty a “donor bounty” and offer it as a service available to supporters of the Zig project. Here’s how it works in practice:

1.  You email us at `donations@ziglang.org` and express your interest in offering a donor bounty for a given already-accepted proposal (i.e. a GitHub Issue that has both the `proposal` and `accepted` labels).
2.  We discuss internally if we think the feature is suitable for this process (e.g. doesn’t have design questions that are still open) and if this is a good time for it.
3.  If we agree to proceed, we will discuss the donation amount, expected timeline and publish a donor bounty announcement.
4.  If the work is completed and merged into Zig before the bounty’s deadline, you donate the agreed amount.

One thing to be aware of is that there’s a tall bar to pass for us to accept a donor bounty: we only accept donor bounties on already-accepted proposals and even then we will only agree to setup a donor bounty if the feature is fit from multiple angles, including a high degree of confidence on our end that the feature in question has a very low risk of getting axed in future iterations of the language, as a donor bounty **does not** guarantee that the feature will be part of Zig’s final design.

Donor bounties are not a contractual agreement between individuals and the Zig Software Foundation; they are instead built upon mutual trust and goodwill between the two parties.

We’re also happy to announce that we have already accepted our first donor bounty!

[You can read all about it here.](/news/first-donor-bounty/)

Kind regards,
Loris Cro