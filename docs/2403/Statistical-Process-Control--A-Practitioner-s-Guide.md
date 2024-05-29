<!--yml
category: 未分类
date: 2024-05-27 14:41:04
-->

# Statistical Process Control: A Practitioner's Guide

> 来源：[https://two-wrongs.com/statistical-process-control-a-practitioners-guide.html](https://two-wrongs.com/statistical-process-control-a-practitioners-guide.html)

Now we know what common cause variation is. We measure it with *process behaviour charts*, also known as *control charts*.

There are many types of process behaviour charts⁷ And with some statistical knowledge, you might be able to reach for the specialised ones in the right context., but the one you can almost always rely on to work is the *individuals chart*, or *XmR chart*.

Here’s how you make it for a time series, like Alice’s weekly call numbers. Start by plotting the values in a run chart, as before.

Then, draw a line at the mean (average) value of that data: \(\overline{\mathrm{X}} = 84\).

Now, compute the absolute differences between consecutive data points:

| **X** | 86 | 96 | 65 | 101 | 90 | 70 | 85 | 75 |
| **mR** |   | 10 | 31 | 36 | 11 | 20 | 15 | 10 |

The upper row contains Alice’s call numbers. The lower row has the differences between adjacent numbers⁸ These differences are sometimes called *moving ranges* – hence the name XmR chart. It is based on the X values themselves and their moving ranges.. The difference between the first two numbers (86 and 96) is 10, the difference between the second two (96 and 65) is 31, and so on.

Compute the mean of the consecutive differences: \(\overline{\mathrm{mR}} = 19\).

Here comes the magic. Compute the *lower and upper natural process behaviour limits* (also known as *lower and upper control limits*) as

\[\mathrm{LPL} = \overline{\mathrm{X}} - 2.66 \times \overline{\mathrm{mR}},\] \[\mathrm{UPL} = \overline{\mathrm{X}} + 2.66 \times \overline{\mathrm{mR}}.\]

Concretely, in the case of Alice, we get

\[\mathrm{LPL} = 84 - 2.66 \times 19 = 33,\] \[\mathrm{UPL} = 84 + 2.66 \times 19 = 135.\]

The number 2.66 seems like a magic constant, and it is. There’s a little information in the appendix on where it comes from. The important thing is that you use specifically 2.66, no more, and no less. It’s very important, because people will ask you to use something different, and you must not oblige.⁹ See below about *The Voice of the Process*.

Add to your chart lines for these process limits.

These natural process limits indicate the range of sales calls numbers we can expect from Alice, assuming she doesn’t change anything fundamental about how she works. The process limits incidate the amount of week-to-week variation in her work process. They are a measure of the amount of common cause variation, the amount of noise.

In other words, you should not be surprised if Alice makes 35 calls one week, because that’s within the process limits. It’s just like rolling snake eyes when throwing dice. Rare, sure, but it does happen. It’s fully within the range of outcomes you should expect of the process. Similarly, some weeks she might get lucky and make 128 calls – nothing extraordinary has changed that week; Alice just got lucky within her regular process.

Alice’s call numbers are an easy case to deal with: they make up a stable process. We can predict her future performance based on the past. The name *stable process* is a bit confusing because a stable process is one where the outcome is completely random. A stable process is one where we cannot predict any individual value, but we know the range into which almost all values will fall.

In traditional spc litterature, stable processes are known as *in (statistical) control* or *controlled*. What is meant by this is that we have already controlled for all the significant external factors, and there is nothing left we can control to determine the outcome. We just have to let the process run on its own and produce what it is tuned for.

When you have a stable process such as this, you don’t have to re-compute the process limits each week. One of the defining features of a stable process is that any given week, statistically, looks like any other week. Because of this, you can just extend the process limits you have already computed indefinitely into the future.