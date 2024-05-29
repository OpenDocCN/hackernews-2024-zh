<!--yml
category: 未分类
date: 2024-05-27 14:48:42
-->

# ChatGPT vs Advent of Code - The Motte

> 来源：[https://www.themotte.org/post/797/chatgpt-vs-advent-of-code](https://www.themotte.org/post/797/chatgpt-vs-advent-of-code)

# ChatGPT does Advent of Code 2023

LLM are all the rage and people are worried that they will soon replace programmers (or, indeed, every possible office job) so I decided to do an experiment to see how well ChatGPT-4 does against Advent of Code 2023.

## What is Advent of Code

Advent of Code (henceforth AoC) is an annual programming "event", held by [Eric Wastl](https://twitter.com/ericwastl), that takes place during the first 25 days of december. Each day at midnight a problem unlocks, consisting of an input file and a description of the required solution (either a number or a sequence of letters and numbers) to be determined by processing the input file. To solve the problem you have to submit to the website the correct solution. Once you do part 2 of the problem unlocks, usually a harder version of the problem in part 1. You don't have to submit any code so, in theory, you could solve everything by hand, however, usually, this is intractable and writing a program to do the work for you is the only easy way to solve the problem.

There's also a leaderboard where participants are scored based on how fast they submitted a solution.

Problems start very easy on day 1 (sometimes as easy as just asking for a program that sums all numbers in the input) and progress towards more difficult ones, but they never get very hard: a CS graduate should be able to solve all problems, except maybe 1 or 2, in a couple of hours each.

## Prior history

This isn't the first time ChatGPT (or LLMs) was used to participate in Advent of Code. In fact last year (2022) it was big news that users of ChatGPT were able, in multiple days, to reach the top of the global leaderboard. And this was enough of a concern that Eric explicitly banned ChatGPT users from submitting solutions before the global leaderboard was full (of course he also doesn't have any way to actually enforce this ban). [Some people](https://old.reddit.com/r/adventofcode/comments/18515qh/2023_the_year_of_gpt/) even expected GPT-4 to finish the whole year.

A lot of noise was made of GPT-3.5 performance in AoC last year but the actual results were quite modest and LLM enthusiasts behaved in a very unscientific way, by boasting successes but becoming very quiet when it started to fail. In fact ChatGPT [struggled to get through day 3 and 5 and probably couldn't solve anything after day 5](https://archive.is/BtuOG).

## Why do AoC with GPT?

I think it's as close to the perfect benchmark as you can get. The problems are roughly in order of increasing difficulty so you can see where it stops being able to solve. Since almost all of the problems in any given year are solvable by a CS graduate in a couple of hours is a good benchmark for AGI. And since all of the problems are novel the solutions can't come from overfitting.

Also around release people tried GPT-4 on AoC 2022 and [found that it performed better](https://twitter.com/geoffreylitt/status/1635757456377917440) so it would be interesting to see how much of the improvement was overfitting vs actual improvement

## Methodology

I don't pay for ChatGPT Plus, I only have a paid API key so I used instead a command line client, [chatgpt-cli](https://github.com/marcolardera/chatgpt-cli/) and manually ran the output programs. The prompt I used for part 1 was:

```
Write a python program to solve the following problem, the program should read its input from a file passed as argument on the command line: 
```

followed by the copypasted text of the problem. I manually removed from the prompt all the story fluff that Eric wrote, which constitutes a small amount of help for ChatGPT. If the output had trivial syntax mistakes I fixed them manually.

I gave up on a solution if it didn't terminate within 15 minutes, and let ChatGPT fail 3 times before giving up. A failure constitutes either an invalid program or a program that runs to completion but returns the wrong output value.

If the program ran to completion with the wrong answer I used the following prompt:

```
There seems to be a bug can you add some debug output to the program so we can find what the bug is? 
```

If the program ran into an error I would say so and copy the error message.

If the first part was solved correctly the prompt for the second part would be:

```
Very good, now I want you to write another python program, that still reads input from a command line argument, same input as before, and solves this additional problem: 
```

I decided I would stop the experiment after 4 consecutive days where ChatGPT was unable to solve part 1.

## ChatGPT Plus

Because I was aware of the possibility that ChatGPT Plus would be better I supplemented my experiment with two other sources. The first one is the [Youtube channel of Martin Zikmund](https://youtube.com/@mzikmund) (hencefort "youtuber") who did videos on how to solve the problems in C# as well as trying to solve them using ChatGPT (with a Plus account).

The second one was the blog of a ChatGPT enthusiast ["Advent of AI"](https://medium.com/@dretyr) (henceforth enthusiast) who tried to solve the problems using ChatGPT Plus and then also wrote a blog about it using ChatGPT Plus. Since the blog is generated by ChatGPT it's absolute shit and potentially contains hallucinations, however the [github repo with the transcripts](https://github.com/dretyr/adventofai) is valuable.

The enthusiast turned out to be completely useless: it resorted often to babystepping ChatGPT through to the result and he stopped on day 6 anyway.

The youtuber was much more informative, for the most part he stuck to letting ChatGPT solve the problem on its own. However he did give it, on a few occasions, some big hints, either by debugging ChatGPT's solution for it or explaining it how to solve the problem. I have noted this in the results.

## Results

|  | part 1 | part 2 | notes |
| day 1 | OK | FAIL |  |
| day 2 | OK | OK |  |
| day 3 | FAIL | N/A |  |
| day 4 | OK | OK | Uses brute force solution for part 2 |
| day 5 | OK | FAIL |  |
| day 6 | FAIL | N/A | ChatGPT Plus solves both parts |
| day 7 | FAIL | N/A |  |
| day 8 | OK | FAIL | ChatGPT Plus solves part 2 if you tell it what the solution is |
| day 9 | FAIL | N/A | ChatGPT Plus solves both parts |
| day 10 | FAIL | N/A |  |
| day 11 | FAIL | N/A | ChatGPT Plus could solve part 1 with a big hint |
| day 12 | FAIL | N/A |  |

The perofrmance of GPT-4 this year was *a bit worse* than GPT-3.5 last year. Last year GPT-3.5 could solve 3 days on its own (1, 2 and 4) while GPT-4 this year could only solve 2 full days (2 and 4).

ChatGPT Plus however did *a bit better*, solving on its own 4 days (2, 4, 6 and 9). This is probably down to its ability to see the problem input (as an attachment), rather than just the problem prompt and the example input to better sytem prompts and to just being able to do more round-trips through the code interpreter (I gave up after 3~4 errors / wrong outputs).

One shouldn't read too much on its ability to solve day 9, the problem difficulty doesn't increase monotonically and day 9 just happened to be very easy.

## Conclusions

Overall my subjective impression is that not much has changed, it can't solve anything that requires something more complicated than just following instructions and its bad at following instructions unless they are very simple.

It could be that LLMs have reached their plateau. Or maybe Q* or Bard Ultra or Grok Extra will wipe the floor next year, like GPT-4 was supposed to do this year. It's hard not to feel jaded about the hype cycle.

I have a bunch of observations about the performance of ChatGPT on AoC which I will report here in no particular order.

### Debugging / world models

Most humans are incapable of solving AoC problems on the first try without making mistakes so I wouldn't expect a human-level AI to be able to do it either (if it could it would be by definition super-human).

Some of my prompting strategy went into the direction of trying to get ChatGPT to debug its flawed solution. I was asking it to add debug prints to figure out where the logic of the solution went wrong.

ChatGPT *never* did this: its debugging skills are completely non-existent. If it encounters an error it will simply rewrite entire functions, or more often the entire program, from scratch.

This is drastically different from what programmers.

This is interesting because debugging techniques aren't really taught. By and large programming textbooks teach you to program, not how to fix errors you wrote. And yet people do pick up debugging skills, implicitly.

ChatGPT has the same access to programming textbooks that humans have and yet it does not learn to debug. I think this points to the fact that ChatGPT hasn't really learned to program, that it doesn't have a "world model", a logical understanding of what it is doing when it's programming.

The bruteforce way to get ChatGPT to learn debugging I think would be to scrape hundreds of hours of programming livestreams from twitch and feed it to the training program after doing OCR on the videos and speech-to-text on the audio. That's the only source of massive amounts of worked out debugging examples that I can think of.

### Difficulty

Could it be that this year of AoC was just harder than last year's and that's why GPT-4 didn't do well? Maybe.

Difficulty is very hard to gauge objectively. There's [scatter plots](https://aoc-stats.fastbee.box.ca/) for leaderboard fill-up time but time-to-complete isn't necessarily equivalent difficulty and the difference between this year and last year isn't big anyway (note: the scatter plots aren't to scale unfortunately).

My own subjective impression is also that this year (so far) was not harder.

The best evidence for an increase in difficulty is day 1 part 2, which contained a small trap in which both human participants and ChatGPT fell.

I think this points to a problem with this AIs trained with enormous amounts of training data: you can't really tell how much better they are. Ideally you would just test GPT-4 on AoC 2022, but GPT-4 training set *contains* many copies of AoC 2022's solutions so it's not really a good benchmark anymore.

Normally you would take out a portion of the training set to use as test set but with massive training set this is impossible, nobody knows what's in them and so nobody knows how many times each individual training example is replicated in them.

I wonder if OpenAI has a secret test dataset that they don't put on the internet anywhere to avoid training set contamination.

[Some people](https://news.ycombinator.com/item?id=38483271#38488265) have even speculated that the problems this year were deliberately formulated to foil ChatGPT, but Eric actually [denied that this is the case](https://old.reddit.com/r/adventofcode/comments/18bp8id/why_does_aoc_care_about_llms/kc646i4/).

### Overfitting

GPT 4 is 10x larger than GPT 3.5 and it does much better on a bunch of standard tests, for example the [bar exam](https://law.stanford.edu/2023/04/19/gpt-4-passes-the-bar-exam-what-that-means-for-artificial-intelligence-tools-in-the-legal-industry/).

Why did it not do much better on AoC? If it isn't difficulty it could be overfitting. It has simply memorized the answers to a bunch of standardized tests.

Is this the case? My experience with AoC day 7 points towards this. The problem asks to write a custom string ordering function, the strings in questions represent hands of cards (A25JQ is ace, 2, 5 jack and queen) and the order it asks for is *similar* to Poker scoring. However it is *not* Poker.

This is a really simple day and I expected ChatGPT would be able to solve it without problems, since you just have to follow instructions. And yet it couldn't it was inesorably pulled towards writing a solution for Poker rather than for this problem.

My guess is that this is an example of overfitting in action. It's seen too many examples of poker in its training set to be able to solve this quasi-poker thing.