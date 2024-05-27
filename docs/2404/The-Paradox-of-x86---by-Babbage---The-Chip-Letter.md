<!--yml
category: 未分类
date: 2024-05-27 13:18:49
-->

# The Paradox of x86 - by Babbage - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/the-paradox-of-x86](https://thechipletter.substack.com/p/the-paradox-of-x86)

**Bodiam Castle and its moat in Sussex in England. That moat looks hard to cross in either direction!** *By WyrdLight.com, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=7910287*

*The ‘moat’ is a familiar business concept. It’s a barrier that protects you from would-be competitors. Everyone wants their business to be protected by a strong moat.*

*Real-life, physical moats can also stop defenders leaving the protection of the fortress they encircle. But staying inside the moat can be risky. It lets would-be attackers build strength outside, until they are ready to breach the moat.*

*In this post we’ll look at an example where a business moat has led to the same phenomenon. It’s the story of Intel’s x86 Instruction Set Architecture (ISA).*

*An ISA, the set of ‘instruction codes’ that a microprocessor can understand and run, may seem like an arcane technical detail. As we’ll discover though, it’s been a key part of Intel’s business for nearly five decades and is arguably right at the root of the firm’s current problems.*

*Those problems have had major economic and geopolitical consequences as Intel, and by extension the U.S., have fallen behind in advanced semiconductor manufacturing. This, in turn, has led to the U.S. government needing to provide tens of billions of dollars of subsides to Intel and to TSMC and Samsung.*

* * *

Over the last week there has been lots of coverage of Intel’s announcement of the financial results of its newly defined and separated ‘Intel Foundry’ business. Intel Foundry is the part of Intel that makes chips both for use in Intel’s own products and, Intel hopes, increasingly for external customers.

Spoiler alert, they’re not good. Not good at all.

That’s a loss of $7.0 billion in 2023 for Intel Foundry.

As if this wasn’t bad enough, Intel, as it now admits, lost manufacturing leadership with its 10nm fabrication process, around a decade ago.

Intel expects (hopes) that it will overtake TSMC again with its Intel 18A process, but only after almost a decade trailing the Taiwanese firm.

How did Intel arrive at this position after leading for so many years? The firm may blame a ‘structurally more competitive environment’, but that’s not the full picture.

As Intel admits, its old model for manufacturing relied on proprietary intellectual property and processes, focused on delivering only internal products and monetization of manufacturing through sales of those products.

As

has [highlighted](https://www.semianalysis.com/p/rebuilding-intel-foundry-vs-idm-decades), this model allowed Intel to accumulate and hide numerous inefficiencies in its manufacturing and design processes.

These inefficiencies have now been exposed and Intel has (re)discovered that Moore’s Law is at its core, about economics (headline below Intel).

I wrote about the importance of economics to Moore’s Law in [Moore on Moore](https://thechipletter.substack.com/p/moore-on-moore) back in February. Quoting Intel founder Gordon Moore:

> Moore’s law is really about economics. My prediction was about the future direction of the semiconductor industry, and I have found that the industry is best understood through some of its underlying economics.
> 
> *Gordon Moore*

Now Intel faces an uphill struggle. It needs to regain its process prowess, invest many tens of billions in new fabs, and build a customer base for Intel Foundry.

For more on Intel’s plans and challenges then please turn to the further reading at the end of this post.

The rest of this post, though, is going to dig into the history and origins of Intel’s malaise. I believe that its origins can be seen much earlier, in two things that happened (or rather, from Intel’s perspective, didn’t happen) in 2007.

[Share](https://thechipletter.substack.com/p/the-paradox-of-x86?utm_source=substack&utm_medium=email&utm_content=share&action=share)

These problems have a single root cause, which also happens to be one of Intel’s biggest strengths, a moat that has protected Intel’s most important business for over four decades: the x86 Instruction Set Architecture (ISA).

An ISA is the collection of machine instructions that a processor can execute.

When we say ‘the x86 ISA’ we really mean the collection of ISAs for a range of processors starting with Intel’s 8086 from 1976 (hence the name x86), leading to today’s latest processor designs from Intel and AMD. As we saw in the Trillion Dollar Stopgap, the 8086 and its successors were only ever meant to last for a few years.

The 8086 wasn’t even a completely new design. As Intel’s launch publicity stated the 8086 was an evolution of the earlier 8-bit 8080 microprocessor, which in turn was an evolution of Intel’s first 8-bit microprocessor the 8008, which was an implementation of the architecture of DataPoint 2200 terminal from 1970\.

x86 has its origins in a very old design and one that Intel didn't even develop!

Over the nearly five decades since the launch of the 16-bit 8086, the x86 ISA has evolved hugely. Today’s x86 code is 64-bit, and benefits from the addition of hundreds of new instructions that enhance its capabilities. Those designs have consistently supported a high degree of ‘backwards compatibility’. Users have been able to run their existing software on new designs, even as the x86 ISA has been evolved and been enhanced.

And over those five decades, x86 has been enormously successful. Billions of processors have been shipped using the x86 ISA, generating over a trillion dollars in revenue for Intel. In part that’s because consumers and businesses have needed to buy an x86 compatible PC in order to run first MS-DOS and then Windows software. More recently, x86 designs have become dominant on servers as Intel’s manufacturing prowess, supported by large volumes of PC sales, has enabled the company to create powerful designs that outperformed competitors.

The x86 ISA has acted as a ‘moat’ for Intel’s business over four decades, as it’s difficult, and has become much harder over time, to create a competitive x86 design from scratch.

This has always not always been impossible, as AMD, NEC, Via, and others, have done so in the past. Over the years they have mostly fallen away, unable to compete with Intel. Today though, as x86 has become more complex, AMD is left as the only other firm left making x86 designs that compete directly with Intel.

> Read more about NEC’s efforts to compete with Intel with its own version of x86 in ‘Intel vs NEC : The case of the V20’s Microcode’.

* * *

The ‘x86 moat’ sounds like it is a great asset, so why has it led to Intel’s recent problems?

Let’s go back to Intel Foundry. To say that the success of Intel foundry is crucial to Intel’s future would be an understatement. Put simply, Intel is building a lot of expensive fabs. It will use them to build its own x86 products. But to fill them, and ultimately make them financially viable, it needs lots of third party customers, and almost none of those customers will be using x86 in their designs.

Intel staged a lavish presentation on Foundry back in February. See the end of the post for a link to a full recording of the event.

One moment stood out in the keynote. The appearance of Arm’s CEO, Rene Haas.

As readers will know, Arm doesn’t make chips. Instead it licenses its Intellectual Property to others to build into their own System-on-Chips (SoCs). That IP was originally processor designs using Arm’s own ISA(s) - it’s common to talk about ‘The Arm ISA’ (singular) but there are multiple current and legacy ISAs designed by Arm - but now extends to licensing of the ISAs themselves to third parties. Major firms that license the Arm ISA and use that ISA in their own CPU designs include Apple, Broadcom, Qualcomm and Nvidia.

> Readers can follow the Arm story from its origins in Acorn computers in Cambridge in the UK, through the “Arm Barn” and into Nokia 3G mobile phones in the posts below.

Today, Arm’s customers using Arm IP-based products completely dominate the market for smartphone SoCs. And they compete with Intel in servers and increasingly on the desktop. Apple Silicon used in today’s Macs uses the 64-bit Arm ISA as Apple has moved away from x86.

> More on Apple’s leapt over the x86 moat and made the transition to Arm-based Apple Silicon:

All three of the biggest US ‘hyperscalers’ starting with Amazon and now joined by Microsoft and most recently Google, with the launch of [Axion](https://cloud.google.com/blog/products/compute/introducing-googles-new-arm-based-cpu), are now building their own Arm based server CPUs.

> More on how Arm found its way into Amazon Web Services:

* * *

Arm is a relatively small company with 2023 revenues of $2.8bn compared to Intel’s $54bn, but customers using Arm IP have over $1.5 tn of revenue (yes that’s over one-and-a-half trillion dollars in revenue).

**Inviting Haas onto stage here can be compared to letting down the drawbridge over the moat and asking the mastermind behind the invading forces to come in for a polite cup of tea.**

But Intel now needs Arm. Without Arm’s CPU designs in Intel made SoC’s then its Foundry business and its newly built fabs will struggle. Welcoming Haas to an Intel stage follows the partnership between the two firms [announced](https://www.theregister.com/2023/04/12/arm_and_intel_fab_deal/) last year.

Which takes us to the first of the two things that didn’t happen for Intel in 2007\. First let’s provide some context for Intel’s business in 2007.

[Share](https://thechipletter.substack.com/p/the-paradox-of-x86?utm_source=substack&utm_medium=email&utm_content=share&action=share)

2006 had been a year of triumph for Intel. After the mis-steps of [Itanium](https://en.wikipedia.org/wiki/Itanium) and [Netburst](https://en.wikipedia.org/wiki/NetBurst) it had staged a comeback with the lower-power x86 [Core](https://en.wikipedia.org/wiki/Intel_Core_(microarchitecture)) architecture, vanquishing a temporarily resurgent AMD on the desktop, and continuing its trajectory towards a dominant position in servers.

And Intel gained a high profile new client when Apple switched from PowerPC to Intel’s CPUs in the Mac.

Apple’s Steve Jobs announces the Mac’s transition from PowerPC to Intel

But Apple was the scene of the first major misstep for Intel in 2007.

Intel passed on making the System on Chip for Apple’s new iPhone, launched at the start of 2007\. Instead Apple used an Arm designed processor in an SoC built by Samsung.

Too late Intel realised the mistake. Over ten billion of dollars were spent on an, ultimately futile, attempt to gain a foothold in the smartphone SoC market. Intel finally [gave up](https://www.theverge.com/2016/5/3/11576216/intel-atom-smartphone-quit) in 2016, ceding the market to Arm processors in SoCs made by Samsung and TSMC.

Today, Apple is TSMC’s biggest customer, and smartphones make up over 40% of the Taiwanese company’s revenues.

* * *

Intel’s then CEO, the late Paul Otellini gave a refreshingly direct mea-culpa on Intel’s missing the iPhone, in an [interview](https://www.theatlantic.com/technology/archive/2013/05/paul-otellinis-intel-can-the-company-that-built-the-future-survive-it/275825/) with the Atlantic, shortly before his retirement from Intel in 2013:

> "The thing you have to remember is that this was before the iPhone was introduced and no one knew what the iPhone would do... At the end of the day, there was a chip that they were interested in that they wanted to pay a certain price for and not a nickel more and that price was below our forecasted cost. I couldn't see it. It wasn't one of these things you can make up on volume. And in hindsight, the forecasted cost was wrong and the volume was 100x what anyone thought."

So, the iPhone ‘miss’ was a straightforward forecasting error? And the decision was driven purely by the financial outcomes indicated by those forecasts? In fact, I think it would be remarkable if this was the case, for reasons that we will discuss in a moment.

In any event, it was a huge lost opportunity. In 2007 Intel was clear process leader.

Intel shipped its first 45nm chip in 2007\. The iPhone 3GS, which shipped in 2009, used an SoC built by Samsung using a 90nm process.

Losing the iPhone and smartphones gifted hundreds of billions of dollars of business to Samsung and TSMC, business that was crucial in their efforts to catch-up with and then overtake Intel.

* * *

The second miss in 2007 was the lack of a direct response to CUDA, Nvidia’s software ecosystem for general purpose computing on its GPUs.

Intel has a long, but sparse, history in discrete GPUs.

Graphics Card with Intel i740 GPU - By Konstantin Lanzet - Own Collection, GFDL, https://commons.wikimedia.org/w/index.php?curid=8400155

*The rest of this post is for premium subscribers. Please click the button below to upgrade.*