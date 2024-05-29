<!--yml
category: 未分类
date: 2024-05-29 12:45:32
-->

# A step beyond Rust's pattern matching :: Radiki Dev — A blog by Chris Gioran

> 来源：[https://radiki.dev/posts/match-and-bind-patterns/](https://radiki.dev/posts/match-and-bind-patterns/)

### If you don’t need a refresh of basic Rust pattern match, go directly to [the new stuff](#glowdust) [#](#if-you-dont-need-a-refresh-of-basic-rust-pattern-match-go-directly-to-the-new-stuffglowdust)

## A very opinionated reminder of patterns in Rust [#](#a-very-opinionated-reminder-of-patterns-in-rust)

Here’s a simple pattern match in Rust:

Pretty anticlimactic, huh? Well, let me spice it up a bit:

Still not good enough? Ok, let’s try something fancier:

```
let (a, 2) = (1, 2); // Does not compile  error[E0005]: refutable pattern in local binding  --> src/example.rs:3:9 | 3 |     let (a, 2) = (1, 2);  |         ^^^^^^ patterns `(_, i32::MIN..=1_i32)` and `(_, 3_i32..=i32::MAX)` not covered | = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant 
```

Well, that escalated quickly.

Now, `rustc` and I have had our disagreements, but in this case I can sort of see the point: This pattern can, in theory, fail. Our puny human eyes may not see it - in fact, you may think “hey, I am pretty sure 2 is equal to 2\. How can that fail?”

Good point. But what if it was

```
let (a, 2) = (1, 1+1); // Does not compile either 
```

Or some other, more complicated expression? No, says `rustc`, as long as there is an equality at play, (by having a constant on the left hand side, for example), I will not let you do this unconditionally.

But *conditionally*? No problem.

Let’s do that, then

```
if let (a, 2) = (1, 2) {  println!("Made it, a is {}", a); }   // prints "Made it, a is 1" 
```

Excellent. I can pattern match on constants and capture variables. Let’s do something more complex

```
if let (a, 2) = (1, 2) &&  let (b, 4) = (3, 4) && println!("Made it, a = {}, b = {}", a, b); }   // prints "Made it, a = 1, b = 3" 
```

Nice, I can do conditionals between pattern matches and variable bindings. The example above is silly because the pattern will always match, but the syntax is what matters here.

I can even build on this, to add conditions for the already bound variables:

```
if let (a, 2) = (1, 2) &&  let (c, 4) = (3, 4) && b == a + 2 { println!("Made it, a = {}, b = {}", a, b); }   // still prints "Made it, a = 1, b = 3" 
```

## And that’s where all the trouble begun [#](#glowdust)

Watching this, I couldn’t help but think that it looks a lot like a declarative query…thing?

Say I have a vector of pairs:

```
let the_data = vec![  (1, 2),  (2, 2)  (2, 3),  (5, 7),  (8, 8)]; 
```

I can grab an iterator out of it and do the same pattern matching in a for loop:

```
for (a, b) in the_data {  println!("({}, {})", a, b); }   // prints // (1, 2) // (2, 3) // (5, 7) // (8, 8) 
```

This is trivial, but it is also clearly a pattern match.

*And I want to do it conditionally, in the match itself.*

Let’s try a `while` instead of a `for`

```
while let Some((a, b)) = iterator.next() && *b == *a + 1  {  println!("({}, {})", a, b); }   // prints // (1, 2) // (2, 3) 
```

Ok, not bad, we’ve got some kind of a conditional going. But I want something more. I want to do it in the pattern binding:

```
while let Some((a, a + 1)) = iterator.next() { // How cool would it be if this worked?  println!("({}, {})", a, b); } 
```

Sadly, it doesn’t work:

```
error: expected a pattern, found an expression  --> src/example.rs:11:24 | 11 |     while let Some((a, a + 1)) = iterator.next() {  |                        ^^^^^ arbitrary expressions are not allowed in patterns 
```

Now, I am not a language designer, and I know next to nothing about Rust internals (I barely know Rust, to be honest). I can’t say if allowing this would break other things in the language or if it’s a well known bad idea.

Thankfully, my ignorance is my shield, and I have no problems whatsoever making this work in my own language.

## Pattern matching in Glowdust [#](#pattern-matching-in-glowdust)

The algorithm in Glowdust is simple.

Just like in Rust, if there is an unbound variable in the left hand side of a pattern, then it is bound to whatever is on the right hand side. From then on, it is bound, and it behaves as a match, again like Rust does.

*(NOTE: I say left hand/right hand side, but in Glowdust syntax the -> operator flips the two sides. Keep that in mind for the following examples)*

The difference is that it does both in the same pattern. Here’s an example, in Glowdust

First define a function to hold our data:

Then populate with the same data as the iterator example above and query that:

```
my(1) = (1, 2); my(2) = (2, 3); my(3) = (5, 7); my(4) = (8, 8);   match my(x) -> (y1, y1 + 1), return y1 
```

This works as expected:

But let’s not stop here. Let’s use this capability to do a join:

```
match my(_) -> (left, middle), my(_) -> (middle, right), return left, middle, right 
```

This works exactly because the first time a variable is met it is bound, and then it’s used as a *refutable* match.

`middle`, in this case, is bound in the outer loop and then its value is used as a refutable pattern in the inner loop.

It also has a very declarative, pattern matching feel to it, but it is very familiar and readable. The predicates it implies can be moved around and be optimized according to cardinalities and other statistics in the data store. They can even be pushed down to the storage layer, if computational storage becomes available.

Did I mention that it works? You can run these examples right now, today, in [Glowdust](https://codeberg.org/glowdust/glowdust).

## Query By Example didn’t go far enough [#](#query-by-example-didnt-go-far-enough)

This may remind you of Query By Example in MS Access or Hibernate.

It isn’t.

It’s Query By Pattern, which is much cooler.

This compiles down to proper bytecode, it isn’t just a DSL. You can have full expressions reusing variables that just came into scope, join on them and use them in further expressions like filters and (eventually) aggregations.

I think the comparison to Rust’s (and other languages’) patern matching is interesting, but I don’t remotely argue that it is better. It is, however, better suited for a database query language.

Which is quite fortunate, because that’s what I’m building.

* * *

As always, let me know your thoughts on [Mastodon](https://fosstodon.org/@chrisg)

And, if you find this interesting enough, you may want to [donate](https://liberapay.com/chris.gioran/) towards the costs of developing Glowdust.