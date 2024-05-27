<!--yml
category: 未分类
date: 2024-05-27 13:25:39
-->

# An exploration of SBCL internals - simonsafar.com

> 来源：[https://simonsafar.com/2020/sbcl/](https://simonsafar.com/2020/sbcl/)

# An exploration of SBCL internals

2020/07/10

[SBCL](http://www.sbcl.org) is a Common Lisp implementation, with a performant compiler and support for a wide range of platforms. It has excellent [documentation](http://www.sbcl.org/manual/), and even a [guide to internals](http://sbcl.org/sbcl-internals/) (... this is written towards people who already know a lot about how it works though). [This paper](https://research.gold.ac.uk/2336/1/sbcl.pdf) about SBCL's fairly unique build process is also worth reading.

The main goal of this article (... it might even turn into a series of articles) is to give a tour of the internals of SBCL (... as an example of how a Lisp system looks like from the inside) while not assuming a lot of implementation-specific knowledge on the reader's part. It's also an experiment in "learning in the public"; at the point of starting to write this, I don't actually know a lot about how SBCL works either. We're going to figure this out together.

We're going to try answering the following questions:

*   How are Lisp objects represented in memory, at runtime? Where we do we store a string's length? What's a function object like? How about just a number? Or a Lisp symbol?
*   Where is actual compiled Lisp code? What kind of machine code do we generate? If a function calls another function, how do we resolve what we're calling into?
*   Where in memory are bindings for special variables? Or thread-locals?
*   Where is the REPL (... the actual loop) implemented?
*   When we evaluate a **defun**, how do we call the compiler? Can we call into the compiler directly, to turn a Lisp form into machine code? Can we take a machine code blob, turn it into a proper Lisp function by manually creating objects, and then call it?
*   Where is the memory allocator? How do ordinary functions call into it? Where does it call down into the OS?
*   What's the rough theory behind the GC? How does it track pointers? Does it relocate memory?
*   Lisp systems generally start up by loading entire Lisp "cores". How do these files look like? Do we map them 1:1 to memory? Can we enumerate all the objects in them?
*   What's "Genesis" or cold start, often encountered in SBCL source code?

To follow along, we're assuming that you're reasonably familiar with Lisp, you're optimally not overly new to x86 assembly, and have SBCL installed (... my setup also includes Emacs and SLIME, but feel free to use whatever fits you best).

# Disassembly

Let's start with a fairly basic tool: the disassembler. Let's write a really basic function:

```
 (defun testfunc (a b)
  (+ a b 33)) 
```

Then, we can invoke the disassembler by

```
(disassemble 'testfunc)
```

What we get is along the lines of

```
 ; disassembly for TESTFUNC
; Size: 43 bytes. Origin: #x1002440235
; 35:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; 39:       48894DF8         MOV [RBP-8], RCX
; 3D:       488B55F0         MOV RDX, [RBP-16]
; 41:       488B7DE8         MOV RDI, [RBP-24]
; 45:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 4C:       BF42000000       MOV EDI, 66
; 51:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 58:       488BE5           MOV RSP, RBP
; 5B:       F8               CLC
; 5C:       5D               POP RBP
; 5D:       C3               RET
; 5E:       CC0F             BREAK 15                         ; Invalid argument count trap
NIL 
```

The interesting part to look for is "MOV EDI, 66". Apparently, RDX and RDI are the two parameters to "GENERIC-+", so what is happening is we take the two parameters, add them, we then fill EDI with a third parameter, and add that one, too. For reasons yet unknown to us, we shift that integer to the left by one bit, thus we have 66 instead of 33... but the value is clearly related to the constant we entered.

So... let's try hacking this a little bit, to check this assumption. If you look at the actual machine code on the left: "BF420000..."... you might notice that 0x42 is 66 in decimal. (Well, the value has to come from *somewhere*?) How about adjusting the actual value in the code so that it reads, for example, 0x50? (... which is 80 as a decimal number... so if we call *(testfunc 0 0)*, we'd expect it to return 40, instead of the current 33.

So... we need a way of actually writing to memory locations. We even know *where* we'd want to write: at the top of the code listing, we can see

```
; Size: 43 bytes. Origin: #x1002440235
```

indicating the origin, with helpful starting points for each instruction. Thus, our "0x42" byte is located at **0x100244024d** (... since ...4C is the first byte, ...4D is the second). SBCL provides a way for creating pointer values from integers: [sb-sys:int-sap](http://www.sbcl.org/manual/#Untyped-memory) does exactly this, which then we can read by invoking

```
 CL-USER> (sb-sys:sap-ref-8 (sb-sys:int-sap #x100244024d) 0)
          66 
```

(the zero value being the offset compared to the memory address). It is exactly what we expected. Then, we can *setf* the same byte to something we want:

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100244024d) 0) #x50)
80 
```

If we disassemble our function now, we can see that the modification was indeed done:

```
 ; disassembly for TESTFUNC
; Size: 43 bytes. Origin: #x1002440235
; 35:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; 39:       48894DF8         MOV [RBP-8], RCX
; 3D:       488B55F0         MOV RDX, [RBP-16]
; 41:       488B7DE8         MOV RDI, [RBP-24]
; 45:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 4C:       BF50000000       MOV EDI, 80
; 51:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 58:       488BE5           MOV RSP, RBP
; 5B:       F8               CLC
; 5C:       5D               POP RBP
; 5D:       C3               RET
; 5E:       CC0F             BREAK 15                         ; Invalid argument count trap
NIL 
```

... and indeed, trying out our function again:

```
 CL-USER> (testfunc 0 0)
40 
```

Well, Lisp is not magic after all!

# Memory layout

Let's now look at how Lisp objects are laid out in memory! How does, for example, a cons cell look like? How about a symbol? Or a string?

We can use our trusty tool the disassembler to obtain addresses of things:

```
 (defun testfunc2 ()
  (format t "hello world")) 
```

... it can't get much simpler than this. What we get is...

```
 ; disassembly for TESTFUNC2
; Size: 70 bytes. Origin: #x10060509AC
; AC:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; B0:       48894DF8         MOV [RBP-8], RCX
; B4:       498BBD28020000   MOV RDI, [R13+552]               ; tls: *STANDARD-OUTPUT*
; BB:       83FF61           CMP EDI, 97
; BE:       480F443C25D8B54A20 CMOVEQ RDI, [#x204AB5D8]       ; *STANDARD-OUTPUT*
; C7:       4883EC10         SUB RSP, 16
; CB:       488B1586FFFFFF   MOV RDX, [RIP-122]               ; "hello world"
; D2:       B904000000       MOV ECX, 4
; D7:       48892C24         MOV [RSP], RBP
; DB:       488BEC           MOV RBP, RSP
; DE:       B8F8943120       MOV EAX, #x203194F8              ; # <fdefn write-string="">; E3:       FFD0             CALL RAX
; E5:       BA17001020       MOV EDX, #x20100017              ; NIL
; EA:       488BE5           MOV RSP, RBP
; ED:       F8               CLC
; EE:       5D               POP RBP
; EF:       C3               RET
; F0:       CC0F             BREAK 15                         ; Invalid argument count trap</fdefn> 
```

So... um... that string "hello world" is supposed to be associated with... [RIP-122]? OK this is confusing. Let's try something else; we'll return here again towards the end of this article.

At this point we can also introduce yet another really useful SBCL tool: **sb-kernel:get-lisp-obj-address**. I have not yet found actual documentation for this (... so I guess it's subject to change any time); I actually learned about it from [a Stack Overflow answer by *sds*](https://stackoverflow.com/a/50613440/391376), warning *against* it. I do agree: don't do this in production. It's perfectly okay to use it for playing around with things though.

Yet another tool is to actually dump out memory contents. There *might* be such a thing around, but it also doesn't take a lot of time to just write something that can dump out memory contents at a specified address.

```
 (defun hexdump-address (address)
  (let ((ptr (sb-sys:int-sap address)))
    (dotimes (row 5)
      (dotimes (col 16)
        (format t "~2,'0X " (sb-sys:sap-ref-8 ptr (+ (* row 16) col))))
      (format t "~%")))) 
```

We just turn the (numeric) address into a System Area Pointer, then iterate over offsets while printing hex numbers, inserting newlines at every 16th one. I'm sure you can come up with more elegant / featureful versions; this was very quick to write though.

So: let's construct a string:

```
 (defparameter *simple-test-string* "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa") 
```

and try finding its address:

```
 CL-USER> (sb-kernel:get-lisp-obj-address *simple-test-string*)
68821502975
CL-USER> (hexdump-address 68821502975)
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
NIL
CL-USER> 
```

Well, we're clearly onto something: 0x61 is the ASCII code for the letter "a", and this is clearly something repeating, just like the many identical letters in our test string. Also, we seem to be using 4 bytes for each character (... [UTF-32](https://en.wikipedia.org/wiki/UTF-32) I guess?) However... just looking at this pointer in hex...

```
 CL-USER> (format t "~X" 68821502975)
100614CBFF 
```

... who in their right mind would allocate a string starting at such a weird address? Sure, x86 (unlike e.g. ARM) is more tolerant to unaligned access, but... it's just plain ugly! And then what's up with the nonzero bytes occurring at offsets 1, 5, 9 etc? This is definitely not how 4-byte integers should look like.

The answer to all of this is that 0x100614CBFF is not really a pointer. It's not *just* a pointer. It's a *tagged* pointer.

In C, the compiler knows the type of arguments to functions ahead of time. If parameter 1, coming in a register as per the relevant calling convention, is a 64-bit integer, we can treat it as such; if it's a pointer to a struct, different code gets generated. However, you could pass in *any type of object* to a Lisp function; how do we generate code that can decide which one is which and can handle both?

If speed is not a concern, we can just take the "everything is an object" approach. That is, all our parameters arriving in registers are assumed to be pointers to the Generic Object Type, which has a well-defined layout, stating somewhere what *actual* type the object has. The code we generate reads this value, and depending on the actual type, can perform different operations on different object types.

But then... how about *numbers*? This is where *boxing* comes in: in order to treat the number 42 as an object, we need to put it into an object that says "this is an object of type 'number', whose value is 42". This is then stored somewhere on the heap.

Of course, this is not overly quick: instead of passing around numbers by value, we need a pointer dereference to just figure out what the actual value is... and then pack the return value into a similar box (if it's a number), allocating memory if needed, and return a pointer to it. Even if we have actual compiled code, this is still a lot slower than Just Returning a Number, the way C does.

Tagged pointers to the rescue. Instead of using an entire 64-bit register to store a pointer (... which will usually point to a well-aligned memory area, so it'll end with zero bits anyway), we can use the last few bits to specify what type this *value* is. For my SBCL here, the tag is 4 bits long, at the end of the value: it's just a single hex digit. The rest of the pointer is, well, an actual pointer.

So, if we just zero out that last hex digit on our pointer:

```
 CL-USER> (hexdump-address #x100614CBF0)
E5 00 00 00 00 00 00 00 46 00 00 00 00 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00 
```

... the result now looks a lot more reasonable. This looks like a bunch of well-aligned, smallish integers (remember that we're on x86 here, which is a little-endian architecture: **little** bits at the **end** you start with, so numbers come out backwards if we list memory addresses in increasing order, the way basically every hex dump does). The first byte is... um... something, then yet another thing that is suspiciously like double the length (remember the one-bit shift for numbers from part 1? They'll be explained soon!)...

```
 CL-USER> (format t "0x~X" (length *simple-test-string*))
0x23 
```

... and then just a bunch of 0x61s for the "a"-s.

We can even poke around in the memory area, replacing the first "a" at offset 16 with a "c":

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100614CBF0) 16) #x63)
99
CL-USER> *simple-test-string*
"caaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
CL-USER> 
```

... or making it shorter by editing the length at offset 8:

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100614CBF0) 8) #x5)
5
CL-USER> *simple-test-string*
"caa" 
```

(... with which we might or might have broken the memory allocator if it ever tries to free this, but we do not particularly care at this point.)

This is the point where having compiled SBCL for yourself becomes fairly useful. (It's really just a git clone followed by running make.sh, if you already have SBCL installed; it took me 4 minutes to run on a decidedly-not-recent laptop.) Namely, after compilation, let's look at **src/runtime/genesis/constants.h** in the SBCL source code. This is a generated piece of code, basically explaining the SBCL runtime (written in C) just about to be built how Lisp objects look like. In case you don't have it around, here is some of the relevant parts:

```
 #define SBCL_GENESIS_CONSTANTS
#define FIXNUM_TAG_MASK 1 /* 0x1 */
#define N_FIXNUM_TAG_BITS 1 /* 0x1 */
#define N_LOWTAG_BITS 4 /* 0x4 */
#define N_WIDETAG_BITS 8 /* 0x8 */
#define N_WORD_BYTES 8 /* 0x8 */
#define LOWTAG_MASK 15 /* 0xF */
#define N_WORD_BITS 64 /* 0x40 */
#define WIDETAG_MASK 255 /* 0xFF */
#define SHORT_HEADER_MAX_WORDS 32767 /* 0x7FFF */

#define EVEN_FIXNUM_LOWTAG 0 /* 0x0 */
#define OTHER_IMMEDIATE_0_LOWTAG 1 /* 0x1 */
#define PAD0_LOWTAG 2 /* 0x2 */
#define INSTANCE_POINTER_LOWTAG 3 /* 0x3 */
#define PAD1_LOWTAG 4 /* 0x4 */
#define OTHER_IMMEDIATE_1_LOWTAG 5 /* 0x5 */
#define PAD2_LOWTAG 6 /* 0x6 */
#define LIST_POINTER_LOWTAG 7 /* 0x7 */
#define ODD_FIXNUM_LOWTAG 8 /* 0x8 */
#define OTHER_IMMEDIATE_2_LOWTAG 9 /* 0x9 */
#define PAD3_LOWTAG 10 /* 0xA */
#define FUN_POINTER_LOWTAG 11 /* 0xB */
#define PAD4_LOWTAG 12 /* 0xC */
#define OTHER_IMMEDIATE_3_LOWTAG 13 /* 0xD */
#define PAD5_LOWTAG 14 /* 0xE */
#define OTHER_POINTER_LOWTAG 15 /* 0xF */ 
```

This describes fairly well what we've seen so far. The "lowtag" is those 4 bytes at the end of Lisp objects; also, we're clearly on a 64-bit architecture here, with N_WORD_BITS being 64, 8 of them reserved for the lowtag. It also details various values for the lowtag, the important one being...

```
 #define OTHER_POINTER_LOWTAG 15 /* 0xF */ 
```

... which is exactly that 0xF we've been seeing at the end of our string pointers. Apparently, a lot of things are "other pointer"-s.

One exception is function pointers (0xB):

```
 CL-USER> (format t "0x~X" (sb-kernel:get-lisp-obj-address #'testfunc3))
0x10060508AB 
```

which, again, matches well with the values in the header. Same for lists (0x7):

```
 CL-USER> (format t "0x~X" (sb-kernel:get-lisp-obj-address (list 'a 'b)))
0x10043CB067 
```

Also, here comes the explanation for numbers. SBCL stores fixnums in the exact same registers as pointers; to keep the most precision possible, we just interpret anything ending with a 0 bit as a number (... shifting the actual number bits to the left once). Meanwhile, if the last bit is 1, we can look at the rest of the final 4 bits to figure out what kind of pointer we're dealing with.

**Update 2024/04/22:** there is a [discussion on Hacker News](https://news.ycombinator.com/item?id=40115083)! (Also, if you're looking for related articles, you might also like [Common Lisp for shell scripting](/2021/lisp_scripting/)).

(also, maybe this should have a "part 2 at some point.)