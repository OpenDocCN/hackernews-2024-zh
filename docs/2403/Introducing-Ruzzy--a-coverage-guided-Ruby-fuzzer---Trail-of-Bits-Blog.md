<!--yml

类别：未分类

日期：2024-05-29 12:46:34

-->

# 介绍 Ruzzy，一个覆盖引导的 Ruby 模糊测试工具 | Trail of Bits 博客

> 来源：[https://blog.trailofbits.com/2024/03/29/introducing-ruzzy-a-coverage-guided-ruby-fuzzer/](https://blog.trailofbits.com/2024/03/29/introducing-ruzzy-a-coverage-guided-ruby-fuzzer/)

* Matt Schwager *

Trail of Bits 非常激动地介绍 [Ruzzy](https://github.com/trailofbits/ruzzy)，这是一个针对纯 Ruby 代码和 Ruby C 扩展的覆盖引导模糊测试工具。模糊测试有助于发现处理不受信任输入的软件中的漏洞。在纯 Ruby 中，这些漏洞可能导致意外异常，进而可能导致拒绝服务，而在 Ruby C 扩展中，可能会导致内存损坏。值得注意的是，Ruby 社区一直缺少一个可以用来模糊测试此类漏洞的工具。我们决定填补这一空白，开发了 Ruzzy。

Ruzzy 非常受 Google 的 [Atheris](https://github.com/google/atheris)（一个 Python 模糊测试工具）的启发。与 Atheris 类似，Ruzzy 使用 [libFuzzer](https://llvm.org/docs/LibFuzzer.html) 进行覆盖引导和模糊测试引擎。在模糊测试 C 扩展时，Ruzzy 还支持 [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) 和 [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)。

本文将介绍我们构建 Ruzzy 的动机，提供安装和运行该工具的简要概述，并讨论一些有趣的实现细节。Ruby 爱好者们欢呼吧，Ruzzy 已经来到，揭示了一个新时代的弹性 Ruby 代码库。

* 如果你好奇，Ruzzy 简单地是 Ruby 和 fuzz 或者 fuzzer 的混成词。

## 将模糊测试引入 Ruby

Trail of Bits 测试手册提供了以下关于 [模糊测试的定义](https://appsec.guide/docs/fuzzing/)：

> 模糊测试是一种动态测试方法，将畸形或不可预测的数据输入到系统中，以检测安全问题、漏洞或系统故障。我们认为这是必不可少的工具，应当包含在您的测试套件中。

在开发高保障软件时，即使在 Ruby 中，模糊测试是一种重要的测试方法。可以考虑 AFL 的广泛 [奖杯橱窗](https://lcamtuf.coredump.cx/afl/)，rust-fuzz 的 [奖杯橱窗](https://github.com/rust-fuzz/trophy-case)，以及 [OSS-Fuzz 的声称](https://google.github.io/oss-fuzz/#trophies) ，它已经帮助发现并修复了超过 10,000 个安全漏洞和 36,000 个 bug。正如之前提到的，Python 有 Atheris。Java 有 [Jazzer](https://github.com/CodeIntelligenceTesting/jazzer/)。Ruby 社区也应当拥有一个高质量、现代化的模糊测试工具。

这并不是说以前没有构建过Ruby模糊测试工具。实际上有：[kisaten](https://github.com/twistlock/kisaten)，[afl-ruby](https://github.com/richo/afl-ruby)，[FuzzBert](https://github.com/krypt/FuzzBert)，或许还有一些我们遗漏了的工具。然而，所有这些工具似乎要么未维护，要么难以使用，要么功能不足，要么以上问题都有。为了解决这些挑战，Ruzzy基于三个原则构建：

1.  模糊测试纯Ruby代码和Ruby C扩展

1.  通过提供一个RubyGems安装过程和简单界面，使模糊测试变得简单。

1.  与广泛的libFuzzer生态系统集成

现在，让我们来试一下这个工具。

## 安装和运行Ruzzy

[Ruzzy存储库](https://github.com/trailofbits/ruzzy#ruzzy)有很好的文档，因此本文将提供一个安装和运行工具的简要版本。本文的目标是快速概述使用Ruzzy的情况。如需更多信息，请查阅存储库。

首先，Ruzzy需要Linux环境和最新版本的Clang（我们已测试到版本14.0.0）。Clang的发布版本可以在其[GitHub发布页面](https://github.com/llvm/llvm-project/releases)找到。如果您使用的是Mac或Windows计算机，则可以将Docker Desktop用作您的Linux环境，Mac上的安装方法见[这里](https://docs.docker.com/desktop/install/mac-install/)，Windows上的安装方法见[这里](https://docs.docker.com/desktop/install/windows-install/)。然后，您可以使用Ruzzy的[Docker开发环境](https://github.com/trailofbits/ruzzy#developing)来运行该工具。现在，让我们开始吧。

运行以下命令从[RubyGems](https://rubygems.org/gems/ruzzy)安装Ruzzy：

```
MAKE="make --environment-overrides V=1" \
CC="/path/to/clang" \
CXX="/path/to/clang++" \
LDSHARED="/path/to/clang -shared" \
LDSHAREDXX="/path/to/clang++ -shared" \
    gem install ruzzy

```

这些环境变量确保工具正确编译和安装。稍后将更详细地探讨它们。请确保更新`/path/to`部分，以指向您的`clang`安装位置。

## 模糊测试Ruby C扩展

为了方便测试工具，Ruzzy包含一个[“虚拟”C扩展](https://github.com/trailofbits/ruzzy/blob/main/ext/dummy/dummy.c)，其中包含一个堆使用后释放漏洞。本节将演示如何使用Ruzzy对这个易受攻击的C扩展进行模糊测试。

首先，我们需要配置Ruzzy所需的[消毒器选项](https://github.com/google/sanitizers/wiki/SanitizerCommonFlags)。

```
export ASAN_OPTIONS="allocator_may_return_null=1:detect_leaks=0:use_sigaltstack=0"
```

（请参见[Ruzzy README](https://github.com/trailofbits/ruzzy#getting-started)，了解为何在此上下文中这些选项是必要的。）

接下来，开始模糊测试：

```
LD_PRELOAD=$(ruby -e 'require "ruzzy"; print Ruzzy::ASAN_PATH') \
    ruby -e 'require "ruzzy"; Ruzzy.dummy'

```

`LD_PRELOAD`之所以需要，与[Atheris需要的原因](https://github.com/google/atheris/blob/master/native_extension_fuzzing.md#option-a-sanitizerlibfuzzer-preloads)相同。也就是说，它使用了一个特殊的共享对象，提供了对libFuzzer消毒器的访问。现在Ruzzy正在进行模糊测试，应该很快产生像下面这样的崩溃：

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

## 模糊测试纯Ruby代码

对于模糊化纯 Ruby 代码，需要两个 Ruby 脚本：一个跟踪器脚本和一个模糊化测试工具。由于 Ruby 解释器的实现细节，需要跟踪器脚本。每个跟踪器脚本看起来几乎相同。唯一的区别是您要跟踪的 Ruby 脚本的名称。

首先是跟踪器脚本。我们称之为 `test_tracer.rb`：

```
require 'ruzzy'

Ruzzy.trace('test_harness.rb')

```

接下来是模糊化测试工具。一个模糊化测试工具包裹一个模糊化目标并将其传递给模糊化引擎。在这种情况下，我们有一个简单的模糊化目标，当接收到“FUZZ”输入时会崩溃。这是一个刻意制造的示例，但它展示了 Ruzzy 发现最大化代码覆盖并产生崩溃的输入的能力。我们称这个工具为 `test_harness.rb`：

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

您可以使用以下命令开始模糊化过程：

```
LD_PRELOAD=$(ruby -e 'require "ruzzy"; print Ruzzy::ASAN_PATH') \
    ruby test_tracer.rb

```

这应该快速产生像下面这样的崩溃：

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

Ruzzy 使用 libFuzzer 的覆盖导向仪器化来发现会导致崩溃的输入（“FUZZ”）。这是 Ruzzy 的一个关键贡献之一：支持纯 Ruby 代码的覆盖支持。我们将在下一节讨论覆盖支持及更多内容。

## 有趣的实现细节

您不需要理解这一节就能使用 Ruzzy，但模糊化通常更像是艺术而不是科学，因此我们想分享一些细节来帮助揭开这种神秘的艺术。我们当然从描述 [Atheris](https://security.googleblog.com/2020/12/how-atheris-python-fuzzer-works.html) 和 [Jazzer](https://www.code-intelligence.com/blog/java-fuzzing-with-jazzer) 的博客文章中学到了很多，因此我们觉得应该继续分享。当然，创建这样一个工具涉及许多有趣的细节，但我们将专注于三个方面：创建 Ruby 模糊化测试工具，用 libFuzzer 编译 Ruby C 扩展，以及为纯 Ruby 代码添加覆盖支持。

### 创建一个 Ruby 模糊化测试工具

在开始模糊化活动时，首先需要的是一个模糊化测试工具。Trail of Bits 测试手册 [定义了模糊化测试工具](https://appsec.guide/docs/fuzzing/#terminology) 如下：

> 一个测试工具用于给定目标的测试设置。该工具包裹软件并初始化，使其准备好执行测试用例。一个测试工具集成目标到测试环境中。

当模糊化 Ruby 代码时，我们自然也希望用 Ruby 编写我们的模糊化测试工具。这与本文开头的第二个目标相符：使模糊化 Ruby 简单而容易。然而，当考虑到 libFuzzer 是用 C/C++ 编写的时，会遇到一个问题。在 [使用 libFuzzer 作为库](https://llvm.org/docs/LibFuzzer.html#using-libfuzzer-as-a-library) 时，我们需要将一个 C 函数指针传递给 `LLVMFuzzerRunDriver` 来启动模糊化过程。如何将任意的 Ruby 代码传递给 C/C++ 库呢？

使用类似 [Ruby-FFI](https://github.com/ffi/ffi) 的外部函数接口（FFI）是一种可能性。但是，通常情况下 FFI 用于反向操作：从 Ruby 调用 C/C++ 代码。[Ruby C 扩展](https://guides.rubygems.org/gems-with-extensions/) 似乎是另一种可能性，但我们仍然需要找到一种方法将任意 Ruby 代码传递给 C 扩展。在深入研究 Ruby C 扩展 API 后，我们发现了 [`rb_proc_call` 函数](https://github.com/ruby/ruby/blob/v3_3_0/proc.c#L985-L986)。这个函数使我们能够使用 Ruby C 扩展来连接 Ruby 代码和 libFuzzer C/C++ 实现之间的鸿沟。

在 Ruby 中，[`Proc`](https://ruby-doc.org/3.3.0/Proc.html) 是“一段代码块的封装，可以存储在局部变量中，传递给方法或另一个 `Proc`，并且可以被调用。” `Proc` 是 Ruby 中一个核心的函数式编程特性概念。非常完美，这正是我们需要的。在 Ruby 中，所有 lambda 函数也是 `Proc`，因此我们可以编写如下的模糊测试挂载：

```
require 'json'
require 'ruzzy'

json_target = lambda do |data|
  JSON.parse(data)
  return 0
end

Ruzzy.fuzz(json_target)
```

在这个例子中，`json_target` lambda 函数被传递给 `Ruzzy.fuzz`。在幕后，Ruzzy 使用两种语言特性来连接 Ruby 代码和 C 接口：Ruby `Proc` 和 C 函数指针。首先，Ruzzy 使用函数指针调用 `LLVMFuzzerRunDriver`。然后，每当调用该函数指针时，它会调用 `rb_proc_call` 来执行 Ruby 目标。这使得 C/C++ 模糊引擎可以重复使用模糊数据调用 Ruby 目标。考虑上述例子，由于所有 lambda 函数都是 `Proc`，这实现了从 C/C++ 库调用任意 Ruby 代码的目标。

如同所有好的高层次概述一样，这是对 Ruzzy 工作原理的简化描述。您可以在 [`cruzzy.c`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/ext/cruzzy/cruzzy.c) 中查看确切的实现。

### 使用 libFuzzer 编译 Ruby C 扩展

在继续之前，重要的是要理解我们考虑的两个 Ruby C 扩展：连接到 libFuzzer 模糊引擎的 Ruzzy C 扩展以及成为我们模糊测试目标的 Ruby C 扩展。前一节讨论了 Ruzzy C 扩展的实现。本节讨论了 Ruby C 扩展目标。这些是使用 Ruby C 扩展的第三方库，我们希望对其进行模糊测试。

要模糊测试一个 Ruby C 扩展，我们需要一种方法来使用 libFuzzer 和其相关的检查器编译扩展。为了进行模糊测试，需要在 C/C++ 代码的编译过程中添加[特殊的编译时标志](https://llvm.org/docs/LibFuzzer.html#fuzzer-usage)，因此我们需要一种方式动态地将这些标志注入到 C 扩展的编译过程中。动态添加这些标志是很重要的，因为我们希望在不修改底层代码的情况下安装和模糊测试 Ruby gems。

[`mkmf`](https://ruby-doc.org/3.3.0/stdlibs/mkmf/MakeMakefile.html)，或者 MakeMakefile 模块，是编译 Ruby C 扩展的主要接口。`gem install` 过程调用一个特定于 gem 的 Ruby 脚本，通常命名为 `extconf.rb`，该脚本再调用 `mkmf` 模块。该过程大致如下：

```
gem install -> extconf.rb -> mkmf -> Makefile -> gcc/clang/CC -> extension.so

```

不幸的是，默认情况下，`mkmf` 不会遵循常见的 C/C++ 编译环境变量如 `CC`、`CXX` 和 `CFLAGS`。但是，我们可以通过设置以下环境变量来强制此行为：`MAKE="make --environment-overrides"`。这告诉 `make` 环境变量优先于 `Makefile` 变量。有了这个设置，我们可以使用以下命令安装包含 C 扩展的 Ruby gem，并附加适当的模糊测试标志：

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

我们正在安装的 gem 是 `msgpack`，一个包含[C 扩展组件](https://github.com/msgpack/msgpack-ruby/tree/master/ext/msgpack)的示例。由于它可以反序列化二进制数据，因此非常适合作为模糊测试的目标。接下来，如果我们想对 `msgpack` 进行模糊测试，我们将创建一个 `msgpack` 模糊测试工具并启动模糊测试过程。

如果您想找更多的模糊目标，可以搜索 GitHub 上的[`extconf.rb`文件](https://github.com/search?q=path%3A**%2Fextconf.rb&type=code)，这是我们发现优秀 C 扩展候选项的最佳方式之一。

### 为纯 Ruby 代码添加覆盖支持

如果不是 Ruby C 扩展，而是想对纯 Ruby 代码进行模糊测试怎么办？也就是说，没有包含 C 扩展组件的 Ruby 项目。如果通过修改安装时功能来设置一些冗长且官方不支持的环境变量是一个巧妙的解决方案，那么接下来的内容可能就不适合胆小者了。但是，嘿，有一个能工作的解决方案，即使有些*艺术上的自由*也总比没有解决方案强。

首先，我们需要解释覆盖支持的动机。模糊测试工具通过分析覆盖信息获得一些[“智能”](https://blog.trailofbits.com/2017/02/16/the-smart-fuzzer-revolution/)。这与单元测试和集成测试提供的代码覆盖信息非常相似。在模糊测试过程中，大多数模糊测试工具优先选择可以触发新代码分支的输入。这样做可以[增加发现](https://mboehme.github.io/paper/ICSE22.pdf)崩溃和错误的可能性。对于模糊测试 Ruby C 扩展，Ruzzy 可以将 C 代码的覆盖工具化处理交给 Clang。而对于纯 Ruby 代码，我们则无法这样做。

在实现 Ruzzy 过程中，我们发现了一个极其有用的功能：Ruby [`Coverage`](https://ruby-doc.org/3.3.0/exts/coverage/Coverage.html) 模块。问题在于，它不能轻易地被 C 扩展实时调用。回想一下，Ruzzy 使用自己的 C 扩展将模糊测试代码传递给 `LLVMFuzzerRunDriver`。为了实现我们的纯 Ruby 覆盖率“智能”，我们需要在模糊引擎执行时实时传递 Ruby 覆盖信息给 libFuzzer。`Coverage` 模块非常适合在执行的起始点和结束点已知时使用，但在需要持续收集覆盖信息并传递给 libFuzzer 时并不适用。不过，我们知道 `Coverage` 模块必须以某种方式实现，因此我们深入研究了 Ruby 解释器的 C 实现以了解更多。

Ruby 事件挂钩登场。[`TracePoint`](https://ruby-doc.org/3.3.0/TracePoint.html) 模块是官方 Ruby API，用于监听特定类型的事件，例如调用函数、从例程返回、执行代码行等等。当这些事件触发时，您可以执行回调函数以任何您喜欢的方式处理事件。听起来很不错，正是我们所需的。当我们尝试跟踪覆盖信息时，我们真正想做的是监听分支事件。这正是 `Coverage` 模块正在做的，因此我们知道它必须在某个地方存在于底层。

幸运的是，公共 Ruby C API 通过 [`rb_add_event_hook2` 函数](https://github.com/ruby/ruby/blob/v3_3_0/include/ruby/debug.h#L789) 提供了访问此事件挂钩功能的途径。此函数接受要挂钩的事件列表以及每当其中一个事件触发时执行的回调函数。通过稍微查看源代码，我们发现可能事件列表与 [`TracePoint` 模块](https://ruby-doc.org/3.3.0/TracePoint.html#class-TracePoint-label-Events) 中的列表非常相似：

```
 37    #define RUBY_EVENT_NONE      0x0000 /**< No events. */
 38    #define RUBY_EVENT_LINE      0x0001 /**< Encountered a new line. */
 39    #define RUBY_EVENT_CLASS     0x0002 /**< Encountered a new class. */
 40    #define RUBY_EVENT_END       0x0004 /**< Encountered an end of a class clause. */
       ...

```

[Ruby 事件挂钩类型](https://github.com/ruby/ruby/blob/v3_3_0/include/ruby/internal/event.h#L37-L40)

如果您继续深入挖掘，您会注意到一种事件明显缺失：覆盖事件。但为什么呢？`Coverage` 模块似乎正在处理这些事件。如果您继续挖掘，您将发现实际上确实存在覆盖事件，这就是 `Coverage` 模块的工作原理，但您无法访问它们。它们被定义为 Ruby C API 的私有、内部专用部分：

```
 2182    /* #define RUBY_EVENT_RESERVED_FOR_INTERNAL_USE 0x030000 */ /* from vm_core.h */
 2183    #define RUBY_EVENT_COVERAGE_LINE                0x010000
 2184    #define RUBY_EVENT_COVERAGE_BRANCH              0x020000

```

[私有覆盖率事件挂钩类型](https://github.com/ruby/ruby/blob/v3_3_0/vm_core.h#L2182-L2184)

这是个坏消息。好消息是，我们可以自定义`RUBY_EVENT_COVERAGE_BRANCH`事件钩子并在代码中设置正确的常量值，`rb_add_event_hook2`仍然会尊重它。因此，我们仍然可以使用Ruby内置的覆盖率跟踪！我们可以实时将这些数据输入到libFuzzer中，它将相应地进行模糊测试。讨论如何将这些数据输入到libFuzzer的具体方法超出了本文的范围，但如果你想了解更多信息，我们使用SanitizerCoverage的[内联8位计数器](https://clang.llvm.org/docs/SanitizerCoverage.html#inline-8bit-counters)，[PC-Table](https://clang.llvm.org/docs/SanitizerCoverage.html#pc-table)和[数据流追踪](https://clang.llvm.org/docs/SanitizerCoverage.html#tracing-data-flow)。

还有最后一件事情。

在我们的测试过程中，即使我们添加了正确的事件钩子，我们仍然无法成功挂接覆盖事件。`Coverage`模块必须在做一些我们没有看到的事情。如果我们调用`Coverage.start(branches: true)`，根据[`Coverage`文档](https://ruby-doc.org/3.3.0/exts/coverage/Coverage.html#module-Coverage-label-Branches+Coverage)，事情就会按预期工作。这里的细节涉及在Ruby解释器源代码中进行大量侦探工作，所以我们将直接说重点。据我们所知，调用`Coverage.start`实际上调用了`Coverage.setup`，在Ruby解释器中初始化了一些全局状态，从而允许挂接覆盖事件。这种初始化功能也是私有的、仅限内部使用的API的一部分。我们能想到的最简单的解决方案是在开始模糊测试之前调用`Coverage.setup(branches: true)`。有了这个，我们开始如预期地成功挂接覆盖事件。

将覆盖事件包含在标准库中大大简化了我们的生活。如果没有这个功能，我们可能不得不采取更加侵入性和繁琐的解决方案，比如实时修改Ruby解释器看到的代码。然而，如果挂接覆盖事件是官方公开的Ruby C API的一部分，我们的生活可能会更加轻松。我们目前正在跟踪此请求：[`trailofbits/ruzzy#9`](https://github.com/trailofbits/ruzzy/issues/9)。

再次强调，这里提供的信息略微简化了实现细节；如果你想了解更多，请查看[`cruzzy.c`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/ext/cruzzy/cruzzy.c)和[`ruzzy.rb`](https://github.com/trailofbits/ruzzy/blob/v0.7.0/lib/ruzzy.rb)，这两个是很好的起点。

## 使用Ruzzy找到更多的Ruby漏洞

在构建这个工具的过程中，我们面临了一些有趣的挑战，并试图将大部分复杂性隐藏在一个简单易用的界面背后。在使用该工具时，实现细节不应成为阻碍或烦恼。然而，在这里详细讨论它们可能会推动下一个模糊测试工具的实现或模糊测试社区的进步。如前所述，[Atheris](https://security.googleblog.com/2020/12/how-atheris-python-fuzzer-works.html) 和 [Jazzer](https://www.code-intelligence.com/blog/java-fuzzing-with-jazzer) 的文章对我们产生了很大的启发，所以我们决定将这种影响传递下去。

构建工具只是开始。真正的价值在于我们开始使用工具找到漏洞。像Atheris为Python，以及之前的Jazzer为Java一样，Ruzzy试图为Ruby社区带来更高级别的软件保障。如果您使用Ruzzy找到了漏洞，请随时提交一个针对我们的[奖杯橱窗](https://github.com/trailofbits/ruzzy#trophy-case)的PR，附上问题链接。

如果您想了解更多关于我们在模糊测试方面的工作，请查阅以下文章：

[联系我们](https://www.trailofbits.com/contact/) 如果您对为您的项目定制模糊测试感兴趣。

### 像这样：

正在加载...

### *相关*
