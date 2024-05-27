<!--yml
category: 未分类
date: 2024-05-27 13:22:53
-->

# Some Volumes Were Slow And We Figured Out Why - Fresh Produce - Fly.io

> 来源：[https://community.fly.io/t/some-volumes-were-slow-and-we-figured-out-why/19394](https://community.fly.io/t/some-volumes-were-slow-and-we-figured-out-why/19394)

## A Bug Report

Fly Volumes are fast. That sounds like a brag, but the truth is, we made tradeoffs to end up with fast Volumes. We back them with a pool of locally-attached NVMe drives, which means they’re pinned to specific physical servers, and while we do back them up, you generally want to be doing something at an upper layer to replicate them. They can lose data! But, the flip side is: they’re very fast.

So it was jarring, earlier this week, to get reports from folks experiencing what appeared to be I/O performance problems. We make it easy to spot I/O issues: you can just click out from our dashboard to `Metrics`, and look at the `I/O Utilization` percentage, which should be low.

One tricky thing about doing infra ops for a public cloud is that every possible thing can go wrong. Our customers exercise our hardware in every conceivable way. A performance problem could be on our side, or it could be an app stuck in an expensive tight loop. We started digging, but didn’t see any patterns.

Then [our metrics cluster started dragging.](https://status.flyio.net/incidents/2sl3hm7h8n3h) Well, we’re confident in the performance envelope of that system. We built it to scale. And we were seeing the Fly Machines running it grinding to a halt. Our digging gained some urgency.

Then a customer reported the same grinding halt. We worked around the problem with them, by re-creating machines (hold that thought, it’s relevant later). But something was obviously up: as the [saying goes](https://gutenberg.ca/ebooks/flemingi-goldfinger/flemingi-goldfinger-00-h-dir/flemingi-goldfinger-00-h.html#:~:text=Mr%20Bond%2C%20they%20have%20a%20saying%20in%20Chicago%3A%20%22Once%20is%20happenstance.%20Twice%20is%20coincidence.%20The%20third%20time%20it%27s%20enemy%20action.), “Once is happenstance. Twice is coincidence. Three times is an incident.”

### Digging deeper

The way [Fly.io](http://Fly.io) works is, an orchestrator service on our physical servers, called `flyd`, spawns and manages KVM virtual machines using Firecracker and Cloud-Hypervisor. Both are lightweight and super fast, we we run lots and lots of them on any given physical server.

We know that customer Fly Machines overwhelmingly aren’t having problems, but we also know something is up. We want to catch a Fly Machine in the act. So we set up a tiny script to alert us when any Firecracker process gets stuck in “uninterruptible sleep” (`D`) for more than a few seconds. That’s the sign that the process is stalling on I/O. Stalling for multiple seconds, or even just coming up in `D` state multiple times back to back, means something’s off.

Stalking a few of these troublesome processes gives us candidates to inspect. Linux makes this easy: you just `cat /proc/$pid/stack`. Here’s what we see:

```
[<0>] blk_io_schedule+0x22/0x40
[<0>] __blkdev_direct_IO_simple+0x222/0x320
[<0>] blkdev_direct_IO+0x71/0x80
[<0>] generic_file_read_iter+0x9c/0x150 
```

This is a normal stack trace: a process waiting for the completion of a scheduled I/O operation. But every process we looked at had the exact same trace. That’s a smell: `blk_io_schedule` should be super fast, unless something is getting in the way. This is like catching a whole room full of people blinking at exactly the same time.

### The obvious question

What changed?

Our hardware didn’t change. And there’s no correlation of these events in particular regions. We’re in the middle of a round of Firecracker version updates (ironically, to improve I/O performance!) but the new version is feature flagged. And we haven’t changed how volumes work.

Except we did, and didn’t realize it.

Earlier this week, we shipped a small feature to allow [first class support for swap devices](https://community.fly.io/t/a-new-way-to-rootfs/19196/2). This has the effect of moving swap off the root filesystem device and onto a dedicated swap device.

The thing about swap is, if you’re using it, you’re probably using it pretty hard. Because of the way swap memory works, mostly invisibly to your apps, it can get pretty thrashy, as the OS lies to your dev framework about how much memory there is and covers its tracks with disk operations.

So: we limit the IO bandwidth swap devices. This is good. Swap is available, to keep your app from crashing in a transient spike, but it’s limited, so it doesn’t inadvertently create noisy neighbor problems. All is in balance, all is right with the world.

You see where this is going.

It’s easy enough to check which devices we’re rate limiting:

```
$ find /sys/fs/cgroup/ -name blkio.throttle.write_bps_device | xargs egrep '253'
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:179 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:101 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:233 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:250 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:86 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:114 16777216
... 
```

Most of those `major,minor` pairs were associated with swap devices. “Most” is not the answer you want here. Now we know what’s happening: some small number of Fly Machines are getting throttled by an incorrect swap rate limit.

The remaining question is “why”. And a quick audit of the `flyd` code that sets up the rate limits yields an answer:

```
writeDev.Major = int64(rdev / 256)
writeDev.Minor = int64(rdev % 256) 
```

To work with a device given its path on the filesystem, you call `stat(2)` on it. `stat` gives us the device `major,minor` numbers packed into a `dev_t`, which you get to unpack. Easy enough: the major is the top 8 bits, the minor the bottom, of a 16 bit value…

… if it’s 2003.

Check [linux/kdev_t.h](https://github.com/torvalds/linux/blob/v5.18/include/linux/kdev_t.h#L7):

```
#define MINORBITS	20 
```

Our orchestrator was potentially setting the `blkio` limits on the wrong device, limiting it to 16MiB/s, and sending any IO-intensive applications straight into their worst nightmare.

The probability of hitting this was not huge. Which explained why the elevated `iowait` times seemed so sporadic. But once in a million events happen constantly at [Fly.io](http://Fly.io) scale.

You can see now why re-creating a Fly Machine worked around this issue: it generated a new swap device, and a new root filesystem device, and those were unlikely to lose the block device number lottery.

### What we did

Clearing the bad rate limits was easy enough.

So was fixing the `flyd` code that handled `st_rdev`.

Meanwhile, we’re investigating better telemetry for container process states. It’s not certain that a machine stuck for an extended period of time in an uninterruptible state is something that warrants us taking action or if it’s just a false positive event, but it might give us some signal to help detect potential issues.

We’d also like to flat out make Firecracker’s IO layer faster. I don’t think that would have helped with this bug, but knowing with greater certainty what the expected IO performance for a machine should be may let us determine if the observed performance is affected by changes to our platform. We’ll write more about this soon.

### In short

You’re hosting your app on [Fly.io](http://Fly.io) and may have seen abnormally slow IO for the past few days. A change we made was the reason that happened and we are bummed if your app was affected. Bare metal performance is a feature of our platform and we’re pretty passionate about it.

As always, if you’re running into issues or see something funky, you should get in touch. Despite millions of metrics being spun through the heuristics that drive our alerts, you have an app you care about and we want to know if you’re hitting any problems. Our users helped us identify this issue and we’re grateful for your engagement.