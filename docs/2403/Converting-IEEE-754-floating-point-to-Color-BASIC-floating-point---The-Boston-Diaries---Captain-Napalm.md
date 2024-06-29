<!--yml

类别：未分类

日期：2024-05-27 14:32:56

-->

# 将 IEEE-754 浮点数转换为 Color BASIC 浮点数 - The Boston Diaries - Captain Napalm

> 来源：[https://boston.conman.org/2024/02/28.1](https://boston.conman.org/2024/02/28.1)

我仍在研究 [6809 上的浮点数](/2024/02/07.1) ——特别是对 Color Computer 的浮点数支持。Color BASIC 的浮点数格式（由 Microsoft 编写）早于几年的 [IEEE-754 浮点数标准](https://codedocs.org/what-is/ieee-754) 并因此不完全兼容。不过它们很接近。它定义了一个 8 位指数，偏置为 129，一个单独的符号位（在指数之后），以及 31 位用于尾数（假定了前导的 1）。它也 *不* 支持 ±∞ 或 NaN。这与 IEEE-754 单精度有所不同，后者使用一个单独的符号位，一个偏置为 127 的 8 位指数，以及 23 位尾数（也假定了一个前导的 1），并支持 ∞ 和 NaN。IEEE-754 双精度使用一个单独的符号位，一个偏置为 1023 的 11 位指数，52 位尾数（假定了前导的 1），以及支持 ∞ 和 NaN。

因此，Color BASIC 处于单精度和双精度之间的中间位置。这导致我在 Color Computer 后端使用 IEEE-754 双精度（生成无穷大和 NaN 的错误）[然后将得到的双精度数据转换为适当的格式](https://github.com/spc476/a09/blob/8682748ac0f98d7f83bbdb0b6e772270cf3fc89f/frsdos.c#L292)。我通过查找《Color BASIC Unravelled II》书中的一些浮点常数来进行了双重检查（[可在计算机档案库找到](https://colorcomputerarchive.com/repo/Documents/Books/Unravelled%20Series/)），例如这张表：

```
4634				* MODIFIED TAYLOR SERIES SIN COEFFICIENTS
4635	BFC7 05			LBFC7	FCB	6-1			SIX COEFFICIENTS
4636	BFC8 84 E6 1A 2D 1B	LBFC8	FCB	$84,$E6,$1A,$2D,$1B	* -((2*PI)**11)/11!
4637	BFCD 85 28 07 FB F8	LBFCD	FCB	$86,$28,$07,$FB,$F8	*  ((2*PI)**9)/9!
4638	BFD2 87 99 68 89 01	LBFD2	FCB	$87,$99,$68,$89,$01	* -((2*PI)**7)/7!
4639	BFD7 87 23 35 DF E1	LBFD7	FCB	$87,$23,$35,$DF,$E1	*  ((2*PI)**5)/5!
4640	BFDC 86 A5 5D E7 28	LBFDC	FCB	$86,$A5,$5D,$E7,$28	* -((2*PI)**3)/3!
4641	BFE1 83 49 0F DA A2	LBFE1	FCB	$83,$49,$0F,$DA,$A2	*    2*PI

```

然后使用字节值来填充一个变量并在 BASIC 中打印它（这是表达式 -2π³/3!）：

```
X=0         ' CREATE A VARIABLE
Y=VARPTR(X) ' GET ITS ADDRESS
POKE Y,&H86 ' AND SET ITS VALUE
POKE Y+1,&HA5 ' THE HARD WAY
POKE Y+2,&H5D
POKE Y+3,&HE7
POKE Y+4,&H28
PRINT X ' LET'S SEE WHAT IT IS
-41.3417023

```

然后使用它来创建一个浮点数值：

```
	org	$1000
	.float	-41.3417023
	end

```

检查生成的字节结果：

```
                         | FILE ff.a
                       1 |         org     $1000
1000: 86A55DE735       2 |         .float  -41.3417023
                       3 |         end

```

并调整浮点数常数直到得到匹配的字节：

```
                         | FILE ff.a
                       1 |         org     $1000
1000: 86A55DE728       2 |         .float  -41.341702110
                       3 |         end

```

我觉得“足够接近”。Color BASIC ROM 中的解析代码过时，并且早于 IEEE-754 浮点数标准，因此我认为末尾的一些不同数字是可以接受的。

作为最后的检查，我编写了以下代码片段来计算并显示 -2π³/3!，显示预计算结果，以及显示 2π 的预计算值：

```
		include	"Coco/basic.i"
		include	"Coco/dp.i"

CB.FSUBx	equ	$B9B9	; FP0 = X   - FP0	; addresses for
CB.FSUB		equ	$B9BC	; FP0 = FP1 - FP0	; these routines 
CB.FADDx	equ	$B9C2	; FP0 = X   + FP0	; from
CB.FADD		equ	$B9C5	; FP0 = FP1 + FP1	; Color BASIC Unravelled II
CB.FMULx	equ	$BACA	; FP0 = X   * FP0
CB.FMUL		equ	$BAD0	; FP0 = FP0 * FP1
CB.FDIVx	equ	$BB8F	; FP0 = X   / FP0
CB.FDIV		equ	$BB91	; FP0 = FP1 / FP0

CB.FP0fx	equ	$BC14	; FP0 = X
CB.xfFP0	equ	$BC35	; X   = FP0
CB.FP1f0	equ	$BC5F	; FP1 = FP0
CB.FP0txt	equ	$BDD9	; result in X, NUL terminated

		org	$4000
start		ldx	#tau		; point to 2*pi
		jsr	CB.FP0fx	; copy to FP0
		ldx	#tau		; 2PI * 2PI
		jsr	CB.FMULx
		ldx	#tau		; 2PI * 2PI * 2PI
		jsr	CB.FMULx
		jsr	CB.FP1f0	; copy fp acc to FP1
		ldx	#fact3		; point to 3!
		jsr	CB.FP0fx	; copy to FP0
		jsr	CB.FDIV		; FP0 = FP1 / FP0
		neg	CB.fp0sgn	; negate result by flippping FP0 sign
		jsr	CB.FP0txt	; generate string
		bsr	display		; display on screen

		ldx	#answer		; point to precalculated result
		jsr	CB.FP0fx	; copy to FP0
		jsr	CB.FP0txt	; generate string
		bsr	display		; display

		ldx	#tau		; now display 2*pi
		jsr	CB.FP0fx	; just to see how close
		jsr	CB.FP0txt	; it is.
		bsr	display
		rts

display.char	jsr	[CHROUT]	; display character
display		lda	,x+		; get character
		bne	.char		; if not NUL byte, display
		lda	#13		; go to next line
		jsr	[CHROUT]
		rts

tau		.float	6.283185307
fact3		.float	3!
answer		.float	-(6.283185307 ** 3 / 3!)

		end	start

```

结果如下：

```
-41.3417023
-41.3417023
 6.23418531

```

计算结果为 -41.3417023，而存储在 `answer` 中的直接结果也打印为 -41.3417023，因此匹配并强化了我对此方法的实现。

但我认为微软在生成一些较大项的浮点常数或将较大项的字节值转录方面存在问题。例如，考虑到-2π^(11)/11!。正确答案是-15.0946426，但ROM中的字节定义常数为-14.3813907，相差0.7。而不像Color BASIC无法正确计算—当我手动输入表达式时，它能够得出-15.0946426。

或者可能是《Color BASIC Unravelled II》的作者Walter K. Zydhek在生成值的表达式或他用于值的解释方面存在误解。我不确定谁在这里有错。

#### 2024年3月1日星期五更新

我错了关于《Color BASIC Unravelled II》的作者身份。它不是Walter K. Zydhek，而是一家已经不再运营的公司Spectral Associates的某位未知作者。Zydhek所做的只是将该书的实体副本转录成PDF，并提供下载。

* * *

#### 讨论关于这个条目

您可以自由链接到这里的任何条目。去吧，我不会咬你。我保证。

这些日期是当天条目的永久链接（或如果只有一个条目，则是条目）。标题是仅次于*该*条目的永久链接。链接的格式很简单：从此站点的基本链接开始：[https://boston.conman.org/](https://boston.conman.org/)，然后添加您感兴趣的日期，比如[2000/08/01](/2000/08/01)，这将使最终URL如下：

[https://boston.conman.org/2000/08/01](https://boston.conman.org/2000/08/01)

您还可以通过省略日期部分来指定整个月份。您甚至可以选择[任意时间段。](/about/technical.html)

您还可以注意到链接的微妙阴影，这是有意的：链接越“接近”（相对于页面），它显得“更亮”。这是使用颜色阴影来表示链接距离这里的一种实验。如果您没有注意到，不要担心；这并不是*那么*重要。

假定在这些页面中提到的每个品牌名称、口号、公司名称、符号、设计元素等均为受保护和/或注册商标的实体，其所有者独有，对此状态的承认是暗示的。
