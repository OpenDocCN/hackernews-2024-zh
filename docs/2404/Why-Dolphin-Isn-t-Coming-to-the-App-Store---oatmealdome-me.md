<!--yml
category: 未分类
date: 2024-05-27 13:22:50
-->

# Why Dolphin Isn't Coming to the App Store - oatmealdome.me

> 来源：[https://oatmealdome.me/blog/why-dolphin-isnt-coming-to-the-app-store/](https://oatmealdome.me/blog/why-dolphin-isnt-coming-to-the-app-store/)

Two weeks ago, Apple modified their App Store guidelines to allow retro game emulators in the App Store. This week, Delta, a multi-system emulator that was previously only available via AltStore, [was released on the App Store](https://apps.apple.com/us/app/delta-game-emulator/id1048524688).

Since these events happened, we’ve been asked many times if we will submit DolphiniOS (our fork of Dolphin) to the App Store.

**Unfortunately, no.**

Apple still does not allow us to use a vital technology that is necessary for Dolphin to run with good performance: JIT.

## What is JIT?

The GameCube and Wii have a PowerPC-based CPU inside them. All modern Apple devices use an ARM-based CPU. It isn’t possible to directly run PowerPC code on an ARM CPU, and vice versa. Therefore, if we want to run a GameCube or Wii game on an iPhone, it is necessary to translate the game’s PowerPC code to ARM so that the CPU can understand it.

Dolphin uses something called a Just-in-Time (JIT) recompiler to achieve this. Whenever the emulated console wants to run game code, Dolphin will use its JIT to translate the PowerPC code to ARM, and then execute the results.

## JIT on iOS

Unfortunately, Apple generally does not allow apps to use JIT recompilers on iOS. The only exceptions are Safari and alternative web browsers in Europe.

We submitted a DMA interoperability request to Apple for JIT support, but Apple denied the request a few weeks ago.

It’s hard to tell exactly why Apple is so hesitant to open up JIT support. It’s possible that they consider it to be a security risk. (Looking at the various restrictions and limitations placed on JavaScript JITs for alternative web browsers in Europe, they do appear to be worried about its potential to be abused.)

## Dolphin without JIT?

It is technically possible to run Dolphin without its JIT recompiler. When doing so, Dolphin uses something called an “interpreter” to run the PowerPC code.

Unfortunately, the interpreter is many times slower than the JIT recompiler.

We’ve attached two videos of DolphiniOS running below, so you can judge the performance difference for yourself. One uses the interpreter, and the other uses JIT.

Without JIT (using Interpreter)

With JIT

As you can see, it is basically unplayable. These clips were even recorded on an iPhone 15 Pro Max, the highest end iPhone currently available.

While we could submit DolphiniOS to the App Store with just the interpreter, we would likely get endless complaints from users about the poor performance. App Review might also just reject us anyway because the app is unusable.

## Conclusion

We’d love to release DolphiniOS on the App Store or work with the Dolphin Emulator project to get an official build on the App Store.

Unfortunately, as of right now, it isn’t possible unless Apple loosens their restrictions on JIT.