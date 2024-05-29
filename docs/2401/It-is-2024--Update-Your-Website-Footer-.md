<!--yml
category: 未分类
date: 2024-05-27 14:26:22
-->

# It is 2024\. Update Your Website Footer.

> 来源：[https://updateyourfooter.com/](https://updateyourfooter.com/)

## JavaScript Snippets

This is pure JavaScript, meaning it will refresh the year browser-side, depending on the user's time settings. Just copy the below snippet and paste it where you want your dynamic text in the footer to appear.

```
<script type="text/javascript">
  document.write(new Date().getFullYear());
</script>
```

This will just give you:

```
            If you want a bit more information, here's a snippet you can customize:

```
&copy; 2010<script>new Date().getFullYear()>2010&&document.write("-"+new Date().getFullYear());</script>, Company.
```

            Which will give you:

```
© 2010, Company.
```

            Because JavaScript works on the client (the user's browser, that is), it is dependent on the user's settings. In most cases it will likely be what you'd expect. Here's more info on the
                [JavaScript Date Object](https://www.w3schools.com/jsref/jsref_obj_date.asp).
            You can even install the
                [Node package 'rainge'](https://github.com/dawsonbotsford/rainge) for having a truly automated way to keep your date stamps updated – you just set the year once, and rainge does the rest.

```

## PHP Snippets

Here's the same in PHP so you can do this server-side. Use Wordpress? Find the 'footer.php' in your Editor and add this there ([here are videos](https://www.youtube.com/results?search_query=edit+wordpress+footer) to help you with that). The below snippet will just show the current year:

```
<?php echo date("Y"); ?>
```

This will just give you:

```
            Or if you want more detail in your footer, use something like this:

```
&copy; <?php
  $fromYear = 2008;
  $thisYear = (int)date('Y');
  echo $fromYear . (($fromYear != $thisYear) ? '-' . $thisYear : '');?> Company.
```

            Or you can go for
                `($fromYear < $thisYear)` in the above to account for any time reversal (or a confused server, more likely). But either will give you:

```
© 2008 Company.
```

            Since PHP works on the server, it will display the year the server is currently in. Thus, if you look at this from Europe in the wee hours of 2024, you still see 2023 here.

```