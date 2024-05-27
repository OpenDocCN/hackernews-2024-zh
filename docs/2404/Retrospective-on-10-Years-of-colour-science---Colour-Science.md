<!--yml
category: 未分类
date: 2024-05-27 13:02:37
-->

# Retrospective on 10 Years of colour-science | Colour Science

> 来源：[https://www.colour-science.org/posts/retrospective-on-10-years-of-colour-science/](https://www.colour-science.org/posts/retrospective-on-10-years-of-colour-science/)

Today is the 10th anniversary of [Colour](https://github.com/colour-science/colour) and [colour-science](https://github.com/colour-science)!

```
(colour-science-py3.12) Eris:colour kelsolaar$ git log --reverse
commit 90bc42b9fedfc7291c7023247eab14b41d5c29af
Author: Thomas Mansencal <thomas.mansencal@gmail.com>
Date:   Sat Apr 5 14:07:48 2014 +0200

    Initial commit.

    Signed-off-by: Thomas Mansencal <thomas.mansencal@gmail.com>

```

It is time for a decade of work retrospective.

We would like to start by thanking all the contributors whether they write code, improve the documentation, create issues or start discussions.

Over the past 10 years, we built a strong community of people loving colour science. There have been many passionate discussions about this complex scientific domain. To this day, we still do not have full understanding of how human vision works, so we are excited about the future.

[Colour](https://github.com/colour-science/colour) has become one of the most comprehensive open source colour science library, irrespective of the programming language.

[colour-science](https://github.com/colour-science) now has almost 50 repositories, the main ones are as follows:

To update what we [wrote 4 years ago](../our-first-1000-stars-on-github/index.html), here are the highlights at this date:

What is the future of **Colour**? We thought we could merge the GPU backend four years ago and ultimately decided against. The main issue is that it was very much vendor locked because of [Cupy](https://cupy.dev). In a hindsight, this was a good decision as there has been some exciting new development with the [Python Array API Standard](https://data-apis.org/array-api/latest/index.html). We haven't started work yet to support it, but we have certainly [discussed about it](https://github.com/colour-science/colour/issues/1244).

We also thought that 1.0.0 would be released, but this did not happened! We instead continued to improve the API and a stable release will only be possible after adoption of the Python Array API standard.

With all the written, we are hoping to continue for at least another decade. Feel free to join us on [Github Discussions](https://github.com/colour-science/colour/discussions), contact us on [Gitter](https://gitter.im/colour-science/colour) or by [email](mailto:colour-developers@colour-science.org).

**The Colour Developers**