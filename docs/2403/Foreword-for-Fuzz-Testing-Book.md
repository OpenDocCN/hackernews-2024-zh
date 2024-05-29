<!--yml
category: 未分类
date: 2024-05-29 12:46:23
-->

# Foreword for Fuzz Testing Book

> 来源：[https://pages.cs.wisc.edu/~bart/fuzz/Foreword1.html](https://pages.cs.wisc.edu/~bart/fuzz/Foreword1.html)

 ## Foreword for Fuzz Testing Book

* * *

Below is the draft of prose that I wrote for the foreword to a book on fuzz testing:

* * *

It was a dark and stormy night. Really.

Sitting in my apartment in Madison in the Fall of 1988, there was a wild midwest thunderstorm pouring rain and lighting up the late night sky. That night, I was logged on to the Unix system in my office via a dial-up phone line over a 1200 baud modem. With the heavy rain, there was noise on the line and that noise was interfering with my ability to type sensible commands to the shell and programs that I was running. It was a race to type an input line before the noise overwhelmed the command.

This fighting with the noisy phone line was not surprising. After all, this was *just* before error-correcting modems were available. What did surprise me was the fact that the noise seemed to be causing programs to crash. And more surprising to me were the programs that were crashing -- common Unix utilities that we all use everyday.

The scientist in me said that we need to make a systematic investigation to try to understand the extent of the problem and the cause.

That semester, I was teaching the graduate Advanced Operating Systems course at the University of Wisconsin. Each semester in this course, we hand out a list of suggested topics for the students to explore for their course project. I added this testing project to the list; the text for this project description appears in the break-out box below.

In the process of writing the project description, I needed to give this kind of testing a name. I wanted a name that would evoke the feeling of random, unstructured data. After trying out several ideas, I settled on the term ***fuzz***.

Three groups attempted the fuzz project that semester and two failed to achieve any crash results. Lars Fredriksen and Bryan So formed the third group, and were more talented programmers and more careful experimenters; they succeeded well beyond my expectations. As we reported in the first fuzz paper [[1990](http://www.cs.wisc.edu/paradyn/papers/fuzz.pdf)], they could crash or hang between 25-33% of the utility programs on the seven Unix variants that they tested.

However, the fuzz testing project was more than a quick way to find program failures. Finding the cause of each failure and categorizing these failures gave the results deeper meaning and more lasting impact. The source code for the tools and scripts, the raw test results, and the suggested bug fixes were all made public. Trust and repeatability were crucial underlying principles for this work.

In the following years, we repeated these tests on more and varied Unix systems for a larger set of command-line utility programs and expanded our testing to GUI programs based on the then-new X-window system [[1995](http://www.cs.wisc.edu/paradyn/papers/fuzz-revisited.pdf)]. Windows followed several years later [[2000](http://www.cs.wisc.edu/paradyn/papers/fuzz-nt.pdf)] and, most recently, MacOS [[2006](http://www.cs.wisc.edu/paradyn/papers/Fuzz-MacOS.pdf)]. In each case, over the span of the years, we found a lot of bugs and, in each case, we diagnosed those bugs and published all of our results.

In our more recent research, as we have expanded to more GUI-based application testing, we discovered that classic 1983 testing tool, "The Monkey" used on the earlier Macintosh computers [cite Hertzfeld book]. Clearly a group ahead of their time.

In the process of writing our early fuzz papers, we came across strong resistance from the testing and software engineering community. The lack of a formal model and methodology and undisciplined approach to testing often offended experienced practitioners in the field. In fact, I still frequently come across hostile attitudes to this "stone axes and bear skins" (my apologies to Mr. Spock) approach to testing.

My response has always been simple: "We're just trying to find bugs". As I have said many times, fuzz testing is not meant to supplant more systematic testing. It is just one more tool, albeit an extremely easy one to use, in the tester's toolkit.

As an aside, note that the fuzz testing has not ever been a funded research effort for me; it is a research avocation rather than a vocation. All the hard work has been done by a series of talented and motivated graduate students as class projects in my Advanced Operating System class. This is how we have fun.

Fuzz testing has grown into a major subfield of research and engineering, with new results taking it far beyond our simple and initial work. As reliability is the foundation of security, so has it become a crucial tool in security evaluation of software. Thus, the topic of this book is both timely and extremely important. Every practitioner who aspires to write safe and secure software needs to add these techniques to their bag of tricks.

Barton Miller
Madison, Wisconsin
April 2008

|  

&#124;  
```

**Operating System Utility Program Reliability - The Fuzz Generator**

The goal of this project is to evaluate the robustness of various UNIX utility
programs, given an unpredictable input stream.  This project has two parts.
First, you will build a "fuzz" generator.  This is a program that will output
a random character stream.  Second, you will take the fuzz generator and use
it to attack as many UNIX utilities as possible, with the goal of trying to
break them.  For the utilities that break, you will try to determine what type
of input cause the break.

THE PROGRAM

The fuzz generator will generate an output stream of random characters.
It will need several options to give you flexibility to test different
programs.  Below is the start for a list of options for features that
fuzz will support.  It is important when writing this program to use
good C and UNIX style, and good structure, as we hope to distribute
this program to others.

    -p          only the printable ASCII characters

    -a          all ASCII characters

    -0          include the null (0 byte) character

    -l          generate random length lines (\\n terminated strings)

    -f name     record characters in file ``name''

    -d nnn      delay nnn seconds following each character

    -r name     replay characters in file ``name'' to output

THE TESTING

The fuzz program should be used to test various UNIX utilities.  These
utilities include programs like vi, mail, cc, make, sed, awk, sort,
etc.  The goal is to first see if the program will break and second to
understand what type of input is responsible for the break.

```

 &#124;

 |

* * *