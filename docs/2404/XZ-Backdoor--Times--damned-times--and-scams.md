<!--yml
category: 未分类
date: 2024-05-27 12:49:18
-->

# XZ Backdoor: Times, damned times, and scams

> 来源：[https://rheaeve.substack.com/p/xz-backdoor-times-damned-times-and](https://rheaeve.substack.com/p/xz-backdoor-times-damned-times-and)

re: [https://www.openwall.com/lists/oss-security/2024/03/29/4](https://www.openwall.com/lists/oss-security/2024/03/29/4)

*Written by Rhea Karty and Simon Henniger*

There has been a recent backdoor found in the xz/liblzma tarball. In what is likely one of the largest breaches of trust in the free software ecosystem, this backdoor is likely to have been put in by Jia Tan, a long-time maintainer of xz. Throughout his tenure as a maintainer, Jia remained relatively mysterious — as is not uncommon in the community, little beyond his name (which is likely a lie) is known about him. Generally, anonymity in the free software sphere is a good thing: software is inherently based on accomplishment and merit, and there is no reason to know anything about a person’s identity. However, in this case, where someone built up the trust of a community for years and then abused it, it is interesting to see who they are. Luckily for us, Jia’s activity does provide some metadata which we can potentially use to learn more about him. So here’s an analysis on what we can learn from his work patterns and time zone.

What is to be learned from times? The conditions under which software is created! Think about the wide range of things that time patterns tell us about — there are people who are paid to do code and those who work on it as a hobby. Those in some areas code during different times as others. There are holidays, sleep schedules, and work-life balance— code is not exempt from these. Understanding when someone codes helps us understand *why* and *where* they are coding for. Figuring these out can give us a much better idea about why and who did this.

The following analysis was conducted on JiaT75’s ([https://github.com/JiaT75?tab=overview&from=2021-12-01&to=2021-12-31](https://github.com/JiaT75?tab=overview&from=2021-12-01&to=2021-12-31)) commits to the XZ repository and their time stamps.

Let’s first address the elephant in the room: Yes, you can change Git timestamps to whatever you want. It doesn’t even take a degree in cybersec to do — setting the environment variables GIT_AUTHOR_DATE and GIT_COMMITTER_DATE as you commit is enough. If you forget and haven’t pushed yet, you can even change the date later as you amend the commit.

However, plausibly forging time data is actually hard: You shouldn’t push a commit that you made to look as though it was made in the future, and you also shouldn’t set a date that is so far in the past that the commit looks older than online discussions that it references. This means you will often not be able to change the time without adding latency, i.e., withholding commits and thus delaying project development -- and even then, getting it right every time is hard. After all, everyone who develops software knows how tempting it is to type the magic words “git commit” without having properly checked everything.

An easier way than actually changing the time would be to change only the time zone! Ideally, you would change it to something that still leads to somewhat plausible hours. For example, someone who works during regular European office hours but claims to be on the East Coast would likely raise some suspicion, as 11 am Central European Time is 5 AM Eastern, and who regularly works in the early morning? For a hacker, it is much more plausible to work in the afternoon and late at night, so someone who keeps regular office hours would want to change into a time zone about five hours ahead or so (so they start working at 2 pm (9 am real-time) and end at 10 pm (5 pm real time)).

I think that is what Jia Tan did. Based on his name, he wanted people to believe he is Asian — specifically Chinese— and the vast majority of his commits (440) appear to have a UTC+08 time stamp. The +0800 is likely CST, the time zone of China (or Indonesia or Philippines or Western Australia), given almost no one lives in Siberia and the Gobi desert.

However, I believe that he is actually from somewhere in the UTC+02 (winter)/UTC+03 (DST) timezone, which includes Eastern Europe (EET), but also Israel (IST), and some others. Forging time zones would be easy — no need to do any math or delay any commits. He likely just changed his system time to Chinese time every time he committed.

We see him usually working 9 am to 6 pm (adjusted to EET). This makes much more sense than someone working at midnight and 1 am on a Tuesday night (non-adjusted, using UTC+08).

Except sometimes, he forgot to change his time zone. There are 3 commits and 6 commits, respectively, with UTC+02 and UTC+03\. The UTC+02 time zones match perfectly with the winter time (February and November), while the UTC+03 matches with summer (Jun, Jul, and early October). This matches perfectly with the daylight savings time switchover that happens in Eastern Europe; we see a switch to +0200 in the winter (past the last weekend of October) and +0300 in the summer (past the last Sunday in March). Incidentally, this seems to be the same time zone as Lasse Collin and Hans Jansen. [Update Apr 2: As Joey Hess points out in the commits, most (but not all!) times when this happens, commits have Lasse Collin as Committer. However, it doesn’t happen on nearly all Jia commits committed by Lasse – many of those show a +0800 time zone. Hence, it is a bit unclear what is special about those +02/03 commits – they could still be a slip-up, or due to a boring change in the workflow.]

OK, you say. That is one theory, but maybe not a plausible one at this point. What if he just flew from wherever he lived in UTC+08 to EET sometimes? Well, it turns out that when we take a closer look, we see that this is not plausible. Let’s analyze the few times when Jia was recorded in a non-+0800 time zone. Here, we notice that there are some situations where Jia switches between +0800 and +0300/+0200 in a seemingly implausible time. Indicating that perhaps he is not actually in +0800 CST time, as his profile would like us to believe. 

Notably, on 6 Oct 2022, we see two commits, one at 21:53:09 +0300 followed by another at 17:00:38 +0800\. If we do the math, there is about an 11-hour difference between the two commits. However, a flight from China to anywhere in Eastern Europe or the Middle East takes at least 10-12 hours raw flight time, and likely much longer, considering non-stop flights between these locations are rare.

Even more damning, on Jun 27, 2023, we see the following: one commit at 23:38:32 +0800, another at 17:27:09 +0300\. This is only a difference in a matter of minutes!

There is also one more vital clue to which country he worked in: Holidays. We notice that Jia’s work schedule and holidays seem to align much better with an Eastern European than a Chinese person. Disclaimer: I am not an expert in Chinese holidays, so this very well could be inaccurate. I am referencing this list of bank holidays: [https://www.bankofchina.co.id/en-id/service/information/latest-news/2022/public-holidays-in-china-hk-and-the-us-in-2023.html](https://www.bankofchina.co.id/en-id/service/information/latest-news/2022/public-holidays-in-china-hk-and-the-us-in-2023.html)

Chinese bank holidays (just looking at 2023):

- Working on 2023, 29 September: Mid Autumn Festival

- Working on 2023, 05 April: Tomb Sweeping Day

- Working on 2023, 26, 22, 23, 24, 26, 27 Jan: Lunar New Year

Eastern European holidays:

- Never working on Dec 25: Christmas (for many EET countries)

- Never working Dec 31 or Jan 1: New Years 

To further investigate, we can try to see if he worked on weekends or weekdays: was this a hobbyist or was he paid to do this? The most common working days for Jia were Tue (86), Wed (85), Thu (89), and Fri (79).

Again, none of this is rock-solid evidence for anything — but we thought this was an interesting enough observation to post. Based on this, “Jia” most likely worked regular office hours and was based somewhere in UTC+02/03 (e.g. EET).

Update (Mar 31): Thank you all for your great feedback! We slightly updated the post: We switched around the times from the two commits where time zones change too quickly (our point stands; we just reversed the examples). We also clarified that UTC +02/03 includes a wider geographical range than just Eastern Europe.