<!--yml

category: 未分类

date: 2024-05-27 14:39:35

-->

# Re: [PATCH 00/45] C++: Convert the kernel to C++ - H. Peter Anvin

> 来源：[https://lore.kernel.org/lkml/3465e0c6-f5b2-4c42-95eb-29361481f805@zytor.com/](https://lore.kernel.org/lkml/3465e0c6-f5b2-4c42-95eb-29361481f805@zytor.com/)

```
From: "H. Peter Anvin" <hpa@zytor.com>
To: David Howells <dhowells@redhat.com>,
	linux-kernel@vger.kernel.org, pinskia@gmail.com
Subject: Re: [PATCH 00/45] C++: Convert the kernel to C++
Date: Tue, 9 Jan 2024 11:57:47 -0800	[thread overview]
Message-ID: <3465e0c6-f5b2-4c42-95eb-29361481f805@zytor.com> (raw)
In-Reply-To: <152261521484.30503.16131389653845029164.stgit@warthog.procyon.org.uk>

Hi all, I'm going to stir the hornet's nest and make what has become the 
ultimate sacrilege.

Andrew Pinski recently made aware of this thread. I realize it was 
released on April 1, 2018, and either was a joke or might have been 
taken as one. However, I think there is validity to it, and I'm going to 
try to motivate my opinion here.

Both C and C++ has had a lot of development since 1999, and C++ has in 
fact, in my personal opinion, finally "grown up" to be a better C for 
the kind of embedded programming that an OS kernel epitomizes. I'm 
saying that as the author of a very large number of macro and inline 
assembly hacks in the kernel.

What really makes me say that is that a lot of things we have recently 
asked for gcc-specific extensions are in fact relatively easy to 
implement in standard C++ and, in many cases, allows for infrastructure 
improvement *without* global code changes (see below.)

C++14 is in my option the "minimum" version that has reasonable 
metaprogramming support has most of it without the type hell of earlier 
versions (C++11 had most of it, but C++14 fills in some key missing pieces).

However C++20 is really the main game changer in my opinion; although 
earlier versions could play a lot of SFINAE hacks they also gave 
absolutely useless barf as error messages. C++20 adds concepts, which 
makes it possible to actually get reasonable errors.

We do a lot of metaprogramming in the Linux kernel, implemented with 
some often truly hideous macro hacks. These are also virtually 
impossible to debug. Consider the uaccess.h type hacks, some of which I 
designed and wrote. In C++, the various casts and case statements can be 
unwound into separate template instances, and with some cleverness can 
also strictly enforce things like user space vs kernel space pointers as 
well as already-verified versus unverified user space pointers, not to 
mention easily handle the case of 32-bit user space types in a 64-bit 
kernel and make endianness conversion enforceable.

Now, "why not Rust"? First of all, Rust uses a different (often, in my 
opinion, gratuitously so) syntax, and not only would all the kernel 
developers need to become intimately familiar to the level of getting 
the same kind of "feel" as we have for C, but converting C code to Rust 
isn't something that can be done piecemeal, whereas with some cleanups 
the existing C code can be compiled as C++.

However, I find that I disagree with some of David's conclusions; in 
fact I believe David is unnecessarily *pessimistic* at least given 
modern C++.

Note that no one in their sane mind would expect to use all the features 
of C++. Just like we have "kernel C" (currently a subset of C11 with a 
relatively large set of allowed compiler-specific extensions) we would 
have "kernel C++", which I would suggest to be a strictly defined subset 
of C++20 combined with a similar set of compiler extensions.) I realize 
C++20 compiler support is still very new for obvious reasons, so at 
least some of this is forward looking.

So, notes on this specific subset based on David's comments.

On 4/1/18 13:40, David Howells wrote:
> 
> Here are a series of patches to start converting the kernel to C++.  It
> requires g++ v8.
> 
> What rocks:
> 
>   (1) Inline template functions, which makes implementation of things like
>       cmpxchg() and get_user() much cleaner. 
Much, much cleaner indeed. But it also allows for introducing things 
like inline patching of immediates *without* having to change literally 
every instance of a variable.

I wrote, in fact, such a patchset. It probably included the most awful 
assembly hacks I have ever done, in order to implement the mechanics, 
but what *really* made me give up on it was the fact that every site 
where a patchable variable is invoked would have to be changed from, say:

	foo = bar + some_init_offset;

... to ...

	foo = imm_add(bar, some_init_offset);

>   (2) Inline overloaded functions, which makes implementation of things like
>       static_branch_likely() cleaner. 
Basically a subset of the above (it just means that for a specific set 
of very common cases it isn't necessary to go all the way to using 
templates, which makes the syntax nicer.)

>   (3) Class inheritance.  For instance, all those inode wrappers that require
>       the base inode struct to be included and that has to be accessed with
>       something like:
> 
> 	inode->vfs_inode.i_mtime
> 
>       when you could instead do:
> 
> 	inode->i_mtime 
This is nice, but it is fundamentally syntactic sugar. Similar things 
can be done with anonymous structures, *except* that C doesn't allow 
another structure to be anonymously included; you have to have an 
entirely new "struct" statement defining all the fields. Welcome to 
macro hell.

> What I would disallow:
> 
>   (1) new and delete.  There's no way to pass GFP_* flags in. 
Yes, there is.

void * operator new (size_t count, gfp_flags_t flags);
void operator delete(void *ptr, ...whatever kfree/vfree/etc need, or a 
suitable flag);

>   (2) Constructors and destructors.  Nests of implicit code makes the code less
>       obvious, and the replacement of static initialisation with constructor
>       calls would make the code size larger. 
Yes and no. It also makes it *way* easier to convert to and from using 
dedicated slabs; we already use semi-initialized slabs for some kinds of 
objects, but it requires new code to make use of.

We already *do* use constructors and *especially* destructors for a lot 
of objects, we just call them out.

Note that modern C++ also has the ability to construct and destruct 
objects in-place, so allocation and construction/destruction aren't 
necessarily related.

There is no reason you can't do static initialization where possible; 
even constructors can be evaluated at compile time if they are constexpr.

Constructors (and destructors, for modules) in conjunction with gcc's 
init_priority() extension is also a nice replacement for linker hack 
tables to invoke intializer functions.

>   (3) Exceptions and RTTI.  RTTI would bulk the kernel up too much and
>       exception handling is limited without it, and since destructors are not
>       allowed, you still have to manually clean up after an error. 
Agreed here, especially since on many platforms exception handling 
relies on DWARF unwind information.

>   (4) Operator overloading (except in special cases). 
See the example of inline patching above. But yes, overloading and 
*especially* operator overloading should be used only with care; this is 
pretty much true across the board.

>   (5) Function overloading (except in special inline cases). 
I think we might find non-inline cases where it matters, too.

>   (6) STL (though some type trait bits are needed to replace __builtins that
>       don't exist in g++). 
Just like there are parts of the C library which is really about the 
compiler and not part of the library. <type_traits> is part of that for C++.

>   (7) 'class', 'private', 'namespace'.
> 
>   (8) 'virtual'.  Don't want virtual base classes, though virtual function
>       tables might make operations tables more efficient. 
Operations tables *are* virtual classes. virtual base classes make sense 
in a lot of cases, and we de facto use them already.

However, Linux also does conversion of polymorphic objects from one type 
to another -- that is for example how device nodes are implemented. 
Using this with C++ polymorphism without RTTI does require some 
compiler-specific hacks, unfortunately.

> Issues:
> 
>   (1) Need spaces inserting between strings and symbols. 
I have to admit I don't really grok this?

>   (2) Direct assignment of pointers to/from void* isn't allowed by C++, though
>       g++ grudgingly permits it with -fpermissive.  I would imagine that a
>       compiler option could easily be added to hide the error entirely. 
Seriously. It should also enforce that it should be a trivial type. 
Unfortunately it doesn't look like there is a way to create user-defined 
implicit conversions from one pointer to another (via a helper class), 
which otherwise would have had some other nice applications.

>   (3) Need gcc v8+ to statically initialise an object of any struct that's not
>       really simple (e.g. if it's got an embedded union). 
Worst case: constexpr constructor.

>   (4) Symbol length.  Really need to extern "C" everything to reduce the size
>       of the symbols stored in the kernel image.  This shouldn't be a problem
>       if out-of-line function overloading isn't permitted. 
This really would lose arguably the absolutely biggest advantage of C++: 
type-safe linkage. This is the one reason why Linus actually tried to 
use C++ in one single version of the kernel in the early days (0.99.14, 
if I remember correctly.) At that time, g++ was nowhere near mature 
enough, and it got dropped right away.

> So far, it gets as far as compiling init/main.c to a .o file. 
;)

```

* * *

```
next prev parent reply  other threads:[~2024-01-09 19:58 UTC|newest]

Thread overview: 103+ messages / expand[flat|nested]  mbox.gz  Atom feed  top
2018-04-01 20:40 [PATCH 00/45] C++: Convert the kernel to C++ David Howells
2018-04-01 20:40 ` [PATCH 01/45] Use UINT_MAX, not -1, to represent an invalid UID, GID or project ID David Howells
2018-04-01 23:04   ` Randy Dunlap
2018-04-01 20:40 ` [PATCH 02/45] Fix exception_enter() return value David Howells
2018-04-05  1:34   ` Sasha Levin
2018-04-01 20:40 ` [PATCH 03/45] Fix loop var in be32_to_cpu_array() and cpu_to_be32_array() David Howells
2018-04-01 20:40 ` [PATCH 04/45] Fix use of ACPI_COMPANION_SET() David Howells
2018-04-01 20:40 ` [PATCH 05/45] C++: Set compilation as C++ for .c files David Howells
2018-04-02  6:10   ` kbuild test robot
2018-04-02  6:10   ` kbuild test robot
2018-04-03 13:16   ` David Howells
2018-04-03 13:27     ` Fengguang Wu
2018-04-10  8:44     ` David Howells
2018-04-10  9:45       ` Fengguang Wu
2018-04-11  1:13         ` Li, Philip
2018-04-01 20:40 ` [PATCH 06/45] C++: Do some basic C++ type definition David Howells
2018-04-02  4:37   ` kbuild test robot
2018-04-02  6:10   ` kbuild test robot
2018-04-01 20:40 ` [PATCH 07/45] C++: Define a header with some C++ type traits for type checking David Howells
2018-04-02  7:00   ` kbuild test robot
2018-04-01 20:41 ` [PATCH 08/45] C++: Implement abs() as an inline template function David Howells
2018-04-01 20:41 ` [PATCH 09/45] C++: x86: Fix the x86 syscall table production for C++ David Howells
2018-04-02  7:57   ` kbuild test robot
2018-04-01 20:41 ` [PATCH 10/45] C++: x86: Turn xchg(), xadd() & co. into inline template functions David Howells
2018-04-01 20:41 ` [PATCH 11/45] C++: x86: Turn cmpxchg() " David Howells
2018-04-01 20:41 ` [PATCH 12/45] C++: x86: Turn cmpxchg_double() " David Howells
2018-04-01 20:41 ` [PATCH 13/45] C++: x86: Turn cmpxchg64() " David Howells
2018-04-01 20:41 ` [PATCH 14/45] C++: x86: Turn put_user(), get_user() " David Howells
2018-04-01 20:41 ` [PATCH 15/45] C++: Need space between string and symbol David Howells
2018-04-01 20:41 ` [PATCH 16/45] C++: Disable VERIFY_OCTAL_PERMISSIONS() for the moment David Howells
2018-04-01 20:41 ` [PATCH 17/45] C++: Turn READ_ONCE(), WRITE_ONCE() & co. into inline template functions David Howells
2018-04-01 20:42 ` [PATCH 18/45] C++: Turn RCU accessors " David Howells
2018-04-01 20:42 ` [PATCH 19/45] C++: Turn ktime_add/sub_ns() " David Howells
2018-04-01 20:42 ` [PATCH 20/45] C++: init/main: Constify pointers David Howells
2018-04-01 20:42 ` [PATCH 21/45] C++: Set the type of atomic64_t to s64 David Howells
2018-04-01 20:42 ` [PATCH 22/45] C++: Define apic_intr_mode after the enum definition, not before David Howells
2018-04-01 20:42 ` [PATCH 23/45] C++: Don't do "extern asmlinkage" David Howells
2018-04-01 20:42 ` [PATCH 24/45] C++: Fix BUILD_BUG_ON_ZERO() David Howells
2018-04-01 20:42 ` [PATCH 25/45] C++: Fix void variables David Howells
2018-04-01 20:42 ` [PATCH 26/45] C++: Can't have variable/member names the same as typedef names David Howells
2018-04-01 20:42 ` [PATCH 27/45] C++: Disable __same_type() for the moment David Howells
2018-04-01 20:43 ` [PATCH 28/45] C++: Move ctx_state enum out of struct context_tracking David Howells
2018-04-01 20:43 ` [PATCH 29/45] C++: Move the print_line_t enum before first use David Howells
2018-04-01 20:43 ` [PATCH 30/45] C++: Include linux/hrtimer.h from linux/timer.h David Howells
2018-04-01 20:43 ` [PATCH 31/45] C++: Avoid using 'compl' and 'and' as names David Howells
2018-04-02  7:57   ` kbuild test robot
2018-04-01 20:43 ` [PATCH 32/45] C++: __to_fd() needs to reduce the size of v for struct fd::flags David Howells
2018-04-01 20:43 ` [PATCH 33/45] C++: Move irqchip_irq_state enum David Howells
2018-04-01 20:43 ` [PATCH 34/45] C++: Fix up use of LIST_POISON* David Howells
2018-04-01 20:43 ` [PATCH 35/45] C++: Fix static_branch_likely/unlikely() David Howells
2018-04-01 20:43 ` [PATCH 36/45] C++: Fix kernfs_type() int->enum David Howells
2018-04-01 20:43 ` [PATCH 37/45] C++: Fix page_zonenum() int->enum David Howells
2018-04-01 20:44 ` [PATCH 38/45] C++: mutex_trylock_recursive_enum() int->enum David Howells
2018-04-01 23:10   ` Randy Dunlap
2018-04-01 20:44 ` [PATCH 39/45] C++: Fix spinlock initialisation David Howells
2018-04-01 20:44 ` [PATCH 40/45] C++: Fix sema_init() David Howells
2018-04-01 20:44 ` [PATCH 41/45] C++: Cast in bitops David Howells
2018-04-02  6:10   ` kbuild test robot
2018-04-01 20:44 ` [PATCH 42/45] C++: Hide C++ keywords David Howells
2018-04-02  7:32   ` kbuild test robot
2018-04-01 20:44 ` [PATCH 43/45] C++: Don't need to declare struct pgd_t after typedef David Howells
2018-04-01 20:44 ` [PATCH 44/45] C++: Can't declare unsized-array in struct cgroup David Howells
2018-04-01 20:44 ` [PATCH 45/45] C++: Move initcall_level_names[] to __initdata section David Howells
2018-04-01 22:20 ` [PATCH 00/45] C++: Convert the kernel to C++ Randy Dunlap
2018-04-02  9:28 ` Vegard Nossum
2024-01-09 19:57 ` [H. Peter Anvin [this message]](#t)
2024-01-09 23:29   ` Andrew Pinski
2024-01-11 21:01     ` Arsen Arsenović
2024-01-10  0:29   ` David Howells
2024-01-10  8:58   ` Jiri Slaby
2024-01-10 13:04     ` Neal Gompa
2024-01-10 15:52       ` Jason Gunthorpe
2024-01-10 16:05         ` H. Peter Anvin
2024-01-10 16:25         ` Neal Gompa
2024-01-10 17:57           ` Theodore Ts'o
2024-01-12  2:23             ` H. Peter Anvin
2024-01-12  2:52               ` Kent Overstreet
2024-01-11  8:06       ` Andreas Herrmann
2024-01-10 15:01   ` Michael de Lang
     [not found]     ` <69fe1c0c-b5ec-4031-b719-d9c14742929c@metux.net>
2024-01-12 21:58       ` Michael de Lang
2024-01-11  4:24   ` John Hubbard
2024-01-11  5:09     ` Dave Airlie
2024-01-11 15:24       ` Eric Curtin
2024-01-11 21:37       ` David Laight
2024-01-11 12:39   ` Chris Down
2024-01-11 19:40     ` David Laight
2024-01-24  6:53       ` Jiri Slaby
2024-01-12  2:54   ` H. Peter Anvin
2024-01-12  8:52   ` David Howells
2024-01-12 23:53     ` H. Peter Anvin
2024-01-09 23:40 ` David Howells
2024-01-10  7:13   ` Alexey Dobriyan
2024-01-12  2:25   ` H. Peter Anvin
2024-01-12  2:40   ` H. Peter Anvin
2024-01-11 23:09 ` Arsen Arsenović
2024-01-12  9:20 ` David Howells
2024-01-12 21:35   ` Arsen Arsenović
2024-01-12 23:41   ` David Howells
2018-04-01 21:32 Alexey Dobriyan
2024-01-11 10:56 Alexey Dobriyan
2024-01-11 10:58 ` Neal Gompa
2024-01-11 11:12   ` Alexey Dobriyan
2024-01-11 12:12   ` Andreas Herrmann

```

* * *

```
Reply instructions:

You may reply publicly to this message via plain-text email
using any one of the following methods:

* Save the following mbox file, import it into your mail client,
  and reply-to-all from there: mbox

  Avoid top-posting and favor interleaved quoting:
  https://en.wikipedia.org/wiki/Posting_style#Interleaved_style

* Reply using the --to, --cc, and --in-reply-to
  switches of git-send-email(1):

  git send-email \
    --in-reply-to=3465e0c6-f5b2-4c42-95eb-29361481f805@zytor.com \
    --to=hpa@zytor.com \
    --cc=dhowells@redhat.com \
    --cc=linux-kernel@vger.kernel.org \
    --cc=pinskia@gmail.com \
    /path/to/YOUR_REPLY

  https://kernel.org/pub/software/scm/git/docs/git-send-email.html

* If your mail client supports setting the In-Reply-To header
  via mailto: links, try the mailto: link

```

请确保你的回复在顶部有一个**主题：**标题，并在消息正文之前有一个空行。

* * *

```
This is a public inbox, see mirroring instructions
for how to clone and mirror all data and code used for this inbox;
as well as URLs for NNTP newsgroup(s).
```
