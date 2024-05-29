<!--yml
category: 未分类
date: 2024-05-27 14:51:16
-->

# Is Delphi A Memory Safe Language?

> 来源：[https://blogs.embarcadero.com/is-delphi-a-memory-safe-language/](https://blogs.embarcadero.com/is-delphi-a-memory-safe-language/)

Last week, the US government released the report “[Back To The Building Blocks: A Path Toward Secure And Measurable Software](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf)”. This report is part of the US Cybersecurity Strategy and focused on multiple areas, including memory safety vulnerabilities and quality metrics. 

This report has been commented on by many online magazines, which have underlined the significant push back against the C and C++ programming languages. 

Some of these articles include:

## What does the US Government report say about memory safe languages?

The report focuses a lot on “memory safety vulnerabilities” and singles out “programming languages that both lack traits associated with memory safety and also have high proliferation across critical systems”. It continues recommending to “use memory safe programming languages at the outset, as recommended by the Cybersecurity and Infrastructure Security Agency’s (CISA) Open-Source Software Security Roadmap.

The reference is to the report [NSA Cybersecurity Information Sheet on Software Memory Safety](https://media.defense.gov/2023/Apr/27/2003210083/-1/-1/0/CSI_SOFTWARE_MEMORY_SAFETY_V1.1.PDF). This document goes into deeper details in terms of explaining what memory safety is, inducing it as a mix of type safety, safe allocation and deallocation (possibly with a garbage collector). This paragraph has the core content:

*Using a memory safe language can help prevent programmers from introducing certain **types of memory-related issues. Memory is managed automatically as part of the **computer language; it does not rely on the programmer adding code to implement **memory protections. The language institutes automatic protections using a combination **of compile time and runtime checks. These inherent language features protect the **programmer from introducing memory management mistakes unintentionally. Examples **of memory safe language include Python®, Java®, C#, Go, Delphi/Object Pascal, Swift®, **Ruby™, Rust®, and Ada. Even with a memory safe language, memory management is not entirely memory safe. *

## Does the NSA list Delphi as a memory safe language?

Yes, Delphi is listed as a memory safe language. Some of the articles initially reported a shorter list of safe languages, including only some of the most popular ones and excluding Delphi. The list has later been rectified matching the NSA report.

There has been some discussion in the Delphi community in terms of considering the language safe or not, considering it lacks one of the traits of memory safety, garbage collection. However, the official assessment was done on multiple grounds and taking into consideration other elements:

*   One core feature of safe programming languages is having a strong type system and having the language verify the data types mapping at compile time, and not at runtime. Dynamic languages, even with a garbage collector, can fall short and be exposed to runtime errors, which can impact their safety
*   The other element is not not having to resort to the use of pointers and more direct memory management in general code. While Delphi doesn’t block the use of direct memory access, this is fairly uncommon. Delphi offers mechanisms that automate and simplify memory management even without a GC.

## Does using a memory safe language mean I am completely protected from security risks?

The other general consideration is that memory safety is presented as a goal but not an absolute. For example, the main report underlines that there are types of applications in which predictability of the execution timing is crucial (referring to the aerospace industry). These are scenarios in which a garbage collector kicking in at unpredictable times can affect the program execution of code with critical timing. Delphi has a significant advantage compared to other popular languages in the industrial automation space precisely for this reason. It allows direct control, while remaining at a higher and simpler level compared to C++.

## The NSA and US Government lists Delphi as a memory safe language – what else does it recommend?

While the report recommends the shift towards memory safe languages, it also underlines the use of formal methods for static code analysis, in particular focused on security. We have been seeing a growing interest in this area also for Delphi, and we have been [promoting third party tools](https://blogs.embarcadero.com/how-secure-is-your-app-static-analysis-finds-security-holes/) that help in this space.

There is also a long section focused on hardware-based or CPU level enforcement of memory security. In the same area, the original NSA report underlines the importance of leveraging features like Control Flow Guard (CFG), Address Space Layout Randomization (ASLR) and Data Execution Prevention (DEP). Some of these security settings have been enforced in [recent versions of Delphi](https://blogs.embarcadero.com/rad-studio-11-1-and-windows-pe-security-flags/) and are now the default for new projects.

## What is supply chain security?

Finally, there is another long area covering the security of the library dependency chain, also sometimes referred to as “supply chain security”, advocating the use of formal methods to assess libraries safety, including that of open source libraries. There is a growing concern that the high number of dependencies of many projects is expanding the security risk not due to the project code, but the libraries used. In this respect the report has a detailed description of the Log4Shell vulnerability in the Log4j Java library. This library vulnerability affected a memory safe language like Java and showed, in the report words, “a critical weakness through which malicious actors could compromise computer systems across the world”. The report continues indicating that “this vulnerability highlighted the critical need to help ensure the security of the open-source ecosystem, which fosters tremendous innovation worldwide.”

## What is Embarcadero doing to help with security risks?

It is clear that there is a growing concern in terms of software security at all levels, from government bodies to businesses of all sizes. Even before the release of this White House and NSA report where Delphi was listed as a memory safe language, Embarcadero had already identified security as a growing concern among customers. We continue to focus on investing in and improving Delphi support for modern security techniques and backing that with clear hype-free education on the genuine risks and available mitigations.