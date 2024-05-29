<!--yml
category: 未分类
date: 2024-05-27 14:49:33
-->

# Maybe Everything Is a Coroutine - Adam Nelson

> 来源：[https://adam.nels.onl//blog/maybe-everything-is-a-coroutine/](https://adam.nels.onl//blog/maybe-everything-is-a-coroutine/)

I was inspired, after reading the excellent blog post [Let Futures Be Futures](https://without.boats/blog/let-futures-be-futures/), by the author's thought experiment of a language in which all functions are coroutines and this is used to express asynchronicity: async functions can yield a type called `Pending` when awaiting some async action, while pure, synchronous functions can yield `Never`, indicating that they never yield at all.

The more I thought about this, the more I realized that a strongly-typed language in which *every function is a coroutine* could actually have a lot of nice properties. I've come up with a design with several cool features, including:

*   A type system in which coroutines are basically state machines
*   A typed (algebraic) effect system based on coroutines
    *   Functions can be generic over effects and purity
    *   Effects can be "intercepted" by a caller and faked
*   A powerful exception system based on simple sum types
    *   An exception pass-through system that feels like a saner, generic version of checked exceptions
    *   Common Lisp-style resumable conditions
*   Immutable data structures and pure-by-default functions, but with the ability to use mutation in algorithms, by keeping all mutation in a function's scope and using coroutines to control state

How does it work? There are three key features: *sequence types*, the *coroutine for loop*, and the *pass operator*.

## Sequence Types

Ordinary functions have *argument types* and a *return type*:

```
fib(n: Int) -> Int 
```

In this everything-is-a-coroutine language, instead of a return type, every function has a *sequence type*. This is a sum type describing every type that the function can yield and every type that it can be resumed with after a yield.

For example, a coroutine version of `fib` that, instead of returning the `n`th Fibonacci number, yields an infinite sequence of the Fibonacci numbers:

```
fib() -> Int() 
```

The `()` means that, after the coroutine yields an `Int`, it can be *resumed* with no arguments. Because there is no base case (no branch that cannot be resumed), this coroutine must be infinite; it continues until the caller chooses not to resume it.

Consider instead a coroutine that yields a *finite* sequence of `Int`s:

```
factors(n: Int) -> Int() | Void 
```

This sequence type has two *cases*: either it emits an `Int` and can be resumed with no arguments, or it emits nothing and cannot be resumed.

Notice the syntax of `Void`: a type with no parentheses is a valid sequence type. This means that the ordinary function `fib(n: Int) -> Int` is a valid coroutine as well—it yields `Int` once, then cannot be resumed. Pure functions are a subset of coroutines!

These strongly-typed coroutines are essentially state machines. Let's define an additional piece of syntax: a *symbol* `:foo` is a unique singleton object whose type is also `:foo`. Using symbols, we can define a simple state machine for a door:

```
door() ->
  | :open(:close)
  | :closed(:open | :lock)
  | :locked(:unlock) 
```

Each state accepts only a specific set of actions. An open door can only be closed; a locked door can only be unlocked; a closed door can be opened or locked.

## Coroutine For Loop

But how do we *use* these coroutines? The syntax for basic function calls remains the same; if a coroutine has only no-resume types in its sequence type, then it can be used directly as an argument or assigned to a variable.

For more complex coroutines, there is a for loop syntax. Consider the door state machine from before; this is an example of a for loop that uses it:

```
for (door()) {
  | :locked ->
    continue(:unlock)
  | :closed ->
    continue(:open)
  | :open ->
    break
} 
```

This loop pattern-matches on the values yielded by `door`. The branches use the familiar keywords `continue` and `break`, but `continue` takes arguments now: it resumes `for`'s coroutine with new arguments, which must match the argument type of the current branch of the sequence type. The loop does not end until it encounters a `break`.

We can add some conveniences for the common case of coroutines that yield a list of values without any resume arguments. `continue()` is inferred at the end of blocks whose resume type is `()`, `break` is inferred at the end of blocks with no resume type, `Void` branches can be omitted, and wildcard matches are assumed to not match `Void`.

```
def someNumbers() -> Int() | Void {
  yield 1
  yield 2
  yield 3
}

var total
for (someNumbers()) { n ->
  total += n
} 
```

## The Pass Operator

Extending this coroutine syntax to support algebraic effects and error handling is straightforward. Errors can be yielded as ordinary values, which usually do not resume:

```
divide(divisor: Int, dividend: Int) -> DivideByZeroError | Int 
```

Algebraic effects can be yielded, presumably all the way up to the main function, which will then resume with a response to the given effect. These get complex, but I'll introduce syntax sugar to deal with these huge signatures later.

```
readFile(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | Bytes 
```

But actually *using* effects and errors with the mechanisms we already have (for loops and `yield`) would be extremely cumbersome. It would look a lot like Go: manually checking for error states or effects everywhere, just to `yield` them to the next level in the stack.

```
readConfigJson(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | JsonParseError
  | JsonFormatError
  | ConfigFile
{
  for (readFile(filename)) {
    | IOFileStat a -> continue(yield a)
    | IOFileRead a -> continue(yield a)
    | FileError e -> yield e
    | Bytes data ->
      // borrowing some Scala syntax
      parseJson(data) match {
        | JsonParseError e -> yield e
        | JsonValue json -> yield parseConfigFile(json)
      }
  }
} 
```

The common pattern here is `a -> continue(yield a)` (for branches that resume) and `e -> yield e` (for branches that don't). This pattern is so common in this style of code that it deserves its own piece of syntax sugar: the *pass operator*, `^`.

Placing `^` before an expression causes any value yielded by that expression that can be directly yielded (as `continue (yield a)` or just `yield a`) to be yielded. In other words, it causes any types contained in the function's sequence type to be *passed through* from this expression to the caller.

This greatly simplifies the example function:

```
readConfigJson(filename: String) ->
  | IOFileStat(FileError | FileInfo)
  | IOFileRead(FileError | Bytes)
  | FileError
  | JsonParseError
  | JsonFormatError
  | ConfigFile
{
  val data = ^readFile(filename)
  val json = ^parseJson(data)
  yield parseConfigFile(json)
} 
```

The sequence type is still a mess, though. And there's another problem: what if a function yields a bunch of effect types, but also a type that you want to capture in a variable... that also happens to be in your function's sequence type?

Here are some changes that solve these problems:

*   The keyword `infer Type` can be used in any branch of the sequence type; its type is inferred as all subtypes of `Type` that are yielded by the function. Values passed through by the `^` operator are included in this inference.
    *   `infer Type(*)` also infers the resume arguments; this can infer multiple branches.
*   The `^` operator can occur in front of sequence type branches. This causes `^` in the function body to only pass through types that also have a `^` in front of them in the sequence type.
*   A block or keyword, like `nopass`, could prevent a type from being passed through; this would work like a catch block, ensuring the function can handle a specific yielded type.

With these changes, and assuming that supertypes `FileIO` and `Error` exist, the function becomes even simpler!

```
readConfigJson(filename: String) -> ^infer IOFile(*) | ^infer Error | ConfigFile {
  val data = ^readFile(filename)
  val json = ^parseJson(data)
  yield parseConfigFile(json)
} 
```

## Putting it All Together

These three key features are a surprisingly solid core for a language, but they aren't a full language on their own. They could fit into several different language designs, but, in my opinion, the ideal language built on this foundation would have:

*   Strict functional purity outside of algebraic effects and mutable local variables. You can only perform side effects by yielding effect objects from `main`.
    *   Mutable local variables can still allow for efficient mutable data structures, thanks to coroutines. A mutable, resizeable array type can exist, but it cannot be yielded, passed as an argument, or stored in a struct. Structures of these arrays can then be manipulated by passing a coroutine into a function, then using actions yielded by that coroutine to manipulate the arrays defined in that function's scope.
*   Immutable, nominally-typed structs with subtyping.
*   A generic tuple type, allowing for tuples-with-symbols message passing a la Erlang, but strongly typed.
*   Full higher-order generics. This allows functions to be generic over purity and generic over checked exceptions, two qualities that are very rarely found in type systems.
*   Type aliases for sequence types, which can include multiple branches. The syntax for this would likely use `(*)`. For example, you can define `type Ints -> Int() | Void`—the `->` indicates that it's a sequence type—and then use it like this: `someFunction() -> Ints(*)`.
    *   Using these types, functions can take sequence arguments. For example, `sum(seq -> Ints(*)) -> Int` is a sum function over a finite sequence of `Int`. It can be called with `sum(someFunction())`, and the *entire sequence* returned by `someFunction` will be passed in to `sum`.
    *   A syntax for standalone sequences should also exist, basically the coroutine equivalent of an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). For example, `-> { yield 1 }` would be a sequence of type `-> Int` by default, but could also be explicitly typed: `-> Int() | Void { yield 1 }`.
    *   Sequence literals could be written with square brackets: `[1, 2, 3]` would be syntax sugar for `-> Int() | Void { yield 1; yield 2; yield 3 }`.

I have no idea when I'll find the time to actually work on this, but I'd like to see this language come together. I feel like this everything-is-a-coroutine type system has a lot of potential as a new architecture for functional programming.