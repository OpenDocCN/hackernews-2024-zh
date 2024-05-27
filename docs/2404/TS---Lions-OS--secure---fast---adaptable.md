<!--yml
category: 未分类
date: 2024-05-27 13:19:11
-->

# TS | Lions OS: secure – fast – adaptable

> 来源：[https://trustworthy.systems/publications/papers/Heiser_24:eo.abstract](https://trustworthy.systems/publications/papers/Heiser_24:eo.abstract)

<main id="page-content">

# Lions OS: secure – fast – adaptable

## Authors

[Gernot Heiser](/people/?cn=Gernot+Heiser)

    School of Computer Science and Engineering
    UNSW,
    Sydney 2052, Australia

## Abstract

seL4 is the world's most secure operating system (OS) kernel, but it is a far cry from being an OS. As a microkernel, it provides core mechanisms for securely multiplexing the hardware, but none of the services application programmers expect, so adopters are forced to develop those themselves. Furthermore, a high level of expertise is required to design performant systems on top of seL4\. The result is frequently a poor design, and far too often people giving up.

The Trustworthy Systems team at UNSW has now decided to build Lions OS, a complete seL4-based OS aimed to support the needs of developers of cyberphysical, IoT and other embedded systems. Lions OS, named after John Lions (of Lions Book fame, arguably one of the fathers of open source), is being designed and implemented from scratch, with the seemingly conflicting goals of high performance, high security and adaptability to a wide class of use cases. Specifically we plan to prove the correct implementation of its critical components.

In the talk I explain why I think this is not only achievable, but will be achieved within 2-3 years, by strictly adhering to the time-honoured engineering principle KISS – keep it simple, stupid! I will present our initial results that show that, with the right design, this simplicity can not only achieve excellent performance (outperforming Linux networking) but also enable proofs of implementation correctness. The talk serves as the public launch of Lions OS -- needless to say, it's open source.

## BibTeX Entry

```
  @inproceedings{Heiser_24:eo,
    address          = {Gladstone, QLD, AU},
    author           = {Gernot Heiser},
    booktitle        = {Everything Open},
    month            = apr,
    organization     = {Linux Australia},
    title            = {{Lions OS}: Secure -- Fast -- Adaptable},
    year             = {2024}
  }
```

## Download

</main>