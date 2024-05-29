<!--yml
category: 未分类
date: 2024-05-29 12:49:32
-->

# Rust developers at Google twice as productive as C++ teams • The Register

> 来源：[https://www.theregister.com/2024/03/31/rust_google_c/](https://www.theregister.com/2024/03/31/rust_google_c/)

Echoing the past two years of Rust evangelism and C/C++ ennui, Google reports that Rust shines in production, to the point that its developers are twice as productive using the language compared to C++.

Speaking at the [Rust Nation UK Conference](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=26575) in London this week, Lars Bergstrom, director of engineering at Google, who works on Android Platform Tools & Libraries, described the web titan's experience migrating projects written in Go or C++ to the Rust programming language.

Bergstrom said that while [Dropbox in 2016](https://blog.rust-lang.org/2016/05/16/rust-at-one-year.html) and [Figma in 2018](https://www.figma.com/blog/rust-in-production-at-figma/) offered early accounts of rewriting code in memory-safe Rust - and doubts about productivity and the language have subsided - concerns have lingered about its reliability and security.

"Even six months ago, this was a really tough conversation," he said. "I would go and I would talk to people and they would say, 'Wait, wait you have an `unsafe` keyword. That means we should all write C++ until the heat death of the Universe.'"

But there's been a shift in awareness across the software development ecosystem, Bergstrom argued, about the challenges of using non-memory safe languages. Such messaging is now coming from government authorities in the US and other nations who understand the role software plays in critical infrastructure.

The reason is that [the majority](https://alexgaynor.net/2020/may/27/science-on-memory-unsafety-and-security/) of security vulnerabilities in large codebases can be traced to memory security bugs. And since Rust code can largely if not totally avoid such problems when properly implemented, memory safety now looks a lot like a national security issue.

Back in September 2022, Microsoft Azure CTO Mark Russinovich argued that software projects that might have been started in C/C++ [should use](https://www.theregister.com/2022/09/20/rust_microsoft_c/) Rust instead. That recommendation now extends beyond greenfield projects to calls for reworking old code written in non-memory safe languages.

Earlier this year, Microsoft put out a call for developers to help [port](https://www.theregister.com/2024/01/31/microsoft_seeks_rust_developers/) its own C# code to Rust. And the Internet Security Research Group (ISRG)'s Prossimo project has been rewriting core open source elements of critical libraries (eg, NTP, DNS, TLS) in Rust for the sake of [memory safety](https://www.memorysafety.org/).

There's been pushback from C++ creator Bjarne Stroustrup and others. In response to a November 2022 NSA [memo](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/1/CSI_SOFTWARE_MEMORY_SAFETY.PDF) [PDF] urging memory safety, Stroustrup [argued](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2739r0.pdf) [PDF] that C++, with proper tooling, can match Rust's memory safety guarantees "at a fraction of the cost of a change to a variety of novel 'safe' languages."

And in February, when the US Office of the National Cyber Director published [a report](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf) [PDF] on security software, some of those leaving [public comments](https://www.regulations.gov/document/ONCD-2023-0002-0001/comment) observed that memory safety is a subset of broader software security challenges, and should not be viewed as an answer to everything.

Carnegie Mellon's Software Engineering Institute, for example, emphasized that all programming languages have trade-offs and the choice of programming language should come down to whether it is fit for purpose.

"Most memory-safe languages do not prioritize timing performance, and therefore are not appropriate for use cases that have strict performance and timing requirements," the software group [claimed](https://www.regulations.gov/comment/ONCD-2023-0002-0068).

"Also, as with any programming language, developers must learn the correct mechanics of the language, such as syntax, semantics, constructs, idioms, and tools. Otherwise, the result might be an unintended tradeoff of less memory-related vulnerabilities for more vulnerabilities or defects of other types."

While there's more to software security than memory safety, the cost advantage that Stroustrup claims can be realized by sticking with existing C++ infrastructure now faces counter-examples from Rust adopters like Google.

### No loss in productivity - quite the opposite

At the Chocolate Factory, turning Go code, which is [considered memory safe](https://www.memorysafety.org/docs/memory-safety/) but not as performant, into Rust has shown noteworthy benefits.

"When we've rewritten systems from Go into Rust, we've found that it takes about the same size team about the same amount of time to build it," [said](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=27048) Bergstrom. "That is, there's no loss in productivity when moving from Go to Rust. And the interesting thing is we do see some benefits from it.

"So we see reduced memory usage in the services that we've moved from Go ... and we see a decreased defect rate over time in those services that have been rewritten in Rust – so increasing correctness."

More significant, Bergstrom said, is the comparison of rewrites of C++ code into Rust.

"In every case we've seen a decrease by more than 2x in the amount of effort required to both build the services in Rust as well as maintain and update those services written in Rust," he [said](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=27094).

"And so that's a really huge thing for us because C++ code is very expensive. These are large teams. It's a lot of work. There's a lot of risk."

Bergstrom said Google has a similar migration underway moving developers from Java to Kotlin and that the time it takes to retrain developers in both cases – Java to Kotlin and C++ to Rust – has been similar. That is, in two months about a third of devs feel they're as productive in their new language as their old one. And in about four months, half of developers say as much, based on anonymous internal surveys.

A bit more than half of his developers say that Rust is easier to review, according to Bergstrom.

"When we sort of look into why that is," he said, "we get to sort of the most incredible question of the survey, the one that kind of blew all of us away, which is the confidence that people have in the correctness of the Rust code that they're looking at – so in comparison to code in other languages, how confident do you feel that your team's Rust code is correct?"

The answer, Bergstrom said, was 85 percent.

"That is a massive number," he said. "I could not get 85 percent of this room to agree that we like M&M's. Eight-five percent of people believe that their Rust code is more likely to be correct than the other code within their system. … I've been through more than one language survey in my life and I've never seen those kinds of numbers before." ®