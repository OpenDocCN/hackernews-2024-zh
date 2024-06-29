<!--yml

类别：未分类

日期：2024-05-29 12:45:16

-->

# Go枚举仍然不好用

> 来源：[https://www.zarl.dev/articles/enums-take-two](https://www.zarl.dev/articles/enums-take-two)

# Go枚举仍然不好用

所以我的[原始帖子](https://www.zarl.dev/articles/enums)获得了比预期更多的关注，我在社交媒体上主要看到了黑客新闻和lobste.rs，令我惊讶。对于我的想法有不同程度的支持，也有一些不太建设性的评论，例如，我引用一下… ***'学会编程吧'***，***'为什么你讨厌Go?'***，***'这不是Java'***。我喜欢在线讨论。

我也会说，有些讨论和支持我尝试实现的东西。有时，Go用户群体对语言的批评反应不那么积极，尤其是在泛型之前泛型的问题上，但我仍然发现Go是我最有效率的语言，仅仅是为了*把事情做完*。

因此，一些主要的批评点包括：

### 使用 `stringer`

第一个批评是很合理的，事实上我真的应该从那一点开始，实际上我本可以分叉该项目并将其用作我自己的基础，因为它甚至使用AST（抽象语法树）方法来解析Go代码并从那里构建。这个工具甚至在`go generate`引入之后被提到在官方的go博客帖子中。

我没有这样做；事后看来我是个白痴，因为从一开始解析JSON文件到一个结构体只是那么简单，我承认我有点懒。至少它证明了我的想法，所以第二次重建它时，我希望这次做得正确（*我希望*）。

我重新从头开始重写了我的生成器，使用了[ast](https://pkg.go.dev/go/ast)包，就像`stringer`命令工具一样。但是，我构建了对我有用的结构化数据，并且这是我第一次真正玩AST包，所以本身就很有趣。我还使用了`stringer`命令的开源代码，因此我可以实现相同的`String`方法，以及在添加新枚举时的编译时检查。

### 不必要的JSON配置

是的 - 这完全是不必要的！因此，再次转向`ast`库意味着我可以通过解析Go代码来构建所需的所有信息，而不是依赖配置文件。

### 可扩展的枚举

这是批评中最广泛的一种，也是完全正确的；当枚举具有与枚举相关的相应属性时，它们的价值会更大。在下面的例子中，我选择了行星的例子，其中我们希望将许多其他属性与枚举关联，而不仅仅是唯一性。最初，我想利用Go的结构标签，但是猜猜看，关键在于名字，它们只能用在结构体上。所以我决定在iota定义中使用的类型上定义一个以逗号分隔的`Name[Type]`对的注释。对于行星的例子，它看起来像这样：

```
type planet int // Gravity[float64],RadiusKm[float64],MassKg[float64],OrbitKm[float64],OrbitDays[float64],SurfacePressureBars[float64],Moons[int],Rings[bool] 
```

这允许我们定义一个属性列表及其类型，将生成用于包装结构体的 iota 定义。然后我们只需将这些属性添加到注释中的 iota 定义中。再次以行星示例为例，看起来是这样的：

```
const (
	unknown planet = iota // invalid
	mercury               // Mercury 0.378,2439.7,3.3e23,57910000,88,0.0000000001,0,false
	venus                 // Venus 0.907,6051.8,4.87e24,108200000,225,92,0,false
	earth                 // Earth 1,6378.1,5.97e24,149600000,365,1,1,false
	mars                  // Mars 0.377,3389.5,6.42e23,227900000,687,0.01,2,false
	jupiter               // Jupiter 2.36,69911,1.90e27,778600000,4333,20,4,true
	saturn                // Saturn 0.916,58232,5.68e26,1433500000,10759,1,7,true
	uranus                // Uranus 0.889,25362,8.68e25,2872500000,30687,1.3,13,true
	neptune               // Neptune 1.12,24622,1.02e26,4495100000,60190,1.5,2,true
) 
```

这使用相应的值来构建包装器，所以它们最终看起来是这样的：

```
var Planets = planetContainer{
	MERCURY: Planet{
		planet:              mercury,
		Gravity:             0.378,
		RadiusKm:            2439.7,
		MassKg:              3.3e23,
		OrbitKm:             57910000,
		OrbitDays:           88,
		SurfacePressureBars: 0.0000000001,
		Moons:               0,
		Rings:               false,
	},
	VENUS: Planet{
		planet:              venus,
		Gravity:             0.907,
		RadiusKm:            6051.8,
		MassKg:              4.87e24,
		OrbitKm:             108200000,
		OrbitDays:           225,
		SurfacePressureBars: 92,
		Moons:               0,
		Rings:               false,
	},
	EARTH: Planet{
		planet:              earth,
		Gravity:             1,
		RadiusKm:            6378.1,
		MassKg:              5.97e24,
		OrbitKm:             149600000,
		OrbitDays:           365,
		SurfacePressureBars: 1,
		Moons:               1,
		Rings:               false,
	},
	MARS: Planet{
		planet:              mars,
		Gravity:             0.377,
		RadiusKm:            3389.5,
		MassKg:              6.42e23,
		OrbitKm:             227900000,
		OrbitDays:           687,
		SurfacePressureBars: 0.01,
		Moons:               2,
		Rings:               false,
	},
	JUPITER: Planet{
		planet:              jupiter,
		Gravity:             2.36,
		RadiusKm:            69911,
		MassKg:              1.90e27,
		OrbitKm:             778600000,
		OrbitDays:           4333,
		SurfacePressureBars: 20,
		Moons:               4,
		Rings:               true,
	},
	SATURN: Planet{
		planet:              saturn,
		Gravity:             0.916,
		RadiusKm:            58232,
		MassKg:              5.68e26,
		OrbitKm:             1433500000,
		OrbitDays:           10759,
		SurfacePressureBars: 1,
		Moons:               7,
		Rings:               true,
	},
	URANUS: Planet{
		planet:              uranus,
		Gravity:             0.889,
		RadiusKm:            25362,
		MassKg:              8.68e25,
		OrbitKm:             2872500000,
		OrbitDays:           30687,
		SurfacePressureBars: 1.3,
		Moons:               13,
		Rings:               true,
	},
	NEPTUNE: Planet{
		planet:              neptune,
		Gravity:             1.12,
		RadiusKm:            24622,
		MassKg:              1.02e26,
		OrbitKm:             4495100000,
		OrbitDays:           60190,
		SurfacePressureBars: 1.5,
		Moons:               2,
		Rings:               true,
	},
} 
```

这允许轻松地通过添加更多的属性来扩展包装器。您还应注意，第一个属性甚至未在类型定义中定义。这是一个特殊情况，它将是枚举的字符串表示。因此，所有包装器都生成后，我决定生成以下 `ExtensivePlanets` 函数来枚举所有有效值。

```
func ExhaustivePlanets(f func(Planet)) {
	for _, p := range Planets.All() {
		f(p)
	}
} 
```

现在我们可以写出以下内容：

```
package main

import (
	"fmt"

	"github.com/zarldev/goenums/examples/milkyway"
)

func main() {
	weightKg := 100.0
	milkyway.ExhaustivePlanets(func(p milkyway.Planet) {
		// calculate weight on each planet
		gravity := p.Gravity
		planetWeight := weightKg * gravity
		fmt.Printf("Weight on %s is %fKg with gravity %f\n", p, planetWeight, gravity)
	})
} 
```

这使我们能够在经过详尽检查的所有有效值中计算我们在所有不同行星上的重量。要标记无效值，您只需在行星 iota 定义的适当位置添加注释即可。这可以在行星 iota 定义的未知值中看到。

## 结论

我收到的反馈是值得的，让我能够从不同的角度来解决问题，并重新评估我在第一次实现中所做的选择。

[这里](https://github.com/zarldev/goenums)是 goenums 的 github 仓库。
