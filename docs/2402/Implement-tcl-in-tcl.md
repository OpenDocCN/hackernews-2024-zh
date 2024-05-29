<!--yml
category: 未分类
date: 2024-05-27 14:49:28
-->

# Implement tcl in tcl

> 来源：[https://wiki.tcl-lang.org/page/Implement+tcl+in+tcl](https://wiki.tcl-lang.org/page/Implement+tcl+in+tcl)

Maybe someone has already done this. But the question in my mind is how much of Tcl can be implimented in Tcl itself. Obviously you can't implement system calls in Tcl, but you could implement just about everthing else. What commands/parts of Tcl would be in the minimal set? [Earl Johnson](/page/Earl+Johnson)

1.  "set" both scalar and array modes.
2.  "eval" command
3.  "unknown"
4.  "string index" command
5.  "string length" command
6.  "lindex" command
7.  "llength" command
8.  "list" command
9.  what else???

*   Things that could be simulated in Tcl (theoretically)

1.  interpreters (design?).
2.  namespaces (design?).
3.  expr (design?).
4.  callstack (design?) .
5.  proc (design?) .
6.  if/while/switch/for/foreach.
7.  regexp (sounds hard) .
8.  package system (design?) .
9.  traces (design?).
10.  event loop (design?).
11.  glob (design?).

1.  expr
2.  regexp *([Lars H](/page/Lars+H): You get a bit of the way with [grammar_fa](/page/grammar%5Ffa).)*

*   What I don't expect to be done

1.  C API
2.  Things that need system calls

*   Other languages are self hosting like this

1.  C => c compiler
2.  Python => PyPy
3.  Smalltalk
4.  Others??

Of course I expect that this TclTcl would be very slow, but interesting, and very easy to port to embedded systems.

[SS](/page/SS) 12Oct2004: One day for fun I implemented a subset of Tcl in Tcl, with some more work may run non trivial programs. Consider this program BSD-licensed.

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

I have got problems testing it (error: invalid command name "::macro::parser").

This is because the call to ::macro::parser doesn't reference any known function. When I changed the call to LctParser, everything worked for me. I have taken the liberty of making the change in the code above. -RL

I suppose also that the parser would have problems with some special ugly code as follow, which is accepted by the original tcl parser:

```
   set {]} t
   set a [list ${]} ]
   set {"} t
   set a "a ${"} "
   set a "ewe [list "sewe"] ewr"
```

Anyway programming tcl in tcl is very interesting idea if one wants to test experimental tcl interpreters or build in new functions which are not supported by original interpreter

[Lars H](/page/Lars+H): I think several of those would actually be handled OK, but

```
   set a [list ${]} ]
```

is not (the only extra rule in command substitution nesting is that ']' terminates a command in places where ';' would, so brackets need not nest properly as LctSubstCmd assumes). Backslash substitution is missing entirely. Also the following isn't handled correctly:

```
   set a x{$a}x
```

The left brace only has special powers if it is the first character in a word. What worries me most about the above parser is however:

*   its use of a variable $inside (that is too simple to keep track of the state of a correct Tcl parser), and
*   the one big loop over characters in the input (hard to analyse, because there are so many cases to consider, and thus likely to buggy).

For a more complete Tcl parser, available here on this Wiki, see [parsetcl](/page/parsetcl).

* * *

[Category Application](/page/Category+Application) - [Category Tcl Implementations](/page/Category+Tcl+Implementations)