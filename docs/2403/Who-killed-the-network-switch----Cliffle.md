<!--yml
category: 未分类
date: 2024-05-29 12:40:57
-->

# Who killed the network switch? - Cliffle

> 来源：[https://cliffle.com/blog/who-killed-the-network-switch/](https://cliffle.com/blog/who-killed-the-network-switch/)

# Who killed the network switch?

A Hubris Bug Story

2024-03-24

We found a neat bug in [Hubris](https://hubris.oxide.computer/) this week. Like many bugs, it wasn’t a bug when it was originally written — correct code *became* a bug as other things changed around it.

I thought the bug itself, and the process of finding and fixing it, provided an interesting window into our development process around Hubris. It’s very rare for us to find a bug in the Hubris kernel, mostly because it’s so small. So I jumped at the opportunity to write this one down.

This is a tale of how two features, each useful on its own, can combine to become a bug. Read on for details.

 ## What’s a Hubris?

Hubris is an operating system designed for deeply embedded systems — all the computers that you don’t think of as “computers,” like the one inside a keyboard. We wrote it to handle all the tasks required to start up the *big* processors — the part you *do* think of as “computers” — in the Oxide Rack.

Hubris is pretty weird. I’ll explain the bits of that are relevant to this story below. If you’d like more background, I recommend reading either

## The scene of the crime

Like any good murder mystery, let’s start with the victim.

My colleague Arjen Roodselaar is largely responsible for the firmware that runs on our network switches at Oxide. He was testing a change he’d made to power sequencing and clock configuration — the very important bits that are responsible for turning everything else on. After a seemingly innocuous change, suddenly, the switch wouldn’t power on.

Curiously, *parts of the firmware* would respond to inquiries, but the really important part — the power supply sequencer — seemed to be dead in its tracks. Without power, the network switch was just a rather large, very heavy paperweight.

Messing up power sequencing can literally fry the hardware. Was the switch dead or merely unresponsive? And how did an apparently unrelated code change, to a different part of the system, kill power sequencing?

A mystery!

Alright, with that stage set, let’s flash-back to talk about a couple of Hubris design details.

## Squeezing more out of limited RAM

On the relatively inexpensive microcontrollers we use with Hubris, one of the challenges is that RAM (and flash, for storing code) is quite limited. For instance, we have an internal board that serves as a useful I2C debugging probe and has 8 kiB of RAM and 32 kiB of flash. The system you’re reading this on, even if it’s an inexpensive phone, might have a million times that.

Hubris faces this challenge a bit more than other operating systems in our niche, because firmware using Hubris is built out of many separately-compiled programs called *tasks.* Each task gets its own copy of everything it needs, including common standard library code. This means our systems tend to have somewhat higher resource requirements than others (though not by as much as you might expect — this will be the topic of another post).

We isolate tasks from each other using the hardware memory protection unit, which keeps them from crashing or corrupting each other. This adds some pressure on our RAM and flash requirements, though. In the ARM microcontrollers that we mostly use to run Hubris — the older members of the Cortex-M family, aka the ARMv7-M architecture — any protected region of memory has to be a *power of two* in size, and aligned according to its size. This means if you have a region that is currently 1024 bytes, and you need one more byte, you can’t just grow it to 1025 — it is now 2048 bytes whether you like it or not.

Originally, Hubris used one memory protection region for a task’s RAM, and one for its flash. Simple, but wasteful. This encouraged [fragmentation](https://en.wikipedia.org/wiki/Fragmentation_(computing)), which in practice meant that there was often unused RAM and flash left between tasks that didn’t quite line up. That unused memory was basically wasted. This is really frustrating when you don’t have enough of it to begin with!

My colleague [Matt Keeter](https://www.mattkeeter.com/blog/) fixed this recently by making the system smarter: it now attempts to pack tasks using multiple power-of-two regions where possible. (“Where possible” is important, because the hardware limits each task to no more than eight regions, total.) In some of our firmware images, this recovered 30% of RAM! Our smallest devices went from being so crammed full that I regularly had to make optimization passes over the code, to having free space to spare.

This is incredibly awesome, and he’s written [an entire post about it](https://www.mattkeeter.com/blog/2024-03-25-packing/). But how is it relevant? I’ll explain in a bit. But first:

## The smoking gun

Arjen reached for Humility, our debugger for Hubris, and gently probed the failed network switch. The service processor responsible for power sequencing appeared to be alive and running, so a hardware problem was unlikely.

One of the first commands we tend to reach for to learn about the state of a running Hubris system is `humility tasks`. This prints a list of tasks running on the processor, as well as information about their status. When Arjen ran `humility tasks`, one line in particular jumped out: the `sequencer` task, which is the component responsible for power sequencing, had the following status:

```
mem fault (precise: 0x801bffd) in syscall (was: wait: reply from i2c_driver/gen0) 
```

The task also indicated that it had been restarted 115 times. Since we pretty much only restart tasks in response to crashes, that suggested that this fault — whatever it was! — was occurring every time the sequencer tried to power the system on.

So the task was getting killed. But who pulled the trigger?

The answer involves an important aspect of Hubris IPC.

## Extending Rust borrows across tasks in Hubris IPC

Hubris tasks can communicate with each other using *messages* in an Inter-Process Communication scheme, or IPC. Each message looks and behaves very much like a function call: the task sending the message stops, the task receiving it takes control of the CPU, and eventually returns some result that causes the sender to wake back up.

We chose this scheme for a bunch of reasons — there are, after all, many other options! — but one of the most useful ones is that it plays nicely with Rust’s resources ownership model. Just as a function can loan some memory it controls to a function it calls, and reliably get ownership back when the function returns, a task on Hubris can loan some of its memory to another task along with an IPC message.

This feature is very widely used in Hubris-based firmware. For example, tasks that want to interact with an I2C device loan sections of their memory to the I2C bus driver, which then reads and/or writes them in-place. This keeps the bus driver itself from needing to have a buffer pool or similar resource internally, and reduces the number of copies needed to move data from place to place.

It’s also a potential security hole if implemented incorrectly. Much of Hubris’s reliability comes from the *isolation* of tasks; tasks don’t share any memory, so one task can do arbitrarily silly things in its own space without hurting any other tasks. So it’s very important that loaning memory via IPC doesn’t let tasks break out of their isolated containers.

And so, attempting to loan memory that you don’t actually *own* is bad. Specifically, it’s forbidden by the Hubris kernel.

If a caller attempts to loan memory it can’t access to a server, once this is discovered, the server is handed an apologetic error code by the kernel, while the client receives a fault. To the server, this is never fatal; to the client, it is *always fatal.* We assume that the access violation indicates a bug, corruption, or exploit in the sending task, so we immediately shut it down without giving it an opportunity to respond.

When the kernel does this, it records information about the triggering event, which (when formatted for human consumption) produces a status like the one Arjen found on the sequencer task. Let’s take it apart:

```
mem fault (precise: 0x801bffd) in syscall (was: wait: reply from i2c_driver/gen0) 
```

This says:

*   `mem fault`, because the task was faulted for incorrect handling of memory;
*   `precise`, because we can tell the specific address that was handled incorrectly;
*   `in syscall`, because the task wasn’t actually *running* when the fault occurred; and
*   `was: wait: reply from i2c_driver/gen0`, because at the time the fault occurred, instead of running, the task was waiting for a reply from a message it sent to the I2C driver. We always try to record the task state *just before* the fault happened to help with debugging.

(If you’re curious, the `gen0` means the `i2c_driver` is on generation 0, which means it hasn’t ever crashed. `sequencer` was on generation 115!)

The kernel distinguishes between *real* and *synthetic* faults. A real fault happens when a task does something simple and wrong, such as dereferencing a null pointer, or trying to write to its code region. Real faults are generated by the hardware (in this case, the memory protection unit) in response to actions performed by a task. As a result, they can only happen when the task is actually running and doing something wrong.

Rather than *hardware* rules, synthetic faults represent violations of *software* rules. The processor has no concept of “IPC” or “loaning memory across tasks,” those are ideas that Hubris added to the system. They have their own rules, and if a task violates them, we treat it just like a null pointer dereference or other “real” hardware fault — just with a different set of fault information that marks it as synthetic.

So, from this rather dense status report, we have learned the following:

*   The `sequencer` task broke the kernel’s rules for memory access.
*   It did so at the address `0x801bffd`, which is an unusual but valid address in flash for the processor.
*   It did so as part of an IPC sent to the I2C driver.

This seems really bad! Memory access violations usually indicate a very serious program bug or data corruption. The fact that one was happening in the sequencer task, despite that task not having changed much recently, suggested that we had some kind of long-hidden data corruption bug!

This would be very unusual — since our firmware is almost entirely safe Rust, data corruption bugs are incredibly rare in our systems. But all signs point to data corruption here.

…or do they?

## When features attack

The address reported for the memory violation was `0x801bffd`.

When Arjen called me in for assistance, this immediately looked weird. It’s a valid flash address, sure, but it’s three bytes below a power of two boundary. We knew this wasn’t a result of an accidental integer overflow in an address computation, because Hubris systems enable runtime integer overflow checks — the task would have crashed and reported the problem.

And that’s when I remembered Matt’s packing change.

Recall that the MPU on our processor requires memory regions to be power-of-two sized and aligned. I used `humility mem` to print the image’s memory layout, and sure enough, there was a region ending at `0x801c000`, and another one starting at the same address.

And they both belonged to the *same task.*

```
LOW         HIGH           SIZE ATTR   ID TASK 0x08018000 - 0x0801bfff   16kiB r-x--- 17 sequencer 0x0801c000 - 0x0801dfff    8kiB r-x--- 17 sequencer 
```

This is perfectly fine, except when it’s not.

The memory access permission checking code is one of the most security-critical parts of the Hubris kernel. We try to keep the most important code as simple as possible. As I mentioned above, early in Hubris’s life — which was when I wrote this part of the kernel — tasks had a single region for RAM and a single region for Flash, and that’s that.

And I simplified the kernel checking code to reflect that. It was:

```
self.region_table().iter().any(|region| {  region.covers(slice) && region.attributes.contains(desired) && !region.attributes.intersects(forbidden) }) 
```

An access was permitted if, and only if, there was any single region in the task’s region table that

1.  Completely covered the slice of memory being loaned,
2.  Had all the required attributes (e.g. it was writable if the IPC said it should be writable), and
3.  Did not have any “forbidden” attributes, like containing memory-mapped registers or being used for hardware DMA. (Such memory cannot currently be loaned, for subtle reasons that are out of scope for this post.)

At the time that code was written, it was correct, but it embodied the assumption that any loaned memory would fit into *one* region.

That assumption became obsolete the moment that Matt implemented task packing, but we didn’t notice. This code, which was still simple and easy to read, was now also *wrong.*

Critically, this code is only used to check loaned memory. Normal accesses from a running program are checked by the hardware memory protection unit directly, and the hardware doesn’t have this bug. This means the task probably had no trouble at all accessing memory across this region boundary… until it tried to loan it out.

## How two innocent features conspired to kill the network switch

Task packing in our build system operates *opportunistically.* Because the MPU hardware limits each task to no more than 8 regions, and because many tasks represent hardware drivers that have various memory-mapped registers accessible through some of those regions, we don’t always have enough region slots left over in the 8-region table to be clever about task layout.

But whenever we *can*, we attempt to pack the tasks using multiple regions.

The net effect of this is that region boundaries are now being introduced in the middle of task flash and RAM regions. Critically, they’re appearing at places that are *very difficult for the author of the task itself to predict.* The layout of tasks in memory, and the places where those boundaries fall, depend on the size required for each task.

This means an apparently innocuous change in task A, if it changes task A’s size very slightly, can now move the positions of MPU region boundaries in unrelated task B.

It was now pretty clear that this behavior was bad. Apparently random crashes that go away when you add some debugging code are the *worst kind* of crash, but that was the situation we were in — since adding any code to the system would shift the allocation decisions and the region boundaries, potentially angering the kernel.

Matt immediately switched off the task packing feature in the build system, which let Arjen build a working firmware image and continue with his network switch hacking. Meanwhile, I [started writing up my analysis on a bug report](https://github.com/oxidecomputer/hubris/issues/1672), and prepared to face this murderer I had accidentally created four years earlier.

## The call is coming from inside the house!

Since the kernel was now killing tasks in a misguided attempt to enforce memory protection, the memory protection algorithm needed to change. A previously appealing *simplification* was now an *over-simplification.*

The basic *intent* of the algorithm remained the same as when I originally wrote it in 2020:

1.  Let tasks specify sections of memory they want to loan with a message.

2.  Require that the task *actually has* the access to that memory that it claims. For instance, if it’s attempting to loan some memory writable, it had better be able to write the memory itself! Otherwise, the mechanism could let tasks gain powers they aren’t supposed to have.

The only thing that’s different is that we now need to tolerate the loaned memory crossing MPU regions, as long as those regions are exactly adjacent.

The [replacement algorithm](https://github.com/oxidecomputer/hubris/blob/b44e677fb39cde8be5b10bbf78a9f26c000f6ad6/sys/kerncore/src/lib.rs#L103) is significantly more complex than the original, because it’s taking pains to perform a single pass over the region table. We try really hard in Hubris to not expose operations that have *task-controlled time complexity.* We’re not perfect at it, but we try. So it was important to me that the algorithm’s performance depend only on the size of the *region table* — fixed at 8 — rather than the amount of memory being loaned. It’s possible to do this in a single pass if you know in advance that the table is in sorted address order; I altered the build system to ensure that this was always true. You can read [the commit](https://github.com/oxidecomputer/hubris/commit/b44e677fb39cde8be5b10bbf78a9f26c000f6ad6#diff-0270185909262248f95c960a9fdd6074669172623c61f59781ee47b50ddc9c69R156) if you want the gritty details, but the important bit was:

```
 regions.sort_by_key(|i| region_table.get_index(*i).unwrap().1.base); 
```

…added to the kernel’s `build.rs` file.

Because the new algorithm is much more complex, my colleague [Eliza Weissman](https://elizas.website/) nudged me into factoring it out of the core Hubris kernel, and into a [more portable crate](https://github.com/oxidecomputer/hubris/tree/b44e677fb39cde8be5b10bbf78a9f26c000f6ad6/sys/kerncore) where it can be more easily unit-tested. It currently has unit tests for the corner cases we’re most concerned about; we plan to add a few more, in the words of Kent Beck, “until fear turns to boredom.”

When I say the code is “much more complex,” I mean relative to Hubris’s original code that performed the same function; it’s still much simpler than the equivalent code in most operating systems. Here’s all of it, with the comments from the original. (Note that `slice` here is not a Rust slice in the kernel, but a struct representing a *task-proposed slice* by address and size.)

```
let mut scan_addr = slice.base_addr(); let end_addr = slice.end_addr();   for region in table {  if region.contains(scan_addr) {  if !region_ok(region) {   break;  }   if end_addr <= region.end_addr() {    return true;  }    scan_addr = region.end_addr();  } else if region.base_addr() > scan_addr {    break;  } }   false 
```

With this new code, we can turn task packing back on without handing task developers a ticking time-bomb.

In all, about three hours had passed from the time Arjen noticed the network switch malfunctioning, until the kernel bug was fixed. I hacked on the unit tests the next morning.

## Failing with Hubris

For me, the interesting thing about this story is all the stuff that didn’t happen. Or more specifically, the way the system failed, and the ways in which it didn’t.

We went from “network switch won’t turn on with new firmware,” to two engineers 3,000 miles apart separately analyzing snapshots of the failure, to having the kernel bug fixed, in about three hours. I’ve easily spent longer than that chasing a *single* memory corruption bug in other firmware. I think the difference comes down to the following things.

**Fault isolation.** We had a crash during power sequencing on a *very* complex piece of hardware. But only part of the system crashed. The network switch firmware consists of 23 isolated components (tasks). Some of them have dependencies on power sequencing at various points in their lifecycle, of course — power sequencing is very important! — but for the most part they kept working through the *one hundred and fifteen crash and restart attempts.* This includes:

*   The in-system firmware update system.
*   The IP network stack providing the management and control interface.
*   Several network services, from a basic `echo` protocol implementation up through our rack control plane interface.
*   I2C, SMBus, and PMBus to all the sensors, fans, and miscellanea monitoring the system’s resource usage and physical health.
*   Drivers for 32 QSFP 100-gigabit transceivers across the front of the switch.

…and more. They kept working because of the fault isolation that Hubris provides, which manifested here in two ways. First, the sequencer task was able to crash without disrupting state in any other task, hardware peripheral, or the kernel. Second, the Hubris IPC mechanism is specifically designed under the assumption that other tasks may fail, and allows operations marked “idempotent” to be transparently retried, letting clients decouple themselves from crashing servers.

**Failing toward safety.** In Hubris, both the implementation and the APIs try to maintain *one-sided error* — preventing some correct programs, rather than allowing some incorrect programs. The original memory access check algorithm was blocking accesses by a correct program, *not* admitting invalid accesses by a wrong or malicious program. This means it’s a rare case of a kernel memory access check bug that had no security implications. We’ve tried to maintain this throughout the stack, by comprehensively parsing all input as untrusted and making any undefined usage a fault. As a result, when the system fails, it tends to fail in non-exploitable ways.

**Safer shared memory.** Because of how we designed the IPC memory loan mechanism, even though the sequencer task and the I2C driver were effectively sharing memory at the time the sequencer crashed, the I2C driver was at no risk of corruption, and could go on with its business. (It never did restart, in the end.) This is because we assumed that a task acting as a server (like the I2C driver, here) will typically serve more than one client, and so needs to be made as robust as possible to arbitrary mistakes by clients. Including, in this case, (apparent) memory access violations.

**Kernel-debugger codesign.** My colleague Bryan Cantrill wrote the code that became Humility, our debugger, at the same time that I was writing the code that would become the Hubris kernel. The two programs have grown and changed together, and I wouldn’t have it any other way. Thanks to Humility, Arjen was able to identify the crashing code (down to the line number) within the first few minutes, and take a self-contained snapshot of the service processor, which he posted in a chat channel for Matt and I to pore over.

**Firmware crash dumps are really great.** I never had physical access to the crashing network switch, and in fact I’m not actually sure where in the country it’s located. It doesn’t really matter; Humility can dump a consistent snapshot of our embedded systems when *at least one person* has physical access, and Hubris itself records compressed coredumps of crashing tasks into RAM where we can retrieve them over the network. This means we can still get crash dumps despite not having writable persistent storage, such as flash.

**Simplicity of design and implementation.** The concepts provided by the Hubris kernel are simple — so simple, in fact, that I was initially concerned that they couldn’t be used to make useful production firmware. (Thankfully I was wrong!) The IPC mechanism has only three operations and no optional or fancy parts, so when a fault points to IPC, there are not a lot of places you might need to hunt for the problem.

But if you *did* have to hunt in a lot of places for the problem, the architecture-independent part of the Hubris kernel is currently 1,789 lines of code. That’s about four copies of this post. In a pinch, a single person could read through the entire kernel in search of a bug.

We haven’t had to do that yet, but it’s nice to know that we could if we had to!

**Tight nonhierarchical integration of the team.** This isn’t a Hubris feature, but it’s hard to separate Hubris from the team that built it. Oxide’s engineering team has essentially no internal silos. Our culture rewards openness, curiosity, and communication, and discourages defensiveness, empire-building, and gatekeeping. We’ve worked hard to create and defend this culture, and I think it shows in the way we organized horizontally, across the borders of what other organizations would call teams, to solve this mystery.

If I hadn’t been in the office that day, somebody else would have found and fixed the bug in my place — if not me it would have been Matt, or Laura Abbott, or Eliza, or Arjen, or Bryan, or any of several other people. This is really important to me. They would have done so without fear of reprisal; at most, they might have done it slightly slower, since I wrote the original buggy code and knew where the bodies were buried — but not dramatically slower. There is a real benefit to keeping critical code simple, well-commented, easy to reason about, and accessible — and there’s a real benefit to maintaining a team culture where *everyone* can take advantage of that.

Incidentally, we could use more help; at the time of this writing, we’re hiring more software folk. For details, [see the job description on our site](https://oxide.computer/careers/software-engineer).

[#api-design](https://cliffle.com/tags/api-design/) [#dayjob](https://cliffle.com/tags/dayjob/) [#embedded](https://cliffle.com/tags/embedded/) [#rust](https://cliffle.com/tags/rust/) [#security](https://cliffle.com/tags/security/)