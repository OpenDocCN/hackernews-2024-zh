<!--yml
category: 未分类
date: 2024-05-27 15:13:18
-->

# HyperDisk Blog - Too many Mac apps are being built with Electron

> 来源：[https://hyperdiskapp.com/blog/too-many-mac-apps-are-being-built-with-electron/](https://hyperdiskapp.com/blog/too-many-mac-apps-are-being-built-with-electron/)

This test gets into the concept of “feel,” as in the way an app feels when you run it. Efficiently written apps feel as if there’s nothing standing between you and the computer, no gunky layer to gum up the works and slow things down.

Bloated or badly-programmed apps often suffer from poor launch times. In this test, there’s no contest between BBEdit’s 1 second launch time and WhatsApp’s 16 seconds on a 2021 MacBook Pro.

#### Test 2: Memory

It’s a common mistake to conflate “memory” and “disk space,” but they're different things. Memory is RAM—it has to do with how much stuff you can run at once. Disk space has to do with how much stuff you can store on your computer.

BBEdit, while open in a typical working scenario, is using 173 MB of memory. That’s no small potatoes, but it’s a pretty small chunk of the 8-32 GB that’s installed on most Macs today.

WhatsApp, on the other hand, is using 110 MB. Wait, what? That’s less than BBEdit... that blows up my whole... oh wait. Never mind, WhatsApp is running 5 weird “helper processes” and using about 1.4 GB in total. That’s actually worse than I expected.

Let’s take a step back once again, and process the fact that a texting app is consuming Photoshop-league memory, *just to display your text messages*. This is Electron Apps, and they must be stopped.

And while we’re at it, let’s talk about those helper processes. Their names are WhatsApp Helper, WhatsApp Helper, WhatsApp Helper (GPU), WhatsApp Helper (Renderer) and WhatsApp Helper (Renderer). A Mac app shouldn't need *any* of these. They’re probably communicating with each other using sockets or ports, or maybe shared files. This really isn’t how you’re supposed to build a Mac app. This shouldn’t become normalized. As Scotty said in Star Trek III, “the more they overthink the plumbing, the easier it is to stop up the drain.”