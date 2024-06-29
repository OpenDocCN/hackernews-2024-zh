<!--yml

category: 未分类

date: 2024-05-29 12:40:19

-->

# 我曾经得到的最好的工程面试问题，第一部分 – Arthur O'Dwyer – 主要关于 C++

> 来源：[https://quuxplusone.github.io/blog/2022/01/06/memcached-interview/](https://quuxplusone.github.io/blog/2022/01/06/memcached-interview/)

这是我最后一次接受软件工程面试已经有些时间了。但我仍然记得我最喜欢的面试问题。那是在 2013 年左右的 MemSQL（他们[甚至换了名字](https://www.singlestore.com/blog/revolution/)，所以我认为他们不再依赖这个具体的面试问题。我并不觉得泄露这个问题会有什么不妥。这是一个我经常告诉人们的好故事；只是我以前从未在博客中提过它。）

好的，这不是一个明确的“问题”，而是一个编程挑战。我忘记了他们为此给了多少时间。假设是三小时，包括解释问题的时间。

因为 MemSQL 是一家数据库公司，这是一个数据库挑战。

## 介绍 memcached

你知道 `memcached` 吗？不知道？那好吧，它是一个内存中的键值存储。([在这里阅读它的关于页面。](https://memcached.org/about)) 让我们下载它的代码并构建它，这样我就可以向你展示它的功能。

> 为了成功构建 memcached，您可能需要 `brew install libevent` 以及可能的其他一些东西。弄清楚它不会太困难；但无论如何，与您的环境斗争并不是测试的一部分。您可以假设面试者已经获得了一个已经安装了所有正确依赖项的 Linux 环境。

为了体验 2013 年的真实感，让我们跳过 [GitHub 仓库](https://github.com/memcached/memcached) 并解压一个当代的源码分发：

```
curl -O https://memcached.org/files/memcached-1.4.15.tar.gz
tar zxvf memcached-1.4.15.tar.gz
cd memcached-1.4.15
./configure
make 
```

现在我们已经构建了 `memcached` 可执行文件。让我们启动它：

我们可以通过默认的 memcached 端口，端口 11211，与服务器通信。它的协议基本上是纯文本，所以我们可以使用老旧的 `telnet` 来与它通信。（如果您没有 `telnet` 了，请使用 [`nc -c`](https://www.unixfu.ch/use-netcat-instead-of-telnet/) 替换 `telnet`。）

```
$ telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'. 
```

## 玩转 memcached

memcached 是一个键-值存储。这意味着我们可以告诉它记住某些东西（键和值之间的关联），然后稍后再问它那个键，它会告诉我们它记住的值。在 memcached 中，键始终是 ASCII 字符串，值始终是任意的字节流（这意味着您必须手动指定它们的长度）。例如，将以下内容输入到您的 telnet 会话中：

```
set fullname 0 3600 10
John Smith 
```

这告诉 memcached 记住了字符串键 `fullname` 与 10 字节值 `John Smith` 之间的关联。该行上的其他数字是“标志”值（`0`），以及一个过期时间（`3600`），以秒计算，之后 memcached 将会忘记这个关联。不管怎样，在你输入这两行之后，memcached 将会响应：

现在你可以通过在同一个 telnet 会话中输入以下内容来检索 `fullname` 的值：

memcached 将会响应：

```
VALUE fullname 0 10
John Smith
END 
```

你可以通过发出另一个`set fullname`命令来覆盖与`fullname`关联的值。你也可以要求memcached以某些方式修改值；例如，有专用的命令用于`append`和`prepend`。

```
append fullname 0 3600 6
-Jones
STORED
get fullname
VALUE fullname 0 16
John Smith-Jones
END 
```

当然，如果你想要在客户端程序中向`fullname`追加`-Jones`，你*可以*这样做：

```
# pip install python-memcached
import memcache
mc = memcache.Client(['127.0.0.1:11211'])
v = mc.get('fullname')    # get the old value from memcached
v += '-Jones'             # append -Jones
mc.set('fullname', v)     # set the new value into memcached 
```

但是，memcached的专用`append`命令的优势在于它保证*原子性*。如果多个客户端连接到同一台memcached服务器，并且它们都试图同时向同一个键进行追加操作，`get/set`版本可能会导致某些更新丢失，而使用`append`命令则可以确保所有更新都会被记录。

另一个执行原子操作的专用命令是`incr`：

```
set age 0 3600 2
37
STORED
incr age 1 
```

memcached以后增加的值做出响应：

由于存在多个客户端的情况，这种响应是有用的。如果你发出单独的`get age`命令，你可能只会在其他几个客户端进行它们自己的更新后才看到新值。如果你打算将该值用作序列号、SQL主键或类似的内容，那么能够*原子性*地查看增加的值是非常好的。

当然，memcached还记得增加后的值：

```
get age
VALUE age 0 2
38
END 
```

注意，`37`和`38`仍然存储和返回为字节字符串；它们被从ASCII解码为整数，然后作为原子操作的一部分重新编码。如果尝试对非整数值进行`incr`操作，将会收到错误：

```
incr fullname 1
CLIENT_ERROR cannot increment or decrement non-numeric value 
```

最后，请注意，`incr`是一个完整的加法操作：你可以递增（或`decr`）任何正值，而不仅仅是`1`。

```
incr age 10
48
decr age 10
38
incr age -1
CLIENT_ERROR invalid numeric delta argument 
```

顺便说一句，当你与memcached交互结束并希望关闭连接时，你可以键入memcached命令`quit`。（如果你使用`nc -c`，Ctrl+D也可以。或者，直接进入运行memcached服务器的终端窗口并使用Ctrl+C来关闭。）

## 挑战：修改memcached

通过其`incr`和`decr`命令，memcached提供了一种内置方式来原子性地增加\(k\)到一个数字。但它并不提供其他算术操作；特别是，并没有“原子性乘以\(k\)”的操作。

你的编程挑战：**为memcached添加一个`mult`命令。**

当你完成后，我应该能够telnet到你的memcached客户端并运行如下命令

你有三个小时的时间。开始吧！

* * *

欲了解更多详细内容和分析，请参见[“我曾经遇到的最好的工程面试问题，第2部分。”](/blog/2022/01/07/memcached-interview-solution/)
