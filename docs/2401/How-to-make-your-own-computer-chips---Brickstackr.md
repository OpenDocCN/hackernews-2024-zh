<!--yml
category: 未分类
date: 2024-05-27 15:21:02
-->

# How to make your own computer chips — Brickstackr

> 来源：[https://www.brickstackr.com/posts/how-to-make-a-computer-chip](https://www.brickstackr.com/posts/how-to-make-a-computer-chip)

*Mommy, Daddy? Where do computer chips come from?*

I’m phrasing the question like this because the answers you usually get when you ask are pretty childish. “Well, Timmy, they come from CHINA and Taiwan. We need to INVADE TAIWAN before CHINA does, because it’s where the chips come from.“

Vague, politicized, and childish. You can’t build much of anything talking to people who dispense wisdom like that. You have to talk to somebody that can identify the correct task (making computer chips) and not separate side interests like hobbling China’s military ambitions or creating jobs. Which can all be done without making any computer chips at all, Starbucks creates lots of jobs and zero computer chips. Most people who just want a job have correctly identified that they are more likely to get one by putting on an apron than a bunny suit, much to the lament of policymakers trying to stimulate the development of a new semiconductor manufacturing industry through lavishing subsidies on a handful of very large corporations to work on construction projects for the next 5-10 years.

During the covid-19 shutdown, I spent a bunch of time researching how to make my own PCB’s (printed circuit boards). I didn’t end up carrying out the project, because it turns out the most popular ways of doing that require some pretty toxic chemicals. But to make the most simple analog circuits, the process seemed to be straightforward. There’s a bunch of CAD-like programs you can use to create the circuit pattern, and then you trace that pattern onto a copper plate with the chemicals. Once you have a power supply to your circuit, electrons do everything else.

Of course what I learned later was that most people skip over trying to figure out how to use chem lab equipment to manually etch copper plates and just buy the finished PCB’s from companies in China that will print 30 PCB’s at a time and ship them to you. Once you have a design, you just upload it to one of their websites and theoretically, the board you get back should work. Or if not, you tweak the design and try again.

I was surprised to find out by making a trip to Silicon Valley to attend the RISC-V Summit and asking a lot of pretty naive and uninformed questions that chip manufacturing is just the big kid version of making a PCB for a science fair project. To create something customized, you can either start with buying “IP blocks” from companies that sell access to designs for circuits they already created, or you can develop the equivalent using a hardware definition language (HDL). Then there are all kinds of complicated software test suite products to evaluate designs and make changes before the final “tapeout” is sent to a fab to be manufactured.

That was where things seemed to get a bit hazier than I might have liked. People on the crowded trade show floor were eager to talk about open source projects to make every aspect of designing a chip easier to do, but trying to talk to anybody doing manufacturing felt a little bit like I was an FBI agent sent to try and buy drugs from a cartel with a GoPro strapped to my forehead. Suddenly it was all whispers. We are NOT CHINA. Definitely, 100%, NOT. CHINA. We do “contracts” with “fabricators”. Ok.

But I think this is a thing where greater manufacturing capacity outside of China and Taiwan will follow on from opening up the software ecosystem, and not the other way around from subsidizing supply of chip fabs first. When all of the tooling to develop and test the IP was either very expensive or lacked the transparency of open source projects where you can just go on Github and find decent enough instructions to get started, there wasn’t sufficient *demand* to have manufacturing capacity available locally.

We need more people who have some independent interest in developing their own hardware IP to have a need for additional capacity. The craft beer revolution did not come from big companies or the government saying that as a matter of official policy, every town should have a couple of breweries. It came from people who wanted to know how to make their own beers mastering that skill first. Craft breweries only scaled up manufacturing later after there was already demand for the product via people trying it.

Semiconductor manufacturing hasn’t been like that yet, but there’s a bunch of promising concepts that have the potential to introduce that type of dynamic to the industry. [ChipFlow](https://www.chipflow.io) was one of the more promising ones that I learned about at the RISC-V Summit. It seems to be an adjacent commercial concept to the [Amaranth](https://github.com/amaranth-lang/amaranth) open source project, which from what I can figure out is a Python-based version of the hardware definition languages Verilog and VHDL. There are just so many more people that know Python than specialized HDL’s that it can only help to make an open hardware ecosystem more accessible. Finally [Libresilicon](https://libresilicon.com) is a project that aims to open up the actual manufacturing ecosystem to hobbyists and others that have some interest in developing their own customized hardware.

I’m excited for where all of this is going. I can’t wait to eventually develop something myself and see if I can get it made somewhere.