<!--yml
category: 未分类
date: 2024-05-27 15:08:19
-->

# cohost! - "TCL on the brain/worms"

> 来源：[https://cohost.org/sakiamu/post/177439-tcl-on-the-brain](https://cohost.org/sakiamu/post/177439-tcl-on-the-brain)

So, first things first, TCL stands for Tool Command Language. It is a language, like SmallTalk, IO, or Lisp, which is built around A Big Idea. TCL's big idea is that All Values Are Strings. There's a *lot* of consequences to that idea, but one of the ones that you hit after the first 10 hours of going "ah, this is super cool, I can build UIs at a REPL" is that TCL's type/value system doesn't expose references, and actively avoids constructions that could become references that aren't based on variables names or namespaces.

So, like, TCL is a *really* cool language and set of libraries, but it, like Dark Souls, represents a Different Evolutionary Path amongst its peers. Unlike Dark Souls, I don't know that it makes a convincing argument that said path should be adopted by the mainstream.

This has consequences. It means that you don't really get mutable closures, a la Javascript. It means that when you write code that yields to the TCL event loop via `vwait`, it has to wait for writes to a global variable, rather than one that's in the current scope. This is at once, almost the promise of await that would take the dev world by storm later, but also with *just* enough flaws to not take off in nearly the same way. It means that you can reason in terms of local scope, the scopes above me, and the global scope, but not in terms of little islands of values that can't be edited from the outside, at least that I've seen. (though, see also coroutines, I've not tested them here).

And this isn't obvious at first. In a way, it means that TCL has more in common with Erlang and it's All Immutable Values, than it does with, say Perl, from a values/terms system standpoint.

But, that "Everything is a String" applies to code as well. Which means that you can metaprogram code, and turn arguments to a command into a DSL, and that TCL, more than many languages, has `eval` and friends at its core.

Of course, TCL is optimized under the hood so that not every usage of a value necessarily creates a string. It's more of a moral imperative that values *could* become strings, than that they *are* strings at any given time.

This also leaks out into the design of Tk. The core interface to Tk a set of commands that create, modify, and layout various Tk widgets. But, those widgets are also store in a name heirarchy, not unlike a filesystem in layout. Also each widget is turned into a proc/command for easy access and configuration at a REPL.

So, if you wanted to make 10-key number pad, it might look something like this in terms of code

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

Here we see a lot of the TCL quirks discussed before. `num_handler` is acting more like a macro here than a regular function via string interpolation. `.numpad_0` is a command, with `-configure` as a subcommand. If we had nested controls, such as putting this in a panel, then we'd have command names like `.panel.numpad_0`.

This means that Tk code that nests widgets has to work inside that name hierarchy. It also means that for someone to be able to move widgets in that heirarchy, there'd need to be tkmove commands that, to my knowledge, do not exist.

Anywho, TCL is neat, but weird, and tends to force you to use more powerful tools, like metaprogramming, where other languages, due to having more data flexiblity, allow you to use less powerful tools, like lambda functions.