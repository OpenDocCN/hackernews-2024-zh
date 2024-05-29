<!--yml
category: 未分类
date: 2024-05-29 12:48:17
-->

# Basic Things

> 来源：[https://matklad.github.io/2024/03/22/basic-things.html](https://matklad.github.io/2024/03/22/basic-things.html)

# Basic Things Mar 22, 2024

After working on the initial stages of several largish projects, I accumulated a list of things that share the following three properties:

*   they are irrelevant while the project is small,
*   they are a productivity multiplier when the project is large,
*   they are much harder to introduce down the line.

Here’s the list:

A project should have a *short* one-page readme that is mostly links to more topical documentation. The two most important links are the user docs and the dev docs.

A common failure is a readme growing haphazardly by accretion, such that it is neither a good landing page, nor a source of comprehensive docs on any particular topic. It is hard to refactor such an unstructured readme later. The information is valuable, if disorganized, but there isn’t any better place to move it to.

For developers, you generally want to have a docs folder in the repository. The docs folder should *also* contain a short landing page describing the structure of the documentation. This structure should allow for both a small number of high quality curated documents, and a large number of ad-hoc append-only notes on any particular topic. For example, `docs/README.md` could point to carefully crafted [`ARCHITECTURE.md`](https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html) and `CONTRIBUTING.md`, which describe high level code and social architectures, and explicitly say that everything else in the `docs/` folder is a set of unorganized topical guides.

Common failure modes here:

1.  There’s no place where to put new developer documentation at all. As a result, no docs are getting written, and, by the time you do need docs, the knowledge is lost.

2.  There’s only highly structured, carefully reviewed developer documentation. Contributing docs requires a lot of efforts, and many small things go undocumented.

3.  There’s only unstructured append-only pile of isolated documents. Things are *mostly* documented, often two or there times, but any new team member has to do the wheat from the chaff thing anew.

Most project can benefit from a dedicated website targeted at the users. You want to have website ready when there are few-to-no users: usage compounds over time, so, if you find yourself with a significant number of users and no web “face”, you’ve lost quite a bit of value already!

Some other failure modes here:

1.  A different team manages the website. This prevents project developers from directly contributing improvements, and may lead to divergence between the docs and the shipped product.

2.  Today’s web stacks gravitate towards infinite complexity. It’s all too natural to pick an “easy” heavy framework at the start, and then get yourself into npm’s bog. Website is about content, and content has gravity. Whatever markup language dialect you choose at the beginning is going to stay with for some time. Do carefully consider the choice of your web stack.

3.  Saying that which isn’t quite done yet. Don’t overpromise, it’s much easier to say more later than to take back your words, and humbleness might be a good marketing. Consider if you are in a domain where engineering credibility travel faster than buzz words. But this is situational. More general advice would be that marketing also compounds over time, so it pays off to be deliberate about your image from the start.

This is more situational, but consider if, in addition to public-facing website, you also need an internal, engineering-facing one. At some point you’ll probably need a bit more interactivity than what’s available in a `README.md`  — perhaps you need a place to display code-related metrics like coverage or some javascript to compute release rotation. Having a place on the web where a contributor can place something they need right now without much red tape is nice!

This is a recurring theme — you should be organized, you should not be organized. *Some* things have large fan-out and should be guarded with careful review. *Other* things benefit from just being there and a lightweight process. You need to create places for both kinds of things, and a clear decision rule about what goes where.

For internal website, you’ll probably need some kind of data store as well. If you want to track binary size across commits, *something* needs to map commit hashes to (lets be optimistic) kilobytes! I don’t know a good solution here. I use a JSON file in a github repository for similar purposes.

There are many possible ways to get some code into the main branch. Pick one, and spell it out in an `.md` file explicitly:

*   Are feature branches pushed to the central repository, or is anyone works off their fork? I find forks work better in general as they automatically namespace everyone’s branches, and put team members and external contributors on equal footing.

*   If the repository is shared, what is the naming convention for branches? I prefix mine with `matklad/`.

*   You use [not rocket-science rule](https://graydon2.dreamwidth.org/1597.html) (more on this later :).

*   Who should do code review of a particular PR? A single person, to avoid bystander effect and to reduce notification fatigue. The reviewer is picked by the author of PR, as that’s a stable equilibrium in a high-trust team and cuts red tape.

*   How the reviewer knows that they need to review code? On GitHub, you want to *assign* rather than *request* a review. Assign is level-triggered — it won’t go away until the PR is merged, and it becomes the responsibility of the reviewer to help the PR along until it is merged (*request review* is still useful to poke the assignee after a round of feedback&changes). More generally, code review is the highest priority task — there’s no reason to work on new code if there’s already some finished code which is just blocked on your review.

*   What is the purpose of review? Reviewing for correctness, for single voice, for idioms, for knowledge sharing, for high-level architecture are choices! Explicitly spell out what makes most sense in the context of your project.

*   Meta process docs: positively encourage contributing process documentation itself.

Speaking about meta process, style guide is where it is most practically valuable. Make sure that most stylistic comments during code reviews are immediately codified in the project-specific style document. New contributors should learn project’s voice not through a hundred repetitive comments on PRs, but through a dozen links to specific items of the style guide.

Do you even need a project-specific style guide? I think you do — cutting down mental energy for trivial decisions is helpful. If you need a result variable, and half of the functions call it `res` and another half of the functions call it `result`, making this choice is just distracting.

Project-specific naming conventions is one of the more useful thing to place in the style guide.

Optimize style guide for extensibility. Uplifting a comment from a code review to the style guide should not require much work.

Ensure that there’s a style tzar — building consensus around *specific* style choices is very hard, better to delegate the entire responsibility to one person who can make good enough choices. Style usually is not about what’s better, it’s about removing needless options in a semi-arbitrary ways.

Document stylistic details pertaining to git. If project uses `area:` prefixes for commits, spell out an explicit list of such prefixes.

Consider documenting acceptable line length for the summary line. Git man page boldly declares that a summary should be under 50 characters, but that is just plain false. Even in the kernel, most summaries are somewhere between 50 and 80 characters.

Definitely explicitly forbid adding large files to git. Repository size increases monotonically, `git clone` time is important.

Document merge-vs-rebase thing. My preferred answer is:

*   A unit of change is a pull request, which might contain several commits
*   Merge commit for the pull request is what is being tested
*   The main branch contains only merge commits
*   Conversely, *only* the main branch contains merge commits, pull requests themselves are always rebased.

Forbidding large files in the repo is a good policy, but it’s hard to follow. Over the lifetime of the project, someone somewhere will sneakily add and revert a megabyte of generated protobufs, and that will fly under code review radar.

This brings us to the most basic thing of them all:

Maintain a well-defined set of automated checks that pass on the main branch at all times. If you don’t want large blobs in git repository, write a test rejecting large git objects and run that right before updating the main branch. No merge commits on feature branches? Write a test which fails with a pageful of Git self-help if one is detected. Want to wrap `.md` at 80 columns? Write a test :)

It is perhaps worth you while to re-read the original post: [https://graydon2.dreamwidth.org/1597.html](https://graydon2.dreamwidth.org/1597.html)

[This mindset of monotonically growing set of properties](https://matklad.github.io/2024/01/03/of-rats-and-ratchets.html) that are true about the codebase is *incredibly* powerful. You start seeing code as temporary, fluid thing that can always be changed relatively cheaply, and the accumulated set of automated tests as the real value of the project.

Another second order effect is that NRSR puts a pressure to optimize your build and test infrastructure. If you don’t have an option to merge the code when an unrelated flaky test fails, you won’t have flaky tests.

A common anti-pattern here is that a project grows a set of semi-checks — tests that exists, but are not 100% reliable, and thus are not exercised by the CI routinely. And that creates ambiguity — are tests failing due to a regression which should be fixed, or were they never reliable, and just test a property that isn’t actually essential for functioning of the project? This fuzziness compounds over time. If a check isn’t reliable enough to be part of NRSR CI gate, it isn’t actually a check you care about, and should be removed.

But to do NRSR, you need to build & CI your code first:

This is a complex topic. Let’s start with the basics: what is a build system? I would love to highlight a couple of slightly unconventional answers here.

*First*, a build system is a bootstrap process: it is how you get from `git clone` to a working binary. The two aspects of this boostrapping process are important:

*   It should be simple. No `sudo apt-get install bazzilion packages`, the single binary of your build system should be able to bring everything else that’s needed, automatically.
*   It should be repeatable. Your laptop and your CI should end up with exactly identical set of dependencies. The end result should be a function of commit hash, and not your local shell history, otherwise NRSR doesn’t work.

*Second*, a build system is developer UI. To do almost anything, you need to type some sort of build system invocation into your shell. There should be a single, clearly documented command for building and testing the project. If it is not a single `makebelieve test`, something’s wrong.

One anti-pattern here is when the build system spills over to CI. When, to figure out what the set of checks even is, you need to read `.github/workflows/*.yml` to compile a list of commands. That’s accidental complexity! Sprawling yamls are a bad entry point. Put all the logic into the build system and let the CI drive that, and not vice verse.

[There is a stronger version of the advice](https://matklad.github.io/2023/12/31/O(1)-build-file.html). No matter the size of the project, there’s probably only a handful of workflows that make sense for it: testing, running, releasing, etc. This small set of workflows should be nailed from the start, and specific commands should be documented. When the project subsequently grows in volumes, this set of build-system entry points should *not* grow.

If you add a Frobnicator, `makebelieve test` invocation *should* test that Frobnicator works. If instead you need a dedicated `makebelieve test-frobnicator` and the corresponding line in some CI yaml, you are on a perilous path.

*Finally*, a build system is a collection of commands to make stuff happen. In larger projects, you’ll inevitably need some non-trivial amount of glue automation. Even if the entry point is just `makebelive release`, internally that might require any number of different tools to build, sign, tag, upload, validate, and generate a changelog for a new release.

A common anti-pattern is to write these sorts of automations in bash and Python, but that’s almost pure technical debt. These ecosystems are extremely finnicky in and off themselves, and, crucially (unless your project itself is written in bash or Python), they are a second ecosystem to what you already have in your project for “normal” code.

But releasing software is also just code, which you can write in your primarly language. [The right tool for the job is often the tool you are already using](https://twitter.com/id_aa_carmack/status/989951283900514304). It pays off to explicitly attack the problem of glue from the start, and to pick/write a library that makes writing subprocess wrangling logic easy.

Summing the build and CI story up:

Build system is self-contained, reproducible and takes on the task of downloading all external dependencies. Irrespective of size of the project, it contains O(1) different entry points. One of those entry points is triggered by the not rocket science rule CI infra to run the set of canonical checks. There’s an explicit support for free-form automation, which is implemented in the same language as the bulk of the project.

Integration with NRSR is the most important aspect of the build process, as it determines how the project evolves over time. Let’s zoom in.

Testing is a primary architectural concern. When the first line of code is written, you already should understand the big picture testing story. It is empathically *not*  “every class and module has unit-test”. Testing should be data oriented — the job of a particular software is to take some data in, transform it, and spit different data out. Overall testing strategy requires:

*   some way to specify/generate input data,
*   some way to assert desired properties of output data, and
*   a way to run many individual checks very fast.

If time is a meaningful part of the input data, it should be modeled explicitly. Not getting the testing architecture right usually results in:

*   Software that is hard to change because thousands of test nail existing internal APIs.
*   Software that is hard to change because there are no test to confidently verify absence of unintended breakages.
*   Software that is hard to change because each change requires hours of testing time to verify.

How to architect a test suite goes beyond the scope of this article, but please read [Unit and Integration Tests](https://matklad.github.io/2022/07/04/unit-and-integration-tests.html) and [How To Test](https://matklad.github.io/2021/05/31/how-to-test.html).

Some specific things that are in scope for this article:

Zero tolerance for flaky tests. Strict not rocket science rules gives this by construction — if you can’t merge *your* pull request because someone elses test is flaky, that flaky test immediately becomes your problem.

Fast tests. Again, NRSR already provides a natural pressure for this, but it also helps to make testing time more salient otherwise. Just by default printing the total test time and five slowest tests in a run goes a long way.

Not all tests could be fast. Continuing the ying-yang theme of embracing order and chaos simultaneously, it helps to introduce the concept of slow tests early on. CI always runs the full suite of tests, fast and slow. But the local `makebelive test` by default runs only fast test, with an opt-in for slow tests. Opt in can be as simple as an `SLOW_TESTS=1` environmental variable.

Introduce a [snapshot testing](https://ianthehenry.com/posts/my-kind-of-repl/) library early. Although the bulk of tests should probably use project-specific testing harness, for everything else inline repl-driven snapshot testing is a good default approach, and is something costly to introduce once you’ve accumulated a body of non-snapshot-based tests.

Alongside the tests, come the benchmarks.

I don’t have a grand vison about how to make benchmark work in a large, living project, it always feels like a struggle to me. I do have a couple of tactical tips though.

*Firstly*, any code that is *not* running during NRSR is effectively dead. It is exceedingly common for benchmarks to be added alongside a performance improvement, and then *not* getting hooked up with CI. So, two month down the line, the benchmark either stops compiling outright, or maybe just panics at a startup due to some unrelated change.

This fix here is to make sure that every benchmark is *also* a test. Parametrize every benchmark by input size, such that with a small input it finishes in milliseconds. Then write a test that literally just calls the benchmarking code with this small input. And remember that your build system should have O(1) entry points. Plug this into a `makebelieve test`, not into a dedicated `makebelieve benchmark --small-size`.

*Secondly*, any large project has a certain amount of very important macro metrics.

*   How long does it take to build?
*   How long does it take to test?
*   How large is the resulting artifact shipping to users?

These are some of the questions that always matter. You need infrastructure to track these numbers, and to see them regularly. This where the internal website and its data store come in. During CI, note those number. After CI run, upload a record with commit hash, metric name, metric value *somewhere*. Don’t worry if the results are noisy — you target the baseline here, ability to notice large changes over time.

Two options for the “upload” part:

*   Just put them into some `.json` file in a git repo, and LLM a bit of javascript to display a nice graph from these data.

*   [https://nyrkio.com](https://nyrkio.com) is a surprisingly good SaaS offering that I can recommend.

Serious fuzz testing curiously shares characteristics of tests and benchmarks. Like a normal test, a fuzz test informs you about a correctness issue in your application, and is reproducible. Like a benchmark, it is (infinitely) long running and infeasible to do as a part of NRSR.

I don’t yet have a good hang on how to most effectively integrate continuous fuzzing into development process. I don’t know what is the not rocket science rule of fuzzing. But two things help:

*First*, even if you can’t run fuzzing loop during CI, you can run isolated seeds. To help ensure that the fuzing code doesn’t get broken, do the same thing as with benchmark — add a test that runs fuzzing logic with a fixed seed and small, fast parameters. One variation here is that you can use commit sha as random a seed — that way the code is still reproducible, but there is enough variation to avoid dynamically dead code.

*Second*, it is helpful to think about fuzzing in terms of level triggering. With tests, when you make an erroneous commit, you immediately know that it breaks stuff. With fuzzing, you generally discover this later, and a broken seed generally persists for several commits. So, as an output of the fuzzer, I think what you want is *not* a set of GitHub issues, but rather a dashboard of sorts which shows a table of recent commits and failing seeds for those commits.

With not rocket science rule firmly in place, it makes sense to think about releases.

Two core insights here:

*First* release *process* is orthogonal from software being *production ready*. You can release stuff before it is ready (provided that you add a short disclaimer to the readme). So, it pays off to add proper release process early on, such that, when the time comes to actually release software, it comes down to removing disclaimers and writing the announcement post, as all technical work has been done ages ago.

*Second*, software engineering in general observes reverse triangle inequality: to get from A to C, it is faster to go from A to B and then from B to C, then moving from A to C atomically. If you make a pull request, it helps to split it up into smaller parts. If you refactor something, it is faster to first introduce a new working copy and then separately retire the old code, rather than changing the thing in place.

Releases are no different: faster, more frequent releases are easier and less risky. Weekly cadence works great, provided that you have a solid set of checks in your NRSR.

It is much easier to start with a state where almost nothing works, but there’s a solid release (with an empty set of features), and ramp up from there, than to hack with reckless abandon *without* thinking much about eventual release, and then scramble to decide which is ready and releasable, a what should be cut.

I think that’s it for today? That’s a lot of small points! Here’s a bullet list for convenient reference:

*   README as a landing page.
*   Dev docs.
*   User docs.
*   Structured dev docs (architecture and processes).
*   Unstructured ingest-optimized dev docs (code style, topical guides).
*   User website, beware of content gravity.
*   Ingest-optimized internal web site.
*   Meta documentation process — its everyone job to append to code style and process docs.
*   Clear code review protocol (in whose court is the ball currently?).
*   Automated check for no large blobs in a git repo.
*   Not rocket science rule.
*   Let’s repeat: at **all** times, the main branch points at a commit hash which is known to pass a set of well-defined checks.
*   No semi tests: if the code is not good enough to add to NRSR, it is deleted.
*   No flaky tests (mostly by construction from NRSR).
*   Single command build.
*   Reproducible build.
*   Fixed number of build system entry points. No separate lint step, a lint is a kind of a test.
*   CI delegates to the build system.
*   Space for ad-hoc automation in the main language.
*   Overarching testing infrastructure, grand unified theory of project’s testing.
*   Fast/Slow test split (fast=seconds per test suite, slow=low digit minutes per test suite).
*   Snapshot testing.
*   Benchmarks are tests.
*   Macro metrics tracking (time to build, time to test).
*   Fuzz tests are tests.
*   Level-triggered display of continuous fuzzing results.
*   Inverse triangle inequality.
*   Weekly releases.