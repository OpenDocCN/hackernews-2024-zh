<!--yml
category: 未分类
date: 2024-05-27 14:54:09
-->

# Lemonade Stand | Possibly Wrong

> 来源：[https://possiblywrong.wordpress.com/2024/03/12/lemonade-stand/](https://possiblywrong.wordpress.com/2024/03/12/lemonade-stand/)

*This article was discussed on [Hacker News](https://news.ycombinator.com/item?id=39694353).*

**Introduction**

This was a fun nostalgia trip. But it ended up being an attempt to collect and preserve some retro computing history as well… and I also learned– or forgot and remembered?– another interesting quirk of that programming environment from nearly 45 years ago.

In 1981, I had a friend down the street who had an Apple II+. Later that year, my parents hired a tutor for programming lessons. I feel like I owe much of the subsequent course of my life to that time with “Mrs. Cathy,” who had several computers in her home: I remember an Apple II+, a VIC-20, and an IBM PC.

But it wasn’t until 1983 that we got our own home computer, an Apple //e. One of the programs that came with the computer, [Lemonade Stand](https://en.wikipedia.org/wiki/Lemonade_Stand), is shown in the screenshot below.

The game is pretty simple: each morning, based on the weather forecast for the day, you decide how many glasses of lemonade and how many advertising signs to make, and how much to charge for each glass. These decisions and the day’s weather affect demand, you sell some number of glasses, rinse and repeat.

It’s about as fun as it sounds… but I remember being intrigued by the prospect of “reverse engineering” the game. The Applesoft BASIC source code contains the formula for computing the demand and the resulting profit as a function of the player’s inputs and the weather. We can, in principle, “solve” for the inputs that would maximize the expected profit. This post is motivated by my failed attempt to do this.

**History**

All of the code discussed here, both old and new, is available on [GitHub](https://github.com/possibly-wrong/lemonade). Let’s start with the old code: `lemonade.bas` is the Applesoft BASIC source by Charlie Kellner, from the .dsk image of my original 5.25″ diskette. I believe this is the earliest Apple version of the game– for completeness I’ve also included source extracts of two subsequent released versions (`lemonade_v4.bas` and `lemonade_v5.bas`), but for our purpose here their behavior is identical, with cosmetic updates to the graphics and sound by Drew Lynch, Bruce Tognazzini, and Jim Gerber.

For comparison, I’ve also extracted the source for `selll.bas` (as in “sell lemonade”), from the Minnesota Educational Computing Consortium (MECC, of *Oregon Trail* fame) Elementary Volume 3 disk [A704](https://mirrors.apple2.org.za/ftp.apple.asimov.net/images/educational/mecc/MECC-A704%20Elementary%20Vol.%203.dsk). Although appearing later in 1980, I think this version is somewhat closer to the actual original version of the game written by Bob Jamison in 1973 for the UNIVAC mainframe. That original 1973 source seems to be lost to the mists of antiquity… but I think `selll.bas` retains more of its parentage than the Kellner version, which removed some helpful comments explaining the terms in the demand function, mistranslated some variable names and `GOTO` line numbering resulting in unreachable code, etc.

**Maximizing profit**

Now for the new code: `lemonade_strategy.py` implements the profit-maximization described above. It’s pure brute force: given the objective function below,

```
def expected_profit(glasses, signs, price, cost, r1=1, p_storm=0):
    n1 = 54 - 2.4 * price if price < 10 else 3000 / (price * price)
    demand = min(int(r1 * n1 * (2 - math.exp(-0.5 * signs))), glasses)
    revenue = demand * price
    expense = glasses * cost + signs * 15
    return ((1 - p_storm) * revenue - expense, revenue - expense,
            glasses, signs, price)

```

where `r1` and `p_storm` are functions of the weather forecast, we simply evaluate the expected profit for all feasible inputs and find the maximum, with the wrinkle that the feasible region– how many glasses and signs we can possibly make– also depends on our current assets:

```
max(expected_profit(glasses, signs, price, cost, r1, p_storm)
    for glasses in range(min(1000, assets // cost) + 1)
    for signs in range(min(50, (assets - cost * glasses) // 15) + 1)
    for price in range(101))

```

I ended up in this rabbit hole in the usual way, after discussion with a student. My objective wasn’t really to “solve” this decades-old game, but just to give an explicit example of how much slower computers were then: this same brute-force approach in Applesoft BASIC, with the Apple //e’s 1 MHz processor, took nearly 4 hours to compute optimal strategy just for the first sunny day in Lemonsville. That’s over 200,000 times slower than the above Python code, which runs in a fraction of a second on my decade-old 2.6 GHz laptop– that is, with a clock speed “only” 2,600 times faster.

**Changing the weather**

You can play an Apple version of the game in a browser at the [Internet Archive](https://archive.org/details/Lemonade_Stand_1979_Apple), linked from the game’s Wikipedia page. I did just that, with my Python script running alongside to compute optimal strategy… and it didn’t work.

The first day’s weather was “hot and dry.” With $2.00 and each glass of lemonade costing 2 cents, the optimal strategy should have been to make 74 glasses, 3 advertising signs, and charge 12 cents per glass, for an expected profit of $6.95\. But when I entered these values in the game, I only sold 37 glasses, for a lousy $2.51 profit.

What was going on? It turns out that there are many different copies of *Lemonade Stand* out there, both at the Internet Archive as well as in various Apple II [disk image archives](https://mirrors.apple2.org.za/ftp.apple.asimov.net/)… but the particular one linked from Wikipedia is unique among all of these in that it contains an important modification to the source code appearing in none of the other copies, highlighted below:

```
400  REM   WEATHER REPORT
410 SC =  RND (1)
420  IF SC < .4 THEN SC = 2: GOTO 460
430  IF SC < .7 THEN SC = 10: GOTO 460
440 SC = 7
460  REM  IF D<3 THEN SC=2
```

(You can play the original unmodified version at the Internet Archive [here](https://archive.org/details/LEMONADE_STAND).)

I don’t know where these changes came from. And I believe these are indeed *changes* from the original release (i.e., not the other way around). But just these three edits were enough to keep my optimal strategy calculation from producing the correct result. This was strange at first glance, since the above section of code merely computes the randomly generated weather *forecast* for the day, which is an *input* to the subsequent profit calculation. In the above modified version, the weather forecast is sunny (`SC=2`) 40% of the time, cloudy (`SC=10`) 30% of the time, and hot and dry (`SC=7`) 30% of the time. The original threshold values are 0.6 and 0.8, corresponding to probabilities (0.6, 0.2, 0.2) of (sunny, cloudy, hot and dry).

The `REM` in line 460 is a “remark” comment, effectively disabling the original behavior of forcing sunny forecasts for the first two days (when `D=1` and `D=2`) of the game. But again, why should this matter? This is all input to the profit calculation, which remains identical to the original, so what is causing the difference in behavior?

**Undeclared variables**

To make it easier to poke (!) around this problem, I wrote `lemonade.py`, which is my attempt at a [shot-for-shot remake](https://possiblywrong.wordpress.com/2013/02/03/hunt-the-wumpus/) of the original `lemonade.bas` in Python, minus the graphics and sound– that is, just a canonical mode text interface, but an otherwise line-for-line direct translation with identical behavior.

That translation process was an interesting exercise, almost a logic puzzle, converting the unstructured `GOTO`-based flow control of Applesoft BASIC into structured Python– I was surprised at the end of it all to find that only a single `def`ined function and extra temporary Boolean variable were needed, despite all of the `GOSUB`s.

That translation helped me to understand the cause of the different behavior I was seeing in the original. The problem is in the following section of code, with the offending lines highlighted:

```
700  REM   AFTER 2 DAYS THINGS CAN HAPPEN
710  IF D > 2 THEN 2000
...
2000  REM   RANDOM EVENTS
2010  IF SC = 10 THEN 2110
2030  IF SC = 7 THEN 2410
...
2410 X4 = 1
2430  PRINT "A HEAT WAVE IS PREDICTED FOR TODAY!"
2440 R1 = 2
2450  GOTO 805
```

The variable `R1` is an input to the profit function. But although the code change in line 460 above allows the first day’s *displayed forecast* to be hot and dry– or anything else other than sunny– line 710 still prevents actually *setting* `R1` to reflect that non-sunny forecast when computing profit.

The effect is that, for the first two days, the weather forecast might *indicate* “hot and dry,” or “cloudy,” etc., but it’s *really* still sunny for those first two days, just like the original version.

All of which leads to the actual motivation for this post: when I initially tried to reproduce the effect of the three changed lines in the Python version, I got a “`NameError: name 'D' is not defined`“, due to commenting out line 460 in the BASIC version above: the variable `D` is never declared or assigned a value *prior* to line 460.

Python can’t handle this… but apparently Applesoft BASIC can. Variables do not need to be declared before use: when evaluating an expression containing a variable not previously assigned, the value of that variable defaults to zero (or the empty string for string variables).

This behavior is actually depended on in a few other places in the game as well. I admit that I *think* this is new to me– in all of those early days of learning programming, I don’t remember ever being aware of nor taking advantage of this “feature.” And it’s not a feature of the original Dartmouth BASIC. And I couldn’t find documentation of it anywhere in the books, manuals, and magazines that I have… but I did find it written down in the original “[Blue Book](https://archive.org/details/Applesoft_Reference_Manual_1978-_bluebook/page/n11/mode/2up)” Applesoft Reference Manual. From page 9: “*Another important fact is that if a variable is encountered in a formula before it is assigned a value, it is automatically assigned the value zero. Zero is then substituted as the value of the variable in the particular formula.*“