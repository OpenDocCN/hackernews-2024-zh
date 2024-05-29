<!--yml
category: 未分类
date: 2024-05-29 13:21:43
-->

# Rust Bytes: The Future is Rusty: 2023 Survey Reveals Widespread Adoption

> 来源：[https://weeklyrust.substack.com/p/rust-bytes-the-future-is-rusty-2023](https://weeklyrust.substack.com/p/rust-bytes-the-future-is-rusty-2023)

*Image: Learn rust crab mascot*

Hello Rustacean! Welcome to another edition of the Rust Bytes newsletter. In this issue, we'll shine a spotlight on an amazing Rust project, present our spot-the-bug challenge, and share some incredible links of the week.

Welcome to Issue 11!

**What do you call a Rust programmer who loves concurrency?**

A multi-threaded menace.

**2023 Annual Rust Survey Results** are out. Here are the key insights:

*   93% of the respondents self-identify as Rust users. Of those who used Rust in 2023, 49% did so on a daily (or nearly daily) basis.

*   31% of those who did not identify as Rust users cited the perception of difficulty as the primary reason for not having used it.

*   Of the former Rust users who participated in the 2023 survey, 31% stopped using Rust due to preferring another language and 24% cited difficulty as the primary reason for giving up.

*   Amongst the IDEs used for Rust development, Visual Studio Code still seems to be the most popular option, with RustRover (which was released last year) also gaining some traction.

*   79% of the people reported that Rust helped their company achieve its goals.

*   9,374 people shared their main worries for the future of Rust and 43% were concerned about Rust becoming too complex.

Check out 2023 [Annual Rust Survey Results](https://blog.rust-lang.org/2024/02/19/2023-Rust-Annual-Survey-2023-results.html) for more details.

Lifetime Trickery:

Identify why the code fails to compile and suggest a way to fix it.

Want to share your solution and be featured in our upcoming newsletter? Share your solution in the comments below, and to be included in the upcoming issue of the newsletter.

**clearcheck** is designed to make assertion statements as clear and concise as possible. It allows chaining multiple assertions together for a fluent and intuitive syntax, leading to more self-documenting test cases.

Features:

*   Fluent API: Chain assertions for a natural and readable experience.

*   Extensive assertions: Variety of assertions covering common validation needs.

*   Customizable: Extend with your own assertions for specific domain requirements.

*   Type-safe: Built with Rust's type system for reliable and expressive assertions.

*   Custom assertions: Craft assertions tailored to your exact needs, ensuring comprehensive validations for various data structures.

The project is maintained by Sarthak Makhija and is open-source on [GitHub](https://github.com/SarthakMakhija/clearcheck).

1.  **Michael Lohr** wrote about [“Embedded Rust in Production ..?, A review after using Rust on embedded in production for over a year”](https://blog.lohr.dev/embedded-rust)

2.  Building on previous efforts, the Rust Foundation released its [“Second Security Initiative Report Details Rust Security Advancements”](https://foundation.rust-lang.org/news/second-security-initiative-report-details-rust-security-advancements/) Report, highlighting progress in securing the language.

3.  Sebastian Martinez, in his article ["Why You Should Consider Rewriting Your Legacy System in Rust,"](https://www.wyeworks.com/blog/2024/02/20/why-you-should-consider-rewriting-your-legacy-system-in-rust/) explores the compelling reasons to breathe new life into aging software by harnessing the power of Rust.

4.  ["Prodzilla: From Zero to Prod with Rust and Shuttle"](https://codingupastorm.dev/about/) by Jordan Andrews details the development of Prodzilla, a synthetic monitoring tool built in Rust. It highlights Prodzilla's unique features like testing complex user flows with YAML e.t.c.

5.  **[](https://arighi.blogspot.com/2024/02/writing-scheduler-for-linux-in-rust.html)**["Writing a scheduler for Linux in Rust that runs in user-space"](https://arighi.blogspot.com/2024/02/writing-scheduler-for-linux-in-rust.html), the arighi's blog details the development of scx_rustland, a Linux scheduler written in Rust for user-space.

6.  This blog post, written by Jakub Beránek, Jack Huey, and Paul Lenz, announces the [Rust Project's participation in Google Summer of Code (GSoC) 2024](https://blog.rust-lang.org/2024/02/21/Rust-participates-in-GSoC-2024.html).

7.  Kasper Hermansen argues in his article [“Building services in rust”](https://blog.kasperhermansen.com/posts/building-business-services-in-rust/) that Rust is ready for building business services.

8.  Author Joshua Mo explores [8 productivity-boosting tools for Rust development, including cargo plugins, testing frameworks, debuggers, and performance profilers.](https://www.shuttle.rs/blog/2024/02/15/best-rust-tooling)

9.  RustConf 2024, an annual gathering for the Rust programming language community, [will be held in Montreal, Canada from September 10-13](https://foundation.rust-lang.org/news/save-the-date-rustconf-2024-september-10-13/).

10.  [How can Rust be so fast in the TechEmpower Web Framework Benchmarks?](https://kerkour.com/rust-fast-techempower-web-framework-benchmarks)

```
Explanation:

The function greater_of_two takes two string slices (&str) and returns the one with greater length. &str is an immutable reference to a sequence of UTF-8 text owned by someone else. This means &str (/string slice) has a definite lifetime: the stretch of the code for which a reference is valid. 

The return type of the function is also a reference, which means Rust needs a way to define an association between the lifetimes of the input parameters and the lifetime of the return.

Solution:
To fix the compilation error, we need to use the lifetime annotation in the function greater_of_two.

fn greater_of_two<'a>(one: &'a str, other: &'a str) -> &'a str {
    if one.len() > other.len() {
        return one;
    }
    return other;
}

The function signature means that for some lifetime 'a, the input parameters live atleast as long as 'a and the returned reference will also live at least as long as the lifetime 'a.
```

To support the newsletter:

That's all for now, Rustaceans! Until next week, have a productive week ahead.