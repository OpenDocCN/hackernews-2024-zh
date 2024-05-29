<!--yml
category: 未分类
date: 2024-05-27 14:31:46
-->

# Ampere in the Wild: How FreeBSD Employs Ampere Arm64 Servers in the Data Center

> 来源：[https://amperecomputing.com/blogs/ampere-in-the-wild](https://amperecomputing.com/blogs/ampere-in-the-wild)

The FreeBSD Foundation and project rely on generous donations of time, equipment, and money to keep FreeBSD at the forefront of computing. In late 2020, [Ampere](https://amperecomputing.com/company/about-us) donated five servers to the FreeBSD community and the FreeBSD Foundation purchased for the community another two in order to support development of FreeBSD.

The servers are located in 365 Data Centers’ Bridgewater, New Jersey facility, and are compatible with the Arm64 instruction set and optimized for users of large cloud platforms. Ampere is a cloud native processor supplier for high-performance and native cloud computing whose chips are widely known for their density, energy efficiency, and performance, making for an ideal configuration for FreeBSD which is also known for many of the same attributes. These combined Ampere systems provide value to FreeBSD in four key areas:

**Package Build Machines**

Three of the Ampere systems in the Bridgewater 365 data center operate as package build machines for FreeBSD. They provide all the Arm64 production packages that are available in FreeBSD 13, 14 and 15, and build two varieties of packages: latest and quarterly:

The latest packages are built on an ongoing basis, or “rolling release”. Essentially, this means that whenever there is a change to a port - a package blueprint - a new package is built. Latest packages are for users who prefer or require the latest software without delay.

Quarterly packages are built from a snapshot of the ports tree taken at the beginning of each quarter. These packages are for users who prefer to only update their packages once every few months. Security patches and updates are backported from latest package versions to quarterly packages. This means that the security fix is taken from the most recent version of upstream software and then applied to an older version, in this case the quarterly updated version.

**Security builds and tests**

The fourth Ampere system in the Bridgewater data center is dedicated to the security team's use, specifically for building and testing security updates for all supported FreeBSD branches on Arm64.

FreeBSD takes security seriously. Developers work continuously to harden the operating system. This, in turn, helps secure the work of other developers working on other projects. It’s no secret that layered protections are required to protect against modern security threats. The FreeBSD security team works constantly to ensure that the operating system level is as tight a defense as possible. To make that happen, the FreeBSD security strategy focuses on three main areas and groups:

1.  **Product Security Incident Response Team (PSIRT)**. PSIRT fields reports on security issues. Its end goal is to respond by identifying the problem and providing a fix for discovered and reported security flaws.
2.  **Proactive Security Work**. As opposed to incident response, proactive security work includes active searches to identify security vulnerability mitigations and a general architectural security review. This team is a combination of security team members and domain specific experts who also cover the random number generator and general design time security in FreeBSD.
3.  **Hardening the security of the FreeBSD infrastructure**, including the FreeBSD website and source code repository and all of the services that FreeBSD runs. There are several groups that work on this aspect of security and consist of more than just security team members.

**Download Servers**

One of the best-known advantages of FreeBSD is its unequaled reliability and performance as an Internet server. The fifth Ampere machine in the Bridgewater data center is running, ftp0.freebsd.org, the primary distribution node for all image artifacts that FreeBSD produces. A modern ftp server has long since outgrown its moniker. To that end, most transfers from ftp.freebsd.org/ftp0.freebsd.org are over HTTP or HTTPS protocol. The purpose of these servers remains the same, however, files are transferred back and forth between servers and through the cloud. By simplifying and speeding distribution in this way, users can quickly and easily download and upload the files and images they need.

**Running Reference Jails**

FreeBSD provides reference platforms to the developer community, to support porting and testing applications on architectures supported by FreeBSD. The sixth Ampere machine, soon to be joined by the seventh, in the Bridgewater data center provides reference jails for several versions of FreeBSD. FreeBSD port maintainers use these Arm64 servers for testing updates to their ports on Arm64.

**Future Development**

These servers also support development work to refine bhyve on Arm64 and high core count scalability. This will be particularly useful to Cloud Service Providers and hosting companies who want to provide multiple instance sizes to their customers, and to enterprise users who want to optimize hardware utilization by ensuring that workloads are given the resources they need and no more. In summary, the combination of Ampere native cloud processors on the latest server hardware boosts FreeBSD features and uses, while also lending powerful potential and support to the FreeBSD community.