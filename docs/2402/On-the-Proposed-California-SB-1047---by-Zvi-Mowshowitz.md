<!--yml
category: 未分类
date: 2024-05-27 14:47:29
-->

# On the Proposed California SB 1047 - by Zvi Mowshowitz

> 来源：[https://thezvi.substack.com/p/on-the-proposed-california-sb-1047](https://thezvi.substack.com/p/on-the-proposed-california-sb-1047)

[California Senator Scott Wiener](https://twitter.com/Scott_Wiener/status/1755650108287578585) of San Francisco introduces [SB 1047](https://t.co/JWaOLP44Iu)  [to regulate AI](https://www.washingtonpost.com/technology/2024/02/08/california-legislation-artificial-intelligence-regulation/). I have put up a market on [how likely it is to become law](https://manifold.markets/ZviMowshowitz/will-california-bill-sb-1047-become).

> “If Congress at some point is able to pass a strong pro-innovation, pro-safety AI law, I’ll be the first to cheer that, but I’m not holding my breath,” Wiener said in an interview. “We need to get ahead of this so we maintain public trust in AI.”

Congress is certainly highly dysfunctional. I am still generally against California trying to act like it is the federal government, even when the cause is good, but I understand.

Can California effectively impose its will here?

On the biggest players, for now, presumably yes.

In the longer run, when things get actively dangerous, then my presumption is no.

There is a potential trap here. If we put our rules in a place where someone with enough upside can ignore them, and we never then pass anything in Congress.

So what does it do, according to the bill’s author?

> California Senator Scott Wiener: SB 1047 does a few things:
> 
> 1.  Establishes clear, predictable, common-sense safety standards for developers of the largest and most powerful AI systems. These standards apply only to the largest models, not startups.
>     
>     
> 2.  Establish CalCompute, a public AI cloud compute cluster. CalCompute will be a resource for researchers, startups, & community groups to fuel innovation in CA, bring diverse perspectives to bear on AI development, & secure our continued dominance in AI.
>     
>     
> 3.  prevent price discrimination & anticompetitive behavior
>     
>     
> 4.  institute know-your-customer requirements
>     
>     
> 5.  protect whistleblowers at large AI companies
>     
>     
> 
> @geoffreyhinton called SB 1047 “a very sensible approach” to balancing these needs. Leaders representing a broad swathe of the AI community have expressed support.
> 
> People are rightfully concerned that the immense power of AI models could present serious risks. For these models to succeed the way we need them to, users must trust that AI models are safe and aligned w/ core values. Fulfilling basic safety duties is a good place to start.
> 
> With AI, we have the opportunity to apply the hard lessons learned over the past two decades. Allowing social media to grow unchecked without first understanding the risks has had disastrous consequences, and we should take reasonable precautions this time around.

As usual, RTFC (Read the Card, or here [the bill](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=202320240SB1047)) applies.

Section 1 names the bill.

Section 2 says California is winning in AI ([see this song](https://www.youtube.com/watch?v=BATf_eUcb8M&ab_channel=ThePresidentsoftheUnitedStatesofAmerica)), AI has great potential but could do harm. A missed opportunity to mention existential risks.

Section 3 22602 offers definitions. I have some notes.

1.  Usual concerns with the broad definition of AI.

2.  Odd that ‘a model autonomously engaging in a sustained sequence of unsafe behavior’ only counts as an ‘AI safety incident’ if it is not ‘at the request of a user.’ If a user requests that, aren’t you supposed to ensure the model doesn’t do it? Sounds to me like a safety incident.

3.  Covered model is defined primarily via compute, not sure why this isn’t a ‘foundation’ model, I like the secondary extension clause: “The artificial intelligence model was trained using a quantity of computing power greater than 10^26 integer or floating-point operations in 2024, or a model that could reasonably be expected to have similar performance on benchmarks commonly used to quantify the performance of state-of-the-art foundation models, as determined by industry best practices and relevant standard setting organizations OR The artificial intelligence model has capability below the relevant threshold on a specific benchmark but is of otherwise similar general capability..”

4.  Critical harm is either mass casualties or 500 million in damage, or comparable.

5.  Full shutdown means full shutdown but only within your possession and control. So when we really need a full shutdown, this definition won’t work. The whole point of a shutdown is that it happens everywhere whether you control it or not.

6.  Open-source artificial intelligence model is defined to only include models that ‘may be freely modified and redistributed’ so that raises the question of whether that is legal or practical. Such definitions need to be practical, if I can do it illegally but can clearly still do it, that needs to count.

7.  Definition (s): [“Positive safety determination” means a determination, pursuant to subdivision (a) or (c) of Section 22603, with respect to a covered model that is not a derivative model that a developer can reasonably exclude the possibility that a covered model has a hazardous capability or may come close to possessing a hazardous capability when accounting for a reasonable margin for safety and the possibility of posttraining modifications.]

    1.  Very happy to see the mention of post-training modifications, which is later noted to include access to tools and data, so scaffolding explicitly counts.

Section 3 22603 (a) says that before you train a new non-derivative model, you need to determine whether you can make a positive safety determination.

I like that this happens before you start training. But of course, this raises the question of how you know how it will score on the benchmarks?

One thing I worry about is the concept that if you score below another model on various benchmarks, that this counts as a positive safety determination. There are at least four obvious failure modes for this.

1.  The developer might choose to sabotage performance against the benchmarks, either by excluding relevant data and training, or otherwise. Or, alternatively, a previous developer might have gamed the benchmarks, which happens all the time, such that all you have to do to score lower is to not game those benchmarks yourself.

2.  The model might have situational awareness, and choose to get a lower score. This could be various degrees of intentional on the part of the developers.

3.  The model might not adhere to your predictions or scaling laws. So perhaps you say it will score lower on benchmarks, but who is to say you are right?

4.  The benchmarks might simply not be good at measuring what we care about.

Similarly, it is good to make a safety determination before beginning training, but also if the model is worth training then you likely cannot actually know its safety in advance, especially since this is not only existential safety.

Section 3 22603 (b) covers what you must do if you cannot make the positive safety determination. Here are the main provisions:

1.  You must prevent unauthorized access.

2.  You must be capable of a full shutdown.

3.  You must implement all covered guidance. Okie dokie.

4.  You must implement a written and separate safety and security protocol, that provides ‘reasonable assurance’ that it would ensure the model will have safeguards that prevent critical harms. This has to include clear tests that verify if you have succeeded.

5.  You must say how you are going to do all that, how you would change how you are doing it, and what would trigger a shutdown.

6.  Provide a copy of your protocol and keep it updated.

You can then make a ‘positive safety determination’ after training and testing, subject to the safety protocol.

Section (d) says that if your model is ‘not subject to a positive safety determination,’ in order to deploy it (you can still deploy it at all?!) you need to implement ‘reasonable safeguards and requirements’ that allow you prevent harms and to trace any harms that happen. I worry this section is not taking such scenarios seriously. To not be subject to such determination, the model needs to be breaking new ground in capabilities, and you were unable to assure that it wouldn’t be dangerous. So what are these ‘reasonable safeguards and requirements’ that would make deploying it acceptable? Perhaps I am misunderstanding here.

Section (g) says safety incidents must be reported.

Section (h) says if your positive safety determination is unreasonable it does not count, and that to be reasonable you need to consider any risk that has already been identified elsewhere.

Overall, this seems like a good start, but I worry it has loopholes, and I worry that it is not thinking about the future scenarios where the models are potentially existentially dangerous, or might exhibit unanticipated capabilities or situational awareness and so on. There is still the DC-style ‘anticipate and check specific harm’ approach throughout.

Section 22604 is about KYC, a large computing cluster has to collect the information and check to see if customers are trying to train a covered model.

Section 22605 requires sellers of inference or a computing cluster to provide a transparent, uniform, publicly available price schedule, banning price discrimination, and bans ‘unlawful discrimination or noncompetitive activity in determining price or access.’

I always wonder about laws that say ‘you cannot do things that are already illegal,’ I mean I thought that was the whole point of them already being illegal.

I am not sure to what extent this rule has an impact in practice, and whether it effectively means that anyone selling such services has to be a kind of common carrier unable to pick who gets its limited services, and unable to make deals of any kind. I see the appeal, but also I see clear economic downsides to forcing this.

Section 22606 covers penalties. The fines are relatively limited in scope, the main relief is injunction against and possible deletion of the model. I worry in practice that there is not enough teeth here.

Section 2207 is whistleblower protections. Odd that this is necessary, one would think there would be such protections universally by now? There are no unexpectedly strong provisions here, only the normal stuff.

Section 4 11547.6 tasks the new Frontier Model Division with its official business, including collecting reports and issuing guidance.

Section 5 11547.7 is for the CalCompute public cloud computing cluster. This seems like a terrible idea, there is no reason for public involvement here, also there is no stated or allocated budget. Assuming it is small, it does not much matter.

Sections 6-9 are standard boilerplate disclaimers and rules.

What should we think about all that?

It seems like a good faith effort to put forward a helpful bill. It has a lot of good ideas in it. I believe it would be net helpful. In particular, it is structured such that if your model is not near the frontier, your burden here is very small.

My worry is that this has potential loopholes in various places, and does not yet strongly address the nature of the future more existential threats. If you want to ignore this law, you probably can.

But it seems like a good beginning, especially on dealing with relatively mundane but still potentially catastrophic threats, without imposing an undo burden on developers. This could then be built upon.

Ah, Tyler Cowen has a link on this and it’s… [California’s Effort to Strange AI](https://hyperdimensional.substack.com/p/californias-effort-to-strangle-ai?utm_source=post-email-title&publication_id=2244049&post_id=141516001&utm_campaign=email-post-title&isFreemail=false&r=3j06n&utm_medium=email).

Because of course it is. We do this every time. People keep saying ‘this law will ban satire’ or spreadsheets or pictures of cute puppies or whatever, based on what on its best day would be a maximalist anti-realist reading of the proposal, if it were enacted straight with no changes and everyone actually enforced it to the letter.

> Dean Ball: This week, California’s legislature introduced [SB 1047: The Safe and Secure Innovation for Frontier Artificial Intelligence Systems Act](https://leginfo.legislature.ca.gov/faces/billTextClient.xhtml?bill_id=202320240SB1047). The bill, introduced by State Senator Scott Wiener (liked by many, myself included, for his pro-housing stance), would create a sweeping regulatory regime for AI, apply the precautionary principle to all AI development, and effectively outlaw all new open source AI models—possibly throughout the United States.

This is a line pulled out whenever anyone proposes that AI be governed by any regulatory regime whatsoever even with zero teeth of any kind. When someone says that someone, somewhere might be legally required to write an email.

At least one of myself and Dean Ball is extremely mistaken about what this bill says.

The definition of covered model seems to me to be clearly intended to apply only to models that are effectively at the frontier of model capabilities.

Let’s look again at the exact definition:

> (1) The artificial intelligence model was trained using a quantity of computing power greater than 10^26 integer or floating-point operations in 2024, **or a model that could reasonably be expected to have similar performance on benchmarks commonly used to quantify the performance of state-of-the-art foundation models**, as determined by industry best practices and relevant standard setting organizations.
> 
> (2) **The artificial intelligence model has capability below the relevant threshold on a specific benchmark but is of otherwise similar general capability.**

That seems clear as day on what it means, and what it means is this:

1.  If your model is over 10^26 we assume it counts.

2.  If it isn’t, but it is as good as state-of-the-art current models, it counts.

3.  Being ‘as good as’ is a general capability thing, not hitting specific benchmarks.

Under this definition, if no one was actively gaming benchmarks, at most three existing models would plausibly qualify for this definition: GPT-4, Gemini Ultra and Claude. I am not even sure about Claude.

If the open source models are gaming the benchmarks so much that they end up looking like a handful of them are matching GPT-4 on benchmarks, then what can I say, maybe stop gaming the benchmarks?

Or point out quite reasonably that the real benchmark is user preference, and in those terms, you suck, so it is fine. Either way.

> But notice that this isn’t what the bill does. The bill applies to large models *and* to any models that reach the same performance regardless of the compute budget required to make them. This means that the bill applies to startups as well as large corporations.

Um, no, because the open model weights models do not remotely reach the performance level of OpenAI?

Maybe some will in the future.

But this very clearly does not ‘ban all open source.’ There are zero existing open model weights models that this bans.

There are a handful of companies that might plausibly have to worry about this in the future, if OpenAI doesn’t release GPT-5 for a while, but we’re talking Mistral and Meta, not small start-ups. And we’re talking about them exactly because they would be trying to fully play with the big boys in that scenario.

Bell is also wrong about the precautionary principle being imposed before training.

I do not see any such rule here. What I see is that if you cannot show that your model will definitely be safe before training, then you have to wait until after the training run to certify that it is safe.

In other words, this is an escape clause. Are we seriously objecting to that?

Then, if you also can’t certify that it is safe after the training run, then we talk precautions. But no one is saying you cannot train, unless I am missing something?

As usual, people such as Ball are imagining a standard of ‘my product could never be used to do harm’ that no one is trying to apply here in any way. That is why any model not at the frontier can automatically get a positive safety determination, which flies in the face of this theory. Then, if you are at the frontier, you have to obey industry standard safety procedures and let California know what procedures you are following. Woe is you. And of course, the moment someone else has a substantially better model, guess who is now positively safe?

The ‘covered guidance’ that Ball claims to be alarmed about does not mean ‘do everything any safety organization says and if they are contradictory you are banned.’ The law does not work that way. Here is what it actually says:

> (e) “Covered guidance” means any of the following:
> 
> (1) Applicable guidance issued by the National Institute of Standards and Technology and by the Frontier Model Division.
> 
> (2) Industry best practices, including relevant safety practices, precautions, or testing procedures undertaken by developers of comparable models, and any safety standards or best practices commonly or generally recognized by relevant experts in academia or the nonprofit sector.
> 
> (3) Applicable safety-enhancing standards set by standards setting organizations.

So what that means is, we will base our standards off an extension of NIST’s, and also we expect you to be liable to implement anything that is considered ‘industry best practice’ even if we did not include it in the requirements. But obviously it’s not going to be best practices if it is illegal. Then we have the third rule, which only counts ‘applicable’ standards. California will review them and decide what is applicable, so that is saying they will use outside help.

Also, note the term ‘non-derivative’ when talking about all the models. If you are a derivative model, then you are fine by default. And almost all models with open weights are derivative models, because of course that is the point, distillation and refinement rather than starting over all the time.

So here’s what the law would actually do, as far as I can tell:

1.  If your model is not projected to be state of the art level and it is not over the 10^26 limit no one has hit yet and no one except the big three are anywhere near, this law has only trivial impact upon you, it is a trivial amount of paperwork. Every other business in America and especially the state of California is jealous.

2.  If your model is a derivative of an existing model, you’re fine, that’s it.

3.  If your model you want to train is projected to be state of the art, but you can show it is safe before you even train it, good job, you’re golden.

4.  If your model is projected to be state of the art, and can’t show it is safe before training it, you can still train it as long as you don’t release it and you make sure it isn’t stolen or released by others. Then if you show it is safe or show it is not state of the art, you’re golden again.

5.  If your model is state of the art, and you train it and still don’t know if it is ‘safe,’ and by safe we do not mean ‘no one ever does anything wrong’ we mean things more like ‘no one ever causes 500 million dollars in damages or mass casualties,’ then you have to implement a series of safety protocols (regulatory requirements) to be determined by California, and you have to tell them what you are doing to ensure safety.

6.  You have to have to have abilities like ‘shut down AIs running on computers under my control’ and ‘plausibly prevent unauthorized people from accessing the model if they are not supposed to.’ Which does not even apply to copies of the program you no longer control. Is that is going to be a problem?

7.  You also have to report any ‘safety incidents’ that happen.

8.  Also some ‘pro-innovation’ stuff of unknown size and importance.

Not only does SB 1047 not attempt to ‘strangle AI,’ not only does it not attempt regulatory capture or target startups, it would do essentially nothing to anyone but a handful of companies unless they have active safety incidents. If there are active safety incidents, then we get to know about them, which could introduce liability concerns or publicity concerns, and that seems like the main downside? That people might learn about your failures and existing laws might sometimes apply?

The arguments against such rules often come from the implicit assumption that we enforce our laws as written, reliably and without discretion. Which we don’t. What would happen if, as Eliezer recently joked, the law actually worked the way critics of such regulations claim that it does? If every law was strictly enforced as written, with no common sense used, as they warn will happen? And someone our courts could handle the case loads involved? Everyone would be in jail within the week.

When people see proposals for treating AI slightly more like anything else, and subjecting it to remarkably ordinary regulation, with an explicit and deliberate effort to only target frontier models that are exclusively fully closed, and they say that this ‘bans open source’ what are they talking about?

They are saying that Open Model Weights Are Unsafe and Nothing Can Fix This, and we want to do things that are patently and obviously unsafe, so asking any form of ‘is this safe?’ and having an issue with the answer being ‘no’ is a ban on open model weights. Or, alternatively, they are saying that their business model and distribution plans are utterly incompatible with complying with any rules whatsoever, so we should never pass any, or they should be exempt from any rules.

The idea that this would “spell the end of America’s leadership in AI” is laughable. If you think America’s technology industry cannot stand a whiff of regulation, I mean, do they know anything about America or California? And have they seen the other guy? Have they seen American innovation across the board, almost entirely in places with rules orders of magnitude more stringent? This here is so insanely nothing.

But then, when did such critics let that stop them? It’s the same rhetoric every time, no matter what. And some people seem willing to amplify such voices, without asking whether their words make sense.

What would happen [if there was actually a wolf](https://www.storyarts.org/library/aesops/stories/boy.html)?