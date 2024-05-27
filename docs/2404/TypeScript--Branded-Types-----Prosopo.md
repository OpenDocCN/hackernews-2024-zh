<!--yml
category: 未分类
date: 2024-05-27 13:29:40
-->

# TypeScript: Branded Types 🔧 - Prosopo

> 来源：[https://prosopo.io/articles/typescript-branding/](https://prosopo.io/articles/typescript-branding/)

**[PART 1: TypeScript Mapped Type Magic](/articles/typescript-mapped-type-magic/)**

Ahoy there TypeScript warriors! 👋 Today we're extending our work in the [TypeScript mapped types article](/articles/typescript-mapped-type-magic/) to provide *branding*. The previous article discussed how to use TypeScript mapped types in a nominal rather than structural nature.

```
type A = {
    x: number
}

type B = {
    x: number
}
```

This is a fancy way of saying TypeScript is structural by default, i.e. it will see type `A` and `B` as equal when dealing with types. Making type `A` and `B` nominal would make TypeScript differentiate them apart, even though their structure is the same.

In this post, we're building on that work to produce a way to *brand* a type, providing an automated and easy-to-use way of making a type nominal. Branding focuses on the type system only, rather than introducing runtime fields like in the previous post, which is a major benefit over the previous approach.

## What's the problem?

Branding, also known as opaque types, enable differentiation of types in TypeScript which otherwise would be classified as the same type. For example

```
 type A = {
    x: number,
    y: boolean,
    z: string,
}

type B = {
    x: number,
    y: boolean,
    z: string,
} 
```

`A` and `B` are structurally the same, ergo TypeScript accepts any instance of `A` or `B` in place of each other:

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

The function is looking for a value of type `A` as input, whereas we're passing it a value of type `B`. TypeScript compares the types structurally, and because they have exactly the same structure it deems this operation to be fine.

But what if we need to tell `A` and `B` apart? What if, conceptually speaking, they must be different? What if we're doing something fancy with `A` and `B` which TypeScript is unaware of but we require the types to be different? That's exactly the situation we found ourselves in lately!

We need branding to do exactly that.

## The solution

Much like in the [TypeScript mapped types article](/articles/typescript-mapped-type-magic/), the key lies in creating a field with the name of a symbol to act as our *id*. However, with branding we only need this field at a type-level rather than the runtime-level. Since types are erased after compilation, we need to add this field to a type without altering the runtime data whatsoever. Casting, anyone?

First, lets introduce the `brand` field.

```
 const brand = Symbol('brand') 

type A = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'A'
}
```

Here we're adding the `brand` field to type `A`. The brand field name is a symbol, akin to a UUID. We use a symbol to ensure the `brand` field never clashes with any other field for `A`, because we'd be overwriting a field otherwise and introducing the worse kind of bugs: type bugs 🐛 . We've set the brand to `'A'` at the moment, though this could be anything you desire. It's akin to the type name. Now let's compare `A` and `B` again:

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

Here's the error:

```
Argument of type 'B' is not assignable to parameter of type 'A'.
  Property '[brand]' is missing in type 'B' but required in type '{ [brand]: "a"; }'.ts(2345) 
```

TypeScript won't let us pass an instance of `B` to the function accepting `A` because it's missing the `brand` field - brilliant! `A` and `B` are now different types. But what about if `B` had its own brand?

```
 type B = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'B'
}
```

Note that we're using the same `brand` variable from before. It's important to keep this constant, otherwise we're declaring fields with different names!

Now lets try the function again:

```
 const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

And here's the error

```
Argument of type 'B' is not assignable to parameter of type 'A'.
  Type 'B' is not assignable to type '{ [brand]: "A"; }'.
    Types of property '[brand]' are incompatible.
      Type '"B"' is not assignable to type '"A"'.ts(2345) 
```

There we go! The error is saying that though both types have a `brand` field, the value for the brand is different for the two types, i.e. `'A' != 'B'`!

Let's see what happens if the `brand` is the same:

```
 type A = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'foobar'
}

type B = {
    x: number,
    y: boolean,
    z: string,
} & {
    [brand]: 'foobar'
}

const fn = (a: A) => {
    console.log('do something with A')
}

const obj: B = {
    x: 1,
    y: true,
    z: 'hello'
}

fn(obj) 
```

No error! `A` and `B` are seen as interchangeable types because they're structurally the same, having the same fields *and* same brand value of `'foobar'`. Excellent!

## Make it generic!

Awesome, so that works. But it's a toy example, not fit for production. Let's create a `Brand` type which can brand any type you wish:

```
const brand = Symbol('brand') 

type Brand<T, U> = T & {
    [brand]: U
}
```

This type is very simple, it takes your type `T` and adds a `brand` field with `U` being the brand value. Here's how to use it:

```
 type A_Unbranded = {
    x: number,
    y: boolean,
    z: string,
}

type A = Brand<A_Unbranded, 'A'> 
```

So now we can brand any type. For completeness, here's the same kind of thing to *remove* the brand and go back to plain ol' TypeScript types:

```
type RemoveBrand<T> = T[Exclude<keyof T, typeof brand>]
```

And this will remove the `brand` field from any branded type. Also note that if the type is not branded, it will not be touched!

## Real world usage

Let's put this into practice. We've got a class which needs branding to identify its type when dealing with [mapped types](/articles/typescript-mapped-type-magic/).

For simplicity, lets boil the class down to a `Dog` class:

```
 class Dog {
    constructor(public name: string) {}
}

type DogBranded = Brand<Dog, 'Dog'>

const dog = new DogBranded('Spot') 
```

TypeScript won't let us construct a branded dog 😢 . We're going to need to do some casting using the constructor to brand the *constructor* rather than the *class* itself.

```
 type Ctor<T> = new (...args: any[]) => T

const addBrand = <T>(ctor: Ctor<T>, name: string) => {
    return ctor as Ctor<Brand<T, typeof name>>
}

const DogBranded = addBrand(Dog, 'Dog')

const dog = new DogBranded('Spot') 
```

The `addBrand` function takes a constructor of a class and casts it to a branded type. This essentially makes an alias for the `Dog` class which can be used in exactly the same way as the `Dog` class, e.g. calling `new` on it.

We can `export` the `DogBranded` type to allow the outer world to use our class whilst ensuring it's always branded:

```
export type DogExported = typeof DogBranded
```

Likewise, we can do the same for brand removal:

```
 const removeBrand = <T>(value: T) => {
    return value as RemoveBrand<T>
}
```

This simply removes the brand by casting the type to a type mapped without the brand field.

And there we go: a sure-fire way to brand and un-brand your types in TypeScript 😃

We've published this work as a library which you can access via [NPM](https://www.npmjs.com/package/@prosopo/ts-brand)!

At Prosopo, we're using TypeScript branding to fortify our types and do clever type mapping for our soon-to-be-released runtime type validator. Stay tuned for updates!

**[PART 1: TypeScript Mapped Type Magic](/articles/typescript-mapped-type-magic/)**

* * *