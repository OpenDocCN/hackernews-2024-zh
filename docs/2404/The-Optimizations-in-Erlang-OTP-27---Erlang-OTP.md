<!--yml

category: 未分类

date: 2024-05-27 13:31:10

-->

# Erlang/OTP 27 的优化 - Erlang/OTP

> 来源：[https://www.erlang.org/blog/optimizations/](https://www.erlang.org/blog/optimizations/)

本文探讨了记录更新的新优化以及其他一些改进。还简要回顾了导致 Erlang/OTP 27 的近期优化历史。

### 近期优化简史 [#](#a-brief-history-of-recent-optimizations)

Erlang 的现代优化历史始于 2018 年 1 月。我们意识到在 Erlang 编译器中处理[BEAM 代码](https://www.erlang.org/blog/a-brief-beam-primer/)的优化已达到可能性的极限。

+   Erlang/OTP 22 在编译器中引入了新的[基于 SSA 的中间表示](https://en.wikipedia.org/wiki/Static_single-assignment_form)。在 [SSA 历史](https://www.erlang.org/blog/ssa-history/)中阅读完整故事。

+   Erlang/OTP 24 引入了[JIT（即时编译器）](https://www.erlang.org/blog/a-first-look-at-the-jit/)，通过在加载时为 BEAM 指令生成本地代码来提高性能。

+   Erlang/OTP 25 引入了[基于类型的 JIT 优化](https://www.erlang.org/blog/type-based-optimizations-in-the-jit/)，使得 Erlang 编译器能够将类型信息传递给 JIT，帮助 JIT 生成更好的本地代码。虽然这改善了 JIT 生成的本地代码，但编译器和 JIT 的限制阻止 JIT 充分利用类型信息。

+   Erlang/OTP 26 [改进了基于类型的优化](https://www.erlang.org/blog/more-optimizations/)。最显著的性能改进是使用位语法进行匹配和二进制构造。这些改进与 `base64` 模块本身的更改结合使用，使得编码到 Base64 快约 4 倍，解码从 Base64 快超过 3 倍。

### Erlang/OTP 27 中 JIT 的预期效果 [#](#what-to-expect-of-the-jit-in-erlangotp-27)

Erlang/OTP 27 中主要的编译器和 JIT 改进是记录操作的优化，但也有许多较小的优化，使代码更小和/或更快。

### 请在家里试试这个！ [#](#please-try-this-at-home)

虽然本博客文章将展示许多生成代码的例子，但我也尝试用英语解释优化。随意跳过代码示例。

另一方面，如果您需要更多的代码示例……

要检查已加载模块的本地代码，请按以下方式启动运行时系统：

```
erl +JDdump true 
```

所有已加载模块的本地代码将转储到扩展名为 `.asm` 的文件中。

要检查模块的 BEAM 代码，请在编译时使用 `-S` 选项。例如：

```
erlc -S base64.erl 
```

### 一个简单的记录优化 [#](#a-simple-record-optimization)

首先，让我们看看一个简单的记录优化，它在 Erlang/OTP 26 及更早版本中未完成。假设我们有这个模块：

```
-record(foo, {a,b,c,d,e}).

update(N) ->
    R0 = #foo{},
    R1 = R0#foo{a=N},
    R2 = R1#foo{b=2},
    R2#foo{c=3}. 
```

这里是关于记录操作的[BEAM 代码](https://www.erlang.org/blog/a-brief-beam-primer/)：

```
 {update_record,{atom,reuse},
                   6,
                   {literal,{foo,undefined,undefined,undefined,undefined,
                                 undefined}},
                   {x,0},
                   {list,[2,{x,0}]}}.
    {update_record,{atom,copy},6,{x,0},{x,0},{list,[3,{integer,2}]}}.
    {update_record,{atom,copy},6,{x,0},{x,0},{list,[4,{integer,3}]}}. 
```

换句话说，所有三个记录更新操作都保留为单独的[`update_record`](https://www.erlang.org/blog/more-optimizations/#updating-records-in-otp-26)指令。每个操作通过复制记录的未更改部分并在正确位置填入新值来创建一个新记录。

Erlang/OTP 27 中的编译器实际上将 `update/1` 重写为：

```
update(N) ->
    #foo{a=N,b=2,c=3}. 
```

这将为记录创建生成以下 BEAM 代码：

```
 {put_tuple2,{x,0},
                {list,[{atom,foo},
                       {x,0},
                       {integer,2},
                       {integer,3},
                       {atom,undefined},
                       {atom,undefined}]}}. 
```

这些优化是通过以下拉取请求实现的：

### 在原地更新记录[#](#updating-records-in-place)

要探索引入于 Erlang/OTP 27 中的更复杂的记录优化，请考虑以下示例：

```
-module(count1).
-export([count/1]).

-record(s, {atoms=0,other=0}).

count(L) ->
    count(L, #s{}).

count([X|Xs], #s{atoms=C}=S) when is_atom(X) ->
    count(Xs, S#s{atoms=C+1});
count([_|Xs], #s{other=C}=S) ->
    count(Xs, S#s{other=C+1});
count([], S) ->
    S. 
```

`count(List)` 计算给定列表中的原子数和其他项的数量。例如：

```
1> -record(s, {atoms=0,other=0}).
ok
2> count1:count([a,b,c,1,2,3,4,5]).
#s{atoms = 3,other = 5} 
```

以下是 `count/2` 所发出的 BEAM 代码：

```
 {test,is_nonempty_list,{f,6},[{x,0}]}.
    {get_list,{x,0},{x,2},{x,0}}.
    {test,is_atom,{f,5},[{x,2}]}.
    {get_tuple_element,{x,1},1,{x,2}}.
    {gc_bif,'+',{f,0},3,[{tr,{x,2},{t_integer,{0,'+inf'}}},{integer,1}],{x,2}}.
    {test_heap,4,3}.
    {update_record,{atom,inplace},
                   3,
                   {tr,{x,1},
                       {t_tuple,3,true,
                                #{1 => {t_atom,[s]},
                                  2 => {t_integer,{0,'+inf'}},
                                  3 => {t_integer,{0,'+inf'}}}}},
                   {x,1},
                   {list,[2,{tr,{x,2},{t_integer,{1,'+inf'}}}]}}.
    {call_only,2,{f,4}}. % count/2
  {label,5}.
    {get_tuple_element,{x,1},2,{x,2}}.
    {gc_bif,'+',{f,0},3,[{tr,{x,2},{t_integer,{0,'+inf'}}},{integer,1}],{x,2}}.
    {test_heap,4,3}.
    {update_record,{atom,inplace},
                   3,
                   {tr,{x,1},
                       {t_tuple,3,true,
                                #{1 => {t_atom,[s]},
                                  2 => {t_integer,{0,'+inf'}},
                                  3 => {t_integer,{0,'+inf'}}}}},
                   {x,1},
                   {list,[3,{tr,{x,2},{t_integer,{1,'+inf'}}}]}}.
    {call_only,2,{f,4}}. % count/2
  {label,6}.
    {test,is_nil,{f,3},[{x,0}]}.
    {move,{x,1},{x,0}}.
    return. 
```

前两个指令测试 `{x,0}` 中的第一个参数是否为非空列表，并在是的情况下提取列表的第一个元素：

```
 {test,is_nonempty_list,{f,6},[{x,0}]}.
    {get_list,{x,0},{x,2},{x,0}}. 
```

接下来的指令测试第一个元素是否为原子。如果不是，则跳转到第二个子句的代码。

```
 {test,is_atom,{f,5},[{x,2}]}. 
```

接下来从记录中获取看到的原子数计数器，并将其增加一：

```
 {get_tuple_element,{x,1},2,{x,2}}.
    {gc_bif,'+',{f,0},3,[{tr,{x,2},{t_integer,{0,'+inf'}}},{integer,1}],{x,2}}. 
```

然后是堆空间的分配和记录的更新：

```
 {test_heap,4,3}.
    {update_record,{atom,inplace},
                   3,
                   {tr,{x,1},
                       {t_tuple,3,true,
                                #{1 => {t_atom,[s]},
                                  2 => {t_integer,{0,'+inf'}},
                                  3 => {t_integer,{0,'+inf'}}}}},
                   {x,1},
                   {list,[3,{tr,{x,2},{t_integer,{1,'+inf'}}}]}}. 
```

`test_heap` 指令确保堆上有足够的空间来复制记录（4 个字）。

`update_record` 指令是在 Erlang/OTP 26 中引入的。其第一个操作数是一个原子，用于帮助 JIT 发出更好的代码。在 Erlang/OTP 26 中，使用了 `reuse` 和 `copy` 这两个提示。有关这些提示的更多信息，请参阅[OTP 26 中的记录更新](https://www.erlang.org/blog/more-optimizations/#updating-records-in-otp-26)。

在 Erlang/OTP 27 中，引入了一个名为 `inplace` 的新提示。当编译器确定运行时系统中除了用于 `update_record` 指令的引用之外，不存在对元组的其他引用时，编译器会发出此提示。换句话说，从**编译器**的角度来看，如果运行时系统直接更新现有记录而不先复制它，程序的可观察行为不会改变。不过，从**运行时系统**的角度来看，直接更新记录并不总是安全。

此新优化由 Frej Drejhammar 实现。它在 Erlang/OTP 26 中添加的编译器通行证的基础上进行了扩展，用于[向二进制附加内容](https://www.erlang.org/blog/more-optimizations/#appending-to-binaries-in-otp-26)。

现在让我们看看当 `record_update` 指令带有 `inplace` 提示时，JIT 会做什么。以下是该指令的完整本地代码：

```
# update_record_in_place_IsdI
    mov rax, qword ptr [rbx+8]
    mov rcx, qword ptr [rbx+16]
    test cl, 1
    short je L38           ; Update directly if small integer.

    ; The new value is a bignum.
    ; Test whether the tuple is in the safe part of the heap.

    mov rdi, [r13+480]     ; Get the high water mark
    cmp rax, r15           ; Compare tuple pointer to heap top
    short jae L39          ; Jump and copy if above
    cmp rax, rdi           ; Compare tuple pointer to high water
    short jae L38          ; Jump and overwrite if above high water

    ; The tuple is not in the safe part of the heap.
    ; Fall through to the copy code.

L39:                       ; Copy the current record
    vmovups ymm0, [rax-2]
    vmovups [r15], ymm0
    lea rax, qword ptr [r15+2] ; Set up tagged pointer to copy
    add r15, 32            ; Advance heap top past the copy

L38:
    mov rdi, rcx           ; Get new value for atoms field
    mov qword ptr [rax+22], rdi
    mov qword ptr [rbx+8], rax 
```

（以 `#` 开头的行是 JIT 发出的注释，而跟在 `;` 后面的文本是我添加的说明。）

BEAM加载器将带有`inplace`提示的`update_record`指令重命名为`update_record_in_place`。

前两个指令将要更新的元组加载到 CPU 寄存器 `rax` 中，并将新的计数器值 (`C + 1`) 加载到 `rcx` 中。

```
 mov rax, qword ptr [rbx+8]
    mov rcx, qword ptr [rbx+16] 
```

下面的两条指令测试新的计数器值是否是适合字的小整数。这个测试已简化为更有效的测试，只有在值已知为整数时才安全。如果是小整数，则始终可以安全地跳转到更新现有元组的代码：

```
 test cl, 1
    short je L38           ; Update directly if small integer. 
```

如果不是小整数，那么它必须是**大整数**，即不适合 60 位的有符号整数，因此必须将其与 `rcx` 包含的标记指针存储到堆中。

如果`rcx`是指向堆上术语的指针，则直接更新现有元组并不总是安全的。这是因为 Erlang [分代垃圾回收器](https://www.erlang.org/doc/apps/erts/garbagecollection#generational-garbage-collection) 的工作方式。每个 Erlang 进程都有两个用于保存 Erlang 术语的堆：年轻堆和老堆。允许年轻堆上的术语引用老堆上的术语，但反之则不行。这意味着如果要更新的元组位于老堆上，则不能安全地更新其元素，使其引用年轻堆上的术语。

因此，JIT 需要生成代码以确保元组的指针位于年轻堆的“安全部分”：

```
 mov rdi, [r13+480]     ; Get the high water mark
    cmp rax, r15           ; Compare tuple pointer to heap top
    short jae L39          ; Jump and copy if above
    cmp rax, rdi           ; Compare tuple pointer to high water
    short jae L38          ; Jump and overwrite if above high water 
```

堆的安全部分位于高水位标和堆顶之间。如果元组位于高水位标以下且仍然存活，则在下一次垃圾回收时将其复制到老堆中。

如果元组位于安全部分，则跳过复制代码，直接跳转到将新值存储到现有元组的代码。

如果不是，则下一部分将复制现有记录到堆中。

```
L39:                       ; Copy the current record
    vmovups ymm0, [rax-2]
    vmovups [r15], ymm0
    lea rax, qword ptr [r15+2] ; Set up tagged pointer to copy
    add r15, 32            ; Advance heap top past the copy 
```

使用[AVX指令](https://zh.wikipedia.org/wiki/高级矢量扩展)进行复制。

接下来是将新值写入元组的代码：

```
L38:
    mov rdi, rcx           ; Get new value for atoms field
    mov qword ptr [rax+22], rdi
    mov qword ptr [rbx+8], rax 
```

如果所有写入现有记录的新值都已知永不标记为指针，则可以简化本机指令。考虑这个模块：

```
-module(whatever).
-export([main/1]).

-record(bar, {bool,pid}).

main(Bool) when is_boolean(Bool) ->
    flip_state(#bar{bool=Bool,pid=self()}).

flip_state(R) ->
    R#bar{bool=not R#bar.bool}. 
```

`update_record` 指令看起来像这样：

```
 {update_record,{atom,inplace},
                   3,
                   {tr,{x,0},
                       {t_tuple,3,true,
                                #{1 => {t_atom,[bar]},
                                  2 => {t_atom,[false,true]},
                                  3 => pid}}},
                   {x,0},
                   {list,[2,{tr,{x,1},{t_atom,[false,true]}}]}}. 
```

根据新值的类型 `{t_atom,[false,true]}`，JIT 能够生成比前面例子更短的代码：

```
# update_record_in_place_IsdI
    mov rax, qword ptr [rbx]
# skipped copy fallback because all new values are safe
    mov rdi, qword ptr [rbx+8]
    mov qword ptr [rax+14], rdi
    mov qword ptr [rbx], rax 
```

对文字的引用（例如 `[1,2,3]`）也是安全的，因为文字存储在特殊的文字区域中，垃圾回收器对它们有特殊处理。考虑这段代码：

```
-record(state, {op, data}).

update_state(R0, Op0, Data) ->
    R = R0#state{data=Data},
    case Op0 of
        add -> R#state{op=fun erlang:'+'/2};
        sub -> R#state{op=fun erlang:'-'/2}
    end. 
```

`case` 中的两个记录更新都可以原地完成。这是第一个条目的记录更新的 BEAM 代码：

```
 {update_record,{atom,inplace},
                   3,
                   {tr,{x,0},{t_tuple,3,true,#{1 => {t_atom,[state]}}}},
                   {x,0},
                   {list,[2,{literal,fun erlang:'+'/2}]}}. 
```

由于要写入的值是文字，JIT 发出了更简单的代码而不带复制回退：

```
# update_record_in_place_IsdI
    mov rax, qword ptr [rbx]
# skipped copy fallback because all new values are safe
    long mov rdi, 9223372036854775807  ; Placeholder for address to fun
    mov qword ptr [rax+14], rdi
    mov qword ptr [rbx], rax 
```

大整数 `9223372036854775807` 是一个占位符，稍后将在知道文字 fun 地址时进行修补。

这是更新元组的拉取请求：

### 通过生成更少垃圾来进行优化 [#](#optimizing-by-generating-less-garbage)

在原地更新记录时，省略复制现有记录应该是一个明显的优势，除了可能对非常小的记录外。

更不清楚的是对垃圾回收的影响。更新元组是通过生成更少的垃圾来进行优化的一个例子。通过减少垃圾生成，期望是垃圾收集发生的频率应该减少，从而提高程序的性能。

因为垃圾回收的执行时间变化很大，优化减少垃圾创建的难度非常大。通常情况下，基准测试的结果不能应用于在真实应用中执行相同任务。

我自己的[轶事证据](https://en.wikipedia.org/wiki/Anecdotal_evidence)表明，在大多数情况下，通过生成更少的垃圾，性能上并没有可测量的胜利。

我还记得，当一项减少 Erlang 术语大小的优化导致基准测试始终较慢时。该优化的作者花了几天时间进行调查，以确认基准测试的减速并非他的优化所致，而是通过生成更少的垃圾，垃圾收集发生在稍后的时间点时，成本更高。

平均而言，我们预计这种优化应该改善性能，特别是对于大型记录而言。

### 函数优化 [#](#optimization-of-funs)

Erlang/OTP 运行时系统中的函数内部表示在 Erlang/OTP 27 中已更改，从而可能实现几种新的优化。

举例来说，考虑这个函数：

```
madd(A, C) ->
    fun(B) -> A * B + C end. 
```

在 Erlang/OTP 26 中，创建函数的本机代码如下所示：

```
# i_make_fun3_FStt
L38:
    long mov rsi, 9223372036854775807 ; Placeholder for dispatch table
    mov edx, 1
    mov ecx, 2
    mov qword ptr [r13+80], r15
    mov rbp, rsp
    lea rsp, qword ptr [rbx-128]
    vzeroupper
    mov rdi, r13
    call 4337160320       ; Call helper function in runtime system
    mov rsp, rbp
    mov r15, qword ptr [r13+80]
# Move fun environment
    mov rdi, qword ptr [rbx]
    mov qword ptr [rax+40], rdi
    mov rdi, qword ptr [rbx+8]
    mov qword ptr [rax+48], rdi
# Create boxed ptr
    or al, 2
    mov qword ptr [rbx], rax 
```

大整数 `9223372036854775807` 是一个占位符，稍后将填充一个值。

实际上创建函数对象的大部分工作是通过调用运行时系统中的辅助函数（`call 4337160320` 指令）完成的。

在 Erlang/OTP 27 中，存在于调用进程堆上的函数部分已经简化，因此现在比 Erlang/OTP 26 中小，最重要的是不包含任何在内联代码中初始化的复杂内容。

创建函数的代码不仅更短，而且不需要调用运行时系统中的任何函数：

```
# i_make_fun3_FStt
L38:
    long mov rax, 9223372036854775807 ; Placeholder for dispatch table
# Create fun thing
    mov qword ptr [r15], 196884
    mov qword ptr [r15+8], rax
# Move fun environment
# (moving two items)
    vmovups xmm0, xmmword ptr [rbx]
    vmovups xmmword ptr [r15+16], xmm0
L39:
    long mov rdi, 9223372036854775807 ; Placeholder for fun reference
    mov qword ptr [r15+32], rdi
# Create boxed ptr
    lea rax, qword ptr [r15+2]
    add r15, 40
    mov qword ptr [rbx], rax 
```

与 Erlang/OTP 26 的区别在于，加载和卸载代码时仅需要的函数部分不再存储在堆上。而是存储在属于模块加载代码的文字池区域，并且被所有相同函数实例共享。

与 Erlang/OTP 26 相比，存储在进程堆上的函数部分要小两个字。

函数环境的创建也已经优化。在 Erlang/OTP 26 中，需要四条指令：

```
# Move fun environment
    mov rdi, qword ptr [rbx]
    mov qword ptr [rax+40], rdi
    mov rdi, qword ptr [rbx+8]
    mov qword ptr [rax+48], rdi 
```

在 Erlang/OTP 27 中，使用[AVX 指令](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions)，变量 (`A` 和 `C`) 可以仅使用两条指令移动：

```
# Move fun environment
# (moving two items)
    vmovups xmm0, xmmword ptr [rbx]
    vmovups xmmword ptr [r15+16], xmm0 
```

由于改变的函数表示方式，还可以测试一个具有特定 arity 的函数（在调用时期望的参数数量）。例如：

```
ensure_fun_0(F) when is_function(F, 0) -> ok. 
```

这是 Erlang/OTP 26 中 JIT 发出的本地代码：

```
# is_function2_fss
    mov rdi, qword ptr [rbx]   ; Fetch `F` from {x,0}.

    rex test dil, 1            ; Test whether the term is a tagged pointer...
    short jne label_3          ; ... otherwise fail.

    mov eax, dword ptr [rdi-2] ; Pick up the header word.
    cmp eax, 212               ; Test whether it is a fun...
    short jne label_3          ; ... otherwise fail.

    cmp byte ptr [rdi+22], 0   ; Test whether the arity is 0...
    short jne label_3          ; ... otherwise fail. 
```

在 Erlang/OTP 27 中，函数的 arity（预期参数数量）存储在函数术语的头部字中，这意味着函数的测试可以与其 arity 的测试结合：

```
# is_function2_fss
    mov rdi, qword ptr [rbx]   ; Fetch `F` from {x,0}.

    rex test dil, 1            ; Test whether the term is a tagged pointer...
    short jne label_3          ; ... otherwise fail.

    cmp word ptr [rdi-2], 20   ; Test whether this is a fun with arity 0...
    short jne label_3          ; ... otherwise fail. 
```

所有外部函数现在都是存储在所有进程堆之外的文字。例如，请考虑以下函数：

```
my_fun() ->
    fun ?MODULE:some_function/0.

mfa(M, F, A) ->
    fun M:F/A. 
```

在 Erlang/OTP 26 中，由 `my_fun/0` 返回的外部函数不会占据调用进程的堆空间，而由 `mfa/3` 返回的动态外部函数则需要占据调用进程的堆空间 5 个字。

在 Erlang/OTP 27 中，任何一个函数都不需要占用调用进程的堆空间。

这些优化已在以下拉取请求中实现：

### 整数算术改进 [#](#integer-arithmetic-improvements)

在去年六月底，我们发布了 Erlang/OTP 26 的[OTP 26.0.2 补丁](https://www.erlang.org/patches/otp-26.0.2)，使得 [`binary_to_integer/1`](https://www.erlang.org/doc/man/erlang#binary_to_integer-1) 更快。

要了解速度提升多少，请运行此基准测试：

```
bench() ->
    Size = 1_262_000,
    String = binary:copy(<<"9">>, Size),
    {Time, _Val} = timer:tc(erlang, binary_to_integer, [String]),
    io:format("Size: ~p, seconds: ~p\n", [Size, Time / 1_000_000]). 
```

它测量将包含 1,262,000 位数的二进制转换为整数所需的时间。

在我的基于 Intel 的 2017 年款 iMac 上运行未打补丁的 Erlang/OTP 26，基准测试大约需要 10 秒完成。

使用 Erlang/OTP 26.0.2 运行相同的基准测试大约需要 0.4 秒完成。

这种加速是通过三个单独的优化实现的：

+   `binary_to_integer/1` 最初在 C 中作为 BIF 实现，使用一个不太适合扩展的朴素算法。它被 Erlang 中实现的[分治算法](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm)替换。 （将新算法作为 BIF 实现并没有比 Erlang 版本更快。）

+   运行时系统的用于执行大整数乘法的函数已修改为使用[卡拉茨巴算法](https://en.wikipedia.org/wiki/Karatsuba_algorithm)，这是一种在 1960 年代发明的分治乘法算法。

+   当 C 编译器支持时，一些用于大整数（bignums）算术的低级辅助函数已修改为利用 64 位 CPU 上支持的 128 位整数数据类型。

这些改进已在以下拉取请求中实现：

在 Erlang/OTP 27 中，实施了一些额外的整数算术改进。这将 `binary_to_integer/1` 基准测试的执行时间减少到大约 0.3 秒。

这些改进在以下拉取请求中找到：

这些算术增强改善了[pidigits benchmark](https://benchmarksgame-team.pages.debian.net/benchmarksgame/program/pidigits-erlang-2.html)的运行时间：

| 版本 |   | 秒数 |
| --- | --- | --- |
| 26.0 |   | `7.635` |
| 26.2.1 |   | `2.959` |
| 27.0 |   | `2.782` |

（在我的 M1 MacBook Pro 上运行。）

### 无数的杂项增强 [#](#numerous-miscellaneous-enhancements)

对许多指令的代码生成进行了许多增强，以及对 Erlang 编译器的少数增强。以下是一个示例，展示了`=:=`运算符的改进之一：

```
ensure_empty_map(Map) when Map =:= #{} ->
    ok. 
```

这里是作为本示例中使用的`=:=`运算符的 BEAM 代码：

```
 {test,is_eq_exact,{f,1},[{x,0},{literal,#{}}]}. 
```

这是 Erlang/OTP 26 的原生代码：

```
# is_eq_exact_fss
L45:
    long mov rsi, 9223372036854775807
    mov rdi, qword ptr [rbx]
    cmp rdi, rsi
    short je L44                  ; Succeeded if the same term.

    rex test dil, 1
    short jne label_1             ; Fail quickly if not a tagged pointer.

    ; Call the general runtime function for comparing two terms.
    mov rbp, rsp
    lea rsp, qword ptr [rbx-128]
    vzeroupper
    call 4549723200
    mov rsp, rbp

    test eax, eax
    short je label_1               ; Fail if unequal.
L44: 
```

代码以几个快速成功或失败的测试开始，但在实践中，这些测试不太可能触发，这意味着几乎总是会调用运行时系统中的通用例程来比较两个术语。

在 Erlang/OTP 27 中，即时编译器发出特殊代码来测试一个术语是否为空映射：

```
# is_eq_exact_fss
# optimized equality test with empty map
    mov rdi, qword ptr [rbx]
    rex test dil, 1
    short jne label_1              ; Fail if not a tagged pointer.

    cmp dword ptr [rdi-2], 300
    short jne label_1              ; Fail if not a map.

    cmp dword ptr [rdi+6], 0
    short jne label_1              ; Fail if size is not zero. 
```

接下来是 Erlang/OTP 27 中杂项增强的主要拉取请求：
