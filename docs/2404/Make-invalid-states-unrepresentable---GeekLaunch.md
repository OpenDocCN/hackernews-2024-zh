<!--yml
category: 未分类
date: 2024-05-27 13:32:23
-->

# Make invalid states unrepresentable - GeekLaunch

> 来源：[https://geeklaunch.io/blog/make-invalid-states-unrepresentable/](https://geeklaunch.io/blog/make-invalid-states-unrepresentable/)

Let’s talk about types.

This post will be applicable to most programming languages, but it is significantly easier to apply these concepts when using languages that have more powerful, functionally-inspired, or mathematically-flavored type systems^.

As you probably know, the Rust compiler can spit out some pretty high-quality error messages.

```
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
  --> src\lib.rs:11:5
   |
10 |     let x = vec![];
   |         - help: consider changing this to be mutable: `mut x`
11 |     x.push(1);
   |     ^^^^^^^^^ cannot borrow as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `invalid-state` due to previous error
```

For example, it will recommend changing the mutability of an immutable binding when code attempts to mutate it.

```
error[E0063]: missing field `date_of_birth` in initializer of `Person`
  --> src\lib.rs:11:16
   |
11 |     let john = Person {
   |                ^^^^^^ missing `date_of_birth`

For more information about this error, try `rustc --explain E0063`.
error: could not compile `invalid-state` due to previous error
```

Here, it suggests a missing field’s name from a type constructor.

```
error: Ferris cannot be used as an identifier
 --> src\lib.rs:6:9
  |
6 |     let 🦀 = 0;
  |         ^^ help: try using their name instead: `ferris`

error: could not compile `invalid-state` due to previous error
```

And here, it suggests changing an invalid identifier (side note: this is possibly my favorite `rustc` error message).

The Rust compiler is *smart*, and in conjunction with the language’s powerful type system, you have a useful tool at your disposal.

## Legal vs. valid states

Types tell the compiler in what manner data can be represented. These might be types from the standard library, types from third-party libraries, primitives from the language, or even types that you write yourself.

Types delineate the set of *legally representable states* $\mathcal{R}$ ^(in your application.)

Then, there’s your business logic. Business logic leverages data that conform to the representable states defined by your types, manipulates those data, and delivers some output. The data that your business logic can handle comprises the set of valid states $\mathcal{V}$ (i.e. “handleable” states), and critically, *the set of valid states is not necessarily equal to the set of representable states*.

$$\mathcal{R} \supseteq \mathcal{V}$$

In fact, $\mathcal{R}$ is often *significantly* larger than $\mathcal{V}$, i.e. the code can handle far fewer states than are actually representable.

The difference between these two sets is the set of invalid states: the data which a program can represent but does not know how to handle properly. This is where bugs occur.

In order to reduce bugs, we therefore need to minimize this gap, and to do that, we can either:

1.  increase the number of cases handled by the code, or,
2.  decrease the number of representable states.

Increasing the number of cases handled by the code tends to increase its complexity, and complex code also has a high tendency to be buggy if you’re not careful. Therefore, in this post, we’re covering the latter strategy: decreasing the number of representable states.

Another way to think of this is that we’re moving as many errors as possible from runtime to compile-time.

## Example: color

Let’s take a look at a simple example:

```
fn accepts_color(color: &str) {
    // ...
}
```

Here, we have a function that accepts a color as input. Right now, it takes a string. Let’s see how this might pan out for our function:

```
accepts_color("#000000");
accepts_color("rgba(255, 255, 255, 0)");
accepts_color("purple");
accepts_color("sapphire");
accepts_color("5");
accepts_color("白");
accepts_color("Call me Ishmael.");
accepts_color("");
```

Right now, the set of representable states is greater than the set of valid states, meaning that our function will have to contain some parsing logic and probably some error handling. If the function propagates the error by returning a `Result` (Rust) or throwing an exception (Java, others), then the invoking code will need to perform some error handling as well.

Let’s instead introduce a simple `Color` data structure with two variants: for RGB and RGBA colors, and we’ll update our function to accept a parameter of this type instead of a string:

```
enum Color {
    Rgb(u8, u8, u8),
    Rgba(u8, u8, u8, u8),
}

fn accepts_color(color: Color) {
    // ...
}
```

Let’s take a look at the representable states now:

```
accepts_color(Color::Rgb(0, 0, 0));
accepts_color(Color::Rgba(255, 255, 255, 0));
```

Turns out, all of the representable states are also valid states! This means that our sets $\mathcal{R}$ and $\mathcal{V}$ are equal, and no runtime error handling is necessary.

Before I continue, let’s take a step back and evaluate how we can benefit from coding like this:

1.  First, it helps us to separate concerns within the codebase. Before you can use user input in your business logic, it has to be validated and parsed into an internal data structure, and this requirement is enforced by the compiler.
2.  Then, the guarantees which that parsing and validating step check for persist as long as the data exist. It provides deeper and earlier guarantees.
3.  If we restrict the possible inputs to a function, for example, this means that the logic in the body of the function has to cover fewer cases.
4.  If your types mirror your business logic, changes to those types will cause compile-time errors until your code is updated respectively. This is a good thing.
5.  Finally, well-structured data types are easier to understand and use. This is especially important in larger codebases with multiple contributors. Well-typed data is harder (though not impossible) to mistakenly use incorrectly.

## Example: modal text editor

Let’s take a look at a little more complicated example.

Imagine you’re writing a Vim-like text editor. In Vim, you can perform different text editing actions, as well as some other functions, like recording macros. A macro is a combination of actions that can be saved and executed multiple times.

In our example editor, we’re going to add the constraint that recorded macros are not allowed to save or record macros inside of them.

(Note that real Vim actually *does* support “recursive” macro operations, but this is a motivated example, so just roll with it.)

If we were to implement a type for actions, it might look something like this:

```
pub struct Motion(/* ... */);
pub struct Register(/* ... */);

pub struct SaveMacro {
    register: Register,
    actions: Vec<Action>,
}

pub enum Action {
    Move(Motion),
    Delete(Motion),
    Insert(String),
    SaveMacro(SaveMacro),
    RunMacro(Register),
}
```

Actions are represented by an enum. Saving a macro requires us to specify the register in which to save the macro, and the list of actions to save. The `Motion` and `Register` structs are irrelevant, so I’ve omitted any definition.

However, this set of types doesn’t enforce the non-recursive constraint. `SaveMacro::actions` isn’t *supposed* to be allowed to contain `Action::SaveMacro` or `Action::RunMacro`, but these type declarations allow for that.

No big deal, we can still enforce this, no problem. We’ll just make the fields of `SaveMacro` private and provide a constructor that enforces the constraint:

```
#[derive(Debug)]
pub enum SaveMacroError {
    IllegalSave,
    IllegalRun,
}

impl SaveMacro {
    pub fn new(register: Register, actions: Vec<Action>) -> Result<Self, SaveMacroError> {
        for action in actions.iter() {
            match action {
                Action::SaveMacro(..) => return Err(SaveMacroError::IllegalSave),
                Action::RunMacro(..) => return Err(SaveMacroError::IllegalRun),
                _ => {}
            }
        }

        Ok(Self { register, actions })
    }
}
```

It’s not too bad. We have a bit of code, a simple, self-explanatory error type, and our constructor works great.

So what’s the issue?

Well, whenever we want to use the constructor, the caller has to perform some error handling.

…which not the end of the world. Error handling is part of the job description.

```
let save_macro = SaveMacro::new(
    Register(/* ... */),
    vec![
        Action::Insert(String::from("code")),
        Action::Move(Motion(/* ... */)),
        // Action::RunMacro(Register(/* ... */)),
    ],
)
.unwrap();
```

Here I’ve used `unwrap`^(, which will halt the program if it encounters any errors.)

What happens if we uncomment the commented-out line?

At compile time, nothing! Unfortunately, this code produces a runtime error. Our validation code only runs at *run*time, so there’s no way for the compiler to tell us anything has gone wrong.

This is not necessarily a bad thing: runtime errors are normal. Sometimes they are unavoidable. However, in this case, we can do better.

## Shrinking the representable state space

Let’s take another look at our type declarations and do a bit of restructuring to shrink the set of representable states and make it closer to the set of valid states.

First, a rewrite of our `Action` and `SaveMacro` types.

```
pub enum Action {
    Edit(EditAction),
    SaveMacro(SaveMacro),
    RunMacro(Register),
}

pub enum EditAction {
    Move(Motion),
    Delete(Motion),
    Insert(String),
}

pub struct SaveMacro {
    pub register: Register,
    pub actions: Vec<EditAction>,
}
```

You’ll notice I’ve extracted the macro-legal actions out into their own enum.

Also note that in this version, there’s no constructor, no type for constructor errors, and the fields of `SaveMacro` can be public.

Here’s what it looks like to construct a `SaveMacro` action that does the same thing as the example for the previous version.

```
let save_macro = SaveMacro {
    register: Register(/* ... */),
    actions: vec![
        EditAction::Insert(String::from("code")),
        EditAction::Move(Motion(/* ... */)),
        // Action::RunMacro(Register(/* ... */)),
    ],
};
```

However, this time, if we uncomment the commented-out line, we get an error at compile-time!

```
error[E0308]: mismatched types
  --> src\compiletime_validation.rs:32:9
   |
32 |         Action::RunMacro(Register(/* ... */)),
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected enum `compiletime_validation::EditAction`, found enum `compiletime_validation::Action`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `invalid-state` due to previous error
```

Perfect! This is exactly what we wanted.

We defined our type system in a way that more closely mirrored our business logic, and in return, the Rust compiler was able to automatically validate more of our code at compile-time.

Compare this to the previous version, where we had to write the validation code ourselves, and even then we only got the error at runtime.

## State machines and state transitions

Let’s look at one more example by implementing a state machine. We will leverage the power of the type system to enforce valid state transitions at compile time.

We’ll use the simple example of a VPN, which has three states: `Disconnected`, `Connecting`, and `Connected`, as well as four state transitions:

*   `Disconnected` → `Connecting`
*   `Connecting` → `Connected` (on connection success)
*   `Connecting` → `Disconnected` (on connection failure)
*   `Connected` → `Disconnected`

Here’s one way to model the data structures:

```
struct Disconnected;
struct Connecting;
struct Connected;

struct Vpn<S> {
    state: S,
}
```

Note that instead of putting the different states in an enum, we are parameterizing the `Vpn` struct with other “`Vpn` state” structs. In the future, all of the state structs might implement a common trait, but this is good enough for now.

Since the state field is private in the struct, we can easily force the initial state of all `Vpn`s to be `Disconnected` by only implementing a `new` function on that type.

```
impl Vpn<Disconnected> {
    pub fn new() -> Self {
        Vpn {
            state: Disconnected,
        }
    }
}

let vpn: Vpn<Disconnected> = Vpn::new(); // ok
let vpn: Vpn<Connected> = Vpn::new(); // compile-time error
```

From here, it’s a simple matter to implement each state transition.

```
impl From<Vpn<Disconnected>> for Vpn<Connecting> {
    fn from(_value: Vpn<Disconnected>) -> Self {
        Vpn { state: Connecting }
    }
}

impl From<Vpn<Connected>> for Vpn<Disconnected> {
    fn from(_value: Vpn<Connected>) -> Self {
        Vpn { state: Disconnected }
    }
}
```

However, the `Connecting` state has two possibilities: it can transition to `Connected` on success, or `Disconnected` on failure.

We can model this with a `TryFrom` implementation.

```
impl TryFrom<Vpn<Connecting>> for Vpn<Connected> {
    type Error = Vpn<Disconnected>;

    fn try_from(_value: Vpn<Connecting>) -> Result<Self, Self::Error> {
        if can_connect() {
            Ok(Vpn { state: Connected })
        } else {
            Err(Vpn {
                state: Disconnected,
            })
        }
    }
}
```

* * *

I do not hope to convince you that your set of types is only good if it cannot represent invalid states, i.e. $\mathcal{R} = \mathcal{V}$. However, I do hope to demonstrate that putting a little more thought into the design of your data structures *could* help you to avoid *more* bugs *earlier* in development.

* * *

I would be remiss if I failed to mention the body of work preceding me on this topic:

* * *

I’m a software engineer for [NEAR Protocol](https://near.org/) and a graduate student at the [Tokyo Institute of Technology](https://www.titech.ac.jp/).

Connect with me on [Twitter](https://twitter.com/sudo_build) and [Mastodon](https://infosec.exchange/@hatchet).