<!--yml
category: 未分类
date: 2024-05-29 12:46:34
-->

# Introducing Ruzzy, a coverage-guided Ruby fuzzer | Trail of Bits Blog

> 来源：[https://blog.trailofbits.com/2024/03/29/introducing-ruzzy-a-coverage-guided-ruby-fuzzer/](https://blog.trailofbits.com/2024/03/29/introducing-ruzzy-a-coverage-guided-ruby-fuzzer/)

*By Matt Schwager*

Trail of Bits is excited to introduce [Ruzzy](https://github.com/trailofbits/ruzzy), a coverage-guided fuzzer for pure Ruby code and Ruby C extensions. Fuzzing helps find bugs in software that processes untrusted input. In pure Ruby, these bugs may result in unexpected exceptions that could lead to denial of service, and in Ruby C extensions, they may result in memory corruption. Notably, the Ruby community has been missing a tool it can use to fuzz code for such bugs. We decided to fill that gap by building Ruzzy.

Ruzzy is heavily inspired by Google’s [Atheris](https://github.com/google/atheris), a Python fuzzer. Like Atheris, Ruzzy uses [libFuzzer](https://llvm.org/docs/LibFuzzer.html) for its coverage instrumentation and fuzzing engine. Ruzzy also supports [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) and [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) when fuzzing C extensions.

This post will go over our motivation behind building Ruzzy, provide a brief overview of installing and running the tool, and discuss some of its interesting implementation details. Ruby revelers rejoice, Ruzzy* is here to reveal a new era of resilient Ruby repositories.

* If you’re curious, Ruzzy is simply a portmanteau of Ruby and fuzz, or fuzzer.

## Bringing fuzz testing to Ruby

The Trail of Bits Testing Handbook provides the following [definition of fuzzing](https://appsec.guide/docs/fuzzing/):

> Fuzzing represents a dynamic testing method that inputs malformed or unpredictable data to a system to detect security issues, bugs, or system failures. We consider it an essential tool to include in your testing suite.

Fuzzing is an important testing methodology when developing high-assurance software, even in Ruby. Consider AFL’s extensive [trophy case](https://lcamtuf.coredump.cx/afl/), `rust-fuzz`’s [trophy case](https://github.com/rust-fuzz/trophy-case), and [OSS-Fuzz’s claim](https://google.github.io/oss-fuzz/#trophies) that it’s helped find and fix over 10,000 security vulnerabilities and 36,000 bugs with fuzzing. As mentioned previously, Python has Atheris. Java has [Jazzer](https://github.com/CodeIntelligenceTesting/jazzer/). The Ruby community deserves a high-quality, modern fuzzing tool too.

This isn’t to say that Ruby fuzzers haven’t been built before. They have: [kisaten](https://github.com/twistlock/kisaten), [afl-ruby](https://github.com/richo/afl-ruby), [FuzzBert](https://github.com/krypt/FuzzBert), and perhaps some we’ve missed. However, all these tools appear to be either unmaintained, difficult to use, lacking features, or all of the above. To address these challenges, Ruzzy is built on three principles:

1.  Fuzz pure Ruby code and Ruby C extensions
2.  Make fuzzing easy by providing a RubyGems installation process and simple interface
3.  Integrate with the extensive libFuzzer ecosystem

With that, let’s give this thing a test drive.

## Installing and running Ruzzy

The [Ruzzy repository](https://github.com/trailofbits/ruzzy#ruzzy) is well documented, so this post will provide an abridged version of installing and running the tool. The goal here is to provide a quick overview of what using Ruzzy looks like. For more information, check out the repository.

First things first, Ruzzy requires a Linux environment and a recent version of Clang (we’ve tested back to version 14.0.0). Releases of Clang can be found on its [GitHub releases](https://github.com/llvm/llvm-project/releases) page. If you’re on a Mac or Windows computer, then you can use Docker Desktop on [Mac](https://docs.docker.com/desktop/install/mac-install/) or [Windows](https://docs.docker.com/desktop/install/windows-install/) as your Linux environment. You can then use Ruzzy’s [Docker development environment](https://github.com/trailofbits/ruzzy#developing) to run the tool. With that out of the way, let’s get started.

Run the following command to install Ruzzy from [RubyGems](https://rubygems.org/gems/ruzzy):

```
MAKE="make --environment-overrides V=1" \
CC="/path/to/clang" \
CXX="/path/to/clang++" \
LDSHARED="/path/to/clang -shared" \
LDSHAREDXX="/path/to/clang++ -shared" \
    gem install ruzzy

```

These environment variables ensure the tool is compiled and installed correctly. They will be explored in greater detail later in this post. Make sure to update the `/path/to` portions to point to your `clang` installation.

## Fuzzing Ruby C extensions

To facilitate testing the tool, Ruzzy includes a [“dummy” C extension](https://github.com/trailofbits/ruzzy/blob/main/ext/dummy/dummy.c) with a heap-use-after-free bug. This section will demonstrate using Ruzzy to fuzz this vulnerable C extension.

First, we need to configure Ruzzy’s required [sanitizer options](https://github.com/google/sanitizers/wiki/SanitizerCommonFlags):

```
export ASAN_OPTIONS="allocator_may_return_null=1:detect_leaks=0:use_sigaltstack=0"
```

(See the [Ruzzy README](https://github.com/trailofbits/ruzzy#getting-started) for why these options are necessary in this context.)

Next, start fuzzing:

```
LD_PRELOAD=$(ruby -e 'require "ruzzy"; print Ruzzy::ASAN_PATH') \
    ruby -e 'require "ruzzy"; Ruzzy.dummy'

```

`LD_PRELOAD` is required for the same reason that [Atheris requires](https://github.com/google/atheris/blob/master/native_extension_fuzzing.md#option-a-sanitizerlibfuzzer-preloads) it. That is, it uses a special shared object that provides access to libFuzzer’s sanitizers. Now that Ruzzy is fuzzing, it should quickly produce a crash like the following:

```
INFO: Running with entropic power schedule (0xFF, 100).
INFO: Seed: 2527961537
...
==45==ERROR: AddressSanitizer: heap-use-after-free on address 0x50c0009bab80 at pc 0xffff99ea1b44 bp 0xffffce8a67d0 sp 0xffffce8a67c8
...
SUMMARY: AddressSanitizer: heap-use-after-free /var/lib/gems/3.1.0/gems/ruzzy-0.7.0/ext/dummy/dummy.c:18:24 in _c_dummy_test_one_input
...
==45==ABORTING
MS: 4 EraseBytes-CopyPart-CopyPart-ChangeBit-; base unit: 410e5346bca8ee150ffd507311dd85789f2e171e
0x48,0x49,
HI
artifact_prefix='./'; Test unit written to ./crash-253420c1158bc6382093d409ce2e9cff5806e980
Base64: SEk=

```

## Fuzzing pure Ruby code

Fuzzing pure Ruby code requires two Ruby scripts: a tracer script and a fuzzing harness. The tracer script is required due to an implementation detail of the Ruby interpreter. Every tracer script will look nearly identical. The only difference will be the name of the Ruby script you’re tracing.

First, the tracer script. Let’s call it `test_tracer.rb`:

```
require 'ruzzy'

Ruzzy.trace('test_harness.rb')

```

Next, the fuzzing harness. A fuzzing harness wraps a fuzzing target and passes it to the fuzzing engine. In this case, we have a simple fuzzing target that crashes when it receives the input “FUZZ.” It’s a contrived example, but it demonstrates Ruzzy’s ability to find inputs that maximize code coverage and produce crashes. Let’s call this harness `test_harness.rb`:

```
require 'ruzzy'

def fuzzing_target(input)
  if input.length == 4
    if input[0] == 'F'
      if input[1] == 'U'
        if input[2] == 'Z'
          if input[3] == 'Z'
            raise
          end
        end
      end
    end
  end
end

test_one_input = lambda do |data|
  fuzzing_target(data) # Your fuzzing target would go here
  return 0
end

Ruzzy.fuzz(test_one_input)
```

You can start the fuzzing process with the following command:

```
LD_PRELOAD=$(ruby -e 'require "ruzzy"; print Ruzzy::ASAN_PATH') \
    ruby test_tracer.rb

```

This should quickly produce a crash like the following:

```
INFO: Running with entropic power schedule (0xFF, 100).
INFO: Seed: 2311041000
...
/app/ruzzy/bin/test_harness.rb:12:in `block in ': unhandled exception
    from /var/lib/gems/3.1.0/gems/ruzzy-0.7.0/lib/ruzzy.rb:15:in `c_fuzz'
    from /var/lib/gems/3.1.0/gems/ruzzy-0.7.0/lib/ruzzy.rb:15:in `fuzz'
    from /app/ruzzy/bin/test_harness.rb:35:in `<top>'
    from bin/test_tracer.rb:7:in `require_relative'
    from bin/test_tracer.rb:7:in `

<main>'
...
SUMMARY: libFuzzer: fuzz target exited
MS: 1 CopyPart-; base unit: 24b4b428cf94c21616893d6f94b30398a49d27cc
0x46,0x55,0x5a,0x5a,
FUZZ
artifact_prefix='./'; Test unit written to ./crash-aea2e3923af219a8956f626558ef32f30a914ebc
Base64: RlVaWg==
</main></top> 
```

Ruzzy used libFuzzer’s coverage-guided instrumentation to discover the input (“FUZZ”) that produces a crash. This is one of Ruzzy’s key contributions: coverage-guided support for pure Ruby code. We will discuss coverage support and more in the next section.

## Interesting implementation details

You don’t need to understand this section to use Ruzzy, but fuzzing can often be more art than science, so we wanted to share some details to help demystify this dark art. We certainly learned a lot from the blog posts describing [Atheris](https://security.googleblog.com/2020/12/how-atheris-python-fuzzer-works.html) and [Jazzer](https://www.code-intelligence.com/blog/java-fuzzing-with-jazzer), so we figured we’d pay it forward. Of course, there are many interesting details that go into creating a tool like this but we’ll focus on three: creating a Ruby fuzzing harness, compiling Ruby C extensions with libFuzzer, and adding coverage support for pure Ruby code.

### Creating a Ruby fuzzing harness

One of the first things you need when embarking on a fuzzing campaign is a fuzzing harness. The Trail of Bits Testing Handbook [defines a fuzzing harness](https://appsec.guide/docs/fuzzing/#terminology) as follows:

> A harness handles the test setup for a given target. The harness wraps the software and initializes it such that it is ready for executing test cases. A harness integrates a target into a testing environment.

When fuzzing Ruby code, naturally we want to write our fuzzing harness in Ruby, too. This speaks to goal number 2 from the beginning of this post: make fuzzing Ruby simple and easy. However, a problem arises when we consider that libFuzzer is written in C/C++. When [using libFuzzer as a library](https://llvm.org/docs/LibFuzzer.html#using-libfuzzer-as-a-library), we need to pass a C function pointer to `LLVMFuzzerRunDriver` to initiate the fuzzing process. How can we pass arbitrary Ruby code to a C/C++ library?

Using a foreign function interface (FFI) like [Ruby-FFI](https://github.com/ffi/ffi) is one possibility. However, FFIs are generally used to go the other direction: calling C/C++ code from Ruby. [Ruby C extensions](https://guides.rubygems.org/gems-with-extensions/) seem like another possibility, but we still need to figure out a way to pass arbitrary Ruby code to a C extension. After much digging around in the Ruby C extension API, we discovered the [`rb_proc_call` function](https://github.com/ruby/ruby/blob/v3_3_0/proc.c#L985-L986). This function allowed us to use Ruby C extensions to bridge the gap between Ruby code and the libFuzzer C/C++ implementation.

In Ruby, a [`Proc`](https://ruby-doc.org/3.3.0/Proc.html) is “an encapsulation of a block of code, which can be stored in a local variable, passed to a method or another `Proc`, and can be called. `Proc` is an essential concept in Ruby and a core of its functional programming features.” Perfect, this is exactly what we needed. In Ruby, all lambda functions are also `Proc`s, so we can write fuzzing harnesses like the following:

```
require 'json'
require 'ruzzy'

json_target = lambda do |data|
  JSON.parse(data)
  return 0
end

Ruzzy.fuzz(json_target)
```

In this example, the `json_target` lambda function is passed to `Ruzzy.fuzz`. Behind the scenes Ruzzy uses two language features to bridge the gap between Ruby code and a C interface: Ruby `Proc`s and C function pointers. First, Ruzzy calls `LLVMFuzzerRunDriver` with a function pointer. Then, every time that function pointer is invoked, it calls `rb_proc_call` to execute the Ruby target. This allows the C/C++ fuzzing engine to repeatedly call the Ruby target with fuzzed data. Considering the example above, since all lambda functions are `Proc`s, this accomplishes the goal of calling arbitrary Ruby code from a C/C++ library.

As with all good, high-level overviews, this is an oversimplification of how Ruzzy works. You can see the exact implementation in [`cruzzy.c`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/ext/cruzzy/cruzzy.c).

### Compiling Ruby C extensions with libFuzzer

Before we proceed, it’s important to understand that there are two Ruby C extensions we are considering: the Ruzzy C extension that hooks into the libFuzzer fuzzing engine and the Ruby C extensions that become our fuzzing targets. The previous section discussed the Ruzzy C extension implementation. This section discusses Ruby C extension targets. These are third-party libraries that use Ruby C extensions that we’d like to fuzz.

To fuzz a Ruby C extension, we need a way to compile the extension with libFuzzer and its associated sanitizers. Compiling C/C++ code for fuzzing requires [special compile-time flags](https://llvm.org/docs/LibFuzzer.html#fuzzer-usage), so we need a way to inject these flags into the C extension compilation process. Dynamically adding these flags is important because we’d like to install and fuzz Ruby gems without having to modify the underlying code.

The [`mkmf`](https://ruby-doc.org/3.3.0/stdlibs/mkmf/MakeMakefile.html), or MakeMakefile, module is the primary interface for compiling Ruby C extensions. The `gem install` process calls a gem-specific Ruby script, typically named `extconf.rb`, which calls the `mkmf` module. The process looks roughly like this:

```
gem install -> extconf.rb -> mkmf -> Makefile -> gcc/clang/CC -> extension.so

```

Unfortunately, by default `mkmf` does not respect common C/C++ compilation environment variables like `CC`, `CXX`, and `CFLAGS`. However, we can force this behavior by setting the following environment variable: `MAKE="make --environment-overrides"`. This tells `make` that environment variables override `Makefile` variables. With that, we can use the following command to install Ruby gems containing C extensions with the appropriate fuzzing flags:

```
MAKE="make --environment-overrides V=1" \
CC="/path/to/clang" \
CXX="/path/to/clang++" \
LDSHARED="/path/to/clang -shared" \
LDSHAREDXX="/path/to/clang++ -shared" \
CFLAGS="-fsanitize=address,fuzzer-no-link -fno-omit-frame-pointer -fno-common -fPIC -g" \
CXXFLAGS="-fsanitize=address,fuzzer-no-link -fno-omit-frame-pointer -fno-common -fPIC -g" \
    gem install msgpack

```

The gem we’re installing is `msgpack`, an example of a gem containing a [C extension component](https://github.com/msgpack/msgpack-ruby/tree/master/ext/msgpack). Since it deserializes binary data, it makes a great fuzzing target. From here, if we wanted to fuzz `msgpack`, we would create an `msgpack` fuzzing harness and initiate the fuzzing process.

If you’d like to find more fuzzing targets, searching GitHub for [`extconf.rb` files](https://github.com/search?q=path%3A**%2Fextconf.rb&type=code) is one of the best ways we’ve found to identify good C extension candidates.

### Adding coverage support for pure Ruby code

Instead of Ruby C extensions, what if we want to fuzz pure Ruby code? That is, Ruby projects that do not contain a C extension component. If modifying install-time functionality via lengthy, not-officially-supported environment variables is a hacky solution, then what follows is not for the faint of heart. But, hey, a working solution with a little *artistic freedom* is better than no solution at all.

First, we need to cover the motivation for coverage support. Fuzzers derive some of their [“smarts”](https://blog.trailofbits.com/2017/02/16/the-smart-fuzzer-revolution/) from analyzing coverage information. This is a lot like code coverage information provided by unit and integration tests. While fuzzing, most fuzzers prioritize inputs that unlock new code branches. This [increases the likelihood](https://mboehme.github.io/paper/ICSE22.pdf) that they will find crashes and bugs. When fuzzing Ruby C extensions, Ruzzy can punt coverage instrumentation for C code to Clang. With pure Ruby code, we have no such luxury.

While implementing Ruzzy, we discovered one supremely useful piece of functionality: the Ruby [`Coverage`](https://ruby-doc.org/3.3.0/exts/coverage/Coverage.html) module. The problem is that it cannot easily be called in real time by C extensions. If you recall, Ruzzy uses its own C extension to pass fuzz harness code to `LLVMFuzzerRunDriver`. To implement our pure Ruby coverage “smarts,” we need to pass in Ruby coverage information to libFuzzer in real time as the fuzzing engine executes. The `Coverage` module is great if you have a known start and stop point of execution, but not if you need to continuously gather coverage information and pass it to libFuzzer. However, we know the `Coverage` module must be implemented somehow, so we dug into the Ruby interpreter’s C implementation to learn more.

Enter Ruby event hooking. The [`TracePoint`](https://ruby-doc.org/3.3.0/TracePoint.html) module is the official Ruby API for listening for certain types of events like calling a function, returning from a routine, executing a line of code, and many more. When these events fire, you can execute a callback function to handle the event however you’d like. So, this sounds great, and exactly like what we need. When we’re trying to track coverage information, what we’d really like to do is listen for branching events. This is what the `Coverage` module is doing, so we know it must exist under the hood somewhere.

Fortunately, the public Ruby C API provides access to this event hooking functionality via the [`rb_add_event_hook2` function](https://github.com/ruby/ruby/blob/v3_3_0/include/ruby/debug.h#L789). This function takes a list of events to hook and a callback function to execute whenever one of those events fires. By digging around in the source code a bit, we find that the list of possible events looks very similar to the list in the [`TracePoint` module](https://ruby-doc.org/3.3.0/TracePoint.html#class-TracePoint-label-Events):

```
 37    #define RUBY_EVENT_NONE      0x0000 /**< No events. */
 38    #define RUBY_EVENT_LINE      0x0001 /**< Encountered a new line. */
 39    #define RUBY_EVENT_CLASS     0x0002 /**< Encountered a new class. */
 40    #define RUBY_EVENT_END       0x0004 /**< Encountered an end of a class clause. */
       ...

```

[Ruby event hook types](https://github.com/ruby/ruby/blob/v3_3_0/include/ruby/internal/event.h#L37-L40)

If you keep digging, you’ll notice a distinct lack of one type of event: coverage events. But why? The `Coverage` module appears to be handling these events. If you continue digging, you’ll find that there are in fact coverage events, and that is how the `Coverage` module works, but you don’t have access to them. They’re defined as part of a private, internal-only portion of the Ruby C API:

```
 2182    /* #define RUBY_EVENT_RESERVED_FOR_INTERNAL_USE 0x030000 */ /* from vm_core.h */
 2183    #define RUBY_EVENT_COVERAGE_LINE                0x010000
 2184    #define RUBY_EVENT_COVERAGE_BRANCH              0x020000

```

[Private coverage event hook types](https://github.com/ruby/ruby/blob/v3_3_0/vm_core.h#L2182-L2184)

That’s the bad news. The good news is that we can define the `RUBY_EVENT_COVERAGE_BRANCH` event hook ourselves and set it to the correct, constant value in our code, and `rb_add_event_hook2` will still respect it. So we can use Ruby’s built-in coverage tracking after all! We can feed this data into libFuzzer in real time and it will fuzz accordingly. Discussing *how* to feed this data into libFuzzer is beyond the scope of this post, but if you’d like to learn more, we use SanitizerCoverage’s [inline 8-bit counters](https://clang.llvm.org/docs/SanitizerCoverage.html#inline-8bit-counters), [PC-Table](https://clang.llvm.org/docs/SanitizerCoverage.html#pc-table), and [data flow tracing](https://clang.llvm.org/docs/SanitizerCoverage.html#tracing-data-flow).

There’s just one more thing.

During our testing, even though we added the correct event hook, we still weren’t successfully hooking coverage events. The `Coverage` module must be doing something we’re not seeing. If we call `Coverage.start(branches: true)`, per the [`Coverage` documentation](https://ruby-doc.org/3.3.0/exts/coverage/Coverage.html#module-Coverage-label-Branches+Coverage), then things work as expected. The details here involve a lot of sleuthing in the Ruby interpreter source code, so we’ll cut to the chase. As best we can tell, it appears that calling `Coverage.start`, which effectively calls `Coverage.setup`, initializes some global state in the Ruby interpreter that allows for hooking coverage events. This initialization functionality is also part of a private, internal-only API. The easiest solution we could come up with was calling `Coverage.setup(branches: true)` before we start fuzzing. With that, we began successfully hooking coverage events as expected.

Having coverage events included in the standard library made our lives a lot easier. Without it, we may have had to resort to much more invasive and cumbersome solutions like modifying the Ruby code the interpreter sees in real time. However, it would have made our lives *even easier* if hooking coverage events were part of the official, public Ruby C API. We’re currently tracking this request at [`trailofbits/ruzzy#9`](https://github.com/trailofbits/ruzzy/issues/9).

Again, the information presented here is a slight oversimplification of the implementation details; if you’d like to learn more, then [`cruzzy.c`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/ext/cruzzy/cruzzy.c) and [`ruzzy.rb`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/lib/ruzzy.rb) are great places to start.

## Find more Ruby bugs with Ruzzy

We faced some interesting challenges while building this tool and attempted to hide much of the complexity behind a simple, easy to use interface. When using the tool, the implementation details should not become a hindrance or an annoyance. However, discussing them here in detail may spur the next fuzzer implementation or step forward in the fuzzing community. As mentioned previously, the [Atheris](https://security.googleblog.com/2020/12/how-atheris-python-fuzzer-works.html) and [Jazzer](https://www.code-intelligence.com/blog/java-fuzzing-with-jazzer) posts were a great inspiration to us, so we figured we’d pay it forward.

Building the tool is just the beginning. The real value comes when we start using the tool to find bugs. Like Atheris for Python, and Jazzer for Java before it, Ruzzy is an attempt to bring a higher level of software assurance to the Ruby community. If you find a bug using Ruzzy, feel free to open a PR against our [trophy case](https://github.com/trailofbits/ruzzy#trophy-case) with a link to the issue.

If you’d like to read more about our work on fuzzing, check out the following posts:

[Contact us](https://www.trailofbits.com/contact/) if you’re interested in custom fuzzing for your project.

### Like this:

Like Loading...

### *Related*