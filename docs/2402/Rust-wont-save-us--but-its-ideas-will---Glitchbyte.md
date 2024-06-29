<!--yml

分类：未分类

日期：2024-05-27 14:44:11

-->

# Rust 不会拯救我们，但它的理念会。 • Glitchbyte

> 来源：[https://glitchbyte.io/posts/rust-wont-save-us/](https://glitchbyte.io/posts/rust-wont-save-us/)

**2024 年 2 月 10 日更新**：在接受一些建设性反馈后，示例中的错误已经修复。

## 我们要拯救什么？

最近，我看到了这篇名为“[Rust Won’t Save Us: 2023 年已知利用漏洞的分析](https://www.horizon3.ai/analysis-of-2023s-known-exploited-vulnerabilities/)”的文章。

作为点击诱饵，我点击了。

我的简短背景：我在网络安全领域工作了将近10年。我对网络安全了解甚深，远远超过我对开发的了解。

我的日常工作是保护基础设施和代码。

这样的文章引起了我的兴趣。

我已经用 Rust 写程序几年了。

我开始写 Rust 是因为它声称具有内存安全性，并且它成为我最喜欢使用的语言。我甚至成功将 Rust 用于我参与过的最酷的项目之一的生产环境中。

那么这篇文章在谈论什么？

TL;DR: Rust 是为了解决与内存相关的漏洞和问题而制作的，但这仅占到 2023 年最受利用漏洞的 19.5%。路由和路径滥用漏洞与内存漏洞并列第二，分别为 4.9% 的默认秘密、请求走私（Request Smuggling）（4.9%）和弱加密（2.4%）。最被滥用的漏洞是什么？是不安全暴露函数（IEF），占到 48.8%。

文章继续提出了任何网络安全专业人士都会知道的最一般化的建议：

> 1.  供应商 1\. 开发您的工程师在他们使用的框架上的知识深度 2\. 加固、标准化并审计跨产品使用这些框架 3\. 为您的产品启用和公开详细的日志记录
> 1.  
> 1.  开发者 1\. 假设您编写的所有代码都可以从未经认证的上下文中访问 2\. 实践深度防御编程，不要让攻击者轻易外壳
> 1.  
> 1.  防御者 1\. 减少不必要地暴露在互联网上的攻击面，如果不需要在那里 2\. 主动启用所有与互联网接触的产品的日志记录，以及远程日志记录（如果可能的话）
> 1.  
> 1.  研究人员 1\. 寻找框架集成的错误

因此，Rust 不会拯救我们。

这里确实有些道理，并且文章给出的建议也是正确的。

但它并没有深入探讨 Rust 最初的制作目的。

它没有询问“我们能否减少/消除类似于我们减少内存漏洞的方式的IEF滥用”这个问题。

## 关注于 IEF

什么是不安全暴露函数（IEF）？

让我们看一下[MITRE](https://cwe.mitre.org/data/definitions/749.html)的定义：

> 该产品提供了应用程序编程接口（API）或类似接口与外部参与者进行交互，但该接口包含未经适当限制的危险方法或函数。
> 
> 这种弱点可能导致多种结果性的弱点，取决于暴露方法的行为。它可以适用于任何数量的技术和方法，如ActiveX控件、Java函数、IOCTL等。
> 
> 暴露可能以几种不同的方式发生。
> 
> +   函数/方法本来不应该暴露给外部用户。
> +   
> +   函数/方法只应该被一组有限的操作者访问，如来自单个网站的基于互联网的访问。

IEF是外部世界本不应该访问的函数。

### 默认情况下是私有的

让我们从MITRE页面上的一个例子开始看一下：

```
public  void  removeDatabase(String databaseName) {
 try {
 Statement stmt = conn.createStatement();
 stmt.execute("DROP DATABASE "  + databaseName);
 } catch (SQLException  ex) {
 ...
 }
}
```

在这个例子中，我们有一个名为`removeDatabase`的Java方法，它将删除参数中指定名称的数据库。

问题在于这个方法本不应该是公开的。通过声明为公共的，应用程序的其余部分可以访问该方法，尽管它应该是受限制的。

```
private  void  removeDatabase(String databaseName) {
 try {
 Statement stmt = conn.createStatement();
 stmt.execute("DROP DATABASE "  + databaseName);
 } catch (SQLException  ex) {
 ...
 }
}
```

Java有几个关键字用于决定代码库中其余部分应该具有的访问级别：

+   `public`

+   `protected`

+   `private`

+   *no modifier*，当你不指定访问级别时，默认为包私有级别。

现在让我们拿同样的例子来看看在Rust中会是什么样子。

```
pub  fn  remove_database(conn:  &Connection, database_name:  &str) ->  Result<()> {
 let  mut stmt = conn.prepare(&format!("DROP DATABASE {}", database_name))?;
 stmt.execute([])?;
 Ok(())
}
```

Rust只有`pub`作为一个关键字来确定一个项是公共的还是私有的。

默认情况下，所有的Rust代码都是私有的。

在Java中，如果没有添加修饰符，Java会假设具有包私有访问权限，这是包级别而不是项级别的。

换句话说，Rust中的可见性控制是显式的，并且通过`pub`修饰符进行控制；如果未指定修饰符，Java的可见性控制是隐式的，允许基于它们在代码库中的位置进行访问控制。如果指定了修饰符，则是显式的。

为了使Rust函数是公共的，我们必须声明为公共：

```
pub  fn  remove_database(conn:  &Connection, database_name:  &str) ->  Result<()> {
...
}
```

这个例子是一个简单的作用域错误，或者说是懒惰。

它很容易被忽视，但Rust不太可能让你犯这种错误。

“好吧，所以默认是私有的，有什么大不了的。还有其他方法可以不正确地访问函数并滥用它们。”

### 野外的IEF

我们将要看看[CVE-2023-22515：Atlassian Confluence](https://attackerkb.com/topics/Q5f0ItSzw5/cve-2023-22515/rapid7-analysis)漏洞以及我们如何潜在地解决它。

问题是什么？

> 应用程序不安全地暴露了一个端点，`/server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false`，允许修改服务器的配置状态。将此状态设置为false允许攻击者重新进入应用程序设置并添加管理用户。
> 
> 我们必须注意到类`com.atlassian.confluence.core.actions.ServerInfoAction`扩展了类`com.atlassian.confluence.core.ConfluenceActionSupport`。这在利用过程中将非常重要。

Rust没有继承，这使得它不太可能意外地继承父类的不良行为。

相反，Rust提供了其他机制来实现代码重用和多态性。

Rust设计师决定从语言中省略继承：

+   简化了语言，减少了与继承相关的问题，如“菱形问题”或“脆弱基类”问题。

+   鼓励组合而非继承。

+   提供了基于特性的多态作为一种替代方案，允许您定义类型可以实现的行为。

继承在某些情况下可能很有用，但Rust的设计哲学优先考虑了简洁性、安全性和表达力，倾向于使用组合、特性和其他语言特性。

> 我们知道我们可以利用XWorks2特性，提供HTTP参数来调用对象的setter方法。我们需要识别一个未经身份验证的端点，其Action对象还公开了一个适合的获取方法，允许我们访问应用程序配置。
> 
> 回想起类`com.atlassian.confluence.core.actions.ServerInfoAction`，在diffing期间看到，我们探索了它继承自的基类`com.atlassian.confluence.core.ConfluenceActionSupport`。

```
public  class  ConfluenceActionSupport  extends  ActionSupport  implements  LocaleProvider, WebInterface, MessageHolderAware {
 // ...snip...
 public  BootstrapStatusProvider  getBootstrapStatusProvider() {
 if (this.bootstrapStatusProvider ==  null)
 this.bootstrapStatusProvider = BootstrapStatusProviderImpl.getInstance();
 return  this.bootstrapStatusProvider;
 }
 // ...snip...
}
```

> 我们可以看到这个类有一个获取器方法`getBootstrapStatusProvider`，它返回我们正在寻找的`BootstrapStatusProviderImpl`实例。
> 
> `BootstrapStatusProviderImpl`反过来有一个获取方法`getApplicationConfig`，返回应用程序的配置。

```
public  class  BootstrapStatusProviderImpl  implements  BootstrapStatusProvider, BootstrapManagerInternal {
 // ...snip...
 public  ApplicationConfiguration  getApplicationConfig() {
 return  this.delegate.getApplicationConfig();
 }
 // ...snip...
}
```

> 最后，我们可以看到类`com.atlassian.config.ApplicationConfig`实现了设置方法`setSetupComplete`。

```
public  class  ApplicationConfig  implements  ApplicationConfiguration {
 public  synchronized  void  setSetupComplete(boolean  setupComplete) {
 this.setupComplete = setupComplete;
 }
 }
```

Rust的可变性方法在这里会有所帮助。

如果你希望设置器是可变的，那么你需要明确说明。

## 查看路由滥用

路由滥用与内存损坏问题并列第二。

[Horizon3提供的示例](https://www.horizon3.ai/moveit-transfer-cve-2023-34362-deep-dive-and-indicators-of-compromise/)涉及到Progress针对其MOVEit Transfer应用的安全咨询：

> Progress发布了一个[安全咨询](https://community.progress.com/s/article/MOVEit-Transfer-Critical-Vulnerability-31May2023)，详细描述了导致远程代码执行的SQL注入，并敦促客户更新到最新版本。该漏洞CVE-2023-34362在发布时被认为已在野外被利用，起码存在30天。
> 
> 提取`X-siLock-Transaction`头部的函数，将其值与`folder_add_by_path`进行比较时存在一个bug。**它将错误地提取以`X-siLock-Transaction`结尾的标头**，因此攻击者可以通过提供例如`xX-siLock-Transaction=folder_add_by_path`的标头来欺骗函数，进而将请求传递到machine2.aspx，同时提供正确格式化的标头，以执行我们自己的任意事务。

使用Rust的[http::header::HeaderName](https://docs.rs/http/1.0.0/http/header/struct.HeaderName.html)能够捕捉这个错误，因为传递的头部不会被当作字符串处理，而是已知类型的头部。

这个http库在几乎每个Rust Web框架中都用于头部解析。

在Rust中，正确的头部解析方式需要创建一个`HeaderName`，然后使用它来获取头部，而不是将头部名称视为字符串。

这是Rust类型系统的一个优势。

它设计成富有表现力，允许开发者以简洁可读的方式表达复杂的思想和模式。

## Rust的内存安全性

值得注意的是，Rust消除了大多数内存腐败问题，这将处理2023年利用的约20%的漏洞。

当你考虑到微软和谷歌Chrome所述的报告，声称它们70%的漏洞与内存安全有关时，这是非常重要的。

Rust甚至没有`NULL`，这是Tony Hoare称之为他的“十亿美元错误”的东西。

在Rust中，我们不会很快遇到NULL指针异常问题（对不起，Java）。

## 我们需要的英雄

平均开发者更关心立即交付产品，而不是如何从一开始设计安全性，后续再修复错误。

在许多流程中，安全性是事后才考虑的，是一个后续添加的东西。

要使安全性起作用，必须从一开始就考虑在内。

你不能在蛋糕烤好后再加蛋；你需要把它加入到混合物中。

安全性是通过在多层次上堆叠许多防御技术来阻止攻击者的过程。

这是一场不断的猫鼠游戏。

我们需要的英雄不是Rust。

Rust不会解决所有漏洞并神奇地修复它们。

然而，Rust从其设计哲学中具有的内在特性使其比普通语言更安全。

这就是我们的英雄。

Rust可能无法拯救我们，但它体现的理念会。

+   默认为私有

+   默认为不可变

+   在编译时检查类型安全性

+   借用检查器和所有权模型减少内存损坏

+   安全的抽象和习惯用法模式可以预防常见的安全漏洞

Rust的设计哲学是朝着正确的方向迈出的一步。

Rust不依赖开发者来处理所有细节。它减轻了开发者的责任，让他们更多地关注开发，少些担心安全性/正确性。

想象一下使用一种可以预防所有这些漏洞类型的语言。

为什么我们谈论编程语言时，似乎没有办法通过设计来改善它们的内在安全性呢？

除了Horizon提出的所有建议外，编程语言也应该是其中之一。

我们应该期望所有的语言都更安全。
