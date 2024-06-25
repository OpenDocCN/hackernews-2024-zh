<!--yml

category: 未分类

日期：2024-05-27 14:36:23

-->

# Chris's Wiki :: blog/web/CGINotSlow

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/web/CGINotSlow](https://utcc.utoronto.ca/~cks/space/blog/web/CGINotSlow)

我最近阅读了 [回忆 CGI 脚本](https://rednafi.com/go/reminiscing_cgi_scripts/)（[via](https://lobste.rs/s/jcm2am/reminiscing_cgi_scripts)），它谈到了 [CGI 脚本](https://en.wikipedia.org/wiki/Common_Gateway_Interface)，并顺便提到了它们为什么不受青睐，嗯，让我引用一下：

> CGI 脚本主要因性能和安全相关问题而不受青睐。[...]

从某种意义上说，这是真的。在 CGIs 被 PHP 和其他更复杂的部署环境（如 Apache 的 [mod_perl](https://perl.apache.org/) 和 [mod_wsgi](https://modwsgi.readthedocs.io/)）推开的时代，它们的性能是一个问题，特别是在当时的重负载下。但这并不是因为 CGI 程序在绝对意义上速度慢；而是因为早期 00 年代的计算机性能不是很强大，甚至可能在虚拟托管环境中被过度共享。当充当网络服务器的计算机总体上无法做很多事情时，你能让它们做的事情越少，包括不为每个请求启动一个单独的程序或两个。

现代计算机比早期的 00 年代服务器快得多，更强大；即使是低端的 VPS 也可能更好，具有更多的内存、更多的 CPU，并且几乎总是更快的磁盘。毫不奇怪，CGI 的运行速度和处理负载能力在绝对意义上都有了很大的提高。

为了说明这一点，我编写了一个非常基本的 Python 和 Go CGI，在[我们通用网络服务器上的我的区域](https://www.cs.toronto.edu/~cks/)中进行测试，看看它们的运行速度有多快。在我们普通的 Ubuntu 网络服务器上，Python 版本运行大约需要 17 毫秒，而 Go 版本大约需要四毫秒（在这两种情况下，它们最近都运行过）。由于这两种情况下的 CGI 都几乎没做任何事情，所以这基本上是在衡量运行 CGI 的执行开销。一个真正的 Python CGI 启动时间会更长，因为它需要导入更多的东西，但即使如此，它也不一定很慢。

（作为另一个数据点，我对 [Wandering Thoughts](/~cks/space/blog/) 的响应时间有持续的数据，它是[一个相当复杂的 Python 通常作为 CGI 运行的程序](/~cks/space/blog/web/DynamicNeedNotBeSlow)。在相当基础的（虚拟）硬件上，它的平均响应时间似乎约为 0.17 秒（包括诸如 TLS 开销之类的内容），这已经明显降低了十年前的水平。）

给定[CGI 脚本对于规模较小的页面和站点具有吸引力](/~cks/space/blog/web/CGIAttractions)，了解 CGI 程序并不像在旧资料（或者从旧资料中工作的人）中经常被描绘得那么糟糕是很有用的。对于许多 Web 应用程序来说，使用 CGI 程序是一种完全可行的部署策略（而且[你可以利用其他通用 Web 服务器功能](/~cks/space/blog/web/ApacheCGIsAndLocationACLs)）。

（是的，如果你的 CGI 每秒接收一百次请求，可能会减慢。这种情况发生的可能性有多大，如果发生了，减慢有多重要？有一些环境是你绝对需要考虑并计划这种情况的，但也有相当多的环境是你不需要考虑这种情况的。）
