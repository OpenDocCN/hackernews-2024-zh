<!--yml
category: 未分类
date: 2024-05-27 14:25:13
-->

# Fuzzing the TCP/IP stack - 37C3

> 来源：[https://events.ccc.de/congress/2023/hub/en/event/fuzzing_the_tcp_ip_stack/](https://events.ccc.de/congress/2023/hub/en/event/fuzzing_the_tcp_ip_stack/)

Our exploration begins with an honest appraisal of traditional fuzzing methodologies that have been applied to TCP/IP stacks before, like ISIC, revealing their inherent limitations, e.g., they can't reach beyond the TCP initial state. Recognizing the need for a more evolved approach, we take a different approach, where we leverage a full-blow active network connection for fuzzing. A key revelation in this journey is the deliberate decision to sidestep the arduous task of constructing a custom TCP/IP stack, a choice rooted in practical considerations.

The reluctance to build a bespoke TCP/IP stack leads us to innovative strategies such as embedding hooks in the Linux kernel and tapping into userland TCP/IP stacks like PyTCP, Netstack (part of Google gVisor), and PicoTCP. PicoTCP takes center stage, offering a userland TCP/IP stack that becomes integral to our state fuzzing methodology. Attendees will gain a deeper understanding of its architecture, APIs, and documentation, appreciating its pivotal role in fortifying network security.

As the presentation unfolds, we navigate through the development of a powerful fuzzer, a core element in our approach to identifying vulnerabilities within the TCP/IP stack. The intricacies of driving traffic through the system, simulating real-world scenarios, and leveraging reproducibility and diagnostics techniques are revealed. The discussion expands to showcase tangible results, including trophies obtained, bugs reported, and the eventual release of the project on GitHub. The session concludes with an engaging Q & A, encouraging participants to delve into the intricacies of TCP/IP stack fuzzing and its profound implications for network security.