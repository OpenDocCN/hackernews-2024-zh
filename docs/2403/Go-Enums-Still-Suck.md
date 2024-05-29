<!--yml
category: 未分类
date: 2024-05-29 12:45:16
-->

# Go Enums Still Suck

> 来源：[https://www.zarl.dev/articles/enums-take-two](https://www.zarl.dev/articles/enums-take-two)

# Go Enums Still Suck

So my [original post](https://www.zarl.dev/articles/enums) got more eyeballs than expected and I saw it on social media mainly hacker news, and lobste.rs to my surprise. With varying levels of support for my ideas and some less than constructive comments such as and I paraphrase… ***‘learn to code’***, ***‘why do you hate Go?’***, ***‘THIS ISN’T JAVA’***. I love online discourse.

I will also say there was good discussion and support for what I was trying to achieve. Sometimes the Go userbase is less than receptive to critiques of the language ~~re:Generics Before Generics~~, but I still find Go my most productive language to just *get shit done*.

So some of the main critiques where:

### Use `stringer`

The first critique is well placed and I really should have started at that point, in fact I could have forked the project and used it as the basis for my own as it even uses the AST **(abstract syntax tree)** approach to parsing the GO code and building up from there. The tool is even mentioned on the official go blog post when `go generate` was introduced after all.

I didn’t do this; In hindsight I’m an idiot for not doing this from the start, but parsing a JSON file to a struct was just so simple and I admit I was lazy. At least it proved out my idea, so I rebuilt it correctly *(i hope)* this second time around.

I re-wrote my generator from scratch to use the [ast](https://pkg.go.dev/go/ast) package like the `stringer` command tool. However I built out the data required in a struct format that worked for me plus it was my first time really playing with the `ast` package so that was fun in and of itself. I also used the open source code for the `stringer` command so I could implement the same `String` method and also the compile-time check when new enums have been added.

### Unnecessary JSON Config

Yes - it was completely unnecessary! So, again, moving to the `ast` library means I can build up all information needed by parsing the Go code rather than rely on a configuration file.

### Extendable Enums

This was the most extensive of the critiques, and rightly pointed out; enums gain much more value when there is corresponding properties that relate to the enum. In the example below, I chose the planets example where there is a myriad of other properties we would want to have associated with the enum more than just uniqueness. I initially wanted to leverage Go’s struct tags, but guess what; the clue is in the name, they can not be used on anything but structs. So I just decided on a comment of a comma separated list of `Name[Type]` pairs defined on the type used in the iota definitions. For the planets example, it looks like this:

```
type planet int // Gravity[float64],RadiusKm[float64],MassKg[float64],OrbitKm[float64],OrbitDays[float64],SurfacePressureBars[float64],Moons[int],Rings[bool] 
```

This allows us to define a list of properties and their types that will be generated for the wrapper struct. We then just add the properties to the comments on the iota definitions. Again, for the planets example, it looks like this:

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

This uses the corresponding values to build out the wrapper so they end up like this:

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

This allows easily extending the wrapper with more properties. You should also note that the first property is not even defined in at the type definition. This is a unique case where this will be the string representation for the enum. So with the wrappers all generated. I decided to also generate the following `ExtensivePlanets` function to enumerate through all the valid values.

```
func ExhaustivePlanets(f func(Planet)) {
	for _, p := range Planets.All() {
		f(p)
	}
} 
```

We can now write something like the following :

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

This allows us to calculate our weight on all the different planets as they are exhaustively checked through all valid values. To mark invalid values, all you need to do is add a comment to the appropriate iota definition with invalid. This can be seen in the unknown value in the planets iota definitions.

## Conclusion

The feedback I got has been worth while and has allowed me to approach the problem from a different perspective and to also re-evaluate the choices I made on the first implementation.

[Here](https://github.com/zarldev/goenums) is the github repository for goenums