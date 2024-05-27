<!--yml
category: 未分类
date: 2024-05-27 12:48:53
-->

# Inside the failed attempt to backdoor SSH globally — that got caught by chance | by Kevin Beaumont | Mar, 2024 | DoublePulsar

> 来源：[https://doublepulsar.com/inside-the-failed-attempt-to-backdoor-ssh-globally-that-got-caught-by-chance-bbfe628fafdd](https://doublepulsar.com/inside-the-failed-attempt-to-backdoor-ssh-globally-that-got-caught-by-chance-bbfe628fafdd)

# Q&A

**Do I need to panic?** No. As an assurance piece, orgs can check they aren’t using the latest unstable releases of Debian and Fedora — but they very likely aren’t.

Andres caught the problem — which has allowed it to be evicted from the Linux distribution ecosystem before any stable releases.. released.

How did Andres find the problem? Well, let us go to Andres:

> “I was doing some micro-benchmarking at the time, needed to quiesce the system to reduce noise. Saw sshd processes were using a surprising amount of CPU, despite immediately failing because of wrong usernames etc. Profiled sshd, showing lots of cpu time in liblzma, with perf unable to attribute it to a symbol. Got suspicious. Recalled that I had seen an odd valgrind complaint in automated testing of postgres, a few weeks earlier, after package updates.
> 
> Really required a lot of coincidences.
> 
> One more aspect that I think emphasizes the number of coincidences that had to come together to find this:
> 
> I run a number “buildfarm” instances for automatic testing of postgres. Among them with valgrind. For some other test instance I had used -fno-omit-frame-pointer for some reason I do not remember. A year or so ago I moved all the test instances to a common base configuration, instead of duplicate configurations. I chose to make all of them use -fno-omit-frame-pointer.
> 
> Afaict valgrind would not have complained about the payload without -fno-omit-frame-pointer. It was because _get_cpuid() expected the stack frame to look a certain way.
> 
> Additionally, I chose to use debian unstable to find possible portability problems earlier. Without that valgrind would have had nothing to complain.
> 
> Without having seen the odd complaints in valgrind, I don’t think I would have looked deeply enough when seeing the high cpu in sshd below _get_cpuid().”

A couple of points here. First of all, the world owes Andres unlimited free beer. He just saved everybody’s arse in his spare time. I am not joking.

Nobody else had raised concerns, and I don’t believe any existing security tooling or processes would have caught this (I realise there will be a torrent of vendors claiming they detect this… but they will detect this *now that somebody told them*).

Because Andres privately researched the issue and got the Linux distributions to take it seriously, he averted this reaching any kind of wide (or even small) deployment in the real world.

Also, Andres had a unique testing environment and a set of coincidental setup issues which allowed him to discover the issue. I don’t know of anybody else has this setup.

When I installed a vulnerable Linux box, I had to double check it was actually vulnerable as I wouldn’t even see a speed issue. For me, it was a completely transparent backdoor — where sshd was running from disk as usual, with the usual file hash and no extra network activity.

**How advanced was the threat actor?** The backdoor attempt was a very serious one, with a very high bar of knowledge, research, development and tradecraft to reach this far into the Linux ecosystem. Additionally, changes made by the threat actor on Github span multiple years, and include things like introducing functions incompatible with OSS Fuzzer due to outstanding small issues since 2015, then getting OSS Fuzzer to exclude XZ Utils from scanning last year. The backdoor itself is super well put together, and even includes the ability to remotely deactivate and remove the backdoor via a kill command. Several days in, despite global focus, I haven’t seen anybody who has finished reverse engineering it.

**Who did it?** Your mum. Just kidding, it was GCHQ in Cheltnam. Just kidding, it was Russia. Just kidding, it was China. Just kidding, it was America. Just kidding, it was definitely your mum.

**We should stop using open source and only buy American vendor products!** Yeah, good luck with that.

**We should start funding every open source project!** Yeah, good luck with that.. I’ll start saving for my trip to Mars.

**This is a wake up call around cybersecurity!** People will have forgotten about it in a week. Except for LinkedIn, who will still be talking in an echo chamber about supply chain in four decades.

**So what can we do?** First of all, if you rely on *any* software, you have a risk of insider threat. Be it open source, closed source, or your own in house developers — if people add backdoors and nobody notices, that’s a problem.

Secondly, ***a core issue here is systemd in Linux***. libsystemd is linked to all systemd servies, which opens a rich attack surface of third party services to backdoor. That is what the threat actor abused here. It mean they didn’t need to backdoor systemd, which has a rich development community paying attention — instead they relied on liblzma loading XZ, which is much further down the chain, where nobody was paying attention.

There’s a plot twist. This issue request in systemd was opened today:

But the fix for this was *already in train before the XZ issue was highlighted*, and long before the Github issue. The fix stopped the XZ backdoor into SSH, but hadn’t yet rolled out into a release of systemd.

I believe there’s a good chance the threat actor realised this, and began rapidly accelerated development and deployment, hence publicly filing bug reports to try to get Ubuntu and such to upgrade XZ, as it was about to spoil several years of work. It also appears this is when they started making mistakes.

**We’re all doomed!!!1! The supply chain will end the world!11! I really want to panic!!!** No. Panic helps nobody. It’s also important we keep things in perspective, e.g. this one was spotted and averted.

Let’s just keep doing the good SBOM work at CISA, and stop doing stunts around Huawei and such — Huawei is a speck of dust compared to the issues around tens of thousands of unpaid developers writing the core of the world’s most critical infrastructure nowadays.

Work around tightening systemd and other core Linux dependencies needs to continue. There’s probably more that needs to be done around EDR detection on Linux, e.g. injection into ssh that nobody detected? Detect it. In truth, Linux EDR products are significantly less robust than Windows EDR products. Ask me how I know.

The reality is that all development has a risk of insider account abuse, and that includes open source.

We should also acknowledge that open source developers are largely unpaid, and face significant amounts of online abuse from users of the software. This is not the first time an open source package has been hijacked after a maintainer was added – it actually happens all the time in Python repositories and such, and has been one of the leading causes of infostealers and coin miners in development pipelines. It is absolutely not a surprise that somebody is targeting open source compression libraries that systemd loads.. and it is also sadly not a surprise that people online bully the creators of these libraries, either.

There are no easy fixes.. we should just try to reduce the risk and calmly work some solutions.

Other reading: