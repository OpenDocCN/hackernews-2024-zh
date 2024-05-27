<!--yml
category: 未分类
date: 2024-05-27 13:05:24
-->

# Flattening Bézier Curves and Arcs

> 来源：[https://minus-ze.ro/posts/flattening-bezier-curves-and-arcs/](https://minus-ze.ro/posts/flattening-bezier-curves-and-arcs/)

## Intro

When processing curves it’s much easier to convert them to a simpler form and work with that, instead of manipulating the curves directly. In this blog post I will present three easy ways to convert quadratics, cubics, and elliptical arcs to a sequence of line segments. These line segments make it almost trivial to solve problems like computing the [arc length](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/pathLength) or [dash offset](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-dashoffset). They can also simplify the process of rendering the curves if you have a renderer that can already handle paths enclosed by polylines, which is much easier to implement compared to rendering the curves directly. If you want an introduction to Bézier curves in general check out [A Primer on Bézier Curves](https://pomax.github.io/bezierinfo/) or [The Beauty of Bézier Curves](https://www.youtube.com/watch?v=aVwxzDHniEw).

While it’s a pretty niche area, I actually needed a memory-efficient algorithm for curve flattening when I was working with a vector renderer on a microcontroller. The default implementation was using way too much memory: it generated a ton of lines and stored them all in an array. For every single curve, there would be hundreds of lines stored, which is a lot more than what I could afford - customers would run out of memory even for very basic scenes. Not to mention the whole process was slow as hell too.

That was because the library was using the reference implementation of OpenVG! In there, a whopping 256 lines are generated for every quadratic/cubic Bézier curve, and the same applies to arcs too. OpenVG-RI simply samples the curves at uniformly-distributed values for \(t\) and generates lines from there. Proof: [here](https://github.com/eendeego/openvg-ri/blob/952aaf2699729203731e8a28eefacd2d47366d50/ri/src/riDefs.h#L118), [here](https://github.com/eendeego/openvg-ri/blob/952aaf2699729203731e8a28eefacd2d47366d50/ri/src/riPath.cpp#L2012), [here](https://github.com/eendeego/openvg-ri/blob/952aaf2699729203731e8a28eefacd2d47366d50/ri/src/riPath.cpp#L2076) and [here](https://github.com/eendeego/openvg-ri/blob/952aaf2699729203731e8a28eefacd2d47366d50/ri/src/riPath.cpp#L2244). Being a reference implementation, it’s of course expected not to be some kind of efficient method. It’s more focused on getting things rendered *correctly*, rather than efficiently.

Thankfully though, it turns out we can flatten curves with far fewer segments without even sacrificing quality that much. And all of that without needing recursive subdivisions!

## Flattening Quadratic Bézier Curves

The case of flattening quadratics has already been taken care of by Raph Levien. He [presents](https://raphlinus.github.io/graphics/curves/2019/12/23/flatten-quadbez.html) a method that:

*   Can tell you up-front how many segments it will generate beforehand.
*   Can generate each segment completely independently, without any state to keep track of.
*   Is adaptive: it doesn’t just split the curve in half repeatedly.
*   Lets you tune the quality and performance by decreasing the tolerance parameter for higher quality output, or increasing the tolerance parameter to generate fewer segments.
*   Is not recursive, unlike the [usual approach](https://hcklbrrfnn.files.wordpress.com/2012/08/bez.pdf) in which a De Casteljau subdivision is used. The consequence of the recursive nature of the De Casteljau subdivision is that you don’t know in advance how many segments it will generate, and computing those segments is not an independent process. You have to store the \(t\) values somehow, and only after that can you process them.

Play around with it by moving the control points and changing the tolerance value, which I have named \(\varepsilon\). Keep in mind that for demonstration purposes the curve is actually stroked using the generated lines, which makes it look worse than if you would use them to render filled paths. In reality the quality is great even when the tolerance is set to 0.25\. [Here’s](/glyph_36_smart_subdivision_test.png) one example of the `@` glyph rendered with the tolerance set to 0.25 (all curves of the glyph are quadratics). In my opinion it looks more than good enough.

I had to tweak the original algorithm a bit: if you draw the quadratic in a way that all its points are collinear and the end point is between the start point and control point, then a straight line between the start and end point will be drawn. That happens no matter what the tolerance is set to, and it’s not really correct. A quick workaround for this is to check when only one segment is generated, compute the \(t\) value where the derivative of the quadratic is 0, and use that to generate two or three segments. This workaround is still not entirely accurate but gives a good estimate in most cases. If you need more accuracy, you can use the traditional recursive method. It won’t be expensive as there will be at most 4 or 5 segments generated even with a very small tolerance.

This flattening method is not only memory-efficient but also has excellent performance. The code is simple too. If you want to see the code for all the solutions in this blog post, you can do that [here](/flattening-bezier-curves-and-arcs.js). Keep in mind that for each of the presented methods it’s entirely possible to create an iterator that will compute lines on demand, without any extra memory. I store them in arrays for simplicity’s sake. You could write it all as one big `fold` if you wanted to.

## Flattening Cubic Bézier Curves

When it comes to cubics, there is [this old Caffeine Owl post](https://web.archive.org/web/20150403003715/http://www.caffeineowl.com/graphics/2d/vectorial/cubic2quad01.html) that explains an algorithm which maintains pretty much all good properties of the one for quadratics. More precisely, we can convert a cubic to a sequence of quadratics, and then use the previous method to convert them to line segments. Consider the polynomial form of a cubic Bézier curve:

$$B(t) = a + b t + c t^2 + d t^3$$

With:

$$a = P_0$$

$$b = 3 (P_1 - P_0)$$

$$c = 3 (P_2 - 2 P_1 + P_0)$$

$$d = P_3 - 3 P_2 + 3 P_1 - P_0$$

Where \(0.0 \le t \le 1.0\) is the parameter along the curve, \(P_0\) is the start point of the cubic, \(P_1\) and \(P_2\) are the control points, and \(P_3\) is the end point. If the third degree term is close enough to zero, then we can approximate our cubic with a quadratic pretty well. We can make use of this fact and derive an error metric based on the third degree term, as explained in the post linked above:

$$err = |P_3 - 3 P_2 + 3 P_1 - P_0|$$

$$|P(x, y)| = \sqrt{x^2 + y^2}$$

Using this metric can estimate how many cubics we’ll need. Given some tolerance parameter, we can obtain:

$$n = \left(\frac{x^2 + y^2}{\frac{36^2}{3} tolerance^2}\right)^\frac{1}{6}$$

As the number of quadratics we need. The math is a bit arcane, but thankfully smart people have already done it for us and we can use it for what we want to achieve.

Now we can use this number to split the cubic at every \(\frac{1}{n}\) step as the \(t\) value, obtaining smaller cubics. And then each of these smaller cubics can be approximated by a quadratic. To get the cubic Bézier curve between two parameters \([t_0, t_1]\) blossoming can be used, as explained [here](https://math.stackexchange.com/a/4173517). Blossoming is simply an expansion of the De Casteljau subdivision algorithm, in which we compute all relevant points for us, which are nothing more than a sequence of linear interpolations. We can display two runs of the subdivison simultaneously, for both \(t_0\) and \(t_1\), from which we can obtain the middle cubic intuitively:

The essence is computing the interpolation for each line on which the points on the curve evaluated at \(t_0\) and \(t_1\) reside, but for the other \(t\) value. This will give us the control points for the cubic in the middle.

Then converting a cubic to a quadratic can be done by getting the intersection point between the lines formed by \(P_0P_1\) and \(P_3P_2\), and using that as the control point for our new quadratic. Of course this approximation only really works if the cubic is sufficiently close to being a quadratic. Since we are doing this in the subdivision step only, that’s okay.

Flattening quadratics has already been covered. All that remains is putting all the pieces together:

Play around with the curve, its tolerance value (\(\varepsilon_c\)), and the tolerance value for the quadratics generated inside of it (\(\varepsilon_q\)). I remind you again that result will look a bit worse here compared to real use cases, because the generated segments are stroked.

## Flattening Arcs

Another graphics primitive that is frequently used is the elliptical arc. In the [SVG arc notation](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths#arcs) the arc is described as fitting an ellipse with certain parameters where two user coordinates lie on it. Given these two points, four possible arcs can be drawn. Here are the parameters that describe how the elliptical arc is constructed, which ultimately decide which of these four possible arcs is rendered:

*   \((x_1, y_1), (x_2, y_2)\) - the points which lie on the underlying ellipse.
*   \((r_x, r_y)\) - the horizontal and vertical radius for the ellipse. If \(r_x = r_y\) then the arc will be circular.
*   \(\theta\) - the rotation of the underlying ellipse, described as x-axis rotation.
*   `large-arc-flag` - a boolean that tells whether the larger or smaller arcs should be chosen. \(1\) is for large, \(0\) is for small. Large means more than \(180^\circ\).
*   `sweep-flag` - a boolean which tells whether the arc should move clockwise or counter-clockwise. \(1\) is for counter-clockwise, where the angle increases from \((x_1, y_1)\) until the arc reaches \((x_2, y_2)\). \(0\) is for clockwise, where the angle decreases from \((x_1, y_1)\) until the arc reaches \((x_2, y_2)\).

[The SVG spec](https://www.w3.org/TR/SVG11/paths.html#PathDataEllipticalArcCommands) contains an illustration that shows how `large-arc` and `sweep` flags decide which arc is chosen out of the four possible elliptical arcs. This notation is intuitive for the user: you know you have two points and you draw an arc between them. Unfortunately it’s not as helpful when rendering those arcs. In that case, converting this representation to a center parametrization is more useful. We compute the center of the ellipse, the start angle where we have the first point, and the angle offset until we reach the second point.

[The SVG spec](https://www.w3.org/TR/SVG/implnote.html#ArcConversionEndpointToCenter) gives us the way to convert to a center parametrization. We can then approximate arcs of the unit circle and scale the resulted points by the ellipse’s radiuses. Part of the math is explained [here](https://pomax.github.io/bezierinfo/#circles_cubic). While Bézier curves are not able to represent elliptical arcs exactly, we make use of the fact that a cubic Bézier can approximate an elliptical arc very well as long as that arc is not longer than \(\frac{\pi}{4}\). As flattening cubics has already been covered, what remains to be done is, again, putting the pieces together:

You can again play around with the arc points, the tolerance for the cubics (\(\varepsilon_c\)) and quadratics (\(\varepsilon_q\)) that are used to approximate the elliptical arc, the radiuses for the underlying ellipse (\(r_x\) and \(r_y\)), the rotation (\(\theta\)), and the `large-arc` and `sweep` flags respectively.

## Conclusion

This post illustrates three simple, efficient ways to flatten Bézier curves and elliptical arcs into line segments. While the [code](/flattening-bezier-curves-and-arcs.js) is made to be simple and easy to read, in a real scenario you can implement flattening for each of these graphics primitives as one big `fold`, without storing results in temporary lists. You could even generate the lines in parallel if you wanted to, thus making these algorithms suited for GPU implementations, possibly in a geometry shader or compute shader.

It’s possible to work with curves analytically too, at least quadratics, like it’s done in [Loop-Blinn](https://www.microsoft.com/en-us/research/wp-content/uploads/2005/01/p1000-loop.pdf), [RAVG](https://hhoppe.com/proj/ravg/), [MPVG](https://w3.impa.br/~diego/projects/GanEtAl14/), [`NV_path_rendering`](https://developer.download.nvidia.com/devzone/devcenter/gamegraphics/files/opengl/gpupathrender.pdf), or [Slug](https://jcgt.org/published/0006/02/02/). Flattening still remains a relevant option though: it’s easy to implement, it gives you the path length almost for free, and it can be made to be efficient, as we’ve seen here.