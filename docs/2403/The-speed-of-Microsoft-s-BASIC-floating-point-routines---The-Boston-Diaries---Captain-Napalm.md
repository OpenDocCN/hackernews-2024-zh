<!--yml

category: 未分类

date: 2024-05-27 14:30:56

-->

# 微软的 BASIC 浮点运算速度 - 波士顿日记 - 娜潘船长

> 来源：[https://boston.conman.org/2024/03/01.1](https://boston.conman.org/2024/03/01.1)

我对微软的 BASIC 浮点运算速度很好奇。现在我可以在汇编器内部[测试汇编代码的时间](/2023/12/19.3)，这很容易测试。代码使用 Color BASIC 例程、IEEE-754 单精度和双精度计算了 -2π³/3!。

首先是 Color BASIC：

```
	.tron	timing
ms_fp		ldx	#.tau
		jsr	CB.FP0fx	; FP0 = .tau
		ldx	#.tau
		jsr	CB.FMULx	; FP0 = FP0 * .tau
		ldx	#.tau
		jsr	CB.FMULx	; FP0 = FP0 * .tau
		jsr	CB.FP1f0	; FP1 = FP0
		ldx	#.fact3
		jsr	CB.FP0fx	; FP0 = 3!
		jsr	CB.FDIV		; FP0 = FP1 / FP0
		neg	CB.fp0sgn	; FP0 = -FP0
		ldx	#.answer
		jsr	CB.xfFP0	; .answer = FP0
	.troff
		rts

.tau		fcb	$83,$49,$0F,$DA,$A2 
.fact3		fcb	$83,$40,$00,$00,$00  
.answer		rmb	5
		fcb	$86,$A5,$5D,$E7,$30	; precalculated result

```

我无法在这里使用 `.FLOAT` 指令，因为它只支持微软格式或 IEEE-754，但不能同时支持两者。所以对于这个测试，我必须为每个浮点数定义单独的字节。最后一行是运行后检查虚拟机内存转储的结果。另外，`.tao` 是 [2π](https://tauday.com/tau-manifesto)，以防不清楚。这个测试在 8,742 个周期内完成，包括 2,124 条指令，平均每条指令 4.12 个周期（我修改了汇编器以记录这些额外信息）。

接下来是 IEEE-754 单精度：

```
	.tron	timing
ieee_single	ldu	#.tau
		ldy	#.tau
		ldx	#.answer
		ldd	#.fpcb
		jsr	REG
		fcb	FMUL	; .answer = .tau * .tau

		ldu	#.tau
		ldy	#.answer
		ldx	#.answer
		ldd	#.fpcb
		jsr	REG
		fcb	FMUL	; .answer = .answer * .tau

		ldu	#.answer
		ldy	#.fact3
		ldx	#.answer
		ldd	#.fpcb
		jsr	REG
		fcb	FDIV	; .answer = .answer / 3!

		ldy	#.answer
		ldx	#.answer
		ldd	#.fpcb
		jsr	REG
		fcb	FNEG	; .answer = -.answer
	.troff
		rts

.fpcb		fcb	FPCTL.single | FPCTL.rn | FPCTL.proj
		fcb	0
		fcb	0
		fcb	0
		fdb	0

.tau		.float	6.283185307
.fact3		.float	3!
.answer		.float	0
		.float	-(6.283185307 ** 3 / 3!)

```

浮点控制块（`.fpcb`）配置了 MC6839 使用单精度、普通舍入和投影封闭（不确定是什么，但这是默认值）。并且它确实计算出了正确的结果。令人惊讶的是，42 *年* 前为 8 位 CPU 编写的代码可以无缺地运行。它*不*快的是。这段代码在 14,204 个周期内完成了 2,932 条指令（平均每条指令 4.84 个周期）。

高于平均周期类型可能是由于位置无关寻址模式，但我并不完全确定它为何需要将时间增加近两倍。ROM 在内部确实使用 IEEE-754 扩展格式（10 字节），需要更多的位移来提取指数和尾数，但是*两倍*的时间呢？

或许这是用来处理 ±∞ 和 NaN 的代码。

IEEE-754 双精度相同，除了浮点控制块配置为双精度和使用 `.FLOATD` 而不是 `.FLOAT`，其他代码都相同。然而，结果却不同。它在 31,613 个周期内完成了 6,865 条指令，平均每条指令 4.60 个周期。尺寸增加了近两倍，因此花费的时间也接近两倍，这是预期的结果。

最后一段代码只是将 ROM 装入内存，并调用每个函数以获取时间：

```
		org	$2000
		incbin	"mc6839.rom"
REG		equ	$203D	; register-based entry point

		org	$A000
		incbin	"bas12.rom"

	.opt	test	prot	rw,$00,$FF	; Direct Page for BASIC
	.opt	test	prot	rx,$2000,$2000+8192 ; MC6839 ROM
	.opt	test	prot	rx,$A000,$A000+8192 ; BASIC ROM

	.test	"BASIC"
		lbsr	ms_fp
		rts
	.endtst

	.test	"IEEE-SINGLE"
		lbsr	ieee_single
		rts
	.endtst

	.test	"IEEE-DOUBLE"
		lbsr	ieee_double
		rts
	.endtst

```

真正令人惊讶的是微软的 BASIC 浮点运算有多快。

欢迎链接到这里的任何条目。别客气，我不会咬人的。我保证。

日期是那天条目（如果只有一个条目，则为条目）的永久链接。标题是仅限于*该*条目的永久链接。链接的格式很简单：从此站点的基础链接开始：[https://boston.conman.org/](https://boston.conman.org/)，然后添加您感兴趣的日期，比如[2000/08/01](/2000/08/01)，因此最终的URL是：

[https://boston.conman.org/2000/08/01](https://boston.conman.org/2000/08/01)

您还可以通过省略日期部分来指定整个月。您甚至可以选择[任意时间段。](/about/technical.html)

你可能还会注意到链接的微妙阴影，这是有意为之的：链接距离页面越“近”，它看起来就越“亮”。这是一种使用颜色阴影来表示链接距离的实验。如果你没有注意到，不要担心；这并不是*那么*重要。

假设这些页面中提到的每个品牌名称、口号、公司名称、符号、设计元素等都是受保护和/或注册商标实体，其所有者独有，对此状态的确认被暗示在内。
