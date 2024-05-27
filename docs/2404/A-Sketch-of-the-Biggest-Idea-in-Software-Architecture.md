<!--yml
category: 未分类
date: 2024-05-27 13:26:12
-->

# A Sketch of the Biggest Idea in Software Architecture

> 来源：[https://www.oilshell.org/blog/2022/03/backlog-arch.html](https://www.oilshell.org/blog/2022/03/backlog-arch.html)

| [blog](/blog/) | [oilshell.org](/)

# A Sketch of the Biggest Idea in Software Architecture

2022-03-12 (Last updated 2022-05-25)

This post was called *Backlog: Software Architecture* until I edited it and saw a coherent theme emerge.

It elaborates on [narrow waists](/cross-ref.html?tag=narrow-waist#narrow-waist): an idea in #[software-architecture](/blog/tags.html?tag=software-architecture#software-architecture) that relates to networking, operating systems, language design, compilers, and distributed systems.

Another title I considered is *An Overview of Software Composition at Runtime*. That is, you can contrast these two styles of building software out of parts:

1.  Fine-grained static types, build-time composition, static linking, [APIs](/cross-ref.html?tag=API#API), and version numbers
2.  Coarse-grained **"waists"**, runtime composition, [ABIs](/cross-ref.html?tag=ABI#ABI), [IPC](/cross-ref.html?tag=IPC#IPC), and versionless protocols

Many programmers are familiar with the first style. This post is about the **second style**, which you see at large scales and long time horizons.

This post is **long and dense** with links, so you may want to read it in multiple sittings. Let me know what you think [in the comments](https://old.reddit.com/r/oilshell/comments/tcy7ko/a_sketch_of_the_biggest_idea_in_software/?)! I especially welcome references to similar material.

## Background

I'm happy that there was great discussion on the last post, [The Internet Was Designed With a Narrow Waist](//www.oilshell.org/blog/2022/02/diagrams.html):

I wrote about abstract ideas, but readers understood and applied them. And a reader answered my question about the history of the [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist), which I repeat in the *Call to Action* below.

On the other hand, there were a few responses that exhibited exactly the misconceptions I want to push back on. In particular, the lack of consideration for tradeoffs:

*   **Local** convenience vs. **global** economy, flexibility, generality, and extension
*   The **code** perspective vs. the **system** perspective
*   Fine-grained **types** vs. coarse-grained **"waists"**

To be convincing about this, I would dive into examples: show code, analyze existing designs, and propose new designs. I collected a great deal of material on [the wiki](https://github.com/oilshell/oil/wiki/Perlis-Thompson-Principle) and in [Zulip](/cross-ref.html?tag=Zulip#Zulip).

But I probably shouldn't spend months writing and arguing about #[software-architecture](/blog/tags.html?tag=software-architecture#software-architecture). It's better to **build** something with the principles I'm espousing.

So I'm squeezing many topics into this single post. I state the main points, with some justification.

## Motivating Design Questions

To be concrete, here are some questions that these ideas will help us with:

1.  Should shells have **two tiers**?

    *   Both external processes and internal "functions"? Both pipelines of bytes and pipelines of structured data?
    *   I argue that processes and byte streams should still be "primary" because it makes the shell more interoperable and useful. They are both fundamental [narrow waists](/cross-ref.html?tag=narrow-waist#narrow-waist). Last January: ["Shells Should Shell Out"](//www.oilshell.org/blog/2021/01/philosophy-design.html#shells-should-shell-out).
2.  Is [JSON](/cross-ref.html?tag=JSON#JSON) the new narrow waist for shell?

    *   It's **a** narrow waist, but it's not as universal as text or byte streams. For example, HTML and CSV are not the same as JSON, and they shouldn't be.
3.  How can we design a better distributed OS?

4.  Is [Docker](/cross-ref.html?tag=docker#docker) designed well? How could it be improved?

    *   I bring up this example because I've seen the claim that "the Unix philosophy is obvious" and has been absorbed into standard practice.
    *   This couldn't be further from the truth: [Docker](/cross-ref.html?tag=Docker#Docker) is a recent design, and its style is profoundly anti-Unix. (Oil's build now uses [podman](/cross-ref.html?tag=podman#podman), which is a nice, compatible improvement.)
    *   But despite its design, Docker solved a real problem, and has [notable innovations](https://lobste.rs/s/ftkfnu/secure_containerized_browser#c_r6xgbh).

So I believe the ideas below are relevant to the biggest forces and developments in the industry. I'm glad that Docker is being "refactored away" into something more Unix-y on two fronts: into [OCI](/cross-ref.html?tag=OCI#OCI) by Red Hat and others, and out of Kubernetes. (Related: [Docker's Second Death](https://www.tariqislam.com/posts/kubernetes-docker-dep/))

## What Is a Narrow Waist?

Most readers understood the [last post](//www.oilshell.org/blog/2022/02/diagrams.html): I borrowed the [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist) term from networking and extended it to software.

But it's become clear to me that not all narrow waists are alike. It's worth distinguishing these categories, and more:

1.  Small, simple mechanisms like the [Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol), [UTF-8](/cross-ref.html?tag=utf8#utf8), and [JSON](/cross-ref.html?tag=JSON#JSON).
    *   These waists are "narrow" in a strong sense. It's not a coincidence that Jon Postel, Ken Thompson, and Doug Crockford were their "editors" or creators.
2.  Language standards like [POSIX shell](/cross-ref.html?tag=posix-shell-spec#posix-shell-spec), JavaScript, and C++.
    *   These are big and hard to reimplement. I know this first hand from working on Oil!
    *   These are narrow waists because they solve the interoperability problem of {user programs ...} × {language implementations ...}
3.  "Accidental" waists like [Win32](https://en.wikipedia.org/wiki/Windows_API) and [x86](https://en.wikipedia.org/wiki/X86).
    *   Their evolution isn't guided by a standards body. They're also big and hard to reimplement.
4.  APIs like [LLVM](https://llvm.org). As the home page says, LLVM isn't a virtual machine. It's really a software library that changes with each release, requiring consumers to change their code. This makes it different than the other narrow waists, which are more about runtime composition.
5.  ... ?

So it's worth being more specific, and the posts below will refine definitions and explore related concepts.

The clearest objection I see to the narrow waist idea is that a narrow waist is simply a **standard**! Standards enable interoperability.

But standards have to come from somewhere. A narrow waist may or may not become a standard. Also, LLVM is not a standard, and isn't intended to be one.

The hourglass metaphor also suggests why the idea is powerful, and what to aim for. You want something small that interfaces with many other things.

### Precisely Defining "The Unix Philosophy"

I spent a week drafting a post called *Diagrams of Three Narrow Waists in Unix*. The first sentence is:

> Have you heard vague claims about "the Unix Philosophy", and are you confused or skeptical about it?

This is a valuable post, because surprisingly the [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist) idea **says something new** and more specific about Unix! I justify this with references, including the classic ones [on this Wikipedia page](https://en.wikipedia.org/wiki/Unix_philosophy).

I have diagrams of these 3 narrow waists:

1.  **Processes**
    *   {native code, shebang script, shell function, ...} × { start, kill, pipe, redirect, run with privileges, ... }
2.  **File Descriptors**
    *   {file, pipe, terminal, socket, ... } × {read, write, ioctl, ... }
3.  Tree-Shaped **Namespaces** of unstructured data (file systems)
    *   {disk, SSD, memory with tmpfs, file with loopback, ... } × { ls, mount, version with git, serve over HTTP, ... }

The diagrams show that Unix uses multiple narrow waists to achieve dynamic and extensible **polymorphism**.

The file descriptor case shows both sides of the tradeoff. You don't statically know what syscalls are valid on a descriptor. You also don't know what errors you'll get! I re-learned this lesson with:

*   [A bug in Oil 0.9.6](//www.oilshell.org/blog/2022/01/notes-themes.html#oil-096-on-december-30th): `write()` can fail with `EISDIR` if the descriptor returned by `open()` points to a directory.
*   [Bugs in Hello World](https://blog.sunfishcode.online/bugs-in-hello-world/): `write()` can fail with `ENOSPC` if the descriptor points to a disk file. Python 2 has the bug but Python 3 fixed it.

Nevertheless, the polymorphic design of file descriptors makes Unix compose, and is one reason why shell is powerful! I give examples in the post.

Go addresses this problem with single function interfaces like `Reader` and `Writer`, and more generally the `-er` pattern. Here's an interesting quote:

> It would be nice if Haskell had [open polymorphism], possibly using Go as a model.
> 
> — [Philip Wadler: Featherweight Go](https://youtu.be/Dq0WFigax_c?t=912)

More:

*   I mention the relationship to Plan 9 (fixing the composition bugs in Unix) and REST (the [uniform interface constraint](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints)).
*   I link to two important academic papers, and related analysis of Unix.
*   I also noticed that **Lines of Text** is distinct narrow waist from **Text**, which the [last post](//www.oilshell.org/blog/2022/02/diagrams.html) depicted.
    *   In fact Oil's [QSN](/cross-ref.html?tag=QSN#QSN) format takes advantage of this narrow waist, while the GNU's NUL-delimited format doesn't. (This the format `xargs -0` accepts , mentioned in [the xargs post](//www.oilshell.org/blog/2021/08/xargs.html).)
    *   In particular, `wc -l`, `head`, `tail`, and `tail -f` work "for free" with [QSN](/cross-ref.html?tag=QSN#QSN), but you need more code like to support the NUL-delimited format, like `head -z` and `tail -z`.

This post isn't done, but it's the one I want to publish the most.

### Characteristics of Narrow Waists

In software, the most important characteristic of a [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist) is that it reduces an [O(M x N) code explosion](/cross-ref.html?tag=m-by-n-explosion#m-by-n-explosion), allowing interoperability and code reuse.

I also realized that there are two distinct senses of the word "narrow":

1.  In terms of architectural **connection** (topology).
    *   For example, applications and physical networks are decoupled by the the Internet's narrow waist. They don't interface directly with each other.
2.  In terms of the **size** of the concept.
    *   IP is a small concept, and Unix is a small handful of concepts.
    *   But the web is a large set of concepts (HTTP, HTML, SVG, etc.). C++ and shell are also large.

So this issue deserves some more thought, and perhaps more terminology.

Here are more ways to characterize narrow waists:

1.  They are **compromises**. They make systems economical and possible, not necessarily optimal.
    *   If you have a small or specialized network, you can design something more efficient than the Internet.
2.  They arise through a mix of explicit **design** and implicit **evolution**.
    *   Both the Internet and the web were designed and subject to evolution. But I'd say the web evolved more.
3.  The design can be done **well or poorly**. The evolution can be guided or haphazard.
    *   [JSON](/cross-ref.html?tag=JSON#JSON) was an explicit design, and it's much better than CSV.
    *   We should try to do better at design, because the resulting network effects mean we often get "stuck" with bad designs.

Regarding evolution:

4.  Narrow waists can last for decades, usually evolving in a **versionless** manner.
    *   For example, Unix shell is one of the oldest languages in wide use, and there's continuous compatibility between [Thompson shell](/cross-ref.html?tag=thompson-shell#thompson-shell), [Bourne Shell](/cross-ref.html?tag=bourne-shell#bourne-shell), [Korn shell](/cross-ref.html?tag=ksh#ksh), [bash](/cross-ref.html?tag=bash#bash), [OSH](/cross-ref.html?tag=osh-language#osh-language), and [Oil](/cross-ref.html?tag=oil-language#oil-language).
    *   A narrow waist has an amount of **inertia** that's proportional to the amount of functionality that hinges on it.
5.  But narrow waists can also **move**!
    *   TCP/IP → HTTP
    *   POSIX C APIs → Linux x86 ABI
6.  They're subject to extreme **economic pressure** and network effects. For example:
7.  The downside of inertia is that narrow waists can **inhibit innovation**.
8.  Narrow waists are often **overextended** to new applications.
    *   The web was arguably overextended from a network of hyperlinked documents to an application delivery platform ([single-page apps](https://en.wikipedia.org/wiki/Single-page_application) in JavaScript)
    *   It was also extended to a desktop UI framework via [Electron](https://www.electronjs.org/).
    *   Linux was arguably overextended to embedded devices, especially those with real-time requirements.

Some recent narrow waists include [Docker](/cross-ref.html?tag=docker#docker) / [OCI](/cross-ref.html?tag=OCI#OCI), the [Language Server Protocol](https://en.wikipedia.org/wiki/Language_Server_Protocol), and [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly). I should be more specific about their varying degrees of success with respect to design and user adoption. For example, I think WebAssembly is useful, but [less general than what's been recently claimed](https://news.ycombinator.com/item?id=28581634). It's a deep compromise which involves winners and losers.

### Fallacies

Here are some common objections to the idea.

(1) *Textual data is hard to manipulate with programs*.

This is **not** an objection to the narrow waist principle! The main claims of the principle are about interoperability and economy of implementation.

I want to make a **Simple vs. Easy** argument. Narrow waists are *simple* in Rich Hickey's terms (not "complected"), but not necessarily *easy* to use. For example, Unix shell can be hard to learn, but its power results in a small, extensible operating system.

(2) *The web is really messy, and thus unreliable*.

I make a strong **Messy vs. Stable** distinction. Messy systems aren't necessarily unreliable. Quite the contrary — the need for stability is the **cause of** the mess! Continuous backward compatibility (like the the many iterations of CSS) makes a mess, but keeps the system working.

This relates to another concept I've been having a hard time describing: **versionless evolution**, which I describe below.

### Related Ideas

We can understand the [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist) more precisely by relating it to these ideas:

1.  [Metcalfe's Law](https://en.wikipedia.org/wiki/Metcalfe%27s_law) states that the value of a network is proportional to N², where N is the number of nodes.
    *   This is related to, but distinct from, the M × N *architectural* connections of a narrow waist. Architectural connections are not network node connections.
    *   Thinking about the architectural hierarchy of narrow waists may clarify this. For example, CSV, JSON, and HTML are narrow waists at "level 1". And each of them **is** literally text, which is at "level 0". Generic operations are "inherited", which makes the system smaller. (This idea really needs diagrams.)
2.  The Internet Protocol follows the [End-to-End Principle](https://en.wikipedia.org/wiki/End-to-end_principle) **and** it's a [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist).
    *   This doesn't mean the two principles are the same. I view the narrow waist as more descriptive and predictive when applied to software.

## Examples and Elaboration

Let's apply these principles to real world systems. Again, I claim the [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist) is the **most important** idea in software architecture, because it describes the biggest and longest-lived systems.

### The Web Evolved In A Versionless Manner

I'd like to elaborate on the "versionless" property of many narrow waists. You can contrast two philosophies of versioning:

1.  Version numbers that indicate breaking changes, e.g. [Semantic Versioning](https://semver.org/).
    *   Linux distros and package managers [like NPM](https://docs.npmjs.com/about-semantic-versioning) like often pair semantic versioning with ad hoc constraint solvers to find a set of compatible versions for dependencies. This model can be **brittle** because you may end up running a set of versions that's never been tested together.
2.  Continuous backward compatibility, i.e. **versionless** evolution.
    *   There are also version numbers here, but they indicate feature additions rather than breaking changes.

For example, the web doesn't have incompatible versions, and [JSON](/cross-ref.html?tag=JSON#JSON) was explicitly designed by Doug Crockford to be versionless.

History proves this rule. In [Don't Break X](//www.oilshell.org/blog/2021/12/backlog-project.html#three-analogies-dont-break-x), I mentioned that XHTML and ECMAScript 4 both tried to **break the web** with radical changes, but they failed because of the inertia of narrow waists.

In contrast, HTML5 and ECMAScript 5 evolved the web in a compatible way. We should study and disseminate the history of the web avoid repeating mistakes we get "stuck with".

* * *

Here's a good way of thinking about versionless evolution:

> Relaxing a requirement should be a compatible change. Strengthening a promise should be a compatible change.
> 
> — Rich Hickey in [Maybe Not](https://www.youtube.com/watch?v=YR5WdGrpoug) (2018, YouTube)

Examples:

*   HTML5 defined `<hr>` and `<hr />` to mean the same thing, whereas previous versions of HTML were stricter. (This is the [self-closing tag issue](http://xahlee.info/js/html5_non-closing_tag.html).) So HTML5 **relaxes a requirement** on web page authors, which is a compatible change.
*   Adding a new feature **strengthens the promise** that the browser makes to the web page author. For example, HTML5 added a `<video>` tag, which is a compatible change.

### Bytes and Text Are Essential Narrow Waists

I have a recurring debate about "text vs. fine-grained types", mostly with people who are frustrated with ad hoc, incorrect #[parsing](/blog/tags.html?tag=parsing#parsing) in shell.

I think that a shell with support for [JSON](/cross-ref.html?tag=JSON#JSON), [QSN](/cross-ref.html?tag=QSN#QSN), [QTT](/cross-ref.html?tag=QTT#QTT) and HTML will address this problem. It will reduce the amount of parsing in shell programs, and make it correct.

I also claim that parsing is an O(M + N) problem, while types can create O(M × N) problems — and often do.

To give more color on that, here's an important comment which I mentioned in [January](//www.oilshell.org/blog/2021/01/audio-and-graphics.html), [June](//www.oilshell.org/blog/2021/06/hotos-shell-panel.html#conclusion), [July](//www.oilshell.org/blog/2021/07/blog-backlog-1.html#concepts), and [August](//www.oilshell.org/blog/2021/08/history-trivia.html#the-first-paper-about-unix-shell-thompson):

I make the M × N argument, and use concrete examples like IntelliJ and WebAssembly's text format:

> You have M formats and N operations, and writing M × N tools is infeasible, even for the entire population of programmers in the world.

I also note the tradeoff:

> It's not an absolute; in reality people do try to fill out every cell in the M × N grid [in certain domains]. They get partway there, and there are some advantages to that for sure.

I also quote Rust designer Graydon Hoare on text. While his "rant" is mostly about the information density of text, it also touches on the wide range of operations that text supports.

> [Text] can be compared, diffed, clustered, corrected, summarized and filtered algorithmically. It permits multiparty editing. It permits branching conversations, lurking, annotation, quoting, reviewing, summarizing, structured responses, exegesis, even fan fic. The breadth, scale and depth of ways people use text is unmatched by anything.

I note a problematic M × N explosion in code **generated** by protocol buffers (as opposed to source code).

For example, equality becomes **schema-dependent** rather than generic. This is worth it in many systems, but it's a tradeoff.

### Slogan: Text Is The Only Thing You Can Agree On

Here are two variations of a slogan. It's meant to drive home the point of text as a narrow waist.

> The lowest common denominator between a [PowerShell](https://en.wikipedia.org/wiki/PowerShell), [Elvish](https://elv.sh/), [Rash](https://docs.racket-lang.org/rash/), and [nushell](https://www.nushell.sh/) script is a Bourne shell script (and eventually an Oil script).

This is because each alternative shell chooses a *different* kind of structured data as its narrow waist (.NET objects, tree-structured data, Racket data structures, and tables, respectively). **Text** is the most structured format they all agree on, and **shell** is the language of coarse-grained composition with text.

I predict that this will be a real thing, and isn't theoretical! I have no doubt that there are already bash scripts invoking PowerShell scripts out there, and more complex agglomerations will arise as alternative shells become popular.

It doesn't mean those shells aren't worth using, or more potentially more convenient. But it highlights the need for a better Bourne-style shell.

and

A second phrasing:

> The lowest common denominator between a [Common Lisp](https://common-lisp.net/), [Clojure](https://clojure.org/), and [Racket](https://racket-lang.org/) program is a Bourne shell script (and eventually an Oil script).

Again, these languages are similar, but have incompatible data models. (It's not just the compound data structures; Clojure's notion of numbers and strings is borrowed from the JVM.)

These two slogans are really another way of phrasing a slogan from the [last post](//www.oilshell.org/blog/2022/02/diagrams.html):

> Unix is equally inconvenient for every programmer, and that's a good thing.

### CSV, JSON, HTML - Tables, Records, Documents

The full title of this post is:

> CSV, JSON, and HTML are Different Because Tables, Records, and Documents Are Different

This is again pushing back on the notion that [JSON](/cross-ref.html?tag=JSON#JSON) is "the" new narrow waist of shell. Tables and documents are essential structures in software, and expressing them in JSON is awkward.

I can give examples of this, e.g.

```
{"name": "alice", "age": 42}
{"name": "bob", "age": 43} 
```

and

```
 ["a", {"href": "/home"}, ["anchor text"]] 
```

This framing comes from the paper [Unifying Tables, Objects, and Documents](https://www.microsoft.com/en-us/research/publication/unifying-tables-objects-and-documents/) (Meijer and Schulte, 2003), but the technical details differ.

### Tradeoffs Between Dynamic and Static Types (FAQ)

Text as a narrow waist is at odds with fine-grained, static types. My goal is to highlight tradeoffs, and analyze situations where each style is natural and efficient.

Many programmers seem to think there is no tradeoff — or at least they *say* that on the Internet. I believe that when they create working systems they often use the dynamic, coarse-grained view!

[This recent comment](https://old.reddit.com/r/ProgrammingLanguages/comments/t0lzeq/types_considered_harmful_pdf_2008/hyfhhu2/) links to typical responses, which by now form a FAQ:

Here's a related, fantastic video which I want to signal-boost:

I don't know the F# language, but it apparently has a very Clojure-like view of data, despite being statically typed. The "type provider" mechanism addresses the problem of types that are only available runtime, e.g. in SQL schemas, or implicitly in JSON and CSV files.

Overall, the fallacy is that we use dynamic typing when we're "too lazy to write down the types". There are many useful programs that aren't 100% statically typed, and I claim this trend is increasing. (Slogan: *Poorly Factored Software is Eating the World*.)

The real issues are **scale** in space and time, heterogeneity, and extensibility!

## Refinements

In the discussion of the extensibility post, I said that I'm getting at *theory and guidelines for runtime composition and versionless evolution*. Shell is about software composition at runtime, as opposed composition via static linking.

So in addition to the [Perlis-Thompson Principle](/cross-ref.html?tag=perlis-thompson#perlis-thompson), [narrow waists](/cross-ref.html?tag=narrow-waist#narrow-waist), and [O(M × N) code explosions](/cross-ref.html?tag=m-by-n-explosion#m-by-n-explosion), here are some more concepts that are worth exploring.

### Projection to Waists

I need a name for the idea of **code reuse** by changing the representation of data to a narrow waist. Examples:

1.  The `/proc` file system projects kernel metadata onto the narrow waist of the file system.
    *   Now you can use existing tools like `ls` and `open()` to explore the state of processes.
2.  Any system that uses [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) is also like this, e.g. Michael Greenberg's [File File System](https://mgree.github.io/ffs/) projects [JSON](/cross-ref.html?tag=JSON#JSON) onto a virtual file system. This allows reuse of tools like `cd` and `ls`.
3.  The [gron](https://github.com/tomnomnom/gron) tool projects tree-like [JSON](/cross-ref.html?tag=JSON#JSON) onto the narrow waist of "lines of text". This allows reuse of tools like [grep](/cross-ref.html?tag=grep#grep) and [awk](/cross-ref.html?tag=awk#awk).

Notice that [JSON](/cross-ref.html?tag=JSON#JSON) is a narrow waist, but it's been projected onto two others: the file system, and lines of text. Which one is appropriate (if any) depends on what set of tools helps you solve a particular problem.

### Emulation of Waists

This is the most straightforward one. As mentioned above, there's a big incentive for Windows to emulate Linux, and vice versa. The platform gets thousands and thousands of applications "for free".

Another example is when Illumos borrowed FreeBSD's Linux syscall ABI emulation in order to run user-uploaded [Docker](/cross-ref.html?tag=docker#docker) containers. This is **dynamic, runtime** composition with [ABIs](/cross-ref.html?tag=ABI#ABI), not static composition by compiling code against kernel [APIs](/cross-ref.html?tag=API#API) expressed as C header files.

### Extension of Waists

I think "waist extension" is a good term for the following ideas:

*   The web is a humble and brilliant **extension of Unix**, adding simple networking and hyperlinks (from [this recent comment](https://news.ycombinator.com/item?id=26865164); [a longer comment](https://news.ycombinator.com/item?id=6131335) from 2013).
    *   This design was not obvious! There is a long history of hypertext systems that were **not** built on Unix. It would be nice to research explain the history.
    *   [apenwarr in 2006](https://apenwarr.ca/log/20061201): *The web works because it mostly just paraphrases Unix's cleverness*. ("Working" is an important property that a lot of software lacks.)
    *   [Web Sites Are Naturally Made With Shell Scripts](//www.oilshell.org/blog/2020/02/good-parts-sketch.html#web-sites-are-naturally-made-with-shell-scripts) (2020)
*   [git](/cross-ref.html?tag=git#git) is a distributed extension of the Unix file system.
    *   Again, earlier systems were **not** like this, e.g. CVS and SVN. As with the web, the idea is only obvious in hindsight. It's obvious when it becomes "air", but someone had to invent it.
    *   git has a messy UI, but a clean [narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist). This article seems like a good explanation, but there may be better ones: [Git is a purely functional data structure](https://blog.jayway.com/2013/03/03/git-is-a-purely-functional-data-structure/).

### Composition Between Waists

Unix has multiple waists: processes, file descriptors, file systems, lines of text, unstructured text, and bytes. Each of them allows M × N things to compose, but they also must compose **amongst themselves**.

"A few things that compose" is tantamount to the [Perlis-Thompson Principle](/cross-ref.html?tag=perlis-thompson#perlis-thompson). When I started this series, I wasn't sure if this term and "narrow waist" were necessary — maybe they're both tantamount to "simplicity".

But after working through examples, I see them as distinct but related. So it would be nice to write more clearly about how narrow waists relate. This seems like a distinct style of long-lived architecture.

Again, let me know if you have references. I **don't** want to write about ideas that other people have already explained, or invent new terms when there are existing ones.

### Addition of Waists

It's common to create a new, larger narrow waist out of existing smaller ones.

For example, the [Language Server Protocol](https://en.wikipedia.org/wiki/Language_Server_Protocol) uses JSON-RPC [for notifications and responses](https://microsoft.github.io/language-server-protocol/overviews/lsp/overview/).

In turn, [JSON-RPC](https://en.wikipedia.org/wiki/JSON-RPC) is built on top of [JSON](/cross-ref.html?tag=JSON#JSON) and a transport like TCP/IP or pipes.

### Hierarchy Among Waists

There's a clear hierarchy among data representations in Unix, which affects which operations are valid:

*   In the [last post](//www.oilshell.org/blog/2022/02/diagrams.html), I mentioned that **text** (ASCII, UTF-8) is a special case of **bytes**, and inherits [operations on bytes](//www.oilshell.org/blog/2022/02/diagrams.html#bytes-flat-files).
*   I mentioned above the **Lines of Text** is a special case of **text**.
*   Likewise, JSON, CSV, and HTML are all **text**. and inherit [operations on text](//www.oilshell.org/blog/2022/02/diagrams.html#text-narrow-waist-of-unix-architecture).

## Call to Action

### Jon Postel Made the Internet's Waist Narrow

After a very helpful reader e-mail, I added [an appendix to the last post](//www.oilshell.org/blog/2022/02/diagrams.html#addendum-february-28th). I want to transcribe the first 10 minutes of [the video of Van Jacobsen](https://www.youtube.com/watch?v=69p78tfm29o). He describes the role of [Jon Postel](https://en.wikipedia.org/wiki/Jon_Postel) as Internet specification editor — specifically, his **relentless, decades-long drive for minimalism** in the Internet's design:

> This narrow waist is not something that God gives you. It's something that you make. It's hard engineering.

and

> We unfortunately don't have a lot of Jon Postel's in the world. It would be nice to get one on nearly every project.

This reminds me of the sentiments by Ken Thompson quoted in [Unix Shell: History and Trivia](//www.oilshell.org/blog/2021/08/history-trivia.html). They share a taste for minimalism that unlocks enormous functionality.

I also enjoyed reading these memorials:

The point is that **people** have to behave differently to create valuable, interoperable systems!

## Conclusion

After [more than a year](//www.oilshell.org/blog/2021/12/review-arch.html) of circling these #[software-architecture](/blog/tags.html?tag=software-architecture#software-architecture) topics, I feel pretty good about them. They've informed Oil's design and will continue to. It helps to be precise about definitions and support claims with examples.

I hope this outline was also useful to you. I wish I could have written a shorter post, but I didn't have time :-)

And again, [please send](https://old.reddit.com/r/oilshell/comments/tcy7ko/a_sketch_of_the_biggest_idea_in_software/?) related references. They will help with future articles on these ideas. (I was surprised that the [history of the narrow waist in networking](//www.oilshell.org/blog/2022/02/diagrams.html#whats-the-history-of-this-idea) is not well documented or agreed upon.)

* * *

Now I want to switch gears to something more "tactical": translating Oil to C++! That has been on hold for a full year, since [the last milestone in March 2021](https://www.oilshell.org/blog/2021/03/release-0.8.8.html).

I also want to expand the project. Please donate to my new [Github Sponsors](https://github.com/sponsors/oilshell) page if you think we need a new, principled shell. I'll ask for donations again in upcoming blog posts. All of the money will go to contributors and "employees", **not** to me!

## Appendices

### The Lambda Calculus Is a Narrow Waist

This a "fun" post to help us with the definition. It's based on this quote from chapter 5 of [Types and Programming Languages](https://mitpress.mit.edu/books/types-and-programming-languages):

> [The importance of the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus)] arises from the fact that it can be viewed simultaneously as a simple programming language **in which** computations can be described and as a mathematical object **about which** rigorous statements can be proved.

*   In other words, it reduces an M × N explosion of {arbitrary algorithms ...} × {inductive cases in proofs about them}.
*   **Derived forms** with respect to lambda calculus are like **intermediate representations** in compilers. Proofs about languages are laborious for a very similar reason that implementing compilers is laborious!

### Wiki, Zulip

This post was long, but there are still important things I left out. As mentioned in the *Motivating Design Questions* section, these ideas relate to the design of foundational cloud software like Docker and Kubernetes.

But I want Oil to be in better shape before I continue writing about these topics. For now here are my Wiki pages and Zulip links. (I really wish I had a single brainstorming and research app.)

I also mentioned the #[containers](https://oilshell.zulipchat.com/#narrow/stream/308821-containers) Zulip stream in December. Here is a (sloppily sketched) overview thread:

*   [Docker Summary](https://oilshell.zulipchat.com/#narrow/stream/308821-containers/topic/Docker.20Summary/near/264915992). The Value of Docker, and its anti-Unix design. Other threads substantiate these claims with experience.