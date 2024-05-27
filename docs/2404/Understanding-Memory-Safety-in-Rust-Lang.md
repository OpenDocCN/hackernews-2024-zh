<!--yml
category: 未分类
date: 2024-05-27 13:04:30
-->

# Understanding Memory Safety in Rust Lang

> 来源：[https://coderoasis.com/rust-lang-memory-safety/](https://coderoasis.com/rust-lang-memory-safety/)

The last decade has been amazing for the Rust Programming Language to become the choice for people has the goal of writing fast, machine native software and web applications. The way that the language was made was for strong guarantees for memory safety. We are here to try to understand this better.

Languages like `C` runs fast and low level as can be. They usually lack the features of ensuring the memory of a program is allocated and disposed of properly when the program is used and terminated. Noted by the White House National Cybersecurity Director, warns that without taking proper care of programming standards, there can be [serious exploits and vulnerabilities which can cause some major real world consequences](https://www.tomshardware.com/software/security-software/white-house-urges-developers-to-avoid-c-and-c-use-memory-safe-programming-languages?ref=coderoasis.com). The languages like Go and Rust – which put memory safety first – are gaining more and more attention.

How does the Rust Language guarantee memory safety? Let's find out!

* * *

> *Do you like what you are reading? We do recommend this next article – *[**Implementing RSA from Scratch in JavaScript**](https://coderoasis.com/implementing-rsa-from-scratch-in-javascript/)* – to you to continue learning with us.*

 [Implementing RSA from Scratch in JavaScript

Please note that it is essential for me to emphasize that the code and techniques presented here are intended solely for educational purposes and should never be employed in real-world applications without careful consideration and expert guidance. At the same time, understanding the principles of RSA cryptography and exploring various](https://coderoasis.com/implementing-rsa-from-scratch-in-javascript/) 

* * *

## Memory Safety is Native

The most important thing to understand about the Rust Programming Language is that they do not require an extra library or analysis tool to check for issues. They can be optional though to double check – yet the memory safety features are built right into the programming language by default. They are mandatory and enforced standards before any of your code ever runs.

By default, in Rust, code that is not memory safe are treated as *compiler errors* and not *runtime errors*. This has major advantages to coding in Rust because code that is invalid will never compile. This means it will never make it to a production environment. With languages like `C` or `C++`, memory errors are far too often are only discovered at runtime.

You also need to remember that this does not mean that the Rust programming language or any software applications coded in it is completely bulletproof either. There can be rare issues – like [race conditions](https://doc.rust-lang.org/nomicon/races.html?ref=coderoasis.com) – that are still the developer's responsibility to make sure are not an issue with their code. Yet, at the same time, Rust does take a lot of common exploits and vulnerabilities out of the equation for us.

Programming languages like `C#`, `Python`, and `Java` require the developer to do memory management manually. With Rust, the developer is more able to focus on writing code and getting the job done more efficiently. This level of convenience also comes at a small cost – usually in speed or the need of a larger runtime. From my experience dabbling around with Rust, the binaries are usually always more compact and runs at native machine level speeds by default while remaining memory safe the whole time.

## Immutable by Default

One of the first things I learned about researching the the Rust programming language is that all variables are immutable by default. This means that they can't be reassigned or modified. If you wanted to change or modify a immutable variable or function, then you have to specifically declare it as mutable for it to be changed.

This may seem like something very small or not important to a lot of developers, but it has the amazing effect to forcing the developer to be highly conscious of what values need to be mutable in the software application that they are developing – this includes *when* they need them to be also. The results of the Rust application are making it way easier for the developer to tell when, where, and what you can change.

The concept of *immutable by default* is very important and different from the ideas of a constant. Let's take a immutable variable as an example. This variable can be computed then stored as an immutable at the runtime. This means that it can first be computed, then stored, and not changed. Now, let's take a constant. The constant must be computable at the time of compile – this is also before the program ever runs. There are many different kinds of values – such as one being of user input, as an example – that can't be stored as a constant this way at all.

The `C++` programming language assumes this is the opposite of the Rust programming language – where everything is highly mutable. By default, you must use the `const` keyword to declare a something as immutable. In theory, you could honestly adopt the `C++` coding style of using `const` by default. This will only ever cover the code that you write for your own software applications. While using Rust by default, the way you code in it ensures that all software applications are written in the programming language are immutable.

## The Cost of Memory Safety

There is also some costs to the memory safety that we need to go over also. The one that comes to the top of my head is the need to learn how to use the language itself.

I will admit that switching to a new programming language is never an easy thing for me. Rust, so far, has a massive learning curve for me to learn. It is taking me some time to grasp the memory management model to be able to produce usable code for my applications which I want to learn to make. The learning curve of the language has been a big discussion online for people starting out to the experts of the language in many different forums, communities, and websites.

The older languages like `C` and `C++` which has even larger and more experienced developer community argues in their favor over Rust. They also have something which Rust lacks in a few ways which is code examples, libraries, and complete application examples to work off of. For me, it's not that hard to understand why some people choose to use any of the `C` languages over Rust at times. They have more development tools and resources that exist around them.

Yet, at the same time, within the decade that Rust has been around, it has gained a lot of development tools, good documentation, and an amazing community to learn and discuss with. They make it super easy to get up to speed and take off development of good software applications. The amazing collection of third-party libraries – also known as Crates – is highly expansive and growing more and more everyday. Developers who are joining the Rust programming language could require some retraining and learning new tools, but you will barely ever lack the resources you need or a library for a given project.

## Buffer Overflow Example

```
#include 
#include 
int main (void){
    printf ("Hello, world!\n");
    printf ("Let's overflow the buffer!\r\n");
    uint32_t array[5] = {0, 0, 0, 0, 0,};
    for(int index = 0; index < 6; index++) {
        printf("Index %d: %d\r\n", index, array[index]);
    }
    return 0;
} 
```

Code Example

When we compiled this code, it compiled without errors and warnings. However. when we run the code, see the following output:

```
Hello, world!
Let's overflow the buffer!

Index 0: 0
Index 1: 0
Index 2: 0
Index 3: 0
Index 4: 0
Index 5: 1
```

Compiled Code Output

## What are Ownership, Borrowing, and References?

By default, every value in the Rust programming language has an *owner*. This means that any value – anywhere or any point in the code also – can only have full read/write control can have it at one time. The ownership of a value can be given away or better yet be *borrowed* from the original owner. Yet at the same time of a value being borrowed, this behavior is strictly tracked by the compiler. If any code tries to violate the ownership rules which are defined in the programming language than the compiler refuses to compile the code.

When we look to see how the `C` programming language handles this, we come to find out that there is no ownership with objects. This means that anything we want can be accessed by any other thing at literally anytime that it wants access to it. All of the responsibility of how anything is changed, modified, or used rests with how the programmer developes the software application.

In other programming languages such as `Python,` `C#`, and `Java`, the ownership rules just flat out do not exist at all. This is also due to that just don't need to exist either. This usually means that that they handle Object access – and also includes memory safety – is handled at the runtime. Like before, this comes at a cost – it can also be big or small depending on the software application – which is the speed or the size or even presence of a runtime.

## Ownership Code Examples

In the following example, the `s1` variable is now longer valid when we assign it to `s2`. This prevents from having double free errors, because `s2` goes out of scope. This allows the memory to be freed only one time. If we try to use `s1` after handing over ownership of the variable to `s2` then we will receive a compile time error.

```
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    // println!("{}, world!", s1); // This line would cause a compile-time error
    println!("{}, world!", s2);
}
```

Variable Ownership

The next example shows us how ownership is transferred via variables when they are passed unto functions. If the `type` implements the `Copy` trait – such as `integers`, `booleans`, and other types – the value is also copied. Then the original variable that was called can still be used also. Otherwise when the ownership is moved, the original variable can no longer be used.

```
fn main() {
    let s = String::from("hello"); // s becomes the owner of the memory holding "hello"

    takes_ownership(s); // s's value moves into the function...
                        // ... and so is no longer valid here

    // println!("{}", s); // This line would cause a compile error because the ownership of s's value has been transferred

    let x = 5;                  
    makes_copy(x); // x would move into the function,
                   // but i32 is Copy, so it's okay to still
                   // use x afterward

    println!("{}", x); // Works fine because i32 is Copy and doesn't move ownership
}

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens. 
```

Functions Ownership

## Some Takeaways

*   Ownership is an amazingly powerful feature of the Rust programming language that ensures memory safety and efficiency.
*   The language enforces a single owner for each piece of memory. Rust eliminates common errors like double free, memory leaks, and dangling pointers.
*   The key to writing an application in Rust is understanding and using ownership correctly from now on.

## Borrowing Code Example

The Rust programming language will allow you to reference a value without needing to take ownership of it. This is known as barrowing. There are two types of borrowing – they are immutable and mutable.

In the following example, `s` is used as a reference to `s1`, meaning it does not own the `String`. Since the `s` variable does not take ownership, the `s1` variable will remain valid after the call to `calculate_length`.

```
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
} 
```

The next example is mutable borrowing. This allows you to change something that is being borrowed. However, there is a big limitation. There can only be one mutable reference to a piece of data in a particular scope. This is to prevent people from data races at the time of compiling.

```
fn main() {
    let mut s = String::from("hello");

    change(&mut s);

    println!("{}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
} 
```

## What are Lifetimes?

Lifetimes are references to values in the Rust programming language that just don't have owners – but they do have Lifetimes. This means it's a scope of which a given reference is allowed to be valid. In most cases coding in Rust, lifetimes are left implicit, since the compiler can trace them easily. Yet, at the same time, lifetimes can also be explicitly used for more complex use cases.

You also need to remember that if you attempt to access or modify something outside of the lifetime that it was given – or better yet if it has gone out of scope – it will result in a compiler error. The goal of this is to prevent whole classes of bugs making it all the way to production environments or live software applications.

`Dangling pointers` can emerge when you try to access a variable or function that has either been deallocated or has gone out of scope. This also tends to happen in the `C` and `C++` programming languages more than Rust. The `C` programming language has no official or proper enforcement at the compile time for a lifetime of an object. The `C++` programming language has *smart pointers* concepts to help prevent this – they are not native or implemented or enforced by default – so you will have to opt-in to using them.

A lot of people leave the safety of the programming language up to the developers. What I mean is when it comes to memory safety practices in `C` or the `C++` programming language, it is usually a coding style that a developer has picked up on or even a institutional requirement – it's not something that the programming language outright requires you to do by default or is built into the language.

With programming languages such as `C#`, `Java`, or even `Python`, the memory management is the core responsibility of that languages runtime. This comes at a pretty big cost – in my personal opinion – of requiring a pretty big runtime. This can highly reduce the execution speeds of the software application. By default, the Rust programming language reduces the execution speed by enforcing the `lifetimes` rules before the code ever gets a chance to run.

## Lifetime Code Example

Lifetimes guarantees that a reference is only valid as long as it is required to be. The compiler uses lifetimes to make sure they do not outlive any of the data which is being referenced.

In the following example, using the `longest` function, we have a lifetime of `a` declared to specify the return value of the function. This will allow it to live as long as the shortest of the two input lifetimes.

```
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
} 
```

## Some Takeaways

*   Lifetimes make sure that the references do not outlive the data that they are referring to.
*   They are apart of the `Type` system that are used to perform checks at compile time without any cost at runtime.
*   Lifetimes in a function define how the lifetime of a difference reference can relate to each other.
*   The key to writing an application in Rust is understanding and using lifetimes correctly from now on.

## The Conclusion

Rust has changed the conversation about adding features or changes to existing programming languages that are lacking memory safety features by default or natively. They are transforming other languages enough to adopt these features.

These can be pretty ambitious ideas for the other languages that could be harder to implement. As an example if they were to add these features to `C` or `C++`, then we would have to talk about backwards compatibility. The behaviors that Rust brings by default to memory safety are harder to because it would force a hard divide between the existing legacy code and the new behaviors which developers will have to account for when switching over.

The main thing that gives the Rust programming Language its place in the programming world and the most notable feature that it offers over other languages is the memory safety and the compile time behaviors that guarantees it. Using these features – the memory safety the way Rust wants you to do it – will demand more work from the developer initially, but the trade offs always pay off later in the software application.

* * *

> Do you like what you're reading from the CoderOasis Technology Blog? We recommend reading our [***Implementing RSA in Python from Scratch***](https://coderoasis.com/implementing-rsa-from-scratch-in-python/) series next.

 [Implementing RSA in Python from Scratch

Please note that it is essential for me to emphasize that the code and techniques presented here are intended solely for educational purposes and should never be employed in real-world applications without careful consideration and expert guidance. At the same time, understanding the principles of RSA cryptography and exploring various](https://coderoasis.com/implementing-rsa-from-scratch-in-python/) 

* * *

> Did you know we have a [**Community Forums**](https://forums.coderoasis.com/?ref=coderoasis.com) and [**Discord Server**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)? which we invite everyone to join us? Want to discuss this article with other members of our community? Want to join a laid back place to chill and discuss topics like programming, cybersecurity, web development, and Linux? Consider joining us today!

 [Join the CoderOasis.com Discord Server!

CoderOasis offers technology news articles about programming, security, web development, Linux, systems admin, and more. | 112 members](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)  [CoderOasis Forums

CoderOasis Community Forums where our members can have a place to discuss technology together and share resources with each other.](https://forums.coderoasis.com/?ref=coderoasis.com)