<!--yml
category: 未分类
date: 2024-05-27 15:00:26
-->

# ECC vs non-ECC RAM and ZFS | TrueNAS Community

> 来源：[https://www.truenas.com/community/threads/ecc-vs-non-ecc-ram-and-zfs.15449/](https://www.truenas.com/community/threads/ecc-vs-non-ecc-ram-and-zfs.15449/)

I've seen many people unfortunately lose their zpools over this topic, so I'm going to try to provide as much detail as possible. If you don't want to read to the end then just go with ECC RAM. For those of you that want to understand just how destructive non-ECC RAM can be, then I'd encourage you to keep reading. Remember, ZFS itself functions entirely inside of system RAM. Normally your hardware RAID controller would do the same function as the ZFS code. And every hardware RAID controller you've ever used that has a cache has ECC cache. The simple reason: they know how important it is to not have a few bits that get stuck from trashing your entire array. The hardware RAID controller(just like ZFS) absolutely NEEDS to trust that the data in RAM is correct.

For those that don't want to read, just understand that ECC is one of the legs on your kitchen table, and you've removed that leg because you wanted to reuse old hardware that uses non-ECC RAM. Just buy ECC RAM and trust ZFS. Bad RAM is like your computer having dementia. And just like those old folks homes, you can't go ask them what they forgot. They don't remember, and neither will your computer.

Here's some assumptions I've made and some explanation for them:

1\. We're going to deal with a system that has a single bit error. A memory location is stuck at "1". Naturally more than a single bit error is going to cause more widespread corruption. For my examples I'm going to provide a hypothetical 8 bits of RAM. Bit number 2 will be stuck at "1". I will properly mark it with an

**underline and bold**

for bits that end up corrupted. Later I will move this location, but I will discuss that with you at that time.

2\. Most servers have very little RAM used by the system processes and large amounts of RAM for the cache, so we're going to assume that the system runs stable without problems as the errors are most likely in the cache. Even if the system happens to run into the bad RAM location resulting in a crash, you are more than likely going to reset it and keep going until you realize after multiple crashes in a short period of time that something else is wrong. At that point, you're going to already be in "oh crap" territory.

3\. I'm going to ignore any potential corruption of the file system for my examples. Clearly corrupting the file system itself is VERY serious and can be fatal for your data. I will cover this topic at the very end of this discussion.

4\. No additional corruption from any other subsystem or data path will occur. Obviously any additional corruption won't make things any easier.

5\. What I am about to explain is a limitation of ZFS. This is

not

FreeNAS specific. It is ZFS specific.

Now on to some examples:

**What happens when non-ECC RAM goes bad in a system for file systems that don't do their own parity and checksumming?**

So pretend a file is loaded into RAM and then saved to an NTFS/ext4/UFS/whatever-file-system-you-want-that-doesn't-do-its-own-parity/checksumming.

The file is 8bits long. 00110011.

Since our file got stored in RAM, its been corrupted. Now its 0

**1**

110011\. Then that is forwarded to the disk subsystems and subsequently stored on the disk trashed. If this were file system data you might have bigger problems too. So now, despite what your destination is, a hard disk, an SSD, or a redundant array, the file will be saved wrong. No big deal except for that file. It might make the file unable to be opened in your favorite office suite or it might cause a momentary corruption of the image on your streaming video.

**What happens when non-ECC RAM goes bad in a ZFS system?**

So pretend that our same 8bit file is stored in RAM wrong.

Same file as above and same length: 00110011.

The file is loaded into RAM, and since its going to be stored on your ultra-safe zpool it goes to ZFS to be paritied and checksummed. So your RAIDZ2 zpool gets the file. But, here's the tricky part. The file is corrupted now thanks to your bad RAM location.

ZFS gets 0

**1**

110011 and is told to safely store that in the pool. So it calculates parity data and checksums for the corrupted data. Then, this data is saved to the disk. Ok, not much worse than above since the file is trashed. But your parity and checksum isn't going to help you since those were made

*after*

the data was corrupted.

But, now things get messy. What happens when you read that data back? Let's pretend that the bad RAM location is moved relative to the file being loaded into RAM. Its off by 3 bits and move it to position 5\. So now I read back the data:

Read from the disk: 0

**1**

110011.

But what gets stored in RAM? 0

**1**

11

**1**

011.

Ok, no problem. ZFS will check against its parity. Oops, the parity failed since we have a

*new*

corrupted bit. Remember, the checksum data was calculated after the corruption from the first memory error occurred. So now the parity data is used to "repair" the bad data. So the data is "fixed" in RAM. Now it's supposed to be corrected to 01110011, right? But, since we have that bad 5th position, its

*still*

bad! Its corrected for potentially one clock cycle, but thanks to our bad RAM location its corrupted immediately again. So we really didn't "fix" anything. 0

**1**

11

**1**

011 is still in RAM. Now, since ZFS has detected corrupted data from a disk, its going to write the fix to the drive. Except its actually corrupting it even more because the repair didn't repair anything. So as you can see, things will only get worse as time goes on.

Now let's think about your backups.

If you use rsync, then rsync is going to backup the file in its corrupted form. But what if the file was correct and later corrupted? Well, thanks to rsync the backup itself is actually corrupted.

What about ZFS replication? Surely that's better, right? Well sort of. Thanks to those regular snapshots your server will happily replicate the corruption to your backup server. And lets not forget the added risk of corruption during replication because when the ZFS checksums are being calculated to be piped over SSH those might be corrupted too!

But we're really smart. We also do religious zpool scrubs. Well, guess what happens when you scrub the pool. As that stuck memory location is continually read and written to, zfs will attempt to "fix" corrupted data that it thinks is from your hard disk and write that data back. But instead it is actually reading

*good*

data from your drive, corrupting it in RAM, fixing it in RAM(which doesn't fix it as I've shown above), and then writing the "fixed" data to your disk. This means the data in entire pool is being trashed while trying to do a scrub.

So in conclusion:

1\. All that stuff about ZFS self-healing goes down the drain if the system isn't using ECC RAM.

2\. Backups will quite possibly be trashed because of bad RAM. Based on forum users over the last 18 months, you've got almost no chance your backups will be safe by the time you realize your RAM is bad.

3\. Scrubs are the best thing you can do for ZFS, but they can also be your worst enemy if you use bad RAM.

4\. The parity data, checksums, and actual data need to all match. If not, then repairs start taking place. And what are you to do a disk needs replaced and parity data and actual data don't match because of corruption? The data is lost.

To protect your data from loss with ZFS, here's what you need to know:

1\. Use ECC RAM. It's a fundamental truth.

2\. ZFS uses parity, checksums, mirrors, and the copies parameter to protect your data in various ways. Checksums prove that the data on the disk isn't corrupted, parity/mirrors/copies corrects those errors. As long as you have enough parity/mirrors/copies to fix any error that ever occurs, your data is 100% safe(again, assuming you are using ECC RAM). So running a RAIDZ1 is

**very**

dangerous because when one disk fails you have no more protection. During the long(and strenuous) task of resilvering your pool you run a very high risk of encountering errors on the remaining disks. So any error is detected, but not corrected. Let's hope that the error isn't in your file system where corruption could be fatal for your entire pool. In fact, about 90% of users that lose their data had a RAIDZ1 and suffered 2 disk failures.

3\. If you run out of parity/mirrors and your pool is unmountable, you are in

deep

trouble. There are no recovery tools for ZFS, and quotes from data recovery specialists

start

in the 5 digit range. All those tools people normally use for recovery of desktop file systems don't work with ZFS. ZFS is nothing like any of those file systems, and recovery tools typically only find billions of 4k blocks of data that looks like fragments to a file. Clearly it would be cheaper(and more reliable) to just make a backup, even if you have to build a second FreeNAS server. Let's not forget that if ZFS is corrupted

*just*

badly enough to be unmountable because of bad RAM, even if your files are mostly safe, you'll have to consider that 5 digit price tag too.

4\. Usually, when RAM goes bad you will normally lose more than a single memory location. The corruption is usually a breakdown of the insulation between locations, so adjacent locations start getting trashed too. This only creates more multi-bit errors.

5\. ZFS is designed to repair corruption and isn't designed to handle corruption that you can't correct. That's why there's no fsck/chkdsk for ZFS. So once you're at the point that ZFS' file structure is corrupted and you can't repair it because you have no redundancy, you are probably going to lose the pool(and the system will probably kernel panic).

So now that you're convinced that ECC really is

*that*

important, you can build a system with ECC for relatively cheap...

Motherboard:

[Supermicro X9SCM-F ($160ish)](http://www.amazon.com/dp/SUPERMICRO/?tag=ozlp-20)

CPU:

[Pentium G2020 ($60ish)](http://www.amazon.com/dp/B00D5ZZZ70/?tag=ozlp-20)

RAM:

[KVR16E11K4/32 32GB of DDR3-ECC-1600Mhz RAM($350ish)](http://www.amazon.com/dp/B008LMO1DG/?tag=ozlp-20)

So total cost is about

**$570**

. Less if you don't want to go to a full 32GB of RAM. If you went with a 2x8GB RAM stick kit you can get the total price down about

**$370**

. The G2020 is a great CPU for FreeNAS.

Of course, if you plan to use plugins like Plex that can be CPU intensive for transcoding you will need a little more power. Be wary of what CPUs do and don't support ECC. All Xeons do, and some i3s do. Check with Intels specification sheets to be sure

before

you spend the money. I use an

[E3-1230v2](http://www.amazon.com/dp/B0085MQUTU/?tag=ozlp-20)

(about $250) and it is AMAZING! No matter what I throw at it I can't get more than about 30% CPU usage. Don't go by the TDP to try to pick the "greenest" CPU either. TDP is for full load heat output. That provides no information on what kind of power usage you will see when idle(which is what your system will probably be doing 99% of the time). My system with no disks installed and the system idle is at about 35w. Unfortunately I can't help with AMD CPUs since I'm not a fan of AMD. I do know that trying to go with "Green" CPUs for AMDs has disappointed many people here. "Green" CPUs perform slower than other CPUs, so be careful and don't buy a CPU that can't perform fast enough to make you happy.

The motherboard I listed above is ideal too. It has dual Intel Gb LAN, IPMI(never need a keyboard, mouse and monitor ever again!), and PCIe slots for that M1015 SAS controller you might want someday. I can't tell you how awesome IPMI is. But its basically remote desktop for your system. Except you can even use it during bootup. For example, you can go into your BIOS and change settings remotely! And you can mount CDs without a CD-ROM on the computer, remotely! How cool is that!? Add in the ability to remotely power on and off the server with IPMI and you have something for that server you want to shove in the corner and forget about.

ECC RAM can only detect

and

correct single bit errors. When you have multi-bit errors(if my experience is any indication) the system can detect(but not correct) those errors and immediately halts the system with a warning message on the screen with the bad memory location. Naturally, halting the system is bad as the system becomes unavailable. But its better to halt the system and let you remove the bad DIMM than to let your zpool get corrupted because of bad RAM. Remember, the whole goal is to protect your zpool and a system halt is the best bet once you've realized things are going badly in RAM.

Here's another way to look at it:

Example 1: Running a server with its native file system(probably ext3, NTFS, or HFS+), non-ECC RAM. The only way you can expect to lose your entire drive's worth of data is if your drive actually fails completely. If your RAM goes bad you'll potentially lose a few files to corruption that have been recently opened. You may have to run a chkdsk/fsck on the partition to get it back in good shape. But you'll be able to take that disk and put it in another machine and get most(if not all) of your data back. Even a worst case scenario there's plenty of tools like

[Ontrack EasyRecovery DIY](http://www.krollontrack.com/data-recovery/recovery-software/)

Software that will work for NTFS and you can expect to have a reasonable chance of getting most(if not all) of your data back. You can also call those data recovery professionals and for 4-figures they might get your data back from a failed disk.

Example 2: Server with ZFS and non-ECC RAM. Now you have more ways to lose your data:

1\. If the drive completely dies(obviously).

2\. Based on prior users that have had non-ECC RAM fail you'll reboot to find your pool unmountable. This means your data is gone. There is no data recovery tools out there for ZFS. NONE. It's ALL gone for good.

3\. What about the fact that you used those non-server grade parts? Guess what, they can also trash your pool and make it unmountable. The outcome is exactly the same as #2\. You just lost

all

of your data because the pool won't mount.

The problem is that your native file systems have tools like chkdsk and fsck to fix file system errors. You also have plenty of options for software recovery with utilities like

[Ontrack EasyRecovery DIY](http://www.krollontrack.com/data-recovery/recovery-software/)

Software.

But, no matter how much searching you do, there is no ZFS recovery tools out there. You are welcome to call companies like Ontrack for data recovery. I know one person that did, and they spent $3k just to find out

*if*

their data was recoverable. Then they spent another $15k to get just 200GB of data back.

So tell me which example you'd rather fall into? #1 where you have fewer opportunities to lose everything and have recovery options. Or #2 where you have quite alot more opportunities to lose everything and have zero recovery options to boot? I'd rather be in scenario 1 than scenario 2\. I can't imagine you'd want scenario 2 either.

So when you read about how using ZFS is an "all or none" I'm not just making this up. I'm really serious as it really does work that way. ZFS either works great or doesn't work at all. That

really

truthfully how it works

Don't like it, be one of the few that "stick it to the man" with non-recommended components. It's your win(or loss). But you will get absolutely no sympathy when you show up and your pool doesn't mount like many people that think they can build a cheap system and get away with it.

**PLEASE TAKE THIS AS A WARNING TO NOT USE NON-ECC RAM WITH ZFS.** [Cool presentation from Intel about RAM errors and non-ECC vs ECC.](http://cache-www.intel.com/cd/00/00/46/78/467819_467819.pdf)

Someone else that confirms this:

[https://pthree.org/2013/12/10/zfs-administration-appendix-c-why-you-should-use-ecc-ram/](https://pthree.org/2013/12/10/zfs-administration-appendix-c-why-you-should-use-ecc-ram/)

List of threads where people unfortunately experienced data loss with bad non-ECC RAM. (Updated when I can remember to):

List of people that have posted info that asserts the fact that ECC saved them: