<!--yml
category: 未分类
date: 2024-05-27 15:20:32
-->

# The C Bounded Model Checker: Criminally Underused | Hey There Buddo!

> 来源：[https://www.philipzucker.com/cbmc_tut/](https://www.philipzucker.com/cbmc_tut/)

Follow along on a google colab: [https://colab.research.google.com/github/philzook58/philzook58.github.io/blob/master/pynb/cbmc_tut.ipynb](https://colab.research.google.com/github/philzook58/philzook58.github.io/blob/master/pynb/cbmc_tut.ipynb)

```
%%bash
# download and install CBMC
wget https://github.com/diffblue/cbmc/releases/download/cbmc-5.95.1/ubuntu-20.04-cbmc-5.95.1-Linux.deb
apt-get install bash-completion
dpkg -i ubuntu-20.04-cbmc-5.95.1-Linux.deb 
```

Whenever I’m tinkering around in C, I would love for some easy way to check my work. There is a variety of options (turn on all warnings, inbuilt compiler [static analyzers](https://developers.redhat.com/articles/2022/04/12/state-static-analysis-gcc-12-compiler#a_taint_mode_for_c), sanitizers ([address](https://clang.llvm.org/docs/AddressSanitizer.html), thread, cfi), [infer](https://fbinfer.com/), going hardcore interactive theorem prover mode), but an undersung one is using automatic static verifiers.

I’m very impressed by the [Software Verification Competition](https://sv-comp.sosy-lab.org/) style solvers, in particular [CBMC](https://github.com/diffblue/cbmc) ([Documentation](https://diffblue.github.io/cbmc//index.html)).

I thought CBMC might be defunct, since it’s [main webpage](https://www.cprover.org/cbmc/) does not show recent development, but actually the [github repo](https://github.com/diffblue/cbmc) is very active. It appears Amazon is investing in it’s usage with a number of significant C projects having CBMC based specs ad verification.

It’s pretty easy to install and use. Just get a release from the [github repo](https://github.com/diffblue/cbmc/releases) (the one in apt is rather stale)

Here’s a basic example:

Obviously the following program will fail the assert

```
%%file /tmp/ex1.c
#include <assert.h> int main(){
    assert(1 == 0);
} 
```

It’s normal code. I can compiler and run it and the assertion triggers.

```
! gcc /tmp/ex1.c -o /tmp/ex1 && /tmp/ex1 
```

```
ex1: /tmp/ex1.c:3: main: Assertion `1 == 0' failed. 
```

But I can also run the source through cbmc

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/ex1.c
Converting
Type-checking ex1
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.00026975s
size of program expression: 20 steps
simple slicing removed 5 assignments
Generated 1 VCC(s), 1 remaining after simplification
Runtime Postprocess Equation: 1.098e-05s
Passing problem to propositional reduction
converting SSA
Runtime Convert SSA: 3.654e-05s
Running propositional reduction
Post-processing
Runtime Post-process: 2.905e-06s
Solving with MiniSAT 2.2.1 with simplifier
0 variables, 0 clauses
SAT checker: instance is SATISFIABLE
Runtime Solver: 2.3624e-05s
Runtime decision procedure: 7.9172e-05s

** Results:
/tmp/ex1.c function main
[2m[main.assertion.1] [0mline 3 assertion 1 == 0: [31mFAILURE[0m

** 1 of 1 failed (2 iterations)
VERIFICATION FAILED 
```

```
%%file /tmp/ex2.c
#include <assert.h> int main(){
    int x;
    assert(x != 12345);
} 
```

We can also compile this program. It runs just fine.

```
! gcc /tmp/ex2.c -o /tmp/ex2 && /tmp/ex2 
```

CBMC however runs all possible executions, discovering an erroring value the uninitialized variable could take on. We can also track the execution line by line with the `--trace` option

```
! cbmc /tmp/ex2.c --trace 
```

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/ex2.c
Converting
Type-checking ex2
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.000307621s
size of program expression: 21 steps
simple slicing removed 5 assignments
Generated 1 VCC(s), 1 remaining after simplification
Runtime Postprocess Equation: 1.093e-05s
Passing problem to propositional reduction
converting SSA
Runtime Convert SSA: 0.000651015s
Running propositional reduction
Post-processing
Runtime Post-process: 4.693e-06s
Solving with MiniSAT 2.2.1 with simplifier
33 variables, 33 clauses
SAT checker: instance is SATISFIABLE
Runtime Solver: 7.3262e-05s
Runtime decision procedure: 0.000780993s
Building error trace

** Results:
/tmp/ex2.c function main
[2m[main.assertion.1] [0mline 4 assertion x != 12345: [31mFAILURE[0m

Trace for main.assertion.1:

State 11 file /tmp/ex2.c function main line 3 thread 0
----------------------------------------------------
  x=12345 [2m(00000000 00000000 00110000 00111001)[0m

[31mViolated property:[0m
  file /tmp/ex2.c function main line 4 thread 0
  [31massertion x != 12345[0m
  x != 12345

** 1 of 1 failed (2 iterations)
VERIFICATION FAILED 
```

If we put this assertion inside a guarding if, it is no longer possible to trigger the assertion.

```
%%file /tmp/ex3.c
int main(){
    int x;
    if (x <= 42){
            assert(x != 12345);
    }
} 
```

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/ex3.c
Converting
Type-checking ex3
file /tmp/ex3.c line 4 function main: function 'assert' is not declared
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.000833781s
size of program expression: 23 steps
simple slicing removed 5 assignments
Generated 1 VCC(s), 1 remaining after simplification
Runtime Postprocess Equation: 1.5136e-05s
Passing problem to propositional reduction
converting SSA
Runtime Convert SSA: 0.000201988s
Running propositional reduction
Post-processing
Runtime Post-process: 4.988e-06s
Solving with MiniSAT 2.2.1 with simplifier
67 variables, 100 clauses
SAT checker inconsistent: instance is UNSATISFIABLE
Runtime Solver: 2.427e-05s
Runtime decision procedure: 0.00026319s

** Results:
/tmp/ex3.c function main
[2m[main.assertion.1] [0mline 4 assertion x != 12345: [32mSUCCESS[0m

** 0 of 1 failed (1 iterations)
VERIFICATION SUCCESSFUL 
```

Here is a simple functional correctness example. We can write a `myabs` function and ask if indeed the result is always positive.

```
%%file /tmp/test.c
#include <assert.h>
#include <stdint.h> 
int64_t myabs(int64_t x){
    return x <= 0 ? -x : x;
}

int64_t nondet_int();

int main(){
    int64_t x = nondet_int();
    int64_t y = myabs(x);
    assert(y >= 0);
} 
```

```
! cbmc /tmp/test.c --trace 
```

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/test.c
Converting
Type-checking test
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.000758549s
size of program expression: 33 steps
simple slicing removed 5 assignments
Generated 1 VCC(s), 1 remaining after simplification
Runtime Postprocess Equation: 1.2014e-05s
Passing problem to propositional reduction
converting SSA
Runtime Convert SSA: 0.000439795s
Running propositional reduction
Post-processing
Runtime Post-process: 3.483e-06s
Solving with MiniSAT 2.2.1 with simplifier
701 variables, 1075 clauses
SAT checker: instance is SATISFIABLE
Runtime Solver: 0.000975965s
Runtime decision procedure: 0.00144989s
Building error trace

** Results:
/tmp/test.c function main
[2m[main.assertion.1] [0mline 13 assertion y >= 0: [31mFAILURE[0m

Trace for main.assertion.1:

State 13 file /tmp/test.c function main line 11 thread 0
----------------------------------------------------
  return_value_nondet_int=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

State 14 file /tmp/test.c function main line 11 thread 0
----------------------------------------------------
  x=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

State 19 file /tmp/test.c function main line 12 thread 0
----------------------------------------------------
  x=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

State 20 file /tmp/test.c function myabs line 5 thread 0
----------------------------------------------------
  goto_symex$$return_value$$myabs=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

State 22 file /tmp/test.c function main line 12 thread 0
----------------------------------------------------
  return_value_myabs=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

State 23 file /tmp/test.c function main line 12 thread 0
----------------------------------------------------
  y=-9223372036854775808l [2m(10000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000)[0m

[31mViolated property:[0m
  file /tmp/test.c function main line 13 thread 0
  [31massertion y >= 0[0m
  y >= (signed long int)0

** 1 of 1 failed (2 iterations)
VERIFICATION FAILED 
```

Say whaaaaaaa???? Oh yeah. Going through the trace see it picked negative maxint. In two’s complement, there is one more negative number available than positive number. So it is not clear to me what the compiler should do here actually. I would guess it is allowed to do anything.

Here is something you are more likely to care about, checking for common memory bugs like buffer overflows, etc

```
%%file /tmp/buffer.c
int main(){
    char buffer[10];
    buffer[10] = 0;
} 
```

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/buffer.c
Converting
Type-checking buffer
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.000799121s
size of program expression: 21 steps
simple slicing removed 0 assignments
Generated 0 VCC(s), 0 remaining after simplification
Runtime Postprocess Equation: 0.000160385s
VERIFICATION SUCCESSFUL 
```

Whaaaaa!? Where’s the bug?!

Yeah, CBMC requires flags to turn on these default checks, for better or worse. I kind of wish there was a do eveything flag, but to my knowledge there isn’t. Here is a pile of possibly useful bugs to look for.

```
! cbmc /tmp/buffer.c --bounds-check --conversion-check --div-by-zero-check --float-overflow-check --malloc-fail-null \
	--malloc-may-fail --nan-check --pointer-check --pointer-overflow-check --pointer-primitive-check \
	--signed-overflow-check --undefined-shift-check --unsigned-overflow-check --memory-leak-check 
```

```
CBMC version 5.95.1 (cbmc-5.95.1) 64-bit x86_64 linux
Parsing /tmp/buffer.c
Converting
Type-checking buffer
Generating GOTO Program
Adding CPROVER library (x86_64)
Removal of function pointers and virtual functions
Generic Property Instrumentation
Running with 8 object bits, 56 offset bits (default)
Starting Bounded Model Checking
Runtime Symex: 0.00054141s
size of program expression: 24 steps
simple slicing removed 8 assignments
Generated 2 VCC(s), 1 remaining after simplification
Runtime Postprocess Equation: 1.7143e-05s
Passing problem to propositional reduction
converting SSA
Runtime Convert SSA: 0.000106257s
Running propositional reduction
Post-processing
Runtime Post-process: 4.634e-06s
Solving with MiniSAT 2.2.1 with simplifier
160 variables, 0 clauses
SAT checker: instance is SATISFIABLE
Runtime Solver: 4.0895e-05s
Runtime decision procedure: 0.000177761s

** Results:
function __CPROVER__start
[2m[__CPROVER__start.memory-leak.1] [0mdynamically allocated memory never freed in __CPROVER_memory_leak == NULL: [32mSUCCESS[0m

/tmp/buffer.c function main
[2m[main.array_bounds.1] [0mline 3 array 'buffer' upper bound in buffer[(signed long int)10]: [31mFAILURE[0m

** 1 of 2 failed (2 iterations)
VERIFICATION FAILED 
```

# Next Time

I think that’s enough for today, but we’ve only scratched the surface. But to whet some appetite, here’s some uneditted bits about loops and comparative checking.

[https://www.youtube.com/embed/AUsNTNq0dbY?si=zfELEKMDjHjIytVZ](https://www.youtube.com/embed/AUsNTNq0dbY?si=zfELEKMDjHjIytVZ)

VIDEO

## Loops

Ok, we’ve painted a rosy picture so far. Obviously CBMC does not scale to arbitrarily large and difficult problems

The basic technique is to uniwind them. All is not lost, many loops (for loops in particular) can be completelyb unwound. You can add `--unwinding-assertions` to know if you’ve covered all possible executions. Ever if you can’t, passing these chekcs does give you some confidence.

If you’re going to the next level there is also the ability to add invariant annotations. I’m not sure if CBMC can infer these successfully

On bigger problems, I’ve had some success fiddling with the

## Comparative Checking

Comparative checking of programs can be useful for reducing both the specification and verification burden. In addition, there are some relational properties (“hyperproperties”) that require talking about two different runs of a program. The canonical example is information security. If you run two programs with identical low security parts of the states, they must end with identical low security parts of the state. This is true if there is only influence or information transferral going from low to high.

*   Translation validation
*   Refinement
*   Security properties
*   Bug fixing

The simplest approach to doing comparative verification so is to use the more commonly available single program verifier on the “product program”.

```
echo '
#include <assert.h>
int safeprog(int low, int high){
    int foo = low ^ high;
    foo = ((foo << 1) ^ high) >> 1;
    return foo;
}

int main(){
    int high = nondet_int();
    int high1 = nondet_int();
    int low = nondet_int();
    //int high, high1, low;

    __ESBMC_assert(safeprog(low,high) == safeprog(low,high1), "information security property");
    return 0;
}
' > /tmp/test.c
esbmc /tmp/test.c 
```

There are many bits of software out there that have security vulnerabilities.

Obviously, the easiest and most likely solution is to change the code in source, recompile, and push out the new version. This is not always possible or desirable however.

*   Even recompiling the original program may introduce bugs due to compiler difference versions
*   ## You may not have the source

It is in general an interesting problem to consdier the 50 year software stack.

Worse is better: A use case for dumb compilers

# Bits and Bobbles

Here’s a list of serious C projects with CBMC instrumentation, many from Amazon: https://model-checking.github.io/cbmc-training/projects.html. They are playing a similar role to unit tests for library functions.

SV Comp https://sv-comp.sosy-lab.org/ - CPAchecker UAutomizer are very successful in the competition. I have not used them as much

Klee, symcc are symbolic executors. Similar in many respects to a bounded model checker. I think the biggest philosophical difference is they aren’t really centered around ensuring bug absence, instead around bug finding, which is a little different.

Frama-C VST https://github.com/verifast/verifast graham seemed to like this one

[esbmc](http://esbmc.org/) is a C bounded model checker.

Comparing a C function to it’s grammar?

```
echo "
#include<stdbool.h>

bool check_balance(char *input){
    int count = 0;
    while(*input != '\0'){
        if(*input == '(') count++;
        if(*input == ')') count--;
        input++;
    }
    return count == 0;
" > /tmp/parens.c
cbmc /tmp/parens.c 
```

Nice, fairly easy to use [homepage](https://www.cprover.org/cbmc/) [https://arxiv.org/abs/2302.02384](https://arxiv.org/abs/2302.02384) `sudo apt install cbmc` [https://github.com/diffblue/cbmc](https://github.com/diffblue/cbmc) gitbub [https://diffblue.github.io/cbmc/](https://diffblue.github.io/cbmc/) docs

[https://github.com/diffblue/aws-training](https://github.com/diffblue/aws-training) [https://model-checking.github.io/cbmc-training/](https://model-checking.github.io/cbmc-training/) [https://github.com/model-checking/cbmc-starter-kit](https://github.com/model-checking/cbmc-starter-kit) starter kit template [https://model-checking.github.io/cbmc-starter-kit/tutorial/index.html](https://model-checking.github.io/cbmc-starter-kit/tutorial/index.html) insstrumenting a maloc [https://github.com/model-checking/cbmc-proof-debugger](https://github.com/model-checking/cbmc-proof-debugger)

[manual](http://www.cprover.org/cprover-manual/) see tutorial

```
echo "
int main()
{
  int buffer[10];
  buffer[20] = 10;
}
" > /tmp/overflow.c
cbmc /tmp/overflow.c --bounds-check --pointer-check --trace 
```

All kinds of analysuis options

User defined stuff via `assert` or specialized

Supports lots of different smt and sat backended. Generic dimacs, Could toss into kissat

ESBMC

*   C++ frontend
*   Float support
*   Still actually developd

[https://awslabs.github.io/aws-proof-build-assistant/](https://awslabs.github.io/aws-proof-build-assistant/) [https://github.com/awslabs/aws-c-common](https://github.com/awslabs/aws-c-common) corejson [https://github.com/FreeRTOS/coreJSON/tree/main/test/cbmc](https://github.com/FreeRTOS/coreJSON/tree/main/test/cbmc) s2n-tls [https://github.com/aws/s2n-tls/tree/main/tests/cbmc](https://github.com/aws/s2n-tls/tree/main/tests/cbmc) [https://github.com/aws/aws-encryption-sdk-c/tree/master/verification/cbmc](https://github.com/aws/aws-encryption-sdk-c/tree/master/verification/cbmc) [https://github.com/aws/s2n-quic](https://github.com/aws/s2n-quic)

bounded proof vs harnes vs contracts

[https://crates.io/crates/libcprover_rust/5.91.0](https://crates.io/crates/libcprover_rust/5.91.0) rust api

[https://dl.acm.org/doi/pdf/10.1145/3551349.3559523](https://dl.acm.org/doi/pdf/10.1145/3551349.3559523) CBMC-SSM: Bounded Model Checking of C Programs with Symbolic Shadow Memory [https://github.com/diffblue/cbmc/issues/7757](https://github.com/diffblue/cbmc/issues/7757)

```
%%script sqlite3 --echo
CREATE TABLE test (x INTEGER, y INTEGER);
INSERT INTO test VALUES (1, 2);
INSERT INTO test SELECT x+1, y+1 FROM test;
SELECT * FROM test;
.quit 
```

```
Available line magics:
%alias  %alias_magic  %autoawait  %autocall  %automagic  %autosave  %bookmark  %cat  %cd  %clear  %colors  %conda  %config  %connect_info  %cp  %debug  %dhist  %dirs  %doctest_mode  %ed  %edit  %env  %gui  %hist  %history  %killbgscripts  %ldir  %less  %lf  %lk  %ll  %load  %load_ext  %loadpy  %logoff  %logon  %logstart  %logstate  %logstop  %ls  %lsmagic  %lx  %macro  %magic  %man  %matplotlib  %mkdir  %more  %mv  %notebook  %page  %pastebin  %pdb  %pdef  %pdoc  %pfile  %pinfo  %pinfo2  %pip  %popd  %pprint  %precision  %prun  %psearch  %psource  %pushd  %pwd  %pycat  %pylab  %qtconsole  %quickref  %recall  %rehashx  %reload_ext  %rep  %rerun  %reset  %reset_selective  %rm  %rmdir  %run  %save  %sc  %set_env  %store  %sx  %system  %tb  %time  %timeit  %unalias  %unload_ext  %who  %who_ls  %whos  %xdel  %xmode

Available cell magics:
%%!  %%HTML  %%SVG  %%bash  %%capture  %%debug  %%file  %%html  %%javascript  %%js  %%latex  %%markdown  %%perl  %%prun  %%pypy  %%python  %%python2  %%python3  %%ruby  %%script  %%sh  %%svg  %%sx  %%system  %%time  %%timeit  %%writefile

Automagic is ON, % prefix IS NOT needed for line magics. 
```

```
IPython -- An enhanced Interactive Python - Quick Reference Card
================================================================

obj?, obj??      : Get help, or more help for object (also works as
                   ?obj, ??obj).
?foo.*abc*       : List names in 'foo' containing 'abc' in them.
%magic           : Information about IPython's 'magic' % functions.

Magic functions are prefixed by % or %%, and typically take their arguments
without parentheses, quotes or even commas for convenience.  Line magics take a
single % and cell magics are prefixed with two %%.

Example magic function calls:

%alias d ls -F   : 'd' is now an alias for 'ls -F'
alias d ls -F    : Works if 'alias' not a python name
alist = %alias   : Get list of aliases to 'alist'
cd /usr/share    : Obvious. cd -<tab> to choose from visited dirs.
%cd??            : See help AND source for magic %cd
%timeit x=10     : time the 'x=10' statement with high precision.
%%timeit x=2**100
x**100           : time 'x**100' with a setup of 'x=2**100'; setup code is not
                   counted.  This is an example of a cell magic.

System commands:

!cp a.txt b/     : System command escape, calls os.system()
cp a.txt b/      : after %rehashx, most system commands work without !
cp ${f}.txt $bar : Variable expansion in magics and system commands
files = !ls /usr : Capture system command output
files.s, files.l, files.n: "a b c", ['a','b','c'], 'a\nb\nc'

History:

_i, _ii, _iii    : Previous, next previous, next next previous input
_i4, _ih[2:5]    : Input history line 4, lines 2-4
exec(_i81)       : Execute input history line #81 again
%rep 81          : Edit input history line #81
_, __, ___       : previous, next previous, next next previous output
_dh              : Directory history
_oh              : Output history
%hist            : Command history of current session.
%hist -g foo     : Search command history of (almost) all sessions for 'foo'.
%hist -g         : Command history of (almost) all sessions.
%hist 1/2-8      : Command history containing lines 2-8 of session 1.
%hist 1/ ~2/     : Command history of session 1 and 2 sessions before current.
%hist ~8/1-~6/5  : Command history from line 1 of 8 sessions ago to
                   line 5 of 6 sessions ago.
%edit 0/         : Open editor to execute code with history of current session.

Autocall:

f 1,2            : f(1,2)  # Off by default, enable with %autocall magic.
/f 1,2           : f(1,2) (forced autoparen)
,f 1 2           : f("1","2")
;f 1 2           : f("1 2")

Remember: TAB completion works in many contexts, not just file names
or python names.

The following magic functions are currently available:

%alias:
    Define an alias for a system command.
%alias_magic:
    ::
%autoawait:

%autocall:
    Make functions callable without having to type parentheses.
%automagic:
    Make magic functions callable without having to type the initial %.
%autosave:
    Set the autosave interval in the notebook (in seconds).
%bookmark:
    Manage IPython's bookmark system.
%cat:
    Alias for `!cat`
%cd:
    Change the current working directory.
%clear:
    Clear the terminal.
%colors:
    Switch color scheme for prompts, info system and exception handlers.
%conda:
    Run the conda package manager within the current kernel.
%config:
    configure IPython
%connect_info:
    Print information for connecting other clients to this kernel
%cp:
    Alias for `!cp`
%debug:
    ::
%dhist:
    Print your history of visited directories.
%dirs:
    Return the current directory stack.
%doctest_mode:
    Toggle doctest mode on and off.
%ed:
    Alias for `%edit`.
%edit:
    Bring up an editor and execute the resulting code.
%env:
    Get, set, or list environment variables.
%gui:
    Enable or disable IPython GUI event loop integration.
%hist:
    Alias for `%history`.
%history:
    ::
%killbgscripts:
    Kill all BG processes started by %%script and its family.
%ldir:
    Alias for `!ls -F -o --color %l | grep /$`
%less:
    Show a file through the pager.
%lf:
    Alias for `!ls -F -o --color %l | grep ^-`
%lk:
    Alias for `!ls -F -o --color %l | grep ^l`
%ll:
    Alias for `!ls -F -o --color`
%load:
    Load code into the current frontend.
%load_ext:
    Load an IPython extension by its module name.
%loadpy:
    Alias of `%load`
%logoff:
    Temporarily stop logging.
%logon:
    Restart logging.
%logstart:
    Start logging anywhere in a session.
%logstate:
    Print the status of the logging system.
%logstop:
    Fully stop logging and close log file.
%ls:
    Alias for `!ls -F --color`
%lsmagic:
    List currently available magic functions.
%lx:
    Alias for `!ls -F -o --color %l | grep ^-..x`
%macro:
    Define a macro for future re-execution. It accepts ranges of history,
%magic:
    Print information about the magic function system.
%man:
    Find the man page for the given command and display in pager.
%matplotlib:
    ::
%mkdir:
    Alias for `!mkdir`
%more:
    Show a file through the pager.
%mv:
    Alias for `!mv`
%notebook:
    ::
%page:
    Pretty print the object and display it through a pager.
%pastebin:
    Upload code to dpaste.com, returning the URL.
%pdb:
    Control the automatic calling of the pdb interactive debugger.
%pdef:
    Print the call signature for any callable object.
%pdoc:
    Print the docstring for an object.
%pfile:
    Print (or run through pager) the file where an object is defined.
%pinfo:
    Provide detailed information about an object.
%pinfo2:
    Provide extra detailed information about an object.
%pip:
    Run the pip package manager within the current kernel.
%popd:
    Change to directory popped off the top of the stack.
%pprint:
    Toggle pretty printing on/off.
%precision:
    Set floating point precision for pretty printing.
%prun:
    Run a statement through the python code profiler.
%psearch:
    Search for object in namespaces by wildcard.
%psource:
    Print (or run through pager) the source code for an object.
%pushd:
    Place the current dir on stack and change directory.
%pwd:
    Return the current working directory path.
%pycat:
    Show a syntax-highlighted file through a pager.
%pylab:
    ::
%qtconsole:
    Open a qtconsole connected to this kernel.
%quickref:
    Show a quick reference sheet 
%recall:
    Repeat a command, or get command to input line for editing.
%rehashx:
    Update the alias table with all executable files in $PATH.
%reload_ext:
    Reload an IPython extension by its module name.
%rep:
    Alias for `%recall`.
%rerun:
    Re-run previous input
%reset:
    Resets the namespace by removing all names defined by the user, if
%reset_selective:
    Resets the namespace by removing names defined by the user.
%rm:
    Alias for `!rm`
%rmdir:
    Alias for `!rmdir`
%run:
    Run the named file inside IPython as a program.
%save:
    Save a set of lines or a macro to a given filename.
%sc:
    Shell capture - run shell command and capture output (DEPRECATED use !).
%set_env:
    Set environment variables.  Assumptions are that either "val" is a
%store:
    Lightweight persistence for python variables.
%sx:
    Shell execute - run shell command and capture output (!! is short-hand).
%system:
    Shell execute - run shell command and capture output (!! is short-hand).
%tb:
    Print the last traceback.
%time:
    Time execution of a Python statement or expression.
%timeit:
    Time execution of a Python statement or expression
%unalias:
    Remove an alias
%unload_ext:
    Unload an IPython extension by its module name.
%who:
    Print all interactive variables, with some minimal formatting.
%who_ls:
    Return a sorted list of all interactive variables.
%whos:
    Like %who, but gives some extra information about each variable.
%xdel:
    Delete a variable, trying to clear it from anywhere that
%xmode:
    Switch modes for the exception handlers.
%%!:
    Shell execute - run shell command and capture output (!! is short-hand).
%%HTML:
    Alias for `%%html`.
%%SVG:
    Alias for `%%svg`.
%%bash:
    %%bash script magic
%%capture:
    ::
%%debug:
    ::
%%file:
    Alias for `%%writefile`.
%%html:
    ::
%%javascript:
    Run the cell block of Javascript code
%%js:
    Run the cell block of Javascript code
%%latex:
    Render the cell as a block of LaTeX
%%markdown:
    Render the cell as Markdown text block
%%perl:
    %%perl script magic
%%prun:
    Run a statement through the python code profiler.
%%pypy:
    %%pypy script magic
%%python:
    %%python script magic
%%python2:
    %%python2 script magic
%%python3:
    %%python3 script magic
%%ruby:
    %%ruby script magic
%%script:
    ::
%%sh:
    %%sh script magic
%%svg:
    Render the cell as an SVG literal
%%sx:
    Shell execute - run shell command and capture output (!! is short-hand).
%%system:
    Shell execute - run shell command and capture output (!! is short-hand).
%%time:
    Time execution of a Python statement or expression.
%%timeit:
    Time execution of a Python statement or expression
%%writefile:
    :: 
```

```
%script z3
(echo "starting z3")
(declare-const x Int)
(declare-const y Int)
(assert (= y (+ x 1)))
(check-sat)
(get-model) 
```

```
 Cell In [16], line 2
    (echo "starting z3")
          ^
SyntaxError: invalid syntax 
```