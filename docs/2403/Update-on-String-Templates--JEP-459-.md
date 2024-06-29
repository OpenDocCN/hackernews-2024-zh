<!--yml

category: 未分类

date: 2024-05-27 14:46:25

-->

# 更新关于字符串模板（JEP 459）

> 来源：[https://mail.openjdk.org/pipermail/amber-spec-experts/2024-March/004010.html](https://mail.openjdk.org/pipermail/amber-spec-experts/2024-March/004010.html)

# 更新关于字符串模板（JEP 459）

**Brian Goetz** [brian.goetz at oracle.com](mailto:amber-spec-experts%40openjdk.org?Subject=Re%3A%20Update%20on%20String%20Templates%20%28JEP%20459%29&In-Reply-To=%3C8EF82E14-DDF9-48A2-8CAE-7E7DC3C2AC9F%40oracle.com%3E "更新关于字符串模板（JEP 459）")

*2024年3月8日 UTC时间18:35:03*

* * *

```
Time to check in with where were are with String Templates.  We’ve gone through two rounds of preview, and have received some feedback.

As a reminder, the primary goal of gathering feedback is to learn things about the design or implementation that we don’t already know.  This could be bug reports, experience reports, code review, careful analysis, novel alternatives, etc.    And the best feedback usually comes from using the feature “in anger” — trying to actually write code with it.  (“Some people would prefer a different syntax” or “some people would prefer we focused on string interpolation only” fall squarely in the “things we already knew” camp.)

In the course of using this feature in the `jextract` project, we did learn quite a few things we didn’t already know, and this was conclusive enough that it has motivated us to adjust our approach in this feature.  Specifically, the role of processors is “outsized” to the value they offer, and, after further exploration, we now believe it is possible to achieve the goals of the feature without an explicit “processor” abstraction at all!  This is a very positive development.

First, I want to affirm that that the goals of the project have not changed.  From JEP 459:

Goals

• Simplify the writing of Java programs by making it easy to express strings that include values computed at run time.
• Enhance the readability of expressions that mix text and expressions, whether the text fits on a single source line (as with string literals) or spans several source lines (as with text blocks).
• Improve the security of Java programs that compose strings from user-provided values and pass them to other systems (e.g., building queries for databases) by supporting validation and transformation of both the template and the values of its embedded expressions.
• Retain flexibility by allowing Java libraries to define the formatting syntax used in string templates.
• Simplify the use of APIs that accept strings written in non-Java languages (e.g., SQL, XML, and JSON).
• Enable the creation of non-string values computed from literal text and embedded expressions without having to transit through an intermediate string representation.

Non-Goals
• It is not a goal to introduce syntactic sugar for Java's string concatenation operator (+), since that would circumvent the goal of validation.
• It is not a goal to deprecate or remove the StringBuilder and StringBuffer classes, which have traditionally been used for complex or programmatic string composition.

Another thing that has not changed is our view on the syntax for embedding expressions.  While many people did express the opinion of “why not ‘just' do what Kotlin/Scala does”, this issue was more than fully explored during the initial design round.  (In fact, while syntax disagreements are often purely subjective, this one was far more clear — the $-syntax is objectively worse, and would be doubly so if injected into an existing language where there were already string literals in the wild.  This has all been more than adequately covered elsewhere, so I won’t rehash it here.)

Now, let’s talk about what we do think should change: the role of processors and the StringTemplate type.

Processors were envisioned as a means to abstract the transformation of templates to their final form (whether string, or something else.)  However, Java already has a well established means of abstracting behavior: methods.   (In fact, a processor application can be viewed as merely a new syntax for a method call.)  Our experience using the feature highlighted the question: When converting a SQL query expressed as a template to the form required by the database (such as PreparedStatement), why do we need to say:

  DB.”… template …”

When we could use an ordinary Java library:

  Query q = Query.of(“…template…”)

Indeed, one of the worst things about having processors in the language is that API designers are put in the difficult situation of not knowing whether to write a processor or an ordinary API, and often have to make that choice before the consequences are fully understood.  (To add to this, processors raise similar questions at the use site.) But the real criticism here is that template capture and processing are complected, when they should be separate, composable features.

This motivated us to revisit some of the reasons why processors were so central to the initial design in the first place.  And it turned out, this choice had been influenced — perhaps overly so — by early implementation experiments.  (One of the background design goals was to enable expensive operations like `String::format` to be (much) cheaper.  Without digressing too deeply on performance, String::format can be more than an order of magnitude worse than the equivalent concatenation operation, and this in turn sometimes motivates developers to use worse idioms for formatting.  The FMT processor brough that cost back in line with the equivalent concatenation.)  These early experiments biased the design towards needing to know the processor at the point of template capture, but upon reexamination we realized that there are other ways to achieve the desired performance goals without requiring processors to be known at capture time.  This, in turn, enabled us to revisit a point in the design space we had transited through earlier, where string templates were “just a new kind of literal” and the job performed by processors could instead be performed by ordinary APIs.

At this point, a simpler design and implementation emerged that met the semantic, correctness, and performance goals: template literals (“Hello \{name}”) are simply the literal form of StringTemplate:

  StringTemplate st = “Hello \{name}”;

String and StringTemplate remain unrelated types.  (We explored a number of ways to interconvert them, but they caused more trouble than they solved.)  Processing of string templates, including interpolation, is done by ordinary APIs that deal in StringTemplate, aided by some clever implementation tricks to ensure good performance.

For APIs where interpolation is known to be safe in the domain, such as PrintWriter, APIs can make that choice on behalf of the domain, by providing overloads to embody this design choice:

   void println(String) { … }
   void println(StringTemplate) { … interpolate and delegate to println(String) …. }

The upshot is that for interpolation-safe APIs like println, we can use a template directly without giving up any safety:

   System.out.println(“Hello \{name}”);

In this example, the string template evaluates to StringTemplate, not String (no implicit interpolation), and chooses the StringTemplate overload of println, which in turn chooses how to process the template.  This stays true to the design principle that interpolation is dangerous enough that it should be an explicit choice in the code — but it allows that choice to be made by libraries when the library is comfortable doing so.

Similarly, the FMT processor is replaced by an overload of String::format that interprets templates with embedded format specifiers (e.g., “%d”):

  String format(String formatString, Object… parameters) { … same as today … }
  String format(StringTemplate template) {... equivalent of FMT ...}

And users can call this as:

  String s = String.format(“Hello %12s\{name}”);

Here, the String::format API has chosen to interpret string templates according to the rules previously specified in the FMT processor (not ordinary interpolation), but that choice is embedded in the library semantics so no further explicit choice at the use site is required.  The user already chose to pass it to String::format; that’s all the processing selection that is needed.

Where APIs do not express a choice of what template expansion means, users continue to be free to process them explicitly before passing them, using APIs that do (such as String::format or ordinary interpolation.).

The result is:

- The need for use-site "goop" (previously, the processor name; now, static or instance methods to process a template) goes away entirely when dealing with libraries that are already template-friendly.
- Even with libraries that require use-site goop, it is no more intrusive than before, and can be reduced over time as APIs get with the program.
- StringTemplate is just another type that APIs can support if they want.  The "DB" processor becomes an ordinary factory method that accepts a string template or an ordinary builder API.
- APIs now can have _more_ control over the timing and meaning of template processing, because we are not biasing so strongly towards early processing.
- It becomes easier to abstract over template processing (i.e., combine or manipulate templates as templates before processing)
- Interpolation remains an explicit choice, but ST-aware libraries can make this choice on behalf of the user.
- The language feature and API surface get considerably smaller, which is good.  Core JDK APIs (e.g., println, format, exception constructors) get upgraded to work with string templates.

The remaining question that everyone is probably asking is: “so how do we do interpolation.”  The answer there is “ordinary library methods”.  This might be a static method (String.join(StringTemplate)) or an instance method (template.join()), shed to be painted (but please, not right now.).

This is a sketch of direction, so feel free to pose questions/comments on the direction.  We’ll discuss the details as we go.

-------------- next part --------------
An HTML attachment was scrubbed...
URL: <https://mail.openjdk.org/pipermail/amber-spec-experts/attachments/20240308/72bafa90/attachment-0001.htm>

```

* * *

* * *

[关于琥珀规范专家邮件列表的更多信息](https://mail.openjdk.org/mailman/listinfo/amber-spec-experts)
