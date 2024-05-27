<!--yml
category: 未分类
date: 2024-05-27 13:22:03
-->

# Programming vs. mathematical curiosity | Demian’s blog

> 来源：[https://demian.ferrei.ro/blog/programmer-vs-mathematician](https://demian.ferrei.ro/blog/programmer-vs-mathematician)

Recently i came across a puzzling property of numbers that got me to reflect on what “tickles” my mind as a programmer. And how programmers and mathematicians, even though they share much in common —a mix of playfulness and rigour, and a knack for logic problem solving—, might be motivated by very different things. Of course, this is all painting with very broad strokes. Every person is different.

Consider this simple question: what would happen if we pick a number, arrange its digits in descending and ascending order to get two different numbers, and then subtract those two?

For example, let’s pick the number 1988\. Arranging its digits in descending and ascending order we get 9881 and 1889, and subtracting those two we get 7992\. Riveting, yes. But what would happen if we repeat this process now starting from 7992, and then keep going?

Please take a moment and consider what would happen. Not necessarily to check it with pen and paper, but what does your intuition say?

Well, let’s see. Starting from 1988 the sequence would go:

Sidenote: if we had started from 2023 —the year i started writing and procrastinating this blog post— the sequence would look almost the same. The first step would be 3220 − 0223 = 2997, and after that it would continue the same way. This is not such a cosmic coincidence as it seems; the numbers on these sequences tend to repeat a lot.

> 1998 → 9881 − 1889 = 7992
> 
> 7992 → 9972 − 2799 = 7173
> 
> 7173 → 7731 − 1377 = 6354
> 
> 6354 → 6543 − 3456 = 3087
> 
> 3087 → 8730 − 0378 = 8352
> 
> 8352 → 8532 − 2358 = 6174
> 
> 6174 → 7641 − 1467 = 6174
> 
> 6174 → 7641 − 1467 = 6174
> 
> …

Once we hit 6174 the sequence starts repeating, as the result of applying this “biggest digit-arrangement minus smallest digit-arrangement” operation to 6174, is 6174 itself. Or in maths parlance: 6174 is a [fixed point](https://en.wikipedia.org/wiki/Fixed_point_(mathematics)) of this operation.

Now here’s the kicker: as long as the starting number is not a single repeated digit, **we can start from any 4-digit number and the sequence will always reach 6174**.

Of course, this is not a new discovery. 6174 is known as [Kaprekar’s constant](https://en.wikipedia.org/wiki/6174_(number)), in honor of the mathematician who found this curious property.

I don’t know about you, but this surprised me when i first heard about it. All numbers converging to a single one? Instead of maybe converging to different numbers, or getting into loops? Who would’ve expected that!?

But i also didn’t have the mathematical tools —or curiosity really— to dig into the maths of this to understand it more deeply. Luckily, there aren’t that many 4-digit numbers, and computers are pretty fast, or so i’ve been told. So instead of mathematically proving this the “correct” way, let’s have some fun and write a simple Ruby program to check this property for every 4-digit number. We could use any language really. I like using Ruby for this kind of explorations because it gets out of the way and allows expressing things quite freely.

### A simple subproblem

Let’s do some bottom-up development and start with the simplest bit: checking if a number consists of a single repeating digit. These numbers are called [repdigits](https://en.wikipedia.org/wiki/Repdigit) in recreational mathematics.

```
def repdigit?(n) = n.digits.uniq.size == 1 
```

There’s quite a bit going on on this line if you’re not used to Ruby, so let’s break it apart. The `n.digits` method gives us an array with the number’s base-10 digits. Then we filter out repeated digits using `uniq`. So we’re basically saying: a number is a repdigit if it has only one distinct digit. The code reads almost as a condensed version of it’s English explanation.

By the way, if you’re wondering about this `def name(args) = expr` weirdness, that is Ruby’s [“endless” method](https://bugs.ruby-lang.org/issues/16746) syntax, which debuted on version 3.0\. I’m still on the fence about it to be honest. On the one hand, it seems like the kind of unnecessary syntax sugar that tends to bloat languages and complicates answering basic questions like “how do i define a method?”. But at the same time, it can be useful to distinguish the cases where something can be expressed succinctly in a single expression, and the cases where multiple statements are needed.

Oh and yes, method names can end with `?`, which is actually the convention for boolean predicates in Ruby.

Anyway, back to our program! This `repdigit?` definition is correct and works for numbers of any length. But we only care about 4-digit numbers. And Kaprekar’s property also works for smaller numbers if we extend them with leading zeroes. For example, 777 is a repdigit, but if we consider it a 4-digit number and write it as 0777 then we can start the sequence with 7770 - 0777 = 6993 and we’d reach Kaprekar’s constant in 3 more steps.

So we only care about repdigits of 4 digits like 1111, 2222, and so on. In other words, multiples of 1111:

```
def repdigit?(n) = n % 1111 == 0 
```

### Kaprekar’s routine

Jumping up a bit in abstraction level, the logic of repeatedly rearranging the digits of a number into ascending and descending order, and subtracting those two until the result starts to repeat —aka Kaprekar’s routine— can be encoded as:

```
def kap(n)
  d = descending_digits(n) - ascending_digits(n)
  n == d ? n : kap(d)
end 
```

We take the difference of the two rearrangements of the number’s digits (which we haven’t defined yet). If that difference is equal to the input number, then we have found where the sequence starts repeating and we return that number. And if it’s a different number, we recur and do the same thing again this time with the difference.

Note that this recursive function only terminates if the operation reaches the same number at some point. But we *know* that to be true; that’s the whole point of writing this program! We just want to check this programmatically. It doesn’t make sense to defend against an infinite recursion that we know won’t happen. Even more: [it is known](https://en.wikipedia.org/wiki/Kaprekar%27s_routine#Kaprekar's_constants_in_base_10) that for 4-digit numbers this operation actually converges in 7 or less steps, so if we preferred iteration over recursion we could’ve written:

```
def kap(n)
  7.times do
    n = descending_digits(n) - ascending_digits(n)
  end
  n
end 
```

Both versions will work fine for our purposes. If for some reason Kaprekar’s property doesn’t hold for some number, the iterative version will give us a wrong result, and the recursive version will hang forever or overflow the stack. Which of the two? That’s [an interesting question](https://nithinbekal.com/posts/ruby-tco/)! Either way, we would know that we have screwed up our “proof” somewhere. *It’s fine.*

### Rearranging digits

Rearranging the digits of a number to be in ascending order is straightforward now that we know about the `digits` method:

```
def ascending_digits(n) = n.digits.sort.join.to_i 
```

We take the digits of the number, sort them, then `join` them into a string, and then convert that string to an integer with `to_i`. Maybe a bit too many unnecessary allocations for some tastes, but for a small proof program like this, it’s totally fine.

We can quickly test this on Ruby’s interactive console, `irb`:

REPLs are great for this kind of exploratory programming. Instead of the more “classic” approach of writing a program on a source file, running it, tweaking it, running it again, etc, a REPL allow for a much tighter, almost immediate, feedback loop. Once you have your little bits of program written and tested on the console, you can copy them onto a program file.

```
$ irb irb(main):001>  def ascending_digits(n) = n.digits.sort.join.to_i
=> :ascending_digits
irb(main):002>  ascending_digits(3713)
=> 1337 
```

Now, for rearranging the digits in *descending* order, a naive solution could be to just add a `reverse` call after sorting:

```
def descending_digits(n) = n.digits.sort.reverse.join.to_i 
```

But this logic hides a sneaky bug. Can you spot it?

We can test `descending_digits` on the console, which seems to work fine and even makes `kap(n)` return the Kaprekar constant for the example number we started with:

```
irb(main):007>  descending_digits(1988)
=> 9881
irb(main):008>  kap(1988)
=> 6174 
```

But when called with the first number of 4 digits, 1000, we get:

```
irb(main):009>  kap(1000)
=> 0 
```

Which is not right. 1000 is not a repdigit. It should not converge to 0.

What’s happening here is that, on the first iteration, `ascending_digits(1000) == 1` and `descending_digits(1000) == 1000`, which are both correct. Their difference is 999 though, and when we try to get its digits’ rearrangements on the second iteration we get 999 for both the ascending and descending variants, which of course then causes the result of the subtraction to be 0 and then repeat.

We need to make `descending_digits` consider its inputs as 4-digit numbers, so that 999 is seen as 0999, and then its descending order digit rearrangement would be 9990 instead of 999\. Here’s one way of doing this:

```
def descending_digits(n) = n.digits.sort.reverse.join.ljust(4, '0').to_i 
```

After sorting the digits and joining them into a string, we’re using `ljust(4, '0')` to pad the string with zeroes to the right so it is 4 characters long. A bit hacky, yes, but it gets the job done. At least it’s quite succinct, and readable enough. If you can think of a better alternative, please let me know!

With that hotfix in, `kap(n)` now seems to work as intended:

```
irb(main):011> kap(1988)
=> 6174
irb(main):012> kap(1000)
=> 6174 
```

### The final “proof”

Now at the top of this bottom-up process, we can define the main logic of the program:

```
1.upto(9999) do |n|
  repdigit?(n) or kap(n) == 6174 or
    fail "#{n} is not a repdigit nor does it converge to 6174"
end 
```

Again, notice how Ruby code can read as a condensed version of its English translation: for every number from 1 up to 9999, the number is either a repdigit, or its Kaprekar sequence converges to 6174\. If neither of these is true, the “proof” fails.

You may be wondering why i used the wordy `or` boolean operator instead of the more common symbolic `||` alternative. Well, it’s a stylistic choice really. First, if i had used `||`, the `fail` method call would’ve needed parentheses around its argument, which i prefer to avoid since i think of `fail` as control flow and i like to visually distinguish control flow methods from other “normal” method calls, like `kap(n)` or `repdigit?(n)` in this case. And second, using `and` and `or` as control flow operators —as alternatives to `if` and `unless` respectively— is [a brilliant idea](https://avdi.codes/using-and-and-or-in-ruby/) and can help expressing things in the “natural” way we conceive them. Although, i’ll admit this might be a form of Stockholm Syndrome; a rationalization of Ruby’s [multiple ways](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it) of doing the same thing. Generally i prefer languages having one obvious way of doing things. So i’ll think about my choice here as a rare exception to my usual structuredness :)

Stitching all these snippets together, the complete “proof” program is:

```
def kap(n)
  d = descending_digits(n) - ascending_digits(n)
  n == d ? n : kap(d)
end

def ascending_digits(n) = n.digits.sort.join.to_i
def descending_digits(n) = n.digits.sort.reverse.join.ljust(4, '0').to_i
def repdigit?(n) = n % 1111 == 0

1.upto(9999) do |n|
  repdigit?(n) or kap(n) == 6174 or
    fail "#{n} is not a repdigit nor does it converge to 6174"
end 
```

And running it confirms that, indeed, Kaprekar’s routine converges to 6174 for all 4-digit numbers as long as the number is not a single repeated digit.

### Wait, wasn’t this about maths or something?

Well, yes. And no. The story starts as a maths puzzle, but it’s really about the journey after that.

In my case at least, when i learned about Kaprekar’s result, my mind immediately jumped from “that cannot be!” to “i *must* write a program to check this for all numbers!”. And you can see where my curiosity wandered while writing that program:

*   to different ways of expressing the same idea and trying to find one i like the best
*   to recursion vs iteration
*   to programming languages’ questionable syntactic choices
*   to programming languages’ philosophies
*   to writing software top-down vs bottom-up

Of course, these are all things that interest me, and so naturally my mind went there. Once i got the program written and running, the little speck of mathematical curiosity i initially had was gone.

I suspect a more maths-oriented person would go through this very differently. Maybe they’d try to find an elegant analytical proof, without the aid of a dumb number-crunching machine brute-forcing its way through. Or they’d try to generalize the problem to different number of digits, or different number bases. I don’t really know.

But i appreciate having played with this. It seems i’m not that into maths as i once thought. And that i love programming. Who would have thought?

And how about you? What was on *your* mind while your eyes read this? Assuming they did, in which case: thank you very much for coming along.