<!--yml
category: 未分类
date: 2024-05-27 15:16:40
-->

# When The "R" Goes Missing From R&D - The Mad Ned Memo

> 来源：[https://madned.substack.com/p/when-the-r-goes-missing-from-r-and](https://madned.substack.com/p/when-the-r-goes-missing-from-r-and)

Credit: Bob Cuthill via Getty Images

I had my suspicions that something was terribly wrong with how our development team was organized, but I didn’t really put it all together until the day the Patent Attorney showed up. We had been working on a new product for a couple of years, a very large beast of a program that was used to analyze simulations of computer chips and provide useful information to the people verifying their correctness. In short, fertile ground for possible patentable ideas.

The company encouraged engineers to seek patents on work we did, and would send out someone from our legal department on a regular basis to solicit ideas for patents. This time though, we were coming up blank — even though the product we all worked on was super innovative and complex. The development team had been brought together to brainstorm with the patent person on possibilities, but everyone was kind of in a bad mood about it. She encouraged us to just start throwing out ideas, even if we thought they were not patentable ones. Silence. People just sat quietly, arms crossed.

The patent attorney shifted uncomfortably in her chair. *Isn’t there some unique analysis feature we had worked on that we could look into patenting?* she prompted.

No, we would sadly reply, the analysis logic for our product was handled by a different group. We were working lately just on a graphical user interface component for it.

*That’s OK*, she reassured. *User interface designs can also be patented, especially if they solve a problem in a unique way*.

And it was certainly true that the component we were building fit that description and did in fact have a lot of unique, probably patentable, ideas behind it. The problem was, even though we were building the component, we did not design it. Our software development group had turned into an implementation factory, churning out features invented by outsiders, but without much input from us on the actual design.

We referred the patent attorney to the other group that had designed the product interface, and she left our meeting without any ideas from our team to work on. It was not unusual for a brainstorming meeting like this to ultimately lead to nothing patented, but what was unusual was to hold a meeting where no one could think of a single good idea to even write down. It was awkward and everyone felt bad after the meeting. And I came away from that determined to try to do something about it.

Credit: Teera Konakan via Getty Images

Trouble had been brewing for a while on this team. A management organizational shift had been made months back, in response to a perceived problem with our product’s current user interface design. Customers had complained about the usability, and to be honest, there were indeed issues there. For most of the development life of this product, software engineers without any UX design experience were building all the user interfaces.

With predictable results. Clunky user flows, obscure, hard-to-use controls, inconsistent use methodology, and so on. We were lucky in one sense that our customers were primarily also engineers, so there may have been some nerdy synergy regarding the very function-over-form design. But the complaints about the quality of the UI eventually reached upper management, who decided to do something about it.

They budgeted for hiring some UX design experts to take over the interface design of the product, which seemed like a good move to me when we first heard it. I initially welcomed having some people on the team who were skilled in interface design, but I soon learned to hate it. The problem turned out not to be with who they hired, but where.

Instead of adding the new designers to work as part of the development team, they were hired into a separate organization I will call the “Applications Group”.

The Applications Group consisted of engineers who interacted more directly with customers, working on technical problems but not directly developing products. It was part of an entirely separate division of the company from R&D, and there were at least three levels of management one had to traverse upward before reaching a common point between this group and our R&D development team.

A few weeks later we were called to a meeting with the new UX team, where they presented a pretty expansive redesign of our existing user interface, incorporating many new features and design changes. We had a lot of questions and feedback on what was proposed, because all of it had been created without any R&D input, and many things seemed difficult to implement or inadviseable for various reasons.

Like predictable engineers, we started providing feedback during the presentation. After several interruptions, the Applications Team presenter seemed visibly annoyed, and asked that we withhold all questions to the end so he could finish the presentation. This is a common thing for a presenter of something to ask, and it is usually only polite to do so. But the R&D staff attending this presentation felt they were being brushed off, and doubly so when at the end of the presentation, the presenter still refused to take questions or discuss the proposal.

*“This is preliminary,”* he said. “*We are working on more detailed specifications, and you will have a chance to review them when they are finished*”.

Nothing illustrated the problem we were having here better than that statement. It suggested that software engineers writing the code should only be permitted to review a design after it has been created, rather than to collaborate on its creation, to begin with.

Credit: Sabina Torres via Getty Images

As a senior member of the R&D staff, I felt it was kind of my job to cause trouble about this, on our team’s behalf. I met with the lead UX designer from the Applications Team, and pointed out to him that one’s ability to affect change once an idea has reached the review stage is severely diminished, compared to what can be done if that person is allowed to participate in the original design discussion.

He didn’t seem to feel there was a fundamental difference, or if he did, did not want to acknowledge it. The general attitude from him and the people in his organization was that the previous R&D-led design was bad, and they had been given the mandate to take charge of the product and fix it.

Some of that was true. But their version of things seemed to preclude any kind of collaboration or input from R&D during the design phase. The “Research” of “Research and Development” evokes images of people in lab coats with test tubes. In software development terms though, the “R” would include the exploration, prototyping, and design work. And that “R” had suddenly been removed from our job descriptions.

Various attempts of mine to convince the UX team to meet with us were rebuffed. I eventually took it to my manager and above, because our group even within the R&D organization was somewhat of a remote group, and it seemed possible to me that our main R&D group was not experiencing this complete takeover of things in as quite a severe manner as our remote team had.

And indeed, our second-level management was surprised to hear how siloed things had become, and offered to speak to the manager of the UX design person and require him to have meeting with us. While I appreciated the gesture, I told them that it probably was not going to be enough. I likened the situation to giving a cat a bath.

Cats don’t like taking a bath. You can force them to, but they will hate it, and try to spend the minimum amount of time they can in the water. Then afterward, they’ll hold it against you. This was the situation with compelling the UX team to attend our meetings. That particular analogy of mine resonated with a lot of people locally and in our remote management, and thereafter the idea of forcing people to collaborate with you was referred to in these cat-bathing terms.

Credit: George Pachantouris via Getty Images

Things kind of only got worse from there. In spite of our immediate R&D management agreeing with our assessment of things, they were pretty powerless to fix it, because the organizational siloing was very deep (or should it be tall?) The upper-level management we shared probably was in favor of letting the Applications Group drive the R&D design of things, given the very abstract level they worked at.

The people in the Applications Group took the mandate quite literally though. As the Agile “Product Owners” of our development Sprint team, they had also assumed the responsibility for determining what would be worked on, and in what order. Normally I would say it is kind of optimal to have someone like an Applications Group person who is close to the customer setting the priority for things. But in this case, it was coupled with all the other roles the team had assumed from our development group.

As a result, almost all actual decision-making of any sort had been removed from the team, including what would be done and when. Observing how things were going as a senior (and perhaps a bit cynical) engineer, the thought did cross my mind that this was an attempt to create a development group where no senior R&D people are needed. After all, if there are no decisions to make, and everything is already designed, it does not require a more expensive older engineer just to write code to spec.

The trouble with that theory was, the junior engineers on the team hated the organizational situation just as much, if not more, than I did. They complained a lot about it, and espoused even more radical ideas about what management intended, and where things would end for our group. They had no faith that they were working somewhere that valued their input or where there was a future for growth, and I was kind of surprised no one left somewhere in here.

At the lowest point, I was getting in arguments with the person prioritizing work because they objected to me scheduling R&D-centric tasks like refactoring, infrastructure work, and so on without first getting permission from him. I angrily reminded this guy that they were in charge of the Product priorities, not the R&D team, but they did not see any difference I suppose.

Also on at least one occasion towards the end of all this, there was a product design meeting attended by Applications Team and UX people from various remote sites, and they chose our building to have the meeting due to its central location. But no R&D people from the building they met in were invited to these meetings, which were about the very thing R&D had been asked to design.

When I called them out on it, I was told R&D wasn’t invited because it would be too big a team and be “Design By Committee”. I said it was instead “Design in an Ivory Tower”. No compromise was ever made. We had reached a zero-trust state, which is of course not a sustainable one for any organization.

Sounds super-crazy, even now to me. The good news is, the company did not go out of business, the product didn’t get canceled, and nobody lost their job. Well, almost nobody. A few of the more arrogant players in this story either chose to leave the company, or were encouraged to choose to leave, after things came to a head and an inevitable second reorg happened. And I would be lying if I said I was sad about it.

Things coming to a head happened because work was just not getting done on time. Part of this could be attributed to extravagant and impractical UX designs getting too far along before they were determined to be unimplementable. Something that could be traced back again to a lack of R&D and UX collaboration.

But a larger part of it was that people in the development team were just showing up to work, and not much else. I had a friend once at [Digital](https://en.wikipedia.org/wiki/Digital_Equipment_Corporation) who gave me this unforgettable advice, right after we were bought by [Compaq](https://en.wikipedia.org/wiki/Compaq):

*“When captured by the enemy, it is best to display model prisoner behavior.”*

And that was exactly what had happened here. It wasn’t that people were deliberately trying to sabotage progress, they were showing up to work and doing their jobs as instructed. But nothing more.

I like to use the marketing term [“Mindshare”](https://www.investopedia.com/terms/m/mindshare.asp) to describe what you have when your employees feel empowered and are truly bought into what they are working on. Amazing things can happen when you have a team’s Mindshare, but in this case, it was nowhere to be found.

The reorg that followed disbanded the Application group, returned the UX function back into direct R&D management’s control, and integrated designers with software engineers. It is a bit [Deus Ex Machina](https://en.wikipedia.org/wiki/Deus_ex_machina) of an ending to the story, but I’ll take it. Things rapidly improved after that, and so they lived happily ever after.

I know what you may think - this was a biased opinion in favor of R&D, and there is probably a UX team side of the story. Certainly true. But I would say my strongest bias is not about favoring R&D over some other team.

My bias is about working collaboratively, instead of in separate groups that due to their organizational distance, create opportunities for conflict and mistrust. Doesn’t matter if that organization ends up being called “R&D”, or something else. Hell, we can call it Design and Development or something like that. The nerd in me would be happy working in the D&D group!

* * *

> **Next Time**: The experience of moving to a company with 5600X less people than the one you were at. Small company adjustments and some pleasant surprises too as we talk serious downsizing in : When You *End up at a Startup*

* * *

*The [Mad Ned Memo](https://madned.substack.com/) covers topics in computer engineering and technology, spanning the past forty or so years. Get your weekly dose of nerdy computer tales and discussions delivered right to your inbox, and never miss an issue! This newsletter comes to you ad-free and cost-free, and you can unsubscribe at any time.*

`The Mad Ned Memo takes subscriber privacy seriously and does not share email or other personal information with third parties. For more information,` [`click here`](https://madned.substack.com/about)`.`