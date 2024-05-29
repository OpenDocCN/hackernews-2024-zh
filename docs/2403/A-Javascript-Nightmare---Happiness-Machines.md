<!--yml
category: 未分类
date: 2024-05-27 15:03:34
-->

# A Javascript Nightmare | Happiness Machines

> 来源：[https://blog.ignaciobrasca.com/opinion/2024/03/15/javascript-nightmare.html](https://blog.ignaciobrasca.com/opinion/2024/03/15/javascript-nightmare.html)

It was Friday, and everyone was leaving or at most about to.

***Suddenly, during a quick demo, I realized something was off.***

Oh boy, the game started…

After an initial smoke review, I realized the system wasn’t working as expected for exporting files. I exported two files in different formats - one worked, but the other kept loading forever.

Our notification Slack channel started queuing messages, letting us know we’d be here until late night on a Friday.

After my first investigation, without panicking, I realized it wasn’t just files - it was only PDF files. I exported a Word document from our internal system, and it worked. I exported a CSV file, and it worked as well. I started a job for generating a PDF again, but it kept loading forever.

I went to our staging environment and exported one PDF document - it worked.

My head was spinning. Given that I had an ongoing demo, I had to jump out from the problem, so I let Martin (a skilled engineer who has been a part of Datia from almost the beginning) investigate this issue with me.

Up to this point, it was 3:33 PM.

## The Usual Suspect

After a few minutes, we started investigating across our instances.

**“It might be Nginx?”** we checked our servers, and the access log appeared to have errors mentioning an upstream timeout error.

Weird (x1).

We kept investigating until we reached the usual suspect (and something that had caused headaches before) - AWS WAF. WAF is a web application firewall provided by AWS that essentially helps us block, control, and prevent harmful requests from the open web on our public domains.

After a few minutes of digging with it and a few unsuccessful trials, we decided to step further and investigate our buckets. Recently, we had run into a few issues where AWS threw us an error regarding hitting quotas for S3 buckets.

Weird (x2), since we thought the cloud was scalable up to infinite. [When in reality, it’s someone else’s machine.](https://blog.codinghorror.com/the-cloud-is-just-someone-elses-computer/)

We checked the logs and found our first hint: an unauthorized error from AWS after an apparent bottleneck. We looked at each other’s faces in the meeting and said in unison:

**“It must be S3.”**

> Narrator: it wasn’t

## The Confusion

We ran to try it out on a new bucket and wrote a few scripts to test it out. We ran back to staging since it was the environment that used to work before but now suddenly wasn’t working anymore - it kept loading forever.

Our assumption was, **“Okay, we might be hitting another obscure quota from S3.**”

Let’s create a bucket from scratch, but first, let’s try it out against these buckets from a virtual machine.

We did it, and it worked like a charm - same document, even. Now our confusion was extreme, but we had to keep digging like Sherlock Holmes digs into crimes.

### The Bullshit Ecosystem

We ran into the questions that one essentially does not encounter in a serious programming environment [(no pun intended, JavaScript boys!)](https://nadh.in/blog/javascript-ecosystem-software-development-are-a-hot-mess/):

1.  Might the instance be running out of memory, and somehow we’re not throwing an error?
2.  Must be the content of the file that’s breaking, but why don’t we see exceptions and just a timeout?
3.  Is this a bug, but where?
4.  Must be an `undefined` stream?
5.  Or a `null` one?
6.  Is the library we’re using to upload stuff deprecated?

Up to this point, it was 6:40 PM for me (in Sweden) and late for Martin in Argentina as well. We started running in circles.

We decided to zoom in a bit and start testing in a production environment directly since it was outside business hours [(and because I didn’t mention: our locals worked like a charm!)](https://dylanbeattie.net/2017/04/27/it-works-on-my-machine.html)

We modified the code and started using a non-library strategy to upload the stream into S3 ([which, by the way, is messy and not that straightforward](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example_s3_Scenario_UploadStream_section.html)).

A few tries, and still nothing - it worked on our local environments but didn’t work on the cloud.

## The Network

Up to this point, I was getting frustrated since it was Friday, and we were still here, debugging a JavaScript mess. Frustrating enough since it feels like in JavaScript, you never have control over what’s going on.

In C, you can go ahead and misconfigure a makefile to output debug files, a segfault, or a bad reference, but at the end of the day, it’s your fault.

The same goes for any serious language ecosystem out there.

But JavaScript is quite the opposite: **it works like magic (and not as a compliment!) even though you know it’s pure software!**

I ended up believing, “Maybe AWS blocked our public domain, and from there, we could not upload files? What if…”

Anyway, I tried on a virtual machine we have, but nothing - it worked like a charm. The funny thing about this is that during this process, we had three pipelines and workflows for the weekend running in the background, and none of those broke or raised an error.

## The Reunion

Arriving at 8:15 PM, we decided to wrap up for the day and continue individually. Each step we took didn’t work at all.

So I decided to start removing and cleaning the things we did for testing, and I took a five-minute walk to the supermarket.

I came back and said, “Okay, let’s start again.” I ran my local against our production environment and emulated the scenario - it worked, even with our domain.

It wasn’t an issue with S3; AWS was discarded. It must be the documents service or the code…

We discussed a bit more with Martin, chatted, and then I continued with the experiment…

I deleted all cache on my local environment, all node_modules references, all lock files. I ran an installation process once again and exported again… **it didn’t work.**

The next message I sent: “We found it!”

Now it was a matter of understanding what was going on since it wasn’t a server issue but a library issue.

Doing a `git diff`, we could observe a few changes on libraries related to:

```
@smithy/util-stream@2.2.0
@smithy/is-array-buffer@2.1.1
... 
```

All of these because `@aws-sdk` got updated a few months ago… Weird (x3) and extremely closely related.

We kept investigating and found that the actual library we used for rendering files inside the server had changed as well, and it changed… `the streaming pipeline`.

Oh boy, we were close.

We kept digging and decided to downgrade the packages to the latest available version on the internet before these changes. We modified the `package.json` and pushed to production (yeah, up to this point, there was no other way around).

…

After a few minutes, the deploy failed: `github timeout`.

Oh my. Anyway, we started a new workflow, and it worked.

We went to our production environment and checked - it didn’t work, but now it was no longer a timeout but a direct error on a function in charge of handling the file streaming.

We were so close.

Checked the logs:

`this.subset.encodeStream is not a function`

Oh my, holy grail… What now?

I did a quick search, and apparently, if you want to use a specific version of the library with a specific version, you need to use a fixed version for registering fonts in the document…

Anyway, it was around 10 PM, and the only thing we wanted was to close our computers and sleep (for me, I don’t know about Martin).

We did an upgrade and installation of the library, pushed, ran the installation process - it didn’t work.

Well…

After a few minutes, we cleaned up the pipeline cache, restarted our environments, and even cleaned up the load balancer (WTF, just in case).

I pushed again and deployed. It worked.

Up to this point, we weren’t even surprised; we just wanted to leave.

We checked the document - it was all empty except for SVG inside.

I’m thinking about opening a restaurant now.

We checked again, did the entire process from scratch once more, in addition to the other downgrade/upgrade actions we took, we had to upgraded a library we didn’t think of was related with this mess, and now - it worked. The document was there, and now everyone could have a PDF.

The system was **stable** again.

Our sanity might not be.

### The Catharsis

It’s almost incredible to believe that this ecosystem is so broad and spread out across the globe. It’s awful, and the developer experience is close to being a paranoid android trying not to shoot himself in the foot with a bazooka.

JavaScript sometimes feels too malleable, too easy, too clean, but it’s not. The paybacks come later in the game.

Now our strategy will surely be to use at least a typed system or a more robust runtime. This mess cannot be there any longer, and if it is, JavaScript won’t be the tool to support it (since it’s strictly a bad tool).

## Conclusion

It feels like you need to know too many nitty gritty details (and the devil lies in those details!) to truly understand what’s going on - and it’s not about being an expert, but an expert on the minute particulars.

Why didn’t this library work in conjunction with the latest version of AWS? Who knows? Not even the developers of the library might know, because the JavaScript ecosystem is so badly interconnected that everyone relies on blindly trusting each other’s code and infrastructure. It’s not even close to the experience of using robust package managers like Cargo (Rust), NuGet (C#), or others.

JS works, and it works well for a lot of things. But don’t try to build a truly scalable system with it. It’s simply too difficult to avoid shooting yourself in the foot entirely when the ecosystem is this fragile and delicate. One tiny version mismatch or breaking change can bring your mission-critical application to a standstill, as we painfully experienced.

The JavaScript world moves too fast, with dependencies changing constantly and breaking backwards compatibility. This makes it extremely high-risk for any project aiming to be an enterprise-grade, long-lasting solution. The deck is stacked against you when you have to wrestle with the ever-changing currents of the npm universe and its nitty gritty issues.

It feels like you’re constantly swimming against the current, needing to be a domain expert on the most granular details just to keep your application afloat. It’s an unnecessarily high cognitive burden that often outweighs the benefits JavaScript provides.

Don’t get me wrong, JavaScript has it’s a good tool sometimes. But we’ve learned the hard way that it simply isn’t the right tool for building robust platforms that need to stand the test of time.

The ecosystem is a beautiful mess, but a mess nonetheless. For mission-critical applications, the costs of dealing with its chaos and nitty gritty issues are too high.

At this point, it’s not even about JavaScript itself, but about the fundamental lack of trust a developer can have with its tools. Imagine a carpenter with a hammer that drives nails most of the time, but then randomly and unpredictably strikes the carpenter instead.

Why would that carpenter want to use such an unreliable and hazardous hammer for every piece of furniture they build? It makes no sense. And yet, that’s the situation we find ourselves in with JavaScript.

I don’t know if upcoming projects like Bun.js or Deno will do anything to rectify the situation. I simply believe the JavaScript language itself has become too amorphous and monstrous to not consistently breed confusion across builds and deployments.