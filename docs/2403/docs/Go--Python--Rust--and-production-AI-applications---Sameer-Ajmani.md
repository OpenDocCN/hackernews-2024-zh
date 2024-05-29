<!--yml
category: 未分类
date: 2024-05-27 14:52:19
-->

# Go, Python, Rust, and production AI applications – Sameer Ajmani

> 来源：[https://ajmani.net/2024/03/11/go-python-rust-and-production-ai-applications/](https://ajmani.net/2024/03/11/go-python-rust-and-production-ai-applications/)

In this article, I’ll talk about Go, Python, and Rust, and each language’s role in building AI-powered applications.

**Python was the first programming language I ever loved, and Go was the second**. But let’s start at the beginning …

I discovered the power of programming in my high school linear algebra class. We were learning the Simplex method for linear programming and were expected to memorize the algorithm for an exam. The steps were entirely mechanical but tedious and error prone. I had noticed that our TI-81 calculators supported programming, so I asked the teacher whether I could use it for the algorithm. She agreed. It took me hours to write the program on the calculator, but with it I completed the 45-minute exam in 5 minutes and walked out like a boss.

Pascal was the first language I learned formally, then Scheme, C, and C++. As a grad student, I learned Java and became enamored with its safety, memory management, and ability to dynamically load bytecode. As a teaching assistant for 6.170, MIT’s software engineering course, I used Java’s dynamic class loading to automatically test and grade students’ programming assignments.

Then I found Python, and it was something truly special. **Python was so simple, so light on the page, so readable—it felt like programming in natural language**. I started using Python everywhere I could. I even used Python to answer programming questions during my Google interviews, so much so that the interviewers asked that I write a header file for a double-ended queue to prove that I actually knew C++.

Once I joined Google in 2004, I was back to writing C++ most of the time. It was fine, but not *joyful*. I missed the simplicity and lightness of Python, but I understood the need for efficiency demanded by “Google scale”. But then, [in 2010, I found Go](https://www.linkedin.com/pulse/gos-early-growth-2012-2016-sameer-ajmani-oxtjc/), and I felt like someone had finally given me the best of both worlds: a simple, light, joyful language that also provided great performance, reliability, and scale. I was thrilled.

I was not alone. While Go had been designed as an alternative to C++ for building networked services, much of Go’s early adoption came from dynamic language users, particularly from Python and Ruby. [Jeremy Manson’s post](https://www.linkedin.com/pulse/language-policy-google-lets-go-jeremy-manson-ffmac/) describes the challenges Google had running Python in production and how Go provided a way forward. The same thing happened outside Google, with many Python and Ruby developers adopting Go to achieve greater reliability and efficiency while retaining a fast inner loop.

Meanwhile, Python was going through a renaissance. While Python’s use for web backends was declining, its use for data science was *booming*. Scientists had discovered what I did: a light, simple language that made it easy to iterate and ran efficiently enough, especially when Python could delegate heavy numerical computations to C libraries. After scientific computing and “big data” came ML development, and now we have LLM application development. Python is the default language for all of these, and its ecosystem is rich and enormous.

But there’s a problem. Python is amazing for working with data, developing models, and prototyping applications, but it doesn’t scale well. The features of static languages—type checking, compilation to machine code, and more—are what enable **scaling to large programs, large development teams, and large systems**. And production systems require their own ecosystem of libraries and tools to enable developers and operators to work effectively at scale.

We’ve been talking with people who are running AI-powered applications in production, and we keep hearing the same thing: these applications are written in Python, but these organizations don’t want to support Python in production. While Python may become better in production over time, the AI revolution is happening right now, and people want an alternative. I believe that alternative is Go. **LLM-powered applications are primarily about orchestration**: calling into one or more models, in sequence or in parallel, and synthesizing the results—and doing this at scale, with strong support for production operators. Go excels at this.

There are several other languages that might make a claim to suitability here, but I’d like to focus on one: Rust. **Rust is an** ***amazing*** **language**. I won’t say I *love* it (yet), but I *respect* it. Rust provides uncompromising safety and uncompromising speed, even in the context of concurrency. The cost is up front programmer effort: Rust demands the programmer provide enough information for the compiler to understand object lifetimes and sharing, whereas “managed languages” like Go figure this out for you. Go and Rust team members explored this trade-off together in the 2021 article “[Rust vs. Go: Why They’re Better Together](https://thenewstack.io/rust-vs-go-why-theyre-better-together/)”.

Imagine Python, Go, and Rust on a spectrum of languages: as we move from Python to Go then Rust, we get increasing safety and efficiency. As we move in the opposite direction—from Rust to Go then Python—we get increasing ease and accessibility. Furthermore, each language has rich but *different* ecosystems of libraries and tools that make them suitable for different kinds of applications.

Python, Go, and Rust each have their strengths, and **each language has a role to play in building AI-powered systems**. Python is fantastic for the iterative, exploratory process of developing AI models and prototyping applications. Go is great for bringing these applications to production at scale. Rust is superb when speed is paramount, such as in serving AI models.

You’ll notice I didn’t say anything about which language is the most “productive”. That’s because **your productivity with a language or tool is a function of the tool’s suitability to task**. You’ll be more “productive” driving a screw with a screwdriver than a hammer. Pick the right tool for the job.

So if we believe in “prototype in Python, productionize in Go”, how do we make this easy? We are actively researching this area. The #1 thing users tell us is that they need **Go equivalents to the Python libraries used to build AI applications**. Python has an *enormous* ecosystem (pandas, numpy, Pytorch, tensorflow, JAX, notebook integrations, and much more)—but we don’t need to provide *all* of this in Go. We are focused on the subset of libraries that are needed for building *production* AI applications, starting with [LangChainGo](https://github.com/tmc/langchaingo). We’re eager to hear from the community what else you need.

Our goal is for Go to *complement* Python to the benefit of *both* communities. **Let’s build bridges between our ecosystems and communities to make Go and Python “better together”**. Together, we will enable developers to create a new generation of production-grade AI-powered applications.