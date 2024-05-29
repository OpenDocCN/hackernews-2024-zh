<!--yml
category: 未分类
date: 2024-05-27 14:39:59
-->

# Beyond the 1 MB barrier in DOS - by Julio Merino

> 来源：[https://blogsystem5.substack.com/p/beyond-the-1-mb-barrier-in-dos](https://blogsystem5.substack.com/p/beyond-the-1-mb-barrier-in-dos)

In [“From 0 to 1 MB in DOS”](https://blogsystem5.substack.com/p/from-0-to-1-mb-in-dos), I presented an overview of all the ways in which DOS and its applications tried to maximize the use of the 1 MB address space inherited from the 8086—even after the 80286 introduced support for 16 MB of memory and the 80386 opened the gates to 4 GB.

I know I promised that this follow-up article would be about DJGPP, but before getting into *that* review, I realized I had to take another detour to cover three more topics. Namely: *unreal mode*, which I intentionally ignored to not derail the post; *LOADALL*, which I didn’t know about until you readers mentioned it; and *DOS extenders*, which I was planning to describe in the DJGPP article but they are a better fit for this one.

So… strap your seat belts on and dive right in for another tour through the ancient techniques that DOS had to pull off to peek into the memory address space above the first MB. And get your hands ready because we’ll go over assembly code for a step-by-step jump into unreal mode.

Having read the preceding post, you should know by now what the real, protected, and VM86 modes of the x86 processors are. But there is one extra unofficial mode I did not talk about, and that is the *unreal mode*. The mode with the coolest name if you ask me.

Unreal mode is a special state of the processor in which the CPU acts as if it was in real mode… but with segment descriptors that allow it to reference memory above the 1 MB limit. These segments are impossible to define in real mode but, with different tricks for the 80286 and 80386, they become a possibility.

To understand how unreal mode works, we need to go back to the diagram I presented earlier describing the address resolution process that the 80286’s MMU executes on every memory access:

Diagram representing the operations that the 80286 does, in hardware, to resolve a segment:address memory reference.

As you can see in the diagram, for every memory access of the form `segment:offset`, the MMU:

1.  decodes the segment selector specified by `segment` to determine whether to query the GDT or LDT and which segment descriptor to read,

2.  reads the determined segment descriptor by indexing into the GDT or LDT tables,

3.  decodes the segments’ base address and limit as stored in the descriptor,

4.  checks the privileges of the executing code against the protections recorded in the descriptor,

5.  and ensures the `offset` is within the segment limit defined in the descriptor.

Only after these steps complete successfully, the memory access is allowed; otherwise, the processor raises a General Protection Fault exception.

This is all conceptually good until you realize that the GDT and LDT live in main memory… and main memory is slow—*excruciatingly* so in processor time. Having to read from main memory to resolve *every* memory access, even with a good L1 or L2 cache, would have a big performance impact on most applications.

To resolve this inefficiency, the processor has a cache of segment descriptors. Instructions that update the segment registers, such as `MOV DS, AX`, fetch descriptors from the GDT or LDT tables and store them in the cache. Instructions that reference memory, such as `MOV AX, DS:1234h`, read the descriptors from the cache without ever reaching out to main memory.

Diagram representing how the instructions used to load a register like DS are decoupled from the instructions that later consume the details about DS. It is crucial to note how the segment descriptor cache completely shields the second instruction from accessing main memory to resolve the address.

So far so good, but let’s add another twist. The diagram I presented earlier showing how the 80286 resolves a memory address does not only apply to protected mode: it applies to real mode too. In both modes, the processor accesses the descriptor cache to peek at the segment limits and protection settings. This means that, for real mode to work as 8086 code expects, the cache must contain values that are compatible with real mode: in particular, the limits must be set to 64 KB.

But what if we could somehow load arbitrary segment descriptors into the processor’s cache and leverage those in real mode? If we could do that, we could increase the segment limits stored in the cache, and then any memory references that use those cached descriptors would be able to bypass the real mode rules and access extended memory directly.

And this, dear reader, is what unreal mode is about. There is a ton of nuance about how this mode was discovered and how it got its name, and to learn more about that story, I’ll redirect you to [OS/2 Museum’s excellent history of the unreal mode](https://www.os2museum.com/wp/a-brief-history-of-unreal-mode/). But keep in mind that unreal mode is a clutch that DOS applications used to access memory above the 1 MB mark while avoiding expensive protected mode switches.

Before describing how we can switch the processor to unreal mode, we must first look at how we can actually take advantage of this mode.

In the 80286 case, unreal mode is annoying to use: the 80286 is a 16-bit CPU, which means that the `offset` part of a memory reference is limited to 64 KB. We can configure segments whose base address lies beyond the 1 MB address space, allowing us to read extended memory from real mode, but we can only do so in 64 KB chunks at a time.

The 80386 fares better due to it being a 32-bit CPU: the `offset` part of a memory reference can be expressed as a 32-bit quantity. We can do both `MOV AX, DS:[SI]` and `MOV AX, DS:[ESI]`, which means that if `DS` has a base of zero, we can reference any memory position in the 4 GB address space when using the `ESI` offset. Crucially, and contrary to what some people think, we can issue these two variants from 16-bit real mode: 32-bit instructions are *not* restricted to protected mode.

The way the 80386 allows 32-bit instructions in 16-bit mode and vice versa is via two properties. The first is an instruction width setting at the code segment level: segment descriptors can indicate that the code they contain is 16-bit or 32-bit. The default for real mode is to assume that code segments are set to 16 bits, but in protected mode we can choose whatever we prefer. The second is the `0x66` instruction prefix, which tells the processor that the following instruction operates in *the opposite* mode of what is configured in the current code segment: e.g. if the current code segment is a 16-bit segment, the `0x66` prefix marks 32-bit instructions.

Knowing this, we can see that unreal mode is most useful in the 80386 because, once enabled, code can easily reference any extended memory address using 32-bit offsets and the `0x66` prefix. But the 16-bit constraints of the 80286 don’t make unreal mode less useful given how much the DOS ecosystem needed to escape the real mode address space limitations and how difficult it was to leave protected mode in the 80286.

With that out of the way, what we are missing in our discussion are the ways by which to enter unreal mode. There are at least two: the undocumented LOADALL instruction and an unsupported jump from protected mode to real mode. Let’s take a look at both.

LOADALL is an undocumented instruction of the 80286 and 80386 processors. This instruction has a long history and I’m not going to cover it in detail because [others have done a much better job than I could](https://rep-lodsb.mataroa.blog/blog/intel-286-secrets-ice-mode-and-f1-0f-04/). I’ll restrict my explanation to what the instruction does and why it is useful to enter unreal mode.

Simply put, all LOADALL does is, as its name implies, set all processor registers at once with values that come from a memory region. This is similar to the widely unused built-in task switching functionality of x86 processors, which reloads the processor state with register values stored in a [Task State Segment (TSS)](https://en.wikipedia.org/wiki/Task_state_segment) descriptor.

But LOADALL has some quirks that the TSS descriptor doesn’t have, and those are what make it interesting. Witness the contents of the memory that LOADALL reads on an 80286:

```
`Physical Address    CPU Register
----------------    ------------
0800h-0805h         None
0806h-0807h         MSW
0808h-0815h         None
0816h-0817h         TR
0818h-0819h         Flag word
081Ah-081Bh         IP
081Ch-081Dh         LDT
081Eh-081Fh         DS
0820h-0821h         SS
0822h-0823h         CS
0824h-0825h         ES
0826h-0827h         DI
0828h-0829h         SI
082Ah-082Bh         BP
082Ch-082Dh         SP
082Eh-082Fh         BX
0830h-0831h         DX
0832h-0833h         CX
0834h-0835h         AX
0836h-083Bh         ES descriptor cache
083Ch-0841h         CS descriptor cache
0842h-0847h         SS descriptor cache
0848h-084Dh         DS descriptor cache
084Eh-0853h         GDTR
0854h-0859h         LDT descriptor cache
085Ah-085Fh         IDTR
0860h-0865h         TSS descriptor cache`
```

Registers, registers, more registers, descriptor cache… Wait, what? Entries for the *cached* segment descriptors? That’s interesting. These are the actual values for the segment descriptors that the processor stores in its internal cache. Which means that, by issuing a carefully-crafted LOADALL, we can tell the processor to set the cached descriptors to values that are nonsensical for the current operation mode—and the processor does *not* complain.

In essence, with just one instruction, we can tell the processor that the cached descriptors have base addresses above the 1 MB limit and/or that they have limits larger than 64 KB. And the processor will just accept those into the cache and use them for future memory references.

As it turns out, `HIMEM.SYS` did leverage LOADALL and we can even find its [original source code](https://github.com/neozeed/himem.sys-2.06) to peek into [how it achieved](https://github.com/neozeed/himem.sys-2.06/blob/main/oemsrc/xm286.asm#L241) this feat. It’s not trivial to do, but the performance gains of such an undocumented instruction were too hard to pass on in order to implement efficient transfers between conventional and extended memory.

This made me curious so I tried replicating LOADALL’s usage in [DOSBox](https://www.dosbox.com/) and… failed because DOSBox does not implement this instruction—which is understandable because it was not documented, it was very specific to two processors, and it apparently was not widely used outside of `HIMEM.SYS`. Maybe more-accurate processor emulators supply it, but I didn’t bother trying.

Which brings us to the other way to enter unreal mode: an unsupported jump from protected mode to real mode in the 80386.

Protected mode fascinated me as a child when I read about it in books that didn’t “belong” to my age. After having dipped my toes in 8086 assembly and having toyed around with boot sectors and the like, I fantasized about how protected mode “unlocked” enormous power. I knew I had to learn such black arts to write my own operating system and I remember writing code that tried to enter protected mode and always failed—yet it sounded so easy from the books: just set bit 0 of `CR0` to 1!

The assembly and processor books I read once upon a time and that got me interested in all of this low-level systems programming.

It wasn’t until many years later, around 2007, that I decided I wanted to fulfill my long-term dream to write an operating system. At that point, I invested some time to get protected mode to work—and I finally did, with interrupts enabled and all. (Spoiler alert: I never got to write an OS though… yet?) But even if that worked, getting into protected mode is always a frustrating experience of having to get many teeny tiny details right just so that the machine doesn’t crash. You do it right and you see absolutely nothing; you do it wrong and you see absolutely nothing either.

But wait. Why am I telling you all this? Ah yes, because another way of entering unreal mode is the following dance:

1.  Switch to protected mode.

2.  Set up segment registers to segment descriptors that provide access to the full address space, causing the cached descriptors to have a 4 GB limit.

3.  Switch back to real mode with the previous unsupported segment configuration in place.

Upon switching back to real mode, the properties of the segment descriptors that don’t make sense in real mode (those limits higher than 64 KB) are *sticky*, meaning that no architectural operations such as `MOV DS, AX` touch those bits of the cache. With that, it becomes possible to address extended memory.

Diagram representing how a real mode segment descriptor load does not modify the limit or flags already stored in the descriptor cache.

Unfortunately, the 80286 did not have a mechanism to switch from protected mode back to real mode. Creative folks found ways to do this by triple-faulting the CPU and carefully setting registers to skip the BIOS POST code, but this process was extremely slow and thus unfeasible for frequent switches. At the time, the lack of this feature in the 80286 was a major complaint from OS vendors because it made it difficult to support running DOS programs from within a modern operating system.

Intel addressed those concerns with the launch of the 80386 and its new VM86 mode. But… that wasn’t the only change. The 80386 also added the ability to return to real mode from protected mode—with a lot of caveats. Intel was prescriptive in how exactly to return to real mode and never described what would happen if you didn’t follow the rules to the letter… which was essentially begging for someone to try and see what would happen when doing that.

The answer, as we saw above, is unreal mode.

All of the above research made me really curious so I got the urge to see unreal mode in action. So, in preparation for this article, I wrote a trivial DOS program that enters unreal mode and shows it working.

As in the past, getting this demo to a functional state was a frustrating experience of fighting with carefully-crafted register values and memory offsets, DOSBox crashes, and [Bochs](https://bochs.sourceforge.io/) oddities. But in the end my demo ran successfully, and because I couldn’t find any trivial, readily-available sample piece of code online that did this, I concluded that I *had to* present it to you here for posterity’s sake.

So. Let’s get our hands dirty. All you need is [~~Doom Emacs~~](https://github.com/doomemacs/doomemacs) a text editor, [NASM](https://www.nasm.org/), and DOSBox. Our goal is to:

1.  write a flat COM executable,

2.  without sections (code and data mixed),

3.  all in 16-bit code with the occasional 32-bit operation override,

4.  that enters protected mode,

5.  that stores a string somewhere in extended memory far from the reach of real mode,

6.  that drops back to (un)real mode,

7.  that copies the string we stored in extended memory to conventional memory,

8.  that tells DOS to print the string we fetched,

9.  and that finally returns to DOS to prove that DOS still works.

Let’s begin.

The first thing we have to do is set up the GDT. We’ll use statically-configured descriptors in the code, like these:

```
`;;; Null descriptor.
                dq 0

;;; Code descriptor for this binary.  The base address needs fixup at
;;; runtime to point to the location where the code was loaded.
CODE_DESC       equ 1 << 3
                dw 0ffffh       ; Low 16 bits of the limit.
code_base_low   dw 0            ; Low 16 bits of the base address.
code_base_mid   db 0            ; Middle 8 bits of the base address.
                db 10011110b    ; Code/data, exec, conforming, read allowed.
                db 00000000b    ; Not 4KB, 16-bit, no long mode, limit 00h.
code_base_high  db 0            ; High 8 bits of the base address.

;;; Data/stack descriptor for this binary.  The base address needs fixup
;;; at runtime to point to the location where the code was loaded.
DATA_DESC       equ 2 << 3
                dw 0ffffh       ; Low 16 bits of the limit.
data_base_low   dw 0            ; Low 16 bits of the base address.
data_base_mid   db 0            ; Middle 8 bits of the base address.
                db 10010010b    ; Code/data, data, grows up, read-write.
                db 00000000b    ; Not 4KB, 16-bit, no long mode, limit 00h.
data_base_high  db 0            ; High 8 bits of the base address.

;;; Linear data descriptor covering the full 4 GB address space.  No fixup
;;; necessary.
LINEAR_DESC     equ 3 << 3
                dw 0ffffh       ; Low 16 bits of the limit.
                dw 0            ; Low 8 bits of the base address.
                db 0            ; Middle 8 bits of the base address.
                db 10010010b    ; Code/data, data, grows up, read-write.
                db 11001111b    ; 4KB, 16-bit, no long mode, limit=0fh.
                db 0            ; High 8 bits of the base address.`
```

The GDT presented above defines four descriptors:

1.  The null descriptor, which is unused but must be present.

2.  The code descriptor (`CODE_DESC`) for our COM executable. This is configured to 16 bits so that we don’t have to mix 16-bit and 32-bit sections in the same source file (which is easy but I wanted to avoid). Note also that the base address is zero, but we’ll have to patch it up at runtime to point to the actual location where DOS loaded our executable. We cannot predict this and all the offsets built into the code must remain valid when in protected mode, so we must compute this dynamically.

3.  The data and stack descriptor (`DATA_DESC`) for our COM executable. For the same reasons as the code descriptor, we must compute its base address at runtime.

4.  A linear data descriptor (`LINEAR_DESC`) to be able to reference the whole 4 GB address space. We’ll use this one to set up the segments for unreal mode.

Next up, we need to define the descriptor for the GDT itself right after the GDT data section:

```
`gdt_desc        equ $-gdt
gdt_base        dd 0`
```

Easy, but again, note how `gdt_base` is zero. The address to the GDT needs to be a linear address, and because we don’t know where the COM file will be loaded, we have to compute this address at runtime.

After this, it’s time to start the code section. The first thing we do is fix up all of the base addresses we left blank in the GDT itself and the GDT descriptor. Note that, because we target a COM binary (or a boot sector if you are so inclined), we can assume that `CS`, `DS`, `ES`, and `SS` all point to the same place, which makes things significantly easier:

```
 `;; Assume CS = DS = ES = SS (COM file or boot sector).
    mov [real_cs], cs

    ;; Populate the GDT code and data descriptors with our actual base address
    ;; so that the built-in code offsets work once we enter protected mode.
    xor eax, eax
    mov ax, cs
    shl eax, 4
    mov [code_base_low], ax
    mov [data_base_low], ax
    shr eax, 16
    mov [code_base_mid], al
    mov [data_base_mid], al
    mov [code_base_high], ah
    mov [data_base_high], ah

    ;; Populate the GDT descriptor with the linear address of the GDT.
    xor eax, eax
    mov eax, ds
    shl eax, 4
    add eax, gdt
    mov [gdt_base], eax`
```

Then, we do preparatory work to enter protected mode: namely, we disable the [Non-Maskable Interrupt (NMI)](https://wiki.osdev.org/Non_Maskable_Interrupt) and interrupts in general, and we enable the A20 gate:

```
 `;; Disable the Non-Maskable Interrupt (NMI) and interrupts.
    in al, 70h
    or al, 80h
    out 70h, al
    in al, 71h
    cli

    ;; Enable the A20 gate.
    in al, 92h
    or al, 2
    out 92h, al`
```

And then, we can finally do the magic to enter protected mode by loading the GDT descriptor, updating the `PE` (0th) bit in the `CR0` register, and doing a long jump:

```
 `;; Load the GDT.
    lgdt [gdt_desc]

    ;; Enable protected mode.
    mov eax, cr0
    or eax, 1
    mov cr0, eax

    ;; Flush out the processor pipeline and reload CS.
    jmp CODE_DESC:protected_mode`
```

That’s it. We have reached the protected mode realm! How exciting is *that*? We are now in a 16-bit code segment but the machine state is mostly “unusable”. None of the segment registers except `CS` are valid and interrupts are disabled (because we didn’t bother to set up the IDT—and we don’t have to for this simple experiment). So let’s do the minimum set up that we need:

```
`protected_mode:
    ;; Set up the data and stack segments.
    mov ax, DATA_DESC
    mov ds, ax
    mov ss, ax`
```

Now that the basic segments are configured, let’s copy the `msg` string built into the binary into extended memory. We do this by pointing `ES` to our linear segment and using the `EXTENDED_ADDR` offset, which I set to an arbitrary 4 MB:

```
 `;; Store a message in extended memory.  We are in protected mode so this
    ;; works by design.
    mov ax, LINEAR_DESC         ; Load the linear address space in ES.
    mov es, ax
    mov esi, msg                ; Point DS:[ESI] to our message.
    mov edi, EXTENDED_ADDR      ; Point ES:[EDI] to extended memory.
    mov ecx, MSGLEN
    o32 rep movsb               ; Must use 32-bit addressing.`
```

Once this is done, we prepare the `FS` segment with a large limit so that we can use it from unreal mode and restore `ES` to the conventional values:

```
 `;; Set up FS as an "unreal mode" segment.
    mov ax, LINEAR_DESC
    mov fs, ax
    ;; ... but restore ES to have standard real mode limits.  Not strictly
    ;; necessary but helps to prove our example.
    mov ax, DATA_DESC
    mov es, ax`
```

And, with that, we are ready to pivot back to real mode…

```
 `;; Disable protected mode.
    mov eax, cr0
    and eax, ~1
    mov cr0, eax

    ;; And now return to real mode with a far jump.
    pushf
    push word [real_cs]
    push unreal_mode
    iret`
```

… except that it isn’t “valid” real mode! Remember that we left `FS` configured with a high, non-standard limit of 4 GB? Such non-standard limit remains in the segment cache. But before leveraging that, let’s make our new machine state functional to continue executing the COM binary:

```
`unreal_mode:
    ;; Reload real mode COM segment layout.
    mov bx, cs
    mov ds, bx
    mov es, bx
    mov ss, bx`
```

We can now also reenable interrupts, which we must do before we decide to return to DOS:

```
 `;; Reenable the NMI and interrupts.
    sti
    in al, 70h
    and al, 7fh
    out 70h, al
    in al, 71h`
```

But, crucially, we keep the A20 enabled in order to correctly reference extended memory later on!

And after all of this dance, that’s really it. We are in unreal mode. Any memory references we make through the `FS` segment register can use offsets beyond the 1 MB limit no matter what its segment base is and no matter if the segment base *changes*.

Easy to say, hard to believe. Let’s prove that this is true. Let’s first fetch the message we stored in extended memory by copying it to conventional memory:

```
 `;; Fetch the message from extended memory by using a large offset.  This
    ;; would not work in real mode or VM86 (even with 32-bit addressing), but
    ;; does because we are actually in unreal mode.
    xor ax, ax                  ; Clear FS to show its high limits remain.
    mov fs, ax
    mov esi, EXTENDED_ADDR      ; Point FS:[ESI] to extended memory.
    mov edi, msgcopy            ; Point ES:[EDI] to our buffer.
    mov ecx, MSGLEN
    o32 fs rep movsb            ; Must use 32-bit addressing.`
```

And now that we got it copied, let’s call into DOS to print the message and return control to DOS:

```
 `;; Print the message we fetched from extended mode.
    mov ah, 40h
    mov bx, 1
    mov cx, MSGLEN
    mov dx, msgcopy             ; Remember this was all ... at first!
    int 21h

    ;; And jump back to DOS.
    mov ax, 4c00h
    int 21h`
```

If all goes well, you should see `Hello, unreal mode!` in the console and the usual `C:\>` prompt should greet you again.

But, wait, there is more! If you happen to be running this under Bochs—and I recommend that you do because it offers much better debugging facilities than DOSBox—you can pause the simulation with `CTRL+C` and dump the segment registers with `sreg`:

```
`<bochs:2> sreg
es:0x010e, dh=0x00009300, dl=0x10e0ffff, valid=7
        Data segment, base=0x000010e0, limit=0x0000ffff, Read/Write, Accessed
cs:0x000f, dh=0x00009300, dl=0x00f0ffff, valid=3
        Data segment, base=0x000000f0, limit=0x0000ffff, Read/Write, Accessed
ss:0x010e, dh=0x00009300, dl=0x10e0ffff, valid=7
        Data segment, base=0x000010e0, limit=0x0000ffff, Read/Write, Accessed
ds:0x010e, dh=0x00009300, dl=0x10e0ffff, valid=3
        Data segment, base=0x000010e0, limit=0x0000ffff, Read/Write, Accessed
fs:0x0000, dh=0x00cf9300, dl=0x0000ffff, valid=11
        Data segment, base=0x00000000, limit=0xffffffff, Read/Write, Accessed
gs:0x0000, dh=0x00009300, dl=0x0000ffff, valid=7
        Data segment, base=0x00000000, limit=0x0000ffff, Read/Write, Accessed
ldtr:0x0000, dh=0x00008200, dl=0x0000ffff, valid=1
tr:0x0000, dh=0x00008b00, dl=0x0000ffff, valid=1
gdtr:base=0x0000000000011c72, limit=0x20
idtr:base=0x0000000000000000, limit=0x3ff`
```

Pay attention to the details for the `limit` field of all the segment registers. You can see how the value is `0x0000ffff` for all of them as you’d expect in real mode… but `FS`’s limit is the non-standard `0xffffffff`. We did it. We are in unreal mode and DOS still runs fine. (In fact, remember that `HIMEM.SYS` itself *does* leverage unreal mode and this driver was active almost all the time in a DOS installation.)

Once you download the source code, you can build it with:

```
`nasm -o unreal.com unreal.asm`
```

And once built, you can run it within DOSBox by copying the binary into a directory you mount within the emulator or use [mtools](https://www.gnu.org/software/mtools/) to copy it into a DOS boot disk image for Bochs.

If you resisted the urge to keep toying with the example code presented above and are still reading, great! We can continue onto our final topic: [DOS extenders](https://en.wikipedia.org/wiki/DOS_extender). These programs are what truly set DOS free from the 1 MB address space limitations and thus are the right way to conclude this article.

A DOS extender, in rough terms, is a wrapper for your code that enters protected mode and transfers control back to you. This is very useful because, by running in protected mode, your *code* segment can span more than 640 KB, finally letting you run large binaries without resorting to ancient spells like [overlays](https://en.wikipedia.org/wiki/Overlay_(programming)). But if that was the only thing that a DOS extender did, it would be too simple of a thing and not make it a DOS-anything.

Representation of the structure of a DOS-extended application and how it relates to a DOS extender, DOS drivers, and raw hardware access.

The crucial feature that DOS extenders provide is a mechanism to call *back* into the BIOS and DOS to access the services these provide. In particular, this means accessing *drivers* and *the file system*. They do so by exposing the same [DOS API](https://en.wikipedia.org/wiki/DOS_API) of the real mode host in protected mode, and they do this so that they can transparently service it in an efficient manner.

The general mechanics behind DOS extenders are very similar to what I described earlier: they enter protected mode to use extended memory and run your code, but they *temporarily* return to real mode to issue BIOS and DOS service calls. Unfortunately, this naive implementation would be quite inefficient, so DOS extenders actually service various APIs in protected mode to avoid returning to real mode, and they optimize large buffer transfers to minimize the number of switches when they *do* have to switch modes. In other words, DOS extenders are their own mini OS on top of DOS.

DOS extenders were widely used in games, and you may well remember the iconic message that DOOM printed on startup:

```
`C:\>DOOM
DOS/4GW Protected Mode Run-time  Version 1.97
Copyright (c) Rational Systems, Inc. 1990-1994`
```

These messages came from [DOS/4G](https://en.wikipedia.org/wiki/DOS/4G), the most common DOS extender of all. The reason it was so common is because the “W version” was the free (but limited) edition of DOS/4G, and it shipped with the popular [Watcom](https://en.wikipedia.org/wiki/Watcom_C/C%2B%2B) C compiler that produced protected mode binaries. But pay attention to “run-time” in that message. Why run-time?

DOS extenders were not the only system component that entered protected mode. By the time DOS extenders became popular, Windows was already a thing and Windows likes protected mode too. And as we saw in the previous post, DOS itself came with `EMM386.EXE`, another mini hypervisor that put DOS inside a VM86 instance so that it could easily reference extended memory. So what if you wanted to run a popular DOS-extended program, say DOOM, inside one of these also-popular protected mode environments?

Tough luck. Nesting protected modes was impossible until [virtualization appeared](https://en.wikipedia.org/wiki/X86_virtualization). VM86 was close to what was necessary but it didn’t allow running protected mode programs from within it. So, how did this work? How could you run a DOS-extended program inside Windows or while `EMM386.EXE` was running?

Like in any computing problem, the answer is simple: by means of an abstraction layer! To support this flow, Microsoft defined the [DOS Protected Mode Interface (DPMI)](https://en.wikipedia.org/wiki/DOS_Protected_Mode_Interface), an API to abstract the core services to interact with protected mode. DPMI is *not* a replacement for a DOS extender though: DPMI is an API that DOS extenders themselves use to deal with protected mode operations.

Simple representation of the role that DPMI plays in DOS-extended applications and different host operating systems.

In the case of DOS-extended programs, the first thing that they do at startup is check if a DPMI kernel is present. If one exists, such as when the DOS program runs within Windows, then the DOS extender leverages Windows’ DPMI services and delegates all operations to Windows. But if such a provider isn’t running, the DOS extender starts the DPMI kernel typically built into itself.

If you want to see even more code in action, I refer you to skim through the sources of the free [DOS/32](https://sourceforge.net/projects/dos32a/) extender and the free [CWSDPMI](https://en.wikipedia.org/wiki/CWSDPMI) host.

And with that, I’m hoping that the next article will finally be the one talking about DJGPP and everything *else* it had to do to make Unix applications run semi-seamlessly on DOS… unless some other preparatory essay becomes necessary!