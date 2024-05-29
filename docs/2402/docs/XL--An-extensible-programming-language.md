<!--yml
category: 未分类
date: 2024-05-27 15:03:43
-->

# XL: An extensible programming language

> 来源：[https://xlr.sourceforge.io/](https://xlr.sourceforge.io/)

The lifetime of a value is the amount of time during which the value exists in the program, in other words the time between its [creation](#creation) and its [destruction](#destruction).

An entity is said to be *live* if it was created but not yet destroyed. It is said to be *dead* otherwise.

|  | Some entities may be live but not accessible from within the current context because they are not visible. This is the case for variables declared in the caller’s context. |

The lifetime information known by the compiler about entity `X` is represented as compile-time constant `lifetime X`. The lifetime values are equipped with a partial order `<`, such that the expression `lifetime X < lifetime Y` being `true` is a compiler guarantee that `Y` will always be live while `X` is live. It is possible for neither `lifetime X < lifetime Y` nor `lifetime X > lifetime Y` to be true. This `lifetime` feature is used to implement [Rust-like](https://doc.rust-lang.org/1.8.0/book/ownership.html) [restrictions on access types](#lifetime), i.e. a way to achieve memory safety at zero runtime cost.

The lifetime of XL values fall in one of the following categories:

*   *global values* become live during the [declaration phase](#declaration-phase) of the program, just before its [evaluation phase](#evaluation-phase), and they remain live until the end of that evaluation phase. Global values are typically preallocated statically in a reserved area of memory, before any program evaluation, by a program called a linker.

*   *local values* become live during the [local declaration phase](#nested-declarations) of the bodies of the declarations corresponding to patterns [being matched](#pattern-matching) during the evaluation phase. Local values are typically allocated dynamically on a [call stack](https://en.wikipedia.org/wiki/Call_stack) allocated for each thread of execuion. That stack has a limited "depth", which may limit the depth of recursion allowed for a program.

*   *dynamic values* are dynamically allocated using a "heap", and remain live as long as some other value [owns them](#ownership). That owning value may itself be a global, local or dynamic value. The heap is typically the largest available memory space for the program. XL offers a number of facilities to help you manage how this dynamic allocation happens, including facilities to build [garbage collectors](#garbage-collection) if and when this is an efficient management strategy.

*   *temporary values* are created during evaluation of expressions, and can be discarded as soon as they have been consumed. For example, assuming a definition for `x+y`, the expression `a+b+c+d` will be processed as `((a+b)+c)+d`, and the result of evaluating `a+b` can be destroyed as soon as `(a+b)+c` has been evaluated. Temporary values are typically also allocated on the stack.

For example, consider the following piece of code:

```
use XL.CONSOLE.TEXT_IO ***(1)**
use XL.TEXT.FORMAT

print "Starting printing Fibonacci sequences" ***(2)**

fib 0 is 1 ***(3)**
fib 1 is 1
fib N is (fib(N-1) + fib(N-2)) ***(4)**

for I in 1..5 loop ***(5)**
    F is fib I ***(6)**
    print format("Fib(%1) is %2", I, F) ***(7)*********
```

 ******| ***1*** | The `use` statements import global values defined in other files. Here, the `XL` module, its sub-module `XL.CONSOLE`, and a third-level sub-module `XL.CONSOLE.TEXT_IO` are all imported by the first statement, and similarly, `XL.TEXT` and `XL.TEXT.FORMAT` are added by the second statement (`XL` being already imported) |
| ***2*** | The declaration of `print` is a global value [defined](#print) in `XL.CONSOLE.TEXT_IO`. This `print` statement is the first thing to be executed during program evaluated. Its evaluation will call code for an implementation of `print`, thereby adding a new context on the call stack. As we indicated [earlier](#print_instances), this may involve further calls making the call stack deeper. |
| ***3*** | The three definitions with `fib` as a prefix are three distinct global values, even if, thanks to dynamic dispatch, they may be considered as implementing a single entity. Evaluating `fib N` or `fib I` may require considering all three global values as candidates. |
| ***4*** | The evaluation of the expression `(fib(N-1)+fib(N-2))` will need to create a number of temporaries, for example to compute `N-1` or `N-2`. Temporaries may be [destroyed](#destruction) as soon as they are no longer needed. For example, the code to evaluate the expression could be someting similar to the following code, where `tmp1`, `tmp2`, `tmp3`, `tmp4` and `tmp5` are the required temporaries: 
```
tmp1 is N-1
tmp2 is fib(tmp1)
delete tmp1
tmp3 is N-2
tmp4 is fib(tmp3)
delete tmp3
tmp5 is tmp2+tmp4
delete tmp2
delete tmp4
tmp5
```

 |
| ***5*** | The `for` loop creates a local variable named `I` that will successively take values `1`, `2`, `3`, `4`, `5`. The value for `I` will only be live within one iteration of the loop. In other words, the execution will be identical to the following (using a [closure](#closure) for the different values of `I`): 
```
tmpBody is
    F is fib I
    print format("Fib(%1) is %2", I, F)
{ I is 1 } ( tmpBody )
{ I is 2 } ( tmpBody )
{ I is 3 } ( tmpBody )
{ I is 4 } ( tmpBody )
{ I is 5 } ( tmpBody )
```

 |
| ***6*** | The value defined by `F is fib I` is a local value that will be live for the duration of the evaluation of the enclosing block. It will be destroyed the end of each block. In other words, a more accurate description for `tmpBody` in the example above would have a `delete F` statement at the end, as follows: 
```
tmpBody is
    F is fib I
    print format("Fib(%1) is %2", I, F)
    delete F
```

 |
| ***7*** | It may come as a suprise to people coming from C or C++ that XL does not *require* the call to `fib I` to be done before this point. The definition `F is fib I` can be read as either a constant initialized with `fib I`, or as a function returning `fib I`. If the compiler can determine that the result of calling evaluating `F` will always be identical, it is allowed to implement memoization, i.e. to store the value computed for `F` the first time, for example in an expression like `F+F`. |

|  | Typically, a good compiler also makes use of machine *registers*, very fast storage in the processor itself, as a cache for values that are logically part of the call stack. In general, we will only talk about the stack, with the understanding that this includes registers where applicable. |******  *******Creation* is the process of preparing a value for use. The XL language rules guarantee that values are never undefined while the value is live, by calling programmer-supplied code at the appropriate times.

The lifetime of a value `V` begins by implicitly evaluating a *creation* statement `create V`. This happens in particular if you create a local variable without initializing it.

For example, consider the following code:

```
Add Z:complex is
   T:complex
   Z+T
```

The code above is really equivalent to the following, where the implicitly-generated code has been put between parentheses:

```
Add Z:complex is
   T:complex
   (create T)
   Z+T
```

Values in [containers](#generic-containers) receive well-defined values through creation. For example, if you create an `array[1..5] of complex`, the 5 `complex` values are created before you can access them.

A `create` operation must take a single `out` argument. All the values in this `out` argument are themselves created before the body of the definition begins. For example, consider the following:

```
create Z:out complex is
    print "Creator called"
```

This code does not lead to uninitialized values, because it is really equivalent to the following:

```
create Z:out complex is
   (create Z.Re)        // Implicit creation of complex fields
   (create Z.Im)
   print "Creator called"
```

The `create` operator for [basic types](#basic-types) is said to *zero initialize* them as follows:

*   `type` and `nil` values receive `nil`.

*   All integer types receive value `0`

*   All real types receive value `0.0`

*   All character types receive `character 0`.

*   `text` receive `""`.

*   `boolean` receive `false`.

The `create` operation can be called by the programmer, and therefore must behave correctly if it is called multiple times. This is true by default because of the rule that `out` parameters are [destroyed](#destruction) before a call.

For example, if you explicitly call `create` as in the following code:

this is really equivalent to the following, where implicit statements are between parentheses:

```
Z:complex
(create Z)      // because of the declaration above
(delete Z)      // because Z is passed as out argument to 'create'
create Z
```

The compiler may be able to elide some of these calls in such cases. Another important case where a compiler should elide creation calls is called *construction*, and is based on the shape defined for types. When you define a type, you need to specify the associate shape. For example, we defined a `complex` type as follows:

```
type complex is matching complex(Re:real, Im:real)
```

This means that a shape like `complex(2.3, 5.6)` is a `complex`. This also means that the *only* elementary way to build an arbitrary `complex` value is by creating such a shape. It is therefore not possible to have an uninitialized element in a `complex`, since for example `complex(X, Y)` would not match the shape unless both `X` and `Y` were valid `real` values.

However, there are contexts where it is desirable to *default initialize* a complex value, for example when creating a container, and this is where the `create` operation is necessary.

Using the shape explicitly given for the type is called the *constructor* for the type, and can be used in definitions or in variable declarations with an initial value. A constructor can never fail nor build a partial object. If an argument returns an [error](#errors) during evaluation, then that `error` value will not match the expected argument, except naturally if the constructor is written to accept `error` values.

Often, developers will offer alternate ways to create values of a given type. These alternate helpers are nothing else than regular definitions that return a value of the type.

For example, for the `complex` type, you may create an imaginary unit, `i`, but you need a constructor to define it. You can also recognize common expressions such as `2+3i` and turn them into constructors.

```
i   is complex(0.0, 1.0)

syntax { POSTFIX 190 i }
Re:real + Im:real i                 is complex(Re, Im)      // Case 1
Re:real + Im:real * [[i]]           is complex(Re, Im)      // Case 2
Re:real + [[i]] * Im:real           is complex(Re, Im)      // Case 3
Re:real as complex                  is complex(Re, 0.0)     // Case 4
X:complex + Y:complex as complex    is ...

2 + 3i              // Calls case 1 (with explicit concersions to real)
2 + 3 * i           // Calls case 2 (with explicit conversions to real)
2 + i * 3           // Calls case 3
2 + 3i + 5.2        // Calls case 4 to convert 5.2 to complex(5.2, 0.0)
2 + 3i + 5          // Error: Two implicit conversions (exercise: fix it)
```

The fact that the only elementary way to create a type is through the constructor is illustrated by the following code:

```
type large is matching (N when N > 42)
A:large := 44   // OK
B:large := 99.1 // OK
C:large         // Error
to create V:out large is
    print "Creator called"
```

The `A:large` and `B:large` initializations are acceptable, because it is possible to validate that the initial values match the `large` pattern. The `C:large` definition, however, is not acceptable, despite the presence of a `create` operation. The reason is that there is no way to `create V.N`, first because the type to use cannot be deduced, second because if we picked a type like `integer`, the default initial value `0` would not match the pattern.

The code above can be fixed, however, by using a constructor for the `large` type inside the creator, which means supplying a value that matches the type’s pattern. The following is an almost acceptable version of the `create` function:

```
type large is matching (N when N > 42)
C:large         // OK
to create V:out large is
    print "Creator called"
    V := 44     // Creator for a large value
```

|  | The above is only *almost* acceptable because it calls `print`, which may fail. Returning an `error` is not allowed by default by the signature of `create` for efficiency reasons. If this is desirable, then you must explicitly mark `create` as returning `mayfail`. |

A type implementation may be *hidden* in a [module interface](#modules), in which case the module interface should also provide some functions to create elements of the type. The following example illustrates this for a `file` interface based on Unix-style file descriptors:

```
module MY_FILE with
    type file
    to open(Name:text) as file
    to close F:io file

module MY_FILE is
    type file matches file(fd:integer)
    to open(Name:text) as file is
        fd:integer := libc.open(Name, libc.O_RDONLY)
        file(fd)
    to close F:inout file is
        if fd >= 0 then
            libc.close(F.fd)
            F.fd := -2
    to delete F:inout file is close F    // Destruction, see below
```

If the interface provides a `create` operation, it must be ready to accept default-created values as input in all other functions of the module. In the module above, however, the only way to get a value of the `file` type is by using the `open` function. This also means that you cannot create a variable of type `file` without initializing it.

|  | **RATIONALE** This mechanism is similar to *elaboration* in Ada or to *constructors* in C++. It makes it possible for programmers to provide strong guarantees about the internal state of values before they can be used. This is a fundamental brick of programming techniques such as encapsulation, programming contracts or [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization). |

When the lifetime of a value `V` terminates, the statement `delete V` automatically evaluates. Declared entites are destroyed in the reverse order of their declaration. A `delete X:T` definition is called a *destructor* for type `T`. It often has an [inout](#inout) parameter for the value to destroy, in order to be able to modify its argument, i.e. a destructor often has a signature like `delete X:inout T`.

Symmetrical to [creation](#field-creation), the body of a `delete V` automatically invokes `delete V.X` for any field `X` in `V` at exit of the body of the definition.

For example, consider the definition below:

```
to delete Z:inout complex is
    print "Deleting complex ", Z
```

That definition is actually equivalent to the definition below:

```
to delete Z:inout complex is
    print "Deleting complex ", Z
    (delete Z.Im)
    (delete Z.Re)
```

There is a built-in default definition of that statement that has no effect and matches any value, and which only deletes the fields:

```
to delete Anything is nil
```

There may be multiple destructors that match a given expression. When this happens, normal lookup rules happen. This means that, unlike languages like C++, a programmer can deliberately override the destruction of an object, and remains in control of the destruction process. More importantly, this means that the destruction process respects the global type semantics.

Consider for example the deletion of the `file` type defined in the `MY_FILE` module above. Since there is a special case for negative values, that might be reflected in the implementation as follows:

```
to delete F:inout file when F.fd < 0  is ... // Invalid flie
to delete F:inout file                is ... // Valid file
```

However, this also means that the programmer could create a `valid_file` type corresponding to the case where `F.fd<0` is false. If you have a `valid_file` value to `delete`, normal type system and lookup rules ensure that the second case will be selected.

Consider another interesting example, where you have the following declarations:

```
type positive is matching (N when N > 0)
integers is string of integer
N:integers > 0 as boolean is
    for I in N loop
        if not (I > 0) then
            return false
    return true

delete N:inout positive is
    print "Deleting positive: ", N

delete N:integer is
    print "Deleting integer: ", N

delete N:integers is
    print "Deleting integers with size: ", size N

example is
   print "Beginning example"
   A:integers := string(1,8,4)
   B:integers := string(-1,0,5)
   print "End of example"
```

In this example, we create an `integers` type based on `string of integers`, for which we implement the `N>0` operator to mean that all elements in the `string` are positive. This in turn means that some values that have type `integers` also have type `positive`. In the body of `example`, `A` is `positive`, but `B` is not.

If one consider [implicit inheritance](#implicit-inheritance) and the implicitly inserted field destruction, the code for the `integers` type and `delete` operations above is really equivalent to the following:

```
type integers is matching(base:string of integer)

delete P:inout positive is
    print "Deleting positive: ", P
    (delete P.N)        // Delete the N bound in 'positive'

delete N:integer is
    print "Deleting integer: ", N
    (delete N.base)     // For 'integer', this is a no-op

delete S:integers is
    print "Deleting integers with size: ", size S
    (delete S.base)     // Delete the underlying 'string of integer'
```

As a result, the output of this program should be something like:

```
Beginning example
End of example
Deleting integers with size 3 ***(1)**
Deleting positive: 5 ***(2)**
Deleting integer: 5 ***(3)**
Deleting integer: 0 ***(4)**
Deleting integer: -1
Deleting positive: string(1,8,4) ***(5)**
Deleting integers with size 3 ***(6)**
Deleting positive: 4
Deleting integer: 4
Deleting positive: 8
Deleting integer: 8
Deleting positive: 1
Deleting integer: 1******
```

 ******| ***1*** | This is deleting local variable `B` using type `integers`, knowing that it failed to pass the test for `positive` because of value `-1`. |
| ***2*** | This is deleting values in the `string of integer` container in local variable `B`, starting with the last one. Containers can destroy their values in any order, but for `string`, an efficient algorithm may start with the end of the container in order to be able to truncate before each element being removed simply by changing a "number of items" in the `string`. The local `delete` definitions are visible to the instantiation of `delete` for the type `string of integer` that is made for the call at the end of `example`. The first matching definition for value `5` is for the `positive` type. |
| ***3*** | This is implicitly deleting the `integer` value called `P.N` in the code above. |
| ***4*** | For value `0` in the `string of integer` value held in `B`, the `positive` test failed, so that the first destructor that works is for `integer`. |
| ***5*** | This is deleting local variable `A`. Since `A` is positive, the destructor for positive is called. |
| ***6*** | Unlike what happened for `B`, the destructor for `integers` is not called directly for `B` but implicitly for `P.N`. |

It is possible to create local destructor definitions. When such a local definition exists, it is possible for it to override a more general definition. The general definition can be accessed using link:#enclosing context[`super` lookup], and generally, it should in order to preserve the language semantics.

```
show_destructors is
    delete Something is
        print "Deleted", Something
        super.delete Something
    X is 42
    Y is 57.2
    X + Y
```

This should output something similar to the following:

```
Deleted 42.0
Deleted 57.2
Deleted 42
```

The first value being output is the temporary value created by the necessary implicit conversion of `X` from `integer` to `real`. Note that additional temporary values may appear depending on the optimizations performed by the compiler. The value returned by the function should not be destroyed, since it’s passed to the caller.

Any destruction code must be able to be called multiple times with the same value, if only because you cannot prevent a programmer from writing:

In that case, `Value` will be destroyed twice, once by the explicit `delete`, and a second time when `Value` goes out of scope. There is obviously no limit on the number of destructions that an object may go through.

```
for I in 1..LARGE_NUMBER loop
    delete Value
```

Also, remember that passing a value as an `out` argument implicitly destroys it. This is in particular the case for the target of an assignment.******  ******Errors in XL are represented by values with the `error` type, or any type that [inherits](#inheritance) from error. The error type has a constructor that takes a simple error message, or a simple message and a payload.:

```
type error is one_of
    error Message:text
    error Message:text, Payload
```

The message is typically a localizable format text taking elements in the payload as numbered argument in a way similar to the [`format` function](#text-format):

```
log X:real as error when X <= 0 is
    error "Logarithm of negative value %1", X
```

A function that may fail will often have a `T or error` return value. There is a specific shortcut for that, `mayfail T`:

```
mayfail T:type as type is T or error
```

For example, a logarithm returns an error for non-positive values, so that the signature of the `log` functions is:

```
log X:real as mayfail real     is ... // May return real or error
```

If possible, error detection should be pushed to the interface of the function. For the `log` function, it is known to fail only for negative or null values, so that a better interface would be:

```
log X:real as real  when X > 0.0    is ... // Always return a real
log X:real as error                 is ... // Always return an error
```

With the definitions above, the type of `log X` will be `real` if it is known that `X > 0.0`, `error` if it is known that the condition is false, and `real or error`, i.e. `mayfail real`, in the more general case. A benefit of writing code this way is that the compiler can more easily figure out that the following code is correct and does not require any kind of error handling:

```
if X > 0.0 then
    print format("Log(%1) is %2", X, log X)
```

|  | **RATIONALE** By returning an `error` for failure conditions, XL forces the programmer to deal with errors simply to satisfy the type system. They cannot simply be ignored like C return values or C++ exceptions can be. Errors that may possibly return from a function are a fundamental part of its type, and error handling is not optional. |

A number of types [derive](#inheritance) from the base `error` type to feature additional properties:

*   A `range_error` indicates that a given value is out of range. The default message provided is supplemented with information comparing the value with the expected range.

    ```
    T:text[I:offset] as character or range_error is
        if I >= length T then
            range_error "Text index %2 is out of bounds for text %2", I, T
        else
            P : memory_address[character] := memory_address(T.first)
            P += I
            *P
    ```

*   A `logic_error` indicates an unexpected condition in the program, and can be returned by contract checks like `assert`, `require` and `ensure`.

    ```
    if X > 0 then
        print "X is positive"
    else if X < 0 then
        print "X is negative"
    else
        logic_error "Some programmer forgot to consider this case"
    ```

*   A `storage_error` is returned whenever a [dynamic value](#dynamic_value) is created, notably each time an `own T` object is created, but also when additional storage is needed for containers.

    ```
    S : string of integer           // The string requires storage
    loop
        V : own integer := 3        // This allocates an integer, freed each loop
        S &= V                      // Accumulate integers in an unbounded way
    ```

*   A `file_error` reports when there is an error opening a file, for example because a file does not exist.

*   A `permission_error` reports when a resource access is denied, whether it’s a file or any other resource.

*   A `compile_error` helps the compiler emit better diagnostic for situations which would lead to an invalid program. All errors can be emitted at compile-time if the compiler can detect that they will occur unconditionally, but `compile_error` makes it clearer that this is intended to detect an error at compile-time. A variant, `compile_warning`, emits a message but lets the compilation proceed.

    ```
    // Emit a specific compile-time error if assigning text to an integer
    X:integer := Y:text is
        compile_error "Cannot assign text %1 to integer %2", Y, X

    // Emit a specific warning when writing a real into an integer
    X:integer := Y:real is
        compile_warning "Assigning real to integer may lose data"
        T is integer Y
        if real T = Y then
            X := T
        else
            range_error "Assigned real value %1 is out of range for integer", Y
    ```

A value is said to be *mutable* if it can change during its lifetime. A value that is not mutable is said to be *constant*. A mutable named entity is called a *variable*. An immutable named entity is called a *named constant*.

The `X:T` type annotations indicates that `X` is a mutable value of type `T`, unless type `T` is explicitly marked as constant. When `X` is a name, the annotation declares that `X` is a variable. The `X as T` type annotation indicates that `X` is a constant value of type `T`, unless type `T` is explicitly marked as variable. When `X` is a name, this may declare either a named constant or a function without parameters, depending on the shape of the body.

```
StartupMessage : text := "Hello World"  // Variable
Answer as integer is 42                 // Named constant
```

A mutable value can be initialized or modified using the `:=` operator, which is called an [*assignment*](##assignments-and-moves). There are a number of derived operators, such as `+=`, that combine a frequent arithmetic operation and an assignment.

```
X : integer := 42       // Initialize with value 42
X := X or 1             // Binary or, X is now 43
X -= 1                  // Subtract 1 from X, now 42
```

Some entities may give [access](#access-types) to individual inner values. For example, a `text` value is conceptually made of a number of individual `character` values that can be accessed individually. This is true irrespective of how `text` is represented. In addition, a `slice` of a `text` value is itself a `text` value. The mutability of a `text` value obviously has an effect on the mutability of accessed elements in the `text`.

The following example shows how `text` values can be mutated directly (1), using a computed assignment (2), by changing a slice (3) or by changing an individual element (4).

```
Greeting : text := "Hello"              // Variable text
Person as text is "John"                // Constant text
Greeting := Greeting & " " & Person     // (1) Greeting now "Hello John"
Greeting &= "!"                         // (2) Greeting now "Hello John!"
Greeting[0..4] := "Good m0rning"        // (3) Greeting now "Good m0rning John!"
Greeting[6] := 'o'                      // (4) Greeting now "Good morning John!"
```

None of these operations would be valid on a constant text such as `Person` in the code above. For example, `Person[3]:='a'` is invalid, since `Person` is a constant value.

|  | In the case (3) above, modifying a `text` value through an access type can change its length. This is possible because `Greeting[0..4]` is not an independent value, but an access type, specifically a `slice`, which keeps track of both the `text` (`Greeting` here) and the index range (`0..4` in that case), with a `:=` operator that modifies the accessed `text` value. |

A constant value does not change over its lifetime, but it may change over the lifetime of the program. More precisely, the lifetime of a constant is at most as long as the lifetime of the values it is computed from. For example, in the following code, the constant `K` has a different value for every interation of the loop, but the constant `L` has the same value for all iterations of `I`

```
for J in 1..5 loop
    for I in 1..5 loop
        K is 2*I + 1
        L is 2*J + 1
        print "I=", I, " K=", K, " L=", L
```

|  | **RATIONALE** There is no syntactic difference between a constant and a function without parameters. An implementation should be free to implement a constant as a function if this is more effective, or to use smarter strategies when appropriate. |

Some data types can be represented by a fixed number of contiguous memory locations. This is the case for example of `integer` or `real`: all `integer` values take the same number of bytes. Such data types are called *compact*.

On the other hand, a `text` value can be of any length, and may therefore require a variable number of bytes to represent values such as `"Hi"` and `"There once was a time where text was represented in languages such as Pascal by fixed-size character array with a byte representing the length. This meant that you could not process text that was longer than, say, 255 characters. More modern languages have lifted this restriction."`. These values are said to be *scattered*.

Scattered types are always built by *interpreting* compact types. For example, a representation for text could be made of two values, the memory address of the first character, and the size of the text. This is not the only possible representation, of course, but any representation require interpreting fixed-size memory locations and giving them a logical structure.

Although this is not always the case, the assignment for compact types generally does a [copy](#copy), while the assignment for scattered types typically does a [move](#move).

Computers offer a number of *resources*: memory, files, locks, network connexions, devices, sensors, actuators, and so on. A common problem with such resources is to control their *ownership*. In other words, who is responsible for a given resource at any given time.

In XL, like in languages like Rust or C++, ownership is largely determined by the type system, and relies heavily on the guarantees it provides, in particular with respect to [creation](#creation) and [destruction](#destruction). In C++, the mechanism is called [RAII](https://en.wikipedia.org/wiki/RAII), which stands for *Resource Acquisition is Initialization*. The central idea is that ownership of a resource is an invariant during the lifetime of a value. In other words, the value gets ownership of the resource during construction, and releases this ownership during destruction. This was illustrated in the `file` type of the module `MY_FILE` [given earlier](#my_file).

Types designed to own the associated value are called *owner types*. There is normally at most one live owner at any given time for each controlled resource, that acquired the resource at construction time, and will release it at destruction time. It may be possible to release the owned resource early using `delete Value`.

The [standard library](#standard-library) provides a number of types intended to own common classes of resources, including:

*   An `own` value owns a single item allocated in dynamic storage. Note that the value `nil` is not a valid `own` value (except for `own nil`). If you need `nil` as a value, you must use `own T or nil`.

*   An `array`, a `buffer` and a `string` all own a contiguous sequence of items of the same type.

    *   An `array` has a fixed size during its lifetime and allocates items directly, e.g. on the execution stack.

    *   A `buffer` has a fixed size during its lifetime, and allocates items dynamically, typically from a heap.

    *   A `string` has a variable size during its lifetime, and consequently may move items around in memory as a result of specific operations.

*   A `text` owns a variable number of `character` items, and inherits from the `string of character` type.

*   A `file` owns an open file.

*   A `mutex` owns execution by a single thread while it’s live.

*   A `timer` owns a resource that can be used to measure time and schedule execution.

*   A `thread` owns an execution thread and the associated call stack.

*   A `task` owns an operation to perform that can be dispatched to one of the available threads of execution.

*   A `process` owns an operating system process, including its threads and address space.

*   A `context` captures an execution context.

Not all types are intended to be owner types. Many types delegate ownership to another type. Such types are called *access types*. When an access type is destroyed, the resources that it accesses are *not* disposed of, since the access type does not own the value. A value of the acces type merely provides *access* to a particular value of the associated owner type.

For example, if `T` is a `text` value and if `A` and `B` are `integer` values, then `T[A..B]` is a particular kind of access value called a *slice*, which denotes the fragment of text between `0`-based positions `A` and `B`. By construction, slice `T[A..B]` can only access `T`, not any other `text` value. Similarly, it is easy to implement bound checks on `A` and `B` to make sure that no operation ever accesses any `character` value outside of `T`. As a result, this access value is perfectly safe to use.

Access types generalize *pointers* or *references* found in other languages, because they can describe a much wider class of access patterns. A pointer can only access a single element, whereas access types have no such restriction, as the `T[A..B]` example demonstrates. Access types can also enforce much stricter ownership rules than mere pointers.

|  | The C language worked around the limitation that pointers access a single element by abusing so-called "pointer arithmetic", in particular to implement arrays. In C, `A[I]` is merely a shortcut for `*(A+I)`. This means that `3[buffer]` is a valid way in C to access the third element of `buffer`, and that there are scenarios where `ptr[-1]` also makes sense as a way to access the element that precedes `ptr`. Unfortunately, this hack, which may have been cute when machines had 32K of memory, is now the root cause of a whole class of programming errors known as *buffer overflows*, which contribute in no small part to the well-deserved reputation of C as being a language that offers no memory safety whatsoever. |

The [standard library](#standard-library) provides a number of types intended to access common owner types, including:

*   A `ref` is a reference to a live `own` value.

*   A `slice` can be used to access range of items in contiguous sequences, including `array`, `buffer` or `string` (and therefore `text` considered as a `string of character`).

*   A `reader` or a `writer` can be used to access a `file` either for reading or writing.

*   A `lock` takes a `mutex` to prevent multiple threads from executing a given piece of code.

*   Several types such as `timing`, `dispatch`, `timeout` or `rendezvous` will combine `timer`, `thread`, `task` and `context` values.

*   The `in`, `out` and `inout` type expressions can sometimes be equivalent to an access types if that is the most efficient way to pass an argument around. However, this is mostly invisible to the programmer.

*   An `XL.SYSTEM.MEMORY.address` references a specific address in memory, and is the closest there is in XL to a raw C pointer. It is purposely verbose and cumbersome to use, so as to discourage its use when not absolutely necessary.

A type is said to *inherit* another type, called its *base type*, if it can use all its operations. The type is then said to *derive* from the base type. In XL, this is achieved simply by providing an *implicit conversion* between the derived type and the base type:

```
Derived:derived as base is ...
```

As a consequence of this approach, a type can derive from any number of other types, a feature sometimes called *multiple inheritance*. There is also no need for the base and derived type to share any specific data representation, although this is [often done in practice](#data-inheritance). For example, there is an implicit conversion from `i16` to `i32`, altough the machine representation is different, so in XL, one can say that `i16` derives from `i32`.

Sometimes, it is necessary to denote a type that inherits from a specific type. For example, if you want to create a constrained generic type for `complex`, you might want it to accept only `number` for its type argument. The following code is an incorrect way to do it, since it creates a type that only accept `number` *values* as an argument, i.e. it would accept `complex[3.5]` but not `complex[real]`:

```
type complex is matching complex[real:number] // WRONG
```

To denote a type that derives from a base type, one can use the `derived like base` notation. The correct way to implement the above restriction is as follows, which indicates that the argument is a type, not a value, and that the type must derive from `number`:

```
type complex is matching complex[real:type like number]
```

The precedence of `like` is lower than that of `:`, so that the above really parses as `(real:type) like number`. The `like` operator can be applied to specify inheritance at any place in a declaration, and notably in scopes. An alternate spelling for `like` is `inherits`, which is generally more readable in global scope. For example, you can indicate that the `complex` type itself inherits from `arithmetic` (providing arithmetic operations) as well as from `compact` (using a contiguous memory range) as follows:

```
type complex inherits arithmetic
type complex inherits compact
```************