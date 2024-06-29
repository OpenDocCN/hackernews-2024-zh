<!--yml

category: 未分类

date: 2024-05-29 13:29:01

-->

# 又到了技术无法处理日期的时候 • The Register

> 来源：[https://www.theregister.com/2024/02/29/fuel_pump_leap_year_bug/](https://www.theregister.com/2024/02/29/fuel_pump_leap_year_bug/)

今天是2月29日，这是一个不寻常的日子，只有在年份是四的倍数时才会加到通常的28天中，以与天文年保持同步。

这个权宜之计防止了我们的季节错乱，但对计算机和软件而言却带来了问题，它们必须被编程以考虑这一额外的一天，以避免错误条件和不正确的数据。

我们都在使用各种计算机来阅读这篇文章，希望没有什么着火了，然而每隔四年一次的闰年，总会有某个地方出大问题。

在全球大多数国家尚未开始之时，新西兰的燃油泵支付系统在经历了超过十个小时的全国性停机后，刚刚重新站起来。

受影响的包括 Allied Petroleum、Gull、Z、Waitomo、BP 等多家石油供应商，这意味着顾客无法使用借记卡和信用卡支付燃油费用。

许多地点关闭了他们的燃油泵，但像 Waitomo 的应用程序这样的内部解决方案运行良好，这可能是因为它们专业编码以适应2月29日。

不像 Invenco 终端点销售软件包括燃油泵终端。

Gull 的发言人朱利安·莱斯表示，使用 Invenco 终端的公司遇到了闰年 bug。他告诉[新西兰先驱报](https://www.nzherald.co.nz/hawkes-bay-today/news/february-29-allied-fuel-pumps-around-nz-ground-to-a-halt-as-systems-forget-leap-year/XEQBK5JLBZG6LO3VGUQ6Q2WGC4/)说：“我们一直在与供应商联系，了解他们正在尽快修复此问题”，并补充说2月29日是“那种导致支付软件出现故障的事情之一”。

"Just one of those things"如果软件未为此事件校准，对我们来说非常暗示人为错误。这就是为什么开发人员使用存在几十年的经过验证的库，通常不敢触碰任何涉及日历和时间逻辑的内容，以免快速被逼疯。

奥克兰成立的 Invenco CEO 约翰·斯科特确认了闰年 bug，并表示已经在网络上推出了修复措施，使燃油泵支付得以恢复。

*The Register* 要求 Invenco 详细说明错误的性质及其修复方法。公司回应后将更新文章。

Invenco并非唯一一个因2月29日发生故障的软件公司。Fastrack FS1智能手表的所有者报告称，时钟在2月28日23:59时[停滞不动](https://twitter.com/Amol_chi/status/1763046970317230119)，或者在[之后不显示日期](https://twitter.com/Manuvktr/status/1763057415417901311)。据称YNAB（You Need A Budget）应用程序也无法[识别](https://www.reddit.com/r/ynab/comments/1b27jpo/funny_glitch_for_leap_year_repeating_scheduled/)2月29日的存在。与此同时，在日本，警察在神奈川、新潟、岡山和爱媛[无法发放或更新驾驶执照](https://japannews.yomiuri.co.jp/society/general-news/20240229-171789/)，同样也被认为是由于日期问题。

正如大多数*Reg*读者所知，跟踪时间在计算中是一个绝对的雷区。以Y2K问题为例，因为系统仅使用最后两位数字表示年份，导致2000年无法与1900年区分，全球曾预测会因此陷入混乱。

尽管一切“良好”，这主要归功于技术团队扩展日期字段或调整窗口的强烈努力，详细解释可见我们关于这个问题的[Retro Tech Week feature](https://www.theregister.com/2024/01/17/y2k_feature/)。

这并不意味着没有任何问题。例如，日本失去了监控核电站安全系统的能力，听起来远非最佳。

然后是[2038年问题](https://theyear2038problem.com/)，它影响着使用Unix时间的系统——即自1970年1月1日00:00:00 UTC（Unix纪元）以来经过的非闰秒数量。Unix开发者决定将其作为带符号的32位整数来跟踪，但这种数据类型只能编码到2038年1月19日03:14:07 UTC，因此得名。一秒后，整数将溢出，系统将其解释为1901年12月13日20:45:52 UTC。

尽管我们仍有这些问题需要面对，但回顾上一次2月29日的情况，世界似乎将要结束，因此一时支付燃料费用的困难是一个更加乐观的结果。®
