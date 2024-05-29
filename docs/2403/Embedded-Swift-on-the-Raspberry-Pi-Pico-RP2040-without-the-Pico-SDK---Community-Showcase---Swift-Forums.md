<!--yml
category: 未分类
date: 2024-05-27 14:40:56
-->

# Embedded Swift on the Raspberry Pi Pico/RP2040 without the Pico SDK - Community Showcase - Swift Forums

> 来源：[https://forums.swift.org/t/embedded-swift-on-the-raspberry-pi-pico-rp2040-without-the-pico-sdk/69338](https://forums.swift.org/t/embedded-swift-on-the-raspberry-pi-pico-rp2040-without-the-pico-sdk/69338)

Interesting. I think a special libc *might* be interesting/necessary for some embedded environments. For example, the Pico C SDK does depend on an embedded libc AFAIK, but that's a topic for the other thread [Embedded Swift running on the Raspberry Pi Pico](https://forums.swift.org/t/embedded-swift-running-on-the-raspberry-pi-pico/69001). You're right that the from-scratch approach we're discussing in this thread doesn't need it.

That would be nice, although I fear the stuff we need to do for the RP2040 will be difficult to wrap in a package:

*   The second-stage bootloader (boot2) is assembled and linked with a special linker script that puts it at the right location in flash
*   Then we calculate a checksum over the bootloader binary and write that checksum into the binary (the RP2040 won't boot unless this checksum is there). This is currently done by a Python script.
*   The runtime (crt0) for the main executable (setting up IRQ handlers, exception handlers etc.) is assembled and linked with the main executable
*   The main executable also needs to be linked with a special linker script

Maybe the checksum calculation can be done in a SwiftPM plugin? But I'm not sure we can integrate this cleanly.

We can prebuild boot2 and crt0, but then we're also no longer talking about a simple package that you can include as a dependency, right (because of the binary files involved)? And it doesn't solve how to pass the linker script to the final link step.

I think my biggest problem in doing any SwiftPM experiments right now is that I can't use `swift` as the linker to build a Pico executable – I have to use Clang for linking. I don't think I can tell SwiftPM to link with Clang unless I build an SE-0387-style SDK?

Here are the errors I get when I try to link with Swift (on Linux; I understand linking ELF files on macOS is work in progress: [[embedded] Start building and including lld even in Darwin toolchains by kubamracek · Pull Request #70715 · apple/swift · GitHub](https://github.com/apple/swift/pull/70715)):

```
/usr/bin/swift --version
Swift version 5.11-dev (LLVM 13124099c3f0229, Swift d6871edc839adec)
Target: aarch64-unknown-linux-gnu

"/usr/bin/swiftc" \
	-enable-experimental-feature Embedded \
	-target armv6m-none-none-eabi \
	-Xlinker --script=pico-sdk-comps/memmap_default.ld \
	-Xlinker -z -Xlinker max-page-size=4096 \
	-Xlinker --gc-sections \
	-Xlinker --wrap=__aeabi_lmul \
	build/bs2_default_padded_checksummed.S.obj build/crt0.S.obj build/bootrom.c.obj build/pico_int64_ops_aeabi.S.obj build/main.o \
	-o "build/SwiftPico.elf"
error: link command failed with exit code 1 (use -v to see invocation)
clang: error: no such file or directory: '/usr/lib/swift/armv6m/swiftrt.o'
error: fatalError
make: *** [Makefile:141: build/SwiftPico.elf] Error 1 
```

OK, so it's looking for `/usr/lib/swift/armv6m/swiftrt.o` and doesn't find it. That directory doesn't exist. Should it exist? Will it exist in the future as the embedded toolchain improves?

Linking with `-nostartfiles` gets rid of that error, but produces more:

```
"/usr/bin/swiftc" \
	-enable-experimental-feature Embedded \
	-target armv6m-none-none-eabi \
	-nostartfiles \
	-Xlinker -nostdlib \
	-Xlinker --script=pico-sdk-comps/memmap_default.ld \
	-Xlinker -z -Xlinker max-page-size=4096 \
	-Xlinker --gc-sections \
	-Xlinker --wrap=__aeabi_lmul \
	build/bs2_default_padded_checksummed.S.obj build/crt0.S.obj build/bootrom.c.obj build/pico_int64_ops_aeabi.S.obj build/main.o \
	-o "build/SwiftPico.elf"
error: link command failed with exit code 1 (use -v to see invocation)
/usr/bin/ld.gold: error: cannot find -lswiftCore
/usr/bin/ld.gold: error: cannot find -lc
/usr/bin/ld.gold: error: cannot find -lm
/usr/bin/ld.gold: error: cannot find -lclang_rt.builtins-armv6m
/usr/bin/ld.gold: internal error in set_address, at ../../gold/output.h:322
clang: error: ld.lld command failed with exit code 1 (use -v to see invocation)
error: fatalError
make: *** [Makefile:142: build/SwiftPico.elf] Error 1 
```

I'm afraid I don't really understand what I'm doing here, so it's hard to ask better questions.