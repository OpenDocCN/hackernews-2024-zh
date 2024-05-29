<!--yml
category: 未分类
date: 2024-05-29 12:50:15
-->

# Stupid RCU Tricks: How Read-Intensive is The Kernel's Use of RCU? - Paul E. McKenney's Journal — LiveJournal

> 来源：[https://paulmck.livejournal.com/67547.html](https://paulmck.livejournal.com/67547.html)

RCU is a specialized synchronization mechanism, and is typically used where there are far more readers (

`rcu_read_lock()`

,

`rcu_read_unlock()`

,

`rcu_dereference()`

, and so on) than there are updaters (

`synchronize_rcu()`

,

`call_rcu()`

,

`rcu_assign_pointer()`

, and so on). But does the Linux kernel really make heavier use of RCU's read-side primitives than of its update-side primitives?

One way to determine this would be to use something like ftrace to record all the calls to these functions. This works, but trace messages can be lost, especially when applied to frequently invoked functions. Also, dumping out the trace buffer can perturb the syatem. Another approach is to modify the kernel source code to count these function invocations in a cache-friendly manner, then come up with some way to dump this to userspace. This works, but I am lazy. Yet another approach is to ask the tracing folks for advice.

This last is what I actually did, and because the tracing person I happened to ask happened to be Andrii Nakryiko, I learned quite a bit about BPF in general and the

`bpftrace`

command in particular. If you don't happen to have Andrii on hand, you can do quite well with Appendix A and Appendix B of Brendan Gregg's “BPF Performance Tools”. You will of course need to install

`bpftrace`

itself, which is reasonably straightforward on many Linux distributions.

## Linux-Kernel RCU Read Intensity

Those of you who have used

`sed`

and

`awk`

have a bit of a running start because you can invoke

`bpftrace`

with a

`-e`

argument and a series of tracepoint/program pairs, where a program is

`bpftrace`

code enclosed in curly braces. This code is compiled, verified, and loaded into the running kernel as a kernel module. When the code finishes executing, the results are printed right there for you on

`stdout`

. For example:

> ```
> bpftrace -e 'kprobe:__rcu_read_lock { @rcu_reader = count(); } kprobe:rcu_gp_fqs_loop { @gp = count(); } interval:s:10 { exit(); }'
> 
> ```

This command uses the

`kprobe`

facility to attach a program to the

`__rcu_read_lock()`

function and to attach a very similar program to the

`rcu_gp_fqs_loop()`

function, which happens to be invoked exactly once per RCU grace period. Both programs count the number of calls, with

`@gp`

being the

`bpftrace`

“variable” accumulating the count, and the

`count()`

function doing the counting in a cache-friendly manner. The final

`interval:s:10`

in effect attaches a program to a timer, so that this last program will execute every 10 seconds (“

`s:10`

”). Except that the program invokes the

`exit()`

function that terminates this

`bpftrace`

program at the end of the very first 10-second time interval. Upon termination,

`bpftrace`

outputs the following on an idle system:

> ```
> Attaching 3 probes...
> 
> @gp: 977
> @rcu_reader: 6435368
> 
> ```

In other words, there were about a thousand grace periods and more than six million RCU readers during that 10-second time period, for a read-to-grace-period ratio of more than six thousand. This certainly qualifies as read-intensive.

But what if the system is busy? Much depends on exactly how busy the system is, as well as exactly how it is busy, but let's use that old standby, the kernel build (but using the

`nice`

command to avoid delaying

`bpftrace`

). Let's also put the

`bpftrace`

script into a creatively named file

`rcu1.bpf`

like so:

> ```
> kprobe:__rcu_read_lock
> {
>         @rcu_reader = count();
> }
> 
> kprobe:rcu_gp_fqs_loop
> {
>         @gp = count();
> }
> 
> interval:s:10
> {
>         exit();
> }
> 
> ```

This allows the command

`bpftrace rcu1.bpf`

to produce the following output:

> ```
> Attaching 3 probes...
> 
> @gp: 274
> @rcu_reader: 78211260
> 
> ```

Where the idle system had about one thousand grace periods over the course of ten seconds, the busy system had only 274\. On the other hand, the busy system had 78 million RCU read-side critical sections, more than ten times that of the idle system. The busy system had more than one quarter million RCU read-side critical sections per grace period, which is seriously read-intensive.

RCU works hard to make the same grace-period computation cover multiple requests. Because

`synchronize_rcu()`

invokes

`call_rcu()`

, we can use the number of

`call_rcu()`

invocations as a rough proxy for the number of updates, that is, the number of requests for a grace period. (The more invocations of

`synchronize_rcu_expedited()`

and

`kfree_rcu()`

, the rougher this proxy will be.)

We can make the

`bpftrace`

script more concise by assigning the same action to a group of tracepoints, as in the

`rcu2.bpf`

file shown here:

```

kprobe:__rcu_read_lock,
kprobe:call_rcu,
kprobe:rcu_gp_fqs_loop
{
        @[func] = count();
}

interval:s:10
{
        exit();
}

```

With this file in place,

`bpftrace rcu2.bpf`

produces the following output in the midst of a kernel build:

```

Attaching 4 probes...

@[rcu_gp_fqs_loop]: 128
@[call_rcu]: 195721
@[__rcu_read_lock]: 21985946

```

These results look quite different from the earlier kernel-build results, confirming any suspicions you might harbor about the suitability of kernel builds as a repeatable benchmark. Nevertheless, there are about 180K RCU read-side critical sections per grace period, which is still seriously read-intensive. Furthermore, there are also almost 2K

`call_rcu()`

invocations per RCU grace period, which means that RCU is able to amortize the overhead of a given grace period down to almost nothing per grace-period request.

## Linux-Kernel RCU Grace-Period Latency

The following

`bpftrace`

program makes a histogram of grace-period latencies, that is, the time from the call to

`rcu_gp_init()`

to the return from

`rcu_gp_cleanup()`

:

```

kprobe:rcu_gp_init {
        @start = nsecs;
}

kretprobe:rcu_gp_cleanup {
        if (@start) {
                @gplat = hist((nsecs - @start)/1000000);
        }
}

interval:s:10 {
        printf("Internal grace-period latency, milliseconds:\n");
        exit();
}

```

The

`kretprobe`

attaches the program to the return from

`rcu_gp_cleanup()`

. The

`hist()`

function computes a log-scale histogram. The check of the

`@start`

variable avoids a beginning-of-time value for this variable in the common case where this script start in the middle of a grace period. (Try it without that check!)

The output is as follows:

```

Attaching 3 probes...
Internal grace-period latency, milliseconds:

@gplat: 
[2, 4)               259 |@@@@@@@@@@@@@@@@@@@@@@                              |
[4, 8)               591 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[8, 16)              137 |@@@@@@@@@@@@                                        |
[16, 32)               3 |                                                    |
[32, 64)               5 |                                                    |

@start: 95694642573968

```

Most of the grace periods complete within between four and eight milliseconds, with most of the remainder completing within between two and four milliseconds and then between eight and sixteen milliseonds, but with a few stragglers taking up to 64 milliseconds. The final

`@start`

line shows that

`bpftrace`

simply dumps out all the variables. You can use the

`delete(@start)`

function to prevent printing of

`@start`

, but please note that the next invocation of

`rcu_gp_init()`

will re-create it.

It is nice to know the internal latency of an RCU grace period, but most in-kernel users will be more concerned about the latency of the

`synchronize_rcu()`

function, which will need to wait for the current grace period to complete and also for callback invocation. We can measure this function's latency with the following

`bpftrace`

script:

```

kprobe:synchronize_rcu {
        @start[tid] = nsecs;
}

kretprobe:synchronize_rcu {
        if (@start[tid]) {
                @srlat = hist((nsecs - @start[tid])/1000000);
                delete(@start[tid]);
        }
}

interval:s:10 {
        printf("synchronize_rcu() latency, milliseconds:\n");
        exit();
}

```

The

`tid`

variable contains the ID of the currently running task, which allows this script to associate a given return from

`synchronize_rcu()`

with the corresponding call by using

`tid`

as an index to the

`@start`

variable.

As you would expect, the resulting histogram is weighted towards somewhat longer latencies, though without the stragglers:

```

Attaching 3 probes...
synchronize_rcu() latency, milliseconds:

@srlat: 
[4, 8)                 9 |@@@@@@@@@@@@@@@                                     |
[8, 16)               31 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[16, 32)              31 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

@start[4075307]: 96560784497352

```

In addition, we see not one but two values for

`@start`

. The

`delete`

statement gets rid of old ones, but any new call to

`synchronize_rcu()`

will create more of them.

## Linux-Kernel Expedited RCU Grace-Period Latency

Linux kernels will sometimes executed

`synchronize_rcu_expedited()`

to obtain a faster grace period, and the following command will further cause

`synchronize_rcu()`

to act like

`synchronize_rcu_expedited()`

:

> ```
> echo 1 >  /sys/kernel/rcu_expedited
> 
> ```

Doing this on a dual-socket system with 80 hardware threads might be ill-advised, but you only live once!

Ill-advised or not, the following

`bpftrace`

script measures

`synchronize_rcu_expedited()`

latency, but in microseconds rather than milliseconds:

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat = hist((nsecs - @start[tid])/1000);
>                 delete(@start[tid]);
>         }
> }
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

The output of this script run concurrently with a kernel build is as follows:

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat: 
> [128, 256)            57 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
> [256, 512)            14 |@@@@@@@@@@@@                                        |
> [512, 1K)              1 |                                                    |
> [1K, 2K)               2 |@                                                   |
> [2K, 4K)               7 |@@@@@@                                              |
> [4K, 8K)               2 |@                                                   |
> [8K, 16K)              3 |@@                                                  |
> 
> @start[4140285]: 97489845318700
> 
> ```

Most

`synchronize_rcu_expedited()`

invocations complete within a few hundred microseconds, but with a few stragglers around ten milliseconds.

But what about linear histograms? This is what the

`lhist()`

function is for, with added minimum, maximum, and bucket-size arguments:

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat = lhist((nsecs - @start[tid])/1000, 0, 1000, 100);
>                 delete(@start[tid]);
>         }
> }
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

Running this with the usual kernel build in the background:

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat: 
> [100, 200)            26 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
> [200, 300)            13 |@@@@@@@@@@@@@@@@@@@@@@@@@@                          |
> [300, 400)             5 |@@@@@@@@@@                                          |
> [400, 500)             1 |@@                                                  |
> [500, 600)             0 |                                                    |
> [600, 700)             2 |@@@@                                                |
> [700, 800)             0 |                                                    |
> [800, 900)             1 |@@                                                  |
> [900, 1000)            1 |@@                                                  |
> [1000, ...)           18 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                |
> 
> @start[4184562]: 98032023641157
> 
> ```

The final bucket is overflow, containing measurements that exceeded the one-millisecond limit.

The above histogram had only a few empty buckets, but that is mostly because the 18 

`synchronize_rcu_expedited()`

instances that overflowed the one-millisecond limit are consolidated into a single

`[1000, ...)`

overflow bucket. This is sometimes what is needed, but other times losing the maximum latency can be a problem. This can be dealt with given the following

`bpftrace`

program:

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat[(nsecs - @start[tid])/100000*100] = count();
>                 delete(@start[tid]);
>         }       
> }       
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

Given the usual kernel-build background load, this produces the following output:

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat[1600]: 1
> @srelat[500]: 1
> @srelat[1000]: 1
> @srelat[700]: 1
> @srelat[1100]: 1
> @srelat[2300]: 1
> @srelat[300]: 1
> @srelat[400]: 2
> @srelat[600]: 3
> @srelat[200]: 4
> @srelat[100]: 20
> @start[763214]: 17487881311831
> 
> ```

This is a bit hard to read, but simple scripting can be applied to this output to produce something like this:

> ```
> 100: 20
> 200: 4
> 300: 1
> 400: 2
> 500: 1
> 600: 3
> 700: 1
> 1000: 1
> 1100: 1
> 1600: 1
> 
> ```

This produces compact output despite outliers such as the last entry, corresponding to an invocation that took somewhere between 1.6 and 1.7 milliseconds.

## Summary

The

`bpftrace`

command can be used to quickly and easily script compiled in-kernel programs that can measure and monitor a wide variety of things. This post focused on a few aspects of RCU, but quite a bit more material may be found in Brendan Gregg's “BPF Performance Tools” book.