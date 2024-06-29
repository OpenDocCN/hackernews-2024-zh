<!--yml

类别：未分类

日期：2024年05月27日 13:04:37

-->

# TSAC：极低比特率音频压缩

> 来源：[https://bellard.org/tsac/](https://bellard.org/tsac/)

# TSAC：极低比特率音频压缩

TSAC 是一种音频压缩实用程序，可达到非常低的比特率，如单声道 5.5 kb/s 或立体声 7.5 kb/s，在44.1 kHz下具有良好的感知质量。因此，TSAC可以将一首3.5分钟的立体声歌曲压缩到192 KiB的文件中。

快速操作需要Nvidia GPU。也支持仅CPU，但速度较慢。

## 压缩结果

音频提取自[这里](https://listening-test.coresv.net/results.htm)。

Waiting（Pops）：

| 原始

<Waiting.wav>

| 立体声 7.26 kb/s

<Waiting_stereo.wav>

| 单声道 5.61 kb/s

<Waiting_mono.wav>

| 立体声 2.99 kb/s

<Waiting_stereo_q5.wav>

|

Greatest_Love_of_All_2min57（Pops）：

| 原始

<Greatest_Love_of_All_2min57.wav>

| 立体声 6.79 kb/s

<Greatest_Love_stereo.wav>

| 单声道 5.02 kb/s

<Greatest_Love_stereo.wav>

| 立体声 2.84 kb/s

<Greatest_Love_stereo_q5.wav>

|

9-有大而昂贵的车辆.441（Pops）：

| 原始

<9-Have-big-expensive-car.441.wav>

| 立体声 7.81 kb/s

<Have-big-expensive-car_stereo.wav>

| 单声道 5.91 kb/s

<Have-big-expensive-car_mono.wav>

| 立体声 3.25 kb/s

<Have-big-expensive-car_stereo_q5.wav>

|

21-classic.441（Classic）：

| 原始

<21-classic.441.wav>

| 立体声 6.21 kb/s

<classic_stereo.wav>

| 单声道 4.71 kb/s

<classic_mono.wav>

| 立体声 2.57 kb/s

<classic_stereo_q5.wav>

|

4-Sound-English-male.441（Voice）：

| 原始

<4-Sound-English-male.441.wav>

| 单声道 6.79 kb/s

<4-Sound-English-male.441_mono.wav>

| 单声道 3.74 kb/s

<4-Sound-English-male.441_mono_q5.wav>

| 单声道 2.18 kb/s

<4-Sound-English-male.441_mono_q3.wav>

|

## 下载

## 技术信息

+   `tsac`基于修改版的[Descript Audio Codec](https://github.com/descriptinc/descript-audio-codec)，扩展支持立体声以及Transformer模型以进一步增加压缩比。这两个模型的参数均量化为每个参数8位。

+   Transformer 模型以确定性和可重复的方式进行评估。因此，结果不依赖于确切的GPU或CPU型号，也不依赖于配置的线程数。这一关键点确保可以使用不同的硬件或软件配置解压缩压缩文件。

* * *

Fabrice Bellard - [https://bellard.org/](https://bellard.org)
