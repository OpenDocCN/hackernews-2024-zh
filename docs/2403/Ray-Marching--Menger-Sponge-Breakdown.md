<!--yml
category: 未分类
date: 2024-05-29 12:45:43
-->

# Ray Marching: Menger Sponge Breakdown

> 来源：[https://connorahaskins.substack.com/p/ray-marching-menger-sponge-breakdown](https://connorahaskins.substack.com/p/ray-marching-menger-sponge-breakdown)

There are two very popular methods to render 3D scenes. *Rasterization* has historically been the most popular method used for real-time graphics (ex: video games). In short, rasterization is a technique that projects 3D models, composed of 3D triangles, onto a 2D plane. Another popular method of rendering is *ray tracing*. This method has historically been used for rendering scenes that require a greater amount of realism and are not required to render in real-time (ex: cinematic visual effects).

Ray tracing involves shooting virtual rays out of each pixel into a virtual scene beyond the display. Calculating the intersections of the ray with the scene to determine the pixel color. Again, these are 3D models being rendered and they are usually composed of 3D triangles. When compared with rasterization, ray tracing works in a sort of reverse order. Rasterization starts with trying to draw a 3D model and determines which pixels that 3D model will influence. Ray tracing starts with a pixel and determines which 3D models influence it.

What we will discuss in this article is a third rendering method. A flavor of ray-casting known as *ray marching*. There are still rays being shot out into the virtual world from each pixel of your screen. But whereas *ray tracing* performs a one-off calculation in determining the intersection between an infinite ray and a 3D model, *ray marching* takes an, often cheaper, more iterative approach.

Each ray shares the same starting position (the virtual camera’s position) and a function is used to determine how far that ray’s current position is from *any* element in the scene. To calculate the distance from each object in the virtual scene, ray marching uses what are called SDFs (signed distance fields). These functions do *not* determine the distance a ray will travel before it hits an object (as a ray tracing algorithm would do). SDFs do not even take the direction of the ray into consideration. They simply determine the distance between the ray’s current position and the closest point of an object. So although it doesn’t give us the exact distance a ray would have to travel before it experiences an intersection, it does tell us a distance that we can safely travel without running into anything. And, as stated before, ray marching is an iterative process. We acquire a safe distance to travel, we travel that distance, and repeat. When the distance returned by an SDF (the distance between the ray’s current position and an object defined by the SDF) falls under some specified threshold (ex: 0.0002 units), we will consider that a *hit* and do the lighting calculations for the surface.
The ray marching algorithm must also handle two edge cases:

*   To avoid infinite looping, we should cap off a ray when it becomes greater than some maximum specified distance (ex: 100 units) away from all objects in the scene. This is helpful, as some rays may simply never hit an object in our scene as they travel straight into the “sky” or “void” of the scene.

*   For ray marching to perform within a reasonable amount of time, we should also consider the maximum amount of iterations we are willing to calculate (ex: 60 iterations).

    *   A great example of when iterations could get costly is an SDF that represents a long flat plane (wall) and a ray that is running parallel to the wall just outside of our specified hit distance. In each iteration, the ray inches forward in very small steps as the distance to the plane remains very small. Since this is a real-time program, we can’t simply wait for this ray to run forever.

The ray marching algorithm considers both of these termination scenarios (too far from all objects or too many iterations) to be a “miss” and no lighting calculations will be performed as no surface was hit. In both of these edge cases, we make a trade-off for performance. The ray may have hit if we let it travel 1 unit further and the ray may have hit if we had performed just one more iteration. Take note of this when specifying your maximum distances and iterations.

To summarize, the algorithm of ray marching looks something like this:

1.  Calculate the distance to object(s) using the ray’s current position and the corresponding SDF

2.  If that distance is smaller than the specified hit distance, we consider that a hit. We *exit the algorithm* and perform lighting calculations for the surface that was hit.

3.  Move the ray forward the distance calculated in step (1)

4.  If the distance to the scene exceeds the maximum specified distance, we consider that a miss and *exit the algorithm*. 

5.  If the next iteration would exceed the specified maximum number of iterations, we consider that a miss and *exit the algorithm*.

6.  Go back to step 1, using the new position of our ray

A [Menger Sponge](https://en.wikipedia.org/wiki/Menger_sponge) is a 3D fractal.

It is constructed by:

1.  Cutting a cube into 3x3x3 equal cubes

2.  Removing the center cube and the center cube of each face

3.  Repeat steps 1 through 3 for all remaining cubes

I do attempt to explain most things but I will not be explaining some concepts like vectors or coordinate systems. If you notice anything that went unexplained, please ask, use your favorite search engine, LLM, or the resources at the end of this article to find other wonderful educational media on the topics.

There is a demo of the many things that are discussed in this article. This demo uses ShaderToy. It can be viewed in any browser and can be found in the link below. Functions in the style of sdXxxx() are the “signed distance fields/functions” that you will learn about in this article. Please play around with the code! There a “?” button in the lower right corner of the text editor with lots of helpful information. And “alt + enter”/”cmd+enter” will trigger a compile so you can see the results of your changes!

[https://www.shadertoy.com/view/MXB3Wt](https://www.shadertoy.com/view/MXB3Wt)

The code in ShaderToy are fragment shaders written in GLSL ES, which stands for the OpenGL Shading Language Embedded Systems. OpenGL being a library used for running various shader programs on a GPU. A shader being a heavily parallel processed program that runs on a GPU. And an embedded system being really anything that isn’t a personal computer but, in our use case of GLSL ES, mostly just means it runs on mobile devices. The benefit of using GLSL ES is that it is a subset of GLSL. So the code will run on your desktop, laptop, or smartphone.

In this article, we strictly concern ourselves with fragment shaders that run once per-pixel for every pixel in a specified viewport (could be a window or could be the whole screen). However, do note that that is not the only (or even the primary) way that fragment shaders are used.

Below are two code snippets of the same OpenGL ES code implementing the ray marching algorithm. The first with explicit comments, and the second without (for improved code readability):

```
`#version 320 es
// ^^ specify OpenGL ES version

// requested precision of float values (ignore this)
precision highp float;

// name of the shader output value (RGBA pixel color)
out vec4 FragColor;

// max number of ray marching iterations
#define MAX_STEPS 100

// max distance from object before considered a miss
#define MISS_DIST 60.0

// max distance to object before considered as hit
#define HIT_DIST 0.01

// "uniform" means this value is set separately using 
// the OpenGL library. Resolution in pixels.
uniform vec2 screenResolution;

// the position at which rays are spawned
const vec3 cameraPos = vec3(6.0, 6.0, -6.0);

// SDF to our scene as a whole
float sdScene(vec3 rayPos);

// main entry point
void main() {
  // gl_FragCoord.xy is a unique, per pixel (or "fragment"),
  // whole number value. With origin in the lower left corner.
  vec2 pixelCoord = gl_FragCoord.xy;

  // Move origin from bottom left to center
  pixelCoord -= (0.5 * screenResolution.xy);

  // Scale y from -1.0 to 1.0, scale x by same factor
  pixelCoord = pixelCoord / viewPortResolution.y;

  // direction of ray through pixel
  vec3 rayDir = vec3(pixelCoord.x, pixelCoord.y, 1.0);

  // modify rayDir to be unit length (1 unit long)
  rayDir = normalize(rayDir);

  // distance the ray has travelled
  float dist = 0.0;

  // assume miss by default
  bool hit = false;

  // ray marching iteration count
  int iterations = 0;

  // break loop after max iterations
  while(iterations < MAX_STEPS) {

    // current position of the ray
    vec3 pos = cameraPos + (dist * rayDir);

    // current distance to scene
    float posToScene = sdScene(pos);

    // add to total ray distance traveled
    dist += posToScene;

    // register hit if close enough to scene
    if(posToScene < HIT_DIST) {
      hit = true;
      break;
    }

    // register miss if too far from scene
    if(posToScene > MISS_DIST) {
      break;
    }

    iterations += 1;
  }

  if(hit) {
    // simple shading based on ray marching iterations
    // fading from white (1.0) to black (0.0)
    vec3 color = vec3(1.0 - (iterations / float(MAX_STEPS));

    // set pixel output value (with 100% opacity)
    FragColor = vec4(color, 1.0);
  } else { // miss
    // dark grey used as miss color
    const vec3 missColor = vec3(0.2, 0.2, 0.2);

    // set pixel output value (with 100% opacity)
    FragColor = vec4(missColor, 1.0);
  }
}

float sdScene(vec3 rayPos) {
 // calculate SDF to the scene
}`
```

```
`#version 320 es

precision highp float;

out vec4 FragColor;

#define MAX_STEPS 100
#define MISS_DIST 60.0
#define HIT_DIST 0.01

uniform vec2 screenResolution;

const vec3 cameraPos = vec3(6.0, 6.0, -6.0);

float sdScene(vec3 rayPos);

void main() {
  vec2 pixelCoord = gl_FragCoord.xy;
  pixelCoord -= (0.5 * screenResolution.xy);
  pixelCoord = pixelCoord / viewPortResolution.y;

  vec3 rayDir = vec3(pixelCoord.x, pixelCoord.y, 1.0);
  rayDir = normalize(rayDir);

  float dist = 0.0;
  bool hit = false;
  int iterations = 0;

  while(iterations < MAX_STEPS) {
    vec3 pos = cameraPos + (dist * rayDir);
    float posToScene = sdScene(pos);
    dist += posToScene;
    if(posToScene < HIT_DIST) {
      hit = true;
      break;
    }
    if(posToScene > MISS_DIST) { break; }
    iterations += 1;
  }

  if(hit) {
    vec3 color = vec3(1.0 - (float(iterations) / float(MAX_STEPS)));
    FragColor = vec4(color, 1.0);
  } else { // miss
    const vec3 missColor = vec3(0.2, 0.2, 0.2);
    FragColor = vec4(missColor, 1.0);
  }
}

float sdScene(vec3 rayPos) {
 // calculate SDF to the scene
}`
```

The above code is pure GLSL ES shader code. The GLSL ES code that runs in ShaderToy will look ever so slightly different. Especially in regards to `#version`, `screenResolution`, `gl_FragCoord`, and `FragColor`. Further understanding of the differences is left as an exercise to the reader.

If it’s your first time using GLSL or fragment shaders or not and the above is just a little overwhelming, I promise it’s not as scary as it looks. If you don’t understand the above code immediately, that’s okay. It will not hinder your understanding of the rest of the article. There are also additional resources at the end of this article that may aid you in your journey. I’d recommend continuing the article and coming back to re-explore the algorithm. From here on out, we are done speaking on the algorithm. We will only concern ourselves with developing an SDF to a Menger Sponge`.`

*vec3*: A data type in GLSL that is simply an array of 3 floating point values. This data type is often used to represent things like points, vectors, or color values (RGB). 

“*halfwidth*”: A value equal to half of a width. Often we will find that it’s more convenient to deal with half the width of an object being rendered. This is analogous to equations that prefer using the radius (halfwidth) of a circle as opposed to the diameter (width).

component: A 2D vector has two components(*x* & *y*). A 3D vector has three components(*x*, *y* & *z*).

GLSL: OpenGL Shading Language. For this article, “OpenGL” is interchangeable with “OpenGL ES”, as we will not be using any GLSL features outside the subset of OpenGL ES.

Ray marching requires the use of signed distance fields (SDFs) to acquire distances between a point and some object. But before we determine the distance between a point and some complex 3D object, let’s first remind ourselves how to find the distance between two points on a 2D plane.

Let’s say you have two points, A and B. When you subtract A from B, you get a vector that can “move” point A to point B. It’s possible this sounds unintuitive to you, so here’s a simple way to solidify this concept. Let’s now say that you have two integers +3 and +7\. When you subtract +3 from +7, you get the integer +4\. This integer is capable of taking +3 to +7\. Following this same logic, when you subtract point (-4, -2) from point (-1, 2) you get the vector <3, 4>. Which is a vector that can “move” point (-4, -2) to point (-1, 2). It is simply addition and subtraction.

Now the distance between two points is fairly simple. If you have a vector <x,y> that takes point A to point B (<x,y> = B - A), then the distance between point A and point B is simply the length of that vector. If you were to draw the vector <x,y>, you’d see that the vector can be viewed as a hypotenuse of a right triangle in which one of our sides is of length x and the other is of length y.

So to get the length of a vector, we simply use the Pythagorean theorem, a² + b² = c².

*   The length a vector <x,y> is equal to sqrt(x² + y²)

*   The length of vector <x, y, z> is equal to sqrt(x² + y² + z²)

*   This pattern can also be extended for all dimensions

To put it all together, the distance between points A and B can be acquired by calculating the length of the resulting vector when you subtract A from B.

The SDF to a point looks something like this:

```
// signed distance to a point
float sdPoint(vec3 rayPosition, vec3 pointPosition) {   

  // vector from point to ray
  vec3 pointToRay = rayPosition - pointPosition;  

  // Pythagorean theorem to get length of vector
  float distanceToPoint = sqrt(pointToRay.x * pointToRay.x +
                               pointToRay.y * pointToRay.y +
                               pointToRay.z * pointToRay.z);

  return distanceToPoint;
}
```

To acquire the distance from a point (for example: the ray’s current position) to a sphere, we start by calculating the distance between the point and the center of the sphere (another point). When traveling to the center of a sphere, you must first travel to the surface of the sphere and then further travel the sphere's radius to finally reach the center point. So if we want a distance to a sphere, we first get the distance to the center point and then subtract the length of the radius, taking us back to the surface of the sphere.

The SDF to a sphere looks something like this:

```
float sdSphere(vec3 rayPosition, vec3 sphereCenterPosition, float radius) {
  vec3 centerToRay = rayPosition - sphereCenterPosition;
  float distToCenter = length(centerToRay);
  float distToSurface = distToCenter - radius;
  return distToSurface;
}
```

NOTE: Another name for ray marching is “sphere tracing”. You can think of an SDF as a function that gives the radius for a sphere that is centered at the ray’s position and just barely touches the object or scene represented by the SDF.

Let’s learn about some ways that you can manipulate objects through the use of our rays and SDFs.

One thing I left out while going over the SDF of a sphere is that SDFs will often define the distance to an object that is centered at the origin. And the way you actually position that object into your scene is to simply subtract the object’s desired position from your ray’s position before sending it to the SDF. By subtracting the object’s desired position, we are transforming the ray’s position to a local coordinate system of the object, in which the object is centered at the origin. 

On the left side of the diagram below, we show the actual location of the ray as well as the desired location of the circle. Near the middle, we show our ray’s position transformed to the local coordinate system of the circle where the circle’s center sits at the origin. That ray’s position was transformed by simply subtracting the desired location of our circle. Notice that in both scenarios of the diagram, the ray shares the same distance to the circle. This is critical as the SDFs only job is to get an accurate distance to an object.

This simplifies the logic within the SDF, as it no longer concerns itself with translations of objects.

Our new SDF for a sphere and translating that sphere now looks like this:

```
float sdSphere(vec3 rayPosition, float radius) {
  float distToCenter = length(rayPosition);
  float distToSurface = distToCenter - radius;
  return distToSurface;
}

// signed distance to a scene
void sdScene() {
  // ...
  vec3 spherePosition = vec3(-5.0, 3.0, 0.0);
  vec3 distToSphere = sdSphere(rayPosition - spherePosition, 1.0);
  // ...
}
```

Similar to translation, the SDF actually doesn’t have to handle the exact dimensions of the object (ex: the radius of a sphere) as we can scale the object to our desired dimensions outside of the SDF. Scaling is a little less obvious than translation but the technique is very easy to remember. Just like translation, we are going to manipulate our ray’s position before sending it to the SDF in order to manipulate our objects.

Let’s see what it is like to scale something up. And really quick, let’s think about what won’t work. If a sphere of radius 1.0 is 2.0 units away, then scaling that sphere up to a radius of 2.0 would mean it is now 1.0 unit away. The sphere is twice as big and our distance is now halved. With this example, it would seem we could simply divide the distance to an object by the value we wish to scale it. But take just one more example and we can see this does not hold water. If a sphere of radius 1.0 is *3.0 units* away, scaling that sphere up to a radius of 2.0 would mean it is now *2.0 units* away. The sphere is twice as big, but now our distance is 2/3rds its original length. Hmmmm...

The approach that we are going to take is to pretend the coordinate system that our ray lives in has been shrunken down, while the object represented by the SDF remains unchanged. From the perspective of the ray’s coordinate system, the object has grown. The only problem is that the distance we calculated is the distance in our shrunken coordinate system. To correct this, we must map the calculated distance back to our original coordinate system (un-shrink it). So if we shrunk our coordinate system down by 7.0 (to scale the object up to 7.0 times its original size), the distances we calculated will then need to be multiplied by 7.0 to map it back to the original coordinate system. 

In the figure below: In black we have the original ray, a unit circle, and the distance between them. In red, we have the desired scaled circle and the desired distance between the scaled circle and the ray. 

In the figure below: In black we have the shrunken down ray, the original circle (“scaled up” from the perspective of the ray), and the distance between them. In red, we have the same distance mapped back to the original coordinate system (un-shrunken).

Scaling an object down is the exact same approach and is left as an exercise for the reader.

Our SDF is even simpler than before, as it no longer concerns itself with the scale nor position of the sphere! Taking in only the ray’s position as a parameter.

```
float sdSphere(vec3 rayPosition) {
  const float radius = 1.0;
  return length(rayPosition) - radius; 
}

void sdScene() {
  // …
  // sphere's position
  vec3 p = vec3(-5.0, 3.0, 0.0);
  // sphere's radius
  float r = 3.0;
  vec3 distToSphere = sdSphere((rayPos - p) / r) * r;
  // …
}
```

If the SDFs don’t need to worry about the scale of an object, why is the size of the `radius` defined in `sdSphere`?

In order to scale the object defined by the SDF, it has to have *some* initial size (scaling zero up/down is still zero). In this case, the sphere has a radius of 1, or a diameter of 2\. That is the number that will be scaled up or down.

When trying to calculate the distance to an object, we can often simplify the problem by taking advantage of an object’s symmetry. Symmetry is an extremely common property of many primitive shapes. For example: cubes, spheres, cylinders, and toruses can be seen as symmetrical across all axes when centered at the origin and thoughtfully oriented. The figure below shows an object that is symmetrical along both the x and y axes.

Notice that each point of the same color shares the same distance to the object. This is where folding becomes incredibly useful. Taking the absolute value of the x & y components of any point will map that point to one with the same color in the positive (upper right) quadrant. By taking advantage of the symmetry of our object and the absolute value of our ray’s position, we can allow our SDF to focus solely on finding the distance to our object for points in the positive quadrant.

To demonstrate how this can simplify things, let’s look at an SDF for a cube.

Let us start with finding the distance to a square. In this example, we are going to use the symmetry of a square to our advantage. We will work exclusively with the absolute values of points. In the figure below, we partition the positive quadrant into four sections to understand the different ways we might calculate the distance to the square.

Let us first take a look at the upper right (yellow) partition. If both our X and Y components are greater than half the width of our square, we simply need to calculate the distance to the corner of our square. With only taking that into consideration, our SDF would look something like this:

```
float sdSquareCornerOnly(vec2 rayPos) {
  // half initial scale of square
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  // fold ray into positive quadrant
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;
  return length(cornerToRay);
}
```

Now, let’s incorporate the upper left (blue) and lower right (purple) sections. Think of the vector pointing from the corner to the upper left point and from the corner to the lower right point (all still within the upper right quadrant). This vector does not represent the shortest path to the square. If any component in our vector from the corner to our ray has a negative value (the x component will be negative when pointing to the upper left point, and the y component will be negative when pointing to the lower right point), we can simply ignore that component in our distance calculation by setting it to zero. The upper point left point simply needs to go down to hit the closest point on the square and the lower right point simply needs to go left.

An updated SDF looks like this:

```
float sdSquareOutsideOnly(vec2 rayPos) {
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;

  // ignore negative components
  vec2 closestToRay = max(cornerToRay, 0.0); 

  return length(closestToRay);
}
```

The `max()` function is a component-wise function, available in GLSL, in which each component of the vector will be the max between its current value and the second parameter passed in (`0.0` in this case). This effectively clamps all negative values of our vector to `0.0`. So what does our SDF currently return if we are inside the square? In that case, all components of our vector would be negative and then clamped to `(0.0, 0.0)`. The length of that vector is zero. The best number to build on.

We are only inside the square if all components of the vector pointing from the corner to our ray’s position are negative. Look back at the diagram to verify that that is true. When inside the square, our closest point is the distance to either the left or right walls. These currently exist as the component values in our `cornerToRay` vector.

This is a good time to explain the “Signed” in a “Signed Distance Field”. An SDF should return a positive, non-zero, value when a point is outside its defining shape. It should return a negative, non-zero, value when a point is inside its defining shape. And it should return zero when a point lies exactly on the surface of its defining shape.

In everyday math, the minimum of -5 & -3 is -5\. That is how the `min` function works in any code you will ever see and it works in GLSL. But, since we are calculating the shortest distance to some object defined by the SDF, -3 units is actually a *smaller signed distance* than -5 units.

Back to our inner, lower left point… Since the vector from the corner to the point is negative for both the x & y components, we take the *maximum* of these values to find the minimum *signed* distance.

With this additional information, let’s finish our SDF.

```
float sdSquare(vec2 rayPos) {
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;
  vec2 closestToOutsideRay = max(cornerToRay, 0.0);

  // Acquire max component
  float cornerToRayMaxComponent = max(cornerToRay.x, cornerToRay.y);

  // If the max component is a positive value, clamp it to zero
  float distToInsideRay = min(cornerToRayMaxComponent, 0.0);

  // return either the distance to outside OR distance to inside
  return length(closestToOutsideRay) + distToInsideRay;
} 
```

The addition at the end is made possible as `distToInsideRay` will be equal to `0.0` if the ray exists outside of the square and, as discussed before, the `closestToOutsideRay` will be equal to `0.0` if the ray exists inside of the square. In other words, at least one operand of the final addition is guaranteed to be `0.0`. This means the returning value will be *either* the length of `closestToOutsideRay`  *or*  `distToInsideRay`.

The SDF to a cube uses all the same logic as just described but adds one more dimension. The SDF can be represented as the following:

```
float sdCube(vec3 rayPos) {
  // half initial scale of square (arbitrary)
  const float halfWidth = 1.0;
  const vec3 corner = vec3(halfWidth, halfWidth, halfWidth);

  // fold ray into positive octant
  vec3 foldedPos = abs(rayPos);
  // corner to ray
  vec3 ctr = foldedPos - corner;

  // ignore negative components for outside points
  vec3 closestToOutsideRay = max(ctr, 0.0);

  // Acquire max component
  float cornerToRayMaxComponent = max(max(ctr.x, ctr.y), ctr.z);

  // If the max component is a positive value, clamp to zero
  float distToInsideRay = min(cornerToRayMaxComponent, 0.0);

  // return either the distance to outside OR distance to inside
  return length(closestToOutsideRay) + distToInsideRay;
} 
```

We now have all the tools needed to create an infinite cross. You could think of creating an infinite cross by starting with our `sdCube` and simply extruding each face out infinitely. The diagram below shows a 2D representation of our infinite cross. As you can see, our distance formula for each of the points has changed quite a bit. Three of our four original points are now within the object with only one outside.

Although the distance functions to each point have changed, the equations don’t look too foreign. Let’s start by covering the distance to the lower left (red) point. Again, as a reminder, we are folding space through the absolute value function and are only concerned with the upper right quadrant. The closest distance to the lower left (red) point is clearly the corner and we’ve already seen this distance calculation before.

```
float sdCrossInnerCornerOnly(vec2 rayPos) {
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;
  // when inside, distances are negative
  return -length(cornerToRay);
}
```

A signed distance field returns positive values when a point is outside of the object and negative values when it is inside of the object. SDFs return a *signed* value but the length of a vector is always a positive value. In this case, our point is inside the cross, so a simple negation solves our problem.

Things continue to look similar approach as we did with the `sdSquare`. With `sdSquare`, `the` upper left (blue) and lower right (purple) points were defined by their distances to the horizontal and vertical edges, respectively. Interestingly, when finding the distance to this infinite cross, their concerns have swapped. The upper left point’s smallest distance lies on the vertical edge, and the lower right point’s smallest distance lies on the horizontal edge. In the `sdSquare`, negative values in the `cornerToRay` vector are clamped to zero. In our `sdCross`, the positive values are clamped to zero. The same `cornerToRay` vector is used for both SDFs, but with components that are clamped differently.

```
float sdCrossInnerOnly(vec2 rayPos) {
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;

  // ignore positive components for inside points
  vec2 closestInsidePoint = min(cornerToRay, 0.0);

  return -length(closestInsidePoint);
}
```

So far our SDF covers all of the possible points that reside within the cross itself, for which the SDF would return a negative value. All that remains is the positive distances. As we saw in our last diagram, the distance to a point outside of the cross is simply the difference between the smallest component of our point and half the width of the cross. Which is just the smallest component of the `cornerToRay` vector.

```
float sdCross(vec2 rayPos) {
  const float halfWidth = 1.0;
  const vec2 corner = vec2(halfWidth, halfWidth);
  vec2 foldedPos = abs(rayPos);
  vec2 cornerToRay = foldedPos - corner;
  vec2 closestInsidePoint = min(cornerToRay, 0.0);

  // acquire min component
  float minComponent = min(cornerToRay.x, cornerToRay.y);

  // If min component is negative, clamp to zero
  float distToOutsidePoint = max(minComponent, 0.0);

  // return either the distance to inside OR outside
  return -length(closestInsidePoint) + distToOutsidePoint;
}
```

For a point to exist outside of the cross, both components must have a value greater than the `halfWidth` of our cross. Subtracting the corner from our point subtracts the halfwidth from both components. If the minimum component is greater than `0.0`, they both are and the distance is the minimum component. Otherwise, we are inside the cross and our previous calculated distance is desired. As with sdSquare, the final addition serves more as an either/or, as one of the operands is guaranteed to be `0.0`.

Adding one more dimension to the cross isn’t incredibly obvious, so let’s talk about it a bit. For the 2D “infinite cross”, we determined that the ray was outside of the cross if both components were greater than half the width of the cross and the ray was inside of the cross otherwise. What do the regions look like for a 3D infinite cross? And what do I mean by a 3D “infinite cross”?

Our definition for a 2D infinite cross is a square in which each face has been extruded out infinitely. Our definition for a 3D infinite cross is a *cube* in which each face has been extruded out infinitely.

For our 2D infinite cross, we had four unique locations that a ray could be: 

1.  Inside the original square. This occurs when both components are smaller than the halfwidth. Closest distance is to the inner corner (negated).

2.  Inside the extruded face that extends in the x direction. This occurs when only the y-component is smaller than the halfwidth. Closest distance is the y component minus the halfwidth.

3.  Inside the extruded face that extends in the y direction. This occurs when only the x-component is smaller than the halfwidth. Closest distance is the x component minus the halfwidth.

4.  Outside of the cross. This occurs when both components are larger than the halfwidth. Closest distance is the smallest component.

NOTE: For the 3D infinite cross, we are folding space into the positive octant and we can think about this problem only in terms of the positive octant. If you find yourself thinking about what happens with negative coordinates...don’t! The symmetry of the object across all axes means we can fold everything into the positive octant using absolute values, create an algorithm for all points within the single octant, and ignore all other cases!

For our 3D infinite cross, we have *five* unique locations:

1.  Inside the original cube. This occurs when all components are less than the `halfwidth`. Closest distance is to the nearest inner edge (negated). Take note, the center is *not* a cube constructed of six faces. It is a cube constructed of 12 edges. It is only a skeleton of a cube. The distance we are interested in is the distance between a point and one of those 12 edges. If you imagine each edge as corners on a 2-dimensional plane, you may notice that the nearest edge is determined by the largest two components of our ray’s 3-dimensional position. 

2.  Inside the extruded face that extends in the x direction. This occurs when only the x component is larger than the `halfwidth`. The closest distance can be framed as the inside of a square on the yz-plane. Closest distance is `max(y,z)` minus the `halfwidth`.

3.  Inside the extruded face that extends in the y direction. This occurs when only the y component is larger than the `halfwidth`. The closest distance can be framed as the inside of a square on the xz-plane. Closest distance is `max(x,z)` minus the `halfwidth`.

4.  Inside the extruded face that extends in the z-direction.  This occurs when only the z component is larger than the `halfwidth`. The closest distance can be framed as the inside of a square on the xy-plane. Closest distance is `max(x,y)` minus the `halfwidth`.

5.  Outside of the cross. This occurs when two or more components are larger than the `halfwidth`. Closest distance is to the closest extruded face. The closest extruding face is the face extruding in the direction of the largest component and the distance to that extruded face is a factor of the two smallest components.

Let’s think about #5, outside of the cross, a bit more. Moving in the x direction can be seen as moving parallel to the extruded face in the x direction and perpendicular to both the extruded faces in the y & z directions. This means that moving along in the x-direction has no effect on the distance of the extruded face in the x-direction, while increasing the distance from the extruded faces in the y & z directions. This is sort of like a game, where the largest component pushes the point further from the other two extruded faces, and its own extruded face “wins” as holding the closest distance to the point. The fact that moving along the x-axis is moving parallel to the extruded face in the x-direction means it cannot also be a factor in determining the distance to that same extruded face.

```
float sdCross(vec3 rayPos) {
  const float halfWidth = 1.0;
  const vec3 corner = vec3(halfWidth, halfWidth, halfWidth);
  vec3 foldedPos = abs(rayPos);

  // corner to ray
  vec3 ctr = foldedPos - corner;

  // isolate minimum/maximum components
  float minComp = min(min(ctr.x, ctr.y), ctr.z);
  float maxComp = max(max(ctr.x, ctr.y), ctr.z);

  // fancy way to acquire middle component
  float midComp = ctr.x + ctr.y + ctr.z - minComp - maxComp;

  // Handles: #1, #2, #3, #4
  vec2 closestOutsidePoint = max(vec2(smallestComp, middleComp), 0.0);
  // Handles: #5
  vec2 closestInsidePoint = min(vec2(middleComp, largestComp), 0.0);

  // return either the distance to inside OR outside
  return length(closestOutsidePoint) + -length(closestInsidePoint);
}
```

Another very powerful tool in ray marching is the modulo operator. If unfamiliar, the modulo operator (commonly denoted as ‘mod’ or ‘%’) simply returns the remainder of some division. For example, if you divide 11 by 4, you get 2 with a remainder of 3\. So 11 mod 4 is 3.

The figure below shows what happens if you use mod 3 to transform the x & y components of various points. It divides our original single unbounded coordinate system into infinite *bounded* coordinate systems that range from [0, 3.0) for each axis. The diagram shows how our original coordinate system is cut into slices. Each slice will be mapped to the highlighted region. To further demonstrate, the modulo operator will map each point to the point with the same color in the highlighted region.

NOTE: The modulo operator is actually not defined for negative numbers. It is very possible that the mod operator on your platform does not produce the same results as seen in the above diagram.

*   In GLSL, `mod(x, y)` is defined as `x - (y * floor(x/y))`. If curious, you can play around with this to see that it produces the results we desire (infinitely repeating identical bounded coordinate spaces). If you’re playing around with the modulo operator outside of GLSL and things don’t look right, this could be your problem. To verify it is your problem, simply replace the modulo operator with the GLSL definition as stated above. 

One minor annoyance you might’ve noticed with our boxed coordinate systems is that the origins are now in the lower left corner. To get an origin at the center, we simply subtract half of the length of our boxed coordinate system.

In the figure above, we are performing mod 3 on the x & y components of our ray’s position

In the figure below, we are doing the same and then subtracting 1.5 (half of 3).

Two things to note. There is no longer a “true” slice that remains unchanged. Our coordinate system now goes from [-1.5, 1.5), where the lower boundaries are included and the upper is excluded. This may have felt more intuitive when the included lower boundaries were zero, but it’s good to remember it’s still true here as well.

There is nothing special about using the modulo operator in two dimensions. Two-dimensional diagrams are simply easier to draw and get a feeling for. In three-dimensional space, modulo creates infinite bounded coordinate systems that resemble the shape of a box/cube, as opposed to a rectangle/square.

Our last step before diving straight into creating our Menger Sponge is understanding how to combine primitive shapes to create more complex objects. The operations we are going to look at will be subtracting one shape from another, the union of two shapes, and the intersection of two shapes.

Let’s first make an effort to analyze the picture above. We have two shapes that intersect each other, a circle and a square. We also have 5 points representing 5 regions with unique signed distances to each of the two shapes.

*   P1: Positive distance to both, where `sdCircle()` < `sdSquare()`.

*   P2: Negative distance to circle. Positive to square. `sdCircle()` < `sdSquare()`.

*   P3: Negative distance to both.

*   P4: Positive distance to circle. Negative to square. `sdSquare()` < `sdCircle()`.

*   P5: Positive distance to both, where `sdSquare()` < `sdCircle()`.

The operations we will use to combine objects are imperfect and we will encounter a limitation that we have not seen before. The signed distance resulting from a point being in the interior of a combined object will not be correct. For our purposes, this will be perfectly fine. When rendering via ray marching, it is already in our best interest to not cast rays inside of objects. The ray should either hit the surface or miss. It should not pass through an object into its interior.

To learn more about the inferiority of these SDF Boolean operations, [Inigo Quilez (creator of ShaderToy and an immense contributor to ray marching & SDFs) wrote an article on this exact topic which I’d consider bookmarking for later reading.](https://iquilezles.org/articles/interiordistance/)

```
float union = min(sdShape1(rayPos), sdShape2(rayPos))
```

*   P1: `sdCircle()` is positive and a smaller distance than `sdSquare()`, so the distance to the circle will be used.

*   P2: Inside union, not well defined.

*   P3: Inside union, not well defined.

*   P4: Inside union, not well defined.

*   P5: `sdSquare()` is a smaller distance than `sdCircle()`, so the distance to the distance to the square will be used.

```
float subtraction = max(-sdShape1(ray), sdShape2(ray))
```

*   P1: `sdSquare()` is a positive distance and `-sdCircle()` is a negative distance, so the distance to the square will be used.

*   P2: `-sdCircle()` and `sdSquare()` are both positive distances, so the largest distance will be used.

*   P3: `-sdCircle()` is a positive distance and `sdSquare()` is a negative distance, so the `-sdCircle()` will be used.

*   P4: Inside subtraction, not well defined.

*   P5: `sdSquare()` is a larger distance than `-sdCircle`, so the distance to the square will be used.

```
float intersection max(sdShape1(ray), sdShape2(ray))
```

*   P1: `sdSquare()` is a larger distance than `sdCircle()`, so the distance to the square will be used.

*   P2: `sdSquare()` is a positive distance and `sdCircle()` is a negative distance, so the distance to the square will be used.

*   P3: Inside subtraction, not well defined.

*   P4: `sdCircle()` is a positive distance and `sdSquare()` is negative, so the distance to the circle is used.

*   P5: `sdCircle()` is a larger distance than `sdSquare()`, so the distance to the circle is used.

We are going to move forward with creating the sponge in very explicit steps. For starters, let’s create an infinite cross that is as wide as our soon-to-be Menger Sponge. We will bound it within a box that is twice the width of the cross. This bounding box is used simply to get a better idea of what we are working with. The code and resulting image look something like this.

```
float sdBoundedCross(vec3 rayPos) {
  float boundingBoxDist = sdCube(rayPos / 2.0) * 2.0;
  float crossDist = sdCross(rayPos);
  float sdIntersection = max(boundingBoxDist, crossDist);
  return sdIntersection;
}
```

For the first iteration of the Menger Sponge, we need to subtract a cross that is a third of the width of the Sponge itself and is properly aligned with the center of each face. With that in mind, let’s skinny up our cross to one-third of the Sponge’s dimensions.

```
float sdBoundedCrossSlimmed(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  float boundingBoxDist = sdCube(rayPos / 2.0) * 2.0;
  float crossDist = sdCross(rayPos / oneThird ) * oneThird;
  float sdIntersection = max(boundingBoxDist, crossDist);
  return sdIntersection;
}
```

Now to make things a little clearer, let’s draw the outermost cube of the Sponge and take a union with what we have.

```
float sdBoundedCrossWithBox(vec3 rayPos) {
  float boundingBoxDist = sdCube(rayPos / 2.0) * 2.0;
  float crossDist = sdCross(rayPos / 3.0) * 3.0;
  float intersection = max(boundingBoxDist, crossDist);
  float spongeBox = sdCube(rayPos);
  float sdBoxCrossUnion = min(intersection, spongeBox);
  return sdBoxCrossUnion;
}
```

If you are familiar with your basic boolean operations, it’s already obvious that we simply should have performed a subtraction to get the first iteration of our Menger Sponge. While we’re at it, we’ll remove the bounding box, as it no longer serves as a visual helper.

```
float sdMengerSpongeIteration1(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  float crossDist = sdCross(rayPos / oneThird) * oneThird;
  float spongeBox = sdCube(rayPos);
  float sdSubtraction = max(spongeBox, -crossDist);
  return sdSubtraction;
}
```

And a little peek inside...

So that’s how you do it once. But how do we do it iteratively? Well, let us start by thinking exactly what we want. The first iteration is taking the full box, cutting each dimension into thirds resulting in 27 equal-sized boxes, removing the 6 boxes in the center of each face, as well as the one box in the very center. In the second iteration, we take the remaining 20 boxes and perform the same operations (cutting into 27 equal boxes & removing). We then perform those operations on the remaining 400 boxes and so on.

As a step in the right direction, let’s see what it looks like to simply cut a box into those 27 equal-sized boxes. As you might imagine, the answer is not manually finding the distance to 27 hard-coded boxes. For this we are going to take advantage of the modulo operator to convert our single infinite space into infinite, repeating, bounded spaces. Let’s start by just creating a bunch of boxes with modulo.

```
float sdBoundedBoxFieldBroken(vec3 rayPos) {
  float boundingBox = sdCube(rayPos / 3.9) * 3.9;
  // transform space from single unbounded space
  // to infinite bounded spaces in range of [0.0, 2.0)
  vec3 repeatingPos = mod(rayPos, 2.0);
  float cubeDist = sdCube(repeatingPos / 1.5) * 1.5;
  float sdIntersection = max(cubeDist, boundingBox);
  return sdIntersection;
}
```

The modulo simply cuts the world into infinite boxes that are all of width 2.0 units. Within each of these repeating “boxed worlds”, we draw a slightly smaller box. There’s a decent problem though. This can be demonstrated by choosing a not-so-nice bounding box scale, and taking a look from another angle.

```
float sdBoundedBoxFieldBroken(vec3 rayPos) {
  // slight change in bounding box scale
  float boundingBox = sdCube(rayPos / 4.0) * 4.0;
  vec3 repeatingPos = mod(rayPos, 2.0);
  float dist = sdCube(repeatingPos / 1.5) * 1.5;
  float sdIntersection = max(dist, boundingBox);
  return sdIntersection;
}
```

More like `sdJankyBoundedBoxField()`, amirite?? It’s not what we want but it is still pretty cool, huh?!

So our problem is that when performing our modulo operator, we are correctly cutting the space into boxes with a width equal to 2 units. The problem is our coordinate system has all axes going from [0, 2) but our SDFs return the distance to a box at the origin. Currently, the boxes we are rendering are “bleeding” out of their spaces past the origin of the boxed worlds and causing all sorts of strange artifacts.

What we need to do to correct the situation is modify things slightly so the origins of each of our boxed worlds reside in the center. We just subtract half of their width and instead of coordinate systems where each axis goes from [0, 2), we have coordinate systems that go from [-1, 1).

```
float sdBoundedBoxField(vec3 rayPos) {
  float boundingBox = sdCube(rayPos / 4.0) * 4.0;
  vec3 repeatingPos = mod(rayPos, 2.0);
  // transform repeating space range from [0,2) to [-1, 1)
  repeatingPos -= 1.0;
  float dist = sdCube(repeatingPos / 0.8) * 0.8;
  float sdIntersection = max(dist, boundingBox);
  return sdIntersection;
}
```

Bada bing, bada boom. We got ourselves a proper box field.

Now that we have a bunch of cubes, let’s try to narrow them down to just the 27 that we need.

```
float sdTwentySevenBoxesKinda(vec3 rayPos) {
  // bounding box spans all axes between [-1.0, 1.0]
  float boundingBox = sdCube(rayPos);

  // sdCube() defines a cube with a halfwidth of 1, or a
  // width of 2 units.
  const float cubeWidth = 2.0;

  // Transform space into infinite cube spaces with
  // dimensions in range of [0, 2/3).
  float boxedWorldDimen = cubeWidth / 3.0;
  vec3 repeatingPos = mod(rayPos, boxedWorldDimen);

  // move origin in cube space from corner to center
  // now with a range between [-1/3, 1/3)
  repeatingPos -= (boxedWorldDimen / 2.0);

  // Stretch repeated spaces from [-1/3, 1/3) to [-1.0, 1.0)
  repeatingPos *= 3.0;

  // Shrink the cubes slightly to reveal gaps between them
  float dist = sdCube(repeatingPos / 0.9) * 0.9;

  // Acquire actual distance with correction to prev stretch
  dist /= 3.0;

  float sdIntersection = max(dist, boundingBox);
  return sdIntersection;
}
```

Pretty close, but it looks like we have another problem. We have the right-sized boxes. They are all stacked properly. But some of them are cut strange and how can we create a Menger Sponge if there are no center boxes to cut out? The problem is that they are not aligned how we would like them. The boxes are currently aligned to run alongside each axis. I’m going to perform an intersection with a cross to give us a nice visual of what is going wrong here. 

```
float sdTwentySevenBoxesCross(vec3 rayPos) {
  float boundingBox = sdCube(rayPos);
  const float cubeWidth = 2.0;
  float boxedWorldDimen = cubeWidth / 3.0;
  vec3 repeatedPos = mod(rayPos, boxedWorldDimen);
  repeatedPos -= (boxedWorldDimen) / 2.0;
  repeatedPos *= 3.0;
  float boxesDist = sdCube(ray / 0.9) * 0.9;
  boxesDist /= 3.0;
  float boxesIntersection = max(boxesDist, boundingBox);

  // Cross that runs along the axes
  float crossDist = sdCross(rayPos / 0.25) * 0.25;
  float sdIntersection = max(boxesDist, crossDist);
  return sdIntersection;
}
```

As you can see here, the modulo operator has our “box worlds” aligned in such a way that eight of them have corners touching the origin and anywhere else along the axes will be surrounded by four box worlds. To align the boxes with one box centered at the origin, we will have to translate the boxes accordingly.

Hopefully, at this point, you understand that there aren’t actually a finite amount of boxes. We created a never-ending world of boxes and are only showing the boxes that intersect with our bounding box, through the use of an intersection with a cross. Since we have infinite boxes, there are many different ways we could translate these cubes to get what we want. We just need *a* center cube and any would work. For our case though, we are going to go ahead and translate the first cube that lies in the positive octant of our world (between <0, 0, 0> & <2/3, 2/3, 2/3>) to the center position.

```
float sdTwentySevenBoxes(vec3 rayPos) {
  float boundingBox = sdCube(rayPos);
  const float cubeWidth = 2.0;
  float boxedWorldDimen = cubeWidth / 3.0;

  // We want to translate the first box in the
  // positive octant. Changing its world coordinates
  // from [0, boxedWorldDimen) to 
  // [-boxedWorldDimen / 2.0, boxedWorldDimen / 2.0)
  float translation = -boxedWorldDimen / 2.0;
  vec3 ray = rayPos - translation;

  vec3 repeatedPos = mod(ray, boxedWorldDimen);
  repeatedPos += translation;
  repeatedPos *= 3.0;
  float dist = sdCube(repeatedPos / 0.9) * 0.9;
  dist /= 3.0;
  float sdIntersection = max(dist, boundingBox);
  return sdIntersection;
}
```

And there you have it! Exactly what we want. The most boring Rubik’s Cube you’ve ever seen. And a perfect start to our Sponge!

You might already have guessed it, but with what we have now, we’re very close to our second iteration of the Menger Sponge! Let us use our stacked boxed worlds to create crosses instead of cubes! Each one-third the width of our boxed worlds (and one-ninth the width of our entire sponge).

```
float sdTwentySevenCrossesBound(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  const float cubeWidth = 2.0;
  float boundingBox = sdCube(rayPos);
  float boxedWorldDimen = cubeWidth * oneThird;
  float translation = -boxedWorldDimen / 2.0;
  vec3 ray = rayPos - translation;
  vec3 repeatedPos = mod(ray, boxedWorldDimen);
  repeatedPos += translation;
  repeatedPos *= 3.0;
  float dist = sdCross(repeatedPos / oneThird) * oneThird;
  dist /= 3.0;
  float sdIntersection = max(dist, boundingBox);
  return sdIntersection;
}
```

Can you see where this is going?

All we need to do is unbound the crosses from our bounding box, as the visual help is no longer needed, and then subtract what we have from our first iteration of the Menger Sponge.

```
float sdMengerSpongeIteration2(vec3 rayPos) {
  const float cubeWidth = 2.0;
  float menger1 = sdMengerSpongeIteration1(rayPos);
  float boxedWorldDimen = cubeWidth / 3.0;
  float translation = -boxedWorldDimen / 2.0;
  vec3 ray = rayPos - translation;
  vec3 repeatedPos = mod(ray, boxedWorldDimen);
  repeatedPos += translation;
  repeatedPos *= 3.0;
  float crossesDist = sdCross(repeatedPos * 3.0) / 3.0;
  crossesDist /= 3.0;
  float sdSubtraction = max(menger1, -crossesDist);
  return sdSubtraction;
}
```

Now we’re really getting somewhere!!

Now that we have two iterations of the Menger Sponge, let us see if we can find a single algorithm that will create both iterations. The way I am going to approach this problem will start with a little intuition. I will attempt to describe the best I can where these feelings come from.

There seems to be a lot of necessary computation involved in creating those 27 crosses for Iteration #2\. For instance, the mod operator seems like it would be incredibly difficult (impossible?) to remove. Whereas creating just the one cross for Iteration #1 felt very easy in comparison. I see a potential easier path forward in creating an “over-complicated” Iteration #1 using logic very close to Iteration #2\. With the hope that this may lead to a more generic algorithm that will help us produce an Nth iteration of the Menger Sponge.

```
float sdOneCrossOvercomplicatedBound(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  const float cubeWidth = 2.0;
  float boundingBox = sdCube(rayPos);

  // boxed world as big as the box
  // division by 1 for illustrative purposes
  float boxedWorldDimen = cubeWidth / 1.0;

  // Translate origin of bounded space to align with origin of space
  float translation = -boxedWorldDimen / 2.0;
  vec3 ray = rayPos - translation;

  // Create infinite boxes the same width of our sponge
  vec3 repeatedPos = mod(ray, boxedWorldDimen);

  // Adjust our coordinate system in the boxed world
  // to have a range of [-boxedWorldDimen/2.0, boxedWorldDimen/2.0)
  // placing the origin in the center
  repeatedPos += translation;

  // Stretch space to [-1.0, 1.0)
  // [Illustrative purposes, No stretching is needed]
  repeatedPos *= 1.0; 

  float crossDist = sdCross(repeatedPos / oneThird) * oneThird;

  // Acquire actual distance with correction to prev stretch
  // [Illustrative purposes, No correction is needed]
  crossDist /= 1.0;

  float sdIntersection = max(boundingBox, crossDist);
  return sdIntersection;
}
```

That over-complicated cross bound gives us the following.

Subtract it from our full sized box to create the 1st iteration! We’re definitely getting somewhere. :)

```
float sdMengerSpongeFirstIterationOvercomplicated(vec3 rayPos) {
  float crossDist = sdOneCrossOvercomplicatedUnbound(rayPos);
  float spongeBox = sdBox(rayPos, vec3(halfBoxDimen));
  float sdSubtraction = max(spongeBox, -crossDist);
  return sdSubtraction;
}
```

The differences between creating 1 cross and 27 crosses are fairly limited. The lines that differ are the following:

1.  Determining the size of the “stacked box worlds”

2.  Scaling (stretching) our the coordinate system of our boxed worlds to match the same range of the whole sponge

3.  Un-stretching our signed distance to compensate for the scaling of the boxed world coordinate system.

These are the interesting spots of our SDF that we should focus on to find an algorithm that calculates the distance to the Nth iteration of a Menger Sponge.

```
// extra argument for number of iterations
float sdMengerSponge(vec3 rayPos, int numIterations) {
  const float cubeWidth = 2.0;
  const float oneThird = 1.0 / 3.0;
  float spongeCube = sdCube(rayPos);
  float mengerSpongeDist = spongeCube;

  float scale = 1.0;
  for(int i = 0; i < numIterations; ++i) {
    // #1 determine repeated box width
    float boxedWidth = cubeWidth / scale;

    float translation = -boxedWidth / 2.0;
    vec3 ray = rayPos - translation;
    vec3 repeatedPos = mod(ray, boxedWidth);
    repeatedPos += translation;

    // #2 scale coordinate systems from 
    // [-1/scale, 1/scale) -> to [-1.0, 1.0)
    repeatedPos *= scale;

    float crossesDist = sdCross(repeatedPos / oneThird) * oneThird;

    // #3 Acquire actual distance by un-stretching
    crossesDist /= scale;

    mengerSpongeDist = max(mengerSpongeDist, -crossesDist);

    scale *= 3.0;
  }
  return mengerSpongeDist;
}
```

Ain’t she a beaut.

If you find this interesting at all, please dive into any of the following additional resources. There are certainly optimizations and other incredibly fun things you can do with ray marching and SDFs. This is only the beginning. Cheers!

Check out this ["Menger Prison"](https://lucodivo.github.io/menger_prison.html) and create the SDF without looking at the source code!
You will only need to use the tools you’ve learned about in this article!