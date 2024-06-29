<!--yml

category: 未分类

date: 2024-05-27 13:03:56

-->

# -   通过卫星观测的日食 - kieranhealy.org

> 来源：[https://kieranhealy.org/blog/archives/2024/04/09/the-eclipse-via-satellite/](https://kieranhealy.org/blog/archives/2024/04/09/the-eclipse-via-satellite/)

-   昨天通过[GOES-East](https://www.star.nesdis.noaa.gov/GOES/)天气卫星拍摄的日食。我用`wget`抓取了完整的地球色彩全圆盘 JPG 图像，并用`ffmpeg`拼接在一起。

<./eclipse_sm.mp4>

<./eclipse_sm.mov>

<./eclipse_sm.webm>

-   顺便说一句，如果你想知道为什么云层在东方夜幕降临时仍然可见，那是因为地球色彩图像是“真彩”白天部分和夜间部分的多光谱红外图像的组合。GEOS-East和West捕获我想是十六个基本波长带，可以合成成各种组合图像。你可以在[GOES图像查看器](https://www.star.nesdis.noaa.gov/GOES/fulldisk.php?sat=G16)看到所有这些图像，以及各种组合图像。
