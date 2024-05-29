<!--yml
category: 未分类
date: 2024-05-27 14:56:26
-->

# C and returning values quickly or safely. But not both. | Sub-Etha Software

> 来源：[https://subethasoftware.com/2024/01/18/c-and-returning-values-quickly-or-safely-but-not-both/](https://subethasoftware.com/2024/01/18/c-and-returning-values-quickly-or-safely-but-not-both/)

**WARNING:** This article contains a C coding approach that many will find uncomfortable.

In my day job as a mild-mannered embedded C programmer, I am usually too busy maintaining what was created before me to be creating something new for others to maintain after me. There was that one time I had two weeks that were very different, and fun, since they were almost entirely spent “creating” versus “maintaining.”

Today’s quick C tidbit is about getting parameters back from a C function. In C, you only get one thing back — typically a variable type like an int or float or whatever:

```
int GetTheUltimateAnswer()
{
    return 42;
}

int answer = GetTheUltimateAnswer();
print ("The Ultimate Answer is %d\n", answer);
```

If you need more than one thing returned, it is common to pass in variables by reference (the address of, or pointer to, the variable in memory) and have the function modify that memory to update the variables:

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

The moment pointers come in to play, things get very dangerous. But fast.

When passing values in, they get copied in to a new variable:

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

Try it: [https://onlinegdb.com/WC3ihCAuj](https://onlinegdb.com/WC3ihCAuj)

Above, Function() gets a new variable (called “x” in this case) with the value of the variable that was passed in to the call. The function is like Las Vegas. Anything that happens to that variable inside the function stays inside the function – the variable disappears at the end of the function, while the original variable remains as-was.

C++ changes this, I have learned, so you can pass in variables that can be modified, but I am not a C++ programmer so this post is only about old-skool C.

## Pointing to a variable’s memory

By passing in the address of a variable, the function can go to that memory and modify the variable. It will be changed:

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

Try it: [https://onlinegdb.com/Y2Z9WUvFG](https://onlinegdb.com/Y2Z9WUvFG)

Passing by value is slower, since a new variable has to be created. Passing by reference just passes an address and the code uses that address – no new variable is created.

But, using a reference for just for speed is dangerous because the function can modify the variable even if you didn’t want it to. Consider passing in a string buffer, which is a pointer to a series of character bytes:

```
void PrintError (char *message)
{
    print ("ERROR: %s\n", message);
}

PrintError ("Human Detected");
```

We do this all the time, but since PrintError() has access to the memory passed in, it could try to modify it. If we passed in a constant string like “Human Detected”, that string would typically be in program memory (though this is not true for Harvard Architecture systems like the PIC and Arduino). At best, an operating system with memory protection would trap that access with an exception and kill the program. At worst, the program would self-modify (which was the case when I learned this on OS-9/6809 back in the late 80s — no memory protection on my TRS-80 CoCo!).

```
void PrintError (char *message)
{
    message[0] = 42;
}

PrintError ("Human Detected");
```

Above would likely crash, though if the user had passed in the buffer holding a string, it would just be modified:

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

Try it: [https://onlinegdb.com/L50JRWYj](https://onlinegdb.com/L50JRWYj_)

## And your point is?

My point is — there are certainly times when speed is the most important thing, and it outweighs the potential problems/crashes that could be caused by a bug with code using the pointer. Take for example anything that passes in a buffer:

```
void UppercaseString (char *buffer)
{
    for (int idx=0; idx<strlen(buffer); idx++)
    {
        buffer[i] = toupper(buffer[I])
    }
}
```

There are many bad things that could happen here. By using “strlen”, the buffer MUST be a string that has a NIL (‘\0’) byte at the end. This routine could end up trampling through memory uppercasing bytes that are beyond the caller’s string.

It is wise to always add another parameter that is the max size of the buffer:

```
void UppercaseString (char *buffer, int bufferSize)
{
    for (int idx=0; idx<bufferSize; idx++)
    {
        buffer[i] = toupper(buffer[I])
    }
}
```

That helps. But it is still up to the compiler to catch the wrong type of pointer being passed in.

```
int Number = 10;

UppercaseString (&Number, 100);
```

The compiler should not let you do that, but some may just issue a warning and build it anyway. (This is why I always try to have NO warnings in my code. The more warnings there are, the more likely you will start ignoring them.)

## Try #1: Passing by Reference

Suppose we have a function that returns the date and time as individual values (year, month, day, hour, minute and second). Since we cannot get six values back from a function, we first try passing in six variables by reference and having the routine modify them:

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

That works fine … as long as you know the parameters are “ints” (whatever that is) and not shorts or longs or any other numeric type. This, for example, would be bad:

```
short year, month, day, hour, minute, second;

GetDateTime1 (&year, &month, &day, &hour, &minute, &second);
```

Above, we are passing in a short (let’s say that is a 16-bit variable on this system) in to a function that expects an int (let’s say that is a 32-bit signed variable on this system). The function would try to place 32-bits of information at the address of a 16-bit value.

Bad things, as they say, can happen.

## Try #2: Passing a structure by reference

Passing in six variable pointers is more work than passing in one, so if we put the values in a structure we could pass in just the pointer to that structure. This has the benefit of making sure the structure is only loaded with values it can handle (unlike passing in an address of something that might be 8, 16, 32 or 64 bits).

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

This should greatly reduce the potential problems since you only have one pointer to screw up, and if you get the type correct (a TimeStruct) the values it contains should be fine since the compiler takes care of trying to set a “uint8_t” to “65535” (a warning, hopefully, and storing 8-bits of that 16-bit value as a “loss of precision”).

## Try #3: Returning the address of a static

An approach various standard C library functions take is having some fixed memory allocated inside the function as a static variable, and then returning a pointer to that memory. The user doesn’t make it and therefore isn’t passing in a pointer that could be wrong.

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

This approach is better, since it gets the speed from using a pointer, and the safety of not being able to get the pointer wrong since the function tells you where it is, not the other way around.

BUT … once you have the address of that static memory, you can modify it.

```
TimeStruct *timePtr;
timePtr = GetDateTime3 ();
timePtr->year = 1969;
```

In a real Date/Time function (like the one in the C library), those variables are populated with the system time when you call the function, so even if the user changed something like this, it would be set back to what it was the next time it was called. But, I can see where there could be issues with other types of functions that just hold on to memory like this.

Plus, it’s always holding on to that memory whether anyone is using it or not. That is a no-no when working on memory constrained systems like an Arduino with 4K of RAM.

## Try #4: Returning a copy of a structure

And now the point of today’s ramblings… I rarely have used this, since it’s probably the slowest way to do things, but … you don’t just have to return a date type like and int or a bool or a pointer. You can return a structure, and C will give the caller a copy of the structure.

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

Above is possibly the safest way to return data, since no pointers are used. The called makes an new structure variable, and then the function creates a new structure variable and the return copies that structure in to the caller’s structure.

Try it: [https://onlinegdb.com/F6rR1V-xb](https://onlinegdb.com/F6rR1V-xb)

This is slower, and consumes more memory during the process of making all these copies, BUT it’s far, far safer. Even ChatGPT agrees that, if going to “safe” code, this is the better approach.

And, at my day job, I experimented with this and it’s been working very well. It’s about the closest thing C has to “objects”. I even use it for a BufferStruct so I can pass a buffer around without using a pointer (though internally there is a pointer to the buffer memory). It looks something like this:

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

The extra overhead may be a problem if you are coding for speed, but doing this trick (while trying not to think about all the extra work and copying the code is doing) gives you a simple way to pass things around without ever using a pointer. You could even do this:

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

And now a certain percentage of C programmers who stumble in to this article should be having night terrors at what is going on here.

Until next time…

### Like this:

Like Loading...

### *Related*