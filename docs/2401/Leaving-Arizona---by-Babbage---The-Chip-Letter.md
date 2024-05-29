<!--yml
category: 未分类
date: 2024-05-27 14:49:07
-->

# Leaving Arizona - by Babbage - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/leaving-arizona](https://thechipletter.substack.com/p/leaving-arizona)

**I was able to scoop up the best people from the Motorola design team.**
Chuck Peddle

In [Motorola’s Pioneering 6800 : Origins and Architecture](https://thechipletter.substack.com/p/motorolas-pioneering-8-bit-6800-origins), we saw how a small team, based in Phoenix, Arizona, developed the Motorola 6800 microprocessor. The announcement of the 6800, in April 1974, meant that Motorola seemed to have a contender for early leadership in microprocessors. Compared to its leading 8-bit rival, the Intel 8080, the 6800 had an elegant and clean instruction set and a design that was easier to build systems around.

Prototype 6800 Board - See https://www.computerhistory.org/collections/catalog/102711296

The 6800 team created a simple single-board computer (pictured above), with just six integrated circuits, which they used to demonstrate the 6800 to Motorola’s senior management. According to Tom Bennett, who led the design of the 6800:

> Chuck Peddle and John Buchanan wired up and put together and programmed it. And all it did was say: “Hi, John” or “Hi, Bob,” …

Bob was Bob Galvin, Motorola’s Chief Executive, son of the company's founder.

Galvin, seeing the potential of the 6800, and concerned that it might replace Motorola’s other products, asked the team:

> You understand that you’re putting our customer’s chip -- or system -- on one of these little boards? What’s that gonna do to my other products?

The 6800 wasn’t available in volume at the time of its announcement, but samples were soon sent to prospective users. By June 1974 key potential customers, such as Hewlett Packard, had prototype systems built around the 6800.

With the launch of the 6800, Motorola didn't just have a promising product. It also had a talented and innovative team in Phoenix. Along with Tom Bennett, the team included Bill Mensch, Bill Lattin, Rod Orgill and Chuck Peddle, each of whom would go on to have highly distinguished careers.

Everything was looking good for Motorola’s nascent microprocessor business. But then, in early 1974, not long after the 6800 was announced, Motorola’s management decided to move the team to Texas.

**… I think his intention was to raid Motorola for engineers.** Bill Mensch on Chuck Peddle’s plans when he joined Motorola

The growth of Motorola’s semiconductor business in the early 1970s meant that the firm needed to expand its manufacturing capabilities. Austin, Texas was chosen as a new site and management decided to move the 6800 team there.

Bill Lattin started to travel to Austin to prepare for the move, whilst becoming increasingly concerned about the disruption it would cause. The semiconductor industry was about to enter recession and Motorola’s semiconductor business was losing millions of dollars. The company would shortly lay off thousands of staff. Lattin later recalled that he told Motorola’s management:

> Listen, you know, conditions are worsening in the environment here, and this is our last time to call off this move. I don’t want to get half-way down to Austin and have this thing blow up on me.

He extracted a promise from his manager that engineers who made the move to Austin would be secure in their jobs. The move went ahead in the middle of 1975, shortly after the 6800 had entered volume production at the end of 1974\. Then, Lattin’s boss lost his own job and his replacement started to demand job cuts in the 6800 team. It was too much, and soon Lattin took up the long-standing offer from his former teacher Andy Grove to join him at Intel.

Other key members of the 6800 team, including Tom Bennett, declined to move to Austin, and instead moved to other business units within Motorola.

Chuck Peddle - from Byte magazine in 1982

And, possibly most damaging of all for Motorola, the move to Austin had already prompted some of the team, including Rod Orgill and Bill Mensch, to leave and start work on the design of a competitor. It was Chuck Peddle, who had joined the 6800 team later in its development, who was the driving force behind the move. Mensch has said that when Peddle joined Motorola “… his intention was to raid Motorola for engineers.”

One can sense Peddle’s discontent at Motorola. In the book ‘Commodore: A Company on the Edge’, Peddle is less than flattering about the 6800:

> They kind of muddled their way through the architecture of the 6800, which had some flaws in it. … I was able to fix some of those flaws, but it was too late for others.

However, when Peddle presented the 6800 and its family to customers, the biggest concern wasn’t the 6800’s architecture, but its price. The 6800 processor was launched with a price of $360, closely matching the price of the latest 8-bit Intel processor, the 8080\. Peddle later recalled that in his presentations: 

> The guys would sit down, we would explain the 6800, and they would … fall in love …

But this was followed by complaints about the price. Customers would say:

> You're charging too much for it. What I want to use it for is not to replace a minicomputer. I want to use it to replace a controller, but at $300 per device it's not cost-effective.

Peddle started to push Motorola’s management to produce a low-cost version of the 6800. For simple control applications, the competition was either a custom design or Intel’s 4-bit 4040 processor. The 4040 cost around $29, so Peddle set a target of $20 for a low-cost version of the 6800\. After months of trying to get Motorola’s management to adopt his proposal, he was instructed, in writing, to stop pursuing the idea. His response was a letter to Motorola’s management, stating that, in Peddle’s words:

> This is product abandonment, therefore I am going to pursue this idea on my own. You don't have any rights to it because this letter says you don't want it.

But there was a reason the 6800 was expensive. It was made using ‘contact lithography’, where the photomask, containing the image that is to created on the silicon die, comes into direct contact with the silicon wafer. This inevitably led, over time, to damage to the photomask, reducing yields and eventually rendering the expensive photomask unusable. Making a low-cost version of the 6800 would be impossible without a more cost-effective manufacturing process.

Peddle was determined to pursue his idea and, whilst still at Motorola, started to talk to a number of other firms about making it a reality. According to Peddle:

> And the guy I went to build first was L.J. Sevin at Mostek. And L.J.’s comment to me much later was, “I didn't do the deal because I was afraid Motorola was going to sue you, and I didn't want to deal with Motorola”.

Eventually, he made contact with John Paivinen whose company, MOS Technology, based in Pennsylvania, was already building complex calculator chips. Peddle’s vision appealed to Paivinen, but there was one problem. His firm was using a PMOS fabrication process and didn’t have a working NMOS process, that would be needed for the new processor. However, Paivinen convinced Peddle that his firm could have NMOS in production in the year or so it would take Peddle and colleagues to design a new processor.

An image of MOS Technology’s Valley Forge base from an early advertisement.

So, in August 1974, Peddle, Orgill, Mensch and five other members of the 6800 team, travelled to MOS Technology’s Valley Forge offices. According to Peddle, “I was able to scoop up the best people from the Motorola design team.”

There they started making the idea of a low-cost 8-bit microprocessor a reality.

The 6800 had been announced in April 1974 but Motorola’a shift from PMOS to a new NMOS process proved problematic and it wasn't until November that it was in volume production, giving Intel’s 8080 a clear six-month lead.

The 6800 team hadn’t been working to make a competitor to the minicomputer. The target was devices such as ‘dumb’ ‘Cathode Ray Tube’ terminals, used to connect to ‘real’ mini and mainframe computers. An early proposal for a more advanced 16-bit processor design, that would have been competitive with minis, had quickly been rejected as impractical.

But the availability of the Intel 8080 in 1974 had enabled the creation of the Altair 8800, the first commercially successful ‘personal computer’. Launched in January 1975, the Altair 8800 was the machine for which Bill Gates would write Microsoft’s first product, a BASIC language interpreter.

The later availability of the 6800 in production meant that the first 6800-based personal computers trailed the Altair 8800, but they still made an impact. Leading the way was the South West Technical Products SWTPC 6800, launched at the end of 1975\. The SWTPC provided a 6800 card plugged into a printed circuit board ‘backpane’ and connected to a range of peripheral cards via the SWTPC’s SS-50 bus. Cards to support floppy disk drives, video displays and other peripherals were soon available.

The Altair and the SWTPC machines may seem primitive now, but their low cost - the SWTPC 6800 was launched with a starting price of $450 - their relative ease of use, and the flexibility of being able to add new peripherals, were revolutionary. Motorola may have been a little late in delivering the 6800 but at the end of 1975, they had a promising foothold in the new personal computer market.

Meanwhile, Peddle and seven of the Motorola engineers had travelled across the country to Valley Forge in Pennsylvania, arriving at MOS Technology’s offices on 19 August 1974\. Their arrival wasn't welcomed by all at their new employer and Mensch has recalled how he once found nails in the tyres of his car parked outside the office, which he attributed to a resentful MOS Technology engineer. One of those engineers has recalled that he thought Peddle could be ‘a bit pompous’ but that ‘he’d had a vision and was pushing that vision. Peddle was the visionary’.

Just a year after they had made the move to Pennsylvania, Peddle, Orgill, Mensch and their colleagues had created several new low-cost 8-bit processors. The key designs were the MOS Technology 6501 and 6502, with the 6503, 6504, 6505 and 6507 as cheaper versions in 28-pin (rather than 40 pin) packages with reduced address spaces.

To create these new microprocessors the engineers had needed to work exceptionally long hours, with cots installed in some of the offices so that they could take naps before returning to work. The microprocessor’s circuits were drawn on paper by hand in a repetitive and error prone process. When the team discovered that one of their designs was too big they had to rework it, circuit by circuit, reclaiming small amounts of space on the die.

There have been competing claims about who should get the credit for the layout of the 6502\. It’s often been attributed to Bill Mensch, and Chuck Peddle has paid tribute to Mensch’s work:

> “He built seven different chips without ever having an error, almost all done by hand. When I tell people that, they don't believe me, but it's true. This guy is a unique person. He is the best layout guy in the world.”

However another member of the team, Harry Bawcom, has disputed this claim

> “Bill Mensch did not layout the 6502, I did, with the help of two others.
> 
> In truth we all worked as a team but Bill didn't draw a single line on the first version of the 6502\. My initials were on the die. … There were two other circuit designers involved, one senior to Bill. They deserve credit, too.”

Equally important, although less well publicised, was the effort of the MOS Technology process engineers to develop a working NMOS process and to move away from contact lithography. As Paivinen had promised, they had a working NMOS process ready in time to make the new microprocessor designs.

One of the key features of the 6501 was that it could be plugged into the same socket on the printed circuit board as the 6800\. According to one colleague:

> Chuck sold the concept that Motorola will be out there building all these boards that will use the 6800 and we’re going to come along with. a cost reduced microprocessor to replace it pin for pin.

And customers could immediately use the hardware ecosystem that Motorola had already designed for the 6800 as the starting point for a 6501-based system, ‘jump starting’ their 6501/2 development.

The 6502 would run the same code as the 6501 but was not ‘pin-compatible’ with the 6800\. However, it was simpler to integrate into systems, needing only a single clock signal, compared to the two clocks that were needed for both the 6501 and 6800.

If we put the dies (dice) of the 6800 and the 6502 side by side then the similarities between the two designs become clear. Both designs have a ‘Programmable Logic Array’ ‘control ROM’ at the top, control logic in the middle and registers and arithmetic and logic circuitry in the bottom half of the die.

Separated at birth? 6800 vs 6502 die shots

Under the surface, though, the two designs were very different. The 6501/2 were each smaller, needing just 3,500 transistors, compared to 4,100 for the 6800.

The 6501/2 would both be fabricated using an NMOS, depletion-load process whereas the 6800 had been designed to use a less advanced process. And MOS Technology pioneered the move away from using contact lithography, removing the need to create expensive new masks on a regular basis.

And the 6501 and 6502 would not be code-compatible with the 6800\. Economies had needed to be made in the new designs to keep the transistor count down. The 16-bit ‘stack pointer’ in the 6800 was reduced to 8-bits, which would always point into page 1 (hexadecimal addresses 0100 to 01FF) in memory. The 6800’s two 8-bit accumulators, ‘A’ and ‘B’, were replaced by a single ‘A’ accumulator. Room was found, though, for two 8-bit index registers ‘X’ and ‘Y’, replacing the single 16-bit index register in the 6800.

The team found a number of other ways of simplifying the designs. The first version of the 6800 had measured 29.0 mm^2\. The 6502 was only 16.6mm^2\. So almost twice as many 6502s could be extracted from a silicon wafer when compared to the 6800\. Crucially too, the smaller die size and the move away from contact lithography meant that yields, which had been a major problem for Motorola, would be significantly higher too. According to Mensch their design “yielded 10 times as many good chips as the competition.” The 6502 would be much, much cheaper to make than the 6800.

This lower cost had been achieved without sacrificing performance. Let’s look at one key example of how this was achieved.

The static RAM cells needed to create registers on the CPU were expensive in terms of die area. This meant that the teams designing both the 6800 and the 6502, were both restricted in the number of registers they could afford to include. As a result, both chips would need to load and store values from memory more often than a competitor like the Intel 8080, which had many more registers. So, both the 6800 and the 6502 had two-byte instructions that would store the value in an accumulator to a specified location in page zero (hexadecimal addresses 0000 to 00FF) in memory. For both processors, this would execute in three clock cycles.

But the 6502 went much further in making use of page zero. It added instructions that would add two bytes from page zero to the Y register to form the address to be used in a load, store, arithmetic or logic operations. These instructions made some common operations such as copying small amounts of memory, faster than the 6800\. Let’s look at one example of this in practice.

Wikipedia has an example of code with the 6502’s new addressing modes in action in a ‘memory copy’ subroutine (actually this does more than copy, it converts a string to lowercase, but the principle is the same). In the extract below the contents of SRC and DST, which are both memory locations in Page Zero, are used as ‘base addresses’ and the Y register is added in each case to create the final source and destination addresses.

So, more specifically, here SRC is #80\. The instruction:

```
LDA (SRC), Y
```

Takes the contents of #80 and #81 creates a two byte memory address from the contents of these (page zero) memory locations, adds the Y register to this address to create a new memory address and then loads the contents of that memory address into the accumulator.

All this happens, typically, in just five ‘clock cycles’ an overhead of just one clock cycle when compared to reading the accumulator from a fixed two byte address in memory. Even better, other instructions offer arithmetic and logical operations that use the same addressing mode for no extra clock cycles when compared to a simple ‘load’. These instructions read two bytes from memory, add Y to those two bytes to form a memory address, then read a further byte from memory and perform an arithmetic operation using that byte. All this can happen in just five clock cycles.

Some commentators have described the 6502 as a sort of early 8-bit RISC ‘Reduced Instruction Set Computer’ processor. In fact, in many respects, the 6502 was quite the opposite of a RISC design. Some common instructions were quite complex, with multiple memory reads and calculations.

Quoting Bill Mensch:

> And we figured out how to have addressable registers by using zero page [the first 256 bytes in RAM]. So you can have one byte for the op code and one byte for the address, and [the code is compact and fast]. There are limitations, but compared to other processors, zero page was a big deal.

In effect the 6502’s ‘page zero’ in memory provides a large number of ‘pseudo’ 8 and 16-bit registers, slower than real registers, but still faster and more versatile than having to use other parts of memory. A skilled programmer making effective use of ‘page zero’ could write code that was fast when compared to other 8-bit designs.

This probably wasn’t obvious to many early users though, who might simply have compared the number of registers with a 6800 or an Intel 8080 and found the 6502 lacking. So an early MOS Technology brochure felt the need to explain the decision to reduce the number of registers when compared to the 6800.

> **Comparing your part to the M6800, why did you cut the stack back to one page and why did you cut out accumulator B, etc.?**
> 
> Although the MCS6501 is a direct plug replacement for the Motorola 6800, no real attempt was made to maintain exact software compatibility with the 6800 because of our desire to have significantly greater addressing flexibility and to allow for upward expansion in the product family. Our primary objective was to develop a low-cost processor-and all other decisions were made on that basis.
> 
> We think that you will find that the use of the second index register more than compensates for the loss of the second accumulator and the eight-bit stack pointer helped us to reduce the cost of the chip. Because of the significantly greater addressing power, one does not need to use the stack for other than hardware and subroutine processing. As a result, a one-page long stack should be more than adequate.

The same brochure proclaimed the performance of the new designs:

> **What is the performance of the MCS6501?**
> 
> The claim that the MCS6501 beats all other competitive eight-bit microprocessors is substantiated in two areas.
> 
> First, utilizing the A H Systems benchmarks (the only set of benchmarks currently available from an independent consultant), we outperform all other standard products indicated in all but one case. Second, although recently several of our competitors have offered premium higher frequency products that may allow them to equal or outperform our processors, we will be announcing higher-speed microprocessors that will maintain or improve our current performance edge.

**With lower-priced and faster designs, Peddle had what he wanted to fulfil his vision. He would make sure that the world knew about it.**

Early MOS Technology Microprocessor Datasheet featuring the short-lived 6501

The 6501 and 6502 launched at the end of 1975 with prices of $20 and $25, much cheaper than the, by then, reduced price of $175 that Motorola was charging for the 6800.

With faster and lower-priced designs, Peddle had what he wanted to fulfil his vision. He would make sure that the world knew about it.

Early advertisements for the 6501 and 6502 proclaimed their low cost and the easy-to-follow instruction set. Peddle kept costs down by charging $10 for the 6502’s documentation, but encouraged users to make their own copies.

Actual 6502’s were made available for sale at the WESCON computer conference in September 1975\. Forbidden by the conference organisers from selling the chips on the conference floor, Peddle set himself up in a hotel room across the road. As visitors crowded the MOS Technology booth at the show, they were directed to the hotel if they wanted to buy a CPU. A steady stream of conference delegates crossed the road and made their way to the hotel room to hand over their dollars in return for one of the new CPUs.

In the hotel room Peddle’s wife Shirley had a glass jar full of 6502’s and would extract one to give to anyone who handed over their $25\. However, there was an element of subterfuge in the use of the glass jars. Peddle wanted to indicate to visitors that they had a plentiful supply of the new chips. But according to Peddle:

> Only half of the jar worked… we knew that the ones at the bottom of the jar didn't work, but it didn't matter. We had to make the jar look full.

Word spread quickly beyond WESCON. Within a few days, Peddle had signed up Atari to buy a cost reduced version of the 6502, the 6507, together with RAM, ROM and I/O chips for use in a video game console. The price? An astonishing $12 for each chipset. The console would be the groundbreaking Atari 2600, which went on to sell 30 million units.

The third edition of Byte magazine, dated November 1975, featured the new MOS Technology CPU on its front cover with the headline “A $20 Microprocessor”.

The article itself, which must have infuriated Motorola, was titled ‘Son of Motorola (or, the $20 CPU Chip)”. It compared the 6501/2 with the 6800 in some detail [See the notes below for the full Byte article]:

> For the purpose of accessing elements of arrays, or tables of many identical elements, the MOS Technology chip comes out way ahead. This is partly due to the lack of certain critical instructions on the Motorola 6800, such as an instruction to add the contents of an accumulator to the index register, or even to transfer the value in the accumulators to the index register.

And the piece concluded with:

> In favor of the 6500 series are price and speed; in favor of the 6800 are availability and very good Motorola documentation.

The 6502 would soon be available in volume and MOS Technology’s 6502 documentation was actually quite good.

Motorola had to respond. They reduced the price of the 6800, from $175 to $69\. And, fulfilling L.J. Sevin’s prophecy, they sued MOS Technology. Microcomputer Digest reported in December 1975 that:

> Motorola is seeking an injunction against MOS Technology to halt the manufacture, marketing and filling of orders for MCS 6500 microprocessor products. The injunction action is intended to stop MOS Technology from further 6500 activities until the outcome of a pending trial of a suit filed in Federal Court in Philadelphia PA by Motorola. As of yet, the injunction attempts have been unsuccessful. Motorola, citing several Motorola patents that led to the development of its own MC6800 microprocessor, alleges that seven former employees of Motorola (Charles J. Peddle, Rodney H. Orgill, William D. Mensch, Wilbur L. Mattys, Terry N. Holdt, Ernie B. Hirt, and Harry E. Bawcom) left Motorola and joined MOS Technology in similar posts and helped establish that firm's line of MCS6500 microprocessors. The suit seeks triple damages plus all profits MOS Technology has made on the 6500 product line. MOS Technology has denied the allegations and stated that Motorola's claims are unfounded.

In reality, although the MOS Technology team had used their experience with the 6800 when designing the 6501/2, there was very little technology used in the new designs that Motorola could claim as their own. The fabrication process and the instruction sets were both fundamentally different. The only significant thing that they shared with the 6800 was the pin layout of the 6501.

The financial and legal resources available to Motorola, though, dwarfed those of the much smaller MOS Technology, which was running out of cash. So MOS Technology settled, dropped the 6501, paid $200,000 in damages and returned some documentation that the engineers had taken with them from Motorola. The two companies also cross licensed patents.

There is a difference of opinion in the MOS Technology team about their original intentions around the 6501\. Rod Orgill, who led its design, bet Mensch, who led the 6502’s design, that the 6501 would outsell the 6502\. Peddle maintained that it was always intended to give Motorola a target to aim for which could be given up when they sued. In any event, the 6501 was designed, built and featured in early documentation and advertisements, but was never sold to customers.

The 6502 remained on sale though. By the end of 1975, MOS Technology had a new and more financially stable owner in the shape of Commodore. With the lawsuit settled and MOS Technology under new ownership, it was soon clear that Motorola’s management had, with the move to Austin, stimulated the creation of a new and credible competitor.

The combination of low price and better performance was a winning formula for the 6502, and it soon found its way into designs that might otherwise have used the 6800\. The most famous example of this happened in 1975 when Steve Wozniak wanted to build his own single-board computer. Having compared the Intel 8080 and the 6800, he settled on the Motorola chip, preferring its more elegant instruction set, and worked to develop his design using the chip.

But even the reduced $175 cost of the 6800 was more than Wozniak could manage. When the 6502 appeared at the end of 1975 its $25 price was, at last, affordable so he switched to the MOS Technology design. Wozniak teamed up with Steve Jobs, turned his single board design into a commercial product, and sold it as the Apple computer. The design would retain the ability to use a 6800, though, with space allocated on the circuit board for the Motorola design to be used in place of the 6502\. However, none of the computers, later known as the Apple I, were ever adapted to use the 6800.

When Wozniak and Jobs created the Apple II, their experience using the 6502 in the Apple I, made it certain that they would use the MOS Technology design.

And now that MOS Technology was owned by Commodore, variants of the 6502 became the processor in a range of popular Commodore computer designs from the PET and VIC 20 to the Commodore 64\. Ownership by Commodore proved to be no barrier to adoption by Apple or other Commodore competitors and the 6502 would go on to be used in personal computers made by Atari, Acorn and many others.

Apple 1 PCB - Note the space for a 6800 top left - By Achim Baqué - https://www.apple1registry.com/en/press.html, CC BY-SA 4.0, https://commons.wikimedia.org/w/index.php?curid=109364693

For more on the 6502, then ‘65 Reasons to Celebrate the 6502’ has lots more on the history of the chip at the heart of the Apple II, Commodore 64 and many more important machines.

* * *

The parallels between the stories of the two most popular microprocessor architectures of the 8-bit era are striking. Intel and Motorola both produced innovative designs that helped to create the early personal computer market. Both firms then drove away the teams that had created them, only to see those teams create better designs, at Zilog and MOS Technology respectively, that would go on to lead the 8-bit market.

Bob Galvin, Motorola’s CEO, immediately understood the significance and the potential of the 6800\. Motorola’s management were told that the move to Austin would be highly disruptive. Yet they persisted with the move, even after they had lost several key members of the 6800 team.

Motorola’s mistake led to the creation, at MOS Technology, of the microprocessor that would power many of the most popular early home and personal computers, including the Apple II, Commodore PET, Commodore 64 and BBC Micro. The 6502 was both powerful and elegant. Quoting Bill Mensch:

> There is a love for this little processor that's undeniable.

Chuck Peddle would later reminisce, fondly, about his experience developing the 6502.

> It was a unique time in history, you only get to do one of those in your lifetime, I think.

* * *

It was a different story for Motorola. The move to Texas and the creation of the 6502 had been major setbacks for the 6800\. They weren’t the end of the story for Motorola’s 8-bit efforts, though. In Part 3 of the story of the 6800, we’ll look at the later variants of the 6800 and the 6809, Motorola’s final 8-bit successor to the 6800 and, arguably, the most powerful 8-bit processor of the 1970s.

* * *

After the break, further reading on the development the 6502 by members of the former Motorola team, with interviews, oral histories and more.