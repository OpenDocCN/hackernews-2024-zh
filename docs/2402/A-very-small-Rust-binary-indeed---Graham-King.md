<!--yml
category: 未分类
date: 2024-05-27 14:58:12
-->

# A very small Rust binary indeed · Graham King

> 来源：[https://darkcoding.net/software/a-very-small-rust-binary-indeed/](https://darkcoding.net/software/a-very-small-rust-binary-indeed/)

*Updated 2022-10-30*

How small can we make an x86_64 Linux Rust binary? Can it compete with a pure assembly program? Let’s find out! On the way we’ll learn things about how programs are loaded on Linux and appreciate how flexible Rust can be.

## Starting point: 3.6 MiB

Create a project: `cargo new --vcs=none smallrs`

Replace `src/main.rs` with the simplest possible Rust program:

```
fn main() {
    std::process::exit(42)
} 
```

You might argue that this isn’t the simplest possible Rust program: `fn main() { }` would be simpler. You’d be right, but only because of what the Rust runtime is doing for you.

As you will know from [studying your Kerrisk](https://amzn.to/3bLdcYg) there are only two ways a program can terminate:

*   being killed by a signal,
*   requesting it’s own termination by calling [exit](https://man7.org/linux/man-pages/man2/exit.2.html).

If you don’t exit the CPU will try running instructions past the end of your program, hit an invalid instruction, and kill your program.

Why don’t you have to do this in Rust? The default return type of `fn main() {}`, or any function that doesn’t provide a return type, is `()`. The [Termination trait](https://doc.rust-lang.org/std/process/trait.Termination.html) is implemented for `()`, which makes your [main function default to returning SUCCESS](https://github.com/rust-lang/rust/blob/2807f28de550fb6074dc4fb2f3099865de01bc1e/library/std/src/process.rs#L2152-L2157).

That SUCCESS return value gets picked up by the [Rust runtime](https://github.com/rust-lang/rust/blob/91ffbc43b18842594adb997c8eea8c51035bf0e1/library/std/src/rt.rs#L139). Something must then make the `exit` syscall with that value, either rustc or LLVM, but let’s not go *too* deep. By the time we’re done, all this machinery will be gone anyway.

Build that program, confirm it works, and check the size:

```
$ cargo build --release
   Compiling smallrs v0.1.0 (/home/graham/src/smallrs)
    Finished release [optimized] target(s) in 0.35s

$ ./target/release/smallrs
$ echo $?
42

$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 3.6M Jul  1 09:10 target/release/smallrs 
```

Three and a half megabytes! That’s not good at all. Let’s do better.

## The biggest gain is strip: 300 KiB

The easiest and largest gain is simply to `strip` the symbols from the binary. You can do this manually:

```
$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 3.6M Jul  1 09:10 target/release/smallrs

$ strip target/release/smallrs

$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 303K Jul  1 09:50 target/release/smallrs 
```

or `cargo` can do it for you (which is better). Add this to your `Cargo.toml`:

```
[profile.release]
strip = true 
```

You were probably going to forget to do this before release, so now you won’t. Easy win.

## Easy but modest gains: 260 KiB

If you search for advice online on shrinking a Rust binary you’ll find these easy changes which aren’t particularly relevant to us here, but let’s add them anyway. In a real program they will probably help.

All of these go in the `[profile.release]` section of `Cargo.toml`.

No gain:

```
opt-level = "z"
codegen-units = 1 
```

Small benefits:

```
panic = "abort"
lto = true 
```

The first line simplifies panic handling and gains us about 8k. The second line enables link-time optimization, gains us about 30 KiB, but slows the build down.

Most projects will be use the standard library (`std`), and so will have to stop here. A final thing you can try is [re-building the standard library](https://doc.rust-lang.org/cargo/reference/unstable.html#build-std), which may allow some optimizations. Remove `panic = "abort"` ([because](https://github.com/rust-lang/wg-cargo-std-aware/issues/29)) and try this:

> cargo build –release -Z build-std –target x86_64-unknown-linux-gnu

It doesn’t help us here, and **where we’re going, there are no standard libraries**.

## libc instead of the standard library: 16 KiB

Removing the Rust standard library (`std::*`) will get us our second biggest gain after `strip`.

[The std::process::exit function](https://github.com/rust-lang/rust/blob/4e808f87ccb706d339c9ea10c3c9a9c9fd7fc6cb/library/std/src/sys/unix/os.rs#L644-L646) just calls libc:

```
pub fn exit(code: i32) -> ! {
    unsafe { libc::exit(code as c_int) }
} 
```

Let’s drop the standard library and use libc directly. First add a dependency on `libc` in `Cargo.toml`:

```
[dependencies]
libc = { version = "0.2", default-features = false } 
```

Intuitively I thought this dependency would make the program *larger* but that’s not the case. The libc crate is a wrapper which contains mostly function definitions. The code itself is dynamically linked.

Then change `src/main.rs` to this:

```
#![no_std]
#![no_main]

extern crate libc;

#[no_mangle]
pub extern "C" fn main(_argc: i32, _argv: *const *const u8) -> i32 {
	// Similar to previous version, but unneccessary:
	// unsafe { libc::exit(42) }

    42
}

#[panic_handler]
fn my_panic(_info: &core::panic::PanicInfo) -> ! {
    loop {}
} 
```

We stopped using the standard library (`#![no_std]`) which forces two changes on us:

*   Rust needs to know which function to call when something panics, and [the default one](https://github.com/rust-lang/rust/blob/ef9b49881ba99248b68dbdebbebd50155587c509/library/std/src/panicking.rs#L531) is in the standard library so we have to provide our own.
*   `#![no_std]` in a binary always seems to imply `#![no_main]`. As we’ll see in a minute programs don’t actually start at `main`. There’s a fair bit of libc and rust code between the start of the program and the normal Rust `fn main` being called. Most of that machinery is in the standard library, which we no longer have, so we have to provide an earlier entry point.

What does that get us?

```
$ cargo build --release
   Compiling libc v0.2.126
   Compiling smallrs v0.1.0 (/home/graham/src/smallrs)
    Finished release [optimized] target(s) in 0.91s
$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 16K Jul  1 10:47 target/release/smallrs 
```

That’s a huge improvement, **but is it the end? Of course not!**

## How small should it be? The assembler version is 352 bytes.

Here’s the same program using [nasm](https://nasm.us/). Save it to `exit.s`:

```
section .text
global _start
_start:
        mov edi, 42  ; return code 42
        mov eax, 60  ; `_exit` syscall
        syscall 
```

Assemble and link it:

```
$ nasm -f elf64 exit.s
$ ld -n -N --strip-all -o exit exit.o 
```

Check it’s size:

```
$ ls -alh exit
-rwxr-xr-x 1 graham graham 352 Jul  1 10:59 exit 
```

352 bytes! Now we’re talking! Let’s try to get closer to that.

Notice that there’s no `main` function in our assembly version. The first 64 bytes of a Linux binary are the [ELF header](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format#File_header). When Linux loads a file it looks at the `e_entry` field of that ELF header, jumps to that address and start decoding the bytes there assuming they are CPU instructions.

Re-link `exit` with symbols:

```
$ ld -n -N -o exit exit.o 
```

Find the entry point:

```
$ readelf -h exit | grep 'Entry point'
  Entry point address:               0x400080 
```

Find the maching symbol:

```
$ nm exit | grep 400080
0000000000400080 T _start 
```

That matches the `_start` symbol in our assembly. Calling it `_start` is just a convention and default, you can tell the linker to start anywhere.

There’s no machinery here - we jump straight to our code. By contrast in our latest Rust version there is a `_start` provided by libc (I think), which calls `__rt_lang_start`, which calls our C-style `main` function (`rt` here stands for “runtime”, C does indeed have a small runtime).

In the first version of our code there were many more layers because the C runtime calls the Rust runtime ([all of this](https://github.com/rust-lang/rust/blob/91ffbc43b18842594adb997c8eea8c51035bf0e1/library/std/src/rt.rs#L139) linked earlier)) which calls our `main`.

In Rust can we provide our own `_start` and go straight there? I was delighted to discover **you very much can**.

## No libc either: 13 KiB

At the very beginning I said that a program has to call `exit`, and that usually we don’t have to worry about that because the runtime takes care of it for. Well, we’re not going to have a runtime. Nothing is going to nicely wrap our `main` function and turn the return value into the program’s exit code. We’ll have to do it ourselves in assembly, using the same code as `exit.s`.

Remove the `libc` dependency from `Cargo.toml`. Then replace `src/main.rs` with this:

```
#![no_std]
#![no_main]

use core::arch::asm;

#[no_mangle]
pub extern "C" fn _start() -> ! {
    unsafe {
        asm!(
            "mov edi, 42",
            "mov eax, 60",
            "syscall",
            options(nostack, noreturn)
        )
        // nostack prevents `asm!` from push/pop rax
        // noreturn prevents it putting a 'ret' at the end
        //  but it does put a ud2 (undefined instruction) instead
    }
}

#[panic_handler]
fn my_panic(_info: &core::panic::PanicInfo) -> ! {
    loop {}
} 
```

We will need to tell the C compiler that we’re providing our own entry point, telling it not to include it’s own start files.

```
RUSTFLAGS="-Ctarget-cpu=native -Clink-args=-nostartfiles" cargo build --release 
```

Note the `target-cpu=native` is not necessary here. (Except it is! I paid for AVX-512, dammit, I expect you to use it!)

Let’s check how we’re doing for size:

```
$ ls -alh ./target/release/smallrs
-rwxr-xr-x 2 graham graham 13K Jul  1 11:52 ./target/release/smallrs 
```

The gains are modest because the C runtime is very small.

We should be able to do better. The reason we can’t is that, **all along, we have been the victims of a dastardly [sabotage](https://www.youtube.com/watch?v=z5rRZdiu1UE)**.

## Linker flags: 400 bytes

Open up `target/release/smallrs` in a hex editor (I like [hexyl](https://github.com/sharkdp/hexyl)) and take a look. What do you see? Page and pages of zeros, that’s what!

The linker, in it’s wisdom, has been page-aligning the sections of our binary. The zeros fill space right before 0x1000 (4k) and 0x3000 (4k * 3). Normally this makes a lot of sense, we want our code to fit into as few 4k pages as possible; but not in this case!

Why didn’t this happen with our assembly version? Because I cheated, that’s why, by passing `ld` the `-n` and `-N` flags, which switch off the page aligning. Let’s do that here also:

```
$ RUSTFLAGS="-Ctarget-cpu=native -Clink-args=-nostartfiles -Clink-args=-Wl,-n,-N,--no-dynamic-linker" cargo build --release
  Compiling smallrs v0.1.0 (/home/graham/src/smallrs)
   Finished release [optimized] target(s) in 0.24s
$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 1.3K Jul  1 12:12 target/release/smallrs 
```

Now that’s a lot better!

I’m not sure why I need `--no-dynamic-linker` here. Presumably the dynamic linker expects sections to be page aligned. We’re a static binary with no dependencies, so it’s not a problem.

Let’s compare the assembly binary to ours to find the remaining differences:

```
$ file target/release/smallrs
target/release/smallrs: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
static-pie linked, BuildID[sha1]=a7be8902583c68d08b22bff637461720db80a1cd, stripped

$ file ../asm-test/exit
../asm-test/exit: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
statically linked, not stripped 
```

The `static-pie` means it’s a [Position Independent Executable](https://en.wikipedia.org/wiki/Position-independent_code#Position-independent_executables) which is a security feature to prevent a type of buffer overflow attack. It’s enabled by default basically everywhere these days. We don’t accept untrusted user input, so we’ll disable it.

(Aside: There might be a better way of disabling PIE than the linker flag I use. The rustc linker code mentions `LinkOutputKind::StaticNoPicExe` which I think is what we want, but I couldn’t figure out how to set that in Cargo.toml)

The `BuildID` is inserted by the linker to uniquely identify the file. I don’t know what it’s for, but it does not bring joy. The best argument I could find online for it was that it might help when analysing core files. Out it goes.

RUSTFLAGS will have:

*   -Ctarget-cpu=native
*   -Clink-args=-nostartfiles
*   -Clink-args=-Wl,-n,-N,–no-dynamic-linker,–no-pie,–build-id=none

Here is our final build command:

```
$ RUSTFLAGS="-Ctarget-cpu=native -Clink-args=-nostartfiles -Clink-args=-Wl,-n,-N,--no-dynamic-linker,--no-pie,--build-id=none" cargo build --release
   Compiling smallrs v0.1.0 (/home/graham/src/smallrs)
    Finished release [optimized] target(s) in 0.24s

$ ls -alh target/release/smallrs
-rwxr-xr-x 2 graham graham 400 Jul  1 12:31 target/release/smallrs 
```

Four hundred bytes!

The next steps would be to figure our where the extra almost 50 bytes is coming from, but I’m going to call it good enough and have lunch.

## Update: Bigger programs

That’s as far as we can go with such a simple example. In a more real-world setup we can get this far and our binary still isn’t small enough (I’m going to use [Demeter Deploy](https://github.com/grahamking/demeter-deploy) as the example, but you don’t need it to follow along), so I’m adding some notes for reducing the size of more complex programs.

The most helpful activity is monitoring the binary size after every change. A quite small Rust change can pull in lots of library code and bloat the binary.

### Don’t panic

Doing anything that can panic will pull in the panic machinery, which pulls in the voluminous print/format machinery. [Indexing is where I noticed this](https://github.com/grahamking/demeter-deploy/blob/b2892473313d9769793cbfd3aa19a6c5c79fa451/seed/src/main.rs#L525):

*   BIG: `my_thing[idx]`, gives a binary of 7608 bytes.
*   SMALL: `my_thing.get_unchecked(idx)`, with that single change the binary is now 2976 bytes.

This only applies if the overflow check is included. If the compiler is able to elide the size check (typically because you do it yourself: `if x < arr.len() { arr[x] }`) then indexing is fine.

### Avoid fat pointers

A slice is a fat pointer containing the address of the backing array and it’s size. An array is just the data itself. If we can avoid storing slices it saves two `usize` (16 bytes). For example in [this ERRS array](https://github.com/grahamking/demeter-deploy/blob/0115ab369179cdbe6d155da9261d3cc36635c2a8/seed/src/main.rs#L94) I gained about 200 bytes by using `&[u8; 8]` instead of `&[u8]`.

A `&str` is also a fat pointer, also 16 bytes. By storing only the address of a null terminated string we can cut that in half, for example in [these error messages](https://github.com/grahamking/demeter-deploy/blob/0115ab369179cdbe6d155da9261d3cc36635c2a8/seed/src/main.rs#L66-L75).

### Remove sections

LLVM will include more ELF sections than necessary. List these with `readelf -S`:

```
$ readelf -W -S target/release/seed
There are 9 section headers, starting at offset 0xa50:

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        00000000004000e8 0000e8 00050c 00 WAX  0   0  4
  [ 2] .rodata           PROGBITS        0000000000400600 000600 0001f4 00   A  0   0 16
  [ 3] .eh_frame_hdr     PROGBITS        00000000004007f4 0007f4 00001c 00   A  0   0  4
  [ 4] .eh_frame         X86_64_UNWIND   0000000000400810 000810 0000b0 00   A  0   0  8
  [ 5] .data.rel.ro      PROGBITS        00000000004008c0 0008c0 000120 00  WA  0   0  8
  [ 6] .got              PROGBITS        00000000004009e0 0009e0 000008 00  WA  0   0  8
  [ 7] .got.plt          PROGBITS        00000000004009e8 0009e8 000018 08  WA  0   0  8
  [ 8] .shstrtab         STRTAB          0000000000000000 000a00 00004c 00      0   0  1 
```

Compared to a pure assembly version of the same program we have many more sections here. Here’s how we’re going to remove them:

*   `.eh_frame_hdr`: Loader flag `--no-eh-frame-hdr`
*   `.eh_frame`: Manually with `objcopy`.
*   `.data.rel.ro`: cargo flag `-Crelocation-model=static`
*   `.got`: We need that. My guess is because Rust’s `core` lib depends on `memcpy`, `memset`, etc, so it must find those symbols (we provide them), but I’m not sure.
*   `.got.plt`: Manually with `objcopy`.

This gives our final final RUSTFLAGS of:

```
export RUSTFLAGS="-Ctarget-cpu=native -Clink-args=-nostartfiles -Crelocation-model=static -Clink-args=-Wl,-n,-N,--no-dynamic-linker,--no-pie,--build-id=none,--no-eh-frame-hdr" 
```

and a post-compile step of:

```
objcopy -R .eh_frame -R .got.plt target/release/my_bin my_final_bin 
```

Note that technically the `.eh_frame` is required by the ABI. As long as we don’t have any exceptions to handle no-one will notice.

## Conclusion

**We went from 3.6 MiB to 400 bytes**. Rust, I am impressed. This is a true systems programming language.

Note that even though our `_start` function here only contains a bit of assembly this is just to make a simple example. We still have access to all of the regular Rust language (including of course the borrow checker and so on) and all of [core](https://doc.rust-lang.org/core/index.html). That means Option, Result, time::Duration, sync::atomic, and lots more. Rust is used with no_std in embedded programming.

Aside from embedded, are there practical applications? Yes. I recently ported [Demeter Deploy](https://github.com/grahamking/demeter-deploy)’s remote helper (called `seed`) from Assembly to Rust, and, wait for it, thanks to all of the above it compiles to **exactly the same size**. That’s incredible.

And because I seem to end all my recent blog posts with a link to a Happy Hardcore tune, here is what Darren Styles from old-school team Force & Styles is up to these days: [Darren Styles - Hard Generation](https://www.youtube.com/watch?v=6QYZkAdFb_Q&t=206s). You’ll need a standing desk for this one.