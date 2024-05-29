<!--yml
category: 未分类
date: 2024-05-27 14:40:35
-->

# The ideal PR is 50 lines long

> 来源：[https://graphite.dev/blog/the-ideal-pr-is-50-lines-long](https://graphite.dev/blog/the-ideal-pr-is-50-lines-long)

Most engineers intuitively know that smaller code changes are better than big ones. The logical arguments flow easy - small pull requests are easier to review, less likely to have bugs, and are faster from inception to deploy. There are a few papers around this that I love - see the references section at the end of this post for further reading.

But how small is small? Can PRs be *too* small? And if a PR is better than a big PR, *how much* better?

**Note**

Greg spends full workdays writing weekly deep dives on engineering practices and dev-tools. This is made possible because these articles help get the word out about Graphite. If you like this post, try [Graphite](https://graphite.dev?utm_source=blog-note) today, and start shipping 30% faster!

## The claim: the ideal PR is 50 lines long[](/blog/the-ideal-pr-is-50-lines-long#the-claim-the-ideal-pr-is-50-lines-long)

After pulling the numbers, the ideal code change is 50 lines long.

50-line code changes are reviewed and merged ~40% faster than 250-line changes. They’re 15% less likely to be reverted than 250-line changes and have 40% more review comments per line changed. If your median PR is 50 lines long, you’re probably shipping 40% more total code than your teammate writing 200+ line PRs.

50 lines is a sweet spot across speed, review comments, revert rate, and total coding volume. If you’re willing to accept a range, I can recommend 25-100 lines per PR. According to the data, we see that time-to-review, time-to-merge, and review comments per line all get better the smaller you make your PRs. There is a limit though: under 25 lines, and you start suffering a higher revert rate, as well as a lower total code shipped.

Let’s talk about why and back this claim with some data.

## Our sample set[](/blog/the-ideal-pr-is-50-lines-long#our-sample-set)

All of the data-based statements in this piece are made using private and public PRs and repos that have been synced with Graphite. To figure out the ideal PR size, I took a look at four main metrics and how they correlated with PR size:

### Caveats[](/blog/the-ideal-pr-is-50-lines-long#caveats)

As with all collected data, there are some caveats to be aware of when extrapolating meaning from the numbers:

*   I used inconsistent, non-linear bucket sizes corresponding to intuitive PR size ranges. Linear buckets would be too granular, and exponential bucket sizes lose too much nuance.

*   I used median PR size rather than average PR size to avoid outlier refactors from skewing the data.

*   Reverts were defined as PRs with the word “Revert” in the title. This felt like a safe assumption because generated GitHub reverts automatically have the word prepended to the title.

*   We find that Graphite users tend to create smaller PRs in general, since many organizations use a trunk-based development style (which encourages smaller PRs)

### Time-to-review and time-to-merge[](/blog/the-ideal-pr-is-50-lines-long#time-to-review-and-time-to-merge)

Let’s dig into the data. Starting with time-to-review and time-to-merge, we see that the smallest PRs are almost five times faster than 2-5k line PRs. Intuitively this makes sense - smaller PRs mean less lines of code, and less code means a lower chance of having destructive or meticulous changes which in turn leads to a faster review.

What’s fascinating to see play out is that post-5k lines, PRs start getting faster again. I can only assume this is a combination of blind stamps, assumed-safe refactors, package additions or generated changes; perhaps both authors and reviewers start shrugging when they can’t even scroll to the bottom of the change.

Note, we assume that we care about the time per PR more than we do the time per line. If we care about landing a PR as fast as possible, the data tells us to make it as small as possible. But if we want to land as high volume of code as fast as possible, a 2000-line PR merges at a rate of about 12 lines per hour, whereas a 10-line PR merges at about 0.25-2 lines per hour.

### Revert rate by PR size[](/blog/the-ideal-pr-is-50-lines-long#revert-rate-by-pr-size)

The revert rate demonstrates the same high-level conclusion: smaller PRs are reverted less than large PRs, the least reverted PRs being those that fall between 25-50 lines of code. But once again, the edges of the graph are interesting. Sub-10 line PRs are reverted noticeably more than 10-100 line PRs. If I had to guess, I’d say that sub-10 line PRs get into the realm of dangerous config changes, but I’d be curious if this finding held true after normalizing for language.

Once PRs start to exceed 10k lines of code, they seem to become slightly “safer.” I suspect this is because the extreme end of PR sizes includes refactors, which maybe start including less functionality change and therefore have a slightly lower chance of breaking. Alternatively, engineers may become progressively more reluctant to revert PRs after 10k lines because of emotional anchoring and merge conflicts.

### Average number of inline comments by PR size[](/blog/the-ideal-pr-is-50-lines-long#average-number-of-inline-comments-by-pr-size)

Depending on what you’re optimizing for (a fast merge or an in-depth code review), you can choose whether or not you want to split your PR into smaller changesets. If you want to get the most feedback on a single PR, write a length 1-2k diff. If you want the highest chance of a blind stamp, keep it under 10 lines. This information is useful to know when all you care about is getting a single specific change out the door with as little debate as possible.

Here we also see that massive PRs start getting progressively less engagement. There’s a practical limit to how much code your reviewer is willing to read - I suspect that 2k lines is the point at which “reading” a PR becomes “skimming” a PR.

If what you care about is maximizing the amount of engagement and feedback on your code over the long run, you’re best off writing as small of PRs as possible. They’re more digestible, and you can approach a max rate of one comment on every 39 lines of code you write. Alternatively, if you hate written feedback, make your PRs larger than 10k lines, and you’ll start receiving stray comments on only every 6000+ lines of code you change - take this with a grain of salt though, since people rarely put up PRs for the sole purpose of getting feedback/engagement.

### Total code output[](/blog/the-ideal-pr-is-50-lines-long#total-code-output)

Some folks might wonder if writing small PRs leads to less code output in total. We all want to be high velocity, but there are times when you need to be high volume. Consistently writing sub-20 line PRs will have a significant impact on your net coding ability - but interestingly, so will writing PRs greater than 100+ lines. The highest volume coders and repositories have a median change size of only 40-80 lines. I suspect this is because the volume of code is change-size * speed of changes. Too small of changes, and the faster merge time doesn't make up for it. Too large, and you start getting weighted down by slower review and merge cycles.

## Conclusion[](/blog/the-ideal-pr-is-50-lines-long#conclusion)

The average developer coding with a team at a tech company should aim for a median PR size of 50 lines - it's generally the size of PR we stick to here at Graphite. Obviously this comes with some edge cases that were discussed above and specific changes may require you to flex up and down in size, but know that doing so may come at a noticeable cost with respect to review quality, speed, and the chance that changes may need to be reverted.

## Further reading[](/blog/the-ideal-pr-is-50-lines-long#further-reading)