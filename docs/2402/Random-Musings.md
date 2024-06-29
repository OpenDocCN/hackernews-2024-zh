<!--yml

category: 未分类

date: 2024-05-27 14:50:28

-->

# 随机的思考

> 来源：[https://daninus14.github.io/posts/Why-is-Common-Lisp-not-the-Most-Popular-Programming-Language.html](https://daninus14.github.io/posts/Why-is-Common-Lisp-not-the-Most-Popular-Programming-Language.html)

标记为[lisp](https://daninus14.github.io/tag/lisp.html), [common-lisp](https://daninus14.github.io/tag/common-lisp.html), [conference](https://daninus14.github.io/tag/conference.html)

Written on 2024-02-13 14:08:59

这个问题总是被问到。我的想法与我在网上读到的有些不同。

我认为真正的原因可能是因为**存在**一个标准。是的，你没看错。

标准的巨大好处在于，无论你使用什么实现，如果它们遵循标准，而且你的代码也遵循标准，你应该能够在不同的实现之间拥有可移植的代码。此外，可能是最大的好处是，我们不必担心向后兼容性。大约40年前的代码（是的，甚至是在标准之前！）今天仍在运行（我手头没有链接，但那是一篇流行的lisp文章，你肯定很容易找到）只需导入一个包即可。

然而，标准是有成本的。这个成本称为[责任扩散](https://en.wikipedia.org/wiki/Diffusion_of_responsibility)。

> 责任扩散是指每个群体成员感觉到的行动责任减少，当他们成为群体的一部分时。

我们缺少的是一个愿意承担推广通用Lisp责任的机构或组织。

看看[Python基金会](https://www.python.org/psf-landing/)作为例子：

> Python软件基金会是一个致力于推进与Python编程语言相关的开源技术的组织。

那么他们从哪里获得[资金](https://www.python.org/psf/sponsorship/)呢？

> PyCon US通常会为PSF贡献超过65%的收入。2020年和2021年的PyCon US以虚拟形式举行，PSF可能会因这两年失去预期收入的120万美元。

即使有不同的团体或个人试图推广CL，我们真正需要的是一个人来领导一个主要的CL专属会议，这不仅仅是一个[学术会议](https://www.european-lisp-symposium.org/)。会议的目的既是推广CL，也是一个CL公司、用户和实现者聚集在一起的场所，并且*也*用于为组织筹款。

这些资金可以用来解决困扰CL最大问题，似乎从古至今就存在的，即文档和营销。营销包括以易于理解和愉悦的方式呈现文档，以及制作详尽和详细的参考资料。

此外，这些资金可以用来解决生态系统中任何缺乏严肃解决方案的问题，并且通过适当的财政投入提供解决方案。我相信，如果他们有财政支持，会有*很多*Common Lisp开发者愿意创建对社区有益的健壮项目。

举个例子，看看一门新语言：Julia。他们似乎有一个[热门会议](https://juliacon.org/2024/)，以及一大群[赞助商](https://julialang.org/research/#sponsors)。仅斯隆基金会在2017年[捐助了近100万美元](https://sloan.org/grant-detail/7999)。

另一个例子是 Racket，它也在上述学术会议中涵盖，并且他们甚至有两个会议！一个是[官方会议](https://con.racket-lang.org/)，还有一个来自用户的[另一个](https://racketfest.com/)。

因此，总结一下，我们需要一个组织，积极筹集资金来推动 Common Lisp 的使用，通过提供解决生态系统问题的方案，这些问题目前没有人以足够健壮的方式解决。
