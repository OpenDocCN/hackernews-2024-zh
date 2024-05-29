<!--yml
category: 未分类
date: 2024-05-27 14:31:32
-->

# We don't need a DAC - ESP32 PDM Audio - by Chris Greening

> 来源：[https://atomic14.substack.com/p/esp32-s3-no-dac](https://atomic14.substack.com/p/esp32-s3-no-dac)

So, there’s no DAC on the ESP32-S3.

You would think this would be a bit of a downer if if you want to get audio out and use an analog amplifier.

But it’s actually surprisingly easy to output **P**ulse **D**ensity **M**odulated audio using Sigma Delta Modulation on the ESP32 and you can recover the audio signal by low pass filtering it - an RC filter can be sufficient for this. And that’s what I’m using in the video.

Though the Espressif docs do suggest a much better active filter.

It’s pretty interesting to look at a PDM signal and view it in the frequency domain. Here’s a piece of audio along with it’s spectrogram:

And here’s a simulated PDM version of the original audio. The sample rate of the PDM data is just over 1MHz and I’m showing the spectrogram from 0 to 500KHz

The PDM data just goes from -1 to 1 and changes density depending on the value of the original signal.

If we just look at the lower 8KHz of the spectrum then we can see what looks like our original signal.

So to recover the original audio we just apply a low pass filter - and hey presto, we have our original audio signal back!

So how to do we do this on the ESP32?

There’s a couple of options available to us.

The [example code](https://github.com/espressif/esp-idf/tree/b4268c874a4cf8fcf7c0c4153cffb76ad2ddda4e/examples/peripherals/sigma_delta/sdm_dac) from Espressif suggests using a timer to output each sample using the **sigmadelta_set_duty** (you could also just use plain old PWM as well). This does work for audio data, and I’ve got some simple [sample code](https://github.com/atomic14/esp32-pdm-audio/blob/main/src/audio_output/PDMTimerOuput.cpp) that will do it, but it’s not very efficient - we’re constantly interrupted by a timer to send out the next sample. There’s also quite a lot of code required if you want to stream samples out from some other source.

A much better way is to use the I2S peripheral which can also output PDM data. There are two annoying things with this which are highlighted in the timing diagram from the docs.

The first issue is that it always wants to output a left and right channel. This is a bit awkward if we just want to feed the PDM signal straight into an analog audio amplifier or headphones. But we can get around this by just outputting the same value for both the left and right channels.

The second issue is that it always wants to output a clock signal - we don’t really need this. My workaround for this was to just assign the clock to IO45 or IO46 - on the S3 you can’t really use these pins for much as they are strapping pins and it’s best to just leave them alone. But you can use them for outputs once the ESP32 has started up.

There are “proper” PDM amplifier ICs that will take this signal - for example the [MAX98358](https://www.analog.com/media/en/technical-documentation/data-sheets/max98358.pdf) or the [SSM2537](https://www.analog.com/media/en/technical-documentation/data-sheets/SSM2537.pdf).

This all works surprisingly well, you can drive headphones directly from the PDM signal and most analog amplifiers will take the combined stereo PDM signal and will have a low enough bandwidth that they’ll just work.

You can even just drive a speaker with a really simple half or full bridge and get reasonable audio out (though it may be quite noisy).

Have a watch of the video and let me know what you think.