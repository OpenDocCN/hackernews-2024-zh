<!--yml
category: 未分类
date: 2024-05-27 13:38:59
-->

# pdf.tocgen

> 来源：[https://krasjet.com/voice/pdf.tocgen/](https://krasjet.com/voice/pdf.tocgen/)

<main>

```
 in.pdf                                    
                            │                                       
     ┌──────────────────────┼────────────────────┐                  
     │                      │                    │                  
     ▽                      ▽                    ▽                  
┌──────────┐  recipe  ┌───────────┐   ToC   ┌──────────┐            
│ pdfxmeta ├─────────▷│ pdftocgen ├────────▷│ pdftocio ├───▷ out.pdf
└──────────┘          └───────────┘         └──────────┘ 
```

[pdf.tocgen](https://sink.krj.st/pdf.tocgen/) is a set of command-line tools for automatically extracting and generating the table of contents (ToC) of a PDF file. It uses the embedded font attributes and position of headings to deduce the basic outline of a PDF file.

# An overview

For example, for the PDF version of Paul Graham’s book *On Lisp*, available for download on [his website](http://www.paulgraham.com/onlisptext.html) but comes *without* a table of content, we can use the `pdfxmeta` command to build a recipe file,

In [1]

```
`$ pdfxmeta --auto 1 --page 14 onlisp.pdf "Extensible" >> recipe.toml` 
```

In [2]

```
`$ pdfxmeta --auto 2 --page 14 onlisp.pdf "^Design" >> recipe.toml` 
```

In [3]

```
`$ sed '/^#.*/d' < recipe.toml` 
```

Out [3]

```
`[[heading]]
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 19.92530059814453

[[heading]]
level = 2
greedy = true
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

saved as `recipe.toml`. Then use `pdftocgen` to automatically generate a table of contents for the book, extracted using the recipe.

In [4]

```
`$ pdftocgen onlisp.pdf < recipe.toml` 
```

Out [4]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
[--snip--]` 
```

We could save the output to a file called `toc`,

In [5]

```
`$ pdftocgen onlisp.pdf < recipe.toml > toc` 
```

and import it to the original PDF file using the `pdftocio` command, saving it as `output.pdf`.

In [6]

```
`$ pdftocio -o output.pdf onlisp.pdf < toc` 
```

This is just an overview of the basic workflow. Please read [section 4](#a-worked-example) for a detailed explanation and walk-through.

pdf.tocgen works best for PDF files produces from a  document using `pdftex` (and its friends `pdflatex`, `pdfxetex`, etc.), but it’s designed to work with any *software-generated* PDF filesThat is, you shouldn’t expect it to work with scanned PDFs. Some examples include `troff`/<wbr>`groff`, Adobe InDesign, Microsoft Word, and probably more.

pdf.tocgen is a free software. The source code can be found in the [sink](https://sink.krj.st/pdf.tocgen/) or on [GitHub](https://github.com/Krasjet/pdf.tocgen/) and is licensed under the GPLv3 license. You are free to tinker with the source code, but any derivatives *must* guarantee the [freedom](https://www.gnu.org/philosophy/free-sw.en.html) of users. If you want to contribute to this project, send a [patch](https://sink.krj.st) or open a pull request on [GitHub](https://github.com/Krasjet/pdf.tocgen/).

# Installation

pdf.tocgen is written in Python 3. It is known to work with Python 3.7 to 3.11 on Linux, Windows, and macOSOn BSDs, you probably need to build PyMuPDF yourself. Use

In [7]

```
`$ pip install -U pdf.tocgen` 
```

to install the latest version systemwide. Alternatively, use [`pipx`](https://pipxproject.github.io/pipx/) or

In [8]

```
`$ pip install -U --user pdf.tocgen` 
```

to install it for the current user. I would recommend the latter approach to avoid messing up the package manager on your system.

If you are using an Arch-based Linux distro, the package is also available on [AUR](https://aur.archlinux.org/packages/pdf.tocgen/). It can be installed using any AUR helper, for example [`yay`](https://github.com/Jguer/yay):

In [9]

```
`$ yay -S pdf.tocgen` 
```

# Note to LaTeX users

Before we continue, note that this tool targets *readers*, not *authors*.

The intended usage is to generate a table of contents for [lecture notes](https://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf), [draft books](https://felleisen.org/matthias/HtDC/htdc.pdf), or possibly the papers on [arXiv](https://arxiv.org/) for *easier navigation*. If you are a  user, please *do not* use this tool to generate table of contents for your own manuscripts.

Use the [hyperref](https://ctan.org/pkg/hyperref) package instead,

since it understands your document better and provides more customizations.

# A worked example

The design of pdf.tocgen is influenced by the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy). I intentionally separated pdf.tocgen to 3 separate programs. They work together, but each of them is useful on its own.

They represents the 3 steps necessary to add table of contents to a PDF file:

1.  `pdfxmeta`: extract the metadata (font attributes, positions) of headings to build a recipe file.
2.  `pdftocgen`: generate a table of contents from the recipe.
3.  `pdftocio`: import the table of contents to the PDF document.

Again, we will use Paul Graham’s book *On Lisp*, freely available on [his website](http://www.paulgraham.com/onlisptext.html) as a PDF but comes without a table of contents embedded, to demonstrate how these 3 programs work together.

If you want to follow along, go ahead and download the book and save it as `onlisp.pdf`.

## Step 1: Build a recipe

`pdftocgen` extracts the headings using the font attributes and position (bounding box) of text embedded in a PDF file. We need to supply it with a recipe, which is a [TOML](https://toml.io) file that tells `pdftocgen` what a heading, subheading, or subsubheading should look like.

The recipe for our example, *On Lisp*, which I have already mentioned before, looks like this:

```
`# filter for level 1 heading
[[heading]]
# heading level
level = 1
# attributes to search for
font.name = "Times-Bold"
font.size = 19.92530059814453

# filter for level 2 heading
[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

A recipe is a list of filters, each of which specifies the attributes `pdftocgen` should look for. For example, in the recipe above, a level 1 heading, which corresponds to chapter titles in *On Lisp*, should

1.  Have font name matching `"Times-Bold"`
2.  Have font size `19.92530059814453`

Of course, the font size don’t have to be this precise, you could set a tolerance level using the `font.size_tolerance` attributes:

```
`[[heading]]
level = 1
font.name = "Times-Bold"
# match anything between 19.9-20.1
font.size = 20
font.size_tolerance = 0.1` 
```

How do we know what attributes a heading should have? This is why we need to use `pdfxmeta` to extract the metadata of text in a PDF file.

Open up the PDF file you just downloaded in your favorite PDF reader. We will use [zathura](https://pwmt.org/projects/zathura/) here, but use anything you are comfortable with.

In [10]

```
`$ zathura onlisp.pdf &` 
```

Scroll down to page 14, counting from the title page. This is where the first chapter’s title “The Extensible Language” is. To look at the metadata associated with this title, use the `pdfxmeta` command:

In [11]

```
`$ pdfxmeta -p 14 onlisp.pdf "The Extensible"` 
```

Out [11]

```
`The Extensible Language:
 font.name = "Times-Bold"
 font.size = 19.92530059814453
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 306.947998046875
 bbox.right = 354.4866638183594
 bbox.bottom = 334.5445251464844` 
```

The `-p` flag tells `pdfxmeta` that it should only search on page 14. It is not required, but I highly recommend specifying it to make the search less ambiguous.

Now that we have extracted the metadata for chapter titles, we could make it a filter by copy-pasting it, or redirect the output, to a recipe file called `recipe.toml`.

In [12]

```
`$ pdfxmeta -p 14 onlisp.pdf "The Extensible" >> recipe.toml` 
```

Use your favorite editor to open up the recipe file and remove the indentationsIn Vim, you could press `<<` in normal mode to dedent a line, and add the `[[heading]]` header and the `level` attribute to specify the heading level.

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
bbox.left = 138.60000610351562
bbox.top = 306.947998046875
bbox.right = 354.4866638183594
bbox.bottom = 334.5445251464844` 
```

This is already a valid recipe file, but it is too specific. It is very unlikely that other chapter titles would match all four bounding box (`bbox`) values, which means they would have exactly the same position and width as this chapter’s title.

If you want to ignore an attribute, simply remove it from the filter. From my experience, `font.name` and `font.size` is usually enough.

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453` 
```

If you are lazy, you can also use the `--auto` or `-a` flag to format the output as a heading filter with the default settings. But the output will be slightly harder to read:

In [13]

```
`$ pdfxmeta -a 1 -p 14 onlisp.pdf "The Extensible"` 
```

Out [13]

```
`[[heading]]
# The Extensible Language
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 19.92530059814453
# font.size_tolerance = 1e-5
# font.color = 0x000000
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 138.60000610351562
# bbox.top = 306.947998046875
# bbox.right = 354.4866638183594
# bbox.bottom = 334.5445251464844
# bbox.tolerance = 1e-5` 
```

The argument after the `-a` flag is the heading level of the output heading filter, which in this case is `1`. Don’t worry about the `greedy` option right now, it is not necessary for this book.

Next, we need to extract the metadata of level 2 headings:

In [14]

```
`$ pdfxmeta -p 14 -i onlisp.pdf "^design"` 
```

Out [14]

```
`Design by Evolution:
 font.name = "Times-Bold"
 font.size = 11.9552001953125
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 165.42388916015625
 bbox.top = 624.02880859375
 bbox.right = 268.2960205078125
 bbox.bottom = 640.5867309570312` 
```

Note that we could use regular expressions (Python-style) as the pattern. The `-i` option can be used to enable case-insensitive search.

Use the `-a` flag to dump it to `recipe.toml`

In [15]

```
`$ pdfxmeta -a 2 -p 14 -i onlisp.pdf "^design" >> recipe.toml` 
```

and pick the attributes we need:

```
`[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

If you can’t find want you are looking for, leave out the query string to dump the entire page,

In [16]

```
`$ pdfxmeta -p 14 onlisp.pdf` 
```

Out [16]

```
`1:
 font.name = "Times-Bold"
 font.size = 24.906600952148438
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 239.2274932861328
 bbox.right = 151.05331420898438
 bbox.bottom = 273.72314453125
The Extensible Language:
 font.name = "Times-Bold"
 font.size = 19.92530059814453
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 306.947998046875
 bbox.right = 354.4866638183594
 bbox.bottom = 334.5445251464844
[--snip--]` 
```

but it is very likely that `pdftocgen` can’t generate a meaningful table of contents for you if you can’t find the heading using `pdfxmeta`. Sorry about that.

As a side node, sometimes you would see the `+` symbol in `font.name`, for example in the PDF draft of [*How to design Classes*](https://felleisen.org/matthias/HtDC/htdc.pdf) by Matthias Felleisen et al.In the latest version, the subset will be automatically stripped off, so you might not see this anymore

In [17]

```
`$ pdfxmeta -p 19 htdc.pdf "The Varieties"` 
```

Out [17]

```
`The Varieties of Data:
 font.name = "UFRVLO+Palatino-Bold"
 font.size = 17.21540069580078
 font.color = 0x221f1f
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 293.6400146484375
 bbox.top = 129.75328063964844
 bbox.right = 459.6872253417969
 bbox.bottom = 146.77931213378906` 
```

The `UFRVLO+` in `font.size` represent a *subset* in the complete font family `Palatino-Bold`. The font in different headings might be distributed into multiple subsets, so it is *almost always* a good idea to match against the entire family by removing the subset prefix:

```
`[[heading]]
level = 1
font.name = "Palatino-Bold"
font.size = 17.21540069580078
font.color = 0x221f1f` 
```

You could also use the `-a` flag as before to handle this automatically.

In [18]

```
`$ pdfxmeta -a 1 -p 19 htdc.pdf "The Varieties"` 
```

Out [18]

```
`[[heading]]
# The Varieties of Data
level = 1
greedy = true
font.name = "Palatino-Bold"
font.size = 17.21540069580078
# font.size_tolerance = 1e-5
# font.color = 0x221f1f
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 293.6400146484375
# bbox.top = 129.75328063964844
# bbox.right = 459.6872253417969
# bbox.bottom = 146.77931213378906
# bbox.tolerance = 1e-5` 
```

Also, note that `font.name` can take a regular expression:

```
`[[heading]]
level = 3
font.name = "(Palatino-Bold|Palatino-Italic)"
font.size = 11.9552001953125
font.color = 0x221f1f` 
```

You might need some experiments to determine the best recipe for the PDF. You could find some pre-made recipes [here](https://github.com/Krasjet/pdf.tocgen/tree/master/recipes), but in most cases you would need to design your own recipes. You are welcome to contribute more recipes by sending patches or pull requests.

## Step 2: Generate table of contents

Now that we have crafted a recipe for *On Lisp* in `recipe.toml`,

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453

[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

we could use the `pdftocgen` command to generate the table of contents for our PDF

In [19]

```
`$ pdftocgen onlisp.pdf < recipe.toml` 
```

Out [19]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
"Functional Programming" 41
 "3.1 Functional Design" 41
 "3.2 Imperative Outside-In" 46
 "3.3 Functional Interfaces" 48
 "3.4 Interactive Programming" 50
[--snip--]` 
```

The output of `pdftocgen` is a dialect of CSV. The main differences are

1.  Every *4 spaces* of indent represents a new heading level so a level 3 heading is *8 spaces* of indent
2.  The separator is a single space, not comma
3.  Titles need to be quoted using double quotes.

This format is intentionally designed to be easily *edited* (in Vim), since the output of `pdftocgen` is *expected to be* inaccurate in many cases and you are likely to tweak the table of contents before you import it to the original PDF file.

If a heading contains the actual double quote character (`"`), use two double quotes (`""`) to escape it,

```
"a ""quoted"" heading" 2
```

but the double quotes in most PDFs are usually smart quotes (`“` and `”`), so escaping is rarely necessary.

If you don’t want to import the table of contents to PDF, you could instead use the `-H` flag to print the table of contents in a more readable format,

In [20]

```
`$ pdftocgen -H onlisp.pdf < recipe.toml` 
```

Out [20]

```
`Preface ··· 5
 Bottom-up Design ··· 5
 Plan of the Book ··· 7
 Examples ··· 9
 Acknowledgements ··· 9
Contents ··· 11
The Extensible Language ··· 14
 1.1 Design by Evolution ··· 14
 1.2 Programming Bottom-Up ··· 16
 1.3 Extensible Software ··· 18
 1.4 Extending Lisp ··· 19
 1.5 Why Lisp (or When) ··· 21
Functions ··· 22
 2.1 Functions as Data ··· 22
 2.2 Defining Functions ··· 23
 2.3 Functional Arguments ··· 26
 2.4 Functions as Properties ··· 28
 2.5 Scope ··· 29
 2.6 Closures ··· 30
 2.7 Local Functions ··· 34
 2.8 Tail-Recursion ··· 35
 2.9 Compilation ··· 37
 2.10 Functions from Lists ··· 40
Functional Programming ··· 41
 3.1 Functional Design ··· 41
 3.2 Imperative Outside-In ··· 46
 3.3 Functional Interfaces ··· 48
 3.4 Interactive Programming ··· 50
[--snip--]` 
```

but this format can’t be read by `pdftocio`.

For now, we will save the table of contents to a file called `toc`:

In [21]

```
`$ pdftocgen onlisp.pdf < recipe.toml > toc` 
```

## Step 3: Import the ToC to PDF

To import the generated table of contents to the original PDF file, simply redirect the `toc` file to `pdftocio`.

In [22]

```
`$ pdftocio onlisp.pdf < toc` 
```

The output PDF document is named `onlisp_out.pdf`Use the `-o` flag if you want a different file name. When you open it up in zathura

In [23]

```
`$ zathura onlisp_out.pdf` 
```

and press the `TAB` key, you should see the table of contents has been successfully imported to the PDF file.

In fact, if you don’t want to edit the table of contents, the output of `pdftocgen` can be directly piped into `pdftocio`

In [24]

```
`$ pdftocgen onlisp.pdf < recipe.toml | pdftocio onlisp.pdf` 
```

I call this program `pdftocio` because it can also be used to *output* the existing table of contents of a PDF document if you don’t supply any external input

In [25]

```
`$ pdftocio onlisp_out.pdf` 
```

Out [25]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
"Functional Programming" 41
 "3.1 Functional Design" 41
 "3.2 Imperative Outside-In" 46
 "3.3 Functional Interfaces" 48
 "3.4 Interactive Programming" 50
[--snip--]` 
```

or with the `-H` flag to display it in a more readable format.

In [26]

```
`$ pdftocio -H onlisp_out.pdf` 
```

Out [26]

```
`Preface ··· 5
 Bottom-up Design ··· 5
 Plan of the Book ··· 7
 Examples ··· 9
 Acknowledgements ··· 9
Contents ··· 11
The Extensible Language ··· 14
 1.1 Design by Evolution ··· 14
 1.2 Programming Bottom-Up ··· 16
 1.3 Extensible Software ··· 18
 1.4 Extending Lisp ··· 19
 1.5 Why Lisp (or When) ··· 21
Functions ··· 22
 2.1 Functions as Data ··· 22
 2.2 Defining Functions ··· 23
 2.3 Functional Arguments ··· 26
 2.4 Functions as Properties ··· 28
 2.5 Scope ··· 29
 2.6 Closures ··· 30
 2.7 Local Functions ··· 34
 2.8 Tail-Recursion ··· 35
 2.9 Compilation ··· 37
 2.10 Functions from Lists ··· 40
Functional Programming ··· 41
 3.1 Functional Design ··· 41
 3.2 Imperative Outside-In ··· 46
 3.3 Functional Interfaces ··· 48
 3.4 Interactive Programming ··· 50
[--snip--]` 
```

## A summary

Although the example above gets quite long, the essential steps are very simple.

First, search for metadata of headings using `pdfxmeta`

In [27]

```
`$ pdfxmeta -a 1 -p page in.pdf "Section" >> recipe.toml
$ pdfxmeta -a 2 -p page in.pdf "Subsection" >> recipe.toml` 
```

Edit the `recipe.toml` file to pick out the attributes you need, or leave it at default if you don’t need any customizations:

In [28]

```
`$ vim recipe.toml # edit` 
```

Then use the recipe to generate a table of contents and import it to the PDF file

In [29]

```
`$ pdftocgen in.pdf < recipe.toml | pdftocio -o out.pdf in.pdf` 
```

Or if you want to edit the table of contents before importing it,

In [30]

```
`$ pdftocgen in.pdf < recipe.toml > toc
$ vim toc # edit
$ pdftocio in.pdf < toc` 
```

Each of the three programs has some extra functionalities. Use the `-h` option to see all the options you could pass in.

# One more example: Mathematics

pdf.tocgen’s recipe format provides some convenient options for handling math symbols in headings. They are especially useful for handling lecture notes, textbook, or papers in mathematics.

Let us look at another example, the [lecture notes](https://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf) for EE261 in Stanford, written by Prof. Brad Osgood. This document also doesn’t come with a table of contents, and the headings contains many math symbols such as , , , etc.

These math symbols do not have the same font as the surrounding text, which can be a problem if we specify the headings using `font.name`:

```
`[[heading]]
level = 1
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 3
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

The recipe above is generated using

In [31]

```
`$ pdfxmeta -a 1 -p 7 ft.pdf "Fourier Series" >> ft.toml` 
```

In [32]

```
`$ pdfxmeta -a 2 -p 7 ft.pdf "Introduction" >> ft.toml` 
```

In [33]

```
`$ pdfxmeta -a 3 -p 28 ft.pdf "punchline" >> ft.toml` 
```

with comments cleaned up.

If we pass the recipe file `pdftocgen`, you would notice that some of the math symbols are missing from the title.

In [34]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

Out [34]

```
`EE 261 ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

Pay attention to the title of section 1.5, 1.12, and 1.17. Comparing it with page 3 of the PDF, you should notice that the math symbols are missing from the headings.

There are several options to deal with this issue.

## Option 1: greedy filter

This is the recommended approach, as you might have noticed. It is the default setting for the `-a` option of `pdfxmeta`, because it’s simple and it works the best among all the options.

A greedy filter will extract any text from the *enclosed* or *surrounding* region of the heading, even if some parts don’t match the current filter.

This is exactly our case here, since a math symbol in headings is usually surrounded by plain text matching the filter, but they are in a different font themselves.

If you want to make a heading filter greedy, just add

```
greedy = true
```

to the corresponding heading filter in the recipe file. For example, for the recipe above, we have

```
`[[heading]]
level = 1
greedy = true
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
level = 2
greedy = true
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 3
greedy = true
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

Now the recipe should extract the math symbols in the headings automatically.

In [35]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

Out [35]

```
`EE 261 ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at c ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best L 2  Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for L 2 ([0 , 1]) ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

Of course, it still needs some further clean up, but it should be easy using a decent text editor.

## Option 2: Regex

Alternatively, since `font.name` takes an regular expression, we could supply multiple font name using the `(a|b)` syntax. For example, we could first use GNU or BSD `grep` to find the metadata of a math symbol:

In [36]

```
`$ pdfxmeta -p 16 ft.pdf | grep -A 25 "Lost at"` 
```

Out [36]

```
`Lost at:
 font.name = "HYPGHW+CMBX12"
 font.size = 14.346190452575684
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 87.12625122070312
 bbox.top = 510.04034423828125
 bbox.right = 137.03665161132812
 bbox.bottom = 524.40087890625
 c:
 font.name = "SYTZSX+CMMIB10"
 font.size = 14.346190452575684
 font.color = 0x000000
 font.superscript = false
 font.italic = true
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 137.03665161132812
 bbox.top = 510.04034423828125
 bbox.right = 150.005615234375
 bbox.bottom = 524.3865356445312` 
```

Now that we know the font name of `c` is `CMMIB10`, we could modify the filter to include it.

```
`[[heading]]
level = 2
font.name = "(CMBX12|CMMIB10)"
font.size = 14.346190452575684` 
```

However, since superscripts usually have different font sizes than the surrounding text, this approach doesn’t work here.

## Option 3: Multiple filters

Another option is to use multiple filters for the same heading level. Internally, matched text of the same level will be collapsed into one if they are in the same block.

For the example in the previous section, it is equivalent to write

```
`[[heading]]
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 2
font.name = "CMMIB10"
font.size = 14.346190452575684` 
```

but this allows us to specify different font sizes.

Using the same approach as in previous section, we could find the metadata of all the math symbols in the text and create separate heading filters for them, which gives us the following recipe:

```
`[[heading]]
level = 1
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
# text
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
# math
level = 2
font.name = "(CMMIB10|CMMI12|CMR12)"
font.size = 14.346190452575684

[[heading]]
# superscript
level = 2
font.name = "CMR10"
font.size = 9.962639808654785
font.superscript = true

[[heading]]
level = 3
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

This filter is able to extract all the math symbols,

In [37]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

Out [37]

```
`EE 261 ··· 1
 Prof. Brad Osgood Electrical Engineering Department Stanford University ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1 ··· 13
 1 ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at c ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best L 2 Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for L 2 ([0 , 1]) ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

but it is less accurate and introduces many unnecessary symbols, but they should be relatively easy to clean up in post processing.

When in doubt, just use option 1 and make the filter greedy. Options 2 and 3 are only the last resort.

# The recipe file

Except all the metadata generated by `pdfxmeta`, there are a few other options you could specify in the recipe file. Here is a list of all the valid keys for each heading filter.

```
`# filter for level 1 heading
[[heading]]
# heading level
level = 1
# makes this filter a *greedy* filter, useful for PDFs
# containing many math symbols
greedy = true
font.name = "CMBX12"
# matches font.size ± font.size_tolerance
font.size = 14.346199989318848
font.size_tolerance = 1e-5
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
# matches bbox.left ± bbox.tolerance
bbox.left = 157.98439025878906
# matches bbox.top ± bbox.tolerance
bbox.top = 335.569580078125
# matches bbox.right ± bbox.tolerance
bbox.right = 477.66058349609375
# matches bbox.bottom ± bbox.tolerance
bbox.bottom = 349.93011474609375
bbox.tolerance = 1e-5

# filter for level 2 heading
[[heading]]
level = 2
greedy = false
font.name = "CMBX10"
font.size = 9.962599754333496
font.size_tolerance = 1e-5
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
bbox.left = 168.76663208007812
bbox.top = 127.2930679321289
bbox.right = 280.66656494140625
bbox.bottom = 137.2556610107422
bbox.tolerance = 1e-5

# ...` 
```

Again, don’t be too specific about bounding boxes, since headings generally have different widths. Use the `-a` option of `pdfxmeta` if you are not sure what attributes to pick.

# Command examples

Because of the modularity of design, each program is useful on its own, despite being part of the pipeline. This section will provide some more examples on how you could use them. Feel free to come up with more.

## `pdftocio`

`pdftocio` should best demonstrate this point, this program can do a lot on its own.

To display existing table of contents in a PDF to `stdout`:

In [38]

```
`$ pdftocio doc.pdf` 
```

Out [38]

```
`"Level 1 heading 1" 1
 "Level 2 heading 1" 1
 "Level 3 heading 1" 2
 "Level 3 heading 2" 3
 "Level 2 heading 2" 4
"Level 1 heading 2" 5` 
```

To write existing table of contents in a PDF to a file named `toc`:

In [39]

```
`$ pdftocio doc.pdf > toc` 
```

To write a `toc` file back to `doc.pdf`:

In [40]

```
`$ pdftocio doc.pdf < toc` 
```

To specify the name of output PDF:

In [41]

```
`$ pdftocio -o out.pdf doc.pdf < toc` 
```

To copy the table of contents from `doc1.pdf` to `doc2.pdf`:

In [42]

```
`$ pdftocio  -v  doc1.pdf  |  pdftocio  doc2.pdf` 
```

Note that the `-v` flag helps preserve the vertical positions of headings during the copy.

To print the table of contents for reading:

In [43]

```
`$ pdftocio -H doc.pdf` 
```

Out [43]

```
`Level 1 heading 1 ··· 1
 Level 2 heading 1 ··· 1
 Level 3 heading 1 ··· 2
 Level 3 heading 2 ··· 3
 Level 2 heading 2 ··· 4
Level 1 heading 2 ··· 5` 
```

## `pdftocgen`

If you have obtained an existing recipe `rcp.toml` for `doc.pdf`, you could apply it and print the outline to `stdout` by

In [44]

```
`$ pdftocgen  doc.pdf  <  rcp.toml` 
```

Out [44]

```
`"Level 1 heading 1" 1
 "Level 2 heading 1" 1
 "Level 3 heading 1" 2
 "Level 3 heading 2" 3
 "Level 2 heading 2" 4
"Level 1 heading 2" 5` 
```

To output the table of contents to a file called `toc`:

In [45]

```
`$ pdftocgen doc.pdf < rcp.toml > toc` 
```

To import the generated table of contents to the PDF file, and output to `doc_out.pdf`:

In [46]

```
`$ pdftocgen doc.pdf < rcp.toml | pdftocio -o doc_out.pdf doc.pdf` 
```

To print the generated table of contents for reading:

In [47]

```
`$ pdftocgen -H doc.pdf < rcp.toml` 
```

Out [47]

```
`Level 1 heading 1 ··· 1
 Level 2 heading 1 ··· 1
 Level 3 heading 1 ··· 2
 Level 3 heading 2 ··· 3
 Level 2 heading 2 ··· 4
Level 1 heading 2 ··· 5` 
```

If you want to include the vertical position in a page for each heading, use the `-v` flag

In [48]

```
`$ pdftocgen -v doc.pdf < rcp.toml` 
```

Out [48]

```
`"Level 1 heading 1" 1 306.947998046875
 "Level 2 heading 1" 1 586.3488159179688
 "Level 3 heading 1" 2 586.5888061523438
 "Level 3 heading 2" 3 155.66879272460938
 "Level 2 heading 2" 4 435.8687744140625
"Level 1 heading 2" 5 380.78875732421875` 
```

`pdftocio` can understand the vertical position in the output to generate table of contents entries that link to the exact position of the heading, instead of the top of the page.

In [49]

```
`$ pdftocgen -v doc.pdf < rcp.toml | pdftocio doc.pdf` 
```

Note that the default output of `pdftocio` here is `doc_out.pdf`.

To search for `Anaphoric` in the entire PDF

In [50]

```
`$ pdfxmeta onlisp.pdf "Anaphoric"` 
```

Out [50]

```
`14\. Anaphoric Macros:
 font.name = "Times-Bold"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 308.6400146484375
 bbox.top = 307.1490478515625
 bbox.right = 404.33282470703125
 bbox.bottom = 320.9472351074219
[--snip--]` 
```

To output the result as a heading filter with the automatic settings,

In [51]

```
`$ pdfxmeta -a 1 onlisp.pdf "Anaphoric"` 
```

Out [51]

```
`[[heading]]
# 14\. Anaphoric Macros
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 9.962599754333496
# font.size_tolerance = 1e-5
# font.color = 0x000000
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 308.6400146484375
# bbox.top = 307.1490478515625
# bbox.right = 404.33282470703125
# bbox.bottom = 320.9472351074219
# bbox.tolerance = 1e-5
# [--snip--]` 
```

which can be directly write to a recipe file:

In [52]

```
`$ pdfxmeta -a 1 onlisp.pdf "Anaphoric" >> recipe.toml` 
```

To case-insensitive search for `Anaphoric` in the entire PDF:

In [53]

```
`$ pdfxmeta -i onlisp.pdf "Anaphoric"` 
```

Out [53]

```
`to compile-time. Chapter 14 introduces anaphoric macros, which allow you to:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 295.6583557128906
 bbox.right = 459.0260009765625
 bbox.bottom = 308.948486328125
[--snip--]` 
```

Use regular expression to case-insensitive search search for `Anaphoric` in the entire PDF:

In [54]

```
`$ pdfxmeta onlisp.pdf "[Aa]naphoric"` 
```

Out [54]

```
`to compile-time. Chapter 14 introduces anaphoric macros, which allow you to:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 295.6583557128906
 bbox.right = 459.0260009765625
 bbox.bottom = 308.948486328125
[--snip--]` 
```

To search only on page 203:

In [55]

```
`$ pdfxmeta -p 203 onlisp.pdf "anaphoric"` 
```

Out [55]

```
`anaphoric if, called:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 283.17822265625
 bbox.right = 214.81094360351562
 bbox.bottom = 296.4683532714844
[--snip--]` 
```

To dump the entire page of 203:

In [56]

```
`$ pdfxmeta -p 203 onlisp.pdf` 
```

Out [56]

```
`190:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 126.09941101074219
 bbox.right = 153.54388427734375
 bbox.bottom = 139.38951110839844
[--snip--]` 
```

To dump the entire PDF document:

In [57]

```
`$ pdfxmeta onlisp.pdf` 
```

Out [57]

```
`i:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 458.0400085449219
 bbox.top = 126.09941101074219
 bbox.right = 460.8096008300781
 bbox.bottom = 139.38951110839844
[--snip--]` 
```

# Development

If you want to modify the source code or contribute anything, first install [`poetry`](https://python-poetry.org/), which is a dependency and package manager for Python used by pdf.tocgen. Then run

in the root directory of the repository to set up development dependencies.

If you want to test the development version of pdf.tocgen, add

```
poetry run
```

prefix to each commands, for example

In [59]

```
`$ poetry run pdfxmeta in.pdf "pattern"` 
```

Alternatively, you could also use the

command to open up a virtual environment and run the development version directly:

In [61]

```
`(pdf.tocgen) $ pdfxmeta in.pdf "pattern"` 
```

Before you send a patch or pull request, make sure the unit test passes by running:

# GUI front end

If you are a Emacs user, you could install Daniel Nicolai’s [toc-mode](https://github.com/dalanicolai/toc-mode) package as a GUI front end for pdf.tocgen, though it offers many more functionalities, such as extracting (printed) table of contents from a PDF file. Note that it uses pdf.tocgen under the hood, so you still need to install pdf.tocgen before using toc-mode as a front end for pdf.tocgen.

# License

pdf.tocgen itself a is free software. The source code of pdf.tocgen is licensed under the GNU GPLv3 license. However, the recipes in the `recipes` directory is separately licensed under the [CC BY-NC-SA 4.0 License](https://creativecommons.org/licenses/by-nc-sa/4.0/) to prevent any commercial usage, and thus not included in the distribution.

pdf.tocgen is based on [PyMuPDF](https://github.com/pymupdf/PyMuPDF), licensed under the GNU GPLv3 license, which is again based on MuPDF, licensed under the GNU AGPLv3 license. A copy of the AGPLv3 license is included in the repository.

If you want to make any derivatives based on this project, please follow the terms of the GNU GPLv3 license.

</main>