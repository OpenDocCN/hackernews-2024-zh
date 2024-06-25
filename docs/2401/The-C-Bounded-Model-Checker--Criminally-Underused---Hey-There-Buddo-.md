<!--yml

类别：未分类

日期：2024-05-27 15:20:32

-->

# C 有界模型检查器：鲜为人知的使用 | 嗨，伙计！

> 来源：[`www.philipzucker.com/cbmc_tut/`](https://www.philipzucker.com/cbmc_tut/)

在 Google Colab 上跟着做：[`colab.research.google.com/github/philzook58/philzook58.github.io/blob/master/pynb/cbmc_tut.ipynb`](https://colab.research.google.com/github/philzook58/philzook58.github.io/blob/master/pynb/cbmc_tut.ipynb)

```
%%bash
# download and install CBMC
wget https://github.com/diffblue/cbmc/releases/download/cbmc-5.95.1/ubuntu-20.04-cbmc-5.95.1-Linux.deb
apt-get install bash-completion
dpkg -i ubuntu-20.04-cbmc-5.95.1-Linux.deb 
```

每当我在 C 语言中摸索时，我都希望有一种简单的方法来检查我的工作。有各种各样的选项（打开所有警告，内置编译器[静态分析器](https://developers.redhat.com/articles/2022/04/12/state-static-analysis-gcc-12-compiler#a_taint_mode_for_c)，[sanitizers](https://clang.llvm.org/docs/AddressSanitizer.html)（地址，线程，cfi），[infer](https://fbinfer.com/)，进入强大的交互式定理证明器模式），但其中一个不太被重视的方法是使用自动静态验证器。

我对[软件验证竞赛](https://sv-comp.sosy-lab.org/)风格的求解器印象深刻，特别是[CBMC](https://github.com/diffblue/cbmc)（[文档](https://diffblue.github.io/cbmc//index.html)）。

我以为 CBMC 可能已经停用了，因为它的[主页](https://www.cprover.org/cbmc/)没有显示最近的开发，但实际上[github 仓库](https://github.com/diffblue/cbmc)非常活跃。看来亚马逊正在投资于它的使用，许多重要的 C 项目都有基于 CBMC 的规范和验证。

安装和使用都很简单。只需从[github 仓库](https://github.com/diffblue/cbmc/releases)获取一个发布版（apt 上的版本相对过时）

这是一个基本的例子：

显然，下面的程序会触发断言失败

```
%%file /tmp/ex1.c
#include <assert.h> int main(){
    assert(1 == 0);
} 
```

这是正常的代码。我可以编译和运行它，断言会触发。

```
! gcc /tmp/ex1.c -o /tmp/ex1 && /tmp/ex1 
```

```
ex1: /tmp/ex1.c:3: main: Assertion `1 == 0' failed. 
```

但我也可以通过 cbmc 运行源代码。

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

我们也可以编译这个程序。它运行得很好。

```
! gcc /tmp/ex2.c -o /tmp/ex2 && /tmp/ex2 
```

然而，CBMC 运行所有可能的执行路径，发现了未初始化变量可能采用的错误值。我们也可以使用`--trace`选项逐行跟踪执行。

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

如果我们把这个断言放在一个守护 if 中，就不可能再触发这个断言了。

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

这是一个简单的功能正确性示例。我们可以编写一个`myabs`函数并询问结果是否始终为正。

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

说什么？？？？？？？？？哦，对。通过跟踪，看到它选了负的 maxint。在二进制补码中，负数的数量比正数多一个。所以实际上不清楚编译器应该在这里做什么。我猜它可以做任何事情。

这是你更可能关心的东西，检查常见的内存错误，如缓冲区溢出等

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

哇！？哪里有 bug？！

是的，CBMC 需要标志来打开这些默认检查，无论好坏。我有点希望有一个做所有事情的标志，但据我所知没有。这里有一堆可能有用的 bug 可以查找。

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

# 下次

我认为今天的内容就够了，但我们只是触及了表面。但为了激发一些兴趣，这里是关于循环和比较检查的一些未编辑的片段。

[`www.youtube.com/embed/AUsNTNq0dbY?si=zfELEKMDjHjIytVZ`](https://www.youtube.com/embed/AUsNTNq0dbY?si=zfELEKMDjHjIytVZ)

视频。

## 循环。

好吧，到目前为止，我们已经描绘了一个美好的画面。显然，CBMC 不会扩展到任意大型和困难的问题。

基本技术是展开它们。一切并没有失去，许多循环（特别是 for 循环）可以完全展开。您可以添加 `--unwinding-assertions` 以了解是否已覆盖所有可能的执行。即使你不能，通过这些检查确实会给你一些信心。

如果您要进入下一个级别，还可以添加不变量注释的功能。我不确定 CBMC 是否能成功推断这些。

在更大的问题上，我在调试方面取得了一些成功。

## 比较检查。

程序的比较检查对于减少规范和验证负担都很有用。此外，还有一些关系属性（“超属性”）需要讨论程序的两个不同运行。典型的例子是信息安全。如果您运行两个具有相同低安全状态部分的程序，它们必须以相同低安全状态部分结束。如果只有从低到高的影响或信息传递，这是正确的。

+   翻译验证。

+   精炼。

+   安全属性。

+   修复错误。

进行比较验证的最简单方法是在“产品程序”上使用更常见的单个程序验证器。

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

有许多软件存在安全漏洞。

显然，最简单且最可能的解决方案是在源代码中更改代码，重新编译并推送新版本。然而，这并不总是可能或可取的。

+   即使重新编译原始程序也可能因为编译器版本的不同而引入错误。

+   ## 您可能没有源代码。

总的来说，考虑 50 年的软件堆栈是一个有趣的问题。

更糟的是更好：一个愚蠢编译器的用例。

# 一些小东西。

这是一个带有 CBMC 仪器的严肃 C 项目列表，其中许多来自亚马逊：[`model-checking.github.io/cbmc-training/projects.html`](https://model-checking.github.io/cbmc-training/projects.html)。它们在库函数的单元测试中扮演着类似的角色。

SV Comp [`sv-comp.sosy-lab.org/`](https://sv-comp.sosy-lab.org/) - CPAchecker UAutomizer 在竞赛中非常成功。我没有多少使用过它们。

Klee，symcc 是符号执行器。在许多方面类似于有界模型检查器。我认为最大的哲学区别在于它们并不真正围绕着确保不存在错误，而是围绕着查找错误，这有些不同。

Frama-C VST [`github.com/verifast/verifast`](https://github.com/verifast/verifast) 格雷厄姆似乎喜欢这个。

[esbmc](http://esbmc.org/) 是一个 C 有界模型检查器。

将 C 函数与它的语法进行比较？

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

简单易用 [首页](https://www.cprover.org/cbmc/) [`arxiv.org/abs/2302.02384`](https://arxiv.org/abs/2302.02384) `sudo apt install cbmc` [`github.com/diffblue/cbmc`](https://github.com/diffblue/cbmc) github [`diffblue.github.io/cbmc/`](https://diffblue.github.io/cbmc/) 文档

[`github.com/diffblue/aws-training`](https://github.com/diffblue/aws-training) [`model-checking.github.io/cbmc-training/`](https://model-checking.github.io/cbmc-training/) [`github.com/model-checking/cbmc-starter-kit`](https://github.com/model-checking/cbmc-starter-kit) 入门套件模板 [`model-checking.github.io/cbmc-starter-kit/tutorial/index.html`](https://model-checking.github.io/cbmc-starter-kit/tutorial/index.html) 对 maloc 进行检测 [`github.com/model-checking/cbmc-proof-debugger`](https://github.com/model-checking/cbmc-proof-debugger)

[手册](http://www.cprover.org/cprover-manual/) 参见教程

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

各种分析选项

用户通过 `assert` 或专用功能定义的内容

支持多种不同的 SMT 和 SAT 后端。通用 dimacs，可以投入 kissat

ESBMC

+   C++ 前端

+   浮点支持

+   仍然在实际开发中

[`awslabs.github.io/aws-proof-build-assistant/`](https://awslabs.github.io/aws-proof-build-assistant/) [`github.com/awslabs/aws-c-common`](https://github.com/awslabs/aws-c-common) corejson [`github.com/FreeRTOS/coreJSON/tree/main/test/cbmc`](https://github.com/FreeRTOS/coreJSON/tree/main/test/cbmc) s2n-tls [`github.com/aws/s2n-tls/tree/main/tests/cbmc`](https://github.com/aws/s2n-tls/tree/main/tests/cbmc) [`github.com/aws/aws-encryption-sdk-c/tree/master/verification/cbmc`](https://github.com/aws/aws-encryption-sdk-c/tree/master/verification/cbmc) [`github.com/aws/s2n-quic`](https://github.com/aws/s2n-quic)

有界证明 vs 起搏器 vs 合同

[`crates.io/crates/libcprover_rust/5.91.0`](https://crates.io/crates/libcprover_rust/5.91.0) rust api

[`dl.acm.org/doi/pdf/10.1145/3551349.3559523`](https://dl.acm.org/doi/pdf/10.1145/3551349.3559523) CBMC-SSM：使用符号化阴影内存对 C 程序进行有界模型检查 [`github.com/diffblue/cbmc/issues/7757`](https://github.com/diffblue/cbmc/issues/7757)

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
