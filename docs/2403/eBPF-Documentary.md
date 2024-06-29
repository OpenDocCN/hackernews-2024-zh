<!--yml

category: 未分类

date: 2024-05-27 14:47:29

-->

# eBPF 纪录片

> 来源：[https://www.brendangregg.com/blog//2024-03-10/ebpf-documentary.html](https://www.brendangregg.com/blog//2024-03-10/ebpf-documentary.html)

eBPF 是一项非常厉害的技术 —— 就像将 JavaScript 放入 Linux 内核一样 —— 并且让它得到接受迄今为止是一段未被讲述的策略和智慧的故事。去年末发布的 eBPF 纪录片通过采访包括我在内的 2014 年的关键人物来讲述这个故事，并涉及包括 Windows 在内的新发展。（如果您对 eBPF 还不熟悉，它是一个在内核中以高性能和安全的沙箱中运行各种新程序的内核执行引擎的名称，就像 JavaScript 可以在浏览器沙箱中安全运行程序一样；它也不再是一个缩写。）这部纪录片在 KubeCon 上播放，并且在 [youtube](https://www.youtube.com/watch?v=Wb_vD3XZYOA) 上可以观看：

[https://www.youtube.com/embed/Wb_vD3XZYOA?si=-i0HID5ek4hPcEOD](https://www.youtube.com/embed/Wb_vD3XZYOA?si=-i0HID5ek4hPcEOD)

VIDEO

观看这部纪录片让我仿佛回到了 2014 年，看到了他们的面孔，听到他们讨论我们试图解决的问题的声音。感谢 Speakeasy Productions 做了一份如此出色的工作，制作了这部 [纪录片](https://ebpfdocumentary.com/)，让您体验那些早期时光。这也是在大型代码库如 Linux 中合并代码背后所有工作的一个很好的例子。

当 Alexei Starovoitov 于 2014 年访问 Netflix 与我和 Amer Ather 讨论 eBPF 时，我们被他的话语吸引得如此入迷，以至于我们忘记了时间，最终被迫离开会议室，因为另一场会议即将开始。那时我意识到我们已经错过了午餐！Alexei 的话听起来如此自信，以至于我深信 eBPF 是未来，但有几次他补充说“如果补丁被合并”。*如果*它们被合并？它们*必须*被合并，这个想法太好了，不容浪费。

尽管只有我们几个人在 2014 年致力于 eBPF，但在 2015 年及以后，更多人加入，现在已有数百人贡献力量，使其成为现在的模样。一部更长的纪录片也可以采访 Brendan Blanco (bcc), Yonghong Song (bcc), Sasha Goldshtein (bcc), Alastair Robertson (bpftrace), Tobais Waldekranz (ply), Andrii Nakryiko, Joe Stringer, Jakub Kicinski, Martin KaFai Lau, John Fastabend, Quentin Monnet, Jesper Dangaard Brouer, Andrey Ignatov, Stanislav Fomichev, Teng Qin, Paul Chaignon, Vicent Marti, Dan Xu, Bas Smit, Viktor Malik, Mary Marchini 等人。感谢每个人为此付出的努力。

十年过去了，eBPF 仍然感觉像是早期阶段，并且现在正是加入的好时机：它很可能已经在您的生产内核中可用，并且有工具、库和文档帮助您入门。

希望您喜欢这部纪录片。附：恭喜 Isovalent，这家模范 eBPF 创业公司，因为 Cisco 最近宣布他们将收购他们！
