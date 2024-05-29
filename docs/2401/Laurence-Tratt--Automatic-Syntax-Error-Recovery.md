<!--yml
category: 未分类
date: 2024-05-27 14:39:17
-->

# Laurence Tratt: Automatic Syntax Error Recovery

> 来源：[https://tratt.net/laurie/blog/2020/automatic_syntax_error_recovery.html](https://tratt.net/laurie/blog/2020/automatic_syntax_error_recovery.html)

Programming is the best antidote to arrogance I’ve come across — I make so many errors that I am continually reminded of my own fallibility. Broadly speaking, I think of errors as severe or minor. Severe errors are where I have fundamentally misunderstood something about the system I am creating. Each severe error is a bespoke problem, often requiring custom tooling to understand and fix it. Minor errors, in contrast, are repetitive and quickly fixed. However, they’re also *much* more numerous than severe errors: even shaving a couple of seconds off of the time it takes a programmer to fix a class of minor errors is worthwhile when you consider how often they occur.

The most minor of minor errors, and also I suspect the most frequent, are syntax errors. They occur for three main reasons: mental sloppiness; inaccurate typing ; or an incomplete understanding of the language’s syntax. The latter case is generally part of a brief-ish learning phase we go through and I’m not sure what a good solution for it might look like. The former two cases, however, are extremely common. When I’ve made a small typo, what I want is the parser in my compiler or IDE to pinpoint the location of the syntax error accurately and then *recover* from it and continue as if I hadn’t made an error at all. Since compilation is often far from instantaneous, and I often make multiple errors (not just syntax errors), good quality syntax error recovery improves my programming efficiency.

Unfortunately, LR parsers – [of which I am particularly fond](/laurie/blog/entries/which_parsing_approach.html) – have a poor reputation for syntax error recovery. I’m going to show in this article that this isn’t inevitable, and that it’s possible to do surprisingly good automatic syntax error recovery for any LR grammar. If you want to know more details, you might be interested in the paper [Lukas Diekmann](https://diekmann.co.uk/) and I recently published called [Don’t Panic! Better, Fewer, Syntax Errors for LR Parsers](https://soft-dev.org/pubs/html/diekmann_tratt__dont_panic/). The paper also has a fairly brief accompanying talk, if you find that sort of thing helpful:

[https://www.youtube.com/embed/PGRnk-bzTdU](https://www.youtube.com/embed/PGRnk-bzTdU)

VIDEO

For everyone else, let’s continue. To make our lives easier, for the rest of this article I’m going to shorten “syntax error recovery” to “error recovery”.

## Outlining the problem

Let’s see error recovery in action in a widely used compiler — `javac`:

 [Click to start/stop animation]

As a quick reminder, ‘`int x y;`’ isn’t valid Java syntax. `javac` correctly detects a syntax error after the ‘`x`’ token and then tries to recover from that error. Since ‘`int x;`’ *is* valid Java, `javac` assumes that I meant to put a semi-colon after ‘`x`’, *repairs* my input accordingly, and continues parsing. This is the good side of error recovery: my silly syntax error hasn’t stopped the compiler from carrying on its good work. However, the bad side of error recovery is immediately apparent: ’ `y;`’ isn’t valid Java, so `javac` immediately prints out a second, spurious, syntax error that isn’t any fault of mine.

Of course, I have deliberately picked an example where `javac` does a poor job but I regret to inform you that it didn’t take me very long to find it. Many parsers do such a poor job of error recovery that experienced programmers often scroll back to the location of the first syntax error, ignoring both its repair and any subsequent syntax errors. Instead of being helpful, error recovery can easily have the opposite effect, slowing us down as we look for the real error amongst a slew of spurious errors.

Let’s look at the modern compiler that I most often use as an exemplar of good error messages, `rustc`. It often does a good job in the face of syntax errors:

 [Click to start/stop animation]

However, even `rustc` can be tripped up when presented with simple syntax errors:

 [Click to start/stop animation]

Some language implementations don’t even bother trying to recover from syntax errors. For example, even if I make two easily fixable syntax errors in a file, CPython stops as soon as it encounters the first:

 [Click to start/stop animation]

As all this might suggest, error recovery is hard to do well, and it’s unlikely that it will ever fully match human intuition about how syntax errors should be fixed. The root of the problem is that when we hit an error while parsing, there are, in general, an infinite number of ways that we could take to try and get parsing back on track. Since exploring infinity takes a while, error recovery has to use heuristics of one sort or another.

The more knowledge a parser has of the language it is parsing, the more refined that heuristic can be. Hand-written parsers have a fundamental advantage here, because one can add as much knowledge about the language’s semantics as one wants to the parser. However, extending a hand-written parser in this way is no small task, especially for languages with large grammars. It’s difficult to get precise figures, but I’ve seen more than one parser that has taken a small number of person years of effort, much of which is devoted to error recovery. Not many of us have that much time to devote to the problem.

Automatically generated parsers, in contrast, are at a clear disadvantage: their only knowledge of the language is that expressed via its grammar. Despite that, automatically generated LL parsers are often able to do a tolerable job of error recovery .

Unfortunately, LR parsers have a not undeserved reputation for doing a poor job of error recovery. Yacc, for example, requires users to sprinkle `error` tokens throughout their grammar in order to have error recovery in the resulting parser: I think I’ve only seen one real grammar which makes use of this feature, and I am sceptical that it can be made to work well. *Panic mode* is a fully automatic approach to error recovery in LR parsing, but it works by gradually deleting the parsing stack, causing it to delete input before a syntax error in order to try and recover after it. Frankly, panic mode’s repairs are so bad that I think on a modern machine it’s worse than having no error recovery at all.

## The roots of a solution

At a certain point when working on [grmtools](https://github.com/softdevteam/grmtools/) I realised that I should think about error recovery, something which I had only ever encountered as a normal user. A quick word about grmtools: it’s intended as a collection of parsing related libraries in Rust. At the moment, the parts that are most useful to users are [lrpar](https://crates.io/crates/lrpar) – a [Yacc](http://dinosaur.compilertools.net/yacc/index.html)-compatible parser – and, to a lesser extent , [lrlex](https://crates.io/crates/lrlex) – a [Lex](http://dinosaur.compilertools.net/lex/index.html)-ish lexer. For the rest of this article, I’ll almost exclusively be talking about lrpar, as that’s the part concerned with error recovery.

Fortunately for me, I quickly came across [Carl Cerecke’s PhD thesis](https://ir.canterbury.ac.nz/bitstream/handle/10092/5492/cerecke_thesis.pdf) which opened my eyes to an entirely different way of doing error recovery. His thesis rewards careful reading, and has some very good ideas in it. Ultimately I realised that Cerecke’s thesis is a member of what these days I call the [Fischer *et al.*](https://minds.wisconsin.edu/bitstream/handle/1793/58168/TR363.pdf?sequence=1&isAllowed=y) family of error recovery algorithms for LR parsing, since they all trace their lineage back to that paper.

When error recovery algorithms in the Fischer *et* *al.* family encounter a syntax error they try to find a *repair sequence* that, when applied to the input, gets parsing back on track. Different algorithms have different repairs at their disposal and different mechanisms for creating a repair sequence. For example, we ended up using the approach of [Corchuelo *et al.*](https://idus.us.es/bitstream/11441/65631/1/Repairing%20syntax%20errors.pdf) — one of the most recent members of the Fischer *et al.* family — as our intellectual base.

## CPCT+ in use

We took the Corchuelo *et al.* algorithm, fixed and semi-formalised it , and extended it to produce a new error recovery algorithm CPCT+ that is now part of lrpar. We can use [`nimbleparse`](https://softdevteam.github.io/grmtools/master/book/nimbleparse.html) — a simple command-line grammar debugging tool — to see CPCT+ in action on our original Java example:

 [Click to start/stop animation]

As with `javac`’s error recovery, CPCT+ is started when lrpar encounters a syntax error at the ‘`y`’ token. Unlike `javac`, CPCT+ presents 3 different *repair sequences* (numbered 1, 2, 3) to the user which, in order , would repair the input to be equivalent to: ‘`int x, y;`’, ‘`int x = y;`’, or ‘`int x;`’. Importantly, repair sequences can contain multiple repairs:

 [Click to start/stop animation]

Since you probably don’t want to watch the animation endlessly I’ll put the repair sequences that are reported here:

```
Parsing error at line 2 column 13\. Repair sequences found:
 1: Insert ), Shift {, Delete if, Delete true 2: Insert ), Shift {, Shift if, Insert (, Shift true, Insert ) 
```

This example shows all of the individual repair types that CPCT+ can generate:

*   ‘`Insert x`’ means ‘insert a token `x` at the current position’;
*   ‘`Shift x`’ means ‘keep the token `x` at the current position unchanged and advance the search’;
*   ‘`Delete x`’ means ‘delete the token `x` at the current position’.

A repair sequence is just that: an ordered sequence of repairs. For example, the first repair sequence above means that the input will be repaired to be equivalent to:

```
class C {
  void f() {
 { } } } 
```

while the second repair sequence will repair the input to be equivalent to:

```
class C {
  void f() {
  if (true) { }
 } } 
```

As this shows, CPCT+ is doing something very different to traditional error recovery: it’s repairing input spanning multiple tokens in one go. This is perfectly complementary to repairing syntax errors at different points in a file as this example shows:

 [Click to start/stop animation]

Although CPCT+ can return multiple repair sequences, it would be impractical to keep all those possibilities running in parallel — I also doubt that users would be able to interpret the resulting errors! lrpar thus takes the first repair sequence returned by CPCT+, applies it to the input, and continues parsing.

At this point you might be rather sick of Java examples. Fortunately, there’s nothing Java specific about CPCT+. If I feed it Lua’s Lex and Yacc files and broken input it’ll happily repair that too :

 [Click to start/stop animation]

Indeed, CPCT+ will happily perform error recovery on any other language for which you can write a Yacc grammar.

Ultimately, CPCT+ has one main novelty relative to previous members of the Fischer *et al.* family: it presents the *complete set of minimum* *cost repair sequences* to the user where other approaches non-deterministically present one member of that set to the users. In other words, when, for our original Java example, CPCT+ presented this to users:

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Insert , 2: Insert = 3: Delete y 
```

approaches such as Corchuelo *et al.* would only have presented one repair sequence to the user. Since those approaches are non-deterministic, each time they’re run they can present a different repair sequence to the one before, which is rather confusing. The intuition behind “minimum cost repair sequence” is that we want to prioritise repair sequences which do the smallest number of alterations to the user’s input: insert and delete repairs increase a repair sequence’s cost, although shift repairs are cost-neutral.

To my mind, in the example above, both ‘`Insert ,`’ and ‘`Insert =`’ are equally likely to represent what the programmer intended, and it’s helpful to be shown both. ‘`Delete y`’ is a bit less likely to represent what the programmer intended, but it’s not a ridiculous suggestion, and in other similar contexts would be the more likely of the 3 repair sequences presented.

## Under the bonnet

[The paper](https://soft-dev.org/pubs/html/diekmann_tratt__dont_panic/) and/or [the code](https://github.com/softdevteam/grmtools/) are the places to go if you want to know exactly how CPCT+ works, but I’ll try and give you a very rough idea of how it works here.

When lrpar encounters a syntax error, CPCT+ is started with the grammar’s statetable (the statemachine that an LR parser turns a grammar into; see e.g. [this example](https://en.wikipedia.org/wiki/LR_parser#Action_and_goto_tables)), the current parsing stack (telling us where we are in the statetable, and how we got there), and the current position in the user’s input. By definition the top of the parsing stack will point to an error state in the statetable. CPCT+’s main job is to return a parsing stack and position to lrpar that allows lrpar to continue parsing; producing repair sequences is a happy by-product of that.

CPCT+ is thus a path-finding algorithm in disguise, and we model it as an instance of [Dijkstra’s algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). In essence, each edge in the graph is a repair, which has a cost; we’re looking to find a path that leads us to success. In this case, “success” can occur in two ways: in rare cases where errors happen near the end of a file we might hit the statetable’s sole *accept* state; more commonly, we settle for shifting 3 tokens in a row (i.e. we’ve got to a point where we can parse some of the user’s input without causing further errors). As this might suggest, the core search is fairly simple.

Most of CPCT+’s complexity comes from the fact that we try to find all minimum cost paths to success and we need ways of optimising the search. There are a few techniques we describe in the paper to improve performance, so I’ll use what’s probably the most effective as an example. Our basic observation is that, when searching, once-distinct paths often end up reaching the same node, at which point we can model them as one henceforth. We therefore identify *compatible* nodes and *merge* them into one. The challenge is then how compatible nodes can be efficiently identified. We make use of an often forgotten facet of hashmaps: a node’s hash behaviour need only be a subset of its equality behaviour. In our case, nodes have three properties (parsing stack, remaining input, repair sequence): we hash based on two of these properties (parsing stack, remaining input) which quickly tells us if a node is potentially compatible; equality then checks (somewhat slowly) all three properties to confirm definite compatibility. This is a powerful optimisation, particularly on the hardest cases, improving average performance by 2x.

## Ranking repairs

CPCT+ collects the complete set of minimum cost repair sequences because I thought that would best match what a user would hope to see from an error recovery algorithm. The costs of creating the complete set of minimum cost repair sequences were clear early on but, to my surprise, there are additional benefits.

The overarching problem faced by all approaches in the Fischer *et al.* family is that the search space is unbounded in size. This is why shifting a mere 3 tokens from the user’s input is enough for us to declare a repair sequence successful: ideally we would like to check all of the remaining input, but that would lead to a combinatorial explosion on all but a handful of inputs. Put another way, CPCT’s core search is inherently local in nature: the repair sequences it creates can still cause subsequent spurious errors beyond the small part of the input that CPCT+ has searched.

The complete set of minimum cost repair sequences allow us to trivially turn the very-local search into a regional search, allowing us to rank repair sequences in a wider context. We take advantage of the fact that CPCT+’s core search typically only finds a small handful of repair sequences. We then temporarily apply each repair sequence to the input and see how far it can parse ahead without causing an error (up to a bound of 250 tokens). We then select the (non-strict) subset which has parsed furthest ahead and discard the rest. Consider this Java example:

```
class C { int x z() { } }  
```

When run through the “full” CPCT+ algorithm, two repair sequences are reported to the user:

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Delete z 2: Insert ; 
```

However if I turn off ranking, we can see that the “core” of CPCT+ in fact generated three repair sequences:

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Insert = 2: Insert ; 3: Delete z Parsing error at line 2 column 15\. Repair sequences found:
 1: Insert ; 
```

In this particular run, lrpar chose to apply the ‘`Insert =`’ repair sequence to the input. That caused a spurious second error just beyond the region that CPCT+ had searched in. However, the other two repair sequences allow the whole file to be parsed successfully (though there is a semantic problem with the resulting parse, but that’s another matter). It might not be immediately obvious, but traditional Fischer *et al.* algorithms wouldn’t be able to throw away the ‘`Insert =`’ repair sequence and keep looking for something better, because they have no point of comparison that would allow them to realise that it’s possible to do better. In other words, the unexpected advantage of the complete set of minimum cost repair sequences is precisely that it allows us to rank repair sequences relative to one another and discard the less good.

I’ve also realised over time that the ranking process (which requires about 20 lines of code) really kills two birds with one stone. First, and most obviously, it reduces spurious syntax errors. Second — and it took me a while to appreciate this — it reduces the quantity of low-quality repair sequences we present to users, making it more likely that users will actually check the repair sequences that are presented.

## Lex errors

Like most parsing systems, grmtools separates out lexing (splitting input up into tokens) from parsing (determining if/how a sequence of tokens conforms to a grammar). This article isn’t the place to get into the pros and cons of this, but one factor is relevant: as soon as the lexer encounters an error in input, it will terminate. That means that parsing won’t start and, since error recovery as we’ve defined it thus far is part of the parser, the user won’t see error recovery. That leads to frustrating situations such as this:

 [Click to start/stop animation]

Although it didn’t occur to me for a very long time, it’s trivial to convert lexing errors into parsing errors, and then have error recovery happen as normal. Even more surprisingly, this doesn’t require any support from lrpar or CPCT+. The user merely needs to catch input that otherwise wouldn’t lex by adding a rule such as the following at the end of their Lex file:

```
. "ERROR" 
```

This matches a single character (the ‘`.`’) as a new token type called ‘`ERROR`’. However, grmtools moans if tokens are defined in the Lexer but not used in the Yacc grammar so let’s shut it up by adding a dummy rule to the grammar:

```
ErrorRule: "ERROR" ; 
```

Now when we run nimbleparse, CPCT+ kicks in on our “lexing error”:

```
Parsing error at line 2 column 11\. Repair sequences found:
 1: Delete #, Delete y 2: Insert ,, Delete # 3: Insert =, Delete # 
```

I find this satisfying for two reasons: first, because users get a useful feature for precisely no additional effort on lrpar’s part; and second because lexing and parsing errors are now presented uniformly to the user, where before they were confusingly separate. It would probably be a good idea to make this a core feature, so that we could do things like merge consecutive error tokens, but that wouldn’t change the underlying technique.

## Integrating error recovery into actions

As far as I have been able to tell, no “advanced” error recovery algorithm has ever made its way into a long-lasting parsing system: I couldn’t find a single implementation which I could run. Indeed, a surprising number of error recovery papers don’t even mention a corresponding implementation, though there must surely have been at least a research prototype at some point.

Whatever software did, or didn’t, exist, none of the papers I’ve read make any mention of how error recovery affects the use of parsers. Think about your favourite compiler: when it encounters a syntax error and recovers from it, it doesn’t just continue parsing, but also runs things like the type checker (though it probably refuses to generate code). Of course, the reason your favourite compiler is doing this is probably because it has a hand-written parser. How should we go about dealing with this in a Yacc-ish setting?

grmtools’ solution is surprisingly simple: action code (i.e. the code that is executed when a production is successfully parsed) isn’t given access to tokens directly, but instead to an `enum` which allows one to distinguish tokens inserted by error recovery from tokens in the user’s input. The reason for this is probably best easy seen via an example, in this case [a very simple calculator grammar](https://github.com/softdevteam/grmtools/blob/master/lrpar/examples/calc_actions/src/calc.y) which calculates numbers as it parses:

 [Click to start/stop animation]

In this case, what I chose to do when writing the calculator evaluator is to continue evaluating expressions with syntax errors in, unless an integer was inserted. The reason for that is that I simply don’t have a clue what value I should insert if CPCT+ generated an ` `Insert INT`’ repair: is 0 reasonable? what about -1? or 1? As this suggests, inserting tokens can be quite problematic: while one might quibble about whether evaluation should continue when CPCT+ deleted the second ‘`+`’ in ‘`2 + + 3`’, at least that case doesn’t require the evaluator to pluck an integer value out of thin air.

This is an example of what I’ve ended up thinking of as the *semantic* *consequences* of error recovery: changing the syntax of the user’s input often changes its semantics, and there is no way for grmtools to know which changes have acceptable semantic consequences and which don’t. For example, inserting a missing ‘`:`’ in Python probably has no semantic consequences, but inserting the integer 999 into a calculator expression will have a significant semantic consequence.

The good news is that lrpar gives users the flexibility to deal with the semantic consequences of token insertion. For example here’s the grmtools-compatible grammar for the calculator language:

```
Expr -> Result<u64, Box<dyn Error>>:
 Expr '+' Term {
 Ok($1?.checked_add($3?)
 .ok_or_else(||
 Box::<dyn Error>::from("Overflow detected."))?)
 } | Term { $1 } ;   Term -> Result<u64, Box<dyn Error>>:
 Term '*' Factor {
 Ok($1?.checked_mul($3?)
 .ok_or_else(||
 Box::<dyn Error>::from("Overflow detected."))?)
 } | Factor { $1 } ;   Factor -> Result<u64, Box<dyn Error>>:
 '(' Expr ')' { $2 }
 | 'INT' {
  parse_int($lexer.span_str($1.map_err(|_|
 "<evaluation aborted>")?.span()))
 } ; 
```

If you’re not used to Rust, that might look a little scary, so let’s start with some of the non-error recovery parts. First, the calculator grammar evaluates mathematical expressions as parsing occurs, and it deals exclusively in unsigned 64-bit integers. Second, unlike traditional Yacc, lrpar requires each rule to specify its return type. In this case, each rule has a return type `Result<u64, Box<dyn Error>>` which says “if successful this returns a `u64`; if unsuccessful it returns a pointer to a description of the error on the heap”. Put another way ‘`dyn Error`’ is Rust’s rough equivalent to “this thing will throw an exception”.

As with traditional Yacc, the ‘`$n`’ expressions in action code reference a symbol’s production, where *n* starts from 1. Symbols reference either rules or tokens. As with most parsing systems, symbols that reference rules have the static type of that rule (in this example `Result<u64, Box<dyn Error>>`).

Where grmtools diverges from existing systems is that tokens always have the static type `Result<Lexeme, Lexeme>`. If you’re used to Rust that might look a bit surprising, as `Result` types nearly always contain two distinct subtypes, but in this case we’re saying that in the “good” case you get a `Lexeme` and in the “bad” case you also get a `Lexeme`. The reason for this is that the “good” case (if you’re familiar with Rust terminology, the ‘`Ok`’ case) represents a token from the user’s actual input and the “bad” case (‘`Err`’) represents an inserted token. Because `Result` is a common Rust type, one can then use all of the standard idioms that Rust programmers are familiar with.

Let’s first look at a simplified version of the first rule in the calculator grammar:

```
Expr -> Result<u64, Box<dyn Error>>:
 Expr '+' Term { $1? + $3? }
 | Term { $1 } 
```

The `Expr` rule has two productions. The second of those (‘`Term`’) simply passes through the result of evaluating another rule unchanged. The first production tries adding the two integers produced by other rules together, but if either of those rules produced a `dyn Error` then the ‘`?`’ operator percolates that error upwards (roughly speaking: throws the exception up the call stack).

Now let’s look at a simplified (to the point of being slightly incorrect) version of the third rule in the grammar:

```
Factor -> Result<u64, Box<dyn Error>>:
 '(' Expr ')' { $2 }
 | 'INT' {
  parse_int($1.map_err(|_| "<evaluation aborted>")?)
 } ; 
```

The second production references the `INT` token type. The action code then contains `$1.map_err(|_| “<evaluation aborted>”)?` which, in English, says “if the token `$1` was inserted by error recovery, throw a `dyn Error`”. In other words, the calculator grammar stops evaluating expressions when it encounters an inserted integer. However, if the token was from the user’s input, it is converted to a `u64` (with `parse_int`) and evaluation continues.

If you look back at the original grammar, you can see that this grammar has made the decision that *only* inserted integers have unacceptable semantic consequences: inserting a ‘`*`’ for example allows evaluation to continue.

After parsing has completed, a list of parsing errors (and their repairs) is returned to users, so that they can decide how much further they want to continue computation. There’s thus no danger of lrpar repairing input and the consumer of the parse not being able to tell that error recovery occurred. However, you might wonder why lrpar only allows fine-grained control of insert repairs. Surely it could also allow users to make fine-grained decisions in the face of delete repairs? Yes, it could, but I don’t think that would be a very common desire on the part of users, nor can I think how one would provide a nice interface for them to deal with such cases. What lrpar has is thus a pragmatic compromise. It’s also worth noting that although the above may seem very Rust specific, I’m confident that other languages can find a different, natural encoding of the same idea.

## Is it fast enough and good enough?

At this point you might be convinced that CPCT+ is a good idea in theory, but are unsure if it’s usable in practise. To me, there are two important questions: is CPCT+ fast enough to be usable? and is CPCT+ good enough to be usable? Answering such questions isn’t easy: until (and, mostly, after…) Carl Cerecke’s thesis, I’m not aware of any error recovery approach that had a vaguely convincing evaluation.

The first problem is that we need syntactically incorrect code to use for an evaluation but nearly all source code you can find in the wild is, unsurprisingly, syntactically correct. While there has been some work on artificially creating syntax errors in files, my experience is that programmers produce such a mind-boggling variety of syntax errors that it’s hard to imagine a tool accurately simulating them.

Unlike most previous approaches, we were fortunate that these days the BlueJ Java editor has an opt-in data-collection framework called [Blackbox](https://bluej.org/blackbox/) which records programmers (mostly beginners) as they’re editing programs. Crucially, this includes them attempting to compile syntactically incorrect programs. We thus extracted a corpus of 200,000 syntactically incorrect files which programmers thought were worth compiling (i.e. we didn’t take files at the point that the user was still typing in code). Without access to Blackbox, I don’t know what we’d have done: I’m not aware of any other language for which such a rich dataset exists.

There are a number of ways of looking at the “fast enough” question. On our corpus, the mean time CPCT+ spends on error recovery per file is 0.014s. To put that into context, that’s 3x faster than Corchuelo *et al.*, despite the fact that CPCT+ calculates the complete set of minimum cost repair sequences, while Corchuelo *et al.* finishes as soon as it finds one minimum(ish) cost repair sequence! I also think that the worst case is important. For various reasons, algorithms like CPCT+ really need a timeout to stop them running forever, which we set to a fairly aggressive 0.5s maximum per file, as it seems reasonable to assume that even the most demanding user will tolerate error recovery taking 0.5s. CPCT+ fully repaired 98.4% of files within the timeout on our corpus comparison Corchuelo *et al.* repaired 94.5% of files. In summary, in most cases CPCT+ runs fast enough that you’re probably not going to notice it.

A much harder question to answer is whether CPCT+ is good enough. In some sense, the only metric that matters is whether real programmers find the errors and repairs reported useful. Unfortunately, that’s an impractical criteria to evaluate in any sensible period of time. Error recovery papers which try to do so typically have fewer than 20 files in their corpus which leads to unconvincing evaluations. Realistically, one has to find an alternative, easier to measure, metric which serves as a proxy for what we really care about.

In our case, we use the total number of syntax errors found as a metric: the fewer the better. Although we know our corpus has at least 200,000 errors (at least 1 per file), we don’t know for sure how many more than that there are. There’s therefore no way of absolutely measuring an error recovery algorithm using this metric: all we can do is make relative comparisons. To give you a baseline for comparison, panic mode reports 981,628 syntax errors while CPCT+ reports 435,812\. One way of putting this into context is that if you use panic mode then, on average, you end up with an additional spurious syntax error for each syntax error that CPCT+ reports i.e. panic mode is much worse on this metric than CPCT+. Comparing CPCT+ with Corchuelo *et al.* is harder, because although Corchuelo *et al.* finds 15% fewer syntax errors in the corpus than does CPCT+, it also fails on more files than CPCT+. This is almost certainly explained by the fact that Corchuelo *et al.* is unable to finish parsing the hardest files which sometimes contain an astonishingly large number of syntax errors (e.g. because of repeated copy and paste errors).

Ultimately a truly satisfying answer to the “is CPCT+ good enough?” question is impossible — we can’t even make a meaningful comparison between CPCT+ and Corchuelo *et al.* with our metric. What we can say, however, pretty conclusively is that CPCT+ is much better than panic mode, the only other equivalent algorithm that’s ever seen somewhat widespread use.

## Limitations and future work

Few things in life are perfect, and CPCT+ definitely isn’t. There’s also clear scope to do better, and if I had a spare year or two to devote to the problem, there are various things I’d look at to make error recovery even better.

CPCT+ only tries repairing input at the point of a syntax error: however, that is often later than the point that a human would consider that they made an error. It’s unrealistic to expect CPCT+, or some variant of it, to deal with situations where the “cause” and “result” of an error are spread far apart. However, my experience is that the cause of an error is frequently just 1 or 2 tokens before the point identified as the actual error. It would be interesting to experiment with “rewinding” CPCT+ 1 or 2 tokens in the input and seeing if that’s a good trade-off. This isn’t trivial in the general case (mostly due to the parsing stack), but might be quite doable in many practical cases.

As we said earlier, CPCT+’s search is inherently local and even with repair ranking, it can suggest repair sequences which cause spurious errors. There are two promising, complementary, possibilities that I think might lessen this problem. The first is to make use of a little known, and beautiful approach, to dealing with syntax errors: [non-correcting error recovery](https://dl.acm.org/doi/10.1145/3916.4019). This works by discarding all of the input before the point of a syntax error and using a modified parser to parse the remainder: it doesn’t tell the user how to repair the input, but it does report the location of syntax errors. Simplifying a bit, algorithms such as CPCT+ over-approximate true syntax errors (i.e. they report (nearly) all “true” syntax errors alongside some “false” syntax errors) whereas non-correcting error recovery under-approximates (i.e. it misses some “true” syntax errors but never reports ”false” syntax errors). I think it would be possible to use non-correcting error recovery as a superior alternative to CPCT+’s current ranking system and, perhaps, even to guide the search from the outset. Unfortunately, I don’t think that non-correcting error recovery has currently been “ported” to LR parsing, but I don’t think that task is insoluble. The second possibility is to make use of machine learning (see e.g. [this paper](https://ieeexplore.ieee.org/document/8330219)). Before you get too excited, I doubt that machine learning is a silver bullet for error recovery, because the search space is too large and the variety of syntax errors that humans make quite astonishing. However, my gut feeling is that machine learning approaches will be good at recovering non-local errors in a way that algorithms like CPCT+ are not.

Less obviously, some Yacc grammars lend themselves to good repairs more than others. Without naming any names, some grammars are surprisingly permissive, letting through “incorrect” syntax which a later part of the compiler (or, sometimes, the parser’s semantic actions) then rejects . The problem with this is that the search space becomes very large, causing CPCT+ to either produce large numbers of surprising repair sequences, or, in the worst cases, not to be able finish its search in the alloted time. One solution to this is to rewrite such parts of the grammar to more accurately specify the acceptable syntax, though this is much easier for me to suggest than for someone to actually carry out. Another solution might be to provide additional hints to CPCT+ (along the lines of lrpar’s [`%avoid_insert`](https://softdevteam.github.io/grmtools/master/book/errorrecovery.html#biasing-repair-sequences) directive) that enable it to narrow down its search.

Programmers unintentionally leave hints in their input (e.g. indentation), and languages have major structural components (e.g. block markers such as curly brackets), that error recovery can take into account (see e.g. [this approach](https://repository.tudelft.nl/islandora/object/uuid%3Ac46d95b0-aae6-4dd9-aa9a-759bee35d577)). To take advantage of this, the grammar author would need to provide hints such as “what are the start / end markers of a block”. Such hints would be optional, but my guess is that most grammar authors would find the resulting improvements sufficiently worthwhile that they’d be willing to invest the necessary time to understand how to use them.

Finally, some parts of grammars necessarily allow huge numbers of alternatives and error recovery at those points is hard work. The most obvious example of this are binary or logical expressions, where many different operators (e.g. ‘`+`’, ‘`||`’ etc.) are possible. This can explode the search space, occasionally causing error recovery to fail, or more often, for CPCT+ to generate an overwhelming number of repair sequences. My favourite example of this – and this is directly taken from a real example, albeit with modified variable names! – is the seemingly innocent, though clearly syntactically incorrect, Java expression `x = f(""a""b);`. CPCT+ generates a comical [23,607 repair sequences for it](/laurie/blog/files/error_recovery/repairs.txt). What is the user supposed to do with all that information? I don’t know! One thing I experimented with at some points was making the combinatorial aspect explicit so that instead of:

```
Insert x, Insert +, Insert y Insert x, Insert +, Insert z Insert x, Insert -, Insert y Insert x, Insert -, Insert z 
```

the user would be presented with:

```
Insert x, Insert {+, -}, Insert {y, z} 
```

For various boring reasons, that feature was removed at some point, but writing this down makes me think that it should probably be reintroduced. It wouldn’t completely solve the “overwhelming number of repair sequences” problem, but it would reduce it, probably substantially.

## Summary

Parsing is the sort of topic that brings conversations at parties to a standstill. However, since every programmer relies upon parsing, the better a job we can do of helping them fix errors quickly, the better we make their lives. CPCT+ will never beat the very best hand-written error recovery algorithms, but what it does do is bring pretty decent error recovery to any LR grammar. I hope and expect that better error recovery algorithms will come along in the future, but CPCT+ is here now, and if you use Rust, you might want to take a look at grmtools — I’d suggest starting with the [quickstart guide in the grmtools book](https://softdevteam.github.io/grmtools/master/book/quickstart.html). Hopefully Yacc parsers for other languages might port CPCT+, or something like it, to their implementations, because there isn’t anything very Rust specific about CPCT+, and it’s a fairly easy algorithm to implement (under 500 lines of code in its Rust incantation).

Finally, one of the arguments that some people, quite reasonably, use against LR parsing is that it has poor quality error recovery. That’s a shame because, in my opinion [LR parsing is a beautiful approach to parsing](/laurie/blog/entries/which_parsing_approach.html). I hope that CPCT+’s error recovery helps to lessen this particular obstacle to the use of LR parsing.

**Acknowledgements:** My thanks to Edd Barrett and Lukas Diekmann for comments.

[Newer](/laurie/blog/2021/the_evolution_of_a_research_paper.html)

2020-11-17 08:00

[Older](/laurie/blog/2020/stick_or_twist.html)

If you’d like updates on new blog posts: follow me on

[Mastodon](https://mastodon.social/@ltratt)

or

[Twitter](https://twitter.com/laurencetratt)

; or

[subscribe to the RSS feed](../blog.rss)

; or

[subscribe to email updates](/laurie/newsletter/)

:

### Footnotes

### Comments