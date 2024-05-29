<!--yml
category: 未分类
date: 2024-05-27 14:46:52
-->

# Sieve: Building blocks & infrastructure for AI

> 来源：[https://www.sievedata.com/](https://www.sievedata.com/)

```
transcriber = sieve.function.get("sieve/whisper")
translator = sieve.function.get("sieve/seamless_text2text")
tts = sieve.function.get("sieve/xtts")
lipsyncer = sieve.function.get("sieve/video_retalking")

transcript = transcriber.run(source_video)
translated_text = translator.run(text, "eng", language)
speech = tts.run(source_audio, language, translated_text)

dubbed_video = lipsyncer.run(source_video, speech)

```