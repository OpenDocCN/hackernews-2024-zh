<!--yml
category: 未分类
date: 2024-05-27 14:47:29
-->

# eBPF Documentary

> 来源：[https://www.brendangregg.com/blog//2024-03-10/ebpf-documentary.html](https://www.brendangregg.com/blog//2024-03-10/ebpf-documentary.html)

eBPF is a crazy technology – like putting JavaScript into the Linux kernel – and getting it accepted had so far been an untold story of strategy and ingenuity. The eBPF documentary, published late last year, tells this story by interviewing key players from 2014 including myself, and touches on new developments including Windows. (If you are new to eBPF, it is the name of a kernel execution engine that runs a variety of new programs in a performant and safe sandbox in the kernel, like how JavaScript can run programs safely in a browser sandbox; it is also no longer an acronym.) The documentary was played at KubeCon and is on [youtube](https://www.youtube.com/watch?v=Wb_vD3XZYOA):

[https://www.youtube.com/embed/Wb_vD3XZYOA?si=-i0HID5ek4hPcEOD](https://www.youtube.com/embed/Wb_vD3XZYOA?si=-i0HID5ek4hPcEOD)

VIDEO

Watching this brings me right back to 2014, to see the faces and hear their voices discussing the problems we were trying to fix. Thanks to Speakeasy Productions for doing such a great job with this [documentary](https://ebpfdocumentary.com/), and letting you experience what it was like in those early days. This is also a great example of all the work that goes on behind the scenes to get code merged in a large codebase like Linux.

When Alexei Starovoitov visited Netflix in 2014 to discuss eBPF with myself and Amer Ather, we were so entranced that we lost track of time and were eventually kicked out of the meeting room as another meeting was starting. It was then I realized that we had missed lunch! Alexei sounded so confident that I was convinced that eBPF was the future, but a couple of times he added "if the patches get merged." *If* they get merged?? They *have* to get merged, this idea is too good to waste.

While only several of us worked on eBPF in 2014, more joined in 2015 and later, and there are now hundreds contributing to make it what it is. A longer documentary could also interview Brendan Blanco (bcc), Yonghong Song (bcc), Sasha Goldshtein (bcc), Alastair Robertson (bpftrace), Tobais Waldekranz (ply), Andrii Nakryiko, Joe Stringer, Jakub Kicinski, Martin KaFai Lau, John Fastabend, Quentin Monnet, Jesper Dangaard Brouer, Andrey Ignatov, Stanislav Fomichev, Teng Qin, Paul Chaignon, Vicent Marti, Dan Xu, Bas Smit, Viktor Malik, Mary Marchini, and many more. Thanks to everyone for all the work.

Ten years later it still feels like it's early days for eBPF, and a great time to get involved: It's likely already available in your production kernels, and there are tools, libraries, and documentation to help you get started.

I hope you enjoy the documentary. PS. Congrats to Isovalent, the role-model eBPF startup, as Cisco recently announced they would acquire them!