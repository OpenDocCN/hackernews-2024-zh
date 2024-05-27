<!--yml
category: 未分类
date: 2024-05-27 13:25:59
-->

# Windows saved techie from moment of security madness • The Register

> 来源：[https://www.theregister.com/2024/04/22/who_me/](https://www.theregister.com/2024/04/22/who_me/)

Who, Me? It's Monday once again, dear reader, and you know what that means: another dive into the Who, Me? confessional, to share stories of IT gone wrong that *Reg* readers managed to pretend had gone right.

This week, meet a reader we'll Regomize as "Declan" who describes himself as "a designer working with CAD to design machines" – but one with enough technical nous that he essentially taught himself how to use the software. Indeed, at his first job in the '90s he was considered the "technical guy" and the go-to for support.

The consultancy he was working for had a bunch of Windows machines, a few Unix boxes – mostly from Sun and Silicon Graphics – plus one Digital Alpha RISC machine, running the special cut of Windows NT Microsoft made for those boxes. For those who don't remember that short-lived project, it was basically Microsoft's half-hearted gesture to the idea that there were non-Intel processors in the world. Most of the work on porting NT to Alpha was done by Digital, and when that firm was bought by Compaq the dream was over.

Anyway, long story short, Windows NT ran on Alpha, but not hugely well, and there were virtually no native applications for it – almost everything ran in emulation. However at the time the 500MHz Alpha was sufficiently speedy that performance was adequate. That was the machine Declan used.

Declan's varied duties (on top of his CAD work) had him installing Windows, supporting the network, and above all trying to stop his co-workers spreading viruses.

Declan understood that a virus hitting his employer's network would probably bring work to a halt, because the antivirus software of the time was mostly updated only after infections were widespread. He was therefore most diligent in instructing everyone to be cautious about opening attachments or unfamiliar files from untrusted sources.

Then as now, prevention was superior to cure.

Also then as now, people make mistakes.

One tired, stressed afternoon, Declan received an email which contained an Excel spreadsheet. Thinking it looked legit enough, he double-clicked to open it.

Immediately his screen was obscured by error messages. He panicked, realizing that the spreadsheet must have contained a macro virus. Then he realized that he was about to take the company down by doing exactly what he warned everyone else not to do. Oh, the shame! The humiliation!

After a moment, though, a calm descended upon him as he read the error messages.

The spreadsheet was indeed infected. However, the worm it contained was designed to open the recipient's contact list in Outlook and send itself to everyone, thus perpetuating its spread – for no reason other than to spread further.

Wasn't the world nice before ransomware?

What this worm had not counted on was that Declan was running both Excel and Outlook in emulation, and the integration between them in that environment was so poor that it couldn't work. The worm kept trying to create emails and send them, but failed every single time.

Declan managed to stop the flood of error messages on his own machine, and realized he would never ever have to tell anyone about his tragic lapse in vigilance.

If you've got something you've always wanted to get off your chest, tell us all about it in [an email to Who, Me?](mailto:whome@theregister.com) and we'll cleanse your soul – anonymously, of course. ®