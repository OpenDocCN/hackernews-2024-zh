<!--yml

category: 未分类

date: 2024-05-27 15:08:19

-->

# cohost! - "TCL on the brain/worms"

> 来源：[https://cohost.org/sakiamu/post/177439-tcl-on-the-brain](https://cohost.org/sakiamu/post/177439-tcl-on-the-brain)

首先要明确一点，TCL代表工具命令语言（Tool Command Language）。它是一种像SmallTalk、IO或者Lisp一样围绕一个大点子构建的语言。TCL的大点子是所有的值都是字符串。这个点子有很多*后果*，但是在前10个小时后你会发现，“啊，这太酷了，我可以在REPL上构建UI”，TCL的类型/值系统不暴露引用，并且积极避免可能会变成引用的构造，这些构造并不基于变量名或命名空间。

所以，就像，TCL是一个*非常*酷的语言和库集，但是它，就像《黑暗之魂》一样，代表了其同行中的不同进化路径。与《黑暗之魂》不同的是，我不知道它是否提出了说服人们采用这种路径的论点。

这是有后果的。这意味着你实际上并不会得到可变闭包，就像JavaScript那样。这意味着当你编写通过`vwait`让TCL事件循环暂停的代码时，它必须等待对全局变量的写入，而不是当前作用域中的变量。这几乎可以说是后来将席卷开发世界的await的承诺，但是同时也有*足够多的缺陷*，以至于无法以同样的方式普及。这意味着你可以按照本地作用域、上层作用域和全局作用域的方式进行推理，但不能按照不能从外部编辑的小值岛屿的方式进行推理，至少我所见到的没有。（虽然，请参见协程，我在这里没有测试过）。

而这一点起初并不明显。在某种程度上，它意味着TCL与Erlang及其全部不可变值有更多共同点，而不是与Perl等在值/术语系统角度的共同点。

但是，“一切皆为字符串”也适用于代码。这意味着你可以元编程代码，并将命令的参数转换为DSL，并且TCL比许多其他语言更多地将`eval`和相关功能作为其核心特性之一。

当然，TCL在内部进行了优化，以便不是每次使用值都会创建一个字符串。这更像是一种道德命令，即值*可能*成为字符串，而不是它们在任何给定时间*都是*字符串。

这也渗透到Tk的设计中。Tk的核心接口是一组命令，用于创建、修改和布局各种Tk小部件。但是，这些小部件也存储在一个类似文件系统的名称层次结构中。此外，每个小部件都被转换为一个过程/命令，以便在REPL上进行轻松访问和配置。

所以，如果你想制作一个10键数字键盘，代码可能看起来像这样

```
set number "0"

proc num_handler {n} {
    return "
        global number
        if {\$number == 0} {
            set number \"\"
        } 
        set number \[string cat \$number $n \]
    "
}

for {set i 1} {$i < 10} {incr i} {
    set name ".numpad_$i"
    ttk::button $name -text $i -command [num_handler $i] 
    grid $name -row [expr {(($i - 1) / 3) + 1}] -column [expr {($i - 1) % 3}]
}

ttk::button .numpad_0 -command [num_handler 0]

.numpad_0 -configure text 0

grid .numpad_0 -row 4 -column 1 

ttk::entry .numpad_entry -textvariable number
grid .numpad_entry -row 0 -column 0 -columnspan 3 
```

在这里，我们看到了之前讨论过的很多 TCL 怪癖。`num_handler` 在这里更像是一个宏，而不是通过字符串插值来实现的常规函数。`.numpad_0` 是一个命令，带有 `-configure` 作为子命令。如果我们有嵌套控件，比如将其放在面板中，那么我们将会有像 `.panel.numpad_0` 这样的命令名称。

这意味着嵌套小部件的 Tk 代码必须在该名称层次结构内运行。这也意味着，要让某人能够在该层次结构中移动小部件，需要有我所知道的 tkmove 命令，但实际上并不存在。

总之，TCL 很整洁，但又很奇怪，并且倾向于迫使您使用更强大的工具，如元编程，而其他语言则由于具有更多的数据灵活性，允许您使用不那么强大的工具，如 lambda 函数。
