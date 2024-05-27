<!--yml
category: 未分类
date: 2024-05-27 12:56:28
-->

# Dr George W Stagg - Fortran on WebAssembly

> 来源：[https://gws.phd/posts/fortran_wasm/](https://gws.phd/posts/fortran_wasm/)

# Introduction

[Fortran](https://fortran-lang.org) is one of the oldest programming languages around. It first appeared in 1957, making it older than the C programming language, the Intel 4004 CPU, and even the IBM System/360 series of mainframe computers. Fortran was created at a time when the byte had just been invented, and computers were still made of vacuum tubes and frustration.

¹ The name is derived from *Formula Translator*. Fortran was originally stylised in all-caps, but modern Fortran has dropped it.

² The System/360 was released around 1965\. The 4004 around 1970\. And [K&R C](https://en.wikipedia.org/wiki/The_C_Programming_Language) was published in 1978.

³ The argument goes that Fortran’s lack of aliasing and use of native arrays rather than pointer arithmetic allow the optimiser to generate more efficient output than an equivalent C program. However, there are also counter-arguments against this.

Over the years, Fortran has formed a rich history of use for computationally intensive scientific and engineering applications. It has powered the fluid dynamics of weather prediction and climate models, provided the condensed matter simulations for my [PhD](../../docs/thesis_gws.pdf), and is still considered by some to be more efficient than C for numerically heavy work. The syntax of modern Fortran is also surprisingly easy to get up and running with. This is not your parent’s Fortran 77 code; most restrictions that make fixed‑form Fortran awful to use are no longer in place in modern Fortran.

In a clash of computational eras, this blog post is about compiling existing Fortran code for WebAssembly so that it can run in a web browser. I’ll describe the method we currently use for the [webR](https://github.com/r-wasm/webr) project, compiling Fortran code using a patched version of [LLVM](https://llvm.org)’s `flang-new` compiler. The post also serves as a request for help. The method I describe unfortunately relies on a hack, and this hack means that I cannot contribute the changes back to LLVM without assistance from a more experienced compiler developer.

⁴ “LLVM” is not an acronym, it is the full name of the project.

# So, what’s the problem?

There are a surprising number of potential methods and toolchains available to compile Fortran to WebAssembly. Unfortunately, none of the available options are feature complete. Each method has its drawbacks, and none are a simple plug-and-play solution.

Back in 2020, the situation was summarised wonderfully by Christoph Honal’s article [FORTRAN in the Browser](https://chrz.de/2020/04/21/fortran-in-the-browser/). Even now, the article is worth reading and provides a nice background for this post. I personally owe a lot to Christoph’s article, particularly for its description of the [Dragonegg](https://dragonegg.llvm.org) toolchain. Without that article, I would have given up on Fortran for WebAssembly a long time ago.

Our goal by the end of this post is to be able to compile a modern Fortran routine to WebAssembly that takes in some numerical arguments, computes the output of [BLAS](http://www.netlib.org/blas/) and [LAPACK](https://netlib.org/lapack/) routines, and either returns the result or prints it to console. From what I remember, in 2020 none of the methods outlined in Christoph’s article could do this satisfactorily. Dragonegg and `f2c` both get close but have some drawbacks, as I’ll describe below.

⁵ LAPACK (Linear Algebra Package) is a popular library of routines used to numerically solve problems in linear algebra. It’s written in Fortran 90 and itself relies on a BLAS (Basic Linear Algebra Subprograms) library.

⁶ via [Pyodide](https://pyodide.org/en/stable/) and [webR](https://github.com/r-wasm/webr).

Together, BLAS and LAPACK routines provide a powerful numerical platform. Running them in the browser opens the door for several higher-level programming environments that rely on them under the hood, such as SciPy or R. The beauty of this approach is that it allows you to bring existing and extensively battle-tested tools and libraries to the web without having to rewrite them all in Rust or JavaScript.

Later, I’ll show an example of this with a machine learning demo that directly uses BLAS routines compiled from Fortran to WebAssembly. Rather than having to write fiddly linear algebra numerical algorithms in JavaScript, we can use reliable and efficient BLAS routines directly.

⁷ Whilst running machine learning algorithms in a web browser will never be as efficient as using dedicated hardware, such as a GPU, I still think it’s a fun demo.

## Compiler round-up

Since [FORTRAN in the Browser](https://chrz.de/2020/04/21/fortran-in-the-browser/) was published things have changed a little, particularly when it comes to the LLVM-based Fortran compilers. As far as I am aware, here’s a brief round-up of the current situation in 2024.

### The `f2c` utility

The [`f2c`](https://en.wikipedia.org/wiki/F2c) program converts Fortran 77 to C code, which Emscripten can then compile into WebAssembly. This is the method that the [Pyodide](https://pyodide.org/en/stable/) project uses to compile Python packages containing Fortran code. They say that this [“does not work very well”](https://pyodide.org/en/0.25.0/project/roadmap.html#find-a-better-way-to-compile-fortran). The tool doesn’t work with modern Fortran code, and even after conversion the result still throws fatal errors and requires extensive patching.

### LFortran

The LFortran compiler has made great strides over the last few years. In 2020, it was [missing a lot of features](https://gitlab.com/lfortran/lfortran/-/issues/121) and only supported a very small subset of Fortran. Now it now supports a much wider range of language features and can be used to compile a reasonable amount of Fortran code. It can even compile to WebAssembly out of the box!

⁸ Check out the LFortran demo at [https://dev.lfortran.org](https://dev.lfortran.org). While extremely impressive, note that the first thing I tried was changing `x ** 2` to `x ** 3` and saw that such a change is currently not supported by the code generator.

However, there are still some barriers that make using LFortran a little rough. The project is currently considered to be in alpha phase and the developers state that issues compiling real-world code are expected. While it can successfully compile some projects, such as [MINPACK](https://github.com/fortran-lang/minpack), the full Fortran specification is not yet supported and so many larger projects still cannot be compiled.

The LFortran developers are targeting full support for Fortran 2018, and its standout feature is an interactive Jupyter-like Fortran REPL. With a few more years of development, I expect that LFortran will be an excellent choice for compiling Fortran code for WebAssembly.

### Dragonegg

[Dragonegg](https://dragonegg.llvm.org) is a plugin for GCC that uses the GNU compilers as a frontend and emits LLVM IR. With this, LLVM can be used as the backend to produce WebAssembly output. The technique works, and it was the original method that I used to compile Fortran sources for the [webR](https://github.com/r-wasm/webr) project.

However, there are some pretty serious drawbacks to this approach. Dragonegg requires a very old version of GCC and LLVM. For most users, this means setting up a virtual machine or Docker container to provide the necessary environment. The LLVM IR emitted by Dragonegg also needs some fairly nasty post-processing before LLVM can produce WebAssembly output. Take a look at the [script originally used by webR](https://github.com/r-wasm/webr/blob/v0.1.0/tools/dragonegg/emfc.in) to get an idea of the extra processing required.

⁹ The latest supported versions are `gcc-4.8` and `llvm-3.3`

Nevertheless, in 2020 this was the only real way to compile Fortran code for WebAssembly.

### Classic flang

[“Classic” Flang](https://github.com/flang-compiler/flang) is another Fortran compiler targeting LLVM, based on an open-sourced PGI/NVIDIA compiler `pgfortran`. Classic Flang never supported 32-bit output, so it is not an option for us since we’ll be using `wasm32` for our target architecture. This will likely be the case until browser support for 64-bit Wasm memory has improved.

^(10) Previously, Flang or Flang-7.

^(11) At the time of writing Firefox, Chrome and Node supports `wasm64`, but locked behind a feature flag.

Even so, the project documentation itself suggests that choosing to use Classic Flang for a new project today is probably not a great idea:

> Classic Flang […] continues to be maintained, but the plan is to replace Classic Flang with the new Flang in the future.

### LLVM Flang

[“LLVM Flang”](https://flang.llvm.org/docs/) is a full ground-up reimplementation of a Fortran frontend for LLVM. It was designed to replace Classic Flang, developed by much of the same team, and was accepted as part of the LLVM project as of LLVM 11\. As such, the Flang sources can now be found in [the official LLVM source tree](https://github.com/llvm/llvm-project/tree/main/flang).

^(12) Also known as Flang, new Flang, or `flang-new`. Previously, F18.

Flang is not yet considered to be ready for production use, but its development is extremely active right now and pre-production versions of the `flang-new` compiler have been made available by the team. In recent years, the compiler has become very usable for compiling real-world Fortran code.

Currently, LLVM Flang cannot generate WebAssembly output out of the box. Despite this, we’ll soon see that with LLVM’s modular design it’s possible to use the Flang frontend with LLVM’s WebAssembly backend. With this, we can take advantage of all the development work put into the Flang frontend by the NVIDIA and PGI teams for our own purposes of compiling Fortran to WebAssembly.

This was also possible back in 2020, though it required larger patches to LLVM, injecting custom maths routines, and a multi-step compilation process. Now, due to the impressive development efforts in the `flang-new` frontend, creating a Fortran to WebAssembly compiler is possible with just a few small changes to LLVM’s source code.

# Building and using LLVM Flang for WebAssembly

If one were interested in trying out LLVM Flang, they might grab an LLVM release using their package manager of choice. However, following that route will disappoint us, as a `flang-new` binary is not included.

^(13) At least, not with LLVM v17.0.6 for macOS using `brew`.

[~/fortran]brew install llvm

==> Downloading https://formulae.brew.sh/api/formula.jws.json

################################################################################ 100.0%

==> Downloading https://ghcr.io/v2/homebrew/core/llvm/manifests/17.0.6_1

################################################################################ 100.0%

==> Fetching llvm

==> Downloading https://ghcr.io/v2/homebrew/core/llvm/blobs/sha256:8d739bdfa4152

################################################################################ 100.0%

==> Installing llvm

==> Pouring llvm--17.0.6_1.arm64_sonoma.bottle.tar.gz

==> Summary

🍺 /opt/homebrew/Cellar/llvm/17.0.6_1: 7,207 files, 1.7GB

==> Checking for dependents of upgraded formulae...

==> No broken dependents to reinstall!

[~/fortran]flang-new

zsh: command not found: flang-new

Since we will be modifying the LLVM Flang source in any case, we’ll have to compile from scratch. Let’s grab the LLVM v18.1.1 sources and start there instead. Feel free to follow along at home; I’ll try to provide all the commands and everything you need.

^(14) I’m going to assume you’re familiar with [Emscripten](https://emscripten.org) and have a version of that toolchain on your path. If not, and you want to play along, start with [emsdk](https://github.com/emscripten-core/emsdk) to setup Emscripten on your machine, get comfortable with compiling C code for WebAssembly, then return here to continue.

[~/fortran]git clone --depth=1 --branch=llvmorg-18.1.1 https://github.com/llvm/llvm-project.git

Cloning into 'llvm-project'...

remote: Enumerating objects: 138937, done.

Receiving objects: 100% (138937/138937), 199.81 MiB | 11.36 MiB/s, done.

Note: switching to '6009708b4367171ccdbf4b5905cb6a803753fe18'.

Updating files: 100% (132077/132077), done.

[~/fortran]cmake -G Ninja -S llvm-project/llvm -B build \

-DCMAKE_INSTALL_PREFIX=llvm-18.1.1 \

-DCMAKE_BUILD_TYPE=MinSizeRel \

-DLLVM_DEFAULT_TARGET_TRIPLE="wasm32-unknown-emscripten" \

-DLLVM_TARGETS_TO_BUILD="WebAssembly" \

-DLLVM_ENABLE_PROJECTS="clang;flang;mlir"

-- The C compiler identification is AppleClang 15.0.0.15000100

-- Found assembler: /Library/Developer/CommandLineTools/usr/bin/cc

-- Detecting C compiler ABI info - done

-- Performing Test HAVE_POSIX_REGEX -- success

-- Configuring done (29.0s)

-- Generating done (2.9s)

-- Build files have been written to: fortran/build

[~/fortran]cmake --build build

...

[6136/6136] Linking CXX executable bin/obj2yaml

Grab a cuppa, this step uses a lot of resources and can take a **very** long time.

^(15) **cuppa** (/ˈkʌp.ə/): noun, informal, UK. A hot drink, usually tea or coffee.

## Interlude — Calling Fortran subroutines from C

While we wait for LLVM to build, start up a new terminal and we’ll remind ourselves how to compile and link a Fortran subroutine as part of a C program. The principles here will help us later when it comes to calling Fortran from JavaScript.

First, let’s write a simple subroutine that takes in three integer arguments: `x`, `y`, and `z`. It will set the value of `z` to the sum of `x` and `y`. Name our new subroutine `foo` and save the file containing your subroutine as `foo.f08`.

```
SUBROUTINE foo(x, y, z)
 IMPLICIT NONE
 INTEGER, INTENT(IN)  :: x, y
 INTEGER, INTENT(OUT) :: z
 z = x + y
END
```

 *Notice how, generally, Fortran routines pass arguments by reference and we can declare how an argument will be used in the subroutine using `INTENT()`. Assuming you already have a traditional Fortran compiler like `gfortran` installed, compile the Fortran source into an object file.

^(16) You don’t *need* a native fortran compiler to follow along with the rest of the post, but if you’d like one you can get `gfortran` from your OS’s usual package manager as part of the GCC compiler suite. There’s also [`ifort`](https://www.intel.com/content/www/us/en/developer/tools/oneapi/fortran-compiler.html), if you’re on an Intel CPU.

[~/fortran]gfortran -c foo.f08 -o foo.o

[~/fortran]file foo.o

foo.o: Mach-O 64-bit object arm64

[~/fortran]nm foo.o

0000000000000038 s EH_frame1

0000000000000000 T _foo_

0000000000000000 t ltmp0

0000000000000038 s ltmp1

I’m on an M1 macOS machine, so my resulting object is a Mach object for ARM. If you’re a Linux user, you should see something like `ELF 64-bit LSB shared object, x86-64`. I’ve also run `nm` to take a look at the names of the symbols in the object that the compiler has built. Keep an eye on the symbol for our subroutine — on my machine it’s named `_foo_`. The leading underscore is fairly standard, but the trailing underscore differs from what is usual for C procedures.

Let’s write a C program that calls our Fortran subroutine. Notice again how we pass the arguments by reference to the external symbol. Also, if your Fortran compiler added the trailing underscore, we’ll need to include it when we declare the symbol name in C.

^(17) Modern Fortran standards provide a Fortran module [`iso_c_binding`](https://fortranwiki.org/fortran/show/iso_c_binding) and a C header file `ISO_Fortran_binding.h` to improve C interoperability, but our code is going to be simple enough that we can do without those today.

```
#include <stdio.h>

extern void foo_(int*, int*, int*);

int main() {
 int x = 1, y = 1, z;
 foo_(&x, &y, &z);

 printf("%d + %d = %d\n", x, y, z);
 return 0;
}
```

 *Compile the C source using `gcc` or equivalent, and then run the resulting binary to observe a truly staggering level of numerical computation.

[~/fortran]gcc main.c foo.o -o main

[~/fortran]./main

1 + 1 = 2**  **## Returning to LLVM Flang

Once LLVM has finished compiling, the `flang-new` binary should be available in the directory `build/bin`. We can now run it and confirm that it has been set up to produce binaries for `wasm32` and Emscripten.

[~/fortran]./build/bin/flang-new --version

flang-new version 18.1.1 (https://github.com/llvm/llvm-project.git dba2a75e9c7ef81fe84774ba5eee5e67e01d801a)

Target: wasm32-unknown-emscripten

Thread model: posix

InstalledDir: .../fortran/build/bin

Great! Let’s try compiling our Fortran subroutine using our freshly built compiler.

[~/fortran]./build/bin/flang-new -c foo.f08 -o foo.o

error: fortran/llvm-project/flang/lib/Optimizer/CodeGen/Target.cpp:1162:

not yet implemented: target not implemented

LLVM ERROR: aborting

Ah, not so great. The `wasm32-unknown-emscripten` target triple unfortunately hasn’t been implemented yet in the `flang-new` compiler.

And so here comes our first patch to LLVM. We will implement the target by extending Flang’s list of known target specifics. The required changes, shown below as a diff, can be mostly deduced by looking at the other targets implemented in the file `flang/lib/Optimizer/CodeGen/Target.cpp`.

```
diff --git a/flang/lib/Optimizer/CodeGen/Target.cpp b/flang/lib/Optimizer/CodeGen/Target.cpp
index 83e7fa9b440b..49e73ec48e0a 100644
--- a/flang/lib/Optimizer/CodeGen/Target.cpp
+++ b/flang/lib/Optimizer/CodeGen/Target.cpp
@@ -1109,6 +1109,44 @@ struct TargetLoongArch64 : public GenericTarget<TargetLoongArch64> {
 };
 } // namespace

+//===----------------------------------------------------------------------===//
+// WebAssembly (wasm32) target specifics.
+//===----------------------------------------------------------------------===//
+
+namespace {
+struct TargetWasm32 : public GenericTarget<TargetWasm32> {
+  using GenericTarget::GenericTarget;
+
+  static constexpr int defaultWidth = 32;
+
+  CodeGenSpecifics::Marshalling
+  complexArgumentType(mlir::Location, mlir::Type eleTy) const override {
+    assert(fir::isa_real(eleTy));
+    CodeGenSpecifics::Marshalling marshal;
+    // Use a type that will be translated into LLVM as:
+    // { t, t }   struct of 2 eleTy, byval, align 4
+    auto structTy =
+        mlir::TupleType::get(eleTy.getContext(), mlir::TypeRange{eleTy, eleTy});
+    marshal.emplace_back(fir::ReferenceType::get(structTy),
+                         AT{/*alignment=*/4, /*byval=*/true});
+    return marshal;
+  }
+
+  CodeGenSpecifics::Marshalling
+  complexReturnType(mlir::Location loc, mlir::Type eleTy) const override {
+    assert(fir::isa_real(eleTy));
+    CodeGenSpecifics::Marshalling marshal;
+    // Use a type that will be translated into LLVM as:
+    // { t, t }   struct of 2 eleTy, sret, align 4
+    auto structTy = mlir::TupleType::get(eleTy.getContext(),
+                                          mlir::TypeRange{eleTy, eleTy});
+    marshal.emplace_back(fir::ReferenceType::get(structTy),
+                          AT{/*alignment=*/4, /*byval=*/false, /*sret=*/true});
+    return marshal;
+  }
+};
+} // namespace
+
 // Instantiate the overloaded target instance based on the triple value.
 // TODO: Add other targets to this file as needed.
 std::unique_ptr<fir::CodeGenSpecifics>
@@ -1158,6 +1196,9 @@ fir::CodeGenSpecifics::get(mlir::MLIRContext *ctx, llvm::Triple &&trp,
 case llvm::Triple::ArchType::loongarch64:
 return std::make_unique<TargetLoongArch64>(ctx, std::move(trp),
 std::move(kindMap), dl);
+  case llvm::Triple::ArchType::wasm32:
+    return std::make_unique<TargetWasm32>(ctx, std::move(trp),
+                                               std::move(kindMap), dl);
 }
 TODO(mlir::UnknownLoc::get(ctx), "target not implemented");
 }
```

 *Save the contents of the above diff as the file `add-wasm32-target.diff`, and then apply it to the `llvm-project` directory using `git` or the `patch` utility. Then, rebuild LLVM Flang. It should be quicker to build the second time, as most generated objects are unaffected by the change.

[~/fortran]patch -p1 -d llvm-project < add-wasm32-target.diff

[~/fortran]cmake --build build

...

[180/180] Generating ../../../../include/flang/ieee_arithmetic.mod

Once LLVM has been recompiled, try compiling our Fortran source once again.

[~/fortran]./build/bin/flang-new -c foo.f08 -o foo.o

[~/fortran]file foo.o

foo.o: WebAssembly (wasm) binary module version 0x1 (MVP)

[~/fortran]llvm-nm foo.o

00000001 T foo_

Success! We can confirm this is a real WebAssembly object using the `file` utility, and `llvm-nm` can see the `foo` symbol within, corresponding to our Fortran subroutine.

^(18) You might need to use a WebAssembly aware version of this tool from Emscripten. If you’re using [emsdk](https://github.com/emscripten-core/emsdk), ensure that `.../emsdk/upstream/bin/` is on your `$PATH`.

^(19) Here I’m using Node v18, but I think anything newer than Node v16 should work. Emscripten is bundled with a version of Node, but I like using [nvm](https://github.com/nvm-sh/nvm) to manage my Node installations.

Let’s continue compiling our C function for WebAssembly using Emscripten and running it using Node. We should see the same output as with our native binary.

[~/fortran]emcc main.c foo.o -o main.js

[~/fortran]node main.js

1 + 1 = 2*  *## Interlude — Calling a Fortran routine from JavaScript

In the previous section we used a C program to call Fortran code, but we don’t technically need to do that. If we tell Emscripten about the Fortran subroutine, we can call it directly from JavaScript without writing any C code.

First, let’s link our Fortran object with Emscripten, producing a script that loads our WebAssembly binary into memory but does not execute any routines. In addition to our symbol `_foo_`, we’ll also export `_malloc` and `_free` so that we can use them from JavaScript.

^(20) See the [Emscripten documentation](https://emscripten.org/docs/tools_reference/settings_reference.html) for more details about `emcc` command line options. By the way, if you’ve not used Emscripten much before you might see extra `cache:INFO` lines emitted during various steps in this post. They are nothing to worry about and can be ignored.

[~/fortran]emcc foo.o -sEXPORTED_FUNCTIONS=_foo_,_malloc,_free -o foo.js

cache:INFO: generating system asset: symbol_lists/ae47d07bfa3321ac4dde96bd87821b4aa93f9a19.json... (this will be cached in ".../emsdk/upstream/emscripten/cache/symbol_lists/ae47d07bfa3321ac4dde96bd87821b4aa93f9a19.json" for subsequent builds)

cache:INFO: - ok

[~/fortran]node foo.js

[~/fortran]

Notice that when we run the script `foo.js` directly… nothing happens.

Next, we’ll write a JavaScript file that loads `foo.js` and then calls our Fortran subroutine. We’ll need to allocate some memory to hold our integers `x`, `y` and `z` using the exported `_malloc()` function. We’ll also need to set our input arguments `x` and `y` to some integer values, and we can do that by setting values in the allocated WebAssembly memory through `Module.HEAPU32`.

```
var Module = require('./foo.js');

setTimeout(() => {
 const x = Module._malloc(4);
 const y = Module._malloc(4);
 const z = Module._malloc(4);
 Module.HEAPU32[x / 4] = 123;
 Module.HEAPU32[y / 4] = 456;

 Module._foo_(x, y, z);

 console.log("x = ", Module.HEAP32[x / 4]);
 console.log("y = ", Module.HEAP32[y / 4]);
 console.log("x + y = ", Module.HEAP32[z / 4]);

 Module._free(x);
 Module._free(y);
 Module._free(z);
}, 100);
```

 *[~/fortran]node standalone.js

x = 123

y = 456

x + y = 579

You should also be able to run the resulting WebAssembly binary in a web browser. Remove the line `var Module = require('./foo.js');` from `standalone.js`, and instead load the script `foo.js` in your HTML.

```
<html>
 <head>
 <title>Fortran Demo</title>
 </head>
 <body>
 <script src="foo.js"></script>
 <script src="standalone.js"></script>
 </body>
</html>
```

 *Spin up a local web server, visit the page, and the same output should be seen in the browser’s JavaScript console.

^(22) Something like `Rscript -e 'httpuv::runStaticServer()'` or `python3 -m http.server` should work well.**  **## The Fortran runtime library: A journey to “Hello, World!”

The ubiquitous “Hello, World!” test program is the usual way to introduce a programming language, but I didn’t introduce Fortran using such a program above. As you’ll see, that was for a good reason. Let’s see what happens when we try to build a “Hello, World!” subroutine in Fortran and call it from C. As before, we’ll compile the Fortran object using `flang-new` and use Emscripten to compile and link the C code.

```
SUBROUTINE hello()
 IMPLICIT NONE
 PRINT *, "Hello, World!"
END
```

 *```
extern void hello_();

int main() {
 hello_();
 return 0;
}
```

 *[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o -o hello.js

wasm-ld: error: hello.o: undefined symbol: _FortranAioBeginExternalListOutput

wasm-ld: error: hello.o: undefined symbol: _FortranAioOutputAscii

wasm-ld: error: hello.o: undefined symbol: _FortranAioEndIoStatement

emcc: error: 'wasm-ld -o hello.wasm [...] --no-entry --stack-first --table-base=1' failed (returned 1)

The build failed due to some missing symbols. This is a consequence of a more general issue in that we have not yet compiled the LLVM Fortran runtime library for WebAssembly. There are a bunch of library symbols that we’re currently missing, including some functions that are required to print output!

Luckily, the runtime library is written in C++ as part of the LLVM source tree at `llvm-project/flang/runtime`. So, in principle, all we need to do is build the library using Emscripten’s `em++` compiler and then link to it whenever we’re using Fortran code in our WebAssembly program.

Here is a `Makefile` designed to make this step easy. Save it in the current directory and then run `make`. It should go ahead and use the version of Emscripten on your path to build a static Fortran runtime library at `build/flang/runtime/libFortranRuntime.a`.

^(24) Be sure to indent the rules in this file using tabs, not spaces.

```
ROOT =  $(abspath .)
SOURCE =  $(ROOT)/llvm-project
BUILD =  $(ROOT)/build

RUNTIME_SOURCES :=  $(wildcard  $(SOURCE)/flang/runtime/*.cpp)
RUNTIME_SOURCES +=  $(SOURCE)/flang/lib/Decimal/decimal-to-binary.cpp
RUNTIME_SOURCES +=  $(SOURCE)/flang/lib/Decimal/binary-to-decimal.cpp
RUNTIME_OBJECTS =  $(patsubst  $(SOURCE)/%,$(BUILD)/%,$(RUNTIME_SOURCES:.cpp=.o))

RUNTIME_CXXFLAGS += -I$(BUILD)/include -I$(BUILD)/tools/flang/runtime
RUNTIME_CXXFLAGS += -I$(SOURCE)/flang/include -I$(SOURCE)/llvm/include
RUNTIME_CXXFLAGS += -DFLANG_LITTLE_ENDIAN
RUNTIME_CXXFLAGS += -fPIC -Wno-c++11-narrowing -fvisibility=hidden
RUNTIME_CXXFLAGS += -DFE_UNDERFLOW=0 -DFE_OVERFLOW=0 -DFE_INEXACT=0
RUNTIME_CXXFLAGS += -DFE_INVALID=0 -DFE_DIVBYZERO=0 -DFE_ALL_EXCEPT=0

$(BUILD)/flang/runtime/libFortranRuntime.a:  $(RUNTIME_OBJECTS)
 @rm -f $@
 emar -rcs $@ $^

$(BUILD)%.o :  $(SOURCE)%.cpp
 @mkdir -p $(@D)
 em++ $(RUNTIME_CXXFLAGS) -o $@ -c $<

.PHONY: clean
clean:
 @rm $(RUNTIME_OBJECTS)  $(BUILD)/flang/runtime/libFortranRuntime.a
```

 *[~/fortran]make

em++ .../ISO_Fortran_binding.o -c .../ISO_Fortran_binding.cpp

em++ .../allocatable.o -c .../allocatable.cpp

em++ .../array-constructor.o -c .../array-constructor.cpp

...

emar -rcs build/flang/runtime/libFortranRuntime.a .../binary-to-decimal.o

[~/fortran]file build/flang/runtime/libFortranRuntime.a

build/flang/runtime/libFortranRuntime.a: current ar archive

Let’s try again, linking in our shiny new library as part of the Emscripten build step.

[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o build/flang/runtime/libFortranRuntime.a -o hello.js

wasm-ld: warning: function signature mismatch: _FortranAioOutputAscii

>>> defined as (i32, i32, i64) -> i32 in hello.o

>>> defined as (i32, i32, i32) -> i32 in build/flang/runtime/libFortranRuntime.a(io-api.o)

Success? Not quite. A warning is issued, letting us know about a signature mismatch. Emscripten has compiled the symbol `_FortranAioOutputAscii` to take three `i32` arguments. However, `flang-new` has compiled `hello.f08` with the expectation that the symbol takes two `i32` arguments and a single `i64` argument.

^(25) This is [LLVM IR notation](https://llvm.org/docs/LangRef.html#integer-type), meaning an integer of size 32 bits.

^(26) This continues to crop up when compiling R packages for webR. Package authors or vendored libraries may have used tools such as `f2c` that declare a Fortran `SUBROUTINE` to return an `int`, while other libraries might declare a Fortran `SUBROUTINE` to return `void`. Who is right? I’m not sure, as I understand it early Fortran did not have a standard interface to C. Personally, I think returning `void` makes most sense.

This is unfortunate. Despite being emitted as just a warning, if you try running the emitted program using Node you will see that the problem is catastrophic. WebAssembly, unlike a lot of target systems, absolutely requires that symbols defined over multiple compilation units have consistent function signatures, both in argument and return type.

[~/fortran]node hello.js

.../fortran/hello.js:128

    throw ex;

    ^

RuntimeError: unreachable

  at wasm://wasm/001a0366:wasm-function[20]:0x15d9

  at removeRunDependency (/Users/georgestagg/fortran/hello.js:630:7)

Node.js v18.18.0

Rather than going over the debugging process that eventually leads us to what is going on here, let me point you directly to the cause of the problem. Take a look at this comment from the LLVM source:

```
flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
```

```
//===----------------------------------------------------------------------===//
// Type builder models
//===----------------------------------------------------------------------===//

// TODO: all usages of sizeof in this file assume build ==  host == target.
// This will need to be re-visited for cross compilation.

/// Return a function that returns the type signature model for the type `T`
/// when provided an MLIRContext*. This allows one to translate C(++) function
/// signatures from runtime header files to MLIR signatures into a static table
/// at compile-time.
///
/// For example, when `T` is `int`, return a function that returns the MLIR
/// standard type `i32` when `sizeof(int)` is 4.
```

 *And therein lies the problem. For us, the host is different to the target, breaking assumptions in the LLVM source code. Surprisingly, this does not cause as much chaos as you might expect. From what I can tell, this machinery is used only to make the Fortran runtime library functions, written in C++, available to Fortran. There is a compile-time calculation using `sizeof()`, and since most of the sizes match anyway it mostly works fine.

^(27) The [C data model](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models) for the host and target defines how many bits certain fundamental C types are represented with. The specific sizes can differ based on the hardware architecture and your OS.

Unfortunately for us, assuming you’re following along on a modern 64-bit Unix-like system such as Linux or macOS, the sizes don’t match for the `long` data type. The result of `sizeof(long)` on our compiler’s host platform is 8 bytes (`i64`), but for the target platform of `wasm32-unknown-emscripten` the returned value should be 4 bytes (`i32`).

When we compile the Fortran runtime library C++ code using Emscripten, things are fine. The resulting symbols are compiled with signatures such that `long` arguments are `i32`. However, when we compile our Fortran code with `flang-new` the external library symbols are declared such that `long` arguments are `i64`. This difference leads to the inconsistent function signature warning and runtime failure.

Why did using `PRINT()` in our “Hello, World!” program invoke a function that takes an argument of type `long`? Well, in some implementations of Fortran there are so-called “hidden” arguments that are added whenever you pass a Fortran `CHARACTER` type to a function or subroutine. These extra arguments pass in the length of the strings. In the Fortran runtime library the hidden arguments are declared with type `size_t` which, following a chain of `typedef`s, ends up being the same as `unsigned long`. This hidden implicit argument is the one with inconsistent size.****  ***## Hacking around the issue

Unfortunately, I don’t know enough about the LLVM or Flang internals to implement a real solution to this problem. Ideally, `flang-new` would emit the correct use of `i32` or `i64` for the target architecture and data model when cross-compiling, no matter the host architecture the compiler is running on.

Since I can’t solve this today, let’s hack around it for now. We’ll build a version of `flang-new` with the size of a `long` hard-coded to what we need for `wasm32` and Emscripten. We’ll also make some changes so that calls to `malloc()` from Fortran are emitted with an `i32` argument.

^(28) This additionally fixes dynamic allocation with `ALLOCATE()`, a feature introduced in Fortran 90.

The required patches are again shown as a diff below. If you’re following along, save it as a file named `force-4-byte-values.diff` and apply it to the `llvm-project` directory using `git` or the `patch` utility. Finally, recompile `flang-new` once more.

```
diff --git a/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h b/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
index b3fe52f4b..c3c7326da 100644
--- a/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
+++ b/flang/include/flang/Optimizer/Builder/Runtime/RTBuilder.h
@@ -146,7 +146,7 @@ constexpr TypeBuilderFunc getModel<void **>() {
 template <>
 constexpr TypeBuilderFunc getModel<long>() {
 return [](mlir::MLIRContext *context) -> mlir::Type {
-    return mlir::IntegerType::get(context, 8 * sizeof(long));
+    return mlir::IntegerType::get(context, 8 * 4);
 };
 }
 template <>
@@ -187,7 +187,7 @@ constexpr TypeBuilderFunc getModel<long long *>() {
 template <>
 constexpr TypeBuilderFunc getModel<unsigned long>() {
 return [](mlir::MLIRContext *context) -> mlir::Type {
-    return mlir::IntegerType::get(context, 8 * sizeof(unsigned long));
+    return mlir::IntegerType::get(context, 8 * 4);
 };
 }
 template <>
diff --git a/flang/lib/Optimizer/CodeGen/CodeGen.cpp b/flang/lib/Optimizer/CodeGen/CodeGen.cpp
index ba5946415..2931753a8 100644
--- a/flang/lib/Optimizer/CodeGen/CodeGen.cpp
+++ b/flang/lib/Optimizer/CodeGen/CodeGen.cpp
@@ -1225,7 +1225,7 @@ getMalloc(fir::AllocMemOp op, mlir::ConversionPatternRewriter &rewriter) {
 return mlir::SymbolRefAttr::get(userMalloc);
 mlir::OpBuilder moduleBuilder(
 op->getParentOfType<mlir::ModuleOp>().getBodyRegion());
-  auto indexType = mlir::IntegerType::get(op.getContext(), 64);
+  auto indexType = mlir::IntegerType::get(op.getContext(), 32);
 auto mallocDecl = moduleBuilder.create<mlir::LLVM::LLVMFuncOp>(
 op.getLoc(), mallocName,
 mlir::LLVM::LLVMFunctionType::get(getLlvmPtrType(op.getContext()),
@@ -1281,6 +1281,7 @@ struct AllocMemOpConversion : public FIROpConversion<fir::AllocMemOp> {
 mlir::Type heapTy = heap.getType();
 mlir::Location loc = heap.getLoc();
 auto ity = lowerTy().indexType();
+    auto i32ty = mlir::IntegerType::get(rewriter.getContext(), 32);
 mlir::Type dataTy = fir::unwrapRefType(heapTy);
 mlir::Type llvmObjectTy = convertObjectType(dataTy);
 if (fir::isRecordWithTypeParameters(fir::unwrapSequenceType(dataTy)))
@@ -1291,9 +1292,10 @@ struct AllocMemOpConversion : public FIROpConversion<fir::AllocMemOp> {
 for (mlir::Value opnd : adaptor.getOperands())
 size = rewriter.create<mlir::LLVM::MulOp>(
 loc, ity, size, integerCast(loc, rewriter, ity, opnd));
+    auto size_i32 = integerCast(loc, rewriter, i32ty, size);
 heap->setAttr("callee", getMalloc(heap, rewriter));
 rewriter.replaceOpWithNewOp<mlir::LLVM::CallOp>(
-        heap, ::getLlvmPtrType(heap.getContext()), size, heap->getAttrs());
+        heap, ::getLlvmPtrType(heap.getContext()), size_i32, heap->getAttrs());
 return mlir::success();
 }
```

 *[~/fortran]patch -p1 -d llvm-project < force-4-byte-values.diff

[~/fortran]cmake --build build

[0/60] Building CXX object tools/flang/lib/Lower/Runtime.cpp.o

...

[49/49] Generating ../../../../include/flang/ieee_arithmetic.f18.mod

Once LLVM has been rebuilt, try compiling our program once again. This time, it should compile without any warnings and successfully run under Node:

[~/fortran]./build/bin/flang-new -c hello.f08 -o hello.o

[~/fortran]emcc hello.c hello.o build/flang/runtime/libFortranRuntime.a -o hello.js

[~/fortran]node hello.js

Hello, World!*********  ***# Compiling BLAS and LAPACK for WebAssembly

Now that we have a Fortran compiler that can output WebAssembly objects, let’s build some Fortran projects. [BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms) (Basic Linear Algebra Subprograms) is a set of low-level routines that perform many common operations in linear algebra, including tools for matrix and vector multiplication. They are a de facto standard in numerical computation, with several different implementations of the BLAS routines available. Some implementations have been tuned for use on certain hardware, others have been well optimised on account of being around for a long time — the original BLAS routines were released in 1979!

Let’s grab a copy of the latest release of the so-called “reference implementation” of BLAS, written in Fortran 90, and compile it using the patched LLVM we built above.

[~/fortran]curl -L https://www.netlib.org/blas/blas-3.12.0.tgz > blas-3.12.0.tgz

% Total % Received % Xferd Average Speed Time Time Time Current

Dload Upload Total Spent Left Speed

100 323k 100 323k 0 0 317k 0 0:00:01 0:00:01 --:--:-- 317k

[~/fortran]tar xzf blas-3.12.0.tgz

We’ll need to modify `BLAS-3.12.0/make.inc` to tell it about our version of `flang-new` and the Emscripten tools. Modify the following settings, leaving the other lines in that file as they are, then build BLAS using `make`.

```
FC = ../build/bin/flang-new
FFLAGS = -O2
FFLAGS_NOOPT = -O0

AR = emar
RANLIB = emranlib
```

 *[~/fortran]cd BLAS-3.12.0

[~/fortran/BLAS-3.12.0]make

../build/bin/flang-new -O2 -c -o isamax.o isamax.f

../build/bin/flang-new -O2 -c -o sasum.o sasum.f

...

emar cr blas_LINUX.a isamax.o ... xerbla_array.o

emranlib blas_LINUX.a

[~/fortran/BLAS-3.12.0]cd ..

[~/fortran]

That went pretty well! Let’s try using it in a Fortran subroutine compiled for WebAssembly. For fun, we’ll try working with double precision complex numbers. We’ll use the BLAS level 2 routine [`ZGEMV()`](https://netlib.org/lapack/explore-html-3.6.1/dc/dc1/group__complex16__blas__level2_gafaeb2abd9fffa7442b938dc384aeaf47.html#gafaeb2abd9fffa7442b938dc384aeaf47), which performs the matrix-vector operation

\[\begin{equation} \mathbf{y} \leftarrow \alpha\mathbf{A}\mathbf{x} + \beta\mathbf{y}, \end{equation}\]

where \(\alpha\) and \(\beta\) are scalar constants, \(\mathbf{x}\) and \(\mathbf{y}\) are vectors, and \(\mathbf{A}\) is a matrix. Our Fortran routine will take in `alpha`, `beta`, `A`, `X`, and `Y`, with a fixed parameter `N` so that \(\mathbf{A}\) is a square matrix with three rows and columns. The result is written back into \(\mathbf{y}\), so we declare that `Y` is of intent `INOUT`.

```
SUBROUTINE bar(alpha, A, X, beta, Y)
 IMPLICIT NONE
 INTEGER, PARAMETER :: N = 3
 COMPLEX(KIND=8), INTENT(IN) :: alpha, beta, A(N,N), X(N)
 COMPLEX(KIND=8), INTENT(INOUT) :: Y(N)
 EXTERNAL zgemv

 CALL zgemv('N', N, N, alpha, A, N, X, 1, beta, Y, 1)
END
```

 *Notice how with some BLAS routines, `CHARACTER` strings of length one control configuration settings. Here, we pass `'N'` as the first argument. It is one of the reasons we spent time and care above building a version of `flang-new` that can deal with `CHARACTER` arguments and their hidden implicit length arguments for the `wasm32` target.

Next, we’ll write a C program to create some complex variables, send them to Fortran and BLAS for processing, and print the result. This will let us know both that passing double precision complex numbers to Fortran and calling BLAS routines works as expected.

```
#include <stdio.h>
#include <complex.h>

extern void bar_(double complex*, double complex*, double complex*,
 double complex*, double complex*);

int main() {
 double complex alpha = 1.;
 double complex beta = 2.*I;
 double complex A[] = {
 1.*I, 4.  , 5.,
 7.  , 2.*I, 6.,
 8.  , 9.  , 3.*I
 };
 double complex X[] = { 0., 1., 2. };
 double complex Y[] = { 3., 4., 5. };

 bar_(&alpha, A, X, &beta, Y);

 printf("Y[0]: %f + %fi, Y[1]: %f + %fi, Y[2]: %f + %fi\n",
 creal(Y[0]), cimag(Y[0]),
 creal(Y[1]), cimag(Y[1]),
 creal(Y[2]), cimag(Y[2]));
 return 0;
}
```

 *[~/fortran]./build/bin/flang-new -c bar.f08 -o bar.o

[~/fortran]emcc bar.c bar.o build/flang/runtime/libFortranRuntime.a BLAS-3.12.0/blas_LINUX.a -o bar.js

[~/fortran]node bar.js

Y[0]: 23.000000 + 6.000000i, Y[1]: 18.000000 + 10.000000i, Y[2]: 6.000000 + 16.000000i

And there we have it: BLAS compiled from Fortran 90 sources and running under WebAssembly! To finish up, let’s confirm for ourselves that this output is correct,

^(31) Keeping in mind Fortran’s column-major array layout.

\[\begin{equation} \begin{split} \alpha\mathbf{A}\mathbf{x} & + \beta\mathbf{y} \\[0.5em] & = \begin{pmatrix} i & 7 & 8\\ 4 & 2i & 9\\ 5 & 6 & 3i \end{pmatrix} \cdot \begin{pmatrix} 0\\1\\2 \end{pmatrix} + 2i \begin{pmatrix} 3\\4\\5 \end{pmatrix}\\[0.5em] & = \begin{pmatrix} 23+6i\\18+10i\\6+16i \end{pmatrix}. \end{split} \end{equation}\]

## Example: A handwritten digit classifier

The following demo uses a [multi-layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) artificial neural network to classify hand-drawn digits. Try it out with your mouse or touchscreen! Just draw a digit from 0-9 in the box, and the classifier will try to label what digit you wrote. The relative probabilities according to the network are shown in a plot on the right.

It’s not a perfect model, but it works fairly well for me! The weights powering the model have been pre-trained using Python, but the classification is performed at runtime using JavaScript and WebAssembly, running in your browser right now.

With an MLP network, the classification process is essentially a repeated application of matrix-vector addition and multiplication. In this demo the heavy lifting is done by a single Fortran subroutine making use of the BLAS level 2 routine [`DGEMV()`](https://netlib.org/lapack/explore-html-3.6.1/d7/d15/group__double__blas__level2_gadd421a107a488d524859b4a64c1901a9.html).

## Building LAPACK

[LAPACK](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms) (Linear Algebra Package) is a software library for solving linear algebra problems numerically. It’s built upon BLAS and has similarly become a standard with many reimplementations designed for specific hardware or systems.

Let’s finish this post by also building the “reference implementation” of LAPACK.

^(32) Also available from [netlib](https://www.netlib.org/lapack/), released under a modified BSD licence.

[~/fortran]curl -L https://github.com/Reference-LAPACK/lapack/archive/refs/tags/v3.12.0.tar.gz > lapack-3.12.0.tgz

% Total % Received % Xferd Average Speed Time Time Time Current

Dload Upload Total Spent Left Speed

100 7747k 0 7747k 0 0 4117k 0 --:--:-- 0:00:01 --:--:-- 6655k

[~/fortran]tar xzf lapack-3.12.0.tgz

Similar to BLAS, we need to modify some configuration options to let LAPACK know about Emscripten and `flang-new`. Copy the file `lapack-3.12.0/make.inc.example` to `lapack-3.12.0/make.inc`, then make the following modifications. Be sure to replace `[...]` with the full path to the build directory on your machine, and leave the other options in the file as they are.

^(33) A relative path doesn’t work here. Alternatively, simply set the option to read `flang-new` and make it available on your `$PATH`.

```
FC = [...]/build/bin/flang-new
FFLAGS = -O2
FFLAGS_DRV =  $(FFLAGS)
FFLAGS_NOOPT = -O0

AR = emar
RANLIB = emranlib

TIMER = INT_CPU_TIME
```

 *Then, build LAPACK using the `make lib` command to create the WebAssembly static library `liblapack.a`.

[~/fortran]cd lapack-3.12.0

[~/fortran/lapack-3.12.0]make lib

make -C SRC

.../build/bin/flang-new -O2 -c -o sbdsvdx.o sbdsvdx.f

...

emar cr ../../libtmglib.a slatms.o ... dlarnd.o

emranlib ../../libtmglib.a

[~/fortran/lapack-3.12.0]cd ..

[~/fortran]

[~/fortran]file lapack-3.12.0/liblapack.a

lapack-3.12.0/liblapack.a: current ar archive

With this, LAPACK routines can be called in a similar way to the BLAS routine example in the previous section.*  *## Example: Polynomial Interpolation with Linear Algebra

The following demo finds interpolating polynomials for a set of points, demonstrating LAPACK routines running in your web browser.

Click the plot to add new points. An interpolating polynomial will be found to pass through all the points using [Vandermonde’s method](https://en.wikipedia.org/wiki/Vandermonde_matrix). The linear algebra equation given by this method is then solved numerically in LAPACK using the [`DGELS()`](https://netlib.org/lapack/explore-html-3.6.1/d7/d3b/group__double_g_esolve_ga1df516c81d3e902cca1fc79a7220b9cb.html#ga1df516c81d3e902cca1fc79a7220b9cb) routine.

^(34)  It is always possible to find an \(n-1\) degree polynomial containing \(n\) data points exactly. However, when \(n\) is large the polynomial fluctuates wildly between successive data points. This problem is known as [Runge’s phenomenon](https://en.wikipedia.org/wiki/Runge%27s_phenomenon) and can be avoided by using [spline interpolation](https://en.wikipedia.org/wiki/Spline_interpolation).*******