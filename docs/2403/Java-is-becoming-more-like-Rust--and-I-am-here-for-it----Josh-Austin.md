<!--yml

category: 未分类

date: 2024-05-27 14:35:23

-->

# Java 正变得越来越像 Rust，我对此感到兴奋！ | Josh Austin

> 来源：[https://joshaustin.tech/blog/java-is-becoming-rust/](https://joshaustin.tech/blog/java-is-becoming-rust/)

# Introduction[#](#introduction)

随着编程增强和复杂化的流行，许多编程语言也在效仿。Java 也不例外。

Rust 尽管在社区内存在问题，但已经获得了开发者的喜爱。理由是显而易见的：Rust 让开发者能够避免整个类别的问题，这要归功于编译器。编译器要求正确性，有时会让开发者感到一些不安。

我想展示一下 Rust 的两个特性，这些特性是使 Rust 出色的一部分，然后与 Java 中的最新创新进行比较。

*免责声明：我并不是在暗示 Rust 的这些特性导致它们在 Java 中出现。如果 Kotlin 和/或 Scala 对此更有影响，我一点也不会感到意外。但我很高兴看到 Rust 的特性在某种程度上也可以在 Java 中找到！*

# 不可变数据[#](#immutable-data)

在 Rust 中，默认情况下数据是不可变的。

```
struct User {  // struct members are immutable name: &'static str, age:  i32 }   fn main() {  // user and members are immutable let user = User { name: "Bob", age: 42 }; } 
```

如果我们希望数据是可变的，必须明确声明其可变性。

```
fn main() {  let mut user = User { name: "Bob", age: 42 }; user.name = "Jim"; } 
```

现在，一个普通的 Java 对象（也称为 POJO）通常默认以可变的方式编写，而且写法相当冗长。以下是 Java 8 中的示例：

```
public class User {  private String name; private int age;  public User(String name, int age) { this.name = name; this.age = age; }  public String getName() { return this.name; }  public int getAge() { return this.age; }  public void setName(String name) { this.name = name; }  public void setAge(int age) { this.age = age; }  public static void main(String[] args) { User user = new User("Bob", 42); user.setName("Jim"); System.out.println(user.getName()); }  // not to mention equals(), toString() and friends! } 
```

然而，由于现代 Java，记录类考虑到不可变性而简化了这一过程，我们的代码变得不那么冗长。以下是启用预览的 Java 21 代码示例。

```
record User(String name, int age) {}   void main() {  final var user = new User("Bob", 42); } 
```

# 代数数据类型[#](#algebraic-data-types)

在 [这个 YouTube 视频](https://www.youtube.com/watch?v=z-0-bbc80JM) 中，我们可以看到 Rust 如何利用代数类型帮助使无效状态无法表示，从而保证程序行为的稳定性。在 Rust 中，这是通过枚举来实现的：

```
enum RealCat {  Alive { is_hungry: bool }, Dead }   fn main() {  let real_cat: RealCat = RealCat::Alive { is_hungry: true }; match real_cat { RealCat::Alive { is_hungry } => { if is_hungry { println!("The cat demands a sacrifice."); } else { println!("The cat is bored."); } }, RealCat::Dead => println!("Oh no!!!") } } 
```

直到最近，这在 Java 中并不能以优雅的方式实现。但是，从 Java 21 开始，您可以使用包含记录和详尽的 switch 语法的封闭接口来优雅地实现这一点：

> RealCat.java

```
public sealed interface RealCat permits RealCat.Alive, RealCat.Dead {   record Alive(boolean isHungry) implements RealCat {} record Dead() implements RealCat {}   static void check(RealCat realCat) { switch (realCat) { case Alive aliveCat -> { if (aliveCat.isHungry()) { System.out.println("The cat demands a sacrifice."); } else { System.out.println("The cat is bored."); } } case Dead _ -> System.out.println("Oh no!!!"); } } } 
```

> RealCatApplication.java

```
void main() {  final var hungryCat = new RealCat.Alive(true); RealCat.check(hungryCat); } 
```

# Conclusion[#](#conclusion)

Java 永远不会变成 Rust，有着无数（好的）原因，但我很高兴看到 Rust 的一些强大特性正在进入 Java。这将加强 Java 在商业世界中作为首选编程语言的长期地位。Java 在过去 28 年中占据了显著的市场份额，并且由于 OpenJDK 项目致力于开发者生产力，它有望在未来 28 年内保持或重获重要市场份额。
