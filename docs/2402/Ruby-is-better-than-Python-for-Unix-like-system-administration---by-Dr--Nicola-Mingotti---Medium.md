<!--yml

category: 未分类

date: 2024-05-29 13:23:15

-->

# Ruby is better than Python for Unix-like system administration | by Dr. Nicola Mingotti | Medium

> 来源：[https://nmingotti.medium.com/ruby-is-better-than-python-for-unix-like-system-administration-e520ef41e66d](https://nmingotti.medium.com/ruby-is-better-than-python-for-unix-like-system-administration-e520ef41e66d)

# Ruby 文化是 Unix 文化，也是您的文化

Ruby 是由一位[深受 Perl 和 Unix 影响的人](https://twobithistory.org/2017/11/19/the-ruby-story.html)创造的。这从语言本身可以很清楚地看出来。如果您是 Unix 专业人士，那么以下至少有一半的名称/符号您是了如指掌的：

```
` `, $?, STDIN, STDOUT, STDERR, ${}, ARGV, $$        ...  # shell-isms
%Q, %q, %x, -e, -F, -wnlp, / /, $VAR, @VAR, HEREDOC  ...  # perl-awk-isms
```

所有这些名称/符号在 Ruby 中都在使用，并且它们的含义与您已知的一样（有少数例外，例如用于变量的 $ 和 @）。

这对您证明了什么？Ruby 掌握了您熟悉的语言，并且专为解决您每天面临的问题而设计。对于 Python，情况并非如此，Python 起初是作为教学语言而非管理员工具而生。Python 自带的简单图形化 IDE 对于 Windows 和一般初学者非常好用，但对于 Unix 管理员来说并不立即实用，因为他们大部分工作都在命令行上完成。相反，Ruby 完全可以从命令行驱动。我要在这里说 Ruby3 甚至胜过 Perl5，因为它内置了一个非常强大的 REPL，名为 `irb`。

# 单行命令

Unix 系统管理员大部分时间都在 shell 中工作，很少使用 GUI 工具，而且理由充分：**[1]** GUI 工具难以文档化 **[2]** GUI 工具经常重新设计，使文档无效 **[3]** GUI 工具难以自动化 **[4]** GUI 工具通常具有大量需要的库依赖 **[5]** GUI 工具在远程服务器上操作不实用，需要低延迟 **[6]** GUI 工具要求服务器安装 X，这是您很少需要的。**相反，我们使用命令行 shell**，如 Bash/Ksh/Csh 等。

单行命令是在单个 shell 提示符中可以编写的程序。使用 Ruby，您可以轻松编写简单和复杂的单行命令。**而在 Python 中，您无法编写复杂的单行命令**，因为要做有用且可读的事情，必须换行并缩进代码。

**单行命令有用吗？** 根据我的经验是有用的。因为您可以将单行命令复制到您的文档 *.odt 文件或 Google 文档中，需要时再将其粘贴到 shell 提示符中。它就能运行。**不需要编写 tmp.sh 文件**，也不需要删除 tmp.sh 文件，您不需要在提示所在位置拥有写权限，也不需要在远程系统上安装您喜欢的编辑器。

**一个一行命令的例子。** 这来自我关于配置**Samba**的笔记。我保留原始语言（意大利语）并发布一张图片以明确这些是真实使用中的笔记。这个脚本查找并删除指定由 `smbd -b` 的输出和 `grep` 过滤的目录中所有 *.tdb 和 *.tlb 文件。

我从笔记中放到 Linux 盒子 Samba 配置期间的 Shell 中的一个一行命令的例子。

# 我的语言是我的世界：正则表达式

我从未使用过 Fortran，但我记得一个物理学家朋友告诉我[可以从 0 或 1 开始索引数组](https://docs.oracle.com/cd/E19957-01/805-4940/z400091044d0/index.html)，这取决于你的选择和方便性。这可能非常有用，有时候从零开始索引只会增加复杂性。**Fortran**确实是一种为**数值计算**而生的语言，它自然地发展出了这种用例的良好特性。

Unix 的一个原则是其配置文件是文本数据，可供人类阅读。数据在可能的情况下是文本的。系统自动化的系统脚本以文本形式存在，程序的输出如果可能的话也是文本形式，最经典的互联网协议都是基于文本的。**文本是 Unix 的统一本质**，**文本流经管道，经过进程，最终到达文件或网络**。在某种意义上，文本行对 Unix 系统来说就像列表对 Lisp 类编程的中心统一模式一样。下面的图形来自我最近阅读的一篇[优秀文章](https://www.abortretry.fail/p/the-berkley-software-distribution)，在那些我们正在讨论的管道中很好地表达了，你看不见它，但如果你能打开这些管道，你很可能会看到一个文本流。

描述文本块最强大的现有工具是**正则表达式**。作为 Perl 的后继者，Ruby 把正则表达式作为一等公民。这意味着你不需要加载任何外部包或使用任何其他数据结构来定义一个正则表达式。它们内置于数字、字符串、数组、字典、文件等中。

让我用一个代码图片来证明，你甚至不需要知道 Ruby 来理解我写的东西，只需要了解正则表达式语法的基础和面向对象编程的点符号。

在这里我们运行‘irb’，从字面意思上创建一个简单的正则表达式对象 /^a/（这个表达式意味着‘匹配每一个以字母 a 开头的字符串’），检查正则表达式对象的类并尝试查看正则表达式对象是否匹配了一些字符串。

作为比较，在 Python 中的情况如何？在 Python 中要使用正则表达式，你需要加载模块 `re`，然后需要通过一个字符串创建正则表达式，并且他们甚至不称之为正则表达式，而是称之为“模式”。

Python 中的正则表达式不是内置于语言中的。要使用正则表达式，你需要调用一个外部模块 ‘re’ 并通过字符串输入一个正则表达式，没有特定的字面量。

假设你正在编写数值分析软件，你会使用一个必须使用字符串来定义浮点数并执行 `import fl` 然后 `pi = Float("3.14")` ，最后如果你询问类型，你得到 `type(pi) = NumberWithDot` 的编程语言吗？我猜你不会。这就是为什么像所有Unix管理员一样工作的人应该优先选择**Ruby**而不是Python，它**具备完成任务所需的正确工具**。

# 完全面向对象和规则的

据我经验，唯一比Ruby更规则的广泛使用的编程语言是**类似Lisp**的语言家族和**类似Smalltalk**的语言家族。这些是非常规则的语言，以至于即使是C风格的if-then-else结构或者现在我们认为是显而易见的for循环，在这些语言中都不存在。你必须以“对象接收消息”模型或列表的形式表达这些概念，没有任何逃逸。

**Ruby** 比Lisp和Smalltalk更宽容/友好一些，如果你愿意的话，有些语法糖用于“if”语句的定义，用于定义函数、类等。它最基本的控制结构类似于我们从C/C++语言家族那里所熟悉的。

如果你忽略了一些[例外情况](https://stackoverflow.com/questions/416047/examples-of-things-that-are-not-objects-in-ruby)，**Ruby 是完全面向对象的**，一切都是对象，你通过向对象发送消息来进行所有编程。你必须思考类、对象和方法。如果你不喜欢这样，最好不要使用这种语言，我建议你在Unix脚本编程时使用Perl。

语言规则性的**好处**主要体现在**阅读**和**记忆**的便利上。我不是一直在写Ruby代码。只有在需要在我的服务器上自动化某些东西时才写它。对我来说，语言的规则性非常重要，因为这样记忆和猜测起来就更容易。

**示例。** 现在对我来说，用这个命令 `ruby -e 'p Time.now.to_i’` 比记住 `date` 的格式化参数更容易。为什么更容易？因为 `ruby -e` 是我过去在Perl5中使用的相同语法，`Time.now` 显而易见 `to_i` 表示转换为整数，在Ruby中编码一段时间后也很明显，`p` 开头相当于无处不在的 `print`，难以忘记。

另一方面，**Python** 更像C++，你可以命令式编程，函数式编程，放入一些面向对象编程，但它**完全不像一个完全面向对象的语言**，到了这种程度，这被定义为 `len([1,2.3])` 但你不能做 `[1,2,3].len()` 。第二种语法对面向对象的人来说是唯一合理的。在Ruby中，我们像这样做 `"hello".size()` 就像我们做 `[1,2,3].size()` 一样，这一切都有意义，它是规则且清晰的。

# Ruby 是一种完全可修改的语言

在与Lisp类似的语言和Smalltalk类似的语言中，Ruby是完全可扩展的。我的意思是你可以随心所欲地扩展和修改语言（当然，要保持总体概念框架，因此，Lisp代码将具有函数调用，而Smalltalk和Ruby代码将具有对象）。如果应用程序足够复杂，你可以扩展和转换语言来描述应用程序对象，而不是将问题扭曲成适应语言。

**示例**。让我们看一个极端且危险的例子，让我们**重新定义数字之间的相等概念**。这并不是那么荒谬，有时这正是你想要的。例如，在数值分析中，你永远不直接测试一个浮点数是否为零，像这样`f == 0.0`，而是做类似于`abs(f) < epsilon`的事情。下面的代码展示了如何在Ruby中重新定义`==`，使其行为类似于数值分析中的方式。

```
# .make sure we don't loose the original '==' method 
#  giving it an alias, a synonim 
class Float 
  alias origEq == 
end 

# . define "small" with the global variable $epsilon
$epsilon = 0.0001

# . change the behaviour of 'A == B' in the case where 
#   A and B are FLOAT and B = 0.0\. 
class Float 
  def ==(other)
    if (other.origEq(0.0)) then 
      return (self.abs < $epsilon)
    else
      return (self.origEq(other))
    end
  end
end

# EXERCISE. if you write == 0 instead of 0.0 it won't work, why?
0.1 == 0.0       # false 
0.001 == 0.0     # false 
0.000001 == 0.0  # => true !
```

**不建议**像这样编码，并且警告你至少考虑七次再做类似的事情。但是，**并不禁止**！Ruby，像Unix一样，假设你是一个可以在打字之前读取和思考的理性人类。

**示例**。**扩展内置类的新方法**。这比重新定义现有方法要安全得多。我经常这样做。让我们扩展`String`类。我希望每个字符串都能够判断其内容是否像一个IPv4地址。

如果`s.class`是`String`，我希望能够使用`s.ipv4?`，并且得到一个布尔值作为输出。例如，我希望`"10.11.12.123".ipv4? => true`和`"foobar".ipv4? => false`。此外，我希望该过程能够容忍字符串中前后的空格。

我从[这里](https://stackoverflow.com/a/36760050/2129178)获取了匹配IPv4的正则表达式，其余部分很轻松。

```
# . Adding a method to the String builtin class
class String 
   # . check if the string looks like an IPv4 address, spaces
   #   at the beginning and end of the string are ignored. 
   def ipv4? 
     self.strip.match? /^((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.?\b){4}$/
   end
end

"foo".ipv4?            # => false 
"781.123.12.4".ipv4?   # => false 
"187.123.12.4".ipv4?   # => true
```

在Python中，所有这些都是科幻，并且[根本不可能](https://stackoverflow.com/questions/237128/why-does-python-code-use-len-function-instead-of-a-length-method/42499450?noredirect=1#comment135919544_42499450)，[你不被允许修改内置类](https://stackoverflow.com/questions/2165200/modifying-a-python-class)。

# 括号：少一点迷雾，更多主动的代码

对我来说，括号尤其让人恼火，因为它们需要按两个键：Shift和‘[’。如果偶尔需要一次的话还好，但在Python中括号到处都是。在Python3中，他们愚蠢地要求在最常输入的内容上也要加上：**print**。现在是`print("hello world")`。在Python2中是`print "Hello world"`。Python2更好。

在Ruby中，我们甚至比Python2更好。不仅可以写`puts "hello world"`，还有像`p "hello world"`这样的快捷方式。这不是很酷吗？想象一下你可以多快地完成，不要打字，动脑筋。在调试中，打印无处不在。

我们可以做得比这更多。在Ruby中，大多数时候，只要不模糊，**你可以在不使用括号的情况下调用方法**。这真的很好，使链式方法看起来非常清晰。让我们看一个微不足道的例子，在下一行中我们测试一个字符串是否包含某个词。我们链接了4个方法，并且只在有意义时使用括号，你可以看到这是多么可读。

```
" Sopra la panca la capra campa ".strip.downcase.
                                  split(/\s+/).include?("capra")
```

# 多行REPL及其包重量

有时在Ruby（或Python）REPL中解决问题比在Bash/Ksh/X shell中更快；它们只是更强大。不过，有一个限制阻碍了Python3 REPL，即它没有多行编辑功能。这是一个相当大的问题，因为Python主要基于空格和换行符，试图在没有它们的情况下编写代码是根本不可能或者说不端正的。如果你想在Python中的REPL中进行任何可读的编程，你必须使用**iPython3**，但这又带来另一个问题，**iPython3很大并且有很多依赖项**，系统管理员不喜欢这样。更多依赖项带来更多问题，更长的升级时间，总体上系统安全性降低。

在**Ruby3**中，情况完全不同，Ruby3带有自己的多行REPL称为**irb**，这是你在REPL上交互式地编写多行代码所需的一切。

**Unix中的REPL有用吗？**对我来说是的，绝对有用，它是当你只需运行代码一次时的最佳解决方案。**[1]** 比一行代码更容易编写，因为你可以换行。**[2]** 它可以逐步开发，观察中间值并进行实验。**[3]** 当你关闭REPL时，一切都消失了，没有垃圾临时文件需要删除。**[4]** 你不受Shell引用机制的约束，一行代码总是一个Shell字符串，必须用`"..."`或`'...'`来界定，对于Ruby而言这不是问题，因为Ruby像Perl一样有非常灵活的额外字符串引用机制，比如`%q{ hello ' " }`，其中分界符`{..}`由你选择。

**示例**。在我的备份系统中，我每周保留公司中几个重要数据库的一次性转储。当我编写数据库备份系统时，我觉得使用`DD-MMM-YYYY-HH-MM`格式作为文件名是个好主意，出于人类可读性的考虑。不过过了一段时间，我确信这不太实用，最好默认情况下在Shell中按创建时间列出文件。因此，我们想要将一堆目录改名为新格式`backup-YYYY-mm-DD-HH-MM`，原始名称列表的一部分出现在下面的图片中，请问如何操作？

我们希望将备份目录名称改为新格式的视图

这是一个**一次性问题**的例子，解决了就永远解决了。没有必要写一个一行代码并把它存放在我的笔记中。它足够简单，不需要我启动编辑器创建一个脚本文件之类的，我们在REPL中解决它。

我使用`irb`启动REPL，在当前目录中获取要重命名的目录列表`dirList0 = Dir.Glob("*")`。注意Ruby具有**globbing**的概念，这是Unix文化的一部分。然后我进行一些实验，确定如何将现有字符串解析为DateTime对象，获取`dt0 = DateTime.strptime(dirList.firt, "%d-%b-%Y-%H-%M")`，最后我找到如何按照我希望的格式打印DateTime对象`dt0.strftime("%Y-%m-%d-%H-%M")`。请注意，我添加了许多不必要的括号，以使代码对非Ruby程序员更易于阅读。好的，我们有了编写我们的循环的所有工具，可以在多行REPL编辑器中执行。

在下面的图片中，您可以看到我在REPL中输入的内容。因此，我打印出应运行以重命名每个目录的命令。如果要运行命令，我只需添加`system(cmd)`或``cmd``，与Perl一样，因此我们的Unix文化不会浪费。我现在不执行`cmd`，因为要写入备份目录，我需要提升权限，这只有在我完全专注于问题时才会这样做。最后，我使用`system`和`mv`来删除文件，以保持对Unix人员的熟悉感，毫无疑问，Ruby有自己的文件操作工具。

使用REPL编写的有用代码块的基本示例，用于重命名一堆文件。在底部是输出的摘录。

在我们激励了多行REPL可能会有很好用途后，让我们看看**安装多行REPL所需的外部包数量**。以下是在**OpenBSD 7.4**上安装**iPython3**或**Ruby3**所需包数量的比较。图像说明一切，毫无比较，从管理员的角度来看，唯一能在这里击败Ruby3的语言是Perl5，因为它随操作系统一起提供，无需安装任何内容。尽管如此，Perl5没有完整的默认REPL，它并不是为此而生，它有一个简单的调试器模式，可以使用`perl -d -e '1'`来运行。

在OpenBSD 7.4上安装Python3和Ruby3的多行REPL的比较成本，以所需包的形式。

# 使用JRuby进行跨操作系统和跨数十年的可移植性

如果将脚本从计算机A移动到计算机B，很可能在B上无法运行。可能B上没有安装Ruby，Ruby的PATH可能不同，RUBY_VERSION可能不同，某些gem可能缺失或版本不兼容，某些代码可能依赖于编译和库，某些可能与操作系统相关...这些是我想到的第一个可能原因。请注意，如果出现“版本问题”，将软件更改为在不同版本解释器上运行可能并不容易：从Python2迁移到Python3是一个漫长的噩梦。我知道有多个人的工作变成了“Python2转Python3转换人员”。

**可能的解决方案**。有一个用**Java**编写的Ruby版本，只是一个***.jar文件**。它被称为[**JRuby**](https://www.jruby.org/)。如果您将*.jar文件和脚本从计算机A和计算机B（两者都已正确安装Java）移动，那么该脚本将以相同的方式运行（至少在理论上是如此）。无需安装任何东西，只需传输文件！这真是太棒了。

Java确实有自己的问题，但将您的脚本移植到Java中确保了比标准Ruby更多的可移植性。如果您的Ruby语言和用户代码只是一堆必须输入JVM的文件，那么由Java环境来确保能够在未来几年内运行这些代码。Java环境负责在不同操作系统之间确保一定程度的可移植性。Java得到大公司的支持和使用，不会在未来十年被摒弃。

**额外奖励**。从JRuby中，您可以直接使用为Java提供的大量库，并且可以使用Java线程。

Python呢？在Python的可移植性方面，情况仍然是一个未解决的混乱。曾经有一个活跃的**Jython**，但它仍处于Python2的休眠状态。您应该尽量不再使用Python2。

# 结论

至今，Ruby大多数与Rails web框架相关联。我从未使用过Rails。我使用Ruby是因为它是我尝试过的Unix中最好的脚本语言（Perl5、Python2–3、Awk、Tcl、Bash、Ksh、Csh）之一。它取代了Perl、Awk、Python，以及几个Unix强大工具都是我可以用Ruby一行命令达到的阴影拷贝，例如xargs、grep、sed等。

虽然Python的主要格言是“只有一种方法可以做到”，Ruby的格言是“Ruby是程序员最好的朋友”。自助一下。

祝您编程愉快！

# 后续修订和澄清

+   [2024年2月26日]。`p`不是`puts`的同义词。它是一个检查工具。不过，在像Float、String、Array、Time、Hash（我测试过的这些）等简单对象上，它会打印出您期望的内容，因此，在某种程度上，您可以将其用作`puts`的快捷方式。查看[ruby-doc, Kernel#p](https://ruby-doc.org/3.2.2/Kernel.html#method-i-p)中的区别。[感谢Reddit读者@rubyrt提醒我注意到这一点]
