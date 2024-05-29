<!--yml
category: 未分类
date: 2024-05-29 13:29:01
-->

# It's that time again when tech cannot handle the date • The Register

> 来源：[https://www.theregister.com/2024/02/29/fuel_pump_leap_year_bug/](https://www.theregister.com/2024/02/29/fuel_pump_leap_year_bug/)

Today is February 29, an unusual day in that it is added to the common 28 in years that are multiples of four to keep the calendar in sync with the astronomical year.

This kludge prevents our seasons from drifting out of whack, but it presents a problem for computers and software, which have to be programmed to account for the extra day to avoid error conditions and incorrect data.

We are all using a computer of one sort or another to read this and hopefully nothing has caught fire, yet every leap year, something somewhere falls over hard.

In New Zealand, which has a head start on most of the world, it was payment systems at fuel pumps that have just staggered back to their feet after a nationwide outage lasting more than ten hours.

Allied Petroleum, Gull, Z, Waitomo, BP and more petrol providers were affected, which meant customers were unable to use debit and credit cards to pay for fuel.

Many locations closed their pumps for the duration, though in-house solutions like Waitomo's app worked just fine, presumably because they are professionally coded to deal with February 29 appropriately.

Unlike Invenco point-of-sale software encompassing fuel pump terminals.

Gull spokesperson Julien Leys said companies using Invenco terminals were suffering a leap year bug. "We have been liaising with our provider and understand they are working as quickly as possible to fix the issue," he told the [The New Zealand Herald](https://www.nzherald.co.nz/hawkes-bay-today/news/february-29-allied-fuel-pumps-around-nz-ground-to-a-halt-as-systems-forget-leap-year/XEQBK5JLBZG6LO3VGUQ6Q2WGC4/), adding that February 29 is "just one of those things that caused payment software to have a glitch."

"Just one of those things" if the software isn't calibrated for the event, which to us is highly suggestive of human error. This is why developers use tried-and-tested libraries that have been around for decades, and don't typically dare to touch anything concerning calendar and time logic themselves lest they be fast-tracked to insanity.

John Scott, CEO of Auckland-founded Invenco, confirmed the leap year glitch and said that the fix had been rolled out to the network, allowing fuel pump payments to resume.

*The Register* asked Invenco to detail the nature of the error and how it was fixed. The article will be updated should the company respond.

Invenco isn't the only software outfit to fall foul of February 29\. Owners of Fastrack FS1 smartwatches have reported the clocks [being stuck at 23:59](https://twitter.com/Amol_chi/status/1763046970317230119) on February 28 or [not displaying the date after](https://twitter.com/Manuvktr/status/1763057415417901311). The YNAB (You Need A Budget) app was also said to [not recognize](https://www.reddit.com/r/ynab/comments/1b27jpo/funny_glitch_for_leap_year_repeating_scheduled/) the existence of February 29\. Meanwhile, in Japan, police could [not issue or renew driving licenses](https://japannews.yomiuri.co.jp/society/general-news/20240229-171789/) in Kanagawa, Niigata, Okayama, and Ehime, again thought to be due to the date.

As most *Reg* readers will know, tracking time is an absolute minefield in computing. Take the Y2K bug, where worldwide carnage was predicted because of systems representing years with only the final two digits, making 2000 indistinguishable from 1900.

While everything was "fine," this was mainly due to intense efforts by technology teams to expand date fields or window them, as explained in our [Retro Tech Week feature](https://www.theregister.com/2024/01/17/y2k_feature/) on the problem.

That isn't to say nothing went wrong, though. Japan losing its ability to monitor nuclear power plant safety systems, for instance, sounds less than optimal.

Then there's the [Year 2038 Problem](https://theyear2038problem.com/), which affects systems working on Unix time – the number of non-leap seconds that have elapsed since 00:00:00 UTC on January 1, 1970 (the Unix epoch). Unix developers decided to track this as a signed 32-bit integer, but this data type is only capable of encoding up to 03:14:07 UTC on January 19, 2038, hence the name. One second later, the integer will overflow, which systems will interpret as 20:45:52 UTC on December 13, 1901.

We have all that still to look forward to, but looking back on the last time it was February 29, the world was about to end, so some temporary difficulties paying for fuel is a much rosier outcome. ®