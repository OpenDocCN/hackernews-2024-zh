<!--yml

类别：未分类

日期：2024-05-27 14:40:56

-->

# 在树莓派Pico/RP2040上嵌入的Swift，没有Pico SDK - 社区展示 - Swift论坛

> 来源：[https://forums.swift.org/t/embedded-swift-on-the-raspberry-pi-pico-rp2040-without-the-pico-sdk/69338](https://forums.swift.org/t/embedded-swift-on-the-raspberry-pi-pico-rp2040-without-the-pico-sdk/69338)

有趣。我认为一种特殊的libc *可能* 对某些嵌入式环境是有趣/必要的。例如，我认为Pico C SDK确实依赖于嵌入式libc AFAIK，但这是另一个主题的话题 [在树莓派Pico上运行的嵌入式Swift](https://forums.swift.org/t/embedded-swift-running-on-the-raspberry-pi-pico/69001)。你说得对，我们正在讨论的这个从头开始的方法不需要它。

那将很好，尽管我担心我们需要为RP2040做的工作可能很难打包：

+   第二阶段引导加载程序（boot2）是通过一个特殊的链接器脚本组装和链接的，该脚本将其放置在闪存中的正确位置

+   然后我们计算引导加载程序二进制文件的校验和，并将该校验和写入二进制文件中（如果没有该校验和，RP2040将无法启动）。目前这是由一个Python脚本完成的。

+   主可执行文件的运行时（crt0）（设置IRQ处理程序、异常处理程序等）与主可执行文件一起组装和链接

+   主可执行文件还需要与一个特殊的链接器脚本进行链接

或许校验计算可以在一个SwiftPM插件中完成？但我不确定我们能干净地集成这个。

我们可以预构建boot2和crt0，但这样我们也不再谈论一个简单的可以作为依赖项包含的包了，对吧（因为涉及到二进制文件）？而且这并不能解决如何将链接器脚本传递给最终的链接步骤。

我现在做任何SwiftPM实验的最大问题，我不能使用`swift`作为链接器来构建Pico可执行文件 - 我必须使用Clang进行链接。我不认为我能告诉SwiftPM使用Clang进行链接，除非我构建一个SE-0387风格的SDK？

当我尝试在Swift上链接时（在Linux上；我了解在macOS上链接ELF文件仍在进行中：

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

好吧，所以它正在寻找`/usr/lib/swift/armv6m/swiftrt.o`并且找不到它。那个目录不存在。它应该存在吗？随着嵌入式工具链的改进，它将来会存在吗？

使用`-nostartfiles`链接可以消除该错误，但会产生更多错误：

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

我怕我真的不明白我在这里做什么，所以很难提出更好的问题。
