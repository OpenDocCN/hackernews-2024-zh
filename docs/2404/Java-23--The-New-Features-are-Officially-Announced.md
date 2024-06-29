<!--yml

category: 未分类

date: 2024-05-27 13:17:14

-->

# Java 23：新功能正式宣布

> 来源：[https://coderoasis.com/java-23-new-features/](https://coderoasis.com/java-23-new-features/)

Java Development Kit 23（JDK）的最新版本在最新发布中有四个新功能。到目前为止，两个主要功能值得关注，分别是 Vector API、流收集器的第二个预览，以及在模式中的基本类型预览，例如 `instanceof` 和 `switch`。

作为友情提醒，最新版本将于 9 月 19 日发布。

[Vector API](https://openjdk.org/jeps/469?ref=coderoasis.com) 在 Java Development Kit 16 发布以来一直在之前的版本中进行孵化，直到最新的 22 版本。这个新版本将引入一个 API 来帮助表达在运行时编译的向量计算。这旨在在不同支持的 CPU 架构上编译优化的向量指令。该提议的一些目标包括提供标准化的 API、运行时编译、在 `x64` 和 `AArch64` CPU 架构上提供更好的性能、优雅降级、平台无关性，并与 Valhalla 项目更加对齐——该项目旨在用值对象增强 Java 对象模型。

* * *

> 你喜欢正在阅读的内容吗？我们推荐您阅读下一篇文章——[**理解 Rust 编程语言中的内存安全性**](Understanding%20Memory%20Safety%20in%20the%20Rust%20Programming%20Language)，与我们一起继续学习。  

[理解 Rust 编程语言中的内存安全性](Understanding%20Memory%20Safety%20in%20the%20Rust%20Programming%20Language)

过去十年对于 Rust 编程语言而言是令人惊叹的，它成为了编写快速、机器原生软件和 Web 应用程序的首选。语言设计强调内存安全性的强大保证。我们在这里努力理解](https://coderoasis.com/rust-lang-memory-safety/)

* * *

## Vector API 代码示例

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

## 流收集器 & Stream API

如果你还记得，[流收集器](https://openjdk.org/jeps/473?ref=coderoasis.com) 在 Java Development Kit 22 发布中预览过。现在它将完全添加到 JDK 23 中。流收集器将增强 [Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html?ref=coderoasis.com)，支持自定义操作。它们还允许流管道以一种当前内置操作难以实现的方式转换数据。该目标是使流管道更加灵活和表达力强——这也将允许自定义操作来操作无限大小的流。

使用 [类文件 API](https://openjdk.org/jeps/466?ref=coderoasis.com) 旨在提供一个处理跟踪 Java 虚拟机规范定义的类文件格式的 API。这也意味着它将使 JDK 组件能够迁移到标准 API 并移除 Java Development Kit 的 [ASM 库](https://asm.ow2.io/?ref=coderoasis.com)。类文件 API 添加了一些微调 - 包括简化 `CodeBuilder` 类，默认包含字节码指令的工厂方法，包括低级工厂、中级工厂和基本块的高级构建器。下面是一个 ASM CodeBuilder 的示例代码，而不是一个 Java CodeBuilder。

## ASM CodeBuilder 代码示例

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

## Java CodeBuilder 示例代码

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

在 Java Development Kit 23 中，Java CodeBuilder 删除了重复低级方法或在更糟糕的情况下很少使用的中级方法。同时，您重命名了其余的中级方法以改进可用性。他们还优化了 `ClassSignature` 类模型。这意味着它已经改进以更准确地模拟 `superclasses` 和 `superinterfaces` 的泛型签名。根据 OpenJDK 提议这一功能背后的特性，Java 平台应该定义并实现一个与类文件格式一起演进的标准类文件 API，这个格式可以很容易地每六个月左右进行更改或演进。

## 模式中的原始类型

正如之前提到的，要包括在最新计划的 Java Development Kit 23 版本的特性和发布中，我们获得了另一个预览特性，这对于 Java 开发者来说非常值得注意。该特性是 [模式中的原始类型、`instanceof` 和 `switch`](https://openjdk.org/jeps/455?ref=coderoasis.com)。该特性将通过允许在模式上下文中使用原始类型模式来显著增强模式匹配。然后我们扩展了 `instanceof` 和 `switch` 以处理所有原始类型。这还包括模式匹配在嵌套和顶层上下文中使用原始类型模式 - 提供易于使用的构造，消除由于不安全的转换而丢失信息的风险。

其他目标包括将模式类型与 `instanceof` 对齐，将 `instanceof` 与安全转换对齐，并允许 `switch` 处理任何原始类型的值。

## 原始类型代码示例

```
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    case int i -> "unknown status: " + i;
}
```

## super(...) 之前的语句

Java 开发工具包 22 版本中预览的其他特性也可能很容易进入 Java 开发工具包 23 版本。这些包括 `super(...)` 前的语句，这将为开发者在表达构造函数行为时提供更大的自由——即字符串模板。这将使得表达包含在运行时计算的值的字符串变得容易——即作用域值。这将支持在线程内外共享不可变数据；以及隐式声明的类和实例主方法。

## Super(...) 代码示例

```
public class PositiveBigInteger extends BigInteger {

    public PositiveBigInteger(long value) {
        if (value <= 0)
            throw new IllegalArgumentException("non-positive value");
        super(value);
    }

}
```

总的来说，这些变化将使初学程序员更容易编写程序，无需了解专为大型或企业级应用程序开发设计的编程语言特性。

Oracle 还揭示了 [2024 年 Java 的计划](https://www.youtube.com/watch?v=iL7d-gGrms8&ref=coderoasis.com)。Oracle 概述了涉及 OpenJDK 项目的改进，从 [Amber](https://openjdk.org/projects/amber/?ref=coderoasis.com) 开始，旨在开发更小、更具生产力的特性，到 [Babylon](https://openjdk.org/projects/babylon/?ref=coderoasis.com)，旨在将 Java 扩展到如 GPU 这样的外部编程模型，再到 [Valhalla](https://openjdk.org/projects/valhalla/?ref=coderoasis.com)，旨在通过值对象增强 Java 对象模型，以消除长期存在的性能瓶颈。

* * *

> 你喜欢阅读 CoderOasis 技术博客的内容吗？我们建议你阅读我们的 [***黑客行动：通过数据泄露和篡改实现社会正义***](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/) 作为下一个选择。

[黑客行动：通过数据泄露和篡改实现社会正义

二月末，号称 JaXpArO 和 My Little Anonymous Revival Project 的黑客活动分子攻破了名为 Gab 的极右社交媒体平台。他们成功从后台数据库中获取了七十吉字节的数据。数据包含用户资料、私人帖子、聊天消息等内容——很多](https://coderoasis.com/hacktivism-social-justice-by-data-leaks-and-defacements/)

* * *

> 你知道我们有 [**社区论坛**](https://forums.coderoasis.com/?ref=coderoasis.com) 和 [**Discord 服务器**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com) 吗？我们邀请大家加入我们！想与社区其他成员讨论这篇文章吗？想加入一个轻松的地方，闲聊和讨论编程、安全、Web 开发和 Linux 等话题吗？今天就考虑加入我们吧！

[加入 CoderOasis.com Discord 服务器！

Coder   CoderOasis 提供关于编程、安全、Web 开发、Linux、系统管理员等技术新闻文章。 | 112 名成员](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com) [CoderOasis 论坛

CoderOasis 社区论坛，成员们可以在这里一起讨论技术，共享资源。](https://forums.coderoasis.com/?ref=coderoasis.com)
