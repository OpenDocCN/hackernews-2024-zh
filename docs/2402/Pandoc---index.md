<!--yml

category: 未分类

date: 2024-05-27 14:32:52

-->

# Pandoc - 索引

> 来源：[https://pandoc.org](https://pandoc.org)

<main>

如果需要将一个标记格式的文件转换为另一个，Pandoc 是您的瑞士军刀。Pandoc 可以在以下格式之间进行转换：

(← = 从...转换为; → = 转换到...; ↔︎ = 双向转换)

文字处理器格式

↔︎ 微软 Word [docx](https://en.wikipedia.org/wiki/Office_Open_XML)

↔︎ 富文本格式 [RTF](https://en.wikipedia.org/wiki/Rich_Text_Format)

↔︎ OpenOffice/LibreOffice [ODT](https://en.wikipedia.org/wiki/OpenDocument)

交互式笔记本格式

↔︎ Jupyter 笔记本 ([ipynb](https://nbformat.readthedocs.io/en/latest/))

页面布局格式

→ [InDesign ICML](http://wwwimages.adobe.com/content/dam/acom/en/devnet/indesign/sdk/cs6/idml/idml-specification.pdf)

↔︎ [Typst](https://typst.app)

Wiki 标记格式

↔︎ [MediaWiki 标记](https://www.mediawiki.org/wiki/Help:Formatting)

↔︎ [DokuWiki 标记](https://www.dokuwiki.org/wiki:syntax)

← [TikiWiki 标记](https://doc.tiki.org/Wiki-Syntax-Text#The_Markup_Language_Wiki-Syntax)

← [TWiki 标记](http://twiki.org/cgi-bin/view/TWiki/TextFormattingRules)

← [Vimwiki 标记](https://vimwiki.github.io)

→ [XWiki 标记](https://www.xwiki.org/xwiki/bin/view/Documentation/UserGuide/Features/XWikiSyntax/)

→ [ZimWiki 标记](http://zim-wiki.org/manual/Help/Wiki_Syntax.html)

↔︎ [Jira wiki 标记](https://jira.atlassian.com/secure/WikiRendererHelpAction.jspa?section=all)

← [Creole](http://www.wikicreole.org/)

幻灯片演示格式

→ [LaTeX Beamer](https://ctan.org/pkg/beamer)

→ 微软 [PowerPoint](https://en.wikipedia.org/wiki/Microsoft_PowerPoint)

→ [Slidy](http://www.w3.org/Talks/Tools/Slidy)

→ [reveal.js](http://lab.hakim.se/reveal-js/)

→ [Slideous](http://goessner.net/articles/slideous/)

→ [S5](http://meyerweb.com/eric/tools/s5/)

→ [DZSlides](http://paulrouget.com/dzslides/)

数据格式

← [CSV](https://tools.ietf.org/html/rfc4180) 表格

← [TSV](https://www.iana.org/assignments/media-types/text/tab-separated-values) 表格

自定义格式

↔︎ 可以使用 [Lua](http://www.lua.org) 编写自定义的读取器（custom-readers.html）和写入器（custom-writers.html）。

PDF

→ 通过 `pdflatex`、`lualatex`、`xelatex`、`latexmk`、`tectonic`、`wkhtmltopdf`、`weasyprint`、`prince`、`pagedjs-cli`、`context` 或 `pdfroff`。

Pandoc 理解多种有用的 Markdown 语法扩展，包括文档元数据（标题、作者、日期）；脚注；表格；定义列表；上标和下标；删除线；增强的有序列表（起始编号和编号样式重要）；示例列表；带语法高亮的分隔代码块；智能引号、破折号和省略号；HTML 块内的 Markdown；以及行内 LaTeX。如果需要严格的 Markdown 兼容性，可以关闭所有这些扩展。

LaTeX 数学（甚至宏）可以在 markdown 文档中使用。提供了在 HTML 中渲染数学的多种不同方法，包括 MathJax 和转换为 MathML。LaTeX 数学会根据输出格式的需要转换为 unicode、本机 Word 方程对象、MathML 或 roff eqn。

Pandoc 包括一个强大的自动引用和文献系统。这意味着你可以写出一个引用如下

```
[see @doe99, pp. 33-35; also @smith04, ch. 1]
```

并且 pandoc 会根据数百种[CSL](http://citationstyles.org/)样式（包括脚注样式、数字样式和作者-年份样式）将其转换为格式正确的引用，并在文档末尾添加格式正确的参考文献。参考文献数据可以使用[BibTeX](http://tug.org/bibtex/)、[BibLaTeX](https://github.com/plk/biblatex)、[CSL JSON](https://citeproc-js.readthedocs.io/en/latest/csl-json/markup.html)或 CSL YAML格式。引用在每种输出格式中均有效。

有许多种方法可以定制 pandoc 以满足您的需求，包括模板系统和编写过滤器的强大系统。

Pandoc 包括一个 Haskell 库和一个独立的命令行程序。该库包括每种输入和输出格式的单独模块，因此添加新的输入或输出格式只需要添加新模块。

Pandoc 是自由软件，按照[GPL](http://www.gnu.org/copyleft/gpl.html)发布。 版权所有 2006–2022 [John MacFarlane](http://johnmacfarlane.net/)。

</main>
