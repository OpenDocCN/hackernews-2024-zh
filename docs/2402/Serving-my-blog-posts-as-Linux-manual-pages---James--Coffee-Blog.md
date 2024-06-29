<!--yml

分类：未分类

日期：2024-05-29 13:28:43

-->

# 将我的博客文章作为Linux手册页服务 | James' Coffee Blog

> 来源：[https://jamesg.blog/2024/02/29/linux-manual-pages/](https://jamesg.blog/2024/02/29/linux-manual-pages/)

*目标受众：如果你对Linux和/或Linux手册页感兴趣，或者喜欢阅读关于奇特编程项目的文章，你可能会喜欢这篇文章。*

Linux计算机预装了描述如何使用特定命令的手册页。通过在终端中键入`man <command>`，可以阅读这些页面。例如，您可以获取`tac`命令的手册页，该命令通过使用命令`man tac`打印出文件的底部到顶部。您安装的一些命令行软件也会添加手册页。

Linux手册页使用`roff`语法格式化，您可以使用它标记文档。`roff`是Unix的第一个命令行排版软件，在贝尔实验室开发。本周早些时候，由于建立的火花但没有特定的想法，我开始思考Linux手册页。我能否将我的博客文章作为Linux手册页提供？这里面蕴含着一次冒险。

TL;DR：您可以通过以下HTTP请求请求博客文章的Linux手册页版本：

```
 curl -sL -H "Accept: text/roff" {{ site.root_url }}/2024/02/28/programming-projects/ > post.page && man ./post.page 
```

## 设计系统：内容协商

我有一个关于我想让这个系统如何运行的想法。我希望用户能够使用内容协商请求博客文章的`roff`版本，内容协商是HTTP的一部分，它允许您指定您希望文件以何种格式发送。例如，您可以请求一个带有`Accept: image/png`的图像。这告诉服务器，如果可能的话，它应该发送一个PNG文件。内容协商有很多复杂性。您可以提供您可以接受的内容类型列表，并指定服务器应尝试返回它们的顺序

但是！那是另一个深入研究的话题。这里重要的是，应用程序可以通过HTTP头部向服务器请求特定格式的内容。

通过内容协商，如果用户发送一个`Accept`头部，我可以路由请求。如果用户请求一个`text/roff`文档，我可以返回一个可以用`man`命令打开的手册页。

## 编写手册页

手册页使用`roff`语法，因此我需要我的博客文章以这种格式存在。为此，我更新了我的网站，为每篇博客文章生成了`man`页面。我用来生成手册页的模板如下：

```
 .TH jamesg.blog 1 "" "jamesg.blog"
.SH TITLE
...
.SH AUTHOR
James' Coffee Blog ({{ site.root_url }})
.SH PUBLISHED
...
.SH POST
...
.SH URL
... 
```

在这里，我设置了一个带有我的域名的标题，并创建了五个部分：标题、作者、发布日期、文章内容和文章的URL。原始内容是Markdown格式。这在手册中并不总是很好，因为有时间距可能会有所偏差。但是，Markdown比HTML更可读，并且导致信息丢失较少（即标题与段落之间没有其他区别，除了它们在自己的行上）。

现在我有了：

1.  正确格式化的手册页，以及；

1.  知道内容协商可以允许某人请求一个手册页面。

现在到了谜题的最后一块：使用内容协商促进对手册页面的请求。

## 请求一个手册页

你可以使用以下命令请求此网站博客文章的`roff`格式：

```
 curl -sL -H "Accept: text/roff" {{ site.root_url }}/2024/02/28/programming-projects/ > post.page 
```

然后，您可以将结果作为Linux手册页打开：

```
 man ./post.page 
```

让我们谈谈这是如何工作的！

当浏览器请求`{{ site.root_url }}/2024/02/19/personal-website-ideas/`时，它请求页面的HTML版本。在上面的`curl`命令中，命令请求`text/roff`版本。我在我的NGINX配置中添加了几行文本，以更改服务器在请求博客文章的`text/roff`时如何响应。

首先，我在`/etc/nginx/nginx.conf`文件中声明了一些变量，让我在识别特定内容类型时引发标志：

```
 map $uri $redirect_suffix {
    ~^/(.*)/$   $1;
    default     "";
}

map $http_accept $redirect_location {
    default "";
    "~^text/roff" 1;
} 
```

您可以添加多个不同的重定向位置，但我只需要两个：默认和我的自定义`text/roff`规则。

在我的站点NGINX配置（位于`/etc/nginx/sites-enabled`文件夹中的文件）中，我使用以下代码来处理请求，如果请求一个`roff`页面：

```
 server {
				...
        location / {
          if ($redirect_location = 1) {
              rewrite ^/(.*)/$ /$1.man last;
          }
        	...
        }
} 
```

在这里，我说：获取一个URL，并在末尾添加`.man`，删除尾部斜杠，只要设置了`Accept: text/roff`头。这告诉NGINX从`.man`文件中读取，而不是与我的网站上每篇文章相关联的`index.html`文件。

也就是说，您现在可以将此网站上的博客文章作为Linux手册页面阅读。这是对在NGINX中使用内容协商进行有趣调查的回顾，并提醒我们从命令行界面到现代排版软件和HTML的技术进步。

*感谢[Todd](https://toddpresta.com/)为设置我的NGINX配置提供的指导。Todd的帮助是真诚地感激！*

[在Hacker News上分享此文章](https://news.ycombinator.com/submitlink?u=/https://jamesg.blog/2024/02/29/linux-manual-pages/&t=Serving%20my%20blog%20posts%20as%20Linux%20manual%20pages)。

[在Lobste.rs上分享此文章](https://lobste.rs/stories/new?url=/https://jamesg.blog/2024/02/29/linux-manual-pages/&title=Serving%20my%20blog%20posts%20as%20Linux%20manual%20pages)。
