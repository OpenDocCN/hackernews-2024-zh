<!--yml

category: 未分类

date: 2024-05-27 14:50:03

-->

# stable-audio-demo

> 来源：[https://stability-ai.github.io/stable-audio-demo/](https://stability-ai.github.io/stable-audio-demo/)

⚠️ *警告：本网站在Safari上可能无法正常运行。为了获得最佳体验，请使用Google Chrome。*

[`arXiv`](https://arxiv.org/abs/2402.04825): Stable Audio的论文

[`stable-audio-tools`](https://github.com/Stability-AI/stable-audio-tools): code to reproduce Stable Audio

[`stable-audio-metrics`](https://github.com/Stability-AI/stable-audio-metrics): code to evaluate Stable Audio

我们的模型可以以44.1kHz生成**可变长度和长形式的立体声音乐**：

| 生成的立体声音乐 | 提示 |
| --- | --- |

|

<audio/berlin-techno-rave-drum-machine-kick-ARP-synthesizer-dark-moody-hypnotic-evolving-135-bpm.wav>

您的浏览器不支持音频。 | 柏林技术派，狂欢，鼓机，踢鼓，ARP合成器，黑暗，阴郁，催眠，进化中，每分钟135拍。循环。 |

|

<audio/uplifting-acoustic-loop-120-bpm.wav>

您的浏览器不支持音频。 | 振奋的原声循环。120 BPM。 |

|

<audio/Disco,%20Driving%20Drum%20Machine,%20Synthesizer,%20Bass,%20Piano,%20Guitars,%20Instrumental,%20Clubby,%20Euphoric,%20Chicago,%20New%20York,%20115%20BPM.wav>

您的浏览器不支持音频。 | 狂欢，驾驶鼓机，合成器，低音，钢琴，吉他，器乐，俱乐部风格，欣快，芝加哥，纽约，每分钟115拍。 |

|

<audio/Calm%20meditation%20music%20to%20play%20in%20a%20spa%20lobby.wav>

您的浏览器不支持音频。 | 宁静冥想音乐，适合播放在温泉大堂。 |

|

<audio/drum%20solo.wav>

您的浏览器不支持音频。 | 鼓独奏。 |

与以往的先进模型不同，我们的模型可以生成**44.1kHz立体声音效**：

| 生成的立体声声音 | 提示 |
| --- | --- |

|

<audio/door-slam-high-quality-stereo.wav>

您的浏览器不支持音频。 | 关门声。高质量，立体声。 |

|

<audio/sports-car-passing-by-high-quality-stereo.wav>

您的浏览器不支持音频。 | 过路跑车。高质量，立体声。 |

|

<audio/motorbike-passing-by-high-quality-stereo.wav>

您的浏览器不支持音频。 | 骑摩托车路过。高质量，立体声。 |

|

<audio/fireworks-high-quality-stereo.wav>

您的浏览器不支持音频。 | 烟火。高质量，立体声。 |

|

<audio/reverberant-foot-steps-inside-a-large-rocky-cave-high-quality-stereo.wav>

您的浏览器不支持音频。 | 大岩洞内回响的脚步声。高质量，立体声。 |

请注意，本网站中的所有示例都是由相同的模型生成的，该模型可以以44.1kHz立体声生成**可变长度的音乐和音效**。我们在声音效果提示中附加“高质量，立体声”，通常是有帮助的。

## 长形式立体声音乐：与MusicCaps提示的最新技术比较

**提示**：此歌曲包含一个人在曼陀林上弹奏旋律，同时有更多人在吹口哨。然后曼陀林、电低音和原声吉他在较低音调上演奏一个简短的旋律，然后伴随长笛和打击乐进入下一个部分。这首歌可能是由表演音乐家在室外演奏。

| 我们的模型 | MusicGen-large | MusicGen-stereo | AudioLDM2 |
| --- | --- | --- | --- |
| *(立体声，44.1kHz)* | *(单声道，32kHz)* | *(立体声，32kHz)* | *(单声道，48kHz)* |
| --- | --- | --- | --- |

|

<audio/ZTVMsW1h3bI_stableaudio.wav>

您的浏览器不支持音频播放。 |

<audio/ZTVMsW1h3bI_musicgenlarge.wav>

您的浏览器不支持音频播放。 |

<audio/ZTVMsW1h3bI_musicgenstereo.wav>

您的浏览器不支持音频播放。 |

<audio/ZTVMsW1h3bI_audioldm248k_stereo.wav>

您的浏览器不支持音频播放。 |

**提示**：商业音乐以活泼的钢琴旋律和第一半循环中的军鼓声为特色。接着，出现了一个下降段落，由有力的“四打一”踢鼓节奏、闪烁的高音帽、拍手声、活泼的钢琴和广阔的合成器主旋律组成。听起来快乐、有趣、欣快而令人兴奋。

| 我们的模型 | MusicGen-large | MusicGen-stereo | AudioLDM2 |
| --- | --- | --- | --- |
| *(立体声，44.1kHz)* | *(单声道，32kHz)* | *(立体声，32kHz)* | *(单声道，48kHz)* |
| --- | --- | --- | --- |

|

<audio/ZK5M3DZejzk_stableaudio.wav>

您的浏览器不支持音频播放。 |

<audio/ZK5M3DZejzk_musicgenlarge.wav>

您的浏览器不支持音频播放。 |

<audio/ZK5M3DZejzk_musicgenstereo.wav>

您的浏览器不支持音频播放。 |

<audio/ZK5M3DZejzk_audioldm248k_stereo.wav>

您的浏览器不支持音频播放。 |

这些提示/音频用于我们在论文中报告的定性研究。

## 音效：与AudioCaps提示进行的最新技术比较

**提示**：点击声和喷嚏，然后逐渐加速的怠速引擎声。

| 模型 | Audiogen-medium | AudioLDM2 |
| --- | --- | --- |
| *(立体声，44.1kHz)* | *(单声道，32kHz)* | *(单声道，48kHz)* |
| --- | --- | --- |

|

<audio/103136_stableaudio_audio.wav>

您的浏览器不支持音频播放。 |

<audio/103136_audiogen_stereo.wav>

您的浏览器不支持音频播放。 |

<audio/103136_audioldm248k_stereo.wav>

您的浏览器不支持音频播放。 |

**提示**：鸟儿在大声啁啾。

| 模型 | Audiogen-medium | AudioLDM2 |
| --- | --- | --- |
| *(立体声，44.1kHz)* | *(单声道，32kHz)* | *(单声道，48kHz)* |
| --- | --- | --- |

|

<audio/37008_stableaudio_audio.wav>

您的浏览器不支持音频播放。 |

<audio/37008_audiogen_stereo.wav>

您的浏览器不支持音频播放。 |

<audio/37008_audioldm248k_stereo.wav>

您的浏览器不支持音频播放。 |

这些提示/音频用于我们在论文中报告的定性研究。请注意，从AudioCaps中随机选择的提示不需要大量的立体声移动，因此生成的音频相对非空间化。

## 自编码器：重构

这种比较对评估自编码器的音频保真度能力非常有用。在左边，我们有原始录音。在右边，我们将原始录音输入自编码器。请注意，自编码器的重建几乎是透明的，非常接近原始录音。

| 原始录音 | 自编码器重建 |
| --- | --- |

|

<audio/1197.flac>

您的浏览器不支持音频元素。 |

<audio/1197_ae.wav>

您的浏览器不支持音频元素。 |

|

<audio/1243.flac>

您的浏览器不支持音频元素。 |

<audio/1243_ae.wav>

您的浏览器不支持音频元素。 |

|

<audio/233076.flac>

您的浏览器不支持音频元素。 |

<audio/233076_ae.wav>

您的浏览器不支持音频元素。 |

|

<audio/451.flac>

您的浏览器不支持音频元素。 |

<audio/451_ae.wav>

您的浏览器不支持音频元素。 |

|

<audio/206251.flac>

您的浏览器不支持音频元素。 |

<audio/206251_ae.wav>

您的浏览器不支持音频元素。 |
