<!--yml

分类：未分类

日期：2024年5月27日14:26:22

-->

# 现在是2024年。更新您的网站页脚。

> 来源：[https://updateyourfooter.com/](https://updateyourfooter.com/)

## JavaScript代码片段

这是纯JavaScript，意味着它将根据用户的时间设置在浏览器端刷新年份。只需复制下面的代码片段，粘贴到您希望在页脚中显示动态文本的位置。

```
<script type="text/javascript">
  document.write(new Date().getFullYear());
</script>
```

这将只给您：

```
            If you want a bit more information, here's a snippet you can customize:

```

&copy; 2010<script>new Date().getFullYear()>2010&&document.write("-"+new Date().getFullYear());</script>, 公司。

```

            Which will give you:

```

© 2010, 公司。

```

            Because JavaScript works on the client (the user's browser, that is), it is dependent on the user's settings. In most cases it will likely be what you'd expect. Here's more info on the
                [JavaScript Date Object](https://www.w3schools.com/jsref/jsref_obj_date.asp).
            You can even install the
                [Node package 'rainge'](https://github.com/dawsonbotsford/rainge) for having a truly automated way to keep your date stamps updated – you just set the year once, and rainge does the rest.

```

## PHP代码片段

这是PHP版本，因此您可以在服务器端执行此操作。使用WordPress吗？在编辑器中找到“footer.php”，并在那里添加这个代码片段（[这里有视频](https://www.youtube.com/results?search_query=edit+wordpress+footer)可帮助您）。下面的代码片段将只显示当前年份：

```
<?php echo date("Y"); ?>
```

这将只给您：

```
            Or if you want more detail in your footer, use something like this:

```

&copy; <?php

$fromYear = 2008;

$thisYear = (int)date('Y');

echo $fromYear . (($fromYear != $thisYear) ? '-' . $thisYear : '');?> 公司。

```

            Or you can go for
                `($fromYear < $thisYear)` in the above to account for any time reversal (or a confused server, more likely). But either will give you:

```

© 2008 公司。

```

            Since PHP works on the server, it will display the year the server is currently in. Thus, if you look at this from Europe in the wee hours of 2024, you still see 2023 here.

```
