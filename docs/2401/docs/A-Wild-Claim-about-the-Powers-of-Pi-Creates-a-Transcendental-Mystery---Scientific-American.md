<!--yml
category: 未分类
date: 2024-05-27 15:13:44
-->

# A Wild Claim about the Powers of Pi Creates a Transcendental Mystery | Scientific American

> 来源：[https://www.scientificamerican.com/article/a-wild-claim-about-the-powers-of-pi-creates-a-transcendental-mystery/](https://www.scientificamerican.com/article/a-wild-claim-about-the-powers-of-pi-creates-a-transcendental-mystery/)

Pi (π) is probably the most famous number in mathematics. Not only have experts [studied it extensively](https://www.scientificamerican.com/video/how-to-calculate-a-bigger-slice-of-pi/), but it also fascinates amateurs: [books](https://books.google.de/books/about/The_Book_Of_Pi_What_is_Pi_It_s_History_a.html?id=HaaBzQEACAAJ&source=kp_book_description&redir_esc=y), [films](https://en.wikipedia.org/wiki/Pi_(film)) and [songs](https://www.youtube.com/watch?v=3HRkKznJoZA) have been dedicated to the number. Part of its appeal may be that even though it [describes the circle](https://www.scientificamerican.com/article/what-is-pi-and-how-did-it-originate/), one of the simplest and most symmetrical geometric objects, it has no symmetry whatsoever in its decimal representation. The decimal values of pi have no end and never repeat.

People have been studying the number pi for thousands of years. You might therefore expect that almost everything is already known about it. But this is far from the case: the circle still holds many mysteries. One of them revolves around the question of what happens when you repeatedly multiply pi by itself: Could it be that π to the power of π to the power of π to the power of π results in [a natural number](https://www.scientificamerican.com/article/all-natural-numbers-are-either-happy-or-sad-some-are-narcissistic-too/)?

The notion that raising an irrational number to multiple powers could result in a number with no numbers after the decimal point may seem far-fetched at first. But there are other examples. For instance, (√2 to the power of √2) to the power of √2 can be simplified to √2^(√2 x √2) and therefore results in √2² = 2.* It is not so easy to see how such calculations would work with π, however.

* * *

## On supporting science journalism

If you're enjoying this article, consider supporting our award-winning journalism by [subscribing](/getsciam/). By purchasing a subscription you are helping to ensure the future of impactful stories about the discoveries and ideas shaping our world today.

* * *

## A Tweet Challenges the Math Community

On May 3, 2013, Dan Piponi, now principal mathematician at Epic Games, made a post on Twitter (now X) that asked people to prove that [π to the power of π to the power of π to the power of π isn’t an integer](https://twitter.com/sigfpe/status/330415672549068800). That message drew a few comments but did not attract too much attention.

[Computer scientist Daniel Spiewak quickly saw through Piponi](https://twitter.com/djspiewak/status/330517207043944448): “So basically, you’re asking your twitter followers to solve one of the significant unanswered questions [with regard to] tetration?” he replied. (Tetration refers to a repeatedly performed exponentiation.) Indeed, not even mathematicians know what kind of number you will get when you raise π to its power four times in a row.

Is that hard to believe? Mathematician Thomas Bloom of the University of Oxford thinks so, too. He [revived the question](https://twitter.com/thomasfbloom/status/1346207600531144704) on Twitter in 2021\. The topic aroused more interest this time: his post was shared 90 times and liked more than 500 times. [Mathematician and Fields Medalist Timothy Gowers commented](https://twitter.com/wtgowers/status/1346212151581700096), “Wow! My first thought was: ‘Why can’t we just work it out to a couple of decimal places?’ and then it hit me. (But that works for pi^pi^pi.)”

To be honest, that was my first thought, too. Why discuss the problem when we can simply calculate the result? I should actually know better—in fact, I myself have [written about large numbers](https://www.scientificamerican.com/article/infinity-is-not-always-equal-to-infinity/). Even if π to the power of π to the power of π to the power of π looks harmless, it is an unimaginably huge number.

## Let’s Do the Math!

Suppose you wanted to attempt the calculation yourself. First off, you need to know that multiple exponentiation is carried out from right to left. This means that you first calculate π to the power of π, which is approximately 36.46.

Then you raise π to the power of 36.46... and get an 18-digit number as the result: 1.34... x 10^(18). That very long number is only the result of a threefold exponentiation. There is still one more execution missing: π to the power of π to the power of π to the power of π is equal to π to the power of 1.34... x 10^(18).

The result will be gigantic—incredibly gigantic. It is a number with almost 10^(18) (one billion billion) digits. For comparison, in 2022 the record number of calculated digits of pi [was 62 x 10^(12)](https://www.guinnessworldrecords.com/news/2022/3/new-value-of-pi-calculated-by-swiss-university-at-over-62-billion-digits-694748). To calculate the integer result of π to the power of π to the power of π to the power of π, you would have to determine a million times more digits. And that’s just the numbers *before* the decimal point.

What we are actually interested in are the numbers after the decimal point. The assertion, after all, was that π to the power of π to the power of π to the power of π is an integer—that is, it has no numbers after the decimal point. This means that we can ignore the almost 10^(18) digits before the decimal point in the calculation. That could simplify the calculation.

You can start with a simpler example. Suppose you want to calculate the repeated exponentiation of an integer—say, 4 to the power of 4 to the power of 4—and are only interested in the last two digits. You quickly realize that 4 to the power of 4 to the power of 4 is the same as 4^(256).

But calculating the latter exactly is quite time-consuming, and because we are only interested in the last two digits of the result, you can take a shortcut. First, calculate 4¹ = 4 and multiply that by 4 to get 4² = 16\. Multiply this by 4 again, and you get 4³ = 64\. Repeat this, and you get the result: 4⁴ = 256\. And now comes the simplification: instead of multiplying the entire three-digit result by 4 in the next step, it is sufficient to just look at the last two digits (i.e., 56). After all, we are only interested in the last two digits of the result. This meant 4⁵ = ...56 x 4 = ...224\. In the next step, you ignore the hundreds again and continue: 4⁶ = ...24 x 4 ...96, and so on. If you do this 256 times, you will end up with the last digits of the number you are looking for.

You can probably already guess why this can’t work with π: the circle number is irrational, so it has an infinite number of decimal places. There is, therefore, no “smallest digit” that can be taken into account when exponentiating.

Sure, you could investigate how many decimal places of π have to be included in a calculation in order to obtain the most precise result possible after exponentiation. In this way, it might be possible to recognize approximately whether π to the power of π to the power of π to the power of π could assume an integer value.

Australian mathematician Matt Parker actually [made that effort](https://www.youtube.com/watch?v=BdHFLfv-ThQ) in a YouTube video. If you break off π after five decimal places and raise this number to the power of 6, only the first two decimal places of the result are correct. Parker could not calculate exactly how many digits must be taken into account in order to get at least a few correct decimal places of π to the power of 1.34... x 10^(18) (which corresponds to π to the power of π to the power of π). But based on his calculations, Parker suspects that you will need at least twice the exponent (i.e., 2 x 1.34... x 10^(18)) of decimal places to get at least one correct digit after the decimal point. “In short, there’s no way we can calculate this in the conceivable future,” [Parker concluded his argument](https://www.youtube.com/watch?v=BdHFLfv-ThQ) in the video.

## Abstract Mathematics to the Rescue

Fortunately, mathematics also offers other ways of discerning whether a number is an integer, irrational or even transcendental. The latter refers to numbers that cannot be expressed as the solution to a simple equation. For instance, √2 is not transcendental because √2 is the solution of x² = 2\. On the other hand, π is transcendental. Finding this out is often complicated. The late American mathematician Stephen Hoel Schanuel, however, made a conjecture in the 1960s that can be used to assess whether a value is transcendent (and therefore irrational). The conjecture itself is quite abstract and requires advanced mathematics.

[Some experts have used Schanuel’s conjecture](https://twitter.com/thomasfbloom/status/1346405512687087617) to investigate π to the power of π to the power of π to the power of π . As they discovered, the result would have to be transcendental—and therefore could not possibly be an integer. But Schanuel’s conjecture is still only a conjecture; no one has yet been able to prove it. Therefore the conclusion that the number is transcendent is still open to debate.

In the end, then, there are just two ways to solve the puzzle of the fourfold exponentiation of π. “We either have to get much better at mathematics and prove Schanuel’s conjecture,” [said Parker in his YouTube video](https://www.youtube.com/watch?v=BdHFLfv-ThQ), “or we need to get a lot better at computing. Until then, we have no idea if π to the π to the π to the π is an integer or not.”

*This article originally appeared in *Spektrum der Wissenschaft* and was reproduced with permission.*

**Editor’s Note (1/31/24): This sentence was edited after posting to correct the placement of parentheses.*