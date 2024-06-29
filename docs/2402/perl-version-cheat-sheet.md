<!--yml

类别：未分类

日期：2024-05-27 14:34:27

-->

# Perl版本速查表

> 来源：[https://sheet.shiar.nl/perl/](https://sheet.shiar.nl/perl/)

# Perl发布摘要

最近Perl脚本语言的重要功能介绍。

核心安全支持提供了3年，因此典型用户应至少运行[5.36](#v5036)。稳定的发行版如Debian 10保持[5.28](#v5028)+。企业平台保留版本至[5.10](#v5010)。

## 5.38 2023-07-02：最新的稳定发布

`use feature "module_true"`

默认在5.37及更高版本中使用，也有`no feature "bareword_filehandles"`

`sub ($var ||= default)`

在`//=`时赋值为假（或未定义的情况）而不是省略

`/(*{ … })/`

乐观的评估：启用正则表达式优化的`(?{ … })`

`class`

定义对象类：具有`field`变量和`method`子程序的包 *(功能，实验性)*

`${^LAST_<wbr>SUCCESSFUL_<wbr>PATTERN}`

明确变量用于访问前一个匹配，如`s//…/`

`%{^HOOK}`

调用`require`时执行任务`require__before`和`require__after`

Unicode

v15.0

## 5.36 2022-05-28：活跃的核心支持

`use v5.36`

`use warnings`; use feature qw'`signatures isa`'; no feature qw'`indirect multidimensional switch`'

`use builtin`

解释器函数的命名空间，例如来自`Scalar::Util`的`weaken`和`blessed`，来自`POSIX`的`ceil`/`floor`，以及类似于`String::Util`的`trim` *(实验性)*

`is_bool(!0)`

区分标量变量类型（通过`builtin`函数）以实现数据互操作性

`for my ($k, $v) (%hash)`

一次迭代多个值（包括数组的`builtin::indexed`） *(功能，实验性)*

`defer {}`

在作用域结束时执行排队的代码 *(功能，实验性)*

`try {} finally {}`

无论失败与否，在`[try](#try)`结构的末尾运行代码 *(功能，实验性)*

用于引用运算符的Unicode分隔符 *(实验性)*

`sub ($var) {!~~pop~~}`

[签名](#signatures)的子程序是稳定的，但与参数数组`@_`混合仍然是实验性 *(功能，实验性)*

`$SIG{FPE}`

浮点异常不再延迟，而是立即交付，如其他信号

perl `-g`

禁用输入记录分隔符（吞吐模式），别名为`-0777`

Unicode

v14.0

## 5.34 2021-05-20

`try {} catch`

类似于eval块的异常处理 *(功能，实验性)*

`/{,*n*}/`

空下限量词被接受作为0的简写

`\x{ … }`

曲折括号内的微不足道的空格，也适用于`\b{}`、`\g{}`、`\k{}`、`\N{}`、`\o{}`以及`/{m,n}/`量词

`0o0`

八进制前缀`0o`替代`0…`和`oct`

`re::optimization(qr//)`

调试正则表达式优化信息在编译时发现

`no feature …`

禁用不推荐的实践，如`bareword_filehandles`和多维数组仿真

## 5.32 2020-06-20

`isa`

用于检查类实例的中缀运算符 *(功能，直至5.36仍然是实验性)*

`$min < $_ <= $max`

链式比较重复内部部分如`$min < $_ and $_ <= $max`

`/\p{Name=$var}/`

匹配Unicode Name属性如`\N{}`，但带插值和子模式

`open F, '+>>', undef`

尊重临时文件上的追加模式混合访问

`no feature 'indirect'`

禁用间接对象表示法，例如`new Class`而不是`Class->new`

streamzip

程序与核心IO::Compress::Base一起分发，将标准输入压缩为zip容器

Unicode

v13.0

## 5.30 2019-05-22

`/(?<=var+)`

变长后顾断言 *(直到5.36版本仍在试验阶段)*

`m(\p{nv=/.*/})`

通过正则表达式匹配Unicode属性 *(实验性内容)*

`~~my $state if 0~~`

对于 `[state](#state)` 的问题，自5.10版本起已被弃用，现在被禁止

`qr'\N'`

分界符必须是字形；未转义的`{`是非法的；单引号中的`\N`

Unicode

v12.1

## 5.28 2018-06-22：由通用供应商继续维护

`delete %hash{…}`

哈希切片可以通过键+值对删除

`/(*…)/`

断言的字母同义词，例如`(*atomic:…)`代替`(?>…)`和`(*nlb:…)`代替`(?<!…)` *(直到5.31.6仍在试验阶段)*

`/(*script_run:)/`

强制所有字符来自同一脚本 *(直到5.31.6仍在试验阶段)*

`state @a`

持久的词法数组或哈希变量（除了[标量](#state)）

`perl -i -pe die`

安全的原地编辑：仅在成功完成后替换文件

`${^SAFE_LOCALES}`

在支持的系统上，区域设置是线程安全的，由此变量指示

Unicode

v10.0

## 5.26 2017-05-30

`<<~EOT`

缩进的Here文档，在每行的定界符之前去除相同的空白字符

`@{^CAPTURE}`

上次匹配的捕获数组，因此`${^CAPTURE}[0]`就是`$1`

`//xx`

扩展修饰符，也忽略括号字符类中的空白

`use Test2::V0`

用于替换`Test::*`和`TAP::*`的通用测试框架

Unicode

v9.0

## 5.24 2016-05-09

`printf '%.*2$x'`

重新排序的精度参数

`/\b{lb}/`

适合连字符的换行边界类型

`/faster/`

各种显著的加速，特别是匹配固定子字符串，对大小写不敏感语言的`/i`，64位算术，作用域开销

Unicode

v8.0

## 5.22 2015-06-01

`\$alias =`

通过引用进行别名处理（截至v5.25.3为止） *(实验性内容)*

`<<>>`

安全地忽略参数中的打开标志`readline`

`/()/n`

禁用带编号捕获，将`()`转换为`(?:)`

`/\b{}/`

边界类型：*gcb*（字符簇），*sb*（句子），*wb*（单词）

`&.`

`& | ^ ~`始终用于字符串的数值点操作符 *(功能，直到5.28仍在试验阶段)*

`use re 'strict'`

对正则表达式模式应用更严格的语法规则 *(实验性内容)*

`0x.beep+0`

具有二进制幂的十六进制浮点表示法；`printf '%a'`用于显示

`~~??~~`

单一匹配简写（自5.14版本起已弃用）需要运算符`*m*?PATTERN?`

`~~use CGI~~`

从核心中移除用于提供http请求的过时接口，请参阅[CGI::Alternatives](https://metacpan.org/pod/CGI::Alternatives)

Unicode

v7.0

## 5.20 2014-05-27：扩展供应商支持至202X

`sub ($var)`

子程序签名 *(功能，直到5.36为实验性)*

`%hash{…}`

哈希切片返回键值对

`[]->@*`

后缀解引用 (例如 `$scalar->$*` 用于 `$$scalar`) *(功能，直到5.23.1为实验性)*

`use warnings 'once'; $a`

变量 $a 和 $b 豁免于 *used once* 警告

Unicode

v6.3

## 5.18 2013-05-18

`${^LAST_FH}`

最后读取文件句柄 (被 `$.` 使用)

`/(?[ a + b ])/`

正则表达式集操作 (字符减法 `-`, 联合 `+`, 交集 `&`, 异或 `^`) *(直到5.36为实验性)*

`my sub`

词法子程序 (还有 `state`, `our`); v5.22 之前存在问题 *(直到5.26为实验性)*

`next $expression`

循环控制允许运行时表达式

`no warnings 'experimental::…'`

实验性特性机制，目前必须用于 *智能匹配*

Unicode

v6.2

## 5.16 2012-05-20

`__SUB__`

当前子程序引用 *(功能)*

`fc, "\F"`

Unicode 折叠大小写以不区分大小写比较 *(功能)*

`"\N{}"`

自动 `use charnames qw( :full :short )`

Unicode

v6.1

## 5.14 2011-05-14

`s///r`

非破坏性替换

`/(?{ m() })/`

正则表达式可以嵌套在 `/(?{})/` 和 `/(??{})/` 中 *(直到5.20为实验性)*

`/dalu`

正则表达式修饰符以限制字符类：默认 **d**、**a**scii、**l**ocale 或 **u**nicode 语义。

`use re '/flags'`

自定义默认修饰符

`/(?^)/`

用于重置默认修饰符的构造

`FH->method`

文件句柄方法调用根据需要加载 IO::File (例如 `STDOUT->flush`)

`\o{}`

八进制值的转义序列超出 \777

`use JSON`

与 JavaScript 对象表示法中的数据接口 {`decode_json <>`}

Unicode

v6.0+#8

## 5.12 2010-04-12

`package` 版本

`package` NAME VERSION 缩写为 `our $VERSION`

`...`

哑元操作符：代码占位符

`use 5.012`

如果使用版本 >= v5.12 隐式 `strict`

`… when`

`when` 现在可以用作语句修饰符

`use overload 'qr'`

可定制的正则表达式转换

`/\N/`

反斜杠 \n 匹配除换行符外的任何字符 *(直到5.18为实验性)*

Unicode

v5.2

## 5.10 2007-12-18: 商业支持直到 2024

`//`

定义或运算符

`~~`

智能匹配操作符用于比较不同数据类型 (在v5.10.1中更新) *(实验性)*

`say`

带换行符打印，等同于 `print @_, "\n"` *(功能)*

`given`

使用 `when`/`default` 进行智能匹配的 switch 语句 *(功能，实验性)*

`/(?<name>)/`

命名捕获缓冲区为 `%+`

`/(?1)/`

递归正则表达式模式

`/(?|)/`

重置每个包含分支的捕获编号

`/.++/`

占有量词 `?+`, `*+`, `++` 贪婪匹配

`s/keep\K//`

浮动正向回溯，用于 `s/(keep)/$1/` 的有效替代

`/p`

可选择保留 `${^MATCH}` 变量 (避免在v5.20之前的COW中的 `$&` 惩罚)

`/\v/, /\h/`

垂直和水平空白字符转义 (`\V` `\H` 反转); 同时 `/\R/` 用于换行符

`state`

持久的 `my` 变量 (仅标量直到 [5.28](#state_ext)) *(功能)*

`use autodie`

替换内置函数以抛出异常而不是返回失败 {`eval {open ...} or $@->matches("open") || die`}

`use IO::Compress::Zip`

各种文件压缩标准 {`zip IO::Uncompress::Gunzip->new("test.gz")`<wbr> `=> "recompressed.zip"`}

`use Time::Piece`

时间戳作为对象 {`localtime->year > 1900`}

`use File::Fetch`

通用数据检索/下载 {`File::Fetch->new(uri => "http://localhost/")`<wbr>`->fetch(to => \$slurp)`}

Unicode

v5.0.0

## 5.8 2002-07-18: 在 20[01]\d 期间的稳定最低版本

`no utf8`

完全支持 Unicode，`utf8` pragma 仅用于脚本编码

`use open`

文件句柄行为通过 PerlIO 层改变 {`binmode $fh, ":bytes"`}

`open $fh, '-|', @cmd`

打开列表以fork一个命令而不产生shell

`open $fh, '>', \$var`

Perl 标量作为虚拟文件

`printf '%1$s', @args`

使用参数无序的语法

`1_2_3 == 123`

数字常量中允许在数字之间使用下划线

`use bignum`

透明的大数支持 {`length 1e100 == 101`}

`use if`

条件模块包含 {`no if $] >= 5.022, "warnings", "redundant"`}

`use Digest`

计算各种消息摘要（数据哈希） {`$hash = sha256_hex($data)`}

`use Encode`

字符集转换 {`encode("utf8", decode("iso-8859-1", $octets))`}

`use File::Temp`

安全创建临时文件或目录 {`$fh = tempfile();`}

`use List::Util`

通用实用列表子例程 {`@cards = shuffle 0..51`}

`use Locale::Maketext`

`Locale::*` 和 `L18N::*` 中的各种本地化和国际化

`use Memoize`

记住函数结果，用空间换时间 {`memoize "stat"`}

`use MIME::Base64`

Base64 编码的字符串如同电子邮件附件

`use Test::More`

现代化的单元测试框架 {`is $got, $expected`}

`use Time::HiRes`

高分辨率定时器 {`$μs = [gettimeofday]; sleep .1;`<wbr> `$elapsed = tv_interval $μs`}

Unicode

v3.2.0

## 5.6 2000-03-23: 现代兼容性的开始

`use warnings`

启用词法作用域中的警告的 pragma

`use utf8`

实验性 Unicode 语义（在 [v5.8](#utf8_data) 中完成） *(实验性功能直到 5.8)*

`use charnames`

字符串转义 `\N{}` 插入命名字符

`our`

声明全局变量

`v1.2.3`

将字符串表示为顺序向量，用于版本号 (`printf '%vd'` 显示)

`0b0`

字面上的二进制数字，`printf '%b'` 和 `oct`

`sub :lvalue`

子程序属性，返回一个可修改的值 *(实验性功能直到 5.20)*

`open my $fh, $mode, $expr`

文件句柄在作用域标量中，第三个参数用于明确的文件名

`pack 'q'`

64位整数支持（也支持大于2GiB的大文件） *(实验性功能直到 5.8.1)*

`sort $coderef ()`

比较函数可以是子例程引用；原型 `($$)` 以正常 `@_` 传递元素

`CHECK {}`

编译结束时调用的特殊块

`/[[:…:]]/`

POSIX 字符类语法，如 `/[[:alpha:]]/`

Unicode

v3.0.1
