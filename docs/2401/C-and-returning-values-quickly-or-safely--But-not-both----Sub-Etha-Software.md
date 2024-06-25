<!--yml

类别：未分类

date: 2024-05-27 14:56:26

-->

# 快速或安全地返回值。但不能两者兼得。| Sub-Etha Software

> 来源：[`subethasoftware.com/2024/01/18/c-and-returning-values-quickly-or-safely-but-not-both/`](https://subethasoftware.com/2024/01/18/c-and-returning-values-quickly-or-safely-but-not-both/)

**警告：**本文包含一种许多人会感到不适的 C  编码方法。

在我作为一名温和的嵌入式 C 程序员的日常工作中，我通常忙于维护之前创建的东西，而不是为其他人创建新的东西，供其他人在我之后维护。有那么一次，我有两周的时间非常不同寻常，也很有趣，因为几乎全部都花在了“创建”而不是“维护”上。

今天的快速 C 小知识是关于从 C 函数中获取参数的。在 C 中，你只会得到一件事情返回-通常是像 int 或 float 这样的变量类型，或者其他任何类型：

```
int GetTheUltimateAnswer()
{
    return 42;
}

int answer = GetTheUltimateAnswer();
print ("The Ultimate Answer is %d\n", answer);
```

如果需要返回多个值，通常通过引用传递变量（变量在内存中的地址）并使函数修改该内存来更新变量：  

```
void GetMinAndMax (int *min, int *max)
{
    *min = 0;
    *max = 100;
}

int min, max;
GetMinAndMax (&min, &max)
printf ("Min is %d and Max is %d\n", min, max);
```

一旦指针出现，事情就变得非常危险。但是快。

在传递值时，它们被复制到一个新变量中：

```
int variable = 42;

printf ("variable = %d\n", variable);
Function (variable);
printf ("variable = %d\n", variable);

void Function (int x)
{
    x = x + 1;
}
```

试试看：[`onlinegdb.com/WC3ihCAuj`](https://onlinegdb.com/WC3ihCAuj)

上面，Function() 得到一个新变量（在这种情况下称为“x”），其值为传入调用的变量的值。该函数就像是拉斯维加斯。发生在函数内部的任何事情都留在函数内部-变量在函数结束时消失，而原始变量保持不变。

我学到了 C++ 的变化，所以你可以传递可以修改的变量，但我不是 C++ 程序员，所以这篇文章只涉及老式的 C。

## 指向变量内存

通过传递变量的地址，函数可以访问该内存并修改变量。它将被更改：

```
int variable = 42;

printf ("variable = %d\n", variable);
Function (&variable);
printf ("variable = %d\n", variable);

void Function (int *x)
{
    *x = *x + 1;
}
```

试试看：[`onlinegdb.com/Y2Z9WUvFG`](https://onlinegdb.com/Y2Z9WUvFG)

按值传递速度较慢，因为必须创建一个新变量。按引用传递只传递一个地址，代码使用该地址-不会创建新变量。

但是，仅仅出于速度考虑使用引用是危险的，因为即使你不希望函数修改变量，它也可以修改变量。考虑传递一个字符串缓冲区，它是一个指向一系列字符字节的指针：

```
void PrintError (char *message)
{
    print ("ERROR: %s\n", message);
}

PrintError ("Human Detected");
```

我们经常这样做，但由于 PrintError()有访问传入内存的权限，它可能会尝试修改它。如果我们传递了一个常量字符串，比如“检测到人类”，那么该字符串通常会在程序内存中（尽管对于像 PIC 和 Arduino 这样的哈佛架构系统来说不是这样）。在最好的情况下，具有内存保护功能的操作系统将使用异常来捕获该访问并终止程序。在最坏的情况下，程序会自我修改（这是我在 80 年代末在 OS-9/6809 上学到的情况——在我的 TRS-80 CoCo 上没有内存保护！）。

```
void PrintError (char *message)
{
    message[0] = 42;
}

PrintError ("Human Detected");
```

上面的做法很可能会崩溃，但如果用户传递了包含字符串的缓冲区，它就会被修改：

```
void PrintError (char *message)
{
    message[0] = 42;
}

char buffer[80];
strncpy (buffer, "Hello, world!", 80);
printf ("buffer: %s\n", buffer);
PrintError (buffer);
printf ("buffer: %s\n", buffer);
```

试一下：[`onlinegdb.com/L50JRWYj`](https://onlinegdb.com/L50JRWYj_)

## 你的观点是？

我的观点是——在某些时候，速度确实是最重要的事情，它超过了使用指针的代码可能引起的潜在问题/崩溃。例如，任何传递缓冲区的东西：

```
void UppercaseString (char *buffer)
{
    for (int idx=0; idx<strlen(buffer); idx++)
    {
        buffer[i] = toupper(buffer[I])
    }
}
```

这里可能发生很多不好的事情。通过使用“strlen”，缓冲区必须是一个以 NIL（‘\0’）字节结尾的字符串。这个例程可能会在超出调用者字符串范围的字节上进行内存大写化。

始终添加另一个参数作为缓冲区的最大大小是明智的：

```
void UppercaseString (char *buffer, int bufferSize)
{
    for (int idx=0; idx<bufferSize; idx++)
    {
        buffer[i] = toupper(buffer[I])
    }
}
```

这有所帮助。但捕捉错误类型的指针是否传递给编译器还取决于编译器。

```
int Number = 10;

UppercaseString (&Number, 100);
```

编译器不应该让你这样做，但有些可能只会发出警告并继续构建。（这就是为什么我总是试图在我的代码中没有警告。警告越多，您忽略它们的可能性就越大。）

## 试一下：通过引用传递

假设我们有一个函数，它以独立值（年、月、日、时、分和秒）的形式返回日期和时间。由于我们无法从函数中获取六个值，因此我们首先尝试通过引用传递六个变量，并让例程修改它们：

```
void GetDateTime1 (int *year, int *month, int *day,
                   int *hour, int *minute, int *second)
{
    *year = 2023;
    *month = 8;
    *day = 19;
    *hour = 4;
    *minute = 20;
    *second = 0;
}

int year, month, day, hour, minute, second;
GetDateTime1 (&year, &month, &day, &hour, &minute, &second);
printf ("GetDateTime1: %d/%d/%d %02d:%02d:%02d\n",
        year, month, day, hour, minute, second);
```

只要您知道参数是“ints”（不管那是什么），而不是 shorts 或 longs 或任何其他数值类型，那就没问题。例如，这样做是不好的：

```
short year, month, day, hour, minute, second;

GetDateTime1 (&year, &month, &day, &hour, &minute, &second);
```

上面，我们将一个 short（假设这在这个系统上是一个 16 位变量）传递给一个函数，该函数期望一个 int（假设这在这个系统上是一个 32 位有符号变量）。该函数将尝试在 16 位值的地址处放置 32 位的信息。

有些事情，就像他们说的那样，可能会发生。

## 试一下：通过引用传递结构

传递六个变量指针比传递一个变量更多工作，因此如果我们将值放入结构中，我们只需传递指向该结构的指针。这样做的好处是确保结构只加载它能处理的值（不像传递某些可能是 8、16、32 或 64 位的东西的地址）。

```
typedef struct
{
    int year;
    int month;
    int day;
    int hour;
    int minute;
    int second;
} TimeStruct;

void GetDateTime2 (TimeStruct *timePtr)
{
    timePtr->year = 2023;
    timePtr->month = 8;
    timePtr->day = 19;
    timePtr->hour = 4;
    timePtr->minute = 20;
    timePtr->second = 0;   
}

TimeStruct time;
GetDateTime2 (&time);
printf ("GetDateTime2: %d/%d/%d %02d:%02d:%02d\n",
        time.year, time.month, time.day,
        time.hour, time.minute, time.second);
```

这应该极大地减少潜在的问题，因为你只有一个指针可以搞砸，如果你把类型搞对了（一个 TimeStruct），它包含的值应该是正确的，因为编译器会尝试将一个“uint8_t”设置为“65535”（一种警告，希望是这样，并将这 16 位值的 8 位存储为“精度丢失”）。

## 尝试＃3：返回静态的地址

一些标准 C 库函数采用的方法是在函数内部分配一些固定的内存作为静态变量，然后返回指向该内存的指针。用户不会去创建它，因此不会传递一个可能错误的指针。

```
TimeStruct *GetDateTime3 (void)
{
    static TimeStruct s_time;

    s_time.year = 2023;
    s_time.month = 8;
    s_time.day = 19;
    s_time.hour = 4;
    s_time.minute = 20;
    s_time.second = 0;

    return &s_time;
}

TimeStruct *timePtr;
timePtr = GetDateTime3 ();  
printf ("GetDateTime3: %d/%d/%d %02d:%02d:%02d\n",
       timePtr->year, timePtr->month, timePtr->day,
       timePtr->hour, timePtr->minute, timePtr->second);
```

这种方法更好，因为它使用指针来提高速度，并且由于函数告诉您指针的位置，而不是相反，所以不会出错。

但是...一旦你有了那个静态内存的地址，你就可以修改它。

```
TimeStruct *timePtr;
timePtr = GetDateTime3 ();
timePtr->year = 1969;
```

在一个真正的日期/时间函数中（比如 C 库中的一个），当你调用函数时，这些变量会被系统时间填充，所以即使用户改变了类似这样的内容，它在下一次被调用时也会被设置回原来的状态。但是，我可以理解其他类型的函数可能存在问题。

此外，它总是持有那个内存，无论有没有人在使用它。当在像具有 4K RAM 的 Arduino 这样的内存受限系统上工作时，这是不允许的。

## 尝试＃4：返回结构的副本

现在今天的唠叨的重点...我很少使用这个，因为这可能是做事情最慢的方法，但是...你不只是要返回一个日期类型，比如 int 或 bool 或指针。你可以返回一个结构，C 会给调用者返回结构的一个副本。

```
TimeStruct GetDateTime4 (void)
{
    TimeStruct time;

    time.year = 2023;
    time.month = 8;
    time.day = 19;
    time.hour = 4;
    time.minute = 20;
    time.second = 0;

    return time;
}

TimeStruct time;
time = GetDateTime4 ();    
printf ("GetDateTime4: %d/%d/%d %02d:%02d:%02d\n",
       time.year, time.month, time.day,
       time.hour, time.minute, time.second);
```

以上可能是返回数据的最安全方式，因为没有使用指针。调用者创建一个新的结构变量，然后函数创建一个新的结构变量，返回将该结构复制到调用者的结构中。

试一试：[`onlinegdb.com/F6rR1V-xb`](https://onlinegdb.com/F6rR1V-xb)

这更慢，并且在制作所有这些副本的过程中消耗更多的内存，但是它远远更安全。即使 ChatGPT 也同意，如果要编写“安全”的代码，这是更好的方法。

而且，在我的日常工作中，我尝试了这个方法，效果非常好。这是 C 具有“对象”最接近的东西。我甚至用它来处理 BufferStruct，这样我就可以传递一个缓冲区而不使用指针（尽管内部有一个指向缓冲区内存的指针）。它看起来像这样：

```
#include <stdio.h>
#include <string.h>

typedef struct
{
    char buffer[80];
    char bufferSize;
} BufferStruct;

BufferStruct GetBuffer ()
{
    BufferStruct buf;

    strncpy (buf.buffer, "Hello, world!", sizeof(buf.buffer));
    buf.bufferSize = strlen(buf.buffer);

    return buf;
}

void ShowBuffer (BufferStruct buf)
{
    printf ("Buffer: %s\n", buf.buffer);
    printf ("Size  : %d\n", buf.bufferSize);
}

int main()
{
    BufferStruct myBuffer;
    myBuffer = GetBuffer ();
    ShowBuffer (myBuffer);

    BufferStruct testBuffer;
    strncpy (testBuffer.buffer, "I put this in here",
             sizeof(testBuffer.buffer));
    testBuffer.bufferSize = strlen (testBuffer.buffer);
    ShowBuffer (testBuffer);

    return 0;
} 
```

如果你为了速度而编码，额外的开销可能会成为问题，但是做这个技巧（尽量不去想代码正在做的所有额外工作和复制）会给你一个简单的方法来传递东西，而不必使用指针。你甚至可以这样做：

```
typedef struct
{
    int year;
    int month;
    int day;
    int hour;
    int minute;
    int second;
} TimeStruct;

// Global time values.
int g_year, g_month, g_day, g_hour, g_minute, g_second;

void SetTime (TimeStruct time)
{
    // Pretend we are setting the clock.
    g_year = time.year;
    g_month = time.month;
    g_day = time.day;
    g_hour = time.hour;
    g_minute = time.minute;
    g_second = time.second;
}

TimeStruct GetTime ()
{
    TimeStruct time;

    // Pretend we are reading the clock.
    time.year = g_year;
    time.month = g_month;
    time.day = g_day;
    time.hour = g_hour;
    time.minute = g_minute;
    time.second = g_second;

    return time;
}

TimeStruct time;

time.year = 2023;
time.month = 8;
time.day = 19;
time.hour = 12;
time.minute = 4;
time.second = 20;
SetTime (time);

...

time = GetTime ();
```

现在，一定有一部分 C 程序员在读这篇文章时应该对这里发生的事情感到恐慌。

直至下次…

### 像这样：

加载中...

### *相关*
