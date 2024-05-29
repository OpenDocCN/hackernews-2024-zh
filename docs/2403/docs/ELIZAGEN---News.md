<!--yml
category: 未分类
date: 2024-05-29 12:29:30
-->

# ELIZAGEN - News

> 来源：[https://sites.google.com/view/elizagen-org/news#h.ykbzq5nuiccs](https://sites.google.com/view/elizagen-org/news#h.ykbzq5nuiccs)

## 20190121: Eliza  on ITS on a DEC 10 Emulator (Lars Brinkhoff)

A few days ago I had the pleasure of taking a trip in a real live time machine.

About a week ago I got an out-of-the-blue message from [Lars Brinkhoff](https://github.com/larsbrinkhoff) via [the ElizaGen issue tracker](https://github.com/jeffshrager/elizagen): "I have a version of ELIZA ported to Maclisp, dated 1977-09-05\. Unfortunately, it's just a compiled binary file. Is this of any interest to you?"

Just a binary wouldn't be too interesting, so my initial reaction was muted: "Is there any way to either run it, or extract the embedded code?"

Turns out that Lars had significantly understated the situation: "The binary file is a compiled FASL, fast load file, so it's not easy to extract any code other than PDP-10 instructions. It might be possible, but a lot of work, to reconstruct Lisp code that compiles to approximately the same FASL code."

However, went on: "...we can run it in Maclisp running on a PDP-10 emulator. I have an emulator online."

Eliza on MacLisp on a PDP-10 emulator? Isn't there a level missing here? On what OS?

The answer gave me pause: [MIT's ITS](https://en.wikipedia.org/wiki/Incompatible_Timesharing_System). ITS is one of the most important operating systems in CS history. (Among many other interesting historical facts, the original EMACS was developed on ITS!) I recommend reading about it here: [ITS on Wikipedia](https://en.wikipedia.org/wiki/Incompatible_Timesharing_System). When I was an undergrad and grad student, in the late '70s and early '80s, we used MIT-AI's ITS remotely through the early ARPANet. Interestingly, at one point (I don't know for how long) the ITS at MIT-AI had an Eliza hacked into its command line driver so that unless you prefixed your commands with a ":", the response was from Eliza!

Eliza on MacLisp on ITS on a Dec 10 emulator? Really?!

Yeah, really!!

Here's the emulator: [https://github.com/PDP-10/its](https://github.com/PDP-10/its)

(Lars notes: "...there are more people involved than me. Foremost Eric Swenson who worked on Maclisp and Macsyma back in the day. We [...] update the emulators to support more hardware, fixing bugs, figuring out how to build and run programs, adding new software, Etc.")

If you want to play with a great piece of software history, you can log into this ITS emulator yourself; It's free and open. No password required!

From Lars:

Telnet to "its.pdp10.se, port 10003". Type Control-Z to log in. Then ":lisp games; eliza (init)" to start Maclisp and load ELIZA. It should look something like this:

==================

You: 

        telnet its.pdp10.se 10003

Trying 88.99.191.74...  Connected to pdp10.se.  Escape character is '^]'.

Connected to the KA-10 simulator MTY device, line 4

You:

     type Control-Z

TT ITS.1648\. DDT.1546.  TTY 25  2\. Lusers, Fair Share = 0%  THIS IS TT ITS, A HIGHTY [sic] VOLATILE SYSTEM FOR TESTING

For brief information, type ?  For a list of colon commands, type :? and press Enter.  For the full info system, type :INFO and Enter.

The management apologizes for the use of the slow TK10 terminal  multiplexer.  We'll upgrade to the much faster Datapoint kludge once  it becomes available.

Then you type:

     :lisp games; eliza (init)

(Please Log In)  QUIT  SPEAK UP!

 Et voila. you're typing back through 40+ years of computer history!

 (Use double ENTERs to terminate your sentence to Eliza.)

==================

To test it, I pasted the "standard" example conversation from Wiezenbaum's Eliza paper. Here's a screenshot from the paper:

And here it is, running live on Lars's reborn ITS (the mess at the bottom is because I accidentally pasted the sentence twice, and had to backspace over it. The repeated chars are apparently how ITS displays backspacing):

There's a bit of a mystery about where exactly this version came form. Lars and I theorize that since BBN was right across from the AI lab, and it's known that BBN were users of the ITS systems, this may well be Bernie Cosell's original code!