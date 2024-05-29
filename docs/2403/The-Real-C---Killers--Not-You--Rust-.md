<!--yml
category: 未分类
date: 2024-05-29 12:30:50
-->

# The Real C++ Killers (Not You, Rust)

> 来源：[https://wordsandbuttons.online/the_real_cpp_killers.html](https://wordsandbuttons.online/the_real_cpp_killers.html)

This is [Words and Buttons Online](index.html) — a collection of interactive [#tutorials](all_tutorials.html), [#demos](all_demos.html), and [#quizzes](all_quizzes.html) about [#mathematics](all_mathematics.html), [#algorithms](all_algorithms.html) and [#programming](all_programming.html).

# The Real C++ Killers (Not You, Rust)

Hello! I’m Oleksandr Kaleniuk and I’m a C++holic. I have been writing in C++ for 17 years and for all those 17 years I’ve been struggling to get rid of this devastating addiction.

It all started in late 2005 with a 3D space simulator engine. The engine had everything C++ had in 2005\. Three-star pointers, eight layers of dependency, and C-style macros everywhere. There were assembly bits too. Iterators Stepanov-style and meta-code Alexandrescu-style. The code had everything. Except, of course, for the answer to the most important question: why?

In a while, even that question was answered. Just not as “what for” but rather as “how come”. As it turned out, the engine has been written for about 8 years by 5 different teams. And every team brought their favorite fad to the project wrapping the old code into fashionably styled wrappers but adding little value for the engine itself.

At first, I was honestly trying to grok every little thing. That was not a gratifying experience, not at all, and at some point, I gave up. I was still closing tasks, and fixing bugs. Can’t say that I was very productive, only productive enough not to get fired. But then my boss asked me: “Do you want to rewrite some of the shader code from Assembly to GLSG?” I thought god knows what this GLSL looks like but it couldn’t possibly be worse than C++ and said yes. It wasn’t worse.

And this kind of became a pattern. I was still mostly writing in C++ but every time someone asked me “Do you want to do that non-C++ thing?” I was “Sure!”. And then I did do that thing whatever it was. I wrote in C89, MASM32, C#, PHP, Delphi, ActionScript, JavaScript, Erlang, Python, Haskell, D, Rust, and even that outrageously bad InstallShield scripting language. I wrote in VisualBasic, in bash, and in a few proprietary languages, I can’t even legally talk about. I even made one myself by accident. I made a simple Lisp-style interpreter to help game designers automate resource loading, and went on a vacation. When I was back, they were writing the whole game scenes in this interpreter so we had to support it for a while.

So for the last 17 years, I was honestly trying to quit C++ but every time, after I tried a new shiny thing, I was coming back. Nevertheless, I do think that writing in C++ is a bad habit. It is unsafe, not as effective as it is thought of, and it wastes a terrible amount of a programmer’s mental capacity on things that have nothing to do with making software. Do you know that in MSVC **uint16_t(50000) * uin16_t(50000) == -1794967296**? Do you know why? Yeah, that’s what I thought.

I believe that it is the moral responsibility of long-time C++ programmers to discourage the young generation from making C++ their profession pretty much as it is the moral responsibility of alcoholics who can’t quit to warn the youth about the danger.

But why can’t I just quit then? What’s the matter? The matter is, none of the languages, especially the so-called “C++ killers” give any real advantage over C++ in the modern world. All those new languages are mostly focused on holding a programmer on a leash for their own good. This is fine, except that writing good code with bad programmers is a problem of the XX century when transistor density grew twofold every 18 months, and programmers’ headcount grew twofold every 5 years.

We’re living in the XXI century now. We have more experienced programmers in the world than ever before in history. And we need efficient software now more than ever too.

In the XX century, things were simpler. You have an idea, you wrap it into some UI and sell it as a desktop product. Is it slow? Who cares! In eighteen months desktops will become 2x faster anyway. What matters is to enter the market, to start selling features, and preferably without bugs. In that climate, sure, if a compiler keeps programmers from making bugs – good! Because bugs don’t bring in the cash, and you have to pay your programmers whether they add features or bugs anyway.

Now things are different. You have an idea, you wrap it in a Docker container and run it in a cloud. Now you get your revenue from people running your software if it makes their problems go away. Even if it does one thing but does it right, you’ll get paid. You don’t have to stuff your product with made-up features just to sell a new version of it. On the other hand, the one who pays for your code ineffectiveness is now yourself. Every suboptimal routine shows in your AWS bill.

So in the new climate, you now need fewer features, but you also need better performance for whatever features you have.

And suddenly it turns out that all the “C++ killers”, even those which I wholeheartedly love and respect like Rust, Julia, and D, do not address the problems of the XXI century. They are still stuck in the XX. They do help you write more features with fewer bugs, yes, but they are not of much help when you need to squeeze the very last FLOPS from the hardware you rent.

As such, they just have a competitive advantage over C++. Or, for that matter, even over each other. Most of them, for instance, Rust, Julia, and Cland even share the same backend. No one wins a car race if all the racers sit in the same car.

So, which technologies do give you a competitive advantage over C++ or, speaking generally, all the traditional ahead-of-time compilers?

Good question!

Glad you asked.

## C++ killer number 1\. SPIRAL

But before we go with SPIRAL itself, let’s check how well your intuition works. Which do you think is faster: a standard C++ sine function, or a 4-piece polynomial model of a sine?

|  
```
// version 1
auto y = std::sin(x);

// version 2
y = -0.000182690409228785*x*x*x*x*x*x*x
    +0.00830460224186793*x*x*x*x*x
    -0.166651012143690*x*x*x
    +x;

```

 |

Next question. What works faster, using logical operations with short-circuiting, or making the logical expression into an arithmetic one?

|  
```
// version 1	
  if (xs[i] == 1 
   && xs[i+1] == 1 
   && xs[i+2] == 1 
   && xs[i+3] == 1) // xs are bools stored as ints

// version 2
  inline int sq(int x) {
      return x*x;
  }

  if(sq(xs[i] - 1) 
   + sq(xs[i+1] - 1) 
   + sq(xs[i+2] - 1) 
   + sq(xs[i+3] - 1) == 0)

```

 |

And one more. What sorts triplets faster: a swap-sort with branching or a branchless index-sort?

|  
```
// version 1
    if(s[0] > s[1])
        swap(s[0], s[1]);
    if(s[1] > s[2])
        swap(s[1], s[2]);
    if(s[0] > s[1])
        swap(s[0], s[1]);

// version 2
    const auto a = s[0];
    const auto b = s[1];
    const auto c = s[2];
    s[int(a > b) + int(a > c)] = a;
    s[int(b >= a) + int(b > c)] = b;
    s[int(c >= a) + int(c >= b)] = c;

```

 |

If you answered all the questions decisively and without even thinking or googling, then your intuition failed you. You didn’t see the trap. None of these questions have a definite answer without context.

1\. A polynomial model is <nobr>[3 times faster](https://wordsandbuttons.online/challenge_your_performance_intuition_with_cpp_sine.html)</nobr> than the standard sine if built with clang 11 with **-O2 -march=native** and ran on Intel Core i7-9700F. But if built with NVCC with **--use-fast-math** and on GPU namely GeForce GTX 1050 Ti Mobile, the standard sine is 10 times faster than the model.

2\. Trading short-circuited logic for vectorized arithmetic makes sense on i7 too. Makes the snippet work twice as fast. But on ARMv7 with the same clang and **-O2**, the standard logic is <nobr>[25% faster](https://wordsandbuttons.online/using_logical_operators_for_logical_operations_is_good.html)</nobr> than the micro-optimization.

3\. And with index-sort vs. swap-sort, the index-sort is 3 times faster on Intel, and swap-sort is <nobr>[3 times faster](https://wordsandbuttons.online/check_if_your_performance_intuition_still_works_with_cuda.html)</nobr> on GeForce.

So the dear micro-optimizations we all love so much may both speed up our code by factor 3, and slow it down 90%. It all depends on the context. How wonderful it would be if a compiler could pick the best alternative for us so e. g. the index-sort would miraculously turn into swap-sort when we switch the build target. But a compiler can't possibly do that.

1\. Even if we allow the compiler to reimplement sine as a polynomial model, to trade precision for speed, it still doesn’t know our target precision. In C++, we can’t say that “this function is allowed to have that error”. All we have are compiler flags like “--use-fast-math” and only in the scope of a translation unit.

2\. In the second example, the compiler doesn’t know that our values are limited to either 0 or 1 and can’t possibly propose the optimization we can. We could have probably hinted at it by using a proper bool type but that would have been a completely different problem.

3\. And in the third example, the pieces of code are vastly different to be recognized as synonymous. We detailed the code too much. If it were just std::sort, this would already have given the compiler more freedom to choose the algorithm. But it wouldn’t have chosen either index-sort not swap sort since they are both inefficient on large arrays and std::sort works with a generic iterable container.

And that’s how we get to [SPIRAL](https://www.spiral.net/). It is a joint project of Carnegie Mellon University and Eidgenössische Technische Hochschule Zürich. TL&DR: signal processing experts got bored of rewriting their favorite algorithms for every new piece of hardware by hand and wrote a program that does this work for them. The program takes a high-level description of an algorithm, and a detailed description of the hardware architecture, and optimizes the code until it makes the most efficient algorithm implementation for the hardware specified.

An important distinction between Fortran and alikes, SPIRAL really solves an optimization problem in the mathematical sense. It defines run time as a target function and looks for its global optimum in the factor space of implementation variants limited by the hardware architecture. This is something compilers never actually do.

A compiler doesn’t look for the true optimum. It optimizes the code guided by the heuristics it was taught by the programmers. Essentially a compiler doesn’t work as a machine searching for the optimal solution, it rather works as an assembly programmer. A good compiler works like a good assembly programmer, but that’s it.

SPIRAL is a research project. It is limited in scope and budget. But the results it shows are already impressive. On the fast Fourier transform, their solution overperforms both MKL and FFTW implementations decisively. Their code is ~2x faster. Even on Intel.

Just to highlight the scale of achievement, MKL is the Math Kernel Library by Intel itself, so by the guys who know how to use their hardware the most. And WWTF A. K. A. “Fastest Fourier Transform in the West” is a highly specialized library from the guys who know the algorithm the best. They are both champions in what they do and the very fact that SPIRAL beats them both twofold is astonishing.

Here's their GitHub page: [https://github.com/spiral-software/spiral-software](https://github.com/spiral-software/spiral-software). If you're not convinced with the numbers from above, you can remeasure them yourself.

When the optimization technology SPIRAL uses gets finalized and commercialized, not only C++ but Rust, Julia, and even Fortran will face competition they never faced before. Why would anyone write in C++ if writing in high-level algorithm description language makes your code 2x faster?

## C++ killer number 2\. Numba

The best programming language is the one you already know well. For several decades straight, the best-known language for most programmers has been C. It also leads TIOBE index with other C-likes tightly populating the top 10\. However, only two years ago something unheard of happened. The C gave its first place to something else.

The “something else” appeared to be Python. A language nobody took seriously in the 90s because it was yet another scripting language we already had plenty of.

Someone will say: “Bah, Python is slow”, and will look like a fool since this is terminological nonsense. Just like an accordion or a frying pan, a language simply can not be fast or slow. Just like the speed of an accordion depends on who’s playing, the “speed” of a language depends on how fast its compiler is.

“But Python is not a compiled language” someone may continue, and misfire once again. There are plenty of Python compilers and the most promising of them is in its turn a Python script. Let me explain.

I once had a project. A 3D-printing simulation that was originally written in Python and then rewritten in C++ “for performance”, and then ported to GPU, all of that before I came in. I then spent months porting the build to Linux, optimizing the GPU code for Tesla M60 since it was the cheapest in AWS at the point, and validating all the changes in the C++/CU code to go along with the original code in Python. So I did everything except the things I normally specialize in namely devising geometric algorithms.

And when I finally had everything working, a student from Bremen a part-timer called me and asked: “So you’re good at heterogeneous stuff, can you help me run one algorithm on GPU?” Of course! I told him about CUDA, CMake, Linux build, testing, and optimizing; spent maybe an hour talking. He listened to all of that very politely, but at the end said: “This is all very interesting, but I have a very specific question. So I have a function, I wrote @cuda.jit before its definition, and Python says something about arrays and doesn’t compile the kernel. Do you know what could be the problem here?”

I didn’t know. He figured it out himself in a day. Apparently, Numba doesn’t work with native Python lists it only accepts data in NumPy arrays. So he figured it out and ran his algorithm on GPU. In Python. He had none of the problems I spent months on. Do you want it on Linux? Not a problem, just run it on Linux. Do you want it to be consistent with the Python code? Not a problem, it is Python code. Do you want to optimize for the target platform? Not a problem again. Numba will optimize the code for the platform you run the code on since it doesn’t compile ahead of time, it compiles on demand when already deployed.

Isn’t that awesome? Well, no. Not for me anyway. I spent months with C++ solving problems that never occur in Numba, and a part-timer from Bremen made the same thing in a few days. It could have been a few hours if it wasn’t his first experience with Numba. So what is this Numba? What kind of sorcery it is?

No sorcery. Python decorators turn every piece of code into its abstract syntax tree for you, so you can then do whatever you want with it. [Numba](https://numba.pydata.org) is a Python library that wants to compile abstract syntax trees with any backend it has and for any platform it supports. If you want to compile your Python code to run on CPU cores in a massively parallel fashion – just tell Numba to compile it so. If you want to run something on GPU, again, [you should only ask](https://numba.readthedocs.io/en/stable/cuda/kernels.html).

|  
```
@cuda.jit
def matmul(A, B, C):
    """Perform square matrix multiplication of C = A * B."""
    i, j = cuda.grid(2)
    if i < C.shape[0] and j < C.shape[1]:
        tmp = 0.
        for k in range(A.shape[1]):
                tmp += A[i, k] * B[k, j]
        C[i, j] = tmp

```

 |

Numba is one of the Python compilers that makes C++ obsolete. In theory, however, it’s not any better than C++ since it uses the same backends. It uses CUDA for GPU programming and LLVM for CPU. In practice, since it doesn’t require an ahead-of-time rebuild for every new architecture, Numba solutions adapt better to every new hardware and its available optimizations.

Of course, it would be better to have a clear performance advantage as is with SPIRAL. But SPIRAL is more of a research project, it might kill C++ but only eventually, and only if it’s lucky. Numba with Python strangles C++ right now, in real time. Because if you can write in Python and have the performance of C++, why would you want to write in C++?

## C++ killer number 3\. ForwardCom

Let’s play another game. I’ll give you three pieces of code, and you’ll guess which one of them, or maybe more, is written in assembly. Here they are:

|  
```
    invoke RegisterClassEx, addr wc     ; register our window class
    invoke CreateWindowEx,NULL,
        ADDR ClassName, ADDR AppName,\
        WS_OVERLAPPEDWINDOW,\
        CW_USEDEFAULT, CW_USEDEFAULT,\
        CW_USEDEFAULT, CW_USEDEFAULT,\
        NULL, NULL, hInst, NULL
        mov   hwnd,eax
    invoke ShowWindow, hwnd,CmdShow     ; display our window on desktop
    invoke UpdateWindow, hwnd           ; refresh the client area

    .while TRUE                         ; Enter message loop
        invoke GetMessage, ADDR msg,NULL,0,0
        .break .if (!eax)
        invoke TranslateMessage, ADDR msg
        invoke DispatchMessage, ADDR msg
   .endw

```

 |

|  
```
(module
  (func $add (param $lhs i32) (param $rhs i32) (result i32)
        get_local $lhs
        get_local $rhs
        i32.add)
  (export "add" (func $add)))

```

 |

|  
```
v0 = my_vector  // we want the horizontal sum of this
int64 r0 = get_len ( v0 )
int64 r0 = round_u2 ( r0 )
float v0 = set_len ( r0 , v0 )
while ( uint64 r0 > 4) {
        uint64 r0 >>= 1
        float v1 = shift_reduce ( r0 , v0 )
        float v0 = v1 + v0
}

```

 |

If you guessed that all three examples are assembly, congratulations! your intuition has gotten much better already!

The first one is in [MASM32](http://www.masm32.com/). It’s a macroassembler with “if”s and “while”s people write native Windows applications in. That’s right, not “used to write” but “write” to this day. Microsoft zealously protects Windows’ backward compatibility with Win32 API so all the MASM32 programs ever written work well on modern PCs too.

What is ironic, C was invented to make UNIX translation from PDP-7 to PDP-11 easier. It was designed as a portable assembler capable of surviving the Cambrian explosion of the 70s hardware architectures. But in the XXI century, the hardware architecture evolves so sluggishly, that the programs that I wrote in MASM32 20 years ago assemble and run perfectly today, but I have no confidence that a C++ application I built last year with CMake 3.21 will build today with CMake 3.25.

The second piece of code is [WebAssembly](https://github.com/mdn/webassembly-examples/blob/main/understanding-text-format/add.wat). It’s not even a macro assembler, it has no “if”s and “while”s, it is more of a human-readable machine code for your browser. Or some other browser. Conceptually, any browser.

Web Assembly code doesn’t depend on your hardware architecture at all. The machine it serves is abstract, virtual, universal, call it whatever you want. If you can read this text, you have one on your physical machine already.

But the most interesting piece of code is the third one. It’s ForwardCom – an assembler Agner Fog, a renowned author of C++ and assembly optimization manuals, proposes. Just as with Web Assembly, the proposition covers not so much an assembler as the universal set of instructions designed to enable not only backward but forward compatibility. Hence the name. ForwardCom’s full name is “[an open forward-compatible instruction set architecture.](https://www.forwardcom.info)” In other words, it’s not so much a proposition of assembly, as a peace treaty proposal.

We know that all the most common architectural families: x64, ARM, and RISC-V have different instruction sets. But no one knows a good reason to keep it this way. All the modern processors, apart from maybe the simplest, run not the code you feed it with but the microcode they translate your input into. So it’s not only M1 that has a backward compatibility layer for Intel, every processor essentially has a backward compatibility layer for all its own earlier versions.

So what prevents architecture designers from agreeing on a similar layer but for forward compatibility? Apart from the conflicting ambitions of companies being in direct competition, nothing. But if the processor makers will at some point settle to have a common instruction set rather than implement a new compatibility layer per every other competitor, ForwardCom will take assembly programming back to the mainstream. This forward compatibility layer would heal the worst neurosis of every assembly programmer out there: “What if I write the one-in-a-lifetime code for this particular architecture, and this particular architecture will make itself obsolete in a year?”

With a forward compatibility layer, it will never make itself obsolete. That’s the point.

Assembly programming is also held back by a myth that writing in assembly is hard and therefore impractical. Fog's proposition addresses this problem as well. If people think that writing in assembly is hard, and writing in C isn’t, well, let’s just make the assembler look like C. Not a problem. There is no good reason for a modern assembly language to look exactly the same as its grandfather looked in the 50s.

You just saw three assembly samples yourself. None of them looks like a “traditional” assembly and none should be.

So, ForwardCom is the assembly in which you can write optimal code that will never go obsolete, and which doesn’t make you learn a “traditional” assembly. For all practical considerations, it is the C of the future. Not C++.

## So when will С++ finally die?

We live in a postmodern world. Nothing dies anymore but people. Just as Latin never actually died, just like COBOL, Algol 68, and Ada, – C++ is doomed to eternal half-existence between life and death. C++ will never actually die, it will only be pushed out of the mainstream by newer more potent technologies.

Well, not “will be pushed” but “being pushed”. I came to my current job as a C++ programmer, and today my workday starts with Python. I write the equations, SymPy solves them for me, and then translates the solution into C++. I then paste this code into the C++ library not even bothering to format it a little, since clang-tidy will do that for me anyway. A static analyzer will check that I didn’t mess up the namespaces, and a dynamic analyzer will check for memory leaks. CI/CD will take care of cross-platform compilation. A profiler will help me understand how my code actually works, and disassembler – why.

If I trade C++ for “not C++”, 80% of my work will remain exactly the same. C++ is simply irrelevant to most of what I do. Could it mean that for me C++ is already 80% dead?