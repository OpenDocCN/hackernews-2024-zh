<!--yml
category: 未分类
date: 2024-05-29 12:44:34
-->

# How a group of Berkeley researchers took over the chip industry

> 来源：[https://boldandopen.substack.com/p/how-a-group-of-berkeley-researchers](https://boldandopen.substack.com/p/how-a-group-of-berkeley-researchers)

*👋 Hey, it’s [Jaime](https://www.linkedin.com/in/jaimearredondo/). Welcome to my weekly newsletter where I share* how thriving open source projects grow their communities.

If you’re not a subscriber yet, here’s what you have missed from past newsletters:

1.  [The Cycles of Bundling and Unbundling to Unleash the Public Sector's Creativity](https://boldandopen.substack.com/p/the-cycles-of-bundling-and-unbundling)

2.  [How Wikipedia got volunteers to create 55 million articles](https://boldandopen.substack.com/p/how-wikipedia-got-volunteers-to-create)

3.  [Why inviting others to steal your ideas can propel you forward](https://boldandopen.substack.com/p/why-piracy-can-be-great-for-you)

*Subscribe to get access to these posts, and all future posts:*

In this week’s newsletter, I share a rather long, deep dive into RISC-V and how a bunch of university researchers broke down the monopolies around processors by offering an open alternative. You’ll learn how to offer an open alternative technology, build a business around it, and also how it’s possible to build a community with the competition.

**Read time**: 23-ish minutes (Due to the length of this piece, [you may need to click here](https://open.substack.com/pub/boldandopen/p/how-a-group-of-berkeley-researchers?r=1boy3&utm_campaign=post&utm_medium=web&showWelcomeOnShare=true) to read the whole thing in your email.)

* * *

Up until 2015, the chip industry was controlled by three players: Intel, ARM, and AMD. But now everybody is quickly moving elsewhere.

What changed? RISC-V released an open source instruction set architecture for developing chips. And the plan worked.

Here’s the story of how RISC-V has grown into an industrial ecosystem capable of challenging its historical players.

(Since this is a pretty long article, if you’re in a hurry, you can get the summary of takeaways at the bottom.)

The RISC-V story traces back to March 2010 at UC Berkeley. 

Krste Asanović originally pitched the idea as a “short, three-month project” to develop the [RISC-V instruction set](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2011/EECS-2011-62.pdf) as a teaching tool for chip design to allow programmers and their software to control computer hardware. He started it with graduate students Yunsup Lee and Andrew Waterman.

Traditionally, students would have to use proprietary CPUs (central processing units), which were too complicated for what they wanted to do, opaque to learn from, and their closed intellectual property made it challenging to share research with others.

They also felt that a lot of commercial products were not that good and that they could do a lot better.

So Asanovic, Lee and Waterman started RISC-V as a new open standard for computer chips. Open RISC-V standards mean you, me, Intel, or anyone else—even competitors—can participate in their standards or design computer chips based on RISC-V instruction sets.

RISC-V is an open-chip architecture that is free to licence. Customers can add extensions and customise the chip for several applications, including cloud, artificial intelligence, mobile, automotive, the internet of things, and other industrial applications.

Previously, if a company needed chips for simple devices, it would buy them off the shelf, but since these chips would have unnecessary features, they would be slow or waste energy.

And if a company wanted to design a chip for more complex hardware or software, it would have to pay millions of dollars in licences to Intel, ARM, or IBM to access their instruction sets for a single design. 

Proprietary licences meant only massive players with deep pockets could try new chip innovations.

So when they released the instruction set to the public, the response from academics, institutions, and companies like Google, IBM, or even Intel became overwhelmingly big for Berkeley, forcing them to start the RISC-V foundation in 2015. 

The foundation's goal is to maintain the RISC-V ISA standard and has grown to over 60 companies, including big players like Qualcomm, Samsung, Alibaba, Meta, Nvidia, Microsoft, Intel, Western Digital, IBM, and Google.

What began as a 3-month university project now ships millions of cores a year.

RISC-V made gradual inroads into the maker sector—slowly at first but gaining momentum with each passing year.

We can now find RISC-V implementations in commercial products, including smartwatches, fitness bands, storage products, and graphics cards, where the allure of true freedom—plus significant savings on licence fees—has won out against the desire to keep proprietary IP suppliers on-side.

The open-source flexibility of RISC-V has made it an [increasingly popular chip architecture](https://spectrum.ieee.org/semiconductors/devices/riscvs-opensource-architecture-shakes-up-chip-design) for companies such as computing storage giants Seagate, Western Digital Corp., and China’s e-commerce giant Alibaba, along with government initiatives backed by the US military’s [Defense Advanced Research Projects Agency](https://spectrum.ieee.org/tech-talk/computing/embedded-systems/darpa-hacks-its-secure-hardware-fends-off-most-attacks) (DARPA).

**Takeaway:**

*   **If an industry doesn't have open standards, creating one can enable innovation** by involving new actors who couldn't participate before. Public funders, academic researchers, and private companies are best positioned to share the standards and tools they use or develop.

Until RISC-V came around, similar instruction sets existed mainly in academic and research settings. 

As Krste Asanovic argued in a 2014 [research paper](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2014/EECS-2014-146.pdf), the main benefits of an open instruction set would be to reduce costs on the software end and spur hardware innovation beyond what any closed or licenced instruction set ever could.

“Given that the industry has been revolutionised by open standards and open source software—with networking protocols like TCP/IP and operating systems (OS) like Linux—why is one of the most important interfaces proprietary?” Asanovic argued. “While instruction-set architectures (ISAs) may be proprietary for historical or business reasons, there is no good technical reason for the lack of free, open ISAs.”

In simpler words, this means that even if an industry might be protected by intellectual property, its players and customers will always be better served if the industry's standards (software, designs, protocols, methods, etc.) become open for everyone to participate and contribute.

Usually, proprietary technology exists for business or historical reasons, but there are no technical justifications for the lack of open standards. 

Closed licences have significant problems that slow the diffusion of innovation and new entrants:

*   **Excessive friction:** It prevents others from using them without licences, and for those who want to pay the licence, negotiations usually take 6–24 months and can cost between $1M and $10M, which rules out academia and startups or SMEs with small volumes.

*   **No new designs**: When you get a licence, you can’t design or adapt a new version, you just get to use the design you got.

*   **No future-proofing: If the licencing company dies, it takes its knowledge and designs with it, making it very hard or impossible to maintain its technology.**

Even if licences make business sense, they stifle competition and innovation by stopping many from designing and sharing new ideas and even teaching what they learn along the way.

70% of people who contribute to RISC-V are people who have never licenced anything from Intel, ARM, or any other chip player before. And the same applies to any other industry. 

70% of people who will use new open source innovations will be people who are currently not buying anything from that industry anyway, like academia, SMEs, or startups, but could become creators, researchers, and promoters.

*Opening an innovation removes the friction to innovate for those who can’t afford the commercial licences or the 6 to 24 months it takes to negotiate a licence.*

And the counterintuitive idea here is that it’s also in the incumbents’ best interest to open their innovations first.

If they don’t open their standards first because they want to keep their control over them, when others create an open alternative, as RISC-V did to Intel, ARM, or AMD, they will take the industry away from them.

And this is bound to happen in any industry sooner or later. If something really matters to your competition, they will figure out your secret recipe.

When those who can’t play become too limited by the proprietary solutions available, they will either create their own open alternatives or improve existing ones.

And once they figure it out, those who succeed do so based on all the things that aren’t a secret, like building a community ecosystem that creates a virtuous cycle of innovation. 

So, as Seth Godin [says](https://seths.blog/2023/01/your-secret-recipe/), time spent securing the secret in a vault is time wasted. Even if you had made a considerable advance, the collaboration of a committed community could quickly take over any proprietary option.

Jeff Bezos said, “Your margin is my opportunity.”

But RISC-V creators could have said, “Your IP is my opportunity.”

If you want to create a new opportunity in a market controlled by a few players, make an open source alternative to its closed IP and share it with other partners.

There is a massive opportunity for publicly funded research labs and universities to create such opportunities, not only in tech but also in pharma or other IP-heavy industries.

It sounds too good to be true, but for states that want to create economic activity and jobs, it's a way to make "free money" and capacity by opening up this untapped potential.

*Opening an innovation also enables the market to generate new revenues that were not previously possible* 

The first to release easy-to-use and effective open standards in industries with heavy IP protection will enable many new opportunities for themselves and others.

**Takeaways: **

*   **Opening a previously closed product creates the opportunity for 70% of the market's smaller players who couldn’t afford to get in** or don’t have volumes big enough to be taken care of.

*   **The more the market scales and fragments itself, the more an open source philosophy enables customers to support themselves.**

*   **If a technology or idea is essential for others, an open alternative will eventually happen, making all proprietary solutions obsolete or marginal.** So it's in the best interest of any company to lead the way and be the one to start the collaboration with its industry.

There are two key ingredients: funding and licencing. 

A new industry's seeds generally come from basic research. But research is risky since the chances of new research finding a commercial use can be pretty low. That's why venture capitalists, banks, and other private investors don't like participating in this phase and why high-risk research usually depends on public sector funding. 

In the graph below, you can see the different funding stages, from the research of an idea to large-scale deployment.

*Source: Inspired by The Entrepreneurial State, by Mariana Mazzucato*

To develop the RISC-V protocol, Berkeley [got funding](https://riscv.org/about/history/) from the public sector—through DARPA and the California State—and private investors—through Intel, Microsoft, and other industrial sponsors—who gave $10 million over five years and were interested in parallel computing systems.

RISC-V doesn’t represent any new technology since it [is based on computer architecture ideas that date back at least 40 years](https://live-risc-v.pantheonsite.io/risc-v-genealogy/), so no patents were filed.

And this open particularity allowed for other implementations to bloom worldwide.

RISC-V didn’t become a massive success because it was a great new chip technology. 

There are better alternatives out there. Its power lies in the fact that it’s the first global open standard that allows anyone to freely develop their own hardware to run the software. 

This open licence model also allows organisations that want to implement or extend RISC-V in commercial products to not disclose their changes to the community at large, which makes it particularly appealing for commercial use in embedded devices as it makes paying the licencing fees for ARM designs unnecessary.

**So public funding and open licencing in basic research for citizens and public bodies are crucial.**

After the invention of RISC-V, many projects used it, including many other research programmes funded by [DARPA](https://www.darpa.mil/work-with-us/for-universities), in many places and companies. 

RISC-V’s open licence became the support infrastructure that enabled other research subjects that couldn’t previously be tested with the paying licences from Intel, ARM, or AMD.

While DARPA did not fund the original RISC-V ISA definition, DARPA’s funding played a significant role in its later development. 

DARPA is currently funding a large set of programmes around open-source hardware technology.

And the lesson for other universities and public funders is that by supporting the development of open standards, their research and investments have more chances to be adopted and benefit the economy by creating jobs and taxes and upskilling its population.

Opening access to innovation enables network effects. The more people use it, the better it works and the more valuable it becomes as more people contribute to and improve the original creation. The more someone starts manufacturing and distributing an invention without permission, the more revenues they generate, thanks to open innovation.

Open standards are also very beneficial to taxpayers, as they make it possible for anyone to innovate and create more qualitative and affordable products without the friction proprietary standards introduce.

And in industries driven by proprietary intellectual property, developing open standards can also break existing monopolies or oligopolies by enabling new actors to bloom in existing industries. Instead of getting big unicorn corporations with outsized global power, enabling open ecosystems made of hundreds of smaller companies prevents "too big to fail" organisations and makes more resilient and healthy markets.

**Takeaways:**

*   **Open standards have the most chances of emerging from high-risk research, universities, and public funding** through public grants and credits, which are the most able to support the high-risk involved. Sharing research results in open source aligns well with the state's mission of serving the common good and developing healthy markets, economic activity, and jobs.

*   **Permissive Open licences that don't force others to disclose the changes in their commercial products are a great way to increase the chances commercial companies will adopt your project** and contribute to developing and maintaining the open standard.

For the RISC-V standard to be embraced by an open-source community, the founders believed it needed a proven commercial record. 

To show the opportunity, they explained in their paper [Instruction Sets should be Free: The case for RISC-V](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2014/EECS-2014-146.pdf) how the computing industry would benefit from a viable free open standard chip protocol just as it has benefited from free open source software.

For example, it would enable a real, free, open market of processor designs, which licenced standards’ quirks prevent.

They argued the open standard could lead to:

*   **Greater innovation via free-market competition** from many more designers, including open vs. proprietary implementations.

*   **Shared open-core designs** would mean shorter time to market, lower cost from reuse, fewer errors given many more eyeballs, and transparency that would make it hard, for example, for government agencies to add secret trap doors.

*   **Processors are becoming more affordable for more devices**, helping expand the Internet of Things (IoT) by lowering their cost to as little as $1.

So the paper needed to explain why this open standard was an opportunity with the potential to build a unicorn ecosystem where every actor could benefit.

You might be familiar with unicorn startups. A unicorn startup is a privately held startup with a current valuation of US$1 billion or more.

I define unicorn ecosystems as an open twist on unicorn startups. Unicorn ecosystems are solutions that create markets able to generate $1 billion or more in revenues. But instead of doing it from a single startup, this market grows very fast through many players instead of just one, thanks to open-source and community-driven innovation. 

Here are some examples of these organisations that generate more value than they extract by enabling others to build on their ideas:

*   Linux enables 99% of internet servers and over $50 billion in business.

*   WordPress is used by 43% of internet websites (in January 2023), and its company is worth $7 billion while enabling its users to create an estimated $140 billion in revenue annually.

*   RepRap has enabled billion-dollar 3D printing companies and tens of billions in revenues for the 3D printing market.

*   RISC-V is an open-chip architecture ecosystem that's doubling every year and is [expected](https://omdia.tech.informa.com/-/media/tech/omdia/brochures/ai/risc-v-processors-report.aspx) to go from $52 million in 2018 to $1.1 billion in 2025.

Unicorn startups are perfect for private investors as they can be sold to large private companies. But selling unicorn ecosystems isn’t as easy or desirable since they come from many open source solutions.

But unicorn ecosystems are great for public funders, as their publicly funded research can later turn into markets that create new jobs, new skills, new taxes, improve product quality, and decrease costs exponentially faster than closed intellectual property or startup unicorns can.

That said, being part of such an ecosystem can allow incumbents that new open players are replacing to identify and support new fast-growing startups in which to invest. RISC-V is replacing Intel or Qualcomm, but they are turning it into an opportunity by investing in RISC-V startups like SiFive.

**Takeaways:**

*   **If you want to gather an industrial ecosystem around an open standard, you will have to show why it's viable**. It might be feasible because it will lead to increased innovation by opening the market to new players, less dependence on a few players, more customisation, lower costs, a shorter time to market, or lower prices, which will make a product more accessible to more people.

*   **By building an industrial ecosystem, public investors can create new markets that weren't possible**, increase competition, and improve quality and prices. And private companies that are becoming obsolete because of open competition can also participate and find new opportunities in this new market.

But to understand an open industry, we need to understand why its actors can also get funding and grow without exclusive IP protection.

So here’s the story of SiFive within the story of RISC-V.

SiFive is the company founded by Krste Asanovic. While researching RISC-V’s architecture, they also did lots of implementations and got many students involved in working on this project. This allowed people from outside to pick it up and do real work with it as well.

And they realised the semiconductor industry was in a perfect storm where Moore’s Law is ending and new technologies and developments are getting more and more expensive, with fewer and fewer companies able to pull off new designs and make money out of them. 

So Asanovic and his co-founders first started a consultancy activity around RISC-V, which helped them realise the opportunity in the growing demand to create custom chips for the Internet of Things. All those devices need processors—and that cannot be the same processor for all solutions, so they decided to turn their consulting activity into a company.

What’s important to understand in this market is that there will be growth in silicon products, but that growth will be in many fragmented markets.

The old semiconductor business model—having one design and selling many millions of it— wo’t work like it did for the traditional computer and mobile phone markets. The future will see perhaps hundreds of designs in lower volumes.

With SiFive, they are figuring out how this works. The traditional users of the chips are now becoming the new manufacturers. Google. Microsoft, Amazon, and many other companies will design and make their chips—not to sell to others but to use them in their products. It will allow them to add capabilities not available on standard off-the-shelf chips.

So the opportunity for SiFive lies in finding out how to help smaller companies and startups do custom silicon design and invent new products with new capabilities. 

"We believe there is a lot of untapped innovation there. But the problem is that the barrier to entering custom silicon design is too high, and those great ideas do not become a product. Solving that problem is the goal of SiFive," says Asanovic.

So SiFive's business model is to do rapid development of new chipsets and help the client get into production at very low costs. 

That can open up new perspectives for makers, startup companies, and medium-sized businesses. All the design files for the chip are on GitHub. This openness is unique in the semiconductor business.

Not needing NDAs or lawyers gives SiFive a low-cost structure, which means lower client costs.

These lower costs drive down the upfront costs of prototyping so that startups, makers, or hackers can get a first chip as cheaply as possible and try their ideas. 

Most of them will probably fail, and that's fine.

But the more people that try, the greater the quantity and quality of new ideas that will come out, increasing the chances that some of them will be successful. And for those that succeed, SiFive can scale with them and help their customers ship millions of chips per year.

That's how they make money. They bet on thousands of ideas their customers have instead of the handful they could take on themselves.

So bootstrapping and getting clients to pay for the company's development is one option. But sometimes, that's not enough to fund a company that can or needs to grow faster because demand is very high.

**But in that case, why would investors back projects that have no intellectual property protection?**

The summary is that investors are ready to invest in open source technology as long as they understand that there is growing demand for it and a company's positioning is strong enough.

From 2019 to 2022, SiFive grew by [1,477%](https://www.inc.com/profile/sifive). This growth led funders to invest up to [$365.5M](https://www.crunchbase.com/organization/sifive/company_financials).

And there are more startups building on top of RISC-V that are getting private funding, like Hex Five, Codasip, and Dover Microsystems, and others growing without funding, like Ant Micro or Andes Technology.

*A chip from Codasip, another fast-growing company in the RISC-V ecosystem*

**Takeaways:**

*   **An open ecosystem of companies can get going when there is growing demand for it** or it expands an industry that was previously closed to the contributions of smaller companies and academia due to licencing costs or patent barriers.

*   **Companies within the open ecosystem can get funding either by selling to their clients or by getting private financing to accelerate their growth by showing the growing demand** enabled by the lower costs and increased quality and customisation made possible by the access to open IP within the industry.

A common concern from leaders who want to start an open source project is that they understand the value in it, but they don’t want to be the ones managing and maintaining the whole project and ecosystem on their own.

But getting a community to contribute is very hard to do and not something you can do on-demand, as it demands lots of resources and time from contributors.

So RISC-V is an excellent example of how to get a community involved and why it needs to be done intentionally.

Let's first look at how it came to be and then at the specific steps they put in place to invite and motivate members to participate.

The RISC-V Foundation was founded in 2015 to build an open, collaborative community of software and hardware innovators based on the RISC-V standard. 

The foundation was set up as a non-profit corporation controlled by its members and directed its development to drive the initial adoption of RISC-V.

In November 2018, the RISC-V Foundation announced a collaboration with the Linux Foundation. 

As part of this collaboration, the Linux Foundation provides operational, technical, and strategic support for RISC-V International, which may include member management, accounting, training programmes, infrastructure tools, community outreach, marketing, legal, and other open standard services and expertise.

But gathering as a foundation can also protect the member's investments from geopolitical disruption, thanks to the open nature of the standard.

Across 2018–2019, the RISC-V members were hit with tensions between the US and China because of their export blacklists. This uncertainty led to concerns from around the world that investment in RISC-V had to come with IP access continuity to ensure a long-term strategic investment.

In 2020, the foundation moved to Switzerland and shifted to a more inclusive membership structure to calm these concerns of political disruption to the open collaboration model.

The RISC-V organisation currently has about one-third of its members in North America, another third in Europe, and 37 percent in Asia-Pacific. 

The latter represents the fastest-growing RISC-V adoption, with countries such as India and Pakistan having already embraced RISC-V as their national instruction set architecture for homegrown chip development.

In 2019, the original authors and owners of the RISC-V standard surrendered their rights to the foundation. Calista Redmond, who led open infrastructure projects as VP at IBM, was appointed CEO.

In 2022, the [RISC-V foundation](https://riscv.org/) was an over 1400-member-strong community made of chip makers, designers, and academia committed to building a joint roadmap for the next 50 years of computer innovation and design via open standard collaboration.

The first step is to build something many people depend on to do their work. And that’s true for RISC-V, but also for many other open source projects like Linux, WordPress, Wikimedia, and many other open programming languages.

The second step is understanding the problems and ambitions shared in common.

The RISC-V Foundation does a great job at this by explaining the benefits of joining on their “[Become a Member](https://riscv.org/membership/)” page:

For the [RISC-V members](https://riscv.org/members/), gathering around a foundation or consortium helps every member to accelerate the industry as a whole by mutualizing their efforts and resources around an open standard.

And it’s also important to create different levels of membership depending on the member's persona.

RISC-V focuses on companies, academia, non-profits, or individuals, and each comes with different benefits and membership fees depending on the number of employees and type of legal entity.

These memberships evolved along with the maturity of the community and the commercial stakes involved. 

In 2016, they had a [free waiting list](https://web.archive.org/web/20160402085748/http://riscv.org/membership-application). In 2018, they started their membership fees from free for individuals to [$25K](https://web.archive.org/web/20180907051726/https://riscv.org/membership-application/) for commercial companies, and in 2020, they multiplied the fees for big companies to [$250K](https://riscv.org/members/) and kept the free memberships for individuals and academia.

These fees allow the foundation to pay RISC-V’s [15 staff members](https://riscv.org/risc-v-staff/) (in 2023) to promote, maintain, and connect the ecosystem, the documentation, and the technical evolutions of the standard.

Members can then plug themselves into regular [technical](https://calendar.google.com/calendar/u/0/embed?src=tech.meetings@riscv.org) and [community meetings](https://calendar.google.com/calendar/u/0/embed?src=community.meetings@riscv.org), [participate in or submit events](https://riscv.org/events-all/), or get access to [learning resources](https://riscv.org/learn/) from partners and other curated sources.

Many different [working groups](https://lists.riscv.org/g/main/subgroups) exist for technical tasks, marketing, programming workshops, or other special interest groups like automotive, academia, Android, or graphics, among many more.

**And who owns and can use what’s produced by the members of the RISC-V Foundation?**

RISC-V's intellectual property is developed and contributed collectively by members. And once IP is provided globally in open source for software and hardware design, it is permanently open and remains available to all.

However, only members of RISC-V International can vote to approve changes, and only member organisations can use the [trademarked](https://en.wikipedia.org/wiki/Trademark) compatibility logo.

**Takeaways:**

*   **Consider the profiles of the people and entities who depend on the open project** and would like to contribute, and develop the roadmap of a consortium.

*   **Define why it makes sense for them to join**. What problems or ambitions do they have that the consortium would solve for them? Is it accelerating a common cause, mutualizing resources, influencing or leading the development of the open project, or gaining insights from the ecosystem?

*   **Define the membership tiers these different personas can access**. What are the different benefits, roles, and fees to access each? Who has the financial resources to contribute and would profit financially from joining? Who can’t contribute financially but would be a valuable contributor?

*   **Can you already partner with other communities doing this work? Instead of doing it all from scratch, RISC-V partnered with the Linux Foundation to find operational support.**

*   **What spaces or resources are essential for the community to work together? These can range from online forums, learning resources, working groups, events, and whatever else might make sense to keep advancing the organisation and the mutual interests of the community. Who can lead these spaces or put these resources in place? If nobody raises their hands, can you invite someone within the community who might be interested?**

RISC-V approach helped them turn an open standard into a thriving ecosystem in collaboration with its most prominent players.

It helps that they serve the fast-growing chip processor market, but still.

RISC-V has become a crucial consortium in their industry, proving that you don't need closed intellectual property to develop an industry and new startups. You need a smart and intentional plan.

Here are the key takeaways you can borrow, modify, and adapt for your organisation based on RISC-V real-life tactics:

It has the potential to enable innovations and markets by opening up the collaboration of actors who weren’t previously involved.

Then enable people to collaborate and support each other in improving and extending the open project.

This increases the chances that the open protocol will be adopted by commercial companies and gives them the incentive to develop and maintain the open standard.

What does it enable?  Does it lead to greater innovation by opening the market to new players? Less dependence on a few players? More customisation? Lower costs or a shorter time to market? Or lower prices, which make a product more accessible to more people? It will probably be a mix of these and other benefits.

Define membership levels, roles, and fees they could subscribe to depending on their commercial interests, capacities, and motivations.

Sources: