<!--yml
category: 未分类
date: 2024-05-27 12:54:06
-->

# XDP for Game Programmers

> 来源：[https://mas-bandwidth.com/xdp-for-game-programmers/](https://mas-bandwidth.com/xdp-for-game-programmers/)

I'm [Glenn Fiedler](https://www.linkedin.com/in/glenn-fiedler-11b735302/?ref=mas-bandwidth.com) and welcome to **Más Bandwidth**, my new blog at the intersection of game network programming and scalable backend engineering.

What's this new blog about? Games with thousands of players. Virtual worlds. Performance in virtual spaces to an audience of *millions*. **The Metaverse** (no, not THAT metaverse, the ***real metaverse**, sans blockchain*). Overlay worlds in AR. Telepresence and remote working in virtual reality. Game streaming *(eww gross!).* OK, everything but that last one and I'm going to explain why in a later article.

But seriously. Within 10 years, everybody is going to have 10gbps internet. How will this change how games are made? How are we going to use all this bandwidth? 5v5 shooters just aren't going to cut it anymore. **What's next?**

As a fitting first post to this blog, if ever you find yourself, as I did, needing the ***absolute maximum bandwidth*** for an application, then you'll need to use a kernel bypass technology. Why? Because otherwise, the overhead of processing each packet in the kernel and passing it down to user space and back up to the kernel and out to the NIC limits the throughput you can achieve. We're talking 10gbps and above here.

The good news is that in the last 5 years the Linux kernel bypass technology known as XDP/eBPF has matured enough that it has moved from the domain of kernel hackers, to now at the beginning of 2024, being generally usable by normal people like you and me.

So in this article I'm going to give you a quick overview of how XDP/eBPF works, show you what you can actually do with XDP/eBPF, *and* give you some working example code ([https://github.com/mas-bandwidth/xdp](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com)) for some simple XDP programs so you can start using this technology in your applications.

You can do some truly amazing things in XDP so read on!

* * *

### What is XDP and how does it work?

XDP stands for "express data path" and it's basically a way for you to write a function that gets called at the very earliest point when a packet comes off the NIC, right before the Linux kernel does any allocations or processing for the packet.

This is incredible. This is powerful. This is programmer crack. You can write a function that runs inside the Linux Kernel and you can do *almost anything you want*. You can:

*   Drop the packet
*   Modify the packet contents, or replace it entirely
*   Grow or shrink the packet at head or the tail
*   Send a response packet, or forward the packet to another address

*OR*

*   *Pass the packet down to the kernel for regular processing*

The last one is key. With other kernel bypass technologies like DPDK you needed to install a second NIC to run your program or basically implement (or license) an entire TCP/IP network stack to make sure that everything works correctly under the hood (the NIC is doing a lot more than just processing UDP packets for your game...).

Now you can just laser focus your XDP program to apply, for example, only to IPv4 UDP packets sent to port 40000, and pass everything else on to the Linux kernel for regular processing. Easy.

***Correction:** Apparently these days you can use a "bifurcated driver" with DPDK now to pass certain packets back to the OS. This wasn't available the last time I worked with DPDK, which was quite some time ago. I still prefer XDP over DPDK though.*

### What is eBPF**?**

eBPF stands for "extended Berkeley Packet Filter" and it's the technology that lets you compile, link and run your XDP program in the Linux Kernel.

In short, eBPF is a byte code and lightweight VM that runs functions inside the Linux Kernel. There are many different places that eBPF functions can be inserted, and XDP is just one of them.

Because eBPF functions run inside the Linux kernel, they must not crash and they absolutely must halt. To make sure this is true, BPF functions have to pass a verifier before they can be loaded into the Kernel.

In practice, this means that XDP functions are very slightly limited in what they can do. They're not Turing complete (halting problem), and you have to do a lot of dancing about to prove to the verifier that you aren't writing outside of bounds. But in practice, as long as you keep things simple, and you're willing to creatively do battle with the verifier, you can *usually* convince it that your program is safe.

### Setting up for eBPF/XDP on Ubuntu 22.04 LTS

Before you can get started wring XDP programs, you need to set up your machine so that it can compile, link and run eBPF programs, and load them in the kernel.

Starting from an Ubuntu 22.04 LTS distribution.

First you need to make sure you have the 6.5 Linux kernel:

```
uname -r 
```

If the output from this isn't version 6.5, update your kernel with:

```
sudo apt install linux-generic-hwe-22.04 -y 
```

Run the following in your command line:

```
# install necessary packages

sudo NEEDRESTART_SUSPEND=1 apt autoremove -y
sudo NEEDRESTART_SUSPEND=1 apt update -y
sudo NEEDRESTART_SUSPEND=1 apt upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt dist-upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt full-upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt install libcurl3-gnutls-dev build-essential vim wget libsodium-dev flex bison clang unzip libc6-dev-i386 gcc-12 dwarves libelf-dev pkg-config m4 libpcap-dev net-tools -y
sudo NEEDRESTART_SUSPEND=1 apt install linux-headers-`uname -r` linux-tools-`uname -r` -y
sudo NEEDRESTART_SUSPEND=1 apt autoremove -y

# install libxdp and libbpf from source

cd ~
wget https://github.com/xdp-project/xdp-tools/releases/download/v1.4.2/xdp-tools-1.4.2.tar.gz
tar -zxf xdp-tools-1.4.2.tar.gz
cd xdp-tools-1.4.2
./configure
make -j && sudo make install

cd lib/libbpf/src
make -j && sudo make install
sudo ldconfig

# setup vmlinux btf

sudo cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/ 
```

In summary, the key step is building **libxdp** from source, and then building and installing the exact version of **libbpf** that is *included* in **libxdp**.

I can only guess why this is necessary, but without this, I have found no other way to get XDP fully working on Ubuntu 22.04, including all functionality like BTFs, kfuncs and kernel modules. More on those later.

### XDP Reflect

Now we'll build and run a simple XDP program. In this program we'll just reflect UDP packets sent to us on port 40000 back to the sender. All other packets are passed to the kernel for regular processing.

First, clone my [XDP example repo](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com) from GitHub:

```
git clone https://github.com/mas-bandwidth/xdp 
```

Change into the reflect dir and make the program:

```
cd xdp/reflect && make 
```

Run the UDP reflect program, passing in the name of the network interface to attach the program to. You can use **ifconfig** to list the network interfaces on your Linux machine.

```
sudo ./reflect enp4s0 
```

Open up another terminal window to watch logs from the XDP program:

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

Next clone the XDP repo again on another machine, and run the corresponding client for the reflect program, replacing 192.168.1.40 with the IP address of your Linux machine running the XDP program:

```
git clone https://github.com/mas-bandwidth/xdp
cd xdp/reflect && go run client.go 192.168.1.40 
```

If everything is working, you should see logs like this:

```
gaffer@batman reflect % go run client.go
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000 
```

Congratulations, just you've built and run your first XDP program, and it's no toy. If you simply comment out the **#define DEBUG 1** line in reflect_xdp.c, it's capable of reflecting packets at line rate on a 10G NIC.

### XDP Drop

Next we'll run a program that listens on a UDP port and drops packets that don't match a pattern. This type of XDP program can be useful to harden your game server against DDoS, although it's certainly not a panacea.

The general idea is to hash key packet data, for example: packet length, source and dest addresses and ports, and potentially also some rolling magic number that changes every minute if you want to get fancy. While it's not perfect, and it doesn't protect against packet replay attacks, at least a randomly generated UDP packet will fail the pattern check.

The trick is to *shmear* this 8 byte hash across 15 bytes at the start of the packet in a reversible way, effectively doing the opposite of compression. We're going to store this data very inefficiently, such that each byte in the header has a very narrow range of values that are actually valid, and most are not. Now we have an extremely low entropy pattern we can check for, without even calculating the hash.

Here's an example that takes the hash and fills a 16 byte header, with 15 bytes of a low entropy encoding of the hash, and the first byte reserved for packet type:

```
func GeneratePacketHeader(packet []byte, sourceAddress *net.UDPAddr, destAddress *net.UDPAddr) {

    var packetLengthData [2]byte
    binary.LittleEndian.PutUint16(packetLengthData[:], uint16(len(packet)))

    hash := fnv.New64a()
    hash.Write(packet[0:1])
    hash.Write(packet[16:])
    hash.Write(sourceAddress.IP.To4())
    hash.Write(destAddress.IP.To4())
    hash.Write(packetLengthData[:])
    hashValue := hash.Sum64()

    var data [8]byte
    binary.LittleEndian.PutUint64(data[:], uint64(hashValue))

    packet[1] = ((data[6] & 0xC0) >> 6) + 42
    packet[2] = (data[3] & 0x1F) + 200
    packet[3] = ((data[2] & 0xFC) >> 2) + 5
    packet[4] = data[0]
    packet[5] = (data[2] & 0x03) + 78
    packet[6] = (data[4] & 0x7F) + 96
    packet[7] = ((data[1] & 0xFC) >> 2) + 100

    if (data[7] & 1) == 0 {
        packet[8] = 79
    } else {
        packet[8] = 7
    }
    if (data[4] & 0x80) == 0 {
        packet[9] = 37
    } else {
        packet[9] = 83
    }

    packet[10] = (data[5] & 0x07) + 124
    packet[11] = ((data[1] & 0xE0) >> 5) + 175
    packet[12] = (data[6] & 0x3F) + 33

    value := (data[1] & 0x03)
    if value == 0 {
        packet[13] = 97
    } else if value == 1 {
        packet[13] = 5
    } else if value == 2 {
        packet[13] = 43
    } else {
        packet[13] = 13
    }

    packet[14] = ((data[5] & 0xF8) >> 3) + 210
    packet[15] = ((data[7] & 0xFE) >> 1) + 17
} 
```

To run the xdp drop program just change into the 'drop' directory and run it on your network interface:

```
cd xdp/drop && sudo ./drop enp4s0 
```

And then on another computer, run the drop client, replacing the addresses with the client and drop XDP program addresses respectively:

```
cd xdp/drop && go run client.go 192.168.1.20 192.168.1.40 
```

On the XDP machine you'll see in the logs that the packet filter passed:

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

Try modifying the client.go to send randomly generated packet data without the header. You'll see in the logs now that the packet filter drops the packets. The encoding of the hash is so low entropy that it's virtually impossible for a randomly generated packet to pass the packet filter.

If you end up using this technique in production, *please* make sure to change the low entropy encoding to something unique for your game, because script kiddies read these articles too. In addition, make sure your encoding is reversible, so you can reconstruct the hash on the receiver side and drop the packet in XDP when the hash doesn't match the expected value. Now people can't spoof their source address or port!

### XDP Whitelist

What about an even simpler approach? Why not just maintain a list of IP addresses that are allowed to communicate with the game server, and drop any packets that aren't from a whitelisted address?

Sure, you need to do some work in the backend to "open" client addresses on the server prior to connect, and you have to do some work to "close" the address when clients disconnect... but this is do-able, and now, packets thrown at your game server from random addresses will be dropped by XDP, before the Linux kernel does any work to process them.

To do this we need a way to communicate the whitelist to the XDP program. And here we get to use a new feature of BPF: **Maps**.

Maps are an incredibly rich set of data structures that you can use from BPF. Arrays, Hashes, Per-CPU arrays, Per-CPU hashes and so on. All of these data structures are lockless, and you can read and write to them both from inside BPF programs, and from user space programs.

If you see where I'm going here, you now have a way to communicate from your BPF program back out to user space and vice-versa. It's almost too easy now: just call functions in your user space program to add and remove entries from a whitelist hash map.

Run the whitelist XDP program, replacing the interface name with your own:

```
cd xdp/whitelist && sudo ./drop enp4s0 
```

And then on another computer, run the whitelist client, replacing the addresses with the client and drop XDP program addresses respectively:

```
cd xdp/whitelist && go run client.go 192.168.1.20 192.168.1.40 
```

If you look at the logs from the XDP program:

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

You'll see that it prints out that it's dropping the packets because they're not in the whitelist. Edit **whitelist/whitelist.c** and add the address of the machine running client.go, then reload the XDP program. Run the client.go again, and the packets should pass. At this point, if you bind a UDP socket to port 40000 on the XDP machine, it would receive only packets that pass the whitelist check.

If you use this in production, you'll need to write your own system to add and remove whitelist entries. Maybe your server hits a backend periodically for the list of open addresses? Maybe it subscribes to some queue? In addition, the whitelist hash values in this example are empty, but you could put data in there. What about a secret key per-client to make the packet filter hash more secure? You can combine whitelist with packet filters and hash checks to stop attackers from spoofing their IPv4 source address to get through the whitelist.

### XDP Relay

What if your game server gets still gets overloaded by DDoS attacks even though XDP is dropping packets with whitelists, packet filter checks and hash checks?

The DDoS attacks are getting bigger. Much bigger. Congratulations, your game is **super successful**. Why not put a relay in front of your game server, that only forwards valid packets, hiding the IP address of your game server entirely? You could have relays in each datacenter protecting your game servers, and these relays could have 10, 40 or 100gbps NICs.

I'm leaving this one as an exercise for the reader. Combine the whitelist approach above with enough information in the whitelist hash value entry for the relay to forward the packet from the client to the server and vice-versa.

Now drop any packets from addresses that aren't in the whitelist as quickly as possible. Bonus points: track sequence numbers per-client connection to avoid replay attacks and rate limit the client connection to some maximum bandwidth envelope per-client. This is starting to become a pretty solid system.

At this point you basically have your own version of Steam Data Relay (SDR) that isn't free. It's quite possibly even better than SDR. Well done! If you have infinite resources and your own money printing machine in your basement like Valve has, you can afford to run this system at scale too.

### Network Acceleration with XDP

Did you know that at any time around 5-10% of your players are experiencing bad network performance like much higher latency than usual, high jitter or high packet loss? It's difficult to believe, but it's true. I have the data from more than 50 million unique players to prove it.

Even more interesting is the fact that this bad network performance moves around and affects ~90% of your players every month. It's not the same 5-10% of players every day.

This is not just some small minority of players with bad internet connections, this is a systemic problem. An inconsistency in network performance from one match to the next that affects the majority of your players.

My company [Network Next](https://networknext.com/?ref=mas-bandwidth.com) fixes this problem by steering game packets through relays, and yes... we even beat Google cloud to their own datacenters with premium transit, Amazon to their own datacenters with Amazon Global Accelerator, and we massively outperform Steam Data Relay (SDR).

And the Network Next relays are implemented in XDP.

### Crypto in XDP

One problem I encountered when implementing the Network Next relay was, how can I access crypto from inside my XDP program? Sure I can forward or drop packets quickly, but my decision whether I would forward or drop packets was based not only on whitelists, packet filters and hash checks, but also on cryptographic tests like sha256 and chachapoly.

While it's certainly possible that I could fight the verifier and write my own cryptographic primitives in BPF directly, it seemed counter productive. I'd spend a lot of time fighting the verifier and in the end, it might not even be possible to implement a given crypto primitive within the limitations of the verifier. It has an uncanny ability *not* to not be able to see why your code is completely fucking safe. Seriously, by the end of writing an XDP program you really want to punch it in the face.

Coming to the rescue are the last two features of BPF that we'll discuss in this article. **BTF** and **kfuncs**.

In short, you can write your own kernel module, then export functions from that module called kernel funcs or (kfuncs), and call them from inside XDP. You can even annotate those functions so that the BPF verifier knows, ok, this is function parameter is a void pointer *data*, and here is the length of data int *data__sz.* These annotations are done via BTF, a sort of light weight type system that exports type data from the Linux kernel, *including kernel modules*, so they can be accessed from BPF.

Using this we can implement this function inside our own custom **crypto_module** kernel module to perform sha256 using the existing Linux kernel crypto primitives.

To see this in action, first build and load the kernel module:

```
cd xdp/crypto
make module 
```

Next, build and run the XDP program, replacing the network interface name with your own:

```
make && sudo ./crypto enp4s0 
```

Now on another computer, change into the crypto dir and run the client, replacing the address with the IP address of the machine running the XDP program:

```
cd xdp/crypto && go run client.go 192.168.1.40 
```

If everything goes correctly, you'll see the XDP program respond with a 32 byte packets for each packet you send, containing the sha256 of the first 256 bytes of your packets. Why only the first 256 bytes? Well, to understand this, you need to understand the limitations of the BPF verifier...

### Limitations of the BPF Verifier

With kfuncs, it would seem that you should be able to now, pass the entire *void * packet*, and *int packet__sz* from XDP in to a kfunc and process it entirely inside your kernel module.

Well, not so fast. The BPF verifier has some limitations and these restrict what you can do (at least in 2024). Hopefully these get fixed in the future by the Linux BPF developers.

The best way I can describe the limitations of the BPF for XDP programs is that it's extremely "anchored" around processing a packet from left to right. Typically, you start at the beginning of the packet and you can check to see if there is enough bytes in the packet to read the ethernet header, then move the pointer right by some constant amount, read the ip header, move a constant amount to the right again and read the UDP header, and so on.

But if you try to write code that reads the last 2 bytes in a packet, I simply cannot find any way to make this code pass the verifier, even though the code is completely safe, and does not read memory outside of bounds.

The next limitation is that it seems (in 2024) that you can only pass constant sized portions of the packet data into kfuncs. For example, I could check to see if there is at least 256 bytes in the UDP packet payload, then call a kfunc with a pointer to the packet data and a constant size of 256 bytes, and this passes the verifier. But if you pass in the *actual size* of the packet derived from the XDP context, there just seems to be no way to convince the verifier it's safe.

This is huge shame, because if we could simply pass the XDP packet into a kernel module and do some stuff there, we'd *really* be cooking with gas. Again, hopefully this gets fixed in a future version of BPF.

### Conclusion

In this article we've explored XDP and eBPF, a kernel bypass technology in Ubuntu 22.04 LTS with Kernel 6.5\. Previously unstable and the domain of only neck-bearded kernel hackers, it's now now stable and mature enough for general use by game developers.

It's a surprisingly powerful and easy to use system once you get the hang of it. You can write an XDP function that reflects packets, drops them, forwards them, run packet filters, perform whitelist checks and even some crypto. You can do all this at line rate of 10gbps and above with minimal work. View the example source code for the article ([https://github.com/mas-bandwidth/xdp](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com)) and see for yourself.

I know it sounds crazy, but in the future I'm actually exploring implementing entire backend systems and scalable game servers in UDP request/response almost entirely inside XDP. With maps, kernel modules and kfuncs, you can really do *almost* anything you want, and if you can't, you can at worst case pass the packet down to user space for processing. For example, if you were creating a new MMO or hyper player count game in 2024, I can't think of a better foundational technology than XDP/eBPF.

I hope this article helps you get started writing XDP programs, they're extremely powerful and fun to write, even if the verifier drives you crazy. I look forward to seeing what you'll create with it!

Best wishes,

- Glenn