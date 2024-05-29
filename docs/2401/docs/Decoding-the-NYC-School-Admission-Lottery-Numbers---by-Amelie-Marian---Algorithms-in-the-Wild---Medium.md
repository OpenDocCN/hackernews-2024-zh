<!--yml
category: 未分类
date: 2024-05-27 14:54:35
-->

# Decoding the NYC School Admission Lottery Numbers | by Amelie Marian | Algorithms in the Wild | Medium

> 来源：[https://medium.com/algorithms-in-the-wild/decoding-the-nyc-school-admission-lottery-numbers-bae7148e337d](https://medium.com/algorithms-in-the-wild/decoding-the-nyc-school-admission-lottery-numbers-bae7148e337d)

# How to decipher the Hexadecimal Lottery Numbers

*tl;dr: To get a rough idea of how “good” your lottery number is, just look at the first character of the hexadecimal string. If it is a number, your lottery number is in roughly the first two-thirds of all ordered lottery numbers, the lower the better. If it is a letter, your lottery number is in the last third. A lottery number starting with 0–3 is in the first quartile, one starting with c-f in the last.*

These 32-characters numbers are in a format called UUID and are likely generated using a random number generator (see below for details) that creates uniformly distributed numbers. The numbers are compared left to right, in increasing order: from 0 to f (0–9 then a-f). This means that the first character is enough to give you a rough idea of how good your number is: a lottery number that starts with 0 is in the first 1/16th (6.25%), one that starts with F in the last 1/16th.

First two characters of lottery number to percentile

To differentiate further, we can look at the first two characters: a lottery number that starts with ‘00’ is in the first 256th (0.4%), ‘01’ the 2nd 256th, and so on. The first two characters are sufficient to identify where your number is expected to stand in comparison to other numbers, with a 0.4% precision. The attached table shows how to convert a lottery number’s first two characters to a percentile.

So why are the numbers so long? As mentioned above, they are UUID ([Universally Unique Identifiers](https://en.wikipedia.org/wiki/Universally_unique_identifier)), Version 4 (you can identify the version by looking at the first character of the third block of characters — the 13th character). UUID V4 are used to generate random unique identifiers, a random version-4 UUID will have a total of 2¹²² (5.3 [undecillion](https://en.wikipedia.org/wiki/Names_of_large_numbers)) possible numbers. There are several existing UUID V4 number generators available, and it makes sense for the programmers of the NYC DOE lottery to have used an existing — and well-tested — random number generator library function, such as the python one used to generate 10 UUIDs in the example below.

Randomly generated UUID V4 numbers

What does not make as much sense is for the DOE to provide the full numbers to families. For a decision process to be ***transparent and accountable, it needs to be simply explainable***. The 32-character numbers look unnecessarily complex because they are. Most of the characters are just noise and have no impact on the student’s admission outcome, only the first 8 characters will ever be used. It would be much easier for families to understand their chances if the DOE were providing them with information in this format:

> Your lottery number is ‘fa8058a5’, it is in the 98th percentile
> Your lottery number is ‘9cf4f694’, it is in the 61.3th percentile
> Your lottery number is ‘1f5124de’, it is in the 12.5th percentile

Some families who requested their lottery numbers by email did receive a detailed explanation with percentile information, along with the numbers. Many just received the 32-character lottery number without additional details. The fact that the lottery numbers and their explanations are not given to all applicants, but only upon request, creates another source of inequity, where some in-the-know families are given more information than others. ***There is no reason why the lottery number information shouldn’t be available to all on the MySchools portal.*** ****update: for the 2022 admission season, the DOE has provided families with their lottery number on their MySchools account****

**Are the first 8 characters really enough?** Each cohort of applicants has historically been between 60,000 and 80,000 thousand. In practice, the implementation will never go past the first eight characters to compare students' lottery numbers: the first 8 characters of the lottery number provide 16⁸, or over 4 billion, possible combinations. Interestingly, similar to the [birthday paradox](https://en.wikipedia.org/wiki/Birthday_problem), there is actually a [50% probability](https://blogs.sas.com/content/iml/2013/07/03/duplicates-in-random-numbers.html) that, citywide, two students in a given year will share the same first 8 characters. However, the numbers are not used as identifiers but as tie-breakers, so one duplicate every other year seems acceptable and unlikely to have any real impact (the implementation can still use the longer numbers if preferred). If it isn’t acceptable, then the first 12 characters would guarantee a 99.999% probability that there are no duplicates.

The truth is that families don’t care much about the actual numbers, rather they want an idea of their student’s chances, and guarantees that the numbers were generated fairly. The use of overly long and opaque numbers is raising more questions than they answer: parents on internet boards are convinced that the DOE is tipping the scales by favoring students from some schools, or demographics, over others; that the numbers are encoding all types of information used in the match. They use anecdotal data to confirm their fears. The lack of transparency is the main cause of mistrust. If the DOE had clearly stated how the numbers were generated (maybe sharing which library function is used), and explained how the numbers are processed from the beginning, families would have more trust in the system.