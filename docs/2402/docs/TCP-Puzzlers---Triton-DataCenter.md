<!--yml
category: 未分类
date: 2024-05-27 15:04:14
-->

# TCP Puzzlers | Triton DataCenter

> 来源：[https://www.tritondatacenter.com/blog/tcp-puzzlers](https://www.tritondatacenter.com/blog/tcp-puzzlers)

It's been said that we don't really understand a system until we understand how it fails. Despite having written a (toy) TCP implementation in college and then working for several years in industry, I'm continuing to learn more deeply how TCP works — and how it fails. What's been most surprising is how basic some of these failures are. They're not at all obscure. I'm presenting them here as puzzlers, in the fashion of [Car Talk](http://www.cartalk.com/content/puzzlers) and the [old Java puzzlers](https://www.youtube.com/watch?v=wbp-3BJWsU8). Like the best of those puzzlers, these are questions that are very simple to articulate, but the solutions are often surprising. And rather than focusing on arcane details, they hopefully elucidate some deep principles about how TCP works.

## Prerequisites

These puzzlers assume some basic knowledge about working with TCP on Unix-like systems, but you don't have to have mastered any of this before diving in. As a refresher:

*   TCP states, the three-way handshake used to establish a connection, and the way connections are terminated are described pretty concisely on the [TCP Wikipedia page](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Protocol_operation).
*   Programs typically interact with sockets using `read`, `write`, `connect`, `bind`, `listen`, and `accept`. There's also `send` and `recv`, but for our purposes, these work the same way as `read` and `write`.
*   I'll be talking about programs that use `poll`. Although most systems use something more efficient like `kqueue`, event ports, or `epoll`, these are all equivalent for our purposes. As for applications that use blocking operations instead of any of these mechanisms: once you understand how TCP failure modes affect poll, it's pretty easy to understand how it affects blocking operations as well.

You can try all of these examples yourself. I used two virtual machines running under VMware Fusion. The results match my experiences in our production systems. I'm testing using the `nc(1)` tool on SmartOS, and I don't believe any of the behavior shown here is OS-specific. I'm using the illumos-specific [truss(1)](http://illumos.org/man/truss) tool to trace system calls and to get some coarse timing information. You may be able to get similar information using [dtruss(1m)](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/dtruss.1m.html) on OS X or [strace(1)](http://linux.die.net/man/1/strace) on GNU/Linux.

`nc(1)` is a pretty simple tool. We'll use it in two modes:

*   As a server. In this mode, `nc` will set up a listening socket, call `accept`, and block until a connection is received.
*   As a client. In this mode, `nc` will create a socket and establish a connection to a remote server.

In both modes, once connected, each side uses `poll` to wait for either stdin or the connected socket to have data ready to be read. Incoming data is printed to the terminal. Data you type into the terminal is sent over the socket. Upon CTRL-C, the socket is closed and the process exits.

In these examples, my client is called `kang` and my server is called `kodos`.

## Warmup: Normal TCP teardown

This one demonstrates a very basic case just to get the ball rolling. Suppose we set up a server on `kodos`:

Server

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464310423.7650  [ Fri May 27 00:53:43 UTC 2016 ] 0.0027 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0028 listen(3, 1, SOV_DEFAULT)                       = 0accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) (sleeping...)
```

(Remember, in these examples, I'm using `truss` to print out the system calls that `nc` makes. The `-d` flag prints a relative timestamp and the `-t` flag selects which system calls we want to see.)

Now on `kang`, I establish a connection:

Client

```
[root@kang ~]# truss -d -t connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464310447.6295  [ Fri May 27 00:54:07 UTC 2016 ]... 0.0062 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

Over on kodos, we see:

Server

```
23.8934 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

At this point, the TCP connections are in state ESTABLISHED and both processes are in `poll`. We can see this using the `netstat` tool on each system:

Server

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.33226   1049792      0 1049800      0 ESTABLISHED...
```

Client

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.33226   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED...
```

The question is, **when we shut down one of these processes, what happens in the other process?** Does it find out? How? Try to predict the specific syscall behavior and explain why each system call does what it does.

Let's try it. I'll CTRL-C the server on kodos:

Server

```
pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)^C127.6307          Received signal #2, SIGINT, in pollsys() [default]
```

Here's what we see on `kang`:

Client

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)126.1771        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1126.1774        read(3, 0x08043670, 1024)                       = 0126.1776        close(3)                                        = 0[root@kang ~]# 
```

What happened here? Let's be specific:

1.  We sent SIGINT to the server, causing the process to exit. Upon exit, file descriptors are closed.
2.  When the last file descriptor for an `ESTABLISHED` socket is closed, the TCP stack on kodos sends a FIN over the connection and enters the `FIN_WAIT_1` state.
3.  The TCP stack on kang receives the FIN packet, transitions its own connection to `CLOSE_WAIT`, and sends an ACK back. Since the `nc` client is blocked on this socket being ready to read, the kernel wakes this thread with the event `POLLIN`.
4.  The `nc` client sees `POLLIN` for the socket and calls `read`, which returns 0 immediately. This indicates end-of-stream. `nc` presumes that we're done with this socket and closes it.
5.  Meanwhile, the TCP stack on kodos receives the ACK and enters `FIN_WAIT_2`.
6.  Since the `nc` client on kang closed its socket, the TCP stack on kang sent a FIN to `kodos`. The connection on kang enters `LAST_ACK`.
7.  The TCP stack on kodos receives the FIN, the connection enters `TIME_WAIT`, and the stack on kodos acknowledges the FIN.
8.  The TCP stack on kang receives the ACK for the FIN and removes the connection entirely.
9.  Two minutes later, the TCP connection on kodos is closed and the stack removes the connection entirely.

(It's possible for steps to be slightly reordered and for kang to transition through `CLOSING` instead of `FIN_WAIT_2`.)

Here's the final state according to netstat:

Server

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.33226   1049792      0 1049800      0 TIME_WAIT
```

There's no output for this connection at all on kang.

The intermediate states transition very quickly, but you could see them using the [DTrace TCP provider](http://dtracebook.com/index.php/Network_Lower_Level_Protocols:tcpstate.d). You can see the packet flow using [snoop(1m)](http://illumos.org/man/snoop) or [tcpdump(1)](http://www.tcpdump.org/tcpdump_man.html).

**Conclusions:** We've seen the normal path of system calls for connection establishment and teardown. Note that kang immediately found out when kodos's connection was closed — it was woken up out of `poll` and `read` returned 0 to indicate end-of-stream. At that point, kang *elected* to close its socket, which cleaned up the connection state on kodos. We'll revisit this later to see how the results can differ if kang doesn't close its socket here.

## Puzzler 1: Power cycling

**What happens to an established, idle TCP connection if one system is power cycled?**

Since most processes go through normal exit as part of a graceful reboot (e.g., if you use the "reboot" command), you get basically the same result if you type "reboot" on kodos instead of killing the server with CTRL-C. But what would happen if we'd power-cycled kodos in the previous example? Surely kang will eventually find out, right?

Let's try it. Set up the connection:

Server

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464312528.4308  [ Fri May 27 01:28:48 UTC 2016 ] 0.0036 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0036 listen(3, 1, SOV_DEFAULT)                       = 0 0.2518 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

Client

```
[root@kang ~]# truss -d -t open,connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464312535.7634  [ Fri May 27 01:28:55 UTC 2016 ]... 0.0055 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

Now I'll use VMware's "reboot" function to power-cycle the system. It's important that this be an actual power cycle (or OS panic) — anything that causes a graceful shut down will look more like the first case above.

20 minutes later, kang is still sitting in the same place:

Client

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

We tend to believe that TCP's job is to maintain a consistent abstraction (namely, the TCP connection) across multiple systems, so it's surprising to discover cases where the abstraction is broken like this. And lest you think this is some nc(1) issue, it's not. "netstat" on kodos shows no connection to kang, but kang shows a perfectly healthy connection to kodos:

Client

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.50277   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED...
```

We can leave this forever without touching it and kang will *never* figure out that kodos rebooted.

**Now, suppose at this point, kang tries to send data to kodos. What happens?**

Let's try it:

Client

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)kodos, are you there?3872.6918       pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 13872.6920       read(0, " k o d o s ,   a r e   y".., 1024)     = 223872.6924       write(3, " k o d o s ,   a r e   y".., 22)      = 223872.6932       pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 13872.6932       read(3, 0x08043670, 1024)                       Err#131 ECONNRESET3872.6933       close(3)                                        = 0[root@kang ~]#
```

When I type the message and hit enter, kodos gets woken up, reads the message from stdin, and sends it over the socket. *The `write` call completes successfully!* `nc` goes back to poll to wait for another event, eventually finds out that the socket can be read without blocking, and then calls `read`. This time, `read` fails with `ECONNRESET`. What does this mean? [POSIX's definition of read(2)](http://pubs.opengroup.org/onlinepubs/009695399/functions/read.html) says that this means:

```
[ECONNRESET]A read was attempted on a socket and the connection was forcibly closed by its peer.
```

The [illumos read(2) man page](http://illumos.org/man/read.2) provides a little more context:

```
 ECONNRESET              The filedes argument refers to a connection oriented              socket and the connection was forcibly closed by the peer              and is no longer valid.  I/O can no longer be performed to              filedes.
```

This error doesn't mean that something specific to the `read` call went wrong, but rather that the socket itself is disconnected. Most other socket operations would fail the same way at that point.

So what happened here? At the point when the `nc` on kang tried to send data, the TCP stack still didn't know the connection was dead. kang sent a data packet over to kodos, which responded with an RST because it didn't know anything about the connection. kang saw the RST and tore down its connection. It cannot close the socket file descriptor — that's not how file descriptors work — but subsequent operations fail with ECONNRESET until nc does close the fd.

**Conclusions:**

1.  A hard power-cycle is very different than a graceful shutdown. When testing distributed systems, this case needs to be specifically tested. You can't just kill a process and expect it to test the same thing.
2.  **There are situations when one side believes a TCP connection is established, but the other side does not, and where this situation will never be automatically resolved.** It's possible to manage this problem using TCP or application-level keep-alive.
3.  The only reason that kang eventually found out that the remote side had disappeared was that it took action: it sent data and received a response indicating the connection was gone.

That raises the question: what if kodos had not responded to the data message for some reason?

## Puzzler 2: Power off

**What happens if one endpoint of a TCP connection is powered off for a while?** Does the other endpoint ever discover this? If so, how? And when?

Once again, I'll set up the connection with `nc`:

Server

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464385399.1661  [ Fri May 27 21:43:19 UTC 2016 ] 0.0030 bind(3, 0x08065790, 32, SOV_SOCKBSD)        = 0 0.0031 listen(3, 1, SOV_DEFAULT)           = 0accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) (sleeping...) 6.5491 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

Client

```
[root@kang ~]# truss -d -t open,connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464330881.0984  [ Fri May 27 06:34:41 UTC 2016 ]... 0.0057 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

Now, I'll cut power to kodos abruptly and attempt to send data from kang:

Client

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)114.4971        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1114.4974        read(0, "\n", 1024)                             = 1114.4975        write(3, "\n", 1)                               = 1pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

The `write` completes normally, and I don't see anything for quite a while. Just over 5 minutes later, I see this:

Client

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)425.5664        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1425.5665        read(3, 0x08043670, 1024)                       Err#145 ETIMEDOUT425.5666        close(3)                                        = 0
```

This looks similar to the case where the system was power-cycled instead of powered off, except for two things: it took 5 minutes for the system to notice, and the error reported was ETIMEDOUT. Note that again, it's not that the `read` timed out per se. We would have seen the same error from other socket operations, and subsequent socket operations would likely fail immediately with the same ETIMEDOUT error. That's because it's the *socket* that has entered a state where the underlying connection has timed out. The specific reason for this is that the remote side failed to acknowledge a data packet for too long — 5 minutes, as configured on this system.

**Conclusions:**

1.  When the remote system is powered off instead of power cycled, again, the first system only finds out if it's trying to send data. If it doesn't send packets, it will never find out about the terminated connections.
2.  When the system has been attempting to send data for long enough without receiving acknowledgments from the remote side, the TCP connection will be terminated and a socket operation on them will fail with ETIMEDOUT.

## Puzzler 3: Broken connection with no failure

This time, instead of giving you a specific case and asking what happens, I'll flip it around: here's an observation, and see if you can figure out how it could happen. We've discussed several cases where kang can believe it has a connection to kodos, but kodos doesn't know about it. **Is it possible for kang to have a connection to kodos without kodos knowing about it — indefinitely (i.e., where this will not resolve itself) and even if there's been no power off, no power cycle, and no other failure of the operating system on kodos and no failure of networking equipment?**

Here's a hint: consider the case above where a connection is stuck in `ESTABLISHED`. In that case, it's fair to say that the application has the responsibility to deal with the problem, since by definition it still has the socket open and could find out by sending data, at which point the connection will eventually be terminated. But what if the application didn't have the socket open any more?

In the Warmup, we looked at the case where kodos's `nc` closed its socket, and we said that kang's `nc` read 0 (indicating end-of-stream) and then closed its socket. What if it didn't close the socket, but kept it open? Obviously, it couldn't read from it. But there's nothing about TCP that says you can't send more data to the other side that has sent you a FIN. **FIN only means end-of-data-stream in the direction that the FIN was sent.**

In order to demonstrate this, we can't use `nc` on kang because it automatically closes its socket when it reads 0\. So I've written a demo version of `nc` called [dnc](https://github.com/davepacheco/experiment-dnc), which simply skips this behavior. It also prints out explicitly which system calls it's making. This will give us a chance to observe the TCP states.

First, we'll set up the connections as usual:

Server

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464392924.7841  [ Fri May 27 23:48:44 UTC 2016 ] 0.0028 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0028 listen(3, 1, SOV_DEFAULT)                       = 0accept(3, 0x08047B2C, 0x08047C2C, SOV_DEFAULT, 0) (sleeping...) 1.9356 accept(3, 0x08047B2C, 0x08047C2C, SOV_DEFAULT, 0) = 4pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

Client

```
[root@kang ~]# dnc 10.88.88.140 80802016-05-27T08:40:02Z: establishing connection2016-05-27T08:40:02Z: connected2016-05-27T08:40:02Z: entering poll()
```

Now let's verify the connections are in the `ESTABLISHED` state we expect on both sides:

Server

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.37259   1049792      0 1049800      0 ESTABLISHED
```

Client

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED
```

Now, let's CTRL-C the `nc` process on kodos:

Server

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)^C[root@kodos ~]# 
```

We immediately see this on kang:

Client

```
2016-05-27T08:40:12Z: poll returned events 0x0/0x12016-05-27T08:40:12Z: reading from socket2016-05-27T08:40:12Z: read end-of-stream from socket2016-05-27T08:40:12Z: read 0 bytes from socket2016-05-27T08:40:12Z: entering poll()
```

Now let's look at the TCP connection states:

Server

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.37259   1049792      0 1049800      0 FIN_WAIT_2
```

Client

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    1049792      0 1049800      0 CLOSE_WAIT
```

This makes sense: kodos sent a FIN to kang. `FIN_WAIT_2` indicates that kodos received an ACK from kang for the FIN that it sent, and `CLOSE_WAIT` indicates that kang received the FIN *but has not sent a FIN back*. **This is a perfectly valid state for a TCP connection for an indefinite period of time.** Imagine kodos had sent a request to kang and was not planning to send any more requests; kang could happily send data in response for hours, and this would work. Only in our case, kodos *has* actually closed the socket.

Let's wait a minute and check the TCP connection states again. A minute later, the connection is completely missing from kodos, but it's still present on kang:

Client

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    1049792      0 1049800      0 CLOSE_WAIT
```

What happened here? We hit a lesser-known special case in the TCP stack: when an application has closed a socket, the TCP stack has sent a FIN, and the remote TCP stack has acknowledged the FIN, the local TCP stack waits a fixed period of time *and then closes the connection*. The reason? In case the remote side has rebooted. This case is actually an analog to the case above where one side had an `ESTABLISHED` connection and the other doesn't know about it. The difference is that in this specific case, the application has closed the socket, so there's no other component that could deal with the problem. As a result, the TCP stack waits a fixed period of time and then tears down the connection (without sending anything to the other side).

Follow-up question: **what happens if kang sends data to kodos at this point?** Remember, kang still thinks the connection is open, but it's been torn down on kodos.

Client

```
2016-05-27T08:40:12Z: entering poll()kodos, are you there?2016-05-27T08:41:34Z: poll returned events 0x1/0x02016-05-27T08:41:34Z: reading from stdin2016-05-27T08:41:34Z: writing 22 bytes read from stdin to socket2016-05-27T08:41:34Z: entering poll()2016-05-27T08:41:34Z: poll returned events 0x0/0x102016-05-27T08:41:34Z: reading from socketdnc: read: Connection reset by peer
```

This is the same as we saw in case Puzzler 1: the `write()` actually succeeds, since the TCP stack doesn't know that the connection is closed yet. But it does get a RST, which wakes up the thread in `poll()`, and the subsequent `read()` returns ECONNRESET.

**Conclusions:**

*   It's possible for two sides to disagree about the state of the connection *even when there's been no OS, network, or hardware failure at all*.
*   In a case like the one above, it's not possible for kang to distinguish between the case where kodos is attentively waiting to receive data or kodos has closed the socket and isn't listening (at least, not without sending a packet). For this reason, maybe it's not a great idea to design a system that uses sockets in these half-open states for an extended period of time under normal operation.

## Conclusions

TCP is typically presented as a protocol for maintaining a consistent abstraction — the "TCP connection" — across two systems. We know that in the face of certain types of network and software problems, connections will fail, but it's not always obvious that there are cases where the *abstraction* fails, in that the two systems disagree about the state of the connection. Specifically:

*   It's possible for one system to think it has a working, established TCP connection to a remote system, while the remote system knows nothing about that connection.
*   This is possible even when there has been no failure of the network, hardware, or operating system.

These behaviors do not reflect deficiencies in TCP. Quite the contrary, in all of these cases, the TCP implementation appears to behave as reasonably as it could given the situation. If we were tempted to implement our own communication mechanism instead of using TCP, the presence of these cases might well remind us how complex the underlying problems are. These are intrinsic problems with distributed systems, and a TCP connection is fundamentally a distributed system.

That said, the single most important lesson in all of this is that **the notion of a TCP connection that spans multiple systems is a convenient fiction.** When it goes wrong, it's critical to think about it as two separate state machines that operate simultaneously to try to maintain a consistent view of the connection. It's the responsibility of the application to handle the cases where these differ (often using a keep-alive mechanism).

Furthermore, there's a disconnect between the application's file descriptor and the underlying TCP connection. TCP connections exist (in various states related to closing) even after an application has closed the file descriptor, and a file descriptor can be open when the underlying TCP connection has been closed as a result of a failure.

Other lessons to keep in mind:

*   Ungraceful reboot of a system (as happens when the OS crashes) is not the same as a userland process exiting or closing its socket. It's important to test this case specifically. Reboot, when the remote system comes back online before the connection times out, is also different than power-off.
*   There's no proactive notification from the kernel when a TCP socket is torn down. You only find this out when you call `read()`, `write()`, or some other socket operation on the socket file descriptor. If your program doesn't do this for some reason, you'll never find out about the connection failure.

Some related notes that I've found aren't so commonly known:

*   `ECONNRESET` is a socket error you can see from `read()`, `write()`, and other operations that indicates that the remote peer has sent a RST.
*   `ETIMEDOUT` is a socket error you can see from `read()`, `write()`, and other operations that indicates that some timeout associated with the connection has elapsed. The cases I've seen most are when the remote side did not acknowledge some packet for too long. These are usually either data packets, a FIN packet, or a KeepAlive probe.

Importantly, neither of these errors means that there was anything wrong with the read or write operation that you tried to do — just that the socket itself is closed.

If you made it this far, and this sort of problem sounds interesting to you, [we're hiring](https://www.joyent.com/about/careers)!

*Post written by Dave Pacheco*