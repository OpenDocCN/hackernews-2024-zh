<!--yml

category: 未分类

date: 2024-05-29 12:40:49

-->

# Java 22的新功能和安全增强

> 来源：[https://coderoasis.com/java-22-is-released/](https://coderoasis.com/java-22-is-released/)

Java开发工具包22 - Java标准版的下一个主要版本 - 现在作为生产版本完全可用！最新版本包括12个功能和重大安全增强。

您可以立即从[oracle.com](https://www.oracle.com/java/technologies/downloads/?ref=coderoasis.com)下载它。此版本是一个短期支持版本，将由Oracle提供6个月的首席支持。长期支持版本如前身Java开发工具包（JDK）21仍将获得5年的首席支持。扩展支持将继续为JDK 21提供，直至2031年9月。

* * *

想了解更多关于像Java这样的编程语言吗？我们有一篇关于编程语言的简介文章，建议您在阅读本文的同时也看一下！

[编程语言简介：目标是什么？

在当前数字化时代，几乎一切都可以并最终将完全数字化，编程语言作为使开发人员能够将想法付诸实现的基础工具而存在。本博客深入探讨了各种编程语言的目标，探索了它们在这方面的独特之处](https://coderoasis.com/introduction-to-programming/)

## 新功能

最新版本现在提供了来自开发项目的功能，包括[Amber](https://openjdk.org/projects/amber/?ref=coderoasis.com) - 开发更小、面向生产力的Java语言特性；[Loom](https://wiki.openjdk.org/display/loom/Main?ref=coderoasis.com) - 开发Java虚拟机特性和API；以及[Panama](https://openjdk.org/projects/panama/?ref=coderoasis.com)，它开发了Java与非Java API之间的互联功能。

[作用域值](https://openjdk.org/jeps/464?ref=coderoasis.com)允许在线程内部和跨线程之间安全高效地共享不可变数据。这比使用大量线程变量时更可取 - 特别是当使用大量线程变量时。主要目标是易用性、易懂性、鲁棒性和性能。

[流聚合器](https://openjdk.org/jeps/461?ref=coderoasis.com)将增强流API以支持自定义中间操作。此预览功能将允许流管道以不易通过现有内置中间操作实现的方式转换数据。

这些最新功能加入了结构化并发的第二次预览，`super(…)`之前语句的预览，类文件 API 的预览，G1 垃圾收集器的区域固定，字符串模板的第二次预览，未命名变量和模式，外部函数和内存 API，以及第七个向量 API 的孵化器。还有第二次预览的隐式声明类和实例主方法，以及对 Java 启动器的增强，使其能够运行多文件程序。

使用[结构化并发](https://openjdk.org/jeps/462?ref=coderoasis.com)，来自项目 Loom，通过一个 API 简化了并发编程，该 API 将运行在不同线程中的一组相关任务视为一个工作单元，从而简化了错误处理和取消操作，提高了可靠性，并增强了可观察性。

Java 应用程序启动器的增强将允许其运行作为多个 Java 源代码文件提供的程序。启动多文件源代码程序的目标是使从小程序向大程序的过渡更加渐进，允许开发人员选择何时以及何时配置构建工具的麻烦。

```
// file MainApplication.java
public class MainApplication {
	   public static void main(String[] args) {
      Person p = new Person("Exiled", "Tech");
      System.out.println("Hello there! " + p.toString() + "!");
  }
}

// file Person.java
record Person(String fName, String lName) {
	public String toString(){
        return fName + " " + lName;
    }
}

$ java MainApplication.java
Hello there! Exiled Tech! 
```

[类文件 API](https://openjdk.org/jeps/457?ref=coderoasis.com) 将提供一个标准的 API 用于解析、生成和转换 Java 类文件。旨在使 JDK 组件能够迁移到标准 API，并最终移除 JDK 内部对第三方[ASM](https://asm.ow2.io/?ref=coderoasis.com)库的依赖。解析、生成和转换类文件是无处不在的，独立工具和库使用它们来检查和扩展程序，而不会危及源代码的可维护性。

然而，由于标准 Java 的每六个月发布一次，Java 类文件格式现在演变得比以前更快。这种演变速度导致框架更频繁地遇到比类文件库更新的类文件。这导致错误，并且框架开发人员试图编写代码来解析未来的类文件，希望不会发生太大的变化。计划是使 Java 平台定义并实现一个标准的类文件 API，与类文件格式一起演进。

[G1的区域固定](https://openjdk.org/jeps/423?ref=coderoasis.com)旨在减少延迟，因此在Java Native Interface (JNI)关键区域期间无需禁用垃圾收集（GC）。目标包括不因关键JNI区域而导致线程停顿，由于这些区域而开始垃圾收集不增加延迟，当这些区域未激活时不会导致GC暂停时间增加，并在这些区域激活时最小化GC暂停时间的回归。当前，默认的GC（G1）在每个关键区域期间禁用垃圾收集，这可能对延迟产生重大影响。通过这一变更，Java线程将不再等待G1 GC操作完成。

[隐式声明的类和实例方法](https://openjdk.org/jeps/463?ref=coderoasis.com)，来自Amber项目，在JDK 21中预览为未命名类和实例方法。此功能将在JDK 22中进行第二次预览，并进行重大更改。该功能旨在使Java语言能够发展，使学生无需理解为大型程序设计的语言特性即可编写其第一个程序。学生可以为单类程序编写简化的声明，然后在其技能增长时无缝扩展程序以使用更高级的功能。

```
void main(){
	System.out.println("Hello World!");
} 
```

* * *

## 预览功能

关于语言中构造函数的[`super(…)`之前的语句预览](https://openjdk.org/jeps/447?ref=coderoasis.com)，同样来自Amber项目，允许在语言中的构造函数中出现不引用正在创建的实例的语句。计划的目标之一是为开发人员提供更大的自由来表达构造函数的行为，从而使当前必须因此而被因素化为辅助静态方法、辅助中间构造函数或构造函数参数的逻辑能够更自然地放置。

```
public class PositiveBigInteger extends BigInteger {
	public PositiveBigInteger(long value) {
		if (value <= 0)     
	    	throw new IllegalArgumentException("non-positive value");
	    super(value);
	}
} 
```

另一个目标是保留现有的保证，即构造函数在类实例化期间按自顶向下顺序运行，确保子类构造函数中的代码不能干扰超类实例化。第三个明确的目标是不需要对JVM进行任何更改。这是到目前为止唯一一个没有在标准Java中预览或孵化过的JDK 22功能。

[字符串模板](https://openjdk.org/jeps/459?ref=coderoasis.com)，Amber项目的另一个特性，将通过将文字文本与嵌入式表达式和模板处理器耦合，产生专业化的结果，来补充Java现有的字符串文字和文本块。目标包括：

+   简化编写Java程序，使得在运行时轻松表达包含计算值的字符串。

+   增强混合文本和表达式的表达式的可读性，无论文本是否适合单个源行或跨多个源行。

+   通过支持验证和转换模板及其嵌入表达式的值，改进从用户提供的值生成字符串并传递到其他系统的程序安全性。

+   通过允许Java库定义字符串模板中使用的格式化语法，保持灵活性。

+   简化接受非Java语言（如XML和JSON）编写的字符串的API的使用。

+   允许创建从文字和嵌入表达式计算出的非字符串值，而无需通过中间字符串表示传递。

字符串模板在JDK 21中首次预览。第二个预览旨在获得额外的经验和反馈。除了模板表达式类型的技术更改外，与第一个预览相比没有变化。

[向量API（第七次孵化器）](https://openjdk.org/jeps/460?ref=coderoasis.com)，来自Project Panama，将表达可靠在支持的CPU架构上最优化的向量指令的向量计算。该API提供了一种在Java中编写复杂向量算法的方式，使用现有的HotSpot自动向量化算法，但具有使向量化更可预测和健壮的用户模型。

此功能已在之前版本的Java中孵化，最早出现在JDK 16中，即2021年3月。API的目标包括清晰简洁、与平台无关，并在x64 AArch64架构上提供可靠的运行时编译和性能，以及优雅降级。

[未命名变量和模式](https://openjdk.org/jeps/456?ref=coderoasis.com)，来自Project Amber，可以在需要但从未使用变量声明或嵌套模式时使用。计划的目标包括：

+   捕捉开发者意图，即给定绑定或lambda参数未使用，并强制执行该属性以澄清程序并减少错误的机会。

+   通过识别必须声明但未使用的变量来改进代码的可维护性。

+   允许多个模式出现在单个case标签中，前提是它们中没有一个声明模式变量。

+   通过省略不必要的嵌套类型模式来改善记录模式的可读性。

该提案在JDK 21中进行了预览，并将在JDK 22中不做任何更改地最终确定。

[外部函数和内存API](https://openjdk.org/jeps/454?ref=coderoasis.com)，来自Project Panama，允许Java程序与Java运行时之外的代码和数据进行互操作。提案指出，通过调用外部函数并安全访问外部内存，Java程序可以调用本地库并处理本地数据，而无需JNI（Java Native Interface）的脆弱性。

外部函数和内存 API 曾在 JDK 19、JDK 20 和 JDK 21 中进行了预览，现在将在 JDK 22 中完成。最新修订涵盖了四个方面：支持原生字符串的任意字符集、使客户端能够以编程方式构建 C 语言函数描述符，以及引入 `Enable-Native-Access` JAR 文件清单属性。后者允许可执行的 JAR 文件调用受限方法，而无需使用 `--enable-native-access` 命令行选项。第四个方面是提供一个新的链接器选项，允许客户端将堆段传递给 downcall 方法句柄。

* * *

## 安全增强功能

在 Oracle 的 [inside.java 网页](https://inside.java/?ref=coderoasis.com) 上的 [3 月 20 日博客文章](https://seanjmullan.org/blog/2024/03/20/jdk22?ref=coderoasis.com)，Java 安全库团队的技术负责人兼 OpenJDK 安全组负责人 Sean Mullan 详细介绍了 JDK 22 中的安全增强功能。

`java -Xshowsettings` 选项已增强，可用于打印系统设置和当前 JDK 配置的其他有用信息，现在还包括有关安全设置的详细信息。`-Xshowsettings:security` 将显示所有安全设置。子选项允许显示安全属性的值、已安装的安全提供者及其支持的算法，或启用的 TLS 协议和密码套件。

在密码学方面，新增了一个新的标准接口 `java.security.AsymmetricKey`。它是 `java.security.key` 的子接口，表示一个非对称密钥，可以是私有的或公共的。现有的 `java.security.PublicKey` 和 `java.security.PrivateKey` 类已经重构为 `AsymmetricKey` 的子接口。根据 Mullan 的说法，随着未来引入更多的非对称算法，`AsymmetricKey` 接口将使早期版本的 Java SE 更容易支持新的非对称算法，这些算法将参数表示为 `NamedParameterSpec`。

`jdk.crytpo.ec` 模块已经被弃用，并计划最终移除。`jdk.crytp.ec` 模块中的所有代码已经迁移到 `java.base` 模块，包括 `SunEC` 安全提供者。`jdk.crypto.ec` 模块现在为空但仍然存在。这一变更将使依赖椭圆曲线加密算法的应用程序更容易部署。

对于 PKI（公钥基础设施），已向 `cacerts` 密钥库添加了 10 个新的根 CA 证书，包括三个 eMudhra Technologies 根 CA 证书、四个 DigiCert 根 CA 证书，以及来自 Let’s Encrypt、Telia 和 Certigna 的各一个。

对于传输层安全性（TLS），新增了额外的属性来控制客户端和服务器证书链的最大长度。而对于 XML 签名，JDK 实现现在支持使用 RSA 签名算法和 SHA-3 摘要签名的 XML 签名。

JDK 22 扩展了 JCE（Java Cryptography Extension）对 HSS/LMS 签名算法的支持功能，将 HSS/LMS 支持添加到 `jarsigner` 和 `keytool` 实用程序中。此外，`jarsigner` 现在支持使用 HSS/LMS 算法对 JAR 文件进行签名和验证，而 `keytool` 现在支持生成 HSS/LMS 公钥对。然而，JDK 只支持 HSS/LMS 签名验证。开发人员将需要第三方提供商来使用 HSS/LMS 签署 JAR 文件。

* * *

> 您喜欢阅读 CoderOasis 技术博客的内容吗？我们推荐阅读我们的 [***从零开始在 Python 中实现 RSA***](https://coderoasis.com/implementing-rsa-from-scratch-in-python/) 作为您的下一个选择。

[Python 从零开始实现 RSA

这是一个关于从零开始在 Python 中实现 RSA 加密的指南。文章介绍了数学原理并提供了代码示例。](https://coderoasis.com/implementing-rsa-from-scratch-in-python/)

* * *

> 您知道我们有一个 [**社区论坛**](https://forums.coderoasis.com/?ref=coderoasis.com) 和 [**Discord 服务器**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com) 吗？我们邀请每个人加入我们。想与我们社区的其他成员讨论本文吗？想要加入一个轻松的地方，讨论编程、网络安全、Web 开发和 Linux 等主题吗？今天就考虑加入我们吧！

[加入 CoderOasis.com 的 Discord 服务器！

CoderOasis 提供关于编程、安全、Web 开发、Linux、系统管理等技术新闻文章。| 112 位成员](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com)  [CoderOasis 论坛

CoderOasis 社区论坛是我们的成员可以共同讨论技术并分享资源的地方。](https://forums.coderoasis.com/?ref=coderoasis.com)
