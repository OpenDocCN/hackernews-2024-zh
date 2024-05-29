<!--yml
category: 未分类
date: 2024-05-27 14:56:28
-->

# Inner-platform effect - Wikipedia

> 来源：[https://en.wikipedia.org/wiki/Inner-platform_effect](https://en.wikipedia.org/wiki/Inner-platform_effect)

Tendency of software architects to replicate their development platform

The **inner-platform effect** is the tendency of software architects to create a system so customizable as to become a replica, and often a poor replica, of the software development platform they are using. This is generally inefficient and such systems are often considered to be examples of an [anti-pattern](/wiki/Anti-pattern "Anti-pattern").

## Examples[[edit](/w/index.php?title=Inner-platform_effect&action=edit&section=1 "Edit section: Examples")]

Examples are visible in [plugin](/wiki/Plug-in_(computing) "Plug-in (computing)")-based software such as some [text editors](/wiki/Text_editor "Text editor") and [web browsers](/wiki/Web_browser "Web browser") which often have developers create plugins that recreate software that would normally run on top of the operating system itself. The [Firefox](/wiki/Firefox "Firefox") add-on mechanism has been used to develop a number of [FTP](/wiki/File_Transfer_Protocol "File Transfer Protocol") clients and [file browsers](/wiki/File_browser "File browser"), which effectively replicate some of the features of the [operating system](/wiki/Operating_system "Operating system"), albeit on a more restricted platform.

In the [database](/wiki/Database "Database") world, developers are sometimes tempted to bypass the [RDBMS](/wiki/RDBMS "RDBMS"), for example by storing everything in one big [table](/wiki/Table_(database) "Table (database)") with three [columns](/wiki/Column_(database) "Column (database)") labelled entity ID, key, and value. While this [entity-attribute-value model](/wiki/Entity-attribute-value_model "Entity-attribute-value model") allows the developer to break out from the structure imposed by an [SQL](/wiki/SQL "SQL") database, it loses out on all the benefits,^([[1]](#cite_note-1)) since all of the work that could be done efficiently by the RDBMS is forced onto the application instead. Queries become much more convoluted,^([[2]](#cite_note-2)) the [indexes](/wiki/Index_(database) "Index (database)") and [query optimizer](/wiki/Query_optimizer "Query optimizer") can no longer work effectively, and [data validity constraints](/wiki/Data_validation "Data validation") are not enforced. Performance and maintainability can be extremely poor.

A similar temptation exists for [XML](/wiki/XML "XML"), where developers sometimes favor generic element names and use attributes to store meaningful information. For example, every element might be named *item* and have attributes *type* and *value*. This practice requires [joins](/wiki/Join_(relational_algebra) "Join (relational algebra)") across multiple attributes in order to extract meaning. As a result, [XPath](/wiki/XPath "XPath") expressions are more convoluted, evaluation is less efficient, and structural validation provides little benefit.

Another example is the phenomenon of [web desktops](/wiki/Web_desktop "Web desktop"), where a whole [desktop environment](/wiki/Desktop_environment "Desktop environment")—often including a [web browser](/wiki/Web_browser "Web browser")—runs inside a browser (which itself typically runs within the desktop environment provided by the [operating system](/wiki/Operating_system "Operating system")). A desktop within a desktop can be unusually awkward for the user, and hence this is generally only done to run programs that cannot easily be deployed on end user systems, or by hiding the outer desktop away.

It is normal for software developers to create a library of custom functions that relate to their specific project. The inner-platform effect occurs when this library expands to include general purpose functions that duplicate functionality already available as part of the programming language or platform. Since each of these new functions will generally call a number of the original functions, they tend to be slower, and if poorly coded, less reliable as well.^([*[citation needed](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation needed")*])

On the other hand, such functions are often created to present a simpler (and often more portable) abstraction layer on top of lower level services that either have an awkward interface, are too complex, non-portable or insufficiently portable, or simply a poor match for higher level application code.

## Appropriate uses[[edit](/w/index.php?title=Inner-platform_effect&action=edit&section=3 "Edit section: Appropriate uses")]

An inner platform can be useful for portability and privilege separation reasons—in other words, so that the same application can run on a wide variety of outer platforms without affecting anything outside a [sandbox](/wiki/Sandbox_(computer_security) "Sandbox (computer security)") managed by the inner platform. For example, Sun Microsystems designed the [Java platform](/wiki/Java_(software_platform) "Java (software platform)") to meet both of these goals.

## See also[[edit](/w/index.php?title=Inner-platform_effect&action=edit&section=4 "Edit section: See also")]

## References[[edit](/w/index.php?title=Inner-platform_effect&action=edit&section=5 "Edit section: References")]

## External links[[edit](/w/index.php?title=Inner-platform_effect&action=edit&section=6 "Edit section: External links")]