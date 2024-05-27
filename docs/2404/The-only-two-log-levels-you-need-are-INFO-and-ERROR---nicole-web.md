<!--yml
category: 未分类
date: 2024-05-27 13:39:29
-->

# The only two log levels you need are INFO and ERROR | nicole@web

> 来源：[https://ntietz.com/blog/the-only-two-log-levels-you-need-are-info-and-error/](https://ntietz.com/blog/the-only-two-log-levels-you-need-are-info-and-error/)

Logging is a critical tool for maintaining any web application, and yet we're getting it wrong.

With great logs, you can see what your application is doing. And without them? Things can be broken left and right without you ever finding out. Instead, you wonder why your customers don't come back, and shrug, and blame someone other than engineering.

Unfortunately, it's common for us to log in ways that are unhelpful. Log levels are inconsistent, and logs are added to fix bugs then removed afterwards. But come on, you saw the title, this is about the log levels, mostly.

# The typical log levels

Most languages and logging libraries have a handful of log levels, at least five. But they vary! Here are three examples:

*   Rust's `tracing` has [five log levels](https://docs.rs/tracing/latest/tracing/struct.Level.html): ERROR, WARN, INFO, DEBUG, and TRACE.
*   Python's `logging` also has [five log levels](https://docs.python.org/3/howto/logging.html): CRITICAL, ERROR, WARNING, INFO, and DEBUG.
*   The infamous `log4j` has [six log levels](https://logging.apache.org/log4j/1.x/apidocs/org/apache/log4j/Level.html): FATAL, ERROR, WARN, INFO, DEBUG, and TRACE.

Three examples and three wholly different sets of log levels. They all function in about the same way: you log certain information at different levels, based on your ideas of "severity" and "granularity", and then as you get toward the fatal/error end of the log levels you see only the most critical alerts, and toward the other end you see *everything* to help you debug your application.

There's really no typical set of log levels, nor common cross-language guidance on what to log at which level. There can't be that advice, because the levels vary so much!

# What do we do with logs?

When we add log statements, that's typically because we think that we will need those logs sometime in the future. We think they'll help us debug something, or help us audit that it happened, or discover a critical error.

Think about a time when you went to debug a deployed web application (not on your local machine) and used the logs. Typically, we're doing a few things:

*   **Discovering errors to fix.** We want to know when something has gone wrong, so we look for errors in the logs. Typically, you also want to get alerted if something needs to be fixed. And related, you *don't* want to be alerted if something happens that doesn't deserve attention. Alert fatigue is a serious problem.
*   **Debugging a problem.** When something does go wrong, whether you were alerted from logs or not, you'll turn to them to understand what's up. The logs can tell you what's happening that's unexpected. They should include any stack traces, related errors, and conditions leading up to the problem.
*   **Understanding usage.** Sometimes logs are good for understanding how things are used! You can see generally which portions of the application are accessed or left untouched. They're not a substitute for metrics or distributed tracing, though!
*   **Understanding how it works.** Logs can also help you understand how a system works! If you have just the code, you'll see a lot of functions and request handlers and data and where do you start? With the logs, you can generally find an entry point where a request or session starts. From there, you can follow along with those logs through the life cycle of the session and follow the execution!

These can be split into two categories: logs that wake me up, and logs that help me fix things.

For logs that wake me up, I want to know a few things. What's the problem *precisely*? When did it happen? And where can I find related logs?

And then for logs that help me fix things, I really want to know everything. Well, when debugging, everything relevant, I don't need to go down a rabbit hole about CNCs right *then*. And you don't know what's going to be relevant until you know the answer to your question! You might have a guess, but it's a guess.

# The levels you need

In practice, I tend to find that you only really want two log levels: ERROR and INFO. That's because we really do only care if something should alert us or not. For all the other uses of logs, we want to see *all* the context.

Let's use WARNING as an example. Suppose you *should* have a WARNING log level, and some of those show up in your logs. What should you do when you see them? There are three choices:

*   If the information isn't useful at *all*, in any scenario, delete them entirely!
*   If it helps you make sense of other logs, potentially related to an error, then these are just info logs for debugging! They should be INFO level to reflect that.
*   And if it is an error you need to fix, well, it's not a warning now is it? Make that an ERROR log.

And similar for things like DEBUG or TRACE. Usually you reach for those log levels when you want to see extremely verbose information, but... That's not practical to enable in production environments, because the volume of logs you produce would be way too high. And if you can't use it in production, you probably shouldn't use it elsewhere! That's what your local debugger is for.

Things like FATAL? Yep, that's *also* an ERROR log, because you do want to be alerted on it!

In every situation that I've run into in production, a single log line at a given level is never sufficient to debug something, only to know it happened. And even then, what actually happened? Shrug. Let's look at more log lines.

And so if you separate things out into different log levels, you end up looking at all of them together *anyway*, because to understand your warnings or errors you need all the info logs to see what was going on! You need that broader context to make sense of the overall situation. It's very slight semantic information, similar to syntax highlighting, but there are better ways to deliver that information.

# Enhance your logs in other ways

There are better ways to increase the usability of your logs than with strict adherence to different log levels. Structured logging is the practice of emitting logs in a machine-readable format. They're often created from maps, and then output in the logs as JSON or output for humans to see in a more readable format. Using structured logging, you can attach a lot more detail to each log in a way that's *useful*.

Here are a few practices I like to use:

*   **Attach a request/trace id.** This goes along with distributed tracing, and helps you filter effectively. Filtering down by log level won't give you context, but correlating by request id will give you all the logs for a given event!
*   **Include timestamps.** All logs should include timestamps, but let's just make *sure* we have them and at a useful precision. Having these in a structured format makes it easier to use them for calculations.
*   **Add related ids.** When an incoming request relates to a given user, document, or other data, adding that id into the logs is so helpful! Then when you're debugging, you can see either all the logs for a particular object, or you can see if a particular set of data is related to the issue you're working on. Without this, you are just guessing "Maybe this is just a BigCo issue," and with it you know whether other customers see it too.
*   **Note information for auditing.** If you log information for audit purposes, how will you find it? By including something in the logs that let you look for the needle in that haystack (and retain the needles when you toss out the haystack because paying the haystack storage company is akin to setting a pile of cash on fire—not to torture the metaphor). This can be as simple as a boolean for if it's an audit log, or more complicated if you need that.
*   **Flags which were set.** Your application probably has feature flags. Which ones were enabled for this request? Let's put them in the logs so we can tell.
*   **Where the log came from.** Including the module or function name that a log came from can be really helpful for looking at surrounding context later. Too often, I've not had this, and had to grep for a specific string to find where that log was emitted. Hopefully only in one spot.

You do want to avoid adding too much information, because it can be an overwhelming amount! But if you add in this information tastefully, it's really helpful.

And don't add logs in to fix one issue, then remove them later, okay? I've seen that done, and what that says is you did not have enough logs in the first place to understand what was going on! You might not have known exactly what you need, but after you do, then make sure the logs you would have needed for *this* issue are present. Ideally they're in a more general form, so that other related but different issues can also be debugged in the future.

* * *

Look.

I know, log levels are useful. I'm not actually terribly dogmatic about this, because there *can* be situations where you want to tune the amount of detail (especially helpful for libraries to limit their log levels, so you don't get everything from them on INFO).

I just don't think it's useful, most of the time, for the kind of work I do—web applications, mostly—to worry about anything beyond: wake me up, or don't. If you do it differently, I'd love to hear about your experience.