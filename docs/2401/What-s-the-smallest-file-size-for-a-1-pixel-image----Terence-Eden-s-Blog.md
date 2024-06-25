<!--yml

类别：未分类

日期：2024-05-27 14:34:00

-->

# 1像素图像的最小文件大小是多少？- Terence Eden的博客

> 来源：[https://shkspr.mobi/blog/2024/01/whats-the-smallest-file-size-for-a-1-pixel-image/](https://shkspr.mobi/blog/2024/01/whats-the-smallest-file-size-for-a-1-pixel-image/)

有很多新的图像压缩格式。它们擅长于将大型、复杂的图片算法性地减小到较小的文件大小。我见过的所有比较都显示它们在压缩大文件方面的出色表现。

我想走另一条路。现代编解码器在处理*微小*文件时有多好？

使用GIMP，我创建了一个单个的白色像素图像，并将其保存为PNG。然后我使用[Squoosh](https://squoosh.app)使用不同的编码选项将其转换为各种现代格式。这是我找到的结果：

这被设计为“最小可*查看*图像”。我喜欢Stoyan Stefanov的“最小可用无图像图像src”。它创建了一个42字节的SVG，其中不包含图像数据 - 所以我想看看如果你制作一个可显示的图像会发生什么。

一些需要注意的重要事项：

+   老的图像格式如BMP和GIF比AVIF等新格式要小。

+   一些压缩选项以意想不到的方式使文件变大。有损的WebP比无损版本还要*大*。

+   同样，增加对AVIF的努力也可能导致较大的文件大小。

+   [JPEG XL](https://en.wikipedia.org/wiki/JPEG_XL)和[QOI](https://qoiformat.org/)目前尚未得到主流浏览器的支持。

+   [AVIF具有相当长且复杂的标头](https://aomediacodec.github.io/av1-avif/#image-item-properties) - 这对于大图片来说是合理的，但对于较小的图片来说会使其变得臃肿。

+   使用[Brotli](https://github.com/google/brotli)，可以进一步压缩AVIF（203字节），JPG（69字节），BMP（63字节）和ICO（30字节）文件。

+   如果你仍然需要一个[Spacer.gif](https://en.wikipedia.org/wiki/Spacer_GIF)，那么WebP是最小的文件！

这是我给你的挑战 - 你能做得更好吗？你能找到一个可*查看*图像的最小文件大小是多少？

* * *
