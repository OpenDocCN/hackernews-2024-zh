<!--yml
category: 未分类
date: 2024-05-27 14:33:58
-->

# Interlisp-D and MIT CADR Lisp Machine demos for Vancouver IJCAI Conference - Tape #1 : Beau Sheil, Johan de Kleer : Free Download, Borrow, and Streaming : Internet Archive

> 来源：[https://archive.org/details/xerox-parc_V-141_1](https://archive.org/details/xerox-parc_V-141_1)

*Simplified ELI5 Abstract*

**Imagine a special computer called Xerox D**

* **Connected like friends:** It can talk to other computers, just

like friends share toys.

* **Fancy screen:** It has a super-clear screen and a mouse for

pointing, making it fun to use.

**This computer also teaches you to code**

* **Like building blocks:** You can edit code by moving pieces around,

almost like playing with building blocks.

* **Changing names:** It's easy to change the names of parts in your code, like renaming your Lego pieces.

* **Testing your work:** You can make pictures and test if your code

changes work as they should.

**Making Mistakes is Okay!**

* **Whoops!** If your code has a boo-boo, the computer shows you where

the problem is, like a teacher!

* **Fixing things:** You can fix mistakes and even ask the computer to

show you what you changed.

**Cool Tricks**

* **Windows like picture frames:** The computer can show you lots of

things at once, each in its own window.

* **Sending secret notes:** You can send messages to other computers,

like passing notes in class.

* **Computer detective:** You have a special tool to look inside the

computer and see how it works, like a detective with a magnifying

glass.

Absolutely! Here's an abstract of the video transcript, based on your

summaries:

**Abstract**

This video offers a detailed demonstration of the Xerox D personal

computing environment and the Interlisp-D programming language. It

covers several key themes:

* **System Overview:** The introduction outlines D's network

capabilities, graphical display, and interactive nature, contrasting

with traditional timesharing systems.

* **Code Modification and Renaming:** The video demonstrates

refactoring techniques in Interlisp-D. This includes renaming fields

for clarity and using the system's analysis tools to understand the

impact of these changes.

* **Visual Editing:** The speaker utilizes Interlisp-D's structured

editor to directly modify code structure, specifically implementing

a conditional based on node positions within a tree editing

program. The visual editing interface allows for convenient

manipulation of code blocks.

* **Error Handling:** The system's comprehensive error handling is

showcased. When an error occurs, it displays extensive debugging

information, including the executing code, argument values, local

variables, and the call stack. This facilitates inspection and the

ability to manually fix execution state.

* **Graphics Programming:** The video explores how graphical output is

handled, using the `draw rectangle` message. A simple bar graph is

drawn by modifying a factorial function, and a dedicated output

window is created to separate graphics from text.

* **Debugging with the Inspector:** The Inspector tool allows in-depth

examination of Lisp data structures. The video demonstrates

inspecting a terminal window, revealing its data components and the

messages it can handle. The ability to modify object properties,

like the blinker's visual behavior, is also shown.

**Overall, the video provides a comprehensive look at Interlisp-D's

features, highlighting its integrated programming environment,

powerful editing tools, and robust error handling mechanisms.**

*Summary*

**00:00:00 Introduction**

* Xerox D is introduced as a personal computing environment with

networking capabilities.

* It features a high-resolution bitmap display, mouse, and offers a

more interactive experience than traditional systems

**00:02:54 Exploring and Editing Programs**

* Xerox D includes the Interlisp-D programming language and powerful

development tools.

* The video will demonstrate how to analyze and change code using a

tree editing program example.

**00:12:41 Renaming Fields for Clarity**

* The speaker observes changes in code behavior, needing field names

to be updated for better clarity.

* They use a systematic command to rename 'from pause' and 'to pause'

fields to 'bottom' and 'top', respectively.

**00:17:22 Editing Code with the Display-Based Editor**

* The speaker uses Interlisp-D's visual editor to directly modify the

'display link' function.

* The editor provides a structured view of the code and convenient

editing commands.

**00:23:17 Testing the Modification**

* Changes to the 'display link' function are tested in the tree

editing program, visually confirming the fix.

* The system summarizes what was modified.

**00:26:29 Window Management**

* The Cater machine's window system is explained. Processes can

perform input/output within individual windows.

* Demonstration of splitting the screen into multiple windows (Lisp

evaluator, editor, statistics)

* The 'blinker' indicates the active window.

**00:32:38 Sending Messages Over Ethernet**

* Explanation of using the Ethernet for communication between

computers.

* The speaker sends a message to themselves as a demonstration.

**00:33:44 Programming Environment: Editor and Error Handling**

* The editor understands Lisp syntax and integrates closely with the

programming environment.

* 'Meta point' is used to quickly find source code for functions

* Introduction of namespaces (

[e.g](http://e.g)

., 'ether:')

* Demonstration of intentional error introduction into a factorial

function.

* Compilation and error handling: The error handler uses the display

to show executing code, arguments, variables, and the call stack.

**00:39:24 Error Handling (Cont.)**

* Details about panes in the error view are provided.

* The user can interact with and inspect values during an error

state.

* Demonstration of navigating the call stack and manually exiting

the error with a return value.

**00:42:55 Programming with Graphics**

* The 'factorial' function is modified to draw a bar graph.

* Graphics use the 'draw rectangle' message sent to the current

window.

* 'ctrl shift D' is used to find the correct arguments for 'draw

rectangle'.

* A new window ('fact window') separates graphical output.

**00:48:49 System Debugging with the Inspector**

* Introduction of the Inspector tool for examining Lisp data

structures.

* The Inspector's panes include a REPL, object history, command menu,

and recently inspected objects.

* Demonstration of inspecting a terminal I/O window.

* Explanation that windows are objects and can have methods in

addition to data.

* The speaker views window message handlers.

* Demonstration of using the Inspector to change a window's 'blinker'

properties.

Disclaimer: I used gemini advanced 1.0 to create this based on the

transcript.