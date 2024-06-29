<!--yml

类别：未分类

日期：2024-05-29 13:26:56

-->

# 妈妈，没有矩阵！

> 来源：[https://enkimute.github.io/LookMaNoMatrices/](https://enkimute.github.io/LookMaNoMatrices/)

# 妈妈，没有矩阵！

**Steven De Keninck**

自2019年SIGGRAPH课程[1]以来，几何代数及欧几里得PGA（基于平面或射影几何代数）在计算机图形和机器学习社区中备受关注[2, 3, 4]。尽管其应用广泛，包括高维几何和物理学，但其在传统3D图形中的采用却受到限制，通常仅将双四元数重新包装为PGA引擎。“妈妈，没有矩阵！”项目旨在通过引入一个现代化的、符合glTF标准的前向渲染3D引擎，完全整合PGA代数，拓展PGA的应用范围。

在这篇文章中，我们将介绍这个项目，重点介绍在转向PGA实现时所需的解决方案和技术。有时候很诱人从现有技术开始，并尝试进行“代数级别”的转换。然而，这往往导致不令人满意的解决方案，为了使PGA真正发挥其潜力，通常需要进行更基本的重新审视。确实，没有几何的代数是盲目的。

## 介绍。

计算机图形中无处不在的是矩阵。事实上，有段时间，4x4矩阵既被固定在GPU中，也是所有图形API的强制部分。这个项目就简单地不可能存在。然而，今天，在AI进步的推动下，GPU变得高度可编程的标量处理器，不再受限于已经消失的固定功能流水线。然而，4x4矩阵仍然无处不在。它们可以表示所有的线性变换，而典型的前向图形管道确实涉及刚性和投影变换。看起来是一个很好的匹配。

四元数在计算机图形中也无处不在。事实证明，矩阵具有不理想的插值特性，而现代格式如Khronos的glTF使用四元数来满足其所有旋转需求。这对动画来说非常棒，并且通常认为其值得为了不可避免的矩阵转换而付出的代价。

然而，在现实世界中，你典型的3D引擎设置中的绝大多数矩阵都将是正交矩阵，仅编码旋转和平移。而PGA的运动流形正是在此时派上用场。以较低的计算和内存成本，PGA运动编码了完整的欧几里得运动，此外还提供了无需转换即可包含四元数和双四元数的能力。

当然，这引发了一个问题 - 我们能否用它们的PGA等价物替换典型前向渲染器中的所有矩阵？真正的无矩阵设置可能或者甚至是可取的吗？只有一种方法来找出答案……

但在开始之前，有一个简短的免责声明。本项目旨在无需妥协地替换矩阵。当然，这并不是解决任何工程问题的正确方法。作为本项目的参考，我们使用了Khronos glTF查看器。这是一个明智的选择，因为毫无疑问其他人也会将其作为参考，但它并不试图成为最佳实现。它完全使用4x4矩阵和它们的内置glsl运算符来进行大多数操作，而经验丰富的图形程序员知道在那里仍然存在优势。这里的重点不是做出最优实现，特别是考虑到这里的一些发现，最优实现很可能是一个混合解决方案，未来会进行详细的介绍！

## FPGA：快速PGA！

本文不涵盖PGA的完整介绍，我们假设读者至少熟悉[1, 5, 6]中的内容。我们将专注于为这个特定实现所做的选择，并详细说明所需的基本运算符，无论是在CPU还是GPU上。

### 基础和基础

PGA代数由四个基向量 $\mathbf e_0$ 到 $\mathbf e_3$ 生成。向量 $\mathbf e_1, \mathbf e_2, \mathbf e_3$ 分别映射到 $x=0, y=0, z=0$ 平面，而特殊的退化向量 $\mathbf e_0$ 表示无穷远平面。然后这四个生成器结合形成六个双向量，四个三向量和一个单个四向量，再加上标量代表所有PGA元素。我们选择的特定基础和内存布局旨在处理典型图形数据时最小化转换。PGA的所有元素都是密切相关的，我们的选择概述以及它们如何映射到变换和几何元素在下表中详细说明（第二行表示每个元素的平方）：

| e1 | e2 | e3 | e0 | 1 | e23 | e31 | e12 | e01 | e02 | e03 | e0123 | e032 | e013 | e021 | e123 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| +1 | +1 | +1 | 0 | +1 | -1 | -1 | -1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | -1 |
| 平面反射 | 电机 / 对偶四元数 / 李群 | 点反射 |
| 四元数 |  |
| 平面 $a\mathbf e_1 + b\mathbf e_2 + c\mathbf e_3 + d\mathbf e_0 = 0$ |  | 通过原点的直线 | ∞ 线 |  | 点 $(x\mathbf e_1 + y\mathbf e_2 + z\mathbf e_3 + w\mathbf e_0)^*$ |
| 线 / 李代数 |
| 向量 | S | 双向量 | PSS | 三向量 |

这些选择体现在以下简单的着色器结构中，我们选择保持在内置类型内以保留加法、减法和标量乘法。（glsl 不支持自定义类型的运算符重载）。

```
#define motor     mat2x4  // [ [s, e23, e31, e12], [e01, e02, e03, e0123] ] 
#define line      mat2x3  // [ [e23, e31, e12], [e01, e02, e03] ]
#define point     vec3    // [ e032, e013, e021 ] implied 1 e123
#define direction vec3    // [ e032, e013, e021 ] implied 0 e123
```

### 获得你的几何积！

我们定义了数据结构后，可以集中精力实现我们将需要的（部分）PGA乘积。特别关注组合和夹心运算符 - 矩阵-向量乘法的数值效率是众所周知的，正如我们将发现的，需要一些创造力来使PGA达到标准。

#### 变换组合。

PGA的8浮点电机，同构于双四元数，自然带有形式化乘积中的高效组合运算符。回想一下，两个4x4矩阵的乘积需要64次乘法和48次加法。对于两个一般PGA电机，它们的组合仅需48次乘法和40次加法。在CPU上实现其系数级产品如下：

```
// 48 mul, 40 add
export const gp_mm = (a,b,res=new baseType(8)) => {
  const a0=a[0],a1=a[1],a2=a[2],a3=a[3],a4=a[4],a5=a[5],a6=a[6],a7=a[7],
        b0=b[0],b1=b[1],b2=b[2],b3=b[3],b4=b[4],b5=b[5],b6=b[6],b7=b[7];
  res[0] = a0*b0-a1*b1-a2*b2-a3*b3;
  res[1] = a0*b1+a1*b0+a3*b2-a2*b3;
  res[2] = a0*b2+a1*b3+a2*b0-a3*b1;
  res[3] = a0*b3+a2*b1+a3*b0-a1*b2;
  res[4] = a0*b4+a3*b5+a4*b0+a6*b2-a1*b7-a2*b6-a5*b3-a7*b1;
  res[5] = a0*b5+a1*b6+a4*b3+a5*b0-a2*b7-a3*b4-a6*b1-a7*b2;
  res[6] = a0*b6+a2*b4+a5*b1+a6*b0-a1*b5-a3*b7-a4*b2-a7*b3;
  res[7] = a0*b7+a1*b4+a2*b5+a3*b6+a4*b1+a5*b2+a6*b3+a7*b0;
  return res;
}
```

该代码块经过重新排列和模式匹配后，可以使用点积和叉积在glsl中编写如下：

```
// 48 mul, 40 add    
motor gp_mm( motor a, motor b ) {
  return motor(
         a[0].x*b[0].x   - dot(a[0].yzw, b[0].yzw), 
         a[0].x*b[0].yzw + b[0].x*a[0].yzw + cross(b[0].yzw, a[0].yzw),
         a[0].x*b[1].xyz + b[0].x*a[1].xyz + cross(b[0].yzw, a[1].xyz) + cross(b[1].xyz, a[0].yzw) - b[1].w*a[0].yzw - a[1].w*b[0].yzw, 
         a[0].x*b[1].w + b[0].x*a[1].w + dot(a[0].yzw, b[1].xyz) + dot(a[1].xyz, b[0].yzw));
}
```

尽管已经相当高效，上述代码块处理一般电机，而有许多情况我们处理例如纯平移或绕原点旋转。在这些情况下，许多电机系数将为零，将上述代码块重新工作以包含这一点是一项简单的任务。例如，对于两个绕原点旋转的组合，我们发现只需要16次乘法和12次加法：

```
// 16 mul, 12 add
motor gp_rr( motor a, motor b ) {
  return motor( a[0].x*b[0] + vec4( -dot(a[0].yzw, b[0].yzw), b[0].x*a[0].yzw + cross(b[0].yzw,a[0].yzw) ), vec4(0.) ); 
}
```

我们的实现为任意组合的平移（_t）、绕原点旋转（_r）和一般运动（_m）提供了这些优化版本。

| 操作 | 乘法 | 加法 |
| --- | --- | --- |
| gp_mm | 48 | 40 |
| gp_rr | 16 | 12 |
| gp_tt | 0 | 3 |
| gp_rt / gp_tr | 12 | 8 |
| gp_rm / gp_mr | 32 | 24 |
| gp_tm / gp_mt | 12 | 12 |

#### 变换点

使用夹心乘积在PGA中对点$p$进行变换，情况更为复杂。$$ p' = M p \widetilde M $$ 对这两个几何乘积进行朴素计算结果是33次乘法和29次加法，比矩阵-向量等价物所需的16次乘法和12次加法多出一倍以上。原因是这种朴素展开没有考虑到PGA电机满足$M\widetilde M = 1$的事实。然而，将这一点纳入我们的夹心乘积中并不困难。为此，我们建议从以下表达式开始：$$ p' = M p \widetilde M + p \cdot (1 - M\widetilde M)$$ 第二项对于归一化电机M为零。在系数级评估这个新表达式允许我们将所需的操作减少到21次乘法和18次加法（这对于同构双四元数是已知的最佳解决方案）：

```
// 21 mul, 18 add
point sw_mp( motor a, point b ) {
  direction t = cross(b, a[0].yzw)  - a[1].xyz;
  return  (a[0].x * t + cross(t, a[0].yzw) - a[0].yzw * a[1].w) * 2\. + b;
}
```

#### 变换方向

对于方向，即无穷远点，隐含的$\mathbf e_{123}$系数为$0$，我们可以更进一步。应用相同的归一化技巧，我们找到了一个只需要18次乘法和12次加法的解决方案。

```
// 18 mul, 12 add
direction sw_md( motor a, direction b ) {
  direction t = cross(b, a[0].yzw);
  return  (a[0].x * t + cross(t, a[0].yzw)) * 2\. + b;
}
```

在处理切空间时，我们也需要预先计算在基方向上的夹积。（与上述的一般方向相反）。在这种情况下，计算成本可以进一步降低。实际上，如果我们将输出归一化到 $0.5$ 而不是 $1$，我们可以将例如 x 轴的转换计算成本减少到令人惊讶的 6 次乘法和 4 次加法 - 大约是默认叉乘的成本的一半：

```
// 6 mul, 4 add
direction sw_mx( motor a ) {
  return direction(
    0.5 - a[0].w*a[0].w - a[0].z*a[0].z, 
    a[0].z*a[0].y - a[0].x*a[0].w, 
    a[0].w*a[0].y + a[0].x*a[0].z
  );
}
```

这是一个重要的观察，正如您将看到的，这将使我们有能力挑战普遍认为矩阵无条件是最快选择的共同信念……

#### 标准化

PGA（保持旋转运动群）电机 $M$ 的平方（伪）范数由以下给出：$$ \lvert M \rvert^2 = M \widetilde M = a + b\mathbf e_{0123}$$ 对于标准化电机，$\lvert M \rvert = 1$，但一般来说，这个表达式的结果是 $a + b\mathbf e_{0123}$，一个 Study Number（一个非标量部分平方为标量的多向量）。因此，标准化电机 $\overline M$，$$ \overline M = \cfrac {M} {\lvert M \rvert} $$ 计算起来稍微复杂一些。在 3D PGA 中，这涉及求逆一个 Study Number，这里同构于对偶数。我们已经为多个代数[7]解析出细节，在这些中，我们仅需 3D PGA 中的逆平方根公式：$$\cfrac {1} {\lvert M \rvert} = \cfrac {1} {\sqrt{M \widetilde M}} = \cfrac{1}{\sqrt{a + b\mathbf e_{0123}}} = \cfrac{1}{\sqrt{a}} - \cfrac{b}{2{\sqrt{a}}^3}\mathbf e_{0123} $$ 这导致以下 3DPGA 的高效实现：

```
// 21 mul, 5 add
motor normalize_m( motor a ) {
  float s = 1\. / length( a[0] );
  float d = (a[1].w * a[0].x - dot( a[1].xyz, a[0].yzw ))*s*s;
  return motor(a[0]*s, a[1]*s + vec4(a[0].yzw*(s*d),-a[0].x*s*d));
}
```

注意，这个过程应该与向量归一化进行比较，而不是与 Gram-Schmidt 正交化比较，因为结果电机保证是一个正交归一化的变换。与之前一样，在处理纯平移或旋转时，我们可以使用更高效的归一化过程版本。

#### 平方根

在PGA中，平方根发挥着重要作用，因为它是构建元素之间转换的关键。对于任意一对点/线/平面$a, b$，存在将$a$移动到$b$的刚性变换。这样的刚性变换总是具有电动机形式，并且该电动机总是由相同的简单表达式给出：$$ M = \sqrt{ \cfrac {b} {a} } $$ 结合事实，对于任何归一化的非零刀片$a$，其逆是$\pm a$，我们可以将其重写为 $$ \pm M^2 = ba $$ 或者说，任何两个点、两条线或两个平面的几何积$ba$产生一个代表从$a$到$b$的两倍变换的电动机。平方根进入其中，将这个结果减半，并确实找到将$a$移动到$b$的电动机。同样地，几何代数提供了一个通用适用的简洁方法：$$ \sqrt M = \overline{1 + M} $$ 这里的上划线表示我们先前模块中的Study标准化过程。因此，平方根的计算成本正好等于带有额外加法的标准化过程的成本。

```
// 21 mul, 6 add
motor sqrt_m( motor R ) {
  return normalize_m( motor( R[0].x + 1., R[0].yzw, R[1] ) ); 
}
```

#### 指数映射

为了完善我们的PGA工具箱，我们还从[7]中借鉴了对数和指数映射的高效实现。请记住，PGA电动机的对数是（一组）缩放线的和，类似地，任何缩放线都可以被指数化以构建围绕其旋转的操作。虽然通用4x4矩阵的指数映射在数值上非常昂贵，但对于我们PGA电动机流形，可以使用高效的封闭形式解决方案。

```
// 14 muls 5 add 1 div 1 acos 1 sqrt
line log_m( motor M ) { 
  if (M[0].x == 1.) return line( vec3(0.), vec3(M[1].xyz) );
  float a = 1./(1\. - M[0].x*M[0].x), b = acos(M[0].x) * sqrt(a), c = a*M[1].w*(1\. - M[0].x*b);
  return line( b*M[0].yzw, b*M[1].xyz + c*M[0].wzy);
}
```

```
// 17 muls 8 add 2 div 1 sqrt 1 cos 1 sin
motor exp_b( line B ) {
  float l = dot(B[0],B[0]);
  if (l==0.) return motor( vec4(1., 0., 0., 0.), vec4(B[1], 0.) );
  float a = sqrt(l), m = dot(B[0].xyz, B[1]), c = cos(a), s = sin(a)/a, t = m/l*(c-s);
  return motor( c, s*B[0], s*B[1] + t*B[0].zyx, m*s );
}
```

#### 逆元素

如果有一个地方，几何代数明显与我们标准的向量和矩阵代数方法不同，那就是（多）向量的逆元素的存在性。这些逆元素不仅存在于我们上下文中正在处理的归一化对象中，而且计算起来非常高效。

| 元素 $x$ | 逆元素 $x^{-1}$ |
| --- | --- |
| 平面 | $x^{-1} = x$ |
| 线 | $x^{-1} = -x$ |
| 点 | $x^{-1} = -x$ |
| 电动机 | $x^{-1} = \widetilde x$ |

其中$\widetilde x$，即还原操作，仅改变双向量和三向量系数的符号。偶尔还需要一个更一般的逆元素，即一个一般双向量$B$的逆元素。请回忆一下，一个双向量$B$仅在$B \wedge B = 0$，即所谓的Plucker条件下代表单一线。如果一个双向量$B$不满足该要求，则它不是刀片，即不是两个平面的交集或两个点的联合的结果。对于这样的元素，其逆元素稍微复杂一些。

要找到这个逆，我们从右侧乘以反向双矢量开始。$$ \cfrac {1} {B} = \cfrac {\widetilde B}{B \widetilde B} $$ 与之前一样，$\lvert B \rvert^2 = B \widetilde B = a + b \mathbf e_{0123}$的平方范数是一个斯图尔数，与双数同构。这使我们能够使用双数逆的定义。$$ \cfrac {1} {(a + b \mathbf e_{0123})} = \cfrac {1} {a} - \cfrac {b} {a^2} \mathbf e_{0123} $$ 将最后一个表达式与$\widetilde B$相乘产生了我们寻找的逆。

#### 电机因子分解

就像分解矩阵的过程可以非常有见地一样，PGA电机的分解也是如此。我们将添加两种特定的因子分解作为我们工具箱的最后工具。其中第一种称为*欧几里得因子分解*，它将电机分解为围绕原点的旋转后跟平移。$$ M = T_e R_e $$ 这种分解特别容易计算，因为欧几里得转子$ R_e $只是我们电机的欧几里得部分 - 前四个浮点数 - 同构于常规四元数。如果需要，平移$ T_e $可以计算为$ T_e = M \widetilde R_e $

感兴趣的第二个因子分解被称为*不变分解*。它将一个电机$ M $分解为一个可交换的平移和旋转，这总是可能的，并且在3D中通常称为Mozzi-Chasles定理。您可能已经听说过，每个刚体变换可以分解为围绕一条线的旋转，前后跟随着沿着同一条线的平移。$$ M = TR = RT $$ 在3D PGA中，不变分解也很容易计算，其中可交换的平移由$$ T = 1 + \cfrac {\langle M \rangle_4} {\langle M \rangle_2} $$给出，其中尖括号表示等级提取，而上面的一般双矢量逆转出现很方便。现在可以构建匹配的旋转$ R = M\widetilde T = \widetilde TM $。

特别是在将切线框架的变换与物体到世界电机的变换组合时，我们将特别使用欧几里得分解，因为这样的框架对平移是不变的，绕原点的旋转组合更为高效。

### 逃离矩阵

在计算机图形学中，矩阵的普遍存在意味着与现有内容交互将不可避免地涉及到矩阵。我们开始使用的Khronos glTF项目在整个过程中都使用矩阵，用于变换、绑定姿势（如皮肤绑定）等。我们致力于无矩阵环境的承诺意味着我们需要在加载时将这些矩阵转换为它们的PGA等效形式。

#### 将矩阵转换为电机。

将一个4x4正交矩阵转换为电机，我们愉快地利用四元数的同构性，并升级行业标准解决方案，以处理整个PGA流形。

```
export const fromMatrix3 = M => {
  // Shorthand.
  var [m00,m01,m02,m10,m11,m12,m20,m21,m22] = M;

  // Quick scale check 
  const scale = [hypot(m00,m01,m02),hypot(m10,m11,m12),hypot(m20,m21,m22)];
  if (abs(scale[0]-1)>0.0001 || abs(scale[1]-1)>0.0001 || abs(scale[2]-1)>0.0001) {
    const i = scale.map(s=>1/s);
    m00 *= i[0]; m01 *= i[0]; m02 *= i[0];
    m10 *= i[1]; m11 *= i[1]; m12 *= i[1];
    m20 *= i[2]; m21 *= i[2]; m22 *= i[2];
    if (abs(scale[0]/scale[1]-1)>0.0001 || abs(scale[1]/scale[2]-1)>0.0001) console.warn("non uniformly scaled matrix !", scale);
  }  

  // Return a pure rotation (in motor format)
  return normalize(   m00 + m11 + m22 > 0 ? [m00 + m11 + m22 + 1.0, m21 - m12, m02 - m20, m10 - m01, 0,0,0,0]:
                   m00 > m11 && m00 > m22 ? [m21 - m12, 1.0 + m00 - m11 - m22, m01 + m10, m02 + m20, 0,0,0,0]:
                                m11 > m22 ? [m02 - m20, m01 + m10, 1.0 + m11 - m00 - m22, m12 + m21, 0,0,0,0]:
                                            [m10 - m01, m02 + m20, m12 + m21, 1.0 + m22 - m00 - m11, 0,0,0,0]);
} 
```

```
export const fromMatrix = M => {
  // Shorthand.
  var [m00,m01,m02,m03,m10,m11,m12,m13,m20,m21,m22,m23,m30,m31,m32,m33] = M;

  // Return rotor as translation * rotation
  return gp_mm( [1,0,0,0,-0.5*m30,-0.5*m31,-0.5*m32,0], fromMatrix3([m00,m01,m02,m10,m11,m12,m20,m21,m22]) );
} 
```

这些转换在我们的导入中的所有矩阵上在加载时运行。

#### 处理均匀缩放。

与4x4矩阵相比，PGA马达不包含缩放，因为它显然不是刚体变换。然而，缩放，特别是均匀缩放，在场景图中通常用于缩放来自潜在不同来源和绝对大小不同的资源。虽然在接近400个随机glTF文件的测试中，不到$0.5$%的文件有任何缩放动画，但其中相当多的文件都应用了固定的均匀缩放。均匀缩放的优势在于它对旋转和平移都是不变的，因此每个节点只需要一个浮点数，其中每个元素的总缩放仅仅是其自身缩放与其父节点缩放的乘积。我们的实现以这种方式跟踪缩放，在加载时或顶点着色器的第一步中应用总缩放，并在更新动画时将父节点的缩放应用于平移。像这样整合均匀缩放的影响是绝对微小的，使我们能够覆盖几乎所有现有内容，而不放弃PGA马达的效率。

#### 处理非均匀缩放。

对于非均匀缩放，情况就比较棘手了——在使用非均匀缩放的场景中，最终无法避免回到4x4矩阵。非均匀缩放不对旋转不变，像我们在均匀情况下追踪这些缩放是令人厌烦的。然而，再次从我们的glTF文件样本中，我们只能发现非均匀缩放应用于叶节点上（考虑到非均匀缩放引起的问题，这并不意外）。对于这种情况，动画关键帧不受影响，我们只需在其余变换之前单独应用非均匀缩放。

## 前向渲染。

拥有我们完备的PGA工具箱，现在我们可以着手处理实际的渲染任务。在Khronos提供的参考实现指导下，让我们重新审视那些矩阵是唯一解决方案的地方。

前向渲染器的一般思想是转换所有网格几何体，并确定每个三角形覆盖的像素。这与射线追踪方法相对立，后者从像素开始，确定其击中的三角形。在典型的前向渲染设置中，将网格从对象空间规范化到屏幕位置通常由一组称为模型、视图和投影矩阵的矩阵处理。

### 模型 - 视图 - 投影。

我们在加载时将所有矩阵和变换转换为PGA马达的转换已经是对更新场景图层次结构所需计算量的实质性优化。对于复杂的设置，需要许多组合运算符，切换到马达的好处是显而易见的。

然而，虽然CPU关心生成更新的转换，GPU则负责将这些转换应用于组成网格的顶点、法线和切线，正如我们所见，所涉及的计算复杂性似乎使我们的马达处于劣势。

正如我们很快会发现的那样，情况更加微妙，在这一点上我们继续推进，用马达替换模型和视图矩阵，并使用上述定义的夹层积来转换传入的顶点属性。

```
vec3 worldPosition = sw_mp( toWorld, attrib_position );
vec3 worldNormal   = sw_md( toWorld, attrib_normal );      
vec4 worldTangent  = vec4(sw_md( toWorld, attrib_tangent.xyz ), attrib_tangent.w);
```

对于投影矩阵，情况则不同。典型的4x4投影矩阵只有5个非零项，即使没有PGA，直接写出结果表达式也更加高效。同样，我们在这里使用一个标准投影函数。

```
vec4 project( const float n, const float f, const float minfov, float aspect, vec3 inpos ){
  float cthf = cos(minfov/2.0) / sin(minfov/2.0);              // cotangent of half the minimal fov.
  float fa = 2.*f*n/(n-f), fb = (n+f)/(n-f);                   // all of these can be precomputed constants.
  vec2 fit = cthf * vec2(-1.0/aspect, 1.0);                    // fit vertical.
  return vec4( inpos.xy * fit, fa - fb*inpos.z, inpos.z );
}
```

配置好我们的基本转换后，让我们将注意力转向当今最常见的一种着色技术，即切线空间法线映射。

### 切线空间法线映射。

#### 顶点着色器

对于切线空间法线映射网格，顶点着色器需要转换位置、法线和切线向量。因此，似乎我们选择PGA意味着我们要承担三倍的更高转换成本。然而，法线、切线和副切线向量一起形成一个标准正交框架。在PGA中，任何标准正交框架都通过$k$-反射与基本标准框架相关联。当$k$为偶数时，这些只是我们之前遇到的仅旋转的马达（同构于四元数），而当$k$为奇数时，这种$k$-反射代表的是一个类似旋转的转换，随后是一个额外的反射。

这意味着我们可以从顶点描述中移除法线和切线向量，取而代之的是一个切线转子，它表示从基本框架到所需切线框架的旋转。事实上，这样一个切线转子$R$实际上对所有可能的切线框架进行了双重覆盖，即$R$和$-R$产生相同的变换。我们可以利用这种双重覆盖来消除偶数和奇数$k$-反射的歧义，只需确保$R$的标量系数的符号与经典的左右手标志匹配。请注意，在这样做时，我们依赖于IEEE754浮点数规范，即我们依赖于零的有符号表示。在顶点着色器中，我们可以明确地提取原始符号，使用：

```
float handedness = sign(1/tangentRotor.x)
```

综合所有这些，我们得出结论，我们可以将最常见的切线空间法线映射设置的顶点描述符从12个浮点数（3个位置、3个法线、4个切线、2个UV）减少到9个（3个位置、4个切线转子、2个UV）。这是一个相当大的节省，它在加载时实现，将加载的法线和切线向量转换为：

```
// Normalize, Orthogonalize
normal  = normalize( normal );
tangent = normalize( sub(tangent, mul(normal, dot(normal,tangent) ) ) );
// Calculate the bitangent.
let bitangent = normalize(cross(normal, tangent));
// Now setup the matrix explicitely.
let mat = [...tangent, ...bitangent, ...normal];
// Convert to motor and store.
let motor = fromMatrix3( mat );
// Use the double cover to encode the handedness.
// in GA language, this means we are using half of the double cover to distinguish even and odd versors.
if (Math.sign(motor[0])!=tangents[i*4+3]) motor = motor.map(x=>-x);
```

但好消息还有更多。请记住，使用4x4矩阵时，位置、法线和切线的转换包括3个矩阵向量乘积，总共48次乘法和36次加法。在PGA版本中，我们可以一次性转换整个切线框架，仅需16次乘法和12次加法的成本。然后，我们实际上可以使用以下方式仅需9次乘法和8次加法直接提取世界空间的法线和切线：

```
// 9 muls, 8 adds
void extractNormalTangent( motor a, out direction normal, out direction tangent ) {
  float yw = a[0].y * a[0].w;
  float xz = a[0].x * a[0].z;
  float zz = a[0].z * a[0].z;

  normal  = direction( yw - xz, a[0].z*a[0].w + a[0].y*a[0].x, 0.5 - zz - a[0].y*a[0].y );
  tangent = direction( 0.5 - zz - a[0].w*a[0].w, a[0].z*a[0].y - a[0].x*a[0].w, yw + xz );
}
```

再加上用于转换顶点位置所需的21次乘法和18次加法，再加上用于提取手性的一次乘法，我们得出了一个了不起的结论，即为了将带有完整切线框架的顶点转换到世界空间，我们的PGA方法仅需（16 + 9 + 21 + 1 = 47）次乘法和（12 + 8 + 18 = 38）次加法。这几乎与如果我们使用4x4矩阵和法线、切线向量而需48次乘法和36次加法相同。

**PGA马达可以像4x4矩阵一样快速地转换您的网格顶点!!!**

| 方法 | 每顶点浮点数 | 每次变换浮点数 | 乘法次数 | 加法次数 |
| --- | --- | --- | --- | --- |
| 矩阵 + 法线 + 切线 | 12 | 32 | 48 | 36 |
| 马达 + 切线旋子 | 9 **-25%** | 8 **-75%** | 47 | 38 |

现在顶点着色器中的生成代码块变成了

```
// Now transform our vertex using the motor from object to worldspace.
worldPosition = sw_mp(toWorld, attrib_position);

// Concatenate the world motor and the tangent frame.
motor tangentRotor = gp_rr( toWorld, motor(attrib_tangentRotor,vec4(0.)) );

// Next, extract world normal and tangent from the tangentFrame rotor.
extractNormalTangent(tangentRotor, worldNormal, worldTangent.xyz);
worldTangent.w = sign(1.0 / attrib_tangentRotor.x); // trick to disambiguate negative zero!
```

此时，片段着色器不需要进行任何更改，因此这是一个可以用于任何现有引擎的即插即用替换。

#### 片段着色器

如果我们希望能够加载现有内容，这就是我们不得不退回到TBN矩阵的时刻。其原因显而易见。在将高细节网格烘焙到低细节网格时，烘焙工具已经在每个三角形面上插值了顶点法线和切线向量。从这些（不再归一化或正交的）向量中，在每个片段处构建一个正交的TBN矩阵，并用于将高细节的世界空间法线转换为存储在纹理中的切线空间法线。

插值基向量的过程引入了一种典型的矩阵误差，不幸的是，这种误差确实被烘焙到了纹理中。这就是为什么我们决定确实从切线旋子中明确提取法线和切线向量的原因。

但是，在控制烘焙工具的情况下，我们还可以做得更好。在这些情况下，我们可以将切线旋子未经修改地传递到片段着色器，那里可以对其进行归一化并用于转换采样法线，而无需构建TBN矩阵。在这种情况下，我们可以节省更多，消除在顶点着色器中提取法线和切线的需求，减少一个更多的变量参数，并消除片段着色器中昂贵的正交化的需求。

### 马达蒙皮。

由于PGA电机同构于双四元数，我们的PGA方法显然是骨骼动画的一个候选。在将反向绑定矩阵转换为其电机等效物之后，我们的电机的皮肤代码遵循双四元数蒙皮的已知模式：

```
// Grab the 4 bone motors.
motor b1 = motors[int(attrib_joints.x)];
motor b2 = motors[int(attrib_joints.y)];
motor b3 = motors[int(attrib_joints.z)];
motor b4 = motors[int(attrib_joints.w)];

// Blend them together, always use short path.
motor r = attrib_weights.x * b1;
if (dot(r[0],b2[0])<=0.0) b2 = -b2;
r += attrib_weights.y * b2;
if (dot(r[0],b3[0])<=0.0) b3 = -b3;
r += attrib_weights.z * b3;
if (dot(r[0],b4[0])<=0.0) b4 = -b4;
r += attrib_weights.w * b4;

// Now renormalize and combine with object to world
toWorld = gp(toWorld, normalize_m(r)); 
```

请注意，就像对双四元数而言，我们确保混合的任何变换都遵循最短弧，并重新归一化结果变换。

### 动画混合

对于动画混合，直接在CPU上混合和重新归一化PGA电机使用了相同的技术。

<select id="anim1"><option>空闲</option> <option>空闲2</option> <option selected="selected">失败</option> <option>成功</option> <option>交谈</option> <option>行走</option></select>

<select id="anim2"><option>空闲</option> <option>空闲2</option> <option>失败</option> <option selected="selected">成功</option> <option>交谈</option> <option>行走</option></select>

## 结论

这个项目最初的目标是演示仅使用PGA实现前向渲染器的可能性。我们发现，不仅这是可能的，而且常见的理解认为这会增加更昂贵的转换费用，结果却更加微妙。所得的改进既出乎意料又非常引人注目，尤其是在内存占用方面，同一存储中额外33%的顶点显著提高。这项技术可以轻松部署到其他现有的3D引擎中，几乎不费一文在顶点着色器上，而且无需修改管道的其余部分。

## 参考文献

1.  几何代数与计算机图形学，Charles Gunn & Steven De Keninck。[https://dl.acm.org/doi/10.1145/3305366.3328099](https://dl.acm.org/doi/10.1145/3305366.3328099)

1.  n维刚体力学，Marc Ten Bosch。SIGGRAPH2020。[https://marctenbosch.com/ndphysics/NDrigidbody.pdf](https://marctenbosch.com/ndphysics/NDrigidbody.pdf)

1.  几何Clifford代数网络，David Ruhe & 合著者。[https://doi.org/10.48550/arXiv.2302.06594](https://doi.org/10.48550/arXiv.2302.06594)

1.  几何代数变换器，Johann Brehmer & 合著者。[https://arxiv.org/pdf/2305.18415.pdf](https://arxiv.org/pdf/2305.18415.pdf)

1.  [计算机科学中的基于平面的几何代数](https://bivector.net/PGA4CS.html)，Leo Dorst & Steven De Keninck。

1.  愿Fork和您同在，Leo Dorst & Steven De Keninck。[https://bivector.net/PGADYN.html](https://bivector.net/PGADYN.html)

1.  小于6D几何代数中的归一化、平方根以及指数和对数映射，Steven De Keninck & Martin Roelfs。[http://dx.doi.org/10.1002/mma.8639](http://dx.doi.org/10.1002/mma.8639)
