<!--yml
category: 未分类
date: 2024-05-27 14:50:54
-->

# How the DevTeam conquered the iPhone

> 来源：[https://fabiensanglard.net/iSummer/](https://fabiensanglard.net/iSummer/)

January 31, 2024

How the DevTeam conquered the iPhone

* * *

The summer of 2007 was an eventful one if you were into cell phones. It all started on June 29 when Apple announced the release of the iPhone. If you have never taken the time to watch the full keynote, I highly recommend it.

 <MacWorld%20keynot%202007.webm>

Your browser does not support the video tag.

Reactions to the announcement were mixed. While some dismissed the device, others were eager to try it.

I belonged to the second category. However I lived in Canada where there was no release date in sight. Indeed, the iPhone was never released in the Great White North?[^([1])](#footnote_1). Apple only came to an agreement with Rogers a year later with the 3G on July 11, 2008[^([2])](#footnote_2).

iPhone Dev Team

* * *

For forgotten countries there was hope. A group had assembled with the goal of enabling the device to run with any carrier, using software only[^([3])](#footnote_3). They called themselves the "iPhone Dev Team".

Committed to openness via a blog on **iphone.fiveforty.net**. They reported their progress on a regular basis. On rich days, updates happened hourly (July 3d came with eight updates from 12AM to 9PM[^([4])](#footnote_4)).

Needless to say that during that summer 2007, my web browser was riveted to their website. However I did not have the skills to understand fully what they were doing. Participants were highly skilled, a lot of lingo was being thrown around, and I never dared asking questions.

It always bugged me to not grasp the technological details of such an historic moment. Today I decided to go back in time thanks to the Wayback Machine and get it done. Maybe you will enjoy tagging along with me.

iphone.fiveforty.net status bar

* * *

To help follow progress, **iphone.fiveforty.net** had a status bar on its homepage. It featured all the milestones the DevTeam had identified to reach their goal. There were three levels of accomplishment going from red to green.

I don't remember ever seeing it fully red and I am unsure it ever was. The earliest date crawled by the Wayback machine is [Jul 6th, 2007](https://web.archive.org/web/20070706064445/http://iphone.fiveforty.net/wiki/index.php?title=Main_Page) when two out of six milestones had been reached.

| Break DMG Password | Bypass Activation | Get Write Access | Get Working Toolchain | Unlock Phone | Enable Third-party Applications |

The journey was completed on September 12th, however the Wayback machine "only" crawled thirteen days later on [September 25th 2007](https://web.archive.org/web/20070925035357/http://iphone.fiveforty.net:80/wiki/index.php?title=Main_Page).

| Decrypt Firmware | Bypass Activation | Get Write Access | Get Working Toolchain | Enable Third-party Applications | Unlock Phone |

We are going to study each stage in order.

The intended way

* * *

Before diving into each milestone, let's remind ourselves how the iPhone was supposed to be used.

1.  A device could be purchased from an Apple Store at the cost of $499 (with 4GB storage) and $599 (for the 8 GB version)[^([5])](#footnote_5).
2.  Unboxing unveiled an [unusable](activate.jpeg) iPhone showing a [Connect to iTunes](itunes.jpg) screen.
3.  The customer had to open iTunes and subscribe ([video](https://www.youtube.com/watch?v=SAhRFE4D6W0), [article](https://archive.is/UjS8d)) to an AT&T [membership](activationtips.pdf).
4.  After that the phone became activated. It remained tied down (locked) to AT&T but the customer was good to go.

Milestones

* * *

The six milestones defined early one by the DevTeam followed a logical path to turn an inert piece of electronics back into a smartphone.

*   Get read access to understand the system -> **Break DMG Password**.
*   Get the device out of its lethargy -> **Bypass Activation**.
*   Get write access to modify the system -> **Get Write Access**.
*   Modify the system with custom executables -> **Get a Working toolchain**.
*   Do something to allow the baseband to connect to any carrier -> **Unlock**.
*   Create an app to automate the whole process -> **Enable Third-Party applications**

Decrypt Milestone

* * *

To restore a device to its factory state, iTunes downloaded an **IP**hone **S**oft**W**are archive (extension `.ipsw`). The goal of this step was to understand every file in it. Conveniently, the archive is still available on ipsw.me[^([6])](#footnote_6) and it uses the zip format so we can look at it.

```
% unzip -l iPhone1,1_1.0_1A543a_Restore.ipsw
Archive:  iPhone1,1_1.0_1A543a_Restore.ipsw
  Length      Date    Time    Name
---------  ---------- -----   ----
 15725706  06-26-2007 18:41   694-5259-38.dmg
 86257664  06-04-2007 19:32   694-5262-39.dmg
        0  05-22-2007 22:25   Firmware/all_flash/
    14474  05-22-2007 22:21   Firmware/all_flash/applelogo.img2
    73866  05-22-2007 22:21   Firmware/all_flash/batterycharging.img2
    65674  05-22-2007 22:21   Firmware/all_flash/batterylow0.img2
    73866  05-22-2007 22:21   Firmware/all_flash/batterylow1.img2
    43146  05-22-2007 22:21   Firmware/all_flash/DeviceTree.m68ap.img2
   145546  05-22-2007 22:21   Firmware/all_flash/iBoot.m68ap.RELEASE.img2
    51338  05-22-2007 22:21   Firmware/all_flash/LLB.m68ap.RELEASE.img2
      175  05-22-2007 22:25   Firmware/all_flash/manifest
    34954  05-22-2007 22:21   Firmware/all_flash/needservice.img2
    26762  05-22-2007 22:21   Firmware/all_flash/recoverymode.img2
   103562  05-22-2007 22:25   Firmware/dfu/iBSS.m68ap.RELEASE.dfu
     9354  05-22-2007 22:25   Firmware/dfu/WTF.s5l8900xall.RELEASE.dfu
  3073247  06-19-2007 13:51   kernelcache.restore.release.s5l8900xrb
     1594  06-26-2007 18:41   Restore.plist
---------                     -------
105700928                     20 files // All iOS is a mere 105 MiB!!

```

There are many files in there whose purpose were identified[^([7])](#footnote_7)[^([8])](#footnote_8) fairly early on[^([9])](#footnote_9). Among then we find `img2` images displayed on the screen during recovery, the `Firmware` folder which contains everything for the baseband, and iOS kernel (`kernelcache`).

More importantly, we see two huge `dmg` archives. The first one, `694-5259-38.dmg`, (called ramdisk[^([10])](#footnote_10)) was only used when iTunes restored the phone. Surprisingly, it was not encrypted. A simple `dd` allowed to mount it.

```
$ dd if=694-5259-38.dmg of=ramdisk.dmg bs=512 skip=4 conv=sync
$ hdiutil attach ramdisk.dmg
/dev/disk4                                            /Volumes/ramdisk
$ find /Volumes/ramdisk > ramdisk.txt

```

This is not the iOS filesystem (barely [100 entries](ramdisk.txt)) but it allowed the DevTeam to poke into `/private/etc/master.passwd` and find the password for the user `mobile` (running the apps), and user `root` (running all other processes).

More importantly, the content of the ramdisk enabled access to the second dmg file, `694-5262-39.dmg`, which contains the iOS filesystem used during normal operation. The archive was encrypted but a key for it was found by looking into `/usr/sbin/asr` from the ramdisk.

Note that they found a key, not the passphrase. They could not use `hdiutil` and had to write their own decrypter `vfdecrypt.c`[^([11])](#footnote_11). Once decrypted, this img could be mounted and they gained read access to the full runtime filesystem (list of files [here](https://web.archive.org/web/20071005010951/http://iphone.fiveforty.net/wiki/index.php/SystemFileAndDirectoryList#System.2FLibrary.2FExtensions.2F_Source_Files)).

| Break DMG Password | Bypass Activation | Get Write Access | Get Working Toolchain | Unlock Phone | Enable Third-party Applications |

Activation Milestone

* * *

An iPhone out of the box was a brick. It was not activated. After a user subscribed to AT&T, the following happened[^([12])](#footnote_12).

1.  iTunes would collect the devices's DeviceID, IMEI, and ICCID.
2.  These three fields would be concatenated into a token.
3.  The token would be sent to Apple server (`albert.apple.com`) where it would be signed with Apple's private key.
4.  The signed token would then be sent back to the device.
5.  A daemon [`lockdownd`](https://iphonedev.wiki/Lockdownd), listening over USB verified the token using Apple's public key.
6.  With the proof that the token came from Apple, and matching DeviceID, IMEI, and ICCID, `lockdownd` updated the device state to "Activated".
7.  The user then had access to the iPhone [homescreen](Apple-iPhone.webp) and the apps.

```
┌─────────┐
│ iPhone  │
├─────────┤              ┌──────┐      ┌────────────────┐
│lockdownd│              │iTunes│      │albert.apple.com│
└────┬────┘              └──┬───┘      └───────┬────────┘
     │                      │                  │
     │   1.dID,IMEI,ICCID   │                  │
     ├─────────────────────►│                  │
     │                      │   2.Send token   │
     │                      ├─────────────────►│
     │                      │                  │
     │                      │   3\. f(token)    │
     │                      │◄─────────────────┤
     │                      │                  │
     ├──────────────────────┤                  │
     │ 4.f(iD, IMEI, ICCID) │                  │
     │         ==           │                  │
     │      f(token) ?      │                  │
     │                      │                  │
     ▼                      ▼                  ▼
 ACTIVATED!

```

The first breakthrough toward activation came from notorious developer dvdjon when he released PhoneActivationServer[^([13])](#footnote_13).

> dvdjon created an activation method by patching iTunes to use HTTP instead of HTTPS for the activation server, and redirected activation requests to the PhoneActivationServer.
> 
> The PhoneActivationServer would then send a valid Account Token to the iPhone. However, the Account Token was for a different IMEI, ICCID, and DeviceID. This method left the phone in the MismatchedICCID state, but allowed access to the user interface.
> 
> - iPhone Elite Dev Team
> 
> [^([14])](#footnote_14)

The key part of the explanation is "valid Token". How do you generate one without the private key? It turned out you didn't need to. PhoneActivationServer always returned the same signed token captured from a successful activation (dvdjon's own phone?), regardless of the input token. That was a simple replay trick.

This was further oulined in George Hotz's presentation "Hacking the iPhone".

> resending activation record to another phone
>     lockdownd didn't check that (iD, IMEI, ICCID) in response actually matched anything

The DevTeam wrote a cli named `tools`[^([16])](#footnote_16) which loaded a hard-coded signed token from a plist file, and sent it to any iPhone by tapping into iTunesMobileDevice.dll function (later improved with a standalone `iPhoneInterface` not requiring iTunes). The tool came with [source code](https://att.newsmth.net/nForum/#!article/Apple/178321) if you want to take a look yourself.

| Break DMG Password | Bypass Activation | Get Write Access | Get Working Toolchain | Unlock Phone | Enable Third-party Applications |

Write access Milestone

* * *

An activated phone would appear in iTunes GUI and an user could upload files such as musics and photos. So there was "some" write access. But the process taking care of the file upload (`acfd`) was `chroot` jailed to `/root/Media`[^([17])](#footnote_17). Moreover only the user partition was mounted "rw". The system partition was mounted "r" only. The double goal was hence to break out of `chroot` jail (that's where the term "jaibreaking" come from btw) AND somehow be able to write in the system partition.

```

                ┌──────┐
                │iPhone│
┌──────┐        ├──────┤
│iTunes│        │ acfd │
└──┬───┘        └──┬───┘
   │               │
   │  write(file)  │
   ├──────────────►│
   │               │
   │               │
   │             chroot
   │               │
   │               ▼
   ▼               /root/Media

```

The team seems to have found a solution around July 8th 2007[^([18])](#footnote_18) and described a process involving the ramdisk[^([19])](#footnote_19).

To understand how it works, we need to know the two ways an iPhone boots. The first instructions to run when the phone boots come from the BootROM. From there a chain of loader bootstrap more and more complex stages. Note that before launching a new stage, its signature is checked so only Apple signed stuff can run. This process establishes a chain of trust all the way to Darwin kernel running the apps.

```
 Normal operation boot chain:

┌───────┐     ┌───┐      ┌─────┐      ┌──────┐      ┌───────────┐
│BootROM├────►│LLB├─────►│iBoot├─────►│Kernel├─────►│Normal Mode│
└───────┘     └───┘      └─────┘      └──────┘      └───────────┘

```

There is a second boot mode allowing iTunes to restore a phone from a bad state to a good state. When the phone boots in Recovery Mode, it stops at the iBoot stage[^([20])](#footnote_20). From there the phone expects the next stage to be loaded from ram (so now we understand the dmg archive named "ramdisk", it is a disk meant to be uploaded to ram).

```
 Recovery mode boot chain:
                                    ┌──────────────────────────────────┐
                                    │              iTunes              │
                                    └───┬───────────┬───────────┬──────┘
                                        │           │           │ push files to
                                        ▼           ▼           ▼ restore iOS
┌───────┐     ┌───┐      ┌─────┐    ┌───────┐   ┌──────┐    ┌───────────┐
│BootROM├────►│LLB├─────►│iBoot├───x│ramdisk├──►│Kernel├───►│RestoreMode│
└───────┘     └───┘      └─────┘    └───────┘   └──────┘    └───────────┘

```

The DevTeam looked inside `iTunesMobile.dll` to find out how iTunes was writing to the filesystem to perform a restore. They found commands such as `mount`, `umount` and `ditto` (to copy files) and wrote a CLI tool called [iPHUC](https://github.com/svn2github/iphuc), able to talk to the device in Restore Mode via `iTunesMobile.dll` private methods[^([21])](#footnote_21).

The source code of `iPHUC` was later published and we can look at how it works.

1.  User puts the phone in Recovery mode.
2.  [Send ramdisk to the phone (grestore command)](https://github.com/svn2github/iphuc/blob/master/RecoveryInterface.cpp#L77C8-L77C24)[^([22])](#footnote_22)[^([23])](#footnote_23).
3.  [Load ramdisk in the phone RAM.](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L87)
4.  [Send the kernelcache.](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L100C5-L100C25)
5.  [Boot kernel pointing at ramdisk.](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L115)
6.  Now phone is in Restore mode.

iBoot in Recovery Mode had an interesting property. It does not check the signature of the kernel before loading it. This is irrelevant to the topic but would later allow the DevTeam to load a patched kernel permitting `execl` of unsigned executables.

Now we have all the knowledge to understand the jailbreak steps detailed [here](https://web.archive.org/web/20071005150518/http://iphone.fiveforty.net/wiki/index.php/How_to_Escape_Jail).

1.  Put phone in Restore Mode
2.  Use restore mode `mount` to mount both system and user partition.
3.  Use `ditto` to copy both `/etc/fstab` and `/System/Library/Lockdown/Services.plist` to `/root/Media`.
4.  Use iTunes to copy these files to the workstation.
5.  Use workstation text editor to modify these files as follows.
    1.  [Make fstab mount](https://www.theiphonewiki.com/wiki//private/etc/fstab) the system partition in "rw" mode instead of "r".
    2.  [Create a second afcd service](https://web.archive.org/web/20071005150518/http://iphone.fiveforty.net/wiki/index.php/How_to_Escape_Jail#:~:text=string%3E%2Dd%3C/string%3E-,%3Cstring%3E/%3C/string%3E,-%3C/array%3E%0A%09%3C/dict%3E) in `Services.plist` based not in `/root/Media` but `/`.
6.  Push back these files to `/root/Media` with iTunes.
7.  `ditto` modified files to their original location in system partition.
8.  Reboot

Upon reboot, iTunes sees the whole filesystem thanks to `acfd2`. Since both system and user partitions are mounted "rw", the DevTeam had achieved full read/write access to the system.

```

                ┌──────┐
                │iPhone│
┌──────┐        ├──────┤
│iTunes│        │ acfd2│
└──┬───┘        └──┬───┘
   │               │
   │  write(file)  │
   ├──────────────►│
   │               │
   │               │
   │               ▼
   ▼               /

```

| Decrypt Firmware | Bypass Activation | Get Write Access | Get Working Toolchain | Unlock Phone | Enable Third-party Applications |

Later on both Activation and Write Access were automated into a Desktop MacOS X application named "INdependence".

Toolchain / Enable Third-Party Milestones

* * *

Not much is known about how this was accomplished, except that at least twelve people worked on it[^([24])](#footnote_24). By July 19, 2007 a `binutils` toolchain able to target ARM [was completed](https://web.archive.org/web/20070904020127/http://iphone.fiveforty.net:80/wiki/index.php/Past_Progress_Reports#:~:text=After%20many%2C%20many%20hours,%2D%20the%20dev%20team). This gave the DevTeam the ability to run programs they authored on the device.

> After many, many hours of intense work from "Nightwatch", the first independent "Hello World"* application has been compiled and launched on the iPhone. This was made possible using the "ARM/Mach-O Toolchain", Nightwatch's "special project", that he has been working on so carefully over the past few weeks. Certain parts of the toolchain (such as the assembler) are being refined and tested and these will be released as soon as possible.
> 
> It should be noted that Nightwatch has been instrumental in creating these tools, working in near isolation to get them finished. Nightwatch was also responsible for the "jail exploit" that he developed from information he and other members of the the dev team discovered.
> 
> Please join us to thank Nightwatch, Tmiw, Darkten and Daeken for making this happen.
> 
> - iPhoneDevTeam Wiki/div>
> 
> > mach-o and ARM: never done before outside apple; we needed to write it ourselves (aka watch in awe as nightwatch did it)
> > 
> > - Geohotz<
> > 
> > [^([25])](#footnote_25)
> > 
> > /div>
> > 
> > Another goal of the toolchain was to rebuild a header (MobileTerminal.h) able to expose private functions from `iTunesMobile.so` and communicate with `afc` without the need to have iTunes running.
> > 
> > Several presentations mention the kernel checking the signature of an executable before allowing an `execl`. The first iPhone did not do that, it was likely something introduced in v1.1.1.
> > 
> > Unlock Milestone
> > 
> > * * *
> > 
> > Finally, we reach the last item and the goal of the whole effort. The DevTeam got there around [Aug 14, 2007](https://web.archive.org/web/20070814011425/http://iphone.fiveforty.net/wiki/index.php/Main_Page). Note that the color is orange, not red. It seem they knew an unlock was imminent.
> > 
> > | Decrypt Firmware | Bypass Activation | Get Write Access | Get Working Toolchain | Enable Third-party Applications | Unlock Phone |
> > 
> > Before we continue, just a few words about the structure of the iPhone. The smartphone is actually made of two parts. The smart, which is iOS and the phone/modem (with its firmware called "baseband"). These are two distinct systems, with their own RAM, own CPU, own storage, own firmware, even their own oscillators. They communicate together over an UART line (mounted on `/dev/tty.baseband`) using AT commands (e.g.: [Send an SMS with AT Commands](https://www.smssolutions.net/tutorials/gsm/sendsmsat/)).
> > 
> > ```
> >         ┌─────────────────────────────────────────────────────────┐
> >         │                        iPhone                           │
> >         │                                                         │
> >         │   ┌────────────┐                     ┌───────────────┐  │
> >         │   │            │         UART        │               │  │
> >         │   │    iOS     │◄───────────────────►│    Baseband   │  │
> >         │   │            │  /dev/tty.baseband  │               │  │
> >         │   └────────────┘                     └───────────────┘  │
> >         └─────────────────────────────────────────────────────────┘
> > 
> > ```
> > 
> > How the unlock process worked was [well known](https://web.archive.org/web/20070904020127/http://iphone.fiveforty.net:80/wiki/index.php/Past_Progress_Reports#:~:text=We%20even%20know,to%20AT%26T.) from almost day one (gory details [here](https://www.theiphonewiki.com/wiki/Unlock#:~:text=At%20%2B0x400%20in%20the%20seczone%2C%20a%20token%20is%20stored%20encrypted%20with%20(NCK%20%2B%20NORID%20%2B%20HWID))).
> > 
> > > We even know the AT command to do the unlock. It's 'AT+CLCK="PN",0,"xxxxxxxx"'.
> > > 
> > > But good luck finding those x's. They are called the NCK, or Network Control Key, and are believed to be unique in everyones phone. Forget brute force (time impractical) and the obvious entries. If you still think bruteforce is a good idea, [read this](https://web.archive.org/web/20070825072344/http://lpahome.com/iPhone/youarestupid.txt).
> > > 
> > > Further, there is a limit of 3-10 unlock attempts per phone, after which the firmware will "hard-lock" itself to AT&T.
> > > 
> > > - iPhoneDevTeam wiki
> > 
> > The baseband also has a BootROM, and a chain of loaders establishing a chain of trust so everything ends up being verified against a signature. The keys are of course only available at runtime, and can't be extracted. Furthermore, working with the baseband was difficult.
> > 
> > > Now the big thing about the baseband, and the most irritating thing, is that there is no DFU/Recovery Mode, and I've always been jealous of planetbeing and wizdaz and pumpkin and all these guys because they always had a failsafe to basically give them a free pass to do everything you think of to the phone.
> > > 
> > > Some of us at several points have completely erased the NOR, and completely invalidated the LLB and things like that. And, what happens if you have an invalid LLB in there, which is sort of that second stage, is your phone basically just rapidly flashes away in black with like these horrible looking sere (?) marks going down the screen, and it's very scary to watch, and you think it's completely gone and we nicknamed it "Christmas Tree Mode".
> > > 
> > > But, as bad as it looked at the time, as long as you were good with your timing of your fingers, you can always enter DFU Mode and recover from that. There is nothing like that in the baseband; there are things you can do the the baseband and to the NOR, or the images in the NOR, that can permanently brick your phone.
> > 
> > By [July 2007](https://gizmodo.com/iphone-reverse-engineering-opens-new-door-to-total-unlo-284614) that DevTeam had reverse engineered the baseband from the ipsw archive. Furthermore they had studied the executable in charge of updating the baseband, `/usr/local/bin/bbupdater` found in the ramdisk. They knew all the commands to upload a new one.
> > 
> > This lead to a first command-line [`iUnlock`](https://web.archive.org/web/20111121110900/http://gizmodo.com/assets/resources/2007/09/iunlock_src.zip) which involved having many files (such as a dumped firmware "nor", and secpack file "ICE03.12.06_G.fls"). Soon after, they released a simpler `anySIM` app which could be uploaded to the phone and ran with a simple button.
> > 
> > Since the [source code](https://code.google.com/archive/p/devteam-anysim/source/default/source) was also published, we can look inside and follow how it works.
> > 
> > 1.  Open `/dev/tty.baseband` and setup modem parameters (e.g.: baud).
> >     
> >     ```
> >      fd = open("/dev/tty.baseband");
> >     ```
> >     
> >     
> > 2.  Dump baseband (a.k.a NOR, size = 4MiB) to /tmp.
> > 3.  Load baseband to RAM.
> > 4.  LoadSecpack (a file from the ramdisk named ICE03.12.06_G.fls)
> > 5.  Patch baseband instruction in RAM (make any NCK allow an unlock).
> >     
> >     ```
> >       int offset = 0x216f28;
> >     
> >       Buffer[offset + 0] = 0x01;
> >       Buffer[offset + 1] = 0x00;
> >       Buffer[offset + 2] = 0xa0;
> >       Buffer[offset + 3] = 0xe3;
> >     ```
> >     
> >     
> > 6.  Push back baseband.
> >     
> >     ```
> >       SendBeginSecpack(fd, secpack); \\ secpack explanation here 
> >       free(secpack);
> >     
> >       SendErase(fd, 0xA0020000, 0xA03bfffe);
> >       Seek(fd, 0xA0020000 - 0x400); // Firmware must be at 0xA0020000\. WTH?
> >       
> >       unsigned char foo[0x400];
> >       memset(foo, 0, 0x400);
> >       SendWrite(fd, foo, 0x400, false);
> >       
> >       SendWrite(fd, fw, fwsize, true);
> >       SendEndSecpack(fd);
> >       ValidateFW(fd, 0xA0020000, fw, 0x800);
> >     
> >     ```
> >     
> >     
> > 7.  Unlock.
> >     
> >     ```
> >       AT+CLCK="PN",0,"00000000"
> >       AT+CLCK="PN",2
> >     
> >     ```
> >     
> >     
> > 
> > In summary, what `anySIM` does is dump the baseband firmware (while the phone is running, pretty cool), patch it [^([27])](#footnote_27) (with raw bytes writing, also pretty cool!), and upload it back. Except this should not work since the firmware signature is checked upon upload and the firmware is patched. The firmware upload should fail the signature check.
> > 
> > It seems the magic trick lies with the [minus 0x400](https://www.theiphonewiki.com/wiki/Minus_0x400) offset. Why it works is less than clear, even with GeoHotz explanation.
> > 
> > > The first 0x400 bytes aren't written until the signature verifies. So start writing 0x400 bytes earlier :-)
> > > 
> > > - Geohotz
> > 
> > After the publication of this article, the kind people of Hackernews chipped in to explain[^([28])](#footnote_28) how `-0x400` works. It goes as follows.
> > 
> > The baseband receives the new firmware in chunks of up to `0x800` bytes. It cannot store the whole 4MiB in RAM, then checksum, and finally write to flash. Instead, bytes are written to flash as they are received, except for the first `0x400` bytes which are buffered in RAM. When the firmware is fully uploaded, the baseband does the checksum. If the test fails, the buffered bytes are discarded without being written to flash (the baseband has a corrupted firmware and will fail to start). If the firmware passes the test, the `0x400` bytes are written at the beginning of the firmware.
> > 
> > The `-0x400` trick works by first writing garbage 0x400 bytes at offset minus `0x400` bytes from where the firmware should be written. Then the 4miB firmware is sent (the offset matches exactly where the firmware should be). When the cheksum test inevitably fails the garbage bytes are discarded. But the entire new firmware was properly flashed!
> > 
> > | Decrypt Firmware | Bypass Activation | Get Write Access | Get Working Toolchain | Enable Third-party Applications | Unlock Phone |
> > 
> > Putting it all together
> > 
> > * * *
> > 
> > The complete list of [instructions to unlock](https://web.archive.org/web/20071011194202/http://iphone.fiveforty.net/wiki/index.php/Software_Unlock) was published on [September 12, 2007](https://gizmodo.com/the-complete-iphone-unlock-star-wars-timeline-304310). Along them were published per-continent testimonies [^([29])](#footnote_29) and most interestingly Canada[^([30])](#footnote_30).
> > 
> > Epilogue
> > 
> > * * *
> > 
> > Apple lost no time in releasing iPhone firmware V1.1.1 on 27th Sept 2007\. The progress bar was reset[^([31])](#footnote_31) and the cat and mouse game started. It has been ongoing ever since.
> > 
> > | Decrypt 1.1.1 | Get Write Access 1.1.1 | Activate 1.1.1 | Unlock 1.1.1 | Enable Third-party Applications 1.1.1 |
> > 
> > Going deeper
> > 
> > * * *
> > 
> > If you are interested in further archeology about the 2007 unlock, here is a list of links to checkout.
> > 
> > One More Thing
> > 
> > * * *
> > 
> > This article was made possible by the awesomeness of the Internet Archive and its WayBack machine. Take a minute to [donate](https://archive.org/donate).
> > 
> > References
> > 
> > * * *
> > 
> > * * *
> > 
> > *