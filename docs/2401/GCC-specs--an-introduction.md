<!--yml
category: 未分类
date: 2024-05-27 14:28:59
-->

# GCC specs: an introduction

> 来源：[https://wozniak.ca/blog/2024/01/02/1/index.html](https://wozniak.ca/blog/2024/01/02/1/index.html)

Have you ever passed `-v` to the driver and studied the output? There's a lot of stuff in there that is revealing. It's also a good way to understand what specs are and why they exist.

Because the output from `-v` is so extensive I'm going to use one example, broken up into parts. Not all of it is relevant for understanding specs, but it may be of general interest to some.

To get the output, I ran the following command on an amd64 machine running Debian 12, which is using GCC 12.2.

```
gcc -v t.c

```

The content of the source file is not important, only that the program is in a single file. (Use `int main (void) {}` if you have nothing sitting around.) I've reformatted the output where needed. In some cases I've omitted output and indicate it with `(...)`.

```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/12/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v (...)
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 12.2.0 (Debian 12.2.0-14) 
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'a-'

```

The first line says where the driver is getting its specs. It's possible to override the builtin specs but that is for a later discussion.

What follows are some environment variables that are set by `gcc` to communicate with the subprograms and various other information. There is also the "Configured with" line that tells you how GCC was configured, which is useful if you want to build it yourself.

In our example the driver has to compile the source using the C compiler, assemble it into an object file, then link the object into an executable. The first subprogram run is the compiler. `cc1` is the C compiler, `cc1plus` is the C++ compiler. Note that subprogram invocations in the output are lines that start with a single space.

```
 /usr/lib/gcc/x86_64-linux-gnu/12/cc1 \
   -quiet \
   -v \
   -imultiarch x86_64-linux-gnu \
   t.c \
   -quiet \
   -dumpdir a- \
   -dumpbase t.c \
   -dumpbase-ext .c \
   -mtune=generic \
   -march=x86-64 \
   -version \
   -fasynchronous-unwind-tables \
   -o /tmp/ccJOdUuR.s
(...)

```

We only provided the arguments `-v` and `t.c` to the driver yet the compiler has many more. Some are even provided twice. If you look at the [documentation](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/Invoking-GCC.html) for options you won't find the "dump" ones listed, for example, so the compiler doesn't use the same options as the driver. That said, some of them match. `-fasynchronous-unwind-tables` is a code generation option helpful when debugging if the target machine supports it. `-march` and `-mtune` control what kind of code is generated for the processor. These are options specific to the target or host. It would be tedious to specify them every time.

I've trimmed all the compiler output because it's mostly version info, although it does print the C standard it is using, some of the compiler's heuristic data, and the compiler executable checksum. It will also print the search order for headers which can be very handy when debugging header problems or just knowing where system headers reside.

As a side note, it's worth pointing out that you can run the compiler directly if you want. You can even pass it `--help` to see the extensive set of options it accepts. Running the compiler directly isn't something you need to do very often, even when debugging it, because `gcc` has a [-wrapper](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/Overall-Options.html#index-wrapper) option to do this for you.

The compiler outputs assembler source, so the next step is for the driver is to run the assembler.

```
 as -v --64 -o /tmp/ccj7Fe2p.o /tmp/ccJOdUuR.s
(...)

```

Note that the input is a temporary file, as is the output. The driver has to manage all these temporaries and coordinate them across the subprograms. If you pass `-save-temps` then it will change the way this is done and use more intuitive names.

Finally, we have the linker ^(invocation, complete with the messiest and most complex set of arguments.)

```
/usr/lib/gcc/x86_64-linux-gnu/12/collect2 \
   -plugin /usr/lib/gcc/x86_64-linux-gnu/12/liblto_plugin.so \
   -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/12/lto-wrapper \
   -plugin-opt=-fresolution=/tmp/cc7cMZRs.res \
   -plugin-opt=-pass-through=-lgcc \
   -plugin-opt=-pass-through=-lgcc_s \
   -plugin-opt=-pass-through=-lc \
   -plugin-opt=-pass-through=-lgcc \
   -plugin-opt=-pass-through=-lgcc_s \
   --build-id --eh-frame-hdr -m elf_x86_64 \
   --hash-style=gnu --as-needed \
   -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/Scrt1.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crti.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/crtbeginS.o \
   -L/usr/lib/gcc/x86_64-linux-gnu/12 \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../../lib \
   -L/lib/x86_64-linux-gnu \
   -L/lib/../lib \
   -L/usr/lib/x86_64-linux-gnu \
   -L/usr/lib/../lib \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../.. \
   /tmp/ccj7Fe2p.o \
   -lgcc \
   --push-state --as-needed -lgcc_s --pop-state \
   -lc -lgcc \
   --push-state --as-needed -lgcc_s --pop-state \
   /usr/lib/gcc/x86_64-linux-gnu/12/crtendS.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crtn.o

```

What the options mean is beyond the scope of this article, but it's worth noting what arguments are needed. There are object files (the `.o` suffix) and libraries (`-l` options) and library search paths (`-L` options). The order of these options matters, too, and the driver has to get this right. Furthermore, which system object files to use (found in `/usr/lib` in this example) can be non-obvious, so we should appreciate all the work the driver does behind the scenes. Manually specifying a link is tedious.

To round out the discussion of the files that the driver has to manage, all the temporary files created during the compile/assemble/link sequence are deleted before the driver exits.

All the arguments passed to the subprograms and the management of files are done with specs.