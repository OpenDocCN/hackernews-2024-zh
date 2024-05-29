<!--yml
category: 未分类
date: 2024-05-27 14:25:12
-->

# Git Things

> 来源：[https://matklad.github.io/2023/12/31/git-things.html#Git-Things](https://matklad.github.io/2023/12/31/git-things.html#Git-Things)

Should every commit pass the tests? If it should, then your [not rocket science rule](https://graydon2.dreamwidth.org/1597.html) implementation must be verifying this property. It probably doesn’t, and only tests the final result of merging the feature branch into the main branch.

That’s why for typical project it is useful to *merge* pull requests into the main branch — the linear sequence of merge commits is a record of successful CI runs, and is a set of commits you want to `git bisect` over.

Within a feature branch, not every commit necessary passes the tests (or even builds), and that is a useful property! Here’s some ways this can be exploited:

*   When fixing a bug, add a failing test first, as a separate commit. That way it becomes easy to verify for anyone that the test indeed fails without the follow up fix.

    Related advice: often I see people commenting out tests that currently fail, or tests that are yet to be fixed in the future. That’s bad, because commented-out code rots faster than the JavaScript framework of the day. Instead, adjust the asserts such that they lock down the current (wrong) behavior, and add a clear `// TODO:` comment explaining what would be the correct result. This prevents such tests from rotting and also catches cases where the behavior is fixed by an unrelated change.

*   To refactor an API which has a lot of usages, split the work in two commits. In the first commit, change the API itself, but don’t touch the usages. In the second commit, mechanically adjust all call sites.

    That way during review it is trivial to separate meaningful changes from a large, but trivial diff.

*   `git mv` is fake. For a long time, I believed that `git mv` adds some special bit of git metadata which tells it that the file was moved, such that it can be understood by `diff` or `blame`. That’s not the case: `git mv` is essentially `mv` followed by `git add`. There’s nothing in git to track that a file was moved specifically, the “moved” illusion is created by the diff tool when it heuristically compares repository state at two points in time.

    For this reason, if you want to reliably record file moves during refactors in git, you should do two commits: the first commit *just* moves the file without any changes, the second commit applies all the required fixups.

    Speaking of moves, consider adding this to your `gitconfig`:

    ```
    [diff]
     colormoved = "default"
     colormovedws = "allow-indentation-change"
    ```

    This way, moved lines will be colored differently in `diff`, so that code motions not confused with additions and deletions, and are easier to review. It is unclear to me why this isn’t the default, and why this isn’t an option in GitHub’s UI.

“Merge into main, but rebase feature branches” might be a hard rule to wrap your head around if you are new to git. Luckily, it’s easy to use not-rocket-science rule to enforce this property. The history is as much a part of your project as is the source code. You can write a test that shells out to git and checks that the only merge commits in the history are those from the merge bot. While you are at it, it would be a good idea to test that no large large files are present in the repository — the size of a repository only grows, and you can’t easily remove large blobs from the repo later on!

Let me phrase this in the most inflammatory way possible :)

If your project has great commit messages, with short and precise summary lines and long and detailed bodies, this probably means that your CI and code review process suck.

Not all changes are equal. In a typical project, most of the changes that *should* be made are small and trivial — some renames, visibility tightening, “attention to details” polish in user-visible features.

However, in a typical project, landing a trivial change is slow. How long would it take you to fix `it's/its` typo in a comment? Probably 30 seconds to push the actual change, 30 minutes to get the CI results, and 3 hours for a review roundtrip.

The fixed costs to making a change are tremendous. Main branch gatekeeping strongly incentivizes against trivial changes. As a result, such changes either are not being made, or are tacked onto larger changes as a drive by bonus. In any case, the total number of commits and PRs goes down. And you are crafting a novel of a commit message because you have to wait for your previous PR to land anyway.

What can be done better?

*First*, make changes smaller and more frequent. Most likely, this is possible for you. At least, I tend to out-commit most colleagues ([example](https://github.com/intellij-rust/intellij-rust/graphs/contributors)). That’s not because I am more productive — I just do work in smaller batches.

*Second*, make CI asynchronous. At no point in your workflow you should be waiting for CI to pass. You should flag a change for merging, move on to the next thing, and only get back if CI fails. This is something bors-ng does right — it’s possible to `r+` a commit immediately on submission. This is something GitHub merge queue does wrong — it’s impossible to add a PR to queue until checks on the PR itself are green.

*Third*, our review process is backwards. Review is done *before* code gets into main, but that’s inefficient for most of the non-mission critical projects out there. A better approach is to optimistically merge most changes as soon as not-rocket-science allows it, and then later review the code in situ, in the main branch. And instead of adding comments in web ui, just changing the code in-place, sending a new PR ccing the original author.

> 14.  Maintainers SHALL NOT make value judgments on correct patches.
> 15.  Maintainers SHALL merge correct patches from other Contributors rapidly.
> 
> …
> 
> 18.  Any Contributor who has value judgments on a patch SHOULD express these via their own patches.

[Collective Code Construction Contract](https://rfc.zeromq.org/spec/42/)

I am skeptical that this exact workflow would ever fly, but I am cautiously optimistic about [Zed’s](https://zed.dev) idea about just allowing two people to code in the same editor at the same time. I think that achieves a similar effect, and nicely dodges unease about allowing temporarily unreviewed code.

Ok, back to git!

*First*, not every project needs a clean history. Have you ever looked at the git history of your personal blog or dotfiles? If you haven’t, feel free to use a `.` as a commit message. I do that for [https://github.com/matklad/matklad.github.io](https://github.com/matklad/matklad.github.io), it works fine so far.

*Second*, not every change needs a great commit message. If a change is really minor, I would say `minor` is an okay commit message!

*Third*, some changes absolutely do require very detailed commit messages. If there *is* a context, by all means, include all of it into the commit message (and spill some as comments in the source code). And here’s a tip for this case: *write the commit message first!*

When I work on a larger feature, I start with `git commit --allow-empty` to type out what I set to do. Most of the time, by the third paragraph of the commit message I realize that there’s a flaw in my plan and refine it. So, by the time I get to actually writing the code, I am already on the second iteration. And, when I am done, I just amend the commit with the actual changes, and the commit message is already there, needing only minor adjustments.

And the last thing I want to touch about commit messages: `man git-commit` tells me that the summary line should be shorter than 50 characters. This feels obviously wrong, that’s much too short! [Kernel docs](https://www.kernel.org/doc/html/v4.10/process/submitting-patches.html) suggest a much more reasonable 70-75 limit! And indeed, looking at a some recent kernel commits, 50 is clearly not enough!

```
<---               50 characters              --->
 get_maintainer: remove stray punctuation when cleaning file emails
get_maintainer: correctly parse UTF-8 encoded names in files
locking/osq_lock: Clarify osq_wait_next()
locking/osq_lock: Clarify osq_wait_next() calling convention
locking/osq_lock: Move the definition of optimistic_spin_node into osq_lock.c
ftrace: Fix modification of direct_function hash while in use
tracing: Fix blocked reader of snapshot buffer
ring-buffer: Fix wake ups when buffer_percent is set to 100
platform/x86/intel/pmc: Move GBE LTR ignore to suspend callback
platform/x86/intel/pmc: Allow reenabling LTRs
platform/x86/intel/pmc: Add suspend callback
platform/x86: p2sb: Allow p2sb_bar() calls during PCI device probe
 <---               50 characters              --->
```

Happy new year, dear reader!