<!--yml

category: 未分类

日期：2024-05-29 12:48:48

-->

# #1068024 - 恢复到不包含恶意更改的版本 - Debian Bug 报告日志

> 来源：[https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1068024](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1068024)

# Debian Bug 报告日志 - [#1068024](mailto:1068024@bugs.debian.org)

恢复到不包含恶意更改的版本

[回复](mailto:1068024@bugs.debian.org) 或 [订阅](mailto:1068024-subscribe@bugs.debian.org) 此 Bug。

切换无用消息

* * *

**报告已转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`。 (Fri, 29 Mar 2024 20:36:04 GMT) ([全文](bugreport.cgi?bug=1068024;msg=2), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=2), [链接](#1)).

* * *

**已发送确认**至 `Joey Hess <id@joeyh.name>`：

收到并转发新的 Bug 报告。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Fri, 29 Mar 2024 20:36:04 GMT) ([全文](bugreport.cgi?bug=1068024;msg=4), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=4), [链接](#3)).

* * *

[第五条信息](#5) 收到于 submit@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=5), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=5), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E&In-Reply-To=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E&body=On%20Fri%2C%2029%20Mar%202024%2016%3A25%3A26%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20Package%3A%20xz-utils%0A%3E%20Version%3A%205.6.1%2Breally5.4.5-1%0A%3E%20Severity%3A%20important%0A%3E%20Tags%3A%20security%0A%3E%20%0A%3E%20I%20count%20a%20minimum%20of%20750%20commits%20or%20contributions%20to%20xz%20by%20Jia%20Tan%2C%20who%0A%3E%20backdoored%20it.%0A%3E%20%0A%3E%20This%20includes%20all%20700%20commits%20made%20after%20they%20merged%20a%20pull%20request%20in%20Jan%207%0A%3E%202023%2C%20at%20which%20point%20they%20appear%20to%20have%20already%20had%20direct%20push%20access%2C%20which%0A%3E%20would%20have%20also%20let%20them%20push%20commits%20with%20forged%20authors.%20Probably%20a%20number%20of%0A%3E%20other%20commits%20before%20that%20point%20as%20well.%0A%3E%20%0A%3E%20Reverting%20the%20backdoored%20version%20to%20a%20previous%20version%20is%20not%20sufficient%20to%0A%3E%20know%20that%20Jia%20Tan%20has%20not%20hidden%20other%20backdoors%20in%20it.%20Version%205.4.5%20still%0A%3E%20contains%20the%20majority%20of%20those%20commits.%0A%3E%20%0A%3E%20Commits%20by%20them%20such%20as%2018d7facd3802b55c287581405c4d49c98708c136%20%0A%3E%20and%20ae5c07b22a6b3766b84f409f1b6b5c100469068a%20show%20that%20they%20were%20deep%0A%3E%20into%20analyzing%20the%20security%20of%20xz.%20They%20were%20well%20placed%20to%20insert%20a%20buffer%0A%3E%20overflow%20that%20could%20allow%20eg%2C%20a%20targeted%20xz%20file%20to%20cause%20arbitrary%20code%0A%3E%20execution.%20The%20impact%20of%20such%20a%20security%20hole%20could%20be%20much%20more%20stealthy%20and%0A%3E%20bad%20than%20the%20known%20backdoor%20since%20it%20would%20allow%20chaining%20xz%20with%20other%0A%3E%20unrelated%20software%20releases%20on%20an%20ongoing%20basis.%0A%3E%20%0A%3E%20The%20package%20should%20be%20reverted%20to%20a%20version%20before%20their%20involvement%2C%0A%3E%20which%20started%20with%20commit%206468f7e41a8e9c611e4ba8d34e2175c5dacdbeb4.%0A%3E%20Or%20their%20early%20commits%20vetted%20and%20revert%20to%20a%20later%20point%2C%20but%20any%20arbitrary%20%0A%3E%20commit%20by%20a%20known%20bad%20and%20malicious%20actor%20almost%20certainly%20has%20less%20value%0A%3E%20than%20the%20risk%20that%20a%20subtle%20change%20go%20unnoticed.%0A%3E%20%0A%3E%20I%27d%20suggest%20reverting%20to%205.3.1.%20Bearing%20in%20mind%20that%20there%20were%20security%0A%3E%20fixes%20after%20that%20point%20for%20ZDI-CAN-16587%20that%20would%20need%20to%20be%20reapplied.%0A%3E%20%0A%3E%20--%20%0A%3E%20see%20shy%20jo%0A&subject=Re%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%20bad%20actor)):

```
[Message part 1 (text/plain, inline)]
```

```
Package: xz-utils
Version: 5.6.1+really5.4.5-1
Severity: important
Tags: security

I count a minimum of 750 commits or contributions to xz by Jia Tan, who
backdoored it.

This includes all 700 commits made after they merged a pull request in Jan 7
2023, at which point they appear to have already had direct push access, which
would have also let them push commits with forged authors. Probably a number of
other commits before that point as well.

Reverting the backdoored version to a previous version is not sufficient to
know that Jia Tan has not hidden other backdoors in it. Version 5.4.5 still
contains the majority of those commits.

Commits by them such as 18d7facd3802b55c287581405c4d49c98708c136 
and ae5c07b22a6b3766b84f409f1b6b5c100469068a show that they were deep
into analyzing the security of xz. They were well placed to insert a buffer
overflow that could allow eg, a targeted xz file to cause arbitrary code
execution. The impact of such a security hole could be much more stealthy and
bad than the known backdoor since it would allow chaining xz with other
unrelated software releases on an ongoing basis.

The package should be reverted to a version before their involvement,
which started with commit 6468f7e41a8e9c611e4ba8d34e2175c5dacdbeb4.
Or their early commits vetted and revert to a later point, but any arbitrary 
commit by a known bad and malicious actor almost certainly has less value
than the risk that a subtle change go unnoticed.

I'd suggest reverting to 5.3.1\. Bearing in mind that there were security
fixes after that point for ZDI-CAN-16587 that would need to be reapplied.

-- 
see shy jo

```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**转发信息** 至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；包`xz-utils`。(Fri, 29 Mar 2024 21:36:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=7)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=7)，[链接](#6))。

* * *

**确认已发送** 至`Aurelien Jarno <aurelien@aurel32.net>`：

已收到额外信息并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。(Fri, 29 Mar 2024 21:36:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=9)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=9)，[链接](#8))。

* * *

[消息 #10](#10) 收到于 submit@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=10)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=10)，[回复](mailto:1068024@bugs.debian.org?body=On%20Fri%2C%2029%20Mar%202024%2022%3A32%3A23%20%2B0100%20Aurelien%20Jarno%20%3Caurelien%40aurel32.net%3E%20wrote%3A%0A%3E%20On%202024-03-29%2016%3A25%2C%20Joey%20Hess%20wrote%3A%0A%3E%20%3E%20I%27d%20suggest%20reverting%20to%205.3.1.%20Bearing%20in%20mind%20that%20there%20were%20security%0A%3E%20%3E%20fixes%20after%20that%20point%20for%20ZDI-CAN-16587%20that%20would%20need%20to%20be%20reapplied.%0A%3E%20%0A%3E%20Note%20that%20reverted%20to%20such%20an%20old%20version%20will%20break%20packages%20that%20use%0A%3E%20new%20symbols%20introduced%20since%20then.%20From%20a%20quick%20look%2C%20this%20is%20at%20least%3A%0A%3E%20-%20dpkg%0A%3E%20-%20erofs-utils%0A%3E%20-%20kmod%0A%3E%20%0A%3E%20Having%20dpkg%20in%20that%20list%20means%20that%20such%20downgrade%20has%20to%20be%20planned%0A%3E%20carefully.%0A%3E%20%0A%3E%20Regards%0A%3E%20Aurelien%0A%3E%20%0A%3E%20--%20%0A%3E%20Aurelien%20Jarno%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20GPG%3A%204096R%2F1DDD8C9B%0A%3E%20aurelien%40aurel32.net%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20http%3A%2F%2Faurel32.net%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E&In-Reply-To=%3CZgczZzqFSq450Nlh%40aurel32.net%3E))：

```
[Message part 1 (text/plain, inline)]
```

```
On 2024-03-29 16:25, Joey Hess wrote:
> I'd suggest reverting to 5.3.1\. Bearing in mind that there were security
> fixes after that point for ZDI-CAN-16587 that would need to be reapplied.

Note that reverted to such an old version will break packages that use
new symbols introduced since then. From a quick look, this is at least:
- dpkg
- erofs-utils
- kmod

Having dpkg in that list means that such downgrade has to be planned
carefully.

Regards
Aurelien

-- 
Aurelien Jarno                          GPG: 4096R/1DDD8C9B
aurelien@aurel32.net                     http://aurel32.net

```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**转发信息** 至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；包`xz-utils`。(Fri, 29 Mar 2024 21:36:04 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=12)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=12)，[链接](#11))。

* * *

**确认已发送** 至`Aurelien Jarno <aurelien@aurel32.net>`：

已收到额外信息并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。(Fri, 29 Mar 2024 21:36:04 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=14)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=14)，[链接](#13))。

* * *

**转发信息** 至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包`xz-utils`。 (Fri, 29 Mar 2024 22:57:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=17), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=17), [链接](#16)).

* * *

**感谢已发送**至`Thorsten Glaser <tg@mirbsd.de>`:

额外信息已收到并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。 (Fri, 29 Mar 2024 22:57:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=19), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=19), [链接](#18)).

* * *

[消息 #20](#20) 已接收到 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=20), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=20), [回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Fri%2C%2029%20Mar%202024%2022%3A47%3A48%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Aurelien%20Jarno%20dixit%3A%0A%3E%20%0A%3E%20%3EHaving%20dpkg%20in%20that%20list%20means%20that%20such%20downgrade%20has%20to%20be%20planned%0A%3E%20%3Ecarefully.%0A%3E%20%0A%3E%20Right.%20Not%20an%20argument%20against%2C%20though.%0A%3E%20Instead%2C%20this%20is%20a%20very%20good%20idea.%0A%3E%20%0A%3E%20What%20symbols%20are%20new%3F%20Can%20we%20somehow%20stub%20them%2C%20or%20at%20least%20where%0A%3E%20those%20are%20used%3F%20Or%20change%20the%20soname%2C%20so%20that%20the%20old%20and%20new-older%0A%3E%20versions%20are%20co%C3%AFnstallable%20for%20during%20the%20upgrade%3F%0A%3E%20%0A%3E%20bye%2C%0A%3E%20%2F%2Fmirabilos%0A%3E%20--%20%0A%3E%20%3Cigli%3E%20exceptions%3A%20a%20truly%20awful%20implementation%20of%20quite%20a%20nice%20idea.%0A%3E%20%3Cigli%3E%20just%20about%20the%20worst%20way%20you%20could%20do%20something%20like%20that%2C%20afaic.%0A%3E%20%3Cigli%3E%20it%27s%20like%20anti-design.%20%20%3Cmirabilos%3E%20that%20too%E2%80%A6%20may%20I%20quote%20you%20on%20that%3F%0A%3E%20%3Cigli%3E%20sure%2C%20tho%20i%20doubt%20anyone%20will%20listen%20%3B%29%0A%3E%20%0A%3E%20%0A&In-Reply-To=%3CPine.BSM.4.64L.2403292246190.8237%40herc.mirbsd.org%3E&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CPine.BSM.4.64L.2403292246190.8237%40herc.mirbsd.org%3E)):

```
Aurelien Jarno dixit:

>Having dpkg in that list means that such downgrade has to be planned
>carefully.

Right. Not an argument against, though.
Instead, this is a very good idea.

What symbols are new? Can we somehow stub them, or at least where
those are used? Or change the soname, so that the old and new-older
versions are coïnstallable for during the upgrade?

bye,
//mirabilos
-- 
<igli> exceptions: a truly awful implementation of quite a nice idea.
<igli> just about the worst way you could do something like that, afaic.
<igli> it's like anti-design.  <mirabilos> that too… may I quote you on that?
<igli> sure, tho i doubt anyone will listen ;)

```

* * *

**信息已转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包`xz-utils`。 (Sat, 30 Mar 2024 00:51:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=22), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=22), [链接](#21)).

* * *

**感谢已发送**至`Stephan Verbücheln <verbuecheln@posteo.de>`:

额外信息已收到并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。 (Sat, 30 Mar 2024 00:51:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=24), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=24), [链接](#23)).

* * *

[消息 #25](#25) 收到于 1068024@bugs.debian.org ([完整内容](bugreport.cgi?bug=1068024;msg=25), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=25), [回复](mailto:1068024@bugs.debian.org?References=%3Ce96cb21e64b1c68d4ec9304802bc7f010bf82535.camel%40posteo.de%3E&In-Reply-To=%3Ce96cb21e64b1c68d4ec9304802bc7f010bf82535.camel%40posteo.de%3E&subject=Re%3A%20Or%20remove%20xz%20altogether%3F&body=On%20Sat%2C%2030%20Mar%202024%2000%3A48%3A34%20%2B0000%20Stephan%20%3D%3FISO-8859-1%3FQ%3FVerb%3DFCcheln%3F%3D%20%3Cverbuecheln%40posteo.de%3E%20wrote%3A%0A%3E%20Maybe%20the%20people%20who%20criticized%20xz%20back%20in%20the%20day%20for%20being%20an%20amateur%0A%3E%20project%20implementing%20a%20defective%20file%20format%20were%20right%20all%20along%3F%0A%3E%20%0A%3E%20https%3A%2F%2Fwww.nongnu.org%2Flzip%2Fxz_inadequate.html%0A%3E%20%0A%3E%20Regards%0A%3E%20Stephan%0A%3E%20%0A%3E%20%0A)).

```
Maybe the people who criticized xz back in the day for being an amateur
project implementing a defective file format were right all along?

https://www.nongnu.org/lzip/xz_inadequate.html

Regards
Stephan

```

* * *

**信息转发** 至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包 `xz-utils`。 (Sat, 30 Mar 2024 02:51:02 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=27), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=27), [链接](#26)).

* * *

**已发送确认** 给 `Mark-Oliver Wolter <mow@mowny.de>`:

额外信息已接收并转发至列表。复制发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sat, 30 Mar 2024 02:51:03 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=29), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=29), [链接](#28)).

* * *

[消息 #30](#30) 收到于 1068024@bugs.debian.org ([完整内容](bugreport.cgi?bug=1068024;msg=30), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=30), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3C1b217452-111c-4f93-9323-1d4b335d9605%40mowny.de%3E&In-Reply-To=%3C1b217452-111c-4f93-9323-1d4b335d9605%40mowny.de%3E&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sat%2C%2030%20Mar%202024%2003%3A38%3A39%20%2B0100%20Mark-Oliver%20Wolter%20%3Cmow%40mowny.de%3E%20wrote%3A%0A%3E%20On%20Fri%2C%2029%20Mar%202024%2022%3A32%3A23%20%2B0100%20Aurelien%20Jarno%20%3Caurelien%40aurel32.net%3E%20%0A%3E%20wrote%3A%0A%3E%20%20%3E%20Having%20dpkg%20in%20that%20list%20means%20that%20such%20downgrade%20has%20to%20be%20planned%0A%3E%20%20%3E%20carefully.%0A%3E%20%0A%3E%20Might%20be%20easier%20overall%20to%20spend%20that%20effort%20on%20a%20hard%20switch%20to%20zstd%20%0A%3E%20instead.%0A%3E%20%0A%3E%20mfG%20mow%0A%3E%20%0A%3E%20%0A%3E%20%0A%3E%20%0A)).

```
On Fri, 29 Mar 2024 22:32:23 +0100 Aurelien Jarno <aurelien@aurel32.net> 
wrote:
> Having dpkg in that list means that such downgrade has to be planned
> carefully.

Might be easier overall to spend that effort on a hard switch to zstd 
instead.

mfG mow

```

* * *

**信息转发** 至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包 `xz-utils`。 (Sat, 30 Mar 2024 02:51:04 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=32), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=32), [链接](#31)).

* * *

**确认已发送**至`Guillem Jover <guillem@debian.org>`：

额外信息已接收并转发至列表。副本发送至`Jonathan Nieder <jrnieder@gmail.com>`。（2024 年 3 月 30 日 02:51:04 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=34)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=34)，[链接](#33)）。

* * *

[消息 #35](#35) 收到于 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=35), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=35), [回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20Or%20remove%20xz%20altogether%3F&body=On%20Sat%2C%2030%20Mar%202024%2003%3A49%3A12%20%2B0100%20Guillem%20Jover%20%3Cguillem%40debian.org%3E%20wrote%3A%0A%3E%20On%20Sat%2C%202024-03-30%20at%2000%3A48%3A34%20%2B0000%2C%20Stephan%20Verb%C3%BCcheln%20wrote%3A%0A%3E%20%3E%20Maybe%20the%20people%20who%20criticized%20xz%20back%20in%20the%20day%20for%20being%20an%20amateur%0A%3E%20%3E%20project%20implementing%20a%20defective%20file%20format%20were%20right%20all%20along%3F%0A%3E%20%3E%20%0A%3E%20%3E%20https%3A%2F%2Fwww.nongnu.org%2Flzip%2Fxz_inadequate.html%0A%3E%20%0A%3E%20%2ASigh%2A%2C%20the%20current%20situation%20is%20bad%20enough%2C%20and%20has%20nothing%20to%20do%0A%3E%20with%20the%20xz%20format%20or%20design%2C%20or%20the%20FUD%20and%20propaganda%20from%20that%0A%3E%20link.%20Please%20drop%20it%E2%80%A6%0A%3E%20%0A%3E%20Thanks%2C%0A%3E%20Guillem%0A%3E%20%0A%3E%20%0A&In-Reply-To=%3CZgd9qAG7rqbrMqN2%40thunder.hadrons.org%3E&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3Ce96cb21e64b1c68d4ec9304802bc7f010bf82535.camel%40posteo.de%3E%0A%20%3CZgd9qAG7rqbrMqN2%40thunder.hadrons.org%3E)):

```
On Sat, 2024-03-30 at 00:48:34 +0000, Stephan Verbücheln wrote:
> Maybe the people who criticized xz back in the day for being an amateur
> project implementing a defective file format were right all along?
> 
> https://www.nongnu.org/lzip/xz_inadequate.html

*Sigh*, the current situation is bad enough, and has nothing to do
with the xz format or design, or the FUD and propaganda from that
link. Please drop it…

Thanks,
Guillem

```

* * *

**信息转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包`xz-utils`。（2024 年 3 月 30 日 08:15:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=37)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=37)，[链接](#36)）。

* * *

**确认已发送**至`Pierre Ynard <linkfanel@yahoo.fr>`：

额外信息已接收并转发至列表。副本发送至`Jonathan Nieder <jrnieder@gmail.com>`。（2024 年 3 月 30 日 08:15:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=39)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=39)，[链接](#38)）。

* * *

[第40条消息](#40) 接收自 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=40), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=40), [回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3C661269552.4371505.1711786362353%40mail.yahoo.com%3E&References=%3C661269552.4371505.1711786362353.ref%40mail.yahoo.com%3E%0A%20%3C661269552.4371505.1711786362353%40mail.yahoo.com%3E&body=On%20Sat%2C%2030%20Mar%202024%2008%3A12%3A42%20%2B0000%20%28UTC%29%20Pierre%20Ynard%20%3Clinkfanel%40yahoo.fr%3E%20wrote%3A%0A%3E%20%3E%20Might%20be%20easier%20overall%20to%20spend%20that%20effort%20on%20a%20hard%20switch%20to%20zstd%0A%3E%20%3E%20instead.%0A%3E%20%0A%3E%20Good%20point.%20Another%20question%20that%20we%27ll%20probably%20have%20to%20ask%20anyway%0A%3E%20is%2C%20what%20is%20the%20future%20of%20xz-utils%20going%20to%20be%20now%3F%20Regarding%20its%0A%3E%20maintainership%20as%20well%2C%20both%20upstream%20and%20in%20Debian%3F%0A%3E%20%0A%3E%20From%20what%20I%20understand%2C%20this%20package%20has%20been%20NMU%27d%20in%20Debian%20for%20more%0A%3E%20than%2010%20years%20now%20-%20thanks%20to%20Sebastian%20for%20his%20work%20-%20and%20that%20might%0A%3E%20have%20been%20a%20point%20of%20weakness%20which%20was%20exploited%20to%20push%20a%20compromised%0A%3E%20version%20into%20the%20Debian%20archive%2C%20as%20seen%20in%20%231067708.%20To%20quote%20Thorsten%0A%3E%20from%20that%20thread%3A%0A%3E%20%0A%3E%20%3E%20Very%20much%20%2Anot%2A%20a%20fan%20of%20NMUs%20doing%20large%20changes%20such%20as%0A%3E%20%3E%20new%20upstream%20versions.%0A%3E%20%0A%3E%20I%20can%27t%20say%20why%20exactly%20he%20would%20not%20be%20a%20fan%2C%20but%20with%20hindsight%20that%0A%3E%20was%20an%20interesting%20call.%0A%3E%20%0A%3E%20Perhaps%20more%20importantly%2C%20it%20is%20said%20that%20the%20whole%20situation%20started%0A%3E%20with%20the%20lack%20of%20resources%20from%20the%20upstream%20maintainer%20to%20maintain%20the%0A%3E%20project%3A%0A%3E%20%0A%3E%20https%3A%2F%2Fwww.mail-archive.com%2Fxz-devel%40tukaani.org%2Fmsg00567.html%0A%3E%20%0A%3E%20which%20again%2C%20we%20can%20suspect%20at%20this%20point%2C%20was%20exploited%20with%20the%20same%0A%3E%20modus%20operandi%20to%20get%20a%20compromise%20vector%20coopted%20in%2C%20in%20the%20form%20of%20a%0A%3E%20new%20maintainer.%0A%3E%20%0A%3E%20And%20now%20if%20what%20we%20suspect%20is%20true%2C%20that%20relief%20maintainer%20has%20turned%0A%3E%20out%20to%20be%20a%20bad%20actor%2C%20which%20leaves%20upstream%20back%20to%20square%20one%20on%20the%0A%3E%20maintainance%20issue.%20I%20understand%20that%20Lasse%20Collin%20hasn%27t%20been%20able%20to%0A%3E%20weigh%20in%20on%20the%20situation%20yet.%0A%3E%20%0A%3E%20Is%20xz-utils%20going%20to%20be%20maintained%3F%20Will%20we%20want%20to%20keep%20in%20the%20archive%0A%3E%20an%20unmaintained%20low-level%20library%20-%20low-level%20as%20in%2C%20susceptible%20of%0A%3E%20getting%20pulled%20as%20a%20dependency%20in%20lots%20of%20places%20-%20and%20rely%20on%20it%20for%0A%3E%20components%20such%20as%20dpkg%3F%0A%3E%20%0A%3E%20I%27m%20not%20presuming%20the%20answers%20-%20it%27s%20still%20early%20and%20there%20is%20probably%0A%3E%20more%20to%20figure%20out%20yet%20-%20but%20back%20to%20the%20original%20point%2C%20we%20might%20want%0A%3E%20to%20consider%20this%20when%20sketching%20longer-term%20plans.%0A%3E%20%0A%3E%20--%20%0A%3E%20Pierre%20Ynard%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor)):

```
> Might be easier overall to spend that effort on a hard switch to zstd
> instead.

Good point. Another question that we'll probably have to ask anyway
is, what is the future of xz-utils going to be now? Regarding its
maintainership as well, both upstream and in Debian?

From what I understand, this package has been NMU'd in Debian for more
than 10 years now - thanks to Sebastian for his work - and that might
have been a point of weakness which was exploited to push a compromised
version into the Debian archive, as seen in #1067708\. To quote Thorsten
from that thread:

> Very much *not* a fan of NMUs doing large changes such as
> new upstream versions.

I can't say why exactly he would not be a fan, but with hindsight that
was an interesting call.

Perhaps more importantly, it is said that the whole situation started
with the lack of resources from the upstream maintainer to maintain the
project:

https://www.mail-archive.com/xz-devel@tukaani.org/msg00567.html

which again, we can suspect at this point, was exploited with the same
modus operandi to get a compromise vector coopted in, in the form of a
new maintainer.

And now if what we suspect is true, that relief maintainer has turned
out to be a bad actor, which leaves upstream back to square one on the
maintainance issue. I understand that Lasse Collin hasn't been able to
weigh in on the situation yet.

Is xz-utils going to be maintained? Will we want to keep in the archive
an unmaintained low-level library - low-level as in, susceptible of
getting pulled as a dependency in lots of places - and rely on it for
components such as dpkg?

I'm not presuming the answers - it's still early and there is probably
more to figure out yet - but back to the original point, we might want
to consider this when sketching longer-term plans.

-- 
Pierre Ynard

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包 `xz-utils`. (Sat, 30 Mar 2024 12:51:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=42), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=42), [链接](#41)).

* * *

**确认已发送**至 `Joey Hess <id@joeyh.name>`：

已接收额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sat, 30 Mar 2024 12:51:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=44), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=44), [链接](#43)).

* * *

[第45条消息](bugreport.cgi?bug=1068024;msg=45)已发送至1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=45)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=45)，[回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sat%2C%2030%20Mar%202024%2008%3A38%3A19%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20Aurelien%20Jarno%20wrote%3A%0A%3E%20%3E%20请注意，重新回退到这样一个旧版本会导致使用%0A%3E%20%3E%20自那时以来引入的新符号的软件包出现问题。经过初步查看，至少涉及以下内容：%0A%3E%20%3E%20-%20dpkg%0A%3E%20%3E%20-%20erofs-utils%0A%3E%20%3E%20-%20kmod%0A%3E%20%3E%20%0A%3E%20%3E%20在该列表中包含dpkg意味着必须仔细计划这种降级。%0A%3E%20%3E%20%0A%3E%20%3E%20我同意这将是一个具有挑战性的降级。我已经尝试过了%0A%3E%20%3E%20实验性地，一旦回退到旧版本的liblzma5，dpkg-deb就损坏了%0A%3E%20%3E%20因为缺少符号'XZ_5.4'。%0A%3E%20%3E%20%0A%3E%20%3E%20将liblzma5重命名为其他名称（例如liblzma6？）并使dpkg-deb%0A%3E%20%3E%20依赖于它似乎是避免混乱情况的一种方法。%0A%3E%20%3E%20%0A%3E%20%3E%20%0A%3E%20%3E%20顺便说一句，我已经在sid上重新构建了xz-utils 5.2.5-2.1~deb11u1（来自bullseye）%0A%3E%20%3E%20并在稍微调整了dpkg的打包后成功地构建了它：%0A%3E%20%3E%20%0A%3E%20%3E%20--- debian/libdpkg-dev.install.orig	2024-03-30 07:31:46.635365816 -0400%0A%3E%20%3E%20+++ debian/libdpkg-dev.install	2024-03-30 07:34:48.667477725 -0400%0A%3E%20%3E%20@@ -1,4 +1,5 @@%0A%3E%20%3E%20 usr/include/dpkg/*.h%0A%3E%20%3E%20-usr/lib/*/pkgconfig/libdpkg.pc%0A%3E%20%3E%20-usr/lib/*/libdpkg.a%0A%3E%20%3E%20+usr/lib/pkgconfig/libdpkg.pc%0A%3E%20%3E%20+usr/lib/libdpkg.a%0A%3E%20%3E%20 usr/share/aclocal/dpkg-*.m4%0A%3E%20%3E%20+usr/lib/libdpkg.la%0A%3E%20%3E%20%0A%3E%20%3E%20（在禁用了测试套件后，因为xz消息输出的变化%0A%3E%20%3E%20导致了测试失败。）%0A%3E%20%3E%20%0A%3E%20%3E%20--%20%0A%3E%20%3E%20看看内向的jo）%0A&参考：<ZgcjtvSjQM59nX_w%40kitenet.net>%0A%20<ZgczZzqFSq450Nlh%40aurel32.net>%0A%20<ZggHu6gxzO6nwMa5%40kitenet.net>%0A&在回复中：<ZggHu6gxzO6nwMa5%40kitenet.net>）：

```
[Message part 1 (text/plain, inline)]
```

```
Aurelien Jarno wrote:
> Note that reverted to such an old version will break packages that use
> new symbols introduced since then. From a quick look, this is at least:
> - dpkg
> - erofs-utils
> - kmod
> 
> Having dpkg in that list means that such downgrade has to be planned
> carefully.

I agree this would be a challanging downgrade. I've tried it myself
experimentally and once a downgraded liblzma5 is unpacked, dpkg-deb is broken
with missing symbol 'XZ_5.4'.

Renaming liblzma5 to something else (liblzma6?) and making dpkg-deb
depend on that seems like one way to go that would avoid messy situations.

FWIW, I rebuilt xz-utils 5.2.5-2.1~deb11u1 (from bullseye) on sid
and then got dpkg to build against that successfully after a few minor
changes to dpkg's packaging:

--- debian/libdpkg-dev.install.orig	2024-03-30 07:31:46.635365816 -0400
+++ debian/libdpkg-dev.install	2024-03-30 07:34:48.667477725 -0400
@@ -1,4 +1,5 @@
 usr/include/dpkg/*.h
-usr/lib/*/pkgconfig/libdpkg.pc
-usr/lib/*/libdpkg.a
+usr/lib/pkgconfig/libdpkg.pc
+usr/lib/libdpkg.a
 usr/share/aclocal/dpkg-*.m4
+usr/lib/libdpkg.la

(And after disabling the test suite since changes in xz message output
caused a test failure.)

-- 
see shy jo

```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**信息转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包`xz-utils`。 （2024年3月30日，GMT 13:21:02）（[完整文本](bugreport.cgi?bug=1068024;msg=47)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=47)，[链接](#46)）。

* * *

**确认已发送**至`Ivan Shmakov <ivan@siamics.net>`：

收到额外信息并转发给列表。副本发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sat, 30 Mar 2024 13:21:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=49), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=49), [链接](#48)).

* * *

[消息＃50](#50) 收到于 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=50)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=50)，[回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%20bad%20actor&body=On%20Sat%2C%2030%20Mar%202024%2012%3A55%3A21%20%2B0000%20Ivan%20Shmakov%20%3Civan%40siamics.net%3E%20wrote%3A%0A%3E%20%3E%3E%3E%3E%3E%20On%202024-03-30%2C%20Guillem%20Jover%20wrote%3A%0A%3E%20%3E%3E%3E%3E%3E%20On%20Sat%2C%202024-03-30%20at%2000%3A48%3A34%20%2B0000%2C%20Stephan%20Verb%C3%BCcheln%20wrote%3A%0A%3E%20%0A%3E%20%20%3E%3E%20主题：%20Re%3A%20Bug%231068024%3A%20Or%20remove%20xz%20altogether%3F%0A%3E%20%0A%3E%20%20%3E%3E%20也许那些批评xz的人在那个时代把它看作是一个业余项目实现了一个有缺陷的文件格式，他们是对的？%0A%3E%20%0A%3E%20%20%3E%3E%20https%3A%2F%2Fwww.nongnu.org%2Flzip%2Fxz_inadequate.html%0A%3E%20%0A%3E%20%20%3E%20*Sigh*，当前的情况已经够糟糕了，并且与xz格式或设计，以及那个链接中的恐慌和宣传无关。请放弃它……%0A%3E%20%0A%3E%20%09虽然我不会自己提及它，也不会就此（显然与缺乏诚信的开发者自愿维护该项目有关）变更的辩论做出可能具有挑衅性的主题，因为它与手头问题只是间接相关（在一些情况下，软件项目的设计质量确实影响维护工作的数量及其他事务），上述评论引起了我的好奇心：链接中的文档的哪些部分是‘恐慌和宣传’？因为我刚刚浏览了一下，没有找到明显的例子。%0A%3E%20%0A%3E%20%09当然，我不是压缩和编码领域的专家，并且不能轻易地就那里提出的大多数论点的有效性发言，但至少其中一些看起来是合理的。有些我觉得是显而易见的，比如，例如：%0A%3E%20%0A%3E%20%20ADD%3E%20CRC的一个众所周知的特性是它们能够检测到与CRC本身大小相等的突发错误。使用一个比数据字大的CRC是一个错误，因为一个与数据字大小相同的CRC同样能检测到所有错误，同时产生更少的误报。%0A%3E%20%0A%3E%20%09如果对那篇文档提出了合理的反驳意见，我认为指向它对这个讨论是合适的。除此之外，我不会在这里继续这个话题。我会监控一些互联网论坛，虽然（新闻：alt.os.linux.debian，新闻：comp.misc，irc：//irc.efnet.org/%2523coders，等等）。%0A%3E%20%0A%3E%20--%20%0A%3E%20FSF联合会成员＃7257＃＃np。Moonsong — Shane Jackman%0A%3E%20%0A%3E%20%0A&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%09%3Ce96cb21e64b1c68d4ec9304802bc7f010bf82535.camel%40posteo.de%3E%0A%09%3CZgd9qAG7rqbrMqN2%40thunder.hadrons.org%3E%0A%20%3C-XNy7LTsg7gxGioANsY_0529Z8WFYAgv%40violet.siamics.net%3E&In-Reply-To=%3C-XNy7LTsg7gxGioANsY_0529Z8WFYAgv%40violet.siamics.net%3E))

```
>>>>> On 2024-03-30, Guillem Jover wrote:
>>>>> On Sat, 2024-03-30 at 00:48:34 +0000, Stephan Verbücheln wrote:

 >> Subject: Re: Bug#1068024: Or remove xz altogether?

 >> Maybe the people who criticized xz back in the day for being an amateur
 >> project implementing a defective file format were right all along?

 >> https://www.nongnu.org/lzip/xz_inadequate.html

 > *Sigh*, the current situation is bad enough, and has nothing to do
 > with the xz format or design, or the FUD and propaganda from that
 > link.  Please drop it…

	While I wouldn’t have brought it up myself, nor the arguably
	provocative Subject: change (reverted), as it’s indeed only
	tangentially related to the issue at hand (which apparently
	have resulted from the absense of good faith developers
	volunteering to maintain this project – and in a word of
	defense, the design quality of a software project /does/
	influence both the amount of maintenance work and, other
	things being equal, the desire to get involved), the comment
	above got me curious: which parts of the document linked are
	‘FUD and propaganda’?  Because I’ve just glanced it over and
	I don’t find any obvious examples.

	Granted, I’m not an expert in the field of compression and
	coding, per se, and can’t readily speak on the validity of
	most of the arguments given there, those at least appear
	plausible.  Some arguments I find obvious, such as, e. g.:

 ADD> A well-known property of CRCs is their ability to detect burst
 ADD> errors up to the size of the CRC itself.  Using a CRC larger
 ADD> than the dataword is an error because a CRC just as large as
 ADD> the dataword equally detects all errors while it produces
 ADD> a lower number of false positives.

	If there’s a reasonable rebuttal to the points raised in that
	document, I believe that a pointer to it would be appropriate
	for this discussion.  Other than that, I’m not going to go on
	this tangent any further here.  I’ll be monitoring a handful
	of Internet fora for that, though (news:alt.os.linux.debian,
	news:comp.misc, irc://irc.efnet.org/%23coders, etc.)

-- 
FSF associate member #7257  np. Moonsong — Shane Jackman

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。（2024 年 3 月 30 日星期六 17:06:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=54), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=54), [链接](#53)）。

* * *

**已发送确认函**至 `noloader@gmail.com`：

收到额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（2024 年 3 月 30 日星期六 17:06:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=56), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=56), [链接](#55)）。

* * *

[消息 #57](#57) 已收到，发送至 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=57), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=57), [回复](mailto:1068024@bugs.debian.org?body=On%20Sat%2C%2030%20Mar%202024%2013%3A02%3A28%20-0400%20Jeffrey%20Walton%20%3Cnoloader%40gmail.com%3E%20wrote%3A%0A%3E%20Lasse%20Collin%20提供了一份声明，位于 <https%3A%2F%2Ftukaani.org%2Fxz-backdoor%2F>。%0A%3E%20%0A%3E%20%0A&subject=Re%3A&In-Reply-To=%3CCAH8yC8nMFjJOJdvyj4JeoVA%2BRq_xe20%3DdDvnD2k-03bQJr-y_w%40mail.gmail.com%3E&References=%3CCAH8yC8nMFjJOJdvyj4JeoVA%2BRq_xe20%3DdDvnD2k-03bQJr-y_w%40mail.gmail.com%3E)):

```
Lasse Collin provided a statement at <https://tukaani.org/xz-backdoor/>.

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。（2024 年 3 月 30 日星期六 18:21:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=59), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=59), [链接](#58)）。

* * *

**已发送确认函**至 `Joey Hess <id@joeyh.name>`：

收到额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（2024 年 3 月 30 日星期六 18:21:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=61), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=61), [链接](#60)）。

* * *

[消息 #62](#62) 收到自 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=62), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=62), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E&In-Reply-To=%3CZghXFP5JiJgCMyiY%40kitenet.net%3E&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sat%2C%2030%20Mar%202024%2014%3A16%3A52%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20I%20have%20prepared%20a%20git%20repository%20that%20is%20a%20fork%20of%20xz%20from%20the%20point%20I%0A%3E%20identified%20before%20the%20attacker%28s%29%20did%20anything%20to%20it.%20In%20my%20fork%2C%20I%20have%0A%3E%20renamed%20liblzma%20to%20liblzmaunscathed.%20That%20allows%20it%20to%20be%20installed%0A%3E%20alongside%20current%20dpkg%20without%20breaking%20dpkg%20with%20an%20old%20version%20of%0A%3E%20liblzma.%20%0A%3E%20%0A%3E%20My%20git%20repository%20is%20here%20%28note%20all%20my%20commits%20are%20gpg%20signed%29%3A%0A%3E%20https%3A%2F%2Fgit.joeyh.name%2Findex.cgi%2Fxz-unscathed%2F%0A%3E%20%0A%3E%20It%20also%20has%20a%20debian%20branch%20which%20contains%20a%20debian%20directory.%20I%27ve%0A%3E%20built%20packages%20of%20that%2C%20as%20well%20as%20building%20dpkg-1.22.6%20against%20it.%0A%3E%20I%27ve%20attached%20the%20patch%20I%20used%20to%20build%20dpkg.%0A%3E%20%0A%3E%20My%20build%20of%20dpkg%20ended%20up%20not%20being%20linked%20to%20a%20lzma%20library%20at%20all%2C%0A%3E%20because%20liblzmaunscathed%20is%20too%20old%20to%20support%20concurrent%20decompression%2C%0A%3E%20which%20the%20configure%20script%20detects.%20So%20dpkg-deb%20instead%20uses%20xz-utils%0A%3E%20to%20decompress%20debs.%20I%20replaced%20xz-utils.deb%20with%20the%20one%20built%20from%20my%0A%3E%20fork%2C%20and%20dpkg%20seems%20to%20work%20fine%20using%20it.%0A%3E%20%0A%3E%20If%20Debian%20decided%20to%20go%20this%20route%2C%20you%20could%20add%20xz-utils-unscathed%0A%3E%20to%20unstable%2C%20and%20at%20the%20same%20time%20update%20xz-utils%20to%20not%20build%0A%3E%20xz-utils.deb.%20Then%20build%20dpkg%20against%20it.%20Then%20look%20into%20forward%20porting%0A%3E%20or%20re-implementing%20concurrent%20decompression%20if%20that%20is%20really%20important%0A%3E%20to%20have.%0A%3E%20%0A%3E%20I%20only%20plan%20to%20maintain%20this%20fork%20minimally%2C%20eg%20backporting%20security%0A%3E%20fixes.%20The%20goal%20is%20not%20to%20take%20over%20from%20xz%20upstream%2C%20but%20to%20get%20the%0A%3E%20possibly%20backdoored%20code%20off%20of%20production%20systems%20ASAP.%20Presumably%20xz%0A%3E%20upstream%20will%20come%20up%20with%20their%20own%20solution%20long-term.%0A%3E%20%0A%3E%20--%20%0A%3E%20see shy jo)):

```
[Message part 1 (text/plain, inline)]
```

```
I have prepared a git repository that is a fork of xz from the point I
identified before the attacker(s) did anything to it. In my fork, I have
renamed liblzma to liblzmaunscathed. That allows it to be installed
alongside current dpkg without breaking dpkg with an old version of
liblzma. 

My git repository is here (note all my commits are gpg signed):
https://git.joeyh.name/index.cgi/xz-unscathed/

It also has a debian branch which contains a debian directory. I've
built packages of that, as well as building dpkg-1.22.6 against it.
I've attached the patch I used to build dpkg.

My build of dpkg ended up not being linked to a lzma library at all,
because liblzmaunscathed is too old to support concurrent decompression,
which the configure script detects. So dpkg-deb instead uses xz-utils
to decompress debs. I replaced xz-utils.deb with the one built from my
fork, and dpkg seems to work fine using it.

If Debian decided to go this route, you could add xz-utils-unscathed
to unstable, and at the same time update xz-utils to not build
xz-utils.deb. Then build dpkg against it. Then look into forward porting
or re-implementing concurrent decompression if that is really important
to have.

I only plan to maintain this fork minimally, eg backporting security
fixes. The goal is not to take over from xz upstream, but to get the
possibly backdoored code off of production systems ASAP. Presumably xz
upstream will come up with their own solution long-term.

-- 
see shy jo

```

```
[dpkg.patch (text/x-diff, attachment)]
```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**信息已转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。（Sat, 30 Mar 2024 20:06:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=64)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=64)），（[链接](#63)）。

* * *

向 `noloader@gmail.com` 发送**确认函**：

收到额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（Sat, 30 Mar 2024 20:06:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=66)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=66)），（[链接](#65)）。

* * *

收到第 `67` 号信息至 `1068024@bugs.debian.org`（[完整文本](bugreport.cgi?bug=1068024;msg=67)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=67)），（[回复](mailto:1068024@bugs.debian.org?subject=Re%3A&body=On%20Sat%2C%2030%20Mar%202024%2016%3A03%3A11%20-0400%20Jeffrey%20Walton%20%3Cnoloader%40gmail.com%3E%20wrote%3A%0A%3E%20It%20looks%20like%20more%20analysis%20has%20revealed%20this%20is%20a%20RCE%20with%20the%0A%3E%20payload%20in%20the%20modulus%20of%20a%20public%20key%3A%20%22The%20payload%20is%20extracted%20from%0A%3E%20the%20N%20value%20%28the%20public%20key%29%20passed%20to%20RSA_public_decrypt%2C%20checked%0A%3E%20against%20a%20simple%20fingerprint%2C%20and%20decrypted%20with%20a%20fixed%20ChaCha20%20key%0A%3E%20before%20the%20Ed448%20signature%20verification...%22%20Also%20see%0A%3E%20%3Chttps%3A%2F%2Fwww.openwall.com%2Flists%2Foss-security%2F2024%2F03%2F30%2F36%3E.%0A%3E%20%0A%3E%20%0A&References=%3CCAH8yC8koLuwMGUFMcbdzFtMpD%2BgWmHeszkmdrDdCcjrfh2CRWg%40mail.gmail.com%3E&In-Reply-To=%3CCAH8yC8koLuwMGUFMcbdzFtMpD%2BgWmHeszkmdrDdCcjrfh2CRWg%40mail.gmail.com%3E）：

```
It looks like more analysis has revealed this is a RCE with the
payload in the modulus of a public key: "The payload is extracted from
the N value (the public key) passed to RSA_public_decrypt, checked
against a simple fingerprint, and decrypted with a fixed ChaCha20 key
before the Ed448 signature verification..." Also see
<https://www.openwall.com/lists/oss-security/2024/03/30/36>.

```

* * *

**信息已转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。（Sun, 31 Mar 2024 00:21:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=69)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=69)），（[链接](#68)）。

* * *

已向 `Christoph Anton Mitterer <calestyo@scientia.org>` 发送**确认函**：

收到额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（Sun, 31 Mar 2024 00:21:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=71)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=71)），（[链接](#70)）。

* * *

[消息 #72](#72) 收到于 1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=72)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=72)，[回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3C80164840620921cfa6986165c8843267e791e601.camel%40scientia.org%3E&References=%3C80164840620921cfa6986165c8843267e791e601.camel%40scientia.org%3E&body=On%20Sun%2C%2031%20Mar%202024%2001%3A16%3A07%20%2B0100%20Christoph%20Anton%20Mitterer%20%3Ccalestyo%40scientia.org%3E%20wrote%3A%0A%3E%20Hey.%0A%3E%20%0A%3E%20Can%20we%20be%20confidently%20sure%20that%20going%20back%20to%205.4.5%20is%20enough%3F%0A%3E%20%0A%3E%20At%20least%20the%20git%20tag%20for%20that%20seems%20to%20be%20still%20signed%20by%20the%0A%3E%20adversary%3A%0A%3E%20https%3A%2F%2Fgit.tukaani.org%2F%3Fp%3Dxz.git%3Ba%3Dtag%3Bh%3D9e4835399118b98954f110f76af2a0d504d2f531%0A%3E%20%0A%3E%20The%20last%20one%2C%20still%20from%20Lasse%20Collin%20seems%20to%20be%205.4.1%3A%0A%3E%20https%3A%2F%2Fgit.tukaani.org%2F%3Fp%3Dxz.git%3Ba%3Dtag%3Bh%3Df52502e78bf84f516a739e8d8a1357f27eeea75f%0A%3E%20%0A%3E%20%0A%3E%20Cheers%2C%0A%3E%20Chris.%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%20bad%20actor))。

```
Hey.

Can we be confidently sure that going back to 5.4.5 is enough?

At least the git tag for that seems to be still signed by the
adversary:
https://git.tukaani.org/?p=xz.git;a=tag;h=9e4835399118b98954f110f76af2a0d504d2f531

The last one, still from Lasse Collin seems to be 5.4.1:
https://git.tukaani.org/?p=xz.git;a=tag;h=f52502e78bf84f516a739e8d8a1357f27eeea75f

Cheers,
Chris.

```

* * *

**信息已转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；包`xz-utils`。（Sun, 31 Mar 2024 00:48:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=74)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=74)，[链接](#73)）。

* * *

**确认已发送**至`Alberto Garcia <berto@igalia.com>`：

额外信息已收到并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。（Sun, 31 Mar 2024 00:48:02 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=76)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=76)，[链接](#75)）。

* * *

[消息 #77](#77) 收到于 1068024@bugs.debian.org ([全文](bugreport.cgi?bug=1068024;msg=77), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=77), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3C80164840620921cfa6986165c8843267e791e601.camel%40scientia.org%3E%0A%20%3CZgiyDD_DS2d4m8Jo%40zeus.local%3E&In-Reply-To=%3CZgiyDD_DS2d4m8Jo%40zeus.local%3E&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sun%2C%2031%20Mar%202024%2001%3A45%3A00%20%2B0100%20Alberto%20Garcia%20%3Cberto%40igalia.com%3E%20wrote%3A%0A%3E%20On%20Sun%2C%20Mar%2031%2C%202024%20at%2001%3A16%3A07AM%20%2B0100%2C%20Christoph%20Anton%20Mitterer%20wrote%3A%0A%3E%20%3E%20The%20last%20one%2C%20still%20from%20Lasse%20Collin%20seems%20to%20be%205.4.1%3A%0A%3E%20%3E%20https%3A%2F%2Fgit.tukaani.org%2F%3Fp%3Dxz.git%3Ba%3Dtag%3Bh%3Df52502e78bf84f516a739e8d8a1357f27eeea75f%0A%3E%20%0A%3E%20There%20are%20commits%20from%20Jia%20before%20that%2C%20and%20some%20that%20are%20authored%20by%0A%3E%20Lasse%20but%20thank%20Jia%20for%20the%20contribution%20%28the%20oldest%20one%20is%2020e7a33e%29.%0A%3E%20%0A%3E%20The%20most%20recent%20release%20that%20does%20not%20contain%20any%20of%20that%20seems%20to%20be%0A%3E%20v5.3.2alpha.%0A%3E%20%0A%3E%20Berto%0A%3E%20%0A%3E%20%0A)):

```
On Sun, Mar 31, 2024 at 01:16:07AM +0100, Christoph Anton Mitterer wrote:
> The last one, still from Lasse Collin seems to be 5.4.1:
> https://git.tukaani.org/?p=xz.git;a=tag;h=f52502e78bf84f516a739e8d8a1357f27eeea75f

There are commits from Jia before that, and some that are authored by
Lasse but thank Jia for the contribution (the oldest one is 20e7a33e).

The most recent release that does not contain any of that seems to be
v5.3.2alpha.

Berto

```

* * *

**信息转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`。 (Sun, 31 Mar 2024 01:03:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=79), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=79), [链接](#78)).

* * *

**确认已发送**至`Christoph Anton Mitterer <calestyo@scientia.org>`：

收到额外信息并转发至列表。拷贝发送至`Jonathan Nieder <jrnieder@gmail.com>`。 (Sun, 31 Mar 2024 01:03:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=81), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=81), [链接](#80)).

* * *

[消息 #82](#82) 收到自 1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=82)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=82)，[回复](mailto:1068024@bugs.debian.org?body=On%20Sun%2C%2031%20Mar%202024%2003%3A01%3A06%20%2B0200%20Christoph%20Anton%20Mitterer%20%3Ccalestyo%40scientia.org%3E%20wrote%3A%0A%3E%20One%20more%20thing%3A%0A%3E%20%0A%3E%20%0A%3E%20Is%20anyone%20on%20the%20Debian%20side%20trying%20to%20figure%20out%20how%20far%20we%27ve%20been%0A%3E%20practically%20affected%3F%0A%3E%20%0A%3E%20I%20mean%20let%27s%20assume%20we%27re%20%22lucky%22%2C%20and%20the%20backdoor%20is%20only%20in%0A%3E%205.6.0%2F5.6.1...%20and%20that%20none%20of%20the%20adversary%27s%20earlier%20commits%0A%3E%20introduced%20any%20serious%20holes%5B0%5D%20which%20wouldn%27t%20be%20known%20yet.%0A%3E%20%0A%3E%20Then%20many%20servers%20running%20Debian%20%28which%20is%20then%20typically%20Debian%0A%3E%20stable%29%2C%20would%20be%20sill%20safe.%0A%3E%20%0A%3E%20However%2C%20many%20people%20%28and%20I%27d%20guess%20that%20includes%20DDs%2FDMs%29%20run%20their%0A%3E%20workstations%2Flaptops%20with%20testing%2Funstable.%0A%3E%20So%20IMO%20it%27s%20not%20enough%20to%20know%20that%20Debian%20stable%20is%20likely%20not%0A%3E%20affected%20-%20I%27d%20be%20rather%20relieved%20if%20I%27d%20knew%20that%20there%27s%20a%20good%0A%3E%20chance%20that%20the%20personal%20computers%20of%20most%20DDs%2FDMs%20%28who%20push%20uploads%2C%0A%3E%20etc.%20pp.%29%20were%20%28at%20least%20practically%29%20safe.%0A%3E%20%0A%3E%20Some%20guy%20decrypted%5B1%5D%20the%20strings%20from%20the%20maleware%27s%20binary%20payload%3A%0A%3E%20https%3A%2F%2Fgist.github.com%2Fq3k%2Faf3d93b6a1f399de28fe194add452d01%23file-hashes-txt-L115%0A%3E%20where%20only%20%2Fusr%2Fsbin%2Fsshd%20would%20be%20named.%0A%3E%20%0A%3E%20Should%28%21%29%20it%20turn%20out%2C%20that%20the%20actual%20binary%20payload%20actually%20only%0A%3E%20affects%20sshd%20and%20especially%20does%20not%20on%20it%27s%20own%20pull%20in%20evil%20code%2C%20it%0A%3E%20might%20at%20least%20be%20that%20all%20people%20who%20hadn%27t%20sshd%20running%20%28or%20not%0A%3E%20directly%20accessible%20because%20a%20router%2Ffirewall%2Fetc.%20was%20in%20front%20of%20it%29%2C%0A%3E%20would%20effectively%20still%20be%20safe.%0A%3E%20No%20idea%20whether%20or%20not%20this%20is%20really%20the%20case%20-%20but%20if%20so%2C%20it%20might%20at%0A%3E%20least%20mean%20that%20many%20workstations%2Flaptops%20are%20safe%2C%20because%20they%27re%20not%0A%3E%20usually%20directly%20accessible%20from%20the%20internet.%0A%3E%20%0A%3E%20%0A%3E%20But%20even%20then%2C%20it%20would%20probably%20need%20to%20be%20checked%20whether%20all%20the%0A%3E%20versions%20that%20Debian%20ever%20used%2Fshipped%20in%0A%3E%20testing%2Funstable%28%2Fexperimental%29%20were%20actually%20fully%20identical%20to%20the%0A%3E%20versions%20that%20are%20now%20analysed%20by%20forensic%20folks.%0A%3E%20%0A%3E%20Is%20the%20security%20team%20doing%20anything%20like%20that%3F%0A%3E%20%0A%3E%20%0A%3E%20Cheers%2C%0A%3E%20Chris.%0A%3E%20%0A%3E%20%0A%3E%20PS%3A%20if%20someone%20from%20the%20security%20team%20reads%20along%2C%0A%3E%20https%3A%2F%2Fgist.github.com%2Fthesamesam%2F223949d5a074ebc3dce9ee78baad9e27%0A%3E%20which%20is%20from%20Sam%20James%20of%20Gentoo%2C%20seems%20to%20collect%20good%20information%20on%0A%3E%20what%20is%20known%20so%20far.%0A%3E%20Might%20be%20worth%20to%20add%20to%20the%20links%20in%20the%20security%20tracker.%0A%3E%20%0A%3E%20%0A%3E%20%5B0%5D%20There%20are%20reports%20about%20an%20added%20%27.%27%20in%20the%20CMakeLists.txt%0A%3E%20disabling%20some%20sandboxing%2C%20but%20AFAICS%20at%20least%20the%20current%20version%20uses%0A%3E%20autotools%20for%20building%3F%21%0A%3E%20%0A%3E%20%5B1%5D%20He%20also%20provided%20the%20code%20he%20used%20to%20do%20so.%0A%3E%20https%3A%2F%2Fgist.github.com%2Fq3k%2Faf3d93b6a1f399de28fe194add452d01%3Fpermalink_comment_id%3D5006546%23gistcomment-5006546%0A%3E%20%0A&subject=Re%3A%20revert%20to%20version%20that%20does%20

```
One more thing:

Is anyone on the Debian side trying to figure out how far we've been
practically affected?

I mean let's assume we're "lucky", and the backdoor is only in
5.6.0/5.6.1... and that none of the adversary's earlier commits
introduced any serious holes[0] which wouldn't be known yet.

Then many servers running Debian (which is then typically Debian
stable), would be sill safe.

However, many people (and I'd guess that includes DDs/DMs) run their
workstations/laptops with testing/unstable.
So IMO it's not enough to know that Debian stable is likely not
affected - I'd be rather relieved if I'd knew that there's a good
chance that the personal computers of most DDs/DMs (who push uploads,
etc. pp.) were (at least practically) safe.

Some guy decrypted[1] the strings from the maleware's binary payload:
https://gist.github.com/q3k/af3d93b6a1f399de28fe194add452d01#file-hashes-txt-L115
where only /usr/sbin/sshd would be named.

Should(!) it turn out, that the actual binary payload actually only
affects sshd and especially does not on it's own pull in evil code, it
might at least be that all people who hadn't sshd running (or not
directly accessible because a router/firewall/etc. was in front of it),
would effectively still be safe.
No idea whether or not this is really the case - but if so, it might at
least mean that many workstations/laptops are safe, because they're not
usually directly accessible from the internet.

But even then, it would probably need to be checked whether all the
versions that Debian ever used/shipped in
testing/unstable(/experimental) were actually fully identical to the
versions that are now analysed by forensic folks.

Is the security team doing anything like that?

Cheers,
Chris.

PS: if someone from the security team reads along,
https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27
which is from Sam James of Gentoo, seems to collect good information on
what is known so far.
Might be worth to add to the links in the security tracker.

[0] There are reports about an added '.' in the CMakeLists.txt
disabling some sandboxing, but AFAICS at least the current version uses
autotools for building?!

[1] He also provided the code he used to do so.
https://gist.github.com/q3k/af3d93b6a1f399de28fe194add452d01?permalink_comment_id=5006546#gistcomment-5006546

```

* * *

**信息已转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`。（周日，2024年3月31日01:45:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=84)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=84)），（[链接](#83)）。

* * *

致 `Thorsten Glaser <tg@mirbsd.de>` 的**确认已发送**：

收到额外信息并已转发至列表。拷贝发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（周日，2024年3月31日01:45:03 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=86)），（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=86)），（[链接](#85)）。

* * *

[第87条信息](#87)接收自1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=87)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=87)，[回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sun%2C%2031%20Mar%202024%2001%3A36%3A31%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Pierre%20Ynard%20dixit%3A%0A%3E%20%0A%3E%20%3Eversion%20into%20the%20Debian%20archive%2C%20as%20seen%20in%20%231067708.%20To%20quote%20Thorsten%0A%3E%20%3Efrom%20that%20thread%3A%0A%3E%20%3E%0A%3E%20%3E%3E%20Very%20much%20%2Anot%2A%20a%20fan%20of%20NMUs%20doing%20large%20changes%20such%20as%0A%3E%20%3E%3E%20new%20upstream%20versions.%0A%3E%20%3E%0A%3E%20%3EI%20can%27t%20say%20why%20exactly%20he%20would%20not%20be%20a%20fan%2C%20but%20with%20hindsight%20that%0A%3E%20%3Ewas%20an%20interesting%20call.%0A%3E%20%0A%3E%20It%20turned%20out%20to%20indeed%20be%20related%2C%20although%20I%20cannot%20blame%20Sebastian%0A%3E%20for%20not%20spotting%20it%2C%20as%20well%20as%20it%20was%20hidden.%20I%20actually%20wrote%20about%0A%3E%20that%20earlier%20on%20Fedi%3A%20%28Markdown%20formatting%20lost%20here%20though%29%0A%3E%20%0A%3E%20%7C%20I%20was%20considering%20replying%20to%20this%20comment%20on%20the%20%E2%80%9Cplease%20update%20xz%0A%3E%20%7C%20package%E2%80%9D%20bugreport%20earlier%20with%20that%20the%20discussion%20is%20not%20irrelevant%0A%3E%20%7C%20and%20that%20it%E2%80%99s%20the%20maintainer%E2%80%99s%20responsibility%20on%20new%20upgrades%20to%20check%0A%3E%20%7C%20for%20new%20legal%20issues%20and%20%E2%80%9Cother%20hidden%20gems%E2%80%9D.%0A%3E%20%7C%0A%3E%20%7C%20I%20didn%E2%80%99t%20because%20I%20didn%E2%80%99t%20want%20to%20bother%20going%20in%20with%20an%20annoyed%0A%3E%20%7C%20self-righteous%20%E2%80%9Cuser%E2%80%9D.%0A%3E%20%7C%0A%3E%20%7C%20Now%20it%20turns%20out%20all%20three%20of%20the%20involved%20ones%20were%20%E2%80%9Cstring%20%2B%20number%0A%3E%20%7C%20%40%20freemailer%E2%80%9D%20%23JiaT75%20sockpuppets%2C%20so%20it%E2%80%99s%20probably%20okay%20I%20didn%E2%80%99t%0A%3E%20%7C%20bother.%0A%3E%20%7C%0A%3E%20%7C%20Not%20that%20I%20blame%20Sebastian%E2%80%8A%E2%80%94%E2%80%8Ait%20was%20very%20well%20hidden%2C%20and%20even%20my%0A%3E%20%7C%20usual%20diffing%20between%20old%20and%20new%20version%20would%20not%20have%20found%20it.%0A%3E%20%7C%0A%3E%20%7C%20I%20do%20take%20away%20from%20this%20to%20also%20check%20the%20diff%20between%20VCS%20repo%20at%0A%3E%20%7C%20the%20time%20of%20the%20release%20and%20release%20tarball.%20Perhaps%20also%20between%0A%3E%20%7C%20branch%20and%20tag%20if%20they%2C%20like%20Apache%20Tomcat%2C%20introduce%20extra%20commits%0A%3E%20%7C%20there.%0A%3E%20%0A%3E%20%3EIs%20xz-utils%20going%20to%20be%20maintained%3F%20Will%20we%20want%20to%20keep%20in%20the%20archive%0A%3E%20%3Ean%20unmaintained%20low-level%20library%20-%20low-level%20as%20in%2C%20susceptible%20of%0A%3E%20%3Egetting%20pulled%20as%20a%20dependency%20in%20lots%20of%20places%20-%20and%20rely%20on%20it%20for%0A%3E%20%3Ecomponents%20such%20as%20dpkg%3F%0A%3E%20%0A%3E%20That%20scenario%20you%20describing%20here%20would%20actually%20be%20much%20less%20of%20a%0A%3E%20problem.%20The%20problem%20comes%20when%20the%20library%20in%20that%20state%20then%20does%0A%3E%20get%20updates%20that%20probably%20are%20not%20even%20necessary%20but%20shiny%20enough%0A%3E%20people%20demand%20them.%0A%3E%20%0A%3E%20bye%2C%0A%3E%20%2F%2Fmirabilos%20%28also%20a%20Debian%20Developer%2C%20despite%20the%20From%29%0A%3E%20--%20%0A%3E%20When%20he%20found%20out%20that%20the%20m68k%20port%20was%20in%20a%20pretty%20bad%20shape%2C%20he%20did%0A%3E%20not%2C%20like%20many%20before%20him%2C%20shrug%20and%20move%20on%3B%20instead%2C%20he%20took%20it%20upon%0A%3E%20himself%20to%20start%20compiling%20things%2C%20just%20so%20he%20could%20compile%20his%20shell.%0A%3

```
Pierre Ynard dixit:

>version into the Debian archive, as seen in #1067708\. To quote Thorsten
>from that thread:
>
>> Very much *not* a fan of NMUs doing large changes such as
>> new upstream versions.
>
>I can't say why exactly he would not be a fan, but with hindsight that
>was an interesting call.

It turned out to indeed be related, although I cannot blame Sebastian
for not spotting it, as well as it was hidden. I actually wrote about
that earlier on Fedi: (Markdown formatting lost here though)

| I was considering replying to this comment on the “please update xz
| package” bugreport earlier with that the discussion is not irrelevant
| and that it’s the maintainer’s responsibility on new upgrades to check
| for new legal issues and “other hidden gems”.
|
| I didn’t because I didn’t want to bother going in with an annoyed
| self-righteous “user”.
|
| Now it turns out all three of the involved ones were “string + number
| @ freemailer” #JiaT75 sockpuppets, so it’s probably okay I didn’t
| bother.
|
| Not that I blame Sebastian — it was very well hidden, and even my
| usual diffing between old and new version would not have found it.
|
| I do take away from this to also check the diff between VCS repo at
| the time of the release and release tarball. Perhaps also between
| branch and tag if they, like Apache Tomcat, introduce extra commits
| there.

>Is xz-utils going to be maintained? Will we want to keep in the archive
>an unmaintained low-level library - low-level as in, susceptible of
>getting pulled as a dependency in lots of places - and rely on it for
>components such as dpkg?

That scenario you describing here would actually be much less of a
problem. The problem comes when the library in that state then does
get updates that probably are not even necessary but shiny enough
people demand them.

bye,
//mirabilos (also a Debian Developer, despite the From)
-- 
When he found out that the m68k port was in a pretty bad shape, he did
not, like many before him, shrug and move on; instead, he took it upon
himself to start compiling things, just so he could compile his shell.
How's that for dedication. -- Wouter, about my Debian/m68k revival

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；包 `xz-utils`。（Sun, 31 Mar 2024 01:45:04 GMT）（[全文](bugreport.cgi?bug=1068024;msg=89)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=89)，[链接](#88)）。

* * *

**确认已发送**至 `Thorsten Glaser <tg@mirbsd.de>`：

收到额外信息并转发至列表。抄送至 `Jonathan Nieder <jrnieder@gmail.com>`。（Sun, 31 Mar 2024 01:45:04 GMT）（[全文](bugreport.cgi?bug=1068024;msg=91)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=91)，[链接](#90)）。

* * *

[消息 #92](#92) 收到于 1068024@bugs.debian.org（[全文](bugreport.cgi?bug=1068024;msg=92)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=92)，[回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Sun%2C%2031%20Mar%202024%2001%3A38%3A51%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Christoph%20Anton%20Mitterer%20dixit%3A%0A%3E%20%0A%3E%20%3ECan%20we%20be%20confidently%20sure%20that%20going%20back%20to%205.4.5%20is%20enough%3F%0A%3E%20%0A%3E%20No.%0A%3E%20%0A%3E%20%3EThe%20last%20one%2C%20still%20from%20Lasse%20Collin%20seems%20to%20be%205.4.1%3A%0A%3E%20%0A%3E%20在此bug报告中，我们正在讨论回退到任何贡献者之前。查看是否是一个选项（现在看起来是一个明智的步骤）由joeyh确认（谢谢）。%0A%3E%20%0A%3E%20再见，%0A%3E%20//mirabilos%0A%3E%20--%20%0A%3E%2022%3A20⎜asarch⎜那种持续存在的疯狂变成了大师%0A%3E%2022%3A21⎜asarch⎜而疯狂与天才之间的距离只有成功衡量18：35⎜asarch⎜“精神病人总是不一致。理智的本质是不一致的不一致%0A%3E%20%0A%3E%20%0A&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%20%3C80164840620921cfa6986165c8843267e791e601.camel%40scientia.org%3E%0A%20%3CPine.BSM.4.64L.2403310137460.25541%40herc.mirbsd.org%3E&In-Reply-To=%3CPine.BSM.4.64L.2403310137460.25541%40herc.mirbsd.org%3E）：

```
Christoph Anton Mitterer dixit:

>Can we be confidently sure that going back to 5.4.5 is enough?

No.

>The last one, still from Lasse Collin seems to be 5.4.1:

In this bugreport, we’re discussing going back to before any
contributions by the adversary. To see whether it’s an option
at all (and it sounds like a sensible step right now) which
joeyh confirmed (thanks).

bye,
//mirabilos
-- 
22:20⎜<asarch> The crazy that persists in his craziness becomes a master
22:21⎜<asarch> And the distance between the craziness and geniality is
only measured by the success 18:35⎜<asarch> "Psychotics are consistently
inconsistent. The essence of sanity is to be inconsistently inconsistent

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；包 `xz-utils`。（Sun, 31 Mar 2024 01:51:03 GMT）（[全文](bugreport.cgi?bug=1068024;msg=94)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=94)，[链接](#93)）。

* * *

**确认已发送**至 `Thorsten Glaser <tg@mirbsd.de>`：

收到额外信息并转发至列表。 复件发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sun, 31 Mar 2024 01:51:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=96), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=96), [链接](#95)).

* * *

[消息 #97](#97) 已收到，发送至 1068024@bugs.debian.org（[全文](bugreport.cgi?bug=1068024;msg=97)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=97)，[回复](mailto:1068024@bugs.debian.org?body=On%20Sun%2C%2031%20Mar%202024%2001%3A46%3A35%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Christoph%20Anton%20Mitterer%20dixit%3A%0A%3E%20%0A%3E%20%3E是否有人在Debian方面试图弄清楚我们到底受到了多少实际影响？%0A%3E%20%0A%3E%20是的，一个多团队工作小组正在处理此事，并将通知用户何时可以继续进行，包括需要抛弃多少内容并重新构建。%0A%3E%20%0A%3E%20再见，%0A%3E%20//mirabilos%0A%3E%20--%20%0A%3E%20“很棒，/usr/share/doc/mksh/examples/uhr.gz是一个安装mksh到每个系统的理由。”%0A%3E%20%09-- XTaran 在 OpenRheinRuhr，非常激动%0A%3E%20（英文：【...】uhr.gz是一个安装mksh到每个系统的理由。）%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%20%3C17377f1e2560f2127e9c896bb87d9941e1b7483e.camel%40scientia.org%3E%0A%20%3CPine.BSM.4.64L.2403310145330.25541%40herc.mirbsd.org%3E&In-Reply-To=%3CPine.BSM.4.64L.2403310145330.25541%40herc.mirbsd.org%3E)):

```
Christoph Anton Mitterer dixit:

>Is anyone on the Debian side trying to figure out how far we've been
>practically affected?

Yes, a multi-team task force is working on it and will inform users
once it is known how to proceed, inclusing how much to throw away
and rebuild.

bye,
//mirabilos
-- 
„Cool, /usr/share/doc/mksh/examples/uhr.gz ist ja ein Grund,
mksh auf jedem System zu installieren.“
	-- XTaran auf der OpenRheinRuhr, ganz begeistert
(EN: “[…]uhr.gz is a reason to install mksh on every system.”)

```

* * *

**信息转发** 给 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`； 软件包 `xz-utils`。 (Sun, 31 Mar 2024 02:57:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=99), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=99), [链接](#98)).

* * *

**致谢已发送** 给 `Joshua Hudson <joshudson@gmail.com>`：

收到额外信息并转发至列表。 复件发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sun, 31 Mar 2024 02:57:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=101), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=101), [链接](#100)).

* * *

收到 [消息 #102](#102) 至 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=102), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=102), [回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3CCA%2BjjjYS4HmdfPOj1RggWRYCMetHqX%2Bh%2BLO3qd-83XrQkSm%2BeUQ%40mail.gmail.com%3E&References=%3CCA%2BjjjYS4HmdfPOj1RggWRYCMetHqX%2Bh%2BLO3qd-83XrQkSm%2BeUQ%40mail.gmail.com%3E&body=On%20Sat%2C%2030%20Mar%202024%2019%3A55%3A20%20-0700%20Joshua%20Hudson%20%3Cjoshudson%40gmail.com%3E%20wrote%3A%0A%3E%20The%20dpkg%20-%3E%20xz-utils%20downgrade%20problem%20has%20a%20suggestion%20that%20suggests%0A%3E%20itself.%0A%3E%20%0A%3E%201%29%20Downgrade%20dpkg%27s%20dependency%20to%20the%20last%20known%20good.%20It%20doesn%27t%20matter%0A%3E%20how%20old%20so%20long%20as%20it%20can%20read%20the%20file%20formats.%20I%20understand%20this%20is%0A%3E%20likely%20to%20be%205.4.1.%0A%3E%20%0A%3E%202%29%20Statically%20link%20all%20the%20decompressor%20libraries%20into%20dpkg.%20Yes%20this%20means%0A%3E%20gzip%2C%20bzip2%2C%20zstd%2C%20and%20xz%27s%20libraries.%20Today%27s%20compile%20and%20link%20time%0A%3E%20optimizers%20are%20pretty%20good%3B%20it%20should%20be%20able%20to%20drop%20all%20references%20to%20the%0A%3E%20compression%20side%20pretty%20much%20by%20itself.%20This%20would%20generally%20be%20a%20pretty%0A%3E%20good%20change%20of%20its%20own%20as%20it%20makes%20the%20minimal%20system%20easier%20to%20understand%0A%3E%20and%20the%20net-install%20CD%20smaller.%0A%3E%20%0A%3E%20然后你可以为 `xz-utils` 的所有其他依赖项做你需要做的事情。

```
[Message part 1 (text/plain, inline)]
```

```
The dpkg -> xz-utils downgrade problem has a suggestion that suggests
itself.

1) Downgrade dpkg's dependency to the last known good. It doesn't matter
how old so long as it can read the file formats. I understand this is
likely to be 5.4.1.

2) Statically link all the decompressor libraries into dpkg. Yes this means
gzip, bzip2, zstd, and xz's libraries. Today's compile and link time
optimizers are pretty good; it should be able to drop all references to the
compression side pretty much by itself. This would generally be a pretty
good change of its own as it makes the minimal system easier to understand
and the net-install CD smaller.

Then you're free to do what you need to do for all the other dependencies
of xz-utils.

```

```
[Message part 2 (text/html, inline)]
```

* * *

**转发的信息** 至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。 (Sun, 31 Mar 2024 04:57:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=104), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=104), [链接](#103)).

* * *

**确认已发送** 至 `Thorsten Glaser <tg@mirbsd.de>`：

已收到额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sun, 31 Mar 2024 04:57:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=106), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=106), [链接](#105)).

* * *

[消息 #107](#107) 已接收至 1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=107)）（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=107)）（[回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3CPine.BSM.4.64L.2403310441550.25541%40herc.mirbsd.org%3E&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CCA%2BjjjYS4HmdfPOj1RggWRYCMetHqX%2Bh%2BLO3qd-83XrQkSm%2BeUQ%40mail.gmail.com%3E%0A%20%3CPine.BSM.4.64L.2403310441550.25541%40herc.mirbsd.org%3E&body=On%20Sun%2C%2031%20Mar%202024%2004%3A43%3A34%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Joshua%20Hudson%20dixit%3A%0A%3E%20%0A%3E%20%3E2%29%20Statically%20link%20all%20the%20decompressor%20libraries%20into%20dpkg.%20Yes%20this%20means%0A%3E%20%0A%3E%20Totally%20no.%0A%3E%20%0A%3E%20Also%2C%20at%20this%20point%20in%20time%2C%20I%E2%80%99m%20pretty%20sure%20that%20new%20external%0A%3E%20suggestions%20are%E2%80%A6%20not%20very%20helpful.%20The%20situation%20is%20being%20analysed%0A%3E%20by%20a%20cross-team%20taskforce%2C%20please%20let%20them%20do%20the%20already-stressing%0A%3E%20job%20%E2%98%BB%0A%3E%20%0A%3E%20bye%2C%0A%3E%20%2F%2Fmirabilos%0A%3E%20--%20%0A%3E%20%2F%E2%81%80%5C%20The%20UTF-8%20Ribbon%0A%3E%20%E2%95%B2%C2%A0%E2%95%B1%20Campaign%20against%0A%3E%20%C2%A0%E2%95%B3%C2%A0%20HTML%20eMail%21%20Also%2C%0A%3E%20%E2%95%B1%C2%A0%E2%95%B2%20header%20encryption%21%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20Bug%231068024%3A%20Potential%20solution%20to%20your%20downgrade%20problem%20in%0A%20dpkg))

```
Joshua Hudson dixit:

>2) Statically link all the decompressor libraries into dpkg. Yes this means

Totally no.

Also, at this point in time, I’m pretty sure that new external
suggestions are… not very helpful. The situation is being analysed
by a cross-team taskforce, please let them do the already-stressing
job ☻

bye,
//mirabilos
-- 
/⁀\ The UTF-8 Ribbon
╲ ╱ Campaign against
 ╳  HTML eMail! Also,
╱ ╲ header encryption!

```

* * *

**信息已转发** 至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`。（Sun, 31 Mar 2024 20:57:04 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=109)）（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=109)）（[链接](#108)）。

* * *

**致谢已发送** 给 `Joey Hess <id@joeyh.name>`：

已接收额外信息并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（Sun, 31 Mar 2024 20:57:04 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=111)）（[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=111)）（[链接](#110)）。

* * *

[消息 #112](#112) 已收到，编号 1068024@bugs.debian.org ([全文](bugreport.cgi?bug=1068024;msg=112), [邮箱](bugreport.cgi?bug=1068024;mbox=yes;msg=112), [回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3CZgnNr-XTmx8gwJcj%40kitenet.net%3E&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E%0A%20%3CZgnNr-XTmx8gwJcj%40kitenet.net%3E&body=On%20Sun%2C%2031%20Mar%202024%2016%3A55%3A11%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20Since%20there%27s%20been%20some%20discussion%20about%20versions%2C%20the%20version%20in%20my%0A%3E%20xz-unscathed%20git%20repository%20is%20the%20same%20as%20xz%205.3.2alpha%2C%20with%20the%0A%3E%20addition%20of%20a%20fix%20for%20CVE-2022-1271%20that%20did%20not%20make%20it%20into%20that%0A%3E%20version.%20%28It%20was%20fixed%20in%205.2.6%2C%20but%205.3.2alpha%20was%20diverged%20from%0A%3E%205.2.5.%20Jia%20Tan%20was%20involved%20in%205.2.6.%29%0A%3E%20%0A%3E%205.2.5%20might%20be%20a%20more%20stable%20version%20to%20revert%20to%3B%20it%20also%20predates%0A%3E%20Jia%20Tan%27s%20involvement.%20The%20CVE-2022-1271%20fix%20would%20need%20to%20be%20included.%0A%3E%20%0A%3E%20Note%20that%20erofs-utils%20apparently%20had%20a%20reason%20to%20need%20the%205.3.2alpha%0A%3E%20release%2C%20so%20reverting%20to%205.2.5%20would%20probably%20cause%20difficulty%20with%20that%0A%3E%20package.%20That%20dependency%20versioning%20information%20is%20not%20included%20in%20the%0A%3E%20debian%20sources%20for%20erofs-utils%20BTW.%20I%20have%20not%20checked%20compatability%0A%3E%20with%20other%20packages%20except%20for%20dpkg.%0A%3E%20%0A%3E%20--%20%0A%3E%20see%20shy%20jo%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor)):

```
[Message part 1 (text/plain, inline)]
```

```
Since there's been some discussion about versions, the version in my
xz-unscathed git repository is the same as xz 5.3.2alpha, with the
addition of a fix for CVE-2022-1271 that did not make it into that
version. (It was fixed in 5.2.6, but 5.3.2alpha was diverged from
5.2.5\. Jia Tan was involved in 5.2.6.)

5.2.5 might be a more stable version to revert to; it also predates
Jia Tan's involvement. The CVE-2022-1271 fix would need to be included.

Note that erofs-utils apparently had a reason to need the 5.3.2alpha
release, so reverting to 5.2.5 would probably cause difficulty with that
package. That dependency versioning information is not included in the
debian sources for erofs-utils BTW. I have not checked compatability
with other packages except for dpkg.

-- 
see shy jo

```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

转发**信息**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包 `xz-utils`。 (Sun, 31 Mar 2024 21:00:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=114), [邮箱](bugreport.cgi?bug=1068024;mbox=yes;msg=114), [链接](#113)).

* * *

已发送**确认函**至 `Joey Hess <id@joeyh.name>`：

收到额外信息并转发至列表。复制发送给 `Jonathan Nieder <jrnieder@gmail.com>`。 (Sun, 31 Mar 2024 21:00:03 GMT) ([全文](bugreport.cgi?bug=1068024;msg=116), [邮箱](bugreport.cgi?bug=1068024;mbox=yes;msg=116), [链接](#115))。

* * *

[消息 #117](#117) 收到于 1068024@bugs.debian.org ([完整内容](bugreport.cgi?bug=1068024;msg=117), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=117), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E%0A%20%3CZgnNr-XTmx8gwJcj%40kitenet.net%3E%0A%20%3CZgnOPmJ7J-P63-18%40kitenet.net%3E&In-Reply-To=%3CZgnOPmJ7J-P63-18%40kitenet.net%3E&body=On%20Sun%2C%2031%20Mar%202024%2016%3A57%3A34%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20%3E%20The%20situation%20is%20being%20analysed%20by%20a%20cross-team%20taskforce%2C%0A%3E%20%3E%20please%20let%20them%20do%20the%20already-stressing%20job%20%E2%98%BB%0A%3E%20%0A%3E%20Sorry%2C%20didn%27t%20see%20that%20before%20sending%20my%20previous%20message.%0A%3E%20I%27ll%20leave%20it%20to%20you%20guys.%0A%3E%20%0A%3E%20--%20%0A%3E%20see%20shy%20jo%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor)):

```
[Message part 1 (text/plain, inline)]
```

```
> The situation is being analysed by a cross-team taskforce,
> please let them do the already-stressing job ☻

Sorry, didn't see that before sending my previous message.
I'll leave it to you guys.

-- 
see shy jo

```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**信息转发** 给 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包`xz-utils`。(Mon, 01 Apr 2024 00:09:02 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=119), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=119), [链接](#118))。

* * *

已发送**确认**至`noloader@gmail.com`：

收到额外信息并转发至清单。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。(Mon, 01 Apr 2024 00:09:02 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=121), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=121), [链接](#120))。

* * *

[消息 #122](#122) 收到于 1068024@bugs.debian.org ([完整内容](bugreport.cgi?bug=1068024;msg=122), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=122), [回复](mailto:1068024@bugs.debian.org?References=%3CCAH8yC8m_cng-PX5FQ9yNr98bj6fpJNBZu_-72N9_3kpRN9Ne_Q%40mail.gmail.com%3E&In-Reply-To=%3CCAH8yC8m_cng-PX5FQ9yNr98bj6fpJNBZu_-72N9_3kpRN9Ne_Q%40mail.gmail.com%3E&subject=Re%3A&body=On%20Sun%2C%2031%20Mar%202024%2020%3A07%3A36%20-0400%20Jeffrey%20Walton%20%3Cnoloader%40gmail.com%3E%20wrote%3A%0A%3E%20An%20important%20comment%20from%20oss-security%20mailing%20list%20message%2C%0A%3E%20%3Chttps%3A%2F%2Fwww.openwall.com%2Flists%2Foss-security%2F2024%2F03%2F31%2F4%3E%3A%0A%3E%20%0A%3E%20commit%20f9cf4c05edd14dedfe63833f8ccbe41b55823b00%20%28HEAD%20-%3E%20master%2C%0A%3E%20origin%2Fmaster%2C%20origin%2FHEAD%29%0A%3E%20Author%3A%20Lasse%20Collin%20%3Classe.collin%40...aani.org%3E%0A%3E%20Date%3A%20%20%20Sat%20Mar%2030%2014%3A36%3A28%202024%20%2B0200%0A%3E%20%0A%3E%20%20%20%20%20CMake%3A%20Fix%20sabotaged%20Landlock%20sandbox%20check.%0A%3E%20%0A%3E%20%20%20%20%20It%20never%20enabled%20it.%0A%3E%20%0A%3E%20%0A)):

```
An important comment from oss-security mailing list message,
<https://www.openwall.com/lists/oss-security/2024/03/31/4>:

commit f9cf4c05edd14dedfe63833f8ccbe41b55823b00 (HEAD -> master,
origin/master, origin/HEAD)
Author: Lasse Collin <lasse.collin@...aani.org>
Date:   Sat Mar 30 14:36:28 2024 +0200

    CMake: Fix sabotaged Landlock sandbox check.

    It never enabled it.

```

* * *

**信息转发** 给 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包 `xz-utils`。（Tue, 02 Apr 2024 12:39:06 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=124)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=124)，[链接](#123)）。

* * *

**确认已发送**至 `Guillem Jover <guillem@debian.org>`：

额外信息已收到并转发至列表。拷贝已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。（Tue, 02 Apr 2024 12:39:06 GMT）（[完整文本](bugreport.cgi?bug=1068024;msg=126)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=126)，[链接](#125)）。

* * *

[消息 #127](#127) 收到于 1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=127)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=127)，[回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E%0A%20%3CZgv7TOijQVR-0ffh%40thunder.hadrons.org%3E&In-Reply-To=%3CZgv7TOijQVR-0ffh%40thunder.hadrons.org%3E&body=On%20Tue%2C%202%20Apr%202024%2014%3A34%3A20%20%2B0200%20Guillem%20Jover%20%3Cguillem%40debian.org%3E%20wrote%3A%0A%3E%20Hi%21%0A%3E%20%0A%3E%20On%20Sat%2C%202024-03-30%20at%2014%3A16%3A52%20-0400%2C%20Joey%20Hess%20wrote%3A%0A%3E%20%3E%20My%20git%20repository%20is%20here%20%28note%20all%20my%20commits%20are%20gpg%20signed%29%3A%0A%3E%20%3E%20https%3A%2F%2Fgit.joeyh.name%2Findex.cgi%2Fxz-unscathed%2F%0A%3E%20%0A%3E%20%3E%20My%20build%20of%20dpkg%20ended%20up%20not%20being%20linked%20to%20a%20lzma%20library%20at%20all%2C%0A%3E%20%3E%20because%20liblzmaunscathed%20is%20too%20old%20to%20support%20concurrent%20decompression%2C%0A%3E%20%3E%20which%20the%20configure%20script%20detects.%20So%20dpkg-deb%20instead%20uses%20xz-utils%0A%3E%20%3E%20to%20decompress%20debs.%20I%20replaced%20xz-utils.deb%20with%20the%20one%20built%20from%20my%0A%3E%20%3E%20fork%2C%20and%20dpkg%20seems%20to%20work%20fine%20using%20it.%0A%3E%20%0A%3E%20dpkg%20应该能够在没有多线程压缩器或解压器支持的情况下使用旧版 liblzma。（两者都应该是可选的，如果可用的话）。我认为问题可能是你似乎错过了重命名 .pc.in 文件，或者这并没有包含在 -dev 包中，也许 dpkg 没有使用你附加的补丁中的内容？我只检查了重命名提交，并没有检查过打包工作，也没有尝试过构建它。

```
Hi!

On Sat, 2024-03-30 at 14:16:52 -0400, Joey Hess wrote:
> My git repository is here (note all my commits are gpg signed):
> https://git.joeyh.name/index.cgi/xz-unscathed/

> My build of dpkg ended up not being linked to a lzma library at all,
> because liblzmaunscathed is too old to support concurrent decompression,
> which the configure script detects. So dpkg-deb instead uses xz-utils
> to decompress debs. I replaced xz-utils.deb with the one built from my
> fork, and dpkg seems to work fine using it.

dpkg should be able to use an old liblzma w/o multi-threaded compressor
or decompressor support (both are intended to be optionally used if
available). I think the problem might be that you seem to have missed
renaming the .pc.in file, and that does not get included in the -dev
package perhaps, or not picked up then by dpkg with your attached
patch to it? I only checked the renaming commit, didn't check the
packaging nor tried to build it.

(Please do not take this mail as endorsing any specific action, just
wanted to clarify/correct the above.)

Thanks,
Guillem

```

* * *

**信息已转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`。

`Bug#1068024`；软件包`xz-utils`。 (Wed, 03 Apr 2024 09:21:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=129), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=129), [链接](#128))。

* * *

**已发送确认**给`Joey Hess <id@joeyh.name>`：

收到额外信息并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。 (Wed, 03 Apr 2024 09:21:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=131), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=131), [链接](#130))。

* * *

[第132条消息](#132) 已接收到 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=132), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=132), [回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Wed%2C%203%20Apr%202024%2002%3A55%3A55%20-0400%20Joey%20Hess%20%3Cid%40joeyh.name%3E%20wrote%3A%0A%3E%20Guillem%20Jover%20wrote%3A%0A%3E%20%3E%20dpkg%20应能够在没有多线程压缩器的情况下使用旧的liblzma%0A%3E%20%3E%20或解压缩器支持%0A%3E%20%0A%3E%20啊，你是对的。configure 确实找到了该库，但我忘了更新%0A%3E%20一些ifdefs。附上更新的 dpkg 补丁，该补丁构建了共享库并运行正常。%0A%3E%20%0A%3E%20--%20%0A%3E%20看到你就怕 jo%0A&References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E%0A%20%3CZgv7TOijQVR-0ffh%40thunder.hadrons.org%3E%0A%20%3CZgz9exBIUTM3Ip8Z%40kitenet.net%3E&In-Reply-To=%3CZgz9exBIUTM3Ip8Z%40kitenet.net%3E)):

```
[Message part 1 (text/plain, inline)]
```

```
Guillem Jover wrote:
> dpkg should be able to use an old liblzma w/o multi-threaded compressor
> or decompressor support

Ah you're right. configure did find the library, but I'd missed updating
some ifdefs. Attached updated dpkg patch which does build with the
shared library and works.

-- 
see shy jo

```

```
[dpkg.patch (text/x-diff, attachment)]
```

```
[signature.asc (application/pgp-signature, inline)]
```

* * *

**信息已转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包`xz-utils`。 (Wed, 03 Apr 2024 11:09:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=134), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=134), [链接](#133))。

* * *

**已发送确认**给`Thorsten Glaser <tg@mirbsd.de>`：

收到额外信息并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。 (Wed, 03 Apr 2024 11:09:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=136), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=136), [链接](#135))。

* * *

[消息 #137](#137) 收到于 1068024@bugs.debian.org（[完整文本](bugreport.cgi?bug=1068024;msg=137)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=137)，[回复](mailto:1068024@bugs.debian.org?subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Wed%2C%203%20Apr%202024%2011%3A06%3A27%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Joey%20Hess%20dixit%3A%0A%3E%20%0A%3E%20%3E---%20orig%2Fdpkg-1.22.6%2Fdebian%2Fcontrol%092024-03-02%2021%3A30%3A15.000000000%20-0400%0A%3E%20%3E%2B%2B%2B%20dpkg-1.22.6%2Fdebian%2Fcontrol%092024-03-30%2013%3A14%3A37.746223895%20-0400%0A%3E%20%0A%3E%20%3E%20%23%20Version%20needed%20for%20multi-threaded%20decompressor%20support.%0A%3E%20%3E-%20liblzma-dev%20%28%3E%3D%205.4.0%29%2C%0A%3E%20%3E%2B%20liblzmaunscathed-dev%2C%0A%3E%20%0A%3E%20这条评论可能也需要调整。%0A%3E%20%0A%3E%20%3E%20%23%20Version%20needed%20for%20multi-threaded%20decompressor%20support.%0A%3E%20%3E-%20xz-utils%20%28%3E%3D%205.4.0%29%20%3C%21nocheck%3E%2C%0A%3E%20%3E%2B%20xz-utils%20%3C%21nocheck%3E%2C%0A%3E%20%0A%3E%20同上%0A%3E%20%0A%3E%20%3E%20%23%20Version%20needed%20for%20multi-threaded%20decompressor%20support.%0A%3E%20%3E-%20liblzma-dev%20%28%3E%3D%205.4.0%29%2C%0A%3E%20%3E%2B%20liblzmaunscathed-dev%2C%0A%3E%20%0A%3E%20同上%0A%3E%20%0A%3E%20%3E%20%23%20Version%20needed%20for%20multi-threaded%20decompressor%20support.%0A%3E%20%3E-%20xz-utils%20%28%3E%3D%205.4.0%29%2C%0A%3E%20%3E%2B%20xz-utils%2C%0A%3E%20%0A%3E%20同上%0A%3E%20%0A%3E%20%3E---%20orig%2Fdpkg-1.22.6%2Fdebian%2Flibdpkg-dev.install%092024-02-04%2022%3A31%3A16.000000000%20-0400%0A%3E%20%3E%2B%2B%2B%20dpkg-1.22.6%2Fdebian%2Flibdpkg-dev.install%092024-03-30%2013%3A25%3A27.043840706%20-0400%0A%3E%20%3E%40%40%20-1%2C4%20%2B1%2C5%20%40%40%0A%3E%20%3E%20usr%2Finclude%2Fdpkg%2F%2A.h%0A%3E%20%3E-usr%2Flib%2F%2A%2Fpkgconfig%2Flibdpkg.pc%0A%3E%20%3E-usr%2Flib%2F%2A%2Flibdpkg.a%0A%3E%20%3E%2Busr%2Flib%2Fpkgconfig%2Flibdpkg.pc%0A%3E%20%3E%2Busr%2Flib%2Flibdpkg.a%0A%3E%20%0A%3E%20为什么这个库要在这里移出 M-A 目录呢？%0A%3E%20（可能是个问题给 Guillem 啊。）%0A%3E%20%0A%3E%20%3E%2Busr%2Flib%2Flibdpkg.la%0A%3E%20%0A%3E%20如果没记错，我们并没有在运送 libtool 文件，对吧？或者我是不是把它和一些 BSD 端口/软件包混淆了？%0A%3E%20%0A%3E%20再见，%0A%3E%20//mirabilos%0A%3E%20--%20%0A%3E%20[...]

```
Joey Hess dixit:

>--- orig/dpkg-1.22.6/debian/control	2024-03-02 21:30:15.000000000 -0400
>+++ dpkg-1.22.6/debian/control	2024-03-30 13:14:37.746223895 -0400

> # Version needed for multi-threaded decompressor support.
>- liblzma-dev (>= 5.4.0),
>+ liblzmaunscathed-dev,

The comment probably needs adjusting as well.

> # Version needed for multi-threaded decompressor support.
>- xz-utils (>= 5.4.0) <!nocheck>,
>+ xz-utils <!nocheck>,

dito

> # Version needed for multi-threaded decompressor support.
>- liblzma-dev (>= 5.4.0),
>+ liblzmaunscathed-dev,

idem

> # Version needed for multi-threaded decompressor support.
>- xz-utils (>= 5.4.0),
>+ xz-utils,

el mismo

> # Version needed for multi-threaded decompressor support.
>- xz-utils (>= 5.4.0),
>+ xz-utils,

the same

>--- orig/dpkg-1.22.6/debian/libdpkg-dev.install	2024-02-04 22:31:16.000000000 -0400
>+++ dpkg-1.22.6/debian/libdpkg-dev.install	2024-03-30 13:25:27.043840706 -0400
>@@ -1,4 +1,5 @@
> usr/include/dpkg/*.h
>-usr/lib/*/pkgconfig/libdpkg.pc
>-usr/lib/*/libdpkg.a
>+usr/lib/pkgconfig/libdpkg.pc
>+usr/lib/libdpkg.a

Why, exactly, does the library move out of the M-A directory here?
(Probably a question for guillem though.)

>+usr/lib/libdpkg.la

IIRC we were not shipping libtool files, were we? Or am I confusing
this with some BSD ports/packages now?

bye,
//mirabilos
-- 
[...] if maybe ext3fs wasn't a better pick, or jfs, or maybe reiserfs, oh but
what about xfs, and if only i had waited until reiser4 was ready... in the be-
ginning, there was ffs, and in the middle, there was ffs, and at the end, there
was still ffs, and the sys admins knew it was good. :)  -- Ted Unangst über *fs

```

* * *

**信息转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`. (Wed, 03 Apr 2024 19:21:02 GMT) ([全文](bugreport.cgi?bug=1068024;msg=139), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=139), [链接](#138)).

* * *

**已发送确认**至 `Sebastian Andrzej Siewior <sebastian@breakpoint.cc>`：

额外信息收到并转发至列表。副本发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Wed, 03 Apr 2024 19:21:02 GMT) ([全文](bugreport.cgi?bug=1068024;msg=141), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=141), [链接](#140)).

* * *

[消息 #142](#142) 收到，发送至 1068024@bugs.debian.org ([全文](bugreport.cgi?bug=1068024;msg=142), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=142), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgczZzqFSq450Nlh%40aurel32.net%3E%0A%20%3CZggHu6gxzO6nwMa5%40kitenet.net%3E%0A%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZghXFP5JiJgCMyiY%40kitenet.net%3E%0A%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3CZgv7TOijQVR-0ffh%40thunder.hadrons.org%3E%0A%20%3C20240403191857.JgiiuZHl%40breakpoint.cc%3E&In-Reply-To=%3C20240403191857.JgiiuZHl%40breakpoint.cc%3E&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor&body=On%20Wed%2C%203%20Apr%202024%2021%3A18%3A57%20%2B0200%20Sebastian%20Andrzej%20Siewior%20%3Csebastian%40breakpoint.cc%3E%20wrote%3A%0A%3E%20On%202024-04-02%2014%3A34%3A20%20%5B%2B0200%5D%2C%20Guillem%20Jover%20wrote%3A%0A%3E%20%3E%20%28请%20不要%20把%20这封%20邮件%20当成%20支持%20任何%20具体%20操作%20的%20背书%2C%20只%0A%3E%20%3E%20是%20想%20澄清%2F%20纠正%20上述%20内容。%29%0A%3E%20%0A%3E%20是%2C%20我%20也%20是%20一样%20的。%20%20%20%205.4.x%20%20系列%20有%20线程化%20解压缩%20%20我%0A%3E%20希望%20保留%20。%20%20%20%205.6.x%20%20系列%20有%20无分支%20解压器%20%0A%3E%20提高%20解压%20性能。%0A%3E%20%0A%3E%20%205.3.x%20%20系列%20是%20alpha%20%20不应%20该%20用于%20生产%20%20而%20仅%0A%3E%20用于%20测试%20。%20我%20不%20确定%20XZ_5.3..alpha%20%20符号%20会%20发生%20什么%0A%3E%20%20我%20希望%20它们%20被%20忽略%20。%0A%3E%20%0A%3E%20如果%20有%20审核%20和%20或%20未来%20计划%20%20我%20将%20联系%20上游%20。%20但%0A%3E%20我%20想%20留%20在%20一个%20官方%20上游%20发布%20%20也%20被%0A%3E%20其他%20发行版%20使用%20。%0A%3E%20%0A%3E%20%3E%20谢谢%0A%3E%20%3E%20Guillem%0A%3E%20%0A%3E%20塞巴斯蒂安%0A%3E%20%0A%3E%20%0A):

```
On 2024-04-02 14:34:20 [+0200], Guillem Jover wrote:
> (Please do not take this mail as endorsing any specific action, just
> wanted to clarify/correct the above.)

Right, same here. The 5.4.x series has threaded decompression which I
would like to keep. The 5.6.x series has branchless decompressor which
improves decompressing performance.

The 5.3.x series is alpha and should not be used in production but only
for testing. I'm not sure what happens with the XZ_5.3..alpha symbols, I
hope they get ignored.

I'm going to poke upstream if there is an audit and or future plans. But
I want to stay on an official upstream release which is also used by
other distros.

> Thanks,
> Guillem

Sebastian

```

* * *

**信息转发**至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`; 软件包 `xz-utils`. (Mon, 08 Apr 2024 13:39:02 GMT) ([全文](bugreport.cgi?bug=1068024;msg=144), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=144), [链接](#143)).

* * *

**已发送确认**至`ashim gena <ashimgena12@gmail.com>`：

额外信息已收到并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Mon, 08 Apr 2024 13:39:02 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=146), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=146), [链接](#145)).

* * *

[消息 #147](#147) 已收到，发送至 1068024@bugs.debian.org ([完整内容](bugreport.cgi?bug=1068024;msg=147), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=147), [回复](mailto:1068024@bugs.debian.org?References=%3CCALL1NVj2v%2BLtmfX7Hw6eXrB%2BDQEd9f9kDucwdgcjoTKtYh5SHg%40mail.gmail.com%3E&In-Reply-To=%3CCALL1NVj2v%2BLtmfX7Hw6eXrB%2BDQEd9f9kDucwdgcjoTKtYh5SHg%40mail.gmail.com%3E&body=On%20Mon%2C%208%20Apr%202024%2010%3A36%3A19%20%2B0300%20ashim%20gena%20%3Cashimgena12%40gmail.com%3E%20wrote%3A%0A%3E%20On%20Fri%2C%2029%20Mar%202024%2022%3A32%3A23%20%2B0100%20Aurelien%20Jarno%20%3Caurelien%40aurel32.net%3E%0A%3E%20wrote%3A%0A%3E%20%3E%20On%202024-03-29%2016%3A25%2C%20Joey%20Hess%20wrote%3A%0A%3E%20%3E%20%3E%20I%27d%20suggest%20reverting%20to%205.3.1.%20Bearing%20in%20mind%20that%20there%20were%20security%0A%3E%20%3E%20%3E%20fixes%20after%20that%20point%20for%20ZDI-CAN-16587%20that%20would%20need%20to%20be%0A%3E%20reapplied.%0A%3E%20%3E%0A%3E%20%3E%20Note%20that%20reverted%20to%20such%20an%20old%20version%20will%20break%20packages%20that%20use%0A%3E%20%3E%20new%20symbols%20introduced%20since%20then.%20From%20a%20quick%20look%2C%20this%20is%20at%20least%3A%0A%3E%20%3E%20-%20dpkg%0A%3E%20%3E%20-%20erofs-utils%0A%3E%20%3E%20-%20kmod%0A%3E%20%3E%0A%3E%20%3E%20Having%20dpkg%20in%20that%20list%20means%20that%20such%20downgrade%20has%20to%20be%20planned%0A%3E%20%3E%20carefully.%0A%3E%20%3E%0A%3E%20%3E%20Regards%0A%3E%20%3E%20Aurelien%0A%3E%20%3E%0A%3E%20%3E%20--%0A%3E%20%3E%20Aurelien%20Jarno%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20GPG%3A%204096R%2F1DDD8C9B%0A%3E%20%3E%20aurelien%40aurel32.net%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20http%3A%2F%2Faurel32.net%0A%3E%20%0A%3E%20%0A%3E%20mobile%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor)):

```
[Message part 1 (text/plain, inline)]
```

```
On Fri, 29 Mar 2024 22:32:23 +0100 Aurelien Jarno <aurelien@aurel32.net>
wrote:
> On 2024-03-29 16:25, Joey Hess wrote:
> > I'd suggest reverting to 5.3.1\. Bearing in mind that there were security
> > fixes after that point for ZDI-CAN-16587 that would need to be
reapplied.
>
> Note that reverted to such an old version will break packages that use
> new symbols introduced since then. From a quick look, this is at least:
> - dpkg
> - erofs-utils
> - kmod
>
> Having dpkg in that list means that such downgrade has to be planned
> carefully.
>
> Regards
> Aurelien
>
> --
> Aurelien Jarno                          GPG: 4096R/1DDD8C9B
> aurelien@aurel32.net                     http://aurel32.net

mobile

```

```
[Message part 2 (text/html, inline)]
```

* * *

**信息已转发** 至 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; Package `xz-utils`. (Fri, 19 Apr 2024 23:09:03 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=149), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=149), [链接](#148)).

* * *

**确认已发送** 给 `Christoph Anton Mitterer <calestyo@scientia.org>`:

额外信息已收到并转发至列表。副本已发送至 `Jonathan Nieder <jrnieder@gmail.com>`。 (Fri, 19 Apr 2024 23:09:03 GMT) ([完整内容](bugreport.cgi?bug=1068024;msg=151), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=151), [链接](#150)).

* * *

[消息 #152](#152)收到于1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=152), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=152), [回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%09%20%3C17377f1e2560f2127e9c896bb87d9941e1b7483e.camel%40scientia.org%3E%0A%09%20%3CPine.BSM.4.64L.2403310145330.25541%40herc.mirbsd.org%3E%0A%20%3Cc5d757b94a82aba1c46eeb944744418c43ad96e5.camel%40scientia.org%3E&In-Reply-To=%3Cc5d757b94a82aba1c46eeb944744418c43ad96e5.camel%40scientia.org%3E&body=On%20Sat%2C%2020%20Apr%202024%2000%3A54%3A40%20%2B0200%20Christoph%20Anton%20Mitterer%20%3Ccalestyo%40scientia.org%3E%20wrote%3A%0A%3E%20Hey.%0A%3E%20%0A%3E%20%0A%3E%20On%20Sun%2C%202024-03-31%20at%2001%3A46%20%2B0000%2C%20Thorsten%20Glaser%20wrote%3A%0A%3E%20%3E%20Yes%2C%20a%20multi-team%20task%20force%20is%20working%20on%20it%20and%20will%20inform%20users%0A%3E%20%3E%20once%20it%20is%20known%20how%20to%20proceed%2C%20inclusing%20how%20much%20to%20throw%20away%0A%3E%20%3E%20and%20rebuild.%0A%3E%20%0A%3E%20Kindly%20wanted%20to%20ask%20whether%20anything%20has%20come%20out%20meanwhile%20of%20that%3F%0A%3E%20%0A%3E%20%0A%3E%20I%27ve%20tried%20to%20follow%20quite%20extensively%20what%20the%20various%20reverse%0A%3E%20engineering%20efforts%20%28e.g.%20%5B0%5D%2C%20%5B1%5D%2C%20%5B2%5D%2C%20%5B3%5D%29%20found%20out%20or%20what%27s%0A%3E%20revealed%20on%20index%20pages%20like%20%5B4%5D%20or%20%5B5%5D.%0A%3E%20%0A%3E%20It%20feels%20as%20if%20there%20are%20still%20many%20discussions%20about%20how%20to%20prevent%0A%3E%20such%20things%20in%20the%20future%2C%20but%20less%20so%20about%20the%20concrete%20fallout%20of%0A%3E%20the%20particular%20backdoor%2C%20where%20it%20seems%20most%20people%20were%20lead%20to%0A%3E%20conclude%20from%20media%20reports%2C%20that%20an%20attack%20was%20only%20possible%20if%20sshd%0A%3E%20was%20actually%20running%20an%20reachable.%0A%3E%20%0A%3E%20This%20may%20of%20course%20be%20true%2C%20which%20would%20mean%20that%20most%20people%20are%0A%3E%20actually%20safe%20and%20we%20had%20quite%20some%20luck%20this%20time%3A%0A%3E%20-%20servers%2C%20because%20they%20run%20stable%20distros%20that%20haven%27t%20had%20the%0A%3E%20%20%20backdoor%0A%3E%20-%20workstations%2Flaptops%2C%20because%20they%20typically%20don%27t%20run%20a%20publicly%0A%3E%20%20%20listending%20sshd%0A%3E%20%0A%3E%20But%20there%20are%20still%20new%20findings%20about%20the%20backdoor%20every%20now%20and%20then%2C%0A%3E%20like%20that%20it%20may%20read%2Fwrite%20on%20IPC%20sockets%20%28contained%20in%20%5B2%5D%29%20and%20I%27ve%0A%3E%20read%20similar%5B6%5D%20without%20the%20restriction%20on%20IPC.%0A%3E%20Also%20I%27ve%20seen%20some%20vague%20statements%5B7%5D%20that%20it%20might%20%22install%22%20public%0A%3E%20keys%20%28didn%27t%20really%20grasp%20what%20was%20meant%20there%20-%20something%20like%20%22in%0A%3E%20authorized_keys%22%29.%20And%20one%20report%5B8%5D%20talked%20about%20it%20collecting%0A%3E%20usernames%20and%20IPs%20and%20passing%20the%20on%20to%20some%20function%20with%20unknown%0A%3E%20purpose.%0A%3E%20%0A%3E%20It%20also%20seems%20like%20these%20effort%20focus%20mostly%20on%20the%205.6.1%20version%20and%0A%3E%20while%20it%27s%20said%20that%20the%205.6.0%20version%20is%20quite%20similar%2C%20who%20knows%20the%0A%3E%20exact%20details%21%3F%0A%3E%20%0A%3E%20%0A%3E%20In%20any%20case%20and%20%28too%29%20long%20story%20short%3A%0A%3E%20%0A%3E%20It%20would%20be%20nice%20to%20know%20whether%20there%27s%20still%20work%20done%20about%20finding%0A%3E%20out%20whether%20people%20who%20had%20the%20malicious%20code%20on%20their%20systems%20%28in%20any%0A%3E%20version%20of%20the%20backdoor%29%2C%20but%0A%3E%20-%20had%20sshd%20not%20running%20at%20all%0A%3E%20and%2For%0A%3E%20-%20it%20was%20not%20reachable%20from%20the%20internet%0A%3E%20%0A%3E%20can%20feel%20safe.%0A%3E%20%0A%3E%20Or%20whether%20it%20may%20be%20possible%20that%3A%0A%3E%20-%20the%20backdoor%20did%20call%20home%20%28loaded%20commands%20from%20

```
Hey.

On Sun, 2024-03-31 at 01:46 +0000, Thorsten Glaser wrote:
> Yes, a multi-team task force is working on it and will inform users
> once it is known how to proceed, inclusing how much to throw away
> and rebuild.

Kindly wanted to ask whether anything has come out meanwhile of that?

I've tried to follow quite extensively what the various reverse
engineering efforts (e.g. [0], [1], [2], [3]) found out or what's
revealed on index pages like [4] or [5].

It feels as if there are still many discussions about how to prevent
such things in the future, but less so about the concrete fallout of
the particular backdoor, where it seems most people were lead to
conclude from media reports, that an attack was only possible if sshd
was actually running an reachable.

This may of course be true, which would mean that most people are
actually safe and we had quite some luck this time:
- servers, because they run stable distros that haven't had the
  backdoor
- workstations/laptops, because they typically don't run a publicly
  listending sshd

But there are still new findings about the backdoor every now and then,
like that it may read/write on IPC sockets (contained in [2]) and I've
read similar[6] without the restriction on IPC.
Also I've seen some vague statements[7] that it might "install" public
keys (didn't really grasp what was meant there - something like "in
authorized_keys"). And one report[8] talked about it collecting
usernames and IPs and passing the on to some function with unknown
purpose.

It also seems like these effort focus mostly on the 5.6.1 version and
while it's said that the 5.6.0 version is quite similar, who knows the
exact details!?

In any case and (too) long story short:

It would be nice to know whether there's still work done about finding
out whether people who had the malicious code on their systems (in any
version of the backdoor), but
- had sshd not running at all
and/or
- it was not reachable from the internet

can feel safe.

Or whether it may be possible that:
- the backdoor did call home (loaded commands from there, leaked
  private keys or so from the system)
- used completely different vectors not involving sshd
- or somehow else infested the system

Right now people might still have backups to torch their possibly
compromised systems and start over from a safe sate.

So Thorsten, in case you or someone else is aware of any [intermediate]
results from these task forces ([9[) it would be nice to hear about
them or better even in form of some "official" statement from Debian.

Thanks,
Chris.

[0] https://discord.gg/u6MzmQm5
[1] https://github.com/smx-smx/xzre
[2] https://github.com/binarly-io/binary-risk-intelligence/tree/master/xz-backdoor
[3] https://securelist.com/xz-backdoor-story-part-1/112354/
[4] https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27
[5] https://github.com/przemoc/xz-backdoor-links
    https://przemoc.github.io/xz-backdoor-links/  (rendering of that)
[6] https://discord.com/channels/1223666474091020432/1223666474972090430/1230974749522530304
[7] https://discord.com/channels/1223666474091020432/1223666474972090430/1230173131746840606
[8] https://isc.sans.edu/diary/30802
[9] E.g. on d-d https://lists.debian.org/debian-devel/2024/03/msg00338.html
    Moritz Mühlenhoff has mentioned that some company was working on it
    and results were expected in some time.

```

* * *

**信息已转发**至`debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`：

`Bug#1068024`；软件包`xz-utils`。(Sat, 20 Apr 2024 09:57:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=154), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=154), [链接](#153)).

* * *

已发送**确认函**至`Boud Roukema <bouddebbug@cosmo.torun.pl>`：

将额外信息接收并转发至列表。副本已发送至`Jonathan Nieder <jrnieder@gmail.com>`。(Sat, 20 Apr 2024 09:57:03 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=156), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=156), [链接](#155)).

* * *

[消息 #157](#157) 已接收于 1068024@bugs.debian.org ([完整文本](bugreport.cgi?bug=1068024;msg=157), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=157), [回复](mailto:1068024@bugs.debian.org?In-Reply-To=%3C46428167-0bfe-235d-efc8-da96c802973a%40cosmo.torun.pl%3E&References=%3C46428167-0bfe-235d-efc8-da96c802973a%40cosmo.torun.pl%3E&body=On%20Sat%2C%2020%20Apr%202024%2011%3A49%3A10%20%2B0200%20%28CEST%29%20Boud%20Roukema%20%3Cbouddebbug%40cosmo.torun.pl%3E%20wrote%3A%0A%3E%20hi%20all%0A%3E%20%0A%3E%20Leaving%20aside%20the%20questions%20of%20dependencies%20of%20other%20Debian%20packages%2C%0A%3E%20I%20agree%20that%205.2.5%20looks%20like%20the%20most%20recent%20version%20that%20is%0A%3E%20definitely%20free%20of%20any%20directly%20or%20indirectly%20authored%20contributions%0A%3E%20by%20Jia%20Tan.%0A%3E%20%0A%3E%20https%3A%2F%2Fsalsa.debian.org%2Fdebian%2Fxz-utils%2F-%2Ftree%2Fv5.2.5%3Fref_type%3Dtags%0A%3E%20%0A%3E%20In%20the%20salsa%20full%20source%3A%0A%3E%20%0A%3E%20%24%20git%20checkout%20v5.2.5%0A%3E%20Previous%20HEAD%20position%20was%20d24a57b7%20Bump%20version%20and%20soname%20for%205.2.7.%0A%3E%20HEAD%20is%20now%20at%202327a461%20Bump%20version%20and%20soname%20for%205.2.5.%0A%3E%20%0A%3E%20%24%20git%20log%20--stat%20--graph%20%7Cgrep%20%22Jia%22%0A%3E%20%0A%3E%20%23%20No%20sign%20of%20Jia%20Tan.%0A%3E%20%0A%3E%20%24%20git%20checkout%20v5.2.6%0A%3E%20Previous%20HEAD%20position%20was%202327a461%20Bump%20version%20and%20soname%20for%205.2.5.%0A%3E%20HEAD%20is%20now%20at%208dfed05b%20Bump%20version%20and%20soname%20for%205.2.6.%0A%3E%20%0A%3E%20%24%20git%20log%20--stat%20--graph%20%7Cgrep%20%22Jia%22%20%7Cwc%0A%3E%20%20%20%20%20%20%2016%20%20%20%20%20129%20%20%20%20%20774%0A%3E%20%0A%3E%20%23%20Two%20commits%20and%20several%20%27Thanks%20to%20...%20%27.%0A%3E%20%0A%3E%20Cheers%0A%3E%20Boud%0A%3E%20%0A%3E%20%0A%3E%20PS%3A%20For%20any%20RedHat%20people%20reading%20this%20thread%3A%20unfortunately%2C%205.2.5%20is%0A%3E%20before%20the%20fix%2031d80c6b%20that%20Lasse%20Collin%20did%20to%20handle%20the%20%22ill%20patch%22%0A%3E%20introduced%20by%20RHEL%2FCentOS7%3A%0A%3E%20%0A%3E%20%20%20%20https%3A%2F%2Fsalsa.debian.org%2Fdebian%2Fxz-utils%2F-%2Fcommit%2F31d80c6b261b24220776dfaeb8a04f80f80e0a24%0A%3E%20%0A%3E%20That%27s%20a%20RHEL%20problem%20to%20handle%2C%20not%20a%20Debian%20problem.%0A%3E%20%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20latest%20Jia-Tan-free%20xz%20version%20seems%20to%20be%205.2.5)):

```
hi all

Leaving aside the questions of dependencies of other Debian packages,
I agree that 5.2.5 looks like the most recent version that is
definitely free of any directly or indirectly authored contributions
by Jia Tan.

https://salsa.debian.org/debian/xz-utils/-/tree/v5.2.5?ref_type=tags

In the salsa full source:

$ git checkout v5.2.5
Previous HEAD position was d24a57b7 Bump version and soname for 5.2.7.
HEAD is now at 2327a461 Bump version and soname for 5.2.5.

$ git log --stat --graph |grep "Jia"

# No sign of Jia Tan.

$ git checkout v5.2.6
Previous HEAD position was 2327a461 Bump version and soname for 5.2.5.
HEAD is now at 8dfed05b Bump version and soname for 5.2.6.

$ git log --stat --graph |grep "Jia" |wc
     16     129     774

# Two commits and several 'Thanks to ... '.

Cheers
Boud

PS: For any RedHat people reading this thread: unfortunately, 5.2.5 is
before the fix 31d80c6b that Lasse Collin did to handle the "ill patch"
introduced by RHEL/CentOS7:

  https://salsa.debian.org/debian/xz-utils/-/commit/31d80c6b261b24220776dfaeb8a04f80f80e0a24

That's a RHEL problem to handle, not a Debian problem.

```

* * *

**信息已转发** 给 `debian-bugs-dist@lists.debian.org, Jonathan Nieder <jrnieder@gmail.com>`:

`Bug#1068024`; 软件包 `xz-utils`. (Mon, 22 Apr 2024 23:51:02 GMT) ([完整文本](bugreport.cgi?bug=1068024;msg=159), [mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=159), [链接](#158)).

* * *

**致谢已发送** 给 `Thorsten Glaser <tg@mirbsd.de>`:

额外信息已收到并转发至列表。副本发送至`Jonathan Nieder <jrnieder@gmail.com>`。（Mon, 22 Apr 2024 23:51:02 GMT）（[全文](bugreport.cgi?bug=1068024;msg=161)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=161)，[链接](#160)）

* * *

[消息 #162](#162) 接收自 1068024@bugs.debian.org（[全文](bugreport.cgi?bug=1068024;msg=162)，[mbox](bugreport.cgi?bug=1068024;mbox=yes;msg=162)，[回复](mailto:1068024@bugs.debian.org?References=%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%20%20%3C17377f1e2560f2127e9c896bb87d9941e1b7483e.camel%40scientia.org%3E%0A%20%20%3CPine.BSM.4.64L.2403310145330.25541%40herc.mirbsd.org%3E%20%3CZgcjtvSjQM59nX_w%40kitenet.net%3E%0A%20%3Cc5d757b94a82aba1c46eeb944744418c43ad96e5.camel%40scientia.org%3E%0A%20%3CPine.BSM.4.64L.2404222342120.12579%40herc.mirbsd.org%3E&In-Reply-To=%3CPine.BSM.4.64L.2404222342120.12579%40herc.mirbsd.org%3E&body=On%20Mon%2C%2022%20Apr%202024%2023%3A43%3A21%20%2B0000%20%28UTC%29%20Thorsten%20Glaser%20%3Ctg%40mirbsd.de%3E%20wrote%3A%0A%3E%20Christoph%20Anton%20Mitterer%20dixit%3A%0A%3E%20%0A%3E%20%3ESo%20Thorsten%2C%20in%20case%20you%20or%20someone%20else%20is%20aware%20of%20any%20%5Bintermediate%5D%0A%3E%20%3Eresults%20from%20these%20task%20forces%20%28%5B9%5B%29%20it%20would%20be%20nice%20to%20hear%20about%0A%3E%20%3Ethem%20or%20better%20even%20in%20form%20of%20some%20%22official%22%20statement%20from%20Debian.%0A%3E%20%0A%3E%20JFTR%20I%E2%80%99m%20not%20involved%20in%20those%20myself.%0A%3E%20%0A%3E%20bye%2C%0A%3E%20%2F%2Fmirabilos%0A%3E%20--%20%0A%3E%20When%20he%20found%20out%20that%20the%20m68k%20port%20was%20in%20a%20pretty%20bad%20shape%2C%20he%20did%0A%3E%20not%2C%20like%20many%20before%20him%2C%20shrug%20and%20move%20on%3B%20instead%2C%20he%20took%20it%20upon%0A%3E%20himself%20to%20start%20compiling%20things%2C%20just%20so%20he%20could%20compile%20his%20shell.%0A%3E%20How%27s%20that%20for%20dedication.%20--%20Wouter%2C%20about%20my%20Debian%2Fm68k%20revival%0A%3E%20%0A%3E%20%0A&subject=Re%3A%20Bug%231068024%3A%20revert%20to%20version%20that%20does%20not%20contain%20changes%20by%0A%20bad%20actor)):

```
Christoph Anton Mitterer dixit:

>So Thorsten, in case you or someone else is aware of any [intermediate]
>results from these task forces ([9[) it would be nice to hear about
>them or better even in form of some "official" statement from Debian.

JFTR I’m not involved in those myself.

bye,
//mirabilos
-- 
When he found out that the m68k port was in a pretty bad shape, he did
not, like many before him, shrug and move on; instead, he took it upon
himself to start compiling things, just so he could compile his shell.
How's that for dedication. -- Wouter, about my Debian/m68k revival

```

* * *

发送一份报告，说明[此错误日志包含垃圾邮件](https://bugs.debian.org/cgi-bin/bugspam.cgi?bug=1068024)。

* * *

Debian漏洞跟踪系统管理员 <[owner@bugs.debian.org](mailto:owner@bugs.debian.org)>。最后修改时间：Wed May 29 04:48:43 2024；机器名称：bembo

[Debian漏洞跟踪系统](https://www.debian.org/Bugs/)

Debbugs是自由软件，根据GNU公共许可证第2版许可。可以从[https://bugs.debian.org/debbugs-source/](https://bugs.debian.org/debbugs-source/)获取当前版本。

版权 © 1999 Darren O. Benham, 1997,2003 nCipher Corporation Ltd, 1994-97 Ian Jackson, 2005-2017 Don Armstrong和许多其他贡献者。
