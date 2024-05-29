<!--yml
category: 未分类
date: 2024-05-27 14:48:53
-->

# Escaping from isolated networks using Broadcast DNS | by lvj | SensorFu | Medium

> 来源：[https://medium.com/sensorfu/escaping-isolated-networks-using-broadcast-dns-5aee866bcaff](https://medium.com/sensorfu/escaping-isolated-networks-using-broadcast-dns-5aee866bcaff)

# Escaping from isolated networks using Broadcast DNS

One of our latest escape methods is the capability send Domain Name System (DNS) queries via a broadcast ethernet packet. We call this the Broadcast DNS escape. Our hypothesis was that DNS server or cache could pick those up from the network and happily redirect it to another network.

And our hypothesis has been proven right. We present you two stories of Broadcast DNS finding real issues in real networks.

# Active Directory, Hidden Server

Our client updated Beacons on their site to the latest version which added our new Broadcast DNS test. After the upgrade we were alerted that the Beacon had leaked from the site using this new escape.

Beacon was deployed in a production network — a network to monitor and control industrial processes — and the DNS resolver was properly segmented in the DMZ area. The DNS servers on their site were on a different IP subnet than the Beacon. Beacon had not leaked with unicast DNS before as the firewalls were blocking the traffic between the Beacon and the DNS servers. So how could the DNS leak out via broadcast?

One possibility was that the isolated network containing the Beacon and the network containing the DNS resolvers were accidentally connected together. This might happen due to incorrect cabling or VLAN settings. Networks connected together at ethernet level could forward ethernet frames within those unintentionally connected networks allowing the Beacon to reach the local DNS resolver via broadcast. This means that the networks are not isolated and the segmentation has failed.

Other possibility is that the network has an unknown DNS resolver which accepts the broadcasted DNS query and is allowed to forward it towards the upstream resolver.

After investigating the finding it was discovered that the network actually had an old and supposedly decommissioned Microsoft Active Directory (AD) server still powered on in the network. That AD server also acted as a DNS server which happily processed the broadcasted DNS packets and forwarded them to the upstream DNS infrastructure. Firewall also had a rule to allow DNS traffic out from that particular DNS server.

DNS queries from the unknown AD server have an exception in the firewall

# Interconnected Networks

Another Broadcast DNS escape case soon followed. The company has been using Beacon for over a year now in a production site, deployed in production and IT networks. After upgrading the Beacons to the latest version an unexpected broadcast DNS escape was observed from the Beacon deployed in the isolated production network. In this case the logical hypotheses were the same; unknown DNS resolver in the production network or that the switched network is somehow connected to another network which has a DNS resolver processing and resolving the packet.

After notifying the customer, they quickly investigated the issue and found indications that there were two networks accidentally connected together. Two networks being connected together, the broadcast packets sent by the Beacon in the isolated production network reached the IT network containing the DNS resolver which then forwarded the query to the upstream DNS resolvers all the way to the Beacon Home. The switch ports connecting the networks were identified and the networks disconnected after which the Beacon stopped calling Home.

Two layer 2 networks accidentally interconnected allowing broadcasted DNS queries to escape the isolated network

# Afterthought

This kind of broadcast packets (be it DNS, ICMP or something else) take advantage of a weakness in TCP/IP network stacks. The ethernet layer takes the broadcast packet and allows it to flow up in the stack to the next network layer. Then the next layer inspects their part of the packet and acts accordingly, completely missing the fact that it was a broadcast packet and maybe not intended for them to process.