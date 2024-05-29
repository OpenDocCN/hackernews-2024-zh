<!--yml
category: 未分类
date: 2024-05-29 12:47:20
-->

# The rev.ng decompiler goes open source + start of the UI closed beta - rev.ng

> 来源：[https://rev.ng/blog/open-sourcing-renvg-decompiler-ui-closed-beta](https://rev.ng/blog/open-sourcing-renvg-decompiler-ui-closed-beta)

Hi there! Long time no see! It has been a while since our last update about rev.ng, but we've been quietly working hard. So, it's time for some news! 🚀

*tl;dr* In this blog post we:

1.  Announce the open sourcing of the rev.ng decompiler, the start of the UI closed beta, the release of the new website, rev.ng Hub and the docs. Also, we do [private demos](mailto:info@rev.ng).
2.  Explain how to try rev.ng.
3.  Explain the design and goals of rev.ng as a whole.
4.  Explain the relation between the open source project, the UI running in the cloud and the standalone UI.
5.  Present our roadmap towards the 1.0 release.
6.  Describe how to interact with us (i.e., [X/Twitter](https://twitter.com/_revng), [Discord](https://discord.gg/wEQtgKJxcX), [Newsletter](/newsletter-subscribe), [Discourse](https://discuss.rev.ng/)).

## [](#1-what-happens-today)1\. What happens today?

Quite a lot.

*   We are open sourcing [`revng-c`](https://github.com/revng/revng-c), the backend of our decompiler.
    As of today, the whole decompilation engine is open source!
    We're also releasing initial [documentation](https://docs.rev.ng) on how to use rev.ng.
*   We'll soon invite newsletter subscribers to the closed beta for the rev.ng UI.
    We're going to invite people on FIFO basis, make sure [you're registered](/newsletter-subscribe).
*   We're releasing the new website. You are looking at it.
*   We're publishing [rev.ng Hub](https://hub.rev.ng), the entrypoint to use the cloud version of rev.ng.
    Beta users will be able to create projects, run the UI in the browser and collaborate with other users without installing anything.
    If you're not part of the closed beta yet, you can still [explore existing public projects](https://hub.rev.ng/).
*   We're starting to look for people curious about getting a private demo of the rev.ng capabilities ([interested?](mailto:info@rev.ng)).

## [](#2-how-to-try-revng)2\. How to try rev.ng

Very briefly, let's install `revng`. No root required, everything will fit into a single directory that you can discard at any time.

```
$  $ curl -L -s https://rev.ng/downloads/revng-distributable/master/install.sh |  bash  $ cd revng $ source ./environment 
```

OK, let's now decompile a simple program. Consider `example.c`:

```
int  main(int argc,  char  *argv[])  {   return argc *  3;  } 
```

You can compile and decompile it:

```
$ gcc example.c -o example -O2 $ revng artifact \  --analyze \  --progress \  decompile-to-single-file \  example \   | revng ptml --color \   |  grep -A2 -B1 '[^_]main\b'  \   > decompiled.c 
```

You should obtain:

```
_ABI(SystemV_x86_64)  generic64_t  main(generic64_t _argument0)  {   return _argument0 *  3  &  0xFFFFFFFF;  } 
```

Alternatively, take a look at the [documentation](https://docs.rev.ng). In particular, make sure you correctly [set up a working environment](https://docs.rev.ng/user-manual/working-environment/), follow the tutorial to [create a model from scratch](http://docs.rev.ng/user-manual/model-tutorial/) or try directly the [analyses tutorial](http://docs.rev.ng/user-manual/analyses/), which will show you how to decompile a program in a single invocation.

To try the UI, [register to our newsletter](/newsletter-subscribe). As mentioned before, we'll soon start inviting people in small batches.

Note that the goal of this release is to demo the UI and the decompilation results on some binaries on which we get interesting results, let people work on some real world binaries and then let people try whatever they like and get feedback, bug reports and so on.

In particular, here are some non-trivial binaries (with their `.text` size) that we use for testing: [hostname](https://rev.ng/downloads/binaries/hostname) (2K), [ntfscat](https://rev.ng/downloads/binaries/ntfscat) (4K), [umount](https://rev.ng/downloads/binaries/umount) (8K), [chroot](https://rev.ng/downloads/binaries/chroot) (15K), [nc.openbsd](https://rev.ng/downloads/binaries/nc.openbsd) (15K), [apt-get](https://rev.ng/downloads/binaries/apt-get) (18K), [df](https://rev.ng/downloads/binaries/df) (45K), [parted](https://rev.ng/downloads/binaries/parted) (38K), [gzip](https://rev.ng/downloads/binaries/gzip) (54K), [updatedb.plocate](https://rev.ng/downloads/binaries/updatedb.plocate) (56K), [resolvectl](https://rev.ng/downloads/binaries/resolvectl) (61K), [ps](https://rev.ng/downloads/binaries/ps) (47K).

rev.ng supports many ABIs and platforms. However, at this stage, we performed initial QA on Linux x86-64 binaries. rev.ng will evolve significantly during the closed beta period, so make sure you follow our [X/Twitter account](https://twitter.com/_revng) and are subscribed to our [monthly newsletter](/newsletter-subscribe) to know when it's time to come back to try out some new features!

If you have problems, head to [Discourse](https://discuss.rev.ng/) for help.

## [](#3-our-goals)3\. Our goals

This is a good time for a refresher about what the rev.ng decompiler is about. In brief, we focus on:

*   automatic recovery of data structures;
*   a modern UX;
*   collaborative reversing;
*   wide platform support;
*   extensibility.

### [](#automatic-data-structures-recovery)Automatic data structures recovery

Thanks to Data Layout Analysis, we can automatically recover the layout of `struct`s *interprocedurally*.

Consider the following `struct`, a node in a linked list with an array of 5 integers:

```
struct  node  {   int64_t data[5];   struct  node  *next;  }; 
```

A function summing the elements of the array:

```
int64_t  sum(struct  node  *n)  {   int64_t result =  0;   for  (int i =  0; i <  5;  ++i)  result += n->data[i];   return result;  } 
```

And a function traversing the linked list:

```
int64_t  compute(struct  node  *n)  {   int64_t result =  0;   while  (n)  {  result +=  sum(n);  n = n->next;   }   return result;  } 
```

rev.ng produces the following output out of the box (except for the comments):

```
typedef  struct  _PACKED  {   

  generic64_t _offset_0[5];   

 _struct_1876 *_offset_40;  } _struct_1876;  

_ABI(SystemV_x86_64)  generic64_t  sum(_struct_1876 *_argument0)  {   generic64_t _var_0 =  0, _var_1 =  0;   do  {   

 _var_0 = _var_0 + _argument0->_offset_0[_var_1];  _var_1 = _var_1 +  1;   }  while  (_var_1 !=  5);   return _var_0;  }  
_ABI(SystemV_x86_64)  generic64_t  compute(_struct_1876 *_argument0)  {  _struct_1876 *_var_0 = _argument0;   generic64_t _var_1 =  0;   generic64_t _var_2;   do  {  _var_2 =  sum(_var_0);  _var_1 = _var_1 + _var_2;   

 _var_0 = _var_0->_offset_40;   }  while  (_var_0);   return _var_1;  } 
```

Pretty nice, isn't it? You can also directly on [Hub](https://hub.rev.ng/revng-demos/linked-list)

Bonus: we emit syntactically valid C code.

**Status as of today**: available.

### [](#a-modern-ux)A modern UX

Our UI is based on VSCode. If you use VSCode, you already know how to use it. Also, the UI can both run in a browser tab or as a standalone application.

Shortcuts:

*   `Ctrl + Click`, navigates to the definition of the function/type under the cursor.
*   `N` renames it.
*   `Y` let's you edit its type.
*   `X` shows references.

rev.ng is not just a decompiler, it's an *interactive* decompiler. This means that you can make changes interactively and only the things that are affected by your change will be recomputed, just like you would expect.

**Status as of today**: the UI is available for participants in the closed beta.

### [](#collaborative-reversing)Collaborative reversing

The rev.ng UI has a client-server architecture.

Multiple users can connect to the same daemon instance and work at the same time on the same project. If you use the cloud version, you also get a GitHub-like application to manage projects, the [rev.ng Hub](https://hub.rev.ng).

**Status as of today**: works, needs some love to improve the user experience ([roadmap item #797](/roadmap#feature-797)).

### [](#wide-architecture-support)Wide architecture support

In order to understand why we can easily support many architectures, let's briefly digress on compilers and emulators.

Compiler are usually organized in frontend, mid-end and backend. The frontend handles details specific to the input language. The mid-end performs optimizations on an intermediate representation (IR) that is independent of both the input language and the target architecture. Finally, the backend performs architecture-specific optimizations and outputs executable code.

Let's consider a part of the LLVM ecosystem:

LLVM IR is LLVM's intermediate representation.

Let's now consider QEMU, a well-known emulator. When running in emulation mode, as opposed to virtualization mode (e.g., KVM), works in a similar fashion. It has its own frontends, its own IR (tiny code) and multiple backends. The main difference relies in the fact that the input is not C or C++ but ARM or x86-64 executable code:

This type of emulation is also known as dynamic binary translation.

Now, rev.ng works like this:

Basically we use the first half of the QEMU pipeline to *lift* executable code to tiny code, convert it to LLVM IR and then proceed with the decompilation process.

This means we can support any [architecture supported by QEMU](https://wiki.qemu.org/Documentation/Platforms) with relative ease.

However, architecture support is just part of the story. The other big topic is how well a certain platform or, more specifically, an ABI, is supported. In this regard, we put a lot of effort to be able to describe in a declarative way all the properties of an ABI. Our [ABI description format](https://github.com/revng/revng/tree/f297712/share/revng/abi) is general enough that supporting new ABIs will be quite easy.

**Status as of today**: we support x86, x86-64, ARM, AArch64, MIPS, and s390x to various degrees of maturity. In terms of binary formats, we support ELF, PE/COFF and Mach-O. We can also import from `.idb`, DWARF debug info and PDB debug info. We performed most QA on Linux x86-64 binaries, all the rest needs additional QA.

If you have a specific interest in some platform (and a budget), [we're happy to talk `:)`](mailto:info@rev.ng).

Track the [roadmap item #58](/roadmap#feature-58) for further QA on more platforms.

### [](#extensibility)Extensibility

rev.ng aspires to become the go-to framework for reverse engineering tools. For this reason, the whole project is open source, with the exception of our interactive UI.

rev.ng is also easily scriptable. Our project file, [the model](https://docs.rev.ng/user-manual/model/), is just a YAML document. Here's an example with some comments.

```
 Architecture: x86_64 DefaultABI: SystemV_x86_64 
 Segments:   -  StartAddress:  "0x400000:Generic64"   VirtualSize:  7   StartOffset:  0   FileSize:  7   IsReadable:  true   IsWriteable:  false   IsExecutable:  true  
 Functions:   -  
  Entry:  "0x400000:Code_x86_64"   
  CustomName:  "Sum"   
  Prototype:  "/Types/1809-CABIFunctionType"  
 Types:   
  -  Kind: PrimitiveType  ID:  1288   PrimitiveKind: Unsigned  Size:  8  

  -  Kind: CABIFunctionType  ABI: SystemV_x86_64  ID:  1809   Arguments:   -  Index:  0   CustomName: FirstAddend  Type:   UnqualifiedType:  "/Types/1288-PrimitiveType"   -  Index:  1   CustomName: SecondAddend  Type:   UnqualifiedType:  "/Types/1288-PrimitiveType"   ReturnType:   UnqualifiedType:  "/Types/1288-PrimitiveType" 
```

The docs offers a [step-by-step tutorial](http://docs.rev.ng/user-manual/model-tutorial/) to understand the model above.

Basically, if you can parse JSON or YAML, you can script rev.ng.

We plan to offer easy-to-use wrappers for Python and TypeScript to easily make changes and obtain artifacts (e.g., decompiled code or disassembly).

Example usage in Python:

```
>>>  import yaml >>>  from revng import model >>>  from pathlib import Path >>> binary = yaml.load(Path("model.yml").read_text(), Loader=model.YamlLoader)  >>> function = binary.Functions[0]  >>> function.CustomName Sum
>>> function.CustomName +=  "42"  >>> function.CustomName Sum42
>>>   >>> function.get_artifact("decompile-to-single-file")  uint64_t Sum(uint64_t FirstAddend, uint64_t SecondAddend)  {  ... 
```

Example usage in TypeScript:

```
import  { parseModel }  from  "revng-model";  import  { readFileSync }  from  "fs";  
const file =  readFileSync("model.yml",  "utf-8");  const binary =  parseModel(file);  const the_function = binary.Functions[0];  console.log(the_function.CustomName.str)  the_function.CustomName.str  +=  "42"  console.log(the_function.CustomName.str)   the_function.get_artifact("decompile-to-single-file") 
```

```
orc shell npm install typescript @types/node revng-model
tsc example.ts
node example.js 
```

Finally, since we use extensively use LLVM IR as our main internal representation, you can exploit many many tool in its ecosystem. For instance, you could use [KLEE](https://klee.github.io/) to perform symbolic execution or [SanitizerCoverage](https://clang.llvm.org/docs/SanitizerCoverage.html) + [libFuzzer](https://llvm.org/docs/LibFuzzer.html) (or other tools) to perform coverage-guided fuzzing. Also, since the output of our decompiler is valid C, we've also been able to play around with [clang static analyzer](https://clang-analyzer.llvm.org/) to look for common vulnerabilities directly on the decompiled code. But this probably deserves a blog post on its own.

**Status as of today**: we have Python and TypeScript wrappers to manipulate the model, but not an easy way to run analyses and fetch artifacts. [Roadmap item #17](/roadmap#feature-17) tracks building a full fledged Python client.

## [](#4-open-source-vs-free-to-use-vs-premium)4\. Open source vs free-to-use vs premium

For a quick comparison of the various components of rev.ng take a look at the [feature comparison page](/pricing).

Very briefly:

1.  The rev.ng framework is fully open source. You can decompile anything you want from the CLI.
2.  The UI will be available in the following forms:
    1.  free to use in the cloud for public projects;
    2.  available through a subscription in the cloud for private projects;
    3.  available at a cost as a fully standalone, fully offline application.

### [](#revng-in-the-cloud)rev.ng in the cloud?

Yes.

1.  You can create a project through [rev.ng Hub](https://hub.rev.ng/) and invite collaborators.
2.  The UI runs in the browser.
3.  The backend runs in our cloud.

Here's the deal:

1.  Public projects are free. If you're OK with having your project file public, you can use the cloud version of rev.ng for free, UI included.
2.  Private projects require a subscription.

We're also willing to discuss installing a private instance of the rev.ng cloud service on premise, if you're into kubernetes. [Interested?](mailto:info@rev.ng)

As mentioned above, we will also offer a fully standalone version of the UI for which you can buy a regular license and run it wherever you prefer.

## [](#5-the-roadmap)5\. The roadmap

rev.ng is a complex project and getting to the 1.0 version is going to take quite some effort. But don't worry, we have a detailed roadmap on how to get there.

The roadmap is divided into 4 tiers:

1.  Tier 1 (done): alpha version, demo to friends.
2.  Tier 2 (starts today): beta version, give access to the cloud version to newsletter subscribers.
3.  Tier 3 (to come): open beta.
4.  Tier 4 (to come): 1.0 release.

Want to know more? Check out the [detailed roadmap page](/roadmap).

## [](#6-how-to-get-in-touch-and-stay-up-to-date)6\. How to get in touch and stay up to date

We publish and can be contacted in several ways:

*   [X/Twitter](https://twitter.com/_revng): frequent news about development and major announcements.
*   [Discord](https://discord.gg/wEQtgKJxcX): random chat and real-time discussions with the dev team.
*   [Discourse](https://discuss.rev.ng/): a forum for users to get support and report bugs.
*   [GitHub](https://github.com/revng): where all our open source development takes place, please open an issue if you're trying to extend rev.ng.
*   [Monthly newsletter](/newsletter-subscribe): subscribe to participate in the closed beta and get monthly news about the project advancements.
*   [E-mail](mailto:info@rev.ng): want some privacy? Drop us an e-mail.

That's all for today. Stay tuned for more! 🚀