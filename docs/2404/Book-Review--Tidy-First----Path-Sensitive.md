<!--yml
category: 未分类
date: 2024-05-27 12:48:00
-->

# Book Review: Tidy First? | Path-Sensitive

> 来源：[https://www.pathsensitive.com/2024/04/book-review-tidy-first.html](https://www.pathsensitive.com/2024/04/book-review-tidy-first.html)

# Book Review: Tidy First?

As you’re working day-to-day, every so often it’s nice to take a step back and improve things.

Some changes require having a strong understanding of how things are done and why, but can lead to huge benefits. Think: a chef finding a better way to brine poultry to preserve flavor, or finding a baking schedule that lets most customers get bread fresh out of the oven.

Other changes lead to a slightly pleasant environment, but can be knocked off without much thought when you walk by. Think: straightening the chairs in the dining area, or throwing away the cracked plates.

Refactoring a codebase can be intense and scary. Kent Beck’s ”Tidy First?” suggests we reduce the barriers by starting small. It offers a catalogue of “tidyings”: small, uncontroversial improvements you can knock out right before you code. Sorta like the software equivalent of straightening the chairs.

And then it builds on that. It talks about whether you should straighten the chairs before or after you cook. About whether you should straighten the chairs and clean the cupboard in one batch or two. About the theory of why having straightened chairs leads to a more pleasant environment and improved business.

By the end of it, you may have forgotten that deeper improvements even exist.

## Shorten first?

This book has one cardinal sin: not understanding when more details are helpful vs. superfluous.

On the side of harmful brevity, there’s Chapter 12 about when and how to extract helper methods. I think everything in the chapter is correct and good advice, but it’s also vague enough that I can’t see it being useful to anyone who doesn’t already understand it. And one of the core tidyings is described by just a few lines of text and example code featuring functions foo and foo_body, so that I myself had trouble understanding what he’s recommending, especially since it mostly applies to just a couple of languages.

But on the side of verbosity, there are a lot of paragraphs like this:

> Behavior creates value. Rather than having to calculate a bunch of numbers by hand, the computer can calculate millions of them every second. Turns out people will pay not to have to calculate numbers by hand. If running the software costs $1 in electricity and you can charge folks $10 to run it on their behalf, then you have a business.
> 
> (Chapter 23)

You know what doesn’t create value? Spending 4 sentences to say that automating stuff is worth money.

This pattern — giving extra detail where it’s obvious, omitting detail where it would be helpful — repeats throughout the book. And so, even though “Tidy First?” is pretty short at around 100 pages, I feel like it’s constantly trying to fill space.

Sometimes the space issues go deeper. Like, I’m reminded in parts of a [story](https://web.archive.org/web/20150407223525/http://squid314.livejournal.com/297579.html) I read about a “computer skills” class taught in a 3rd-world classroom without computers. Think: a lecturer stands in front of a chalkboard, and drones, “When you click the Start button, a menu would appear. Then you would mouse over…”

Chapter 17, “Chaining,” is like this. It’s supposed to be a list of examples of how doing one tidying might lead to another, like how you might see a new opportunity to reorder code after deleting a branch that never runs. But it’s tough to follow and impossible to remember if you haven’t tried tidying and seen this play out. And if you have, then your own experience will show you far better than this chapter could.

The actual list of tidyings is fine. Most are pretty uncontroversial small improvements. He says some things about reordering code that I find overly simplistic, and generally a smaller improvement than adding [good code sections](https://www.pathsensitive.com/2023/12/should-you-split-that-file.html). He repeats the typical line of “Delete code that’s not used instead of commenting it out, because you can always recover it from VCS.” (That’s a view that I’ve [controversially] started to turn against, for the same reason that “you can recover it from backups” is not a compelling reason to delete files currently not in use.) He gives some examples of ways that programmers, even after being taught in intro classes not to use magic numbers, still litter their code with constants like 404\. But I wish he’d tell Python programmers to stop designing APIs where you write string constants like “r--” and “bs” to denote that your scatterplot should use red dashes and blue squares. His catalogue lists the changes of “Add a comment that you wish you had when reading the code” and “Remove a comment that just says what the code says.” But I’d rather have a deeper discussion that offers a bigger toolbox of how to improve comments. (The discussion in [A Philosophy of Software Design](https://www.pathsensitive.com/2018/10/book-review-philosophy-of-software.html) is my favorite part of that book.)

## Connect the ideas last?

On the whole, Beck seeks to provide a repertoire of easy wins you can earn when you drive by some code. I think he succeeds in that goal, although it’s a rather impoverished repertoire. (E.g.: they can’t solve any of the [refactoring challenges](https://twitter.com/jimmykoppel/status/1742794903216689212) I’ve been running lately.) But if you’re someone who lacks a repertoire at all, I can see them as great training wheels.

But then he spends 2/3 of the book talking about how to schedule time for tidying and how reducing coupling reduces the cost of change and the benefits of making decisions reversible and other stuff that’s not very relevant to deleting useless comments and adding blank lines to break up code. It feels a bit like he wrote a book on these shallow tidyings, and then also wrote a book on deep refactoring, and then accidentally got the chapters mixed up.

And then when I asked him about this, he actually said that I’m right but should wait for his next book: “I'm giving concepts bottom up--tidyings--and top-down--theory. I *will* meet in the middle, I promise.” And he’s pursuing this strategy because: “I think having people practice designing consciously for a year is good prep for being able to understand the next layer.”

Maybe he envisions his readers growing up with the series as it matures from microscopic code improvements to aligning a team? Perhaps I have a higher opinion of other people than he does, but I don’t understand how he can say that while also marketing it to senior engineers.

If you read this book, I recommend just skimming the first part with the list of tidyings and figuring out what everything is. He does say some stuff later about how to send tidyings for code review and why they provide value, but I’d expect most readers will be able to produce the same insights themselves after trying them a few times.

But really, the interesting contents could be shortened to a list of Tweets. And I’ve done so [here](https://twitter.com/jimmykoppel/status/1770028266017239059).

Kent Beck is a rather renowned software engineer, having created Test-Driven Development, Extreme Programming, and jUnit. His Substack has many pieces I enjoyed a lot, such as his posts on [bitemporality](https://tidyfirst.substack.com/p/eventual-business-consistency) and [measuring developer productivity](https://tidyfirst.substack.com/p/measuring-developer-productivity). In our interactions, he’s been an exemplar of a gentleman, and also writes about the insights he’s learned that’s made him so. Overall, he has a lot of interesting things to say, both about software engineering and about dealing with people and emotions. But you won’t find them in this book.

So, if you’re considering buying this book, just purchase a subscription to his Substack instead. He’ll earn more and you’ll learn more. The only one who loses is the publisher.

*Thanks to [Hillel Wayne](https://www.hillelwayne.com/) for comments on earlier drafts of this post.*