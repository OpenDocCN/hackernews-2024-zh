<!--yml
category: 未分类
date: 2024-05-27 14:44:11
-->

# Rust wont save us, but its ideas will • Glitchbyte

> 来源：[https://glitchbyte.io/posts/rust-wont-save-us/](https://glitchbyte.io/posts/rust-wont-save-us/)

**UPDATE 2/10/2024**: After receiving some constructive feedback, the example errors were fixed.

## What are we saving?

Recently, I came across this article titled ”[Rust Won’t Save Us: An Analysis of 2023’s Known Exploited Vulnerabilities](https://www.horizon3.ai/analysis-of-2023s-known-exploited-vulnerabilities/)“.

Being the clickbait it is, I clicked.

Quick background on me: I’ve worked in cybersecurity for almost 10 years. I know cybersecurity way more than I know development.

My day job is securing infrastructure and code.

An article like this piques my interest.

I’ve been writing programs in Rust for a few years now.

I started writing Rust because of its claim to memory safety, and it became my favorite language to use. I’ve even managed to ship Rust to prod in one of the coolest projects I’ve had the honor of being apart of.

So what is this article talking about?

TL;DR: Rust was made to solve memory-related vulnerabilities and issues, but that only makes up 19.5% of the most exploited vulnerabilities in 2023\. Routing and Path abuse exploits tied for second place with memory vulns, followed by Default Secrets (4.9%), Request Smuggling(4.9%), and Weak Encryption (2.4%). The most abused exploit? Insecure Exposed Functions (IEF), at 48.8%.

The article goes onto making the most generic recommendations any cybersec professional would know:

> 1.  Vendors 1\. Develop the depth of knowledge of your engineers in the frameworks they use 2\. Harden, standardize, and audit the use of those frameworks across products 3\. Enable and expose verbose logging for your products
> 2.  Developers 1\. Assume all code you write is reachable from an unauthenticated context 2\. Practice defense-in-depth programming and don’t make it easy for an attacker to shell out
> 3.  Defenders 1\. Reduce any attack surface exposed to the internet if its not needed there 2\. Proactively enable logging, and remote logging if possible, for all products that touch the internet
> 4.  Researchers 1\. Look for bugs in the places frameworks come together

Therefore, Rust won’t save us.

There is some truth to that, and the advice given by the article is also correct.

But it doesn’t dig into why Rust was made in the first place.

It doesn’t ask the question “Can we reduce/eliminate IEF abuse similar to how we reduced memory vulnerabilities?”

## Looking at IEF

What are Insecure Exposed Functions, exactly?

Lets take a look at the [MITRE](https://cwe.mitre.org/data/definitions/749.html) definition:

> The product provides an Applications Programming Interface (API) or similar interface for interaction with external actors, but the interface includes a dangerous method or function that is not properly restricted.
> 
> This weakness can lead to a wide variety of resultant weaknesses, depending on the behavior of the exposed method. It can apply to any number of technologies and approaches, such as ActiveX controls, Java functions, IOCTLs, and so on.
> 
> The exposure can occur in a few different ways
> 
> *   The function/method was never intended to be exposed to outside actors.
> *   The function/method was only intended to be accessible to a limited set of actors, such as Internet-based access from a single web site.

IEF is access to functions the outside world should never have had access to in the first place.

### Private by default

Lets look at an example from the MITRE page:

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

In this example, we have a Java method `removeDatabase` that will delete a database with the name specified in the parameter.

The problem is this method should never have been public. By declaring it public, the rest of the application has access to this method, even though it should be restricted.

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

Java has a few keywords for deciding the level of access the rest of the codebase should have:

*   `public`
*   `protected`
*   `private`
*   *no modifier*, where you don’t specify an access level; package-private default

Now lets take that same example and see what it would look like in Rust.

```
pub  fn  remove_database(conn:  &Connection, database_name:  &str) ->  Result<()> {
 let  mut stmt = conn.prepare(&format!("DROP DATABASE {}", database_name))?;
 stmt.execute([])?;
 Ok(())
}
```

Rust only has `pub` as a keyword for determining whether an item has a public or private scope.

By default, all of Rust code is inherently private.

In Java, if no modifier is added, Java assumes is has package-private access, which is package-level rather than item-level.

In other words, visibility control in Rust is explicit and controlled through the `pub` modifier; Java’s visibility control is implicit if no modifier is specified, allowing access control based on their location within the codebase. If a modifier is specified, then it is explicit.

In order for the Rust function to be public, we would have to declare it public:

```
pub  fn  remove_database(conn:  &Connection, database_name:  &str) ->  Result<()> {
...
}
```

This example is a simple scoping error, or laziness.

It’s easy to miss, but Rust is less likely to let you make this mistake.

“Okay, so it’s private by default, big deal. Theres other ways of improperly accessing functions and abusing them.”

### IEF in the Wild

We’re going to look at [CVE-2023-22515: Atlassian Confluence](https://attackerkb.com/topics/Q5f0ItSzw5/cve-2023-22515/rapid7-analysis) vulnerability and how we can potentially solve it.

What was the problem?

> The application insecurely exposed an endpoint, `/server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false`, that allows modification to the server’s configuration state. Setting this state to false allows an attacker to re-enter application setup and add an administrative user.
> 
> We must note that the class `com.atlassian.confluence.core.actions.ServerInfoAction` extends the class `com.atlassian.confluence.core.ConfluenceActionSupport`. This will be important during exploitation.

Rust doesn’t have inheritance, which makes it less likely to accidentally inherit unintended behavior from a parent class.

Instead, Rust offers other mechanisms for code reuse and polymorphism.

The decision to omit inheritance from Rust by its designers:

*   simplifies the language, reducing the potential for inheritance-related issues such as the “diamond problem” or “fragile base class” problem.
*   encourages composition over inheritance.
*   provides trait-based polymorphism as an alternative, allowing you to define a behavior that types can implement.

Inheritance can be useful in certain contexts, but Rust’s design philosophy prioritizes simplicity, safety, and expressiveness, leaning in favor of composition, traits, and other language features.

> We know we can leverage the XWorks2 feature of supplying HTTP parameters to call setter methods on objects. We need to identify an unauthenticated endpoint whose Action object also exposes a suitable get method that will allow us to access the application configuration.
> 
> Remembering the class `com.atlassian.confluence.core.actions.ServerInfoAction`, seen during diffing, we explore the base class it inherits from, `com.atlassian.confluence.core.ConfluenceActionSupport`.

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

> We can see this class has a getter method `getBootstrapStatusProvider` which returns the `BootstrapStatusProviderImpl` instance we are looking for.
> 
> `BootstrapStatusProviderImpl`, in turn, has a getter method `getApplicationConfig` to return the application’s configuration.

```
public  class  BootstrapStatusProviderImpl  implements  BootstrapStatusProvider, BootstrapManagerInternal {
 // ...snip...
 public  ApplicationConfiguration  getApplicationConfig() {
 return  this.delegate.getApplicationConfig();
 }
 // ...snip...
}
```

> Finally, we can see the class `com.atlassian.config.ApplicationConfig` implements the setter method `setSetupComplete`.

```
public  class  ApplicationConfig  implements  ApplicationConfiguration {
 public  synchronized  void  setSetupComplete(boolean  setupComplete) {
 this.setupComplete = setupComplete;
 }
 }
```

Rust’s approach to mutability would have helped here.

If you want setters to be mutable, then you’d need to make that explicit.

## Looking at Routing Abuse

Routing abuse tied for second with memory corruption issues.

The [example Horizon3 provided](https://www.horizon3.ai/moveit-transfer-cve-2023-34362-deep-dive-and-indicators-of-compromise/) involves a security advisory by Progress for their MOVEit Transfer application:

> Progress released a [security advisory](https://community.progress.com/s/article/MOVEit-Transfer-Critical-Vulnerability-31May2023) for their MOVEit Transfer application which detailed a SQL injection leading to remote code execution and urged customers to update to the latest version. The vulnerability, CVE-2023-34362, at the time of release was believed to have been exploited in-the-wild as a 0-day dating back at least 30 days.

> The function that extracts the `X-siLock-Transaction` header to compare its value to `folder_add_by_path` has a bug. **It will incorrectly extract headers** that end in `X-siLock-Transaction`, so an attacker can trick the function to passing the request onto the machine2.aspx by providing a header such as `xX-siLock-Transaction=folder_add_by_path` and additionally providing the correctly formatted header with our own arbitrary transaction to be executed by the machine2.aspx endpoint.

Using Rust’s [http::header::HeaderName](https://docs.rs/http/1.0.0/http/header/struct.HeaderName.html) would have caught this error since the headers passed would not have been treated as a string, but to known typed headers.

This http lib is used in almost every Rust web framework for header parsing.

The right way to do header parsing, in Rust, requires creating a `HeaderName` and using that to get the header, rather than treating header names as strings.

This is one advantage of Rust’s type system.

It’s designed to be expressive, allowing developers to express complex ideas and patterns into a concise and readable manner.

## Rusts Memory Safety

It worth noting Rust eliminates most memory corruption issues, which would take care of ~20% of vulnerabilities exploited in 2023.

This is pretty huge when you think about the reports [Microsoft](https://www.zdnet.com/article/microsoft-70-percent-of-all-security-bugs-are-memory-safety-issues/) and [Google Chrome](https://www.zdnet.com/article/chrome-70-of-all-security-bugs-are-memory-safety-issues/) have dropped, stating 70% of their vulnerabilities were memory-safety related.

Rust doesn’t even have `NULL`, which [Tony Hoare called his “One Billion Dollar Mistake”](https://www.youtube.com/watch?v=ybrQvs4x0Ps).

We wont be having NULL-pointer exceptions issues in Rust anytime soon (sorry Java).

## The hero we need

The average developer is more concerned with shipping the product now and worry about fixing bugs later than how security can be designed from the start.

Security is an afterthought in many processes, something that gets bolted on.

For security to work, it has to be there from the start.

You can’t add the egg onto your cake once you’ve baked it; you need to add it to the mixture.

Security is a process of layering many defensive techniques on top of each other in an effort to thwart attackers.

Its a constant cat and mouse game.

The hero we need isn’t Rust.

Rust wont address all vulnerabilities and magically fix them.

However, Rust has inherent qualities from its design philosophy that make it safer to use than the average language.

That is our hero.

Rust may not save us, but the ideas it embodies will.

*   Private by default
*   Immutable by default
*   Type-safety checked at compile time
*   Borrow checker and ownership model reducing memory corruption
*   Safe abstractions and idiomatic patterns that prevent common security vulnerabilities

Rust’s design philosophy is a step in the right direction.

Rust doesn’t rely on the developer to put in place all the details. It lifts responsibility from the developer so they can worry more on developing and less on safety/correctness.

Imagine using a language that prevents all these kind of vulnerabilities.

Why do we talk about around programming languages as if there isn’t a way to improve their inherent security by design as well?

Besides all the recommendations Horizon made, programming languages should also be among them.

We should expect all our languages to be safer.