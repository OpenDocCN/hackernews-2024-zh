<!--yml
category: 未分类
date: 2024-05-27 15:03:56
-->

# Self-contained Linux applications with lone lisp

> 来源：[https://www.matheusmoreira.com/articles/self-contained-lone-lisp-applications](https://www.matheusmoreira.com/articles/self-contained-lone-lisp-applications)

# Self-contained Linux applications with lone lisp

I started [the lone lisp project](https://github.com/lone-lang/lone) to create a lisp language and environment exclusively for Linux. I built into it support for arbitrary [Linux system calls](/articles/linux-system-calls) so that it would be possible to implement any program without need for any external dependencies and so that every Linux feature would be available to programmers.

Although far from ready for production use, I have achieved a significant milestone in the development of the language: it is now possible to create self-contained, standalone, freestanding, redistributable Linux applications written entirely in lisp.

Lisp code can now be embedded directly into a copy of the lone interpreter which may then be copied to and run on any Linux system of the same architecture, unmodified and without any external dependencies. The application is limited only by the system calls it uses: newer system calls will naturally require newer kernels.

In this article I will demonstrate this capability, explain how it works and the journey to implementing it. Every script bundling tool I've ever seen unpacks the code to some file system location and then reads it back in. I came up with a different solution.

## A simple implementation of `env` in lone lisp

The `env` utility is among the simplest programs available in Unix-like operating systems. It's most fundamental function is to print the user's environment. That function can be trivially implemented in lone lisp:

```
(import (lone print) (linux environment))

(print environment) 
```

Running this simple program produces a table of environment variables and their values:

```
$ ./lone < env.ln
{ "HOME" "/home/matheus" "EDITOR" "vim" … } 
```

### Embedding `env.ln` into the interpreter

In order to transform `env.ln` into a self-contained `env` program, the lone lisp code must be embedded into a copy of the interpreter. This can be achieved by the purpose-built `lone-embed` tool:

```
$ cp lone env
$ lone-embed env env.ln 
```

The interpreter will then seamlessly load and execute the code:

```
$ ./env
{ "HOME" "/home/matheus" "EDITOR" "vim" … } 
```

## Elven segments

Tracing the system calls of the application with `strace` reveals an interesting property:

```
$ strace env
execve("env", ["env"], 0x7fe9752d40 /* 31 vars */) = 0
write(1, "{ ", 2)                                  = 2
write(1, "\"", 1)                                  = 1
write(1, "HOME", 4)                                = 4
write(1, "\"", 1)                                  = 1
write(1, " ", 1)                                   = 1
… 
```

It begins writing its output immediately. There was no need for it to read from the file system. The code must be somewhere else.

```
$ xxd env | tail -n7
00032fe0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00032ff0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00033000: 7b20 7275 6e20 2830 202e 2036 3329 207d  { run (0 . 63) }
00033010: 0a28 696d 706f 7274 2028 6c6f 6e65 2070  .(import (lone p
00033020: 7269 6e74 2920 286c 696e 7578 2065 6e76  rint) (linux env
00033030: 6972 6f6e 6d65 6e74 2929 0a0a 2870 7269  ironment))..(pri
00033040: 6e74 2065 6e76 6972 6f6e 6d65 6e74 290a  nt environment). 
```

It's at the very end of the executable itself, at an aligned offset. Through some elven magic the interpreter is reflecting upon its own contents at runtime. It's loading the code from inside itself and evaluating it. Without using any system calls to do it.

The source of this magic is the ELF segment.

```
$ readelf --segments env | grep 33000
LOAD           0x0000000000033000 0x0000000000312000 0x0000000000312000
LOOS+0xc6f6e65 0x0000000000033000 0x0000000000312000 0x0000000000312000 
```

ELF segments, also called program headers, describe the memory image of the program. There are many types of segments but especially interesting are the `LOAD` segments which tell Linux which parts of the file must be mapped to which addresses in order for the program to work correctly.

The `lone-embed` tool copies the lisp code into the ELF and then creates a `LOAD` segment for it. Linux then maps in the embedded code automatically at load time before the interpreter has even started.

### Finding the segment

The code will be in memory but its location and size are unknown. It could be anywhere inside the vastness of virtual address space. To that problem Linux provides an ingenious solution: it tells us where it is.

In addition to argument and environment vectors, processes receive the auxiliary vector. It's essentially a null terminated array of key-value pairs of various types and it's placed right there on the stack just after the environment vector.

```
struct lone_auxiliary_value {
    union {
        char *c_string;
        void *pointer;
        long integer;
        unsigned long unsigned_integer;
    } as;
};

struct lone_auxiliary_vector {
    long type;
    struct lone_auxiliary_value value;
} *auxv;

void **pointer = (void **) envp;
while (*pointer++ != 0);
auxv = (struct auxiliary_vector *) pointer; 
```

Through this mechanism, Linux passes various bits of useful information to programs. These include things like processor architecture and capabilities, current user and group IDs, some random bytes, the location of the [vDSO](https://www.man7.org/linux/man-pages/man7/vdso.7.html), the system's page size...

And the location of the program header table.

```
struct lone_auxiliary_value lone_auxiliary_vector_value(struct lone_auxiliary_vector *auxiliary, long type)
{
    for (; auxiliary->type != AT_NULL; ++auxiliary)
        if (auxiliary->type == type)
            return auxiliary->value;

    return (struct lone_auxiliary_value) { .as.integer = 0 };
}

struct lone_elf_segments lone_auxiliary_vector_elf_segments(struct lone_auxiliary_vector *auxv)
{
    return (struct lone_elf_segments) {
        .entry_size  = lone_auxiliary_vector_value(auxv, AT_PHENT).as.unsigned_integer,
        .entry_count = lone_auxiliary_vector_value(auxv, AT_PHNUM).as.unsigned_integer,
        .segments    = lone_auxiliary_vector_value(auxv, AT_PHDR).as.pointer
    };
} 
```

The table is just a contiguous array of [ELF program header structures](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format#Program_header). Given this pointer, all the program has to do is scan the table and find the correct entry. One does not simply search for `LOAD` entries, however. Attempting to do it uncovers a couple of problems.

The first problem is the fact there are lots of these loadable segments and they're all indistinguishable from one another. ELF sections have unique names for identification, program headers have nothing.

The second problem is their alignment requirements. The addresses and sizes are usually aligned to page boundaries. This obfuscates the true size of the data they contain.

#### The lone segment

In order to solve this, I created my own custom segment type: the `LONE` segment.

ELF provides an incredibly generous numeric range for operating system-specific segments. All the values between `LOOS` and `HIOS` are free for operating systems to allocate. Those definitions represent a range of integers between `0x60000000` and `0x6FFFFFFF` inclusive. That's 268,435,455 magic numbers.

So I just picked one. That's what the `LOOS+0xc6f6e65` means. Spelled out `lone` in ASCII and it just worked itself out.

```
 #define PT_LONE 0x6c6f6e65 
```

I figured if GNU can do it then I can do it too.

The `LONE` segment is not loadable and thus does not have any alignment requirements, allowing it to describe the embedded segment exactly. It also serves as a magic number which makes it trivial to search for it in the program header table. Once found, it contains all the information lone needs to load and execute the code.

```
typedef Elf64_Phdr lone_elf_segment; 
typedef Elf32_Phdr lone_elf_segment; 

lone_elf_segment *lone_auxiliary_vector_embedded_segment(struct lone_auxiliary_vector *auxv)
{
    struct lone_elf_segments table = lone_auxiliary_vector_elf_segments(auxv);

    for (size_t i = 0; i < table.entry_count; ++i) {
        lone_elf_segment *entry = &table.segments[i];

        if (entry->p_type == PT_LONE)
            return entry;
    }

    return 0;
}

lone_elf_segment *segment = lone_auxiliary_vector_embedded_segment(auxv);
struct lone_bytes data;
data.count = segment->p_memsz;
data.pointer = (unsigned char *) segment->p_vaddr; 
```

The interpreter now has the address and size of the data embedded into its own executable. At this point it's smooth sailing.

All that's left to do is to make sense of the bytes. I chose to simply put a descriptor object in there and have the interpreter read it in. Seemed like the simplest possible solution.

```
{ run (0 . 63) } 
```

Just a good old hash table in lone lisp syntax. The run key contains the offset and size of the code the interpreter should run, relative to the end of the descriptor object. It just reads in that slice and evaluates it.

Since it's just a normal hash table, it's also infinitely extensible with arbitrary schemas. I plan to implement a modules key to contain an arbitrary number of lone modules so that programs can statically link their libraries into the lone interpreter. All I gotta do is place the embedded segment into the module search path, before all the others. I suppose I could also allow configuring the interpreter via the descriptor object, eliminating the need for command line switches.

## Special linker features

Remember the `lone-embed` tool which I briefly mentioned at the beginning of the articled? It's an ELF patching tool which I built specifically for this purpose and it's doing *a lot of* heavy lifting here.

When programs are compiled and linked, the program headers are set in stone. Yet to do all of this I needed to append new segments to the program header table. This turned out to be *much* harder than anticipated.

The program header table usually follows the ELF header and precedes the contents of the file. Appending new items to it would require resizing the table, which would require shifting up the addresses of *everything* that comes after it, invalidating pointers to the old addresses. As far as I can tell, it just can't be done without reinventing the linker itself.

I tried to move the table to the end of the file instead but couldn't get that to work either. My program was somehow segfaulting before it even reached the entry point, gdb was useless, I couldn't understand what was going on and was reduced to desperately dumping `readelf` output on stackoverflow in hopes someone would spot the problem. Well *someone* did and quickly at that but clearly this was not a sustainable software development strategy.

### The `mold` linker saves the day

The linker is in a privileged position. It knows everything there is to know about the program's memory layout and can easily add new ELF segments to it seemingly at will. If a solution exists, I was convinced it would be found in the linker.

So I started learning [linker script](https://sourceware.org/binutils/docs/ld/Scripts.html). Turns out it has a `PHDRS` command which is exactly what I needed but I couldn't figure out how to use it. I just kept getting "unable to allocate headers" errors no matter what I tried. I concluded it would be easier to simply *ask* for this feature instead.

So I [emailed](https://sourceware.org/pipermail/binutils/2023-November/130640.html) the GNU binutils mailing list... Then I created an [LLVM issue](https://github.com/llvm/llvm-project/issues/72386)...

Then I opened a `mold` issue. The maintainer immediately understood what I wanted to do and [made it happen](https://github.com/rui314/mold/commit/eb6c213f2a9aa8a101b2b52a791be369d165e6a9) with what was essentially a single line of code change. Just *beyond awesome*.

I waited eagerly for the new release but got so excited I built this huge C++ project from source *on my smartphone* just to integrate lone with it. All I had to do was put `-Wl,--spare-program-headers,2` in the `LDFLAGS`. It gave me two `NULL` program headers for my patching utility to edit any way it wanted. It worked *perfectly*.

So far `mold` is the only linker with this feature and it's absolutely required for `lone-embed` to work. It will outright refuse to patch the ELF if it doesn't find at least two `NULL` program headers in it.

Would be nice if the others gained this feature too. Unless I can figure out a way to move the program header table to the end of the file without breaking everything, I'm pretty much locked into using `mold`. Well, I don't really mind. It's an awesome linker *and* it's free software. I'm OK with this!