<!--yml

category: 未分类

date: 2024-05-27 14:45:29

-->

# 如何绕过 Safari 17 的高级音频指纹识别保护

> 来源：[https://fingerprint.com/blog/bypassing-safari-17-audio-fingerprinting-protection/](https://fingerprint.com/blog/bypassing-safari-17-audio-fingerprinting-protection/)

你知道浏览器可以生成你听不到的音频文件，并且这些音频文件可以用来识别网站访客吗？苹果知道，并且公司决定在 Safari 17 中抵制这种识别可能性，但他们的措施并没有完全奏效。

## [](#identifying-with-audio)使用音频进行识别

这种技术被称为音频指纹识别，你可以在我们的[先前文章](https://fingerprint.com/blog/audio-fingerprinting/)中了解它的工作原理。简而言之，音频指纹识别利用浏览器的[音频 API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)通过[OfflineAudioContext](https://developer.mozilla.org/en-US/docs/Web/API/OfflineAudioContext)接口渲染音频信号，然后通过将所有音频信号样本相加转换为一个单一数字。这个数字就是指纹，也称为“标识符”。

音频标识符是稳定的，这意味着在清除 cookies 或进入隐身模式时不会更改。这是指纹识别的关键特性。然而，这个标识符并不是非常独特，很多用户可能会有相同的标识符。

音频指纹识别是[FingerprintJS](https://github.com/fingerprintjs/fingerprintjs)的一部分，我们的库源代码可以在 GitHub 上找到。

当想要保持匿名时，指纹识别用于识别不良行为者。例如，当他们想要登录您的帐户或使用窃取的信用卡凭据时。指纹识别可以识别重复的不良行为者，帮助您防止他们进行欺诈行为。然而，很多人认为这是一种侵犯隐私的行为，因此不喜欢它。

## [](#how-safari-17-breaks-audio-fingerprinting)Safari 17 如何破解音频指纹识别

苹果在 Safari 17 中引入了[高级指纹保护](https://www.apple.com/au/newsroom/2023/06/apple-announces-powerful-new-privacy-and-security-features/)。高级指纹保护旨在通过限制可用信息或增加随机性来降低指纹识别的准确性。

默认情况下，高级保护在隐私（隐身）模式下启用，在普通模式下禁用。它影响桌面和移动平台。高级指纹保护还影响[Screen API](https://developer.mozilla.org/en-US/docs/Web/API/Screen)和[Canvas API](https://developer.mozilla.org/en-US/docs/Glossary/Canvas)，但本文将重点放在音频 API 上。

使用音频API生成的音频信号是一个数字数组，表示每个时刻的信号振幅（也称为“音频样本”）。当开启指纹保护时，Safari会为每个样本单独添加随机噪音。一个带噪音的样本位于 `sample*(1-magnitude)` 和 `sample*(1+magnitude)` 之间，并且分布是 [均匀的](https://zh.wikipedia.org/wiki/%E5%9D%87%E5%8C%80%E5%88%86%E5%B8%83)。这是Safari的 [实现方式](https://github.com/WebKit/WebKit/blob/167dc5118a3f6228a19df40e673ea0a6d03b9bec/Source/WebCore/platform/audio/AudioUtilities.cpp#L80)：

```
void applyNoise(float* values, size_t numberOfElementsToProcess, float magnitude)
{
    WeakRandom generator;
    for (size_t i = 0; i < numberOfElementsToProcess; ++i)
        values[i] *= 1 + magnitude * (2 * generator.get() - 1);
}
```

*注：Safari正在积极开发中，因此当您阅读本文时，这些事实可能已过时。*

所有允许读取音频信号的音频API接口都会应用噪音：

每次应用时噪音都不同。因此，在私密模式下计算时，整个音频指纹都会发生变化。这些变化导致指纹在正常和私密模式下不匹配。这破坏了稳定性；因此，指纹不能用于识别。

在M1 MacBook Air上，Safari 17中的音频指纹波动在 `124.03516` 和 `124.04545` 之间。差异大约为0.008%。这听起来可能不多，但接下来我们将解释为什么这是一个巨大的差异。

## [](#how-we-bypass-safari-17s-advanced-fingerprinting-protection)我们如何绕过Safari 17的先进指纹保护

目标是消除Safari添加的噪音。为了实现这一目标，我们必须在三个步骤中改进 [我们的指纹算法](https://github.com/fingerprintjs/fingerprintjs/blob/a555410e925e0d6a4d548aa555b6945bd713e9cd/src/sources/audio.ts#L41)：

1.  减少噪音的分散度。

1.  将浏览器识别号码推远。

1.  将指纹四舍五入以去除剩余的噪音。

本文中将称此改进的算法为“新算法”。

步骤1和步骤2是必要的，因为Safari添加的噪音范围要比各种浏览器产生的指纹之间的差异大得多。以下表格显示了一些浏览器生成的音频指纹及其与其他浏览器最接近的指纹的百分比差异：

| 浏览器 | 指纹 | 与最接近浏览器的差异 |
| --- | --- | --- |
| MacBook Air 2020, Safari 17.0 | 124.04345259929687 | 0.0000023% |
| MacBook Pro 2015, Safari 16.6 | 124.04345808873768 | 0.0000044% |
| iPhone SE, Safari 13.1 | 35.10893253237009 | 1.8% |
| MacBook Air 2020, Chrome 116 | 124.04344968475198 | 0.0000023% |
| MacBook Pro 2015, Chrome 116 | 124.04347657808103 | 0.000015% |
| Galaxy S23, Chrome 114 | 124.08072766105033 | 0.030% |
| MacBook Pro 2015, Firefox 118 | 35.749968223273754 | 0.0000055% |
| MacBook Air 2020, Firefox 118 | 35.74996626004577 | 0.0000055% |
| BrowserStack Windows 8, Firefox 67 | 35.7383295930922 | 0.033% |

正如您所见，最小差异为0.0000023%，远小于Safari的噪音范围（0.008%）。消除Safari添加的噪音需要向下舍入到小数点后1位，但我们不能舍入到少于6位小数。换句话说，指纹的唯一性将会降低。

### [](#step-1-cutting-through-the-noise)第一步：穿透噪音

噪音减少的基本思路是将许多单独的音频指纹组合在一起。每个指纹都是使用相同的算法收集的，因此唯一的区别是浏览器添加的噪音。

首先，让我们更仔细地查看[指纹算法](https://github.com/fingerprintjs/fingerprintjs/blob/a555410e925e0d6a4d548aa555b6945bd713e9cd/src/sources/audio.ts#L41)。一个指纹是500个音频样本的总和，每个音频样本都加上一个[均匀分布](https://en.wikipedia.org/wiki/Continuous_uniform_distribution)的随机数。因此，根据[中心极限定理](https://en.wikipedia.org/wiki/Central_limit_theorem)，指纹的噪音具有[正态分布](https://en.wikipedia.org/wiki/Normal_distribution)。分布的均值是我们要找到的未加噪音的指纹。

均值可以通过大量的随机样本找到（不要与“音频样本”混淆）。这不会是真实的均值，但随着随机样本数量的增加，结果会更加精确。均匀分布和正态分布需要不同的方法来找到均值：

+   对于均匀分布，最精确的公式是`(min+max)/2`，其中`min`和`max`是最小和最大的随机样本。

+   对于正态分布，最精确的公式是所有随机样本的平均值。

找到均匀噪音的均值比正态分布的噪音要容易得多。对于给定的精度，在均匀分布的情况下，需要的样本数量要少得多才能猜出均值。这段JavaScript代码在实践中证明了这一点：

```
const sessionCount = 1000
const desiredMaxError = 0.005

const uniformRandom = (mean, variance) => {
  width = Math.sqrt(12 * variance)
  shift = mean - width / 2
  return () => Math.random() * width + shift
}
const normalRandom = (mean, variance) => {

  const pi2 = Math.PI * 2
  const sigma = Math.sqrt(variance)
  return () => Math.sqrt(-2 * Math.log(Math.random())) * Math.cos(pi2 * Math.random()) * sigma + mean
}

const averageMeanFind = samples => samples.reduce((a, b) => a + b) / samples.length
const midRangeMeanFind = samples => {
  let min = samples[0]
  let max = samples[0]
  for (let i = 1; i < samples.length; ++i) {
    if (samples[i] < min) {
      min = samples[i]
    } else if (samples[i] > max) {
      max = samples[i]
    }
  }
  return (min + max) / 2
}

const findAdequateSampleCount = (makeRandom, findMean) => {
  const mean = 0
  const variance = 1
  const random = makeRandom(mean, variance)

  sampleCountLoop:
  for (let sampleCount = 2; sampleCount < 1e7; sampleCount *= 2) {
    for (let session = 0; session < sessionCount; ++session) {
      const samples = [...Array(sampleCount)].map(random)
      const foundMean = findMean(samples)
      if (Math.abs(mean - foundMean) > desiredMaxError) {
        continue sampleCountLoop
      }
    }
    return sampleCount
  }

  return 'Too much time to compute'
}

console.log('Normal needs samples', findAdequateSampleCount(normalRandom, averageMeanFind))
console.log('Uniform needs samples', findAdequateSampleCount(uniformRandom, midRangeMeanFind)) 
```

旧的音频指纹需要更多的计算，并且需要100倍的指纹样本来减少噪音。因此，为了在合理的时间内减少噪音，我们改变了指纹算法，仅收集具有均匀噪音分布的一个音频样本。需要的随机样本数量取决于我们需要的四舍五入精度，稍后将进行演示。

算法的改变也意味着新的指纹与旧的指纹不兼容。由于四舍五入，音频指纹将发生变化，因此坚持使用旧的指纹标识是无用的。请注意，您需要使用[特殊方法](https://github.com/fingerprintjs/fingerprintjs/blob/5ae80b7b946fcd824e35d033bc44e180334109f6/docs/version_policy.md#how-to-update-without-losing-the-identifiers)，以在不丢失访问者身份的情况下从旧指纹切换到新指纹。

### [](#getting-many-noised-copies-of-the-same-audio-sample)获取相同音频样本的多个带噪音的副本

获得多个带噪音的副本的一个方法是在`AudioBuffer`实例上多次调用[`getChannelData`](https://developer.mozilla.org/en-US/docs/Web/API/AudioBuffer/getChannelData)。请记住，[`getChannelData`](https://developer.mozilla.org/en-US/docs/Web/API/AudioBuffer/getChannelData)返回计算指纹所需的音频样本。这种方法不起作用，因为每个`AudioBuffer`实例只应用一次噪音，并且`getChannelData`返回相同的信号。

这可以通过多次运行整个音频信号生成过程创建许多`AudioBuffer`实例来实现。对于6000个带噪样本，M1 MacBook上最快的时间为7秒。对于60,000个样本，Safari甚至无法完成处理。这对于指纹来说时间太长了。因此，这种方法不可行。

更好的方法是通过重复使用相同音频信号创建`AudioBuffer`实例：

1.  渲染音频信号与往常一样，但不要调用`getChannelData`，因为这会给信号[添加噪音](https://github.com/WebKit/WebKit/blob/167dc5118a3f6228a19df40e673ea0a6d03b9bec/Source/WebCore/Modules/webaudio/AudioBuffer.cpp#L167)。

1.  创建另一个比原始实例长得多的`OfflineAudioContext`实例。使用[`AudioBufferSourceNode`](https://developer.mozilla.org/en-US/docs/Web/API/AudioBufferSourceNode)将原始信号作为源。

1.  使用`loop`、`loopStart`和`loopEnd`使`AudioBufferSourceNode`循环原始信号的所需部分。

1.  渲染第二个（循环的）音频上下文并调用`getChannelData`。结果音频信号将由原始信号和重复片段组成，直至结束。Safari在循环后添加了噪音，因此重复副本具有相同的音频样本，但应用了不同的噪音。

下面是实施此方法的方式：

```
const sampleRate = 44100

getClonedPieces()

async function getClonedPieces() {
  const pieceLength = 500 
  const cloneCount = 1000

  const baseSignal = await getBaseSignal()
  const loopStart = baseSignal.length - pieceLength

  const context = new OfflineAudioContext(1, loopStart + cloneCount * pieceLength, sampleRate)
  const sourceNode = context.createBufferSource()
  sourceNode.buffer = baseSignal
  sourceNode.loop = true
  sourceNode.loopStart = (baseSignal.length - pieceLength) / sampleRate
  sourceNode.loopEnd = baseSignal.length / sampleRate
  sourceNode.connect(context.destination)
  sourceNode.start()

  const signalBuffer = await renderAudio(context)
  const clones = signalBuffer.getChannelData(0).subarray(loopStart)

  console.log(clones)
}

async function getBaseSignal() {
  const context = new OfflineAudioContext(1, 5000, sampleRate)

  return await renderAudio(context)
}

function renderAudio(context) {

} 
```

2个音频渲染中可以生成任意数量的带噪音的样本副本。

以下代码结合了几种方法来去噪单个选定的音频样本：

```
const sampleRate = 44100

console.log(denoiseAudioSample(10000))

async function denoiseAudioSample(cloneCount) {

  const baseSignal = await getBaseSignal()

  const context = new OfflineAudioContext(1, baseSignal.length - 1 + cloneCount, sampleRate)
  const sourceNode = context.createBufferSource()
  sourceNode.buffer = baseSignal
  sourceNode.loop = true
  sourceNode.loopStart = (baseSignal.length - pieceLength) / sampleRate
  sourceNode.loopEnd = baseSignal.length / sampleRate
  sourceNode.connect(context.destination)
  sourceNode.start()

  const signalBuffer = await renderAudio(context)

  return getMiddle(signalBuffer.getChannelData(0).subarray(baseSignal.length - 1))
}

async function getBaseSignal() {
  const context = new OfflineAudioContext(1, 5000, sampleRate)

  return await renderAudio(context)
}

function renderAudio(context) {

}

function getMiddle(samples) {
  let min = samples[0]
  let max = samples[0]
  for (let i = 1; i < samples.length; ++i) {
    if (samples[i] < min) {
      min = samples[i]
    } else if (samples[i] > max) {
      max = samples[i]
    }
  }
  return (min + max) / 2
} 
```

此时，噪音被抑制，但并未完全消除。结果数字仍然不稳定，但变化幅度小于原始音频样本的变化幅度。

此表显示了上述代码片段中去噪的精度和时间如何取决于样本数（`cloneCount`）：

| 复制数量 | 结果范围：(最大-最小)/最小（在100次运行中） | M1 MacBook上的时间 |
| --- | --- | --- |
| 2,048 | 0.194% | 2.0ms |
| 4,096 | 0.190% | 2.3ms |
| 8,192 | 0.000387% | 2.6ms |
| 16,384 | 0.0000988% | 2.9ms |
| 32,768 | 0.0000411% | 4.0ms |
| 65,536 | 0.0000123% | 4.1ms |
| 131,072 | 0.00000823% | 5.2ms |
| 262,144 | 0%（最终精度） | 7.5ms |
| 524,288 | 0% | 11.9ms |
| 1,048,576 | 0% | 20.5ms |

### [](#step-2-push-browser-identifier-numbers-farther-apart)步骤 2：将浏览器标识号码推得更远

前表中显示的时间在低端设备或重页面上可能要长100倍。指纹的性能很重要，因此复制越少越好。然而，复制越少意味着结果分散越大，因此也需要增加浏览器中音频样本的差异。可以通过改变基础信号来实现这些差异。

### [](#audio-nodes-with-heavy-distortion)带有严重失真的音频节点

经过几个小时的实验尝试所有内置音频节点后，我们找到了一个能在不同浏览器间产生更大音频样本变化的音频信号生成器。该生成器是一串音频节点：

1.  最初的信号由方形[OscillatorNode](https://developer.mozilla.org/en-US/docs/Web/API/OscillatorNode)生成。

1.  接下来，信号经过[DynamicsCompressorNode](https://developer.mozilla.org/en-US/docs/Web/API/DynamicsCompressorNode)。

1.  最终，信号由类型为“allpass”的[BiquadFilterNode](https://developer.mozilla.org/en-US/docs/Web/API/BiquadFilterNode)处理。

无需详细了解音频节点的工作原理。它们可以被视为黑盒子。

信号的第3396号音频样本在不同浏览器间有最大的差异。3396号是通过简单比较不同浏览器中音频信号的所有样本找到的。以下是代码实现的信号：

```
async function getBaseSignal() {
  const context = new OfflineAudioContext(1, 3396, 44100)

  const oscillator = context.createOscillator()
  oscillator.type = 'square'
  oscillator.frequency.value = 1000

  const compressor = context.createDynamicsCompressor()
  compressor.threshold.value = -70
  compressor.knee.value = 40
  compressor.ratio.value = 12
  compressor.attack.value = 0
  compressor.release.value = 0.25

  const filter = context.createBiquadFilter()
  filter.type = 'allpass'
  filter.frequency.value = 5.239622852977861
  filter.Q.value = 0.1

  oscillator.connect(compressor)
  compressor.connect(filter)
  filter.connect(context.destination)
  oscillator.start(0)

  return await renderAudio(context)
}

function renderAudio(context) {

}

const audioSample = (await getBaseSignal()).getChannelData(0).at(-1)
```

下表显示了不同浏览器中生成的音频样本：

| Browser | Audio sample | Difference from the closest browser |
| --- | --- | --- |
| MacBook Air 2020, Safari 17.0 | 0.000059806101489812136 | 0.0014% |
| iPhone 13, Safari 15.4（BrowserStack） | 0.00005980528294458054 | 0.0014% |
| MacBook Pro 2015, Safari 16.6 | 0.00006429151108022779 | 0.046% |
| MacBook Pro 2015, Chrome 116 | 0.0000642621744191274 | 0.046% |
| MacBook Air 2020, Chrome 116 | 0.00006128742097644135 | 2.42% |
| Galaxy S23, Chrome 114 | 0.0000744499484426342 | 11.8% |
| Acer Chromebook 314, Chrome 117 | 0.00008321150380652398 | 10.53% |
| iPhone SE, Safari 13.1 | 0.00011335541057633236 | 26.6% |
| BrowserStack Windows 8, Firefox 67 | 0.00016917561879381537 | 0.0063% |
| MacBook Air 2020, Firefox 118 | 0.00016918622714001685 | 0.0040% |
| MacBook Pro 2015, Firefox 118 | 0.00016919305198825896 | 0.0040% |

现在最小的差异是0.0014%，比原始指纹（0.0000023%）大得多。这意味着可以进行更粗略的去噪处理。

### [](#step-3-round-the-result)步骤 3：将结果四舍五入

最后一步是稳定样本以用作指纹。样本范围小但仍不稳定，不适合FingerprintJS，因为即使对样本进行微小更改也会导致整个指纹改变。

舍入是稳定音频样本的一种方式。通常，舍入会保留小数点后特定数量的数字。在这种情况下，这不是一个好选择，因为正如开头提到的，噪声不是绝对的，而是相对于音频样本数。因此，在舍入过程中应该保留一定数量的*有效*数字。有效数字是从开始的零后面的所有数字。你可以在[GitHub](https://github.com/fingerprintjs/fingerprintjs/blob/c411aff111e5c79cdc37608d42632d4a66a8c1dc/src/sources/audio.ts#L244)上看到一个舍入的实现。

上表显示，保留5个有效数字足以区分选定的浏览器。但由于我们无法检查世界上所有的浏览器，也无法预测它们未来的变化，所以我们使用了更多的数字，以防万一。

下表显示了在给定精度下舍入后在 Safari 17 的私密模式中使去噪结果稳定所需的音频样本复制次数：

| 有效数字 | 复制次数 | 在 M1 MacBook 上的 Safari 17 中的时间（热启动） | 在 M1 MacBook 上的 Chrome 116 中的时间（热启动） | 在 Pixel 2 上的 Chrome 114 中的时间（热启动） |
| --- | --- | --- | --- | --- |
| 6 | 15,000 | 3ms | 4ms | 13ms |
| 7，但最后一个是最接近的5的倍数 | 30,000 | 4ms | 5ms | 15ms |
| 7，但最后一个是最接近的偶数 | 70,000 | 6ms | 7ms | 16ms |
| 7 及以上 | 400,000 | 12ms | 13ms | 34ms |

*“热”浏览器是指之前已经运行过给定代码的浏览器。当浏览器重新启动时，它就会“冷却”。热浏览器产生更稳定的时间测量结果。*

我们选择了“7，但最后一个是0或5”作为性能和独特性之间的良好平衡点。我们还增加了复制次数到40,000，以增加稳定性。

舍入后的数字是最终的新音频指纹，即使在启用 Safari 17 的高级指纹保护时也不会改变。独特性是指纹技术的重要特性。新指纹与旧音频指纹具有相同的独特性。

## [](#performance)性能

下表显示了在热浏览器中空白页面上的指纹时间：

| 浏览器 | 旧指纹 | 新指纹 |
| --- | --- | --- |
| MacBook Air 2020，Safari 17.3 | 2ms | 4ms |
| MacBook Air 2020，Chrome 120 | 5ms | 8ms |
| MacBook Air 2020，Firefox 121 | 6ms | 8ms |
| MacBook Pro 2015，Safari 16.6 | 4ms | 6ms |
| MacBook Pro 2015，Chrome 120 | 5ms | 7ms |
| MacBook Pro 2015，Firefox 121 | 5ms | 7ms |
| iPhone 13 mini，Safari 17.3 | 8ms | 12ms |
| iPhone SE，Safari 13.1 | 9ms | 17ms |
| Acer Chromebook 314，Chrome 120 | 7ms | 13ms |
| Galaxy S23，Chrome 120 | 6ms | 8ms |
| Galaxy J7 Prime，Chrome 120 | 33ms | 45ms |
| Pixel 3，Chrome 120 | 8ms | 15ms |
| BrowserStack Windows 11，Chrome 120 | 5ms | 7ms |
| BrowserStack Windows 11，Firefox 121 | 10ms | 18ms |

与旧的指纹算法相比，新算法的性能下降了1.5至2倍。尽管如此，新的指纹算法在低端设备上计算时间很短。

浏览器将一些工作委托给 OfflineAudioRender 线程，从而释放主线程。因此，在大部分音频指纹计算期间页面保持响应。Web Audio API 对于[Web Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API)不可用，因此我们无法在那里计算音频指纹。

为了提高性能，在 Safari 17 中使用新指纹算法，同时在其他浏览器中保留旧算法。通过检查用户代理字符串确定当前浏览器是否为 Safari 17 或更新版本。基于此，运行旧版或新版的指纹算法。

## [](#how-it-works-in-privacy-focused-browsers)隐私保护浏览器中的工作原理

Tor 和 Brave 等注重隐私的浏览器也尝试限制音频指纹识别。在 Tor 中，Web Audio API 完全被禁用，因此音频指纹[不可能](https://gitlab.torproject.org/legacy/trac/-/issues/21984)实现。然而，Brave 采用类似 Safari 17 的方法，向音频信号添加噪声。我们的[先前文章](https://fingerprint.com/blog/audio-fingerprinting/#brave)详细解释了 Brave 的音频指纹保护。

Brave 的噪音有一个重要区别。虽然 Safari 为每个音频样本单独添加随机噪声，但 Brave 只生成一个随机乘数（称为“fudge factor”），然后将其用于所有音频样本。也就是说，所有音频样本都乘以相同的数字。这个“fudge factor”在页面内保持不变，在新的常规或无痕会话中才会改变。

```
 const audioSignal = new Float32Array()
const magnitude = 0.001

for (let i = 0; i < audioSignal.length; i++) {
  audioSignal[i] *= random(1 - magnitude, 1 + magnitude)
}

const fudgeFactor = random(1 - magnitude, 1 + magnitude)
for (let i = 0; i < audioSignal.length; i++) {
  audioSignal[i] *= fudgeFactor
} 
```

无论我们制作多少音频样本的副本，每个副本中添加的噪声都是相同的。副本不会在真实（在加噪之前）音频样本周围分散。因此，[数学去噪方法](#step-1-cutting-through-the-noise)不起作用。

尽管如此，《先前的文章》中描述的 Brave 降噪方法仍然有效。[增加浏览器生成指纹之间差异的方法](#step-2-push-browser-identifier-numbers-farther-apart)也可以增加误差容忍度。

## [](#usage-in-fingerprintjs)在 FingerprintJS 中的应用

新的音频指纹算法取代了 FingerprintJS 中的旧算法。该算法首次发布于版本 [4.2.0](https://github.com/fingerprintjs/fingerprintjs/releases/tag/v4.2.0)。您可以在我们的 GitHub 仓库中查看音频指纹实现的完整代码 [audio.ts](https://github.com/fingerprintjs/fingerprintjs/blob/c411aff111e5c79cdc37608d42632d4a66a8c1dc/src/sources/audio.ts)。

音频指纹技术是我们的一个[开源库](https://github.com/fingerprintjs/fingerprintjs)用来生成浏览器指纹的多个信号之一。然而，我们并不是盲目地将浏览器中的每个信号都纳入考虑。相反，我们分析每个信号的稳定性和独特性，单独确定它们对指纹准确性的影响。

对于音频指纹技术，我们发现其信号对唯一性的贡献略微，但非常稳定，从而略微增加了指纹准确性。

如果您想了解更多关于指纹技术的信息，请加入我们的[Discord](https://discord.gg/39EpE2neBg)，或者通过[oss-support@fingerprint.com](mailto:oss-support@fingerprint.com)联系我们获取关于使用FingerprintJS的支持。
