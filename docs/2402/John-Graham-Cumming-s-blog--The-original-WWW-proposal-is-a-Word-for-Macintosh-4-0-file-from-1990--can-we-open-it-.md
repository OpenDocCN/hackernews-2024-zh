<!--yml
category: 未分类
date: 2024-05-27 14:49:26
-->

# John Graham-Cumming's blog: The original WWW proposal is a Word for Macintosh 4.0 file from 1990, can we open it?

> 来源：[https://blog.jgc.org/2024/02/the-original-www-proposal-is-word-for.html](https://blog.jgc.org/2024/02/the-original-www-proposal-is-word-for.html)

The W3C has a page with the [original WWW proposal](https://www.w3.org/History/1989/proposal.html) from Tim Berners-Lee. One of the downloads says 

The "I can't test it" made me sad. There are two other files (an RTF version and an HTML version generated in 1998 from the original file). But can we open the original document?

The original document is 68,608 bytes and file on my Mac says it's a Microsoft Word for Macintosh 4.0 file. That matches with TBL's note on the W3C page saying: "A hand conversion to HTML of the original MacWord (or Word for Mac?) document written in March 1989 and later redistributed unchanged apart from the date added in May 1990." 

Microsoft Office for Mac came out in 1989 with System 6.0\. That was Microsoft Word 4.0 so we're looking for compatibility with Microsoft Word for Macintosh 4.0\. Let's see what modern software can open this. What I really want to be able to do is open it and convert it to, say, PDF with high fidelity.

## Microsoft Word

Let's begin with Microsoft Word itself. I uploaded the file to Microsoft OneDrive with the extension .doc and clicked on it to open it in Microsoft Word.

## Apple Pages

I switched to the Mac and hoped that Apple Pages might understand an old Microsoft Word for Macintosh file. No such luck.

## Apache OpenOffice

Next let's hope open source software will come to the rescue. I downloaded the latest Apache OpenOffice and it did open the file but the formatting is gone and the diagrams are missing.

## LibreOffice

OK, maybe I need different open source software, so I switched to the latest

[LibreOffice](https://www.libreoffice.org/)

and it opened it. And the diagrams are crisp! Although there's something weird about the margins and there are other formatting problems.

## CERN PDF

CERN makes available

[a PDF version](https://cds.cern.ch/record/369245/files/dd-89-001.pdf)

of the proposal which was apparently created in 1998 using Acrobat Distiller Daemon 2.1 for SunOS/Solaris (SPARC). It has 20 pages. The LibreOffice imported version has 24 pages. 

To get an overview of what's different I created a PDF from the LibreOffice version and then looked at it and the CERN PDF in the contact sheet version in Apple Preview.

Here's the CERN PDF:

Here's the LibreOffice-generated PDF:

Things that are different:

1\. The right-hand margin is missing in the LibreOffice version.

2\. The LibreOffice version is using 14 pt vs. 12 pt for most of the text.

3\. The LibreOffice version has turned headers with TBL's initials in them into footers.

4\. The page breaks look in the right places (see how the images are correctly placed towards the end); thus it's probably the font size that's the biggest problem.

5\. There CERN PDF has a space under the heading and the LibreOffice version does not.

## Emulation

To make sure that I knew what the actual original document looked like I decided to use

[Infinite Mac](https://infinitemac.org/1990/System%206.0.5)

to boot a 1990-era Macintosh and run actual Word for Macintosh 4.0 on the original document.

That way I can see actual fonts, font sizes and layout to confirm how the document should have looked. And that's where it became obvious that the original document on the original Mac and the CERN PDF are quite different. The CERN PDF has 20 pages. On the Mac running Word for Macintosh 4.0 with A4 paper it has 22 pages. So I decided to aim to get us close to the original document on the Mac.

So... set to A4 paper and set right margin to same size as left margin. Change the first page format to be different since it doesn't have the same gutters, footers or headers. Manually change the body text from 14 pt (and other sizes) to 12 pt. Manually deal with text that breaks across pages incorrectly and other alignment problems. Fix the footer that should be a header.

In the end I got pretty close to what's visible on the Mac. 

Converting this document from its original format was a bit of a victory for open source software. And a lesson in how hard document preservation is. To help preserve it a bit, and in an open format, I've uploaded my .odt version to GitHub

[here](https://github.com/jgrahamc/www-proposal)

. It's interesting, and a little disheartening to see that this 34 year old document is difficult to open, and even when opened the resulting output isn't exactly the same as the original.

PS If you're wondering why I ever started this project. I just wanted a high quality version of the diagrams in the original proposal for a presentation. Took me a lot longer than I thought it would.

PPS A

[comment](https://news.ycombinator.com/item?id=39358074)

on Hacker News pointed out that I could probably either create a PostScript file or a PDF via an emulated Mac. I was able to boot another Mac (System 7) that had Word from 1992 and Print2PDF (a driver that creates a printer that makes a PDF file) and print directly from Word for Macintosh 5.1a.

I've added the generated PDF file to the GitHub. This version has 20 pages and the fonts are different but it does meet my original requirement of a PDF.

PPPS A Hacker News

[comment](https://news.ycombinator.com/item?id=39363611)

links to another conversion done using different versions of Word and bit of fiddling around to get a really nice version of the document in modern formats.