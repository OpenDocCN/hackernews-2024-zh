<!--yml
category: 未分类
date: 2024-05-27 15:04:36
-->

# Loading...

> 来源：[https://bugs.openjdk.org/browse/JDK-8321133](https://bugs.openjdk.org/browse/JDK-8321133)

## Summary

Enhance the Java language with derived creation for records. Records are immutable objects, so developers frequently create new records from old records to model new data. Derived creation streamlines code by deriving a new record from an existing record, specifying only the components that are different. This is a [preview language feature](https://openjdk.org/jeps/12).

## Goals

*   Provide a concise means to create new record values derived from existing record values.

*   Streamline the declaration of record classes by eliminating the need to provide explicit *wither* methods, which are the immutable analogue of setter methods.

## Non-Goals

*   It is not a goal to provide a Pascal-style [<code class="prettyprint" >with</code>] construct that simplifies access to arbitrary complex expressions.

*   It is not a goal to provide support for a distinguished class of wither methods.

*   It is not a goal to provide derived creation expressions for ordinary, non-record values; this may be the subject of a future JEP.

## Motivation

Immutability is a powerful technique for creating safe, reliable code that is easy to reason about. Writing immutable classes in the Java language was traditionally a tedious exercise involving considerable boilerplate, but record classes ([JEP 395](https://openjdk.org/jeps/395)), introduced in Java 16, make it easy to declare (shallowly) immutable data-centric classes.

The immutability of record classes ensures both safety and predictability. It also enables a number of features that make them easy to use, including canonical constructors, accessor methods, and well-defined `Object` methods. But the systems that we need to model still have state, and so we need to model the natural evolution of this state. Unfortunately, it can be quite cumbersome to evolve state modeled by record classes. For example, consider a record class modeling a state which is a location in three-dimensional space:

```
record Point(int x, int y, int z) { }
```

Suppose we want to evolve the state by doubling the `x` coordinate of a `Point` `oldLoc`, resulting in `Point` `newLoc`:

```
Point newLoc = new Point(oldLoc.x() * 2, oldLoc.y(), oldLoc.z());
```

This code, while straightforward, is laborious. Deriving `newLoc` from `oldLoc` means extracting every component of `oldLoc`, whether it changes or not, and providing a value for every component of `newLoc`, even if unchanged from `oldLoc`.

It would be a constant tax on productivity if developers had to repeatedly deconstruct a record value, extracting all its components, in order to instantiate a new record value with mostly the same components. Accordingly, the authors of record classes often hide the deconstruction and instantiation inside such classes by declaring so-called *wither* methods:

```
record Point(int x, int y, int z) {
    Point withX(int newX) { return new Point(newX, y, z); }
    Point withY(int newY) { return new Point(x, newY, z); }
    Point withZ(int newZ) { return new Point(x, y, newZ); }
}
```

It is now possible to derive `Point` `newLoc` from `oldLoc` concisely, and to chain method calls in order to change more than one component:

```
Point newLoc = oldLoc.withX(oldLoc.x()*2);

// Double newLoc's y and z components
Point nextLoc = newLoc.withY(newLoc.y()*2)
                      .withZ(newLoc.z()*2);
```

However, wither methods have two problems:

*   They add boilerplate to the record class, which is unfortunate given that record classes aim to eliminate boilerplate such as JavaBean getter and setter methods.

*   Some record classes have semantic constraints that involve more than one component, enforced in the canonical constructor. This means that special care must be taken when writing wither methods. For example, if a record class has two `List` components that must have the same length, there must not be any wither methods that update only one `List` component at a time; we must carefully declare a wither method that calls the canonical constructor in such a way that both components are updated in one go. Put another way, wither methods are not always simple boilerplate — we need to take semantic constraints into account to determine exactly what kind of boilerplate is required.

A better way to derive new record values from old record values would be to let developers focus on transforming the components, and have the Java compiler handle the deconstruction and instantiation of record values automatically.

## Description

A *derived record creation expression* has the form

```
e with { ... }
```

For example, to create a new `Point` value whose `x` component is set to `0`, but whose `y` and `z` values are derived from an existing `Point` value `oldLoc`, a derived record creation expression can be used as follows:

```
Point nextLoc = oldLoc with {
    x = 0;
};
```

The expression to the left of the `with` keyword, namely `oldLoc`, is the initial record value. The block to the right of the `with` keyword is a transformation of the state of the initial record value. The state consists of three local variables, `x`, `y`, and `z`, which correspond to the components of the initial record value and are automatically initialized from those components; e.g., `x` has the value `oldLoc.x()`. Evaluating the derived record creation expression runs the code in the block to transform some or all of `x`, `y`, and `z`, and then creates a new record value by passing those variables to the canonical constructor, i.e., `new Point(x, y, z)`.

The state of the initial record value is represented by mutable local variables so that the state can be both queried and updated within the block, which can update some or all of the components. For example, to scale the point `nextLoc` by a factor of two:

```
Point finalLoc = nextLoc with { 
    x *= 2; 
    y *= 2;
    z *= 2;
};
```

The new state is validated by automatically invoking the canonical constructor at the end of the block. This ensures that any cross-component semantic constraints are respected by the code in the block. In the example of a record class with two `List` components of equal length, the canonical constructor will detect whether the block appended to one local `List` variable but not the other.

To try the examples in JDK 23 you must enable preview features:

### Using derived record creation expressions

Derived record creation expressions are an important part of the programming model for record classes. Using `with` to derive new record values is good for readability, since it focuses only on the changed state. It is also good for correctness, since it engages the canonical constructor automatically.

The block to the right of the `with` keyword need only mention the components that are transformed; thus the common situation of deriving a new record value by making only small changes to an existing record value is expressed succinctly.

If the block to the right of the `with` keyword is empty, then the result of the derived record creation expression is a fresh copy of the initial record value.

Derived record creation expressions can be chained. This allows a large transformation to be split into a series of smaller steps, aiding readability:

```
Point finalLoc = nextLoc
            with { x *= 2; }
            with { y *= 2; }
            with { z *= 2; };
```

Record values can be nested, with components which are themselves record values. Derived record creation expressions can likewise be nested in order to transform nested record values:

```
record Marker(Point loc, String label, Icon icon) { }

Marker m = new Marker(new Point(...), ..., ...);
Marker scaled = m with { loc = loc with { x *= 2; y *= 2; z *= 2; }};
```

Derived record creation expressions can also be used inside record classes to simplify the implementation of basic operations:

```
record Complex(double re, double im) {
    Complex conjugate() { return this with { im = -im; }; }
    Complex realOnly()  { return this with { im = 0; }; }
    Complex imOnly()    { return this with { re = 0; }; }
}
```

### Syntax and semantics

The grammar for a derived record creation expression is:

```
DerivedRecordCreationExpression:
  Expression with Block
```

The expression on the left-hand side is the *origin expression*. The type of the origin expression must be a record class type which is then taken to be the overall type of the derived record creation expression.

The block on the right-hand side is the *transformation block*. It is a normal block containing arbitrary block statements, but it has two control flow restrictions:

*   It may not contain a `return` statement, and
*   It may not contain a `yield`, `break`, or `continue` statement whose target contains the derived record creation expression.

In other words, control can be transferred out of a transformation block only by completing normally or by completing abruptly, throwing an exception.

For each record component, if any, in the header of the record class named by the type of the origin expression, a local variable with the same name and type is implicitly declared immediately before the content of the transformation block and initialized to the component's value. These local component variables can shadow any local variables whose declarations are in scope.

Any assignment statements that occur within the transformation block have the following constraint: If the left-hand side of the assignment is an unqualified name then that name must be either the name of a local component variable or the name of a local variable that is declared explicitly in the transformation block.

A derived record creation expression is not a statement expression; in other words, it cannot be used as a statement by following it with a semicolon:

```
finalLoc with { sideEffectingMethod(); }; // Error
```

### Evaluation, in detail

A derived record creation expression is evaluated as follows:

1.  Evaluate the origin expression, whose compile-time type names a record class *R*. If evaluation completes abruptly then evaluation of the derived record creation expression completes abruptly for the same reason.

2.  If the value of the origin expression is `null` then evaluation of the derived record creation expression completes abruptly with a `NullPointerException`.

3.  Before executing the content of the transformation block, execute a sequence of implicit local variable declaration statements derived from the record components in the header of record class *R*, in order, such that:

    *   The local variable declaration has the same name and declared type as the record component.

    *   The local variable declaration has an initializer which is given as if by invoking the corresponding component accessor method on the value of the origin expression.

    If evaluation of any of these local variable declaration statements completes abruptly then evaluation of the derived record creation expression completes abruptly for the same reason.

4.  Execute the content of the transformation block. If execution completes abruptly, evaluation of the derived record creation expression completes abruptly for the same reason.

5.  Create a new instance of record class *R* as if by evaluating a new class instance creation expression (`new`) with the compile-time type of the origin expression and an argument list containing the local component variables, if any, in the order that they appear in the header of record class *R*. (This ensures that the canonical constructor is invoked to create an instance of the record class.) The resulting instance of the record class *R* is taken as the overall value of the derived record creation expression. If evaluating the new class creation expression completes abruptly then evaluation of the derived record creation expression completes abruptly for the same reason.

### Analogy to pattern matching

A derived record creation expression such as

```
Point finalLoc = nextLoc with {
    x *= 2;
    y *= 2;
    z *= 2;
};
```

can be thought of as the `switch` expression

```
Point finalLoc = switch (nextLoc) {
    case Point(var x, var y, var z) -> {
        x *= 2;
        y *= 2;
        z *= 2;
        yield new Point(x, y, z);
    }
};
```

This makes it clear that a derived record creation expression essentially pattern matches the value of `nextLoc` against a record pattern — thereby deconstructing it and initializing the pattern variables `x`, `y`, and `z` — and then executes the statements in the transformation block, and then creates a new instance of `Point` by invoking the canonical constructor with the final values of the pattern variables as the arguments.

### Enforcing constraints

If the canonical constructor of a record class enforces constraints — as it should — then a derived record creation expression implicitly enforces them when constructing the new record value. For example:

```
record Rational(int num, int denom) {
    Rational {
        if (denom == 0)
            throw new IllegalArgumentException("denom must not be zero");
    }
}

Rational r = new Rational(3, 1);     // OK
Rational s = r with { denom = 0; };  // throws IllegalArgumentException
```

Evaluating this derived record creation expression initializes the local variables `num` to 3 and `denom` to 1, and then assigns `denom` to 0, and then invokes the canonical constructor via `new Rational(num, denom)`. This throws an exception just as if we had tried to evaluate `new Rational(3, 0)` explicitly.

(Transformation blocks in derived record creation expressions are similar to the body of a compact constructor in a record class in the sense that they both have the same control flow restrictions (must complete normally or throw an exception), and both have a set of implicitly declared variables in scope, which are expected to be assigned by the block.)

## Alternatives

*   Instead of supporting an expression form for use-site creation of new record values, we could support it at the declaration site with some form of special support for wither methods. We prefer the flexibility of use-site creation because declaring wither methods would add bloat to record class declarations, which currently enjoy a high degree of succinctness.

*   Instead of supporting transformation blocks, we could instead support only a list of variable declarators on the right-hand side of a derived record creation expression, e.g., `e with { x1 = e1; ..., xn = en; }`. This would serve to discourage impure code, but would likely be cumbersome and unnecessarily restrictive.

## Risks and Assumptions

*   There is a very small risk that the interaction of record components and local variables inside a transformation block can lead to surprising behavior. For example, if a transformation block refers to a local variable, and the relevant record class is then modified to have a component with the same name, then the local variable could be shadowed by the local variable implicitly declared in the transformation block. We assume this scenario is rare, and that compilers will warn about it as necessary.