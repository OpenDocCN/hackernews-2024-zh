<!--yml
category: 未分类
date: 2024-05-29 12:45:55
-->

# Some Git poll results

> 来源：[https://jvns.ca/blog/2024/03/28/git-poll-results/](https://jvns.ca/blog/2024/03/28/git-poll-results/)

A new thing I’ve been trying while writing this Git zine is doing a bunch of polls on Mastodon to learn about:

*   which git commands/workflows people use (like “do you use merge or rebase more?” or “do you put your current git branch in your shell prompt?”)
*   what kinds of problems people run into with git (like “have you lost work because of a git problem in the last year or two?”)
*   which terminology people find confusing (like “how confident do you feel that you know what HEAD means in git?”)
*   how people think about various git concepts (“how do you think about git branches?”)
*   in what ways my usage of git is “normal” and in what ways it’s “weird”. Where am I pretty similar to the majority of people, and where am I different?

It’s been a lot of fun and some of the results have been surprising to me, so here are some of the results. I’m partly just posting these so that I can have them all in one place for myself to refer to, but maybe some of you will find them interesting too.

### these polls are highly unscientific

Polls on social media that I thought about for approximately 45 seconds before posting are not the most rigorous way of doing user research, so I’m pretty cautious about drawing conclusions from them. Potential problems include: I phrased the poll badly, the set of possible responses aren’t chosen very carefully, some of the poll responses I just picked because I thought they were funny, and the set of people who follow me on Mastodon is not representative of all git users.

But here are a couple of examples of why I still find these poll results useful:

*   The first poll is “what’s your approach to merge commits and rebase in git”? 600 people (30% of responders) replied “I usually use merge, rarely/never rebase”. It’s helpful for me to know that there are a lot of people out there who rarely/never use rebase, because I use rebase all the time – it’s a good reminder that my experiences isn’t necessarily representative.
*   For the poll “how confident do you feel that you know what HEAD means in git?“, 14% of people replied “literally no idea”. That tells me to be careful about assuming that people know what `HEAD` means in my writing.

### where to read more

If you want to read more about any given poll, you can click at the date at the bottom – there’s usually a bunch of interesting follow-up discussion.

Also this post has a lot of CSS so it might not work well in a feed reader.

Now! Here are the polls! I’m mostly just going to post the results without commenting on them.

### merge and rebase

poll: what's your approach to merge commits and rebase in git?

### merge conflicts

poll: if you use git, how often do you deal with nontrivial merge conflicts? (like where 2 people were really editing the same code at the same time and you need to take time to think about how to reconcile the edits)

another merge conflict poll:

have you ever seen a bug in production caused by an incorrect merge conflict resolution? I've heard about this as a reason to prefer merges over rebase (because it makes the merge conflict resolution easier to audit) and I'm curious about how common it is

I thought it was interesting in the next one that “edit the weird text file by hand” was most people’s preference:

poll: when you have a merge conflict, how do you prefer to handle it?

merge conflict follow up: if you prefer to edit the weird text file by hand instead of using a dedicated merge conflict tool, why is that?

poll: did you know that in a git merge conflict, the order of the code is different when you do a merge/rebase?

merge:

<<<<<<< HEAD
YOUR CODE
=======
OTHER BRANCH'S CODE
>>>>>>> c694cf8aabe

rebase:

<<<<<<< HEAD
OTHER BRANCH'S CODE
=======
YOUR CODE
>>>>>>> d945752 (your commit message)

(where "YOUR CODE" is the code from the branch you were on when you ran `git merge` or `git rebase`)

### git pull

poll: do you prefer `git fetch` or `git pull`?

(no lectures about why you think `git pull` is bad please but if you use both I'd be curious to hear in what cases you use fetch!)

### commits

[poll] how do you think of a git commit?

(sorry, you can't pick “it’s all 3”, I'm curious about which one feels most true to you)

### branches

poll: how do you think about git branches? (I'll put an image in a reply with pictures for the 3 options)

as with all of these polls obviously all 3 are valid, I'm curious which one feels the most true to you

### git environment

poll: do you put your current git branch in your shell prompt?

poll: do you use git on the command line or in a GUI?

(you can pick more than one option if it’s a mix of both, sorry magit users I didn't have space for you in this poll)

### losing work

poll: have you lost work because of a git problem in the last year or two? (it counts even if it was "your fault" :))

### meaning of various git terms

These polls gave me the impression that for a lot of git terms (fast-forward, reference, HEAD), there are a lot of git users who have “literally no idea” what they mean. That makes me want to be careful about using and defining those terms.

poll: how confident do you feel that you know what HEAD means in git?

another poll: how do you think of HEAD in git?

poll: when you see this message in `git status`:

”Your branch is up to date with 'origin/main’.”

do you know that your branch may not actually be up to date with the `main` branch on the remote?

poll: how confident do you feel that you know what the term "fast-forward" means in git, for example in this error message:

`! [rejected] main -> main (non-fast-forward)`

or this one:

fatal: Not possible to fast-forward, aborting.

(I promise this is not a trick question, I'm just writing a blog post about git terminology and I'm trying to gauge how people feel about various core git terms)

poll: how confident do you feel that you know what a "ref" or "reference" is in git? (“ref” and “reference” are the same thing)

for example in this error message (from `git push`)

error: failed to push some refs to 'github.com:jvns/int-exposed'

or this one: (from `git switch mybranch`)

fatal: invalid reference: mybranch

another git terminology poll: how confident do you feel that you know what a git commit is?

(not a trick question, I'm mostly curious how this one relates to people's reported confidence about more "advanced" terms like reference/fast-forward/HEAD)

poll: in git, do you think of "detached HEAD state" and "not having any branch checked out" as being the same thing?

poll: how confident do you feel that you know what the term "current branch" means in git?

(deleted & reposted to clarify that I'm asking about the meaning of the term)

### other version control systems

I occasionally hear “SVN was better than git!” but this “svn vs git” poll makes me think that’s a minority opinion. I’m much more cautious about concluding anything from the hg-vs-git poll but it does seem like some people prefer git and some people prefer Mercurial.

poll 2: if you've used both svn and git, which do you prefer?

(no replies please, i have already read 300 comments about git vs other version control systems today and they were great but i can't read more)

gonna do a short thread of git vs other version control systems polls just to get an overall vibe

poll 1: if you've used both hg and git, which do you prefer?

(no replies please though, i have already read 300 comments about git vs other version control systems today and i can't read more)

### that’s all!

It’s been very fun to run all of these polls and I’ve learned a lot about how people use and think about git.