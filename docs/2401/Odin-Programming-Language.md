<!--yml

类别：未分类

日期：2024-05-27 14:27:35

-->

# 奥丁编程语言

> 来源：[https://odin-lang.org/](https://odin-lang.org/)

<select class="form-select mb-2" id="example-code-select"><option id="example-1-select" value="example-1-code">Hellope</option><option id="example-2-select" value="example-2-code">Array Programming</option><option id="example-3-select" value="example-3-code">SOA Types</option><option id="example-4-select" value="example-4-code">Context System</option><option id="example-5-select" value="example-5-code">Reflection</option></select>

```
package main

import "core:fmt"

main :: proc() {
	program := "+ + * 😃 - /"
	accumulator := 0

	for token in program {
		switch token {
		case '+': accumulator += 1
		case '-': accumulator -= 1
		case '*': accumulator *= 2
		case '/': accumulator /= 2
		case '😃': accumulator *= accumulator
		case: // Ignore everything else
		}
	}

	fmt.printf("The program \"%s\" calculates the value %d\n",
	           program, accumulator)
}
```

```
package main

import "core:fmt"

main :: proc() {
	{
		a := [3]f32{1, 2, 3}
		b := [3]f32{5, 6, 7}
		c := a * b
		d := a + b
		e := 1 +  (c - d) / 2
		fmt.printf("%.1f\n", e) // [0.5, 3.0, 6.5]
	}

	{
		a := [3]f32{1, 2, 3}
		b := swizzle(a, 2, 1, 0)
		assert(b == [3]f32{3, 2, 1})

		c := a.xx
		assert(c == [2]f32{1, 1})
		assert(c == 1)

		d := swizzle(a, 0, 0)
		assert(d == [2]f32{1, 1})
		assert(d == 1)
	}

	{
		Vector3 :: distinct [3]f32
		a := Vector3{1, 2, 3}
		b := Vector3{5, 6, 7}
		c := (a * b)/2 + 1
		d := c.x + c.y + c.z
		fmt.printf("%.1f\n", d) // 22.0

		cross :: proc(a, b: Vector3) -> Vector3 {
			i := a.yzx * b.zxy
			j := a.zxy * b.yzx
			return i - j
		}

		cross_explicit :: proc(a, b: Vector3) -> Vector3 {
			i := swizzle(a, 1, 2, 0) * swizzle(b, 2, 0, 1)
			j := swizzle(a, 2, 0, 1) * swizzle(b, 1, 2, 0)
			return i - j
		}

		blah :: proc(a: Vector3) -> f32 {
			return a.x + a.y + a.z
		}

		x := cross(a, b)
		fmt.println(x)
		fmt.println(blah(x))
	}
}
```

```
package main

import "core:fmt"

main :: proc() {
	{
		Vector3 :: struct {x, y, z: f32}

		N :: 2
		v_aos: [N]Vector3
		v_aos[0].x = 1
		v_aos[0].y = 4
		v_aos[0].z = 9

		fmt.println(len(v_aos))
		fmt.println(v_aos[0])
		fmt.println(v_aos[0].x)
		fmt.println(&v_aos[0].x)

		v_aos[1] = {0, 3, 4}
		v_aos[1].x = 2
		fmt.println(v_aos[1])
		fmt.println(v_aos)

		v_soa: #soa[N]Vector3

		v_soa[0].x = 1
		v_soa[0].y = 4
		v_soa[0].z = 9

		// Same syntax as AOS and treat as if it was an array
		fmt.println(len(v_soa))
		fmt.println(v_soa[0])
		fmt.println(v_soa[0].x)
		fmt.println(&v_soa[0].x)
		v_soa[1] = {0, 3, 4}
		v_soa[1].x = 2
		fmt.println(v_soa[1])

		// Can use SOA syntax if necessary
		v_soa.x[0] = 1
		v_soa.y[0] = 4
		v_soa.z[0] = 9
		fmt.println(v_soa.x[0])

		// Same pointer addresses with both syntaxes
		assert(&v_soa[0].x == &v_soa.x[0])

		// Same fmt printing
		fmt.println(v_aos)
		fmt.println(v_soa)
	}
	{
		// Works with arrays of length &lt= 4 which have the implicit fields xyzw/rgba
		Vector3 :: distinct [3]f32

		N :: 2
		v_aos: [N]Vector3
		v_aos[0].x = 1
		v_aos[0].y = 4
		v_aos[0].z = 9

		v_soa: #soa[N]Vector3

		v_soa[0].x = 1
		v_soa[0].y = 4
		v_soa[0].z = 9
	}
	{
		// SOA Slices
		// Vector3 :: struct {x, y, z: f32}
		Vector3 :: struct {x: i8, y: i16, z: f32}

		N :: 3
		v: #soa[N]Vector3
		v[0].x = 1
		v[0].y = 4
		v[0].z = 9

		s: #soa[]Vector3
		s = v[:]
		assert(len(s) == N)
		fmt.println(s)
		fmt.println(s[0].x)

		a := s[1:2]
		assert(len(a) == 1)
		fmt.println(a)

		d: #soa[dynamic]Vector3

		append_soa(&d, Vector3{1, 2, 3}, Vector3{4, 5, 9}, Vector3{-4, -4, 3})
		fmt.println(d)
		fmt.println(len(d))
		fmt.println(cap(d))
		fmt.println(d[:])
	}
	{ // soa_zip and soa_unzip
		fmt.println("\nsoa_zip and soa_unzip")

		x := []i32{1, 3, 9}
		y := []f32{2, 4, 16}
		z := []b32{true, false, true}

		// produce an #soa slice the normal slices passed
		s := soa_zip(a=x, b=y, c=z)

		// iterate over the #soa slice
		for v, i in s {
			fmt.println(v, i) // exactly the same as s[i]
			// NOTE: 'v' is NOT a temporary value but has a specialized addressing mode
			// which means that when accessing v.a etc, it does the correct transformation
			// internally:
			//         s[i].a === s.a[i]
			fmt.println(v.a, v.b, v.c)
		}

		// Recover the slices from the #soa slice
		a, b, c := soa_unzip(s)
		fmt.println(a, b, c)
	}
}
```

```
package main

import "core:mem"

main :: proc() {
	// In each scope, there is an implicit value named context. This
	// context variable is local to each scope and is implicitly passed
	// by pointer to any procedure call in that scope (if the procedure
	// has the Odin calling convention).

	// The main purpose of the implicit context system is for the ability
	// to intercept third-party code and libraries and modify their
	// functionality. One such case is modifying how a library allocates
	// something or logs something. In C, this was usually achieved with
	// the library defining macros which could be overridden so that the
	// user could define what they wanted. However, not many libraries
	// supported this in many languages by default which meant intercepting
	// third-party code to see what it does and to change how it does it is
	// not possible.

	c := context // copy the current scope's context

	context.user_index = 456
	{
		context.allocator = my_custom_allocator()
		context.user_index = 123
		what_a_fool_believes() // the `context` for this scope is implicitly passed to `what_a_fool_believes`
	}

	// `context` value is local to the scope it is in
	assert(context.user_index == 456)

	what_a_fool_believes :: proc() {
		c := context // this `context` is the same as the parent procedure that it was called from
		// From this example, context.user_index == 123
		// An context.allocator is assigned to the return value of `my_custom_allocator()`
		assert(context.user_index == 123)

		// The memory management procedure use the `context.allocator` by
		// default unless explicitly specified otherwise
		china_grove := new(int)
		free(china_grove)

		_ = c
	}

	my_custom_allocator :: mem.nil_allocator
	_ = c

	// By default, the context value has default values for its parameters which is
	// decided in the package runtime. What the defaults are are compiler specific.

	// To see what the implicit context value contains, please see the following
	// definition in package runtime.
} 
```

```
package main

import "core:fmt"
import "core:reflect"

main :: proc() {
	Foo :: struct {
		x: int    `tag1`,
		y: string `json:"y_field"`,
		z: bool, // no tag
	}

	id := typeid_of(Foo)
	names := reflect.struct_field_names(id)
	types := reflect.struct_field_types(id)
	tags  := reflect.struct_field_tags(id)

	assert(len(names) == len(types) && len(names) == len(tags))

	fmt.println("Foo :: struct {")
	for tag, i in tags {
		name, type := names[i], types[i]
		if tag != "" {
			fmt.printf("\t%s: %T `%s`,\n", name, type, tag)
		} else {
			fmt.printf("\t%s: %T,\n", name, type)
		}
	}
	fmt.println("}")

	for tag, i in tags {
		if val, ok := reflect.struct_tag_lookup(tag, "json"); ok {
			fmt.printf("json: %s -> %s\n", names[i], val)
		}
	}
}
```
