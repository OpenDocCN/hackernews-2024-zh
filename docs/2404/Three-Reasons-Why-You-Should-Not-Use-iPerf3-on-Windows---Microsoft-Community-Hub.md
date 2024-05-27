<!--yml
category: 未分类
date: 2024-05-27 13:20:57
-->

# Three Reasons Why You Should Not Use iPerf3 on Windows - Microsoft Community Hub

> 来源：[https://techcommunity.microsoft.com/t5/networking-blog/three-reasons-why-you-should-not-use-iperf3-on-windows/ba-p/4117876](https://techcommunity.microsoft.com/t5/networking-blog/three-reasons-why-you-should-not-use-iperf3-on-windows/ba-p/4117876)

James Kehr here with the Microsoft Commercial Support – Windows Networking team. This article will explain why you should not use iPerf3 on Windows for synthetic network benchmarking and testing.  Followed by a brief explanation of why you should use ntttcp and ctsTraffic instead.

***UPDATE (22 April 2024): Various update tags were made throughout the article based on feedback in the comments.***

# Reason 1 – ESnet Does Not Support Windows

[iPerf3](https://software.es.net/iperf/) is owned and maintained by an organization called ESnet (Energy Sciences Network). They do not officially support nor recommend that iPerf3 be used on Windows. Their recommendation is to use iPerf2\. More on the Microsoft recommendation later.

Here are some direct quotes from the official [ESnet iPerf3 FAQ](https://software.es.net/iperf/faq.html), retrieved on 18 April 2024.

***I’m trying to use iperf3 on Windows, but having trouble. What should I do?***

*iperf3 is not officially supported on Windows, but iperf2 is. We recommend you use iperf2.*

***UPDATE (22 April 2024): Please read the [comment](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Ftechcommunity.microsoft.com%2Ft5%2Fnetworking-blog%2Fthree-reasons-why-you-should-not-use-iperf3-on-windows%2Fbc-p%2F4119600%2Fhighlight%2Ftrue%23M633&data=05%7C02%7CJames.Kehr%40microsoft.com%7C27c73f372e86443d986208dc62ecea72%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638494016911846998%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C0%7C%7C%7C&sdata=mXIpLUx6lxFC8R%2FMoMHfQM7aeNsscyXRkFLzIiQqWUw%3D&reserved=0)(s) from [@rjmcmahon](/t5/user/viewprofilepage/user-id/2431963) for details about iPerf2 vs iPerf3.***

And from the ESnet “[Obtaining iPerf3](https://software.es.net/iperf/obtaining.html)” article, retrieved on 18 April 2024.

*Primary development for iperf3 takes place on CentOS 7 Linux, FreeBSD 11, and macOS 10.12\. At this time, these are the only officially supported platforms…*

Microsoft does not recommend using iPerf3 for a different reason.

# Reason 2 – iPerf3 is Emulated on Windows

iPerf3 does not make Windows native API calls. It only knows how to make Linux/POSIX calls.

The iPerf3 community uses Cygwin as an emulation layer to get iPerf3 working on Windows. You can read more about Cygwin in their [FAQ](https://www.cygwin.com/faq.html).

The iPerf3 calls are sent to Cygwin, which translates them to Windows APIs calls. Only then does the Windows network stack come into play. The iPerf3 on Windows maintainers do an excellent job of making it all work together, but, ultimately, there are potential issues with this approach.

Not all the iPerf3 features will work on Windows. The basic options work well, but advanced capabilities needed for certain network testing may not be available on Windows or may behave in unexpected ways.

Emulation tends to have a performance penalty. The emulation overhead on a latency sensitive operation, such as network testing, can result in lower than expected throughput.

Finally, iPerf3 uses uncommon Windows Socket (winsock) options versus native Windows applications. For generic throughput testing this is fine. For application testing the uncommon socket options will not mimic real-world Windows-native application behavior.

# Reason 3 – You Are Probably Using an Old Version of iPerf3.

***UPDATE (22 April 2024): iperf.fr no longer serves the old Windows iPerf3 binaries. The site now links to other sites which have actively maintained iPerf3 for Windows binaries. A big thank you to [@Harvester](/t5/user/viewprofilepage/user-id/598981) for pointing this out in comments and to the iperf.fr team for updating their site!***

Go search for “iPerf3 on Windows” on the web. Go ahead, open a tab, and use your search engine of choice. Which I am certain is Bing with Copilot.

What is the top result, and thus the most likely link you will click on? I bet the site was iperf.fr.

The newest version of iPerf3 for Windows on iperf.fr is 3.1.3 from 8 June 2016\. That was nearly 8 years ago at the time of writing.

The current version of iPerf3, [directly from ESnet](https://github.com/esnet/iperf), is 3.16\. A full 15 versions of missing bug fixes, features, and changes from the version people are most likely to download.

This specific copy of iPerf3, from iperf.fr, includes a version of cygwin1.dll that contains a bug which limits the socket buffer to 1MB. This will cause poor performance on high speed-high latency and high bandwidth networks because iPerf3 will not be capable of putting enough data in-flight to saturate the link, resulting in inaccurate testing.

Where should you look for iPerf3 on Windows?

From ESnet’s article, “Obtaining iPerf3” they say:

*Windows: iperf3 binaries for Windows (built with [Cygwin](https://www.cygwin.com/)) can be found in a variety of locations, including [https://files.budman.pw/](https://files.budman.pw/) ([discussion thread](https://www.neowin.net/forum/topic/1234695-iperf/)).*

# What Does Microsoft Recommend

Microsoft maintains two synthetic network benchmarking tools: ntttcp (Windows NT Test TCP) and ctsTraffic. The newest version of [ntttcp is maintained on GitHub](https://github.com/microsoft/ntttcp). This is a Windows native tool which utilizes Windows networking in the same way a native Windows application does.

But what about Linux?

There is a Linux version of ntttcp, too! Details can be found on the [ntttcp for Linux GitHub](https://github.com/microsoft/ntttcp-for-linux) repo. This is a separate codebase built for Linux that is compatible with ntttcp for Windows, but it is not identical to the Windows counterpart.

Ntttcp allows you to perform API native synthetic network tests between Windows and Windows, Linux and Linux, and between Windows and Linux.

[ctsTraffic](https://github.com/microsoft/ctsTraffic) is Windows-to-Windows only. Where ntttcp is more iPerf3-like, ctsTraffic has a different set of options and goals. ctsTraffic focuses on end-to-end goodput scenarios, where ntttcp and iPerf3 focus more on isolating network stack throughput.

## How do you use ntttcp?

The Azure team has written a great article about basic ntttcp functionality for Windows and Linux. I do not believe in reinventing the wheel, so I will simply link you to the article.

[https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-bandwidth-testing?tabs=windo...](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-bandwidth-testing?tabs=windows)

There is a known interoperability limitation when testing between Windows and Linux. Details can be found in this [ntttcp for Linux wiki article on GitHub](https://github.com/microsoft/ntttcp-for-linux/wiki/How-to-interop-with-Windows-NTttcp%3F).

# Testing

I built a lab while preparing this article using two Windows Server 2022 VMs. The tests used the newest versions of iPerf3 (3.16), ntttcp (5.39), and ctsTraffic (2.0.3.3).

The default iPerf3 parameters are the most common configuration I see among Microsoft support customers. So, I am tuning ntttcp and ctsTraffic to better match iPerf3’s default single connection, 128KB buffer length behavior. While this is not a perfect comparison, this does make it a better comparison.

Single stream tests are used for targeted analyses since many applications do not perform multi-threaded transfers. Bandwidth and maximum throughput testing should be multi-threaded with large buffers, but that is a topic for a different day.

Don’t forget to allow the network traffic on the Windows Defender Firewall if you wish to run your own tests.

## iPerf3

iPerf3 server command:

```
iperf3 -s
```

iPerf3 client command:

```
iperf3 -c <IP> -t 60
```

The average across multiple tests was about 7.5 Gbps. The top result was 8.5 Gbps, with a low of 5.26 Gbps.

## ntttcp

Ntttcp server command:

```
ntttcp -r -m 1,*,<IP> -t 60
```

Ntttcp client command:

```
ntttcp -s -m 1,*,<IP> -l 128K -t 60
```

Ntttcp averaged about 12.75 Gbps across multiple tests. The top test averaged 13.5 Gbps, with a low test of 12.5 Gbps.

Ntttcp does something called pre-posting receives, which is unique to this tool. This reduces application wait time as part of network stack isolation, allowing for quicker than normal application responses to socket messages.

*-r* is receiver, and *-s* is sender.

*-m* is a mapping of values that are: <num threads>, <CPU affinity>, <Target IP>. In this test we use a single thread, no CPU affinity (*), and both -r and -s side uses the target IP address as the final value.

*-t* is test time, in seconds.

*-l* sets the buffer length. You can use K|M|G with ntttcp as shorthand for kilo-, mega-, and giga-bytes.

## ctsTraffic

These commands are run in PowerShell to make reading values easier.

ctsTraffic server command:

```
.\ctstraffic.exe -listen:* -Buffer:"$(128KB)" -Transfer:"$(1TB)" -ServerExitLimit:1 -consoleverbosity:1 -TimeLimit:60000
```

ctsTraffic client command:

```
.\ctstraffic.exe -target:<IP> -Connections:1 -Buffer:"$(128KB)" -Transfer:"$(1TB)" -Iterations:1 -consoleverbosity:1 -TimeLimit:60000
```

The result, about 9.2 Gbps average. It is a little faster and far more consistent than iPerf3, but not quite as fast as ntttcp. The two primary reasons why ctsTraffic is slower are data integrity checks and the use of the recommended overlapped IO model. This means ctsTraffic uses a single pending receive versus pre-posting receives like ntttcp.

*-Buffer* is the buffer length (ntttp: -l).

*-Transfer* is the amount of data to send per iteration.

*-Iterations/-ServerExitLimit* is the number of times a data sets will be transferred.

*-Connections* is the number of concurrent TCP streams that will be used.

*-TimeLimit* is the number of milliseconds to run the test. The test stops even if the iteration transfer has not been completed when the time limit is reached.

Thank you for reading and I hope this helps improve your understanding of synthetic network benchmarking on Windows!