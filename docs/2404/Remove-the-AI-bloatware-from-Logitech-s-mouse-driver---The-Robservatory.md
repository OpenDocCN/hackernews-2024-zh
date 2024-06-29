<!--yml

category: 未分类

date: 2024-05-27 13:34:19

-->

# 从Logitech鼠标驱动程序中删除AI浮夸功能 – The Robservatory

> 来源：[https://robservatory.com/remove-the-ai-bloatware-from-logitechs-mouse-driver/](https://robservatory.com/remove-the-ai-bloatware-from-logitechs-mouse-driver/)

我非常喜欢Logitech的Mac [MX Keys键盘](https://robservatory.com/review-logitech-mx-keys-for-mac/) 和 [MX Master鼠标](https://robservatory.com/review-logitech-mx-master-2s-mouse/)（尽管我现在已经更新到第三代鼠标）。总的来说，他们的软件也相当不错。

但最近的更新为鼠标添加了“AI提示生成器”功能，这简直是垃圾——我并不是说它很差，因为我从未尝试过。它之所以是垃圾，是因为没有理由让我的鼠标需要一个连接到按钮的AI提示生成器。更糟糕的是，正如[Stephen Hackett发现的](https://512pixels.net/2024/04/ai-overlay-tmp-home-folder-mac-os/)，它会在你的主文件夹顶层创建一个名为`ai_overlay_tmp`的丑陋文件夹。

幸运的是，当Stephen在Mastodon上[发布此事](https://mastodon.social/@ismh@eworld.social/112320597255841194)时，用户@flipneus [提供了解决方案](https://mastodon.social/@flipneus/112321297224942175)。以防那篇帖子消失，这里是内容：

在Finder中，打开顶层Library → Application Support文件夹，然后转到Logitech → LogiOptionsPlus，并在你喜欢的纯文本编辑器中打开*app_permission.json*。在最后一个`}`之前的行后添加逗号，然后添加以下行：

```
 "aipromptbuilder": {
  "value": false
 }
}
```

完成后，文件末尾应如下所示（尽管你的命令可能有所不同）：

```
...
  },
 "backlight": {
  "value": true
 },
 "aipromptbuilder": {
  "value": false
 }
}
```

重要的是在（在我的文件中）与背光相关的部分后添加逗号。编辑完成后保存文件，并重新启动。

重新启动后，你可以删除`ai_overlay_tmp`文件夹——这样Logi Options+应用程序中将不再有AI生成器选项。（另外，Stephen指出你可以使用[SteerMouse](https://plentycom.jp/en/steermouse/index.html)来为Logitech的按钮编程。）

谢谢你，Stephen和@flipneus！
