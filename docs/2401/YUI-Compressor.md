<!--yml

类别：未分类

日期：2024-05-27 14:36:38

-->

# YUI压缩器

> 来源：[https://yui.github.io/yuicompressor/](https://yui.github.io/yuicompressor/)

根据[Yahoo！的卓越性能团队](http://developer.yahoo.com/performance/)的说法，40%至60%的Yahoo！用户经历了空缓存体验，大约20%的页面浏览是在空缓存状态下完成的（有关浏览器缓存使用的更多信息，请参阅[Tenni Theurer在YUIBlog上的这篇文章](http://yuiblog.com/blog/2007/01/04/performance-research-part-2/)）。这个事实突显了尽可能保持网页轻量级的重要性。改进页面或Web应用的工程设计通常会带来最大的节省，这应该始终是主要策略。有了正确的设计，还有许多改进性能的次要策略，例如代码的最小化，[HTTP压缩](http://en.wikipedia.org/wiki/Http_compression)，使用CSS精灵等等。

在代码最小化方面，最广泛使用的工具是Douglas Crockford的[JSMIN](http://crockford.com/javascript/jsmin)，[Dojo压缩器](http://dojotoolkit.org/docs/shrinksafe)和Dean Edwards的[Packer](http://dean.edwards.name/packer/)。然而，这些工具都有缺点。例如，JSMIN不能产生最佳的节省（由于其简单的算法，它必须在代码中留下许多换行字符，以免引入任何新的错误）。

JavaScript和CSS最小化的目标始终是保持代码的操作特性，同时减少其总体字节占用量（无论是原始术语还是经过gzip压缩后的字节占用量，因为大多数从生产Web服务器提供的JavaScript和CSS都是作为HTTP协议的一部分进行gzip压缩的）。YUI压缩器是一个设计为100%安全并且比大多数其他工具具有更高压缩比的JavaScript压缩器。对[YUI库](http://yuilibrary.com/)的测试显示，与JSMin相比可以节省超过20%的空间（在HTTP压缩后变为10%）。YUI压缩器还能够使用[Isaac Schlueter](http://foohack.com/)的基于正则表达式的CSS压缩器端口来压缩CSS文件。

## 快速链接

+   [文档](https://github.com/yui/yuicompressor/blob/master/README.md)：YUI压缩器的详细描述以及如何使用它。

+   [发布说明](https://github.com/yui/yuicompressor/blob/master/doc/CHANGELOG)：YUI压缩器的详细变更日志。

+   [CSS最小化](css.html)：压缩器执行的CSS最小化描述。

+   **许可证：** YUI 压缩器特定的所有代码都使用[BSD许可证](http://opensource.org/licenses/bsd-license.php)发布。YUI 压缩器扩展和实现了[Mozilla的Rhino项目](http://www.mozilla.org/rhino/)中的代码。Rhino根据[Mozilla公共许可证(MPL)](http://www.mozilla.org/MPL/)发布，并且MPL适用于与YUI压缩器一起分发的Rhino源代码和二进制文件，包括YUI压缩器所做的Rhino修改。YUI 压缩器还使用并分发[JArgs](http://jargs.sourceforge.net/)的二进制文件；JArgs的BSD许可证适用于该二进制文件。

+   [下载](https://github.com/yui/yuicompressor/releases)：下载 YUI 压缩器。

## YUI 压缩器是如何工作的？

YUI 压缩器是用[Java](http://java.sun.com/)编写的（需要Java >= 1.4），并依赖[Rhino](http://www.mozilla.org/rhino/)对源JavaScript文件进行标记化处理。它首先分析源JavaScript文件来了解其结构。然后打印出标记流，尽量省略尽可能多的空格字符，并在适当的情况下用1（或2、或3）个字母符号替换所有本地符号（面对[恶意功能](http://www.jslint.com/lint.html)如`eval`或`with`，YUI 压缩器采取了防御性的方法，不会混淆包含恶意语句的任何范围）。CSS压缩算法使用一组精调的正则表达式来压缩源CSS文件。YUI 压缩器是开源的，所以不要犹豫查看代码，以了解它的运作方式。

## 使用 YUI 压缩器的命令行

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

**注意：** 如果未指定输入文件，则默认为stdin。

以下命令行（x.y.z代表版本号）：

```
$ java -jar yuicompressor-x.y.z.jar myfile.js -o myfile-min.js
```

将会压缩文件`myfile.js`，并输出文件`myfile-min.js`。有关如何使用 YUI 压缩器的更多信息，请参考存档中附带的文档。

字符集参数并不总是必须的，但如果文件的编码与系统默认编码不兼容，压缩器可能会抛出错误。特别是如果你的文件编码为utf-8，你应该提供这个参数。

```
$ java -jar yuicompressor-x.y.z.jar myfile.js -o myfile-min.js --charset utf-8
```

### 附加说明

+   不要犹豫使用`-v`选项。虽然不能取代[JSLint](http://www.jslint.com/lint.html)，但当它感知到你的代码可能出错时，它会输出一些有用的提示。

+   如果你希望在后端（也称为即时压缩）对文件进行压缩而不是在构建时，你会希望将压缩后的文件缓存到内存中以获得最佳性能（而不是一遍又一遍地压缩相同的文件，压缩是一个耗时的过程）。请注意，YUI 压缩器可以轻松在基于Java的环境（Servlet）中实例化和使用。

## 支持与社区

YUICompressor在[YUICompressor Google Group](https://groups.google.com/forum/#!forum/yuicompressor)上进行了讨论。

还请务必查看 [YUIBlog](http://yuiblog.com)，以获取由该库的开发人员撰写的有关 YUI 库的更新和文章。

## 提交错误和功能请求

YUICompressor 使用 [Github Issues](https://github.com/yui/yuicompressor) 来跟踪问题。

### 关于 JavaScript/CSS 最小化和 YUI 压缩器的更多阅读
