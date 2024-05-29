<!--yml
category: 未分类
date: 2024-05-29 13:19:36
-->

# Your GitHub pull request workflow is slowing everyone down

> 来源：[https://graphite.dev/blog/your-github-pr-workflow-is-slow](https://graphite.dev/blog/your-github-pr-workflow-is-slow)

As developers, we strive to write clean, modular code that's easy to maintain. 

However, between the pressure to deliver features quickly and the complexity of modern applications, putting this into practice takes a lot of work. Any meaningful engineering task can easily result in large, tangled pull requests that become bottlenecks in the development process.

In this guide, we'll explore how the stacking workflow can help you overcome these challenges and ship better code faster. Whether you're new to stacking or looking to optimize an existing process, this workflow can streamline collaboration, accelerate review cycles, and promote coding best practices on your team.

## **Why your GitHub pull request workflow is slow**[](/blog/your-github-pr-workflow-is-slow#why-your-github-pull-request-workflow-is-slow)

Here at Graphite, we analyzed data from thousands of repositories to understand how common pull request (PR) workflows on GitHub lead to blocked developers, slow reviews, and reduced code quality. 

### **PRs are too big**[](/blog/your-github-pr-workflow-is-slow#prs-are-too-big)

The single most important bottleneck is PR size - large PRs can make code reviews frustrating and ineffective. The average PR on GitHub has [900+ lines](https://www.keypup.io/product/average-pull-request-size-metric) of code changes.

For speed and quality, PRs should be maintained under 200 lines—with 50 lines being ideal. 

To put this in perspective, where giant 500+ line PRs take around 9 days to get merged on average, tiny PRs under 100 lines can make it from creation to landing within hours.

### **PRs take too long to review and progress**[](/blog/your-github-pr-workflow-is-slow#prs-take-too-long-to-review-and-progress)

When PRs have thousands of lines changed, properly reviewing them becomes incredibly difficult. Developers can easily waste days revising the code after initial feedback, and reviewers need to then rescan all the code, spot changes, and approve or reject every revision.

These back-and-forth review cycles on large PRs slow down team velocity, and aren’t necessarily thorough. According to Graphite’s data, only 24% of massive 1000+ line PRs receive **any** review comments.

### **Large PRs introduce more bugs**[](/blog/your-github-pr-workflow-is-slow#large-prs-introduce-more-bugs)

When code isn’t reviewed thoroughly, it’s easy for bugs to slip through since reviewers [often don’t have the time to give each PR](https://graphite.dev/blog/how-large-prs-slow-down-development) the attention they deserve. 

Problems can easily get hidden between the diffs, and reviewers often make assumptions instead of testing to avoid feeling overwhelmed. One particularly interesting finding is that as the size of a PR increases (by number of files changed), the amount of time reviewers spend on each file decreases significantly (for PRs with 8 or more files changed).

These quick reviews can easily lead to quality issues, which yield more bug fix PRs, technical debt, and team frustration. Engineers feel their progress reversing as they devote more and more time to patching bugs rather than developing features.

## **Standard GitHub pull request workflow**[](/blog/your-github-pr-workflow-is-slow#standard-github-pull-request-workflow)

Millions of developers use GitHub's pull request model daily, but in practice teams often encounter pain points that slow progress to a crawl. 

Let's understand the standard GitHub PR workflow.

### **1\. Create a feature branch**[](/blog/your-github-pr-workflow-is-slow#1-create-a-feature-branch)

Typically, GitHub projects follow a branching model where developers create a new branch for each user story or feature. 

First, you **checkout** the **main branch** *(where the production-ready code lives)* and create a feature branch using the following:

```
git checkout -b new-feature
```

**This spawns an isolated stream for your changes.**

Now, this may seem great—you have a safe space to build new functionality without impacting **main**. It lets **main** remain stable so your CI/CD pipeline is unobstructed. 

**But as your branch history diverges further from main, you’ll notice a few significant problems:**

*   Merging [upstream changes](https://stackoverflow.com/questions/2739376/definition-of-downstream-and-upstream) becomes tedious due to rebasing conflicts.

*   You may disrupt other dependent branches while pushing a [hotfix](https://en.wikipedia.org/wiki/Hotfix), losing your place.

*   After weeks and multiple new main releases, your code now depends on outdated branches and you may require many changes before it works with the latest branch.

What was once a productive branch devolves into a stale silo as you start drowning in more and more technical debt.

### **2\. Write code and commit changes**[](/blog/your-github-pr-workflow-is-slow#2-write-code-and-commit-changes)

Since workflows don’t change overnight, you continue writing your big new feature. You optimize for getting prototypes built instead of maintaining strict isolation. 

It feels faster to iterate on related components within your dedicated branch.

**Before you know it, you have:**

*   1,500 lines modifying critical services.

*   Data model changes spanning multiple microservices.

*   Multiple new frontend routes and UI flows.

All of this is smashed together in an unmanageable commit history

But at least it's still safely quarantined behind your feature branch, right? Not really—your branch is highly coupled across all of those fronts, and decoupling things later can become challenging without a lot of rebasing.

Once ready, you commit and push the local code changes to the remote feature branch. Now your changes are saved on the remote repository.

### **3\. Open a pull request**[](/blog/your-github-pr-workflow-is-slow#3-open-a-pull-request)

Once your feature branch contains the changes to be merged into the main branch, the next step is to open a pull request (PR). 

**A pull request lets you notify your teammates that your code is ready for review and feedback.**

To create a PR, you will go to your code management tool and open a pull request for your branch. 

You write a short, descriptive summary of the changes in your PR e.g., "Added user profile feature."

For the PR body, you write a longer description explaining in more detail:

*   The specifics of what changes you have made and what new functionality you have added.

*   Technical details on how you implemented things.

*   Your reasoning for why these changes are needed.

*   Steps for how teammates can test out the new functionality you built.

The PR now becomes a centralized place for discussion tied to the branch. You will ping team members and stakeholders to request their code review.

### **4\. Address review comments**[](/blog/your-github-pr-workflow-is-slow#4-address-review-comments)

After your code goes under review, teammates thoroughly inspect it for bugs, bad coding practices that slipped in, edge cases you could’ve considered, and opportunities for simplification.

**Issues large and small will inevitably come up. **

You have iterative back-and-forth code-level conversations on the PR page. Teammates can comment on specific lines of code to point out problems or ask clarifying questions.

For each round of feedback, you dive back into your branch to address the comments. This leads to a lot of rework—fixing defects, refactoring architecture, adding test cases, etc., based on the insights from reviewers.

Once you push new commits to your branch, they automatically appear in the pull request for further feedback. This review cycle continues until all issues are resolved.

### **5\. Merge your pull request**[](/blog/your-github-pr-workflow-is-slow#5-merge-your-pull-request)

Finally, after all the feedback has been sufficiently addressed and your code receives a LGTM ("looks good to me") from teammates, it’s time to merge your approved pull request into **main**.

This merge integrates your code with **main**, which will:

*   Merge your branch commits into the main branch.

*   Automatically close the pull request.

*   Delete your feature branch *(optional, depends on your internal workflows)*.

Your changes are now fully incorporated into the production code. But for every new (even closely related) feature, you’ll start from square one, repeating the entire workflow. 

**What if there was a better way?** Enter [stacking](https://stacking.dev). 

## **Improving PR workflows with stacking**[](/blog/your-github-pr-workflow-is-slow#improving-pr-workflows-with-stacking)

*Stacking is a development approach that optimizes the GitHub pull request workflows by breaking large features into small, dependent pull requests that get incrementally integrated. Once all incremental pull requests have been approved, they get merged with main.*

Let's explore how stacking addresses the painpoints in traditional PR workflows and meaningfully increases your development speed.

### **What is stacking?**[](/blog/your-github-pr-workflow-is-slow#what-is-stacking)

Stacking structures feature development using “stacks” of small pull requests built on top of one another:

By stacking PRs, each PR becomes more focused and is easier to review and fix. And new work can continue to build on existing PRs before they get approved.

This lets engineers rapidly open multiple small PRs and progress concurrently without being blocked. Changes ship continuously rather than landing in massive monolithic code dumps.

### **How does stacking work?**[](/blog/your-github-pr-workflow-is-slow#how-does-stacking-work)

Stacking transforms how large features get built by having developers split their work into multiple small, interconnected pull requests. Instead of one massive PR, they create a stack of focused changes that build on each other.

*For example, a developer may be coding a new registration system. Rather than complete all the frontend, backend, and database work in a single huge 1000 line PR, they would break it down. One PR implements the registration API endpoints. Another connects these to the database. A third builds out the registration page tapping into the new API.*

By default, every PR is restricted to only 1 commit of <200 lines, keeping changes tightly scoped. This forces developers to consciously limit work to related changes—the registration endpoint PR can't sneak in unrelated styling tweaks.

These focused PRs interlink together into a cohesive stack—the registration page PR branches off the API PR, which builds on the database PR. Changes integrate upstream incrementally as they’re reviewed.

A developer tool like Graphite, that automates stacking handles all of the dependencies between PRs, updating branches and continuously rebasing to keep branches in sync. So,  with smaller changes, stacking dramatically reduces the time required for reviews and revisions, helping you implement new features and fix bugs faster than the traditional workflows. 

### **Why stacking improves the pull request workflow**[](/blog/your-github-pr-workflow-is-slow#why-stacking-improves-the-pull-request-workflow)

Transitioning teams from the traditional workflow to a stacking workflow fundamentally upgrades how changes get structured, reviewed, and integrated. It optimizes for productivity, quality, and comprehension. Here are a few ways it helps your dev productivity.

#### **Breaks down large changes**[](/blog/your-github-pr-workflow-is-slow#breaks-down-large-changes)

Stacking centers around breaking down big feature work into chains of smaller pull requests. Each PR is typically limited to 1 commit focused on an isolated change. This restriction guides developers to consciously make only a single change, squashing and rebasing along the way, instead of cluttering the PR with random unnecessary commits like "typo fixes".

#### **Accelerates review cycles**[](/blog/your-github-pr-workflow-is-slow#accelerates-review-cycles)

With traditional practices, devs batch changes into huge PRs that overwhelm reviewers. Feedback gets pushed further and further in time because of how long it’d take to review big chunks of code.

In contrast, stacking small PRs speeds up feedback loops, as reviewers have much less complexity to parse and can give both timely and thorough feedback.

#### **Simplifies bug fixes**[](/blog/your-github-pr-workflow-is-slow#simplifies-bug-fixes)

Atomization ensures fixes stay clean—if any PR introduces issues, reverting that single, contained change is simple. The smaller this PR is, the quicker your code can be fixed. For instance, a bug in a 50-line PR may take a few minutes to fix compared to a 500-line PR.

#### **Enables steady integration**[](/blog/your-github-pr-workflow-is-slow#enables-steady-integration)

With typical huge PR workflows, there ends up being an uncomfortable pressure around finally getting giant changes approved and merged. Especially as the lengthy review process drags on, managers start pushing hard to accelerate the merge and unblock other work.

On the contrary, stacked PRs can flow into main rapidly through steady continuous delivery of tiny, incremental changes. Each change is isolated and low risk, making reverts painless if a merge leads to a bug. By splitting reviews before releasing batches of hundreds or thousands of lines, stacking removes the bottlenecks and tensions that accumulate around giant PRs

Now that you know stacking can greatly benefit your internal developer and Ops workflows, how do you implement this flow for your team?

## **GitHub pull request workflow with stacking**[](/blog/your-github-pr-workflow-is-slow#github-pull-request-workflow-with-stacking)

Let’s look at a stacking workflow in action. The goal is to optimize GitHub workflows through steady integration instead of having large, monolithic code changes. We want our changes to flow through fluidly rather than turning them into massive PRs that block other developers. 

To simplify things, we’ll be working on [**Graphite**](https://graphite.dev/)—a powerful tool that automates many stacking workflow steps. 

### **1\. Checkout main branch to start fresh**[](/blog/your-github-pr-workflow-is-slow#1-checkout-main-branch-to-start-fresh)

Let’s suppose you’re working on a new feature—**app notifications**.

Once you install and integrate Graphite with your Git repo, you can check out the latest main branch code to ensure you begin with up-to-date context:

Unlike Git workflows, where it is easy to neglect staying updated, Graphite centers your workflow around continually integrating with the current mainline state.

### **2\. Build feature incrementally through stacked branches**[](/blog/your-github-pr-workflow-is-slow#2-build-feature-incrementally-through-stacked-branches)

With main checked out, start constructing your feature through iterative, focused work.

As a first step for our **app notifications** feature, we’ll add initial notification scaffolding by staging changes and committing them to a new branch in one flow using `gt create`:

```
gt create -am "feat: add notification scaffolding”
```

This command will add your changes and create a new branch in one motion. You can then continue iterating by creating and stacking additional branches:

```
gt create -am "feat: implement email templates""
```

The email templates build on the scaffolding foundation while the review happens asynchronously. Engineers avoid blocking progress on dependent work. Changes integrate smoothly through steady streams instead of massive pull requests.

### **3\. Squash multi-commit branches**[](/blog/your-github-pr-workflow-is-slow#3-squash-multi-commit-branches)

If you have multiple commits for a branch, Graphite makes it simple to squash and merge them, bringing consistency and following the 1 commit per PR standard.

```
gt log◉ 06-28-second_branch (current)│ 10 minutes ago│| 6s7a8d7 - last committed change│ d7d41b6 - committing another change│ 8c6d8de - committing some changes│◯ 06-28-first_branch│ 5 minutes ago││ 232e8cf - initial commit│◯ main│ 10 minutes ago││ 1e0b290 - Merging a pull requestgt squashgt log◉ 06-28-second_branch (current)│ just now|│ 9e13a52 - a single commit│◯ 06-28-first_branch│ 5 minutes ago││ 232e8cf - initial commit│◯ main│ 10 minutes ago││ 1e0b290 - Merging a pull request
```

By cleaning up your PR commit history, you ensure a clear and concise main branch history that makes it easy to see exactly what’s changed over time.

### **4\. Merge to main incrementally**[](/blog/your-github-pr-workflow-is-slow#4-merge-to-main-incrementally)

After completing all your feature work incrementally, your small PRs can easily be merged into the main branch without becoming a bottleneck or blocker for other developers. 

Simply submit the stack of branches to generate a stack of linked pull requests:

Graphite auto-creates PRs for the branches so changes can flow downstream. It even autopopulates the titles and descriptions with templates so you have a basic PR ready to go—fill out the template, and you’re good to go. 

Finally, after approval, your feature stack smoothly merges into **main**. 

### **5\. Greatly reduces the possibility of bugs**[](/blog/your-github-pr-workflow-is-slow#5-greatly-reduces-the-possibility-of-bugs)

In a stacked workflow, made up of many small, dependent pull requests, each change is small enough to ensure thorough review. Because large features are broken up into atomic changes and merged incrementally, it’s easy to roll back regressions and fix bugs, avoiding unnecessary downtime that comes with big, monolithic changes. 

## **Simplify your dev workflows with stacking in Graphite**[](/blog/your-github-pr-workflow-is-slow#simplify-your-dev-workflows-with-stacking-in-graphite)

Most developers still struggle with all of the moving parts—scattered tools, tangled branches, and stalled reviews of code review. 

[**Graphite**](https://graphite.dev/) **offers a better way forward.**

Graphite parallelizes review and development, automates all of the complexities of Git, and gives you more time to build.

*   Developers stay unblocked on code review and continue pushing changes

*   Reviewers provide rapid, thorough feedback, as features are broken up into small pieces

*   Managers rest easy as projects ship reliably

Graphite organizes GitHub PRs into stacks while simplifying and accelerating code review. 

**Start building with** [**Graphite**](https://graphite.dev/) **today.**