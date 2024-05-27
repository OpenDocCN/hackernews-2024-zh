<!--yml
category: 未分类
date: 2024-05-27 13:17:45
-->

# The Rust Calling Convention We Deserve · mcyoung

> 来源：[https://mcyoung.xyz/2024/04/17/calling-convention/](https://mcyoung.xyz/2024/04/17/calling-convention/)

I will often say that the so-called “C ABI” is a very bad one, and a relatively unimaginative one when it comes to passing complicated types effectively. A lot of people ask me “ok, what would you use instead”, and I just point them to the [Go register ABI](https://go.googlesource.com/go/+/refs/heads/dev.regabi/src/cmd/compile/internal-abi.md), but it seems most people have trouble filling in the gaps of what I mean. This article explains what I mean in detail.

I have discussed [calling conventions](https://mcyoung.xyz//2021/11/09/assembly-1/#the-calling-convention) in the past, but as a reminder: the *calling convention* is the part of the ABI that concerns itself with how to pass arguments to and from a function, and how to actually call a function. This includes which registers arguments go in, which registers values are returned out of, what function prologues/epilogues look like, how unwinding works, etc.

This particular post is primarily about x86, but I intend to be reasonably generic (so that what I’ve written applies just as well to ARM, RISC-V, etc). I will assume a general familiarity with x86 assembly, LLVM IR, and Rust (but not rustc’s internals).

Today, like many other natively compiled languages, Rust defines an unspecified0- calling convention that lets it call functions however it likes. In practice, Rust lowers to LLVM’s built-in C calling convention, which LLVM’s prologue/epilogue codegen generates calls for.

Rust is fairly conservative: it tries to generate LLVM function signatures that Clang could have plausibly generated. This has two significant benefits:

1.  Good probability debuggers won’t choke on it. This is not a concern on Linux, though, because DWARF is very general and does not bake-in the Linux C ABI. We will concern ourselves only with ELF-based systems and assume that debuggability is a nonissue.

2.  It is less likely to tickle LLVM bugs due to using ABI codegen that Clang does not exercise. I think that if Rust tickles LLVM bugs, we should actually fix them (a very small number of rustc contributors do in fact do this).

However, we are too conservative. We get terrible codegen for simple functions:

```
fn extract(arr: [i32; 3]) -> i32 {
  arr[1]
}
```

```
extract:
  mov   eax, dword ptr [rdi + 4]
  ret
```

`arr` is 12 bytes wide, so you’d think it would be passed in registers, but no! It is passed by pointer! Rust is actually *more* conservative than what the Linux C ABI mandates, because it actually passes the `[i32; 3]` in registers when `extern "C"` is requested.

```
extern "C" fn extract(arr: [i32; 3]) -> i32 {
  arr[1]
}
```

```
extract:
  mov   rax, rdi
  shr   rax, 32
  ret
```

The array is passed in `rdi` and `rsi`, with the `i32`s packed into registers. The function moves `rdi` into `rax`, the output register, and shifts the upper half down.

Not only does clang produce patently *bad* code for passing things by value, but it also knows how to do it better, if you request a standard calling convention! We could be generating *way* better code than Clang, but we don’t!

Hereforth, I will describe how to do it.

Let’s suppose that we keep the current calling convention for `extern "Rust"`^(, but we add a flag `-Zcallconv` that sets the calling convention for `extern "Rust"` when compiling a crate. The supported values will be `-Zcallconv=legacy` for the current one, and `-Zcallconv=fast` for the one we’re going to design. We could even let `-O` set `-Zcallconv=fast` automatically.)

Why keep the old calling convention? Although I did sweep debugability under the rug, one nice property `-Zcallconv=fast` will not have is that it does not place arguments in the C ABI order, which means that a reader replying on the “Diana’s silk dress cost $89” mnemonic on x86 will get fairly confused.

I am also assuming we may not even support `-Zcallconv=fast` for some targets, like WASM, where there is no concept of “registers” and “spilling”. It may not even make sense to enable it for for debug builds, because it will produce much worse code with optimizations turned off.

There is also a mild wrinkle with function pointers, and `extern "Rust" {}` blocks. Because this flag is per-crate, even though functions can advertise which version of `extern "Rust"` they use, function pointers have no such luxury. However, calling through a function pointer is slow and rare, so we can simply force them to use `-Zcallconv=legacy`. We can generate a shim to translate calling conventions as needed.

Similarly, we can, in principle, call any Rust function like this:

```
fn secret_call() -> i32 {
  extern "Rust" {
    fn my_func() -> i32;
  }
  unsafe { my_func() }
}
```

However, this mechanism can only be used to call unmangled symbols. Thus, we can simply force `#[no_mangle]` symbols to use the legacy calling convention.

In an ideal world, LLVM would provide a way for us to specify the calling convention directly. E.g., this argument goes in that register, this return goes in that one, etc. Unfortunately, adding a calling convention to LLVM requires writing a bunch of C++.

However, we can get away with specifying our own calling convention by following the following procedure.

1.  First, determine, for a given target triple, the maximum number of values that can be passed “by register”. I will explain how to do this below.

2.  Decide how to pass the return value. It will either fit in the output registers, or it will need to be returned “by reference”, in which case we pass an extra `ptr` argument to the function (tagged with the `sret` attribute) and the actual return value of the function is that pointer.

3.  Decide which arguments that have been passed by value need to be demoted to being passed by reference. This will be a heuristic, but generally will be approximately “arguments larger than the by-register space”. For example, on x86, this comes out to 176 bytes.

4.  Decide which arguments get passed by register, so as to maximize register space usage. This problem is NP-hard (it’s the knapsack problem) so it will require a heuristic. All other arguments are passed on the stack.

5.  Generate the function signature in LLVM IR. This will be all of the arguments that are passed by register encoded as various non-aggregates, such as `i64`, `ptr`, `double`, and `<2 x i64>`. What valid choices are for said non-aggregates depends on the target, but the above are what you will generally get on a 64-bit architecture. Arguments passed on the stack will follow the “register inputs”.

6.  Generate a function prologue. This is code to decode each Rust-level argument from the register inputs, so that there are `%ssa` values corresponding to those that would be present when using `-Zcallconv=legacy`. This allows us to generate the same code for the body of the function regardless of calling convention. Redundant decoding code will be eliminated by DCE passes.

7.  Generate a function exit block. This is a block that contains a single `phi` instruction for the return type as it would be for `-Zcallconv=legacy`. This block will encode it into the requisite output format and then `ret` as appropriate. All exit paths through the function should `br` to this block instead of `ret`-ing.

8.  If a non-polymorphic, non-inline function may have its address taken (as a function pointer), either because it is exported out of the crate or the crate takes a function pointer to it, generate a shim that uses `-Zcallconv=legacy` and immediately tail-calls the real implementation. This is necessary to preserve function pointer equality.

The main upshot here is that we need to cook up heuristics for figuring out what goes in registers (since we allow reordering arguments to get better throughput). This is equivalent to the knapsack problem; knapsack heuristics are beyond the scope of this article. This should happen early enough that this information can be stuffed into `rmeta` to avoid needing to recompute it. We may want to use different, faster heuristics depending on `-Copt-level`. Note that correctness requires that we forbid linking code generated by multiple different Rust compilers, which is already the case, since Rust breaks ABI from release to release.

Assuming we do that, how do we actually get LLVM to pass things in the way we want it to? We need to determine what the largest “by register” passing LLVM will permit is. The following LLVM program is useful for determining this on a particular version of LLVM:

```
%InputI = type [6 x i64]
%InputF = type [0 x double]
%InputV = type [8 x <2 x i64>]

%OutputI = type [3 x i64]
%OutputF = type [0 x double]
%OutputV = type [4 x <2 x i64>]

define void @inputs({ %InputI, %InputF, %InputV }) {
  %p = alloca [4096 x i8]
  store volatile { %InputI, %InputF, %InputV } %0, ptr %p
  ret void
}

%Output = { %OutputI, %OutputF, %OutputV }
@gOutput = constant %Output zeroinitializer
define %Output @outputs() {
  %1 = load %Output, ptr @gOutput
  ret %Output %1
}
```

When you pass an aggregate by-value to an LLVM function, LLVM will attempt to “explode” that aggregate into as many registers as possible. There are distinct register classes on different systems. For example, on both x86 and ARM, floats and vectors share the same register class (kind of^().)

The above values are for x86^(. LLVM will pass six integers and eight SSE vectors by register, and return half as many (3 and 4) by register. Increasing any of the values generates extra loads and stores that indicate LLVM gave up and passed arguments on the stack.)

The values for `aarch64-unknown-linux` are 8 integers and 8 vectors for both inputs and outputs, respectively.

This is the maximum number of registers we get to play with for each class. Anything extra gets passed on the stack.

I recommend that *every function* have the same number of by-register arguments. So on x86, EVERY `-Zcallconv=fast` function’s signature should look like this:

```
declare {[3 x i64], [4 x <2 x i64>]} @my_func(
  i64 %rdi, i64 %rsi, i64 %rdx, i64 %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1, <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5, <2 x i64> %xmm6, <2 x i64> %xmm7,
  ; other args...
)
```

When passing pointers, the appropriate `i64`s should be replaced by `ptr`, and when passing `double`s, they replace `<2 x i64>`s.

But you’re probably saying, “Miguel, that’s crazy! Most functions don’t pass 176 bytes!” And you’d be right, if not for the magic of LLVM’s very well-specified `poison` semantics.

We can get away with not doing extra work if every argument we do not use is passed `poison`. Because `poison` is equal to “the most convenient possible value at the present moment”, when LLVM sees `poison` passed into a function via register, it decides that the most convenient value is “whatever happens to be in the register already”, and so it doesn’t have to touch that register!

For example, if we wanted to pass a pointer via `rcx`, we would generate the following code.

```
; This is a -Zcallconv=fast-style function.
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @load_rcx(
  i64 %rdi, i64 %rsi, i64 %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1,
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  %load = load i64, ptr %rcx
  %out = insertvalue %Out poison,
                      i64 %load, 0, 0
  ret %Out %out
}

declare ptr @malloc(i64)
define i64 @make_the_call() {
  %1 = call ptr @malloc(i64 8)
  store i64 42, ptr %1
  %2 = call %Out @by_rcx(
    i64 poison, i64 poison, i64 poison,
    ptr %1,     i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %3 = extractvalue %Out %2, 0, 0
  %4 = add i64 %3, 42
  ret i64 %4
}
```

```
by_rcx:
  mov   rax, qword ptr [rcx]
  ret

make_the_call:
  push  rax
  mov   edi, 8
  call  malloc
  mov   qword ptr [rax], 42
  mov   rcx, rax
  call  load_rcx
  add   rax, 42
  pop   rcx
  ret
```

It is perfectly legal to pass poison to a function, if it does not interact with the poisoned argument in any proscribed way. And as we see, `load_rcx()` receives its pointer argument in `rcx`, whereas `make_the_call()` takes no penalty in setting up the call: loading poison into the other thirteen registers compiles down to nothing^(, so it only needs to load the pointer returned by malloc into `rcx`.)

This gives us almost total control over argument passing; unfortunately, it is not total. In an ideal world, the same registers are used for input and output, to allow easier pipelining of calls without introducing extra register traffic. This is true on ARM and RISC-V, but not x86\. However, because register ordering is merely a suggestion for us, we can choose to allocate the return registers in whatever order we want. For example, we can pretend the order registers should be allocated in is `rdx`, `rcx`, `rdi`, `rsi`, `r8`, `r9` for inputs, and `rdx`, `rcx`, `rax` for outputs.

```
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @square(
  i64 %rdi, i64 %rsi, i64 %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1,
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  %sq = mul i64 %rdx, %rdx
  %out = insertvalue %Out poison,
                      i64 %sq, 0, 1
  ret %Out %out
}

define i64 @make_the_call(i64) {
  %2 = call %Out @square(
    i64 poison, i64 poison, i64 %0,
    i64 poison, i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %3 = extractvalue %Out %2, 0, 1

  %4 = call %Out @square(
    i64 poison, i64 poison, i64 %3,
    i64 poison, i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %5 = extractvalue %Out %4, 0, 1

  ret i64 %5
}
```

```
square:
  imul rdx, rdx
  ret

make_the_call:
  push rax
  mov rdx, rdi
  call square
  call square
  mov rax, rdx
  pop rcx
  ret
```

`square` generates extremely simple code: the input and output register is `rdi`, so no extra register traffic needs to be generated. Similarly, when we effectively do `@square(@square(%0))`, there is no setup between the functions. This is similar to code seen on aarch64, which uses the same register sequence for input and output. We can see that the “naive” version of this IR produces the exact same code on aarch64 for this reason.

```
define i64 @square(i64) {
  %2 = mul i64 %0, %0
  ret i64 %2
}

define i64 @make_the_call(i64) {
  %2 = call i64 @square(i64 %0)
  %3 = call i64 @square(i64 %2)
  ret i64 %3
}
```

```
square:
  mul x0, x0, x0
  ret

make_the_call:
  str x30, [sp, #-16]!
  bl square
  ldr x30, [sp], #16
  b square  // Tail call.
```

Now that we’ve established total control on how registers are assigned, we can turn towards maximizing use of these registers in Rust.

For simplicity, we can assume that rustc has already processed the users’s types into basic aggregates and unions; no enums here! We then have to make some decisions about which portions of the arguments to allocate to registers.

First, return values. This is relatively straightforward, since there is only one value to pass. The amount of data we need to return is *not* the size of the struct. For example, `[(u64, u32); 2]` measures 32 bytes wide. However, eight of those bytes are padding! We do not need to preserve padding when returning by value, so we can flatten the struct into `(u64, u32, u64, u32)` and sort by size into `(u64, u64, u32, u32)`. This has no padding and is 24 bytes wide, which fits into the three return registers LLVM gives us on x86\. We define the *effective size* of a type to be the number of non-`undef` bits it occupies. For `[(u64, u32); 2]`, this is 192 bits, since it excludes the padding. For `bool`, this is one. For `char` this is technically 21, but it’s simpler to treat `char` as an alias for `u32`.

The reason for counting bits this way is that it permits significant compaction. For example, returning a struct full of bools can simply bit-pack the bools into a single register.

So, a return value is converted to a by-ref return if its effective size is smaller than the output register space (on x86, this is three integer registers and four SSE registers, so we get 88 bytes total, or 704 bits).

Argument registers are much harder, because we hit the [knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem), which is NP-hard. The following relatively naive heuristic is where I would start, but it can be made infinitely smarter over time.

First, demote to by-ref any argument whose effective size is larget than the total by-register input space (on x86, 176 bytes or 1408 bits). This means we get a pointer argument instead. This is beneficial to do first, since a single pointer might pack better than the huge struct.

Enums should be replaced by the appropriate discriminant-union pair. For example, `Option<i32>` is, internally, `(union { i32, () }, i1)`, while `Option<Option<i32>>` is `(union { i32, (), () }, i2)`. Using a small non-power-of-two integer improves our ability to pack things, since enum discriminants are often quite tiny.

Next, we need to handle unions. Because mucking about with unions’ uninitialized bits behind our backs is allowed, we need to either pass it as an array of `u8`, unless it only has a single non-empty variant, in which case it is replaced with that variant^.

Now, we can proceed to flatten everything. All of the converted arguments are flattened into their most primitive components: pointers, integers, floats, and bools. Every field should be no larger than the smallest argument register; this may require splitting large types such as `u128` or `f64`.

This big list of primitives is next sorted by effective size, from smallest to largest. We take the largest prefix of this that will fit in the available register space; everything else goes on the stack.

If part of a Rust-level input is sent to the stack in this way, and that part is larger than a small multiple of the pointer size (e.g., 2x), it is demoted to being passed by pointer-on-the-stack, to minimize memory traffic. Everything else is passed directly on the stack in the order those inputs were before the sort. This helps keep regions that need to be copied relatively contiguous, to minimize calls to `memcpy`.

The things we choose to pass in registers are allocated to registers in reverse size order, so e.g. first 64-bit things, then 32-bit things, etc. This is the same layout algorithm that `repr(Rust)` structs use to move all the padding into the tail. Once we get to the `bool`s, those are bit-packed, 64 to a register.

Here’s a relatively complicated example. My Rust function is as follows:

```
struct Options {
  colorize: bool,
  verbose_debug: bool,
  allow_spurious_failure: bool,
  retries: u32,
}

trait Context {
  fn check(&self, n: usize, colorize: bool);
}

fn do_thing<'a>(op_count: Option<usize>, context: &dyn Context,
                name: &'a str, code: [char; 6],
                options: Options,
) -> &'a str {
  if let Some(op_count) = op_count {
    context.check(op_count, options.colorize);
  }

  for c in code {
    if let Some((_, suf)) = name.split_once(c) {
      return suf;
    }
  }

  "idk"
}
```

The codegen for this function is quite complex, so I’ll only cover the prologue and epilogue. After sorting and flattening, our raw argument LLVM types are something like this:

```
gprs: i64, ptr, ptr, ptr, i64, i32, i32
xmm0: i32, i32, i32, i32
xmm1: i32, i1, i1, i1, i1
```

Everything fits in registers! So, what does the LLVM function look like on x86?

```
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @do_thing(
  i64 %rdi, ptr %rsi, ptr %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <4 x i32> %xmm0, <4 x i32> %xmm1,
  ; Unused.
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  ; First, unpack all the primitives.
  %r9.0 = trunc i64 %r9 to i32
  %r9.1.i64 = lshr i64 %r9, 32
  %r9.1 = trunc i64 %r9.1.i64 to i32
  %xmm0.0 = extractelement <4 x i32> %xmm0, i32 0
  %xmm0.1 = extractelement <4 x i32> %xmm0, i32 1
  %xmm0.2 = extractelement <4 x i32> %xmm0, i32 2
  %xmm0.3 = extractelement <4 x i32> %xmm0, i32 3
  %xmm1.0 = extractelement <4 x i32> %xmm1, i32 0
  %xmm1.1 = extractelement <4 x i32> %xmm1, i32 1
  %xmm1.1.0 = trunc i32 %xmm1.1 to i1
  %xmm1.1.1.i32 = lshr i32 %xmm1.1, 1
  %xmm1.1.1 = trunc i32 %xmm1.1.1.i32 to i1
  %xmm1.1.2.i32 = lshr i32 %xmm1.1, 2
  %xmm1.1.2 = trunc i32 %xmm1.1.2.i32 to i1
  %xmm1.1.3.i32 = lshr i32 %xmm1.1, 3
  %xmm1.1.3 = trunc i32 %xmm1.1.3.i32 to i1

  ; Next, reassemble them into concrete values as needed.
  %op_count.0 = insertvalue { i64, i1 } poison, i64 %rdi, 0
  %op_count = insertvalue { i64, i1 } %op_count.0, i1 %xmm1.1.0, 1
  %context.0 = insertvalue { ptr, ptr } poison, ptr %rsi, 0
  %context = insertvalue { ptr, ptr } %context.0, ptr %rdx, 1
  %name.0 = insertvalue { ptr, i64 } poison, ptr %rcx, 0
  %name = insertvalue { ptr, i64 } %name.0, i64 %r8, 1
  %code.0 = insertvalue [6 x i32] poison, i32 %r9.0, 0
  %code.1 = insertvalue [6 x i32] %code.0, i32 %r9.1, 1
  %code.2 = insertvalue [6 x i32] %code.1, i32 %xmm0.0, 2
  %code.3 = insertvalue [6 x i32] %code.2, i32 %xmm0.1, 3
  %code.4 = insertvalue [6 x i32] %code.3, i32 %xmm0.2, 4
  %code = insertvalue [6 x i32] %code.4, i32 %xmm0.3, 5
  %options.0 = insertvalue { i32, i1, i1, i1 } poison, i32 %xmm1.0, 0
  %options.1 = insertvalue { i32, i1, i1, i1 } %options.0, i1 %xmm1.1.1, 1
  %options.2 = insertvalue { i32, i1, i1, i1 } %options.1, i1 %xmm1.1.2, 2
  %options = insertvalue { i32, i1, i1, i1 } %options.2, i1 %xmm1.1.3, 3

  ; Codegen as usual.
  ; ...
}
```

Above, `!dbg` metadata for the argument values should be attached to the instruction that actually materializes it. This ensures that gdb does something halfway intelligent when you ask it to print argument values.

On the other hand, in current rustc, it gives LLVM eight pointer-sized parameters, so it winds up spending all six integer registers, plus two values passed on the stack. Not great!

This is not a complete description of what a completely over-engineered calling convention could entail: in some cases we might know that we have additional registers available (such as AVX registers on x86). There are cases where we might want to split a struct across registers and the stack.

This also isn’t even getting into what returns *could* look like. `Result`s are often passed through several layers of functions via `?`, which can result in a lot of redundant register moves. Often, a `Result` is large enough that it doesn’t fit in registers, so each call in the `?` stack has to inspect an ok bit by loading it from memory. Instead, a `Result` return might be implemented as an out-parameter pointer for the error, with the ok variant’s payload, and the is ok bit, returned as an `Option<T>`. There are some fussy details with `Into` calls via `?`, but the idea is implementable.

Now, because we’re Rust, we’ve also got a trick up our sleeve that C doesn’t (but Go does)! When we’re generating the ABI that all callers will see (for `-Zcallconv=fast`), we can look at the function body. This means that a crate can advertise the *precise* ABI (in terms of register-passing) of its functions.

This opens the door to a more extreme optimization-based ABIs. We can start by simply throwing out unused arguments: if the function never does anything with a parameter, don’t bother spending registers on it.

Another example: suppose that we know that an `&T` argument is not retained (a question the borrow checker can answer at this point in the compiler) and is never converted to a raw pointer (or written to memory a raw pointer is taken of, etc). We also know that `T` is fairly small, and `T: Freeze`. Then, we can replace the reference with the pointee directly, passed by value.

The most obvious candidates for this is APIs like `HashMap::get()`. If the key is something like an `i32`, we need to spill that integer to the stack and pass a pointer to it! This results in unnecessary, avoidable memory traffic.

Profile-guided ABI is a step further. We might know that some arguments are hotter than others, which might cause them to be prioritized in the register allocation order.

You could even imagine a case where a function takes a very large struct by reference, but three `i64` fields are very hot, so the caller can *preload* those fields, passing them both by register *and* via the pointer to the large struct. The callee does not see additional cost: it had to issue those loads anyway. However, the caller probably has those values in registers already, which avoids some memory traffic.

Instrumentation profiles may even indicate that it makes sense to duplicate whole functions, which are identical except for their ABIs. Maybe they take different arguments by register to avoid costly spills.

This is a bit more advanced (and ranty) than my usual writing, but this is an aspect of Rust that I find really frustrating. We could be doing *so much better* than C++ ever can (because of their ABI constraints). None of this is new ideas; this is *literally* how Go does it!

So why don’t we? Part of the reason is that ABI codegen is complex, and as I described above, LLVM gives us very few useful knobs. It’s not a friendly part of rustc, and doing things wrong can have nasty consequences for usability. The other part is a lack of expertise. As of writing, only a handful of people contributing to rustc have the necessary grasp of LLVM’s semantics (and mood swings) to emit the Right Code such that we get good codegen and don’t crash LLVM.

Another reason is compilation time. The more complicated the function signatures, the more prologue/epilogue code we have to generate that LLVM has to chew on. But `-Zcallconv` is intended to only be used with optimizations turned on, so I don’t think this is a meaningful complaint. Nor do I think the project’s Goodhartization of compilation time as a metric is healthy… but I do not think this is ultimately a relevant drawback.

I, unfortunately, do not have the spare time to dive into fixing rustc’s ABI code, but I do know LLVM really well, and I know that this is a place where Rust has a low bus factor. For that reason, I am happy to provide the Rust compiler team expert knowledge on getting LLVM to do the right thing in service of making optimized code faster.