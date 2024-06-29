<!--yml

category: 未分类

date: 2024-05-27 12:56:43

-->

# RubyGems 不受 xz/liblzma 后门影响 - RubyGems Blog

> 来源：[https://blog.rubygems.org/2024/03/31/rubygems-and-xz.html](https://blog.rubygems.org/2024/03/31/rubygems-and-xz.html)

过去几天，安全领域的关注点集中在[xz/liblzma后门](https://xeiaso.net/notes/2024/xz-vuln/)的揭示上。更多背景信息，请参阅[此问题的早期报告](https://xeiaso.net/notes/2024/xz-vuln/)，[这篇 GitHub Gist](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27)，[这个详细的时间线](https://boehs.org/node/everything-i-know-about-the-xz-backdoor)，以及[CVE-2024-3094](https://nvd.nist.gov/vuln/detail/CVE-2024-3094)的官方详细页面。

针对此后门被公开的情况，我们不仅对运行 RubyGems.org 本身的软件进行了内部审计，还对曾经发布的每一个 gem 进行了审计。

我们很高兴地宣布，RubyGems.org 并不受此问题的影响。此外，我们很高兴地确认，目前在 RubyGems.org 上发布的任何 gem 都不包含有漏洞的`liblzma`库。

我要感谢 RubyGems.org 安全团队的支持，他们在此次调查中提供了帮助，并持续致力于生态系统的安全。我还要感谢 AWS 对 RubyGems 安全的持续支持，包括他们赞助我作为[Ruby Central的驻场安全工程师](https://rubycentral.org/news/ruby-central-welcomes-new-software-engineer-in-residence-sponsored-by-aws/)，以及资助[rubygems-research](https://github.com/segiddins/rubygems-research)项目。

多亏了该项目中的数据（公开可用于[research.rubygems.info](https://research.rubygems.info)），我们能够迅速确认目前已发布的 gem 中没有任何与有漏洞的`liblzma`库有关的内容。

### 技术细节

RubyGems.org 应用容器是通过[Dockerfile构建](https://github.com/rubygems/rubygems.org/blob/master/Dockerfile)，不包含有漏洞的`liblzma`或`xz`。我们的镜像基于[Alpine 3.18 stable](https://github.com/rubygems/rubygems.org/blob/master/Dockerfile#L5)，该版本从未包含或访问有漏洞的库版本。此外，Alpine Linux 使用`musl` libc，不包含(glibc-only)[IFUNC机制](https://sourceware.org/glibc/wiki/GNU_IFUNC)，该机制用于激活后门。

我们部署到生产环境的容器构建过程是公开的，每次提交到公共 RubyGems.org 代码库后都会通过 GitHub Actions 运行。任何感兴趣的方可以使用[最新的容器构建作业日志](https://github.com/rubygems/rubygems.org/actions/runs/8498544592/job/23278360812)来重现构建过程。

通过在我们构建的容器中运行`find / -name '*lzma*'`验证，我们仅依赖于版本5.4.3，而不依赖于有漏洞的版本5.6.0或5.6.1。

<details><summary>完整的命令输出</summary>

```
$ find / -name '*lzma*'
/usr/bin/lzma
/usr/bin/unlzma
/usr/lib/liblzma.so.5
/usr/lib/liblzma.so.5.4.3
/app/vendor/ruby/3.3.0/gems/bindata-2.5.0/lib/bindata/transform/lzma.rb

$ find / -name '*xz*'
/sys/module/xz_dec
/usr/bin/unxz
/usr/bin/xzcat
/app/vendor/ruby/3.3.0/gems/bindata-2.5.0/lib/bindata/transform/xz.rb 
```</details>

截至2024年3月31日，RubyGems.org 上唯一包含 `liblzma` 的宝石是名为 [liblzma 的宝石](https://rubygems.org/gems/liblzma)。该宝石仅包含库的版本0.2和0.3，这两个版本都不包含后门。没有包含 `xz` 命令行工具的宝石。

<details><summary>完整的命令输出</summary>

```
 irb(main):005> attrs = ['version_data_entries.full_name', 'rubygems.name',  'versions.number', 'versions.platform', 'versions.uploaded_at']
=> ["version_data_entries.full_name", "rubygems.name", "versions.number", "versions.platform", "versions.uploaded_at"]
irb(main):006> VersionDataEntry.where('version_data_entries.name LIKE ?', 'liblzma%.so').joins(:version, :rubygem).pluck(*attrs).map { |p| attrs.zip(p).to_h }
=>
[{"version_data_entries.full_name"=>"lib/liblzma.so", "rubygems.name"=>"liblzma", "versions.number"=>"0.2", "versions.platform"=>"mingw32", "versions.uploaded_at"=>Sat, 31 Mar 2012 05:57:47.212691000 UTC +00:00},
 {"version_data_entries.full_name"=>"lib/1.9.1/liblzma.so",
  "rubygems.name"=>"liblzma",
  "versions.number"=>"0.3",
  "versions.platform"=>"x86-mingw32",
  "versions.uploaded_at"=>Thu, 21 Feb 2013 13:21:51.961608000 UTC +00:00},
 {"version_data_entries.full_name"=>"lib/2.0.0/liblzma.so",
  "rubygems.name"=>"liblzma",
  "versions.number"=>"0.3",
  "versions.platform"=>"x86-mingw32",
  "versions.uploaded_at"=>Thu, 21 Feb 2013 13:21:51.961608000 UTC +00:00}]

irb(main):008> VersionDataEntry.where('version_data_entries.name = ?', 'xz').joins(:version, :rubygem).pluck(*attrs).map { |p| attrs.zip(p).to_h }
=> [] 
```</details>
