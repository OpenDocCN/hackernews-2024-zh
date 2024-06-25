<!--yml

category: 未分类

date: 2024-05-27 15:03:56

-->

# draft-omar-ipv10-06

> 来源：[https://datatracker.ietf.org/doc/html/draft-omar-ipv10-06.html](https://datatracker.ietf.org/doc/html/draft-omar-ipv10-06.html)

```
draft-omar-ipv10-06                                    Khaled Omar
Internet-Draft                                           The Road
Intended status: Standards Track
Expires: March 2, 2018                          September 2, 2017

                  Internet Protocol version 10 (IPv10)
                             Specification
                          draft-omar-ipv10-06

Status of this Memo

   This Internet-Draft is submitted in full conformance with the provisions
   of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering Task
   Force (IETF). Note that other groups may also distribute working documents
   as Internet-Drafts. The list of current Internet-Drafts is at
   http://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months and
   may be updated, replaced, or obsoleted by other documents at any time.
   It is inappropriate to use Internet-Drafts as reference material or to cite
   them other than as "work in progress."

   This Internet-Draft will expire on March 2, 2018.

Copyright Notice

   Copyright (c) 2017 IETF Trust and the persons identified as the document
   authors. All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal Provisions
   Relating to IETF Documents (http://trustee.ietf.org/license-info) in effect
   on the date of publication of this document. Please review these documents
   carefully, as they describe your rights and restrictions with respect to this
   document. Code Components extracted from this document must include
   Simplified BSD License text as described in Section 4.e of the Trust Legal
   Provisions and are provided without warranty as described in the Simplified
   BSD License.

Abstract

   This document specifies version 10 of the Internet Protocol (IPv10), sometimes
   referred to as IP Mixture (IPmix).

Table of Contents

   1. Introduction..................................................1
   2. Internet Protocol version 10 (IPv10)..........................3
   3\. The Four Types of Communication...............................3.
   3.1. IPv10: IPv6 Host to IPv4 Host...............................4
   3.2. IPv10: IPv4 Host to IPv6 Host...............................5
   3.3. IPv10: IPv6 Host to IPv6 Host...............................6
   3.4. IPv10: IPv4 Host to IPv4 Host...............................7
   4. IPv10 Packet Header Format....................................8
   5. Advantages of IPv10...........................................8
   6. Security Considerations.......................................9
   7. Acknowledgments...............................................9
   8. Author Address................................................9
   9. References....................................................9
   10. IANA Considerations..........................................9
   11. Full Copyright Statement.....................................9

Khaled Omar             Internet-Draft                   [Page 1]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

[1](#section-1).  Introduction

    IP version 10 (IPv10) is a new version of the Internet Protocol,
    designed to allow IP version 6 [[RFC-2460](/doc/html/rfc2460)] to communicate to
    IP version 4 (IPv4) [[RFC-791](/doc/html/rfc791)] and vice versa.

- Internet is the global wide network used for communication between
  hosts connected to it.

- These connected hosts (PCs, servers, routers, mobile devices, etc.)
  must have a global unique addresses to be able to communicate
  through the Internet and these unique addresses are defined in the
  Internet Protocol (IP).

- The first version of the Internet Protocol is IPv4.

- When IPv4 was developed in 1975, it was not expected that the number
  of connected hosts to the Internet reach a very huge number of hosts
  more than the IPv4 address space, also it was aimed to be used for
  experimental purposes in the beginning.

- IPv4 is (32-bits) address allowing approximately 4.3 billion unique
  IP addresses.

- A few years ago, with the massive increase of connected hosts to the
  Internet, IPv4 addresses started to run out.

- Three short-term solutions (CIDR, Private addressing, and NAT) were
  introduced in the mid-1990s but even with using these solutions,
  the IPv4 address space ran out in February, 2011 as announced by
  IANA, The announcement of depletion of the IPv4 address space by
  the RIRs is as follows:

  * April, 2011:      APNIC announcement.
  * September, 2012:  RIPE NCC announcement.
  * June, 2014:       LACNIC announcement.
  * September, 2015:  ARIN announcement.

- A long term solution (IPv6) was introduced to increase the address
  space used by the Internet Protocol and this was defined in the
  Internet Protocol version 6 (IPv6).

Khaled Omar             Internet-Draft                   [Page 2]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

- IPv6 was developed in 1998 by the Internet Engineering Task Force
  (IETF).

- IPv6 is (128-bits) address and can support a huge number of unique
  IP addresses that is approximately equals to 2^128 unique addresses.

- So, the need for IPv6 became a vital issue to be able to support
  the massive increase of connected hosts to the Internet after the
  IPv4 address space exhaustion.

- The migration from IPv4 to IPv6 became a necessary thing, but
  unfortunately, it would take decades for this full migration to be
  accomplished.

- 19 years have passed since IPv6 was developed, but no full migration
  happened till now and this would cause the Internet to be divided
  into two parts, as IPv4 still dominating on the Internet traffic
  (85% as measured by Google in April, 2017) and new Internet hosts
  will be assigned IPv6-only addresses and be able to communicate with
  15% only of the Internet services and apps.

- So, the need for solutions for the IPv4 and IPv6 coexistence became
  an important issue in the migration process as we cannot wake up in
  the morning and find all IPv4 hosts are migrated to be IPv6 hosts,
  especially, as most enterprises have not do this migration for
  creating a full IPv6 implementation.

- Also, the request for using IPv6 addresses in addition to the
  existing IPv4 addresses (IPv4/IPv6 Dual Stacks) in all enterprise
  networks have not achieve a large implementation that can make IPv6
  the most dominated IP in the Internet as many people believe that
  they will not have benefits from just having a larger IP address
  bits and IPv4 satisfies their needs, also, not all enterprises
  devices support IPv6 and also many people are afraid of the service
  outage that can be caused due to this migration.

- The recent solutions for IPv4 and IPv6 coexistence are:

   Native dual stack (IPv4 and IPv6)
   Dual-stack Lite
   NAT64
   464xlat
   MAP
  (other technologies also exist, like lw6over4; they may have more
  specific use cases)

- IPv4/IPv6 Dual Stack, allows both IPv4 and IPv6 to coexist by
  using both IPv4 and IPv6 addresses for all hosts at the same time,
  but this solution does not allows IPv4 hosts to communicate to
  IPv6 hosts and vice versa. Also, after the depletion of the IPv4
  address space, new Internet hosts will not be able to use IPv4/IPv6
  Dual Stacks.

- Tunneling, allows IPv6 hosts to communicate to each other through
  an IPv4 network, but still does not allows IPv4 hosts to communicate
  to IPv6 hosts and vice versa.

- NAT-PT, allows IPv6 hosts to communicate to IPv4 hosts with only
  using hostnames and getting DNS involved in the communication process
  but this solution was inefficient because it does not allows
  communication using direct IP addresses, also the need for so much
  protocol translations of the source and destination IP addresses
  made the solution complex and not applicable thats why it was moved
  to the Historic status in the [RFC 2766](/doc/html/rfc2766). Also, NAT64 requires so much
  protocol translations and statically configured bindings, and also
  getting a DNS64 involved in the communication process.

Khaled Omar             Internet-Draft                   [Page 3]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

[2](#section-2).  Internet Protocol version 10 (IPv10).

- IPv10 is the solution presented in this Internet draft.

- It solves the issue of allowing IPv6 only hosts to communicate to
  IPv4 only hosts and vice versa in a simple and very efficient way,
  especially when the communication is done using both direct IP
  addresses and when using hostnames between IPv10 hosts, as there
  is no need for protocol translations or getting the DNS involved
  in the communication process more than its normal address
  resolution function.

- IPv10 allows hosts from two IP versions (IPv4 and IPv6) to be able
  to communicate, and this can be accomplished by having an IPv10
  packet containing a mixture of IPv4 and IPv6 addresses in the same
  IP packet header.

- From here the name of IPv10 arises, as the IP packet can contain
  (IPv6 + IPv4 /IPv4 + IPv6) addresses in the same layer 3 packet
  header.

Khaled Omar             Internet-Draft                   [Page 4]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

[3](#section-3).  The Four Types of Communication.

     3.1) IPv10: IPv6 Host to IPv4 Host.
         ------------------------------

- IPv10 Packet:

         |          128-bit        |                    128-bit                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Data|   Source IPv6 Address   | 0000..0  ASN  MAC  Destination IPv4 Address |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

- Sending IPv10 host TCP/IP Configuration:

      IP Address:               IPv6 Address
      Prefix Length:            /length
      Default Gateway:          IPv6 Address (Optional)
      DNS Addresses:            IPv6/IPv4 Address

- Example of IPv10 Operation:
  ---------------------------

                R1 & R2 have both IPv4/IPv6 routing enabled
IPv10 Host                                                     IPv10 Host

   PC-1             R1             *            R2                PC-2
  +----+                         *   *                           +----+
  |    |             *         *       *         *               |    |
  |    |o---------o* X *o---o* IPv4/IPv6 *o---o* X *o-----------o|    |
  +----+  2001:1::1  *     *               *     *  192.168.1.1  +----+
 /    /                      *  Network  *                      /    /
+----+                         *       *                       +----+
                                 *   *
IPv6: 2001:1::10/64                *                IPv4: 192.168.1.10/24
DG  : 2001:1::1                                     DG  : 192.168.1.1

      |      128-bit    |            128-bit          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Data |    2001:1::10   | 000..0 ASN MAC 192.168.1.10 |--->
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       Src. IPv6 Address      Dest. IPv4 Address

                     IPv10: IPv6 host to IPv4 host

Khaled Omar             Internet-Draft                   [Page 5]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

     3.2) IPv10: IPv4 Host to IPv6 Host.
          ------------------------------

- IPv10 Packet:

         |              128-bit                |           128-bit           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Data| 000..0 ASN MAC  Source IPv4 Address |  Destination IPv6 Address   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

- Sending IPv10 host TCP/IP Configuration:

      IP Address:               IPv4 Address
      Subnet Mask:              /mask
      Default Gateway:          IPv4 Address
      DNS Addresses:            IPv4/IPv6 Address

- Example of IPv10 Operation:
  ---------------------------

              R1 & R2 have both IPv4/IPv6 routing enabled
IPv10 Host                                                    IPv10 Host

   PC-1             R1             *            R2                PC-2
  +----+                         *   *                           +----+
  |    |             *         *       *         *               |    |
  |    |o---------o* X *o---o* IPv4/IPv6 *o---o* X *o-----------o|    |
  +----+  2001:1::1  *     *               *     *  192.168.1.1  +----+
 /    /                      *  Network  *                      /    /
+----+                         *       *                       +----+
                                 *   *
IPv6: 2001:1::10/64                *               IPv4: 192.168.1.10/24
DG  : 2001:1::1                                    DG  : 192.168.1.1

                                   |       128-bit     |           128-bit           |
                                   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                               <---|     2001:1::10    | 000..0 ASN MAC 192.168.1.10 | Data|
                                   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                    Dest. IPv6 Address        Src. IPv4 Address

                    IPv10: IPv4 host to IPv6 host

Khaled Omar             Internet-Draft                   [Page 6]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

     3.3) IPv10: IPv6 Host to IPv6 Host.
          ------------------------------

- IPv10 Packet:

         |        128-bit        |          128-bit            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Data|  Source IPv6 Address  |  Destination IPv6 Address   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

- Sending IPv10 host TCP/IP Configuration:

      IP Address:               IPv6 Address
      Prefix Length:            /Length
      Default Gateway:          IPv6 Address (Optional)
      DNS Addresses:            IPv6/IPv4 Address

- Example of IPv10 Operation:
  ---------------------------

              R1 & R2 have both IPv4/IPv6 routing enabled
IPv10 Host                                                   IPv10 Host

   PC-1             R1             *            R2              PC-2
  +----+                         *   *                         +----+
  |    |             *         *       *         *             |    |
  |    |o---------o* X *o---o* IPv4/IPv6 *o---o* X *o---------o|    |
  +----+  2001:1::1  *     *               *     *  3001:1::1  +----+
 /    /                      *  Network  *                    /    /
+----+                         *       *                     +----+
                                 *   *
IPv6: 2001:1::10/64                *              IPv6: 3001:1::10/64
DG  : 2001:1::1                                   DG  : 3001:1::1

      |     128-bit     |      128-bit      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Data |   2001:1::10    |     3001:1::10    |--->
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       Src. IPv6 Address  Dest. IPv6 Address

                         IPv10: IPv6 host to IPv6 host

Khaled Omar             Internet-Draft                   [Page 7]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

     3.4) IPv10: IPv4 Host to IPv4 Host.
          ------------------------------

- IPv10 Packet:

         |               128-bit               |                 128-bit                 |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Data| 000..0 ASN MAC Source IPv4 Address  | 000..0 ASN MAC Destination IPv4 Address |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

- Sending IPv10 host TCP/IP Configuration:

      IP Address:               IPv4 Address
      Subnet Mask:              /Mask
      Default Gateway:          IPv4 Address
      DNS Addresses:            IPv6/IPv4 Address

- Example of IPv10 Operation:
  ---------------------------

              R1 & R2 have both IPv4/IPv6 routing enabled
 IPv10 Host                                                   IPv10 Host

    PC-1            R1             *            R2                PC-2
   +----+                        *   *                           +----+
   |    |            *         *       *         *               |    |
   |    |o--------o* X *o---o* IPv4/IPv6 *o---o* X *o-----------o|    |
   +----+  10.1.1.1  *     *               *     *  192.168.1.1  +----+
  /    /                     *  Network  *                      /    /
 +----+                        *       *                       +----+
                                 *   *
IPv4: 10.1.1.10/24                 *              IPv6: 192.168.1.10/24
DG  : 10.1.1.1                                    DG  : 192.168.1.1

      |          128-bit          |            128-bit          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Data | 000..0 ASN MAC 10.1.1.10  | 000..0 ASN MAC 192.168.1.10 |--->
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           Src. IPv4 Address             Dest. IPv4 Address

                     IPv10: IPv4 host to IPv4 host

Important Notes: - IPv4 and IPv6 routing must be enabled on all routers, so
                   when a router receives an IPv10 packet, it should use
                   the appropriate routing table based on the destination
                   address within the IPv10 packet.

                 - That means, if the received IPv10 packet contains an IPv4
                   address in the destination address field, the router
                   should use the IPv4 routing table to make a routing
                   decision, and if the received IPv10 packet contains an IPv6
                   address in the destination address field, the router should
                   use the IPv6 routing table to make a routing decision.

                 - All Internet connected hosts must be IPv10 hosts to be
                   able to communicate regardless the used IP version,
                   and the IPv10 deployment process can be accomplished
                   by ALL technology companies developing OSs for hosts
                   networking and security devices.

Khaled Omar             Internet-Draft                   [Page 8]
```

* * *

```
 RFC                   IPv10 Specification            September 2, 2017

[4](#section-4).  IPv10 Packet Header Format.

- The following figure shows the IPv10 packet header which is almost
  the same as the IPv6 packet header:

   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Version              4-bit Internet Protocol version number.

                        - 0100 : IPv4 Packet
                                 (Src. and dest. are IPv4).
                        - 0110 : IPv6 Packet
                                 (Src. and dest. are IPv6).
                        - 1010 : IPv10 Packet
                                 (Src. and dest. are IPv4/IPv6).

   Traffic Class        8-bit traffic class field.

   Flow Label           20-bit flow label.

   Payload Length       16-bit unsigned integer.  Length of the payload,
                        i.e., the rest of the packet following
                        this IP header, in octets.  (Note that any
                        extension headers [[section 4](#section-4)] present are
                        considered part of the payload, i.e., included
                        in the length count.)

   Next Header          8-bit selector.  Identifies the type of header
                        immediately following the IP header.

   Hop Limit            8-bit unsigned integer.  Decremented by 1 by
                        each node that forwards the packet. The packet
                        is discarded if Hop Limit is decremented to
                        zero.

   Source Address       128-bit address of the originator of the packet.

                                    |     32-bit    |  16-bit |  48-bit |    32-bit     |
    +-+-+-+-+-+-+-+-+-+-+-+         +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   IPv6 Address      |   OR    | 00000......0  |   ASN   |   MAC   | IPv4 Address  |
    +-+-+-+-+-+-+-+-+-+-+-+         +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      128-bit        |         |                      128-bit                      |

   Destination Address  128-bit address of the intended recipient of the
                        packet (possibly not the ultimate recipient, if
                        a Routing header is present).

                                    |     32-bit    |  16-bit |  48-bit |    32-bit     |
    +-+-+-+-+-+-+-+-+-+-+-+         +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   IPv6 Address      |   OR    | 00000......0  |   ASN   |   MAC   | IPv4 Address  |
    +-+-+-+-+-+-+-+-+-+-+-+         +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      128-bit        |         |                      128-bit                      |

[5](#section-5). Advantages of IPv10.

    1) Introduces an efficient way of communication between IPv6 hosts
       and IPv4 hosts.

    2) Allows IPv4 only hosts to exist and communicate with IPv6 only
       hosts even after the depletion of the IPv4 address space.

    3) Adds flexibility when making a query sent to the DNS for
       hostname resolution as IPv4 and IPv6 hosts can communicate with
       IPv4 or IPv6 DNS servers and the DNS can reply with any record
       it has (either an IPv6 record Host AAAA record or an IPv4
       record Host A record).

    4) There is no need to think about migration as both IPv4 and IPv6
       hosts can coexist and communicate to each other which will
       allow the usage of the address space of both IPv4 and IPv6
       making the available number of connected hosts be bigger.

    5) IPv10 support on "all" Internet connected hosts can be deployed
       in a very short time by technology companies developing OSs
       (for hosts and networking devices, and there will be no
       dependence on enterprise users and it is just a software
       development process in the NIC cards of all hosts to allow
       encapsulating both IPv4 and IPv6 in the same IP packet header.

    6) Offers the four types of communication between hosts:

          - IPv6 hosts to IPv4 hosts (6 to 4).

          - IPv4 hosts to IPv6 hosts (4 to 6).

          - IPv6 hosts to IPv6 hosts (6 to 6).

          - IPv4 hosts to IPv4 hosts (4 to 4).

Khaled Omar             Internet-Draft                   [Page 9]
```

* * *

```
RFC                   IPv10 Specification            September 2, 2017
Expires: 2-3-2018

Security Considerations

   The security features of IPv10 are described in the Security
   Architecture for the Internet Protocol [RFC-2401].

Acknowledgments

   The author would like to thank S. Krishnan, W. Haddad, C. Huitema,
   T. Manderson, JC. Zuniga, A. Sullivan, , K. Thomann, M. Abrahamsson,
   S. Bortzmeyer, J. Linkova, T. Herbert and Lee H. for the useful inputs and
   discussions about IPv10.

Author Address

   Khaled Omar Ibrahim Omar
   The Road
   6th of October City,
   Giza, Egypt
   Passport ID no.: A19954283

   Phone: +2 01003620284
   E-mail: eng.khaled.omar@hotmail.com

References

   [RFC-2401]   Stephen E. Deering and Robert M. Hinden, "IPv6
                Specification", RFC 2460, December 1998.

IANA Considerations

   IANA must reserve version number 10 for the 4-bit Version Field
   in the Layer 3 packet header for the IPv10 packet.

Full Copyright Statement

   Copyright (C) IETF (2017).  All Rights Reserved.

   This document and translations of it may be copied and furnished to
   others, and derivative works that comment on or otherwise explain it
   or assist in its implementation may be prepared, copied, published
   and distributed, in whole or in part, without restriction of any
   kind, provided that the above copyright notice and this paragraph are
   included on all such copies and derivative works.  However, this
   document itself may not be modified in any way, such as by removing
   the copyright notice or references, except as needed for the purpose of
   developing Internet standards in which case the procedures for
   copyrights defined in the Internet Standards process must be
   followed, or as required to translate it into languages other than
   English.

   The limited permissions granted above are perpetual and will not be
   revoked.

   This document and the information contained herein is provided on
   THE INTERNET ENGINEERING TASK FORCE DISCLAIMS ALL WARRANTIES,
   EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTY THAT
   THE USE OF THE INFORMATION HEREIN WILL NOT INFRINGE ANY RIGHTS OR
   ANY IMPLIED WARRANTIES OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR
   PURPOSE.

```
