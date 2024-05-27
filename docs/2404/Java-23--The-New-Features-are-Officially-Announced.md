<!--yml
category: 未分类
date: 2024-05-27 13:17:14
-->

# Java 23: The New Features are Officially Announced

> 来源：[https://coderoasis.com/java-23-new-features/](https://coderoasis.com/java-23-new-features/)

The newest version of the Java Development Kit 23 (JDK) has four new features in the newest release. The two major features which are noteworthy so far is the Vector API, a second preview of the Stream Gatherers, and a preview of primitive types in patterns – such as `instanceof` and `switch`.

As a friendly reminder, the newest version is due on September 19th.

The [Vector API](https://openjdk.org/jeps/469?ref=coderoasis.com) has been incubated in previous versions of Java since the release of the Java Development Kit 16 all the way to the latest version of 22\. This new version will introduce an API to help express vector computations that compile at run time. This is to have an optimal vector instructions on different supported CPU architectures. Some of the goals of this proposal includes providing a standardized API, runtime compilation, better performance on the `x64` and `AArch64` CPU architectures, graceful degradation, being platform-agnostic, and aligning more with Project Valhalla – which is supposed to augment the Java Object Model with Value Objects instead.

* * *

> Do you like what you are reading? We do recommend this next article – [**Understanding Memory Safety in the Rust Programming Language**](Understanding%20Memory%20Safety%20in%20the%20Rust%20Programming%20Language) – to you to continue learning with us.

 [Understanding Memory Safety in the Rust Programming Language

The last decade has been amazing for the Rust Programming Language to become the choice for people has the goal of writing fast, machine native software and web applications. The way that the language was made was for strong guarantees for memory safety. We are here to try to understand](https://coderoasis.com/rust-lang-memory-safety/) 

* * *

## Vector API Code Example

```
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}
```

## Stream Gatherers & Stream API

If you remember, [Stream Gatherers](https://openjdk.org/jeps/473?ref=coderoasis.com) was previewed part of the Java Development Kid 22 release. Now it is going to be fully added to JDK 23\. The Stream Gatherers will enhance the [Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html?ref=coderoasis.com) to support custom operations. They will also allow stream pipelines to help transform data in ways that are harder to achieve with the current built-in operations. The goals for this is to making the stream pipelines far more flexible and expressive – this will also allow for custom operations to manipulate streams of infinite sizes.

With using the [class-file API](https://openjdk.org/jeps/466?ref=coderoasis.com) is intended to provide an API for processing class files that tracks the class file format defined by the [Java Virtual Machine specification](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-4.html?ref=coderoasis.com). This also means that it would enable JDK components to migrate to the standard API and remove the Java Development Kit’s copy of the [ASM library](https://asm.ow2.io/?ref=coderoasis.com). The class-file API adds some fine-tuning – including streamlining the `CodeBuilder` class, which by default contains factory methods for bytecode instructions, including low-level factories, mid-level factories, and high-level builders for basic blocks. Right below is some example code of an ASM CodeBuilder than a Java CodeBuilder.

## ASM CodeBuilder Code Example

```
ClassWriter classWriter = ...;
MethodVisitor mv = classWriter.visitMethod(0, "fooBar", "(ZI)V", null, null);
mv.visitCode();
mv.visitVarInsn(ILOAD, 1);
Label label1 = new Label();
mv.visitJumpInsn(IFEQ, label1);
mv.visitVarInsn(ALOAD, 0);
mv.visitVarInsn(ILOAD, 2);
mv.visitMethodInsn(INVOKEVIRTUAL, "Foo", "foo", "(I)V", false);
Label label2 = new Label();
mv.visitJumpInsn(GOTO, label2);
mv.visitLabel(label1);
mv.visitVarInsn(ALOAD, 0);
mv.visitVarInsn(ILOAD, 2);
mv.visitMethodInsn(INVOKEVIRTUAL, "Foo", "bar", "(I)V", false);
mv.visitLabel(label2);
mv.visitInsn(RETURN);
mv.visitEnd();
```

## Java CodeBuilder Example Code

```
ClassBuilder classBuilder = ...;
classBuilder.withMethod("fooBar", MethodTypeDesc.of(CD_void, CD_boolean, CD_int), flags,
                        methodBuilder -> methodBuilder.withCode(codeBuilder -> {
    Label label1 = codeBuilder.newLabel();
    Label label2 = codeBuilder.newLabel();
    codeBuilder.iload(1)
        .ifeq(label1)
        .aload(0)
        .iload(2)
        .invokevirtual(ClassDesc.of("Foo"), "foo", MethodTypeDesc.of(CD_void, CD_int))
        .goto_(label2)
        .labelBinding(label1)
        .aload(0)
        .iload(2)
        .invokevirtual(ClassDesc.of("Foo"), "bar", MethodTypeDesc.of(CD_void, CD_int))
        .labelBinding(label2);
        .return_();
});
```

In the Java Development Kit 23, the Java CodeBuilder removes mid-level methods that duplicated low-level methods, or at worse, were infrequently used. While you rename the remaining mid-level methods to improve usability. They also refined the `ClassSignature` class model. This means that it has been improved to model the generic signatures of `superclasses` and `superinterfaces` more accurately. According to the OpenJDK proposal behind this feature, the Java platform should define and implement a standard class-file API that evolves together with the class-file format, which can easily change or evolve about every six months.

## Primitive Types in Patterns

As previously mentioned to be included with the newest planned features and release of the Java Development Kit 23, we gained another preview feature that is highly worth noting for Java Developers. The feature is [primitive types in patterns, `instanceof`, and `switch`](https://openjdk.org/jeps/455?ref=coderoasis.com). This feature will majorly enhance pattern matching by allowing primitive type patterns in pattern contexts. Then we extend the `instanceof` and `switch` to work with all primitive types. This also includes pattern matching to use primitive type patterns in both nested and top-level contexts – providing easy-to-use constructs that eliminate the risk of losing information due to unsafe casts

Other goals include aligning pattern types with `instanceof`, aligning `instanceof` with safe casting, and allowing `switch` to process values of any primitive type.

## Primitive Type Code Example

```
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    case int i -> "unknown status: " + i;
}
```

## Statements before super(...)

Other features that were previewed in the Java Development Kit 22 release could also make it quite easily into the Java Development Kit 23 release. These include statements before `super(…)`, which would give developers greater freedom in expressing constructor behavior – meaning string templates. This would make it easy to express strings that include values computed at run time – meaning scoped values. This would enable sharing of immutable data within and across threads; and implicitly declared classes and instance main methods.

## Super(...) Code Example

```
public class PositiveBigInteger extends BigInteger {

    public PositiveBigInteger(long value) {
        if (value <= 0)
            throw new IllegalArgumentException("non-positive value");
        super(value);
    }

}
```

Overall, these changes will make it easier for beginning programmers to write programs without needing to understand the programming language features designed for larger or enterprise application development. 

Oracle also has revealed [plans for Java in 2024](https://www.youtube.com/watch?v=iL7d-gGrms8&ref=coderoasis.com). Oracle outlined improvements that involve  OpenJDK projects ranging from[ Amber](https://openjdk.org/projects/amber/?ref=coderoasis.com), for developing smaller, productivity-oriented features, to [Babylon](https://openjdk.org/projects/babylon/?ref=coderoasis.com), for extending Java to foreign programming models such as GPUs, to [Valhalla](https://openjdk.org/projects/valhalla/?ref=coderoasis.com), for augmenting the Java object model with value objects to eliminate longstanding performance bottlenecks.

* * *

> Do you like what you're reading from the CoderOasis Technology Blog? We recommend reading our [***Hacktivism: Social Justice by Data Leaks and Defacements***](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/) as your next choice.

 [Hacktivism: Social Justice by Data Leaks and Defacements

Around the end of February, a hacktivist that calls himself JaXpArO and My Little Anonymous Revival Project breached the far-right social media platform named Gab. They managed to gain seventy gigabytes of data from the backend databases. The data contained user profiles, private posts, chat messages, and more – a lot](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/) 

* * *

> Did you know we have a [**Community Forums**](https://forums.coderoasis.com/?ref=coderoasis.com) and [**Discord Server**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)? which we invite everyone to join us? Want to discuss this article with other members of our community? Want to join a laid back place to chill and discuss topics like programming, cybersecurity, web development, and Linux? Consider joining us today!

 [Join the CoderOasis.com Discord Server!

CoderOasis offers technology news articles about programming, security, web development, Linux, systems admin, and more. | 112 members](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)  [CoderOasis Forums

CoderOasis Community Forums where our members can have a place to discuss technology together and share resources with each other.](https://forums.coderoasis.com/?ref=coderoasis.com)