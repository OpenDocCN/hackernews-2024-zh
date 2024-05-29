<!--yml
category: 未分类
date: 2024-05-27 14:54:21
-->

# systemd by example - Part 1: Minimization - Sebastian Jambor's blog

> 来源：[https://seb.jambor.dev/posts/systemd-by-example-part-1-minimization/](https://seb.jambor.dev/posts/systemd-by-example-part-1-minimization/)

November 23, 2021

# systemd by example

## Part 1: Minimization

### Series overview

This article is part of the series *systemd by example*. The following articles are available.

### Introduction

This is the first article in a series where I try to understand systemd by creating small containerized examples.

systemd always has been a bit of a mystery to me. I knew that it is used for system initialization and for service management, but I didn’t really understand how it worked. Every time I tried to dig deeper, for example by looking at the setup of my machine or reading the docs, I was quickly overwhelmed. There are over 300 systemd units active on my system, and it’s not easy to know which ones are important and what they are used for. The man pages are comprehensive, but it is easy to get lost in details. Similarly for the resources online: there are a lot of them, but none of them really made it click for me. (Although the original announcement of systemd, [Rethinking PID 1](http://0pointer.de/blog/projects/systemd.html), cleared up quite a few things.)

What usually helps me in situations like this is to start with a minimal example which only contains the essentials and try to understand how this works; then incrementally extend it: add new features, explore things described in the documentation, try different settings; and finally iterate. With systemd, this seems hard to do at first. After all, I don’t really want to mess around with my system configuration if I don’t know what I’m doing. Furthermore, experimentation inevitably means breaking things, which I definitely don’t want to do with my live system.

I then found this article on [how to run systemd in a container](https://developers.redhat.com/blog/2019/04/24/how-to-run-systemd-in-a-container). This allows me to do exactly what I want! It gives a testbed for examples and allows quick iteration on experiments. It’s ok to break things since it is confined to the container. And it’s easy to keep track of different examples by using different directories and version control.

So let’s start by creating a minimal systemd example!

*Note*: It should be clear by now that I’m not a systemd expert, so take everything I write with a grain of salt. Instead of authoritative information, I hope that you take this article as an invitation to do your own systemd experiments.

### systemd units

The basic building block of systemd is a unit. Most of them are defined in unit files which live in `/lib/systemd/system` and `/etc/systemd/system` (the former is supposed to contain unit files installed by the distribution, and the latter unit files created by the system administrator). Every unit has a type, which is encoded in the file extension. There are currently eleven different unit types, but in this article we will only consider three of them: targets, services, and sockets.

*Targets* are the simplest of them. systemd activates certain targets based on the system state. For example, there is the `bluetooth.target` which is activated as soon as a Bluetooth controller becomes available; the `sleep.target` which is activated when the system goes to sleep; and the `default.target` which is the unit that systemd activates at bootup.

A target by itself it is pretty useless. It only becomes useful through its *dependencies*, which can be any other units. That’s where *services* come into play, which are processes that are controlled by systemd. They can be services in the usual sense, like an http server or an ssh daemon, which are started when the system boots up (by adding them as dependencies of the `default.target`). But a systemd service can also be any other program; for example, you could start a screen lock when your system goes to sleep by adding a service as a dependency to `sleep.target`. Then when the system wakes up again, the screen lock will still be running and require a password to use the system. So by adding a service unit as a dependency of a target, the program will be executed when the target becomes active. Roughly speaking, by defining a service we tell systemd *what* to do, and by adding it as a dependency of a target we tell systemd *when* to do it.

The final unit type we will need for our minimal example is the socket; we will get back to that later.

### Starting with a clean slate

We want to create a minimal systemd example, and that means as few units as possible. On a real system, there are a lot of unit files, for example, there are 355 entries in `/lib/systemd/system` on my machine. We will get rid of all of this and start with nothing.

We’ll use a Ubuntu base image and install systemd on it, and then remove all unit files.

```
FROM ubuntu:20.04   RUN apt-get update && \  apt-get install --assume-yes systemd   RUN rm -rf /lib/systemd/system/* /etc/systemd/system/*   CMD ["/lib/systemd/systemd"] 
```

Dockerfile

Let’s build the image with

```
podman build --tag systemd . 
```

and then run the container with

```
podman run --tty --rm --name systemd systemd 
```

We use `--name systemd` so that we can address the container more easily later; `--rm` so that the container is automatically removed when we stop it, which allows us to iterate on our examples more easily; and `--tty` to see the output of the container (thanks Даниил Леонтьев for notifying me that the last parameter was missing in an earlier version). Note that we are using `podman` instead of `docker` as container runtime, since it has built-in support to run systemd; see the article I linked above for more details.

Running the container fails with the following output.

```
[...]
Unit default.target not found.
Falling back to rescue.target.
Unit rescue.target not found.
[!!!!!!] Failed to load rescue.target.
Exiting PID 1... 
```

So we went a little over board and removed too much. But systemd also tells us what’s missing. As I mentioned above, `default.target` is the target that is activated when the system boots up. So let’s add it to our system.

### Making the system start

On a real system, `default.target` is usually a symlink to a different target unit. For example, on my system it points to `graphical.target`. This target has a dependency on a service that starts the Gnome Display Manager, so when my system boots up, I’m greeted with a graphical login. It has other dependencies as well. A lot of them. We can see them with `systemctl`, using

```
systemctl list-dependencies graphical.target 
```

On my machine, this lists 180 dependencies. They are not all direct dependencies; most of them are dependencies of dependencies, or dependencies of those, etc. As I mentioned, a real system can be overwhelming.

For our example, let’s start with zero dependencies. We will also use a plain file for `default.target` instead of a symlink. Here is the simplest target I can think of.

```
[Unit] Description=A minimal default target 
```

default.target

Unit files are all structured like this. They contain sections (like `[Unit]`) and key-value pairs, called *directives*. You can learn more about which directives exist for each unit type in the man pages (for example, `man systemd.service` for services, or `man systemd.unit` for directives available for all units; use `man systemd.directives` if you have a directive and want to know in which man page it is defined).

Add the target to the image by adding a line

```
COPY default.target /lib/systemd/system/ 
```

to the Dockerfile, build and start up the container, and voilà! A container running systemd successfully.

```
Welcome to Ubuntu 20.04.2 LTS!
Set hostname to <7f02fa832078>.
[  OK  ] Reached target A minimal default target.
Startup finished in 45ms. 
```

### Making the system stop

So starting systemd works now. But stopping … not so much. Running

throws an error in the container:

```
Failed to enqueue halt.target job: Unit halt.target not found. 
```

It also doesn’t exit the process. It does nothing for ten seconds, after which `podman` sends a `SIGKILL` to the process.

Let’s try to fix this. First, we follow the hint that systemd has given us and add a `halt.target` file, again keeping it as simple as possible.

```
[Unit] Description=A minimal halt target 
```

halt.target

(Remember to add it to the Dockerfile so that it gets copied to the image). If we run this container and stop it again, the error message about the missing `halt.target` is gone (progress!), but it still takes ten seconds for the process to terminate. What’s happening here? As I said before, the target itself is pretty useless, we need to add a service as a dependency which does the actual work.

So let’s create our first service. The command to shut down systemd is `systemctl --force halt`, and we can encode it in our service unit as follows.

```
[Unit] Description=Halt systemd DefaultDependencies=no   [Service] ExecStart=systemctl --force halt 
```

halt.service

This unit contains a service section with the `ExecStart=` directive which tells systemd the command to execute when the service starts (there’s also `ExecStop=` to execute a command when the service stops, and `ExecReload=` for when the service is reloaded). The `DefaultDependencies=` directive is also new, we’ll get to that in a bit.

Now we need to tell systemd to execute `halt.service` when it reaches `halt.target`. We do that by adding `halt.service` as a dependency of `halt.target`, by adding a `Requires=` directive to the target unit file. (We will get into more details about defining dependencies in a later article.)

```
[Unit] Description=A minimal halt target Requires=halt.service 
```

halt.target

And indeed, when we run this container and then try to stop it, this time the process exits immediately. It also emits a warning before exiting:

```
halt.service: Failed to connect stdout to the journal socket, ignoring: No such file or directory 
```

We will address this in the next section. But first, let’s get back to the `DefaultDependencies=no` line. We want a minimal example, so what happens when we remove it? If we do and then go through the flow of building, starting, and stopping the container, we get the message

```
Failed to enqueue halt.target job: Unit sysinit.target not found. 
```

and the process keeps running. The reason is that services can also have dependencies (in fact, every systemd unit can have a dependency on any other unit), and systemd adds some dependencies for each service by default. One of those default dependencies is a requirement on `sysinit.target`. On a real system, this target ensures that the system is properly set up (by in turn having dependencies on units that do the actual set up). In our system the target doesn’t exist, so the service fails since the requirement cannot be met. Adding `DefaultDependencies=no` tells systemd not to add those default dependencies.

### Adding a logging system

systemd complained about a missing journal socket above. This socket is part of journald, systemd’s logging framework. It receives log messages from different sources, like kernel logs and system logs. It also receives everything written to stdout and stderr of a systemd service: systemd connects those two to the journal by default. Once log messages are stored in journald, we can query them with `journalctl`: we can show all log messages stored in the journal, or we can refine our query, showing only logs in a certain time frame, or belonging to a certain service. This is really useful, especially if something is going wrong and we want to find out why.

To start journald we need a service. This is similar to the halt service above: we supply the command that should be executed and then add the service as a dependency of a target; this time it’s the default target, since we want journald to be started once the container “boots”. But in addition to the service we need something else: a socket unit.

In a “classical” service, for example a PostgreSQL server, a socket is set up in the service itself. It may be configurable by a command line setting or through a config file, but the actual setup is done by the service. For example, I can execute

```
postgres -p 2345 -D test.db 
```

and it will create PostgreSQL server listening on TCP port 2345.

systemd unties sockets from services. It makes a socket a first class concept that can live outside of a service. This enables for example to open a socket without running a service, and only start the service once there is traffic on the socket (we will see this in action in a later article). In a socket unit file we can specify different socket types to listen on, like file system sockets or IPv4 or IPv6 sockets. For journald, we create a socket unit with two file sockets, one streaming socket and one datagram socket. The filename for those are defined by systemd; we cannot change them, otherwise services trying to log to journald would fail.

```
[Unit] Description=Journal Socket DefaultDependencies=no   [Socket] ListenStream=/run/systemd/journal/stdout ListenDatagram=/run/systemd/journal/socket 
```

systemd-journald.socket

Then we create a service unit, specifying the socket unit which should be passed in.

```
[Unit] Description=Journal Service DefaultDependencies=no   [Service] ExecStart=/lib/systemd/systemd-journald Sockets=systemd-journald.socket 
```

systemd-journald.service

And finally we add the service as a dependency to our default target.

```
[Unit] Description=A minimal default target Requires=systemd-journald.service 
```

default.target

With this, when we run the container, journald is started automatically.

```
Welcome to Ubuntu 20.04.2 LTS!

Set hostname to <311635618eb2>.
[  OK  ] Reached target A minimal default target.
[  OK  ] Listening on Journal Socket.
[  OK  ] Started Journal Service. 
```

We can execute `journalctl` on the container to see what has been logged:

```
podman exec systemd journalctl 
```

```
-- Logs begin at Tue 2021-11-16 19:23:40 UTC, end at Tue 2021-11-16 19:23:40 UTC. --
Nov 16 19:23:40 311635618eb2 systemd-journald[15]: Journal started
Nov 16 19:23:40 311635618eb2 systemd-journald[15]: Runtime Journal (/run/log/journal/93a63408ad78310aa88bd408619404da) is 8.0M, max 788.1M, 780.1M free.
Nov 16 19:23:40 311635618eb2 systemd: Startup finished in 43ms. 
```

It’s not much, just two log statements from journald itself and one from systemd. But if we had other services, their output would show up here as well.

And finally, when we shut down the container, it does so immediately and without warnings.

### Future proofing

As a final touch, we will add a sysinit target. This is not strictly required, but it’s useful if we want to add further services later. All services have a default dependency on this target, so if it is missing, the service will fail to start; we saw this above for `halt.service`.

On a real system, `sysinit.target` is activated during bootup, and it has a lot of direct and indirect dependencies which are responsible for setting up the system. For us, no setup is needed, so we keep it as simple as possible.

```
[Unit] Description=Empty sysinit target 
```

sysinit.target

Then we add it as a dependency of default.target. (On a real system, it is not a direct dependency, but instead a dependency of a dependency of a dependency etc.)

```
[Unit] Description=A minimal default target Requires=systemd-journald.service sysinit.target 
```

default.target

And that’s it! A functioning, minimal systemd setup.

### Conclusion

Let’s review what we have created. We have a `default.target` which is activated when the container starts. This target pulls in `systemd-journald.service` which sets up journald (with the help of `systemd-journald.socket`), so that services have something to log to. It also pulls in `sysinit.target` which will allow us to easily add service units later. On the other end, we have `halt.target` which is activated when the system is supposed to shut down, and which pulls in `halt.service` to do the actual shutdown.

Combined, the files have less than 30 lines; here are they again in their entirety (you can also find them on [GitHub](https://github.com/sgrj/systemd-by-example/tree/main/minimization)).

```
[Unit] Description=A minimal default target Requires=systemd-journald.service sysinit.target 
```

default.target

```
[Unit] Description=Journal Service DefaultDependencies=no   [Service] ExecStart=/lib/systemd/systemd-journald Sockets=systemd-journald.socket 
```

systemd-journald.service

```
[Unit] Description=Journal Socket DefaultDependencies=no   [Socket] ListenStream=/run/systemd/journal/stdout ListenDatagram=/run/systemd/journal/socket 
```

systemd-journald.socket

```
[Unit] Description=Empty sysinit target 
```

sysinit.target

```
[Unit] Description=A minimal halt target Requires=halt.service 
```

halt.target

```
[Unit] Description=Halt systemd DefaultDependencies=no   [Service] ExecStart=systemctl --force halt 
```

halt.service

These six units are enough to make systemd run, if only barely. Of course, this setup is nothing what you would run in production, because it is lacking a lot of what makes up a real system and what makes it reliable. It is purposefully designed for minimalism, which I find helpful to understand new concepts.

From here, we can now go in several directions. One direction is to investigate our minimal system further, for example with the `systemctl` utility. If you execute

```
podman exec systemd systemctl list-units --all 
```

you’ll see that systemd has more units than the six that we defined; you can then try to find out what they are used for. Or if you execute

```
podman exec systemd systemctl list-dependencies default.target --all 
```

you’ll see all dependencies of `default.target`, including all transitive dependencies. Again, there are more than we explicitly defined, so you could follow that path.

Another direction is to add more units to the system to understand different aspects of systemd. That’s what I’ll do in future articles, so stay tuned.

—Written by Sebastian Jambor. Follow me on Mastodon [@crepels@mastodon.social](https://mastodon.social/@crepels) for updates on new blog posts.