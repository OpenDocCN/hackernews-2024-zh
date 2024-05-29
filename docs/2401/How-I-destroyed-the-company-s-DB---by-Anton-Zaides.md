<!--yml
category: 未分类
date: 2024-05-27 14:24:26
-->

# How I destroyed the company's DB - by Anton Zaides

> 来源：[https://zaidesanton.substack.com/p/how-i-destroyed-the-companys-db](https://zaidesanton.substack.com/p/how-i-destroyed-the-companys-db)

It was on a quiet Saturday.

I received a message from the support team, telling me one of our customers has a problem. I decided it was important enough to start debugging. After 15 minutes, I understood the issue - there were some corrupted orders created in our DB, and we needed to delete them.

Sounds trivial, right?

*At the end of the article I share my lessons, and how you can apply them to your own team. And welcome to the 600+ new subscribers since last week!*

*For those of you not working in startups - please don’t judge* 😅

There were a few hundred orders to delete, so I decided to not do it manually but to write a short SQL query (red flag 🚩)

It was a bit more complex than that, but to simplify:

> `UPDATE orders
> SET is_deleted = true
> 
> WHERE id in (1, 2, 3)`

You can already see the scale of the disaster…

I pressed CTRL+Enter and ran the command. When it took longer than a second, I understood what happened. The program I used (DBeaver) saw the empty 3rd line, and ignored the 4th line.

Yes, I deleted all the orders in the DB 😢

I felt physically ill, and helpless.

After a deep breath, I knew I had to act, and fast. There was no place to make more mistakes, or waste time.

The recovery was done much better.

1.  Stopping the systems - ~5 minutes

2.  Creating a clone of our DB from before the change (luckily we had Point-in-time-recovery set up) - ~20 minutes

3.  Calling my manager during the wait 😨

4.  Updating the production DB’s information based on the clone* - ~15 minutes

5.  Starting our systems - ~5 minutes

** I decided to not restore the whole DB, because I couldn’t stop ALL systems, as we have multiple independent ones. I didn’t want to lose the changes made during the recovery process. We use managed PostgreSQL by GCP, so I created a new clone from a time before the update. Then I exported just the ‘id’ and ‘is_deleted’ columns from the clone, and imported the result into the production DB. Afterwards, it was a simple update+select query.*

So 45 minutes of easily avoidable downtime…

This may sound like a very stupid mistake you will never make (or even can’t make - in bigger companies). It might be true. The problem is not the wrong SQL command. **A small human mistake is never the true problem**. Me running that command, is just the end of a whole chain of failures.

1.  Working on production during the weekend - why? In this case, it was not even that urgent. Nobody asked me to fix it immediately. I could have easily waited for Monday.

2.  Who the hell runs something on a prod DB without running it on QA first?

3.  Why did I manually edit the DB and not use an API call?

4.  And if there was no API - why didn’t I call a teammate and have ‘4-eyes’ on such a sensitive action?

5.  **And the worst part - why didn’t I use transactions?** It’s as easy as writing ‘Begin’, and then using Rollback in case of a mistake.

The mistakes are built one atop the other. If at least one of them was avoided - the whole thing would have never happened. The answer to most questions is that I was just overconfident.

At least the organized recovery stopped the chain. Imagine the disaster if I couldn’t restore the DB to the correct state…

A few months ago, I finished reading *‘Chernobyl: The History of a Nuclear Catastrophe’.*The chain of mistakes there, reminded me of that cursed weekend (without underestimating or comparing to the dimensions of the Chernobyl disaster).

1.  There was a root technological problem in the RBMK reactor.

2.  It was not communicated properly. There were previous incidents involving the problem, but Chernobyl’s team was not familiar with them.

3.  The team didn’t follow the procedure during a safety check.

4.  After the explosion - the Soviet government tried to hide it, which greatly increased the severity of the damage.

**Who is responsible?**

The designers of the reactor? The teams at other power plants who didn’t communicate the issues they had? The team at Chernobyl? The Soviet government?

All of them. **Disasters are never caused by a single mistake,** but by a chain of them. Our job, is to do the best we can to cut the chain as early as possible.

I didn’t know what to expect from the talk with my manager on Monday.

He surprised me: “Make sure it won’t happen again. But I prefer it that way - you made the mistake because you are dedicated, and love to move fast. People with a bias for action break things.”

That is exactly what I needed to hear. A too ‘cuddly’ approach saying “That’s ok, don’t worry, thanks for fixing it!” would have felt fake. On the other hand, I already felt like shit, so no point in humiliating me further.

Since then:

*   We try hard to remove the need for DB access, creating the relevant API calls

*   I always run queries on QA first (I know.. obvious, right? Nothing makes a lesson stick more than a disaster)

*   I consult with the PM to understand what’s really urgent, and what can wait.

*   Any update/insert/delete query on production is done by 2 people. This one actually prevented other mistakes!

*   I started to use transactions 🤦‍♂️

After the incident, I shared a detailed explanation with my team. Not hiding anything, or downplaying my fault.

There is a thin balance, between shaming people, and not holding them accountable. When YOU make mistakes - it’s a great opportunity to send the right message.

**If you apologize** 1000 times, they’ll think you expect the same from them.

**If you’ll make fun** of the incident, ignoring the implications, they’ll think it’s ok.

**If you’ll be accountable**, learn, and improve - they’ll behave the same way.

*   Encourage people to **act,** care about the customer, and solve problems. That’s how startups succeed.

*   When mistakes are made, hold the person **accountable**. Together, understand how it could have been avoided.

*   No need to make people feel worse than they already do. **Some people need more accountability, some more encouragement**. I always prefer to err on the side of encouragement.