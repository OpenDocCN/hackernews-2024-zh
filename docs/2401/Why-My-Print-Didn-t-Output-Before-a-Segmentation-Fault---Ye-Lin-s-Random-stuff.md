<!--yml

类别：未分类

日期：2024年05月27日14:25:18

-->

# 为什么我的打印在分段错误之前没有输出 - 叶林的随机东西

> 来源：[https://blog.yelinaung.com/posts/what-happened-to-my-print/](https://blog.yelinaung.com/posts/what-happened-to-my-print/)

因此，当我学习一些[C](https://beej.us/guide/bgc/)时，在指针/段错误中，我自己尝试了这段代码。

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); int *p = NULL; *p = 5; // Will not be reached due to crash above printf("%s", "Another Hello!"); } 
```

然后在终端中编译并运行它。

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Segmentation fault (core dumped) 
```

令我惊讶的是，它没有打印出“Hello！”和“Another Hello！”并且程序只是以“分段错误（核心已转储）”（也称为segfault）崩溃。

对于“Another Hello!”，我明白了。程序在达到那一行之前崩溃了。

“Hello！”怎么样？这里发生了什么？是段错误阻止了打印吗？在我们进一步之前，让我稍微扩展一下这个段错误的东西。

第一行声明一个指针`p`，应该保存一个整数类型，并将其设置为`NULL`（没有实际内容）。

在第二行尝试将“5”写入由`p`指向的内存位置。但是`p`不存在。这是无效的内存访问，导致崩溃，即[段错误](https://en.wikipedia.org/wiki/Segmentation_fault)!

等等，这个崩溃怎么阻止程序在终端打印“Hello！”？好像与我们的`printf`无关，因为它发生在崩溃之前。

当你执行`printf`时会发生什么？在大多数系统中，它会将数据打印到输出设备，即终端。结果输出设备是“行缓冲”。行缓冲意味着输出被存储在一个称为“缓冲区”的地方，直到满足特定条件才会被打印出来。根据[GNU C指南](https://www.gnu.org/software/libc/manual/html_node/Flushing-Buffers.html)的条件

* * *

（直接从指南中复制）

在缓冲流上刷新输出意味着将所有累积的字符传输到文件中。有许多情况下，流上的缓冲输出会自动刷新：

+   当尝试进行输出并且输出缓冲区已满时。

+   当流关闭时。参见关闭流。

+   当程序通过调用退出终止时。参见正常终止。

+   当写入新行时，如果流是行缓冲的。

+   每当在任何流上进行输入操作实际上从其文件中读取数据时。

* * *

对于将某些东西打印到终端以及将某些东西写入文件，这是正确的。在我们的小程序中，由于段错误而崩溃，并且未能刷新缓冲区。程序突然结束了。

现在，我们如何才能强制程序在任何东西崩溃之前在终端打印出“Hello！”？这将有助于调试诸如在崩溃之前我的程序执行到哪个点之类的事情。

嗯，简单的方法是移除程序的错误部分！

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); printf("%s", "Another Hello!"); } 
```

再次编译并运行它！

```
ubuntu@playground:~$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello!AnotherHello!ubuntu@playground:~$ 
```

现在，一切都顺利进行，`main()`成功返回，并且所有内容都被刷新到输出设备。我看到我的打印！

### 换行符（`\n`）

但是..如果我想保持事情原样并输出我的内容呢。我想两全其美。

记住，在上面的某个条件中，它说“当写入换行时，如果流是行缓冲的”。

对此，我可以使用换行符（`\n`）强制程序刷新缓冲区。我们可以以同样的方式保持其余部分。让我们看看这是否有效。

```
#include <stdio.h>  int main(void) {  // This should prints "Hello!" and // flushes to an output device (stdout) due to newline printf("%s\n", "Hello!"); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

是的！我在这里看到了我的“Hello！”。

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello! Segmentation fault (core dumped) 
```

“另一个Hello！”没有出现，但没关系。程序在到达那一行之前就崩溃了。

检查程序的退出代码：

```
$ ./hello Hello! Segmentation fault (core dumped) $ echo $? 139 
```

`fflush` 函数还可以在不使用换行符 (`\n`) 的情况下刷新输出缓冲区到输出设备。

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); fflush(stdout); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

输出结果

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello!Segmentation fault (core dumped) 
```

### stderror 总是不带缓冲的

HN 上的某人[说](https://news.ycombinator.com/reply?id=38830109&goto=item%3Fid%3D38803367%2338830109)

> 如果你使用 stderr，你就可以免费获得这个：stderr 总是不带缓冲的（参见[这里](https://linux.die.net/man/3/stderr)）。

```
#include <stdio.h>  int main(void) {  fprintf(stderr, "%s", "Hello!"); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

也能起作用

```
$ gcc -Wall -Wextra -o hello_stderr hello_stderr.c && ./hello_stderr Hello!Segmentation fault (core dumped) 
```

与前面例子相同的退出代码。

好吧，这就是我的小发现，希望你也能学到一些！

### 参考资料
