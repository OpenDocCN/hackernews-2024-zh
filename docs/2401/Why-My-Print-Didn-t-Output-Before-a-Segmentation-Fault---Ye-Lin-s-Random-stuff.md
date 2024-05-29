<!--yml
category: 未分类
date: 2024-05-27 14:25:18
-->

# Why My Print Didn't Output Before a Segmentation Fault - Ye Lin's Random stuff

> 来源：[https://blog.yelinaung.com/posts/what-happened-to-my-print/](https://blog.yelinaung.com/posts/what-happened-to-my-print/)

So, as I was learning some [C](https://beej.us/guide/bgc/), at the pointers/segfault, I was trying this code myself.

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); int *p = NULL; *p = 5; // Will not be reached due to crash above printf("%s", "Another Hello!"); } 
```

Then compile it and run it in the terminal.

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Segmentation fault (core dumped) 
```

To my surprise, it did not print out both “Hello!” and “Another Hello!” and the program just crashed with “Segmentation fault (core dumped)” (aka segfault).

For “Another Hello!”, I get it. Program crashed before it reached to that line.

How about “Hello!” ? What’s going on here ? Is it the segfault that prevented printing out ? Before we go further, allow me to expand a bit on this segfault thing.

The first line declares a pointer `p` that should hold an integer type and we are setting it to `NULL` (nowhere/no actual thing).

In the second line tries to writes “5” to the memory location pointed to by `p`. But `p` doesn’t exist. It’s invalid memory access and causes a crash which is [segfault](https://en.wikipedia.org/wiki/Segmentation_fault)!

Wait, how does this crash prevent the program to print out “Hello!” in terminal? It seems like nothing to do with our `printf` since it comes before the crash happens.

What happens when you do `printf` ? In most system, it prints out the data to the output device, in this case, the terminal. It turns out the output devices are “line buffered”. Line buffering means the outputs are stored in a place called “buffer” before they are printed out and will be printed out later on certain conditions. The condidtions, according to the [GNU C guidelines](https://www.gnu.org/software/libc/manual/html_node/Flushing-Buffers.html) are

* * *

(Copied straight from the guide)

Flushing output on a buffered stream means transmitting all accumulated characters to the file. There are many circumstances when buffered output on a stream is flushed automatically:

*   When you try to do output and the output buffer is full.
*   When the stream is closed. See Closing Streams.
*   When the program terminates by calling exit. See Normal Termination.
*   When a newline is written, if the stream is line buffered.
*   Whenever an input operation on any stream actually reads data from its file.

* * *

It is true for printing something to the terminal as well as writing something to a file. In our little program, the program crashed due to segfault and did not manage to flush the buffer. The program abruptly ended.

Now, how can we force the program to print out “Hello!” in terminal before anything crashed ? That will help with debugging things like until at whihc point my program executed before it crashed.

Well, the easy way is remove the error part of the program!

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); printf("%s", "Another Hello!"); } 
```

Compile it and run it again!

```
ubuntu@playground:~$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello!AnotherHello!ubuntu@playground:~$ 
```

Now, all goes well, `main()` returns successfully and everything is flushed out to the output device. I see my prints!

### Newline (`\n`)

But..how about I want to keep things the same way and have my prints. I wanna have the cake and eat it too.

Rememer, in one of the conditions above, it says that “When a newline is written, if the stream is line buffered”.

For that, I can use a newline character (`\n`) to force the program flush to the buffer. We can keep the rest in the same way. Let’s see if that works.

```
#include <stdio.h>  int main(void) {  // This should prints "Hello!" and // flushes to an output device (stdout) due to newline printf("%s\n", "Hello!"); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

Yes! I see my “Hello!” here.

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello! Segmentation fault (core dumped) 
```

The “Another Hello!” didn’t make it but that’s ok. The program crashed before it reaches to that line.

Checking the exit code of the program:

```
$ ./hello Hello! Segmentation fault (core dumped) $ echo $? 139 
```

The `fflush` function also flushes the output buffer to the output device without the use of newline character (`\n`).

```
#include <stdio.h>  int main(void) {  printf("%s", "Hello!"); fflush(stdout); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

will output

```
$ gcc -Wall -Wextra -o hello hello.c && ./hello Hello!Segmentation fault (core dumped) 
```

### stderror is always unbuffered

Someone on HN [said](https://news.ycombinator.com/reply?id=38830109&goto=item%3Fid%3D38803367%2338830109)

> If you use stderr, you get this for free: stderr is always unbuffered (see [here](https://linux.die.net/man/3/stderr)).

```
#include <stdio.h>  int main(void) {  fprintf(stderr, "%s", "Hello!"); int *p = NULL; *p = 5; printf("%s\n", "Another Hello!"); } 
```

works the same

```
$ gcc -Wall -Wextra -o hello_stderr hello_stderr.c && ./hello_stderr Hello!Segmentation fault (core dumped) 
```

Same exit code as the previous example as well.

Well, that’s been my little TIL and hope you learn something as well!

### References