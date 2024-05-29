<!--yml
category: 未分类
date: 2024-05-27 15:19:22
-->

# Higher Order Derivatives of Transforms - Nosferalatu

> 来源：[https://nosferalatu.com/HigherOrderDerivativesTransforms.html](https://nosferalatu.com/HigherOrderDerivativesTransforms.html)

In [a previous post](DerivativesLogarithmsTransforms.html) I discussed the relationship between derivatives, logarithms, and transforms. This blog post adds to that and explains how to find the second and higher order derivatives of a transform.

### The Concept of Derivatives of Transforms

Let’s quickly recap my earlier blog post. Suppose we have a transform \(T\) and a point at the initial position of \(x(0)\). The position of the point at time t and the first derivative of that point are

\(x(t) = T^t * x(0)\)

\(x'(t) = log(T) * x(t)\).

where \(log(T)\) is the logarithm of the transform. Both \(T\) and \(log(T)\) are constant with respect to t.

### What’s the second derivative?

We know the first derivative (\(x'(t) = log(T) * x(t)\)), so to find the second derivative, we start with

\(\dfrac{d}{dt}x'(t) = \dfrac{d}{dt}log(T) * x(t)\)

Since \(log(T)\) is constant with respect to time, we then have

\(x''(t) = log(T) * \dfrac{d}{dt}x(t)\)

and earlier we stated that \(x'(t) = log(T) * x(t)\), so we just plug that in, giving us our answer:

\(x''(t) = log(T) * log(T) * x(t)\).

Neat! In other words, if we have the first derivative, then the second derivative is just another multiplication by \(log(T)\).

In the earlier blog post, we found the tangent vector of a point by multiplying the point by \(log(T)\). Similarly, we can now find the acceleration vector of a point by multiplying it by \(log(T) * log(T)\).

### What’s the nth derivative?

The beauty of this approach is how simple it is. For the nth derivative, we can follow the same pattern. More generally, the nth derivative is

\(x^{n}(t) = log(T)^n * x(t)\)

### Interactive Example

Here’s an interactive example where you can see the tangent vector (green arrow) and acceleration vector (red arrow) of a point as it is interpolated from the origin to the gizmo’s transform. You can manipulate the gizmo to translate and rotate the transform. You can also enable a visualization of the velocity vector field or the acceleration vector field.

The source code for the applet can be found [here](../../js/HigherOrderDerivativesOfTransforms_example.js).

### Comments

Leave comments on this post with Github Issues [here](https://github.com/nosferalatu/nosferalatu.github.io/issues/2).