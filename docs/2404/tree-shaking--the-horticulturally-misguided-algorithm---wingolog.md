<!--yml
category: 未分类
date: 2024-05-27 13:10:12
-->

# tree-shaking, the horticulturally misguided algorithm — wingolog

> 来源：[https://wingolog.org/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm](https://wingolog.org/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)

Let’s talk about tree-shaking!

### looking up from the trough

But first, I need to talk about WebAssembly’s dirty secret: despite the hype, WebAssembly has had limited success on the web.

There is [Photoshop](https://medium.com/@addyosmani/photoshop-is-now-on-the-web-38d70954365a), which does appear to be a real success. 5 years ago there was [Figma](https://www.figma.com/blog/figma-faster/), though they don’t talk much about Wasm these days. There are quite a number of little NPM libraries that use Wasm under the hood, usually compiled from C++ or Rust. I think [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) probably gets used for a few in-house corporate apps, though I could be fooled by their marketing.

You might recall the hyped demos of 3D first-person-shooter games with Unreal engine again from 5 years ago, but that was the previous major release of Unreal and was always experimental; the current Unreal 5 does not support targetting WebAssembly.

Don’t get me wrong, I think WebAssembly is great. It is having fine success in off-the-web environments, and I think it is going to be a key and growing part of the Web platform. I suspect, though, that we are only just now getting past the [trough of disillusionment](https://en.wikipedia.org/wiki/Gartner_hype_cycle).

It’s worth reflecting a bit on the nature of web Wasm’s successes and failures. Taking Photoshop as an example, I think we can say that Wasm does very well at bringing large C++ programs to the web. I know that it took quite some work, but I understand the end result to be essentially the same source code, just compiled for a different target.

Similarly for the JavaScript module case, Wasm finds success in getting legacy C++ code to the web, and as a way to write new web-targetting Rust code. These are often tasks that JavaScript doesn’t do very well at, or which need a shared implementation between client and server deployments.

On the other hand, WebAssembly has not been a Web success for DOM-heavy apps. Nobody is talking about rewriting the front-end of [wordpress.com](https://wordpress.com) in Wasm, for example. Why is that? It may sound like a silly question to you: Wasm just isn’t good at that stuff. But *why*? If you dig down a bit, I think it’s that the programming models are just too different: the Web’s primary programming model is JavaScript, a language with dynamic typing and managed memory, whereas WebAssembly 1.0 was about static typing and linear memory. Getting to the DOM from Wasm was a hassle that was overcome only by the most ardent of the true Wasm faithful.

Relatedly, Wasm has also not really been a success for languages that aren’t, like, C or Rust. I am guessing that wordpress.com isn’t written mostly in C++. One of the sticking points for this class of language. is that C#, for example, will want to ship with a garbage collector, and that it is annoying to have to do this. Check my [article from March this year for more details](https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us).

Happily, this restriction is going away, as all browsers are going to ship support for reference types and garbage collection within the next months; Chrome and Firefox already ship Wasm GC, and Safari shouldn’t be far behind thanks to the efforts from my colleague Asumu Takikawa. This is an extraordinarily exciting development that I think will kick off a whole ‘nother Gartner hype cycle, as more languages start to update their toolchains to support WebAssembly.

### if you don’t like my peaches

Which brings us to the meat of today’s note: web Wasm will win where compilers create compact code. If your language’s compiler toolchain can manage to produce useful Wasm in a file that is less than a handful of over-the-wire kilobytes, you can win. If your compiler can’t do that yet, you will have to instead rely on hype and captured audiences for adoption, which at best results in an unstable equilibrium until you figure out what’s next.

In the JavaScript world, managing bloat and deliverable size is a huge industry. Bundlers like [esbuild](https://esbuild.github.io/) are a ubiquitous part of the toolchain, compiling down a set of JS modules to a single file that should include only those functions and data types that are used in a program, and additionally applying domain-specific size-squishing strategies such as minification (making monikers more minuscule).

Let’s focus on tree-shaking. The visual metaphor is that you write a bunch of code, and you only need some of it for any given page. So you imagine a tree whose, um, branches are the modules that you use, and whose leaves are the individual definitions in the modules, and you then violently shake the tree, probably killing it and also annoying any nesting birds. The only thing that’s left still attached is what is actually needed.

This isn’t how trees work: holding the trunk doesn’t give you information as to which branches are somehow necessary for the tree’s mission. It also [primes your mind to look for the wrong fixed point](https://pvk.ca/Blog/2012/02/19/fixed-points-and-strike-mandates/), removing unneeded code instead of keeping only the necessary code.

But, tree-shaking is an evocative name, and so despite its horticultural and algorithmic inaccuracies, we will stick to it.

The thing is that maximal tree-shaking for languages with a thicker run-time has not been a huge priority. Consider Go: [according to the golang wiki](https://zchee.github.io/golang-wiki/WebAssembly/), the most trivial program compiled to WebAssembly from Go is 2 megabytes, and adding imports can make this go to 10 megabytes or more. Or look at Pyodide, the Python WebAssembly port: the [REPL example](https://pyodide.org/en/stable/console.html) downloads about 20 megabytes of data. These are fine sizes for technology demos or, in the limit, very rich applications, but they aren’t winners for web development.

### shake a different tree

To be fair, both the built-in Wasm support for Go and the Pyodide port of Python both derive from the upstream toolchains, where producing small binaries is nice but not necessary: on a server, who cares how big the app is? And indeed when targetting smaller devices, we tend to see alternate implementations of the toolchain, for example [MicroPython](https://micropython.org/) or [TinyGo](https://github.com/tinygo-org/tinygo). TinyGo has a Wasm back-end that can apparently go down to less than a kilobyte, even!

These alternate toolchains often come with [some restrictions or peculiarities](https://tinygo.org/docs/reference/lang-support/), and although we can consider this to be an evil of sorts, it is to be expected that the target platform exhibits some co-design feedback on the language. In particular, running in the sea of the DOM is sufficiently weird that a Wasm-targetting Python program will necessarily be different than a “native” Python program. Still, I think as toolchain authors we aim to provide the same language, albeit possibly with a different implementation of the standard library. I am sure that the ClojureScript developers would prefer to remove their [page documenting the differences with Clojure](https://clojurescript.org/about/differences) if they could, and perhaps if Wasm becomes a viable target for Clojurescript, they will.

### on the algorithm

To recap: now that it supports GC, Wasm could be a winner for web development in Python and other languages. You would need a different toolchain and an effective tree-shaking algorithm, so that user experience does not degrade. So let’s talk about tree shaking!

I work on the [Hoot Scheme compiler](https://gitlab.com/spritely/guile-hoot/), which targets Wasm with GC. We manage to get down to 70 kB or so right now, in the minimal “main” compilation unit, and are aiming for lower; auxiliary compilation units that import run-time facilities (the current exception handler and so on) from the main module can be sub-kilobyte. Getting here has been tricky though, and I think it would be even trickier for Python.

Some background: [like Whiffle](https://wingolog.org/archives/2023/11/16/a-whiff-of-whiffle), the Hoot compiler prepends a [prelude](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/prelude.scm) onto user code. Tree-shakind happens in a number of places:

*   [partial evaluation](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/tree-il/peval.scm) will evaluate unused bindings for effect, possibly eliding them

*   [fixing letrec](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/tree-il/fix-letrec.scm) will do the same

*   CPS frequently traverses the program, following only referenced function, value, and control edges, e.g. via [renumbering](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/renumber.scm)

*   There is an explicit [dead-code elimination](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/dce.scm) pass which tries to elide unused effect-free allocations, a situation that can arise due to other optimizations

*   Finally there is a [standard library written in raw-ish WebAssembly](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/stdlib.scm?ref_type=heads), whose definitions (globals, tables, imports, functions, etc) are included in the residual binary [only as neeeded](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/link.scm?ref_type=heads).

Generally speaking, procedure definitions (functions / closures) are the easy part: you just include only those functions that are referenced by the code. In a language like Scheme, this gets you a long way.

However there are three immediate challenges. One is that the evaluation model for the definitions in the prelude is `letrec*`: the scope is recursive but ordered. Binding values can call or refer to previously defined values, or capture values defined later. If evaluating the value of a binding requires referring to a value only defined later, then that’s an error. Again, for procedures this is trivially OK, but as soon as you have non-procedure definitions, sometimes the compiler won’t be able to prove this nice “only refers to earlier bindings” property. In that case the [fixing `letrec` (reloaded)](https://legacy.cs.indiana.edu/~dyb/pubs/letrec-reloaded-abstract.html) algorithm will end up residualizing bindings that are `set!`, which of all the tree-shaking passes above require the delicate DCE pass to remove them.

Worse, some of those non-procedure definitions are *record types*, which have vtables that define how to print a record, how to check if a value is an instance of this record, and so on. These vtable callbacks can end up keeping a lot more code alive even if they are never used. We’ll get back to this later.

Similarly, say you print a string via [`display`](https://www.r6rs.org/final/html/r6rs-lib/r6rs-lib-Z-H-9.html#node_idx_832). Well now not only are you bringing in the whole buffered I/O facility, but you are also calling a highly polymorphic function: `display` can print anything. There’s a case for bitvectors, so you pull in code for bitvectors. There’s a case for pairs, so you pull in that code too. And so on.

One solution is to instead call `write-string`, which only writes strings and not general data. You’ll still get the generic buffered I/O facility (ports), though, even if your program only uses one kind of port.

This brings me to my next point, which is that optimal tree-shaking is a flow analysis problem. Consider `display`: if we know that a program will never have bitvectors, then any code in `display` that works on bitvectors is dead and we can fold the branches that guard it. But to know this, we have to know what kind of arguments `display` is called with, and for that we need [higher-level flow analysis](https://wingolog.org/archives/2014/07/01/flow-analysis-in-guile).

The problem is exacerbated for Python in a few ways. One, because [object-oriented dispatch is higher-order programming](https://matt.might.net/articles/implementation-of-kcfa-and-0cfa/). How do you know what `foo.bar` actually means? Depends on `foo`, which means you have to thread around representations of what `foo` might be everywhere and to everywhere’s caller and everywhere’s caller’s caller and so on.

Secondly, lookup in Python is generally more dynamic than in Scheme: you have `__getattr__` methods (is that it?; been a while since I’ve done Python) everywhere and users might indeed use them. Maybe this is not so bad in practice and flow analysis can exclude this kind of dynamic lookup.

Finally, and perhaps relatedly, the object of tree-shaking in Python is a mess of modules, rather than a big term with lexical bindings. This is like JavaScript, but without the established ecosystem of tree-shaking bundlers; Python has its work cut out for some years to go.

### in short

With GC, Wasm makes it thinkable to do DOM programming in languages other than JavaScript. It will only be feasible for mass use, though, if the resulting Wasm modules are small, and that means significant investment on each language’s toolchain. Often this will take the form of alternate toolchains that incorporate experimental tree-shaking algorithms, and whose alternate standard libraries facilitate the tree-shaker.

Welp, I’m off to lunch. Happy wassembling, comrades!