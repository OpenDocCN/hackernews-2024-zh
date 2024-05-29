<!--yml
category: 未分类
date: 2024-05-27 15:23:18
-->

# Constructing a four-point egg – Tony Finch

> 来源：[https://dotat.at/@/2024-01-29-four-point-egg.html](https://dotat.at/@/2024-01-29-four-point-egg.html)

For reasons beyond the scope of this entry, I have been investigating elliptical and ovoid shapes. The [Wikipedia article for Moss’s egg](https://en.wikipedia.org/wiki/Moss%27s_egg) has a link to [a tutorial on Euclidean Eggs by Freyja Hreinsdóttir](https://web.archive.org/web/20200618202007/https://www.dynamat.oriw.eu/upload_pdf/20121022_154322__0.pdf) which (amongst other things) describes how to construct the “four point egg”. I think it is a nicer shape than Moss’s egg.

Freyja’s construction uses a straight edge and compasses in classical style, so it lacks dimensions. Below is my version, with the numbers needed to draw it on a computer.

At first this construction seems fairly rigid, but some of its choices are more arbitrary than they might appear to be. I have made [an interactive four-point egg](https://dotat.at/@/2024-01-eggsperiment.html) so you can drag the points around and observe how its shape changes.

In the following, I will measure angles in fractions of a turn `𝜏`, clockwise from noon because the egg’s pointy end is upwards.

1.  Start with axes and a unit circle.

2.  Imagine two diagonal scaffolding lines: one through the unit circle’s south and east points; and another through its south and west points.

3.  Draw an arc centred on the unit circle’s south point and equidistant from its north point, from one scaffolding line to the other. This forms the big end of the egg.

```
x: 0 y: -1 radius: 2 from: 𝜏*3/8 to: 𝜏*5/8

```

4.  Draw the egg’s lower right arc, centred on the unit circle’s west point, from the right end of the bottom arc up to the positive X axis.

```
x: -1 y: 0 radius: 2+√2 from: +𝜏*2/8 to: +𝜏*3/8

```

5.  Draw the egg’s lower left arc mirroring the lower right arc.

```
x: +1 y: 0 radius: 2+√2 from: -𝜏*3/8 to: -𝜏*2/8

```

6.  The lower right and lower left arcs meet the X axis at the egg’s east and west points. Imagine a scaffolding circle centred at the origin joining these points.

7.  The egg’s north centre is where this scaffolding circle meets the Y axis at `+1+√2`.

    Imagine two new diagonal scaffolding lines: one through the egg’s west point and north centre; and another through its east point and north centre.

8.  Draw the egg’s upper right arc, centred on the egg’s west point, from the egg’s east point up to a scaffolding line.

```
x: -1-√2 y: 0 radius: 2+2√2 from: +𝜏*1/8 to: +𝜏*2/8

```

9.  Draw the egg’s upper left arc mirroring the upper right arc.

```
x: +1+√2 y: 0 radius: 2+2√2 from: -𝜏*2/8 to: -𝜏*1/8

```

10.  Draw an arc on the egg’s north centre joining the ends of the upper right and upper left arcs. This arc forms the little end of the egg.

```
x: 0 y: 1+√2 radius: √2 from: -𝜏*1/8 to: +𝜏*1/8

```

The six coloured points marked in the diagram above are the centres of the six arcs that make the egg. These comprise the four points the egg is named after, plus their mirror images.

Moss’s egg is a three-point egg. It has the same top half as this four-point egg, but its bottom half is a simple semi-circle.