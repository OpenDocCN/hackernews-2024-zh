<!--yml

category: 未分类

date: 2024-05-29 12:45:43

-->

# Ray Marching: Menger Sponge Breakdown

> 来源：[https://connorahaskins.substack.com/p/ray-marching-menger-sponge-breakdown](https://connorahaskins.substack.com/p/ray-marching-menger-sponge-breakdown)

有两种非常流行的方法来渲染3D场景。*光栅化*历来是实时图形（例如视频游戏）中最流行的方法。简而言之，光栅化是一种将由3D三角形组成的3D模型投影到2D平面上的技术。另一种流行的渲染方法是*光线追踪*。这种方法历来用于渲染需要更高现实感的场景，且不需要实时渲染（例如电影视觉效果）。

光线追踪涉及从每个像素向虚拟场景中的虚拟射线发射。计算射线与场景的交点以确定像素颜色。同样，这些都是渲染的3D模型，通常由3D三角形组成。与光栅化相比，光线追踪的工作顺序有所不同。光栅化从尝试绘制3D模型开始，并确定该3D模型将影响哪些像素。光线追踪从像素开始，并确定哪些3D模型会影响它。

本文将讨论的是第三种渲染方法。一种称为*射线行进*的射线投射方法。仍然会从屏幕的每个像素向虚拟世界发射射线。但是与*光线追踪*不同，后者仅进行一次计算以确定无限射线与3D模型之间的交点，*射线行进*采用更便宜、更迭代的方法。

每条射线都有相同的起始位置（虚拟摄像机的位置），并使用函数确定该射线当前位置与场景中*任何*元素的距离。为了计算虚拟场景中每个物体的距离，射线行进使用称为SDFs（有符号距离场）的函数。这些函数*不*确定射线在击中物体之前将行进的距离（光线追踪算法会这样做）。SDFs甚至不考虑射线的方向。它们只是确定射线当前位置与对象最近点之间的距离。因此，尽管它不能告诉我们射线遇到交点前需行进的确切距离，但它确实告诉我们可以安全行进的距离。并且，正如前文所述，射线行进是一个迭代过程。我们获得一个安全的行进距离，行进该距离，然后重复。当SDF返回的距离（射线当前位置与由SDF定义的对象之间的距离）低于某个指定阈值（例如0.0002单位）时，我们将认为这是一次*命中*，并为表面进行光照计算。

射线行进算法还必须处理两个边缘情况：

+   为了避免无限循环，当一条光线远离场景中的所有物体超过某个指定的最大距离（例如：100个单位）时，我们应该停止其继续行进。这很有帮助，因为有些光线可能根本不会在我们的场景中命中任何物体，而是直接穿过“天空”或“场景的空白”。

+   为了使光线行进在合理时间内完成，我们还应考虑我们愿意计算的最大迭代次数（例如：60次迭代）。

    +   当迭代成本昂贵时的一个很好的例子是表示长平面（墙）的SDF和射线与墙平行但位于我们指定的击中距离之外。在每次迭代中，由于到平面的距离保持非常小，射线以非常小的步长缓慢前进。由于这是一个实时程序，我们不能简单地等待这条射线永远运行下去。

光线行进算法考虑这两种终止情况（距离所有物体太远或迭代次数过多）为“未命中”，因为没有击中表面，所以不进行光照计算。在这两种极端情况下，我们为了性能进行了一种折衷。如果我们让光线再行进1个单位，可能会命中；如果我们再执行一次迭代，可能会命中。在指定最大距离和迭代次数时，请注意这一点。

总结一下，光线行进算法大致如下：

1.  使用光线当前位置和相应的SDF计算到物体的距离

1.  如果该距离小于指定的击中距离，我们认为这是一个击中。我们**退出算法**，并为击中的表面执行光照计算。

1.  将光线向前移动步骤（1）中计算的距离

1.  如果到场景的距离超过了指定的最大距离，我们认为这是一个未命中，并**退出算法**。

1.  如果下一次迭代将超过指定的最大迭代次数，我们认为这是一个未命中，并**退出算法**。

1.  回到步骤1，使用我们的光线的新位置

[曼德尔海绵](https://en.wikipedia.org/wiki/Menger_sponge)是一个三维分形。

它是这样构建的：

1.  将一个立方体切成3x3x3个相等的小立方体

1.  移除中心立方体和每个面的中心立方体

1.  对所有剩余的小立方体重复步骤1到3

我确实尝试解释大多数事物，但我不会解释一些概念，比如向量或坐标系。如果您注意到有任何未解释的内容，请询问，使用您喜欢的搜索引擎、LLM或本文末尾的资源寻找其他关于这些主题的精彩教育媒体。

本文讨论的许多内容都有演示。这个演示使用ShaderToy。它可以在任何浏览器中查看，并且可以在以下链接中找到。本文中的sdXxxx()风格的函数是您将在本文中学习的“有符号距离场/函数”。请随意修改代码！文本编辑器右下角有一个“?”按钮，提供大量有用信息。按下“alt + enter”或“cmd+enter”将触发编译，以便您可以查看更改的结果！

[https://www.shadertoy.com/view/MXB3Wt](https://www.shadertoy.com/view/MXB3Wt)

ShaderToy中的代码是使用GLSL ES编写的片段着色器，GLSL ES代表OpenGL着色语言嵌入系统。OpenGL是一个库，用于在GPU上运行各种着色器程序。着色器是在GPU上大量并行处理的程序。嵌入式系统实际上指除个人计算机外的任何东西，但在我们GLSL ES的用例中，主要指运行在移动设备上。使用GLSL ES的好处是它是GLSL的一个子集。因此，代码将在您的台式机、笔记本电脑或智能手机上运行。

本文严格关注一次在指定视口（可以是窗口，也可以是整个屏幕）中每个像素运行一次的片段着色器。但请注意，这并不是片段着色器的唯一（甚至不是主要）使用方式。

下面是两个OpenGL ES代码片段，实现了射线行军算法。第一个带有详细的注释，第二个没有（用于提高代码可读性）：

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

上述代码是纯粹的GLSL ES着色器代码。在ShaderToy中运行的GLSL ES代码会稍有不同。特别是关于`#version`、`screenResolution`、`gl_FragCoord`和`FragColor`部分。读者可以进一步理解它们之间的差异。

如果这是你第一次使用GLSL或片段着色器，或者以上内容让你有些不知所措，我向你保证，它并不像看上去那么可怕。如果你对以上代码不是立刻理解，也没关系。这不会影响你理解文章的其余部分。本文末尾还有其他资源，可能会帮助你在学习过程中。我建议继续阅读文章，并回来重新探索算法。从现在开始，我们不再讨论这个算法。我们只关注将SDF开发到一个Menger海绵中。

*vec3*：GLSL中的数据类型，简单来说是由3个浮点值组成的数组。这种数据类型通常用来表示点、向量或颜色值（RGB）。

“*halfwidth*”：等于宽度的一半的值。通常情况下，我们会发现处理正在渲染的对象宽度的一半更方便。这类似于使用圆的半径（halfwidth）而不是直径（width）的方程。

成分：一个2D向量有两个成分（*x*和*y*）。一个3D向量有三个成分（*x*、*y*和*z*）。

GLSL：OpenGL着色语言。在本文中，“OpenGL”可以与“OpenGL ES”互换使用，因为我们不会使用OpenGL ES之外的任何GLSL功能。

射线行进需要使用签名距离场(SDFs)来获取点与某个对象之间的距离。但在确定点与某个复杂的3D对象之间的距离之前，让我们先提醒自己如何在2D平面上找到两点之间的距离。

假设你有两个点，A和B。当你从B中减去A时，你得到一个可以“移动”点A到点B的向量。可能这听起来对你来说不直观，所以这里有一个简单的方法来巩固这个概念。现在假设你有两个整数+3和+7。当你从+7中减去+3时，你得到整数+4。这个整数能够将+3带到+7。按照同样的逻辑，当你从点(-4, -2)减去点(-1, 2)时，你得到向量<3, 4>。这是一个可以将点(-4, -2)“移动”到点(-1, 2)的向量。这只是加法和减法。

现在两点之间的距离相当简单。如果你有一个向量<x, y>，它把点A带到点B（<x, y> = B - A），那么点A到点B的距离就是那个向量的长度。如果你画出向量<x, y>，你会看到这个向量可以看作是一个直角三角形的斜边，其中一条边长为x，另一条边长为y。

因此，要获得向量的长度，我们简单地使用勾股定理，即a² + b² = c²。

+   向量<x, y>的长度等于sqrt(x² + y²)

+   向量<x, y, z>的长度等于sqrt(x² + y² + z²)

+   这种模式也可以扩展到所有维度。

总结一下，点A和点B之间的距离可以通过计算从A减去B时得到的结果向量的长度来获取。

到点的SDF看起来像这样：

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

要获得从一个点（例如：射线当前位置）到一个球的距离，我们首先计算点与球的中心（另一个点）之间的距离。当前往球的中心时，您必须首先到达球的表面，然后进一步行进球的半径，最终到达中心点。因此，如果我们想要到一个球的距离，我们首先得到到中心点的距离，然后减去半径的长度，这使我们回到了球的表面。

到球的SDF看起来像这样：

```
float sdSphere(vec3 rayPosition, vec3 sphereCenterPosition, float radius) {
  vec3 centerToRay = rayPosition - sphereCenterPosition;
  float distToCenter = length(centerToRay);
  float distToSurface = distToCenter - radius;
  return distToSurface;
}
```

注意：射线行进的另一个名称是“球追踪”。你可以将SDF视为一个函数，它给出以射线位置为中心并刚好接触由SDF表示的对象或场景的球的半径。

让我们学习一些通过使用我们的射线和SDFs操作对象的方法。

在讨论球体的SDF时我遗漏了一件事情，那就是SDF通常会定义到原点为中心的对象的距离。而你实际上将该对象定位到场景中的方式是，在发送到SDF之前，只需将对象的期望位置从你的射线位置中减去即可。通过减去对象的期望位置，我们将射线的位置转换为对象的局部坐标系，其中对象以原点为中心。

在下面的图表的左侧，我们展示了射线的实际位置以及圆圈的期望位置。在中间附近，我们展示了我们射线的位置转换为圆圈的局部坐标系，其中圆圈的中心位于原点。这个射线的位置是通过简单地减去圆圈的期望位置来转换的。请注意，在图表的两种情况下，射线与圆圈的距离是相同的。这是至关重要的，因为SDF的唯一工作就是准确获取到对象的距离。

这简化了SDF内部的逻辑，因为它不再关注对象的翻译。

我们的新球体的SDF及其平移现在看起来是这样的：

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

与平移类似，实际上SDF并不需要处理对象的确切尺寸（例如球的半径），因为我们可以在SDF外部将对象缩放到我们期望的尺寸。缩放比平移略微不明显，但技术非常容易记住。就像平移一样，我们要在发送到SDF之前操作我们射线的位置，以便操作我们的对象。

让我们看看如何缩放某些东西。而且非常快速地，让我们考虑一下什么是不行的。如果半径为1.0的球体离我们2.0个单位远，那么将该球体放大到半径为2.0意味着它现在距离为1.0单位。这个球体变大了，我们的距离现在减半了。通过这个例子，似乎我们可以简单地将到对象的距离除以我们希望缩放的值。但是再举一个例子，我们就会发现这个方法行不通了。如果半径为1.0的球体距离为*3.0个单位*，将该球体放大到半径为2.0意味着它现在距离为*2.0个单位*。球体变大了，但现在我们的距离只有原始长度的2/3。嗯嗯嗯……

我们将采取的方法是假装我们的射线存在的坐标系已经缩小了，而 SDF 所代表的物体保持不变。从射线坐标系的角度来看，物体已经增大了。唯一的问题是我们计算的距离是在我们缩小的坐标系中的距离。为了纠正这一点，我们必须将计算的距离映射回我们原始的坐标系中（取消缩小）。因此，如果我们将我们的坐标系缩小了 7.0 倍（以将物体放大到其原始尺寸的 7.0 倍），那么我们计算的距离将需要乘以 7.0 来映射回原始坐标系。

在下图中：黑色表示我们的原始射线，单位圆和它们之间的距离。红色表示我们期望的缩放圆和缩放圆与射线之间的距离。

在下图中：黑色表示我们缩小的射线，原始圆形（从射线的视角“放大”）和它们之间的距离。红色表示我们将距离映射回原始坐标系（取消缩小）。

缩小物体的方法与此相同，并留作读者的练习。

我们的 SDF 比以往更简单，因为它不再关心球体的尺寸或位置！它只接收射线位置作为参数。

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

如果 SDF 不需要担心物体的尺度，为什么在 `sdSphere` 中定义了 `radius` 的大小？

为了缩放由 SDF 定义的物体，它必须具有*一定*的初始尺寸（缩放零是零）。在这种情况下，球体的半径为 1，直径为 2。这就是将被缩放的数字。

当试图计算到物体的距离时，我们经常可以通过利用物体的对称性来简化问题。对称性是许多基本形状的一种极为常见的属性。例如：立方体、球体、圆柱体和环面在原点对称中心且合理取向时，可以看作沿所有轴对称。下图显示了一个在 x 轴和 y 轴上均对称的物体。

请注意，相同颜色的每个点与物体的距离相同。这就是折叠变得非常有用的地方。将任意点的 x 和 y 分量的绝对值映射到正象限（右上角）内的相同颜色点。通过利用物体的对称性和射线位置的绝对值，我们可以使我们的 SDF 专注于仅找到正象限内点到物体的距离。

为了演示这样可以简化事务，让我们来看一个立方体的 SDF。

让我们从计算到正方形的距离开始。在这个例子中，我们将利用正方形的对称性。我们将仅使用点的绝对值。在下图中，我们将正象限分为四个部分，以理解我们可能计算到正方形距离的不同方法。

让我们先来看看右上角（黄色）分区。如果我们的X和Y分量都大于正方形宽度的一半，我们只需计算到正方形角落的距离。仅考虑这一点，我们的SDF看起来会像这样：

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

现在，让我们结合左上角（蓝色）和右下角（紫色）部分。想象一下从角落指向左上角点和从角落指向右下角点的向量（仍然在右上象限内）。这个向量并不代表到正方形的最短路径。如果我们从角落到我们的射线的向量中有任何一个分量的值为负（当指向左上角点时，x分量为负，指向右下角点时，y分量为负），我们可以简单地通过将其设置为零来在我们的距离计算中忽略该分量。左上点只需向下移动即可击中正方形上的最近点，而右下点只需向左移动。

更新后的SDF如下所示：

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

`max()`函数是一个分量级函数，在GLSL中可用，其中向量的每个分量将是其当前值和传入的第二个参数（在本例中为`0.0`）之间的最大值。这有效地将我们向量的所有负值夹在了`0.0`。那么，如果我们在正方形内部，我们的SDF当前返回什么呢？在这种情况下，我们向量的所有分量都将为负，然后被夹在了`(0.0, 0.0)`。该向量的长度为零。这是构建上述的最佳数字。

只有当从角落指向我们的射线位置的向量的所有分量都为负时，我们才在正方形内部。回顾图表以验证其真实性。在正方形内部时，我们的最近点就是距离左墙或右墙的距离。这些目前存在于我们的`cornerToRay`向量的分量值中。

现在是解释“Signed Distance Field”中的“Signed”好时机。当一个点在其定义的形状外部时，SDF应返回一个正的非零值。当一个点在其定义的形状内部时，它应返回一个负的非零值。当一个点恰好位于其定义形状的表面上时，它应返回零。

在日常数学中，-5和-3的最小值是-5。这就是在任何你将看到的代码中`min`函数的工作方式，它也适用于GLSL。但是，因为我们正在计算到由SDF定义的某个对象的最短距离，-3单位实际上是比-5单位更小的*有符号距离*。

回到我们内部的左下点... 由于从角到点的向量的x和y分量都为负，我们取这些值的*最大*值来找到最小的*有符号*距离。

有了这些额外的信息，让我们完成我们的SDF。

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

由于`distToInsideRay`将在射线存在于正方形外部时等于`0.0`，并且如前所述，`closestToOutsideRay`将在射线存在于正方形内部时等于`0.0`，因此最终加法的至少一个操作数保证为`0.0`。这意味着返回的值将是*closestToOutsideRay*的长度或*distToInsideRay*的长度。

对于一个立方体的SDF使用与刚才描述的所有逻辑相同，但增加了一个维度。SDF可以表示为以下内容：

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

现在我们拥有了创建无限十字所需的所有工具。您可以考虑通过从我们的`sdCube`开始并无限延伸每个面来创建无限十字。下图显示了我们无限十字的2D表示。如您所见，每个点的距离公式已经发生了相当大的变化。我们四个原始点中的三个现在位于对象内部，只有一个位于外部。

尽管到每个点的距离函数已经改变，但方程看起来并不陌生。让我们从覆盖到左下（红色）点的距离开始。再次提醒一下，我们通过绝对值函数折叠空间，并且只关心右上象限。到左下（红色）点的最近距离显然是角点，我们已经在之前看到了这个距离计算。

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

当一个点在对象外部时，有符号距离场返回正值，而当它在对象内部时返回负值。SDF返回一个*有符号*值，但向量的长度始终为正值。在这种情况下，我们的点在十字内部，因此简单的否定解决了我们的问题。

事情继续看起来与我们使用`sdSquare`的方法相似。对于`sdSquare`，上左（蓝色）和下右（紫色）点的定义是它们到水平和垂直边缘的距离。有趣的是，在找到到这个无限十字的距离时，它们的关注点已经交换了。上左点的最小距离位于垂直边缘，下右点的最小距离位于水平边缘。在`sdSquare`中，`cornerToRay`向量中的负值被夹紧到零。在我们的`sdCross`中，正值被夹紧到零。相同的`cornerToRay`向量用于两个SDF，但其组件的夹紧方式不同。

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

到目前为止，我们的 SDF 覆盖了所有位于十字形内部的可能点，对于这些点，SDF 会返回负值。剩下的只是正距离。正如我们在上一个图中看到的，离十字形外的点的距离只是我们点的最小分量与十字形半宽度的差异。也就是 `cornerToRay` 向量的最小分量。

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

要使一个点存在于十字形之外，两个分量的值必须大于我们的十字形的 `halfWidth`。从点中减去角点相当于从两个分量中减去半宽度。如果最小分量大于 `0.0`，那么它们都是如此，距离就是最小分量。否则，我们就在十字形内部，我们之前计算的距离是所需的。就像 sdSquare 一样，最终的加法更多地起到一个是/否的作用，因为其中一个运算数保证为 `0.0`。

为十字形添加一个维度并不是很明显，所以我们来谈谈这个问题。对于二维“无限十字”，我们确定当两个分量都大于十字形的半宽度时，射线在十字形之外，反之则在十字形内。三维无限十字的区域是什么样的？我说的三维“无限十字”是什么？

我们对二维无限十字的定义是一个正方形，每个面都被无限延伸出去。我们对三维无限十字的定义是一个*立方体*，其中每个面都被无限延伸出去。

对于我们的二维无限十字，我们有四个射线可能的位置：

1.  在原始正方形内部。这发生在两个分量都小于半宽度时。最近距离是内角（取反）。

1.  在沿 x 方向延伸的挤出面内部。这发生在只有 y 分量小于半宽度时。最近距离是 y 分量减去半宽度。

1.  在沿 y 方向延伸的挤出面内部。这发生在只有 x 分量小于半宽度时。最近距离是 x 分量减去半宽度。

1.  超出十字形。这发生在两个分量都大于半宽度时。最近距离是最小的分量。

注意：对于三维无限十字，我们将空间折叠到正八分位，我们只能从正八分位的角度考虑这个问题。如果你发现自己在思考负坐标的情况……不要！物体在所有轴上的对称性意味着我们可以使用绝对值将所有内容折叠到正八分位，创建一个算法来处理单八分位内的所有点，并忽略所有其他情况！

对于我们的三维无限十字，我们有*五个*独特的位置：

1.  在原始立方体内部。当所有分量都小于 `halfwidth` 时发生。最近距离是到最近的内部边缘（取反）。请注意，中心不是由六个面构成的立方体。它是由12条边构成的立方体。它只是一个立方体的骨架。我们感兴趣的距离是点与这12条边之一之间的距离。如果你想象每条边都是二维平面上的角，你可能会注意到最近的边是由我们射线的三维位置的最大两个分量确定的。

1.  在延伸至 x 方向的挤压面内部。当只有 x 分量大于 `halfwidth` 时发生。最近距离可以被构建为 yz 平面上的一个正方形内部。最近距离是 `max(y,z)` 减去 `halfwidth`。

1.  在延伸至 y 方向的挤压面内部。当只有 y 分量大于 `halfwidth` 时发生。最近距离可以被构建为 xz 平面上的一个正方形内部。最近距离是 `max(x,z)` 减去 `halfwidth`。

1.  在延伸至 z 方向的挤压面内部。当只有 z 分量大于 `halfwidth` 时发生。最近距离可以被构建为 xy 平面上的一个正方形内部。最近距离是 `max(x,y)` 减去 `halfwidth`。

1.  超出十字之外。当两个或更多分量大于 `halfwidth` 时发生。最近距离是到最近的挤压面。最近的挤压面是在最大分量方向上延伸的面，到该挤压面的距离是两个最小分量的因素。

让我们更深入地思考 #5，超出十字之外。在 x 方向移动可以看作是沿着 x 方向平行于挤压面，且垂直于 y 和 z 方向的挤压面移动。这意味着沿着 x 方向移动对 x 方向的挤压面距离没有影响，而增加了与 y 和 z 方向的挤压面的距离。这有点像一场游戏，其中最大的分量将点从其他两个挤压面推得更远，并且其自身的挤压面“赢得”了保持到点的最近距离。沿着 x 轴移动是移动平行于 x 方向的挤压面，这意味着它不能同时成为决定到同一挤压面距离的因素。

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

射线行进中另一个非常强大的工具是模运算符。如果不熟悉，模运算符（通常表示为‘mod’或‘%’）简单地返回除法的余数。例如，如果你将 11 除以 4，你得到商为 2，余数为 3。所以 11 mod 4 是 3。

下图显示了使用模3转换各点的x和y分量时会发生什么。它将我们原始的单一无界坐标系统划分为每个轴的范围为[0, 3.0)的无限*有界*坐标系统。该图表展示了我们原始坐标系统如何被切成片段。每个片段将被映射到突出显示的区域中的彩色点。为了进一步演示，模运算符将每个点映射到突出显示区域中相同颜色的点上。

注意：模运算符实际上对负数未定义。很可能你的平台上的模运算符不会产生与上图中所示相同的结果。

+   在GLSL中，`mod(x, y)`被定义为`x - (y * floor(x/y))`。如果感兴趣，你可以尝试一下，看它是否产生我们期望的结果（无限重复的相同有界坐标空间）。如果在GLSL之外使用模运算看起来不正确，这可能是你的问题。要验证这是否是你的问题，只需将模运算符替换为上述GLSL定义。

你可能已经注意到我们的盒式坐标系统有一个小烦恼，即原点现在位于左下角。为了获得一个位于中心的原点，我们只需减去盒式坐标系统长度的一半。

在上面的图中，我们对我们的射线位置的x和y分量执行模3运算。

在下图中，我们正在执行相同的操作，然后减去1.5（3的一半）。

有两点需要注意。再也没有一个“真正”的片段保持不变。我们的坐标系现在从[-1.5, 1.5)开始，其中包括下边界但不包括上边界。当包含下边界为零时，这可能感觉更直观，但记住这在这里仍然是正确的。

在二维中使用模运算没有什么特别的。二维图表只是更容易绘制和感受。在三维空间中，模运算创建了无限有界的坐标系统，其形状类似于一个长方体或立方体，而不是一个矩形或正方形。

在我们直接创建曼格尔海绵之前的最后一步是理解如何组合原始形状以创建更复杂的对象。我们将要查看的操作包括从一个形状中减去另一个形状，两个形状的并集以及两个形状的交集。

让我们首先努力分析上面的图片。我们有两个相互交叉的形状，一个圆形和一个正方形。我们还有5个点代表5个具有唯一符号距离的区域，每个区域分别与这两个形状中的一个有关。

+   P1：对两者都有正距离，其中`sdCircle()` < `sdSquare()`。

+   P2：对圆的距离为负。对正方形的距离为正。`sdCircle()` < `sdSquare()`。

+   P3：对两者都有负距离。

+   P4：对圆的距离为正。对正方形的距离为负。`sdSquare()` < `sdCircle()`。

+   P5: 对于两者的正距离，其中 `sdSquare()` < `sdCircle()`。

我们将使用的操作来组合对象并不完美，我们将遇到一个我们以前没有见过的限制。由于一个点位于组合对象内部导致的有符号距离不正确。对于我们的目的，这将是完全可以接受的。在通过光线行进进行渲染时，不希望将光线投射到对象的内部。光线应该命中表面或者完全错过，而不应穿过对象进入其内部。

要了解更多关于这些 SDF 布尔运算的局限性，[Inigo Quilez（ShaderToy 的创作者和光线行进与 SDF 的重要贡献者）撰写了一篇关于这个主题的文章，我建议稍后阅读并收藏。](https://iquilezles.org/articles/interiordistance/)

```
float union = min(sdShape1(rayPos), sdShape2(rayPos))
```

+   P1: `sdCircle()` 是正距离并且比 `sdSquare()` 更小的距离，因此将使用到圆的距离。

+   P2: 合并内部，定义不明确。

+   P3: 合并内部，定义不明确。

+   P4: 合并内部，定义不明确。

+   P5: `sdSquare()` 比 `sdCircle()` 更小的距离，因此将使用到距离正方形的距离。

```
float subtraction = max(-sdShape1(ray), sdShape2(ray))
```

+   P1: `sdSquare()` 是正距离，而 `-sdCircle()` 是负距离，因此将使用到正方形的距离。

+   P2: `-sdCircle()` 和 `sdSquare()` 都是正距离，因此将使用最大的距离。

+   P3: `-sdCircle()` 是正距离，而 `sdSquare()` 是负距离，因此将使用到 `-sdCircle()`。

+   P4: 减法内部，定义不明确。

+   P5: `sdSquare()` 比 `-sdCircle` 更大的距离，因此将使用到正方形的距离。

```
float intersection max(sdShape1(ray), sdShape2(ray))
```

+   P1: `sdSquare()` 比 `sdCircle()` 更大的距离，因此将使用到正方形的距离。

+   P2: `sdSquare()` 是正距离，而 `sdCircle()` 是负距离，因此将使用到正方形的距离。

+   P3: 减法内部，定义不明确。

+   P4: `sdCircle()` 是正距离，而 `sdSquare()` 是负距离，因此将使用到圆的距离。

+   P5: `sdCircle()` 比 `sdSquare()` 更大的距离，因此将使用到圆的距离。

我们将继续以非常明确的步骤创建海绵。首先，让我们创建一个宽度与即将成为门格尔海绵相同的无限十字架。我们将其限制在一个盒子内，其大小是十字架宽度的两倍。这个边界框仅用于更好地理解我们正在处理的内容。代码和生成的图像看起来像这样。

```
float sdBoundedCross(vec3 rayPos) {
  float boundingBoxDist = sdCube(rayPos / 2.0) * 2.0;
  float crossDist = sdCross(rayPos);
  float sdIntersection = max(boundingBoxDist, crossDist);
  return sdIntersection;
}
```

对于门格尔海绵的第一次迭代，我们需要减去一个十分之一的十字架，其宽度为海绵本身的三分之一，并且与每个面的中心对齐。考虑到这一点，让我们将我们的十字架变窄到海绵尺寸的三分之一。

```
float sdBoundedCrossSlimmed(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  float boundingBoxDist = sdCube(rayPos / 2.0) * 2.0;
  float crossDist = sdCross(rayPos / oneThird ) * oneThird;
  float sdIntersection = max(boundingBoxDist, crossDist);
  return sdIntersection;
}
```

现在为了使事情更清晰一些，让我们绘制门格尔海绵的最外层立方体，并与我们所拥有的内容进行合并。

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

如果你熟悉基本的布尔运算，那么显然我们只需进行减法就可以得到我们门格尔海绵的第一次迭代。顺便说一句，我们将移除边界框，因为它不再作为视觉助手。

```
float sdMengerSpongeIteration1(vec3 rayPos) {
  const float oneThird = 1.0 / 3.0;
  float crossDist = sdCross(rayPos / oneThird) * oneThird;
  float spongeBox = sdCube(rayPos);
  float sdSubtraction = max(spongeBox, -crossDist);
  return sdSubtraction;
}
```

还有一点小窥视...

这就是你一次做到的。但我们如何迭代地做到这一点呢？好吧，让我们从准确思考我们想要的东西开始。第一次迭代是取全盒子，将每个维度切成三分之一，结果是27个等大的盒子，移除每个面中心的6个盒子，以及中心的一个盒子。在第二次迭代中，我们取剩下的20个盒子并执行相同的操作（切割成27个等盒子并移除）。然后我们对剩余的400个盒子执行这些操作，依此类推。

作为朝正确方向迈出的一步，让我们看看将一个框简单地切割成这27个等大的盒子是什么样子。正如你所想象的，答案并不是手动找到27个硬编码的盒子的距离。为此，我们将利用模运算符将我们的单个无限空间转换为无限重复的有界空间。让我们首先用模创建一堆盒子。

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

模运算简单地将世界切割成无限盒子，它们每个的宽度都是2.0单位。在每个这些重复的“盒子世界”内，我们画了一个稍小的盒子。不过有一个相当大的问题。通过选择一个不太好的边界框比例来展示这一点，并从另一个角度看一下。

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

更像是`sdJankyBoundedBoxField()`，对吧？？虽然不是我们想要的，但它还是挺酷的，是吧？！

因此，我们的问题是，当执行我们的模运算符时，我们正确地将空间切割成宽度为2单位的盒子。问题在于我们的坐标系所有轴都从[0, 2)开始，但我们的SDF返回到原点处的盒子距离。当前，我们正在渲染的盒子“渗出”超出其空间，超出了盒子世界的原点，并引起各种奇怪的伪影。

我们需要做的是稍微调整一下，使每个我们盒子世界的原点位于中心。我们只需减去它们宽度的一半，而不是坐标系统，其中每个轴从[0, 2)变为从[-1, 1)的坐标系统。

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

*Bada bing, bada boom.* 我们得到了一个合适的盒子场。

现在我们有了一堆立方体，让我们试着将它们缩减到我们所需的27个。

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

差不多了，但看起来我们又有了另一个问题。我们有了正确大小的盒子。它们都正确地堆叠了。但有些盒子切得很奇怪，如果没有中心盒子来切割，我们怎么能创建门格尔海绵呢？问题在于它们不是我们希望的对齐方式。这些盒子当前是沿着每个轴并排对齐的。我将执行一个与十字形的交叉，给我们一个清晰的视觉，看看这里出了什么问题。

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

正如你在这里看到的，取模运算符使我们的“盒子世界”对齐，这样其中八个角接触原点，沿轴的任何其他地方都将被四个盒子世界所环绕。为了将盒子与以原点为中心的一个盒子对齐，我们将必须相应地平移这些盒子。

希望到这一步，你已经明白我们并没有实际上有限数量的盒子。我们创建了一个无尽的盒子世界，并且只显示与我们的边界框相交的盒子，通过与十字架的交叉点。由于我们有无限的盒子，我们可以以许多不同的方式平移这些立方体以获得我们想要的结果。我们只需要一个中心立方体，任何一个都可以。但对于我们的情况，我们将继续将位于我们世界正半轴（在<0, 0, 0>和<2/3, 2/3, 2/3>之间）的第一个立方体平移到中心位置。

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

就是这样！这正是我们想要的。你曾见过的最无聊的魔方。也是我们海绵的完美开端！

你可能已经猜到了，但根据我们现在所拥有的内容，我们非常接近蒙格海绵的第二次迭代！让我们使用我们堆叠的盒子世界来创建十字架，而不是立方体！每一个盒子世界的宽度的三分之一（以及我们整个海绵的九分之一）。

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

你能看到这件事的发展方向吗？

我们需要做的就是解绑十字架与我们的边界框，因为视觉帮助不再需要，然后从我们蒙格海绵的第一次迭代中减去我们所拥有的。

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

现在我们真的有了进展！

现在我们有了两个蒙格海绵的迭代，让我们看看是否能找到一个单一的算法来创建这两个迭代。我解决这个问题的方法将从直觉开始。我会尽力描述我能感受到这些情感的最佳位置。

制作第二轮的27个十字架似乎需要大量必要的计算。例如，取模运算符似乎非常难（不可能？）移除。相比之下，创建第一轮的一个十字架感觉非常容易。我看到在使用与第二轮非常接近的逻辑创建一个“过于复杂”的第一轮中存在着一个更简单的前进路径。希望这可能会导致一个更通用的算法，帮助我们产生蒙格海绵的第N次迭代。

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

那个过于复杂的跨界给我们带来了以下情况。

从我们的全尺寸盒子中减去，以创建第一次迭代！我们确实有所进展。 :)

```
float sdMengerSpongeFirstIterationOvercomplicated(vec3 rayPos) {
  float crossDist = sdOneCrossOvercomplicatedUnbound(rayPos);
  float spongeBox = sdBox(rayPos, vec3(halfBoxDimen));
  float sdSubtraction = max(spongeBox, -crossDist);
  return sdSubtraction;
}
```

创建1个十字架和27个十字架之间的差异是非常有限的。不同的线路如下：

1.  确定“堆叠的盒子世界”的大小

1.  缩放我们的盒子世界坐标系，以匹配整个海绵的相同范围

1.  为了补偿盒子世界坐标系的缩放，取消拉伸我们的符号距离。

这些是我们的SDF中有趣的地方，我们应该专注于找到一个计算到门格尔海绵第N次迭代距离的算法。

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

这难道不是美吗？

如果你对这个有兴趣，请深入研究以下任意一个额外的资源。当然，使用射线行进和SDF可以进行优化和其他非常有趣的事情。这只是一个开始。加油！

查看这个["门格尔监狱"](https://lucodivo.github.io/menger_prison.html)，并且不看源代码创建SDF！

你只需要使用本文介绍的工具！
