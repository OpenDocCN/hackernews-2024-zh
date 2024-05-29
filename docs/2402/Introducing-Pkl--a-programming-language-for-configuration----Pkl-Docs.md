<!--yml
category: 未分类
date: 2024-05-27 14:33:56
-->

# Introducing Pkl, a programming language for configuration :: Pkl Docs

> 来源：[https://pkl-lang.org/blog/introducing-pkl.html](https://pkl-lang.org/blog/introducing-pkl.html)

by the Pkl Team on February 1st, 2024

We are delighted to announce the open source first release of Pkl (pronounced *Pickle*), a programming language for producing configuration.

When thinking about configuration, it is common to think of static languages like JSON, YAML, or Property Lists. While these languages have their own merits, they tend to fall short when configuration grows in complexity. For example, their lack of expressivity means that code often gets repeated. Additionally, it can be easy to make configuration errors, because these formats do not provide any validation of their own.

To address these shortcomings, sometimes formats get enhanced by ancillary tools that add special logic. For example, perhaps there’s a need to make code more [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), so a special property is introduced that understands how to resolve references, and merge objects together. Alternatively, there’s a need to guard against validation errors, so some new way is created to validate a configuration value against an expected type. Before long, these formats almost become programming languages, but ones that are hard to understand and hard to write.

On the other end of the spectrum, a general-purpose language might be used instead. Languages like Kotlin, Ruby, or JavaScript become the basis for [DSL](https://en.wikipedia.org/wiki/Domain-specific_language)s that generate configuration data. While these languages are tremendously powerful, they can be awkward to use for describing configuration, because they are not oriented around defining and validating data. Additionally, these DSLs tend to be tied to their own ecosystems. It is a hard sell to use a Kotlin DSL as the configuration layer for an application written in Go.

We created Pkl because we think that configuration is best expressed as a blend between a static language and a general-purpose programming language. We want to take the best of both worlds; to provide a language that is declarative and simple to read and write, but enhanced with capabilities borrowed from general-purpose languages. When writing Pkl, you are able to use the language features you’d expect, like classes, functions, conditionals, and loops. You can build abstraction layers, and share code by creating packages and publishing them. Most importantly, you can use Pkl to meet many different types of configuration needs. It can be used to produce static configuration files in any format, or be embedded as a library into another application runtime.

We designed Pkl with three overarching goals:

*   To provide safety by catching validation errors before deployment.

*   To scale from simple to complex use-cases.

*   To be a joy to write, with our best-in-class IDE integrations.