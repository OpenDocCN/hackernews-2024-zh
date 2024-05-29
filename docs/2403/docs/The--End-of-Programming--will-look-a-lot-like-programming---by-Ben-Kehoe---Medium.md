<!--yml
category: 未分类
date: 2024-05-27 14:30:31
-->

# The “End of Programming” will look a lot like programming | by Ben Kehoe | Medium

> 来源：[https://ben11kehoe.medium.com/the-end-of-programming-will-look-a-lot-like-programming-8b877c8efef8](https://ben11kehoe.medium.com/the-end-of-programming-will-look-a-lot-like-programming-8b877c8efef8)

# The “End of Programming” will look a lot like programming

Communications of the ACM has [a new article titled “The End of Programming”](https://m-cacm.acm.org/magazines/2023/1/267976-the-end-of-programming/fulltext) by Matt Welsh. It posits that traditional programs “will be replaced by AI systems that are *trained* rather than *programmed*” (emphasis in the original). Welsh is “the CEO and co-founder of Fixie.ai, a recently founded startup developing AI capabilities to support software development teams”.

I’m generally skeptical of these broad claims. More than anything, I think when people imagine integrating AI (and especially LLMs) into software development (or any other process), they tend to be overly optimistic. But I want to focus on something in particular here. Welsh says “[w]e are rapidly moving toward a world where the fundamental building blocks of computation are temperamental, mysterious, adaptive agents.”

I think this is the least likely outcome. If you had to describe characteristics that drive users away from products, “temperamental and mysterious” system behavior would be high on that list (we usually just call it “buggy”). *Especially* because in this world, the people behind the product would likely say, “Ah, sorry, the AI did that. We’ll see if we can explain to it what needs to be fixed and maybe it’ll get fixed if we ask it the right thing”.

In general, a lot of the AI takes I see assert that AI will be able to assume the entire *responsibility* for a given task for a person, and implicitly assume that the person’s *accountability* for the task will just sort of…evaporate? Like, if the AI got it wrong, it’s not your fault? But if you have no real way to ensure the task is correctly performed, they’re probably going to find someone else to accomplish that task after it’s failed a few times.

So in a world where software still has to actually do the thing required of it, let’s imagine what it would look like if we no longer needed human software developers. A human is acting as a product manager, dictating their business requirements to an AI, which then constructs software. Let’s assume that the resulting software that is largely functional (that is, free of low-level bugs) — I think this is a long ways off, and there is lots to say about what it will look like until then, but that’s a separate discussion.

If you’ve worked as a software developer, you know that business requirements often come as vague, ill-defined, even contradictory ideas written down in ambiguous language. The primary question about this AI-only software development is, how will it make software that implements what the product manager *intends* the software to do?

I think there are two general directions, which are not mutually exclusive.

The first is that the AI has to ask the product manager about every individual choice and ambiguity. It has to do this because it is good enough to know what the choices and ambiguity are, but not good enough to consistently guess the correct answer. This back-and-forth will start in plain language, and take up a lot of time for the product manager. Over time, the AI’s designers will start offering shortcuts that allow the input requirements to mean specific things when framed a certain way, so the product manager can make their choice clear from the outset. So we’ve got a method for expressing system behavior with formal guarantees. That is, we’ve invented a new programming language. At this point, the product manager is now a software developer.

The other is that the AI *is* good enough to consistently correctly guess the right answer to choices and ambiguity in the requirements *and* good enough to know when it doesn’t have confidence it can guess correctly. To do this requires an enormous amount of human cultural knowledge and probably a high degree of knowledge of the specific person acting as the product manager. The AI is doing the work of translating the business requirements into formal system behavior requirements, as well as implementing them. At this point, the AI is now a software developer.

You could argue the second is the same as Welsh’s vision of “the end of programming”, as you’re still working without a formal language. I’m saying it’s different, because it’s not “temperamental and mysterious” any more than the software developers (the ones you *like* working with) are — it’s reliable and consistent.

I think the second direction, AI-as-software-developer, is quite a ways off, specifically because cultural context and self-awareness are hard things. I doubt you need AGI to get it, but it seems like it would be a good chunk of the way there. So if you’re bullish on AGI, you can hope we’ll get it sooner rather than later.

I’d love to see the first direction taken explicitly. One of my problems with GitHub Copilot (and similar systems) is that you still own the resulting code, and it does not provide any path to gaining confidence that it has given you a correct implementation, despite you owning the correctness of the code it generates (this also warrants its own article, I think). I’d love for it to identify common patterns and formalize them, such that developers could use some shorthand to express that complicated logic and have confidence they are getting what they expect. Maybe that can be done with the existing system, or maybe it’s an entirely new Copilot-oriented programming language.

I think my main takeaway here is that, when looking at claims AI is going to automate some process, look for what the really hard, inherent complexity of that process is, and whether the process would be successful if a large degree of (new) uncertainty was injected into that complexity. For software development, I think the answer is no. That doesn’t mean AI won’t be successful, it just means we need to look deeper to refine what and how (and when) automation will play a role in it.