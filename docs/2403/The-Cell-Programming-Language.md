<!--yml
category: 未分类
date: 2024-05-27 14:41:59
-->

# The Cell Programming Language

> 来源：[https://www.cell-lang.net/](https://www.cell-lang.net/)

## The Cell Programming Language

Cell is a very high-level embeddable language designed to implement some fairly general classes of software systems (described below) that are difficult to implement using conventional languages. For those specific types of software Cell allows the developer to work at a much higher level of abstraction, and automatically provides a number of major functionalities whose implementation in conventional languages either requires a lot of tedious, low-level and error prone coding or is not feasible at all in practice.

Cell is designed to complement and integrate with your primary language, not to replace it. Once a component of an application has been implement in Cell, the compiler will generate from it an easy-to-use C++, Java or C# class that can be integrated into an existing codebase. Support for other languages will be added in the future. The generated code is designed to integrate smoothly with hand-written code and to be non-invasive, and is also [very fast](benchmarks-relational.html). Unlike your primary language, whose choice is determined by a number of complex factors (not all of them technical, and not all of them under your control), Cell is just another tool in your toolbox, one that can be adopted gradually, with minimal risk, only when, and to the extent that, it provides a clear advantage over more conventional alternatives.

### Persistent Systems

The first class of systems Cell is specifically designed for is those that have a complex state or data model that needs to be persisted. For that purpose a new type of software artifact is provided, relational automata. They offer a number of features that have no equivalent in conventional languages, like a very flexible, entirely structural type system; deterministic, repeatable execution; the ability to use relations to store data; declarative integrity constraints (keys and foreign keys); a very robust error handling mechanism based on transactions; and orthogonal persistence.

Relational automata can be used to implement many different types of stateful software systems, but one of the things they're especially well-suited for is managing persistent data in desktop, mobile and embedded applications. Compared to an embedded SQL database, they provide a much better data model and type system, are orders of magnitude faster and fully programmable, have a smaller memory footprint, allow more control over how the data is stored, and interface smoothly with the host language, with no need to do any object/relational mapping. To learn more about them start with this introductory [example](example.html).

### Reactive systems

The second type of systems Cell is designed for is reactive systems, that is, systems that need to deal with multiple asynchronous data or event sources that trigger a cascade of changes throughout the application every time they activate, change their state or emit a value. Examples can be user input events, readings from a GPS sensor or an accelerometer, or data coming from remote computers or external hardware devices. But these data streams may also originate inside the application: internal events in many video games for example pose similar issues.

These signals vary asynchronously and independently of one another. An update can be triggered by any of them at any time, and it affects all the parts of the application state that depend on them, either directly or indirectly. This often involves propagating these changes through several layers of software, and usually that has to be done in a very specific and hard-to-get-right order to ensure that the various components of the application never read stale values. Just as importantly, sometimes updates have to be triggered by the lack of change or activity, when some signals have been idle or some conditions have been true for a certain amount of time.

Systems like these are difficult to fit into the rigid, sequential execution model of imperative languages. The traditional way to implement them involves the use of threads, observers, callbacks and timers. The resulting code is typically error prone, brittle and unnecessarily complex. In Cell these systems can be implemented using a second type of automata, reactive automata, which allow the developer to specify declaratively only the dependencies between the signals, with the compiler taking care of everything else. The [reactive automata](reactive.html) page provides a comprehensive introduction.

### Distributed systems

A longer-term and more ambitious goal of the Cell project is to provide support for a number of programming models specifically designed for components of distributed applications that need to be replicated at different network nodes (typically the server and one or more clients) and whose state must be automatically persisted and kept synchronized in the face of concurrent updates that can originate at any of the nodes.

More specifically, the goal is to have the compiler generate code for both clients and server (in different languages if need be) and automatically take care of all aspects of network programming that can be abstracted away (things like communication between clients and server, error handling and persistence) and to minimize the developer intervention when dealing with those issues (mainly concurrency) that cannot be hidden by the language and the network infrastructure.

Such programming models will be made possible by the unconventional capabilities of Cell, like orthogonal persistence and the ability to rewind and replay the execution of a program. You can read more about it [here](network-architecture.html).

### Miscellanea

If you have any question, there's the [cell-lang](https://groups.google.com/d/forum/cell-lang) Google Group. For announcements, there's [cell_lang](https://twitter.com/cell_lang) on Twitter.

The Cell compiler is free and open source software, and all source code is available on [github](https://github.com/cell-lang).

### News

July 4, 2023: [Cell to C++ compiler (version 0.7 for Linux) released](version-0.7.html).

May 22, 2020: New post: [Replacing SQLite with Cell, part 2](https://medium.com/swlh/replacing-sqlite-with-cell-part-2-a5eb6a21de2d).

May 22, 2020: New post: [Replacing SQLite with Cell, part 1](https://medium.com/swlh/replacing-sqlite-with-cell-part-1-fb5d636b5fd0).