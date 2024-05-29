<!--yml
category: 未分类
date: 2024-05-27 14:35:46
-->

# 60-Bit Computing - by Babbage - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/60-bit-computing](https://thechipletter.substack.com/p/60-bit-computing)

Control Data Corporation 6600

Sometimes looking at historic systems can prompt us to challenge our thinking.

The Control Data Corporation 6600 was the fastest computer in the world from 1964 to 1969\. It was, in many ways, a pioneer of modern computing using, for example, some of the techniques that would form the basis of RISC architectures.

Quoting Wikipedia:

> Generally considered to be the first successful supercomputer, it outperformed the industry's prior recordholder, the IBM 7030 Stretch, by a factor of three.

The CDC 6600 was designed by Seymour Cray, who later founded Cray Research and designed the famous Cray supercomputers, and system architect John Thornton.

There is something about the CDC 6600 architecture though that, from a modern perspective, looks a little odd. The word length was 60 bits.

The CDC 6600 and its successors were the only major systems to use 60\. So this starts us off on a small detective story. Why did they use 60 and not 64?

It’s worth emphasising that 60-bit wasn’t just used for memory addresses. It was used throughout the system, crucially for register length and floating point arithmetic.

Before we look at the CDC 6600 there was one major design that considered and rejected 60 bits. The IBM Stretch, the CDC6600’s predecessor as the fastest computer in the world.

We can share the insights of Stretch’s designers in a memo from 1956\. The first striking thing is that the industry had not yet settled on an 8-bit byte. In fact, 6-bit ‘bytes’ were seen as more important than 8-bit bytes.

Second is the fact that there was no settled standard for floating point arithmetic so 60 bits was seen as viable for ‘higher precision’ floating point calculations.

Finally, there was cost. The Stretch would be an expensive machine and so a small saving in memory costs would still be significant.

The document hinted that 64-bit might be the better choice in the long term.

The Stretch’s designers eventually decided on 64 bits. The full memo can be read below.

It’s worth mentioning here that non-power-of-two architectures were common in this period. Several DEC PDP minicomputers and mainframes used 18-bit or 36-bit words.

So what about the CDC6600? There are two publications by John Thornton, the architect of the CDC6600, that set out its design in detail. Neither of these includes any consideration of the word-length. 60-bits is taken as a given throughout.

So we don’t have much on the decision-making process, but there are some clues.

The CDC6600 was a monster. It weighed over five tons. It cost $2.3m in 1965 which would be over $21m in 2022 terms. So one can easily see how an extra four bits would add a non-negligible amount to an already very expensive machine. If not absolutely necessary then those extra four bits could go.

Seymour Cray would later move to a 64-bit word length with his Cray-1 design of 1975\. Now no one is going to design a 60-bit general purpose computer today. We don’t have the mental burden of choosing a byte length. The 8-bit byte won. But looking at the CDC6600 reminds us that this was a choice and that making this choice involved trade-offs. And just occasionally it’s worth revisiting these trade-offs.

By Jitze Couperus - Flickr: Supercomputer - The Beginnings, CC BY 2.0, https://commons.wikimedia.org/w/index.php?curid=19382150