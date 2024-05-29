<!--yml
category: 未分类
date: 2024-05-29 12:29:48
-->

# Java 22 / JDK 22: General Availability

> 来源：[https://mail.openjdk.org/pipermail/jdk-dev/2024-March/008827.html](https://mail.openjdk.org/pipermail/jdk-dev/2024-March/008827.html)

# Java 22 / JDK 22: General Availability

**Mark Reinhold** [mark.reinhold at oracle.com](mailto:jdk-dev%40openjdk.org?Subject=Re%3A%20Java%2022%20/%20JDK%2022%3A%20General%20Availability&In-Reply-To=%3C20240319132004.DAD846C4EC2%40eggemoggin.niobe.net%3E "Java 22 / JDK 22: General Availability")
*Tue Mar 19 13:20:07 UTC 2024*

* * *

```
JDK 22, the reference implementation of Java 22, is now Generally
Available.  We shipped build 36 as the second Release Candidate of
JDK 22 on 16 February, and no P1 bugs have been reported since then.
Build 36 is therefore now the GA build, ready for production use.

GPL-licensed OpenJDK builds from Oracle are available here:

  https://jdk.java.net/22

Builds from other vendors will no doubt be available soon.

This release includes twelve JEPs [1], including the final versions of
the Foreign Function & Memory API (454) and Unnamed Variables & Patterns
(456):

  423: Region Pinning for G1
  447: Statements before super(...) (Preview)
  454: Foreign Function & Memory API
  456: Unnamed Variables & Patterns
  457: Class-File API (Preview)
  458: Launch Multi-File Source-Code Programs
  459: String Templates (Second Preview)
  460: Vector API (Seventh Incubator)
  461: Stream Gatherers (Preview)
  462: Structured Concurrency (Second Preview)
  463: Implicitly Declared Classes and Instance Main Methods (Second Preview)
  464: Scoped Values (Second Preview)

This release also includes, as usual, hundreds of smaller enhancements
and thousands of bug fixes.

Thank you to everyone who contributed this release, whether by designing
and implementing features or enhancements, by fixing bugs, or by
downloading and testing the early-access builds!

- Mark

[1] https://openjdk.org/projects/jdk/22/

```

* * *

* * *

[More information about the jdk-dev mailing list](https://mail.openjdk.org/mailman/listinfo/jdk-dev)