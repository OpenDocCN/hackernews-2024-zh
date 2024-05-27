<!--yml
category: 未分类
date: 2024-05-27 13:01:06
-->

# How Engineers at Digital Equipment Corp. Saved Ethernet - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/how-dec-engineers-saved-ethernet](https://spectrum.ieee.org/how-dec-engineers-saved-ethernet)

I’ve enjoyed reading magazine articles about Ethernet’s 50th anniversary, including one in the [*The Institute*](https://spectrum.ieee.org/ethernet-ieee-milestone). Invented by computer scientists [Robert Metcalfe](https://ethw.org/Robert_M._Metcalfe) and [David Boggs](https://ethw.org/David_Boggs), Ethernet has been extraordinarily impactful. Metcalfe, an IEEE Fellow, received the 1996 [IEEE Medal of Honor](https://corporate-awards.ieee.org/recipients/ieee-medal-of-honor-recipients/) as well as the 2022 [Turing Award](https://amturing.acm.org/) from the [Association for Computing Machinery](https://www.acm.org/) for his work. But there is more to the story that is not widely known.

During the 1980s and early 1990s, I led [Digital Equipment Corp.](https://en.wikipedia.org/wiki/Digital_Equipment_Corporation)’s networking advanced development group in Massachusetts. I was a firsthand witness in what was a period of great opportunity for LAN technologies and intense competition between standardization efforts.

DEC, [Intel](https://www.google.com/aclk?sa=l&ai=DChcSEwi2hOvzgu-EAxWZH60GHXwkAToYABAAGgJwdg&ase=2&gclid=Cj0KCQjw-r-vBhC-ARIsAGgUO2DU3b9FN2WnvoVFOiwwrcDDz5q7PPS2MiSG6DRs-u-xJc9j5_yBWkcaAn2LEALw_wcB&ei=dHLwZY3bOdDB0PEP7_WwiAU&sig=AOD64_3OM2YVYFwaHUS5T6_5SPLa3qL80Q&q&sqi=2&nis=4&adurl&ved=2ahUKEwiN5ePzgu-EAxXQIDQIHe86DFEQ0Qx6BAgIEAE), and [Xerox](https://www.xerox.com/en-us/about) poised themselves to profit from Ethernet’s launch in the 1970s. But during the 1980s other LAN technologies emerged as competitors. Prime contenders included the [token ring](https://www.geeksforgeeks.org/efficiency-of-token-ring/), promoted by [IBM](https://www.ibm.com/about), and the [token bus](https://www.sciencedirect.com/topics/computer-science/token-bus). (Today Ethernet and both token-based technologies are part of the [IEEE 802](https://www.ieee802.org/) family of standards.)

All those LANs have some basic parts in common. One is the 48-bit media access control (MAC) address, a unique number assigned during a computer’s network port manufacturing process. The MAC addresses are used inside the LAN only, but they are critical to its operation. And usually, along with the general-purpose computers on the network, they have at least one special-purpose computer: a router, whose main job is to send data to—and receive it from—the Internet on behalf of all the other computers on the LAN.

In a decades-old conceptual model of networking, the LAN itself (the wires and low-level hardware) is referred to as Layer 2, or the data link layer. Routers mostly deal with another kind of address: a network address that is used both within the LAN and outside it. Many readers likely have heard the terms *Internet Protocol* and *IP address*. With some exceptions, the IP address (a network address) in a data packet is sufficient to ensure that packet can be delivered anywhere on the Internet by a sequence of other routers operated by service providers and carriers. Routers and the operations they perform are referred to as Layer 3, or the network layer.

In a token ring LAN, shielded twisted-pair copper wires connect each computer to its upstream and downstream neighbors in an endless ring structure. Each computer forwards data from its upstream neighbor to its downstream one but can send its own data to the network only after it receives a short data packet—a token—from the upstream neighbor. If it has no data to transmit, it just passes the token to its downstream neighbor, and so on.

In a token bus LAN, a coaxial cable connects all the network’s computers, but the wiring doesn’t control the order in which the computers pass the token. The computers agree on the sequence in which they pass the token, forming an endless virtual ring around which data and tokens circulate.

Ethernet, meanwhile, had become synonymous with coaxial cable connections that used a method called *carrier sense multiple access with collision detection* for managing transmissions. In the CSMA/CD method, computers that want to transmit a data packet first listen to see if another computer is transmitting. If not, the computer sends its packet while listening to determine whether that packet collides with one from another computer. Collisions can happen because signal propagation between computers is not instantaneous. In the case of a collision, the sending computer resends its packet with a delay that has both a random component and an exponentially increasing component that depends on the number of collisions.

The need to detect collisions involves tradeoffs among data rate, physical length, and minimum packet size. Increasing the data rate by an order of magnitude means either reducing the physical length or increasing the minimum packet size by roughly the same factor. The designers of Ethernet had wisely chosen a sweet spot among the tradeoffs: 10 megabits per second and a length of 1,500 meters.

## A threat from fiber

Meanwhile, a coalition of companies—including my employer, DEC—was developing a new ANSI LAN standard: the [Fiber Distributed Data Interface](https://www.techtarget.com/searchnetworking/definition/FDDI). The FDDI approach used a variation of the token bus protocol to transmit data over optical fiber, promising speeds of 100 Mb/s, far faster than Ethernet’s 10 Mb/s.

A barrage of technical publications released analyses of the throughputs and latencies of competing LAN technologies under various workloads. Given the results and the much greater network performance demands expected from speedier processors, RAM, and nonvolatile storage, Ethernet’s limited performance was a serious problem.

FDDI seemed a better bet for creating higher speed LANs than Ethernet, although FDDI used expensive components and complex technology, especially for fault recovery. But all shared media access protocols had one or more unattractive features or performance limitations, thanks to the complexity involved in sharing a wire or optical fiber.

## A solution emerges

I thought that a better approach than either FDDI or a faster version of Ethernet would be to develop a LAN technology that performed store-and-forward switching.

One evening in 1983, just before leaving work to go home, I visited the office of [Mark Kempf](https://www.linkedin.com/in/mark-kempf-4a2668b7), a principal engineer and a member of my team. Mark, one of the best engineers I have ever worked with, had designed the popular and profitable DECServer 100 terminal server, which used the local-area transport (LAT) protocol created by Bruce Mann from DEC’s corporate architecture group. Terminal servers connect groups of dumb terminals, with only RS-232 serial ports, to computer systems with Ethernet ports.

I told Mark about my idea of using store-and-forward switching to increase LAN performance.

The next morning he came in with an idea for a learning bridge (also known as a Layer 2 switch or simply a switch). The bridge would connect to two Ethernet LANs. By listening to all traffic on each LAN, the device would learn the MAC addresses of the computers on both Ethernets (remembering which computer was on which Ethernet) and then selectively forward the appropriate packets between the LANs based upon the destination MAC address. The computers on the two networks didn’t need to know which path their data would take on the extended LAN; to them, the bridge was invisible.

The bridge would need to receive and process some 30,000 packets per second (15,000 pp/s per Ethernet) and decide whether to forward each one. Although the 30,000 pp/s requirement was near the limit of what could be done using the best microprocessor technology of the time, the Motorola 68000, Mark was confident he could build a two-Ethernet bridge using only off-the-shelf components including a specialized hardware engine he would design using programmable array logic (PAL) devices and dedicated static RAM to look up the 48-bit MAC addresses.

Mark’s contributions have not been widely recognized. One exception is the textbook [*Network Algorithmics*](https://www.amazon.com/Network-Algorithmics-Interdisciplinary-Designing-Networking/dp/0120884771) by George Varghese.

In a misconfigured network—one with bridges connecting Ethernets in a loop—packets could circulate forever. We felt confident that we could figure out a way to prevent that. In a pinch, a product could ship without the safety feature. And clearly a two-port device was only the starting point. Multiple-port devices could follow, though they would require custom components.

I took our idea to three levels of management, looking for approval to build a prototype of the learning bridge that Mark envisioned. Before the end of the day, we had a green light with the understanding that a product would follow if the prototype was successful.

## Developing the bridge

My immediate manager at DEC, Tony Lauck, challenged several engineers and architects to solve the problem of packet looping in misconfigured networks. Within a few days, we had several potential solutions. [Radia Perlman](https://en.wikipedia.org/wiki/Radia_Perlman), an architect in Tony’s group, provided the clear winner: the spanning tree protocol.

In Perlman’s approach, the bridges detect each other, select a root bridge according to specified criteria, and then compute a minimum spanning tree. An MST is a mathematical structure that, in this case, describes how to efficiently connect LANs and bridges without loops. The MST was then used to place any bridge whose presence would create a loop into backup mode. As a side benefit, it provided automated recovery in the case of a bridge failure.

The logic module of a disassembled LANBridge 100, which was released by Digital Equipment Corp. in 1986\. Alan Kirby

Mark designed the hardware and timing-sensitive low-level code, while software engineer Bob Shelly wrote the remaining programs. And in 1986, DEC introduced the technology as the LANBridge 100, product code DEBET-AA.

Soon after, DEC developed DEBET-RC, a version that supported a 3-kilometer optical fiber span between bridges. Manuals for some of the DEBET-RCs can be found on the [Bitsavers website](https://bitsavers.org/).

Mark’s idea didn’t replace Ethernet—and that was its brilliance. By allowing store-and-forward switching between existing CSMA/CD coax-based Ethernets, bridges allowed easy upgrades of existing LANs. Since any collision would not propagate beyond the bridge, connecting two Ethernets with a bridge would immediately double the length limit of a single Ethernet cable alone. More importantly, placing computers that communicated heavily with each other on the same Ethernet cable would isolate that traffic to that cable, while the bridge would still allow communication with computers on other Ethernet cables.

That reduced the traffic on both cables, increasing capacity while reducing the frequency of collisions. Taken to its limit, it eventually meant giving each computer its own Ethernet cable, with a multiport bridge connecting them all.

That is what led to a gradual migration away from CSMA/CD over coax to the now ubiquitous copper and fiber links between individual computers and a dedicated switch port.

The speed of the links is no longer limited by the constraints of collision detection. Over time, the change completely transformed how people think of Ethernet.

A bridge could even have ports for different LAN types if the associated packet headers were sufficiently similar.

Our team later developed GIGAswitch, a multiport device supporting both Ethernet and FDDI.

The existence of bridges with increasingly higher performance took the wind out of the sails of those developing new shared media LAN access protocols. FDDI later faded from the marketplace in the face of faster Ethernet versions.

Bridge technology was not without controversy, of course. Some engineers continue to believe that Layer 2 switching is a bad idea and that all you need are faster Layer 3 routers to transfer packets between LANs. At the time, however, IP had not won at the network level, and DECNet, IBM’s SNA, and other network protocols were fighting for dominance. Switching at Layer 2 would work with any network protocol.

Mark received a [U.S. patent](https://patents.google.com/patent/US4597078A/en) for the device in 1986\. DEC offered to license it on a no-cost basis, allowing any company to use the technology.

That led to an IEEE standardization effort. Established networking companies and startups adopted and began working to improve the switching technology. Other enhancements—including switch-specific ASICs, virtual LANs, and the development of faster and less expensive physical media and associated electronics—steadily contributed to Ethernet’s longevity and popularity.

The lasting value of Ethernet lies not in CSMA/CD or its original coaxial media but in the easily understood and functional service that it provided for protocol designers.

The switches in many home networks today are directly descended from the innovation. And modern data centers have numerous switches with individual ports running between 40 and 800 gigabits per second. The data center switch market alone accounts for more than US $10 billion in annual revenue.

Lauck, my DEC manager, once said that the value of an architecture can be measured by the number of technology generations over which it is useful. By that measure, Ethernet has been enormously successful. The same can be said of Layer 2 switching.

No one knows what would have happened to Ethernet had Mark not invented the learning bridge. Perhaps someone else would have come up with the idea. But it’s also possible that Ethernet would have slowly withered away.

To me, Mark saved Ethernet.