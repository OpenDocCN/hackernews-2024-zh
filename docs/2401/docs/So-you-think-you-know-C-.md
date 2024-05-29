<!--yml
category: 未分类
date: 2024-05-27 14:56:47
-->

# So you think you know C?

> 来源：[https://wordsandbuttons.online/so_you_think_you_know_c.html](https://wordsandbuttons.online/so_you_think_you_know_c.html)

A lot of programmers claim they know C. Well, it has the most famous syntax, it has been there for 44 years, and it’s not cluttered with obscure features. It’s easy!

If you think it is — take this test. It only has 5 questions. Every question is basically the same: what the return value would be? And each question has a choice of four answers, of which one and only one is right.

So, the right answers are:

You scored 0 points, sorry. You don't know what you don't know. You scored 1 point, which is not so bad. You know that there is something you don't know. You scored 2 points, which is not so bad. You know that there are things you don't know. You scored 3 points, which is rather well. You know that you don't know. You scored 4 points, which is excellent. You know what you don't know. You scored 5 points, congratulations! You know exactly what you don't know.

And yes, the right answer to every question is “I don’t know”.

Let’s untangle them one by one now.

**First one** is actually about structure padding. C compiler knows that storing unaligned data in RAM may be costly, so it pads your data for you. If you have 5 bytes of data in a structure, it will probably make it 8\. Or 16\. Or 6\. Or whatever it wants. There are extensions like GCC attributes aligned and packed that let you get some control over this process, but they are non-standard. C itself does not define padding attributes, so the right answer is: “I don’t know”.

The **second one** is about integer promotion. It’s only reasonable that the type of short int and an expression with the largest integer being short int would be the same. But the reasonable doesn’t mean right for C. There is the rule that every integer expression gets promoted to int. Actually, it’s much more complicated than that. Take a peek at the standard, you’ll enjoy it.

But even so, we don’t compare types, we compare sizes. And the standard only guarantees that the range of short int should not exceed int. They ranges may be equal, and the size of the *short int* may even be grates than the size of *int* because of the padding bits. So the right answer is: “I don’t know”.

The **third one** is all about dark corners. Starting from that neither integer overflows, nor char type sign are defined by the standard. First one is undefined behavior, the second is implementation specific. But even more, the size of the *char* type itself is not specified in bits either. Historically, there were platforms where it was 6 bits (remember trigraphs?), and there are platforms today where all five integer types are 32 bits. Even the value of the space character is implementation defined. Without all these details specified, every speculation about the result is invalid, so the answer is: “I don’t know”.

The **fourth one** looks tricky, but it’s not that hard in retrospective since you already know that int size is not directly specified in the standard. It can easily be 16 bits, then the very first operation will cause the over-shift and that’s plain undefined behavior. It’s not C fault, on some platforms, it is even undefined in assembly, so the compiler simply can’t give you valid guarantees without eating up a lot of performance.

So once again “I don’t know” is the right answer.

And the **last one** is classic. The result depends on undefined behavior: [whether the post-increment is deferred and which argument is fetched first](https://gynvael.coldwind.pl/?id=372). It might work just like you expect on one platform and might fail easily on the other. Usually, it just evaluates to *2*, so you get used to it until one day it doesn't. That’s the problem with undefined things. When you meet one, the right answer is always: “I don’t know”.

## P. S.

And at this point, I only have to apologize. The test is clearly provocative and may even be a little offensive. I’m sorry if it causes any aggravation.

The thing is, I learned C in roughly 1998, and for the whole 15 years thought that I’m good at it. It was my language of choice in college, and I’ve done some successful projects in C on my first full-time job, and even then, when I was mostly working with C++, I thought of it as over-bloated C.

The pivoting moment came in 2013, when I’ve got myself involved with some safety-critical PLC programming. It was a research project in nuclear power plant automation, where absolutely no underspecification was tolerable. I had to learn that, while I did know a lot about C programming, the absolute majority of what I knew was false. And I had to learn it the hard way too.

Eventually, I had to learn to rely on the standard instead of folklore; to trust measurements and not presumptions; to take “things that simply work” skeptically, — I had to learn an engineering attitude. This is what matters the most, not some particular WAT anecdotes.

I only hope this little test would help someone like myself from the past to learn this attitude in some 15 minutes and not 15 years.