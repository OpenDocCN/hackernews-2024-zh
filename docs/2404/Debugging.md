<!--yml
category: 未分类
date: 2024-05-27 12:50:12
-->

# Debugging

> 来源：[https://airs.com/ian/essays/debug/debug.html](https://airs.com/ian/essays/debug/debug.html)

|   |  **"The realization came over me with full force that a good part of the remainder of my life was going to be spent in finding errors in my own programs."**  |
|   | *Maurice Wilkes* |

Last changed on $Date: 2003/04/15 04:18:55 $.

A practicing programmer inevitably spends a lot of time tracking down and fixing bugs. Debugging, particularly debugging of other people's code, is a skill separate from the ability to write programs in the first place. Unfortunately, while debugging is often practiced, it is rarely taught. A typical course in debugging techniques consists merely of reading the manual for a debugger.

This essay contains my thoughts on debugging, developed over the course of my twenty years as a professional programmer. This essay is aimed at people who already have some programming experience.

* * *

Debugging means removing bugs from programs. A bug is unexpected and undesirable behaviour by a program.

Occasionally there is a formal specification that the program is required to follow, in which case a bug is a failure to follow the spec. More frequently the program specification is informal, in which case people may disagree as to whether a particular program behaviour is in fact a bug or not.

As one of the people writing a program, you should be a source of bug reports. Don't simply rely on testers or users. If you notice something odd while running a program, it can be tempting to disregard it and hope that it will go away. This is especially true if you are working on something else at the time. Resist the temptation. Record the bug. If you have data files or logs, save them. If you have time later, return to the issue; otherwise, pass it on to somebody else. As one of the people most familiar with how the program is supposed to work, you are in the best possible position for detecting unexpected behaviour.

Once you have a bug report, the first step in removing the bug is identifying it. This is of particular importance when working with a bug report produced by somebody else, such as the testing team or a user. Some bugs are relatively obvious, as when the program crashes unexpectedly. Others are obscure, as when the program generates output which is slightly incorrect.

Many bug reports received from users are of the form "I did such and such, and something went wrong." Before doing anything else, you must find out what went wrong--that is, you must identify the bug by determining the program behaviour which was unexpected and undesirable. Any attempt to fix the bug before understanding what went wrong is generally wasted time.

Identifying a bug reported by a user typically requires getting the answer to two questions: "What did the program do?" and "What did you expect the program to do?" The goal is to determine precisely the behaviour of the program which was unexpected and undesirable.

Once the bug has been identified, the easiest and fastest way to fix it is to determine that it is not a bug at all. If there is a formal specification for the program, you may have to modify the specification. In other cases, you may have to modify the expectations of the user. This rapid fix is often known as declaring the behaviour to be an "undocumented feature." Despite the obvious potential for abuse, this is in fact sometimes the correct way to handle the problem.

Unfortunately, most bugs are real bugs, and require further work.

* * *

The first step in fixing a bug is to replicate it. This means to recreate the undesirable behaviour under controlled conditions. The goal is to find a precisely specified set of steps which demonstrate the bug.

In many cases this is straightforward. You run the program on a particular input, or you press a particular button on a particular dialog, and the bug occurs. In other cases, replication can be very difficult. It may require a lengthy series of steps, or, in an interactive program such as a game, it may require precise timing. In the worst cases, replication may be nearly impossible.

* * *

Programmers are sometimes tempted to skip the replication step, and to go straight from the bug report to the fix. However, failing to replicate the bug means that it is impossible to verify the fix. The fix may turn out to be for a different bug, or it may have no significant effect at all. If the bug has not been replicated, there is no way to tell.

Failure to replicate the bug is a real problem which I've seen happen many times. In a complex program, it's often quite easy to find something to fix--perhaps a real problem, perhaps merely something which looks like a problem. It's human nature to assume that any particular fix solves the problem at hand. Without a means of verification, any plausible fix will be accepted, whether it is right or wrong. An incorrect fix leads to future problems. On average, skipping the replication step wastes more time than it saves.

* * *

By far the best way to replicate the bug is on a system wholly under your own control, with a copy of the program you built yourself. Replicating the bug on your own system means that you can easily test a patched version of the program. Unfortunately, in some cases this is impossible.

One reason that replication on your system may be impossible is that the user reporting the bug may have a unique system configuration, or the user may be using private input files which you can not obtain. In these cases you must try to ensure that the user can reliably replicate the bug, and that the user can test the patched program. Without this help from the user, debugging is reduced to little more than guessing. It's generally not worth proceeding. If that is not an option, then you should at least try to construct some sort of test case for any possible patch, to confirm that the patch does do something useful even if you can't verify that it fixes the original bug.

A more common problem is that the user can demonstrate the bug, but you can not replicate it on your own system, and you don't know why. There is no definitive way to solve this problem, but here are some of the possibilities to consider.

*   The user may simply be misreporting the problem, or the way in which it is replicated. Go over it again. Confirm what the user actually types and where the user actually clicks. Confirm what the user sees on the screen. Try not to make any assumptions about how the program is being run--the user may be doing something which is so off the wall that you would never consider it.

*   Some programs have odd behaviour under unusual conditions. Check whether the user's system is very low on disk space, or has bad network connectivity, or is very overloaded. Check whether any other programs are failing unexpectedly--hardware problems are rare, but they do happen. Check for messages in the system logs.

*   Check the version of the software the user is running. Check the versions of the operating system and any system libraries. Check which system patches have been applied. Check the exact system type, monitor type, keyboard type, etc.

In some cases a bug may happen rarely and apparently randomly, so that it is very difficult to figure out what triggers it, and how to replicate it. In this frustrating situation, I only know of one reliable approach, which is to automatically log all potentially relevant events, and save the logs when the bug occurs. In some cases it's particularly useful to be able to replay the program using the log, which can be a powerful mechanism for replicating a bug. Of course this requires a logging facility, and adding one to a large program can be a large undertaking. Nevertheless, it's probably worth doing the second time you find yourself searching for a bug which appears to occur randomly.

* * *

Once you are able to replicate the bug, you must figure out what causes it. This is generally the most time-consuming step.

* * *

As a practicing programmer, you are probably familiar with the tool called a debugger. The name of this tool makes it seem like the tool of choice for debugging--it seems logical that in order to debug you would use a debugger. However, this is inaccurate. As I said initially, debugging means removing bugs from programs; while a debugger is a powerful tool, one thing it does not do is remove bugs. A better name for the tool would be "inspector". (As it is unfortunately too late to rename the tool, I will continue referring to it as a debugger.)

In order to fix a bug, you must understand it. A debugger can sometimes help you understand a bug, but it is only one of several tools for that purpose. Only in certain specific circumstances should it be the first tool you reach for.

* * *

In order to understand a bug in a program, you must have some understanding of the program.

If you wrote the program, then you presumably understand it. If not, then you have more serious problems. [[1]](#FTN.AEN75)

If you didn't write the program, you need to grasp its general structure. Most programs are organized in a fairly sensible fashion, once you know the general approach. If you are lucky, the general approach is documented, or you can ask the original designer.

More commonly, you need to pull the structure out of the source code. The best approach is to start looking at the source code from the start of the program (e.g., the `main` function in a C program). Skim through the program, stepping down through functions, until you find the main center of action--in most programs, some sort of loop. This can normally be done fairly quickly. The nature of this center of action should tell you where to look in the source code for any particular activity. It should also tell you the general way in which the program acts.

The worst cases are large programs written over many years by many different people. These often become a hodge-podge of different ideas with little consistency. The situation is depressingly common. You must simply do the best you can. At least try to avoid making the mess worse.

When examining a large program, some people find it helpful to use a source code analysis tool, such as [Source Navigator](https://sourcenav.sourceforge.net/). Personally, I normally rely simply on Emacs tags tables.

A debugger can also be helpful when trying to understand a program. By running the program under the debugger and setting breakpoints, you may be able to see the dynamic behaviour of the program. When you reach a breakpoint, look at the call stack to see how you got there, and look at key variables. Or if you don't reach a breakpoint you expected to reach, you've also learned something.

* * *

The next step is to locate the bug in the program source code.

There are two source code locations which you need to consider: the code which causes the visible incorrect behaviour, and the code which is actually incorrect. It's fairly common for these to be the same pieces of code. However, it's also fairly common for these to be in different parts of the program. A typical example of this is when an error in one part of the program causes memory corruption which leads to visible bad behaviour in a completely different part of the program. Do not let your eagerness to fix the bug mislead you into thinking that the code which directly causes the bad behaviour is actually incorrect.

Ordinarily you must first find the code which causes the incorrect behaviour. Knowing the incorrect behaviour, and knowing how the source code is arranged, will often lead you quickly to the part of the program which is at fault. Sometimes a quick scan of the source code is enough to identify the problematic code.

Otherwise, narrowing down the bad behaviour to a particular piece of code is where a debugger can be very useful. If you are lucky enough to have a core dump, a debugger can immediately identify the line which fails. Otherwise, judiciously setting breakpoints while replicating the bug can quickly hone in on the code you are after.

Modern debuggers, such as gdb, have powerful capabilities to make this process more manageable, such as conditional breakpoints, data watchpoints, ignoring breakpoints certain numbers of time, and support for executing simple code at breakpoints. These features are very useful when locating bad behaviour in code which is executed many times, and only fails under particular circumstances. It's a good idea to learn the features which your debugger provides.

Of course, sometimes a debugger does not help. In some cases, a bug will not occur when the program is run under a debugger, even though it can be reliably replicated without the debugger; this typically indicates a problem which depends upon precise timing or memory layout. In other cases, you may not have access to a debugger, or the debugger may not be very powerful; this can happen when working on embedded systems or other impoverished programming environments, or when using programming features which are not supported by the debugger (for example, in some environments, threads).

In such cases simple print statements can sometimes help locate the source of the bad behaviour. Add print statements to relevant locations, rebuild the program, replicate the problem with the new program, and use the print statements to hone in on exactly what code is being executed when the problem occurs. Ideally the program already has some sort of logging facility which you can reuse. Even if it doesn't, you should consider a systematic approach to the print statements you add, so that you can build on that when doing future debugging of the same program. In particular, every print statement should clearly indicate where it is located in the program, so that it can be quickly found again later.

Another useful approach is to add check routines to the code to verify that data structures are in a valid state. Such routines can help narrow down where data corruption occurs. If the check routines are fast, you may want to always enable them. Otherwise, leave them in the code, and provide some sort of mechanism to turn them on when you need them.

In the specific case of a memory corruption bug, you may be able to replace the standard memory allocation routines with ones that perform various checks. For example, on GNU/Linux systems, read the `malloc` documentation to see how the environment variable `MALLOC_CHECK_` can be used to do this.

The final fallback for locating the source of the bad behaviour is simple source code inspection. This is the only option if you can't replicate the problem. A clear understanding of the overall program source code is an absolute requirement for this to work. Unfortunately, a complex problem is nearly impossible to isolate by simply reading the source code. You will have to guess at likely possibilities, and try to trace through the code carefully to see if they are really problems.

If you are very unlucky, the bug may not be in the program source code at all. It may be in a library routine, or in the operating system, or in the compiler. These cases are rare, and it is a mark of an inexperienced programmer to suspect a compiler bug too quickly. However, they do happen--they've all happened to me--so when all else fails, consider these possibilities. Verify a bug in a library routine or the OS by writing a check program, and verify a bug in the compiler by examining the machine code directly.

* * *

Now that you have found the code which causes the bad behaviour, you need to identify the actual coding error. Often they are the same code--that is, the coding error directly causes the bad behaviour. However, you should always consider the possibility that the actual error is elsewhere.

For example, the routine which causes the bad behaviour may be behaving correctly, but be called with bad input, or at the wrong time. A coding error elsewhere may cause a data structure to hold unreasonable values. Another possibility is bad user input.

The fix in such cases may be two-fold. You should, of course, fix the code which called the routine incorrectly or otherwise created the bad input data. In the case of bad user input, you should validate the input. In addition, however, you may want to add checks to the code which used the values. It should check for unreasonable input, and report an error or otherwise handle the error without causing invalid behaviour.

* * *

The final step in the debugging process is, of course, to fix the bug. I won't discuss this step in detail, as fixing a bug is where you leave the debugging phase and return to programming. I'll just mention a couple of points.

If you want a program which can be maintained in the future, then make sure you fix the bug in the right way. This means making a fix which fits in with the rest of the program, and which fixes all aspects of the problem, without introducing any new problems. Don't forget to update any relevant documentation.

In some cases you may need a quick patch to fix an immediate problem. There is nothing wrong with doing that, as long as you take the time afterward to go back and make the right fix.

Obviously, always test any fix you make by ensuring that you can no longer replicate the bad behaviour. Don't forget to make sure that the program continues to pass its test suites. Consider extending the test suites to detect the case which you just fixed, to make sure it doesn't reappear.

* * *

After you fix the bug, consider whether there is anything you can learn from it. Here are some things to think about:

*   Does the same programming error occur anywhere else in the program?

*   What new problems might be introduced by the bug fix?

*   How could this error have been prevented? What could you have done differently such that the bug was not created in the first place? Even some excellent programmers repeatedly make characteristic errors; are there types of programming where you need to exercise particular care?

*   If you didn't find the bug yourself, could it have been detected sooner? Do your testing procedures need improvement? If you have not already done so, can you automate the testing? Would it have been possible to use automated code inspection tools to find the error?

* * *

This essay incorporates suggestions from Jonathan Mark.

* * *