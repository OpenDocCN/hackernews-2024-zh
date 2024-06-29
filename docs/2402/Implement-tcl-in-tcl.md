<!--yml

类别：未分类

日期：2024年5月27日14时49分28秒

-->

# 在Tcl中实现Tcl

> 来源：[https://wiki.tcl-lang.org/page/Implement+tcl+in+tcl](https://wiki.tcl-lang.org/page/Implement+tcl+in+tcl)

也许有人已经做到了这一点。但是我心中的问题是Tcl中有多少可以用Tcl本身实现的内容。显然，你不能在Tcl中实现系统调用，但你可以实现几乎所有其他东西。最小的命令/ Tcl的哪些部分将包含在其中？

1.  "set"，标量和数组模式。

1.  "eval" 命令

1.  "unknown"

1.  "string index" 命令

1.  "string length" 命令

1.  "lindex" 命令

1.  "llength" 命令

1.  "list" 命令

1.  还有其他吗？

+   在Tcl中可以模拟的功能（从理论上讲）

1.  解释器（设计？）。

1.  命名空间（设计？）。

1.  表达式（设计？）。

1.  调用栈（设计？）。

1.  过程（设计？）。

1.  if/while/switch/for/foreach

1.  正则表达式（听起来很难）。

1.  软件包系统（设计？）。

1.  跟踪（设计？）。

1.  事件循环（设计？）。

1.  全局（设计？）。

1.  表达式

1.  正则表达式 *([Lars H](/page/Lars+H)：您可以用[grammar_fa](/ page / grammar%5Ffa)做到一点)*

+   我不希望完成的事情

1.  C API

1.  需要系统调用的功能

+   其他语言的自托管方式就像这样

1.  C => c 编译器

1.  Python => PyPy

1.  Smalltalk

1.  其他？

当然，我预计这个TclTcl会非常慢，但是很有趣，并且非常容易移植到嵌入式系统。

[SS](/page/SS) 2004年10月12日：有一天，我为了好玩而在Tcl中实现了Tcl的一个子集，再加点工作就能运行一些非平凡的程序。考虑将此程序授权许可为BSD许可证。

```
 # Lct - A Tcl-like language implemented in Tcl
 # The implementation does NOT try to exploit the fact we
 # are implementing Tcl in Tcl exposing some Tcl built-in
 # to make the work simpler, thus this implementation is
 # quite portable. It should be quite easy to port it
 # to Python, PHP, Perl, Scheme and alike.
 #
 # Copyright (C) 2003 Salvatore Sanfilippo
 #
 # Biggest differences with Tcl:
 #
 # A lot of course, but this are important stuff you should
 # know before to try even some trivial example:
 #
 # - No 'expr' support, this also changes if/for/while semantic:
 #   What in Tcl is "if {$a > $b} ..." here is "if {> $a $b} ..." and so on.
 #   Math is done lispwise using +, -, *, / commands and so on.
 #   Note that conditionals doesn't perform substitution also, so
 #   There is no need of [] in most cases, but you need to use
 #   the "pass" identity function to test for a variable, like:
 #   if {pass $test} {...} elseif {...} else {...}
 # - No arrays for now
 # - No namespaces
 # - False is both zero and an empty string, no implicit expr in
 #   conditionals means to face the need to handle any kind of return
 #   string as false or true.
 # - True is just what's not false ;) any string that isn't "0" nor "".
 # - The 'pass' command is the "identity" command returning it's only argument.
 #   pass foobar ;# => foobar
 # - Unbalanced {} in comments are not a problem (but you need to quote
 #   anyway if your Lct program body is defined inside a Tcl script).
 #
 # TODO:
 #
 # - variable number of arguments to procedures
 # - exceptions, i.e. [catch]
 # - upvar
 # - break/for
 # - uproc foo args body, foo will be executed in the caller context
 # - set x(foo), $x(foo) as syntax glue for some "dict set" "dict get" command
 # - be able to save the continuation may be cool, but this interpreter
 #   calls itself from Tcl, so I will remove the recursion or figure how
 #   to reenter the recursion starting from state-information.
 # - handle line numbers in the parser to specify error's line number
 # - check for raised error condition in LctEval to exit with an error

 ################################################################################
 #                                 Lct Core
 ################################################################################

 # Interpreter state is implemented as global vars and arrays
 # no support for multiple interpreters (for now at least).
 array set ::LctCommands {}
 array set ::LctProcs {}
 set ::LctStack {}
 set ::LctError {}
 set ::LctStackLen 0

 proc LctAddStackFrame {} {
     incr ::LctStackLen
     namespace eval StackFrame$::LctStackLen {
        variable EvalLevel 0
        array set Locals {}
        array set Globals {}
     }
 }

 proc LctRemoveStackFrame {} {
     namespace delete StackFrame$::LctStackLen
     incr ::LctStackLen -1
 }

 proc LctSaveStackFrame varname {
     upvar $varname saved
     set sf $::LctStackLen
     set saved {}
     lappend saved [array get StackFrame$sf\::Locals]
     lappend saved [array get StackFrame$sf\::Globals]
     lappend saved [set StackFrame$sf\::EvalLevel]
 }

 proc LctRestoreStackFrame varname {
     upvar $varname saved
     set sf $::LctStackLen
     namespace eval StackFrame$sf {}
     array set StackFrame$sf\::Locals [lindex $saved 0]
     array set StackFrame$sf\::Globals [lindex $saved 1]
     set StackFrame$sf\::EvalLevel [lindex $saved 2]
 }

 # Create the top-level stack frame
 LctAddStackFrame

 # The parser is the core of the interpreter, being this
 # interpreted from the source code directly. No compilation.
 proc LctParser {text tokenvar indexvar {dosubst 0}} {
     upvar $tokenvar token
     upvar $indexvar i
     set token {}
     set inside {}
     set dontstop $dosubst
     while 1 {
        # skip spaces
        while {!$dontstop && [string match "\[ \t\]" [string index $text $i]]} {
            incr i
        }
        # skip comments
        if {!$dontstop && [string equal [string index $text $i] #]} {
            while {[string length [string index $text $i]] &&
                  ![string match [string index $text $i] \n]} \
            {
                incr i
            }
        }
        # check for special conditions
        if {!$dontstop} {
            switch -exact -- [string index $text $i] {
                {} {return EOF}
                {;} -
                "\n" {incr i; return EOL}
            }
        }
        # main parser loop
        while 1 {
            switch -exact -- [string index $text $i] {
                {} break
                { } -
                "\t" -
                "\n" -
                ";" {
                    if {!$dontstop} {
                        break;
                    }
                }
                \" {
                    if {[string equal $inside {}]} {
                        incr dontstop
                        set inside \"
                        incr i
                        continue
                    } elseif {[string equal $inside \"]} {
                        incr dontstop -1
                        set inside {}
                        incr i
                        continue
                    }
                }
                "\{" {
                    if {[string equal $inside {}]} {
                        incr dontstop
                        set inside "\{"
                        incr i
                        continue
                    } elseif {[string equal $inside "\{"]} {
                        incr dontstop
                    }
                }
                "\}" {
                    if {[string equal $inside "\{"]} {
                        incr dontstop -1
                        if {$dontstop == 0} {
                            set inside {}
                            incr i
                            continue
                        }
                    }
                }
                \$ {
                    if {![string equal $inside "\{"]} {
                        if {![string equal [string index $text [expr {$i+1}]] $]} {
                            set res [LctSubstVar $text $indexvar]
                            append token $res
                            continue
                        }
                    }
                }
                \[ {
                    if {![string equal $inside "\{"]} {
                        set res [LctSubstCmd $text $indexvar]
                        append token $res
                        continue
                    }
                }
            }
            append token [string index $text $i]
            incr i
        }
        return TOK
     }
 }

 proc LctSubstCmd {text indexvar} {
     upvar $indexvar i
     set go 1
     set cmd {}
     incr i
     while {$go} {
        switch -exact -- [string index $text $i] {
            {} break
            \[ {incr go}
            \] {incr go -1}
        }
        append cmd [string index $text $i]
        incr i
     }
     set cmd [string range $cmd 0 end-1]
     return [LctEval $cmd]
 }

 # Get the control when a '$' (not followed by $) is encountered,
 # extract the name of the variable, and return its content.
 proc LctSubstVar {text indexvar} {
     upvar $indexvar i
     set dontstop 0
     set varname {}
     incr i
     while {1} {
        switch -exact -- [string index $text $i] {
            \[ -
            \] -
            "\t" -
            "\n" -
            "\"" -
            \; -
            \{ -
            \} -
            \$ -
            ( -
            ) -
            { } -
            {} {
                if {!$dontstop} {
                    break
                }
            }
            ( {incr dontstop}
            ) {incr dontstop -1}
            default {
                append varname [string index $text $i]
            }
        }
        incr i
     }
     if {![LctLookupVar $varname content]} {
        error "No such variable '$varname'"
     } else {
        return $content
     }
 }

 proc LctLookupVar {varname contentvar} {
     set sf $::LctStackLen
     upvar $contentvar content
     if {[info exists StackFrame$sf\::Globals($varname)]} {
        set sf 1
     }
     if {![info exists StackFrame$sf\::Locals($varname)]} {
        return 0
     }
     set content [set StackFrame$sf\::Locals($varname)]
     return 1
 }

 proc LctGetEvalLevel {} {
     return [set StackFrame$::LctStackLen\::EvalLevel]
 }

 proc LctSetEvalLevel newlevel {
     set StackFrame$::LctStackLen\::EvalLevel $newlevel
 }

 proc LctEval script {
     set result {}
     set eof 0
     set i 0

     set level [LctSetEvalLevel [expr {[LctGetEvalLevel]+1}]]
     while {!$eof && ([LctGetEvalLevel] >= $level)} {
        set argv {}
        set argc 0
        while 1 {
            set state [LctParser $script token i]
            if {[string equal $state EOF]} {
                set eof 1
            }
            switch $state {
                EOF -
                EOL break
                default {
                    lappend argv $token
                    incr argc
                }
            }
        }
        if {$argc} {
            set cmd [lindex $argv 0]
            if {![info exists ::LctCommands($cmd)]} {
                error "No such command '$cmd'"
            } else {
                set result [$::LctCommands($cmd) $argv]
                if {[string length $::LctError]} {error "$::LctError\n  in script:\n$script"}
            }
        }
     }
     if {$level == [LctGetEvalLevel]} {
        LctSetEvalLevel [expr {[LctGetEvalLevel]-1}]
     }
     return $result
 }

 proc LctUplevel {level script result} {
     upvar $result res
     if {$::LctStackLen <= $level} {
        LctSetError "Bad Level"
        return
     }
     LctSaveStackFrame stackframe
     incr ::LctStackLen -$level
     set res [LctEval $script]
     incr ::LctStackLen $level
     LctRestoreStackFrame stackframe
     return $res
 }

 # Do substitution of commands and vars
 proc LctSubst string {
     set i 0
     set s [LctParser $string token i 1]
     return $token
 }

 proc LctRegisterCommand {name function} {
     set ::LctCommands($name) $function
 }

 proc LctSetVar {varname value} {
     set sf $::LctStackLen
     if {[info exists StackFrame$sf\::Globals($varname)]} {
        set sf 1
     }
     return [set StackFrame$sf\::Locals($varname) $value]
 }

 proc LctMarkGlobal varname {
     set sf $::LctStackLen
     set StackFrame$sf\::Globals($varname) {}
 }

 proc LctSetError error {
     set ::LctError $error
 }

 # In Lct both 0 and empty string is false.
 proc LctIsFalse value {
     if {[string equal $value 0] || [string equal $value {}]} {
        return 1
     } else {
        return 0
     }
 }

 # Define it as the negation of LctIsFalse
 proc LctIsTrue value {
     return [expr {![LctIsFalse $value]}]
 }

 ################################################################################
 #                                Core Commands
 ################################################################################

 proc LctSet argv {
     if {[llength $argv] != 3} {
        LctSetError "Bad number of arguments, try: set varname value"
        return
     }
     return [LctSetVar [lindex $argv 1] [lindex $argv 2]]
 }

 proc LctPut argv {
     set nonewline 0
     if {[llength $argv] >= 2 && [string match [lindex $argv 1] -nonewline]} {
        set nonewline 1
        set argv [lrange $argv 1 end]
     }
     if {[llength $argv] != 2} {
        LctSetError "Bad number of arguments, try: put string"
        return
     }
     puts -nonewline stdout [lindex $argv 1]
     if {!$nonewline} {
        puts {}
     }
     return {}
 }

 # That's a generic binding for math stuff. It uses expr, and
 # 'sens' what operator to use from the name of the procedure itself.
 proc LctGenericMathOp argv {
     if {[llength $argv] != 3} {
        LctSetError "Bad number of arguments, try: + number number"
        return
     }
     set e [lindex $argv 1][lindex $argv 0][lindex $argv 2]
     set e [string map "\[ \\\[ \] \\\]" $e]
     return [expr $e]
 }

 proc LctIncr argv {
     if {[llength $argv] != 2 && [llength $argv] != 3} {
        LctSetError "Bad number of arguments, try: incr varname ?increment?"
        return
     }
     set varname [lindex $argv 1]
     if {[llength $argv] == 3} {
        set increment [lindex $argv 2]
     } else {
        set increment 1
     }
     if {![LctLookupVar $varname val]} {
        LctSetError "No such var '$varname'"
        return
     }
     if {[catch {expr {$val+$increment}} result]} {
        LctSetError "Expected integer, got something else ($result)"
        return
     }
     return [LctSetVar $varname $result]
 }

 proc LctProc argv {
     if {[llength $argv] != 4} {
        LctSetError "Bad number of arguments, try: proc name args body"
        return
     }
     LctRegisterCommand [lindex $argv 1] LctCallProc
     set ::LctProcs([lindex $argv 1]) [list [lindex $argv 2] [lindex $argv 3]]
     return {}
 }

 # This built-in is used to call user-defined procedures
 # It checks for argv(0} in order to get the name of the
 # procedure to call, then create a new stack frame and call it.
 proc LctCallProc argv {
     foreach {arglist body} $::LctProcs([lindex $argv 0]) break
     if {[llength $argv]-1 != [llength $arglist]} {
        LctSetError "Wrong number of args calling procedure '[lindex $argv 0]'"
        return
     }
     set l [llength $arglist]
     LctAddStackFrame
     for {set i 0} {$i < $l} {incr i} {
        LctSetVar [lindex $arglist $i] [lindex $argv [expr {$i+1}]]
     }
     set result [LctEval $body]
     LctRemoveStackFrame
     return $result
 }

 # Return is simple, we set the stack frame's EvalLevel to 0 in order
 # to be sure eval will return to the previous procedure.
 proc LctReturn argv {
     if {[llength $argv] != 1 && [llength $argv] != 2} {
        LctSetError "Bad number of arguments, try: return ?value?"
        return
     }
     LctSetEvalLevel 0
     if {[llength $argv] == 2} {
        return [lindex $argv 1]
     } else {
        return {}
     }
 }

 # Facility to pop arguments in varargs proc.
 proc LctPopArg varname {
     upvar $varname argv
     set arg [lindex $argv 0]
     set argv [lreplace $argv 0 0]
     return $arg
 }

 # The if command implemented as trivial FSA.
 proc LctIf argv {
     set argv [lreplace $argv 0 0] ;# Drop the 'if' first argument.
     set state EXPR
     while {[llength $argv]} {
        switch -exact $state {
            EXPR {
                set e [LctPopArg argv]
                set res [LctIsTrue [LctEval $e]]
                if {$res} {
                    set state EVAL_NEXT
                } else {
                    set state SKIP_TRUE_BRANCH
                }
            }
            EVAL_NEXT {
                set script [LctPopArg argv]
                return [LctEval $script]
            }
            SKIP_TRUE_BRANCH {
                LctPopArg argv ;# Just skip it
                set state ELSE_OR_ELSEIF_OR_FALSE_BRANCH
            }
            ELSE_OR_ELSEIF_OR_FALSE_BRANCH {
                set x [LctPopArg argv]
                if {[string equal $x else]} {
                    set state EVAL_NEXT
                } elseif {[string equal $x elseif]} {
                    set state EXPR
                } else {
                    return [LctEval $x]
                }
            }
        }
     }
     switch -exact $state {
        EXPR {
            LctSetError "Missing expression in if"
            return
        }
        EVAL_NEXT {
            LctSetError "Missing script in if"
            return
        }
     }
 }

 proc LctWhile argv {
     if {[llength $argv] != 3} {
        LctSetError "Bad number of arguments, try: while cond body"
        return
     }
     foreach {_ cond body} $argv break
     while {[LctIsTrue [LctEval $cond]]} {
        set result [LctEval $body]
     }
     return $result
 }

 proc LctPass argv {
     if {[llength $argv] != 2} {
        LctSetError "Bad number of arguments, try: pass ?value?"
        return
     }
     return [lindex $argv 1]
 }

 proc LctEvalCmd argv {
     set script {}
     foreach x [lrange $argv 1 end] {append script $x}
     return [LctEval $script]
 }

 proc LctUplevelCmd argv {
     if {[llength $argv] < 2} {
        LctSetError "Bad number of arguments, try: uplevel ?level? arg ... ?arg?"
        return {}
     }
     foreach {- level script} $argv break
     LctUplevel $level $script result
     return $result
 }

 proc LctGlobalCmd argv {
     set argv [lrange $argv 1 end]
     foreach varname $argv {
        LctMarkGlobal $varname
     }
     return {}
 }

 # Sort part of the core
 LctRegisterCommand proc LctCallProc
 LctRegisterCommand set LctSet
 LctRegisterCommand proc LctProc
 LctRegisterCommand return LctReturn
 LctRegisterCommand if LctIf
 LctRegisterCommand pass LctPass
 LctRegisterCommand while LctWhile
 LctRegisterCommand eval LctEvalCmd
 LctRegisterCommand uplevel LctUplevelCmd
 LctRegisterCommand global LctGlobalCmd

 # Random stuff that are really needed, but not part of the core itself.
 LctRegisterCommand puts LctPut
 LctRegisterCommand + LctGenericMathOp
 LctRegisterCommand - LctGenericMathOp
 LctRegisterCommand * LctGenericMathOp
 LctRegisterCommand / LctGenericMathOp
 LctRegisterCommand % LctGenericMathOp
 LctRegisterCommand > LctGenericMathOp
 LctRegisterCommand >= LctGenericMathOp
 LctRegisterCommand < LctGenericMathOp
 LctRegisterCommand =< LctGenericMathOp
 LctRegisterCommand == LctGenericMathOp
 LctRegisterCommand != LctGenericMathOp
 LctRegisterCommand incr LctIncr

 ################################################################################
 #                                   Example
 ################################################################################

 set text {
     # Test for comment
     # Test basic substitution
     set name "All The Tclers"
     puts "Hello to $name [+ 1 2] [+ 10 20]"
     puts "(6*3)+(8*2)=[+ [* 6 3] [* 8 2]]"
     # Test procedures
     proc test a {
        puts "I'm printing $a"
     }
     test "Hello World"
     # Test return from procedure without value
     proc testreturn {} {
        puts "Test Return"
        return
        puts "FooBar"
     }
     testreturn
     testreturn
     # Test return with value
     proc testreturn2 x {
        return $x$x
     }
     puts "Return With Value: [testreturn2 XyZ]"
     # Test conditionals
     set a 1
     if {pass $a} {puts "($a) is true"} else {puts "($a) is false"}
     set a 0
     if {pass $a} {puts "($a) is true"} else {puts "($a) is false"}
     set a foobar
     if {pass $a} {puts "($a) is true"} else {puts "($a) is false"}
     set a {}
     if {pass $a} {puts "($a) is true"} else {puts "($a) is false"}
     # Test incr
     set x 10
     puts "Now it is $x"
     incr x
     puts "Now it is $x"
     incr x -2
     puts "Now it is $x"
     # Test Loops
     while {< $x 20} {
        incr x 1
        puts "Hello World $x"
     }
     set script {puts -nonewline we;}
     eval $script {puts " are inside eval"}
     # Test global
     proc testglobal {} {
        global x
        puts "x is global, value: $x"
        set x 30
        puts "x value changed, will print the new value outside the proc"
     }
     testglobal
     puts "x new value is $x"
     # Test uplevel
     proc testuplevel {} {
        puts ---
        uplevel 1 {puts "testglobal was called from uplevel: [testglobal]"}
        uplevel 1 {puts "In uplevel, x = $x"}
     }
     testuplevel
 }

 LctEval $text
```

* * *

我测试它时遇到了问题（错误：无效的命令名称“::macro::parser”）。

这是因为对::macro::parser的调用没有涉及到任何已知的函数。当我将调用更改为LctParser时，一切都对我起作用。我已经擅自在上面的代码中做出了更改。- RL

我还觉得解析器会对一些特殊的丑陋代码产生问题，如下所示，它是原始Tcl解析器所接受的：

```
   set {]} t
   set a [list ${]} ]
   set {"} t
   set a "a ${"} "
   set a "ewe [list "sewe"] ewr"
```

无论如何，在Tcl中编程Tcl是一个非常有趣的想法，如果有人想测试实验性的Tcl解释器或者构建原始解释器不支持的新功能。

[Lars H](/page/Lars+H)：我认为其中几个实际上会被很好地处理, 但

```
   set a [list ${]} ]
```

在命令替换嵌套中的唯一额外规则不该是“]”在“;”的位置终止一个命令，并且方括号不需要正确嵌套，如LctSubstCmd所假定的。反斜杠替换完全消失。还没有正确处理以下内容：

```
   set a x{$a}x
```

如果左花括号是单词的第一个字符，它才有特别的功能。然而，上述解析器最让我担心的是：

+   它的使用变量$inside（过于简单不能保持正确的Tcl解析器的状态），以及

+   输入中的字符大循环（分析起来很困难，因为有这么多情况要考虑，因此可能会有错误）。

要了解更完整的Tcl解析器，请参见[Wiki](/page/parsetcl)上的[parsetcl]。

* * *

[应用类别](/page/Category+Application) - [Tcl 实现类别](/page/Category+Tcl+Implementations)
