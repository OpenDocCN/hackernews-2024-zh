<!--yml
category: 未分类
date: 2024-05-27 14:31:42
-->

# People Don’t Understand OOP – Sigma's Blog

> 来源：[https://blog.sigma-star.io/2024/01/people-dont-understand-oop/](https://blog.sigma-star.io/2024/01/people-dont-understand-oop/)

> “OOP to me means only messaging, local retention and protection and hiding of state-process, and extreme late-binding of all things.”
> 
> Alan Kay (the guy who coined the term “Object Oriented Programming”)

It seems like a lot of people dislike Object Oriented Programming. The first things that come to mind when hearing these three letters are cars, inheritance, getters, setters, and ObjectFactoryFactorySingleton.

This always seemed kinda odd to me. Not only do I like OOP, I even feel that it often is the best/most obvious way to model a problem. Here is why I think that is:

## OO Who?

I think before doing anything further we should probably define what we are talking about. Unfortunately OOP is not that well defined. So for the sake of coherence let’s settle on a clear and unambiguous definition first.

We will be talking about “objects” a lot. So what are they? Most introduction texts to OOP use physical things like cars and animals to illustrate what objects are. And while that’s not wrong (it is literally where the object metaphor comes from; Alan Kay was thinking in terms of biological cells and networks.) it’s certainly misleading, because objects are much more than that.

Peter Wegner writes: “Objects are collections of operations that share a state.”

Mark Stefik & Daniel Bobrow define objects as follows:
“Objects are entities that combine the properties of procedures and data since they perform computations and save local state. Uniform use of objects contrasts with the use of separate procedures and data in conventional programming.”

Here’s another definition by The Gang of Four: “Object-Oriented Programs are made up of objects. An object packages both data and procedures that can operate on data. Procedures are typically called methods or operations.”

Okay, that’s a good start, but I still think an important characteristic of objects is missing. Maybe Tim Rentsch can help: Objects are units of state, that are generally opaque to the outside (This, I think, is the important part. We will see later why that is.). An object can however provide the possibility to interact with its state by the means of message-passing (= “methods”).

Wait a second. “[…] collections of operations that share state”? “[…] entities that combine […] procedures and data […]”? “[…] units of state”?
The hell is that supposed to mean?
Well, it means that “object” is an abstract term. An object can potentially be anything – anything with state that is. It can be a physical item like car, an abstract concept, it can be any random piece of data with some sort of behavior attached to it. OOP just means we model our problem using these objects. That’s it.

### A Class Of Its Own

You might be thinking: “Hold on, we defined OOP without even touching on classes. What gives?”

The answer is simple: Classes are not strictly necessary for OOP. A shocker, I know.

Of course we need to be able to construct new objects, and class-based languages are admittedly way more prevalent. However, this is not the only way to achieve this goal.

Languages like JavaScript (although ES6 introduced classes to the language) or Lua use a concept called prototype-based or prototypal OOP. Instead of providing a schema for constructing new objects we use an existing object as a prototype. This approach can even have real-world benefits, as it reduces the language complexity.

(Just as a side note: Classes don’t need to be called classes. Languages like Go or Rust – also C++ to some extend – call them structs for example.)

### It’s Hereditary

Another term that – while not technically necessary – is often associated with OOP is inheritance.

There are two reasons to use inheritance:

The first is to reuse existing code. However, in modern programming this is usually discouraged in favor of object composition (an object inside another object).

The second reason (to me the more important one) is for abstraction and polymorphism. The technical term for this is “subtyping”.

### ‘Sup, Typing?

(Yes, I think this topic is important enough to warrant its own heading. 😅)

Subtyping is not exclusive to OOP, however it’s of special significance here since it’s the primary way of modelling polymorphism. The idea is to combine multiple different classes that share common messages (i.e. have methods with similar semantics) into a super type that defines those messages. Now, the super type can be used instead of specifying a subtype.

My favorite example for how subtyping can be used in practice is the Java collections framework. It defines interfaces (we will later talk about what exactly interfaces are) for common use-cases like Lists, Queues, Sets, Maps, … as well as different implementations with different characteristics which support those different use-cases.

wpg_div_collectionmap

(The graph was generated from JavaDocs by scrapping all known subclasses of [Collection](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/Collection.html) and [Map](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/Map.html) and removing non-relevant nodes.)

So let’s say I want to process a list of data, I’d just use the List interface everywhere. At the point where I instantiate the List I choose ArrayList, since it’s usually the more performant implementation. Later on it turns out that the program is doing a lot of inserts/deletes at the beginning of the list, which is pretty slow on arrays. To speed up the program I can switch to a LinkedList without changing any of the type signatures.

Side Note: When calling a method we need to know the actual class of the object, not just it’s declared class, otherwise subtyping won’t work properly. This is called “late/dynamic-binding”. It’s technical execution is a bit tricky and is the main reason why in C++ objects and object-pointers behave differently (see vtables).

### Odd Behavior

I think we can’t (and shouldn’t) talk about subtyping without mentioning behavioral subtyping and Barbara Liskov. The basic idea of behavioral subtyping is that a subtype should behave in a way similar to the parent type.

Barbara Liskov (who later won a Turing award for her work on programming languages and OOP in particular) formalized this concept in 1987 into “strong behavioral subtyping”: A subtype should be able to be used in every situation its parent type can be used in.

“Subtype Requirement: Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T.”

This is called the Liskov substitution principle. I won’t to go into details here, but the basic idea is that any precondition (for types, data or state) for parameters can not be stronger than the super-type, while any postcondition for results can not be weaker than the super-type. The notion is related to the design-by-contract methodology that also started to pop up around the same time.

### Way Too Abstract

In some cases we don’t care about the code-sharing aspect of inheritance, but still want to profit off of subtyping – we might never actually use the super type implementation of the methods and can therefore omit it entirely. This is in fact so common, it even has a name: Virtual or abstract methods.

We might even end up removing all state from our abstract super type, and only use it as a stencil for defining methods. This is called an interface.

Some languages even go a step further and completely decouple interfaces from classes. There are two different schools of thought:

Structural typing (as opposed to the usual nominal typing) is when interface implementations are not declared at all. You can simply use the object as an implementation as long as all necessary methods are defined. This is statically checked at compile time. Examples of languages that support structural typing are Go (both for interfaces themself and type constraints) and C++ (for concepts).
Duck typing is similar but the existence of methods is only checked at runtime. Languages that use this pattern include Python and JavaScript. One disadvantage that’s often cited is that it is more complicated to figure out which classes can be expected at a particular point in the program.

The second pattern doesn’t seem to have an established name yet. The idea is to declare that a class is implementing an interface after the class was already defined. An example of a language that does this is Rust with traits. Unfortunately, “traits” is a horrible name for this concept, since “traits” usually just refers to mix-ins. I’ve heard the term “extension traits” – in reference to “extension methods” in C#/Kotlin – but this doesn’t seem to be very common either. Another language that support this feature is Haskell (they call it “type classes”; but Haskell is arguably not object-oriented).

### Hide & Seek

One term that’s often used with OOP is “encapsulation”. There are actually two applicable definitions of the term. The first one refers to bundling data with behavior (= the object metaphor). And the second one refers to restricting access to the state only to the objects itself. I would like to focus a bit on the latter since I think a lot of people don’t understand it properly.

> “Encapsulation is a technique for minimizing interdependencies among separately-written modules by defining strict external interfaces.”
> 
> Alan Snyder, 1986

So why is it important to restrict access to state? Well, there are multiple reasons. We could argue that it would violate Liskov’s history constraint. But I think it’s much more practical to look at it from the perspective of a developer who wants to refactor the code base. Let’s say we want to change the internal structure of an object (like in the List example earlier, maybe we want to switch from an ArrayList to a LinkedList). But if other components are reliant on the internal state (in the case of the ArrayList: this could be the internal primitive array) we can not easily change it. We would need to find all places outside of the class where the internal structure is referenced. The problem gets even worse when the class is exported and used by modules that we might not even control.

“(Object) coupling” and “(class) cohesion” are often used to talk about encapsulation. “Object coupling” describes how much different objects depend on each other. High “object coupling” implies that the objects in question rely a lot on each other, which usually means they should be one single object instead. If objects rely on each others internal structure they are highly coupled. “Class cohesion” describes the same characteristic but from a different perspective. It’s a measure of how coherent a class’ responsibilities are. A class should ideally represent one idea and only do stuff related to that idea. Low “class cohesion” usually means high “object coupling” and vice versa.

I’m sure if you have done any object-oriented programming you’ve heard something like “Don’t use public properties […]” (properties in the sense of member variables) at some point. And this is true, because public properties expose the internal state, and can potentially cause high object coupling. However, as with any dogma, it’s usually a good idea to question it. In this case the complete “guideline” is “Don’t use public properties, use getters and setters instead.”, which is completely wrong. In terms of encapsulation getters and setters are just as bad as public properties, as they do nothing to prevent object coupling. If you have a class without any methods (besides getters and setters) it doesn’t really fit our object definition. A term that has been used for this is “record”.

### };

Okay, so what it OOP? OOP is when related state and behavior are bundled into units (= objects). Other properties object-oriented languages *may* have are: Classes, prototypes, encapsulation, subtyping, inheritance, …

Let’s take a look at some modern languages (these are the top 15 languages from the [StackOverflow Developer Survey 2023](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-programming-scripting-and-markup-languages) – excluding stuff like HTML, …):

| Language | Objs | Obj Creation | Encapsulation | Subtyping |
| --- | --- | --- | --- | --- |
| JavaScript | ✔️ | Classes/ Prototypes
 | ✔️ (since ES2022) | Inheritance/ Duck typing |
| Python | ✔️ | Classes | ❌ (not on a language level) | Inheritance/ Duck typing |
| TypeScript | ✔️ | Classes/ Prototypes | ✔️ | Inheritance/ Structural typing/
Duck typing |
| ShellScript | ❌ | ❌ | ❌ | ❌ |
| Java | ✔️ | Classes | ✔️ | Inheritance/ Nominal typing |
| C# | ✔️ | Classes | ✔️ | Inheritance/ Nominal typing |
| C++ | ✔️ | Classes + Structs | ✔️ | Inheritance/ Nominal typing +
Structural typing (concepts) |
| C | ❌ (no methods) | Structs | ✔️ (kinda using incomplete types) | ❌ (single “inheritance” by nesting structs; no real subtyping) |
| PHP | ✔️ | Classes | ✔️ | Inheritance/ Duck typing |
| PowerShell | ✔️ | Classes | ❌ | Inheritance/ Duck typing* |
| Go | ✔️ | Structs | ✔️ (on package level) | Structural typing |
| Rust | ✔️ | Structs | ✔️ | Extension Traits/ Nominal typing |
| Kotlin | ✔️ | Classes | ✔️ | Inheritance/ Nominal typing |
| Ruby | ✔️ | Classes | ✔️ (enforced) | Inheritance/ Duck typing |
| Lua | ✔️ | Tables (Prototypes) | ❌ | Inheritance/ Duck typing |

*) not sure

## O-Oh, No…

Okay. Now that we have a good understanding of what exactly OOP is and what we can expect from languages that implement the OOP paradigm, let’s take a look at some common points of criticism. (I shamelessly crowdsourced most of the following part by asking my friends what they hate about OOP. 😋)

### But What ARE Objects?

So, objects can be anything, right? So how do I know what should be an object? When should I combine things, what should be separate?

Well, in the end it’s just practice and experience. With time you’ll get a feeling about what should and what shouldn’t be an object. However, to get started there are some tricks that might help you.
Here’s what The Gang of Four has to say:

“Object-oriented design methodologies favor many different approaches. You can write a problem statement, single out the nouns and verbs, and create corresponding classes and operations. Or you can focus on the collaborations and responsibilities in your system. Or you can model the real world and translate the objects found during analysis into design. There will always be disagreement on which approach is best.”

### Speedy Thing Goes In, Speedy Thing Comes Out

EDIT: It has been pointed out to me that a made a few mistakes when designing the benchmarks. Thanks to NoNaeAbC on Github for pointing out that I’m allocating and zeroing way too much memory in the OOP and SP tests, and u9vata on Youtube for critiquing my benchmark design. With regard to the latter, while I don’t agree with everything they said, it’s definitely true that I was making some unfounded assumptions about compiler optimisation.
I’m not sure when I’ll have the time to redesign the benchmarks, so for the time being, please take the following with a big grain of salt.
While we are at it, I also came up with another explanation on why the FP tests are so slow: The band is stored as increasingly nested closures, which have to store their arguments on the heap, while the OOP and SP version can exclusively work on the stack.

OOP is slow. Or so I’ve heard. The rationale is that vtable lookups are overhead compared to direct function calls. I don’t actually know whether that’s true, so I decided to test it.

The test setup is as follows: I wrote the same program (a Turing machine checking for binary palindromes) 3 times, once using object-oriented programming, once using structured programming (only using functions, loops, tuples, arrays – stuff like that), and once using functional programming for good measure.

I implemented everything in C++ so it’s an equal playing field (also, C++ has first class functions/lambda expressions for the functional version). There is 100 000 test cases, the total time is measured. The compiler is clang 14.0.3, the target platform is Apple Silicon (M1). I ran each test with both -O0 and -O3.

For the OOP implementation I made sure to not rely on heap allocations, since the context switches would probably completely ruin the runtime. I did however use inheritance (the template pattern to be specific) to make the vtable lookups as realistic as possible.

The structured version also allocates everything on the stack. I build two different versions. The first one uses tuples in the transition table lookup, however I wasn’t sure how tuples are implemented under the hood and I wanted to avoid using objects by accident if possible, so I wrote another version that only relies on functions. But it turned out the results were so close I couldn’t tell the difference.

<canvas id="myiChartbenchmarks--o0-1"></canvas>

<canvas id="myiChartbenchmarks--o3-2"></canvas>

As we can see, the structured version is marginally (~ 5 %) faster than the object-oriented one when not using any optimization (Although I should mention that I observed the values jumping quite a lot between runs). When using -O3 the performance is basically identical (~ 1 %), so my guess is that the C++ optimizer was able to get rid of whatever impacted the performance.

The functional implementation is not even remotely close. To a certain extend this is probably caused by the benchmark I chose. Turing machines are inherently stateful, which is pretty awkward to model in a functional way. Another aspect is that even though I used C++14 (which supports return type inference) I was forced to use the std::function template as a wrapper for lambda expressions (anonymous types are a pain in the backside) which (according to my tests) are quite a bit slower than native lambda expressions.

I should have probably done some rigorous statistical tests, or at least calculated the deviation. But honestly, I was too lazy. I may write an update with a proper analysis later on.

In case you want to do some tests on your own, feel free to send me the results afterwards. 😛 The source code is on [Github](https://github.com/overflowerror/oop-benchmarks/tree/blog-version) (also I should probably apologise for the horrible code, C++ is not my native language and I hacked it together in like an hour or so 😅).

Anyway, without rigorous statistics my conclusion of the tests is that there is only a very small difference in performance. Adding more abstraction layers (or using different data structures) probably has a more significant impact.

However, other benchmarks on embedded systems have found a ~10 % performance penalty compared to a procedural implementation.

Another paper comparing the performance of different aspects of OOP as well as different design pattern has shown that virtual functions (which I used in my implementation) can impact the performance negatively (~ 5 %). The template pattern (which I also used) can also decrease the performance by about 3 – 4 % (but this might also just be because it relies on virtual functions).

### Abstract Nonsense

For some reason OOP leads us to overcomplicate everything. We needlessly build abstractions on top of abstractions, seemingly for the sole purpose of making pretty UML diagrams.

The thing is: It’s caused by how we use the tools, not the tools themselves. My suspicion is that most of these issues arise from developers wanting to be clever and build generic solutions to cover every possible future development.

I think a lot of this can be avoided by adjusting the workflow. Specifically: If the end goal is not determined from the beginning, don’t plan for every eventuality from the start, only plan for what you know you’ll need. The requirements might change later, so your amazing, highly generic solution that you worked on for 4 weeks straight might not be used in the end – a waste of time.

### The Threat of Get and Set

OOP is so verbose, there is so much boilerplate code. Getters and setters for example.

*sigh* This is a personal pet peeve of mine. We touched on this earlier but I would really like to hammer this part home: If you really need getters and setters for ever single member variable it’s probably not a proper object to start with. I’d highly encourage reconsidering your object model, try to reduce coupling. If it’s really a record class with no internal behavior, everything might as well be public – there is hardly a point in using getters and setters. A similar thing (though admittedly it’s a better) applies to properties in languages like C#, and of course code generators like the infamous Lombok.

The only real reason for the use of getters and setters over public members, is when there is some additional logic like validation of invariants for example.

Kinda related: If you have a value object with no setters but lots of getters, make sure to not accidentally expose a modifiable reference to internal state. Otherwise you’ve got setters – just not intentionally.

### ObjectFactoryFactorySingleton

I guess there are two topics that fit this heading. The first being the naming madness that has been established in enterprise software development. This is again not per se an issue with OOP although for some reason this seems to happen a lot more with OOP. I happen to be a Kevlin Henney fan, and he gave an [amazing talk on naming in programming](https://www.youtube.com/watch?v=CzJ94TMPcD8) at DevWeek 2015\. Among other things he talks about how naming can influence modelling. I highly recommend watching it.

The second topic is the rabbit hole of design patterns, that are often blindly applied, seemingly without any thought on why exactly. Specifically, the factory pattern has some valid uses, but because people overuse the pattern so much it’s now synonymous with unnecessary abstractions.

There are of course also established patterns where you should really have a damn good reason to actually use it – at least in a strictly object-oriented context. Singletons for example. “Singleton” is in essence just a fancy name for a global variable – great. Fun little side note: In the Spring framework Beans by default get the Singleton scope, meaning if not stated otherwise every single bean is global.

### A Dream of Spring

Another thing I’ve been noticing with modern “enterprise” applications is that they are not actually object-oriented. Entities, DTOs, … are records, not objects. Beans, Services, Repositories, … don’t hold state and could just as well be plain functions in modules.

We are using languages that force us to think in classes with architectures that don’t require objects – Spring Boot could just as well be written in C.

## Conclusion

What a ride. I think this my longest blog post so far. Maybe even a bit too long… I’ll make sure the next one is shorter. 😅

I also found this really interesting [talk by Barbara Liskov about abstraction](https://www.youtube.com/watch?v=dtZ-o96bH9A), but I just wasn’t sure where to put it, so here you go. (I particularly like the stab against Python for throwing encapsulation out the window. 😆)

Anyway, I hope I could shed some light on the topic, maybe you learned something, or at the very least you found my ramblings somewhat entertaining.

See you soon,
Sigma

### Footnotes