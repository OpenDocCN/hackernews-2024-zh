<!--yml
category: 未分类
date: 2024-05-27 14:32:03
-->

# OpenBSD system-call pinning [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/959562/0578b8e463f790c1/](https://lwn.net/SubscriberLink/959562/0578b8e463f790c1/)

| **This article brought to you by LWN subscribers**Subscribers to LWN.net made this article — and everything that surrounds it — possible. If you appreciate our content, please [buy a subscription](/subscribe/) and make the next set of articles possible. |

By **Daroc Alden**
January 31, 2024

[Return-oriented programming](https://en.wikipedia.org/wiki/Return-oriented_programming) (ROP) attacks are hard to defend against. Partial mitigations such as address-space layout randomization, stack canaries, and other techniques are commonly deployed to try and frustrate ROP attacks. Now, OpenBSD is experimenting with a new mitigation that makes it harder for attackers to make system calls, although some security researchers have expressed doubt that it will prove effective at stopping real-world attacks. In his announcement message, Theo de Raadt said that this work "<q>makes some specific low-level attack methods unfeasable on OpenBSD, which will force the use of other methods.</q>"

Return-oriented programming is one of a family of techniques that use indirect jumps to call bits of code that already exist in a process's address space in an attacker-controlled order. The original attack involved overwriting the stack with carefully chosen addresses so that a function would "return" to a new location. Since the original discovery, other related attacks that use jumps through function pointers, signals, and other indirect jumps have been developed.

In December, De Raadt sent a [patch](/Articles/959664/) to the OpenBSD mailing list expanding OpenBSD's restrictions on the locations from which a process can make system calls. [A previous commit](https://github.com/openbsd/src/commit/83762a71f74848f4d09174ce350838b4204957c5) added code that declares a new ELF section which specifies where particular system calls are located within a program, so that the kernel can detect when a program tries to call a system call from the wrong location. Since OpenBSD does not have a stable system-call interface (instead suggesting that programs go through the C library for a stable interface), the new sections will not need to be explicitly added to most binary programs. Now that patch has [been merged](/Articles/959883/), finishing a process which De Raadt said has taken five years.

#### Background

OpenBSD [already restricted](/Articles/806776/) where programs can make system calls. In 2019, De Raadt added code to ensure that system calls could only be made from four locations: in the text of a static binary (that links the C library statically, and so doesn't have a separate section at runtime), in the signal trampoline (where a system call is required to return from a signal handler), in the text of `ld.so` (the dynamic linker, that needs to make system calls to set up the process's address space), and in the text of `libc.so` (where the OpenBSD system call stubs live).

This code relied on a new [`msyscall()`](https://man.openbsd.org/msyscall) system call to let the linker inform the kernel of where `libc.so` (the shared object for the system's C library) is mapped within the address space. In February 2023, De Raadt [extended](/Articles/959668/) these protections with the introduction of [`pinsyscall()`](https://man.openbsd.org/OpenBSD-7.4/pinsyscall), which is used to say where in the binary a process is allowed to call [`execve()`](https://man.openbsd.org/execve.2). Both of these system calls can only be invoked once by a given process, which is done by the dynamic linker.

```
    int msyscall(void *addr, size_t len);
    int pinsyscall(int syscall, void *start, size_t len);

```

Despite its more generic signature, `pinsyscall()` only supports specifying a location for `execve()` calls.

These mechanisms were also intended to make it harder for ROP attacks to gain a foothold. Requiring the address from which a system call is made to be within the `msyscall()` block ensures that an attack cannot make use of any ROP gadgets ending in a system call that may be present outside of the specially designated areas. Requiring that `execve()` calls come from one specific location is also intended to make it harder for an attack to figure out where to make a call from, since calling `execve()` to execute an attacker-controlled program is a common stepping stone in an attack.

The new work obsoletes both of these mechanisms, with De Raadt [suggesting](/Articles/959666) that once the new code had been adopted for a release or two, they could "<q>turn msyscall() and the less powerful pinsyscall(2) into NOPs, and eventually remove them</q>".

#### The patch

The new work adds a new [`pinsyscalls()`](https://man.openbsd.org/pinsyscalls.2) system call:

```
    int pinsyscalls(void *start, size_t len, u_int *pintable, int npins);

```

`pinsyscalls()` sends a "pintable" specifying from where in the process's address space each possible system call is expected to be made. The kernel uses the information in the table to check on entry to the kernel whether it is being invoked from a specified location. This check is intended to prevent a ROP attack from setting the system call number and then jumping directly to a system-call CPU instruction corresponding to a different system call. For example, an attack wishing to make an `execve()` call would need to jump to the specific instruction in the C library that has been added to the allowlist for that call, not another stub or the middle of an unrelated instruction which simply happens to decode as a system call instruction.

When setting up a new process, the dynamic linker uses `pinsyscalls()` to inform the kernel about from where the process expects to make system calls. The new work adds an "openbsd.syscalls" ELF section to select programs: `ld.so`, `libc.so`, and `libc.a`. The new ELF section contains an array of program offsets and system call numbers, indicating which system call is expected at each location. This section is read by the dynamic linker and used to provide a suitable pintable to the kernel. Programs that link against the C library can therefore benefit from the new protection immediately, without requiring changes to their build process. Unlike Linux, OpenBSD develops the kernel and user-space together, so the user-space components of this work are already in place.

Security researchers have expressed doubt about how useful this check is at preventing compromises. One researcher, "stein", [noted](https://isopenbsdsecu.re/mitigations/pinsyscall/) that "<q>an attacker able to perform ROP can simply use the libc stub, instead of issuing raw syscalls</q>", referring to the possibility of an attack jumping directly to the instruction which has been added to the allowlist for a particular system call. Another researcher, Saagar Jha, [commented](https://federated.saagarjha.com/notice/AcmzyfPcwc8KDVlOqm) on the new patch, saying "<q>if you take this to its logical conclusion it's just 'applications should specify which system calls they use' which is literally just what pledge does and it’s enforced by the kernel and not in some weird ad-hoc IP to syscall number lookup scheme</q>".

OpenBSD does have existing mitigations designed to make it difficult for ROP attacks to determine the location of the C library system call stubs. One such protection is [address-space layout randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization) (ASLR), which has been standard in many operating systems for a long time. OpenBSD takes randomization of a program's address space a step farther by also re-linking the sections of the C library in a random order on boot, meaning that an attack must determine not only the offset of the C library in memory, but also the offset of the specific code to which the attack wishes to jump within the library. Unfortunately, dynamically linked programs have to have this information in the symbol relocation table in order to allow for calls to the shared object. Therefore attacks that can construct a way to read memory can frequently leak enough offset information to circumvent these protections. De Raadt gave [a talk](https://twitter.com/i/broadcasts/1kvJpmkYkaZxE) (with [slides](https://www.openbsd.org/papers/csw2023.pdf)) about ROP mitigations in OpenBSD at CanSecWest in 2023, including several other protections designed to make leaking information about the contents of a program harder.

Unlike `pledge()`, this patch has the advantage of securing an application even if the developer does not make any special effort. However, this protection is most useful to programs that statically link OpenBSD's C library; programs that use dynamic linking will still have all of the system calls used by the C library in their address space. `pledge()` also permits dropping unnecessary permissions after startup, which allows applications to use a more restrictive set of permissions than a static defense like `pinsyscalls()` can permit.

This work was difficult to bring to completion. One of the largest obstacles were programs written in Go. In his announcement that the new work had been merged, De Raadt said: "<q>The direct-syscalls-inside-the-binary model used by go (and only go, noone else in the history of the unix software does this) provided the biggest resistance against this effort</q>". He thanked Joel Sing specifically for his work to make the Go ecosystem compatible with the changes.

Since Linux permits programs to make system calls directly, without going through a wrapper from a blessed C library, and is unlikely to change this policy, additional steps would be needed to incorporate a similar mechanism there. Some Linux programs make system calls directly in order to avoid depending on a specific C library, but others make system calls directly in order to use new features which have not yet been wrapped by the system's C library; OpenBSD doesn't have this problem since its C library and kernel are developed in lockstep.

#### Conclusion

OpenBSD has a long history of adding novel mitigations, some of which are adopted by other projects and some of which are not. This work seems unlikely to be adopted elsewhere, given the doubts around the practical benefit and the costs of adding additional complexity to how system calls are performed. This work does add another barrier to constructing a ROP attack on OpenBSD, however, and seems especially beneficial for statically linked programs that use only a few system calls and have not yet made use of pledge.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/959562/)

to post comments)