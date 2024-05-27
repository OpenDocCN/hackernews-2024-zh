<!--yml
category: 未分类
date: 2024-05-27 13:07:04
-->

# Our Response to Hashicorp's Cease and Desist Letter | OpenTofu

> 来源：[https://opentofu.org/blog/our-response-to-hashicorps-cease-and-desist/](https://opentofu.org/blog/our-response-to-hashicorps-cease-and-desist/)

On April 3rd, we received a Cease and Desist letter from HashiCorp regarding our implementation of the "removed" block in OpenTofu, claiming copyright infringement on the part of one of our core developers. We were also made aware of an article posted that same day with the same accusations. We have investigated these claims and are publishing the C&D letter, our response and the source code origin document resulting from our investigation.

**The OpenTofu team vehemently disagrees with any suggestion that it misappropriated, mis-sourced, or otherwise misused HashiCorp’s BSL code. All such statements have zero basis in facts.**

HashiCorp has made claims of copyright infringement in a cease & desist letter. These claims are **completely unsubstantiated**.

The code in question can be clearly shown to have been copied from older code under the MPL-2.0 license. HashiCorp seems to have copied the same code itself when they implemented their version of this feature. All of this is easily visible in our detailed SCO analysis, as well as their own comments which indicate this.

*To prevent further harassment of individual people, we have redacted any personal information from these documents.*

Despite these events, we have managed to carry out significant development on OpenTofu 1.7, including state encryption, “for_each” implementation for “import” blocks, as well as the all-new **provider-defined functions** supported by the recently released provider plugin protocol.

On that note, we will be releasing a new pre-release version next week, and we are eager to gather feedback from the community.

— The OpenTofu Team

* * *

 *The image in this blog post contains code licensed under the BUSL-1.1 by HashiCorp. However, for the purposes of this post we are making non-commercial, transformative fair use under [17 U.S. Code § 107](https://www.govinfo.gov/content/pkg/USCODE-2022-title17/html/USCODE-2022-title17-chap1-sec107.htm).
You can read more about fair use on the [website of the US Copyright Office](https://www.copyright.gov/fair-use/).*