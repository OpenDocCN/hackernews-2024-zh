<!--yml
category: 未分类
date: 2024-05-27 13:12:46
-->

# A Feat of Engineering - by Thorsten Ball - Register Spill

> 来源：[https://registerspill.thorstenball.com/p/a-feat-of-engineering](https://registerspill.thorstenball.com/p/a-feat-of-engineering)

Whenever I talk with programmer friends about Apple I try to sneak the following story in. Usually I start with “did you know that Apple…?” and end by leaning back in the chair, my index finger pointing at the table, and me saying “… now that’s engineering.”

In 2017 Apple [released iOS 10.3](https://www.theverge.com/2017/3/27/15076244/apple-file-system-apfs-ios-10-3-features). A minor release that didn’t include any major features but something else:

> Apple is actually undertaking a pretty huge shift for all iPad and iPhone users today. Within iOS 10.3, Apple is moving supported devices to its new Apple File System (APFS).

The Verge also noted:

> Most iPhone and iPad users won’t notice a difference after today’s iOS 10.3 update

Let that sink in.

They shipped an update that migrates the file system of “all iPad and iPhone users” — already hundreds of millions in 2017 — and “user’s won’t notice a difference.”

Shipping a file system migration that users won’t even notice, over night, silently, to hundreds of millions of devices.

Automatic file system migration of hundreds of millions of devices.

I don’t know, maybe I get nervous just thinking about having to program a migration like that because my personal success rate with file system changes isn’t the greatest. Multiple times in the past I’ve accidentally destroyed Linux and Windows installations by messing something up on the CLI. But personal clumsiness aside: whenever you touch the file system you have to acknowledge there’s a very real chance you brick your machine. It doesn’t get a lot more load-bearing than the file system.

Obviously there are differences between 15-year-old me messing up an `mkfs` command and a team at Apple writing a file system migrator: for one, they know what a file system actually is, but they also wrote the new file system, have experience migrating from the old to the new one, probably test their migrator for multiple months on hundreds or thousands of testing devices, have excellent tooling for such migrations, and can lean on the experience that Apple as a company has with automatically updating iOS devices.

In other words: they’re pretty good at what they’re doing and chances that their file system migration would end up bricking devices were probably very, very slim. Near zero, possibly zero.

But, still. Take near-zero and multiply it with hundreds of millions and I ask you: would you click the button that rolls out that update? How much testing would you want to do before you press it? How many test devices? How many different configurations? I don’t know how to answer these questions with anything except “a lot”, but I do know this: more than sweat would leave my body before clicking that button.

They did click that button and the update rolled out and it migrated hundreds of millions of devices, over night, and as far as I know no device was bricked and only a few even noticed that something happened.

Now that’s engineering.