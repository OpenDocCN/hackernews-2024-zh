<!--yml
category: 未分类
date: 2024-05-29 12:48:30
-->

# XZ Utils backdoor

> 来源：[https://tukaani.org/xz-backdoor/](https://tukaani.org/xz-backdoor/)

<main>

This page will get updated as I learn more about the incident.

## Facts

*   CVE-2024-3094

*   XZ Utils 5.6.0 and 5.6.1 release tarballs contain a backdoor. These tarballs were created and signed by *Jia Tan*.

*   Tarballs created by Jia Tan were signed by him. Any tarballs signed by me were created by me.

*   GitHub accounts of both me (Larhzu) and Jia Tan were suspended. Mine was reinstated on 2024-04-02.

*   Only I have had access to the main tukaani.org website, git.tukaani.org repositories, and related files. Jia Tan only had access to things hosted on GitHub, including xz.tukaani.org subdomain (and only that subdomain).

*   The XZ projects have been moved to their old URLs on tukaani.org. XZ Utils’s home page is under construction still though.

*   xz.tukaani.org DNS name (CNAME) has been removed and won’t be restored.

*   The email address xz *at* tukaani *dot* org forwards to me only. This change was made on 2024-03-30. Previously it forward to Jia Tan too.

*   2024-04-09: The Git repositories of XZ projects are available on [GitHub](https://github.com/tukaani-project) again. Please use these for downloading the code to reduce the bandwidth usage on git.tukaani.org.

*   GitHub had closed all pull requests and marked them as if I had closed them on 2024-04-05. I didn’t close any PRs on that day, GitHub just misattributed it to me. On 2024-04-12 I tried to re-open PRs and GitHub immediately closed them again, again marking them as if I had done it. (On IRC it was suggested that this might have happened because the PRs refer to forks that are disabled.)

*   After the primary location of xz.git was officially moved to GitHub, I didn’t detected any force pushes to the master or v5.x branches. I haven’t force pushed to git.tukaani.org.

I won’t reply for now. I might reconsider after my planned article is out.

## Email

I have gotten a lot of email. Thanks for the positive comments. Unfortunately I don’t have time to reply to most of them.

## Donations

A few have suggested that I should enable GitHub Sponsors.

The Finnish law doesn’t allow a private person to request money from the general public and give nothing in return. Certain types of organizations can ask for donations but to do that even they need to get a permit first.

This means that things like GitHub Sponsors don’t seem legal to use.

## Plans

### Git repositories

xz.git needs to be gotten to a state where I’m happy to say I fully approve its contents. Work-in-progress [review comments](review.html) are available.

There was discussion if the master branch should be rebased to purge the malicious files. This would have been equivalent to dropping ([eight commits](https://git.tukaani.org/?p=xz.git;a=commit;h=e93e13c8b3bec925c56e0c0b675d8000a0f7f754)). The main benefit would have been that then antivirus software wouldn’t complain about them. Otherwise the files are harmless without the trigger code that was never included in the Git repository.

It has been decided that the Git repository will *not* be rebased. This is simpler (less work) and the benefits of rebasing seem minor compared to not-so-trivial downsides.

A clean stable XZ Utils release version might jump to 5.8.0. It would clearly separate the clean one from the bad 5.6.x.

The other Git repositories (xz-embedded and xz-java) are fine.

### What can be learned from this

I plan to write an article how the backdoor got into the releases and what can be learned from this. However, the review of the Git repository has a higher priority right now than the pending article.

</main>