<!--yml
category: 未分类
date: 2024-05-27 15:01:34
-->

# Jazelle - Wikipedia

> 来源：[https://en.wikipedia.org/wiki/Jazelle](https://en.wikipedia.org/wiki/Jazelle)

Hardware extension for ARM processors

**Jazelle DBX** (direct bytecode execution)^([[1]](#cite_note-patent-1)) is an extension that allows some [ARM](/wiki/ARM_architecture "ARM architecture") processors to execute [Java bytecode](/wiki/Java_bytecode "Java bytecode") in [hardware](/wiki/Computer_hardware "Computer hardware") as a third execution state alongside the existing ARM and [Thumb](/wiki/ARM_architecture#Thumb "ARM architecture") modes.^([[2]](#cite_note-product-2)) Jazelle functionality was specified in the ARMv5TEJ architecture^([[3]](#cite_note-armarm-3)) and the first processor with Jazelle technology was the [ARM926EJ-S](/wiki/ARM9 "ARM9").^([[4]](#cite_note-Shanghai-4)) Jazelle is denoted by a "J" appended to the CPU name, except for post-v5 cores where it is required (albeit only in trivial form) for architecture conformance.

[Jazelle RCT](/wiki/Jazelle_RCT "Jazelle RCT") (Runtime Compilation Target) is a different technology based on ThumbEE mode; it supports [ahead-of-time](/wiki/Ahead-of-time_compilation "Ahead-of-time compilation") (AOT) and [just-in-time](/wiki/Just-in-time_compilation "Just-in-time compilation") (JIT) compilation with Java and other execution environments.

The most prominent use of Jazelle DBX is by manufacturers of mobile phones to increase the execution speed of [Java ME](/wiki/Java_ME "Java ME") games and applications.^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) A Jazelle-aware [Java virtual machine](/wiki/Java_virtual_machine "Java virtual machine") (JVM) will attempt to run Java bytecode in hardware, while returning to the software for more complicated, or lesser-used bytecode operations. ARM claims that approximately 95% of bytecode in typical program usage ends up being directly processed in the hardware.

The published specifications are very incomplete, being only sufficient for writing [operating system](/wiki/Operating_system "Operating system") code that can support a JVM that uses Jazelle.^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) The declared intent is that only the JVM software needs to (or is allowed to) depend on the hardware interface details. This tight binding facilitates the hardware and JVM evolving together without affecting other software. In effect, this gives [ARM Holdings](/wiki/ARM_Holdings "ARM Holdings") considerable control over which JVMs are able to exploit Jazelle.^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) It also prevents open source JVMs from using Jazelle. These issues do not apply to the ARMv7 ThumbEE environment, the nominal successor to Jazelle DBX.

## Implementation[[edit](/w/index.php?title=Jazelle&action=edit&section=1 "Edit section: Implementation")]

The Jazelle extension uses low-level [binary translation](/wiki/Binary_translation "Binary translation"), implemented as an extra stage between the fetch and decode stages in the processor [instruction pipeline](/wiki/Instruction_pipeline "Instruction pipeline"). Recognised bytecodes are converted into a string of one or more native ARM instructions.

The Jazelle mode moves JVM interpretation into hardware for the most common simple JVM instructions. This is intended to significantly reduce the cost of interpretation. Among other things, this reduces the need for [Just-in-time compilation](/wiki/Just-in-time_compilation "Just-in-time compilation") and other JVM accelerating techniques.^([[5]](#cite_note-CPM-5)) JVM instructions that are not implemented in Jazelle hardware cause appropriate routines in the Jazelle-aware JVM implementation to be invoked. Details are not published, since all JVM innards are transparent (except for performance) if correctly interpreted.

Jazelle mode is entered via the BXJ instructions. A hardware implementation of Jazelle will only cover a subset of JVM bytecodes. For unhandled bytecodes—or if overridden by the operating system—the hardware will invoke the software JVM. The system is designed so that the software JVM does not need to know which bytecodes are implemented in hardware and a software fallback is provided by the software JVM for the full set of bytecodes.

## Instruction set[[edit](/w/index.php?title=Jazelle&action=edit&section=2 "Edit section: Instruction set")]

The Jazelle [instruction set](/wiki/Instruction_set "Instruction set") is well documented as [Java bytecode](/wiki/Java_bytecode "Java bytecode"). However, ARM has not released details on the exact execution environment details; the documentation provided with Sun's [HotSpot](/wiki/HotSpot_(virtual_machine) "HotSpot (virtual machine)") [Java Virtual Machine](/wiki/Java_virtual_machine "Java virtual machine") goes as far as to state: "For the avoidance of doubt, distribution of products containing software code to exercise the BXJ instruction and enable the use of the ARM Jazelle architecture extension without [..] agreement from ARM is expressly forbidden."^([[6]](#cite_note-Hotspot-6))

Employees of ARM have in the past published several white papers that do give some good pointers about the processor extension.^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*]) Versions of the ARM Architecture reference Manual available from 2008 have included pseudocode for the "BXJ" (Branch and eXchange to Java) instruction, but with the finer details being shown as "SUB-ARCHITECTURE DEFINED" and documented elsewhere.

## Application binary interface (ABI)[[edit](/w/index.php?title=Jazelle&action=edit&section=3 "Edit section: Application binary interface (ABI)")]

The Jazelle state relies on an agreed [calling convention](/wiki/Calling_convention "Calling convention") between the JVM and the Jazelle hardware state. This [application binary interface](/wiki/Application_binary_interface "Application binary interface") is not published by ARM, rendering Jazelle an [undocumented feature](/wiki/Undocumented_feature "Undocumented feature") for most users and Free Software JVMs.

The entire VM state is held within normal ARM registers, allowing compatibility with existing operating systems and interrupt handlers unmodified. Restarting a bytecode (such as following a return from interrupt) will re-execute the complete sequence of related ARM instructions.

Specific registers are designated to hold the most important parts of the JVM state: registers R0–R3 hold an alias of the top of the Java stack, R4 holds Java local operand zero (pointer to `*this`) and R6 contains the Java stack pointer.^([[7]](#cite_note-accelerating-7))

Jazelle reuses the existing [program counter](/wiki/Program_counter "Program counter") PC or its synonym register R15\. A pointer to the *next* bytecode goes in R14,^([[8]](#cite_note-Intel-8)) so the use of the PC is not generally user-visible except during debugging.

### CPSR: Mode indication[[edit](/w/index.php?title=Jazelle&action=edit&section=4 "Edit section: CPSR: Mode indication")]

Java bytecode is indicated as the current instruction set by a combination of two bits in the ARM CPSR (Current Program Status Register). The "T"-bit must be cleared and the "J"-bit set.^([[9]](#cite_note-lkml-9))

Bytecodes are decoded by the hardware in two stages (versus a single stage for Thumb and ARM code) and switching between hardware and software decoding (Jazelle mode and ARM mode) takes ~4 clock cycles.^([[10]](#cite_note-embedded-10))

For entry to Jazelle hardware state to succeed, the JE (Jazelle Enable)^([[3]](#cite_note-armarm-3)) bit in the CP14:C0(C2)[bit 0] register must be set; clearing of the JE bit by a [privileged] operating system provides a high-level override to prevent application programs from using the hardware Jazelle acceleration.^([[11]](#cite_note-armarmjp-11)) Additionally, the CV (Configuration Valid) bit^([[3]](#cite_note-armarm-3)) found in CP14:c0(c1)[bit 1]^([[11]](#cite_note-armarmjp-11)) must be set to show that there is a consistent Jazelle state setup for the hardware to use.

### BXJ: Branch to Java[[edit](/w/index.php?title=Jazelle&action=edit&section=5 "Edit section: BXJ: Branch to Java")]

The BXJ instruction attempts to switch to Jazelle state, and if allowed and successful, sets the "J" bit in the CPSR; otherwise, it "falls through" and acts as a standard BX ([Branch](/wiki/Branch_(computer_science) "Branch (computer science)")) instruction.^([[3]](#cite_note-armarm-3)) The only time when an operating system or debugger must be fully aware of the Jazelle mode is when decoding a faulted or trapped instruction. The Java [program counter](/wiki/Program_counter "Program counter") (PC) pointing to the next instructions must be placed in the Link Register (R14) before executing the BXJ branch request, as regardless of hardware or software processing, the system must know where to begin decoding.

Because the current state is held in the CPSR, the bytecode instruction set is automatically reselected after task-switching and processing of the current Java bytecode is restarted.^([[7]](#cite_note-accelerating-7))

Following an entry into the Jazelle state mode, bytecodes can be processed in one of three ways: decoded and executed natively in hardware, handled in software (with optimised ARM/ThumbEE JVM code), or treated as an invalid/illegal opcode. The third case will cause a branch to an ARM exception mode, as will a Java bytecode of 0xff, which is used for setting JVM breakpoints.^([[12]](#cite_note-ARM1026EJ-S-12))

Execution will continue in hardware until an unhandled bytecode is encountered, or an exception occurs. Between 134 and 149 bytecodes (out of 203 bytecodes specified in the JVM specification) are translated and executed directly in the hardware.

### Low-level registers[[edit](/w/index.php?title=Jazelle&action=edit&section=6 "Edit section: Low-level registers")]

Low-level configuration registers, for the hardware virtual machine, are held in the ARM Co-processor "CP14 register c0". The registers allow detecting, enabling or disabling the hardware accelerator (if it is available).^([[13]](#cite_note-CIHIGDHI-13))

*   The Jazelle Identity Register in register CP14:C0(C0) is read-only accessible in all modes.
*   The Jazelle OS Control Register at CP14:c0(c1) is only accessible in kernel mode and will cause an exception when accessed in user mode.
*   The Jazelle Main Configuration Register at CP14:C0(C2) is write-only in user mode and read-write in kernel mode.

A "trivial" hardware implementation of Jazelle (as found in the [QEMU](/wiki/QEMU "QEMU") emulator) is only required to support the BXJ opcode itself (treating BXJ as a normal BX instruction^([[3]](#cite_note-armarm-3))) and to return RAZ (Read-As-Zero) for all of the CP14:c0 Jazelle-related registers.^([[14]](#cite_note-Cortex-A8-14))

## Successor: ThumbEE[[edit](/w/index.php?title=Jazelle&action=edit&section=7 "Edit section: Successor: ThumbEE")]

The ARMv7 architecture has de-emphasized Jazelle and *Direct Bytecode Execution* of JVM bytecodes. In implementation terms, only trivial hardware support for Jazelle is now required: support for entering and exiting Jazelle mode, but not for executing any Java bytecodes.

Instead, the *Thumb Execution Environment* ([ThumbEE](/wiki/ARM_architecture#Thumb_Execution_Environment_(ThumbEE) "ARM architecture")) was to be preferred, but [has since also been deprecated.](/wiki/ARM_architecture#Thumb_Execution_Environment_(ThumbEE) "ARM architecture") Support for ThumbEE was mandatory in ARMv7-A processors (such as the Cortex-A8 and Cortex-A9), and optional in ARMv7-R processors. ThumbEE targeted compiled environments, perhaps using [JIT](/wiki/Just-in-time_compilation "Just-in-time compilation") technologies. It was not at all specific to Java, and was fully documented; much broader adoption was anticipated than Jazelle was able to achieve.

ThumbEE was a variant of the Thumb2 16/32-bit instruction set. It integrated null pointer checking; defined some new fault mechanisms; and repurposed the 16-bit LDM and STM opcode space to support a few instructions such as range checking, a new handler invocation scheme, and more. Accordingly, compilers that produced Thumb or Thumb2 code could be modified to work with ThumbEE-based runtime environments.

## References[[edit](/w/index.php?title=Jazelle&action=edit&section=8 "Edit section: References")]