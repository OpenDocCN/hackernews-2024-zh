<!--yml
category: 未分类
date: 2024-05-27 14:26:03
-->

# Was Javascript really made in 10 days? • Buttondown

> 来源：[https://buttondown.email/hillelwayne/archive/did-brendan-eich-really-make-javascript-in-10-days/](https://buttondown.email/hillelwayne/archive/did-brendan-eich-really-make-javascript-in-10-days/)

<date>September 28, 2023</date>

# Was Javascript really made in 10 days?

I once heard that Javascript has so many warts because the first version was made in just ten days. I was curious 1) if this is true, and 2) if it explains the language's warts.

After some research, I can unconfidently say: it's complicated.

The "first version" of JavaScript did in fact take ten days. The exact dates aren't confirmed, but Brendan Eich recalls it being [May 6-15, 1995](https://www.quora.com/In-which-10-days-of-May-did-Brendan-Eich-write-JavaScript-Mocha-in-1995). But this was only a minimal prototype ("Mocha") for internal demonstration. JavaScript 1.0 was publicly released in March 1996 ([pg 10](http://www.wirfs-brock.com/allen/jshopl.pdf)) and the first "complete" version in August 1996 (ibid). Even after that point, the Netscape team regularly tweaked the design of JS; Eich [recalls that](https://brendaneich.com/2011/06/) "Bill Gates was bitching about us changing JS all the time" in the Fall of 1996.

Eich also had about ten years of experience with language design and compiler developer and was explicitly hired by Netscape to put a programming language in the browser (pg 7). Originally this was supposed Scheme, but then Netscape signed a deal with Sun and agreed to make it more "Java-like".

### Does this explain the warts?

Most of JavaScript's modern flaws are arguably *not* due to the short development time:

*   Mocha didn't originally have implicit type conversion, but users requested Eich add that to 1.0 ([video link](https://youtu.be/krB0enBeSiE?si=s9_oUf9Tp9Nxz-qh&t=2503)). He deeply regrets this.
*   JS 1.0 added `null` to be more compatible with Java ([pg 13](http://www.wirfs-brock.com/allen/jshopl.pdf). Java compatibility is also why `typeof null = object`.
*   Any JavaScript API warts must have come afterwards because *all* the API work happened after Mocha. Mocha's a pretty minimal language!
*   The "all numbers are floats" problem was originally in Mocha, but I think that was always the intended behavior. The [JavaScript 1.0 manual](https://web.archive.org/web/19970613234917/http://home.netscape.com/eng/mozilla/2.0/handbook/javascript/index.html) cites HyperTalk as a major inspiration. I've never used HyperTalk skimming the manual suggest to me it does the same thing ([pg 102](https://cancel.fm/stuff/share/HyperCard_Script_Language_Guide_1.pdf), 517).

I found one way the 10-day sprint *definitely did* harm JavaScript: Brendan Eich didn't have time to add a garbage collector, and later attempts to add that in added a bunch of security holes ([43:04](https://youtu.be/krB0enBeSiE?si=dHTMzc0DXbj-1ELQ&t=2584)).

This newsletter takes the new record for "most hours of research per word."

*If you're reading this on the web, you can subscribe [here](/hillelwayne). Updates are once a week. My main website is [here](https://www.hillelwayne.com).*