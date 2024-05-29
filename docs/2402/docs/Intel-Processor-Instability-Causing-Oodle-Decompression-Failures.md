<!--yml
category: 未分类
date: 2024-05-29 13:19:40
-->

# Intel Processor Instability Causing Oodle Decompression Failures

> 来源：[https://www.radgametools.com/oodleintel.htm](https://www.radgametools.com/oodleintel.htm)

# Intel Processor Instability Causing Oodle Decompression Failures

RAD has become aware of a problem that can cause Oodle Data decompression failures, or crashes in games built with Unreal. We believe that this is a hardware problem which affects primarily Intel 13900K and 14900K processors, less likely 13700, 14700 and other related processors as well. Only a small fraction of those processors will exhibit this behavior. The problem seems to be caused by a combination of BIOS settings and the high clock rates and power usage of these processors, leading to system instability and unpredictable behavior under heavy load.

**UPDATE April 22, 2024**: Some of the settings recommendations have changed! Please re-check.

As far as we can tell, there is not any software bug in Oodle or Unreal that is causing this. Due to what seem to be overly optimistic BIOS settings, some small percentage of processors go out of their functional range of clock rate and power draw under high load, and execute instructions incorrectly. This is being seen disproportionately in Oodle Data decompression because unlike most gameplay, simulation, audio or rendering code, decompression needs to perform extra integrity checks to handle accidentally or maliciously corrupted data, and is thus likely to spot inconsistencies very soon after they occur. These decode failures then typically result in an error message.

When starting an Unreal Engine-based game, the most common failure is of this type:

> DecompressShader(): Could not decompress shader (GetShaderCompressionFormat=Oodle)

However, this problem does not only affect Oodle, and machines that suffer from this instability will also exhibit failures in standard benchmark and stress test programs. Any programs which heavily use the processor on many threads may cause crashes or unpredictable behavior. There have been crashes seen in RealBench, CineBench, Prime95, Handbrake, Visual Studio, and more. This problem can also show up as a GPU error message, such as spurious "out of video memory" errors, even though it is caused by the CPU.

We do not have acccess to diagnostic processor information that would nail down the exact cause and best workaround for this problem. It seems that many motherboard/BIOS manufacturers are shipping with settings that push the processor outside its safe operating range. Because this problem appears to affect only a small fraction of processors, some users have had success with returning their processor to the manufacturer and getting a new one which doesn't exhibit the problem.

Other workarounds require using tuning utilities or modifying BIOS settings. Note that doing so incorrectly can cause damage to your system. The changes we are recommending here are, to the best of our knowledge, completely safe, but you are solely responsible for any damages or loss caused by changing these settings from their factory defaults. If you are uncomfortable or worried about using tuning utilities (even officially sanctioned ones) or changing your BIOS settings, and frequent crashes also occur in the benchmark programs mentioned previously, you should be able to return the CPU or the entire computer to the manufacturer instead.

A reportedly successful workaround for many people is to use [Intel XTU](https://www.intel.com/content/www/us/en/download/17881/intel-extreme-tuning-utility-intel-xtu.html) and lower the Performance Core multiplier from x55 to x54 or x53\. Apparently, affected titles may then crash one more time during load immediately after, but will work afterwards (we have not been able to confirm this ourselves). Using XTU is likely the quickest and easiest way to go and doesn't even require rebooting to try different settings, but you might need to reapply it after every start if you don't save the profile. (Alternatively, set the P-core multiplier in the BIOS instead.)

In the BIOS, if you have enabled any overclocking, please turn it off; do not use "AI" or "automatic" overclocking. Even if you have not explicitly enabled any overclocking, many BIOSes are doing some by default, so on affected machines you will have to find those settings and turn them off. Every BIOS has slightly different names for the settings; we cannot provide exact instructions of which settings to look for all of them. Some of these settings may be in the Advanced or Overclocking submenu of the BIOS.

1.  First look for settings to put the power limits and voltages of the processor into the Intel recommended safe ranges. You can find the correct limits for your processor at [ark.intel.com](https://ark.intel.com). These might be:

*   "SVID behavior" → "Auto" (NOTE: previous versions of this document recommended the "Intel fail-safe" setting, but even though it's documented as setting Intel-recommended settings, this [does not appear to be the case](https://hothardware.com/news/falcon-nw-potential-fix-raptor-lake-crashes).
*   "Long duration power limit" → reduce to 125W if set higher ("Processor Base Power" on ARK)
*   "Short duration power limit" → reduce to 253W if set higher (for 13900/14900 CPUs, other CPUs have other limits! "Maximum Turbo Power" on ARK)

*   If those don't work, another thing to look for is BIOS "enhanced turbo" or "enhanced multithreading" settings. For example:

*   "ASUS MultiCore Enhancement" → disabled (not Auto)
*   "ASUS Performance Enhancement 3.0" → disabled

*   There have been reports of users finding stability by turning down the maximum clock rate. This can be done with BIOS settings or with Intel XTU. Some possibilities:

*   Turn down the maximum P core multiplier from 55X to 53X or 54X. (for example)
*   Turn down maximum turbo boost clock rate
*   Turn off or turn down "thermal velocity boost"

Note that many motherboard/BIOS settings turn on XMP (Extreme Memory Profile) by default with unstable settings that can cause similar symptoms. Instability due to XMP is a separate issue, but if you have instability problems, you may wish to also disable XMP and see if that helps.

* * *

**Troubleshooting Update from Intel Corporation - Additional troubleshooting steps for ASUS, Gigabyte and MSI motherboards:**

First, install Intel XTU and run the AVX2 test. If the AVX2 test failure is seen, try these options:

1.  For ASUS:

    Ask customer to change BIOS settings: Advanced (F7)- SVID Behavior: Change to "Intel's Fail Safe"

    Reboot the OS and run XTU test again and if the AVX2 test can pass. Run games and see if the issue happens again.
2.  For Gigabyte:

    Solution A): In BIOS, select "ADVANCED MODE", in the Tweaker tab, locate the CPU Vcore and select "Normal" option, select "Dynamic Vcore(DVID)" option, change it from "Auto" to "+0.005V" Increase the DVID by +0.005 and reboot OS, until the game crash disappears and the system is running stable.

    Solution B): In BIOS, select "Tweaker", select "Advanced Voltage Settings", select "CPU/VRAM Settings", adjust "CPU Vcore Loadline Calibration", recommend starting from "Low" to "Medium" until system is stable.

    After implementing solution A or B, run the XTU test again and if the AVX2 test can pass. Run games and see if the issue happens again.
3.  For MSI:

    Solution A): In BIOS, select "OC", select "CPU Core Voltage Mode", select "Offset Mode", select "+(By PWM)", adjust the voltage until the system is stable, recommend not to exceed 0.025V for a single increase.

    Solution B): In BIOS, select "OC", select "DigitALL Power", change "CPU Loadline Calibration Control", recommend starting from "Mode 7" to a lower value until system is stable.

    After implementing solution A or B, run the XTU test again and if the AVX2 test can pass. Run games and see if the issue happens again.

    Solution C): In BIOS, select "OC", select "Advanced CPU Configuration", change "CPU Lite Load Control", set to "Intel Default".

    After implementing solution A, B, or C, run the XTU test again and if the AVX2 test can pass. Run games and see if the issue happens again.

* * *

Note that we cannot directly support end users of games impacted by this problem, please use the official support channels of the game publisher, as well as the support channels of the processor and motherboard manufacturers.