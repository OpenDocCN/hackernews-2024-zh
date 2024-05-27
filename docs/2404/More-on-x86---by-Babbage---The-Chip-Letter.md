<!--yml
category: 未分类
date: 2024-05-27 13:32:07
-->

# More on x86 - by Babbage - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/more-on-x86](https://thechipletter.substack.com/p/more-on-x86)

*A quick programming announcement. This is the first of two shorter posts this week. This is a follow-on from last week’s [The Paradox of x86](https://thechipletter.substack.com/p/the-paradox-of-x86). The next will look at a historic moment in the development of modern semiconductors.*

* * *

Last week’s post on x86 was well-timed as it coincided with

writing about whether the x86 Instruction Set Architecture does or doesn’t ‘need to die’.

Casey’s post followed his discussion on the topic with Twitcher [ThePrimegen](https://www.twitch.tv/theprimeagen), recorded for posterity on YouTube.

The whole discussion had been prompted by two recent articles. The first on Hackaday [Why x86 Needs To Die](https://hackaday.com/2024/03/21/why-x86-needs-to-die/) in turn prompted a response from the Chips and Cheese blog entitled [Why x86 Doesn’t Need To Die](https://chipsandcheese.com/2024/03/27/why-x86-doesnt-need-to-die/).

Casey is on the ‘doesn’t need to die ’ side of the argument and makes a range of very valid points, pretty much all of which I agree with. He summarises his position as follows:

> That said, on the measure, I would disagree that x86 “needs to die”. The reason I say that is because x86 (now x64) has done a remarkable job stabilizing software development over the past forty years. Sure, you can call that “legacy” or a “maintenance problem” if you want to look at the downsides. But you could also call it a *remarkably flexible instruction set* if you’d like to look at the upsides.

Casey mentions Intel’s APX proposals for ‘spring cleaning’ the x86 architecture, which we discussed briefly when they were announced back in July last year.

Key proposals in APX include:

*   Doubling the number of general purpose registers from 16 to 32;

*   New three operand instructions (e.g. adding ability to subtract register1 from register2, and place the result in register3);

*   New instructions to PUSH / POP two general purpose registers at once;

*   New conditional load, store and compare instructions;

*   Adds the option to suppress status flag writes for common instructions;

*   New 64-bit absolute jump instruction.

Casey highlights, in particular, the ability to suppress flag writes as a potential performance win for APX.

While very largely agreeing with Casey, I do think that there are other aspects to x86 that these discussions of the topic omit. In particular, the lock-in to two hardware vendors has been highly problematic, both for competition and for innovation. In the APX post, we asked the question:

> Why didn't Intel / AMD do this a decade ago?

x86-64, the 64-bit version of x86 was announced (by AMD) in 1999\. The first Intel x86-64 cores appeared in 2004\. When will the first APX cores appear? As far as I can tell Intel hasn’t told us yet, which must mean that it is 2025 at the earliest.

I don’t think it’s any coincidence that these improvements - which probably could have been introduced a decade ago - are appearing only when x86 faces a lot more competition. If we needed a demonstration of the problems of the x86 ‘lock-in’ then this is it.

In any event, I think calling, as some do, x86 a ‘legacy’ architecture is premature. Intel and AMD have proven that it can remain competitive and it will become more so with APX. As we’ve discussed before, x86 will be around for a long time - many decades at least - and will continue to be built at scale on leading-edge fabrication processes. It just won't be as dominant as it has been for the last couple of decades and that is surely healthy.

[Share](https://thechipletter.substack.com/p/more-on-x86?utm_source=substack&utm_medium=email&utm_content=share&action=share)

AMD Ryzen 5 3600(Zen 2 | Matisse | CCD) - Courtesy of Fritzchens Fritz

I’m conscious of the fact that [The Paradox of x86](https://thechipletter.substack.com/p/the-paradox-of-x86) glossed over several interesting details. In particular, AMD’s role in the development of x86 and the potential challenge that ‘it’s not a moat if AMD is also making x86 designs.’

Intel and AMD have been fierce competitors in the x86 CPU market for almost five decades now. If you’d like to know how it all started, then

does his usual awesome job in the video below.

Intel’s efforts to compete with AMD have sometimes spilled over into, what has been ruled to be, illegal and anti-competitive behavior. For example, the European Commission [fined](https://competitionlawblog.kluwercompetitionlaw.com/2024/02/06/what-fine-should-be-imposed-to-intel-for-the-so-called-naked-restrictions-after-the-annulation-of-the-part-of-the-decision-relating-to-loyalty-rebates/#:~:text=On%2022%20September%202023%2C%20the,decision%20of%2013%20May%202009.) Intel over a billion Euros in 2009 for:

> … anti-competitive practices implemented by Intel to the detriment of its competitor on the x86 CPU market, AMD, between October 2002 and December 2007\.

(Note that this fine was later overturned and replaced by a smaller fine last year.)

Given this fierce competition and Intel’s decades-long manufacturing lead over AMD, it’s perhaps remarkable that AMD survived at all.

AMD’s achievement of a market cap greater than Intel’s ($236bn vs $145bn at the time of writing on 22/4/24) has been made possible by access to TSMC’s superior manufacturing. Even before that though, AMD has often proven to be a nimble and innovative competitor. The 64-bit version of x86 was, of course, an AMD creation - as AMD64 - whilst Intel pursued Itanium.

On the ‘is it really a moat if AMD is also making x86 CPUs’ challenge, a moat is just a defensive barrier. In this case, one that protects both firms, just not from each other.

It’s arguable too that AMD’s presence as a maker of x86 CPUs has been good for Intel in several ways.

*   Extending the physical moat analogy, AMD has plugged occasional gaps in the moat when Intel has been distracted with other things, with AMD64 being the most notable example.

*   More recently, as Intel stumbled on 10nm, AMD has shown that x86 remains a competitive architecture, and possibly delayed some switches to Arm.

*   Perhaps more significantly, I suspect that competition from AMD has helped to keep Intel ‘match fit’, and reduce the complacency that might otherwise have seen the firm falter earlier.

*   Finally, I think that there would have been a lot more antitrust interest in x86 both in the US and the EU without AMD’s presence in the market.

I suspect that Intel’s management believes this last point too. Perhaps then AMD’s survival shouldn't be such a surprise after all?!