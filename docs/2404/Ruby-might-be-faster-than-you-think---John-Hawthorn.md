<!--yml
category: 未分类
date: 2024-05-27 13:32:38
-->

# Ruby might be faster than you think - John Hawthorn

> 来源：[https://www.johnhawthorn.com/2024/ruby-might-be-faster-than-you-think/](https://www.johnhawthorn.com/2024/ruby-might-be-faster-than-you-think/)

I saw a project a couple weeks back which allows [writing and running Crystal methods inline inside a Ruby file](https://github.com/wouterken/crystalruby).

It’s a neat project, and I don’t want to take away from it but something in the [README example](https://github.com/wouterken/crystalruby/blob/1d8b38e103d5f3a210e0b6fbc5c1465cade62258/README.md) looked off to me.

require 'crystalruby' require 'benchmark' module Fibonnaci crystalize [n: :int32] => :int32 def fib_cr(n) a = 0 b = 1 n.times { a, b = b, a + b } a end module_function def fib_rb(n) a = 0 b = 1 n.times { a, b = b, a + b } a end end puts(Benchmark.realtime { 1_000_000.times { Fibonnaci.fib_rb(30) } }) puts(Benchmark.realtime { 1_000_000.times { Fibonnaci.fib_cr(30) } })

My benchmark runs look pretty similar to the README’s. The “crystalized” version runs about 4x faster than the pure Ruby version.

```
$ ruby test.rb
2.113752398872748
0.6535678738728166 
```

But something is a bit off here. The Ruby implementation has a subtle mistake which causes signficantly more work than it needs to.

Ruby’s multiple assignment `a, b = b, a + b` is equivalent to `a, b = [b, a + b]`. Most of the time that Array allocation doesn’t actually happen, but because in this case it’s the last line of the block, and because Ruby has an implicit return at the end of the block the Array is required (even though `Integer#times` doesn’t use the return we don’t yet have an optimization which “knows” that).

Let’s see how we do avoiding that… (with a slightly unsightly `; nil` replacing the return)

def fib_rb(n) a = 0 b = 1 n.times { a, b = b, a + b; nil } a end

```
$ ruby test.rb
1.245176327880472
0.6385665240231901 
```

Not bad. We’re making up the difference. Now we’re only about 2x slower than crystal.

To make this a bit faster, instead of calling `Integer#times`, let’s inline the loop.

def fib_rb(n) a = 0 b = 1 while n > 0 a, b = b, a + b n -= 1 end a end

```
$ ruby test.rb
0.7689670620020479
0.638991191983223 
```

Nearly on par now! Finally, let’s enable YJIT, Ruby’s built-in JIT compiler to see the real comparison.

```
$ ruby --yjit test.rb
0.10502525512129068
0.5002881051041186 
```

Now it’s Ruby that’s 5 times faster than Crystal!!! And 20x faster than our original version. Though most likely that’s some cost from the FFI, or something similar, though that does seem like a surprising amount of overhead.

I thought it was notable that by making some minor tweaks to Ruby code it can now outperform a precompiled statically typed language in a purpose-built example of when it is slow. I’m hopeful that someday with future advancements in the Ruby JIT even the small tweaks might not be necessary.