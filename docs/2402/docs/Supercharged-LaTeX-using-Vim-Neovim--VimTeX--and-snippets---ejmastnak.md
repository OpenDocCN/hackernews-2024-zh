<!--yml
category: 未分类
date: 2024-05-27 15:03:59
-->

# Supercharged LaTeX using Vim/Neovim, VimTeX, and snippets | ejmastnak

> 来源：[https://www.ejmastnak.com/tutorials/vim-latex/intro/](https://www.ejmastnak.com/tutorials/vim-latex/intro/)

# A guide to supercharged mathematical typesetting

This is a tutorial series to help you set up the Vim or Neovim text editors for efficiently writing math in LaTeX. Here is an example of what I have in mind:

The blue bar with white text shows the keys I am typing, the bottom shows the resulting LaTeX source code, and the top is the compiled output. More on how this works below.

<details class="my-8 py-2.5 px-4 border border-gray-500 dark:border-gray-500 rounded-lg overflow-hidden bg-blue-50 dark:bg-blue-900"><summary class="text-sky-600 dark:text-neutral-300 font-medium cursor-pointer">Wait, what are you talking about, what is LaTeX?</summary>[LaTeX](https://www.latex-project.org/)

is the industry standard typesetting software for writing papers, books, reports, etc. in mathematics, physics, computer science, and other quantitative sciences (but is mostly unknown outside this niche, so it’s quite reasonable to have never heard of it). LaTeX has a reputation for producing high-quality documents but being clumsy to type—this series presents a framework aimed at eliminating the clumsiness.</details> 

**Goal of this guide:** make writing LaTeX as easy (fast, efficient, enjoyable…) as writing math by hand. Tech stack: the Vim text editor using the UltiSnips or LuaSnip snippet plugin and the VimTeX plugin’s LaTeX editing features. The series should help if you…

*   are interested in taking real-time lecture notes using LaTeX, à la [Gilles Castel](https://castel.dev/),
*   want a LaTeX experience decidedly more pleasant and efficient than whatever you were probably first taught, whether your motivation is real-time university lecture speed or not,
*   hope to switch to Vim from a different LaTeX editor, but are unsure how to proceed, or
*   just want to browse someone else’s workflow and configuration out of curiosity.

**What it costs you:** everything in the guide is free, but it will cost you time and effort. You can skim through the guide in about 15-30 minutes; a closer read-through might take a few hours; and you’ll realistically need a few weekends (or perhaps a few weeks if you’re new to Vim) of dedicated focus and effort to become fully functional. From that point reaching the speed in this page’s GIFs would probably take months of practice.

## Contents

1.  [**Prerequisites**](/tutorials/vim-latex/prerequisites/)
    Covers prerequisites for getting the most out of the series, along with references that should get you up to speed if needed.

2.  [**UltiSnips**](/tutorials/vim-latex/ultisnips/) or [**LuaSnip**](/tutorials/vim-latex/luasnip/)
    Explains snippets, the key to real-time LaTeX. Both articles cover the same content—once using the UltiSnips plugin and once using the LuaSnip plugin.

3.  [**Vim’s ftplugin system**](/tutorials/vim-latex/ftplugin/)
    Introduces Vim’s filetype plugin system, which will help you understand the VimTeX plugin.

4.  [**The VimTeX plugin**](/tutorials/vim-latex/vimtex/)
    The excellent VimTeX plugin is *the reason* to use Vim over another LaTeX editor.

5.  [**Compilation**](/tutorials/vim-latex/compilation/)
    How to compile LaTeX documents from within Vim.

6.  [**PDF reader**](/tutorials/vim-latex/pdf-reader/)
    How to integrate Vim and a PDF reader for viewing LaTeX documents.

7.  [**Vim configuration**](/tutorials/vim-latex/vimscript/)
    A Vim configuration guide explaining the key mappings and Vimscript functions used in this tutorial.

## More about the series

### Shut up and show me results

As concrete evidence that the techniques in this tutorial work, here are [1500+ pages of typeset physics notes](/notes/fmf/fmf/) from my undergraduate studies, most of them written during university lecture in real time (although grammar and style were improved later). Here are some examples of what these notes look like:

And here are more GIFs showing that LaTeX can be written at handwriting speed:

This is actually a little *faster* than I can write by hand—try taking out a pencil and paper and see if you can keep up! (Yes, I know I’m cheating by throwing in a bunch of hard-to-handwrite integrals.) If you like, you can see [**more examples on YouTube**](https://www.youtube.com/watch?v=P7iMX1lqGnU).

**Credit where credit is due**: the above GIFs are inspired by Gilles Castel’s video [Fast LaTeX editing with Vim and UltiSnips](https://www.youtube.com/watch?v=a7gpx0h-BuU)—it is beautifully done and I encourage you to watch it.

### The original Vim-LaTeX article

By the way: the seminal work on the subject of Vim and LaTeX, and my inspiration for attempting and ultimately succeeding in writing real-time LaTeX using Vim, is Gilles Castel’s [*How I’m able to take notes in mathematics lectures using LaTeX and Vim*](https://castel.dev/post/lecture-notes-1/). You’ve probably seen it on the Internet if you dabble in Vim or LaTeX circles, and you should definitely read it if you haven’t yet.

This series builds on Castel’s article by more thoroughly walking you through the technical implementation details (e.g. the details of setting up a PDF reader with forward and inverse search, how to use the VimTeX plugin, how to write Vimscript functions and key mappings, how Vim’s `ftplugin` system works, how to compile LaTeX documents, and so on).

### Config

Here is an overview of the setup used in this series:

### Feedback, suggestions, etc.

If you have ideas for improving the series, I will quite likely implement them, appreciate your input, and give you a shoutout for your contributions. Feedback is welcome and appreciated.

Shoutouts to previous readers: many thanks to [@rltyty](https://github.com/rltyty), [Carl von Randow](https://github.com/carlvr), Ehud Gordon, Andrey Rukhin, [Merlin Büge](https://github.com/camoz), [Albert Gu](https://github.com/albertfgu), Pano Otis, Jason Yao, [@Glirastes](https://github.com/Glirastes), [Daniele Avitabile](https://www.danieleavitabile.com/), Kai Breucker, Maxwell Jiang, [@lodisy](https://github.com/lodisy), and [@subnut](https://github.com/subnut) for catching mistakes and offering good ideas on how to improve this series.

You can reach me by email at [[email protected]](/cdn-cgi/l/email-protection#a0c5ccc9cac1cee0c5cacdc1d3d4cec1cb8ec3cfcd) or by opening an issue or pull request at [github.com/ejmastnak/ejmastnak.com](https://github.com/ejmastnak/ejmastnak.com).

### Want to say thank you?

You could:

*   [Send me an email!](/contact/) Seriously, if this material helped you, it will make my day to know. I love hearing from readers, and you’ll almost certainly get a message back from me.

*   [Contribute financially.](https://www.buymeacoffee.com/ejmastnak) Based on reader input, there are in fact people out there interested in compensating me financially for this guide. That’s awesome—thank you! You can [Buy Me a Coffee here.](https://www.buymeacoffee.com/ejmastnak)