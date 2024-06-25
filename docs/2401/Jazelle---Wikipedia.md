<!--yml

类别: 未分类

日期: 2024-05-27 15:01:34

-->

# Jazelle - 维基百科

> 来源：[https://en.wikipedia.org/wiki/Jazelle](https://en.wikipedia.org/wiki/Jazelle)

ARM 处理器的硬件扩展

**Jazelle DBX**（直接字节码执行）^([[1]](#cite_note-patent-1)) 是一个扩展，允许一些[ARM](/wiki/ARM_architecture "ARM architecture")处理器在[硬件](/wiki/Computer_hardware "Computer hardware")中以第三个执行状态执行[Java字节码](/wiki/Java_bytecode "Java bytecode")，与现有的 ARM 和[Thumb](/wiki/ARM_architecture#Thumb "ARM architecture") 模式并存。^([[2]](#cite_note-product-2)) Jazelle 功能在 ARMv5TEJ 架构中被规定^([[3]](#cite_note-armarm-3))，具有 Jazelle 技术的第一款处理器是[ARM926EJ-S](/wiki/ARM9 "ARM9")。^([[4]](#cite_note-Shanghai-4)) 除了在架构符合性上（尽管只是形式上）要求后 v5 核心中都会在 CPU 名称后添加 "J" 表示 Jazelle。

[Jazelle RCT](/wiki/Jazelle_RCT "Jazelle RCT")（运行时编译目标）是基于 ThumbEE 模式的不同技术；它支持与 Java 和其他执行环境一起进行[提前](/wiki/Ahead-of-time_compilation "Ahead-of-time compilation")（AOT）和[即时](/wiki/Just-in-time_compilation "Just-in-time compilation")（JIT）编译。

Jazelle DBX 最突出的用途是由手机制造商使用，以提高[Java ME](/wiki/Java_ME "Java ME")游戏和应用的执行速度。^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) 一个 Jazelle-aware 的[Java虚拟机](/wiki/Java_virtual_machine "Java virtual machine")（JVM）将尝试在硬件中运行 Java 字节码，同时在更复杂或较少使用的字节码操作时返回到软件中执行。ARM 声称，典型程序使用中大约 95% 的字节码最终会在硬件中直接处理。

已发布的规范非常不完整，只足以编写可支持使用 Jazelle 的 JVM 的[操作系统](/wiki/Operating_system "Operating system")代码。^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) 声明的意图是只有 JVM 软件需要（或被允许）依赖于硬件接口细节。这种紧密的绑定有助于硬件和 JVM 一起演化而不影响其他软件。实际上，这使得[ARM Holdings](/wiki/ARM_Holdings "ARM Holdings")对能够利用 Jazelle 的 JVM 有相当大的控制权。^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) 这也阻止了开源 JVM 使用 Jazelle。这些问题不适用于 ARMv7 ThumbEE 环境，这是 Jazelle DBX 的名义上的继任者。

## 实施[[编辑](/w/index.php?title=Jazelle&action=edit&section=1 "Edit section: Implementation")]

Jazelle扩展使用低级别的[二进制翻译](/wiki/Binary_translation "二进制翻译")，作为处理器[指令流水线](/wiki/Instruction_pipeline "指令流水线")中提取和解码阶段之间的额外阶段实现。识别的字节码被转换为一个或多个本地ARM指令字符串。

Jazelle模式将JVM解释移入硬件，用于最常见的简单JVM指令。这旨在显著减少解释成本。除此之外，这减少了对[即时编译](/wiki/Just-in-time_compilation "即时编译")和其他JVM加速技术的需求。^([[5]](#cite_note-CPM-5)) 在Jazelle硬件中未实现的JVM指令会导致调用Jazelle感知的JVM实现中的适当例程。细节未公布，因为只要正确解释，所有JVM内部都是透明的（除了性能）。

通过BXJ指令进入Jazelle模式。Jazelle的硬件实现只覆盖了JVM字节码的一个子集。对于未处理的字节码，或者如果被操作系统覆盖，硬件将调用软件JVM。该系统设计使得软件JVM无需知道哪些字节码是在硬件中实现的，并且为完整的字节码集提供了软件回退。

## 指令集[[编辑](/w/index.php?title=Jazelle&action=edit&section=2 "编辑：指令集")]

Jazelle [指令集](/wiki/Instruction_set "指令集") 得到了良好的文档支持，被作为[Java字节码](/wiki/Java_bytecode "Java字节码")进行了描述。然而，ARM并没有公布关于确切执行环境细节的详细信息；Sun的[HotSpot](/wiki/HotSpot_(virtual_machine) "HotSpot (虚拟机)") [Java虚拟机](/wiki/Java_virtual_machine "Java虚拟机")提供的文档甚至声明：“为了避免疑问，未经ARM同意，分发含有软件代码以执行BXJ指令并启用ARM Jazelle架构扩展的产品是明确禁止的。”^([[6]](#cite_note-Hotspot-6))

ARM的员工过去曾发布过几篇白皮书，提供了一些关于处理器扩展的有益指导。^([*[需要引证](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) 2008年起可获取的ARM架构参考手册版本包括了“BXJ”（Branch and eXchange to Java）指令的伪代码，但更详细的细节被显示为“子架构定义”，并在其他地方记录。

## 应用程序二进制接口 (ABI)[[编辑](/w/index.php?title=Jazelle&action=edit&section=3 "编辑：应用程序二进制接口 (ABI)")]

Jazelle 状态依赖于 JVM 和 Jazelle 硬件状态之间约定的[调用约定](/wiki/Calling_convention "调用约定")。这种[应用程序二进制接口](/wiki/Application_binary_interface "应用程序二进制接口")未由 ARM 发布，使 Jazelle 成为大多数用户和自由软件 JVM 的一个[未记录功能](/wiki/Undocumented_feature "未记录功能")。

整个 VM 状态保存在正常的 ARM 寄存器中，允许与现有操作系统和未修改的中断处理程序兼容。重新启动字节码（例如在中断返回后）将重新执行相关 ARM 指令的完整序列。

特定寄存器被指定为保存 JVM 状态的最重要部分：寄存器 R0–R3 保存 Java 栈顶的别名，R4 保存 Java 局部操作数零（指向 `*this` 的指针），R6 包含 Java 栈指针。^([[7]](#cite_note-accelerating-7))

Jazelle 重用现有的[程序计数器](/wiki/Program_counter "程序计数器") PC 或其同义寄存器 R15。指向*下一个*字节码的指针存放在 R14 中，^([[8]](#cite_note-Intel-8)) 因此在调试过程中，通常不会直接看到 PC 的使用。

### CPSR：模式指示[[编辑](/w/index.php?title=Jazelle&action=edit&section=4 "编辑章节：CPSR：模式指示")]

Java 字节码通过 ARM CPSR（Current Program Status Register）中的两个比特的组合来表示当前指令集。必须清除“T”比特并设置“J”比特。^([[9]](#cite_note-lkml-9))

字节码由硬件在两个阶段解码（与 Thumb 和 ARM 代码的单个阶段相比），在硬件和软件解码（Jazelle 模式和 ARM 模式）之间切换需要约 4 个时钟周期。^([[10]](#cite_note-embedded-10))

为了成功进入 Jazelle 硬件状态，必须设置 CP14:C0(C2) 寄存器中的 JE（Jazelle Enable）^([[3]](#cite_note-armarm-3)) 位；[特权]操作系统通过清除 JE 位提供了一个高级别的覆盖，以防止应用程序使用硬件 Jazelle 加速。^([[11]](#cite_note-armarmjp-11)) 此外，必须设置 CP14:c0(c1) 寄存器中的 CV（Configuration Valid）^([[3]](#cite_note-armarm-3)) 位，以表明硬件使用的 Jazelle 状态设置是一致的。

### BXJ：跳转到 Java[[编辑](/w/index.php?title=Jazelle&action=edit&section=5 "编辑章节：BXJ：跳转到 Java")]

BXJ 指令尝试切换到 Jazelle 状态，如果允许且成功，则设置 CPSR 中的“J”位；否则，它将“穿透”并充当标准的 BX（[Branch](/wiki/Branch_(computer_science) "Branch (computer science)")）指令。^([[3]](#cite_note-armarm-3)) 当操作系统或调试器必须完全了解 Jazelle 模式时，唯一的时间是解码出现故障或陷入的指令时。在执行 BXJ 分支请求之前，指向下一条指令的 Java [程序计数器](/wiki/Program_counter "Program counter")（PC）必须被放置在链接寄存器（R14）中，因为无论硬件还是软件处理，系统都必须知道从哪里开始解码。

由于当前状态保存在 CPSR 中，所以在任务切换后会自动重新选择字节码指令集，并重新启动当前 Java 字节码的处理。^([[7]](#cite_note-accelerating-7))

进入 Jazelle 状态模式后，字节码可以通过以下三种方式之一进行处理：在硬件中进行解码和执行原生处理，以软件方式处理（使用优化的 ARM/ThumbEE JVM 代码），或将其视为无效/非法操作码。第三种情况将导致分支到 ARM 异常模式，以及 Java 字节码为 0xff 时，用于设置 JVM 断点。^([[12]](#cite_note-ARM1026EJ-S-12))

执行将在硬件中继续，直到遇到一个未处理的字节码，或发生异常。在 JVM 规范中指定的 203 个字节码中，约有 134 到 149 个字节码被直接翻译并在硬件中执行。

### 低级别寄存器

低级别的配置寄存器，用于硬件虚拟机，保存在 ARM 协处理器“CP14 寄存器 c0”中。这些寄存器允许检测、启用或禁用硬件加速器（如果可用）。^([[13]](#cite_note-CIHIGDHI-13))

+   在 CP14:C0(C0) 寄存器中，Jazelle 身份寄存器在所有模式下都是只读的。

+   在内核模式下，CP14:c0(c1) 寄存器中的 Jazelle 操作系统控制寄存器是唯一可访问的，在用户模式下访问时会引发异常。

+   在 CP14:C0(C2) 寄存器中，Jazelle 主配置寄存器在用户模式下是只写的，在内核模式下是可读可写的。

"Jazelle" 的一个“简单”硬件实现（如在 [QEMU](/wiki/QEMU "QEMU") 模拟器中找到的）只需支持 BXJ 操作码本身（将 BXJ 视为正常的 BX 指令^([[3]](#cite_note-armarm-3)）并对所有 CP14:c0 与 Jazelle 相关的寄存器返回 RAZ（Read-As-Zero）。^([[14]](#cite_note-Cortex-A8-14))

## 后继者：ThumbEE

ARMv7架构已经不再强调Jazelle和JVM字节码的*直接字节码执行*。在实施方面，现在只需要对Jazelle提供微不足道的硬件支持：支持进入和退出Jazelle模式，但不支持执行任何Java字节码。

相反，*Thumb执行环境* ([ThumbEE](/wiki/ARM_architecture#Thumb_Execution_Environment_(ThumbEE) "ARM架构")) 是首选，但[自那时以来也已被弃用。](/wiki/ARM_architecture#Thumb_Execution_Environment_(ThumbEE) "ARM架构") ARMv7-A处理器（如Cortex-A8和Cortex-A9）中对ThumbEE的支持是强制性的，而在ARMv7-R处理器中是可选的。 ThumbEE面向编译环境，可能使用[JIT](/wiki/Just-in-time_compilation "即时编译")技术。它与Java毫无关系，并且有完整的文档；预期它的广泛采用将比Jazelle能够实现的更多。

ThumbEE是Thumb2 16/32位指令集的一个变体。它集成了空指针检查；定义了一些新的故障机制；并重新分配了16位LDM和STM的操作码空间，以支持一些指令，如范围检查，新的处理程序调用方案等。因此，生成Thumb或Thumb2代码的编译器可以修改为与基于ThumbEE的运行时环境配合使用。

## 参考资料[[编辑](/w/index.php?title=Jazelle&action=edit&section=8 "编辑：参考资料")]
