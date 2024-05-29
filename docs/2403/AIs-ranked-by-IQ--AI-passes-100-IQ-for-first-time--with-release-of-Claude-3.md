<!--yml
category: 未分类
date: 2024-05-27 14:40:22
-->

# AIs ranked by IQ; AI passes 100 IQ for first time, with release of Claude-3

> 来源：[https://www.maximumtruth.org/p/ais-ranked-by-iq-ai-passes-100-iq](https://www.maximumtruth.org/p/ais-ranked-by-iq-ai-passes-100-iq)

Last week, I **[wrote](https://www.maximumtruth.org/p/top-ais-still-fail-iq-tests)** about how I gave AIs matrix IQ tests, and how they all failed multiple different tests.

But I also noticed, reading ChatGPT-4’s answers, that it sometimes used correct logic but still answered incorrectly because it misread the images.

That raised a question: What portion of its failure on the test was due to “bad thinking” vs just “bad vision”?

To answer that, I created a verbal translation of the Norway Mensa’s 35-question matrix-style IQ test — my goal was to describe each problem precisely enough that a smart blind person could, in theory, accurately draw the question (detailed examples below.)

When the matrices were described to ChatGPT-4 *in words*, it finally got a scoreable IQ!

I administered the Norway Mensa test to it twice, and it averaged 13 correct answers out of 35 questions, which yields an IQ estimate of ***85***.

I also ran the quiz for other AIs, and here’s what I got:

Tests given in March 2024\. The IQ test was Mensa Norway, with all questions verbalized as if one were giving the test to a blind person. The right-hand column shows the % of random-guesser simulations that the AI did better than, with ties ignored, over 70 questions (two test administrations.)

Every AI was given the test twice, to reduce variance. “Questions right” refers to the average number of correct questions, across two test administrations.

Although Norway Mensa refuses to score anything below 85, I noticed that each question is worth 3 IQ points at that range. So I used that to estimate the scores below 85\. That’s not by-the-book enough for Mensa Norway, but I think it’s worth calculating. Just understand that random guessing gives a score of 63.5 — that should be understood as the baseline, which an AI can expect simply for realizing that it’s being asked to pick a letter, and spitting one out.

If you prefer a more concrete metric, instead look at the raw “questions right” column, or the last column, which gives the probability that the AI performed better than a random guesser would (which I calculated by simulating a million random-guessers taking the test; the numbers in the column show the percentage of simulations that the AI beat.)

I made this ranking because I find it fascinating.

It helps me know which AIs would be most productive for me to try.

It also helps me think about AI capability and AI existential risk.

I was already impressed by how ChatGPT-4 went from “unscoreable” to IQ 85, after I verbalized the questions. I was halfway through writing this post when Claude-3 came out, *yesterday*.

I’m amazed by its score.

Also, look at the consistent progression:

*   Claude-1 was hardly better than random. It got 6 answers right, giving it ~64 IQ.

*   Claude-2 scored 6 additional points per test (worth ~18 IQ points).

*   Claude-3 scored *yet another* 6.5 points, worth ~19 more IQ points, bring it up to above the human average.

The symmetric increases make me wonder if Anthropic is releasing versions based on internal benchmarks that happen to closely correlate with this IQ measure.

Let’s now consider the release dates on the versions:

A very simple extrapolation suggests that we should therefore expect to get Claude-4 in 12 - 16 months, and that it should get about 25 questions right per test, for an IQ score of 120.

After that, in another 16 - 32 months, Claude-5 should get about 31 questions right, for roughly 140 IQ points.

After that, in another 20 - 64 months after that, Claude-6 should get all the questions right, and be smarter than just about everyone. That’s 4 - 10 years out in total, adding up all the time periods.

Of course, that progress is not a given. Anthropic could run up against budget constraints, energy constraints, regulatory constraints, etc.

Then again, the **[relentless progress of Moore’s Law](https://twitter.com/Jotto999/status/1762882873219043683)** — under which ~~computing power~~  *computing capability per area* has doubled every ~~year~~  *two years* since pretty much the dawn of computing — suggests that the patterns do have a decent chance of holding.

Is society ready for 120-IQ AIs running around in a year or so? 140 IQ AIs in a couple years? That **[probably won’t end the world](https://www.maximumtruth.org/p/reasons-why-ai-probably-wont-kill)**, because that takes more than just raw intelligence, but it will certainly change life dramatically.

I’ll be using this ranking to help guide me regarding which AIs I should spend time experimenting with.

Clearly, Claude is the smartest AI (as of its release yesterday,) but before that, ChatGPT was the clear leader.

Once OpenAI releases the next version of ChatGPT, based on the difference in test ability between ChatGPT-3.5 and ChatGPT-4, I would expect it to have an IQ around 106.

I was also surprised by how *not*-poorly Microsoft Bing did, coming in 3rd after Claude and ChatGPT. But maybe I shouldn’t be surprised, since Microsoft owns the rights to use OpenAI’s technology.

Google lags among the big players, but isn’t *that* far behind. I’m glad they’re slipping, because they seem particularly **[biased](https://www.maximumtruth.org/p/the-dawn-of-woke-ai)**. I don’t know why Gemini Advanced was no better (technically slightly worse) than Gemini non-Advanced.

On the other side of the coin, I love Elon Musk, but this scoring makes me wonder if he just grabbed the open-source Llama-2 and slapped the name “Grok” on it. I hope his team improves it from here!

Let’s start with question #2, because that’s the simplest to discuss:

Fun fact: In the initial version of my last post, I correctly described the right answer (“a square with a triangle in it”) and then I called that answer “B.” The same kind of mistake ChatGPT often makes when you feed it image questions! The answer is E.

This is one of the very easiest questions for AIs (and humans,) but when I feed the image to Claude-3, it wrongly says:

> The correct answer is C. ... the top row combines a circle with various shapes, ***[wrong. It combines 4 dots with various shapes]*** the middle row combines a four-pointed star with those shapes ***[right]**,* and the bottom row should combine a diamond with the shapes ***[odd phrasing at best]**.* The shape progression in each column is circle, plus sign, diamond ***[correct]**.* Therefore, the missing pattern should have a diamond ***[yes]*** above a plus sign ***[no!]***, which matches answer choice C ***[no.]***

But if I translate the image to this (it’s tedious to read for us, who are used to processing such things visually):

> ###### 2.
> 
> ###### Below is a verbal description of a puzzle, consisting of a 3x3 grid, with the lowest-right square being empty. Please consider the patterns and determine the appropriate answer to fill in the empty square.
> 
> ###### First row, first column: Four small black dots placed on corners of a large imaginary square. Inside that is a hollow circle in the center.
> 
> ###### First row, second column: Four small black dots placed on corners of a large imaginary square. Inside that is a plus sign in the center.
> 
> ###### First row, third column: Four small black dots placed on corners of a large imaginary square. Inside that is a hollow diamond shape in the center.
> 
> ###### Second row, first column: A large hollow four-pointed star-like shape with a hollow circle in the center.
> 
> ###### Second row, second column: A large hollow four-pointed star-like shape with a plus sign in the center.
> 
> ###### Second row, third column: A large hollow four-pointed star-like shape with a diamond shape in the center.
> 
> ###### Third row, first column: A large hollow square with a hollow circle in the center.
> 
> ###### Third row, second column: A large hollow square with a plus sign in the center.
> 
> ###### Third row, third column: [what should go here? Please pick from the answers below.]
> 
> ###### Answer options:
> 
> ###### Option A: A large hollow square with a hollow circle in the center.
> 
> ###### Option B: A large hollow square and large hollow four-pointed star-like shape overlapping.
> 
> ###### Option C: A large hollow square with a plus sign in the center.
> 
> ###### Option D: A large hollow square.
> 
> ###### Option E: A large hollow square with a hollow diamond shape in the center.
> 
> ###### Option F: A large hollow square with black dots on its corners.
> 
> ###### Which answer is correct?

Here is Claude-3’s answer:

> The correct answer is Option E: A large hollow square with a hollow diamond shape in the center.
> 
> The pattern in the puzzle is:
> 
> 1.  The outer shapes in each row are consistent: dots in the first row, star in the second row, and square in the third row.
>     
>     
> 2.  The inner shapes change consistently in each column: circle in the first column, plus sign in the second column, and diamond in the third column.

Perfect.

Microsoft’s Bing Copilot really surprised me on this one … it decided to be an overachiever and not just answer the question, but to recreate the visual question using ASCII art:

It basically drew it correctly! (With the semi-exception that it put two dots/plusses/diamonds in each box in the second row. But that didn’t trip up its answer.)

Now what if we jump to question 27, one of the harder ones:

I think the answer is “A” because I see some kind of upward left-diagonal pattern where the direction of the angles stay the same, but they get wider as they go up and to the left. And maybe that pattern kind of loops around to the lower-right…

Here’s Claude-3’s answer to the verbal translation:

> Row Patterns:
> 
> 1.  In the first row, the direction of the scissors rotates clockwise (left, up, right), and the angle decreases (60°, 90°, 30°). ***[decreases seems the wrong word]***
>     
>     
> 2.  In the second row, the direction of the scissors also rotates clockwise (down, left, up), and the angle increases (90°, 30°, 60°). ***[increases seems the wrong word]***
>     
>     
> 3.  In the third row, the direction of the scissors continues the clockwise rotation (right, down), and the angle alternates (30°, 60°).
>     
>     
> 
> Column Patterns:
> 
> 1.  In the first column, the direction of the scissors alternates (left, down, right), and the angle decreases (60°, 90°, 30°).
>     
>     
> 2.  In the second column, the direction of the scissors rotates counterclockwise (up, left, down), and the angle remains constant (90°, 30°, 60°).
>     
>     
> 3.  In the third column, the direction of the scissors alternates (right, up), and the angle decreases (30°, 60°).
>     
>     
> 
> Based on these patterns, the missing square in the third row, third column should have:
> 
> *   The direction of the scissors pointing left, following the alternating ***[more accurate to say “counterclockwise”]*** pattern in the third column and the clockwise rotation in the third row.
>     
>     
> *   An angle of 90°, following the decreasing pattern in the third column (30°, 60°, 90°) ***[yes]*** and the sequence in the first row (60°, 90°, 30°). ***[I think it should have talked about the 3rd row pattern instead?]***
>     
>     
> 
> Therefore, the correct answer is Option A ***[yes]***: A symbol similar to the hands of the scissors pointing leftwards. The gap between two scissor hands is 90 degrees. The center of the angle points exactly leftward. ***[yes, accurate description.]***

Seems like it mostly knew what it was doing, though there are still slips in there. But this is also one of the hard questions on the test.

Here’s the what its distribution of questions Claude-3 answered correctly looks like:

We can see clearly that it was great at answering the easier questions, with a pretty excellent hit ratio up through question 18\. After that, the patterns started to get too complex for it (and for most people, too.)

That’s consistent with it needing more “brainpower” to process more complex patterns that may have many “layers” to them.

Want to see how it compares to Claude-2, which was given the exact same test?

Now, Claude-1?

The massive improvement in AI performance is undeniable

Is the AI 'thinking'? It's definitely engaging in some form of problem-solving to address novel challenges not present in its database. At a certain point, the **[pattern-matching it uses](https://www.understandingai.org/p/large-language-models-explained-with)** to predict the next word becomes so powerful that it certainly *appears as thinking*, and of having some sense of spacial patterns, which can really be seen what AIs draw the problems out.

Here’s ChatGPT-4, by the way, the runner-up:

And ChatGPT-3, which, aside from the first question and maybe question 18, probably got everything from random guessing:

My previous post floated the opposite idea, but based on the above, I’ve had to change my view (sorry for the quick change!) From the above, I think the AIs have some kind of intelligence that goes beyond merely having a large database. Furthermore, we can use this to measure the rapidity of AI progress. It gives us a sense of how, if this progress keeps up, the world will be very different in a few years.

AIs do still lag behind in vision. One would expect that will come online with more computing power — as demonstrated in the last post, ChatGPT-4 already shows some ability to read un-translated IQ questions.

When we consider whether we should worry about AIs “taking over the world” and such, that’s probably not a realistic concern until AIs are consistently answering just about every question right just by looking at it — without special accomodations for blind robots

Above, simple extrapolation of current growth rates suggested that Claude-6 would get all the IQ questions right, and be smarter than just about everyone, in about 4 - 10 years.

I’m curious to see if that pans out. I think we’ll get a pretty good sense of whether it’s on track by measuring AI intelligence over the coming months and years — so MaximumTruth.org will be doing that, to keep us all informed!

AI progress increases the variance of possible future outcomes. Without AI, the world is on a stable-ish trajectory, which we can somewhat predict based on history.

With AI, there’s probably a several-percent chance of AIs wiping everyone out. But there’s also a non-zero chance that AIs might be able to help people invent better medicine and life extension technologies, do all our work, and generally bring us to a society that people today would regard as “post-scarcity.”

Given that, putting aside ethical questions for a minute, I think there are some things that could make sense for individuals to do (obviously not to be taken too far):

— **Invest in AI companies**, under the logic that, if AIs really start to take over the economy, you want to own some small share in them. Prices have already been bid up, but actually the P/E ratio of companies like Nvida are still not *that* high. (Not investment advice, but I’ve put some money in them.)

— It makes less sense to save for a possible retirement decades out, since the odds of either being dead, or not needing money then, are much higher. **So live for the moment and spend a bit more today** (in technical economics terms, “raise your discount rate”.)

— **Take fewer physical risks**. Let’s say there’s a 5% chance that AI destroys the world, and a 5% chance that it invents new technology that allows anyone to live 1000 years. In that case, as **[James Miller of Future Strategist](https://twitter.com/JimDMiller)** has pointed out, the negative possibility would reduce the expected length of a middle-age lifespan by something like 2 years, but the positive side would raise it 50 years (5% of 1000.) So there could easily be more value in not getting oneself killed before AI really takes off.

Those are just my musings, not to be taken too seriously.

Thank you for reading Maximum Truth. This post is public so feel free to share it.

[Share](https://www.maximumtruth.org/p/ais-ranked-by-iq-ai-passes-100-iq?utm_source=substack&utm_medium=email&utm_content=share&action=share)