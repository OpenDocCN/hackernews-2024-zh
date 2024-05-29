<!--yml
category: 未分类
date: 2024-05-29 12:41:15
-->

# Why Google failed to make GPT-3 + why Multimodal Agents are the path to AGI — with David Luan of Adept

> 来源：[https://www.latent.space/p/adept](https://www.latent.space/p/adept)

*Our next SF event is [AI UX 2024](https://lu.ma/aiux) - let’s see the new frontier for UX since [last year](https://www.latent.space/p/build-ai-ux)!*

*Last call: we are recording a preview of [the AI Engineer World’s Fair](https://twitter.com/aiDotEngineer/status/1754929063993737721) with swyx and Ben Dunphy, [send any questions](mailto:ben@ai.engineer) about [Speaker CFPs](https://docs.google.com/forms/d/e/1FAIpQLScc-47zw-tWjYbhAkwTeLy_-MQW3L-3uwtaVnEzudrEZcQ7bg/viewform?usp=sf_link) and [Sponsor Guides](mailto:ben@ai.engineer) you have!*

*Alessio is **now hiring engineers** for a new startup he is incubating at Decibel: Ideal candidate is an “ex-technical co-founder type”. [Reach out to him](https://twitter.com/FanaHOVA/) for more!*

* * *

**[David Luan](https://twitter.com/jluan)** has been at the center of the modern AI revolution: he was the ~30th hire at OpenAI, he led Google's LLM efforts and co-led Google Brain, and then started Adept in 2022, one of the leading companies in the AI agents space. In today's episode, we asked David for some war stories from his time in early OpenAI (including working with [Alec Radford](https://twitter.com/swyx/status/1699369076529971545) ahead of the GPT-2 demo with Sam Altman, that resulted in [Microsoft’s initial $1b investment](https://openai.com/blog/microsoft-invests-in-and-partners-with-openai)), and how Adept is **building agents that can “do anything a human does** ***on a computer*****" — his definition of useful AGI.**

While we wanted to discuss Adept, we couldn’t talk to a former VP Eng of OpenAI and former LLM tech lead at Google Brain and not ask about the elephant in the room.

It’s often asked how Google had such a huge lead in 2017 with Vaswani et al creating the Transformer and [Noam Shazeer predicting trillion-parameter models](https://www.youtube.com/watch?v=9P_VAMyb-7k) and yet it was David’s team at OpenAI who ended up making GPT 1/2/3\.

David has some interesting answers:

> *“So I think the real story of GPT starts at Google, of course, right? Because that's where Transformers sort of came about. However, the number one shocking thing to me was that, and this is like a consequence of the way that Google is organized…what they (should) have done would be say, hey, Noam Shazeer, you're a brilliant guy. You know how to scale these things up. **Here's half of all of our TPUs. And then I think they would have destroyed us.** He clearly wanted it too…*
> 
> *You know, every day we were scaling up GPT-3, I would wake up and just be stressed. And I was stressed because, you know, you just look at the facts, right? Google has all this compute. Google has all the people who invented all of these underlying technologies. There's a guy named Noam who's really smart, who's already gone and done this talk about how he wants a trillion parameter model. And I'm just like, we're probably just doing duplicative research to what he's doing. He's got this decoder only transformer that's probably going to get there before we do.*
> 
> ***And it turned out the whole time that they just couldn't get critical mass.** So during my year where I led the Google LM effort and I was one of the brain leads, you know, it became really clear why. At the time, there was a thing called the Brain Credit Marketplace. Everyone's assigned a credit. So if you have a credit, you get to buy end chips according to supply and demand. **So if you want to go do a giant job, you had to convince like 19 or 20 of your colleagues not to do work.** And if that's how it works, it's really hard to get that bottom up critical mass to go scale these things. And the team at Google were fighting valiantly, but **we were able to beat them simply because we took big swings and we focused.”***

Human intelligence got to where it is today through evolution. Some argue that **to get to AGI, we will approximate all the “FLOPs” that went into that process**, an approach most famously mapped out by Ajeya Cotra’s [Biological Anchors report](https://www.lesswrong.com/posts/KrJfoZzpSDpnrv9va/draft-report-on-ai-timelines):

The early days of OpenAI were very reinforcement learning-driven with the [Dota project](https://openai.com/research/openai-five-defeats-dota-2-world-champions), but that's a very inefficient way for these models to re-learn everything. (Kanjun from Imbue shared [similar ideas in her episode](https://www.latent.space/p/imbue)).

**David argues that there’s a shortcut.** We can bootstrap from existing intelligence.

> *“Years ago, I had a debate with a Berkeley professor as to what will it actually take to build AGI. And his view is basically that you have to reproduce all the flops that went into evolution in order to be able to get there… I think we are ignoring the fact that you have a giant shortcut, which is **you can behaviorally clone everything humans already know**. And that's what we solved with LLMs!”*

LLMs today basically model intelligence using all (good!) written knowledge (see our [Datasets 101 episode](https://www.latent.space/p/datasets-101)), and have now expanded to non-verbal knowledge (see our [HuggingFace episode](https://www.latent.space/p/idefics) on multimodality). The SOTA self-supervised pre-training process is surprisingly data-efficient in taking large amounts of unstructured data, and approximating reasoning without overfitting.

But how do you cross the gap from the LLMs of today to building the AGI we all want?

This is why David & friends left to start Adept.

> “*We believe the clearest framing of general intelligence is **a system that can do anything a human can do in front of a computer**. A foundation model for actions, trained to use every software tool, API, and webapp that exists, is a practical path to this ambitious goal*” — [ACT-1 Blogpost](https://www.adept.ai/blog/act-1)

The AGI dream is fully autonomous agents, but there are [levels to autonomy](https://www.latent.space/p/agents) that we are comfortable giving our agents, based on how reliable they are. In David’s word choice, we always want higher levels of “abstractions” (aka autonomy), but our need for “reliability” is the practical limit on how high of an abstraction we can use.

> *“The critical path for Adept is we want to build agents that can do a higher and higher level abstraction things over time, all while keeping an insanely high reliability standard. Because that's what turns us from research into something that customers want. And **if you build agents with really high reliability standard, but are continuing pushing a level of abstraction, you then learn from your users how to get that next level of abstraction faster.** So that's how you actually build the data flow.*
> 
> *That's the critical path for the company. Everything we do is in service of that.”*

We saw how Adept thinks about different levels of abstraction at the 2023 Summit:

The highest abstraction is the “AI Employee”, but we’ll get there with “AI enabled employees”. Alessio [recently gave a talk](https://twitter.com/FanaHOVA/status/1771235466207195379) about the future of work with “services as software” at this week’s Nvidia GTC ([slides](https://github.com/FanaHOVA/nvidia-gtc-2024-slides)).

Unlike a lot of large research labs, Adept's framing of AGI as "*being able to use your computer*  **like a human**" carries with it a useful environmental constraint:

> *“Having a human robot **lets you do things that humans do*** **without changing everything along the way***. It's the same thing for software, right? If you go itemize out the number of things you want to do on your computer for which every step has an API, those numbers of workflows add up pretty close to zero. And so then many points along the way, you need the ability to actually control your computer like a human. It also lets you **learn from human usage of computers as a source of training data that you don't get** if you have to somehow figure out how every particular step needs to be some particular custom private API thing. And so I think this is actually the most practical path (to economic value).”*

This realization and conviction means that multimodal modals are the way to go. Instead of using function calling to call APIs to build agents, which is what OpenAI and most of the open LLM industry have done to date, Adept wants to “drive by vision”, (aka see the screen as a human sees it) and pinpoint where to click and type as a human does. No APIs needed, because most software don’t expose APIs.

> Extra context for readers: You can see [the DeepMind SIMA model](https://buttondown.email/ainews/archive/ainews-deepmind-sima-one-ai-9-games-600-tasks/) in the same light:
> 
> One system that learned to play a diverse set of games (instead of one dedicated model per game) using **only** pixel inputs and keyboard-and-mouse action outputs!
> 
> The OpenInterpreter team is working on a “[Computer API](https://api.openinterpreter.com/)” that also does the same.

To do this, Adept had to double down on a special kind of **multimodality for knowledge work**:

> *“A giant thing that was really necessary is really fast multimodal models that are really good at **understanding knowledge work** and really good at **understanding screens**. And that is needs to kind of be the base for some of these agents…*
> 
> *…I think one big hangover primarily academic focus for multimodal models is most multimodal models are primarily trained on like **natural images, cat and dog photos**, stuff that's come out of the camera… (but) where are they going to be the most useful? **They're going to be most useful in knowledge work tasks. That's where the majority of economic value is going to be**. It's not in cat and dogs.*
> 
> *And so if that's what it is, what do you need to train? I need to train on like **charts, graphs, tables, invoices, PDFs, receipts, unstructured data, UIs**. That's just a totally different pre-training corpus. And so Adept spent a lot of time building that.”*

With this context, you can now understand the full path of Adept’s public releases:

*   **[ACT-1](https://www.adept.ai/blog/act-1)** (Sept 2022): a large Transformers model optimized for browser interactions. It has a custom rendering of the browser viewport that allows it to better understand it and take actions.

*   **[Persimmon-8B](https://www.adept.ai/blog/persimmon-8b)** (Sept 2023): a permissive open LLM ([weights and code here](https://github.com/persimmon-ai-labs/adept-inference))

*   **[Fuyu-8B](https://www.adept.ai/blog/fuyu-8b)** (Oct 2023): a small version of the **multimodal** model that powers Adept. Vanilla decoder-only transformer with no specialized image encoder, which allows it to handle input images of varying resolutions without downsampling.

*   **[Adept Experiments](https://www.adept.ai/blog/experiments)** (Nov 2023): A public tool to build **automations in the browser**. This is powered by Adept's core technology but it's just a piece of their enterprise platform. They use it as a way to try various design ideas.

*   **[Fuyu Heavy](https://www.adept.ai/blog/adept-fuyu-heavy)** (Jan 2024) - a new **multimodal** model designed specifically for digital agents and the world’s third-most-capable multimodal model (beating Gemini Pro on MMMU, AI2D, and ChartQA), “behind only GPT4-V and Gemini Ultra, which are 10-20 times bigger”

The Fuyu-8B post in particular exhibits a great number of examples on knowledge work multimodality:

With OpenAI now worth >$90b and Anthropic >$18b, it is tempting to conclude that the AI startup metagame is to build a large research lab, and attract the brightest minds and highest capital to build AGI.

Our past guests

(see [the Humanloop episode](https://latent.space/p/humanloop)) and (from [Imbue](https://www.latent.space/p/imbue)) combined to ask the most challenging questions of the pod - with David/Adept’s deep research pedigree from Deepmind and OpenAI, why is Adept not building more general foundation models (like Persimmon) and playing the academic benchmarks game? Why is Adept so focused on commercial agents instead?

> *“I feel super good that we're doing foundation models in service of agents and all of the reward within Adept is flowing from “Can we make a better agent”…*
> 
> *… I think **pure play foundation model companies are just going to be pinched by how good the next couple of (Meta Llama models) are going to be**… And then seeing the really big players put ridiculous amounts of compute behind just training these base foundation models, **I think is going to commoditize a lot of the regular LLMs and soon regular multimodal models**. So I feel really good that we're just focused on agents.”*

and the commercial grounding is his answer to Kanjun too (whom we also asked the inverse question to compare with Adept):

> *“… the second reason I work at Adept is if you believe that **actually having customers and a reward signal from customers lets you build AGI faster**, which we really believe, then you should come here. And I think the examples for why that's true is for example, **our evaluations are not academic evals**. They're not simulator evals. They're like, okay, we have a customer that really needs us to do these particular things. We can do some of them. These are the ones they want us to, we can't do them at all. We've turned those into evals.. I think that's a degree of practicality that really helps.”*

And his customers seem pretty happy, because David didn’t need to come on to do a sales pitch:

> *David: “One of the things we haven't shared before is we're completely sold out for Q1.”*
> 
> *Swyx: “Sold out of what?”*
> 
> *David: “Sold out of bandwidth to onboard more customers.”*

Well, that’s a great problem to have.

*   [00:00:00] Introductions

*   [00:01:14] Being employee #30 at OpenAI and its early days

*   [00:13:38] What is Adept and how do you define AGI?

*   [00:21:00] Adept's critical path and research directions

*   [00:26:23] How AI agents should interact with software and impact product development

*   [00:30:37] Analogies between AI agents and self-driving car development

*   [00:32:42] Balancing reliability, cost, speed and generality in AI agents

*   [00:37:30] Potential of foundation models for robotics

*   [00:39:22] Core research questions and reasons to work at Adept

**Alessio** [00:00:00]: Hey everyone, welcome to the Latent Space Podcast. This is Alessio, partner and CTO in Residence at [Decibel Partners](https://decibel.vc/), and I'm joined by my co-host Swyx, founder of [Smol.ai](https://smol.ai/).

**Swyx** [00:00:15]: Hey, and today we have David Luan, CEO, co-founder of Adept in the studio. Welcome.

**David** [00:00:20]: Yeah, thanks for having me.

**Swyx** [00:00:21]: Been a while in the works. I've met you socially at one of those VC events and you said that you were interested in coming on and glad we finally were able to make this happen.

**David**: Yeah, happy to be part of it.

**Swyx**: So we like to introduce the speaker and then also just like have you talk a little bit about like what's not on your LinkedIn, what people should just generally know about you. You started a company in college, which was the first sort of real time video detection classification API that was Dextro, and that was your route to getting acquired into Axon where you're a director of AI. Then you were the 30th hire at OpenAI?

**David** [00:00:53]: Yeah, 30, 35, something around there. Something like that.

**Swyx** [00:00:56]: So you were VP of Eng for two and a half years to two years, briefly served as tech lead of large models at Google, and then in 2022 started Adept. So that's the sort of brief CV. Is there anything else you like want to fill in the blanks or like people should know more about?

**David** [00:01:14]: I guess a broader story was I joined OpenAI fairly early and I did that for about two and a half to three years leading engineering there. It's really funny, I think second or third day of my time at OpenAI, Greg and Ilya pulled me in a room and we're like, you know, you should take over our directs and we'll go mostly do IC work. So that was fun, just coalescing a bunch of teams out of a couple of early initiatives that had already happened. The company, the Dota effort was going pretty hard and then more broadly trying to put bigger picture direction around what we were doing with basic research. So I spent a lot of time doing that. And then I led Google's LLM efforts, but also co-led Google Brain was one of the brain leads more broadly. You know, there's been a couple of different eras of AI research, right? If we count everything before 2012 as prehistory, which people hate it when I say that, kind of had this like you and your three best friends write a research paper that changes the world period from like 2012 to 2017\. And I think the game changed in 2017 and like most labs didn't realize it, but we at OpenAI really did. I think in large part helped by like Ilya's constant beating of the drum that the world would be covered in data centers. And I think-

**Swyx** [00:02:15]: It's causally neat.

**David** [00:02:16]: Yeah. Well, like I think we had conviction in that, but it wasn't until we started seeing results that it became clear that that was where we had to go. But also part of it as well was for OpenAI, like when I first joined, I think one of the jobs that I had to do was how do I tell a differentiated vision for who we were technically compared to, you know, hey, we're just smaller Google Brain, or like you work at OpenAI if you live in SF and don't want to commute to Mountain View or don't want to live in London, right? That's like not enough to like hang your technical identity as a company. And so what we really did was, and I spent a lot of time pushing this, is just how do we get ourselves focused on a certain class of like giant swings and bets, right? Like how do you flip the script from you just do bottom-up research to more about how do you like leave some room for that, but really make it about like, what are the big scientific outcomes that you want to show? And then you just solve them at all costs, whether or not you care about novelty and all that stuff. And that became the dominant model for a couple of years, right? And then what's changed now is I think the number one driver of AI products over the next couple of years is going to be the deep co-design and co-evolution of product and users for feedback and actual technology. And I think labs, every tool to go do that are going to do really well. And that's a big part of why I started Adept.

**Alessio** [00:03:20]: You mentioned Dota, any memories thinking from like the switch from RL to Transformers at the time and kind of how the industry was evolving more in the LLM side and leaving behind some of the more agent simulation work?

**David** [00:03:33]: Like zooming way out, I think agents are just absolutely the correct long-term direction, right? You just go to find what AGI is, right? You're like, Hey, like, well, first off, actually, I don't love AGI definitions that involve human replacement because I don't think that's actually how it's going to happen. Even this definition of like, Hey, AGI is something that outperforms humans at economically valuable tasks is kind of implicit view of the world about what's going to be the role of people. I think what I'm more interested in is like a definition of AGI that's oriented around like a model that can do anything a human can do on a computer. If you go think about that, which is like super tractable, then agent is just a natural consequence of that definition. And so what did all the work we did on our own stuff like that get us was it got us a really clear formulation. Like you have a goal and you want to maximize the goal, you want to maximize reward, right? And the natural LLM formulation doesn't come with that out of the box, right? I think that we as a field got a lot right by thinking about, Hey, how do we solve problems of that caliber? And then the thing we forgot is the Novo RL is like a pretty terrible way to get there quickly. Why are we rediscovering all the knowledge about the world? Years ago, I had a debate with a Berkeley professor as to what will it actually take to build AGI. And his view is basically that you have to reproduce all the flops that went into evolution in order to be able to get there. Right.

**Swyx** [00:04:44]: The biological basis theory. Right.

**David** [00:04:46]: So I think we are ignoring the fact that you have a giant shortcut, which is you can behavioral clone everything humans already know. And that's what we solved with LLMs. We've solved behavioral cloning, everything that humans already know. Right. So like today, maybe LLMs is like behavioral cloning every word that gets written on the internet in the future, the multimodal models are becoming more of a thing where behavioral cloning the visual world. But really, what we're just going to have is like a universal byte model, right? Where tokens of data that have high signal come in, and then all of those patterns are like learned by the model. And then you can regurgitate any combination now. Right. So text into voice out, like image into other image out or video out or whatever, like these like mappings, right? Like all just going to be learned by this universal behavioral cloner. And so I'm glad we figured that out. And I think now we're back to the era of how do we combine this with all of the lessons we learned during the RL period. That's what's going to drive progress.

**Swyx** [00:05:35]: I'm still going to pressure you for a few more early opening stories before we turn to the ADET stuff. On your personal site, which I love, because it's really nice, like personal, you know, story context around like your history. I need to update it. It's so old. Yeah, it's so out of date. But you mentioned GPT-2\. Did you overlap with GPT-1? I think you did, right?

**David** [00:05:53]: I actually don't quite remember. I think I was joining right around- Right around then?

**Swyx** [00:05:57]: I was right around that, yeah. Yeah. So what I remember was Alec, you know, just kind of came in and was like very obsessed with Transformers and applying them to like Reddit sentiment analysis. Yeah, sentiment, that's right. Take us through-

**David** [00:06:09]: Sentiment neuron, all this stuff.

**Swyx** [00:06:10]: The history of GPT as far as you know, you know, according to you. Ah, okay.

**David** [00:06:14]: History of GPT, according to me, that's a pretty good question. So I think the real story of GPT starts at Google, of course, right? Because that's where Transformers sort of came about. However, the number one shocking thing to me was that, and this is like a consequence of the way that Google is organized, where like, again, you and your three best friends write papers, right? Okay. So zooming way out, right? I think about my job when I was a full-time research leader as a little bit of a portfolio allocator, right? So I've got really, really smart people. My job is to convince people to coalesce around a small number of really good ideas and then run them over the finish line. My job is not actually to promote a million ideas and never have critical mass. And then as the ideas start coming together and some of them start working well, my job is to nudge resources towards the things that are really working and then start disbanding some of the things that are not working, right? That muscle did not exist during my time at Google. And I think had they had it, what they would have done would be say, hey, Noam Shazir, you're a brilliant guy. You know how to scale these things up. Here's half of all of our TPUs. And then I think they would have destroyed us. He clearly wanted it too.

**Swyx** [00:07:17]: He's talking about trillion parameter models in 2017.

**David** [00:07:20]: Yeah. So that's the core of the GPT story, right? Which is that, and I'm jumping around historically, right? But after GPT-2, we were all really excited about GPT-2\. I can tell you more stories about that. It was the last paper that I even got to really touch before everything became more about building a research org. You know, every day we were scaling up GPT-3, I would wake up and just be stressed. And I was stressed because, you know, you just look at the facts, right? Google has all this compute. Google has all the people who invented all of these underlying technologies. There's a guy named Noam who's really smart, who's already gone and done this talk about how he wants a trillion parameter model. And I'm just like, we're probably just doing duplicative research to what he's doing, right? He's got this decoder only transformer that's probably going to get there before we do. And I was like, but like, please just like let this model finish, right? And it turned out the whole time that they just couldn't get critical mass. So during my year where I led the Google LM effort and I was one of the brain leads, you know, it became really clear why, right? At the time, there was a thing called the brain credit marketplace. And did you guys know the brain credit marketplace? No, I never heard of this. Oh, so it's actually, it's a, you can ask any Googler.

**Swyx** [00:08:23]: It's like just like a thing that, that, I mean, look like, yeah, limited resources, you got to have some kind of marketplace, right? You know, sometimes it's explicit, sometimes it isn't, you know, just political favors.

**David** [00:08:34]: You could. And so then basically everyone's assigned a credit, right? So if you have a credit, you get to buy end chips according to supply and demand. So if you want to go do a giant job, you had to convince like 19 or 20 of your colleagues not to do work. And if that's how it works, it's really hard to get that bottom up critical mass to go scale these things. And the team at Google were fighting valiantly, but we were able to beat them simply because we took big swings and we focused. And I think, again, that's like part of the narrative of like this phase one of AI, right? Of like this modern AI era to phase two. And I think in the same way, I think phase three company is going to out execute phase two companies because of the same asymmetry of success.

**Swyx** [00:09:12]: Yeah. I think it's underrated how much NVIDIA works with you in the early days as well. I think maybe, I think it was Jensen. I'm not sure who circulated a recent photo of him delivering the first DGX to you guys.

**David** [00:09:24]: I think Jensen has been a complete legend and a mastermind throughout. I have so much respect for NVIDIA. It is unreal.

**Swyx** [00:09:34]: But like with OpenAI, like kind of give their requirements, like co-design it or just work of whatever NVIDIA gave them.

**David** [00:09:40]: So we work really closely with them. There's, I'm not sure I can share all the stories, but examples of ones that I've found particularly interesting. So Scott Gray is amazing. I really like working with him. He was on one of my teams, the supercomputing team, which Chris Berner runs and Chris Berner still does a lot of stuff in that. As a result, like we had very close ties to NVIDIA. Actually, one of my co-founders at Adept, Eric Elson, was also one of the early GPGPU people. So he and Scott and Brian Catanzaro at NVIDIA and Jonah and Ian at NVIDIA, I think all were very close. And we're all sort of part of this group of how do we push these chips to the absolute limit? And I think that kind of collaboration helped quite a bit. I think one interesting set of stuff is knowing the A100 generation, that like quad sparsity was going to be a thing. Is that something that we want to go look into, right? And figure out if that's something that we could actually use for model training. Really what it boils down to is that, and I think more and more people realize this, six years ago, people, even three years ago, people refused to accept it. This era of AI is really a story of compute. It's really the story of how do you more efficiently map actual usable model flops to compute,

**Swyx** [00:10:38]: Is there another GPT 2, 3 story that you love to get out there that you think is underappreciated for the amount of work that people put into it?

**David** [00:10:48]: So two interesting GPT 2 stories. One of them was I spent a good bit of time just sprinting to help Alec get the paper out. And I remember one of the most entertaining moments was we were writing the modeling section. And I'm pretty sure the modeling section was the shortest modeling section of any ML, reasonably legitimate ML paper to that moment. It was like section three model. This is a standard vanilla decoder only transformer with like these particular things, those paragraph long if I remember correctly. And both of us were just looking at the same being like, man, the OGs in the field are going to hate this. They're going to say no novelty. Why did you guys do this work? So now it's funny to look at in hindsight that it was pivotal kind of paper, but I think it was one of the early ones where we just leaned fully into all we care about is solving problems in AI and not about, hey, is there like four different really simple ideas that are cloaked in mathematical language that doesn't actually help move the field forward?

**Swyx** [00:11:42]: Right. And it's like you innovate on maybe like data set and scaling and not so much the architecture.

**David** [00:11:48]: We all know how it works now, right? Which is that there's a collection of really hard won knowledge that you get only by being at the frontiers of scale. And that hard won knowledge, a lot of it's not published. A lot of it is stuff that's actually not even easily reducible to what looks like a typical academic paper. But yet that's the stuff that helps differentiate one scaling program from another. You had a second one? So the second one is, there's like some details here that I probably shouldn't fully share, but hilariously enough for the last meeting we did with Microsoft before Microsoft invested in OpenAI, Sam Altman, myself and our CFO flew up to Seattle to do the final pitch meeting. And I'd been a founder before. So I always had a tremendous amount of anxiety about partner meetings, which this basically this is what it was. I had Kevin Scott and Satya and Amy Hood, and it was my job to give the technical slides about what's the path to AGI, what's our research portfolio, all of this stuff, but it was also my job to give the GPT-2 demo. We had a slightly bigger version of GPT-2 that we had just cut maybe a day or two before this flight up. And as we all know now, model behaviors you find predictable at one checkpoint are not predictable in another checkpoint. And so I'd spent all this time trying to figure out how to keep this thing on rails. I had my canned demos, but I knew I had to go turn it around over to Satya and Kevin and let them type anything in. And that just, that really kept me up all night.

**Swyx** [00:13:06]: Nice. Yeah.

**Alessio** [00:13:08]: I mean, that must have helped you talking about partners meeting. You raised $420 million for Adept. The last round was a $350 million Series B, so I'm sure you do great in partner meetings.

**Swyx** [00:13:18]: Pitchers meetings. Nice.

**David** [00:13:20]: No, that's a high compliment coming from a VC.

**Alessio** [00:13:22]: Yeah, no, I mean, you're doing great already for us. Let's talk about Adept. And we were doing pre-prep and you mentioned that maybe a lot of people don't understand what Adept is. So usually we try and introduce the product and then have the founders fill in the blanks, but maybe let's do the reverse. Like what is Adept? Yeah.

**David** [00:13:38]: So I think Adept is the least understood company in the broader space of foundational models plus agents. So I'll give some color and I'll explain what it is and I'll explain also why it's actually pretty different from what people would have guessed. So the goal for Adept is we basically want to build an AI agent that can do, that can basically help humans do anything a human does on a computer. And so what that really means is we want this thing to be super good at turning natural language like goal specifications right into the correct set of end steps and then also have all the correct sensors and actuators to go get that thing done for you across any software tool that you already use. And so the end vision of this is effectively like I think in a couple of years everyone's going to have access to like an AI teammate that they can delegate arbitrary tasks to and then also be able to, you know, use it as a sounding board and just be way, way, way more productive. Right. And just changes the shape of every job from something where you're mostly doing execution to something where you're mostly actually doing like these core liberal arts skills of what should I be doing and why. Right. And I find this like really exciting and motivating because I think it's actually a pretty different vision for how AGI will play out. I think systems like Adept are the most likely systems to be proto-AGIs. But I think the ways in which we are really counterintuitive to everybody is that we've actually been really quiet because we are not a developer company. We don't sell APIs. We don't sell open source models. We also don't sell bottom up products. We're not a thing that you go and click and download the extension and like we want more users signing up for that thing. We're actually an enterprise company. So what we do is we work with a range of different companies, some like late stage multi-thousand people startups, some fortune 500s, et cetera. And what we do for them is we basically give them an out of the box solution where big complex workflows that their employees do every day could be delegated to the model. And so we look a little different from other companies in that in order to go build this full agent thing, the most important thing you got to get right is reliability. So initially zooming way back when, one of the first things that DEP did was we released this demo called Act One, right? Act One was like pretty cool. It's like kind of become a hello world thing for people to show agent demos by going to Redfin and asking to buy a house somewhere because like we did that in the original Act One demo and like showed that, showed like Google Sheets, all this other stuff. Over the last like year since that has come out, there's been a lot of really cool demos and you go play with them and you realize they work 60% of the time. But since we've always been focused on how do we build an amazing enterprise product, enterprises can't use anything that isn't in the nines of reliability. And so we've actually had to go down a slightly different tech tree than what you might find in the prompt engineering sort of plays in the agent space to get that reliability. And we've decided to prioritize reliability over all else. So like one of our use cases is crazy enough that it actually ends with a physical truck being sent to a place as the result of the agent workflow. And if you're like, if that works like 60% of the time, you're just blowing money and poor truck drivers going places.

**Alessio** [00:16:30]: Interesting. One of the, our investment teams has this idea of services as software. I'm actually giving a talk at NVIDIA GTC about this, but basically software as a service, you're wrapping user productivity in software with agents and services as software is replacing things that, you know, you would ask somebody to do and the software just does it for you. When you think about these use cases, do the users still go in and look at the agent kind of like doing the things and can intervene or like are they totally removed from them? Like the truck thing is like, does the truck just show up or are there people in the middle checking in?

**David** [00:17:04]: I think there's two current flaws in the framing for services as software, or I think what you just said. I think that one of them is like in our experience, as we've been rolling out Adept, the people who actually do the jobs are the most excited about it because they don't go from, I do this job to, I don't do this job. They go from, I do this job for everything, including the shitty rote stuff to I'm a supervisor. And I literally like, it's pretty magical when you watch the thing being used because now it parallelizes a bunch of the things that you had to do sequentially by hand as a human. And you can just click into any one of them and be like, Hey, I want to watch the trajectory that the agent went through to go solve this. And the nice thing about agent execution as opposed to like LLM generations is that a good chunk of the time when the agent fails to execute, it doesn't give you the wrong result. It just fails to execute. And the whole trajectory is just broken and dead and the agent knows it, right? So then those are the ones that the human then goes and solves. And so then they become a troubleshooter. They work on the more challenging stuff. They get way, way more stuff done and they're really excited about it. I think the second piece of it that we've found is our strategy as a company is to always be an augmentation company. And I think one out of principle, that's something we really care about. But two, actually, if you're framing yourself as an augmentation company, you're always going to live in a world where you're solving tasks that are a little too hard for what the model can do today and still needs a human to provide oversight, provide clarifications, provide human feedback. And that's how you build a data flywheel. That's how you actually learn from the smartest humans how to solve things models can't do today. And so I actually think that being an augmentation company forces you to go develop your core AI capabilities faster than someone who's saying, ah, okay, my job is to deliver you a lights off solution for X.

**Alessio** [00:18:42]: Yeah. It's interesting because we've seen two parts of the market. One is we have one company that does agents for SOC analysts. People just don't have them, you know, and just they cannot attract the talent to do it. And similarly, in a software development, you have Copilot, which is the augmentation product, and then you have sweep.dev and you have these products, which they just do the whole thing. I'm really curious to see how that evolves. I agree that today the reliability is so important in the enterprise that they just don't use most of them. Yeah. Yeah. No, that's cool. But it's great to hear the story because I think from the outside, people are like, oh, a dev, they do Act One, they do Persimon, they do Fuyu, they do all this stuff. Yeah, it's just the public stuff.

**Swyx** [00:19:20]: It's just public stuff.

**David** [00:19:21]: So one of the things we haven't shared before is we're completely sold out for Q1\. And so I think...

**Swyx** [00:19:26]: Sold out of what?

**David** [00:19:27]: Sold out of bandwidth to go on board more customers. And so we're like working really hard to go make that less of a bottleneck, but our expectation is that I think we're going to be significantly more public about the broader product shape and the new types of customers we want to attract later this year. So I think that clarification will happen by default.

**Swyx** [00:19:43]: Why have you become more public? You know, if the whole push has... You're sold out, you're my enterprise, but you're also clearly putting effort towards being more open or releasing more things.

**David** [00:19:53]: I think we just flipped over that way fairly recently. That's a good question. I think it actually boils down to two things. One, I think that, frankly, a big part of it is that the public narrative is really forming around agents as being the most important thing. And I'm really glad that's happening because when we started the company in January 2022, everybody in the field knew about the agents thing from RL, but the general public had no conception of what it was. They were still hanging their narrative hat on the tree of everything's a chatbot. And so I think now one of the things that I really care about is that when people think agent, they actually think the right thing. All sorts of different things are being called agents. Chatbots are being called agents. Things that make a function call are being called agents. To me, an agent is something that you can give a goal and get an end step workflow done correctly in the minimum number of steps. And so that's a big part of why. And I think the other part is because I think it's always good for people to be more aware of Redept as they think about what the next thing they want to do in their careers. The field is quickly pivoting in a world where foundation models are looking more and more commodity. And I think a huge amount of gain is going to happen from how do you use foundation models as the well-learned behavioral cloner to go solve agents. And I think people who want to do agents research should really come to Redept.

**Swyx** [00:21:00]: When you say agents have become more part of the public narrative, are there specific things that you point to? I'll name a few. Bill Gates in his blog post mentioning that agents are the future. I'm the guy who made OSes, and I think agents are the next thing. So Bill Gates, I'll call that out. And then maybe Sam Altman also saying that agents are the future for open AI.

**David** [00:21:17]: I think before that even, I think there was something like the New York Times, Cade Metz wrote a New York Times piece about it. Right now, in a bit to differentiate, I'm seeing AI startups that used to just brand themselves as an AI company, but now brand themselves as an AI agent company. It's just like, it's a term I just feel like people really want.

**Swyx** [00:21:31]: From the VC side, it's a bit mixed. Is it? As in like, I think there are a lot of VCs where like, I would not touch any agent startups because like- Why is that? Well, you tell me.

**Alessio** [00:21:41]: I think a lot of VCs that are maybe less technical don't understand the limitations of the-

**Swyx** [00:21:46]: No, that's not fair.

**Alessio** [00:21:47]: No, no, no, no. I think like- You think so? No, no. I think like the, what is possible today and like what is worth investing in, you know? And I think like, I mean, people look at you and say, well, these guys are building agents. They needed 400 million to do it. So a lot of VCs are maybe like, oh, I would rather invest in something that is tacking on AI to an existing thing, which is like easier to get the market and kind of get some of the flywheel going. But I'm also surprised a lot of funders just don't want to do agents. It's not even the funding. Sometimes we look around and it's like, why is nobody doing agents for X? Wow.

**David** [00:22:17]: That's good to know actually. I never knew that before. My sense from my limited perspective is there's a new agent company popping up every day.

**Swyx** [00:22:24]: So maybe I'm- They are. They are. But like I have advised people to take agents off of their title because it's so diluted.

**David** [00:22:31]: It's now so diluted.

**Swyx** [00:22:32]: Yeah. So then it doesn't stand for anything. Yeah.

**David** [00:22:35]: That's a really good point.

**Swyx** [00:22:36]: So like, you know, you're a portfolio allocator. You have people know about Persimmon, people know about Fuyu and Fuyu Heavy. Can you take us through like how you think about that evolution of that and what people should think about what that means for adepts and sort of research directions? Kind of take us through the stuff you shipped recently and how people should think about the trajectory of what you're doing.

**David** [00:22:56]: The critical path for adepts is we want to build agents that can do a higher and higher level abstraction things over time, all while keeping an insanely high reliability standard. Because that's what turns us from research into something that customers want. And if you build agents with really high reliability standard, but are continuing pushing a level of abstraction, you then learn from your users how to get that next level of abstraction faster. So that's how you actually build the data flow. That's the critical path for the company. Everything we do is in service of that. So if you go zoom way, way back to Act One days, right? Like the core thing behind Act One is can we teach large model basically how to even actuate your computer? And I think we're one of the first places to have solved that and shown it and shown the generalization that you get when you give it various different workflows and texts. But I think from there on out, we really realized was that in order to get reliability, companies just do things in various different ways. You actually want these models to be able to get a lot better at having some specification of some guardrails for what it actually should be doing. And I think in conjunction with that, a giant thing that was really necessary is really fast multimodal models that are really good at understanding knowledge work and really good at understanding screens. And that is needs to kind of be the base for some of these agents. Back then we had to do a ton of research basically on how do we actually make that possible? Well, first off, like back in forgot exactly one month to 23, like there were no multimodal models really that you could use for things like this. And so we pushed really hard on stuff like the Fuyu architecture. I think one big hangover primarily academic focus for multimodal models is most multimodal models are primarily trained on like natural images, cat and dog photos, stuff that's come out of the camera. Coco. Yeah, right. And the Coco is awesome. Like I love Coco. I love TY. Like it's really helped the field. Right. But like that's the build one thing. I actually think it's really clear today. Multimodal models are the default foundation model, right? It's just going to supplant LLMs. Like you just train a giant multimodal model. And so for that though, like where are they going to be the most useful? They're going to be most useful in knowledge work tasks. That's where the majority of economic value is going to be. It's not in cat and dogs. Right. And so if that's what it is, what do you need to train? I need to train on like charts, graphs, tables, invoices, PDFs, receipts, unstructured data, UIs. That's just a totally different pre-training corpus. And so a depth spent a lot of time building that. And so the public for use and stuff aren't trained on our actual corpus, it's trained on some other stuff. But you take a lot of that data and then you make it really fast and make it really good at things like dense OCR on screens. And then now you have the right like raw putty to go make a good agent. So that's kind of like some of the modeling side, we've kind of only announced some of that stuff. We haven't really announced much of the agent's work, but that if you put those together with the correct product form factor, and I think the product form factor also really matters. I think we're seeing, and you guys probably see this a little bit more than I do, but we're seeing like a little bit of a pushback against the tyranny of chatbots as form factor. And I think that the reason why the form factor matters is the form factor changes what data you collect in the human feedback loop. And so I think we've spent a lot of time doing full vertical integration of all these bits in order to get to where we are.

**Swyx** [00:25:44]: Yeah. I'll plug Amelia Wattenberger’s talk at our conference, where she gave a little bit of the thinking behind like what else exists other than chatbots that if you could delegate to reliable agents, you could do. I was kind of excited at Adept experiments or Adept workflows, I don't know what the official name for it is. I was like, okay, like this is something I can use, but it seems like it's just an experiment for now. It's not your product.

**David** [00:26:06]: So you basically just use experiments as like a way to go push various ideas on the design side to some people and just be like, yeah, we'll play with it. Actually the experiments code base underpins the actual product, but it's just the code base itself is kind of like a skeleton for us to go deploy arbitrary cards on the side.

**Swyx** [00:26:22]: Yeah.

**Alessio** [00:26:23]: Makes sense. I was going to say, I would love to talk about the interaction layer. So you train a model to see UI, but then there's the question of how do you actually act on the UI? I think there was some rumors about open app building agents that are kind of like, they manage the end point. So the whole computer, you're more at the browser level. I read in one of your papers, you have like a different representation, kind of like you don't just take the dome and act on it. You do a lot more stuff. How do you think about the best way the models will interact with the software and like how the development of products is going to change with that in mind as more and more of the work is done by agents instead of people?

**David** [00:26:58]: This is, there's so much surface area here and it's actually one of the things I'm really excited about. And it's funny because I've spent most of my time doing research stuff, but there's like a whole new ball game that I've been learning about and I find it really cool. So I would say the best analogy I have to why Adept is pursuing a path of being able to use your computer like a human, plus of course being able to call APIs and being able to call APIs is the easy part, like being able to use your computer like a human is a hard part. It's in the same way why people are excited about humanoid robotics, right? In a world where you had T equals infinity, right? You're probably going to have various different form factors that robots could just be in and like all the specialization. But the fact is that humans live in a human environment. So having a human robot lets you do things that humans do without changing everything along the way. It's the same thing for software, right? If you go itemize out the number of things you want to do on your computer for which every step has an API, those numbers of workflows add up pretty close to zero. And so then many points along the way, you need the ability to actually control your computer like a human. It also lets you learn from human usage of computers as a source of training data that you don't get if you have to somehow figure out how every particular step needs to be some particular custom private API thing. And so I think this is actually the most practical path. I think because it's the most practical path, I think a lot of success will come from going down this path. I kind of think about this early days of the agent interaction layer level is a little bit like, do you all remember Windows 3.1? Like those days? Okay, this might be, I might be, I might be too old for you guys on this. But back in the day, Windows 3.1, we had this transition period between pure command line, right? Being the default into this new world where the GUI is the default and then you drop into the command line for like programmer things, right? The old way was you booted your computer up, DOS booted, and then it would give you the C colon slash thing. And you typed Windows and you hit enter, and then you got put into Windows. And then the GUI kind of became a layer above the command line. The same thing is going to happen with agent interfaces is like today we'll be having the GUI is like the base layer. And then the agent just controls the current GUI layer plus APIs. And in the future, as more and more trust is built towards agents and more and more things can be done by agents, if more UIs for agents are actually generative in and of themselves, then that just becomes a standard interaction layer. And if that becomes a standard interaction layer, what changes for software is that a lot of software is going to be either systems or record or like certain customized workflow execution engines. And a lot of how you actually do stuff will be controlled at the agent layer.

**Alessio** [00:29:19]: And you think the rabbit interface is more like it would like you're not actually seeing the app that the model interacts with. You're just saying, hey, I need to log this call on Salesforce. And you're never actually going on salesforce.com directly as the user. I can see that being a model.

**David** [00:29:33]: I think I don't know enough about what using rabbit in real life will actually be like to comment on that particular thing. But I think the broader idea that, you know, you have a goal, right? The agent knows how to break your goal down into steps. The agent knows how to use the underlying software and systems or record to achieve that goal for you. The agent maybe presents you information in a custom way that's only relevant to your particular goal, all just really leads to a world where you don't really need to ever interface with the apps underneath unless you're a power user for some niche thing.

**Swyx** [00:30:03]: General question. So first of all, I think like the sort of input mode conversation. I wonder if you have any analogies that you like with self-driving, because I do think like there's a little bit of how the model should perceive the world. And you know, the primary split in self-driving is LiDAR versus camera. And I feel like most agent companies that I'm tracking are all moving towards camera approach, which is like the multimodal approach, you know, multimodal vision, very heavy vision, all the Fuyu stuff that you're doing. You're focusing on that, including charts and tables. And do you find that inspiration there from like the self-driving world? That's a good question.

**David** [00:30:37]: I think sometimes the most useful inspiration I've found from self-driving is the levels analogy. I think that's awesome. But I think that our number one goal is for agents not to look like self-driving. We want to minimize the chances that agents are sort of a thing that you just have to bang your head at for a long time to get to like two discontinuous milestones, which is basically what's happened in self-driving. We want to be living in a world where you have the data flywheel immediately, and that takes you all the way up to the top. But similarly, I mean, compared to self-driving, like two things that people really undervalue is like really easy to driving a car down highway 101 in a sunny day demo. That actually doesn't prove anything anymore. And I think the second thing is that as a non-self-driving expert, I think one of the things that we believe really strongly is that everyone undervalues the importance of really good sensors and actuators. And actually a lot of what's helped us get a lot of reliability is a really strong focus on actually why does the model not do this thing? And the non-trivial amount of time, the time the model doesn't actually do the thing is because if you're a wizard of ozzing it yourself, or if you have unreliable actuators, you can't do the thing. And so we've had to fix a lot of those problems.

**Swyx** [00:31:43]: I was slightly surprised just because I do generally consider the way most that we see all around San Francisco as the most, I guess, real case of agents that we have in very material ways.

**David** [00:31:55]: Oh, that's absolutely true. I think they've done an awesome job, but it has taken a long time for self-driving to mature from when it entered the consciousness and the driving down 101 on a sunny day moment happened to now. Right. So I want to see that more compressed.

**Swyx** [00:32:07]: And I mean, you know, cruise, you know, RIP. And then one more thing on just like, just going back on this reliability thing, something I have been holding in my head that I'm curious to get your commentary on is I think there's a trade-off between reliability and generality, or I want to broaden reliability into just general like sort of production readiness and enterprise readiness scale. Because you have reliability, you also have cost, you have speed, speed is a huge emphasis for a debt. The tendency or the temptation is to reduce generality to improve reliability and to improve cost, improve speed. Do you perceive a trade-off? Do you have any insights that solve those trade-offs for you guys?

**David** [00:32:42]: There's definitely a trade-off. If you're at the Pareto frontier, I think a lot of folks aren't actually at the Pareto frontier. I think the way you get there is basically how do you frame the fundamental agent problem in a way that just continues to benefit from data? I think one of the main ways of being able to solve that particular trade-off is you basically just want to formulate the problem such that every particular use case just looks like you collecting more data to go make that use case possible. I think that's how you really solve. Then you get into the other problems like, okay, are you overfitting on these end use cases? You're not doing a thing where you're being super prescriptive for the end steps that the model can only do, for example.

**Swyx** [00:33:17]: Then the question becomes, do you have one house model that you can then customize for each customer and you're fine-tuning them on each customer's specific use case?

**David** [00:33:25]: Yeah.

**Swyx** [00:33:26]: We're not sharing that. You're not sharing that. It's tempting, but that doesn't look like AGI to me. You know what I mean? That is just you have a good base model and then you fine-tune it.

**David** [00:33:35]: For what it's worth, I think there's two paths to a lot more capability coming out of the models that we all are training these days. I think one path is you figure out how to spend, compute, and turn it into data. In that path, I consider search, RL, all the things that we all love in this era as part of that path, like self-play, all that stuff. The second path is how do you get super competent, high intelligence demonstrations from humans? I think the right way to move forward is you kind of want to combine the two. The first one gives you maximum sample efficiency for a little second, but I think that it's going to be hard to be running at max speed towards AGI without actually solving a bit of both.

**Swyx** [00:34:16]: You haven't talked much about synthetic data, as far as I can tell. Probably this is a bit too much of a trend right now, but any insights on using synthetic data to augment the expensive human data?

**David** [00:34:26]: The best part about framing AGI as being able to help people do things on computers is you have an environment.

**Swyx** [00:34:31]: Yes. So you can simulate all of it.

**David** [00:34:35]: You can do a lot of stuff when you have an environment.

**Alessio** [00:34:37]: We were having dinner for our one-year anniversary. Congrats. Yeah. Thank you. Raza from HumanLoop was there, and we mentioned you were coming on the pod. This is our first-

**Swyx** [00:34:45]: So he submitted a question.

**Alessio** [00:34:46]: Yeah, this is our first, I guess, like mailbag question. He asked, when you started GPD 4 Data and Exist, now you have a GPD 4 vision and help you building a lot of those things. How do you think about the things that are unique to you as Adept, and like going back to like the maybe research direction that you want to take the team and what you want people to come work on at Adept, versus what is maybe now become commoditized that you didn't expect everybody would have access to?

**David** [00:35:11]: Yeah, that's a really good question. I think implicit in that question, and I wish he were tier two so he can push back on my assumption about his question, but I think implicit in that question is calculus of where does advantage accrue in the overall ML stack. And maybe part of the assumption is that advantage accrues solely to base model scaling. But I actually believe pretty strongly that the way that you really win is that you have to go build an agent stack that is much more than that of the base model itself. And so I think like that is always going to be a giant advantage of vertical integration. I think like it lets us do things like have a really, really fast base model, is really good at agent things, but is bad at cat and dog photos. It's pretty good at cat and dog photos. It's not like soda at cat and dog photos, right? So like we're allocating our capacity wisely, right? That's like one thing that you really get to do. I also think that the other thing that is pretty important now in the broader foundation modeling space is I feel despite any potential concerns about how good is agents as like a startup area, right? Like we were talking about earlier, I feel super good that we're doing foundation models in service of agents and all of the reward within Adept is flowing from can we make a better agent? Because right now I think we all see that, you know, if you're training on publicly available web data, you put in the flops and you do reasonable things, then you get decent results. And if you just double the amount of compute, then you get predictably better results. And so I think pure play foundation model companies are just going to be pinched by how good the next couple of llamas are going to be and the next what good open source thing. And then seeing the really big players put ridiculous amounts of compute behind just training these base foundation models, I think is going to commoditize a lot of the regular LLMs and soon regular multimodal models. So I feel really good that we're just focused on agents.

**Swyx** [00:36:56]: So you don't consider yourself a pure play foundation model company?

**David** [00:36:59]: No, because if we were a pure play foundation model company, we would be training general foundation models that do summarization and all this other...

**Swyx** [00:37:06]: You're dedicated towards the agent. Yeah.

**David** [00:37:09]: And our business is an agent business. We're not here to sell you tokens, right? And I think like selling tokens, unless there's like a...

**Swyx** [00:37:14]: Not here to sell you tokens. I love it.

**David** [00:37:16]: It's like if you have a particular area of specialty, right? Then you won't get caught in the fact that everyone's just scaling to ridiculous levels of compute. But if you don't have a specialty, I find that, I think it's going to be a little tougher.

**Swyx** [00:37:27]: Interesting. Are you interested in robotics at all? Just a...

**David** [00:37:30]: I'm personally fascinated by robotics. I've always loved robotics.

**Swyx** [00:37:33]: Embodied agents as a business, you know, Figure is like a big, also sort of open AI affiliated company that raises a lot of money.

**David** [00:37:39]: I think it's cool. I think, I mean, I don't know exactly what they're doing, but...

**Swyx** [00:37:44]: Robots. Yeah.

**David** [00:37:46]: Well, I mean, that's a...

**Swyx** [00:37:47]: Yeah. What question would you ask? If we had them on, what would you ask them?

**David** [00:37:50]: Oh, I just want to understand what their overall strategy is going to be between now and when there's reliable stuff to be deployed. But honestly, I just don't know enough about it.

**Swyx** [00:37:57]: And if I told you, hey, fire your entire warehouse workforce and, you know, put robots in there, isn't that a strategy? Oh yeah.

**David** [00:38:04]: Yeah. Sorry. I'm not questioning whether they're doing smart things. I genuinely don't know what they're doing as much, but I think there's two things. One, I'm so excited for someone to train a foundation model of robots. It's just, I think it's just going to work. Like I will die on this hill, but I mean, like again, this whole time, like we've been on this podcast, we're just going to continually saying these models are basically behavioral cloners. Right. So let's go behavioral clone all this like robot behavior. Right. And then you figure out everything else you have to do in order to teach you how to solve a new problem. That's going to work. I'm super stoked for that. I think unlike what we're doing with helping humans with knowledge work, it just sounds like a more zero sum job replacement play. Right. And I'm personally less excited about that.

**Alessio** [00:38:46]: We had a Ken June from InBoo on the podcast. We asked her why people should go work there and not at Adept.

**Swyx** [00:38:52]: Oh, that's so funny.

**Alessio** [00:38:54]: Well, she said, you know, there's space for everybody in this market. We're all doing interesting work. And she said, they're really excited about building an operating system for agent. And for her, the biggest research thing was like getting models, better reasoning and planning for these agents. The reverse question to you, you know, why should people be excited to come work at Adept instead of InBoo? And maybe what are like the core research questions that people should be passionate about to have fun at Adept? Yeah.

**David** [00:39:22]: First off, I think that I'm sure you guys believe this too. The AI space to the extent there's an AI space and the AI agent space are both exactly as she likely said, I think colossal opportunities and people are just going to end up winning in different areas and a lot of companies are going to do well. So I really don't feel that zero something at all. I would say to like change the zero sum framing is why should you be at Adept? I think there's two huge reasons to be at Adept. I think one of them is everything we do is in the service of like useful agents. We're not a research lab. We do a lot of research in service of that goal, but we don't think about ourselves as like a classic research lab at all. And I think the second reason I work at Adept is if you believe that actually having customers and a reward signal from customers lets you build a GI faster, which we really believe, then you should come here. And I think the examples for why that's true is for example, our evaluations, they're not academic evals. They're not simulator evals. They're like, okay, we have a customer that really needs us to do these particular things. We can do some of them. These are the ones they want us to, we can't do them at all. We've turned those into evals, solve it, right? I think that's really cool. Like everybody knows a lot of these evals are like pretty saturated and the new ones that even are not saturated. You look at someone and you're like, is this actually useful? Right? I think that's a degree of practicality that really helps. Like we're equally excited about the same problems around reasoning and planning and generalization and all of this stuff. They're very grounded in actual needs right now, which is really cool.

**Swyx** [00:40:45]: Yeah. This has been a wonderful dive. You know, I wish we had more time, but I would just leave it kind of open to you. I think you have broad thoughts, you know, just about the agent space, but also just in general AI space. Any, any sort of rants or things that are just off of mind for you right now?

**David** [00:40:57]: Any rants?

**Swyx** [00:40:59]: Mining you for just general...

**David** [00:41:01]: Wow. Okay. So Amelia has already made the rant better than I have, but, but like not just, not just chatbots is like kind of rant one. And two is AI has really been the story of compute and compute plus data and ways in which you could change one for the other. And I think as much as our research community is really smart, we have made many, many advancements and that's going to continue to be important. But now I think the game is increasingly changing and the rapid industrialization era has begun. And I think we unfortunately have to embrace it.

**Swyx** [00:41:30]: Yep.

**Alessio** [00:41:31]: Excellent. Awesome, David. Thank you so much for your time.

**David** [00:41:34]: Cool. Thanks guys.