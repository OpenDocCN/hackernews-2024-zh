<!--yml
category: 未分类
date: 2024-05-27 15:12:48
-->

# Nobody Cares About Your Git History

> 来源：[https://spin.atomicobject.com/git-history/](https://spin.atomicobject.com/git-history/)

The topic of Git history, rebasing, [squashing](https://spin.atomicobject.com/squash-merge/), and commit messaging often surfaces, igniting debates as fiery as those over Vim vs. Emacs, tabs vs. spaces, or pineapple on pizza. For some, Git history is the unsung hero of software teams; for others, it’s a persistent thorn in the side. Opinions vary widely, and those who have them often hold them with conviction (it’s me, hi, I’m the problem).

This post isn’t about preaching or gatekeeping; it’s about sharing thoughts and sparking a dialogue. I don’t claim to have all the answers, nor do I consider myself the ultimate authority on this topic. If you think I’m off the mark, I welcome your corrections in the comments. After all, I might just be on the wrong side of history (pun intended).

## Decoding “Clean Git History”

The phrase “clean Git history” commonly conjures up images of a linear, immaculate sequence of commits—each one polished and purposeful. Achieving this often involves rebasing or squashing feature branches to create a simple-to-follow narrative. It may also include tagging, releases, and other Git features. But here’s a thought: is this meticulous curation necessary, or even beneficial?

Mastering your tools, [especially Git](https://spin.atomicobject.com/favorite-git-commands/), is something I firmly believe in. However, it’s important to acknowledge that we’re all at different stages in our development journey. For those just starting out, and already drinking from the firehose of professional software development, a complex Git process could be more daunting and dangerous than helpful.

Opinions on Git usage vary: some see code as the ultimate truth and pay less attention to Git history, while others craft and rewrite their commits like poetry. I tend to rely on Git as a tool for sharing code and tracking changes, and I do recognize the value in the history. This post is for everyone, no matter where you stand.

## Why Rebase?

So, who should rebase, and why?

### Large organizations and open-source projects

These groups often manage complex codebases with numerous contributors, where a neater history can assist in navigating changes and understanding the evolution of the code. Rebasing helps maintain a readable and coherent history, which is more important when many people are simultaneously working on different features or fixes.

*   **Why:** A cleaner commit history in these environments aids in tracking changes and features over time. Individual commits are meaningful and self-contained, which makes them easier to review, test, and potentially revert without affecting unrelated changes. It also simplifies the task of pinpointing where bugs were introduced and facilitates the process of reverting to stable states. Fewer, but more thoughtfully crafted commits will reduce noise for the rest of the contributors.
*   **Addendum**: This may especially be true for a project utilizing a **monorepo** strategy, where a number of different teams and contributors are committing to the same codebase. Maintaining a clean history can be useful to reduce the amount of noise that other teams see.

### Small, highly experienced teams

Teams that are well-versed in Git and showcase a number too big to fit on their “days since last rebase accident” sign might find rebasing a useful tool to keep their history concise. They’re less likely to encounter the pitfalls that can come with a complex rebase, and they have the necessary skills to resolve conflicts and understand the implications of rewriting history.

*   **Why:** For these teams, rebasing can be part of a routine to ensure that the main branch remains clean and that features are integrated seamlessly. It’s also a way to condense work into logical chunks that make sense to others and to their future selves.

### Individual contributors working on isolated features

When working alone on a feature or in a branch that doesn’t have frequent interactions with others, rebasing can be a way to tidy up before merging back into main. It allows the contributor to present their work in a clear and structured manner, which can be helpful for some code reviewers.

*   **Why:** The rebase here serves to streamline the individual’s contributions into a narrative that’s easier for others to follow. It can help to highlight the development process of the feature or fix, showing a clear progression from start to finish and making it easier for reviewers to understand the rationale behind each change.

## The Upside of a Clean Commit History

Let’s look at the pros and cons of each approach. First, the upside of a clean commit history.

### **A streamlined log**

An orderly history can be easier to read and understand. Single, self-contained commits for feature branches can quickly show when things were introduced.

**Commit-centric PR reviews:** Some find value in this method, though it’s not universally applicable.

*   **My perspective**: Reviewing PRs commit-by-commit can lead to scrutinizing outdated code. While reviewing smaller changes might seem helpful, the lack of context of the full PR can diminish its utility. I don’t recommend this approach.

### **Less WIP clutter**

Carefully rewritten commits mean fewer half-baked or broken changes, avoiding the frustration of build errors and test failures when checking out a teammate’s code.

### **Easier to revert changes**

While not the only way to manage this (see: feature flags), being able to quickly revert a commit is easier than going through the codebase and manually commenting things out. Rolling back a whole release can also be an option, but there are times when it’s not tenable.

## The Downside of Over-Curation

So, what’s the downside?

### Loss of detail

The development process is inherently messy and non-linear. Squashing commits can remove the context around why certain decisions were made, which can be crucial for understanding the rationale behind the current state of the code when revisiting it in the future.

### **Finding old code**

At times, the code that gets removed during the development process can be as informative as the code that remains. When commits are squashed, the ability to retrieve and examine these discarded snippets is lost, which can be a disadvantage when trying to understand past decisions or resurrect a shelved idea. If you’ve written and removed code in a single branch, it can be nearly impossible to retrieve those changes after a squash or rebase.

### **Tracking the author**

In collaborative environments, understanding who wrote what can be essential for seeking clarification or addressing issues. Squashing commits often attributes all changes to the person performing the squash, removing the granularity of authorship and complicating accountability. Simply put: `git blame` is not effective when commits are rewritten.

### **Git bisect to the rescue**

Bisecting is a lifesaver when isolating issues, and it’s far easier with smaller commits than with a huge squashed feature branch. It’s quick to churn through even thousands of commits to find the offending one, but if it only turns up a squashed feature branch, it will take more time to comb through every line of code.

### **Potential for errors**

Rebasing, especially when done across branches that are widely divergent, can introduce errors that are difficult to trace. A single misstep during conflict resolution can lead to subtle bugs that may not be immediately apparent, creating a hidden cost that manifests later. A botched rebase can wreak havoc, leading to lost work at worst, or wasted time at best. I know plenty of horror stories of colleagues trying to untangle codebases that have been afflicted by the rebase bug. This isn’t to say that this doesn’t occur via other Git usage —- conflicts are a fact of life — but it can be particularly challenging to work through with rebasing. This is especially evident with folks who are new to Git, as the ramifications of rewriting history aren’t as obvious.

### **When, not if**

At some point, even the most experienced of Git users will need to deal with a botched rebase. This is a frustrating experience, a tremendous time-suck, and a risky entry point for new bugs. New errors may be introduced as old commits are rewritten, and finding bugs can be complicated. While a merge may also introduce bugs, there is a single point in the commit history to mark that. Bugs could be introduced at any time in a chain of rewritten commits.

### **Time consumption**

Crafting the perfect history is a significant time sink with debatable benefits. Developers may spend hours resolving conflicts during rebasing, time that could otherwise be spent writing new features or fixing bugs.

### **Perils of perfectionism**

In striving for a pristine commit history, developers might hesitate to commit frequently or push their changes, which can lead to significant portions of work existing only on local machines. This not only increases the risk of data loss but also delays collaboration and feedback. I’d rather a teammate push up a broken WIP commit as they rush out the door than nothing at all.

### **The value of commit messages**

While well-crafted commit messages are helpful, an overemphasis on their importance can detract from the primary goal of version control: collaborating with a team. Developers might spend more time managing their Git artifacts than coding, or worse, avoid committing frequently to avoid having to write messages for “trivial” changes.

### **They’re just lies, after all**

Rewriting old history, especially with rebasing, but even with squashing, is simply that: rewriting history. It’s not a true representation of when changes were made, who made them, how the process went, and most importantly, why they were made. We’re lying to ourselves by saying a commit was created today when it was created days or weeks ago. Some people may choose to do this because it presents a better picture of how something went: no WIP commits, no broken builds, no test failures, and no mistakes taken to arrive at a solution. It’s simply not the truth, but it looks nice on paper.

### **One rebase, all rebase**

Rebasing isn’t a solo activity; it has team-wide implications. When one team member starts rebasing and force-pushing, it requires everyone on the team to adapt to this workflow. This can be disruptive, especially if team members are at different skill levels with Git. Newer members may struggle with the complexities of rebasing, which can lead to errors and frustration. I advocate for managing your Git workflow as you see fit *without* imposing this standard on everyone.

### **Why this matters**

Rebasing rewrites history, and when that rewritten history is shared, it can cause confusion and conflicts for others who have the old history in their local repositories. Team members must then spend additional time learning advanced Git commands to safely synchronize their work, which can be a steep learning curve for some and may introduce even more potential errors in that process.

### **The ripple effect**

Even a well-intentioned rebase can create a ripple effect requiring others to halt their work to resolve unexpected issues. This can lead to a significant productivity hit for the team as a whole, as members must pause feature development to address problems introduced by history rewrites.

## **Utilizing Git History Correctly**

I seldom pore over commit messages, relying instead on tools like `git grep` and `git log -p` for understanding code changes. Commit messages don’t tell the whole story; that is the purpose of the codebase itself. The story of the code is best told first-hand, when commits are introduced, not as an archaeologist or anthropologist going back in time and piecing things together. The original commit message, timestamp, author, and order are all more effective when left untampered.

### **Respecting diverse workflows**

Best practices should be adaptable to fit various team and project dynamics. When in doubt, opt for the approach that’s safer, quicker, and preserves more data.

### **Avoiding disruption**

Frequent context switching between coding and crafting detailed commit messages can hamper productivity. Sometimes it’s best to note it’s a work-in-progress (WIP) and move on. Is it worth revisiting those and fixing them in the future?

### **Embracing change**

As requirements and knowledge evolve, so does our code. Time spent polishing history is pointless if those changes are later altered or discarded.

### **Finding the right process**

Too many processes can get in the way of actually building software. Rebasing can add unnecessary steps to completing a task, and I hesitate to add another one to a constantly growing list in modern development.

Organize and rebase your commits if you like, but consider your team’s preferences, the team-wide implications, and potential drawbacks.

## **The Ultimate Purpose of Git**

How you choose to use Git is up to you, but at its core, it is a tool to share code and teach changes with teammates, not a holy text of changelogs.

Rewriting commit history demands a substantial investment of time, adds complexity and risk, obscures the development story, and offers limited benefit. It’s just not worth it.

While it’s not an apples-to-apples comparison, consider this: we change source code via minification and obfuscation at the final stages of the release pipeline to maintain readability and debuggability during the development phase. Rewriting commit history obfuscates and minifies our source code at the first stage of that same pipeline, which obscures the natural, iterative problem-solving that unfolds as the code evolves.

The appeal of a pristine Git history is understandable, yet I’ve often found more practical value in a detailed, albeit cluttered, commit log. Small commits tracking all changes over time and accurate contributor logs are more beneficial than lumping them into one. Time spent managing this is often wasted effort. It can be put to better use.

Finally, my experiences are mostly with smaller teams focused on delivering functional code rather than maintaining spotless commit narratives. As I haven’t worked in an organization or on an open-source project where this is more important, it’s possible that I just haven’t developed the necessary skills, and I’m afraid of change. If you’ve struck a balance between meticulous Git habits and efficiency, I’m all ears.

I hope this post has provided some food for thought, whether you’re a Git purist or a pragmatist. I would love to continue the conversation in the comments. Please tell me why I’m wrong!