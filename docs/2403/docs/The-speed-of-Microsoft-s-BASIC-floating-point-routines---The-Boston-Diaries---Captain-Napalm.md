<!--yml
category: 未分类
date: 2024-05-27 14:30:56
-->

# The speed of Microsoft's BASIC floating point routines - The Boston Diaries - Captain Napalm

> 来源：[https://boston.conman.org/2024/03/01.1](https://boston.conman.org/2024/03/01.1)

I was curious about how fast [Microsoft's BASIC floating point](https://en.wikipedia.org/wiki/Microsoft_Binary_Format) routines were. This is easy enough to test, now that I can [time assembly code inside the assembler](/2023/12/19.3). The code calculates -2π³/3! using Color BASIC routines, IEEE-754 single precision and double precision.

First, Color BASIC:

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

I can't use the `.FLOAT` directive here since that only supports either the Microsoft format or IEEE-754 but not both. So for this test, I have to define the individual bytes per float. The last line is what the result should be (by checking a memory dump of the VM after running). Also, `.tao` is [2π](https://tauday.com/tau-manifesto), just in case that wasn't clear. This ran in 8,742 cycles, taking 2,124 instructions and 4.12 cycles per instruction (I modified the assembler to record this additional information).

Next up, IEEE-754 single precision:

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

The floating point control block (`.fpcb`) configures the MC6839 to use single precision, normal rounding and projective closure (not sure what that is, but it's the default value). And it does calculate the correct result. It's amazing that code written 42 *years* ago for an 8-bit CPU works flawlessly. What it *isn't* is fast. This code took 14,204 cycles over 2,932 instructions (average 4.84 cycles per instruction).

The higher than average cycle type could be due to position independent addressing modes, but I'm not entirely sure what it's doing to take nearly twice the time. The ROM does use the IEEE-754 extended format (10 bytes) internally, with more bit shifts to extract the exponent and mantissa, but *twice* the time?

Perhaps it's code to deal with ±∞ and NaNs.

The IEEE-754 double precision is the same, except for the floating point control block configuring double precision and the use of `.FLOATD` instead of `.FLOAT`; otherwise the code is identical. The result, however, isn't. It took 31,613 cycles over 6,865 instructions (average 4.60 cycles per instruction). And being twice the size, it took nearly twice the time as single precision, which is expected.

The final bit of code just loads the ROMs into memory, and calls each function to get the timing:

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

Really, the only surprising thing here was just how fast Microsoft BASIC was at floating point.

You have my permission to link freely to any entry here. Go ahead, I won't bite. I promise.

The dates are the permanent links to that day's entries (or entry, if there is only one entry). The titles are the permanent links to *that* entry only. The format for the links are simple: Start with the base link for this site: [https://boston.conman.org/](https://boston.conman.org/), then add the date you are interested in, say [2000/08/01](/2000/08/01), so that would make the final URL:

[https://boston.conman.org/2000/08/01](https://boston.conman.org/2000/08/01)

You can also specify the entire month by leaving off the day portion. You can even select [an arbitrary portion of time.](/about/technical.html)

You may also note subtle shading of the links and that's intentional: the “closer” the link is (relative to the page) the “brighter” it appears. It's an experiment in using color shading to denote the distance a link is from here. If you don't notice it, don't worry; it's not all *that* important.

It is assumed that every brand name, slogan, corporate name, symbol, design element, et cetera mentioned in these pages is a protected and/or trademarked entity, the sole property of its owner(s), and acknowledgement of this status is implied.