<!--yml
category: 未分类
date: 2024-05-27 13:35:26
-->

# Planes in 3D space

> 来源：[https://alexharri.com/blog/planes](https://alexharri.com/blog/planes)

<main class="css-fkkl8v">

# Planes in 3D space

April 27, 2024

A plane in 3D space can be thought of as a flat surface that stretches infinitely far, splitting space into two halves.

Planes have loads of uses in applications that deal with 3D geometry. I've mostly been working with them in the context of an [architectural modeler](https://www.arkio.is/), where geometry is defined in terms of planes and their intersections.

Learning about planes felt abstract and non-intuitive to me. *“Sure, that's a plane equation, but what do I do with it? What does a plane look like?”* It took some time for me to build an intuition for how to reason about and work with them.

In writing this, I want to provide you with an introduction that focuses on building a practical, intuitive understanding of planes. I hope to achieve this through the use of visual (and interactive!) explanations which will accompany us as we work through progressively more complex problems.

With that out of the way, let's get to it!

## Describing planes

There are many ways to describe planes, such as through

1.  a point in 3D space and a normal,
2.  three points in 3D space, forming a triangle, or
3.  a normal and a distance from an origin.

Throughout this post, the term *normal* will refer to a *normalized direction vector* (unit vector) whose magnitude (length) is equal to 1, typically denoted by <mjx-container classname="MathJax" jax="SVG"></mjx-container>where <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

Starting with the point-and-normal case, here's an example of a plane described by a point in 3D space <mjx-container classname="MathJax" jax="SVG"></mjx-container>and a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

The normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes the plane's orientation, where the surface of the plane is perpendicular to <mjx-container classname="MathJax" jax="SVG"></mjx-container>, while the point <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes *a* point on the plane.

We described this plane in terms of a single point <mjx-container classname="MathJax" jax="SVG"></mjx-container>, but keep in mind that this plane—let's call it <mjx-container classname="MathJax" jax="SVG"></mjx-container>—contains infinitely many points.

If <mjx-container classname="MathJax" jax="SVG"></mjx-container>were described by one of those other points contained by <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we would be describing the exact same plane. This is a result of the infinite nature of planes.

This way of describing planes—in terms of a point and a normal—is the [point-normal form](https://en.wikipedia.org/wiki/Euclidean_planes_in_three-dimensional_space#Point%E2%80%93normal_form_and_general_form_of_the_equation_of_a_plane) of planes.

We can also describe a plane using three points in 3D space <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>forming a triangle:

The triangle forms an implicit plane, but for us to be able to do anything useful with the plane we'll need to calculate its normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Once we've calculated the plane's normal, we can use that normal along with one of the triangle's three points to describe the plane in point-normal form.

As mentioned earlier, the normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>describing a plane is a unit vector (<mjx-container classname="MathJax" jax="SVG"></mjx-container>) perpendicular to the plane.

We can use <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>as two edge vectors that are parallel to the plane's surface.

By virtue of being parallel to the plane's surface, the vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are perpendicular to the plane's normal. This is where the cross product becomes useful to us.

The [cross product](https://en.wikipedia.org/wiki/Cross_product) takes in two vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>and returns a vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>that is perpendicular to both of them.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

For example, given the vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>, their cross product is the vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>, which we'll label <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

This explanation is simple on purpose. We'll get into more detail about the cross product later on.

Because the edge vectors of the triangle, <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>, are both parallel to the triangle's surface, their cross product will be perpendicular to the triangle's surface. Let's name the cross product of our two edge vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

<mjx-container classname="MathJax" jax="SVG"></mjx-container>has been scaled down for illustrative purposes

<mjx-container classname="MathJax" jax="SVG"></mjx-container>points in the right direction, but it's not a normal. For <mjx-container classname="MathJax" jax="SVG"></mjx-container>to be a normal, its magnitude needs to equal 1\. We can normalize <mjx-container classname="MathJax" jax="SVG"></mjx-container>by dividing it by its magnitude, the result of which we'll assign to <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

This gives us a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>where <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

Having found the triangle's normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>we can use it and any of the points <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>to describe the plane containing the three points in point-normal form.

It doesn't matter which of <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>we use as the point in the point-normal form; we always get the same plane.

### Constant-normal form

There's one more way to describe a plane that we'll look at, which is through a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>and a distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

This is the *constant-normal form* of planes. It makes lots of calculations using planes much simpler.

In the constant-normal form, the distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>denotes how close the plane gets to the origin. Thought of another way: multiplying the normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>by <mjx-container classname="MathJax" jax="SVG"></mjx-container>yields the point on the plane that's closest to the origin.

This is a simplification. More formally, given a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>on a plane whose normal is <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can describe all points <mjx-container classname="MathJax" jax="SVG"></mjx-container>on the plane in two forms: the point-normal form <mjx-container classname="MathJax" jax="SVG"></mjx-container>, and the constant-normal form <mjx-container classname="MathJax" jax="SVG"></mjx-container>where <mjx-container classname="MathJax" jax="SVG"></mjx-container>. See [further reading](/blog/planes#further-reading).

In getting a feel for the difference between the point-normal and constant-normal forms, take this example which describes the same plane in both forms:

The green arrow represents <mjx-container classname="MathJax" jax="SVG"></mjx-container>from the constant-normal form, while the blue point and arrow represent the point <mjx-container classname="MathJax" jax="SVG"></mjx-container>and normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>from the point-normal form.

Translating from the point-normal to the constant-normal form is very easy: the distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the [dot product](https://en.wikipedia.org/wiki/Dot_product) of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

If you're not familiar with the dot product, don't worry. We'll cover it later on.

The notation for <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>might seem to indicate that they're of different types, but they're both vectors. I'm differentiating between points in space (e.g. <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>) and direction vectors (e.g. <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>) by using the arrow notation only for direction vectors.

The normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>stays the same across both forms.

## Distance from plane

Given an arbitrary point <mjx-container classname="MathJax" jax="SVG"></mjx-container>and a plane <mjx-container classname="MathJax" jax="SVG"></mjx-container>in constant-normal form, we may want to ask how far away the point is from the plane. In other words, what is the minimum distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>needs to travel to lie on the plane?

We can frame this differently if we construct a plane <mjx-container classname="MathJax" jax="SVG"></mjx-container>containing <mjx-container classname="MathJax" jax="SVG"></mjx-container>that is parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>, which we can do in point-normal form using <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the point and <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the normal:

With two parallel planes, we can frame the problem as finding the distance between the two planes. This becomes trivial using their constant-normal form since it allows us to take the difference between their distance components <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

So let's find <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s distance using the <mjx-container classname="MathJax" jax="SVG"></mjx-container>equation we learned about:

With two distances <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>from the planes <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>the solution simply becomes:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

So, to simplify, given a plane <mjx-container classname="MathJax" jax="SVG"></mjx-container>having a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>and distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can calculate a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s distance from <mjx-container classname="MathJax" jax="SVG"></mjx-container>like so:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The distance may be positive or negative depending on which side of the plane the point is on.

### Projecting a point onto a plane

A case where calculating a point's distance from a plane becomes useful is, for example, if you want to project a point onto a plane.

Given a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>which we want to project onto plane <mjx-container classname="MathJax" jax="SVG"></mjx-container>whose normal is <mjx-container classname="MathJax" jax="SVG"></mjx-container>and distance is <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can do that fairly easily. First, let's define <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the point's distance from the plane:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Multiplying the plane's normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>by <mjx-container classname="MathJax" jax="SVG"></mjx-container>gives us a vector which when added to <mjx-container classname="MathJax" jax="SVG"></mjx-container>projects it onto the plane. Let's call the projected point <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The projection occurs along the plane's normal, which is sometimes useful. However, it is much more useful to be able to project a point onto a plane along an *arbitrary* direction instead. Doing that boils down finding the point of intersection of a line and a plane.

## Line-plane intersection

We can describe lines in 3D space using a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>and normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>. The normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes the line's orientation, while the point <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes a point which the line passes through.

In this chapter, the line will be composed of the point <mjx-container classname="MathJax" jax="SVG"></mjx-container>and normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>, while the plane—given in constant-normal form—has a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>and a distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

Our goal will be to find a distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>that <mjx-container classname="MathJax" jax="SVG"></mjx-container>needs to travel along <mjx-container classname="MathJax" jax="SVG"></mjx-container>such that it lies on the plane.

We can figure out the distance <mjx-container classname="MathJax" jax="SVG"></mjx-container>that we'd need to travel if <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>were parallel, which is what we did when projecting along the plane's normal.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Let's try projecting <mjx-container classname="MathJax" jax="SVG"></mjx-container>along <mjx-container classname="MathJax" jax="SVG"></mjx-container>using <mjx-container classname="MathJax" jax="SVG"></mjx-container>as a scalar like so:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

We'll visualize <mjx-container classname="MathJax" jax="SVG"></mjx-container>as a red point:

As <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>become parallel, <mjx-container classname="MathJax" jax="SVG"></mjx-container>gets us closer and closer to the correct solution. However, as the angle between <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>increases, <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes increasingly too small.

Here, the dot product comes in handy. For two vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>, the dot product is defined as

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

where <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the angle between <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

Consider the dot product of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Since both normals are unit vectors whose magnitudes are 1

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

we can remove their magnitudes from the equation,

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

making the dot product of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>the cosine of the angle between them.

For two vectors, the cosine of their angles approaches 1 as the vectors become increasingly parallel, and approaches 0 as they become perpendicular.

Since <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes increasingly too small as <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>become more perpendicular, we can use <mjx-container classname="MathJax" jax="SVG"></mjx-container>as a denominator for <mjx-container classname="MathJax" jax="SVG"></mjx-container>. We'll assign this scaled-up version of <mjx-container classname="MathJax" jax="SVG"></mjx-container>to <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

With <mjx-container classname="MathJax" jax="SVG"></mjx-container>as our scaled-up distance, we find the point of intersection <mjx-container classname="MathJax" jax="SVG"></mjx-container>via:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

We can now get rid of <mjx-container classname="MathJax" jax="SVG"></mjx-container>, which was defined as <mjx-container classname="MathJax" jax="SVG"></mjx-container>, giving us the full equation for <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Putting this into code, we get:

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  float dist = Vector3.Dot(plane.normal, line.point);  float D =  (plane.distance - dist)  / denom;  return line.point + line.normal * D;}
```

However, our code is not completely yet. In the case where the line is parallel to the plane's surface, the line and plane do not intersect.

That happens when <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are perpendicular, in which case their dot product is zero. So if <mjx-container classname="MathJax" jax="SVG"></mjx-container>, the line and plane do not intersect. This gives us an easy test we can add to our code to yield a result of "no intersection".

However, for many applications we'll want to treat being *almost* parallel as actually being parallel. To do that, we can check whether the dot product is smaller than some very small number—customarily called epsilon

```
float denom = Vector3.Dot(line.normal, plane.normal);if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;  }
```

See if you can figure out why Mathf.Abs is used here. We'll cover it later, so you'll see if you're right.

We'll take a look at how to select the value of epsilon in a later chapter on two plane intersections.

With this, our line-plane intersection implementation becomes:

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }  float dist = Vector3.Dot(plane.normal, line.point);  float D =  (plane.distance - dist)  / denom;  return line.point + line.normal * D;}
```

### Rays and lines

We've been talking about line-plane intersections, but I've been lying a bit by visualizing ray-plane intersections instead for visual clarity.

A ray and a line are very similar; they're both represented through a normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>and a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

The difference is that a ray (colored red) extends in the direction of <mjx-container classname="MathJax" jax="SVG"></mjx-container>away from <mjx-container classname="MathJax" jax="SVG"></mjx-container>, while a line (colored green) extends in the other direction as well:

What this means for intersections is that a ray will not intersect planes when traveling backward along its normal:

Our implementation for ray-plane intersections will differ from our existing line-plane intersection implementation only in that it should yield a result of "no intersection" when the ray's normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>is pointing "away" from the plane's normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>at an obtuse angle.

Since <mjx-container classname="MathJax" jax="SVG"></mjx-container>represents how far to travel along the normal to reach the point of intersection, we could yield "no intersection" when <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes negative:

```
if  (D <  0)  {  return  null;}
```

But then we'd have to calculate <mjx-container classname="MathJax" jax="SVG"></mjx-container>first. That's not necessary since <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes negative as a consequence of the dot product <mjx-container classname="MathJax" jax="SVG"></mjx-container>being a negative number when <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are at an obtuse angle between 90° and 180°.

If this feels non-obvious, it helps to remember that the dot product encodes the cosine of the angle between its two component vectors, which is why the dot product becomes negative for obtuse angles.

Knowing that, we can change our initial "parallel normals" test from this:

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }}
```

To this:

```
Vector3  RayPlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (denom < EPSILON)  {  return  null;  }}
```

The <mjx-container classname="MathJax" jax="SVG"></mjx-container>check covers both the *"line parallel to plane"* case *and* the case where the two normal vectors are at an obtuse angle.

Note: <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the symbol for epsilon.

## Plane-plane intersection

The intersection of two planes forms an infinite line.

As a quick refresher: lines in 3D space are represented using a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>and normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>where normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes the line's orientation, while the point <mjx-container classname="MathJax" jax="SVG"></mjx-container>describes a point which the line passes through.

Let's take two planes <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>whose normals are <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

Finding the direction vector of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s intersection is deceptively simple. Since the line intersection of two planes lies on the surface of both planes, the line must be perpendicular to both plane normals, which means that the direction of the intersection is the cross product of the two plane normals. We'll assign it to <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The magnitude of the cross product is equal to the [area of the parallelogram](https://en.wikipedia.org/wiki/Cross_product#/media/File:Cross_product_parallelogram.svg) formed by the two component vectors. This means that we can't expect the cross product to be a unit vector, so we'll normalize <mjx-container classname="MathJax" jax="SVG"></mjx-container>and assign the normalized direction vector to <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

This gives us the intersection's normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Let's zoom in and see this close up.

But this is only half of the puzzle! We'll also need to find a point in space to represent the line of intersection (i.e. a point which the line passes through). We'll take a look at how to do just that, right after we discuss the no-intersection case.

### Handling parallel planes

Two planes whose normals are parallel will never intersect, which is a case that we'll have to handle.

The cross product of two parallel normals is <mjx-container classname="MathJax" jax="SVG"></mjx-container>. So if <mjx-container classname="MathJax" jax="SVG"></mjx-container>, the planes do not intersect.

As previously mentioned, for many applications we'll want to treat planes that are *almost* parallel as being parallel. This means that our plane-plane intersection procedure should yield a result of "no intersection" when the magnitude of <mjx-container classname="MathJax" jax="SVG"></mjx-container>is less than some very small number called epsilon.

```
Line  PlanePlaneIntersection(Plane P1,  Plane P2)  {  Vector3 direction = Vector3.cross(P1.normal, P2.normal);  if  (direction.magnitude < EPSILON)  {  return  null;    }}
```

But what should the value of epsilon be?

Given two normals <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>where the angle between <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>is <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can find a reasonable epsilon by charting <mjx-container classname="MathJax" jax="SVG"></mjx-container>for different values of <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

Both of the axes are [logarithmic](https://en.wikipedia.org/wiki/Logarithmic_scale).

The relationship is linear: as the angle between the planes halves, so does the magnitude of the cross product of their normals. <mjx-container classname="MathJax" jax="SVG"></mjx-container>yields a magnitude of <mjx-container classname="MathJax" jax="SVG"></mjx-container>, and <mjx-container classname="MathJax" jax="SVG"></mjx-container>yields half of that.

So to determine the epsilon, we can ask: how low does the angle in degrees need to become for us to consider two planes parallel? Given an angle <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can find the epsilon <mjx-container classname="MathJax" jax="SVG"></mjx-container>via:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

If that angle is 1/256°, then we get:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

With this you can determine the appropriate epsilon based on how small the angle between the planes needs to be for you to consider them parallel. That will depend on your use case.

### Finding a point of intersection

Having computed the normal and handled parallel planes, we can move on to finding a point <mjx-container classname="MathJax" jax="SVG"></mjx-container>along the line of intersection.

Since the line describing a plane-plane intersection is infinite, there are infinitely many points we could choose as <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

We can narrow the problem down by taking the plane parallel to the two plane normals <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>and observing that it intersects the line at a single point.

Since the point lies on the plane parallel to the two plane normals, we can find it by exclusively traveling along those normals.

The simplest case is the one where <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are perpendicular. In that case, the solution is just <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Here's what that looks like visually:

When dragging the slider, notice how the tip of the parallelogram gets further away from the point of intersection as the planes become more parallel.

We can also observe that as we get further away from the point of intersection, the longer of the two vectors (colored red) pushes us further away from the point of intersection than the shorter (blue) vector does. This is easier to observe if we draw a line from the origin to the point of intersection:

Let's define <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the scaling factors that we apply to <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>(the result of which are the red and blue vectors). Right now we're using the distance components <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>of the planes as the scaling factors:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

To solve this asymmetric pushing effect, we need to travel less in the direction of the longer vector as the planes become more parallel. We need some sort of "pulling factor" that adjusts the vectors such that their tip stays on the line as the planes become parallel.

Here our friend the dot product comes in handy yet again. When the planes are perpendicular the dot product of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>equals 0, but as the planes become increasingly parallel, it approaches 1\. We can use this to gradually increase our yet-to-be-defined pulling factor.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Let's give the dot product <mjx-container classname="MathJax" jax="SVG"></mjx-container>the name <mjx-container classname="MathJax" jax="SVG"></mjx-container>to make this a bit less noisy:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The perfect pulling factors happen to be the distance components <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>used as counterweights against each other!

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Consider why this might be. When <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are perpendicular, their dot product equals 0, which results in

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

which we know yields the correct solution.

In the case where <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are parallel, their dot product equals 1, which results in:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Because the absolute values of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are equal, it means that the magnitude of the two vectors—defined as <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>—is equal:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

This means that the magnitude of our vectors will become *more* equal as the planes become parallel, which is what we want!

Let's see this in action:

The vectors stay on the line, but they become increasingly too short as <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>become parallel.

Yet again, we can use the dot product. Since we want the length of the vectors to increase as the planes become parallel, we can divide our scalars <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>by <mjx-container classname="MathJax" jax="SVG"></mjx-container>where <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the dot product of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the absolute value of <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The result of this looks like so:

Using <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the denominator certainly increases the size of the parallelogram, but by too much.

However, notice what happens when we visualize the quadrants of the parallelogram:

As the planes become more parallel, the point of intersection approaches the center of the parallelogram.

In understanding why that is, consider the effect that our denominator <mjx-container classname="MathJax" jax="SVG"></mjx-container>has on the area of the parallelogram. When <mjx-container classname="MathJax" jax="SVG"></mjx-container>, both of the vectors forming the parallelogram double in length, which has the effect of quadrupling the area of the parallelogram.

This means that when we scale the component vectors of the parallelogram by

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

it has the effect of scaling the area of the parallelogram by:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

To instead scale the *area* of the parallelogram by <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we need to square <mjx-container classname="MathJax" jax="SVG"></mjx-container>in the denominator:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Squaring allows us to remove <mjx-container classname="MathJax" jax="SVG"></mjx-container>because the square of a negative number is positive.

With this, our scalars <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>become

<mjx-container classname="MathJax" jax="SVG"></mjx-container>
<mjx-container classname="MathJax" jax="SVG"></mjx-container>

which scales the parallelogram such that its tip lies at the point of intersection:

Putting all of this into code, we get:

```
float dot = Vector3.Dot(P1.normal, P2.normal);float denom =  1  - dot * dot;float k1 =  (P1.distance - P2.distance * dot)  / denom;float k2 =  (P2.distance - P1.distance * dot)  / denom;Vector3 point = P1.normal * k1 + P2.normal * k2;
```

Based on code from [Real-Time Collision Detection by Christer Ericson](/blog/planes#further-reading)

Which through some mathematical magic can be optimized down to:

```
Vector3 direction = Vector3.cross(P1.normal, P2.normal);float denom = Vector3.Dot(direction, direction);Vector3 a = P1.distance * P2.normal;Vector3 b = P2.distance * P1.normal;Vector3 point = Vector3.Cross(a - b, direction)  / denom;
```

How this optimization works can be found in chapter 5.4.4 of [Real-Time Collision Detection by Christer Ericson](/blog/planes#further-reading).

This completes our plane-plane intersection implementation:

```
Line  PlanePlaneIntersection(Plane P1,  Plane P2)  {  Vector3 direction = Vector3.cross(P1.normal, P2.normal);  if  (direction.magnitude < EPSILON)  {  return  null;    }  float denom = Vector3.Dot(direction, direction);  Vector3 a = P1.distance * P2.normal;  Vector3 b = P2.distance * P1.normal;  Vector3 point = Vector3.Cross(a - b, direction)  / denom;  Vector3 normal = direction.normalized;  return  new  Line(point, normal);}
```

By the way, an interesting property of only traveling along the plane normals is that it yields the point on the line of intersection that is closest to the origin. Cool stuff!

## Three plane intersection

Given three planes <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, there are five possible configurations in which they intersect or don't intersect:

1.  All three planes are parallel, with none of them intersecting each other.
2.  Two of the planes are parallel, and the third plane intersects the other two.
3.  All three planes intersect along a single line.
4.  The three planes intersect each other in pairs, forming three parallel lines of intersection.
5.  All three planes intersect each other at a single point.

When finding the point of intersection, we'll first need to determine whether all three planes intersect at a single point—which for configurations 1 through 4, they don't.

Given <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>as the plane normals for <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can determine whether the planes intersect at a single point with the formula:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

When I first saw this, I found it hard to believe this would work for all cases. Still, it does! Let's take a deep dive to better understand what's happening.

### Two or more planes are parallel

We'll start with the configurations where two or more planes are parallel:

If <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are parallel then <mjx-container classname="MathJax" jax="SVG"></mjx-container>is a vector whose magnitude is zero.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

And since the dot product is a multiple of the magnitudes of its component vectors:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

the final result is zero whenever <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are parallel.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

This takes care of the "all-planes-parallel" configuration, and the configuration where <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are parallel

With that, let's consider the case where <mjx-container classname="MathJax" jax="SVG"></mjx-container>is parallel to either <mjx-container classname="MathJax" jax="SVG"></mjx-container>or <mjx-container classname="MathJax" jax="SVG"></mjx-container>but <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are not parallel to each other.

Let's take the specific case where <mjx-container classname="MathJax" jax="SVG"></mjx-container>is parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>but <mjx-container classname="MathJax" jax="SVG"></mjx-container>is parallel to neither.

Here the cross product <mjx-container classname="MathJax" jax="SVG"></mjx-container>is a vector (colored red) that's perpendicular to both <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

Since <mjx-container classname="MathJax" jax="SVG"></mjx-container>is parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>, that means that <mjx-container classname="MathJax" jax="SVG"></mjx-container>is also perpendicular to <mjx-container classname="MathJax" jax="SVG"></mjx-container>. As we've learned, the dot product of two perpendicular vectors is zero, meaning that:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

This also holds in the case where <mjx-container classname="MathJax" jax="SVG"></mjx-container>is parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>instead of <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

### Parallel lines of intersection

We've demonstrated that two of the three normals being parallel results in <mjx-container classname="MathJax" jax="SVG"></mjx-container>. But what about the configurations where the three planes intersect along parallel lines? Those configurations have no parallel normals.

As we learned when looking at plane-plane intersections, the cross product of two plane normals gives us the direction vector of the planes' line of intersection.

When all of the lines of intersection are parallel, all of the plane normals defining those lines are perpendicular to them.

Yet again, because the dot product of perpendicular vectors is 0 we can conclude that <mjx-container classname="MathJax" jax="SVG"></mjx-container>for these configurations as well.

We can now begin our implementation. As usual, we'll use an epsilon to handle the *"roughly parallel"* case:

```
Vector3  ThreePlaneIntersection(Plane P1,  Plane P2,  Plane P3)  {  Vector3 cross = Vector3.Cross(P2.normal, P3.normal);  float dot = Vector3.Dot(P1.normal, cross);  if  (Mathf.Abs(dot)  < EPSILON)  {  return  null;    }}
```

## Computing the point intersection

We want to find the point at which our three planes <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>intersect:

Some of what we learned about two-plane intersections will come into play here. Let's start by taking the line of intersection for <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>and varying the position of <mjx-container classname="MathJax" jax="SVG"></mjx-container>. You'll notice that the point of intersection is the point at which <mjx-container classname="MathJax" jax="SVG"></mjx-container>intersects the line.

When <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s distance from the origin is 0, the vector pointing from the origin to the point of intersection is parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>(and perpendicular to <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal).

This vector—let's call it <mjx-container classname="MathJax" jax="SVG"></mjx-container>—will play a large role in computing the point of intersection.

We can find <mjx-container classname="MathJax" jax="SVG"></mjx-container>through the cross product of two other vectors <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>. The first of those, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, is just <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The latter vector can be found via the equation

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

where <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are the distances in the constant-normal form of planes <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

With <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>defined, we assign their cross product to <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Let's see what it looks like:

Hmm, not quite long enough. <mjx-container classname="MathJax" jax="SVG"></mjx-container>certainly points in the right direction, but to make <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s tip lie on the line of intersection, we need to compute some scaling factor for <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

As it turns out, we've already computed this scaling factor:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

The product of <mjx-container classname="MathJax" jax="SVG"></mjx-container>—let's call that <mjx-container classname="MathJax" jax="SVG"></mjx-container>—can be thought to represent how parallel <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal is to the line intersection of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>approaches <mjx-container classname="MathJax" jax="SVG"></mjx-container>as <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal becomes parallel to the line of intersection <mjx-container classname="MathJax" jax="SVG"></mjx-container>, and approaches 0 as they become perpendicular.

We want the <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s magnitude to increase as <mjx-container classname="MathJax" jax="SVG"></mjx-container>decreases, so we'll make <mjx-container classname="MathJax" jax="SVG"></mjx-container>the scaling factor for <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Fully expanded, the equation for <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Bam! The problem is now reduced to traveling along the direction of the line intersection until we intersect with <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

We could use our knowledge of line-plane intersections to solve this, but there is a more efficient approach I want to demonstrate.

It involves finding a scaling factor for the direction vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>that scales it such that it's tip ends at <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Let's call this direction vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

There's one observation we can make that simplifies that. Since <mjx-container classname="MathJax" jax="SVG"></mjx-container>is perpendicular to <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal, the distance from <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s tip to <mjx-container classname="MathJax" jax="SVG"></mjx-container>along the direction vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>is the same as the distance from the origin to <mjx-container classname="MathJax" jax="SVG"></mjx-container>along that same direction.

With that, consider the vector <mjx-container classname="MathJax" jax="SVG"></mjx-container>where <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are the normal and distance of <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

If <mjx-container classname="MathJax" jax="SVG"></mjx-container>were parallel to <mjx-container classname="MathJax" jax="SVG"></mjx-container>, then <mjx-container classname="MathJax" jax="SVG"></mjx-container>would be the scaling factor we need, but let's see what happens with <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

As <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>become less parallel, <mjx-container classname="MathJax" jax="SVG"></mjx-container>becomes increasingly too short.

One thing to note as well is that even when <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>are completely parallel, <mjx-container classname="MathJax" jax="SVG"></mjx-container>is still too short, which is due to <mjx-container classname="MathJax" jax="SVG"></mjx-container>not being a unit vector. If we normalize <mjx-container classname="MathJax" jax="SVG"></mjx-container>prior to multiplying with <mjx-container classname="MathJax" jax="SVG"></mjx-container>that problem goes away.

But we're getting ahead of ourselves—we won't need to normalize <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Let's take a fresh look at how <mjx-container classname="MathJax" jax="SVG"></mjx-container>is defined:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Having defined <mjx-container classname="MathJax" jax="SVG"></mjx-container>as <mjx-container classname="MathJax" jax="SVG"></mjx-container>, we can simplify this to

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Earlier I mentioned that we could think of <mjx-container classname="MathJax" jax="SVG"></mjx-container>as a measure of how parallel <mjx-container classname="MathJax" jax="SVG"></mjx-container>'s normal <mjx-container classname="MathJax" jax="SVG"></mjx-container>is to <mjx-container classname="MathJax" jax="SVG"></mjx-container>(the line intersection of <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>). That's correct, but it's not the whole truth!

Since the dot product is a multiple of the magnitudes of its component vectors, <mjx-container classname="MathJax" jax="SVG"></mjx-container>also encodes the magnitude of <mjx-container classname="MathJax" jax="SVG"></mjx-container>. Hence, scaling <mjx-container classname="MathJax" jax="SVG"></mjx-container>by <mjx-container classname="MathJax" jax="SVG"></mjx-container>does two things:

1.  it normalizes <mjx-container classname="MathJax" jax="SVG"></mjx-container>, and
2.  it increases the length of <mjx-container classname="MathJax" jax="SVG"></mjx-container>as it becomes less parallel with <mjx-container classname="MathJax" jax="SVG"></mjx-container>.

So <mjx-container classname="MathJax" jax="SVG"></mjx-container>is both the scaling factor we need for <mjx-container classname="MathJax" jax="SVG"></mjx-container>, as well as <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

We've got our solution! Let's do a quick overview.

We define <mjx-container classname="MathJax" jax="SVG"></mjx-container>as:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

We'll redefine <mjx-container classname="MathJax" jax="SVG"></mjx-container>to include <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Our denominator, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, remains defined as :

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

With this, we find our point of intersection <mjx-container classname="MathJax" jax="SVG"></mjx-container>by adding <mjx-container classname="MathJax" jax="SVG"></mjx-container>and <mjx-container classname="MathJax" jax="SVG"></mjx-container>together and scaling them by <mjx-container classname="MathJax" jax="SVG"></mjx-container>:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Which fully expanded becomes:

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

Putting this into code, we get:

```
Vector3  ThreePlaneIntersection(Plane P1,  Plane P2,  Plane P3)  {  Vector3 dir = Vector3.Cross(P2.normal, P3.normal);  float denom = Vector3.Dot(u);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }  Vector3 a = P2.normal * P3.distance;  Vector3 b = P3.normal * P2.distance;  Vector3 V = Vector3.Cross(P1.normal, a - b);  Vector3 U = dir * P1.distance;  return  (V + U)  / denom;}
```

## Parting words

Thanks for reading!

A whole lot of hours went into writing and building the visualizations for this post, so I hope it achieved its goal of helping you build an intuitive mental model of planes.

Massive thanks goes to [Gunnlaugur Þór Briem](https://www.linkedin.com/in/gunnlaugur-briem/) and [Eiríkur Fannar Torfason](https://eirikur.dev/) for providing invaluable feedback on this post. I worked with them at [GRID](https://grid.is/); they're fantastic people to work with and be around.

— Alex Harri

PS: If you're interested in taking a look at how the visualizations in this post were built, this website is [open source on GitHub](https://github.com/alexharri/website).

## Further reading

I highly recommend checking out [Real-Time Collision Detection by Christer Ericson](https://www.amazon.com/Real-Time-Collision-Detection-Interactive-Technology/dp/1558607323). If you're building applications using 3D geometry, it will prove to be an incredibly useful resource. This post would not exist were it not for this book—especially the two chapters on the intersections of planes.

I recently analyzed the edit performance in [Arkio](https://www.arkio.is/) and noticed that a method for solving three-plane intersections took around half of the total compute time when recalculating geometry. By implementing the more efficient method for three-plane intersections described in the book, we made the method **~500% faster**, increasing Arkio's edit performance by over **1.6x**. Crazy stuff!

I started writing this post to understand how the three-plane intersection method worked. However, I felt that readers would need a better foundation and understanding of planes for this post to be of any value. In building that foundation, this post ended up quite a bit longer than I intended.

Anyway, it's a great book. [Check it out](https://www.amazon.com/Real-Time-Collision-Detection-Interactive-Technology/dp/1558607323)!

</main>