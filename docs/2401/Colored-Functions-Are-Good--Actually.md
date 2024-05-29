<!--yml
category: 未分类
date: 2024-05-27 15:22:17
-->

# Colored Functions Are Good, Actually

> 来源：[https://www.danmailloux.com/blog/colored-functions-are-good-actually](https://www.danmailloux.com/blog/colored-functions-are-good-actually)

# Colored Functions Are Good, Actually

In Bob Nystrom's excellent article [What Color is Your Function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/), he argues that "colored" (async) functions are bad because they create two separate code universes with rules about where you can call what and when. Kind of.

I'm going to flip that assertion on its head and argue that, in practice, this limitation is often a good thing. I'll admit that I'm setting up a bit of a strawman here, and because Bob is much smarter than me, his article also contains lots of technical details that may outweigh the benefits of what I'm about to say. Indulge my *sort of* correct intro, and I promise what's to come will be at least somewhat interesting.

 ## IO in Haskell

If you've spent some time in online programming circles, you've likely heard it said that JavaScript's promises are monads or that they are monadic. While a quick Google search reveals that this is not quite true, it is *kind of* true, and anyways this is just a botched segue in order to introduce monads.

Don't worry, this isn't another article about monads. I'm not gonna "say the line." (Sike - monads are just monoids in the category of endofunctors!!!) Really, I just want to talk about a specific monad - Haskell's `IO` monad. For the uninitiated, Haskell is a [*purely* functional programming language](https://en.wikipedia.org/wiki/Functional_programming), emphasis on pure. In Haskell, in order to do things like reading from a database, writing logs, or making an HTTP request, you must wrap that code in the `IO` monad.

"What's a monad?" you might be asking. It's a real rabbit hole, and to be honest, it doesn't really matter right now. If you're interested, I can recommend Arialdo Martini's [Monads For The Rest Of Us](https://arialdomartini.github.io/monads-for-the-rest-of-us) series, but you really don't need it to proceed, and to reiterate, monads are a *real* rabbit hole. **For the purposes of this article, think of `IO` as a simple type that wraps the return type of a function.** Much like how a function in JavaScript can return `promise<string>`, a function in Haskell can return `IO String`.

Let's see an example!

```
If we were to try and call `someUnexpectedIOThing` from inside the top `add` function, our program would not compile. The type signature of a function with side effects is different from the type signature of a similar function without side effects. Furthermore, functions with side effects can call other functions with or without side effects, but functions without side effects can only call other functions without side effects. You can probably see where I'm going with this.
It's easy to imagine the benefits of such an approach already. If you would think that working with `IO` functions is relatively more awkward than working with equivalent non-`IO` functions, you'd be correct. The effect of this is that the `IO` portion of your program naturally tends towards being only as large as it has to be, and the non-`IO` parts tend towards being as much of the program as is possible. This is a huge win because pure functions are easy to read, easy to reason about, and easy to change.
Async/Await in C#
Let's see a similar example in a language with colored functions like C#.

```
Again, we see a program where side effects are suggested by the presence of a type. In this case, that type is `Task`, in JavaScript, it would be `Promise`, and as we saw above - in Haskell that type is `IO`. Of course, there are important technical distinctions between `Task<int>` and `IO Int`, but they share one important benefit. **It signals to the programmer that something effectful is probably happening inside that code.**
Of course, in C# the compiler does not enforce this distinction, and not all effectful operations even are `async` such as writing to the console or generating a random number, but in practice, this turns out to be a pretty useful convention.
Acknowledgements
I'd like to point out that I am not the first person to notice this. Mark Seemann has written the excellent article [Async as surrogate IO](https://blog.ploeh.dk/2016/04/11/async-as-surrogate-io/) in which he uses much more rigor to truly compare `IO` and F#'s `Async`. His article is wonderful, and technical, and thought-provoking, and every programmer should read it.
This article, however, is not meant to be that. It's a more casual look at the topic which asks, "even if we can't rely on the similarity, is there still something valuable there?"
As programmers, we can often get fixated on edge cases and technical details, but I think it's okay to acknowledge helpful rules of thumb as well, even if they're not always 100% accurate. In our day to day work, many of us are often more like tradespeople than scientists. If noticing that a function is `async` makes you think, "hey, there's probably a side effect in there" - I think that's nice. :)
```

```