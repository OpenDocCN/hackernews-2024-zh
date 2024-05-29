<!--yml
category: 未分类
date: 2024-05-27 14:35:23
-->

# Java is becoming more like Rust, and I am here for it! | Josh Austin

> 来源：[https://joshaustin.tech/blog/java-is-becoming-rust/](https://joshaustin.tech/blog/java-is-becoming-rust/)

# Introduction[#](#introduction)

As programming enhancements and sophistications gain popularity, many programming languages follow suit. Java is no different.

Rust has gained developer love year-over-year despite problems within the community. And for good reason: Rust enables developers to avoid entire categories of problems thanks to the compiler. The compiler demands correctness to the point some developers begin to experience some insecurity.

I’d like to showcase two Rust features that are part of what makes Rust excellent, and then compare them with recent innovations in Java.

*Disclaimer: In no way am I claiming that these Rust features caused their counterparts to appear in Java. I would not be surprised if Kotlin and/or Scala were more influential in bringing this to life. But I am excited to see Rust features that can also be (sort of) found in Java!*

# Immutable Data[#](#immutable-data)

In Rust, data is immutable by default.

```
struct User {  // struct members are immutable name: &'static str, age:  i32 }   fn main() {  // user and members are immutable let user = User { name: "Bob", age: 42 }; } 
```

If we want data to be mutable, it must be declared mutable explicitly.

```
fn main() {  let mut user = User { name: "Bob", age: 42 }; user.name = "Jim"; } 
```

Now, a plain ol’ Java object (aka POJO) is often written with mutability by default, and rather verbosely. Here is how it looks like in Java 8:

```
public class User {  private String name; private int age;  public User(String name, int age) { this.name = name; this.age = age; }  public String getName() { return this.name; }  public int getAge() { return this.age; }  public void setName(String name) { this.name = name; }  public void setAge(int age) { this.age = age; }  public static void main(String[] args) { User user = new User("Bob", 42); user.setName("Jim"); System.out.println(user.getName()); }  // not to mention equals(), toString() and friends! } 
```

However, thanks to modern Java, record classes simplify this with immutability in mind and our code is far less verbose. The below is valid Java 21 code with previews enabled.

```
record User(String name, int age) {}   void main() {  final var user = new User("Bob", 42); } 
```

# Algebraic Data Types[#](#algebraic-data-types)

In [this YouTube video](https://www.youtube.com/watch?v=z-0-bbc80JM), we see how Rust can leverage algebraic types to aid in making invalid state unrepresentable, with strong guarantees against buggy behavior. In Rust, this is done using enums:

```
enum RealCat {  Alive { is_hungry: bool }, Dead }   fn main() {  let real_cat: RealCat = RealCat::Alive { is_hungry: true }; match real_cat { RealCat::Alive { is_hungry } => { if is_hungry { println!("The cat demands a sacrifice."); } else { println!("The cat is bored."); } }, RealCat::Dead => println!("Oh no!!!") } } 
```

Up until recently, this could not be implemented elegantly in Java. As of Java 21 and beyond, however, you can implement this in Java elegantly using sealed interfaces containing records and using exhaustive switch syntax:

> RealCat.java

```
public sealed interface RealCat permits RealCat.Alive, RealCat.Dead {   record Alive(boolean isHungry) implements RealCat {} record Dead() implements RealCat {}   static void check(RealCat realCat) { switch (realCat) { case Alive aliveCat -> { if (aliveCat.isHungry()) { System.out.println("The cat demands a sacrifice."); } else { System.out.println("The cat is bored."); } } case Dead _ -> System.out.println("Oh no!!!"); } } } 
```

> RealCatApplication.java

```
void main() {  final var hungryCat = new RealCat.Alive(true); RealCat.check(hungryCat); } 
```

# Conclusion[#](#conclusion)

There are myriad (good) reasons that Java will never become Rust, but I’m glad to see some of Rust’s strong features making their way into Java. This will strengthen Java’s long-term placement as one of the programming languages of choice in the business world. Java has owned significant market share for the last 28 years and is poised to retain, if not regain significant market share over the next 28 years because of the OpenJDK project’s dedication to developer productivity.