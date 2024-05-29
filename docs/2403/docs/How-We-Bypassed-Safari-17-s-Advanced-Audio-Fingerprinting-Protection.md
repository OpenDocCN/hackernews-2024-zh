<!--yml
category: 未分类
date: 2024-05-27 14:45:29
-->

# How We Bypassed Safari 17's Advanced Audio Fingerprinting Protection

> 来源：[https://fingerprint.com/blog/bypassing-safari-17-audio-fingerprinting-protection/](https://fingerprint.com/blog/bypassing-safari-17-audio-fingerprinting-protection/)

Did you know that browsers can produce audio files you can’t hear, and those audio files can be used to identify web visitors? Apple knows, and the company decided to fight the identification possibility in Safari 17, but their measures don’t fully work.

## [](#identifying-with-audio)Identifying with audio

The technique is called audio fingerprinting, and you can learn how it works in our [previous article](https://fingerprint.com/blog/audio-fingerprinting/). In a nutshell, audio fingerprinting uses the browser’s [Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) to render an audio signal with [OfflineAudioContext](https://developer.mozilla.org/en-US/docs/Web/API/OfflineAudioContext) interface, which then transforms into a single number by adding all audio signal samples together. The number is the fingerprint, also called “identifier”.

The audio identifier is stable, meaning it doesn’t change when you clear the cookies or go into incognito mode. This is the key feature of fingerprinting. However, the identifier is not very unique, and many users can have the same identifier.

Audio fingerprinting is a part of [FingerprintJS](https://github.com/fingerprintjs/fingerprintjs), our library with source code available on GitHub.

Fingerprinting is used to identify bad actors when they want to remain anonymous. For example, when they want to sign in to your account or use stolen credit card credentials. Fingerprinting can identify repeat bad actors, allowing you to prevent them from committing fraud. However, many people see it as a privacy violation and therefore don’t like it.

## [](#how-safari-17-breaks-audio-fingerprinting)How Safari 17 breaks audio fingerprinting

Apple [introduced](https://www.apple.com/au/newsroom/2023/06/apple-announces-powerful-new-privacy-and-security-features/) advanced fingerprinting protection in Safari 17\. Advanced fingerprinting protection aims to reduce fingerprinting accuracy by limiting available information or adding randomness.

By default, the advanced protection is enabled in private (incognito) mode and disabled in normal mode. It affects both desktop and mobile platforms. Advanced fingerprinting protection also affects [Screen API](https://developer.mozilla.org/en-US/docs/Web/API/Screen) and [Canvas API](https://developer.mozilla.org/en-US/docs/Glossary/Canvas), but we’ll focus only on Audio API in this article.

An audio signal produced with the Audio API is an array of numbers representing the signal amplitude at each moment of time (also called “audio samples”). When fingerprinting protection is on, Safari adds a random noise to every sample individually. A noised sample lies between `sample*(1-magnitude)` and `sample*(1+magnitude)`, and the distribution is [uniform](https://en.wikipedia.org/wiki/Continuous_uniform_distribution). This is how it’s [implemented in Safari](https://github.com/WebKit/WebKit/blob/167dc5118a3f6228a19df40e673ea0a6d03b9bec/Source/WebCore/platform/audio/AudioUtilities.cpp#L80):

```
void applyNoise(float* values, size_t numberOfElementsToProcess, float magnitude)
{
    WeakRandom generator;
    for (size_t i = 0; i < numberOfElementsToProcess; ++i)
        values[i] *= 1 + magnitude * (2 * generator.get() - 1);
}
```

*Note: Safari is being developed actively, so this and the other facts may be outdated when you read the article.*

All Audio API interfaces that allow reading the audio signal apply noise:

The noise is different every time it’s applied. As a result, the whole audio fingerprint changes every time it’s calculated in private mode. These changes cause the fingerprint to mismatch in normal and private modes. This breaks the stability; therefore, the fingerprint can’t be used for identification.

The fingerprint fluctuates between `124.03516` and `124.04545` in Safari 17 on an M1 MacBook Air. The difference is about 0.008%. That may not sound like much, but further on, we’ll explain why this is a huge difference.

## [](#how-we-bypass-safari-17s-advanced-fingerprinting-protection)How we bypass Safari 17’s advanced fingerprinting protection

The goal is to remove the noise added by Safari. To achieve this, we must improve [our fingerprinting algorithm](https://github.com/fingerprintjs/fingerprintjs/blob/a555410e925e0d6a4d548aa555b6945bd713e9cd/src/sources/audio.ts#L41) in 3 steps:

1.  Reduce the dispersion of the noise.
2.  Push browser identifier numbers farther apart.
3.  Round the fingerprint to remove the remaining noise.

We’ll call this improved algorithm “the new algorithm” throughout the article.

Steps 1 and 2 are necessary because Safari’s range of added noise is much bigger than the difference between fingerprints produced by various browsers. This table shows audio fingerprints produced by some browsers and the percent difference between them and the closest fingerprint from other browsers:

| Browser | Fingerprint | Difference from the closest browser |
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

As you can see, the smallest difference is 0.0000023%, much smaller than the Safari noise range (0.008%). Eliminating the noise Safari adds requires rounding down by 1 decimal place, but we can’t round to fewer than 6 decimal places. Otherwise, some browsers from the above table will be indistinguishable. In other words, the fingerprint will have poor uniqueness.

### [](#step-1-cutting-through-the-noise)Step 1: Cutting through the noise

The base idea for noise reduction is combining many separate audio fingerprints together. Each fingerprint is collected using the same algorithm, so the only difference is the noise added by the browser.

First, let’s take a closer look at the [fingerprinting algorithm](https://github.com/fingerprintjs/fingerprintjs/blob/a555410e925e0d6a4d548aa555b6945bd713e9cd/src/sources/audio.ts#L41). A fingerprint is a sum of 500 audio samples, and each audio sample is added with a random number with a [uniform distribution](https://en.wikipedia.org/wiki/Continuous_uniform_distribution). Therefore, according to [the central limit theorem](https://en.wikipedia.org/wiki/Central_limit_theorem), the fingerprint noise has a [normal distribution](https://en.wikipedia.org/wiki/Normal_distribution). The mean of the distribution is the un-noised fingerprint that we want to find.

The mean can be found using a large number of random samples (don’t confuse this with “audio samples”). This won’t be the true mean, but the more random samples there are, the more precise the result is. Uniform and normal distributions require different methods to find the mean:

*   For a uniform distribution, the most precise formula is `(min+max)/2`, where `min` and `max` are the minimum and the maximum random samples
*   For a normal distribution, the most precise formula is the average of all the random samples

Finding the mean of a uniform noise is much easier than a normally distributed noise. For a given precision, one needs much fewer samples in case of a uniform distribution to guess the mean. This JavaScript code proves the point in practice:

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

The old audio fingerprint is more computation-demanding and requires 100 times more fingerprint samples to reduce the noise. So, to reduce the noise in a reasonable time, we changed the fingerprinting algorithm to collect only one audio sample, which has a uniform noise distribution. The exact number of randomized samples needed depends on the rounding precision we need, which will be demonstrated later.

The algorithm change also means new fingerprints aren’t compatible with old fingerprints. Because of the rounding, the audio fingerprint will change, so sticking to the old fingerprint identifiers is useless. Note that you need to use [a special approach](https://github.com/fingerprintjs/fingerprintjs/blob/5ae80b7b946fcd824e35d033bc44e180334109f6/docs/version_policy.md#how-to-update-without-losing-the-identifiers) to switch from the old fingerprint to the new one without losing the visitor identities.

### [](#getting-many-noised-copies-of-the-same-audio-sample)Getting many noised copies of the same audio sample

One approach for getting multiple noised copies is [calling](https://github.com/fingerprintjs/fingerprintjs/blob/a555410e925e0d6a4d548aa555b6945bd713e9cd/src/sources/audio.ts#L77) `getChannelData` on the `AudioBuffer` instance many times. Remember that [`getChannelData`](https://developer.mozilla.org/en-US/docs/Web/API/AudioBuffer/getChannelData) returns the audio samples that the fingerprint is calculated from. This approach doesn’t work because noise is [applied once per](https://github.com/WebKit/WebKit/blob/167dc5118a3f6228a19df40e673ea0a6d03b9bec/Source/WebCore/Modules/webaudio/AudioBuffer.cpp#L336) each `AudioBuffer` instance, and `getChannelData` returns the same signal.

This can be circumvented by creating many `AudioBuffer` instances by running the whole audio signal generation process many times. For 6,000 noised samples, the fastest time is 7 seconds on an M1 MacBook. For 60,000, Safari can’t even finish the process. This is way too long for a fingerprint. Therefore, this approach is not viable.

A better approach is to make an `AudioBuffer` instance with the same audio signal on repeat:

1.  Render an audio signal as usual, but don’t call `getChannelData`, because it will [add noise](https://github.com/WebKit/WebKit/blob/167dc5118a3f6228a19df40e673ea0a6d03b9bec/Source/WebCore/Modules/webaudio/AudioBuffer.cpp#L167) to the signal.
2.  Create another `OfflineAudioContext` instance, much longer than the original instance. Use the original signal as a source using an [`AudioBufferSourceNode`](https://developer.mozilla.org/en-US/docs/Web/API/AudioBufferSourceNode).
3.  Make the `AudioBufferSourceNode` loop the needed piece of the original signal using `loop`, `loopStart`, and `loopEnd`. The piece can be as narrow as a single audio sample.
4.  Render the second (looped) audio context and call `getChannelData`. The resulting audio signal will consist of the original signal followed by the piece repeating until the end. Safari adds a noise after the looping, so the repeating copy has the same audio samples with different noise applied.

Here’s how to implement this approach:

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

Any number of noised sample copies can be produced in 2 audio renderings.

The code below combines the methods to denoise a single selected audio sample:

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

At this point, the noise is suppressed, not removed completely. The resulting number is still not stable, but the variance is smaller than that of a raw audio sample.

This table shows how the denoising precision and time in the above code snippet depend on the number of samples (`cloneCount`):

| Number of copies | Result range: (max-min)/min in 100 runs | Time on an M1 MacBook |
| --- | --- | --- |
| 2,048 | 0.194% | 2.0ms |
| 4,096 | 0.190% | 2.3ms |
| 8,192 | 0.000387% | 2.6ms |
| 16,384 | 0.0000988% | 2.9ms |
| 32,768 | 0.0000411% | 4.0ms |
| 65,536 | 0.0000123% | 4.1ms |
| 131,072 | 0.00000823% | 5.2ms |
| 262,144 | 0% (the ultimate precision) | 7.5ms |
| 524,288 | 0% | 11.9ms |
| 1,048,576 | 0% | 20.5ms |

### [](#step-2-push-browser-identifier-numbers-farther-apart)Step 2: Push browser identifier numbers farther apart

The times shown in the previous table can be 100 times longer on low-end devices or heavy webpages. The performance of the fingerprinting is important, so the fewer copies there are, the better. However, fewer copies mean bigger result dispersion, so it’s necessary to increase the difference between the audio samples in browsers too. These differences can be achieved by changing the base signal.

### [](#audio-nodes-with-heavy-distortion)Audio nodes with heavy distortion

After hours of experimenting with all the built-in audio nodes, we found an audio signal generator that gives a much bigger audio sample variance between browsers. The generator is a chain of audio nodes:

1.  The initial signal is produced by a square [OscillatorNode](https://developer.mozilla.org/en-US/docs/Web/API/OscillatorNode).
2.  Then, the signal goes through a [DynamicsCompressorNode](https://developer.mozilla.org/en-US/docs/Web/API/DynamicsCompressorNode).
3.  Finally, the signal is processed by a [BiquadFilterNode](https://developer.mozilla.org/en-US/docs/Web/API/BiquadFilterNode) of type “allpass”.

It is not necessary to know what the audio nodes do in detail. They can be treated as black boxes.

The audio sample number 3396 of the signal has the biggest difference between browsers. The number 3396 was found by simply comparing all samples of the audio signals in different browsers. This is how the signal is implemented in code:

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

The following table shows the resulting audio sample in different browsers:

| Browser | Audio sample | Difference from the closest browser |
| --- | --- | --- |
| MacBook Air 2020, Safari 17.0 | 0.000059806101489812136 | 0.0014% |
| iPhone 13, Safari 15.4 (BrowserStack) | 0.00005980528294458054 | 0.0014% |
| MacBook Pro 2015, Safari 16.6 | 0.00006429151108022779 | 0.046% |
| MacBook Pro 2015, Chrome 116 | 0.0000642621744191274 | 0.046% |
| MacBook Air 2020, Chrome 116 | 0.00006128742097644135 | 2.42% |
| Galaxy S23, Chrome 114 | 0.0000744499484426342 | 11.8% |
| Acer Chromebook 314, Chrome 117 | 0.00008321150380652398 | 10.53% |
| iPhone SE, Safari 13.1 | 0.00011335541057633236 | 26.6% |
| BrowserStack Windows 8, Firefox 67 | 0.00016917561879381537 | 0.0063% |
| MacBook Air 2020, Firefox 118 | 0.00016918622714001685 | 0.0040% |
| MacBook Pro 2015, Firefox 118 | 0.00016919305198825896 | 0.0040% |

Now the smallest difference is 0.0014%, which is much bigger than the original fingerprint (0.0000023%). It means that a much coarser denoising is possible.

### [](#step-3-round-the-result)Step 3: Round the result

The final step is stabilizing the sample to be used as a fingerprint. The sample range is small but still unstable, which is not suitable for FingerprintJS, because even a tiny change to the sample causes the whole fingerprint to change.

Rounding is a way to stabilize the audio sample. Usually, rounding preserves a specific number of digits after the decimal point. This is not a good choice in this case because, as mentioned at the beginning, the noise is not absolute; it’s relative to the audio sample number. Therefore, some number of *significant* digits should be preserved during rounding. Significant digits are all digits after the beginning zeros. You can see a rounding implementation on [GitHub](https://github.com/fingerprintjs/fingerprintjs/blob/c411aff111e5c79cdc37608d42632d4a66a8c1dc/src/sources/audio.ts#L244).

The table above shows that 5 significant digits are enough to tell the selected browsers apart. But since we can’t check all browsers in the world and can’t predict how they will change in the future, we use a few more digits, just in case.

The table below shows the number of audio sample copies needed to make the denoising result stable in private mode of Safari 17 after rounding with the given precision:

| Significant digits | # of copies | Time in Safari 17 on an M1 MacBook (warm) | Time in Chrome 116 on an M1 MacBook (warm) | Time in Chrome 114 on Pixel 2 (warm) |
| --- | --- | --- | --- | --- |
| 6 | 15,000 | 3ms | 4ms | 13ms |
| 7, but the last is the nearest multiple of 5 | 30,000 | 4ms | 5ms | 15ms |
| 7, but the last is the nearest even digit | 70,000 | 6ms | 7ms | 16ms |
| 7 and more | 400,000 | 12ms | 13ms | 34ms |

*A ”warm” browser is a browser that has run the given code before. A browser becomes “cold” when it’s restarted. A warm browser produces more stable time measurements.*

We chose “7, but the last is 0 or 5” as a good balance between the performance and uniqueness. We also increased the number of copies to 40,000 to increase stability.

The rounded number is the final new audio fingerprint that doesn’t change, even when Safari 17’s advanced fingerprinting protection is on. Uniqueness is an important property of fingerprinting. The new fingerprint has the same uniqueness as the old audio fingerprint.

## [](#performance)Performance

The following table shows the fingerprinting time on a blank page in warm browsers:

| Browser | Old fingerprint | New fingerprint |
| --- | --- | --- |
| MacBook Air 2020, Safari 17.3 | 2ms | 4ms |
| MacBook Air 2020, Chrome 120 | 5ms | 8ms |
| MacBook Air 2020, Firefox 121 | 6ms | 8ms |
| MacBook Pro 2015, Safari 16.6 | 4ms | 6ms |
| MacBook Pro 2015, Chrome 120 | 5ms | 7ms |
| MacBook Pro 2015, Firefox 121 | 5ms | 7ms |
| iPhone 13 mini, Safari 17.3 | 8ms | 12ms |
| iPhone SE, Safari 13.1 | 9ms | 17ms |
| Acer Chromebook 314, Chrome 120 | 7ms | 13ms |
| Galaxy S23, Chrome 120 | 6ms | 8ms |
| Galaxy J7 Prime, Chrome 120 | 33ms | 45ms |
| Pixel 3, Chrome 120 | 8ms | 15ms |
| BrowserStack Windows 11, Chrome 120 | 5ms | 7ms |
| BrowserStack Windows 11, Firefox 121 | 10ms | 18ms |

Compared to the old fingerprinting algorithm, the performance of the new one degrades 1.5–2 times. Even so, the new fingerprint algorithm takes little time to compute, even on low-end devices.

The browser delegates some work to the OfflineAudioRender thread, freeing the main thread. Therefore, the page stays responsive during most of the audio fingerprint calculation. Web Audio API is not available for [web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API), so we cannot calculate audio fingerprints there.

To improve the performance, the new fingerprint can be used only in Safari 17 while keeping the old algorithm in other browsers. Check whether the current browser is Safari 17 or newer using the user-agent string. Based on that, run either the old or the new fingerprinting algorithm.

## [](#how-it-works-in-privacy-focused-browsers)How it Works in Privacy-Focused Browsers

Privacy-focused browsers like Tor and Brave also make attempts to restrict audio fingerprinting. Web Audio API is completely disabled in Tor, so audio fingerprinting is [impossible](https://gitlab.torproject.org/legacy/trac/-/issues/21984). Brave, however, follows an approach like Safari 17 and adds noise to the audio signal. Our [previous article](https://fingerprint.com/blog/audio-fingerprinting/#brave) explains more about Brave’s audio fingerprinting protection.

The Brave noise has an important difference. While Safari adds a random noise for each audio sample individually, Brave makes a random multiplier (called “fudge factor”) once and uses it for all audio samples. That is, all audio samples are multiplied by the same number. The fudge factor persists within a page. It changes only in a new regular or incognito session.

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

No matter how many audio sample copies we make, the noise addition will be the same in every copy. The copies won’t be dispersed around the true (before noising) audio sample. Therefore, [the mathematical denoising method](#step-1-cutting-through-the-noise) doesn’t work.

Nevertheless, the Brave denoising method described in [the previous article](https://fingerprint.com/blog/audio-fingerprinting/#brave) still works. [The method for increasing the difference between fingerprints produced by browsers](#step-2-push-browser-identifier-numbers-farther-apart) can also increase the error tolerance.

## [](#usage-in-fingerprintjs)Usage in FingerprintJS

The new audio fingerprinting algorithm replaced the old one in FingerprintJS. It was first published in version [4.2.0](https://github.com/fingerprintjs/fingerprintjs/releases/tag/v4.2.0). You can see the full code for the audio fingerprint implementation [in our GitHub repository](https://github.com/fingerprintjs/fingerprintjs/blob/c411aff111e5c79cdc37608d42632d4a66a8c1dc/src/sources/audio.ts).

Audio fingerprinting is one of the many signals our [source-available library](https://github.com/fingerprintjs/fingerprintjs) uses to generate a browser fingerprint. However, we do not blindly incorporate every signal available in the browser. Instead, we analyze the stability and uniqueness of each signal separately to determine their impact on fingerprint accuracy.

For audio fingerprinting, we found that the signal contributes only slightly to uniqueness but is highly stable, resulting in a slight net increase in fingerprint accuracy.

If you want to learn more about Fingerprint join us on [Discord](https://discord.gg/39EpE2neBg) or reach out to us at [oss-support@fingerprint.com](mailto:oss-support@fingerprint.com) for support using FingerprintJS.