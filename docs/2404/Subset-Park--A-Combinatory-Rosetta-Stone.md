<!--yml
category: 未分类
date: 2024-05-27 13:24:19
-->

# Subset Park: A Combinatory Rosetta Stone

> 来源：[https://blog.zdsmith.com/posts/a-combinatory-rosetta-stone.html](https://blog.zdsmith.com/posts/a-combinatory-rosetta-stone.html)

2024-04-14

In [Combinatory Programming](./combinatory-programming.html), we attempt to provide motivating examples for the various combinators identified as useful to everyday programmers.

The aim of that piece was to extract the basic concept of combinators from as much its context as possible, and present certain particularly useful combinators as higher-order functions callable inside of nearly any programming language, applicable to most styles of programming. In practice, of course, tacit forms—the style of programming that we can use combinator functions to achieve—are more at home in certain languages and certain contexts than others, and compose most nicely with other features of a programming environment: plentiful pure, first-class functions; partial application; and terse combinator syntax, among others.

In this supplement, we can dig deeper into some of those examples; where useful, expressing the same logic in multiple styles, and at times commenting on features of different styles or languages and the effects they produce.

## identity

```
const maybeAbs = n => shouldNormalizeNegatives ? Math.abs(n) : n
return nums.map(maybeAbs) 
```

Here, `maybeAbs` is a function which either calls `Math.abs` on `n` or returns it directly, depending on the value of `shouldNormalizeNegatives`. This is the most explicit, referring directly to `n` by name three times, but also the easiest to read.

```
const maybeAbs = shouldNormalizeNegatives ? Math.abs : identity 
```

```
(def maybe-abs (if should-normalize-negatives math/abs identity))
(map mayb-abs nums) 
```

The JavaScript and Janet versions behave the same: we bind *either* the absolute-value function *or* `identity` to our new symbol depending on the value of the conditional. In later examples, we’ll see less trivial usages of `identity`.

### A Conditional Combinator

```
 is_between_negative_3_and_0 =: (<&0)*.(>&_4)
    maybe_abs =: [`| @. is_between_negative_3_and_0
    maybe_abs _5 _4 _3 _2 _1 0 1 2 3
_5 _4 3 2 1 0 1 2 3 
```

Though we haven’t put it into our personal library of combinators, we can imagine a function `agenda` that behaves like this:

```
function agenda(p, f, g) {
    return x => {
        if (p(x)) {
            return f(x)
        } else {
            return g(x)
        }
} 
```

In other words, we can imagine a simple combinator that lets us express conditional logic tacitly. J’s Agenda `@.` is such a function. In the slightly silly J example above, we are able to reproduce behaviour closer to the JavaScript example: rather than using a conditional to choose a function `f` or `g`, which is then applied to some values, our J code takes a verb `between_negative_3_and_0`, which is then called on every element of the argument to `maybe_abs`. Thus we can use *both* `f` *and* `g` within a single call. In this example, `identity` is spelled `[`.

## compose

```
const cleanData = datum => removeUndefineds(coalesceNaNs(datum))
const cleanedData = dirtyData.map(cleanData) 
```

In the case of composed functions (as opposed to the minimal `identity` example above), explicit code rarely has many advantages over tacit code. We have to refer to `datum` twice, so:

*   We have to introduce a name;
*   It’s not very short.

But in addition to that, the explicit version is structurally complex. Nested function calls are difficult to read.

```
const cleanData = compose(removeUndefineds, coalesceNaNs) 
```

```
(def clean-data (comp remove-undefineds coalesce-nans)) 
```

In both tacit versions, we avoid an unnecessary name in addition to obtaining a simpler structure, with less nesting.

```
clean_data =: remove_undefineds @ coalesce_nans 
```

In J, function composition is an infix operator.

### Absolute Difference

```
const absoluteDifference = (x, y) => Math.abs(x - y)
return absoluteDifference(9, 13) // => 4 
```

```
const absoluteDifference = compose(Math.abs, _.subtract) 
```

In this case, the explicit code requires us to bind to both `x` and `y`; in a case like subtraction, which is so intuitively transparent, this seems particularly unnecessary.

```
(def absolute-difference (comp math/abs -))
(absolute-difference 11 24) # => 13 
```

The Janet code also takes advantage of the fact that `-` is a normal function, and not an infix operator, while the JavaScript has to rely on Lodash’s `subtract` to get a first-class function that does subtraction.

```
absolute_difference =: |@:- 
```

The J code is another straightforward application of function composition. In this case, however, the composition of subtraction with absolute value is spelled `|@:-`, which is shorter than the variable name `absolute_difference`.

Array language practitioners will contend that this obviates the need for binding the verb to a name at all; why assign it to a variable name when it would require more characters than simply re-spelling it out each time?

One possible answer: it’s more straightforward to understand the *semantics* of `absolute_difference` than of `|@:-`. I’m not a very experienced J programmer, so I can’t argue one way or the other.

### Pipes

It’s worth calling out that function composition is traditionally written “to the left”; that is, the innermost function is the last argument and the last function to be applied is the first in the argument list. This maps cleanly to traditional function call syntax—`f(g(x))` has `f` and `g` in the same order as `(compose f g)`—but ends up being harder to read, as the order of application isn’t the same as the order of the syntax.

Many newer languages have syntax, or standard library functions, to effectively perform function composition in reverse, so that the call to the “combinator” (such as it is) can be read in the same order as function application.

For instance, in Janet we could natively write

```
(-> dirty-data (coalesce-nans) (remove-undefineds)) 
```

This is not properly a combinator, because it’s not a higher-order function. It’s syntax for a particular tacit form of control flow; it represents the actual application of those two functions to `dirty-data`.

Nevertheless we can imagine a so-called `pipe` that behaves just like `compose` but in reverse order, which might feel more comfortable to those programmers who have grown used to the conveniences of `->` and its cousins.

## apply

```
const basesAndExponents = [[0, 1], [1, 2], [3, 5], [8, 13]]
return basesAndExponents.map(([base, exponent]) => base ** exponent) // => [ 0, 1, 243, 549755813888 ] 
```

Absent `apply`, we explicitly destructure the two elements of the argument value and then pass them back to `Math.pow`. This is relatively verbose, though still readable. If we couldn’t avail ourselves of destructuring either, we’d be forced to write

```
args => {
    const base = args[0]
    const exponent = args[1]
    return base ** exponent
} 
```

Which begins to feel quite clumsy indeed.

```
return basesAndExponents.map(apply(Math.pow)) 
```

```
(def bases-and-exponents [[0 1] [1 2] [3 5] [8 13]])
(map (apply math/pow) bases-and-exponents) 
```

In the explicit example, we availed ourselves of JavaScript’s infix exponent operator, `**`. In the tacit examples, we use their respective languages’ standard library `pow` function in order to be able to pass a first-class argument to the `apply` combinator.

```
 exponentsAndBases: (0 1;1 2;3 5;8 13)
  pow: */# 
  pow .' exponentsAndBases
1 2 125 815730721 
```

The K language isn’t really designed with tacit programming in mind, but, as a member of the array family, it nevertheless has some features that make certain tacit forms the most idiomatic way to write programs. One of them is that `apply` is spelled `.`. Here `.'` means “apply each”, that is, it applies `pow` to each pair in the list of pairs `exponentsAndBases`.

We have used the flipped form `exponentsAndBases` here, because the tacit form `pow`, without being flipped itself, expects the exponent first.

`*/#` is itself an example of `compose`; as a function of two arguments, it applies `#` to them, creating an `exponent`-length array of `base`s, and then reduces `*` over them. In K, unlike J, function composition is not an infix operator; it’s accomplished simply by juxtaposing the two verbs `#` and `*/`.

In JavaScript and Janet, juxtaposition is not very meaningful. On the other hand, we will see later on that J relies heavily on juxtaposition, but assigns the semantics of combinators more complex than `compose` to that syntax.

## flip

```
const exponentsAndBases = [[0, 1], [1, 2], [3, 5], [8, 13]]
return exponentsAndBases.map(([exponent, base]) => base ** exponent) // => [ 1, 2, 125, 815730721 ] 
```

```
return exponentsAndBases.map(apply(flip(Math.pow))) // => [ 1, 2, 125, 815730721 ] 
```

```
(def exponents-and-bases [[0 1] [1 2] [3 5] [8 13]])
(map (apply (flip math/pow)) exponents-and-bases) 
```

### Flipping Without `apply`

The minimal example for `flip` includes usage of `apply`, because that’s the most natural way to imagine dealing with data consisting of multiple ordered values, where the order must be changed.

However, we can also imagine cases where our combinator accepts a variadic function such that ordering comes into play. For instance:

```
function explicit(x, y) {
    return [x ** y, y / x]
}
return explicit(2, 3) // => [ 8, 1.5 ] 
```

```
const tacit = recombine(Array.of, Math.pow, flip(_.divide)) 
```

In this slightly artificial example, we want to both take some `x` to the `y`, as well as divide `y` by `x`. In a case where the `g` and `h` of a `recombine` call expect the same arguments, but in different orders, we can use `flip` with one of them.

### Flipping Under Partial Application

Up until now we’ve excluded from consideration the topic of *partial functions*; in practice, the easy availability of partial application makes more tacit programming convenient.

```
(def double (partial * 2)) 
```

In Janet, with a built-in `partial` function, we can trivially use partial application to define `double`; we bind the first argument and produce a new function that takes a single argument and multiplies it by 2.

```
(defn square [n] (math/pow n 2)) 
```

If we would like to define `square` in terms of `math/pow`, the same technique isn’t naively applicable: in this case, the argument we want to bind is the second one.

```
(def square (partial (flip math/pow) 2)) 
```

In this case, we can work in a tacit style by employing `flip`; now the argument we want to bind is the first argument to the function, so we can pass that flipped function directly to `partial`.

### Flipping Under Partial Application (2)

Another example, from a refactor of [bagatto](https://git.sr.ht/~subsetpark/bagatto/commit/1f10e88) into tacit style. To be refactored is a higher-order function which takes some attribute name and returns a function that calls `sort` by getting that attribute.

```
(defn attr-sorter
  [key &opt descending?]
  (defn by [x y] ((if descending? > <) (x key) (y key)))
  (fn [items] (sort items by))) 
```

In tacit style, this becomes:

```
(defn attr-sorter
  ...
  (partial (flip sort) by)) 
```

By calling `flip` on sort, we obtain a function which can easily have our new `by` applied as the first argument.

## recombine

```
const mean = xs => _.sum(xs) / xs.length
return mean([0, 1, 1, 2, 3, 5, 8, 13]) // => 4.125 
```

```
const mean = recombine(_.divide, _.sum, _.size) 
```

```
(def mean (recombine / sum length)) 
```

```
mean =: +/ % # 
```

In the case of J we see perhaps the starkest example of that language’s orientation towards tacit programming. J combines a few syntactic characteristics such that simple juxtaposition of verbs, that is, placing syntactic verbs directly next to each other with no operator, triggers the behaviour of application combinators. In the case of `mean`, we see that creating a so-called 3-train of 3 verbs, `f g h`, creates a new verb whose behaviour is analogous to `recombine(g, f, h)`, called a *fork* in J.

### Min-max

```
const minMax = xs => [_.min(xs), _.max(xs)]
return minMax([1, 1, 2, 3, 5, 8, 13]) // => [ 1, 13 ] 
```

```
const minMax = recombine(Array.of, _.min, _.max) 
```

```
(def min-max (recombine array min-of max-of)) 
```

```
 min_max =: <./ , >./
   min_max 0 1 1 2 3 5 8 13
0 13 
```

### Plus or Minus

```
const plusOrMinus = (x, y) => [x + y, x - y]
return plusOrMinus(13, 8) // => [ 21, 5 ] 
```

```
const plusOrMinus = recombine(Array.of, _.add, _.subtract) 
```

As in Absolute Difference above, we see that functions of two arguments, when treated explicitly, result in quite a bit of noise. Explicit `plusOrMinus` takes two arguments, neither of which has a particularly meaningful name (judging from the Lodash docs, under addition they should be named `augend` and `addend`; under subtraction, `minuend` and `subtrahend`—but what would we call them when both operations come into play?), each of which then has to be referred to twice.

```
(def plus-or-minus (recombine array + -))
(plus-or-minus 13 8) 
```

```
 plus_minus =: - , +
   1 plus_minus 2
_1 3 
```

It’s worth noting that in both our JavaScript and Janet examples, the distinction between a unary recombined function, like `minMax`, and a variadic one, like `plusOrMinus`, is purely semantic: the syntax of application doesn’t change when the number of arguments expected by `g` and `h` do.

In the J example, on the other hand, our application syntax is by default infix; so while our `min_max` is written before its argument, our `plus_minus` is written in between its two arguments, syntactically identical to its constituent `g` and `h`.

### item-getter

Another example from [bagatto](https://git.sr.ht/~subsetpark/bagatto/commit/1f10e88) shows a real-world refactoring using `recombine`, as well as a non-trivial usage of `constant`.

`item-getter` is a function which takes a path of keywords and should return a function that, given two arguments, will retrieve the attribute at that path in the second argument.

A brief test to demonstrate the expected behaviour:

```
(deftest item-getter
  (let [item {:foo {:bar :baz}}
        getter (bagatto/item-getter [:foo :bar])]
    (is (== :baz (getter {} item))))) 
```

The explicit version:

```
(defn item-getter
  [path]
  (fn [site item] (get-in item path))) 
```

Tacit:

```
(defn item-getter
  [path]
  (recombine get-in right (constant path))) 
```

The tacit version, at least with all of the names spelled out, is *not shorter*. And if we’re still trying to remember what `recombine` does, it’s no easier to read.

But once we have internalized the behaviour of `recombine` a little bit, the tacit example has the advantage of not having to introduce any additional names at all. We can read that it returns a function that calls `get-in` on its second argument and `path`.

### Population Standard Deviation

```
function sum(xs) {
    return xs.reduce((n, m) => n + m) 
}
function std(xs) {
    const mean = sum(xs) / xs.length
    const squaredDifferences = xs.map((x - mean) ** 2)
    const meanSquares = sum(squaredDifferences) / xs.length
    return Math.sqrt(meanSquares)
}
return std([0, 1, 1, 2, 3, 5, 8, 13]) // => 4.136348026943574 
```

```
pow: {*/y#x}
mean: %/ (+/;#:) @\:
std: % mean @ pow[;2]' -/ 1 mean\ 
```

In this example, we showcase the effectiveness of partial functions towards programming in a tacit style.

In the first place, it’s important to note that in our K code we’ve defined `pow` differently from the `apply` example: here we’ve opted for an explicit definition so that the arguments to pow are `base`, `exponent`, which is arguably the more natural ordering.

We have also defined `mean` using the K translation of `recombine`: where our J code in that example took advantage of *juxtaposition* to create a so-called *3-train*, which J defines to behave as `recombine`, K has no such built-in support. The only effect of juxtaposition is composition, as we saw in the example of `*/#`. Instead we’ve used the phrase `@\:`, which means “apply each of the functions on the left”, with an array containing `sum` and `length`, and composed `%/` to its left, reducing the array containing the results of both operations with division.

Our final definition, `std`, is a tacit expression describing the composition of all the operations performed iteratively in the JavaScript code. In particular I want to highlight the spelling `pow[;2]`. This is a so-called *projection*, and its semantics are to produce a function which fixes `2` as the second argument to the function `pow`. K’s primitive and flexible syntax for projections makes it extremely convenient to describe partial function application; whereas in our `partial` example above, we needed to include `flip` in order to bind `2` to the second argument of `pow`: K’s projection syntax allows partial application in any argument positions.

The resulting function is a juxtaposition of the following functions

*   `1 mean\`: produce an array containing `x` and the mean of `x`.
*   `-/`: join those two values with `-`. Implicit in this step is the `map` that’s explicit in the JavaScript; as an array language, K “does the right thing” when subtracing a single value from an array of values.
*   `pow[;2]'`: square the differences.
*   `mean @`: take the mean.
*   `%`: take the square root.

In all cases besides the explicit `@`, the juxtaposition of two verbs is syntactically treated as function composition, producing a single composed function.

### Each Value is Unique

```
const eachValueIsUnique = xs => xs === _.uniq(xs)
return eachValueIsUnique([0, 1, 1, 2, 3, 5, 8, 13]) // => false
return eachValueIsUnique([0, 1, 2, 3, 5, 8, 13]) // => true 
```

```
const eachValueIsUnique = recombine(_.isEqual, identity, _.uniq) 
```

```
(def each-value-is-unique (recombine deep= identity distinct))
(each-value-is-unique [0 1 2 3 5 8 13]) 
```

```
 each_value_is_unique =: -: ~.
   each_value_is_unique 0 1 1 2 3 5 8 13
0
   each_value_is_unique 0 1 2 3 5 8 13
1 
```

J boasts yet another specialization for tacit programming; whereas we’ve seen the 3-train create a *fork*, a 2-train (which we express in terms of `recombine` by passing in `identity` as either `g` or `h`) has its own combinatorial meaning: it creates a *hook*, such that `f g` behaves like `x => f(x, g(x))`.

```
 eachValueIsUnique: ~/ 1 ?:\
    eachValueIsUnique 0 1 1 2 3 5 8 13
0
    eachValueIsUnique 0 1 2 3 5 8 13
1 
```

Other members of the array family tend to reserve 2-trains for simple function composition, as we have seen in other K examples. In K, the specialized hook behaviour can be achieved with `n-dos`, where `1 g\ x` produces an array of `(x;g x)`, which we can then fold some `f` over. If `f` needed to be passed `g(x), x`, then we could reverse the array before folding.

Equivalently, we could use `eachleft` in the same way we did to write `mean`:

```
 ~/ (?:;::) @\: 0 1 1 2 3 5 8 13
0
    ~/ (?:;::) @\: 0 1 2 3 5 8 13
1 
```

Here, the second of the two functions we apply is `::`, the identity function.

## under

```
const isAnagram = (x, y) => _.sortBy(x) === _.sortBy(y)
return isAnagram("live", "evil") 
```

```
const isAnagram = under(_.isEqual, _.sortBy) 
```

```
(def is-anagram (under deep= sort))
(is-anagram @"live" @"evil") 
```

```
sort =: /:~
    is_anagram =: -:&:sort
    is_anagram =: -:&:sort
   'evil' is_anagram 'live'
1 
```

In J, dyadic `under` is spelled Appose, `&:`.

It’s worth noting that with only one argument, `under` behaves just like `compose`: `x => _.isEqual(_.sortBy(x))`. (Thus, monadic Appose `&:` is equivalent to At `@:`, which we’ve seen above!)

The noteworthy difference, then, is how we want to wield the innermost function when given two or more arguments. Within `compose(f, g)`, all those arguments are passed to a single application of `g`, and the result is passed to `f`. Within `under`, `g` is treated as a single-argument function and each argument to the composition has `g` called on it individually.