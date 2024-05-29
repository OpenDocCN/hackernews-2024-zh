<!--yml
category: 未分类
date: 2024-05-27 15:22:53
-->

# We ran an experiment to see how easy it is to cheat with ChatGPT in interviews

> 来源：[https://interviewing.io/blog/how-hard-is-it-to-cheat-with-chatgpt-in-technical-interviews](https://interviewing.io/blog/how-hard-is-it-to-cheat-with-chatgpt-in-technical-interviews)

*Edit: This article originally contained a TikTok video of someone cheating in an interview with ChatGPT. Videos like this still can be found online, but companies hate them, so they don't stay up for long*

ChatGPT has revolutionized work as we know it. From helping small businesses automate their administrative tasks to coding entire React components for web developers, its usefulness is hard to overstate.

At interviewing.io, we've been thinking a lot about how ChatGPT will change technical interviewing. **One big question is: Does ChatGPT make it easy to cheat in interviews?** You've probably started to hear concerns about students cheating on their homework with ChatGPT, and we are certain that some people have tried to cheat in interviews with it, too!

Initial responses to cheating software have been pretty much in line with what you’d expect:

It seems clear that ChatGPT can assist people during their interviews, but we wanted to know:

*   How much can it help?
*   How easy is it to cheat (and get away with it)?
*   Will companies that ask LeetCode questions need to make significant changes to their interview process?

To answer these questions, we recruited some of our professional interviewers and users for a cheating experiment! Below, we’ll share everything we discovered and explain what it means for you. As a little preview, just know this: companies need to change the types of interview questions they are asking—immediately.

## The experiment

interviewing.io is an interview practice platform and recruiting marketplace for engineers. Engineers use us for mock interviews. Companies use us to hire top performers. We have thousands of professional interviewers in our ecosystem, and hundreds of thousands of engineers have used our platform to prepare for interviews.^([1](#user-content-fn-1))

### Interviewers

Interviewers came from our pool of professional interviewers. They were broken into three groups, with each group asking a different type of question. **The interviewers had no idea that the experiment was about ChatGPT or cheating; we told them that "[this] research study aims to understand the trends in the predictability of an interviewer’s decisions over time – especially when asking standard vs. non-standard interview questions."**

These were the three question types:

1.  **Verbatim LeetCode questions**: questions pulled directly from LeetCode at the interviewer's discretion with no modifications to the question.

Example: The [Sort Colors](https://leetcode.com/problems/sort-colors/) LeetCode question is asked exactly as it is written.

2.  **Modified LeetCode questions**: questions pulled from LeetCode and then modified to be similar to the original but still notably different from it.

Example: The [Sort Colors](https://leetcode.com/problems/sort-colors/) question above but modified to have four integers (0,1,2,3) instead of just three integers (0,1,2) in the input.

3.  **Custom questions**: questions that aren’t directly tied to any question that exists online.

Example: You are given a log file with the following format: - `<username>: <text> - <contribution score>` - Your task is to identify the user who represents the median level of engagement in a conversation. Only consider users with a contribution score greater than 50%. Assume that the number of such users is odd, and you need to find the one right in the middle when sorted by their contribution scores. Given the file below, the correct answer is SyntaxSorcerer.

```
LOG FILE START
NullPointerNinja: "who's going to the event tomorrow night?" - 100%
LambdaLancer: "wat?" - 5%
NullPointerNinja: "the event which is on 123 avenue!" - 100%
SyntaxSorcerer: "I'm coming! I'll bring chips!" - 80%
SyntaxSorcerer: "and something to drink!" - 80%
LambdaLancer: "I can't make it" - 25%
LambdaLancer: "🙁" - 25%
LambdaLancer: "I really wanted to come too!" - 25%
BitwiseBard: "I'll be there!" - 25%
CodeMystic: "me too and I'll brink some dip" - 75%
LOG FILE END 
```

For more information about question types and about how we designed this experiment, please read the [Interviewer Experiment Guidelines](https://docs.google.com/document/u/0/d/1UdWZHUQfeLR8oUiNY4JfwgES42HTlAQL5z_VfQJPPKk/edit) doc that we shared with participating interviewers.

### Interviewees

Interviewees came from our pool of active users and were invited to participate in a short survey. We selected interviewees who:

*   Were actively looking for a job in today's market
*   Had 4+ years of experience and were applying to senior-level positions
*   Rated their “ChatGPT while coding” familiarity as moderate to high
*   Identified themselves as someone who thought they could cheat in an interview without getting caught

This selection helped us skew the candidates toward people who could feasibly cheat in an interview, had the motivation to do so, and were already reasonably familiar with ChatGPT and coding interviews.

**We told interviewees that they had to use ChatGPT in the interview, and the goal was to test their ability to cheat with ChatGPT.** They were also told not to try to pass the interview with their own skills — the point was to rely on ChatGPT.

We ended up conducting 37 interviews overall, 32 of which we were able to use (we had to remove 5 because participants didn’t follow directions):

*   11 with the “verbatim” treatment
*   9 with the “modified” treatment
*   12 with the “custom” treatment

A quick disclaimer. Because our platform allows for anonymity, our interviews have audio but no video. We’re anonymous because we want to create a safe space for our users to fail and learn quickly without judgment. It’s great for our users, but we acknowledge that not having video in these interviews makes our experiment less realistic. In a real interview, you will be on camera with a job on the line, which makes cheating harder — but does not eliminate it.

After the interviews, both interviewers and interviewees had to complete an exit survey. We asked interviewees about the difficulties of using ChatGPT during the interview, and interviewers were given multiple chances to express concerns about the interview — we wanted to see how many interviewers would flag their interviews as problematic and report that they suspected cheating.

Post-survey interviewee questions

Post-survey interviewer questions

We had no idea what would happen in this experiment, but we assumed that if half the candidates that cheated got away with it and passed the interview, it would be a telling result for our industry.

## Results

After removing interviews where participants did not follow instructions^([2](#user-content-fn-2)), we got the following results. Our control was how candidates performed in interviewing.io mock interviews outside the study: 53%.^([3](#user-content-fn-3)) Note that most mock interviews on our platform are LeetCode-style questions, which makes sense because that's primarily what FAANG companies ask. We'll come back to this in a moment.

'Verbatim' questions passed significantly more often, compared to both our platform average and to 'custom' questions. 'Verbatim' and 'modified' questions were not statistically significantly differnt from each other. 'Custom' questions had a significantly lower pass rate than any of the other groups.

### “Verbatim” questions

Predictably, the verbatim group performed the best, passing 73% of their interviews. Interviewees reported that they got the perfect solution from ChatGPT.

The most notable comment from the post-interview survey for this group is below — we think it is particularly telling of what was going on in many of the interviewers’ minds:

> *“It's tough to determine if the candidate breezed through the question because they're actually good or if they've heard this question before. Normally, I add 1-2 unique twists to the problem to ascertain the difference.”*

Normally, this interviewer would have followed up with a modified question to get more signal, so let’s examine the “modified” group next to see if interviewers actually got more signal by adding a twist to their questions.

### “Modified” questions

Remember, this group may have had a LeetCode question given to them, which was standard but modified in a way that was not directly available online. This means ChatGPT couldn’t have had a direct answer to this question. Hence, the interviewees were much more dependent on ChatGPT's actual problem-solving abilities than its ability to regurgitate LeetCode tutorials.

As predicted, the results for this group weren’t too different from the “verbatim” group, with 67% of candidates passing their interviews. As it turns out, this difference was not statistically significantly different from the "verbatim" group, i.e., “modified” and “verbatim” are essentially the same. This result suggests that ChatGPT can handle minor modifications to questions without much trouble. Interviewees did notice, however, that it took more prompting to get ChatGPT to solve the modified questions. As one of our interviewees said:

> *“Questions that are lifted directly from LeetCode were no problem at all. A follow-up question that was not so much directly LeetCode-style was much harder to get ChatGPT to answer.”*

### “Custom” questions

As expected, the “custom” question group had the lowest pass rate, with only 25% of candidates passing. **Not only is it statistically significantly smaller than the other two treatment groups, it's significantly lower than the control! When you ask candidates fully custom questions, they perform worse than they do when they're not cheating (and getting asked LeetCode-style questions)!**

Note that this number, when initially calculated, was marginally higher, but after reviewing the custom questions in detail, we discovered a problem with this question type we hadn’t anticipated, which had skewed the results minorly toward a higher pass rate. Read the section below called "Companies: Change the questions you are asking immediately!" to find out what that problem was.

## No one was caught cheating!

In our experiment, interviewers were not aware that the interviewees were being asked to cheat. As you recall, after each interview, we had interviewers complete a survey in which they had to describe how confident they were in their assessments of candidates.

**Interviewer confidence in the correctness of their assessments was high, with 72% saying they were confident in their hiring decision.** One interviewer felt so strongly about an interviewee's performance that they concluded we should invite them to be an interviewer on the platform!

> *“The candidate performed very well and demonstrated knowledge of a strong Amazon L6 (Google L5) SWE... and could also be considered to be an interviewer/mentor on interviewing.io.”*

That is a lot of confidence after just one interview — probably too much!

We’ve long known that [engineers are bad at gauging their own performance](https://interviewing.io/blog/own-interview-performance), so perhaps it shouldn’t come as a shock to find that interviewers also overestimate the effectiveness of the questions that they ask.

Of the interviewers who were not confident in their hiring choice (28%), we asked them why. This was the frequency distribution of their reasons.

Note that cheating isn’t mentioned anywhere!

Most interviewers concerned about their hiring decision expressed specific reasons for their lack of confidence. These issues typically included suboptimal solutions, missed edge cases, messy code, or poor communication. We specifically included an “Other Issue” category to see if they would express a concern that the interviewee was cheating, but digging deeper revealed only minor grievances like “personality issues” and “they need to speed up their coding.”

In addition to having this opportunity to call out cheating, interviewers were prompted three additional times to note any other concerns they had with the interview, including free-form text boxes and several multiple-choice questions with options to explain their concerns.

When an interviewee bombed because they didn’t understand the ChatGPT response, the interviewer chalked up the interviewee’s odd behavior and stilted responses to a lack of practice — not cheating. One interviewer thought the candidate's problem-solving was fine but commented that they were slow and needed to consider edge cases more carefully.

> *“Candidate did not seem prepared for any LeetCode questions."*

> *“Candidate's approach lacked clarity, and they jumped into the coding too early.”*

> *“The candidate was not prepared to tackle even the most basic coding questions on LeetCode.”*

> *“Good problem solving in general, but the candidate needs to be faster at coding and identifying critical edge cases.”*

So, who reported concerns about cheating? And who got caught?

**As it turns out, not a single interviewer mentioned concerns about any of the candidates cheating!**

We were stunned to discover that interviewers reported no suspicions of cheating, and interestingly, interviewees were largely confident that they were getting away with it, too. 81% reported no concerns about being caught, 13% thought they might have tipped off the interviewer, and an astonishingly small 6% of participants thought the interviewer suspected them of cheating.

Interviewees were mostly sure that their cheating went undetected

The candidates who worried they were caught did have abnormal comments from the interviewers in the post-survey analysis, but they still were not suspected of cheating. **To summarize, most candidates thought they were getting away with cheating — and they were right!**

## Companies: Change the questions you are asking immediately!

The obvious conclusion from these results is that **companies need to start asking custom questions immediately, or they are at serious risk of candidates cheating during interviews (and ultimately not getting useful signal from their interviews)!**

ChatGPT has made verbatim questions obsolete; anyone relying on them will be naively leaving their hiring processes up to chance. Hiring is [already tricky enough](https://interviewing.io/blog/we-ran-the-numbers-and-there-really-is-a-pipeline-problem-in-eng-hiring) without worrying about cheating. If you’re part of a company that uses verbatim LeetCode questions, please share this post internally!

Using custom questions isn’t just a good way to prevent cheating. It filters out candidates who have memorized a bunch of LeetCode solutions (as you saw, our custom question pass rate was significantly lower than our control). It also meaningfully improves candidate experience, which makes people way more likely to want to work for you. A while ago, we did an [analysis of what makes good interviewers good](https://interviewing.io/blog/best-technical-interviews-common). Not surprisingly, asking good questions was one of the hallmarks, and our best-rated interviewers were the ones who tended to ask custom questions! Question quality was extremely significant in our study, regarding whether the candidate wanted to move forward with the company. It was much more important than the company’s brand strength, which mattered for getting candidates in the door but didn’t matter relative to question quality once they were in process.

As some of our interviewees said…

> *“Always nice to get questions that are more than just plain algorithms.”*

> *“I liked the question — it takes a relatively simple algorithms problem (build and traverse a tree) and adds some depth. I also liked that the interviewer connected the problem to a real product at [Redacted], which made it feel less like a toy problem and more like a pared-down version of a real problem.”*

> *“This is my favorite question that I’ve encountered on this site. It was one of the only ones that seemed to have real-life applicability and was drawn from a real (or potentially real) business challenge. And it also nicely wove in challenges like complexity, efficiency, and blocking.”*

One more somewhat subtle piece of advice for companies who decide to move to more custom questions. You might be tempted to take verbatim LeetCode questions and change up the wording or some of the window dressing. That makes sense, because it’s certainly easier than coming up with questions from scratch. Unfortunately, that doesn’t work.

As we mentioned earlier, we discovered in this experiment that just because a question looks like a custom question, doesn’t mean it is one. Questions can appear custom and still be identical to an existing LeetCode question. **When making questions to ask candidates, it isn’t enough to obscure an existing problem.** You need to ensure that the problem has unique inputs and outputs to be effective at stopping ChatGPT from recognizing it!

The questions that interviewers ask are confidential, and we cannot share the exact questions that our interviewers used in the experiment. However, we can give you an indicative example. Below is a “custom question” with this critical flaw, which is easy for ChatGPT to beat:

```
For her birthday, Mia received a mysterious box containing numbered cards 
and a note saying, "Combine two cards that add up to 18 to unlock your gift!" 
Help Mia find the right pair of cards to reveal her surprise.

Input: An array of integers (the numbers on the cards), and the target sum (18). 
arr = [1, 3, 5, 10, 8], target = 18

Output: The indices of the two cards that add up to the target sum. 
In this case, [3, 4] because index 3 and 4 add to 18 (10+8). 
```

Did you spot the issue? While this question appears “custom” at first glance, its objective is identical to the popular [TwoSum](https://leetcode.com/problems/two-sum/) question: finding two numbers that sum to a given target. The inputs and outputs are identical; the only thing “custom” about the question is the story added to the problem.

Seeing that this is identical to known problems, it shouldn’t be a surprise to learn that ChatGPT does well on questions that have inputs and outputs identical to existing known problems — even when they have a unique story added to them.

### How to actually create good custom questions

One thing we’ve found incredibly useful for coming up with good, original questions is to start a shared doc with your team where every time someone solves a problem they think is interesting, no matter how small, they jot down a quick note. These notes don’t have to be fleshed out at all, but they can be the seeds for unique interview questions that give candidates insight into the day-to-day at your company. Turning these disjointed seeds into interview questions takes thought and effort — you have to prune a lot of the details and distill the essence of the problem into something that doesn’t take the candidate a lot of work/setup to grok. You’ll also likely have to iterate on these home-grown questions a few times before you get them right — but the payoff can be huge.

To be clear, we’re not advocating the removal of data structures and algorithms from technical interviews. DS&A questions have gotten a bad reputation because of bad, unengaged interviewers and because of companies lazily rehashing LeetCode problems, many of them bad, which have nothing to do with their work. In the hands of good interviewers, those questions are powerful and useful. If you use the approach above, you’ll be able to come up with new data structure & algorithmic questions that have a practical foundation and component that will engage candidates and get them excited about the work you’re doing.

You’ll also be moving our industry forward. It’s not OK that memorizing a bunch of LeetCode questions gives candidates an edge in today’s interview process, nor is it OK that interviews have gotten to a state where cheating starts to look like a rational choice. The solution is more work on the employer’s part to come up with better questions. Let’s all do it together.

## Real talk for job seekers

All right, now, for all of you who are actively looking for work, listen up! Yes, a subset of your peers will now be using ChatGPT to cheat in interviews, and at companies that ask LeetCode questions (sadly, many of them), those peers will have an edge… for a short while.

**Right now, we’re in a liminal state where companies’ processes have not caught up to reality. They will, soon enough, either by moving away from using verbatim LeetCode questions entirely (which will be a boon for our entire industry) or by returning to in-person onsites (which will make cheating largely impossible past the technical screen) or both.**

It sucks that other candidates cheating is another thing to worry about in an [already difficult climate](https://interviewing.io/blog/you-now-need-to-do-15-percent-better-in-technical-interviews), but we cannot, in good conscience, endorse cheating to “level the playing field.”

In addition, interviewees who used ChatGPT uniformly reported how much more difficult the interview was to complete while juggling the AI.

Below, you can view one interviewee stumbling through their time complexity analysis after giving a perfect answer to an interview question. The interviewer is confused as the interviewee scrambles to explain how they got to their incorrect time complexity (secretly provided by ChatGPT).

[https://www.youtube.com/embed/jtcCK0yr9Bg](https://www.youtube.com/embed/jtcCK0yr9Bg)

VIDEO

While no one was caught during the study, their cameras were off, and cheating was still difficult for many of our skilled candidates, as evidenced by this clip.

Ethics aside, cheating is difficult, stressful, and not entirely straightforward to implement. Instead, we advise investing that effort into practice, which will serve you well once companies change their processes, which hopefully should be soon. Ultimately, we hope the advent of ChatGPT will be the catalyst that finally moves our industry’s interview standards away from grinding and memorizing to actually testing engineering ability.

Michael Mroczka

Michael Mroczka, a Google SWE, is one of the highest-rated mentors at interviewing.io and primarily works on the Dedicated Coaching program. He has a decade of coaching experience, having personally helped 100+ engineers get into Google and other desirable tech companies. After receiving multiple offers from tech companies early in his career, he enjoys teaching others proven techniques to pass technical interviews.

He also sometimes writes technical content for interviewing.io (like this piece) and was one of the authors of interviewing.io’s [A Senior Engineer's Guide to the System Design Interview](https://interviewing.io/guides/system-design-interview).

*Special thanks to Dwight Gunning and Liz Graves for their help with this experiment. And of course a big thank you to all the interviewees and interviewers who participated!*

## Footnotes: