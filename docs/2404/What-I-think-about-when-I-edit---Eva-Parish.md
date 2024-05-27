<!--yml
category: 未分类
date: 2024-05-27 12:59:01
-->

# What I think about when I edit — Eva Parish

> 来源：[https://evaparish.com/blog/how-i-edit](https://evaparish.com/blog/how-i-edit)

I’m often asked to edit friends’ or coworkers’ writing, anything from emails to short stories to documentation. Recently, someone asked me *how* I edit. What am I looking for? How do I know what changes to make? That made me stop and think about what I’ve been doing semi-instinctually.

In this post, I want to distill the major points of editing that I believe in but haven’t spelled out until now. Much of this advice applies across genres. Personally, I wrote academic papers in college, write poetry and short fiction in my free time, and write technical documentation for work, and I’ve applied the same basic editing techniques to all.

**I also think that different genres inform each other.** There are principles I’ve taken from fiction writing that make my technical language even clearer, and learning just how much people skim when reading technical documentation has improved how I format and write things like emails.

Here are my recommendations:

* * *

## Decide what you’re actually saying

Before you ever get to editing at the sentence level, you have to determine whether you’ve said what you meant to say.

I recommend writing a *preamble**, just for your own use, on everything you write. Take a few minutes and consider what you’re trying to say. **What is your main point? Who are you writing for?** Then *actually write this information down* at the top of your document (or notebook, or cafe napkin) so it’s there staring you in the face as you work. As you write, and as you edit, you can compare what you have on the page to what you set out to do.

It also forces you to think about what your message actually is. Say you want to write a blog post: if you can’t summarize the point you want to make in a sentence or two, how are you going to write a coherent post?

*I took this name from something we do in my creative writing classes at the [Writers Studio](https://www.writerstudio.com/about/) in New York. As part of every assignment, we have to write a preamble: a few sentences at the top of the page mentioning what we intended to do, including the kind of narrator, tone, and mood. When other students critique your work in class, they’re holding it up to this model and evaluating whether you achieved your aim, focusing on *craft*, not whether they like your main character.

## Repeat yourself (within reason)

Even if you think you’ve made your point very clearly, it’s worth restating it at the beginning and end of what you’re writing to make sure the reader gets it.

This principle applies across most genres. In documentation, a good tutorial will have a brief introduction to what you’re going to do, then the actual procedure, and finally a way for you to verify that you’ve done the thing correctly. In a blog post, you should introduce what you’re going to discuss in the post, then actually do that, and have a short summary at the end. And so on.

“Repeat yourself” applies at the language level too. One of the best writing tips I've ever gotten was to avoid using [demonstrative pronouns](https://www.gingersoftware.com/content/grammar-rules/demonstrative-pronouns/). Instead of saying “this” or “that,” you should add a noun to spell out exactly what you’re referring to, *even if you’ve just mentioned it*.

> *Example:* We only have two boxes left. To solve **this**, we should order more.

> *Revision:* We only have two boxes left. To solve this shortage, we should order more.

> *Example:* Click next and enter your credentials when prompted. **That** will take you to the home screen.

> *Revision:* Click next and enter your credentials when prompted. Successfully authenticating will take you to the home screen.

This duplication can feel repetitive when you’re writing it, but it won’t feel repetitive to your reader—it’ll make your writing clearer and easier to follow.

In summary: when editing, **look for ways that you can restate your point, clarify, or provide closure for the reader**.

## Simplify

When I edit someone else’s work, my number one quest is to remove words. Eliminate the fluff. Are there constructions that can be shortened? Any extraneous words that don’t add to the meaning of the sentence?

> *Example:* You will need to run this script.

> *Revision:* Run this script.

> *Example:* You can aid in readability by making sure that the names of things properly communicate what they do.

> *Revision:* Make sure the names of things communicate what they do.

**Unless you have a very specific reason not to, strive to get to the point as quickly as possible.** Don’t bury your meaning in excess words and flowery constructions.

Other ways to simplify:

### “You should”/“You can”

When writing instructions, anywhere you say “*You should X*” or “*You can X,*” replace it with the [imperative mood of the verb](http://www.cws.illinois.edu/workshop/writers/verbmood/).

> *Example:* You should save the file to your home directory.

> *Revision:* Save the file to your home directory.

This change eliminates a couple words, making the sentence easier to read, and brings the reader straight to the point.

### “Of” and “for” clauses

Instead of using constructions with “of” or “for,” rewrite the sentence to put more information before the noun. This ordering makes the sentence more efficient.

> *Example:* The manager of the team responsible for marketing

> *Revision:* The marketing team’s manager

In the rearranged version above, the reader can more quickly grasp what you mean, instead of having to revise her understanding after each clause.

### Split it up

Break up long sentences into multiple shorter sentences.

> *Example:* Due to the Acme project which just completed a major milestone of having all non-staging servers running in the Foobaz environment, we now see build times of sub-10 minutes which were previously taking over an hour when running with the XYZ plan.

> *Revision:* The Acme project recently completed a major milestone: all non-staging servers are now running in the Foobaz environment. Builds now take fewer than 10 minutes to complete. This change is a significant improvement, as builds on the XYZ plan previously took over an hour to complete.

Also, break up sentences by adding commas [where appropriate](https://owl.purdue.edu/owl/general_writing/punctuation/commas/extended_rules_for_commas.html). For example, I’ve noticed a trend towards people dropping commas after subordinate clauses. I always add them back when I edit:

> *Example:* If you’re looking for me I’ll be in my office.

> *Revision:* If you’re looking for me, I’ll be in my office.

> *Example:* Due to the fog our flight was delayed.

> *Revision:* Due to the fog, our flight was delayed.

I suspect that this is because, when we speak colloquially, we don’t pause at that point in the sentence. However, grammatically, [you need a comma after a subordinate (or dependent) clause when it comes at the start of a sentence](https://owl.purdue.edu/owl/general_writing/punctuation/independent_and_dependent_clauses/index.html). Besides being “correct,” the comma helps the reader pause and process what they’ve just read before moving on to the rest of the sentence. Using commas after subordinate clauses improves reader comprehension.

## Eliminate passive voice

You’ve heard this advice before. But you should understand *why* you shouldn’t use passive voice in your writing. It’s not just “bad style.”

**Passive voice obscures who or what is performing the action.** Rewriting a passive construction to be active *almost always* makes what you’re saying clearer and makes the sentence easier to read, because your reader can attribute the action to the right person or thing.

> *Example:* The fire alarm was pulled and the building was evacuated.

> *Revision:* The fire marshal pulled the alarm and the employees evacuated the building.

> *Example:* Millions of dollars were embezzled from the company.

> *Revision:* Two executives embezzled millions of dollars from the company.

It may even help *you* understand what you’re saying better. If you’re describing a system you built, and you say “An alert is triggered and the job is started”—do you know *how* those things happen? Which service triggers the alert? Which component is responsible for executing the job? In rewriting, you may realize that something doesn’t work as you expected, or that you don’t know how it works.

In technical documentation, you lose precision when you don’t attribute the action to someone or something. And in all writing, refining your language refines your understanding of the world.

## Don’t use adverbs

This aversion to adverbs is one of the principles I’ve taken from fiction-writing.

**You can almost always replace an adverb with a better, more specific verb, or describe what you mean instead.** Being more specific is especially key in fiction, but I believe in stripping out adverbs in all types of writing.

There is nothing *inherently* wrong with adverbs. They are just part of a category of things that I believe are lazy in writing. When I say “He laughed loudly,” I’m relying on my reader somehow intuiting the precise volume of his laughter. “Loudly” could mean a million things, but what I really had in mind is that “He laughed with the kind of booming abandon that made the whole restaurant turn around and look.”

People also often use adverbs as a hedge: “Basically, it's this.” “Essentially, this is what I’m saying.” Is it, or isn’t it? Remove the adverb and commit to saying whatever you’re saying.

## Don’t assume knowledge

It’s easy to fall into this trap when you’re writing about something you know well: you forget to consider what *you* know that your audience *doesn’t*. You don’t take a step back and provide relevant context. Imagine how much more pleasant it would be to read emails, documentation, etc., if people actually spelled out those TLAs (three-letter acronyms)!

Let’s start with an example, and look at a few ways we can improve it.

> *Example:* This chart shows the TTFB for our website over the past week.

To some people, this sentence makes perfect sense. To many people… not so much.

*   *Spell out acronyms on first use.* Any time you introduce an acronym or an initialism in a document, spell out what it means and put the acronym in parentheses. Thereafter, you can use the acronym by itself.

    > *Revision 1:* This chart shows the time to first byte (TTFB) metric for our website over the past week.

    You might think it’s obvious what an acronym means, but a new reader may not.

By now, your readers are with you, and they’re ready to proceed, feeling confident that they have an idea of what you’re talking about.

## Be aware of your tone

Know what kind of tone you’re going for, and be consistent. You can be colloquial or formal, but not both.

> *Example:* We were really into this new framework that we found for like a minute or two, but the metrics captured by the system do not correspond precisely enough to our investigative goals to be useful.

> *Revision:* We were initially enthusiastic about the X framework, but we found that it did not capture the metrics we were looking for.

The original sentence starts out *very* colloquial and then morphs into formal, almost academic language. In the best case, you’ll confuse your readers about why you switched. In the worst case, you’ll completely distract them from what you’re saying.

## Avoid jargon and cliches

In the business world, [jargon](https://www.grantthornton.com/-/media/content-page-files/press-releases/2018/Jargon-Index-2018.ashx?la=en) means things like “deep dive“ and “low-hanging fruit”. Elsewhere, we love to use cliches. Especially baseball metaphors, for some reason: “step up to the plate,” “hit it out of the park,” “take a swing at it.”

It will always be better and clearer when you say exactly what you mean. **Using jargon is lazy, and it assumes that the reader is part of the in-group that uses that jargon** (see also: Don’t assume knowledge, above). It can be difficult for non-native English speakers (or non-Americans, when it comes to baseball) to follow your writing when you use jargon and cliches.

> *Example:* tl;dr, if you can hack something together by EOD, that would be great.

> *Revision:* Can you deliver a prototype by the end of today?

The original sentence has incomprehensible acronyms and tech slang, and doesn’t even sound like a request. The second one is straightforward, and asks for what the writer needs and by when.

## Make use of whitespace

Whitespace is key for technical documentation but can also be used to great effect in blog posts, emails, and elsewhere. It’s hard for people to read long paragraphs, especially on a computer screen. They will zone out. Ensure that readers stay with you by visually breaking up the page and making your key points easy to identify.

A few suggestions:

*   Break up long paragraphs into multiple shorter ones.
*   Use useful **subheadings** to give your document some structure and allow readers to skip ahead to the section they’re interested in.
*   Use **lists** where relevant, because it’s easier to read a bulleted list of items than to read a paragraph with the same information.
    *   When you need to convey large amounts of information (for example, in reference documentation), **tables** are even better than lists.
*   Use **bold** so that readers who skim (i.e., everyone) will still pick out your main points. (For an example, see the body of this post.)

* * *

## Conclusion

To simplify what you’ve just read, my editing philosophy can be reduced to two tenets:

*   *Say exactly what you mean*, which means not relying on adverbs, jargon, cliches, or hedges, and
*   *Take out all unnecessary words*.

Keeping these two tenets in mind will help you in your own writing, and will give you a framework for evaluating other people’s writing. Over time, as you practice, you’ll develop your own style and preferences. You may end up diverging from some of my recommendations, and that’s great, as long as you know *why* you’re doing so. **The point of editing is to think about how you’re using language and to make choices that suit the message you want to deliver**, *not* to unquestioningly follow rules—mine or anyone else’s.