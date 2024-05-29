<!--yml
category: 未分类
date: 2024-05-27 14:36:38
-->

# YUI Compressor

> 来源：[https://yui.github.io/yuicompressor/](https://yui.github.io/yuicompressor/)

According to [Yahoo!'s Exceptional Performance Team](http://developer.yahoo.com/performance/), 40% to 60% of Yahoo!'s users have an empty cache experience and about 20% of all page views are done with an empty cache (see [this article by Tenni Theurer on the YUIBlog for more information on browser cache usage](http://yuiblog.com/blog/2007/01/04/performance-research-part-2/)). This fact outlines the importance of keeping web pages as lightweight as possible. Improving the engineering design of a page or a web application usually yields the biggest savings and that should always be a primary strategy. With the right design in place, there are many secondary strategies for improving performance such as minification of the code, [HTTP compression](http://en.wikipedia.org/wiki/Http_compression), using CSS sprites, etc.

In terms of code minification, the most widely used tools to minify JavaScript code are Douglas Crockford's [JSMIN](http://crockford.com/javascript/jsmin), [the Dojo compressor](http://dojotoolkit.org/docs/shrinksafe) and Dean Edwards' [Packer](http://dean.edwards.name/packer/). Each of these tools, however, has drawbacks. JSMIN, for example, does not yield optimal savings (due to its simple algorithm, it must leave many line feed characters in the code in order not to introduce any new bugs).

The goal of JavaScript and CSS minification is always to preserve the operational qualities of the code while reducing its overall byte footprint (both in raw terms and after gzipping, as most JavaScript and CSS served from production web servers is gzipped as part of the HTTP protocol). The YUI Compressor is JavaScript minifier designed to be 100% safe and yield a higher compression ratio than most other tools. Tests on the [YUI Library](http://yuilibrary.com/) have shown savings of over 20% compared to JSMin (becoming 10% after HTTP compression). The YUI Compressor is also able to compress CSS files by using a port of [Isaac Schlueter](http://foohack.com/)'s regular-expression-based CSS minifier.

## Quick Links

*   [Documentation](https://github.com/yui/yuicompressor/blob/master/README.md): Detailed description of the YUI Compressor and how to use it.
*   [Release Notes](https://github.com/yui/yuicompressor/blob/master/doc/CHANGELOG): Detailed change log for the YUI Compressor.
*   [CSS minification](css.html): Description of the CSS minification performed by the compressor.
*   **License:** All code specific to YUI Compressor is issued under a [BSD license](http://opensource.org/licenses/bsd-license.php). YUI Compressor extends and implements code from [Mozilla's Rhino project](http://www.mozilla.org/rhino/). Rhino is issued under the [Mozilla Public License (MPL)](http://www.mozilla.org/MPL/), and MPL applies to the Rhino source and binaries that are distributed with YUI Compressor, including Rhino modifications made by YUI Compressor. YUI Compressor also makes use of and distributes a binary of [JArgs](http://jargs.sourceforge.net/); the JArgs BSD license applies to this binary.
*   [Download](https://github.com/yui/yuicompressor/releases): Download the YUI Compressor.

## How does the YUI Compressor work?

The YUI Compressor is written in [Java](http://java.sun.com/) (requires Java >= 1.4) and relies on [Rhino](http://www.mozilla.org/rhino/) to tokenize the source JavaScript file. It starts by analyzing the source JavaScript file to understand how it is structured. It then prints out the token stream, omitting as many white space characters as possible, and replacing all local symbols by a 1 (or 2, or 3) letter symbol wherever such a substitution is appropriate (in the face of [evil features](http://www.jslint.com/lint.html) such as `eval` or `with`, the YUI Compressor takes a defensive approach by not obfuscating any of the scopes containing the evil statement) The CSS compression algorithm uses a set of finely tuned regular expressions to compress the source CSS file. The YUI Compressor is open-source, so don't hesitate to look at the code to understand exactly how it works.

## Using the YUI Compressor from the command line

```
$ java -jar yuicompressor-x.y.z.jar
Usage: java -jar yuicompressor-x.y.z.jar [options] [input file]

  Global Options
    -h, --help                Displays this information
    --type <js|css>           Specifies the type of the input file
    --charset <charset>       Read the input file using <charset>
    --line-break <column>     Insert a line break after the specified column number
    -v, --verbose             Display informational messages and warnings
    -o <file>                 Place the output into <file> or a file pattern.
                              Defaults to stdout.

  JavaScript Options
    --nomunge                 Minify only, do not obfuscate
    --preserve-semi           Preserve all semicolons
    --disable-optimizations   Disable all micro optimizations

GLOBAL OPTIONS

  -h, --help
      Prints help on how to use the YUI Compressor

  --line-break
      Some source control tools don't like files containing lines longer than,
      say 8000 characters. The linebreak option is used in that case to split
      long lines after a specific column. It can also be used to make the code
      more readable, easier to debug (especially with the MS Script Debugger)
      Specify 0 to get a line break after each semi-colon in JavaScript, and
      after each rule in CSS.

  --type js|css
      The type of compressor (JavaScript or CSS) is chosen based on the
      extension of the input file name (.js or .css) This option is required
      if no input file has been specified. Otherwise, this option is only
      required if the input file extension is neither 'js' nor 'css'.

  --charset character-set
      If a supported character set is specified, the YUI Compressor will use it
      to read the input file. Otherwise, it will assume that the platform's
      default character set is being used. The output file is encoded using
      the same character set.

  -o outfile

      Place output in file outfile. If not specified, the YUI Compressor will
      default to the standard output, which you can redirect to a file.
      Supports a filter syntax for expressing the output pattern when there are
      multiple input files.  ex:
          java -jar yuicompressor.jar -o '.css$:-min.css' *.css
      ... will minify all .css files and save them as -min.css

  -v, --verbose
      Display informational messages and warnings.

JAVASCRIPT ONLY OPTIONS

  --nomunge
      Minify only. Do not obfuscate local symbols.

  --preserve-semi
      Preserve unnecessary semicolons (such as right before a '}') This option
      is useful when compressed code has to be run through JSLint (which is the
      case of YUI for example)

  --disable-optimizations
      Disable all the built-in micro optimizations.
```

**Note:** If no input file is specified, it defaults to stdin.

The following command line (x.y.z represents the version number):

```
$ java -jar yuicompressor-x.y.z.jar myfile.js -o myfile-min.js
```

will minify the file `myfile.js` and output the file `myfile-min.js`. For more information on how to use the YUI Compressor, please refer to the documentation included in the archive.

The charset parameter isn't always required, but the compressor may throw an error if the file's encoding is incompatible with the system's default encoding. In particular, if your file is encoded in utf-8, you should supply the parameter.

```
$ java -jar yuicompressor-x.y.z.jar myfile.js -o myfile-min.js --charset utf-8
```

### Additional notes

*   Don't hesitate to use the `-v` option. Although not a replacement for [JSLint](http://www.jslint.com/lint.html), it will output some helpful hints when it senses that something might be wrong with your code.
*   If you wish to minify your files on the backend (also known as on-the-fly minification) instead of at build time, you will want to cache the minified files in memory for optimal performance (instead of minifying the same files over and over & minification is a time consuming process) Note that the YUI Compressor can easily be instantiated and used from a Java-based environment (Servlet).

## Support & Community

YUICompressor is discussed on the on the [YUICompressor Google Group](https://groups.google.com/forum/#!forum/yuicompressor).

Also be sure to check out [YUIBlog](http://yuiblog.com) for updates and articles about the YUI Library written by the library's developers.

## Filing Bugs & Feature Requests

YUICompressor uses [Github Issues](https://github.com/yui/yuicompressor) to track issues.

### More Reading about JavaScript/CSS minification and the YUI Compressor