<!--yml
category: 未分类
date: 2024-05-27 14:32:56
-->

# Converting IEEE-754 floating point to Color BASIC floating point - The Boston Diaries - Captain Napalm

> 来源：[https://boston.conman.org/2024/02/28.1](https://boston.conman.org/2024/02/28.1)

I'm still playing around with [floating point on the 6809](/2024/02/07.1)—specifically, support for floating point for the Color Computer. The format for floating point for Color BASIC (written by Microsoft) predates the [IEEE-754 Floating Point Standard](https://codedocs.org/what-is/ieee-754) by a few years and thus, isn't quite compatible. It's close, though. It's defined as an 8-bit exponent, biased by 129, a single sign bit (after the exponent) and 31 bits for the mantissa (the leading one assumed). It also does *not* support ±∞ nor NaN. This differs from the IEEE-754 single precision that uses a single sign bit, an 8-bit exponent biased by 127 and 23 bits for the mantissa (which also assumes a leafing one) and support for infinities and NaN. The IEEE-754 double precision uses a single sign bit, an 11-bit exponent biased by 1023 and 52 bit for the mantissa (leading one assumed) plus support for infinities and NaN.

So the Color BASIC is about halfway between single precision and double precision. This lead me to use IEEE-754 double precision for the Color Computer backend (generating an error for inifinities and NaN) [then massaging the resulting double into the proper format](https://github.com/spc476/a09/blob/8682748ac0f98d7f83bbdb0b6e772270cf3fc89f/frsdos.c#L292). I double checked this by finding some floating point constants in the Color BASIC ROM as shown in the book *Color BASIC Unravelled II*, ([available on the Computer Computer Archives](https://colorcomputerarchive.com/repo/Documents/Books/Unravelled%20Series/)), like this table:

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

Then using the byte values to populate a variable and printing it inside BASIC (this is the expression -2π³/3!):

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

Then using that to create a floating point value:

```
	org	$1000
	.float	-41.3417023
	end

```

Checking the resulting bytes that were generated:

```
                         | FILE ff.a
                       1 |         org     $1000
1000: 86A55DE735       2 |         .float  -41.3417023
                       3 |         end

```

And adjusting the floating point constant until I got bytes that matched:

```
                         | FILE ff.a
                       1 |         org     $1000
1000: 86A55DE728       2 |         .float  -41.341702110
                       3 |         end

```

I figure it's “close enough.” The parsing code in the Color BASIC ROM is old and predates the IEEE-754 floating point standard, so a few different digits at the end I think is okay.

As a final check, I wrote the following bit of code to calculate and display -2π³/3!, display the pre-calculated result, as well as display the pre-calculated value of 2π:

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

The results were:

```
-41.3417023
-41.3417023
 6.23418531

```

The calculation results in -41.3417023 and the direct result stored in `answer` also prints out -41.3417023, so that matches and it reinforces my approach to this nominally right.

But I think Microsoft had issues with either generating some of the floating point constants for the larger terms, or transcribing the byte values of the larger terms. Take for instance -2π^(11)/11!. The correct answer is -15.0946426, but the bytes in the ROM define the constant -14.3813907, a difference of .7. And it's not like Color BASIC can't calculate that correctly—when I typed in the expression by hand, it was able to come up with -15.0946426.

Or it could be that ~~Walter K. Zydhek,~~  author of *Color BASIC Unravelled II*, is wrong in his interpretation of the expressions used to generate the values, or his interpretation of what the values are used for. I'm not sure who is at fault here.

#### Update on Friday, March 1^(st), 2024

I was wrong about the authorship of *Color BASIC Unravelled II*. It was not Walter K. Zydhek, but some unknown author of Spectral Associates, a company that is no longer in business. All Zydhek did was to transcribe a physical copy of the book (which is no longer available for purchase anywhere) into a PDF and make it available.

* * *

#### Discussions about this entry

You have my permission to link freely to any entry here. Go ahead, I won't bite. I promise.

The dates are the permanent links to that day's entries (or entry, if there is only one entry). The titles are the permanent links to *that* entry only. The format for the links are simple: Start with the base link for this site: [https://boston.conman.org/](https://boston.conman.org/), then add the date you are interested in, say [2000/08/01](/2000/08/01), so that would make the final URL:

[https://boston.conman.org/2000/08/01](https://boston.conman.org/2000/08/01)

You can also specify the entire month by leaving off the day portion. You can even select [an arbitrary portion of time.](/about/technical.html)

You may also note subtle shading of the links and that's intentional: the “closer” the link is (relative to the page) the “brighter” it appears. It's an experiment in using color shading to denote the distance a link is from here. If you don't notice it, don't worry; it's not all *that* important.

It is assumed that every brand name, slogan, corporate name, symbol, design element, et cetera mentioned in these pages is a protected and/or trademarked entity, the sole property of its owner(s), and acknowledgement of this status is implied.