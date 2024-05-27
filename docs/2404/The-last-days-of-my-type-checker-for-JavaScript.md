<!--yml
category: 未分类
date: 2024-05-27 13:28:39
-->

# The last days of my type checker for JavaScript

> 来源：[https://jsmonk.github.io/2024-01-29-last-days-of-hegel/](https://jsmonk.github.io/2024-01-29-last-days-of-hegel/)

**TL;DR** Last year, I started to work on the 2.0 version of [my static type checker for JavaScript](https://hegel.js.org/) written in [Rust](https://www.rust-lang.org/), but, unfortunately, I see that nowadays, [TypeScript](https://www.typescriptlang.org/) won. The idea of strict type-checking for JavaScript is less popular than in 2019\. Additionally, another way to bring stricter types into the web exists. So, I lost my belief that there could be an audience for such a tool and decided to close the project.

## To make the story short

At the beginning of 2019, I worked in a small outsourcing company called [WookieeLabs](https://github.com/wookieelabs). In most of the projects, we preferred to use [Flow](https://flow.org/) over the [TypeScript](https://www.typescriptlang.org/). There were a lot of benefits such as [nominal classes](https://flow.org/en/docs/lang/nominal-structural/#toc-classes-are-nominally-typed), [type variances](https://flow.org/en/docs/lang/variance), [opaque types](https://flow.org/en/docs/types/opaque-types/), [smooth integration with React](https://flow.org/en/docs/react/). But the main reason was that Flow in 2019 was just JavaScript with types.

But the way Flow type inference was working in 2019 was not ideal:

```
const id = x => x;
const a = id(2); // number | string
const b = id("string"); // number | string 
```

You can play with this example [here](https://flow.org/try/#1N4Igxg9gdgZglgcxALmAXwDTggEwKYrZQDOALgARw7kC85AHrQHwMDcAOlJCRQIa2UcACgBMASg5doZcgCMBVIexBkATnCgJlEkFgBueVcTjRCegAwA6AIwAOAKw2QaIA)

So, Flow analyzed the usage of the function and, based on it, inferred the type of a function. In the example above, we provided to the `id` function a number and a string, so, based on the usage, the tool infered the type of the `id` as `(x: number | string) => number | string`.

However, the Flow team did a significant job of fixing such behavior by requiring annotations in places where the tool knows it will not have a local typing context. You can read about it [here](https://medium.com/flow-type/local-type-inference-for-flow-aaa65d071347).

At the same time, there was hype about the new “language” (syntax for [OCaml](https://ocaml.org/)) called [ReasonML](https://reasonml.github.io/) (today there is also an alternative project called [ReScript](https://rescript-lang.org/)).

And I was wondering, what if we take the power of the [Hindley-Milner type inference](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system), as strict as possible type system for JavaScript and the simple Flow like types syntax and combine it inside a single tool. So, I created [Hegel](https://hegel.js.org/) as my university bachelor project and started to talk about it at some conferences.

You can check the main differences in comparison with TypeScript [here](https://hegel.js.org/docs#benefits-and-disadvantages-over-typescript) and with Flow [here](https://hegel.js.org/docs#benefits-and-disadvantages-over-flowjs)

But my favorite example that describes what the tool provides is [this](https://hegel.js.org/try#GYVwdgxgLglg9mABAEwKYGdUCcYEMA2MAXqgKqZYAU6UOYA5jMDKsudgFLoICUiA3gChEiCAhqIAtrgCeAIzIVEAXkQcAygHkAcgDoADriyZqtGAyYs2FLrwDcwxE0SVHIqDP2o4wKbIXsWCrKqgBEcHIAVqjQoYgAZPFufvKK2IgAhCGIYCD4+AlJIiKhYLiSqHHmKQFKickeXj41aVi6ZRXBYTR09KGOfELFiFioUCBYSNKpgQ4iAL6OUAAWWHAA7jmomwAqnqgAolhrVKEACmsAbjBoyIgUeIQkdyBKMOhOYJcENxmhPA5FoJaDIBI4xGAJMA4HAVCgMNhHsRWpRQgAWABM-0BolwUAgyxcqEGgkEiyAA):

```
// The type of the function is: (string) => { name: string, ... } throws SyntaxError | TypeError
function deserializeUser(stringifiedUserJson) {
  /* the result type is unknown because there is no `any` type */
  const maybeUser  = JSON.parse(stringifiedUserJson); /* it throws a SyntaxError, so it was added to the type signature */
  if (
    typeof maybeUser === "object" && maybeUser !== null       // a proof that `maybeUser` is an object
    "name" in maybeUser && typeof maybeUser.name === "string" // a proof that there is a property called `name` with `string` type
  ) {
    return maybeUser; // the result type is { name: string, ... }
  }
  throw new TypeError("Provided serialized user is invalid!");
}

try {
  /* the type of `foo` is { name: string, ... } */
  const foo = deserializeUser("42");
} catch (e) {
  // the type of `e` is SyntaxError | TypeError | unknown
} 
```

## And there were difficulties

Because most of the time I was the only contributor in the core modules, working on a full-time job and continuing study as a master didn’t give me enough time to contribute more, so there were (and still) a lot of issues and bugs, that should be fixed, so the development took a long time.

Also, speaking at local conferences didn’t help a lot in finding a lot of new contributors.

> I want to thank everyone who supported me with the project, proposed exciting ideas, and helped me prepare different examples. Some people also helped me fix grammar mistakes in the documentation, review all the sections, and set up the CI/CD. You all are unbelievable.

At some point (around March 2021), I struggled with all the COVID-19 restrictions and the rumors about the upcoming war between Ukraine and Russia. So, my wife and I relocated to the Netherlands two months before the war started.

> I also want to say a big thank you to [JetBrains](https://www.jetbrains.com/). You not only gave me the possibility to work on such exciting projects but literally saved my and my wife’s lives.

After a while, I started to rework the core system differently to support an incremental compilation (to improve analysis speed) and an intermediate representation (to support multiple syntaxes, including TypeScript syntax; the tool had previously worked directly with the AST).

But during that time, I felt like I was trying to overtake a Ferrari, and the community interest in such a project was already insignificant.

## Reflection on the project

> For today, I have only results for the [State of JavaScript 2022](https://2022.stateofjs.com/en-US/), and as soon as the results of [State of JavaScript 2023](https://stateofjs.com/en-US) become available, I will update the numbers.

The following things made me lose the belief that I should continue my work:

*   The developers of TypeScript are doing an excellent language and working a lot on its improvement, so it is the most popular alternative for JavaScript nowadays (98.9% [question](https://2022.stateofjs.com/en-US/other-tools#javascript_flavors) respondents use TypeScript, the nearest competitor is Elm with the result 2.3%)
*   JavaScript as a technology and community is already mature, so the new tools, frameworks, and libraries take a small part of the market. [Here](https://2022.stateofjs.com/en-US/other-tools#runtimes) you can take a look on the usage of [Deno](https://deno.com/) (one of the Deno prototypes [was presented](https://www.youtube.com/watch?v=M3BM9TB-8yA) in 2019) comparing to the [Node.js](https://nodejs.org/en), or [the comparison of frameworks usage](https://2022.stateofjs.com/en-US/libraries/front-end-frameworks#front_end_frameworks_experience_linechart) where we can see that the new frameworks such as [Solid.js](https://www.solidjs.com/) or [qwik](https://qwik.builder.io/) take 6% and 2%
*   Based on my communication with different developers, I realized that TypeScript changed how developers think about the types in JavaScript. In 2019, people thought of types as the rules restricting the language for developing safer software (fewer bugs or unexpected runtime errors). Still, today, people think more about describing **any** (even not so secure) software with types. All of these fancy [template literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html), [conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html), an unchecked body of [assert functions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions) and [type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates), and of course [any](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any), give people a fantastic tool to cover existed code with types but doesn’t force to re-design it to make it safer.
*   I see other opportunities to increase the language diversity on the web (further in the article)

So, primarily based on those things, I decided to close the project [Hegel](https://hegel.js.org/).

## Other projects and other opportunities

Last year, I unexpectedly realized what a big and fantastic job was done on [wasm](https://webassembly.org/) as a technology. Proposals such as [GC proposal](https://github.com/WebAssembly/gc/blob/main/proposals/gc/Overview.md) and [Exception handling](https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/Exceptions.md) help to bring to the web more and more programming languages with stricter type system that are already familiar to people, that have already a mature ecosystem of build tools, libraries, frameworks, but also [Component Model](https://github.com/WebAssembly/component-model) proposes the solution to interop between of them, that means sharing parts of the extensive ecosystems between each other.

So, this is one of the reasons why I decided to join [Kotlin/Wasm](https://kotlinlang.org/docs/wasm-overview.html) team and focus most of the time on helping to bring the amazing [Kotlin](https://kotlinlang.org/) and its ecosystem into the web.

Suppose you are still looking for the analyzing tool for your JavaScript project. In that case, you can bring your attention to a cool project called [Ezno](https://github.com/kaleidawave/ezno), which provides strict type checking and also (because it’s a full compiler) focuses on optimizing your code. [Ben](https://twitter.com/kaleidawave) is doing an incredible job.