<!--yml
category: 未分类
date: 2024-05-27 14:54:18
-->

# Solving the Dining Philosophers Problem with systemd - Part 1 - Blog - Brightbox

> 来源：[https://www.brightbox.com/blog/2024/01/10/solving-dining-philosophers-with-systemd-part-1/](https://www.brightbox.com/blog/2024/01/10/solving-dining-philosophers-with-systemd-part-1/)

# Solving the Dining Philosophers Problem with systemd - Part 1

by **Neil Wilson** • **10 Jan 2024**

Over the years, developers seem to have forgotten that Unix is a Programming Environment and not just something you must install to access your web server. Unix is a powerful multi-user system with numerous tools that have stood the test of time and, with the advent of systemd, we have a new tool that can easily model state machines. Now, we can address a new class of problems using Unix alone.

In this blog post series, we’ll solve the classic [“Dining Philosophers”](https://en.wikipedia.org/wiki/Dining_philosophers_problem) concurrency problem with some of the forgotten mechanisms and tools within the Unix Programming Environment, some advanced capabilities of systemd and a sprinkling of shell scripts to glue it together - all without a single deadlock.

Let’s start by building a simple state machine that moves between the `Thinking`, `Hungry`, and `Eating` states using the user-level systemd facilities.

## Create the server

In these examples, we’ll be using Ubuntu on Brightbox, but any modern distro running systemd running anywhere should work.

Create a server and log in with the default `ubuntu` user:

```
$ ssh ubuntu@public.srv-hgy08.gb1.brightbox.com
Welcome to Ubuntu 23.10 (GNU/Linux 6.5.0-10-generic x86_64)
...
```

When you log in, systemd automatically starts a user instance to manage user-level units. You can view this via systemctl:

```
ubuntu@srv-hgy08:~$ systemctl --user status
● srv-hgy08
    State: running
    Units: 171 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Mon 2023-11-20 09:41:56 UTC; 1min 19s ago
  systemd: 253.5-1ubuntu6
   CGroup: /user.slice/user-1000.slice/user@1000.service
           └─init.scope
             ├─857 /lib/systemd/systemd --user
             └─858 "(sd-pam)"
```

## Create the states

We can model the three states of the dining philosophers problem using systemd target units. We can make the targets mutually exclusive by stating that they conflict with each other:

```
# thinking.target [Unit]
Description=Thinking...
Conflicts=eating.target hungry.target
```

This means only one of the targets can remain active at any point in time. For a target to transition to another, we need a timer unit.

```
# thinking.timer [Unit]
Description=Thinking Period
PartOf=thinking.target

[Timer]
Unit=hungry.target
RandomizedDelaySec=60
OnActiveSec=0
AccuracySec=1us
RemainAfterElapse=false
```

We make the timer unit part of the target so that it stops at the same time as the target. This combination of `RandomizedDelay`, `OnActive`, and `Accuracy` distributes the transition time from 0 to 60 seconds.

We want to know what the philosopher is thinking about, which we can simulate by running a service to call a script.

```
# contemplation.service [Unit]
Description=Contemplation
PartOf=thinking.target
Before=hungry.target

[Service]
Type=oneshot
SyslogFacility=local0
ExecStart=/usr/local/bin/contemplation
```

We log the script’s output to a different syslog facility, which allows us to find them easily in the log later. A `Before` restriction ensures that the philosopher logs their thoughts before dealing with their hunger.

Create the contemplation script to output an appropriate thought:

```
#!/bin/sh
# /usr/local/bin/contemplation
set -e

default_thought="the enigmatic nature of reality:
the existential grasp for an elusive fork, the categorical imperative
of simulated existence, and the irony in questioning the true essence
of the held object."
{
    printf '%s' 'Currently contemplating '
    printf '%s\n' "${default_thought}"
} | fmt
exit 0
```

Now, we update the target to pull in these new units:

```
# thinking.target [Unit]
Description=Thinking...
Conflicts=eating.target hungry.target
Wants=thinking.timer contemplation.service
```

And repeat the process for the eating and hungry targets.

## Running the state machine

To start the simulation, put the files in `$HOME/.config/systemd/user` and activate the `thinking` target:

```
$ systemctl --user start thinking.target
```

Look in the journal to see the simulation in progress:

```
$ journalctl -f
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Started thinking.timer - Thinking Period.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Stopped target eating.target - Eating....
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Stopped eating.timer - Eating Period.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Starting contemplation.service - Contemplation...
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: existence, and the irony in questioning the true essence of the held
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: object.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Finished contemplation.service - Contemplation.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Reached target thinking.target - Thinking....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Started hungry.timer - Hungry Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped target thinking.target - Thinking....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped thinking.timer - Thinking Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Starting hunger.service - Hunger...
Nov 20 10:49:17 srv-hgy08 hunger[2165]: Getting hungry...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Finished hunger.service - Hunger.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Reached target hungry.target - Hungry....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Started eating.timer - Eating Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped target hungry.target - Hungry....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped hungry.timer - Hungry Period.
Nov 20 10:49:17 srv-hgy08 consumption[2166]: Eating...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Starting eating-food.service - Eating Food...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Finished eating-food.service - Eating Food.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Reached target eating.target - Eating....
```

We can filter out the state noise by looking just for the output from our scripts:

```
$ journalctl -f --facility local0
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: existence, and the irony in questioning the true essence of the held
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: object.
Nov 20 10:51:14 srv-hgy08 hunger[2177]: Getting hungry...
Nov 20 10:51:15 srv-hgy08 consumption[2178]: Eating...
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: existence, and the irony in questioning the true essence of the held
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: object.
```

Or, if we want to see just one particular script, we can filter by identifier:

```
$ journalctl -f --identifier=contemplation
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: existence, and the irony in questioning the true essence of the held
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: object.
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: existence, and the irony in questioning the true essence of the held
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: object.
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: existence, and the irony in questioning the true essence of the held
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: object.
```

To stop the simulation, stop the targets:

```
$ systemctl --user stop thinking.target eating.target hungry.target
```

or log out.

## Summary

In this blog, we’ve created a simple state machine with systemd that moves neatly between states over time and logs its activity in the journal. It’s hardly a fully modelled philosopher, but it’s a good start.

In the next part, we’ll take this simple model and expand it to cover multiple philosophers, introducing the shared dining room where their contemplations take place.