<!--yml
category: 未分类
date: 2024-05-27 14:50:28
-->

# Random Musings

> 来源：[https://daninus14.github.io/posts/Why-is-Common-Lisp-not-the-Most-Popular-Programming-Language.html](https://daninus14.github.io/posts/Why-is-Common-Lisp-not-the-Most-Popular-Programming-Language.html)

Tagged as [lisp](https://daninus14.github.io/tag/lisp.html), [common-lisp](https://daninus14.github.io/tag/common-lisp.html), [conference](https://daninus14.github.io/tag/conference.html)

Written on 2024-02-13 14:08:59

This question is always asked. My thoughts are a little bit different than what I've read online.

I think the real reason may be because of the fact that **there is** a standard. Yes, you read that correctly.

The great benefit of a standard is that no matter what implementation you use, if they follow the standard, and your code follows the standard, you should have portable code between implementations. Further, and probably the greatest benefit, is that we don't have to worry about backwards compatibility. Code from about 40 years ago (yeah, even before the standard!) still runs today (I don't have a link at hand, but it was one of those popular lisp posts that I'm sure you can easily find) by simply importing a package.

However, a standard comes with a cost. The cost is called [diffusion of responsibility](https://en.wikipedia.org/wiki/Diffusion_of_responsibility).

> The diffusion of responsibility refers to the decreased responsibility of action each member of a group feels when they are part of a group.

What we are lacking is a body or organization who will take on the responsibility of promoting common lisp.

Take a look at the [Python Foundation](https://www.python.org/psf-landing/) for example:

> The Python Software Foundation is an organization devoted to advancing open source technology related to the Python programming language.

And where do they get their [money from](https://www.python.org/psf/sponsorship/)?

> PyCon US typically generates over 65% of the PSF’s revenue. With PyCon US 2020 and 2021 happening virtually, the PSF may lose $1.2 million USD of expected revenue for those two years.

Even if there are different groups or individuals trying to promote CL, what we really need is one person who will lead a main CL only conference which is not just an [academic conference](https://www.european-lisp-symposium.org/). The purpose of the conference is both in itself to promote CL and serve as a place for CL companies, users, and implementors to get together, and *also* for fundraising for the organization.

Those funds can then be used to address the biggest issues plaguing CL, it seems from time immemorial, namely documentation and marketing. Marketing includes presenting the documentation in both an easy to understand and pleasant way, and also producing thorough and detailed references.

Further, the funds can be used to address anything lacking in the ecosystem for which there has not been a serious enough effort to provide a solution, and for which a solution could be provided with a given financial investment. I am sure there are *many* common lisp developers out there who would be thrilled to create common lisp projects that are robust and for the benefit of the community, if they had the financial support to be able to do it.

As an example of this, take a look at a new language: Julia. They have what looks like a [popular conference](https://juliacon.org/2024/), and a [bunch of sponsors](https://julialang.org/research/#sponsors). The Sloan Foundation by itself [gave almost 1 million USD in 2017](https://sloan.org/grant-detail/7999).

Another example is Racket, which is also covered in the academic conference mentioned above, and they even have two conferences! An [official one](https://con.racket-lang.org/), and [another](https://racketfest.com/) one from the users.

So in conclusion, we need an organization which will aggressively work to fundraise for Common Lisp in order to promote the usage of the language by providing solutions to ecosystem issues that no one is taking care of in a robust enough way.