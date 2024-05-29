<!--yml
category: 未分类
date: 2024-05-27 14:30:29
-->

# The One Billion Row Challenge - Gunnar Morling

> 来源：[https://www.morling.dev/blog/one-billion-row-challenge/](https://www.morling.dev/blog/one-billion-row-challenge/)

*Update Jan 4: Wow, this thing really took off!* *1BRC is discussed at a couple of places on the internet, including [Hacker News](https://news.ycombinator.com/item?id=38851337), [lobste.rs](https://lobste.rs/s/u2qcnf/one_billion_row_challenge), and [Reddit](https://old.reddit.com/r/programming/comments/18x0x0u/the_one_billion_row_challenge/).*

*Thanks a lot for all the submissions, this is going way beyond what I’d have expected!* *I am behind a bit with evalutions due to the sheer amount of entries, I will work through them bit by bit.* *I have also made a few clarifications to [the rules](https://github.com/gunnarmorling/1brc#faq) of the challenge; please make sure to read them before submitting any entries.*

Let’s kick off 2024 true coder style—​I’m excited to announce the [One Billion Row Challenge](https://github.com/gunnarmorling/onebrc) (1BRC), running from Jan 1 until Jan 31.

Your mission, should you decide to accept it, is deceptively simple: write a Java program for retrieving temperature measurement values from a text file and calculating the min, mean, and max temperature per weather station. There’s just one caveat: the file has **1,000,000,000 rows**!

The text file has a simple structure with one measurement value per row:

```
 |  
```
1
2
3
4
5
6

```

 |  
```
Hamburg;12.0
Bulawayo;8.9
Palembang;38.8
St. John's;15.2
Cracow;12.6
...

```

 | 
```

The program should print out the min, mean, and max values per station, alphabetically ordered like so:

```
 |  
```
1

```

 |  
```
{Abha=5.0/18.0/27.4, Abidjan=15.7/26.0/34.1, Abéché=12.1/29.4/35.6, Accra=14.7/26.4/33.1, Addis Ababa=2.1/16.0/24.3, Adelaide=4.1/17.3/29.7, ...}

```

 | 
```

The goal of the 1BRC challenge is to create the fastest implementation for this task, and while doing so, explore the benefits of modern Java and find out how far you can push this platform. So grab all your (virtual) threads, reach out to the Vector API and SIMD, optimize your GC, leverage AOT compilation, or pull any other trick you can think of.

There’s a few simple rules of engagement for 1BRC (see [here](https://github.com/gunnarmorling/onebrc#running-the-challenge) for more details):

*   Any submission must be written in Java

*   Any Java distribution available through [SDKMan](https://sdkman.io/) as well as early access builds from [openjdk.net](https://openjdk.net) may be used, including EA builds for OpenJDK projects like Valhalla

*   No external dependencies may be used

To enter the challenge, clone the [1brc repository](https://github.com/gunnarmorling/1brc) from GitHub and follow the instructions in the README file. There is a very [basic implementation](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_baseline.java) of the task which you can use as a baseline for comparisons and to make sure that your own implementation emits the correct result. Once you’re satisfied with your work, open a pull request against the upstream repo to submit your implementation to the challenge.

All submissions will be evaluated by running the program on a [Hetzner Cloud CCX33](https://www.hetzner.com/cloud) instance (8 dedicated vCPU, 32 GB RAM). The `time` program is used for measuring execution times, i.e. end-to-end times are measured. Each contender will be run five times in a row. The slowest and the fastest runs are discarded. The mean value of the remaining three runs is the result for that contender and will be added to the [leaderboard](https://github.com/gunnarmorling/onebrc#results). If you have any questions or would like to discuss any potential 1BRC optimization techniques, please join [the discussion](https://github.com/gunnarmorling/1brc/discussions) in the GitHub repo.

As for a prize, by entering this challenge, you may learn something new, get to inspire others, and take pride in seeing your name listed in the scoreboard above. Rumor has it that the winner may receive a unique 1️⃣🐝🏎️ t-shirt, too.

So don’t wait, join this challenge, and find out how fast Java can be—​I’m really curious what the community will come up with for this one. Happy 2024, coder style!