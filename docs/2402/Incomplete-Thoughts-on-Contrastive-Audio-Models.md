<!--yml
category: 未分类
date: 2024-05-27 14:34:34
-->

# Incomplete Thoughts on Contrastive Audio Models

> 来源：[https://zelalabs.substack.com/p/incomplete-thoughts-on-contrastive](https://zelalabs.substack.com/p/incomplete-thoughts-on-contrastive)

I’ve been spending my time recently preparing various pieces of infrastructure to train an audio model contrastively on both text-audio and audio-audio.

I’ve been so heads down in the infrastructure that I haven’t been spending much time thinking about the potential novelties of the model itself, so here’s some incomplete thinking I have managed.

* * *

Audio fits strangely in the list of modalities. Like text, it’s strictly causal and the raw input to a model, a waveform is strictly one dimensional. But in contrast to text, audio has an extremely low information density, often sampled at between 24,000–48,000 frames per second. Accordingly, 75 English spoken words covers a sequence length of 720k frames of audio, or approximately 10,000x less dense, from a strictly semantic standpoint. In this characteristic, it sits more like an image - individual pixels are less important, and you can maintain high information transmission even with dramatic reductions in quality or precision. Probably it’s closest parallel then is video (unsurprisingly?) - causality, immense amounts of redundancy, etc.

Majority of work on Audio focuses on SST or TTS applications, which each have various techniques to deal with the interesting shape of this problem, and are currently focused heavily on finding ways to reshape it to fit the LM and Diffusion paradigms of today.  Contrastive models (think CLIP) though are pretty underrepresented here. There’s a few, like Betker’s CLVP, but most others are focused on either sounds rather than human voice.

* * *

So, what does happen when you summarise a chunk of audio down to a single vector? What might that vector encode?

Unlike other modalities, I’d argue voice has a unique property that *style* and *semantics* are heavily disjoint. Of course, characteristics like emotion cross this boundary, and the handling of semantics (text) and style (3s voice clone snippets) as being largely separate components has been an obvious shortcoming in most of the TTS world. But it is remarkable how well models can operate despite these two being conventionally separated, and places a strong prior on them being largely  two, slightly-overlapping, components.

Accordingly, we might expect a large amount of attention (both figuratively, and literally in the case of transformers) to be placed on the very early chunk of an audio segment whereby the stylistic properties like the speakers voice characteristics, background noise, audio quality, etc. are first encountered. Then, a more diffuse attention would be present for the rest of the sequence where the actual words spoken are propagated.

A another novel thing to consider is what might occur in samples where two speakers are present. Text embeddings and image embeddings tend to perform reasonably well at encoding information about multiple focal points and subjects, so we’d expect something similar. Could we perform something reminiscent of the classic word-to-vec style demos, where we take a vector containing speaker A and speaker B, subtract a vector containing only speaker A, and then use the resulting vector to find segments with strictly speaker B?

How might information like background noise, audio quality, etc, be encoded? Drawing from what we've seen with GANs, we might expect a contrastive model to strive for maximal disambiguation of samples, and given that I plan to train on neighbouring audio-audio pairs, it is possible the model learns to cheat by placing specific emphasis on the soundscape present outside of the human voice spectrum. I do hope to experiment with whether such model can be used to filter samples for such things as presence of music, background noise, etc, so it will be exciting to see.

* * *

That’s all for now. Obviously very incomplete thoughts here, early training runs are kicking off now so hopefully some findings to report back soon.